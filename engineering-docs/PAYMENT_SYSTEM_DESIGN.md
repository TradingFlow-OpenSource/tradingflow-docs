# TradingFlow 加密货币支付系统设计

> 本文档详细设计 TradingFlow 的加密货币支付系统，支持用户使用链上资产购买 Credits。

---

## 📋 目录

1. [系统概览](#系统概览)
2. [架构设计](#架构设计)
3. [数据库设计](#数据库设计)
4. [API 设计](#api-设计)
5. [实现细节](#实现细节)
6. [安全考虑](#安全考虑)
7. [测试方案](#测试方案)

---

## 系统概览

### 功能需求

1. **生成收款地址**：为每个用户在每条链上生成唯一的收款地址
2. **监听支付**：实时监听用户向收款地址的转账
3. **确认支付**：验证支付金额和确认数，确认后发放 Credits
4. **手动刷新**：用户可以手动触发支付状态检查
5. **自动检查**：系统定时检查待确认的支付订单
6. **多链支持**：第一期支持 Aptos，后续扩展其他链

### 核心流程

```
用户购买 Credits
  ↓
Control: 创建支付订单 → 返回收款地址和金额
  ↓
用户: 向收款地址转账
  ↓
Monitor: 监听链上交易 → 检测到转账
  ↓
Monitor: 验证金额和确认数 → 回调 Control
  ↓
Control: 发放 Credits → 更新用户余额
  ↓
前端: 显示支付成功 ✅
```

---

## 架构设计

### 系统组件

```
┌─────────────────┐
│  01_frontend    │
│   PricePage     │ ← 用户交互界面
└────────┬────────┘
         │ HTTP API
         ↓
┌─────────────────┐
│  02_control     │
│  Payment API    │ ← 订单管理、Credits 发放
└────────┬────────┘
         │ ①创建订单
         │ ④确认支付回调
         ↓
┌─────────────────┐
│  05_depot       │
│   Database      │ ← 存储订单和地址
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│  04_monitor     │
│  Payment Module │ ← 链上交互、交易监听
└─────────────────┘
         │ ②监听地址
         │ ③检测到转账
         ↓
    Blockchain
    (Aptos)
```

### 职责划分

| 组件 | 职责 |
|------|------|
| **Frontend** | 显示价格、收款地址、支付状态；手动刷新 |
| **Control** | 创建订单、管理订单状态、发放 Credits |
| **Monitor** | 生成收款地址、监听交易、验证支付 |
| **Depot** | 存储订单、地址、交易记录 |

---

## 数据库设计

### 表 1: `payment_addresses`

用户的收款地址表（每个用户每条链一个地址）

```sql
CREATE TABLE payment_addresses (
  id SERIAL PRIMARY KEY,
  user_id VARCHAR(255) NOT NULL,
  chain VARCHAR(50) NOT NULL,           -- 'aptos', 'flow_evm', 'ethereum', etc.
  address VARCHAR(255) NOT NULL UNIQUE, -- 链上收款地址
  private_key_encrypted TEXT,           -- 加密的私钥（可选，用于自动提现）
  derivation_path VARCHAR(100),         -- HD 钱包派生路径（如 m/44'/637'/0'/0/1）
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_checked_at TIMESTAMP,            -- 最后检查时间
  total_received DECIMAL(30, 8) DEFAULT 0, -- 累计收到的金额
  
  UNIQUE(user_id, chain),
  INDEX idx_user_chain (user_id, chain),
  INDEX idx_address (address),
  INDEX idx_last_checked (last_checked_at)
);
```

### 表 2: `payment_orders`

支付订单表

```sql
CREATE TABLE payment_orders (
  id SERIAL PRIMARY KEY,
  order_id VARCHAR(100) NOT NULL UNIQUE,  -- 订单号（如 'ORDER_20250108_ABC123'）
  user_id VARCHAR(255) NOT NULL,
  
  -- 订单信息
  credits_amount INT NOT NULL,            -- 购买的 Credits 数量
  payment_amount DECIMAL(30, 8) NOT NULL, -- 需要支付的金额
  payment_currency VARCHAR(20) NOT NULL,  -- 支付币种（'APT', 'USDC', 'ETH'）
  chain VARCHAR(50) NOT NULL,             -- 支付链（'aptos', 'flow_evm'）
  payment_address VARCHAR(255) NOT NULL,  -- 收款地址
  
  -- 订单状态
  status VARCHAR(50) NOT NULL DEFAULT 'pending', -- 'pending', 'paid', 'confirmed', 'expired', 'cancelled'
  
  -- 支付信息
  tx_hash VARCHAR(255),                   -- 支付交易哈希
  paid_amount DECIMAL(30, 8),             -- 实际支付的金额
  paid_at TIMESTAMP,                      -- 支付时间
  confirmed_at TIMESTAMP,                 -- 确认时间
  confirmations INT DEFAULT 0,            -- 确认数
  
  -- 时间信息
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expires_at TIMESTAMP,                   -- 订单过期时间（24小时后）
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- 元数据
  metadata JSON,                          -- 额外信息（如促销代码、备注等）
  
  INDEX idx_user (user_id),
  INDEX idx_order_id (order_id),
  INDEX idx_status (status),
  INDEX idx_payment_address (payment_address),
  INDEX idx_created_at (created_at),
  INDEX idx_expires_at (expires_at)
);
```

### 表 3: `payment_transactions`

支付交易记录表（用于审计和对账）

```sql
CREATE TABLE payment_transactions (
  id SERIAL PRIMARY KEY,
  order_id VARCHAR(100),                  -- 关联的订单ID（可为空，用于未匹配的交易）
  user_id VARCHAR(255),
  
  -- 交易信息
  chain VARCHAR(50) NOT NULL,
  tx_hash VARCHAR(255) NOT NULL UNIQUE,
  from_address VARCHAR(255) NOT NULL,
  to_address VARCHAR(255) NOT NULL,      -- 我们的收款地址
  amount DECIMAL(30, 8) NOT NULL,
  currency VARCHAR(20) NOT NULL,
  
  -- 区块信息
  block_number BIGINT,
  block_timestamp TIMESTAMP,
  confirmations INT DEFAULT 0,
  
  -- 状态
  status VARCHAR(50) NOT NULL DEFAULT 'pending', -- 'pending', 'confirmed', 'matched', 'refunded'
  matched_at TIMESTAMP,                   -- 匹配到订单的时间
  
  -- 时间信息
  detected_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_order (order_id),
  INDEX idx_tx_hash (tx_hash),
  INDEX idx_to_address (to_address),
  INDEX idx_status (status),
  INDEX idx_detected_at (detected_at)
);
```

### 表 4: `payment_pricing`

定价配置表

```sql
CREATE TABLE payment_pricing (
  id SERIAL PRIMARY KEY,
  chain VARCHAR(50) NOT NULL,
  currency VARCHAR(20) NOT NULL,           -- 'APT', 'USDC', 'ETH'
  credits_per_unit DECIMAL(20, 8) NOT NULL, -- 1 单位货币可购买的 Credits 数量
  min_amount DECIMAL(30, 8) NOT NULL,      -- 最小支付金额
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_timestamp,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE(chain, currency),
  INDEX idx_active (is_active)
);

-- 初始定价配置
INSERT INTO payment_pricing (chain, currency, credits_per_unit, min_amount) VALUES
('aptos', 'APT', 100, 1),      -- 1 APT = 100 Credits, 最少充 1 APT
('aptos', 'USDC', 10, 10);     -- 1 USDC = 10 Credits, 最少充 10 USDC
```

---

## API 设计

### 1. Control API（02_weather_control）

#### 1.1 创建支付订单

```
POST /api/v1/payment/orders
Authorization: Bearer <token>

Request:
{
  "creditsAmount": 1000,      // 要购买的 Credits 数量
  "chain": "aptos",           // 支付链
  "currency": "APT"           // 支付币种
}

Response (200 OK):
{
  "success": true,
  "data": {
    "orderId": "ORDER_20250108_ABC123",
    "creditsAmount": 1000,
    "paymentAmount": 10,      // 需要支付 10 APT
    "paymentCurrency": "APT",
    "chain": "aptos",
    "paymentAddress": "0x1234...5678",
    "expiresAt": "2025-01-09T18:00:00Z",
    "status": "pending"
  }
}
```

#### 1.2 查询支付订单

```
GET /api/v1/payment/orders/:orderId
Authorization: Bearer <token>

Response (200 OK):
{
  "success": true,
  "data": {
    "orderId": "ORDER_20250108_ABC123",
    "status": "confirmed",      // 'pending', 'paid', 'confirmed', 'expired'
    "creditsAmount": 1000,
    "paymentAmount": 10,
    "paidAmount": 10.5,         // 实际支付金额
    "txHash": "0xabcd...ef01",
    "confirmations": 15,
    "paidAt": "2025-01-08T18:05:00Z",
    "confirmedAt": "2025-01-08T18:06:00Z"
  }
}
```

#### 1.3 手动刷新订单状态

```
POST /api/v1/payment/orders/:orderId/refresh
Authorization: Bearer <token>

Response (200 OK):
{
  "success": true,
  "data": {
    "orderId": "ORDER_20250108_ABC123",
    "status": "confirmed",
    "updated": true
  }
}
```

#### 1.4 查询用户支付历史

```
GET /api/v1/payment/orders
Authorization: Bearer <token>
Query: ?page=1&limit=10&status=confirmed

Response (200 OK):
{
  "success": true,
  "data": {
    "orders": [...],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 25
    }
  }
}
```

#### 1.5 获取定价信息

```
GET /api/v1/payment/pricing

Response (200 OK):
{
  "success": true,
  "data": {
    "aptos": {
      "APT": {
        "creditsPerUnit": 100,
        "minAmount": 1,
        "currentPrice": 8.50  // 当前 APT 价格（USD）
      },
      "USDC": {
        "creditsPerUnit": 10,
        "minAmount": 10
      }
    }
  }
}
```

---

### 2. Monitor API（04_weather_monitor）

#### 2.1 生成收款地址

```
POST /payment/addresses
Body:
{
  "userId": "user123",
  "chain": "aptos"
}

Response (200 OK):
{
  "success": true,
  "data": {
    "userId": "user123",
    "chain": "aptos",
    "address": "0x1234...5678",
    "createdAt": "2025-01-08T18:00:00Z"
  }
}
```

#### 2.2 查询地址余额和交易

```
GET /payment/addresses/:address/balance?chain=aptos

Response (200 OK):
{
  "success": true,
  "data": {
    "address": "0x1234...5678",
    "chain": "aptos",
    "balance": {
      "APT": "15.5",
      "USDC": "100.0"
    },
    "transactions": [
      {
        "txHash": "0xabcd...ef01",
        "from": "0x9876...5432",
        "amount": "10.5",
        "currency": "APT",
        "timestamp": "2025-01-08T18:05:00Z",
        "confirmations": 15
      }
    ]
  }
}
```

#### 2.3 手动检查支付（内部API）

```
POST /payment/check
Body:
{
  "orderId": "ORDER_20250108_ABC123"
}

Response (200 OK):
{
  "success": true,
  "data": {
    "orderId": "ORDER_20250108_ABC123",
    "paymentDetected": true,
    "txHash": "0xabcd...ef01",
    "amount": "10.5",
    "confirmations": 15
  }
}
```

---

## 实现细节

### 1. 收款地址生成策略

**选项 A：HD 钱包派生** ✅ （推荐）

使用 BIP-44 标准从主种子派生子地址：

```
m/44'/637'/0'/0/0  → 用户1的 Aptos 地址
m/44'/637'/0'/0/1  → 用户2的 Aptos 地址
m/44'/637'/0'/0/n  → 用户n的 Aptos 地址
```

优点：
- 只需保管一个主种子
- 可以离线生成大量地址
- 符合行业标准

缺点：
- 需要安全保管主种子

**选项 B：独立生成** ❌

为每个用户生成独立的私钥。

优点：
- 实现简单

缺点：
- 需要保管大量私钥
- 不便于备份和恢复

**推荐实现**：使用选项 A（HD 钱包）

```typescript
// monitor/src/modules/payment/address_service.ts
import { Aptos, AptosConfig, Network, Account, Ed25519PrivateKey } from "@aptos-labs/ts-sdk";
import * as bip39 from "bip39";
import { derivePath } from "ed25519-hd-key";

const MASTER_SEED = process.env.PAYMENT_MASTER_SEED; // 从环境变量读取主种子

async function generateAptosAddress(userId: string): Promise<{address: string, derivationPath: string}> {
  // 1. 从用户ID生成索引
  const userIndex = hashUserIdToIndex(userId);
  
  // 2. 派生路径
  const path = `m/44'/637'/0'/0/${userIndex}`;
  
  // 3. 从主种子派生私钥
  const seed = bip39.mnemonicToSeedSync(MASTER_SEED);
  const derived = derivePath(path, seed.toString('hex'));
  
  // 4. 创建 Aptos 账户
  const privateKey = new Ed25519PrivateKey(derived.key);
  const account = Account.fromPrivateKey({ privateKey });
  
  return {
    address: account.accountAddress.toString(),
    derivationPath: path
  };
}
```

### 2. 交易监听机制

**方案：轮询 + 事件通知**

```typescript
// monitor/src/tasks/handle_payment_check.ts

async function checkPendingPayments() {
  // 1. 获取所有待支付的订单
  const pendingOrders = await getOrdersByStatus('pending');
  
  for (const order of pendingOrders) {
    // 2. 检查收款地址的交易
    const txs = await getAddressTransactions(order.paymentAddress, order.chain);
    
    for (const tx of txs) {
      // 3. 验证交易
      if (
        tx.amount >= order.paymentAmount &&
        tx.currency === order.paymentCurrency &&
        tx.confirmations >= REQUIRED_CONFIRMATIONS
      ) {
        // 4. 回调 Control 确认支付
        await notifyPaymentConfirmed({
          orderId: order.orderId,
          txHash: tx.hash,
          amount: tx.amount,
          confirmations: tx.confirmations
        });
      }
    }
  }
}

// 定时任务：每 30 秒检查一次
cron.schedule('*/30 * * * * *', checkPendingPayments);
```

### 3. Control 确认支付流程

```javascript
// control/src/packages/payment/services/PaymentService.js

async confirmPayment(orderId, txHash, amount) {
  // 1. 查询订单
  const order = await PaymentOrder.findOne({ orderId });
  
  // 2. 验证订单状态
  if (order.status !== 'pending' && order.status !== 'paid') {
    throw new Error('Order already processed');
  }
  
  // 3. 更新订单状态
  order.status = 'confirmed';
  order.txHash = txHash;
  order.paidAmount = amount;
  order.confirmedAt = new Date();
  await order.save();
  
  // 4. 发放 Credits
  await creditsService.addCredits(
    order.userId,
    order.creditsAmount,
    'payment',
    {
      orderId,
      txHash,
      chain: order.chain,
      currency: order.paymentCurrency
    }
  );
  
  // 5. 发送通知（可选）
  await notificationService.sendPaymentConfirmedEmail(order.userId, order);
  
  return order;
}
```

---

## 安全考虑

### 1. 主种子保护

```bash
# 环境变量设置（使用强加密）
PAYMENT_MASTER_SEED="<24个助记词>"

# 加密存储
# 使用 AWS KMS、HashiCorp Vault 或硬件安全模块(HSM)
```

### 2. 私钥加密存储

```typescript
// 如果需要存储私钥（用于自动提现），必须加密
import * as crypto from 'crypto';

function encryptPrivateKey(privateKey: string, masterKey: string): string {
  const cipher = crypto.createCipher('aes-256-gcm', masterKey);
  let encrypted = cipher.update(privateKey, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return encrypted;
}
```

### 3. 防止重放攻击

- 每个交易哈希只能使用一次
- 在 `payment_transactions` 表中记录所有已处理的交易
- 检查交易时验证 `tx_hash` 是否已存在

### 4. 订单过期机制

- 订单创建后 24 小时自动过期
- 定时任务清理过期订单
- 防止地址复用导致的混乱

---

## 测试方案

### 1. 单元测试

- 地址生成测试
- HD 钱包派生测试
- 金额验证测试
- 状态转换测试

### 2. 集成测试

- 创建订单 → 支付 → 确认流程
- 手动刷新功能
- 过期订单处理
- 多个订单并发支付

### 3. 测试网测试

- 使用 Aptos Testnet
- 创建测试账户
- 模拟真实支付流程
- 验证 Credits 发放

---

## 实现步骤

### Phase 1: 数据库和基础设施 ✅

1. 创建数据库表
2. 实现 weather_depot 数据库服务
3. 配置主种子环境变量

### Phase 2: Monitor 支付模块 ✅

1. 实现地址生成服务
2. 实现交易查询服务
3. 实现支付检查定时任务
4. 添加 API 路由

### Phase 3: Control 支付 API ✅

1. 实现订单创建接口
2. 实现订单查询接口
3. 实现支付确认逻辑
4. 实现 Credits 发放

### Phase 4: 前端集成 ✅

1. 更新 PricePage 显示定价
2. 实现创建订单流程
3. 显示收款地址和二维码
4. 实现手动刷新
5. 显示支付状态

### Phase 5: 测试和部署 ✅

1. 单元测试
2. 集成测试
3. Testnet 测试
4. 生产部署

---

## 注意事项

1. **确认数要求**：
   - Aptos: 至少 10 个确认
   - 以太坊: 至少 12 个确认

2. **金额容差**：
   - 允许用户多支付（如支付 10.5 APT 购买 10 APT 的订单）
   - 不允许少支付

3. **货币精度**：
   - Aptos APT: 8 位小数
   - USDC: 6 位小数

4. **订单唯一性**：
   - 一个交易只能匹配一个订单
   - 防止重复发放 Credits

5. **异常处理**：
   - 链 RPC 不可用
   - 交易确认延迟
   - 用户支付错误金额

---

最后更新：2025-01-08

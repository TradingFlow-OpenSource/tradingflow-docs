# Vault Node

Vault Node 是一个交易执行节点，用于连接和查询 TradingFlow 金库系统中的用户资产信息。节点支持多链查询，包括 Aptos 和 Flow EVM，可以获取金库余额、持仓组合和资产价值等信息。

---

## 节点信息

| 属性         | 值                |
| ------------ | ----------------- |
| **节点类型** | `vault_node`      |
| **显示名称** | Vault             |
| **节点分类** | Trade（交易执行） |
| **图标**     | 🏦 金库图标       |
| **句柄颜色** | Amber（橙色）     |

---

## 功能说明

Vault Node 连接到 TradingFlow 的去中心化金库系统，查询指定用户地址的资产持仓信息。节点自动计算资产价值，支持多链查询，并将结果传递给下游交易节点。

**主要用途：**

- 查询金库资产持仓和余额
- 获取多链资产价值信息
- 为交易节点提供资金来源
- 监控资产组合变化
- 计算总资产价值（USD）

**核心特性：**

- 🌐 **多链支持**：Aptos 和 Flow EVM
- 💰 **实时价格**：自动获取代币价格并计算资产价值
- 📊 **投资组合**：完整的持仓构成和比例信息
- 🔗 **链式传递**：将金库地址和链信息传递给下游交易节点
- 🔄 **自动刷新**：每次执行更新最新的资产信息

---

## 输入参数

### 参数列表

| 参数            | 类型   | 必填 | 默认值  | 说明                               |
| --------------- | ------ | ---- | ------- | ---------------------------------- |
| `chain`         | select | ✅   | `aptos` | 区块链网络                         |
| `vault_address` | text   | ✅   | -       | 金库地址或用户地址                 |
| `chain_id`      | number | ❌   | `545`   | Flow EVM 链 ID（仅 Flow EVM 需要） |

### chain 参数

**支持的区块链：**

| 链           | 值         | 说明              |
| ------------ | ---------- | ----------------- |
| **Aptos**    | `aptos`    | Aptos 主网/测试网 |
| **Flow EVM** | `flow_evm` | Flow EVM 测试网   |

**选择指南：**

- 如果您的金库在 Aptos 链上，选择 `aptos`
- 如果您的金库在 Flow EVM 链上，选择 `flow_evm`

### vault_address 参数

**格式要求：**

| 链       | 地址格式                  | 示例                                                                 |
| -------- | ------------------------- | -------------------------------------------------------------------- |
| Aptos    | `0x` 开头的十六进制地址   | `0x6a1a233e9c3871fc3719e4238bf61218c98d3b89fa5c2a37c87e7f6d60e07292` |
| Flow EVM | `0x` 开头的以太坊格式地址 | `0x1234567890123456789012345678901234567890`                         |

**说明：**

- 这是您在 TradingFlow 系统中创建的金库地址
- 可以在 Windmill 页面的 Vaults 标签页查看您的金库地址
- 地址必须是有效的已创建金库

### chain_id 参数

**仅用于 Flow EVM：**

| 链 ID | 网络         | 说明                      |
| ----- | ------------ | ------------------------- |
| `545` | Flow Testnet | Flow EVM 测试网（默认）   |
| `747` | Flow Mainnet | Flow EVM 主网（即将支持） |

---

## 输出参数

### 输出列表

| 输出 ID | 显示名称 | 数据类型 | 说明                                         |
| ------- | -------- | -------- | -------------------------------------------- |
| vault   | Vault    | object   | 完整的金库信息对象，包含链、地址、余额和持仓 |

### vault 输出

**数据类型：** object

**完整数据结构：**

```typescript
{
  success: boolean;              // 查询是否成功
  vault_address: string;         // 金库地址
  chain: string;                 // 区块链网络
  chain_id: number;             // 链 ID（Flow EVM）
  holdings: Array<{             // 持仓列表
    token_address: string;      // 代币合约地址
    amount: string;             // 原始数量（最小单位）
    symbol: string;             // 代币符号
    decimals: number;           // 小数位数
    price_usd: number;          // 单价（USD）
    value_usd: number;          // 持仓价值（USD）
    amount_human_readable: number; // 可读数量
    percentage?: number;        // 占比（仅 Flow EVM）
  }>;
  total_value_usd: number;      // 总价值（USD）
  token_count?: number;         // 代币种类数（Flow EVM）
  timestamp?: string;           // 查询时间戳
  message: string;              // 状态消息
}
```

**说明：**

- vault 输出包含完整的金库信息，包括链、地址、余额、持仓等
- 下游节点可以从这个对象中提取所需的任何字段
- 包含实时价格和计算后的资产价值

---

## 信号传输

### 发送的信号

**信号句柄：** `vault_balance`

**信号类型：** `SignalType.VAULT_INFO`

**信号负载：** 完整的金库信息对象（见上文 vault_balance 输出结构）

### 信号流向示例

```
Vault Node
    ↓ vault (金库信息)
    ↓ chain (链信息)
    ↓ vault_address (地址)
    ↓
Swap/Buy/Sell Node (使用金库执行交易)
```

---

## 工作流程

### 节点执行流程

```
1. 参数验证
   ├─ 检查 chain 是否支持（aptos/flow_evm）
   ├─ 检查 vault_address 是否提供
   └─ Flow EVM 检查 chain_id（默认 545）

2. 初始化服务
   ├─ Aptos: AptosVaultService
   └─ Flow EVM: FlowEvmVaultService

3. 查询持仓
   ├─ Aptos: get_investor_holdings()
   └─ Flow EVM: get_vault_info_with_prices()

4. 价格计算
   ├─ 获取每个代币的 USD 价格
   ├─ 计算每个持仓的价值
   └─ 汇总总资产价值

5. 格式化输出
   ├─ 统一 Aptos 和 Flow EVM 的数据格式
   ├─ 添加可读数量和百分比
   └─ 构建完整的输出对象

6. 发送信号
   └─ 发送 VAULT_INFO 信号给下游节点

7. 完成
   └─ 设置节点状态为 COMPLETED
```

### 多链处理差异

| 特性         | Aptos                 | Flow EVM                   |
| ------------ | --------------------- | -------------------------- |
| **服务类**   | AptosVaultService     | FlowEvmVaultService        |
| **查询方法** | get_investor_holdings | get_vault_info_with_prices |
| **价格获取** | 节点内获取            | 服务返回包含价格           |
| **持仓格式** | 需要转换小数位        | 已转换为可读格式           |
| **额外字段** | timestamp             | percentage, token_count    |

---

## 信号传输

### 发送的信号

**信号句柄：** `vault`

**信号类型：** `SignalType.VAULT_INFO`

**信号负载：** 完整的金库信息对象（见上文 vault 输出结构）

### 信号流向示例

```
Vault Node
    ↓ vault (包含 chain, vault_address, holdings, total_value_usd 等)
    ↓
Swap/Buy/Sell Node (从 vault 对象中提取所需字段执行交易)
```

---

## 使用示例

### 示例 1：Aptos 金库查询

**场景：** 查询 Aptos 链上的金库持仓，然后执行 Swap 操作。

**Vault Node 配置：**

```json
{
  "chain": "aptos",
  "vault_address": "0x6a1a233e9c3871fc3719e4238bf61218c98d3b89fa5c2a37c87e7f6d60e07292"
}
```

**工作流结构：**

```
Vault Node (Aptos)
    ↓ vault (持仓信息: 100 APT, 50 USDT)
    ↓
Swap Node (USDT → APT)
    ↓ trade_receipt
Telegram Sender Node (发送交易通知)
```

**vault 输出示例：**

```json
{
  "success": true,
  "vault_address": "0x6a1a...",
  "chain": "aptos",
  "holdings": [
    {
      "symbol": "APT",
      "amount_human_readable": 100.0,
      "value_usd": 725.0
    },
    {
      "symbol": "USDT",
      "amount_human_readable": 50.0,
      "value_usd": 50.0
    }
  ],
  "total_value_usd": 775.0
}
```

---

### 示例 2：Flow EVM 金库查询

**场景：** 查询 Flow EVM 测试网金库，检查余额后执行买入。

**Vault Node 配置：**

```json
{
  "chain": "flow_evm",
  "chain_id": 545,
  "vault_address": "0x1234567890123456789012345678901234567890"
}
```

**工作流结构：**

```
Vault Node (Flow EVM)
    ↓ vault (持仓信息)
    ↓
Code Node (检查是否有足够 USDC)
    ↓ decision
Condition Node (余额 > 100 USDC?)
    ↓ (Yes)
Buy Node (买入 FLOW)
```

---

### 示例 3：动态金库选择

**场景：** 根据 AI 分析结果动态选择要操作的金库。

**工作流结构：**

```
AI Model Node (分析市场，决定在哪条链交易)
    ↓ ai_response { chain: "aptos" }
    ↓
Vault Node (接收 chain 参数)
    ↓ vault_balance
    ↓
Swap Node (执行交易)
```

---

## API 依赖

### Aptos Vault Service

**功能：** 查询 Aptos 链上的金库信息

**方法：** `get_investor_holdings(vault_address)`

**返回格式：**

```python
{
  "holdings": [
    {
      "token_address": "0x1::aptos_coin::AptosCoin",
      "amount": "100000000",
      "decimals": 8,
      "token_symbol": "APT"
    }
  ]
}
```

### Flow EVM Vault Service

**功能：** 查询 Flow EVM 链上的金库信息

**方法：** `get_vault_info_with_prices(vault_address)`

**返回格式：**

```python
{
  "success": True,
  "vault": {
    "total_value_usd": "6.00",
    "portfolio_composition": [
      {
        "token_address": "0x...",
        "amount": "5.0",
        "token_symbol": "FLOW",
        "decimals": 18,
        "price_usd": 0.85,
        "value_usd": "4.25",
        "percentage": 70.83
      }
    ],
    "token_count": 2
  }
}
```

---

## 最佳实践

### 1. 金库地址获取

**推荐流程：**

```
1. 登录 TradingFlow 平台
2. 进入 Windmill 页面
3. 查看 "My Vaults" 标签页
4. 复制金库地址
5. 粘贴到 Vault Node 配置
```

### 2. 链选择

✅ **推荐：**

- 确认您的金库在哪条链上创建
- 使用对应的 chain 参数
- Flow EVM 需要指定正确的 chain_id

❌ **避免：**

- 不要在 Aptos 金库上使用 flow_evm
- 不要使用无效的金库地址

### 3. 工作流设计

**基本模式：**

```
Vault Node → 决策节点 → 交易节点
```

**带余额检查：**

```
Vault Node → Code Node (检查余额) → Condition Node → 交易节点
```

**多金库对比：**

```
Vault Node (Aptos) ─┐
                    ├─→ Code Node (对比价值) → 选择最佳金库
Vault Node (Flow) ──┘
```

---

## 注意事项

### ⚠️ 重要提示

1. **金库地址验证**

   - 地址必须是有效的已创建金库
   - 使用错误地址会导致查询失败
   - 建议在 Windmill 页面确认地址

2. **链参数一致性**

   - chain 参数必须与金库实际所在链匹配
   - Flow EVM 必须指定正确的 chain_id
   - 不支持跨链金库查询

3. **价格数据时效性**

   - 价格数据实时从市场获取
   - 可能存在短暂延迟
   - 网络问题可能导致价格获取失败

4. **资产价值计算**

   - 小数位转换可能存在精度损失
   - 价格为 0 的代币价值为 0
   - 总价值为所有持仓价值之和

5. **输出信号使用**
   - 下游交易节点必须接收 vault_address 参数
   - chain 参数确保交易在正确的链上执行
   - vault 可用于余额检查和决策

---

## 故障排查

**Q: 提示 "vault_address is required but not provided"？**

A:

1. 确认 Vault Node 的 vault_address 参数已填写
2. 如果使用信号传递，确认上游节点正确发送了 vault_address
3. 检查地址格式是否正确（Aptos: 0x..., Flow EVM: 0x...）

---

**Q: 查询返回空持仓或价值为 0？**

A:

1. 确认金库地址正确
2. 确认金库中确实有资产
3. 检查网络连接是否正常
4. 查看节点日志获取详细错误信息

---

**Q: Flow EVM 查询失败？**

A:

1. 确认 chain_id 正确（测试网: 545）
2. 检查 Flow EVM companion 服务是否运行
3. 确认金库地址格式正确
4. 查看错误日志确认具体问题

---

**Q: 代币价格显示为 0？**

A:

1. 某些代币可能暂时无法获取价格
2. 新代币可能未被价格服务收录
3. 网络问题可能导致价格获取失败
4. 检查日志中的 "Failed to get price for token" 警告

---

## 技术规格

| 规格项       | 值                          |
| ------------ | --------------------------- |
| **节点版本** | 1.0.0                       |
| **支持的链** | Aptos, Flow EVM             |
| **最大并发** | 1                           |
| **执行模式** | 单次执行（查询一次后完成）  |
| **超时时间** | 30 秒（服务调用）           |
| **日志级别** | DEBUG, INFO, WARNING, ERROR |

---

## 相关节点

- **Swap Node** - 使用金库执行代币交换
- **Buy Node** - 使用金库买入代币
- **Sell Node** - 使用金库卖出代币
- **Code Node** - 处理金库数据和余额检查
- **Condition Node** - 根据金库余额做决策

---

**相关文档：**

- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [Weather 语法](../core-concepts/weather-syntax.md) - 工作流文件格式

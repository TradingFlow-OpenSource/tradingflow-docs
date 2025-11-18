# Swap Node

Swap Node 是 TradingFlow 的核心交易执行节点，用于在去中心化交易所（DEX）上执行代币交换操作。节点支持多链（Aptos、Flow EVM），具有智能池子搜索、动态滑点保护和精确的价格计算功能。

---

## 节点信息

| 属性         | 值                              |
| ------------ | ------------------------------- |
| **节点类型** | `swap_node`                     |
| **显示名称** | Swap                            |
| **节点分类** | Trade（交易执行）               |
| **图标**     | ⚖️ 天平图标                     |
| **句柄颜色** | Amber（橙色） / Emerald（绿色） |
| **节点类别** | Base Node（基类节点）           |

---

## 功能说明

Swap Node 在去中心化交易所执行代币交换，自动搜索最优交易池、计算最佳价格、应用滑点保护，并返回完整的交易收据。节点是 Buy Node 和 Sell Node 的基类。

**主要用途：**

- 在 DEX 上交换代币
- 自动寻找最优流动性池
- 按百分比或固定金额交易
- 滑点保护和价格计算
- 多链交易执行（Aptos Hyperion、Flow EVM）

**核心特性：**

- 🔍 **智能池子搜索**：自动查找最优费率的流动性池
- 💹 **动态价格计算**：基于池子实时数据计算交换比率
- 🛡️ **滑点保护**：动态计算 sqrt_price_limit 防止价格异常
- 📊 **灵活金额设置**：支持百分比和固定金额两种方式
- 🌐 **多链支持**：Aptos（Hyperion DEX）和 Flow EVM

---

## 输入参数

### 参数列表

| 参数                       | 类型         | 必填 | 默认值                     | 说明                                   |
| -------------------------- | ------------ | ---- | -------------------------- | -------------------------------------- |
| `chain`                    | select       | ✅   | `aptos`                    | 区块链网络                             |
| `from_token`               | searchSelect | ✅   | -                          | 源代币符号                             |
| `to_token`                 | searchSelect | ✅   | -                          | 目标代币符号                           |
| `vault_address`            | text         | ✅   | -                          | 金库地址                               |
| `amount_in_human_readable` | **switch**   | ✅   | `{mode:"number",value:""}` | 交易金额（Number/Percentage 模式切换） |
| `slippery`                 | number       | ✅   | `1.0`                      | 滑点容忍度（%）                        |

### chain 参数

**支持的区块链：**

| 链           | 值         | DEX      | 说明            |
| ------------ | ---------- | -------- | --------------- |
| **Aptos**    | `aptos`    | Hyperion | 主网/测试网支持 |
| **Flow EVM** | `flow_evm` | Flow DEX | 测试网支持      |

### from_token 和 to_token 参数

**Aptos 支持的代币：**

| 符号   | 名称       | 地址                           |
| ------ | ---------- | ------------------------------ |
| `APT`  | Aptos Coin | `0x1::aptos_coin::AptosCoin`   |
| `USDT` | Tether USD | `0xf22bede237...::asset::USDT` |
| `USDC` | USD Coin   | `0xf22bede237...::asset::USDC` |
| `BTC`  | Bitcoin    | 代币地址                       |
| `ETH`  | Ethereum   | 代币地址                       |

**Flow EVM 支持的代币：**

| 符号   | 名称       | 地址格式        |
| ------ | ---------- | --------------- |
| `FLOW` | Flow Token | `0xEeee...EEeE` |
| `USDC` | USD Coin   | ERC20 地址      |
| `USDT` | Tether USD | ERC20 地址      |

### amount_in_human_readable 参数

**说明：** 交易金额，支持两种模式：精确数量（Number）和百分比（Percentage）

**类型：** Switch（模式切换器）

**数据结构：**

```typescript
{
  mode: "number" | "percentage",  // 模式：精确数量 或 百分比
  value: string                    // 数值（字符串形式）
}
```

**模式 1：Number（精确数量）**

指定精确的 `from_token` 数量进行交换。

```json
{
  "from_token": "USDT",
  "to_token": "APT",
  "amount_in_human_readable": {
    "mode": "number",
    "value": "100"
  }
}
// 含义：交换 100 USDT 为 APT
```

**模式 2：Percentage（百分比）**

按 `from_token` 余额的百分比进行交换（0-100）。

```json
{
  "from_token": "USDT",
  "to_token": "APT",
  "amount_in_human_readable": {
    "mode": "percentage",
    "value": "50"
  }
}
// 含义：交换 50% 的 USDT 余额为 APT
```

**前端 UI：**

- 用户首先选择模式（Number 或 Percentage）
- Number 模式：显示数字输入框
- Percentage 模式：显示 0-100% 滑块 + 输入框

**选择建议：**

- 精确控制交易数量 → 使用 `number` 模式
- 按余额比例交易 → 使用 `percentage` 模式

**注意：**

- 此金额为源代币（from_token）的数量
- 自动根据代币小数位转换为链上格式

### slippery 参数

**格式：** 百分比（推荐 0.5-5.0）

| 场景         | 推荐值   | 说明       |
| ------------ | -------- | ---------- |
| 高流动性代币 | 0.5-1.0% | 主流交易对 |
| 中等流动性   | 1.0-3.0% | 一般代币   |
| 低流动性     | 3.0-5.0% | 小市值代币 |

**说明：**

- 防止价格在交易执行时发生不利变化
- 值越低，保护越强，但交易失败风险越高
- Aptos 使用动态 sqrt_price_limit 计算

### vault_address 参数

**来源：** Vault Node 输出

**格式：** 区块链地址

**说明：**

- 通常从上游 Vault Node 接收
- 必须是有效的金库地址
- 用于执行交易和查询余额

---

## 输出参数

### 输出列表

| 输出 ID         | 显示名称      | 数据类型 | 说明           |
| --------------- | ------------- | -------- | -------------- |
| `trade_receipt` | Trade Receipt | object   | 完整的交易收据 |

### trade_receipt 输出

**数据类型：** `object`

**完整数据结构：**

```typescript
{
  success: boolean; // 交易是否成功
  message: string; // 状态消息
  chain: string; // 区块链网络
  vault_address: string; // 金库地址
  from_token: string; // 源代币符号
  to_token: string; // 目标代币符号
  input_token_address: string; // 源代币合约地址
  output_token_address: string; // 目标代币合约地址
  slippery: number; // 滑点容忍度

  // 交易详情
  tx_hash: string; // 交易哈希
  status: string; // 交易状态
  dex_name: string; // DEX 名称
  amount_in: string; // 实际输入金额
  amount_out: string; // 实际输出金额
  amount_out_min: string; // 最小输出金额
  timestamp: string; // 交易时间戳
  gas_used: string; // Gas 消耗

  raw_result: object; // 原始结果
}
```

**Aptos 交易示例：**

```json
{
  "success": true,
  "message": "Swap executed successfully",
  "chain": "aptos",
  "vault_address": "0x6a1a233...",
  "from_token": "USDT",
  "to_token": "APT",
  "input_token_address": "0xf22bede237...::asset::USDT",
  "output_token_address": "0x1::aptos_coin::AptosCoin",
  "slippery": 1.0,
  "tx_hash": "0x1234567890abcdef...",
  "status": "Success",
  "dex_name": "hyperion",
  "amount_in": "100000000",
  "amount_out": "13793103",
  "amount_out_min": "13655732",
  "timestamp": "2024-01-15T10:30:00Z",
  "gas_used": "5432"
}
```

**Flow EVM 交易示例：**

```json
{
  "success": true,
  "message": "Swap executed successfully",
  "chain": "flow_evm",
  "vault_address": "0x1234...",
  "from_token": "USDC",
  "to_token": "FLOW",
  "tx_hash": "0xabcdef...",
  "status": "Success",
  "dex_name": "flow_evm",
  "amount_in": "100000000",
  "amount_out": "117647058823529412",
  "gas_used": "150000"
}
```

---

## 核心算法

### 1. 智能池子搜索

**Aptos Hyperion DEX：**

节点自动搜索最优流动性池，按以下优先级：

| 优先级 | Fee Tier | 费率  | 适用场景     |
| ------ | -------- | ----- | ------------ |
| 1      | 1        | 0.05% | 稳定币对     |
| 2      | 2        | 0.3%  | 主流代币对   |
| 3      | 0        | 0.01% | 极低费率池   |
| 4      | 3        | 1.0%  | 低流动性代币 |

**池子适用性检查：**

- 最小流动性：$1,000 USD
- 有效价格：sqrt_price > 0
- TVL 数据可用

**工作流程：**

```
1. 向 Monitor API 请求池子数据
   ↓
2. 按优先级检查 fee_tier
   ↓
3. 验证流动性充足
   ↓
4. 返回最优池子或使用默认 fee_tier=1
```

### 2. 动态价格计算

**基于池子的价格计算（推荐）：**

```
1. 从池子获取 sqrt_price (X64 格式)
   ↓
2. 转换为实际价格
   price = (sqrt_price / 2^64)^2
   ↓
3. 调整代币小数位差异
   adjusted_price = price * 10^(decimals_out - decimals_in)
   ↓
4. 确定交易方向
   - token0 → token1: output = input * price
   - token1 → token0: output = input / price
   ↓
5. 应用滑点
   output_min = output * (1 - slippage%)
```

**示例计算：**

```
输入：100 USDT (decimals=6)
输出：APT (decimals=8)
池子 sqrt_price：18446744073709551616 (X64)
滑点：1.0%

步骤：
1. sqrt_price_x64 = 18446744073709551616 / 2^64 = 1.0
2. price = 1.0^2 = 1.0
3. adjusted_price = 1.0 * 10^(8-6) = 100.0
4. output = 100 / 7.25 = 13.79 APT (假设 APT=$7.25)
5. output_min = 13.79 * 0.99 = 13.65 APT
```

### 3. 滑点保护

**动态 sqrt_price_limit 计算：**

```python
# 确定交易方向
if selling_apt:
    # 卖出APT：防止价格下跌，使用下限
    sqrt_multiplier = sqrt(1 - slippage%)
else:
    # 买入APT：防止价格上涨，使用上限
    sqrt_multiplier = sqrt(1 + slippage%)

sqrt_price_limit = current_sqrt_price * sqrt_multiplier
```

**保护机制：**

- **卖出波动代币**：设置价格下限，防止卖便宜
- **买入波动代币**：设置价格上限，防止买贵
- **自适应方向**：根据交易对自动判断

---

## 工作流程

### 完整执行流程

```
1. 参数验证
   ├─ 检查 chain 是否支持
   ├─ 验证 from_token 和 to_token
   ├─ 确认 vault_address 存在
   └─ 验证金额参数

2. 代币地址解析
   ├─ Aptos: 从符号查询合约地址
   ├─ Flow EVM: 使用符号作为地址（占位）
   └─ 获取代币小数位信息

3. 计算交易金额
   ├─ 优先使用 amount_in_human_readable
   ├─ 或查询余额 * amount_in_percentage
   └─ 转换为链上最小单位（wei）

4. 搜索最优池子（Aptos）
   ├─ 调用 Monitor API
   ├─ 按优先级检查 fee_tier
   ├─ 验证流动性
   └─ 确定使用的池子或默认值

5. 价格计算
   ├─ 从池子数据计算当前价格
   ├─ 调整代币小数位
   ├─ 确定交易方向
   └─ 应用滑点得到 amount_out_min

6. 滑点保护计算
   ├─ 判断交易方向（买入/卖出）
   ├─ 计算 sqrt_multiplier
   └─ 得到 sqrt_price_limit

7. 执行交易
   ├─ Aptos: admin_execute_swap
   └─ Flow EVM: execute_swap

8. 构建交易收据
   ├─ 提取交易哈希和状态
   ├─ 记录输入输出金额
   └─ 添加 Gas 消耗

9. 发送信号
   └─ 发送 DEX_TRADE_RECEIPT 信号

10. 完成
    └─ 设置节点状态为 COMPLETED
```

---

## 使用示例

### 示例 1：简单代币交换

**场景：** 使用 100 USDT 购买 APT。

**工作流结构：**

```
Vault Node (Aptos, 获取地址)
    ↓ vault_address
Swap Node (USDT → APT)
    ↓ trade_receipt
Telegram Sender Node (发送交易通知)
```

**Swap Node 配置：**

```json
{
  "chain": "aptos",
  "from_token": "USDT",
  "to_token": "APT",
  "amount_in_human_readable": 100.0,
  "slippery": 1.0
}
```

**输出示例：**

```json
{
  "success": true,
  "from_token": "USDT",
  "to_token": "APT",
  "amount_in": "100000000",
  "amount_out": "13793103",
  "tx_hash": "0x..."
}
```

---

### 示例 2：按百分比交易

**场景：** 使用当前 USDT 余额的 25% 购买 APT。

**Swap Node 配置：**

```json
{
  "chain": "aptos",
  "from_token": "USDT",
  "to_token": "APT",
  "amount_in_percentage": 25.0,
  "slippery": 0.5
}
```

**执行过程：**

```
1. 查询金库 USDT 余额：400 USDT
2. 计算交易金额：400 * 25% = 100 USDT
3. 执行交换
```

---

### 示例 3：AI 驱动的动态交易

**场景：** AI 分析后决定交易代币和金额。

**工作流结构：**

```
Binance Price Node (获取价格)
    ↓ kline_data
AI Model Node (分析趋势)
    ↓ ai_response { from_token, to_token, amount }
    ↓
Vault Node (获取金库)
    ↓ vault_address
    ↓
Swap Node (执行AI推荐的交易)
    ↓ trade_receipt
Dataset Output Node (保存交易记录)
```

---

### 示例 4：多池子对比

**场景：** 智能选择最优池子执行交易。

**执行日志示例：**

```
[INFO] Searching for best pool: USDT -> APT
[INFO] Found optimal pool: fee_tier=1, TVL=$1,234,567
[INFO] Using pool with fee_tier=1
[INFO] Pool-based calculation: output_after_1%_slippage=13655732
[INFO] Dynamic sqrt_price_limit: selling APT, lower limit
[INFO] Transaction successful: tx_hash=0x...
```

---

## API 依赖

### Aptos Vault Service

**方法：** `admin_execute_swap()`

**参数：**

```python
{
  "vault_address": "0x...",
  "input_token_address": "0x...",
  "output_token_address": "0x...",
  "amount_in": 100000000,
  "fee_tier": 1,
  "sqrt_price_limit": "18446744073709551616",
  "amount_out_min": 13655732
}
```

### Flow EVM Vault Service

**方法：** `execute_swap()`

**参数：**

```python
{
  "vault_address": "0x...",
  "token_in": "0x...",
  "token_out": "0x...",
  "amount_in": 100000000,
  "amount_out_min": 117647058823529412
}
```

### Monitor API

**端点：** `GET /aptos/pools/pair`

**参数：**

- `token1`: 第一个代币地址
- `token2`: 第二个代币地址
- `feeTiers`: 费率优先级列表（如 "1,2,0,3"）

**响应：**

```json
{
  "success": true,
  "pool": {
    "feeTier": 1,
    "sqrtPrice": "18446744073709551616",
    "token1Info": { "symbol": "USDT" },
    "token2Info": { "symbol": "APT" }
  },
  "tvlUSD": "1234567.89"
}
```

---

## 最佳实践

### 1. 金额设置

**固定金额（推荐用于精确控制）：**

```json
{
  "amount_in_human_readable": 100.0
}
```

**百分比金额（推荐用于动态调整）：**

```json
{
  "amount_in_percentage": 25.0
}
```

### 2. 滑点设置

| 代币类型              | 建议滑点 | 原因       |
| --------------------- | -------- | ---------- |
| 稳定币对（USDT/USDC） | 0.1-0.5% | 价格稳定   |
| 主流币对（APT/USDT）  | 0.5-1.0% | 流动性好   |
| 一般代币              | 1.0-3.0% | 中等流动性 |
| 小市值代币            | 3.0-5.0% | 价格波动大 |

### 3. 错误处理

✅ **推荐模式：**

```
Swap Node
    ↓ trade_receipt
Condition Node (检查 success)
    ↓ (true)
    ├─→ 成功分支
    ↓ (false)
    └─→ 失败处理分支
```

### 4. 交易确认

**添加确认步骤：**

```
Swap Node
    ↓ trade_receipt { tx_hash }
Code Node (轮询交易状态)
    ↓ confirmation
Condition Node (确认完成?)
    ↓
后续操作
```

---

## 注意事项

### ⚠️ 重要提示

1. **金额参数互斥**

   - 同时提供两个金额参数时，优先使用 `amount_in_human_readable`
   - 建议只提供一个金额参数

2. **滑点设置**

   - 过低的滑点可能导致交易失败
   - 过高的滑点可能导致不利成交
   - 根据代币流动性合理设置

3. **池子流动性**

   - 确保交易金额不超过池子流动性
   - 大额交易可能需要更高滑点
   - 查看日志中的 TVL 信息

4. **Gas 费用**

   - Aptos 交易消耗 APT 作为 Gas
   - 确保金库有足够 APT 支付 Gas
   - 典型 Gas 消耗：3000-6000 Gas Units

5. **交易失败处理**
   - 检查 trade_receipt.success 状态
   - 查看 message 字段了解失败原因
   - 常见失败原因：余额不足、滑点超限、池子流动性不足

---

## 故障排查

**Q: 提示 "No valid amount specified"？**

A:

1. 确认提供了 `amount_in_human_readable` 或 `amount_in_percentage`
2. 检查金额值是否有效（大于 0）
3. 百分比值必须在 0-100 范围内

---

**Q: 池子搜索失败，使用默认 fee_tier？**

A:

1. 这是正常回退机制，不影响交易
2. Monitor API 可能暂时不可用
3. 代币对可能没有最优池子
4. 默认 fee_tier=1（0.05%）适用于大多数交易

---

**Q: 交易失败，提示滑点超限？**

A:

1. 增加 `slippery` 参数值（如从 0.5 增加到 1.0）
2. 检查池子流动性是否充足
3. 尝试减少交易金额
4. 等待市场波动平复

---

**Q: Gas 不足错误？**

A:

1. 确保金库有足够 APT（Aptos）或 FLOW（Flow EVM）
2. 典型交易需要 0.001-0.005 APT
3. 可以先执行一次小额充值

---

**Q: 输出金额与预期不符？**

A:

1. 检查日志中的价格计算过程
2. 确认当前市场价格
3. 滑点保护会降低最小输出金额
4. 实际输出通常高于 amount_out_min

---

## 技术规格

| 规格项         | 值                          |
| -------------- | --------------------------- |
| **节点版本**   | 0.0.2                       |
| **节点类别**   | Base Node                   |
| **支持的链**   | Aptos, Flow EVM             |
| **支持的 DEX** | Hyperion (Aptos), Flow DEX  |
| **最大并发**   | 1                           |
| **执行超时**   | 60 秒                       |
| **Gas 估算**   | 动态（3000-6000 Units）     |
| **日志级别**   | DEBUG, INFO, WARNING, ERROR |

---

## 实例节点

Swap Node 是基类节点，有两个专门的实例节点：

- **[Buy Node](buy-node.md)** - 专门用于买入代币
- **[Sell Node](sell-node.md)** - 专门用于卖出代币

这些实例节点提供更清晰的语义，底层使用 Swap Node 的功能。

---

## 相关节点

- **Vault Node** - 提供金库地址和余额信息
- **Buy Node** - Swap Node 的买入实例
- **Sell Node** - Swap Node 的卖出实例
- **Code Node** - 预处理交易参数
- **Condition Node** - 检查交易结果

---

**相关文档：**

- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [Vault Node](vault-node.md) - 金库管理节点
- [Weather 语法](../core-concepts/weather-syntax.md) - 工作流文件格式

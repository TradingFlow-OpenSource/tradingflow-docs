# Sell Node

Sell Node 是专门用于卖出代币的交易执行节点，是 Swap Node 的实例节点。它提供清晰的卖出语义，简化了代币出售操作的配置。

---

## 节点信息

| 属性 | 值 |
|------|-----|
| **节点类型** | `sell_node` |
| **显示名称** | Sell |
| **节点分类** | Trade（交易执行） |
| **图标** | 📉 下跌趋势图标 |
| **句柄颜色** | Amber（橙色） / Emerald（绿色） |
| **节点类别** | Instance Node（实例节点） |
| **基类节点** | `swap_node` |

---

## 功能说明

Sell Node 专门用于卖出代币操作，内部使用 Swap Node 的完整交换功能，但提供了更直观的参数命名。用户只需指定要卖出的代币和换取的代币，节点自动处理交换逻辑。

**主要用途：**
- 卖出持有的代币
- 清晰的卖出语义
- 简化参数配置
- 自动处理交换方向

**核心特性：**
- 💸 **卖出语义**：`sell_token` 和 `base_token` 参数更直观
- 🔄 **自动映射**：内部转换为 Swap Node 的 from/to 参数
- 📊 **完整功能**：继承 Swap Node 的所有功能
- 🎯 **专注卖出**：优化的用户体验

---

## 与 Swap Node 的关系

Sell Node 是 Swap Node 的**实例节点**，关系如下：

```
SwapNode (基类)
    ├─ from_token (源代币)
    └─ to_token (目标代币)
        ↓ 实例化
SellNode (实例)
    ├─ sell_token → from_token (卖什么)
    └─ base_token → to_token (换什么)
```

**参数映射：**
- `sell_token` → `from_token`（卖出的代币）
- `base_token` → `to_token`（换取的代币）

**与 Buy Node 的对比：**

| 节点 | 操作 | from_token | to_token |
|------|------|-----------|----------|
| Buy Node | 买入 | base_token（支付） | buy_token（获得） |
| Sell Node | 卖出 | sell_token（卖出） | base_token（换取） |

---

## 输入参数

### 参数列表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `sell_token` | searchSelect | ✅ | - | 要卖出的代币符号 |
| `base_token` | searchSelect | ✅ | - | 换取的代币符号 |
| `chain` | searchSelect | ✅ | `aptos` | 区块链网络 |
| `vault_address` | text | ✅ | - | 金库地址 |
| `amount_in_human_readable` | number | ❌ | - | 卖出金额（sell_token 数量） |
| `amount_in_percentage` | number | ❌ | - | 卖出金额（sell_token 余额百分比） |
| `slippery` | number | ✅ | `1.0` | 滑点容忍度（%） |
| `order_type` | select | ✅ | `market` | 订单类型 |
| `limited_price` | number | ❌ | - | 限价（未实现） |

### sell_token 参数

**说明：** 您想要卖出的代币符号

**示例：**
- `APT` - 卖出 Aptos Coin
- `BTC` - 卖出 Bitcoin
- `ETH` - 卖出 Ethereum

### base_token 参数

**说明：** 换取的代币符号（"换成什么"）

**示例：**
- `USDT` - 换取 USDT
- `USDC` - 换取 USDC
- `APT` - 换取 APT

### amount_in_human_readable 参数

**说明：** 卖出金额，单位为 `sell_token`

**示例：**
```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": 10.0
}
// 含义：卖出 10 APT 换取 USDT
```

### amount_in_percentage 参数

**说明：** 卖出金额占 `sell_token` 余额的百分比

**示例：**
```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_percentage": 50.0
}
// 含义：卖出 50% 的 APT 余额换取 USDT
```

### order_type 参数

**支持的类型：**

| 值 | 说明 | 状态 |
|----|------|------|
| `market` | 市价单 | ✅ 已实现 |
| `limit` | 限价单 | ⏳ 未实现 |

**说明：**
- 当前仅支持市价单
- 限价单功能计划中
- 限价单可设置最低卖出价格

---

## 输出参数

### 输出列表

| 输出 ID | 显示名称 | 数据类型 | 说明 |
|---------|---------|---------|------|
| `trade_receipt` | Trade Receipt | object | 完整的交易收据 |

### trade_receipt 输出

**数据结构与 Swap Node 完全相同**，参见 [Swap Node 文档](swap-node.md#trade_receipt-输出)。

**关键字段：**
- `from_token`: `sell_token` 值
- `to_token`: `base_token` 值
- `amount_in`: 实际卖出数量
- `amount_out`: 实际换取数量
- `tx_hash`: 交易哈希

---

## 使用示例

### 示例 1：卖出 APT 换取 USDT

**场景：** 卖出 10 APT 换取 USDT。

**工作流结构：**
```
Vault Node (Aptos)
    ↓ vault_address
Sell Node (卖出 APT)
    ↓ trade_receipt
Telegram Sender Node (发送通知)
```

**Sell Node 配置：**
```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": 10.0,
  "slippery": 1.0,
  "chain": "aptos"
}
```

**交易理解：**
- 卖出 **10 APT** 换取 **USDT**
- 根据当前价格，预计换取约 72.5 USDT（假设 APT=$7.25）
- 滑点保护：至少换取 71.78 USDT

---

### 示例 2：卖出 50% 持仓

**场景：** 卖出当前 BTC 持仓的 50%。

**Sell Node 配置：**
```json
{
  "sell_token": "BTC",
  "base_token": "USDT",
  "amount_in_percentage": 50.0,
  "slippery": 0.5,
  "chain": "aptos"
}
```

**执行过程：**
```
1. 查询金库 BTC 余额：2.5 BTC
2. 计算卖出金额：2.5 * 50% = 1.25 BTC
3. 执行卖出：1.25 BTC → USDT
```

---

### 示例 3：止盈策略

**场景：** 价格达到目标位时自动卖出。

**工作流结构：**
```
Binance Price Node (获取 APT 价格)
    ↓ kline_data
Code Node (检查价格)
    ↓ should_sell (true/false)
Condition Node (价格 > $10?)
    ↓ (true)
    ├─→ Vault Node → Sell Node (卖出)
    ↓ (false)
    └─→ 继续持有
```

**Sell Node 配置：**
```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_percentage": 100.0,
  "slippery": 2.0
}
```

---

### 示例 4：AI 驱动止损

**场景：** AI 检测到风险信号时自动卖出。

**工作流结构：**
```
Binance Price Node (价格数据)
    ↓ kline_data
AI Model Node (风险分析)
    ↓ ai_response { risk_level: "high", action: "sell" }
    ↓
Code Node (解析 AI 建议)
    ↓
Condition Node (是否卖出?)
    ↓ (true)
Vault Node
    ↓
Sell Node (执行卖出)
    ↓ trade_receipt
Dataset Output Node (记录交易)
```

---

### 示例 5：定期获利

**场景：** 定期卖出一定比例的盈利代币。

**Sell Node 配置：**
```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_percentage": 10.0,
  "slippery": 1.0
}
// 每次卖出 10% 的 APT 持仓
```

---

## 最佳实践

### 1. 参数命名理解

**Sell Node 语义：**
```
我要卖出 [sell_token] 换取 [base_token]
```

**示例：**
- 卖出 **APT** 换取 **USDT** ✅ 清晰
- 用 **APT** 换 **USDT** ✅ 同义

### 2. 与 Buy Node 的区别

**Sell Node（卖出持仓）：**
```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": 10.0
}
// 卖出 10 APT，换取 USDT
```

**Buy Node（买入新仓）：**
```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": 100.0
}
// 用 100 USDT 买入 APT
```

**选择指南：**
- 卖出持有的代币 → Sell Node
- 买入新的代币 → Buy Node
- 任意方向交换 → Swap Node

### 3. 卖出策略

**全部卖出：**
```json
{
  "amount_in_percentage": 100.0
}
```

**分批卖出：**
```json
{
  "amount_in_percentage": 25.0
}
// 分 4 次卖出，每次 25%
```

**固定数量：**
```json
{
  "amount_in_human_readable": 5.0
}
// 每次卖出固定 5 个代币
```

### 4. 滑点设置建议

| 场景 | 建议滑点 | 原因 |
|------|---------|------|
| 紧急止损 | 3.0-5.0% | 确保成交 |
| 常规卖出 | 1.0-2.0% | 平衡价格和成交 |
| 止盈卖出 | 0.5-1.0% | 获得更好价格 |

---

## 注意事项

### ⚠️ 重要提示

1. **参数方向理解**
   - `sell_token` 是卖出的代币（减少）
   - `base_token` 是换取的代币（增加）
   - 金额参数指的是 `sell_token` 的数量

2. **与 Buy Node 的对称性**
   - Sell Node 和 Buy Node 是互为反向操作
   - Buy: 用 USDT 买 APT ↔ Sell: 卖 APT 换 USDT
   - 参数顺序相反但逻辑相同

3. **继承的限制**
   - 所有 Swap Node 的限制都适用
   - 滑点、Gas、流动性等要求相同
   - 参见 [Swap Node 注意事项](swap-node.md#注意事项)

4. **卖出时机**
   - 市场波动大时考虑增加滑点
   - 大额卖出可能影响市场价格
   - 流动性不足时可能无法完全成交

5. **订单类型**
   - 当前仅支持市价单
   - 限价单（`limit`）功能未实现
   - `limited_price` 参数暂时无效

---

## 故障排查

**Q: 提示 "Token not found in holdings"？**

A:
1. 确认金库中确实持有要卖出的代币
2. 检查 `sell_token` 符号是否正确
3. 确认余额足够支付 Gas 费

---

**Q: 实际换取的金额少于预期？**

A:
1. 这是正常现象，受市场价格影响
2. 检查交易时的实时价格
3. 滑点保护会设置最低价格
4. 查看 `amount_out_min` 和实际 `amount_out`

---

**Q: 应该使用 Sell Node 还是 Swap Node？**

A:
- **使用 Sell Node**：
  - ✅ 明确的卖出操作
  - ✅ 前端用户界面
  - ✅ 止盈/止损策略
  
- **使用 Swap Node**：
  - ✅ 需要灵活的交换方向
  - ✅ 程序化交易
  - ✅ 不确定交换方向时

---

**Q: 如何实现止损功能？**

A:
```
Binance Price Node (监控价格)
    ↓
Code Node (判断是否触发止损)
    ↓ { trigger_stop_loss: true }
Condition Node (触发?)
    ↓ (true)
Vault Node → Sell Node (卖出止损)
```

---

**Q: 如何分批卖出？**

A:
方案1 - 多次执行：
```
Sell Node (amount_in_percentage: 25%)
// 手动执行 4 次
```

方案2 - 定时卖出：
```
每天执行一次工作流
Sell Node (amount_in_percentage: 25%)
// 4 天内分批卖完
```

---

## 技术规格

| 规格项 | 值 |
|--------|-----|
| **节点版本** | 0.0.2 |
| **节点类别** | Instance Node |
| **基类节点** | `swap_node` |
| **继承功能** | 100% |
| **支持的链** | Aptos, Flow EVM |
| **支持的 DEX** | Hyperion (Aptos), Flow DEX |

---

## 常见卖出策略

### 1. 固定止盈

```
价格 > 目标价 → 卖出全部
```

### 2. 分批止盈

```
价格每上涨 10% → 卖出 20% 持仓
```

### 3. 追踪止损

```
价格从最高点回落 5% → 卖出全部
```

### 4. 时间止盈

```
持有 30 天 → 卖出 50% 锁定收益
```

### 5. 风险控制

```
AI 风险评级 > 8 → 卖出减仓
```

---

## 相关节点

- **[Swap Node](swap-node.md)** - 基类节点，通用代币交换
- **[Buy Node](buy-node.md)** - 买入代币的实例节点
- **[Vault Node](vault-node.md)** - 金库管理节点
- **Code Node** - 预处理交易参数
- **Condition Node** - 条件判断

---

**相关文档：**
- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [Swap Node](swap-node.md) - 基类节点完整文档
- [Weather 语法](../core-concepts/weather-syntax.md) - 工作流文件格式

# Buy Node

Buy Node 是专门用于买入代币的交易执行节点，是 Swap Node 的实例节点。它提供清晰的买入语义，简化了代币购买操作的配置。

---

## 节点信息

| 属性 | 值 |
|------|-----|
| **节点类型** | `buy_node` |
| **显示名称** | Buy |
| **节点分类** | Trade（交易执行） |
| **图标** | 📈 上涨趋势图标 |
| **句柄颜色** | Amber（橙色） / Emerald（绿色） |
| **节点类别** | Instance Node（实例节点） |
| **基类节点** | `swap_node` |

---

## 功能说明

Buy Node 专门用于买入代币操作，内部使用 Swap Node 的完整交换功能，但提供了更直观的参数命名。用户只需指定要买入的代币和支付的代币，节点自动处理交换逻辑。

**主要用途：**
- 买入目标代币
- 清晰的买入语义
- 简化参数配置
- 自动处理交换方向

**核心特性：**
- 💰 **买入语义**：`buy_token` 和 `base_token` 参数更直观
- 🔄 **自动映射**：内部转换为 Swap Node 的 from/to 参数
- 📊 **完整功能**：继承 Swap Node 的所有功能
- 🎯 **专注买入**：优化的用户体验

---

## 与 Swap Node 的关系

Buy Node 是 Swap Node 的**实例节点**，关系如下：

```
SwapNode (基类)
    ├─ from_token (源代币)
    └─ to_token (目标代币)
        ↓ 实例化
BuyNode (实例)
    ├─ base_token → from_token (用什么买)
    └─ buy_token → to_token (买什么)
```

**参数映射：**
- `base_token` → `from_token`（支付的代币）
- `buy_token` → `to_token`（买入的代币）

**继承的功能：**
- 智能池子搜索
- 动态价格计算
- 滑点保护
- 多链支持
- 完整的交易收据

---

## 输入参数

### 参数列表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `buy_token` | searchSelect | ✅ | - | 要买入的代币符号 |
| `base_token` | searchSelect | ✅ | - | 用于支付的代币符号 |
| `chain` | searchSelect | ✅ | `aptos` | 区块链网络 |
| `vault_address` | text | ✅ | - | 金库地址 |
| `amount_in_human_readable` | number | ❌ | - | 支付金额（base_token 数量） |
| `amount_in_percentage` | number | ❌ | - | 支付金额（base_token 余额百分比） |
| `slippery` | number | ✅ | `1.0` | 滑点容忍度（%） |
| `order_type` | select | ✅ | `market` | 订单类型 |
| `limited_price` | number | ❌ | - | 限价（未实现） |

### buy_token 参数

**说明：** 您想要买入的代币符号

**示例：**
- `APT` - 买入 Aptos Coin
- `BTC` - 买入 Bitcoin
- `ETH` - 买入 Ethereum

### base_token 参数

**说明：** 用于支付的代币符号（"用什么买"）

**示例：**
- `USDT` - 使用 USDT 支付
- `USDC` - 使用 USDC 支付
- `APT` - 使用 APT 支付

### amount_in_human_readable 参数

**说明：** 支付金额，单位为 `base_token`

**示例：**
```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": 100.0
}
// 含义：使用 100 USDT 买入 APT
```

### amount_in_percentage 参数

**说明：** 支付金额占 `base_token` 余额的百分比

**示例：**
```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_percentage": 50.0
}
// 含义：使用 50% 的 USDT 余额买入 APT
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

---

## 输出参数

### 输出列表

| 输出 ID | 显示名称 | 数据类型 | 说明 |
|---------|---------|---------|------|
| `trade_receipt` | Trade Receipt | object | 完整的交易收据 |

### trade_receipt 输出

**数据结构与 Swap Node 完全相同**，参见 [Swap Node 文档](swap-node.md#trade_receipt-输出)。

**关键字段：**
- `from_token`: `base_token` 值
- `to_token`: `buy_token` 值
- `amount_in`: 实际支付金额
- `amount_out`: 实际买入数量
- `tx_hash`: 交易哈希

---

## 使用示例

### 示例 1：使用 USDT 买入 APT

**场景：** 花费 100 USDT 购买 APT。

**工作流结构：**
```
Vault Node (Aptos)
    ↓ vault_address
Buy Node (买入 APT)
    ↓ trade_receipt
Telegram Sender Node (发送通知)
```

**Buy Node 配置：**
```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": 100.0,
  "slippery": 1.0,
  "chain": "aptos"
}
```

**交易理解：**
- 用 **100 USDT** 买入 **APT**
- 根据当前价格，预计买入约 13.79 APT（假设 APT=$7.25）
- 滑点保护：至少买入 13.65 APT

---

### 示例 2：使用 50% 余额买入

**场景：** 使用当前 USDT 余额的 50% 买入 BTC。

**Buy Node 配置：**
```json
{
  "buy_token": "BTC",
  "base_token": "USDT",
  "amount_in_percentage": 50.0,
  "slippery": 0.5,
  "chain": "aptos"
}
```

**执行过程：**
```
1. 查询金库 USDT 余额：1000 USDT
2. 计算支付金额：1000 * 50% = 500 USDT
3. 执行买入：500 USDT → BTC
```

---

### 示例 3：AI 推荐买入

**场景：** AI 分析后推荐买入某个代币。

**工作流结构：**
```
X Listener Node (监听KOL推文)
    ↓ latest_tweets
AI Model Node (分析推文)
    ↓ ai_response { buy_token: "BTC", amount: 100 }
    ↓
Vault Node (获取金库)
    ↓ vault_address
    ↓
Buy Node (执行买入)
    ↓ trade_receipt
Dataset Output Node (保存记录)
```

**AI Model Node 输出：**
```json
{
  "buy_token": "BTC",
  "base_token": "USDT",
  "amount": 100.0,
  "confidence": 0.85,
  "reason": "技术面突破关键阻力位"
}
```

---

### 示例 4：条件买入

**场景：** 价格低于某个值时买入。

**工作流结构：**
```
Binance Price Node (获取 BTC 价格)
    ↓ kline_data
Code Node (检查价格)
    ↓ should_buy (true/false)
Condition Node (价格 < $50,000?)
    ↓ (true)
    ├─→ Vault Node → Buy Node (买入)
    ↓ (false)
    └─→ 不执行
```

---

## 最佳实践

### 1. 参数命名理解

**Buy Node 语义：**
```
我要用 [base_token] 买入 [buy_token]
```

**示例：**
- 用 **USDT** 买入 **APT** ✅ 清晰
- 买入 **APT** 用 **USDT** ✅ 同义

### 2. 与 Swap Node 对比

**Buy Node（推荐用于买入）：**
```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": 100.0
}
```

**Swap Node（通用交换）：**
```json
{
  "from_token": "USDT",
  "to_token": "APT",
  "amount_in_human_readable": 100.0
}
```

**选择建议：**
- 明确的买入操作 → 使用 Buy Node
- 双向交换操作 → 使用 Swap Node
- AI 生成的工作流 → Buy Node 语义更清晰

### 3. 金额设置

**固定金额买入：**
```json
{
  "amount_in_human_readable": 100.0
}
// 花费固定 100 USDT
```

**比例买入：**
```json
{
  "amount_in_percentage": 30.0
}
// 花费 USDT 余额的 30%
```

**建议：**
- 精确控制成本 → 使用 `amount_in_human_readable`
- 动态调整仓位 → 使用 `amount_in_percentage`

---

## 注意事项

### ⚠️ 重要提示

1. **参数方向理解**
   - `base_token` 是支付的代币（减少）
   - `buy_token` 是买入的代币（增加）
   - 金额参数指的是 `base_token` 的数量

2. **与 Sell Node 的区别**
   - Buy Node: 获得目标代币
   - Sell Node: 卖出持有代币
   - 内部实现相同，语义相反

3. **继承的限制**
   - 所有 Swap Node 的限制都适用
   - 滑点、Gas、流动性等要求相同
   - 参见 [Swap Node 注意事项](swap-node.md#注意事项)

4. **订单类型**
   - 当前仅支持市价单
   - 限价单（`limit`）功能未实现
   - `limited_price` 参数暂时无效

---

## 故障排查

**Q: 提示参数错误？**

A:
1. 确认提供了 `buy_token` 和 `base_token`
2. 检查代币符号是否正确
3. 确保金额参数有效

---

**Q: 买入数量与预期不符？**

A:
1. 检查当前市场价格
2. 确认 `amount_in` 是 base_token 的数量
3. 滑点会影响最终买入数量
4. 查看 `trade_receipt.amount_out` 获取实际数量

---

**Q: 应该使用 Buy Node 还是 Swap Node？**

A:
- **使用 Buy Node**：
  - ✅ 明确的买入操作
  - ✅ 前端用户界面
  - ✅ AI 生成的工作流
  
- **使用 Swap Node**：
  - ✅ 需要灵活的交换方向
  - ✅ 程序化交易
  - ✅ 基于其他节点输出的交换

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

## 相关节点

- **[Swap Node](swap-node.md)** - 基类节点，通用代币交换
- **[Sell Node](sell-node.md)** - 卖出代币的实例节点
- **[Vault Node](vault-node.md)** - 金库管理节点
- **Code Node** - 预处理交易参数
- **Condition Node** - 条件判断

---

**相关文档：**
- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [Swap Node](swap-node.md) - 基类节点完整文档
- [Weather 语法](../core-concepts/weather-syntax.md) - 工作流文件格式

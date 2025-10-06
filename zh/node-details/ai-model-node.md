# AI Model Node

AI Model Node 是一个智能计算节点，通过调用大语言模型（如 GPT-4）来分析数据、生成交易洞察和做出智能决策。节点能够自动根据输出连接生成 JSON 格式要求，实现智能化的数据流转。

---

## 节点信息

| 属性 | 值 |
|------|-----|
| **节点类型** | `ai_model_node` |
| **显示名称** | AI Model |
| **节点分类** | Compute（计算处理） |
| **图标** | 🤖 机器人图标 |
| **句柄颜色** | Sky（天蓝色） |

---

## 功能说明

AI Model Node 接收上游节点的各种数据信号作为上下文，通过大语言模型进行分析和处理，生成结构化的响应数据。节点具有智能输出格式化功能，能够根据下游节点的输入需求自动调整输出格式。

**主要用途：**
- 分析市场数据并生成交易建议
- 解读社交媒体内容提取交易信号
- 根据多源数据做出智能决策
- 生成符合特定格式要求的结构化数据
- 自动化复杂的数据分析任务

**核心特性：**
- 🎯 **智能格式化**：自动根据下游节点需求生成 JSON 格式
- 🔗 **多信号融合**：整合多个输入信号进行综合分析
- 📊 **结构化提取**：从 AI 响应中自动提取结构化数据
- 🤖 **多模型支持**：支持 GPT-3.5、GPT-4 等多种模型

---

## 输入参数

### 参数列表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `model` | select | ✅ | `gpt-3.5-turbo` | 使用的大语言模型 |
| `prompt` | paragraph | ✅ | 见下文 | 主提示词，指导 AI 如何处理数据 |
| `parameters` | paramMatrix | ❌ | `{}` | 模型参数（temperature、max_tokens 等） |

### model 参数

**可选模型：**

| 模型名称 | 显示名称 | 适用场景 |
|---------|---------|---------|
| `openai/gpt-3.5-turbo` | GPT-3.5 Turbo | 快速响应、成本优化 |
| `openai/gpt-4` | GPT-4 | 复杂分析、高精度决策 |
| `openai/gpt-4-turbo` | GPT-4 Turbo | 大量数据处理 |

*注意：节点使用 OpenRouter API，支持更多模型选择。*

### prompt 参数

**默认提示词：**
```
Please analyze the following information:
```

**提示词编写建议：**
1. **清晰的任务描述**：明确告诉 AI 需要完成什么任务
2. **上下文说明**：解释输入数据的含义和来源
3. **输出要求**：说明期望的输出格式和内容
4. **示例引导**：提供输出示例帮助 AI 理解需求

**示例提示词：**
```text
分析以下推文内容，判断是否包含加密货币交易信号。
如果发现明确的买入或卖出信号，请提取以下信息：
- 代币符号
- 操作类型（买入/卖出）
- 建议金额
- 信号强度（1-10）
- 理由说明
```

### parameters 参数

**支持的参数：**

| 参数名 | 类型 | 范围 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `temperature` | number | 0.0-2.0 | 0.7 | 控制输出随机性，越低越确定 |
| `max_tokens` | number | 1-4000 | 1000 | 最大输出 token 数量 |
| `top_p` | number | 0.0-1.0 | 1.0 | 核采样参数 |
| `frequency_penalty` | number | -2.0-2.0 | 0.0 | 频率惩罚，减少重复 |
| `presence_penalty` | number | -2.0-2.0 | 0.0 | 存在惩罚，鼓励新话题 |

**参数配置示例：**
```json
[
  {
    "name": "temperature",
    "value": "0.3"
  },
  {
    "name": "max_tokens",
    "value": "2000"
  }
]
```

---

## 输出参数

### 输出列表

| 输出 ID | 显示名称 | 数据类型 | 说明 |
|---------|---------|---------|------|
| `ai_response` | AI Response | text/object | AI 生成的分析结果或结构化数据 |

### ai_response 输出

**数据类型：** `text` 或 `object`（根据 `auto_format_output` 配置）

#### 文本模式输出

当没有下游连接或 `auto_format_output` 为 `false` 时：

```json
{
  "model_name": "openai/gpt-3.5-turbo",
  "response": "根据提供的推文内容分析，特朗普提到了...",
  "input_signals_count": 2,
  "input_signal_types": ["SOCIAL_MEDIA", "PRICE_DATA"],
  "timestamp": 1672531200.123
}
```

#### 结构化模式输出

当有下游连接且 `auto_format_output` 为 `true` 时，自动提取结构化数据：

```json
{
  "model_name": "openai/gpt-3.5-turbo",
  "response": "基于分析，建议买入 BTC...",
  "token": "BTC",
  "action": "buy",
  "amount": 1000.0,
  "chain": "aptos",
  "confidence": 0.85,
  "reason": "技术面突破关键阻力位，基本面积极",
  "input_signals_count": 2,
  "timestamp": 1672531200.123
}
```

---

## 智能格式化机制

### 自动格式生成

AI Model Node 的核心特性是**自动输出格式化**。节点会分析所有输出边的目标节点输入需求，自动在提示词中添加 JSON 格式要求。

### 工作原理

```
1. 分析输出连接
   ├─ 检查所有下游节点
   ├─ 识别目标节点的输入句柄名称
   └─ 推断需要的数据字段

2. 生成格式要求
   ├─ 构建 JSON 示例
   ├─ 添加字段说明
   └─ 插入到提示词中

3. AI 响应处理
   ├─ 提取 JSON 代码块
   ├─ 解析结构化数据
   └─ 合并到输出 payload

4. 数据传递
   └─ 下游节点直接获取所需字段
```

### 字段识别规则

节点根据目标句柄名称智能推断字段类型：

| 句柄关键词 | 推断类型 | 示例值 | 说明 |
|-----------|---------|--------|------|
| `chain` | string | `"aptos"` | 区块链网络标识 |
| `token` | string | `"BTC"` / `"USDT"` | 代币符号 |
| `from_token` | string | `"USDT"` | 源代币 |
| `to_token` | string | `"BTC"` | 目标代币 |
| `amount` | number | `1000.0` | 数量金额 |
| `price` | number | `50000.0` | 价格 |
| `action` | string | `"buy"` / `"sell"` | 操作类型 |
| `vault` | string | `"0x123..."` | Vault 地址 |
| `slippage` / `tolerance` | number | `1.0` | 滑点容忍度 |

### 格式化示例

**场景：** AI Model Node 连接到 Buy Node

**自动生成的格式要求：**
```text
输出格式要求:
请确保你的回复包含一个符合以下格式的JSON对象：

```json
{
  "token": "BTC",
  "amount": 1000.0,
  "chain": "aptos"
}
```

字段说明：
- token: 代币符号
- amount: 交易金额，数值类型
- chain: 区块链网络标识

请在你的分析后，提供一个符合上述格式的JSON对象。
```

---

## 信号处理

### 接收的信号

AI Model Node 可以接收任意类型的输入信号，并将它们转换为文本上下文：

**信号转换流程：**
```
输入信号 → 信号类型识别 → 格式化为文本 → 添加到上下文
```

**支持的信号类型：**
- `PRICE_DATA` - K 线和价格数据
- `SOCIAL_MEDIA` - 社交媒体内容
- `TEXT` - 纯文本数据
- `DEX_TRADE` - 交易信号
- 其他自定义信号类型

### 发送的信号

**信号句柄：** `ai_response`

**信号类型：** 根据配置动态确定

**信号负载：** 包含 AI 响应和提取的结构化数据

---

## 使用示例

### 示例 1：社交媒体情绪分析

**场景：** 分析 Twitter 推文，提取交易信号。

**工作流结构：**
```
X Listener Node (监听 @realDonaldTrump)
    ↓ latest_tweets
AI Model Node (分析推文情绪)
    ↓ ai_response
Buy Node (执行买入)
```

**AI Model Node 配置：**
```json
{
  "model": "openai/gpt-4",
  "prompt": "分析以下特朗普的推文，判断是否包含对加密货币的正面或负面情绪。如果推文提到特定加密货币且情绪积极，返回买入建议；如果情绪消极，返回卖出建议。",
  "parameters": [
    {
      "name": "temperature",
      "value": "0.3"
    }
  ]
}
```

**自动生成的输出（连接到 Buy Node）：**
```json
{
  "token": "BTC",
  "action": "buy",
  "amount": 500.0,
  "confidence": 0.82,
  "reason": "推文中提到比特币是'数字黄金'，表达了强烈的看涨情绪"
}
```

---

### 示例 2：多源数据综合分析

**场景：** 结合价格数据和社交媒体数据进行综合分析。

**工作流结构：**
```
Binance Price Node (BTC/USDT 价格) ─┐
                                    ├─→ AI Model Node (综合分析)
X Listener Node (加密货币 KOL) ────┘        ↓
                                       ai_response
                                            ↓
                                    Condition Node (决策)
```

**AI Model Node 配置：**
```json
{
  "model": "openai/gpt-4-turbo",
  "prompt": "你是一位专业的加密货币交易分析师。请综合分析以下信息：\n1. 价格数据：最近的 K 线走势\n2. 社交媒体：KOL 的观点和情绪\n\n基于这些信息，给出交易建议（买入/卖出/持有）、建议金额和置信度（0-1）。",
  "parameters": [
    {
      "name": "temperature",
      "value": "0.5"
    },
    {
      "name": "max_tokens",
      "value": "1500"
    }
  ]
}
```

---

### 示例 3：风险评估

**场景：** 评估交易策略的风险等级。

**工作流结构：**
```
Binance Price Node (价格波动率)
    ↓
AI Model Node (风险评估)
    ↓ ai_response { risk_level, recommendation }
Condition Node (根据风险调整仓位)
    ↓
Buy/Sell Node
```

**AI Model Node 配置：**
```json
{
  "model": "openai/gpt-4",
  "prompt": "作为风险管理专家，分析当前市场数据并评估交易风险。请提供：\n- risk_level: 风险等级（low/medium/high）\n- volatility_score: 波动率评分（0-100）\n- max_position_size: 建议最大仓位百分比\n- recommendation: 风险管理建议",
  "parameters": [
    {
      "name": "temperature",
      "value": "0.2"
    }
  ]
}
```

---

## API 配置

### 环境变量

| 环境变量 | 说明 | 必填 |
|---------|------|------|
| `OPENROUTER_API_KEY` | OpenRouter API 密钥 | ✅ |
| `OPENROUTER_SITE_URL` | 站点 URL（用于流量统计） | ❌ |
| `OPENROUTER_SITE_NAME` | 站点名称 | ❌ |

**获取 API Key：**
1. 访问 [OpenRouter](https://openrouter.ai/)
2. 注册账号并创建 API Key
3. 将 API Key 添加到环境变量

### API 限制

| 限制类型 | 说明 |
|---------|------|
| **请求频率** | 根据 OpenRouter 套餐而定 |
| **Token 限制** | 根据选择的模型而定（GPT-3.5: 4K/16K, GPT-4: 8K/32K） |
| **超时时间** | 节点设置为 60 秒 |
| **成本** | 按 Token 使用量计费 |

---

## 最佳实践

### 1. 提示词工程

✅ **推荐做法：**
```text
你是一位加密货币交易专家。请分析以下数据：

【任务】判断是否应该买入 BTC
【输入】最近 24 小时的价格走势和社交媒体情绪
【输出】买入建议（是/否）、建议金额（USDT）、置信度（0-1）

请提供 JSON 格式的输出。
```

❌ **避免：**
```text
看一下这个数据，给我一些建议。
```

### 2. 温度参数选择

| 任务类型 | 推荐 Temperature | 说明 |
|---------|-----------------|------|
| 数据提取 | 0.0 - 0.3 | 需要确定性输出 |
| 分析判断 | 0.3 - 0.7 | 平衡创造性和准确性 |
| 创意生成 | 0.7 - 1.0 | 需要多样化输出 |

### 3. 成本优化

- **使用 GPT-3.5**：适用于简单的数据提取和格式转换
- **使用 GPT-4**：适用于复杂的分析和决策任务
- **控制 max_tokens**：根据实际需求设置合理值
- **优化提示词**：简洁明确的提示词可减少 token 消耗

### 4. 错误处理

节点自动处理以下情况：
- API 调用失败 → 节点状态设为 FAILED
- 响应格式错误 → 返回原始响应文本
- 结构化数据提取失败 → 记录警告，继续执行

---

## 注意事项

### ⚠️ 重要提示

1. **API Key 安全**
   - 不要在代码中硬编码 API Key
   - 使用环境变量管理敏感信息
   - 定期轮换 API Key

2. **成本控制**
   - GPT-4 成本是 GPT-3.5 的约 15-20 倍
   - 监控 API 使用量和费用
   - 为开发和生产环境设置不同的配额

3. **响应时间**
   - GPT-4 响应时间通常较长（5-20 秒）
   - 设置合理的超时时间
   - 考虑使用异步工作流

4. **输出可靠性**
   - AI 输出可能包含错误或偏见
   - 始终验证结构化数据的完整性
   - 建议添加 Condition Node 进行输出验证

5. **提示词注入风险**
   - 谨慎处理用户输入的文本
   - 避免直接将不可信内容添加到提示词
   - 考虑对输入进行清理和验证

---

## 故障排查

**Q: AI 响应不包含期望的 JSON 格式？**

A: 检查以下几点：
1. 确认 `auto_format_output` 已启用
2. 提示词中明确要求 JSON 输出
3. 增加 `max_tokens` 参数值
4. 降低 `temperature` 提高确定性
5. 查看节点日志中的原始响应

---

**Q: 节点执行失败，显示 API 错误？**

A: 
1. 验证 `OPENROUTER_API_KEY` 环境变量是否正确
2. 检查 API Key 是否有足够的余额
3. 确认网络连接正常
4. 查看错误日志获取详细信息

---

**Q: 输出的结构化数据字段缺失？**

A:
1. 检查提示词是否清晰说明了所需字段
2. 尝试提供输出示例
3. 增加 `max_tokens` 确保完整响应
4. 使用更强大的模型（GPT-4）

---

## 技术规格

| 规格项 | 值 |
|--------|-----|
| **节点版本** | 1.0.0 |
| **支持的 API** | OpenRouter API (OpenAI Compatible) |
| **默认超时** | 60 秒 |
| **最大并发** | 根据 API 限制 |
| **执行模式** | 单次执行（处理一次信号后完成） |
| **日志级别** | DEBUG, INFO, WARNING, ERROR |

---

## 相关节点

- **Code Node** - 用于数据预处理和后处理
- **Condition Node** - 根据 AI 输出做条件判断
- **Buy/Sell Node** - 执行 AI 推荐的交易决策
- **Dataset Output Node** - 保存 AI 分析结果

---

**相关文档：**
- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [Code Node](code-node.md) - Python 代码执行节点
- [Weather 语法](../core-concepts/weather-syntax.md) - 工作流文件格式

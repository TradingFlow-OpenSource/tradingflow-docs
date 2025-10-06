# AI Model Node

**节点类型：** `ai_model_node`  
**版本：** 1.0.0  
**分类：** AI 处理节点  
**文件位置：** `tradingflow/station/nodes/ai_model_node.py`

---

## 节点概述

AI Model Node 是一个智能 AI 处理节点，能够接收各种类型的输入信号作为上下文，调用大语言模型（LLM）进行分析和决策，并根据下游节点需求自动生成结构化的 JSON 输出。

### 主要功能

- 调用多种 LLM（OpenRouter API）
- 自动上下文构建（从上游信号）
- **智能输出格式化**：根据下游节点连接自动生成 JSON 格式要求
- 支持流式和非流式响应
- 完整的错误处理和重试机制

---

## 输入输出

### 输入句柄

| 句柄名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `model` | string | 否 | 模型名称，可覆盖配置中的模型 |
| `prompt` | string | 否 | 提示词，可覆盖配置中的提示词 |
| `parameters` | object | 否 | 模型参数（temperature, max_tokens 等） |
| **任意输入** | any | 否 | 其他输入会自动转换为上下文 |

### 输入参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `model_name` | string | `"gpt-3.5-turbo"` | LLM 模型名称 |
| `temperature` | float | `0.7` | 温度参数，控制随机性（0.0-2.0） |
| `system_prompt` | string | `"You are a helpful assistant."` | 系统提示词 |
| `prompt` | string | `"Please analyze..."` | 主提示词 |
| `max_tokens` | number | `1000` | 最大返回 token 数 |
| `auto_format_output` | boolean | `true` | 是否自动生成 JSON 格式要求 |

#### 支持的模型

通过 OpenRouter API 支持：

| 模型系列 | 模型名称示例 |
|---------|------------|
| **OpenAI** | `gpt-3.5-turbo`, `gpt-4`, `gpt-4-turbo` |
| **Anthropic** | `claude-3-opus`, `claude-3-sonnet` |
| **DeepSeek** | `deepseek-r1`, `deepseek-chat` |
| **Google** | `gemini-pro`, `gemini-1.5-pro` |
| **Meta** | `llama-3-70b`, `llama-3.1-405b` |

### 输出句柄

**ai_response** (SignalType.AI_RESPONSE)

输出 AI 模型的响应数据，格式根据下游节点自动调整。

**标准格式：**
```json
{
  "response": "AI 的文本响应",
  "model": "gpt-4",
  "usage": {
    "prompt_tokens": 150,
    "completion_tokens": 200,
    "total_tokens": 350
  }
}
```

**智能格式化示例：**

当连接到 SwapNode 时，自动提取结构化数据：
```json
{
  "chain_id": "aptos",
  "from_token": "USDT",
  "to_token": "BTC",
  "amount_in_percentage": 25.0
}
```

---

## 核心功能

### 1. 自动上下文构建

节点会自动收集所有上游输入信号，构建为 LLM 的上下文：

```python
async def _prepare_context(self) -> str:
    """准备上下文"""
    context_parts = [self.prompt]
    
    # 1. 添加所有输入信号的内容
    for handle_name, handle_obj in self._input_handles.items():
        attr_name = handle_obj.auto_update_attr
        if hasattr(self, attr_name):
            value = getattr(self, attr_name)
            if value is not None:
                context_parts.append(f"\n{handle_name}: {json.dumps(value, ensure_ascii=False)}")
    
    # 2. 如果启用了自动格式化，添加输出格式要求
    if self.auto_format_output:
        format_requirements = self._analyze_output_format_requirements()
        if format_requirements:
            context_parts.append("\n\n输出格式要求:")
            context_parts.append(format_requirements)
    
    return "\n".join(context_parts)
```

### 2. 智能输出格式化 ⭐

根据输出边连接情况，自动生成 JSON 格式要求：

```python
def _analyze_output_format_requirements(self) -> str:
    """分析输出格式要求"""
    # 1. 获取所有输出边
    output_edges = self._output_edges
    
    # 2. 识别目标节点需要的字段
    required_fields = {}
    for edge in output_edges:
        target_handle = edge.target_node_handle
        
        # 根据句柄名称推断字段类型
        if 'chain' in target_handle.lower():
            required_fields['chain_id'] = "区块链网络标识符 (例如: 'aptos')"
        elif 'amount' in target_handle.lower():
            required_fields['amount_in_percentage'] = "交易金额百分比 (例如: 25.0)"
        elif 'from_token' in target_handle.lower():
            required_fields['from_token'] = "源代币符号 (例如: 'USDT')"
        elif 'to_token' in target_handle.lower():
            required_fields['to_token'] = "目标代币符号 (例如: 'BTC')"
        # ... 更多字段映射
    
    # 3. 生成格式说明
    if required_fields:
        format_text = "请确保你的回复包含一个符合以下格式的JSON对象：\n\n"
        format_text += "{\n"
        for field, desc in required_fields.items():
            format_text += f'  "{field}": {desc},\n'
        format_text += "}\n"
        return format_text
    
    return ""
```

### 3. 结构化数据提取

从 AI 响应中智能提取结构化数据：

```python
def _extract_structured_data_from_response(self, response_text: str) -> Dict:
    """从响应中提取结构化数据"""
    # 策略 1: 提取 JSON 代码块
    json_match = re.search(r'```json\s*\n(.*?)\n```', response_text, re.DOTALL)
    if json_match:
        try:
            return json.loads(json_match.group(1))
        except json.JSONDecodeError:
            pass
    
    # 策略 2: 提取 JSON 对象
    json_match = re.search(r'\{[^{}]*(?:\{[^{}]*\}[^{}]*)*\}', response_text)
    if json_match:
        try:
            return json.loads(json_match.group(0))
        except json.JSONDecodeError:
            pass
    
    # 策略 3: 文本模式匹配
    extracted_data = {}
    for field in self.expected_fields:
        pattern = rf'["\']?{field}["\']?\s*[:=]\s*["\']?([^",\n]+)["\']?'
        match = re.search(pattern, response_text, re.IGNORECASE)
        if match:
            extracted_data[field] = match.group(1).strip()
    
    return extracted_data if extracted_data else None
```

---

## 执行流程

### 完整执行流程

```
1. 等待所有输入信号就绪
    ↓
2. 构建上下文 (_prepare_context)
    ├─> 收集所有输入数据
    ├─> 分析输出格式要求
    └─> 生成完整提示词
    ↓
3. 调用 LLM API (_call_llm_api)
    ├─> 构建请求参数
    ├─> 发送 HTTP 请求
    └─> 处理响应（流式/非流式）
    ↓
4. 提取结构化数据 (_extract_structured_data)
    ├─> JSON 代码块提取
    ├─> JSON 对象提取
    └─> 文本模式匹配
    ↓
5. 发送输出信号
    └─> 包含原始响应和提取的数据
    ↓
6. 更新节点状态为完成
```

---

## 使用示例

### 示例 1：价格分析决策

```json
{
  "nodes": [
    {
      "id": "binance_price",
      "type": "binance_price_node",
      "config": {
        "symbol": "BTCUSDT",
        "interval": "1h",
        "limit": 50
      }
    },
    {
      "id": "ai_analyzer",
      "type": "ai_model_node",
      "config": {
        "model_name": "gpt-4",
        "system_prompt": "You are a crypto trading analyst.",
        "prompt": "Analyze the BTC price trend and decide whether to buy or sell.",
        "temperature": 0.3,
        "auto_format_output": true
      }
    },
    {
      "id": "swap_executor",
      "type": "swap_node",
      "config": {
        "chain": "aptos"
      }
    }
  ],
  "edges": [
    {
      "source": "binance_price",
      "sourceHandle": "kline_data",
      "target": "ai_analyzer",
      "targetHandle": "price_data"
    },
    {
      "source": "ai_analyzer",
      "sourceHandle": "ai_response",
      "target": "swap_executor",
      "targetHandle": "chain_id"
    },
    {
      "source": "ai_analyzer",
      "sourceHandle": "ai_response",
      "target": "swap_executor",
      "targetHandle": "amount_in_percentage"
    }
  ]
}
```

**AI 自动生成的格式要求：**
```
输出格式要求:
请确保你的回复包含一个符合以下格式的JSON对象：

{
  "chain_id": "aptos",
  "amount_in_percentage": 25.0,
  "from_token": "USDT",
  "to_token": "BTC"
}

字段说明：
- chain_id: 区块链网络标识符
- amount_in_percentage: 交易金额百分比
- from_token: 源代币符号
- to_token: 目标代币符号
```

### 示例 2：多源数据聚合分析

```json
{
  "nodes": [
    {
      "id": "price_node",
      "type": "binance_price_node"
    },
    {
      "id": "news_node",
      "type": "rsshub_node"
    },
    {
      "id": "social_node",
      "type": "x_listener_node"
    },
    {
      "id": "ai_decision",
      "type": "ai_model_node",
      "config": {
        "model_name": "gpt-4-turbo",
        "prompt": "综合分析价格、新闻和社交媒体情绪，给出交易建议。",
        "auto_format_output": true
      }
    }
  ],
  "edges": [
    {
      "source": "price_node",
      "sourceHandle": "kline_data",
      "target": "ai_decision",
      "targetHandle": "price_data"
    },
    {
      "source": "news_node",
      "sourceHandle": "news_data",
      "target": "ai_decision",
      "targetHandle": "news_data"
    },
    {
      "source": "social_node",
      "sourceHandle": "tweets",
      "target": "ai_decision",
      "targetHandle": "social_data"
    }
  ]
}
```

---

## 高级配置

### 模型参数调优

```python
{
  "model_name": "gpt-4",
  "temperature": 0.3,      # 降低随机性，更稳定的输出
  "max_tokens": 2000,      # 增加输出长度限制
  "top_p": 0.9,           # Nucleus sampling
  "frequency_penalty": 0.5,# 减少重复
  "presence_penalty": 0.5  # 鼓励新话题
}
```

### 自定义格式提取

```python
# 在节点配置中指定期望的字段
{
  "auto_format_output": true,
  "expected_output_fields": [
    "action",      # buy/sell/hold
    "confidence",  # 0.0-1.0
    "reason"       # 文本说明
  ]
}
```

---

## 错误处理

### 常见错误

| 错误类型 | 原因 | 解决方案 |
|---------|------|----------|
| `API Key Invalid` | OpenRouter API Key 无效 | 检查环境变量 `OPENROUTER_API_KEY` |
| `Rate Limit Exceeded` | 请求频率过高 | 增加请求间隔或升级套餐 |
| `Timeout` | API 响应超时 | 增加 `timeout` 参数 |
| `JSON Parse Error` | 无法提取结构化数据 | 检查提示词，确保要求 JSON 格式 |
| `Model Not Found` | 模型名称错误 | 查看支持的模型列表 |

### 重试机制

节点内置自动重试：
- 超时：最多重试 2 次
- 速率限制：指数退避重试
- 网络错误：立即重试一次

---

## 性能优化

### API 成本优化

1. **选择合适的模型**：
   - 简单任务：`gpt-3.5-turbo`（便宜）
   - 复杂分析：`gpt-4`（准确）
   - 推理任务：`deepseek-r1`（性价比高）

2. **控制 token 使用**：
   - 精简提示词
   - 限制上下文长度
   - 设置合理的 `max_tokens`

3. **缓存响应**：
   - 对于相同输入，缓存 AI 响应
   - 使用 Redis 存储缓存

### 响应速度优化

1. **使用流式响应**：对于长文本生成
2. **并行请求**：多个独立的 AI 节点可并行执行
3. **选择快速模型**：`gpt-3.5-turbo` 比 `gpt-4` 快 5-10 倍

---

## 配置要求

### 环境变量

```bash
# 必需
OPENROUTER_API_KEY=your_openrouter_api_key_here

# 可选
OPENROUTER_SITE_URL=https://tradingflows.ai
OPENROUTER_SITE_NAME=TradingFlow
```

### 依赖包

```python
httpx>=0.25.0      # 异步 HTTP 客户端
```

---

## 相关文档

- [节点开发指南](../weather-station-node-execution.md#开发新节点)
- [消息队列详解](../weather-station-message-queue.md)
- [Swap Node](swap-node.md)
- [OpenRouter API 文档](https://openrouter.ai/docs)

---

**维护者：** TradingFlow 开发团队  
**最后更新：** 2025-10-06

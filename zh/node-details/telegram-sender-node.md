# Telegram Sender Node

Telegram Sender Node 是 TradingFlow 的通知和消息发送节点，用于将工作流的执行结果、交易通知和重要信息发送到 Telegram。节点支持富文本格式、消息前缀和自动重试机制。

---

## 节点信息

| 属性         | 值                     |
| ------------ | ---------------------- |
| **节点类型** | `telegram_sender_node` |
| **显示名称** | Telegram Sender        |
| **节点分类** | Core（核心功能）       |
| **图标**     | ✈️ Telegram 图标       |
| **句柄颜色** | Rose（玫瑰红）         |

---

## 功能说明

Telegram Sender Node 通过 Telegram Bot API 发送消息到指定的聊天或频道。节点自动格式化消息内容，支持 HTML/Markdown 格式，并具有完善的错误处理和重试机制。

**主要用途：**

- 发送交易执行通知
- 报告工作流执行状态
- 推送价格预警
- 发送 AI 分析结果
- 记录重要事件

**核心特性：**

- 📱 **Telegram Bot API**：官方 API 集成
- 💬 **富文本支持**：HTML、Markdown 格式
- 🔄 **自动重试**：失败自动重试，指数退避
- 📝 **消息前缀**：自定义消息标识
- ⚙️ **灵活配置**：静音、预览等选项

---

## 输入参数

### 参数列表

| 参数                       | 类型      | 必填 | 默认值  | 说明               |
| -------------------------- | --------- | ---- | ------- | ------------------ |
| `bot_token`                | text      | ✅   | -       | Telegram Bot Token |
| `chat_id`                  | text      | ✅   | -       | 聊天 ID            |
| `message`                  | paragraph | ✅   | -       | 消息内容           |
| `message_prefix`           | text      | ❌   | `""`    | 消息前缀           |
| `parse_mode`               | select    | ❌   | `HTML`  | 解析模式           |
| `disable_web_page_preview` | boolean   | ❌   | `true`  | 禁用网页预览       |
| `disable_notification`     | boolean   | ❌   | `false` | 静音发送           |
| `timeout`                  | number    | ❌   | `30`    | 请求超时（秒）     |
| `retry_count`              | number    | ❌   | `3`     | 重试次数           |

### bot_token 参数

**获取方式：**

1. 在 Telegram 中搜索 `@BotFather`
2. 发送 `/newbot` 创建新 Bot
3. 按提示设置 Bot 名称和用户名
4. 获取 Bot Token（如 `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`）

**配置方式：**

- **环境变量**：`TELEGRAM_BOT_TOKEN=your_token_here`
- **参数传递**：直接在节点配置中提供

**安全建议：**

- 使用环境变量存储
- 不要将 Token 提交到代码仓库
- 定期更新 Token

### chat_id 参数

**获取方式：**

**方法 1 - 使用 @userinfobot：**

1. 在 Telegram 中搜索 `@userinfobot`
2. 点击 Start
3. Bot 会返回您的 User ID

**方法 2 - 使用 getUpdates API：**

1. 向 Bot 发送任意消息
2. 访问：`https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
3. 在返回的 JSON 中查找 `chat.id`

**格式：**

- 个人聊天：数字 ID（如 `123456789`）
- 群组：负数 ID（如 `-987654321`）
- 频道：`@channel_username` 或频道 ID

### message 参数

**来源：** 上游节点输出或直接输入

**支持格式：**

- 纯文本字符串
- JSON 对象（自动格式化）
- 复杂数据结构（自动转换）

**最大长度：** 4096 字符（Telegram 限制）

### parse_mode 参数

**支持的模式：**

| 模式           | 值           | 说明             | 示例                        |
| -------------- | ------------ | ---------------- | --------------------------- |
| **HTML**       | `HTML`       | HTML 标签格式    | `<b>Bold</b> <i>Italic</i>` |
| **Markdown**   | `Markdown`   | Markdown 格式    | `**Bold** *Italic*`         |
| **MarkdownV2** | `MarkdownV2` | Markdown V2 格式 | `*Bold* _Italic_`           |

**HTML 标签示例：**

```html
<b>粗体</b>
<i>斜体</i>
<u>下划线</u>
<code>代码</code>
<pre>预格式化</pre>
<a href="https://example.com">链接</a>
```

**Markdown 示例：**

```markdown
**粗体**
_斜体_
[链接](https://example.com)
`代码`
```

### message_prefix 参数

**用途：** 为所有消息添加统一前缀

**示例：**

```
[TradingFlow Bot]
[交易通知]
🤖 AI Assistant:
```

**效果：**

```
[TradingFlow Bot]

交易执行成功
买入 100 APT
```

### disable_web_page_preview 参数

**说明：** 消息中的链接是否显示预览

**选择：**

- `true`：禁用预览（推荐，保持消息简洁）
- `false`：启用预览

### disable_notification 参数

**说明：** 是否静音发送消息

**选择：**

- `true`：静音（接收者不会收到通知音）
- `false`：正常通知

**使用场景：**

- `true`：日志、非紧急通知
- `false`：重要警报、交易通知

---

## 输出参数

### 输出列表

| 输出 ID  | 显示名称 | 数据类型 | 说明                                               |
| -------- | -------- | -------- | -------------------------------------------------- |
| `result` | Result   | object   | 完整的发送结果，包含成功状态、消息信息和错误详情 |

### result 输出

**数据类型：** `object`

**完整数据结构：**

```json
{
  "success": true,              // 发送是否成功
  "message_id": 12345,          // Telegram 消息 ID（成功时）
  "chat_id": "123456789",       // 目标聊天 ID
  "timestamp": 1234567890.0,    // 发送时间戳
  "execution_time": 0.5,        // 执行耗时（秒）
  "error": null                 // 错误信息（失败时不为 null）
}
```

**成功示例：**

```json
{
  "success": true,
  "message_id": 12345,
  "chat_id": "123456789",
  "timestamp": 1234567890.0,
  "execution_time": 0.5,
  "error": null
}
```

**失败示例：**

```json
{
  "success": false,
  "error": "Failed to send message: Invalid bot token",
  "timestamp": 1234567890.0,
  "execution_time": 0.3,
  "message_id": null,
  "chat_id": "123456789"
}
```

**说明：**

- **统一输出**：成功和失败都使用同一个 `result` 输出
- **success 字段**：布尔值，指示发送是否成功
- **条件字段**：
  - 成功时：`message_id` 包含 Telegram 返回的消息 ID，`error` 为 `null`
  - 失败时：`error` 包含详细错误信息，`message_id` 为 `null`
- **通用字段**：`timestamp`、`execution_time`、`chat_id` 始终存在

---
---

## 使用示例

### 示例 1：交易成功通知

**场景：** 交易执行后发送成功通知。

**工作流结构：**

```
Vault Node (获取金库)
    ↓ vault_address
Swap Node (执行交换)
    ↓ trade_receipt { success, tx_hash, amount_in, amount_out }
Code Node (格式化消息)
    ↓ message
Telegram Sender Node (发送通知)
```

**Code Node 输出：**

```python
# Python code in Code Node
tx = trade_receipt

if tx['success']:
    message = f"""
<b>✅ 交易成功</b>

<b>交易详情：</b>
• 类型：{tx['from_token']} → {tx['to_token']}
• 输入：{tx['amount_in']} {tx['from_token']}
• 输出：{tx['amount_out']} {tx['to_token']}
• 交易哈希：<code>{tx['tx_hash'][:16]}...</code>

⏰ {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
"""
else:
    message = f"❌ 交易失败：{tx.get('message', 'Unknown error')}"

output_data = message
```

**Telegram Sender Node 配置：**

```json
{
  "message_prefix": "🤖 TradingFlow Bot",
  "parse_mode": "HTML",
  "disable_notification": false
}
```

---

### 示例 2：价格预警

**场景：** 价格达到目标时发送警报。

**工作流结构：**

```
Binance Price Node (获取价格)
    ↓ kline_data { close_price }
Code Node (检查是否达到目标价)
    ↓ alert_message
Condition Node (需要警报?)
    ↓ (true)
Telegram Sender Node (发送警报)
```

**alert_message 示例：**

```
⚠️ 价格警报

BTC 价格已达到 $50,000
当前价格：$50,123
涨幅：+5.2% (24h)

建议：考虑获利了结
```

---

### 示例 3：AI 分析报告

**场景：** 发送 AI 分析结果。

**工作流结构：**

```
X Listener Node (获取推文)
    ↓ latest_tweets
AI Model Node (情绪分析)
    ↓ ai_response
Code Node (生成报告)
    ↓ report
Telegram Sender Node (发送报告)
```

**report 示例：**

```html
<b>📊 市场情绪分析报告</b>

<b>数据来源：</b>
• 推文数量：50 • 分析账户：@elonmusk, @VitalikButerin • 关键词：Bitcoin, BTC

<b>情绪评分：</b>
• 总体情绪：<b>看涨 (0.75)</b>
• 积极：65% • 中立：25% • 消极：10%

<b>关键观点：</b>
1. 机构采用加速 2. 技术面突破阻力位 3. 监管环境改善

<b>交易建议：</b>
考虑适度增加持仓
```

---

### 示例 4：工作流执行日志

**场景：** 记录工作流执行情况。

**Code Node 输出：**

```python
log_message = f"""
<b>🔄 工作流执行完成</b>

<b>执行信息：</b>
• 工作流：Trading Strategy Alpha
• 执行时间：{execution_time:.2f}s
• 节点数：5
• 状态：✅ 全部成功

<b>节点详情：</b>
✅ Vault Node: 1.2s
✅ Swap Node: 3.5s
✅ AI Model Node: 2.1s
✅ Code Node: 0.3s
✅ Telegram Sender: 0.5s

<i>下次执行：30分钟后</i>
"""
```

---

## Bot 配置

### 1. 创建 Telegram Bot

**步骤：**

1. 在 Telegram 中搜索 `@BotFather`
2. 发送 `/newbot`
3. 设置 Bot 名称（显示名称）
4. 设置 Bot 用户名（必须以 `bot` 结尾）
5. 获取 Bot Token
6. (可选) 设置 Bot 头像和描述

**示例对话：**

```
You: /newbot
BotFather: Alright, a new bot. How are we going to call it?
You: TradingFlow Alert Bot
BotFather: Good. Now let's choose a username for your bot.
You: tradingflow_alert_bot
BotFather: Done! Your token is: 123456:ABC-DEF...
```

### 2. 获取 Chat ID

**个人聊天：**

1. 向您的 Bot 发送 `/start`
2. 访问：`https://api.telegram.org/bot<TOKEN>/getUpdates`
3. 查找 `"chat":{"id":123456789}`

**群组聊天：**

1. 将 Bot 添加到群组
2. 在群组中发送消息
3. 使用相同方法获取 Chat ID（负数）

**频道：**

1. 将 Bot 添加为频道管理员
2. Chat ID 为 `@channel_username` 或频道 ID

### 3. 配置环境变量

```bash
export TELEGRAM_BOT_TOKEN="123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11"
export TELEGRAM_CHAT_ID="123456789"
```

---

## 最佳实践

### 1. 消息格式化

**✅ 推荐：**

```python
# 使用 HTML 格式，结构清晰
message = """
<b>标题</b>

<b>关键信息：</b>
• 项目1
• 项目2

<i>补充说明</i>
"""
```

**❌ 避免：**

```python
# 过长的单行消息
message = "很长很长的消息内容，没有换行和格式..."

# 混乱的格式
message = "标题 项目1 项目2 结束"
```

### 2. 错误处理

**在 Code Node 中：**

```python
try:
    # 处理数据
    result = process_data()
    message = f"✅ 成功：{result}"
except Exception as e:
    message = f"❌ 错误：{str(e)}"

output_data = message
```

### 3. 消息长度控制

```python
# 检查消息长度
if len(message) > 4000:
    message = message[:4000] + "\n\n[消息已截断]"
```

### 4. 使用表情符号

**提高可读性：**

- ✅ 成功
- ❌ 失败
- ⚠️ 警告
- 📊 数据/统计
- 💰 金钱/交易
- 🤖 AI/自动化
- ⏰ 时间

---

## 注意事项

### ⚠️ 重要提示

1. **Bot Token 安全**

   - 不要公开 Token
   - 不要提交到代码仓库
   - 使用环境变量配置

2. **消息长度限制**

   - 最大 4096 字符
   - 超长消息会被截断
   - 建议控制在 4000 字符以内

3. **发送频率限制**

   - 每秒最多 30 条消息
   - 同一群组每分钟 20 条消息
   - 超限会被暂时封禁

4. **HTML/Markdown 转义**

   - 特殊字符需要转义
   - HTML: `<`, `>`, `&`
   - Markdown: `_`, `*`, `[`, `]`, `(`, `)`, `~`, `` ` ``, `>`, `#`, `+`, `-`, `=`, `|`, `{`, `}`, `.`, `!`

5. **重试机制**
   - 默认重试 3 次
   - 使用指数退避策略
   - 失败后自动重试

---

## 故障排查

**Q: 提示 "Bot token is required"？**

A:

1. 确认环境变量 `TELEGRAM_BOT_TOKEN` 已设置
2. 或在节点配置中提供 `bot_token` 参数
3. 检查 Token 格式是否正确

---

**Q: 提示 "Chat not found"？**

A:

1. 确认 Chat ID 正确
2. 个人聊天需要先向 Bot 发送 `/start`
3. 群组需要将 Bot 添加为成员
4. 频道需要 Bot 为管理员

---

**Q: 消息格式显示异常？**

A:

1. 检查 `parse_mode` 设置
2. 确认 HTML/Markdown 语法正确
3. 特殊字符需要转义
4. 尝试使用 `parse_mode: "HTML"`

---

**Q: 消息发送失败？**

A:

1. 检查网络连接
2. 确认 Bot Token 有效
3. 检查消息长度（<4096 字符）
4. 查看错误日志

---

**Q: 如何发送到多个聊天？**

A:

```
方案1：使用多个 Telegram Sender Node
Node 1 → Chat ID 1
Node 2 → Chat ID 2

方案2：在 Code Node 中循环发送
(需要自定义实现)
```

---

## 技术规格

| 规格项           | 值                         |
| ---------------- | -------------------------- |
| **节点版本**     | 1.0.0                      |
| **API**          | Telegram Bot API           |
| **最大消息长度** | 4096 字符                  |
| **默认超时**     | 30 秒                      |
| **默认重试次数** | 3 次                       |
| **重试策略**     | 指数退避                   |
| **支持格式**     | HTML, Markdown, MarkdownV2 |

---

## 相关节点

- **Code Node** - 格式化消息内容
- **AI Model Node** - 生成智能消息
- **Dataset Output Node** - 同时保存到 Google Sheets
- **Condition Node** - 条件发送通知

---

**相关文档：**

- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [Code Node](code-node.md) - 消息格式化
- [Telegram Bot API](https://core.telegram.org/bots/api) - 官方 API 文档

# X Listener Node

X Listener Node（Twitter 监听节点）是 TradingFlow 的数据输入节点，用于从 X（Twitter）平台获取用户推文和进行高级搜索。节点支持关键词过滤、多账户监听和实时数据采集，是社交媒体情绪分析的核心组件。

---

## 节点信息

| 属性 | 值 |
|------|-----|
| **节点类型** | `x_listener_node` |
| **显示名称** | X Listener |
| **节点分类** | Input（数据输入） |
| **图标** | 🐦 Twitter 图标 |
| **句柄颜色** | Sky（天蓝色） / Emerald（绿色） |

---

## 功能说明

X Listener Node 连接到 X（Twitter）平台，实时获取指定账户的推文或执行高级搜索。节点支持关键词过滤、多账户并行监听，并将推文数据传递给下游分析节点。

**主要用途：**
- 监听KOL（关键意见领袖）推文
- 追踪特定话题和关键词
- 收集市场情绪数据
- 社交媒体信号捕捉
- 舆情监控和分析

**核心特性：**
- 🔍 **双模式搜索**：用户推文模式和高级搜索模式
- 🏷️ **关键词过滤**：支持多关键词匹配和过滤
- 👥 **多账户支持**：同时监听多个账户
- 📄 **分页获取**：自动处理大量推文的分页
- ⚡ **速率限制**：内置API调用速率控制

---

## 输入参数

### 参数列表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `accounts` | paragraph | ❌* | `[]` | X 账户列表 |
| `keywords` | paragraph | ❌ | `""` | 关键词过滤 |
| `search_mode` | select | ✅ | `user_tweets` | 搜索模式 |
| `query_type` | select | ✅ | `Latest` | 搜索类型 |
| `limit` | number | ✅ | `20` | 推文数量限制 |
| `api_key` | text | ✅ | - | Twitter API 密钥 |

\* `accounts` 在 `user_tweets` 模式下必填

### accounts 参数

**格式：** 逗号分隔的账户列表

**支持格式：**
- **用户名**：`elonmusk, VitalikButerin`
- **用户ID**：`44196397, 295218901`（纯数字）
- **混合格式**：`elonmusk, 295218901`

**示例：**
```
elonmusk, VitalikButerin, cz_binance
```

**说明：**
- 用户名不需要 `@` 符号
- 支持 userId 和 userName 两种格式
- 多个账户用逗号分隔
- 空格会被自动去除

### keywords 参数

**格式：** 逗号分隔的关键词列表

**匹配规则：**
- 不区分大小写
- 部分匹配（包含即可）
- OR 逻辑（任一关键词匹配）

**示例：**
```
Bitcoin, BTC, crypto
```

**工作方式：**
```
推文内容："Bitcoin is going to the moon! 🚀"
关键词："Bitcoin, ETH, crypto"
结果：✅ 匹配（包含 Bitcoin）

推文内容："Great weather today"
关键词："Bitcoin, ETH, crypto"
结果：❌ 不匹配
```

### search_mode 参数

**支持的模式：**

| 模式 | 值 | 说明 | 必需参数 |
|------|-----|------|---------|
| **用户推文** | `user_tweets` | 获取指定用户的推文 | accounts |
| **高级搜索** | `advanced_search` | 基于关键词全局搜索 | keywords 或 accounts |

#### user_tweets 模式

**特点：**
- 从指定用户的时间线获取推文
- 支持多个账户并行获取
- 可选关键词过滤
- 按时间排序

**使用场景：**
- 监听特定KOL的推文
- 追踪特定账户动态
- 收集用户历史推文

#### advanced_search 模式

**特点：**
- 全局搜索包含关键词的推文
- 可限定特定用户（from: 操作符）
- 支持Latest（最新）和Top（热门）
- 搜索范围更广

**使用场景：**
- 追踪热门话题
- 发现讨论特定主题的推文
- 市场情绪分析

### query_type 参数

**仅用于 advanced_search 模式**

| 类型 | 值 | 说明 |
|------|-----|------|
| **最新** | `Latest` | 按时间排序的最新推文 |
| **热门** | `Top` | 按热度排序的热门推文 |

### limit 参数

**范围：** 1-100

**说明：**
- 限制返回的推文总数
- 防止数据过载
- 跨所有账户的总数限制

**示例：**
```json
{
  "limit": 50,
  "accounts": ["elonmusk", "VitalikButerin"]
}
// 最多返回50条推文（两个账户合计）
```

### api_key 参数

**获取方式：**
1. 注册 TwitterAPI.io 账户
2. 获取 API Key
3. 配置环境变量 `TWITTER_API_KEY`

**配置方式：**
- **环境变量**：`TWITTER_API_KEY=your_key_here`
- **参数传递**：直接在节点配置中提供

---

## 输出参数

### 输出列表

| 输出 ID | 显示名称 | 数据类型 | 说明 |
|---------|---------|---------|------|
| `latest_tweets` | Latest Tweets | object | 推文数据集合 |

### latest_tweets 输出

**数据类型：** `object`

**完整数据结构：**

```typescript
{
  tweets: Array<{              // 推文列表
    id: string;                // 推文ID
    text: string;              // 推文文本
    created_at: string;        // 创建时间
    author: {                  // 作者信息
      id: string;
      username: string;
      name: string;
    };
    metrics: {                 // 互动数据
      like_count: number;
      retweet_count: number;
      reply_count: number;
    };
    source_account?: string;   // 来源账户（多账户时）
  }>;
  count: number;               // 推文数量
  accounts: string[];          // 监听的账户列表
  keywords: string;            // 使用的关键词
  search_mode: string;         // 搜索模式
  query_type: string;          // 查询类型
  timestamp: number;           // 获取时间戳
  next_cursor: string;         // 分页游标
}
```

**示例输出（user_tweets 模式）：**

```json
{
  "tweets": [
    {
      "id": "1234567890",
      "text": "Bitcoin is breaking new highs! 🚀",
      "created_at": "2024-01-15T10:30:00Z",
      "author": {
        "id": "44196397",
        "username": "elonmusk",
        "name": "Elon Musk"
      },
      "metrics": {
        "like_count": 125000,
        "retweet_count": 25000,
        "reply_count": 5000
      },
      "source_account": "elonmusk"
    }
  ],
  "count": 15,
  "accounts": ["elonmusk", "VitalikButerin"],
  "keywords": "Bitcoin, crypto",
  "search_mode": "user_tweets",
  "query_type": "Latest",
  "timestamp": 1705316400,
  "next_cursor": "abc123..."
}
```

**示例输出（advanced_search 模式）：**

```json
{
  "tweets": [
    {
      "id": "9876543210",
      "text": "The future of crypto is decentralized finance #DeFi",
      "created_at": "2024-01-15T11:00:00Z",
      "author": {
        "id": "295218901",
        "username": "VitalikButerin",
        "name": "Vitalik Buterin"
      },
      "metrics": {
        "like_count": 50000,
        "retweet_count": 10000,
        "reply_count": 2000
      }
    }
  ],
  "count": 20,
  "accounts": [],
  "keywords": "DeFi, crypto",
  "search_mode": "advanced_search",
  "query_type": "Latest",
  "timestamp": 1705318200
}
```

---

## 工作流程

### 执行流程

```
1. 参数验证
   ├─ 检查 API Key
   ├─ 验证搜索模式
   └─ 确认必需参数

2. 模式选择
   ├─ user_tweets: 用户推文模式
   └─ advanced_search: 高级搜索模式

3. 数据获取
   ├─ 构建API请求
   ├─ 执行HTTP调用
   ├─ 处理分页
   └─ 应用速率限制

4. 关键词过滤（可选）
   ├─ 解析关键词列表
   ├─ 不区分大小写匹配
   └─ 过滤推文

5. 数据聚合
   ├─ 合并多账户结果
   ├─ 按时间排序
   └─ 应用数量限制

6. 构建输出
   ├─ 格式化推文数据
   ├─ 添加元数据
   └─ 生成输出对象

7. 发送信号
   └─ 发送 DATASET 信号

8. 完成
   └─ 设置节点状态为 COMPLETED
```

### user_tweets 模式流程

```
1. 遍历 accounts 列表
   ↓
2. 对每个账户：
   ├─ 构建请求（userId 或 userName）
   ├─ 调用 /user/last_tweets API
   ├─ 获取推文（支持分页）
   └─ 应用关键词过滤
   ↓
3. 合并所有账户的推文
   ↓
4. 按时间排序
   ↓
5. 限制总数（limit 参数）
```

### advanced_search 模式流程

```
1. 构建搜索查询
   ├─ 处理关键词（OR 逻辑）
   ├─ 添加 from: 操作符（如果指定账户）
   └─ 设置查询类型（Latest/Top）
   ↓
2. 调用 /tweet/advanced_search API
   ↓
3. 处理分页（max_pages=5）
   ↓
4. 收集所有结果
   ↓
5. 应用数量限制
```

---

## 使用示例

### 示例 1：监听KOL推文

**场景：** 监听 Elon Musk 和 Vitalik Buterin 关于加密货币的推文。

**工作流结构：**
```
X Listener Node (监听KOL)
    ↓ latest_tweets
AI Model Node (分析情绪)
    ↓ ai_response { sentiment, key_points }
Code Node (生成交易信号)
    ↓ trade_signal
Condition Node (是否交易?)
    ↓
Buy/Sell Node (执行交易)
```

**X Listener Node 配置：**
```json
{
  "accounts": "elonmusk, VitalikButerin",
  "keywords": "Bitcoin, crypto, blockchain",
  "search_mode": "user_tweets",
  "limit": 20
}
```

**输出示例：**
```json
{
  "tweets": [
    {
      "text": "Bitcoin adoption is accelerating faster than expected",
      "author": { "username": "elonmusk" },
      "metrics": { "like_count": 200000 }
    }
  ],
  "count": 18
}
```

---

### 示例 2：话题追踪

**场景：** 追踪 #DeFi 相关的热门讨论。

**X Listener Node 配置：**
```json
{
  "keywords": "DeFi, decentralized, yield farming",
  "search_mode": "advanced_search",
  "query_type": "Top",
  "limit": 50
}
```

**工作流结构：**
```
X Listener Node (话题搜索)
    ↓ latest_tweets (Top 50 DeFi相关推文)
Code Node (提取提及的项目)
    ↓ mentioned_projects
AI Model Node (评估项目热度)
    ↓ hot_projects
Dataset Output Node (保存分析结果)
```

---

### 示例 3：情绪分析系统

**场景：** 实时监控市场情绪并生成交易信号。

**工作流结构：**
```
X Listener Node (监听多个KOL)
    ↓ latest_tweets
    ↓
AI Model Node (情绪分析)
    ↓ sentiment_score (-1 到 +1)
    ↓
Code Node (计算加权情绪)
    ├─ 根据KOL影响力加权
    └─ 计算综合情绪指数
    ↓ weighted_sentiment
    ↓
Condition Node (情绪 > 0.7?)
    ↓ (true - 极度看涨)
    ├─→ Buy Node (买入)
    ↓ (false)
Condition Node (情绪 < -0.7?)
    ↓ (true - 极度看跌)
    └─→ Sell Node (卖出)
```

**X Listener Node 配置：**
```json
{
  "accounts": "elonmusk, VitalikButerin, cz_binance, SBF_FTX",
  "keywords": "Bitcoin, BTC",
  "search_mode": "user_tweets",
  "limit": 100
}
```

---

### 示例 4：组合过滤

**场景：** 监听特定账户，只保留包含特定关键词的推文。

**X Listener Node 配置：**
```json
{
  "accounts": "elonmusk",
  "keywords": "Dogecoin, DOGE",
  "search_mode": "user_tweets",
  "limit": 20
}
```

**执行逻辑：**
```
1. 获取 @elonmusk 的最近推文
2. 过滤只包含 "Dogecoin" 或 "DOGE" 的推文
3. 返回匹配的推文
```

---

## API 依赖

### TwitterAPI.io

**基础URL：** `https://api.twitterapi.io`

#### 用户推文端点

**端点：** `GET /twitter/user/last_tweets`

**参数：**
- `userId` 或 `userName`：用户标识
- `cursor`：分页游标（可选）

**响应：**
```json
{
  "status": "success",
  "data": {
    "tweets": [...],
    "user_info": {...}
  },
  "has_next_page": true,
  "next_cursor": "..."
}
```

#### 高级搜索端点

**端点：** `GET /twitter/tweet/advanced_search`

**参数：**
- `query`：搜索查询字符串
- `queryType`：`Latest` 或 `Top`
- `cursor`：分页游标（可选）

**查询语法：**
```
关键词: "Bitcoin OR crypto"
用户过滤: "Bitcoin from:elonmusk"
组合: "(Bitcoin OR crypto) from:elonmusk"
```

**响应：**
```json
{
  "tweets": [...],
  "has_next_page": true,
  "next_cursor": "..."
}
```

---

## 最佳实践

### 1. API Key 管理

**推荐方式（环境变量）：**
```bash
export TWITTER_API_KEY="your_api_key_here"
```

**不推荐（硬编码）：**
```json
{
  "api_key": "your_api_key_here"  // ❌ 不安全
}
```

### 2. 搜索模式选择

| 场景 | 推荐模式 | 原因 |
|------|---------|------|
| 监听特定KOL | `user_tweets` | 精确、稳定 |
| 追踪热门话题 | `advanced_search` + `Top` | 发现热门内容 |
| 实时监控关键词 | `advanced_search` + `Latest` | 最新信息 |
| 多账户监听 | `user_tweets` | 支持并行获取 |

### 3. 关键词设计

**✅ 推荐：**
```
"Bitcoin, BTC"  // 主词 + 缩写
"DeFi, decentralized finance"  // 主词 + 完整名称
"pump, moon, bullish"  // 情绪关键词
```

**❌ 避免：**
```
"a, the, is"  // 过于通用
"Bitcoin Bitcoin Bitcoin"  // 重复无意义
```

### 4. 数量限制设置

| 用途 | 建议 limit | 原因 |
|------|-----------|------|
| 实时监控 | 10-20 | 快速响应 |
| 情绪分析 | 50-100 | 样本充足 |
| 数据收集 | 100 | 最大允许 |

---

## 注意事项

### ⚠️ 重要提示

1. **API 配额限制**
   - TwitterAPI.io 有调用次数限制
   - 注意查看您的套餐配额
   - 使用 limit 参数控制数据量

2. **速率限制**
   - 节点内置 0.5秒 延迟
   - 防止API调用过快
   - 多账户监听自动添加延迟

3. **搜索模式限制**
   - `user_tweets` 需要 `accounts` 参数
   - `advanced_search` 需要 `keywords` 或 `accounts`
   - 高级搜索不支持纯数字 userId

4. **关键词过滤**
   - 不区分大小写
   - 部分匹配（包含即可）
   - 仅过滤推文文本，不包括用户名等

5. **数据时效性**
   - 推文数据为节点执行时的快照
   - 不是实时流式数据
   - 需要重新执行节点获取最新数据

---

## 故障排查

**Q: 提示 "Twitter API key not provided"？**

A:
1. 检查环境变量 `TWITTER_API_KEY` 是否设置
2. 或在节点配置中提供 `api_key` 参数
3. 确认 API Key 有效且未过期

---

**Q: 返回的推文数量少于 limit？**

A:
1. 用户可能没有发布足够多的推文
2. 关键词过滤可能过滤掉了部分推文
3. API 返回的数据有限
4. 这是正常现象，不是错误

---

**Q: "Account parameter is required" 错误？**

A:
1. 确认在 `user_tweets` 模式下提供了 `accounts` 参数
2. 检查 `accounts` 格式是否正确（逗号分隔）
3. 确保至少有一个有效账户

---

**Q: 高级搜索返回结果较少？**

A:
1. 尝试放宽关键词（减少关键词数量）
2. 检查关键词拼写
3. 使用 `Top` 类型可能返回更少但更相关的结果
4. 去除 `from:` 限制扩大搜索范围

---

**Q: 如何实现实时监控？**

A:
```
方案1 - 定时执行：
创建定时触发的工作流，每隔N分钟执行一次

方案2 - 循环执行：
在工作流中添加循环逻辑，持续监控

方案3 - Webhook触发：
配合外部服务实现事件驱动
```

---

## 技术规格

| 规格项 | 值 |
|--------|-----|
| **节点版本** | 1.0.0 |
| **支持的模式** | user_tweets, advanced_search |
| **最大 limit** | 100 |
| **默认 limit** | 20 |
| **分页支持** | ✅ 是 |
| **速率限制** | 0.5秒/账户 |
| **执行超时** | 120秒 |

---

## 相关节点

- **AI Model Node** - 分析推文情绪和内容
- **Code Node** - 处理推文数据
- **Dataset Output Node** - 保存推文数据
- **Condition Node** - 基于推文内容做决策
- **Telegram Sender Node** - 发送推文通知

---

**相关文档：**
- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [AI Model Node](ai-model-node.md) - AI 分析节点
- [Weather 语法](../core-concepts/weather-syntax.md) - 工作流文件格式

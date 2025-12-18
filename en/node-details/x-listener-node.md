# X Listener Node

X Listener Node (Twitter Listener) is a TradingFlow data input node for fetching tweets from X (Twitter) platform. It supports keyword filtering, multi-account monitoring, and real-time data collection, making it a core component for social media sentiment analysis.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `x_listener_node` |
| **Display Name** | X Listener |
| **Node Category** | Input (Social Media) |
| **Icon** | 🐦 Twitter Icon |
| **Handle Color** | Sky (blue) / Emerald (green) |

---

## Functionality

X Listener Node connects to the X (Twitter) platform to fetch tweets from specified accounts or perform advanced searches in real-time. It supports keyword filtering, parallel multi-account monitoring, and passes tweet data to downstream analysis nodes.

**Primary Use Cases:**
- Monitor KOL (Key Opinion Leader) tweets
- Track specific topics and keywords
- Collect market sentiment data
- Capture social media signals
- Monitor public opinion

**Key Features:**
- 🔍 **Dual Search Modes**: User timeline and advanced search
- 🏷️ **Keyword Filtering**: Multi-keyword matching and filtering
- 👥 **Multi-Account Support**: Monitor multiple accounts simultaneously
- 📄 **Pagination**: Automatic handling of large tweet volumes
- ⚡ **Rate Limiting**: Built-in API call rate control

---

## Input Parameters

### Primary Parameters (Always Visible)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `accounts` | paragraph | Yes | - | X account list (comma-separated) |
| `keywords` | paragraph | No | `""` | Keyword filter (comma-separated) |

### Advanced Parameters (Hidden by Default)

The following parameters are hidden by default and can be expanded when needed:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `search_mode` | select | `latest` | Search mode: `latest` or `top` |
| `query_type` | select | `user_timeline` | Query type: `user_timeline` or `search` |
| `limit` | number | `20` | Tweet count limit (1-100) |

**About Advanced Parameters:**
- Click "Show advanced params" button at the bottom of the node to expand
- Select needed parameters in the popup, or click "Show All" to display all
- Displayed parameters can be hidden by clicking the "Hide" button (which also clears the value)
- When the Agent specifies a value for an advanced parameter, it automatically becomes visible

### accounts Parameter

**Format:** Comma-separated account list

**Supported Formats:**
- **Usernames:** `elonmusk, VitalikButerin`
- **User IDs:** `44196397, 295218901` (numeric only)
- **Mixed:** `elonmusk, 295218901`

**Example:**
```
elonmusk, VitalikButerin, cz_binance
```

**Notes:**
- No `@` symbol needed for usernames
- Supports both userId and userName formats
- Multiple accounts separated by commas
- Whitespace is automatically trimmed

### keywords Parameter

**Format:** Comma-separated keyword list

**Matching Rules:**
- Case-insensitive
- Partial matching (contains)
- OR logic (any keyword matches)

**Example:**
```
Bitcoin, BTC, crypto
```

### search_mode Parameter

| Mode | Value | Description |
|------|-------|-------------|
| **Latest** | `latest` | Most recent tweets first |
| **Top** | `top` | Popular/trending tweets first |

### query_type Parameter

| Type | Value | Description |
|------|-------|-------------|
| **User Timeline** | `user_timeline` | Fetch tweets from user's timeline |
| **Search** | `search` | Global keyword search |

### limit Parameter

**Range:** 1-100

**Notes:**
- Limits total returned tweets
- Prevents data overload
- Total limit across all accounts

---

## Output Parameters

### Output List

| Output ID | Display Name | Data Type | Description |
|-----------|--------------|-----------|-------------|
| `latest_tweets` | Latest Tweets | object | Tweet data collection |

### latest_tweets Output

**Data Structure:**

```typescript
{
  tweets: Array<{              // Tweet list
    id: string;                // Tweet ID
    text: string;              // Tweet text
    created_at: string;        // Creation time
    author: {                  // Author info
      id: string;
      username: string;
      name: string;
    };
    metrics: {                 // Engagement data
      like_count: number;
      retweet_count: number;
      reply_count: number;
    };
    source_account?: string;   // Source account (multi-account)
  }>;
  count: number;               // Tweet count
  accounts: string[];          // Monitored accounts
  keywords: string;            // Keywords used
  search_mode: string;         // Search mode
  query_type: string;          // Query type
  timestamp: number;           // Fetch timestamp
  next_cursor: string;         // Pagination cursor
}
```

---

## Usage Examples

### Example 1: Monitor KOL Tweets

**Scenario:** Monitor Elon Musk and Vitalik Buterin's crypto-related tweets.

**Workflow Structure:**
```
X Listener Node (monitor KOLs)
    ↓ latest_tweets
AI Model Node (analyze sentiment)
    ↓ ai_response { sentiment, key_points }
Code Node (generate trading signals)
    ↓ trade_signal
Condition Node (should trade?)
    ↓
Buy/Sell Node (execute trade)
```

**Node Configuration:**
```json
{
  "accounts": "elonmusk, VitalikButerin",
  "keywords": "Bitcoin, crypto, blockchain"
}
```

---

### Example 2: Topic Tracking

**Scenario:** Track trending #DeFi discussions.

**Node Configuration:**
```json
{
  "keywords": "DeFi, decentralized, yield farming",
  "query_type": "search",
  "search_mode": "top",
  "limit": 50
}
```

---

### Example 3: Sentiment Analysis System

**Scenario:** Real-time market sentiment monitoring and trading signal generation.

**Workflow Structure:**
```
X Listener Node (monitor multiple KOLs)
    ↓ latest_tweets
    ↓
AI Model Node (sentiment analysis)
    ↓ sentiment_score (-1 to +1)
    ↓
Code Node (calculate weighted sentiment)
    ├─ Weight by KOL influence
    └─ Calculate composite sentiment index
    ↓ weighted_sentiment
    ↓
Condition Node (sentiment > 0.7?)
    ↓ (true - extremely bullish)
    ├─→ Buy Node (buy)
    ↓ (false)
Condition Node (sentiment < -0.7?)
    ↓ (true - extremely bearish)
    └─→ Sell Node (sell)
```

---

## Best Practices

### 1. Search Mode Selection

| Scenario | Recommended Mode | Reason |
|----------|-----------------|--------|
| Monitor specific KOL | `user_timeline` | Precise, stable |
| Track trending topics | `search` + `top` | Discover popular content |
| Real-time keyword monitoring | `search` + `latest` | Latest information |
| Multi-account monitoring | `user_timeline` | Supports parallel fetching |

### 2. Keyword Design

**Recommended:**
```
"Bitcoin, BTC"  // Main term + abbreviation
"DeFi, decentralized finance"  // Main term + full name
"pump, moon, bullish"  // Sentiment keywords
```

**Avoid:**
```
"a, the, is"  // Too generic
"Bitcoin Bitcoin Bitcoin"  // Meaningless repetition
```

### 3. Limit Settings

| Use Case | Suggested Limit | Reason |
|----------|-----------------|--------|
| Real-time monitoring | 10-20 | Fast response |
| Sentiment analysis | 50-100 | Sufficient sample |
| Data collection | 100 | Maximum allowed |

---

## Important Notes

1. **API Quota Limits**
   - TwitterAPI.io has call limits
   - Check your plan quota
   - Use limit parameter to control data volume

2. **Rate Limiting**
   - Built-in 0.5s delay
   - Prevents excessive API calls
   - Auto-delays for multi-account monitoring

3. **Search Mode Restrictions**
   - `user_timeline` requires `accounts` parameter
   - `search` requires `keywords` or `accounts`
   - Advanced search doesn't support pure numeric userId

4. **Keyword Filtering**
   - Case-insensitive
   - Partial matching (contains)
   - Only filters tweet text, not usernames

5. **Data Freshness**
   - Tweet data is a snapshot at execution time
   - Not real-time streaming data
   - Re-execute node for latest data

---

## Technical Specifications

| Specification | Value |
|---------------|-------|
| **Node Version** | 1.0.0 |
| **Supported Modes** | user_timeline, search |
| **Max Limit** | 100 |
| **Default Limit** | 20 |
| **Pagination Support** | Yes |
| **Rate Limit** | 0.5s/account |
| **Timeout** | 120 seconds |

---

## Related Nodes

- **AI Model Node** - Analyze tweet sentiment and content
- **Code Node** - Process tweet data
- **Google Sheet Output Node** - Save tweet data
- **Condition Node** - Make decisions based on tweet content
- **Telegram Sender Node** - Send tweet notifications

---

**Related Documentation:**
- [Nodes and Workflows](../core-concepts/nodes-and-workflows.md) - Node fundamentals
- [AI Model Node](ai-model-node.md) - AI analysis node
- [Weather Syntax](../core-concepts/weather-syntax.md) - Workflow file format

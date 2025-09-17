# Node Reference Guide

Complete reference for all TradingFlow nodes. Each node serves a specific purpose in your trading workflows, from data collection to trade execution.

## Node Categories

### Data Collection Nodes
- **Price Feed Nodes**: Real-time market data
- **Social Media Nodes**: Sentiment and trend analysis
- **Dataset Nodes**: External data integration

### Decision Making Nodes
- **AI Model Nodes**: Machine learning predictions
- **Code Execution Nodes**: Custom logic implementation
- **Technical Analysis Nodes**: Indicator calculations

### Action Nodes
- **Trading Nodes**: Execute buy/sell orders
- **Vault Management Nodes**: Asset management
- **Notification Nodes**: Communication and alerts

### Utility Nodes
- **Timer Nodes**: Scheduled execution
- **Math Nodes**: Calculations and transformations
- **Flow Control Nodes**: Conditional logic

## Data Collection Nodes

### Binance Price Node

**Purpose**: Fetch real-time price data from Binance exchange
**Node Type**: `binance_price_node`
**Category**: Data Collection

#### Configuration
```javascript
{
  symbol: "BTCUSDT",        // Trading pair symbol
  interval: "1h",           // Time interval (1m, 5m, 1h, 1d)
  limit: 100,              // Number of data points
  include_volume: true      // Include volume data
}
```

#### Input Handles
- `symbol` (string): Trading pair to monitor
- `interval` (string): Time interval for data points
- `limit` (number): Number of historical points to fetch

#### Output Handles
- `price_data` (object): Complete price and volume data
  ```javascript
  {
    symbol: "BTCUSDT",
    current_price: 43250.50,
    price_change_24h: 2.34,
    volume_24h: 1250000,
    high_24h: 44000,
    low_24h: 42800,
    timestamp: "2024-01-15T10:30:00Z",
    kline_data: [
      {
        open: 43000,
        high: 43500,
        low: 42900,
        close: 43250,
        volume: 125.67,
        timestamp: "2024-01-15T10:00:00Z"
      }
    ]
  }
  ```

#### Use Cases
- Market monitoring and alerting
- Technical analysis data source
- Portfolio valuation updates
- Arbitrage opportunity detection

### X (Twitter) Listener Node

**Purpose**: Monitor Twitter for mentions, trends, and sentiment
**Node Type**: `x_listener_node`
**Category**: Data Collection

#### Configuration
```javascript
{
  accounts: ["elonmusk", "cz_binance"],  // Twitter accounts to monitor
  keywords: "Bitcoin, BTC, crypto",      // Keywords to filter
  search_mode: "user_tweets",            // "user_tweets" or "advanced_search"
  query_type: "Latest",                  // "Latest" or "Top"
  limit: 20,                            // Number of tweets to fetch
  api_key: "your_twitter_api_key"       // Twitter API credentials
}
```

#### Input Handles
- `account` (string): Twitter username or user ID
- `keywords` (string): Comma-separated keywords for filtering

#### Output Handles
- `latest_tweets` (array): Array of tweet objects
  ```javascript
  [
    {
      id: "1234567890",
      text: "Bitcoin is looking bullish today! 🚀",
      author: "crypto_trader",
      created_at: "2024-01-15T10:30:00Z",
      public_metrics: {
        retweet_count: 45,
        like_count: 128,
        reply_count: 23
      },
      sentiment_score: 0.7,  // -1 (negative) to +1 (positive)
      contains_keywords: ["Bitcoin", "bullish"]
    }
  ]
  ```

#### Advanced Features
- **Keyword Filtering**: Multi-keyword filtering with comma separation
- **Sentiment Analysis**: Built-in sentiment scoring
- **Search Modes**: User tweets or advanced search
- **Rate Limiting**: Automatic API rate limit handling

### Dataset Node

**Purpose**: Read/write data from external sources like Google Sheets
**Node Type**: `dataset_node`
**Category**: Data Collection

#### Configuration
```javascript
{
  mode: "read",                    // "read", "write", or "append"
  doc_link: "google_sheets_url",   // Data source URL
  auto_format: true,               // Auto-detect data format
  cache_duration: 300              // Cache data for 5 minutes
}
```

#### Input Handles
- `doc_link` (string): Google Sheets URL or document ID
- `data` (object): Data to write (write/append modes only)

#### Output Handles
- `data` (object): Structured data from source
  ```javascript
  {
    headers: ["Date", "Symbol", "Price", "Volume"],
    data: [
      ["2024-01-15", "BTC", "43250", "1500"],
      ["2024-01-15", "ETH", "2650", "2300"]
    ],
    metadata: {
      rows: 2,
      columns: 4,
      last_updated: "2024-01-15T10:30:00Z"
    }
  }
  ```

#### Data Format Support
- Google Sheets integration
- CSV file processing
- JSON data transformation
- Automatic type detection

## Decision Making Nodes

### AI Model Node

**Purpose**: Use large language models for intelligent decision making
**Node Type**: `ai_model_node`
**Category**: Decision Making

#### Configuration
```javascript
{
  model_name: "gpt-4",              // AI model to use
  temperature: 0.7,                 // Creativity level (0-1)
  system_prompt: "You are a crypto trading assistant",
  prompt: "Analyze the market data and provide trading signals",
  max_tokens: 1000,                 // Maximum response length
  auto_format_output: true          // Auto-generate JSON format
}
```

#### Input Handles
- `model` (string): AI model selection
- `prompt` (string): Main instruction prompt
- `parameters` (object): Model parameters (temperature, max_tokens)

#### Output Handles
- `ai_response` (object): Structured AI response
  ```javascript
  {
    decision: "BUY",
    confidence: 0.85,
    reasoning: "Strong bullish signals from technical analysis and positive sentiment",
    suggested_amount: 1000,
    risk_level: "medium",
    timeframe: "short_term",
    metadata: {
      model_used: "gpt-4",
      processing_time: 1.23,
      token_count: 150
    }
  }
  ```

#### Advanced Features
- **Dynamic Output Formatting**: Automatically adjusts output based on connected nodes
- **Multi-Model Support**: GPT-4, GPT-3.5, Claude, and others
- **Context Management**: Maintains conversation context
- **Error Handling**: Graceful fallbacks for API issues

### Code Execution Node

**Purpose**: Run custom Python code for complex logic
**Node Type**: `code_node`
**Category**: Decision Making

#### Configuration
```javascript
{
  python_code: `
# Custom trading logic
def analyze_data(price_data, sentiment):
    rsi = calculate_rsi(price_data.prices)
    
    if rsi < 30 and sentiment > 0.3:
        return {"signal": "BUY", "strength": 0.8}
    elif rsi > 70 and sentiment < -0.3:
        return {"signal": "SELL", "strength": 0.8}
    else:
        return {"signal": "HOLD", "strength": 0.3}

result = analyze_data(inputs.price_data, inputs.sentiment_score)
`,
  timeout: 30,                      // Execution timeout in seconds
  max_memory: 128,                  // Memory limit in MB
  allowed_imports: ["numpy", "pandas", "math"]  // Allowed Python modules
}
```

#### Input Handles
- `python_code` (string): Python code to execute
- Dynamic inputs based on code requirements

#### Output Handles
- `output_data` (any): Result of code execution
- `execution_logs` (string): Execution logs and debug info

#### Security Features
- **Sandboxed Execution**: Isolated Python environment
- **Module Restrictions**: Only allowed modules can be imported
- **Resource Limits**: CPU time and memory constraints
- **Timeout Protection**: Prevents infinite loops

#### Supported Libraries
- **Data Analysis**: pandas, numpy
- **Mathematics**: math, statistics
- **Technical Analysis**: talib (limited functions)
- **Date/Time**: datetime, time
- **Web Requests**: requests (with rate limiting)

## Action Nodes

### Swap Node

**Purpose**: Execute token swaps on supported DEXs
**Node Type**: `swap_node`
**Category**: Action

#### Configuration
```javascript
{
  from_token: "USDC",               // Source token symbol
  to_token: "APT",                  // Destination token symbol
  chain: "aptos",                   // Blockchain network
  vault_address: "0x123...",        // Vault contract address
  amount_in_percentage: 25.0,       // Percentage of balance to swap
  amount_in_human_readable: 1000.0, // Fixed amount to swap
  slippery_tolerance: 1.0           // Slippage tolerance (%)
}
```

#### Input Handles
- `from_token` (string): Source token symbol
- `to_token` (string): Destination token symbol
- `chain` (string): Target blockchain ("aptos", "flow_evm")
- `vault_address` (string): Vault contract address
- `amount_in_percentage` (number): Percentage of balance to use
- `amount_in_human_readable` (number): Fixed amount in tokens
- `slippery_tolerance` (number): Maximum slippage tolerance

#### Output Handles
- `trade_receipt` (object): Transaction receipt and details
  ```javascript
  {
    transaction_hash: "0xabc123...",
    from_token: "USDC",
    to_token: "APT",
    amount_in: "1000.00",
    amount_out: "131.25",
    exchange_rate: 0.13125,
    slippage_actual: 0.8,
    gas_fee: "0.001",
    status: "success",
    timestamp: "2024-01-15T10:30:00Z",
    dex_used: "Hyperion"
  }
  ```

#### Supported Networks
- **Aptos**: Hyperion DEX integration
- **Flow EVM**: PunchSwap V2 integration
- **BSC**: PancakeSwap integration (coming soon)

#### Advanced Features
- **Multi-Chain Support**: Automatic chain detection and routing
- **Slippage Protection**: Configurable slippage limits
- **Gas Optimization**: Smart gas fee calculation
- **MEV Protection**: Front-running protection mechanisms

### Vault Node

**Purpose**: Manage vault operations and portfolio tracking
**Node Type**: `vault_node`
**Category**: Action

#### Configuration
```javascript
{
  chain: "aptos",                   // Blockchain network
  vault_address: "0x123...",        // Vault contract address
  operation: "get_balance",         // Operation type
  auto_refresh: true                // Automatically refresh data
}
```

#### Input Handles
- `chain` (string): Blockchain network
- `vault_address` (string): Vault contract address
- `operation` (string): Operation to perform

#### Output Handles
- `vault_info` (object): Complete vault information
  ```javascript
  {
    address: "0x123...",
    chain: "aptos",
    total_value_usd: 15420.50,
    assets: [
      {
        symbol: "APT",
        balance: "125.67",
        value_usd: 1200.30,
        percentage: 7.78
      },
      {
        symbol: "USDC",
        balance: "14220.20",
        value_usd: 14220.20,
        percentage: 92.22
      }
    ],
    last_updated: "2024-01-15T10:30:00Z",
    performance_24h: 2.34
  }
  ```

#### Available Operations
- `get_balance`: Retrieve current portfolio balance
- `get_history`: Fetch transaction history
- `get_performance`: Portfolio performance metrics
- `health_check`: Vault status verification

### Telegram Sender Node

**Purpose**: Send notifications and alerts via Telegram
**Node Type**: `telegram_sender_node`
**Category**: Action

#### Configuration
```javascript
{
  bot_token: "telegram_bot_token",  // Telegram bot token
  chat_id: "-1001234567890",        // Target chat/channel ID
  message_template: "Trading Alert: {signal} signal detected for {symbol}",
  parse_mode: "HTML",               // Message formatting
  disable_notification: false       // Silent notifications
}
```

#### Input Handles
- `message` (string): Message content to send
- `chat_id` (string): Target chat or channel ID
- `data` (object): Data for message template

#### Output Handles
- `send_result` (object): Message sending result
  ```javascript
  {
    success: true,
    message_id: 12345,
    timestamp: "2024-01-15T10:30:00Z",
    chat_id: "-1001234567890",
    error: null
  }
  ```

#### Message Templates
- **Trading Signals**: Alert for buy/sell signals
- **Portfolio Updates**: Balance and performance reports
- **Error Notifications**: System alerts and warnings
- **Custom Messages**: Flexible template system

## Utility Nodes

### Timer Node

**Purpose**: Schedule periodic execution of workflows
**Node Type**: `timer_node`
**Category**: Utility

#### Configuration
```javascript
{
  schedule: "0 9 * * 1",            // Cron expression (every Monday 9 AM)
  timezone: "UTC",                  // Timezone for scheduling
  max_executions: 0,                // 0 = unlimited
  start_delay: 0                    // Initial delay in seconds
}
```

#### Common Schedule Patterns
- **Every Minute**: `* * * * *`
- **Hourly**: `0 * * * *`
- **Daily at 9 AM**: `0 9 * * *`
- **Weekly on Monday**: `0 9 * * 1`
- **Monthly on 1st**: `0 9 1 * *`

### Math Operation Node

**Purpose**: Perform mathematical calculations and transformations
**Node Type**: `math_node`
**Category**: Utility

#### Supported Operations
- **Basic Math**: addition, subtraction, multiplication, division
- **Statistical**: mean, median, standard deviation
- **Financial**: percentage change, returns calculation
- **Comparison**: greater than, less than, equal to

### Condition Logic Node

**Purpose**: Implement if/then/else logic for flow control
**Node Type**: `condition_node`
**Category**: Utility

#### Logic Types
- **Simple Comparison**: value > threshold
- **Multiple Conditions**: AND/OR combinations
- **Range Checks**: value between min and max
- **Pattern Matching**: regex and string matching

## Node Development Guidelines

### Custom Node Creation

When building custom nodes, follow these patterns:

#### 1. Node Registration
```python
from tradingflow.station.common.node_decorators import register_node_type

@register_node_type(
    "my_custom_node",
    default_params={
        "parameter1": "default_value",
        "parameter2": 42
    }
)
class MyCustomNode(NodeBase):
    # Implementation
```

#### 2. Input/Output Handles
```python
# Define handle constants
INPUT_HANDLE = "data_input"
OUTPUT_HANDLE = "processed_data"

class MyCustomNode(NodeBase):
    def _register_input_handles(self):
        return [
            InputHandle(
                name=INPUT_HANDLE,
                data_type=dict,
                description="Input data to process"
            )
        ]
```

#### 3. Execution Logic
```python
async def execute(self):
    try:
        # Get input data
        input_data = await self.get_input_data(INPUT_HANDLE)
        
        # Process data
        result = self.process_data(input_data)
        
        # Send output
        await self.send_signal(
            OUTPUT_HANDLE,
            SignalType.PROCESSED_DATA,
            result
        )
        
        return NodeStatus.COMPLETED
        
    except Exception as e:
        self.persist_log(f"Error: {str(e)}", level="ERROR")
        return NodeStatus.FAILED
```

### Best Practices

#### Error Handling
- Always wrap execution in try/catch blocks
- Use meaningful error messages
- Log errors with appropriate severity levels
- Return appropriate NodeStatus values

#### Performance
- Use async/await for I/O operations
- Implement caching for expensive operations
- Set reasonable timeouts for external calls
- Monitor memory usage in data processing

#### Security
- Validate all input parameters
- Sanitize user-provided data
- Use secure API key management
- Implement rate limiting for external APIs

#### Testing
- Write unit tests for core logic
- Test error conditions and edge cases
- Validate input/output formats
- Performance test with realistic data volumes

---

Ready to build advanced trading strategies? Check out the [Technical Documentation](../technical-documentation/) for system architecture and development guides.

## Next Steps
- [Technical Documentation →](../technical-documentation/)
- [API Reference →](../technical-documentation/api-reference.md)
- [Advanced Trading Patterns →](advanced-trading-patterns.md)

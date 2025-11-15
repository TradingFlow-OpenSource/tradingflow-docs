# Binance Price Node

Binance Price Node is a data input node used to fetch real-time market data from Binance exchange, including K-line (candlestick) data and current price.

---

## Node Information

| Property          | Value                     |
| ----------------- | ------------------------- |
| **Node Type**     | `binance_price_node`      |
| **Display Name**  | Binance Price             |
| **Node Category** | Input (Data Input)        |
| **Icon**          | 🕯️ Candlestick Chart Icon |
| **Handle Color**  | Amber (Orange)            |

---

## Functionality

Binance Price Node fetches historical K-line data and current price for specified trading pairs through Binance API. The node supports multiple time intervals (from 1 minute to 1 year) and allows specification of the number of K-lines to retrieve.

**Main Uses:**

- Fetch cryptocurrency price data for technical analysis
- Monitor price changes for specific trading pairs
- Provide real-time market data for trading strategies
- Generate price charts and trend analysis

---

## Input Parameters

### Parameter List

| Parameter  | Type         | Required | Default   | Range/Options          | Description                  |
| ---------- | ------------ | -------- | --------- | ---------------------- | ---------------------------- |
| `symbol`   | searchSelect | ✅       | `BTCUSDT` | See trading pairs list | Trading pair symbol to query |
| `interval` | searchSelect | ✅       | `1m`      | See interval list      | K-line time interval         |
| `limit`    | number       | ❌       | `100`     | 1-1000                 | Number of K-lines to fetch   |

### Supported Trading Pairs

| Pair       | Display Name | Description         |
| ---------- | ------------ | ------------------- |
| `BTCUSDT`  | BTC/USDT     | Bitcoin to USDT     |
| `BTCUSDC`  | BTC/USDC     | Bitcoin to USDC     |
| `ETHUSDT`  | ETH/USDT     | Ethereum to USDT    |
| `ETHUSDC`  | ETH/USDC     | Ethereum to USDC    |
| `ETHBTC`   | ETH/BTC      | Ethereum to Bitcoin |
| `APTUSDT`  | APT/USDT     | Aptos to USDT       |
| `APTUSDC`  | APT/USDC     | Aptos to USDC       |
| `APTxBTC`  | APT/xBTC     | Aptos to xBTC       |
| `USDCUSDT` | USDC/USDT    | USDC to USDT        |

_Note: Available trading pairs depend on Binance exchange support._

### Time Interval Options

| Value | Display Name | Description     |
| ----- | ------------ | --------------- |
| `1m`  | 1 minute     | 1-minute K-line |
| `1h`  | 1 hour       | 1-hour K-line   |
| `1d`  | 1 day        | 1-day K-line    |
| `1w`  | 1 week       | 1-week K-line   |
| `1M`  | 1 month      | 1-month K-line  |
| `1y`  | 1 year       | 1-year K-line   |

_Binance API also supports other intervals like 3m, 5m, 15m, 30m, 2h, 4h, 6h, 8h, 12h, 3d, etc., but not all are listed in the frontend interface._

---

## Output Parameters

### Output List

| Output ID | Display Name | Data Type | Description                                                         |
| --------- | ------------ | --------- | ------------------------------------------------------------------- |
| `data`    | Data         | object    | Complete market data object including current price and K-line data |

### data Output

**Data Type:** `object`

**Complete Data Structure:**

```typescript
{
  symbol: string;           // Trading pair symbol
  interval: string;         // K-line time interval
  limit: number;           // Number of K-lines
  header: string[];        // K-line data column names
  kline_data: Array[];     // K-line data array
  current_price: string;   // Current price
}
```

**Actual Example:**

```json
{
  "symbol": "BTCUSDT",
  "interval": "1h",
  "limit": 3,
  "header": [
    "open_time",
    "open",
    "high",
    "low",
    "close",
    "volume",
    "close_time",
    "quote_volume",
    "count",
    "taker_buy_volume",
    "taker_buy_quote_volume"
  ],
  "kline_data": [
    [
      1672531200000,       // open_time: Opening time (millisecond timestamp)
      "16590.00000000",    // open: Opening price
      "16650.00000000",    // high: Highest price
      "16580.00000000",    // low: Lowest price
      "16620.00000000",    // close: Closing price
      "1234.56789000",     // volume: Trading volume (base currency)
      1672534799999,       // close_time: Closing time (millisecond timestamp)
      "20498765.43210000", // quote_volume: Trading amount (quote currency)
      15234,               // count: Number of trades
      "617.28394500",      // taker_buy_volume: Taker buy volume
      "10249382.71605000", // taker_buy_quote_volume: Taker buy quote volume
      "0"                  // Ignored field
    ],
    [...],  // More K-line data
    [...]
  ],
  "current_price": "16620.00000000"
}
```

### K-line Data Field Descriptions

| Index | Field Name               | Data Type | Description                                 |
| ----- | ------------------------ | --------- | ------------------------------------------- |
| 0     | `open_time`              | number    | Opening time (Unix millisecond timestamp)   |
| 1     | `open`                   | string    | Opening price                               |
| 2     | `high`                   | string    | Highest price                               |
| 3     | `low`                    | string    | Lowest price                                |
| 4     | `close`                  | string    | Closing price                               |
| 5     | `volume`                 | string    | Trading volume (base currency, e.g., BTC)   |
| 6     | `close_time`             | number    | Closing time (Unix millisecond timestamp)   |
| 7     | `quote_volume`           | string    | Trading amount (quote currency, e.g., USDT) |
| 8     | `count`                  | number    | Number of trades                            |
| 9     | `taker_buy_volume`       | string    | Taker buy volume                            |
| 10    | `taker_buy_quote_volume` | string    | Taker buy quote volume                      |

---

## Signal Transmission

### Sent Signals

**Signal Handle:** `data`

**Signal Type:** `SignalType.PRICE_DATA`

**Signal Payload:** Complete market data object including trading pair, K-line data, and current price (see data output structure above)

### Signal Flow Example

```
Binance Price Node
    ↓ (data signal)
    ↓ payload: { symbol, interval, limit, header, kline_data, current_price }
    ↓
Other Nodes (e.g., Code Node, AI Model Node, etc.)
```

---

## Workflow

### Node Execution Flow

```
1. Initialization
   ├─ Create Binance API client
   ├─ Support both with/without API Key modes
   └─ Connect to Binance server and validate

2. Fetch Data
   ├─ Call get_klines API to get historical K-line data
   └─ Call get_symbol_ticker API to get current price

3. Process Data
   ├─ Integrate K-line data and current price
   ├─ Add header information
   └─ Build complete output object

4. Send Signal
   ├─ Send PRICE_DATA signal through data handle
   └─ Pass complete market data object (including K-line data and current price) to downstream nodes

5. Complete
   └─ Set node status to COMPLETED
```

### Error Handling

The node has comprehensive error handling mechanisms:

| Error Type            | Handling Method                             | Retry Count |
| --------------------- | ------------------------------------------- | ----------- |
| API Connection Failed | Exponential backoff retry (1s, 2s, 4s)      | 3 times     |
| API Call Exception    | Log error, set node status to FAILED        | 0 times     |
| Data Fetch Failed     | Log warning, set node status to FAILED      | 0 times     |
| Task Cancelled        | Set status to TERMINATED, cleanup resources | N/A         |

---

## Usage Examples

### Example 1: Fetch BTC Price for Trading Decision

**Scenario:** Monitor BTC/USDT 1-hour K-lines, fetch last 50 K-lines for trend analysis.

**Configuration:**

```json
{
  "symbol": "BTCUSDT",
  "interval": "1h",
  "limit": 50
}
```

**Workflow Connection:**

```
Binance Price Node (BTCUSDT, 1h)
    ↓ data
Code Node (Calculate moving averages)
    ↓ ma_signal
Condition Node (Determine buy signal)
    ↓ decision
Buy Node (Execute buy)
```

---

### Example 2: Multi-Pair Price Monitoring

**Scenario:** Monitor price changes for multiple trading pairs simultaneously.

**Workflow Structure:**

```
Binance Price Node (BTCUSDT, 1m) ─┐
Binance Price Node (ETHUSDT, 1m) ─┼─→ Code Node (Price comparison analysis)
Binance Price Node (APTUSDT, 1m) ─┘
```

---

### Example 3: K-line Data Visualization

**Scenario:** Fetch daily K-line data and output to Google Sheets to generate charts.

**Configuration:**

```json
{
  "symbol": "ETHUSDT",
  "interval": "1d",
  "limit": 30
}
```

**Workflow Connection:**

```
Binance Price Node (ETHUSDT, 1d, 30 bars)
    ↓ data
Code Node (Format data as table)
    ↓ formatted_data
Dataset Output Node (Write to Google Sheets)
```

---

## API Configuration

### Environment Variables

The node uses the following environment variables to configure Binance API:

| Environment Variable | Description        | Required |
| -------------------- | ------------------ | -------- |
| `BINANCE_API_KEY`    | Binance API key    | ❌       |
| `BINANCE_API_SECRET` | Binance API secret | ❌       |

**Note:**

- If API Key is not provided, the node will use Binance public API endpoints
- Public API has rate limits; API Key is recommended for high-frequency usage
- Fetching K-line data and current price does not require API Key

### API Limits

According to Binance API documentation:

| Limit Type       | Public API  | With API Key      |
| ---------------- | ----------- | ----------------- |
| **Request Rate** | Lower       | Higher            |
| **Weight Limit** | Shared pool | Independent quota |
| **Max K-lines**  | 1000 bars   | 1000 bars         |

---

## Important Notes

### ⚠️ Important Reminders

1. **Trading Pair Validity**

   - Trading pair symbol must be a valid pair supported by Binance
   - Recommended to select from preset list to avoid invalid symbols
   - Trading pair symbols are automatically converted to uppercase

2. **Time Interval Selection**

   - Different time intervals return different data volumes and precision
   - 1m interval suitable for short-term trading strategies
   - 1d or longer intervals suitable for long-term trend analysis

3. **Data Volume Control**

   - Maximum limit parameter value is 1000
   - Fetching large amounts of historical data may increase response time
   - Recommended to set reasonable limit value based on actual needs

4. **API Stability**

   - Node has built-in connection retry mechanism (up to 3 times)
   - Uses exponential backoff strategy to avoid frequent retries
   - Unstable network may cause node execution failure

5. **Data Precision**
   - Prices and volumes are returned in string format
   - Maintains Binance API original precision
   - Needs conversion to numeric type for calculations

---

## FAQ

**Q: How to process K-line data for technical analysis?**

A: Connect to Code Node and use Python's pandas library to process data:

```python
import pandas as pd

# Input data from Binance Price Node
kline_data = input_data.get('kline_data', [])
header = ['open_time', 'open', 'high', 'low', 'close', 'volume', ...]

# Convert to DataFrame
df = pd.DataFrame(kline_data, columns=header)
df['close'] = df['close'].astype(float)

# Calculate moving averages
df['ma_20'] = df['close'].rolling(window=20).mean()
df['ma_50'] = df['close'].rolling(window=50).mean()

# Output processed data
output_data = {
    'prices': df['close'].tolist(),
    'ma_20': df['ma_20'].tolist(),
    'ma_50': df['ma_50'].tolist()
}
```

---

**Q: What's the difference between current_price and close price in the data output?**

A:

- `data.current_price` comes from real-time ticker endpoint, is the latest trade price
- The `close` price in the last K-line of `data.kline_data` is the closing price for that time period
- If current K-line is not closed yet, the last K-line's close may differ from current_price

---

**Q: How to debug if node execution fails?**

A: Check the following:

1. Is the trading pair symbol correct and supported by Binance
2. Is network connection normal
3. Is API Key valid (if configured)
4. Check node logs for detailed error information
5. Confirm Binance API service is running normally

---

**Q: How to get real-time price instead of K-line data?**

A: The `data` output contains the `current_price` field. You can access it in downstream nodes using `input_data.get('current_price')` to get the real-time price. If you only need the price without full K-line data, you can extract this field in a Code Node.

---

## Related Nodes

- **Code Node** - Used to process and analyze K-line data
- **Condition Node** - Trigger conditional judgment based on price data
- **AI Model Node** - Use AI to analyze price trends
- **Dataset Output Node** - Export price data to external storage

---

## Technical Specifications

| Specification                | Value                                                 |
| ---------------------------- | ----------------------------------------------------- |
| **Node Version**             | 1.0.0                                                 |
| **Supported API Version**    | Binance API v3                                        |
| **Max Concurrent Execution** | Depends on API limits                                 |
| **Execution Mode**           | Single execution (completes after fetching data once) |
| **Resource Cleanup**         | Automatic API client cleanup                          |
| **Log Levels**               | DEBUG, INFO, WARNING, ERROR                           |

---

**Related Documentation:**

- [Nodes and Workflows](../core-concepts/nodes-and-workflows.md) - Node basics
- [Weather Syntax](../core-concepts/weather-syntax.md) - Workflow file format

# Price Node

**Node Type**: `price_node`
**Category**: Input
**Version**: 0.4.0

## Overview

The Price Node fetches cryptocurrency price data from CoinGecko, supporting both real-time current prices and historical OHLC (Open, High, Low, Close) kline data.

## Input Parameters

| Parameter   | Type   | Required | Description                                                                            |
| ----------- | ------ | -------- | -------------------------------------------------------------------------------------- |
| `source`    | string | Yes      | Data source (currently only 'coingecko' is supported)                                  |
| `data_type` | string | Yes      | Type of data to fetch: 'kline' for OHLC data or 'current_price' for current price only |
| `symbol`    | string | Yes      | Cryptocurrency symbol/ID (e.g., 'bitcoin', 'ethereum', 'aptos')                        |

## Output

| Output | Type        | Description                                               |
| ------ | ----------- | --------------------------------------------------------- |
| `data` | json object | Price data containing source, data_type, symbol, and data |

## Data Output Format

### Current Price

```json
{
  "source": "coingecko",
  "data_type": "current_price",
  "symbol": "bitcoin",
  "data": {
    "coin_id": "bitcoin",
    "current_price": 50000.0,
    "last_updated": 1700000000
  }
}
```

### OHLC/Kline Data

```json
{
  "source": "coingecko",
  "data_type": "kline",
  "symbol": "bitcoin",
  "data": {
    "coin_id": "bitcoin",
    "vs_currency": "usd",
    "days": 30,
    "ohlc": [
      [1700000000000, 35000, 36000, 34500, 35800],
      [timestamp, open, high, low, close]
    ],
    "count": 120
  }
}
```

## Credits Cost

- **Current Price**: 2 credits (lower cost for simple price query)
- **OHLC/Kline Data**: 5 credits (higher cost due to larger dataset)

## Common Use Cases

1. **Price Monitoring**: Fetch current prices for trading decisions
2. **Technical Analysis**: Get historical OHLC data for pattern recognition
3. **Alert Systems**: Monitor price movements and trigger notifications
4. **Portfolio Tracking**: Track asset prices in real-time

## Example Usage

### Scenario 1: Get Bitcoin Current Price

```
Inputs:
- source: "coingecko"
- data_type: "current_price"
- symbol: "bitcoin"

Output:
- Current BTC price in USD
```

### Scenario 2: Get Ethereum OHLC Data

```
Inputs:
- source: "coingecko"
- data_type: "kline"
- symbol: "ethereum"

Output:
- Last 30 days of ETH OHLC data
```

## Notes

- Data is fetched from CoinGecko API via the Monitor service
- Symbol must be a valid CoinGecko coin ID (lowercase)
- OHLC data includes 30 days of historical data
- Multi-key support ensures high availability and rate limit management

## Migration from Binance Price Node

The Price Node replaces the deprecated Binance Price Node with the following changes:

| Old (Binance)               | New (CoinGecko)                          |
| --------------------------- | ---------------------------------------- |
| `symbol` (e.g., "BTCUSDT")  | `symbol` (e.g., "bitcoin")               |
| `interval` (e.g., "1h")     | `data_type` ("kline" or "current_price") |
| `limit` (number of candles) | Not configurable (fixed to 30 days)      |
| Binance API                 | CoinGecko API                            |

## Related Nodes

- **AI Model Node**: Use price data as input for AI analysis
- **Code Node**: Process price data with custom Python code
- **Buy/Sell Nodes**: Make trading decisions based on price data

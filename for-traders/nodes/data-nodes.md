# Data Nodes

Data nodes are the foundation of any trading strategy. They collect, process, and provide real-time information from various sources to feed your decision-making workflows.

## Price Feed Nodes

### Binance Price Node

**Real-time market data from the world's largest cryptocurrency exchange.**

#### Key Features
- Real-time price streaming
- Historical OHLCV data
- Volume and liquidity metrics
- Multiple timeframe support
- Rate limit optimized

#### Configuration Parameters
```javascript
{
  symbol: "BTCUSDT",              // Trading pair
  interval: "1h",                 // 1m, 5m, 15m, 1h, 4h, 1d
  limit: 100,                     // Data points to fetch
  include_volume: true,           // Include volume data
  stream_mode: "websocket"        // "rest" or "websocket"
}
```

#### Output Data Structure
```javascript
{
  symbol: "BTCUSDT",
  current_price: 43250.50,
  price_change_24h: 2.34,        // Percentage change
  volume_24h: 1250000,           // 24h volume in quote currency
  high_24h: 44000,
  low_24h: 42800,
  timestamp: "2024-01-15T10:30:00Z",
  bid_price: 43249.50,
  ask_price: 43251.50,
  spread: 0.0046,                // Bid-ask spread percentage
  kline_data: [
    {
      open_time: 1705315200000,
      open: 43000,
      high: 43500,
      low: 42900,
      close: 43250,
      volume: 125.67,
      close_time: 1705318799999,
      quote_volume: 5425632.50,
      trades_count: 8543,
      taker_buy_volume: 62.33,
      taker_buy_quote_volume: 2698451.25
    }
  ]
}
```

#### Best Use Cases
- **DCA Strategies**: Monitor price for regular purchases
- **Technical Analysis**: Source data for indicators
- **Arbitrage Detection**: Compare with other exchanges
- **Portfolio Valuation**: Real-time asset pricing

#### Example Integration
```
Binance Price Node → Technical Analysis Node → Buy Signal Logic
        ↓                      ↓                     ↓
   BTC/USDT Price       RSI Calculation         RSI < 30 = Buy
```

### DEX Price Aggregator Node

**Multi-chain DEX price aggregation with liquidity analysis.**

#### Supported DEXs
- **Aptos**: Hyperion DEX, Liquidswap
- **Flow EVM**: PunchSwap V2
- **BSC**: PancakeSwap V2/V3
- **Ethereum**: Uniswap V2/V3

#### Advanced Features
```javascript
{
  token_pair: "APT/USDC",
  chains: ["aptos", "flow_evm"],
  include_liquidity: true,
  price_impact_analysis: true,
  trade_sizes: [100, 1000, 10000]  // Analyze price impact for different sizes
}
```

#### Liquidity Analysis Output
```javascript
{
  aggregated_price: 8.45,
  best_price_source: "hyperion_dex",
  price_sources: [
    {
      dex: "hyperion_dex",
      chain: "aptos",
      price: 8.45,
      liquidity_usd: 1250000,
      price_impact_1k: 0.12,
      price_impact_10k: 1.23
    }
  ],
  arbitrage_opportunities: [
    {
      buy_dex: "punchswap",
      sell_dex: "hyperion",
      profit_percentage: 0.8,
      max_profitable_size: 5000
    }
  ]
}
```

## Social Media Nodes

### X (Twitter) Listener Node

**Real-time social sentiment analysis from Twitter.**

#### Advanced Configuration
```javascript
{
  search_mode: "advanced_search",     // "user_tweets" or "advanced_search"
  query_type: "Latest",               // "Latest" or "Top"
  accounts: ["elonmusk", "cz_binance", "VitalikButerin"],
  keywords: "Bitcoin, BTC, bullish, bearish",
  sentiment_analysis: true,
  influence_weighting: true,          // Weight by follower count
  language_filter: ["en"],
  exclude_retweets: false,
  min_engagement: 10                  // Minimum likes + retweets
}
```

#### Sentiment Analysis Features
- **VADER Sentiment**: Rule-based sentiment analysis
- **Custom Crypto Lexicon**: Crypto-specific sentiment words
- **Emoji Analysis**: Decode emoji sentiment
- **Context Understanding**: Sarcasm and context detection

#### Output with Enhanced Analytics
```javascript
{
  tweets: [
    {
      id: "1234567890",
      text: "Bitcoin hitting new ATH! 🚀📈",
      author: {
        username: "crypto_whale",
        follower_count: 150000,
        influence_score: 0.85
      },
      metrics: {
        retweet_count: 245,
        like_count: 1280,
        reply_count: 89,
        engagement_rate: 0.067
      },
      sentiment: {
        score: 0.8,                   // -1 to +1
        confidence: 0.92,
        emotions: ["joy", "excitement"],
        keywords_matched: ["Bitcoin", "ATH"]
      },
      market_relevance: 0.9,
      created_at: "2024-01-15T10:30:00Z"
    }
  ],
  summary: {
    total_tweets: 25,
    sentiment_distribution: {
      positive: 16,
      neutral: 6,
      negative: 3
    },
    average_sentiment: 0.34,
    trending_keywords: ["Bitcoin", "ATH", "bullish"],
    influence_weighted_sentiment: 0.41
  }
}
```

#### Real-World Applications

**Sentiment-Based DCA**:
```
Twitter Listener → Sentiment Analysis → DCA Amount Adjustment
       ↓                ↓                      ↓
   Crypto Tweets    Sentiment Score     High Fear = Buy More
                   (Fear & Greed)       High Greed = Buy Less
```

**News Event Detection**:
```javascript
// Example: Detect major announcements
if (tweet.author.influence_score > 0.8 && 
    tweet.sentiment.score > 0.7 && 
    tweet.metrics.engagement_rate > 0.1) {
    return {
        event_type: "major_announcement",
        urgency: "high",
        recommended_action: "monitor_closely"
    };
}
```

### Reddit Sentiment Node

**Community sentiment from cryptocurrency subreddits.**

#### Target Subreddits
- r/CryptoCurrency (general sentiment)
- r/Bitcoin (Bitcoin-specific)
- r/ethereum (Ethereum ecosystem)
- r/AptosLabs (Aptos community)
- r/CryptoMoonShots (high-risk sentiment)

#### Advanced Analytics
```javascript
{
  sentiment_with_upvotes: 0.65,      // Weighted by upvote count
  comment_sentiment: 0.58,           // Comment thread analysis
  hot_topics: [
    {
      keyword: "DeFi",
      mention_count: 234,
      sentiment: 0.42,
      trending_score: 0.8
    }
  ],
  whale_alerts: [
    {
      post_title: "Just bought 1000 BTC",
      credibility_score: 0.3,
      market_impact_estimate: "low"
    }
  ]
}
```

## Market Data Nodes

### Technical Analysis Node

**Comprehensive technical indicator calculations.**

#### Supported Indicators
- **Trend**: SMA, EMA, MACD, ADX
- **Momentum**: RSI, Stochastic, Williams %R
- **Volatility**: Bollinger Bands, ATR, Standard Deviation
- **Volume**: OBV, Volume SMA, VWAP
- **Support/Resistance**: Pivot Points, Fibonacci

#### Configuration Example
```javascript
{
  indicators: [
    {
      type: "RSI",
      period: 14,
      overbought: 70,
      oversold: 30
    },
    {
      type: "MACD",
      fast_period: 12,
      slow_period: 26,
      signal_period: 9
    },
    {
      type: "BOLLINGER_BANDS",
      period: 20,
      std_dev: 2
    }
  ],
  timeframes: ["1h", "4h", "1d"],
  min_data_points: 100
}
```

#### Multi-Timeframe Analysis
```javascript
{
  RSI: {
    "1h": {
      current: 65.2,
      trend: "neutral",
      signal: "hold"
    },
    "4h": {
      current: 72.8,
      trend: "overbought",
      signal: "caution"
    },
    "1d": {
      current: 58.3,
      trend: "neutral",
      signal: "hold"
    },
    confluence: "mixed_signals"
  },
  MACD: {
    "1h": {
      macd: 125.5,
      signal: 118.2,
      histogram: 7.3,
      trend: "bullish_crossover"
    }
  },
  overall_signal: {
    direction: "neutral",
    strength: 0.4,
    confidence: 0.6
  }
}
```

### On-Chain Analytics Node

**Blockchain metrics and whale activity monitoring.**

#### Supported Metrics
- **Network Activity**: Transaction count, active addresses
- **Whale Movements**: Large transfers, exchange flows
- **DeFi Metrics**: TVL changes, yield farming activity
- **Token Metrics**: Holder distribution, supply changes

#### Whale Alert Integration
```javascript
{
  large_transactions: [
    {
      hash: "0xabc123...",
      from: "unknown_wallet",
      to: "binance_hot_wallet",
      amount: "1000.00",
      token: "BTC",
      usd_value: 43250000,
      significance: "very_high",
      market_impact_estimate: "negative"
    }
  ],
  exchange_flows: {
    net_flow_24h: -2500.5,           // Negative = outflow (bullish)
    inflow_24h: 8932.2,
    outflow_24h: 11432.7,
    trend: "increasing_outflows"
  }
}
```

## External Data Integration

### Dataset Node

**Flexible external data integration.**

#### Supported Data Sources
- **Google Sheets**: Real-time spreadsheet data
- **CSV Files**: Local and remote CSV processing
- **REST APIs**: Custom API integration
- **Database Connections**: PostgreSQL, MySQL

#### Google Sheets Integration
```javascript
{
  mode: "read",
  doc_link: "https://docs.google.com/spreadsheets/d/...",
  sheet_name: "TradingSignals",
  range: "A1:E100",
  auto_refresh: 300,                // Refresh every 5 minutes
  data_validation: true
}
```

#### Advanced Data Processing
```javascript
{
  headers: ["Date", "Symbol", "Signal", "Confidence", "Notes"],
  data: [
    ["2024-01-15", "BTC", "BUY", "0.85", "Strong technical breakout"],
    ["2024-01-15", "ETH", "HOLD", "0.60", "Waiting for confirmation"]
  ],
  processed_signals: [
    {
      symbol: "BTC",
      signal: "BUY",
      confidence: 0.85,
      timestamp: "2024-01-15T10:30:00Z",
      source: "external_research",
      weight: 0.3                   // Weight in overall decision
    }
  ]
}
```

### News Feed Node

**Automated news monitoring and sentiment analysis.**

#### News Sources
- **CoinTelegraph**: Latest crypto news
- **CoinDesk**: Market analysis and updates
- **RSS Feeds**: Custom RSS feed monitoring
- **Press Releases**: Official announcements

#### News Impact Analysis
```javascript
{
  articles: [
    {
      title: "Bitcoin ETF Approved by SEC",
      source: "coindesk",
      published: "2024-01-15T09:00:00Z",
      sentiment: 0.9,
      market_impact: "very_high",
      affected_assets: ["BTC", "ETH", "crypto_index"],
      article_url: "https://...",
      summary: "SEC approves first Bitcoin ETF...",
      credibility_score: 0.95
    }
  ],
  market_moving_news: [
    {
      event_type: "regulatory_approval",
      impact_score: 0.9,
      time_sensitivity: "immediate",
      recommended_action: "monitor_price_action"
    }
  ]
}
```

## Data Quality and Reliability

### Data Validation Features

#### Real-Time Quality Checks
- **Latency Monitoring**: Track data freshness
- **Anomaly Detection**: Identify unusual values
- **Source Comparison**: Cross-validate between sources
- **Missing Data Handling**: Graceful degradation

#### Quality Metrics
```javascript
{
  data_quality: {
    freshness_score: 0.95,          // How recent is the data
    completeness: 0.98,             // Percentage of expected data points
    accuracy_score: 0.92,           // Compared to reference sources
    reliability: 0.96,              // Historical uptime/accuracy
    confidence_interval: 0.05       // Margin of error
  },
  alerts: [
    {
      type: "stale_data",
      message: "Price data is 5 minutes old",
      severity: "warning"
    }
  ]
}
```

### Error Handling and Fallbacks

#### Redundancy Strategies
- **Multiple Data Sources**: Primary and backup sources
- **Cached Data**: Use recent cached data during outages
- **Interpolation**: Fill missing data points intelligently
- **Circuit Breakers**: Prevent cascade failures

---

Ready to make decisions with your data? Explore [Decision Nodes](decision-nodes.md) to learn about processing and analyzing collected information.

## Next Steps
- [Decision Nodes →](decision-nodes.md)
- [Action Nodes →](action-nodes.md)
- [Building Your First Strategy →](../first-strategy.md)

# Integration Ecosystem

TradingFlow's extensive integration ecosystem connects your trading strategies with external services, data sources, and platforms.

## API Integrations

### Webhook Integrations

#### Discord Webhooks

Send trading alerts and notifications to Discord channels.

**Configuration:**
```javascript
{
  webhook_url: "https://discord.com/api/webhooks/1234567890/abcdef...",
  username: "TradingFlow Bot",
  avatar_url: "https://tradingflow.ai/logo.png",
  embeds: true
}
```

**Message Templates:**
```javascript
// Trade execution alert
{
  content: null,
  embeds: [{
    title: "Trade Executed 🚀",
    color: 3066993,  // Green
    fields: [
      { name: "Strategy", value: "DCA BTC", inline: true },
      { name: "Action", value: "BUY", inline: true },
      { name: "Amount", value: "0.005 BTC", inline: true },
      { name: "Price", value: "$43,250", inline: true },
      { name: "Vault", value: "Main Vault", inline: true }
    ],
    timestamp: new Date().toISOString()
  }]
}

// Risk alert
{
  content: "@everyone ⚠️ Risk Alert",
  embeds: [{
    title: "Portfolio Risk Alert",
    color: 15158332,  // Red
    description: "Portfolio drawdown exceeded 10%",
    fields: [
      { name: "Current Drawdown", value: "-12.5%", inline: true },
      { name: "Portfolio Value", value: "$8,750", inline: true },
      { name: "Trigger Level", value: "-10%", inline: true }
    ]
  }]
}
```

#### Slack Integration

Connect TradingFlow with Slack for team notifications.

**Setup:**
```bash
# Create Slack app
# Add incoming webhook
# Get webhook URL
```

**Configuration:**
```javascript
{
  webhook_url: "https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX",
  channel: "#trading-alerts",
  username: "TradingFlow",
  icon_emoji: ":chart_with_upwards_trend:",
  link_names: true
}
```

**Advanced Features:**
```javascript
// Threaded conversations
{
  text: "New trade executed",
  thread_ts: "1609459200.000100",  // Reply in thread
  reply_broadcast: false
}

// Interactive buttons
{
  text: "Strategy requires approval",
  attachments: [{
    text: "Review and approve this trade",
    callback_id: "trade_approval",
    actions: [
      { name: "approve", text: "Approve", type: "button", value: "approve" },
      { name: "reject", text: "Reject", type: "button", value: "reject" }
    ]
  }]
}
```

### Email Integration

**SMTP Configuration:**
```javascript
{
  smtp_server: "smtp.gmail.com",
  smtp_port: 587,
  username: "alerts@tradingflow.ai",
  password: "app_password",  // Use app passwords for Gmail
  use_tls: true,
  from_address: "alerts@tradingflow.ai",
  from_name: "TradingFlow Alerts"
}
```

**Email Templates:**
```html
<!-- Trade Summary Email -->
<h2>Daily Trading Summary</h2>
<p>Here's your trading activity for {{ date }}:</p>

<table border="1">
  <tr>
    <th>Strategy</th>
    <th>Trades</th>
    <th>P&L</th>
    <th>Win Rate</th>
  </tr>
  {{#each strategies}}
  <tr>
    <td>{{ name }}</td>
    <td>{{ tradeCount }}</td>
    <td>{{ pnl }}</td>
    <td>{{ winRate }}%</td>
  </tr>
  {{/each}}
</table>

<p>Total Portfolio Value: {{ portfolioValue }}</p>
<p>24h Change: {{ dailyChange }}</p>
```

## Data Source Integrations

### Financial Data Providers

#### Alpha Vantage Integration

Real-time and historical stock and forex data.

```javascript
class AlphaVantageClient {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://www.alphavantage.co/query';
  }

  async getQuote(symbol) {
    const response = await fetch(
      `${this.baseUrl}?function=GLOBAL_QUOTE&symbol=${symbol}&apikey=${this.apiKey}`
    );
    const data = await response.json();

    return {
      symbol: data['Global Quote']['01. symbol'],
      price: parseFloat(data['Global Quote']['05. price']),
      change: parseFloat(data['Global Quote']['09. change']),
      volume: parseInt(data['Global Quote']['06. volume'])
    };
  }

  async getTimeSeries(symbol, interval = '5min') {
    const functionName = interval === 'daily' ? 'TIME_SERIES_DAILY' : 'TIME_SERIES_INTRADAY';
    const response = await fetch(
      `${this.baseUrl}?function=${functionName}&symbol=${symbol}&interval=${interval}&apikey=${this.apiKey}`
    );
    const data = await response.json();

    return this.parseTimeSeries(data);
  }
}
```

#### Yahoo Finance Integration

Comprehensive market data and fundamentals.

```python
import yfinance as yf

class YahooFinanceClient:
    def __init__(self):
        pass

    def get_stock_data(self, symbol, period="1mo", interval="1d"):
        """Get historical stock data"""
        ticker = yf.Ticker(symbol)
        data = ticker.history(period=period, interval=interval)

        return {
            'symbol': symbol,
            'data': data.to_dict('index'),
            'info': ticker.info
        }

    def get_fundamentals(self, symbol):
        """Get company fundamentals"""
        ticker = yf.Ticker(symbol)

        return {
            'balance_sheet': ticker.balance_sheet.to_dict(),
            'income_stmt': ticker.income_stmt.to_dict(),
            'cash_flow': ticker.cash_flow.to_dict(),
            'earnings': ticker.earnings.to_dict()
        }

    def get_options_chain(self, symbol, expiration_date=None):
        """Get options chain data"""
        ticker = yf.Ticker(symbol)

        if expiration_date:
            options = ticker.option_chain(expiration_date)
        else:
            # Get nearest expiration
            expirations = ticker.options
            options = ticker.option_chain(expirations[0])

        return {
            'calls': options.calls.to_dict(),
            'puts': options.puts.to_dict(),
            'expiration': expiration_date or expirations[0]
        }
```

### Social Media Data

#### Twitter API Integration

Real-time sentiment analysis from Twitter.

```javascript
const TwitterApi = require('twitter-api-v2');

class TwitterSentimentAnalyzer {
  constructor(apiKeys) {
    this.client = new TwitterApi(apiKeys);
  }

  async searchTweets(query, options = {}) {
    const { maxResults = 100, startTime, endTime } = options;

    const tweets = await this.client.v2.search(query, {
      max_results: maxResults,
      'tweet.fields': ['created_at', 'public_metrics', 'context_annotations'],
      'user.fields': ['username', 'verified', 'public_metrics'],
      start_time: startTime,
      end_time: endTime
    });

    return tweets.data.map(tweet => ({
      id: tweet.id,
      text: tweet.text,
      created_at: tweet.created_at,
      metrics: tweet.public_metrics,
      user: tweet.users[tweet.author_id],
      sentiment: this.analyzeSentiment(tweet.text)
    }));
  }

  async getUserTweets(username, options = {}) {
    const { maxResults = 50 } = options;

    const user = await this.client.v2.userByUsername(username);
    const tweets = await this.client.v2.userTimeline(user.data.id, {
      max_results: maxResults,
      'tweet.fields': ['created_at', 'public_metrics']
    });

    return tweets.data;
  }

  analyzeSentiment(text) {
    // Simple sentiment analysis (replace with ML model)
    const positiveWords = ['bullish', 'moon', 'pump', 'up', 'gain', 'profit'];
    const negativeWords = ['bearish', 'dump', 'crash', 'down', 'loss', 'sell'];

    const lowerText = text.toLowerCase();
    const positiveCount = positiveWords.filter(word => lowerText.includes(word)).length;
    const negativeCount = negativeWords.filter(word => lowerText.includes(word)).length;

    const score = (positiveCount - negativeCount) / Math.max(positiveCount + negativeCount, 1);

    return {
      score: score,
      magnitude: positiveCount + negativeCount,
      label: score > 0.1 ? 'positive' : score < -0.1 ? 'negative' : 'neutral'
    };
  }

  async streamTweets(keywords, callback) {
    const stream = await this.client.v2.searchStream({
      'tweet.fields': ['created_at', 'public_metrics']
    });

    // Filter for keywords
    stream.on('data', (tweet) => {
      const hasKeyword = keywords.some(keyword =>
        tweet.data.text.toLowerCase().includes(keyword.toLowerCase())
      );

      if (hasKeyword) {
        callback(tweet.data);
      }
    });

    return stream;
  }
}
```

### Blockchain Data Providers

#### The Graph Protocol Integration

Query decentralized data from subgraphs.

```graphql
# Sample GraphQL queries for Uniswap V3
query GetPoolData($poolId: ID!) {
  pool(id: $poolId) {
    token0 {
      symbol
      decimals
    }
    token1 {
      symbol
      decimals
    }
    feeTier
    liquidity
    volumeUSD
    feesUSD
    token0Price
    token1Price
  }
}

query GetSwaps($poolId: ID!, $first: Int = 100) {
  swaps(
    where: { pool: $poolId }
    orderBy: timestamp
    orderDirection: desc
    first: $first
  ) {
    timestamp
    amount0
    amount1
    amountUSD
    transaction {
      id
    }
  }
}
```

```javascript
class TheGraphClient {
  constructor(endpoint) {
    this.endpoint = endpoint;
  }

  async query(query, variables = {}) {
    const response = await fetch(this.endpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ query, variables })
    });

    const result = await response.json();
    return result.data;
  }

  async getUniswapPoolData(poolAddress) {
    const query = `
      query GetPoolData($poolId: ID!) {
        pool(id: $poolId) {
          token0 { symbol decimals }
          token1 { symbol decimals }
          feeTier
          liquidity
          volumeUSD
          token0Price
          token1Price
        }
      }
    `;

    return await this.query(query, { poolId: poolAddress.toLowerCase() });
  }

  async getRecentSwaps(poolAddress, limit = 100) {
    const query = `
      query GetSwaps($poolId: ID!, $first: Int) {
        swaps(
          where: { pool: $poolId }
          orderBy: timestamp
          orderDirection: desc
          first: $first
        ) {
          timestamp
          amount0
          amount1
          amountUSD
        }
      }
    `;

    return await this.query(query, { poolId: poolAddress.toLowerCase(), first: limit });
  }
}
```

## Trading Platform Integrations

### Broker API Integrations

#### Interactive Brokers TWS API

Connect to Interactive Brokers for advanced order types.

```python
from ib_insync import IB, Stock, MarketOrder, StopOrder, BracketOrder

class InteractiveBrokersClient:
    def __init__(self, host='127.0.0.1', port=7497, client_id=1):
        self.ib = IB()
        self.host = host
        self.port = port
        self.client_id = client_id

    async def connect(self):
        """Connect to IB TWS/Gateway"""
        await self.ib.connectAsync(self.host, self.port, clientId=self.client_id)
        return self.ib.isConnected()

    async def place_market_order(self, symbol, quantity, action='BUY'):
        """Place a market order"""
        contract = Stock(symbol, 'SMART', 'USD')
        order = MarketOrder(action, quantity)

        trade = await self.ib.placeOrderAsync(contract, order)
        return {
            'order_id': trade.order.orderId,
            'status': trade.orderStatus.status,
            'filled': trade.orderStatus.filled,
            'remaining': trade.orderStatus.remaining
        }

    async def place_bracket_order(self, symbol, quantity, entry_price, stop_loss, take_profit):
        """Place a bracket order with stop loss and take profit"""
        contract = Stock(symbol, 'SMART', 'USD')

        # Entry order
        entry_order = MarketOrder('BUY', quantity)

        # Stop loss order
        stop_order = StopOrder('SELL', quantity, stop_loss)

        # Take profit order
        profit_order = MarketOrder('SELL', quantity)

        # Create bracket order
        bracket = BracketOrder(entry_order, profit_order, stop_order)

        # Place the bracket order
        trades = await self.ib.placeOrderAsync(contract, bracket.parent)
        return trades

    async def get_portfolio(self):
        """Get current portfolio positions"""
        positions = await self.ib.positionsAsync()

        return positions.map(pos => ({
            'symbol': pos.contract.symbol,
            'quantity': pos.position,
            'avg_cost': pos.avgCost,
            'market_value': pos.marketValue,
            'unrealized_pnl': pos.unrealizedPNL
        }))

    async def get_account_summary(self):
        """Get account summary"""
        summary = await self.ib.accountSummaryAsync()
        return summary.reduce((acc, item) => {
            acc[item.tag] = item.value;
            return acc;
        }, {});
```

#### Alpaca Trading API

Commission-free stock and crypto trading API.

```python
import alpaca_trade_api as tradeapi

class AlpacaTradingClient:
    def __init__(self, api_key, api_secret, base_url='https://paper-api.alpaca.markets'):
        self.api = tradeapi.REST(api_key, api_secret, base_url, api_version='v2')

    def get_account_info(self):
        """Get account information"""
        account = self.api.get_account()
        return {
            'account_id': account.id,
            'cash': float(account.cash),
            'portfolio_value': float(account.portfolio_value),
            'buying_power': float(account.buying_power),
            'equity': float(account.equity),
            'daytrade_count': account.daytrade_count
        }

    def get_positions(self):
        """Get current positions"""
        positions = self.api.list_positions()
        return [{
            'symbol': pos.symbol,
            'qty': float(pos.qty),
            'avg_entry_price': float(pos.avg_entry_price),
            'current_price': float(pos.current_price),
            'market_value': float(pos.market_value),
            'unrealized_pl': float(pos.unrealized_pl),
            'unrealized_plpc': float(pos.unrealized_plpc)
        } for pos in positions]

    def place_order(self, symbol, qty, side, order_type='market', time_in_force='day', **kwargs):
        """Place an order"""
        order = self.api.submit_order(
            symbol=symbol,
            qty=qty,
            side=side,
            type=order_type,
            time_in_force=time_in_force,
            **kwargs
        )

        return {
            'order_id': order.id,
            'status': order.status,
            'symbol': order.symbol,
            'qty': order.qty,
            'side': order.side,
            'type': order.type,
            'submitted_at': order.submitted_at
        }

    def get_order_history(self, status=None, limit=100):
        """Get order history"""
        orders = self.api.list_orders(status=status, limit=limit)
        return [{
            'order_id': order.id,
            'symbol': order.symbol,
            'qty': order.qty,
            'side': order.side,
            'type': order.type,
            'status': order.status,
            'filled_qty': order.filled_qty,
            'avg_fill_price': order.avg_fill_price,
            'submitted_at': order.submitted_at,
            'filled_at': order.filled_at
        } for order in orders]

    def get_barset(self, symbols, timeframe='1D', limit=100, start=None, end=None):
        """Get historical bar data"""
        barset = self.api.get_barset(symbols, timeframe, limit=limit, start=start, end=end)

        result = {}
        for symbol in symbols:
            bars = barset[symbol]
            result[symbol] = [{
                'timestamp': bar.t,
                'open': bar.o,
                'high': bar.h,
                'low': bar.l,
                'close': bar.c,
                'volume': bar.v
            } for bar in bars]

        return result
```

## Cloud Service Integrations

### AWS Lambda Integration

Serverless function execution for complex computations.

```javascript
const AWS = require('aws-sdk');

class AWSLambdaClient {
  constructor(region = 'us-east-1') {
    this.lambda = new AWS.Lambda({ region });
  }

  async invokeFunction(functionName, payload, invocationType = 'RequestResponse') {
    const params = {
      FunctionName: functionName,
      Payload: JSON.stringify(payload),
      InvocationType: invocationType
    };

    try {
      const response = await this.lambda.invoke(params).promise();

      if (response.Payload) {
        return JSON.parse(response.Payload);
      }

      return { statusCode: response.StatusCode };
    } catch (error) {
      console.error('Lambda invocation error:', error);
      throw error;
    }
  }

  async invokeMLModel(data) {
    """Invoke machine learning model for prediction"""
    return await this.invokeFunction('trading-ml-model', {
      action: 'predict',
      data: data,
      model_version: 'v1.2'
    });
  }

  async processLargeDataset(dataset) {
    """Process large datasets using Lambda"""
    return await this.invokeFunction('data-processor', {
      action: 'process',
      dataset: dataset,
      processing_type: 'technical_analysis'
    }, 'Event');  // Async invocation
  }

  async generateReport(portfolioData) {
    """Generate detailed reports"""
    return await this.invokeFunction('report-generator', {
      action: 'generate',
      data: portfolioData,
      format: 'pdf',
      template: 'monthly_performance'
    });
  }
}
```

### Google Cloud Functions

Serverless computing with Google Cloud.

```python
from google.cloud import functions_v1
from google.api_core import exceptions
import json

class GoogleCloudFunctionsClient:
    def __init__(self, project_id, region='us-central1'):
        self.project_id = project_id
        self.region = region
        self.client = functions_v1.CloudFunctionsServiceClient()

    def invoke_function(self, function_name, payload):
        """Invoke a Cloud Function"""
        name = self.client.cloud_function_path(self.project_id, self.region, function_name)

        request = functions_v1.CallFunctionRequest(
            name=name,
            data=json.dumps(payload).encode('utf-8')
        )

        try:
            response = self.client.call_function(request=request)
            return json.loads(response.result)
        except exceptions.GoogleAPICallError as e:
            print(f"Error calling function: {e}")
            raise

    def analyze_sentiment(self, text_data):
        """Analyze sentiment using Cloud Function"""
        return self.invoke_function('sentiment-analyzer', {
            'action': 'analyze',
            'texts': text_data,
            'model': 'bert'
        })

    def calculate_risk_metrics(self, portfolio_data):
        """Calculate risk metrics"""
        return self.invoke_function('risk-calculator', {
            'action': 'calculate',
            'portfolio': portfolio_data,
            'metrics': ['var', 'sharpe', 'drawdown']
        })

    def optimize_portfolio(self, assets, constraints):
        """Portfolio optimization"""
        return self.invoke_function('portfolio-optimizer', {
            'action': 'optimize',
            'assets': assets,
            'constraints': constraints,
            'objective': 'max_sharpe'
        })
```

## Database Integrations

### PostgreSQL Integration

Advanced analytics and historical data storage.

```python
import psycopg2
import psycopg2.extras
from psycopg2.pool import SimpleConnectionPool

class PostgreSQLClient:
    def __init__(self, host, database, user, password, port=5432, min_conn=1, max_conn=10):
        self.pool = SimpleConnectionPool(
            min_conn, max_conn,
            host=host,
            database=database,
            user=user,
            password=password,
            port=port
        )

    def get_connection(self):
        """Get database connection from pool"""
        return self.pool.getconn()

    def release_connection(self, conn):
        """Return connection to pool"""
        self.pool.putconn(conn)

    def execute_query(self, query, params=None, fetch=True):
        """Execute SQL query"""
        conn = self.get_connection()
        try:
            with conn.cursor(cursor_factory=psycopg2.extras.RealDictCursor) as cursor:
                cursor.execute(query, params or [])

                if fetch:
                    result = cursor.fetchall()
                    return [dict(row) for row in result]
                else:
                    conn.commit()
                    return cursor.rowcount
        finally:
            self.release_connection(conn)

    def store_trade_data(self, trade_data):
        """Store trade execution data"""
        query = """
        INSERT INTO trades (
            vault_id, strategy_id, symbol, side, quantity,
            price, executed_at, exchange, fees
        ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
        """

        params = (
            trade_data['vault_id'],
            trade_data['strategy_id'],
            trade_data['symbol'],
            trade_data['side'],
            trade_data['quantity'],
            trade_data['price'],
            trade_data['executed_at'],
            trade_data['exchange'],
            trade_data['fees']
        )

        return self.execute_query(query, params, fetch=False)

    def get_portfolio_history(self, vault_id, start_date, end_date):
        """Get historical portfolio values"""
        query = """
        SELECT date, total_value, cash, invested
        FROM portfolio_history
        WHERE vault_id = %s AND date BETWEEN %s AND %s
        ORDER BY date ASC
        """

        return self.execute_query(query, (vault_id, start_date, end_date))

    def calculate_performance_metrics(self, vault_id, period_days=30):
        """Calculate performance metrics"""
        query = """
        WITH period_data AS (
            SELECT total_value,
                   LAG(total_value) OVER (ORDER BY date) as prev_value
            FROM portfolio_history
            WHERE vault_id = %s
            AND date >= CURRENT_DATE - INTERVAL '%s days'
            ORDER BY date
        )
        SELECT
            AVG(CASE WHEN prev_value > 0 THEN (total_value - prev_value) / prev_value END) as avg_daily_return,
            STDDEV(CASE WHEN prev_value > 0 THEN (total_value - prev_value) / prev_value END) as volatility,
            MIN(total_value) as min_value,
            MAX(total_value) as max_value
        FROM period_data
        """

        result = self.execute_query(query, (vault_id, period_days))
        return result[0] if result else {}

    def store_market_data(self, symbol, data_points):
        """Bulk insert market data"""
        query = """
        INSERT INTO market_data (symbol, timestamp, open, high, low, close, volume)
        VALUES (%s, %s, %s, %s, %s, %s, %s)
        ON CONFLICT (symbol, timestamp) DO UPDATE SET
            open = EXCLUDED.open,
            high = EXCLUDED.high,
            low = EXCLUDED.low,
            close = EXCLUDED.close,
            volume = EXCLUDED.volume
        """

        conn = self.get_connection()
        try:
            with conn.cursor() as cursor:
                psycopg2.extras.execute_batch(cursor, query, data_points)
                conn.commit()
        finally:
            self.release_connection(conn)
```

### MongoDB Integration

Flexible document storage for unstructured data.

```javascript
const { MongoClient } = require('mongodb');

class MongoDBClient {
  constructor(uri, dbName) {
    this.uri = uri;
    this.dbName = dbName;
    this.client = null;
  }

  async connect() {
    this.client = new MongoClient(this.uri);
    await this.client.connect();
    this.db = this.client.db(this.dbName);
  }

  async disconnect() {
    if (this.client) {
      await this.client.close();
    }
  }

  async storeStrategy(strategyData) {
    """Store strategy configuration"""
    const strategies = this.db.collection('strategies');

    const result = await strategies.insertOne({
      ...strategyData,
      created_at: new Date(),
      version: 1,
      status: 'active'
    });

    return result.insertedId;
  }

  async getStrategy(strategyId) {
    """Retrieve strategy by ID"""
    const strategies = this.db.collection('strategies');
    return await strategies.findOne({ _id: strategyId });
  }

  async updateStrategyPerformance(strategyId, performanceData) {
    """Update strategy performance metrics"""
    const strategies = this.db.collection('strategies');

    await strategies.updateOne(
      { _id: strategyId },
      {
        $set: {
          last_performance_update: new Date(),
          performance: performanceData
        },
        $push: {
          performance_history: {
            date: new Date(),
            ...performanceData
          }
        }
      }
    );
  }

  async storeBacktestResult(strategyId, backtestData) {
    """Store backtest results"""
    const backtests = this.db.collection('backtests');

    await backtests.insertOne({
      strategy_id: strategyId,
      ...backtestData,
      created_at: new Date()
    });
  }

  async getBacktestResults(strategyId, limit = 10) {
    """Get recent backtest results"""
    const backtests = this.db.collection('backtests');

    return await backtests
      .find({ strategy_id: strategyId })
      .sort({ created_at: -1 })
      .limit(limit)
      .toArray();
  }

  async storeMarketSentiment(sentimentData) {
    """Store market sentiment analysis"""
    const sentiment = this.db.collection('market_sentiment');

    await sentiment.insertOne({
      ...sentimentData,
      timestamp: new Date()
    });
  }

  async getSentimentHistory(symbol, hours = 24) {
    """Get sentiment history for symbol"""
    const sentiment = this.db.collection('market_sentiment');
    const cutoff = new Date(Date.now() - hours * 60 * 60 * 1000);

    return await sentiment
      .find({
        symbol: symbol,
        timestamp: { $gte: cutoff }
      })
      .sort({ timestamp: -1 })
      .toArray();
  }

  async aggregateTradeData(pipeline) {
    """Aggregate trade data using MongoDB aggregation"""
    const trades = this.db.collection('trades');
    return await trades.aggregate(pipeline).toArray();
  }

  // Example aggregation: Daily volume by symbol
  async getDailyVolume(symbol, days = 30) {
    const pipeline = [
      {
        $match: {
          symbol: symbol,
          timestamp: {
            $gte: new Date(Date.now() - days * 24 * 60 * 60 * 1000)
          }
        }
      },
      {
        $group: {
          _id: {
            $dateToString: {
              format: '%Y-%m-%d',
              date: '$timestamp'
            }
          },
          totalVolume: { $sum: '$volume' },
          tradeCount: { $sum: 1 },
          avgPrice: { $avg: '$price' }
        }
      },
      {
        $sort: { '_id': 1 }
      }
    ];

    return await this.aggregateTradeData(pipeline);
  }
}
```

---

TradingFlow's integration ecosystem enables seamless connectivity with external services, data providers, and platforms. These integrations extend TradingFlow's capabilities and allow for more sophisticated trading strategies and comprehensive market analysis.

## Next Steps
- [Performance Optimization →](performance-optimization.md) - Optimize your integrations
- [Monitoring & Analytics →](monitoring-analytics.md) - Track integration performance
- [Developer Resources →](../resources/developer-resources.md) - Build custom integrations

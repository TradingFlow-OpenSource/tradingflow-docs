# Tutorials & Guides

Comprehensive learning resources to help you master TradingFlow, from beginner basics to advanced techniques.

## Getting Started Tutorials

### 1. Platform Setup (15 minutes)

**Goal**: Complete platform setup and first login

#### Step-by-Step Guide

1. **Account Creation**
   - Visit app.tradingflow.ai
   - Choose email or Web3 wallet signup
   - Complete KYC verification
   - Set up two-factor authentication

2. **Wallet Connection**
   ```javascript
   // Supported wallets
   const supportedWallets = [
     'MetaMask',     // Ethereum, BSC, Flow EVM
     'Phantom',      // Solana
     'Petra',        // Aptos
     'Martian'       // Aptos
   ];
   ```

3. **Security Setup**
   - Enable biometric authentication
   - Set up emergency contact
   - Configure notification preferences
   - Review security best practices

#### Video Tutorial
**"Complete Platform Setup"** - 15-minute guided walkthrough

### 2. First Vault Creation (20 minutes)

**Goal**: Create and fund your first trading vault

#### Prerequisites
- ✅ Account verified
- ✅ Wallet connected
- ✅ Test funds available

#### Vault Configuration

```json
{
  "name": "My Learning Vault",
  "network": "aptos_testnet",
  "vault_type": "personal",
  "initial_deposit": {
    "token": "APT",
    "amount": 10
  },
  "security_settings": {
    "multi_sig": false,
    "emergency_stop": true,
    "daily_limits": {
      "max_trades": 5,
      "max_volume": 100
    }
  }
}
```

#### Funding Process
1. Select vault from dashboard
2. Click "Deposit" button
3. Choose token and amount
4. Confirm transaction in wallet
5. Wait for confirmation

### 3. Building Your First Strategy (30 minutes)

**Goal**: Create a simple Dollar-Cost Averaging strategy

#### Strategy Components

**Timer Node**: Daily execution trigger
```javascript
{
  "schedule": "daily",
  "time": "09:00",
  "timezone": "UTC",
  "enabled": true
}
```

**Price Feed Node**: Get current BTC price
```javascript
{
  "exchange": "Binance",
  "symbol": "BTCUSDT",
  "update_interval": "real-time"
}
```

**Calculator Node**: Determine purchase amount
```javascript
{
  "operation": "fixed_amount",
  "amount_usd": 50,
  "currency": "USD"
}
```

**Swap Node**: Execute the purchase
```javascript
{
  "from_token": "USDC",
  "to_token": "BTC",
  "dex": "hyperion",
  "slippage": 1.0
}
```

#### Workflow Connection
```
Timer → Price Feed → Calculator → Swap
   ↓        ↓           ↓         ↓
Trigger  Get Price   Calc Amount  Execute Trade
```

## Intermediate Tutorials

### 4. Technical Analysis Integration (45 minutes)

**Goal**: Add technical indicators to your strategies

#### RSI-Based Trading Strategy

```javascript
// RSI Configuration
const rsiConfig = {
  period: 14,
  overbought: 70,
  oversold: 30,
  timeframe: "4h"
};

// Strategy Logic
function rsiStrategy(price, rsi) {
  if (rsi <= 30) {
    return {
      action: "BUY",
      reason: "RSI oversold",
      confidence: 0.8
    };
  } else if (rsi >= 70) {
    return {
      action: "SELL",
      reason: "RSI overbought",
      confidence: 0.8
    };
  }
  return { action: "HOLD" };
}
```

#### MACD Divergence Strategy

```javascript
// MACD Configuration
const macdConfig = {
  fast_period: 12,
  slow_period: 26,
  signal_period: 9,
  timeframe: "1h"
};

// Signal Detection
function detectMacdSignal(macd, signal, histogram) {
  const prevHistogram = getPreviousHistogram();
  const currentHistogram = histogram;

  // Bullish crossover
  if (prevHistogram < 0 && currentHistogram > 0 && macd > signal) {
    return { signal: "BUY", strength: "strong" };
  }

  // Bearish crossover
  if (prevHistogram > 0 && currentHistogram < 0 && macd < signal) {
    return { signal: "SELL", strength: "strong" };
  }

  return { signal: "HOLD" };
}
```

### 5. Multi-Chain Arbitrage (60 minutes)

**Goal**: Profit from price differences across chains

#### Arbitrage Setup

```javascript
// Chain Configuration
const chains = {
  aptos: {
    dex: "hyperion",
    tokens: ["APT", "USDC", "BTC"]
  },
  flow_evm: {
    dex: "punchswap",
    tokens: ["FLOW", "USDC", "ETH"]
  }
};

// Arbitrage Logic
class ArbitrageEngine {
  async findArbitrageOpportunity(token) {
    // Get prices from both chains
    const aptosPrice = await getAptosPrice(token);
    const flowPrice = await getFlowPrice(token);

    // Calculate spread
    const spread = Math.abs(aptosPrice - flowPrice) / Math.min(aptosPrice, flowPrice);

    // Check if profitable after fees
    const bridgeFee = 0.003;  // 0.3% bridge fee
    const dexFee = 0.003;     // 0.3% DEX fee
    const totalFees = bridgeFee + dexFee * 2;

    if (spread > totalFees + 0.005) {  // 0.5% minimum profit
      return {
        token,
        direction: aptosPrice > flowPrice ? "SELL_APTOS_BUY_FLOW" : "SELL_FLOW_BUY_APTOS",
        spread: spread,
        estimatedProfit: spread - totalFees
      };
    }

    return null;
  }
}
```

#### Risk Management

```javascript
// Arbitrage Risk Controls
const arbitrageRisk = {
  maxPositionSize: 1000,      // $1000 max per trade
  minSpread: 0.008,          // 0.8% minimum spread
  maxSlippage: 0.02,         // 2% max slippage
  timeout: 300,               // 5 minutes max execution time
  maxConcurrentTrades: 3      // Max 3 concurrent arbitrages
};
```

## Advanced Tutorials

### 6. AI-Powered Strategy Development (90 minutes)

**Goal**: Build strategies using machine learning and AI

#### Sentiment Analysis Integration

```javascript
// Twitter Sentiment Analysis
class SentimentAnalyzer {
  async analyzeTweets(symbol) {
    const tweets = await fetchRecentTweets(symbol, 100);

    const sentiment = await analyzeSentiment(tweets);

    return {
      overall_sentiment: sentiment.score,  // -1 to +1
      confidence: sentiment.confidence,
      trending_keywords: sentiment.keywords,
      social_volume: tweets.length
    };
  }
}

// AI Trading Decision
function aiTradingDecision(marketData, sentiment, technicals) {
  // Combine multiple signals
  const signals = {
    technical: technicals.rsi < 30 ? 0.8 : technicals.rsi > 70 ? -0.8 : 0,
    sentiment: sentiment.overall_sentiment * 0.6,
    volume: marketData.volume_change > 0.2 ? 0.4 : 0,
    correlation: technicals.correlation_with_btc
  };

  // Weighted decision
  const weights = { technical: 0.4, sentiment: 0.3, volume: 0.2, correlation: 0.1 };
  const decision = Object.entries(signals).reduce((sum, [key, value]) => {
    return sum + value * weights[key];
  }, 0);

  return {
    action: decision > 0.3 ? "BUY" : decision < -0.3 ? "SELL" : "HOLD",
    confidence: Math.abs(decision),
    reasoning: generateReasoning(signals, weights)
  };
}
```

#### Machine Learning Model Training

```python
# Price Prediction Model
import tensorflow as tf
from sklearn.preprocessing import MinMaxScaler

class PricePredictor:
    def __init__(self, lookback_days=30):
        self.lookback_days = lookback_days
        self.scaler = MinMaxScaler()
        self.model = self.build_model()

    def build_model(self):
        model = tf.keras.Sequential([
            tf.keras.layers.LSTM(50, return_sequences=True, input_shape=(self.lookback_days, 5)),
            tf.keras.layers.Dropout(0.2),
            tf.keras.layers.LSTM(50),
            tf.keras.layers.Dense(1)
        ])

        model.compile(optimizer='adam', loss='mse')
        return model

    def train(self, historical_data):
        # Prepare training data
        X, y = self.prepare_data(historical_data)

        # Train model
        self.model.fit(X, y, epochs=100, batch_size=32, validation_split=0.2)

    def predict(self, recent_data):
        # Make prediction
        scaled_data = self.scaler.transform(recent_data)
        prediction = self.model.predict(scaled_data.reshape(1, -1, 5))
        return self.scaler.inverse_transform(prediction)[0][0]
```

### 7. Custom Node Development (120 minutes)

**Goal**: Create custom nodes for specialized trading logic

#### Node Template Structure

```python
from tradingflow.station.common.node_decorators import register_node_type
from tradingflow.station.common.signal_types import SignalType
from tradingflow.station.nodes.node_base import NodeBase, NodeStatus

@register_node_type(
    "custom_indicator_node",
    default_params={
        "period": 14,
        "multiplier": 2,
        "source": "close"
    }
)
class CustomIndicatorNode(NodeBase):
    """
    Custom Indicator Node - Calculates specialized technical indicators
    """

    def _register_input_handles(self):
        return [
            self.create_input_handle(
                name="price_data",
                data_type=dict,
                description="OHLCV price data"
            )
        ]

    def _register_output_handles(self):
        return [
            self.create_output_handle(
                name="indicator_value",
                data_type=float,
                description="Calculated indicator value"
            ),
            self.create_output_handle(
                name="signal",
                data_type=str,
                description="Trading signal"
            )
        ]

    async def execute(self):
        try:
            # Get input data
            price_data = await self.get_input_data("price_data")

            # Calculate indicator
            indicator_value = self.calculate_indicator(price_data)

            # Generate signal
            signal = self.generate_signal(indicator_value, price_data)

            # Send outputs
            await self.send_signal("indicator_value", SignalType.DATA, indicator_value)
            await self.send_signal("signal", SignalType.SIGNAL, signal)

            return NodeStatus.COMPLETED

        except Exception as e:
            self.persist_log(f"Error: {str(e)}", level="ERROR")
            return NodeStatus.FAILED

    def calculate_indicator(self, price_data):
        # Implement your custom indicator logic
        return self.config["period"] * self.config["multiplier"]

    def generate_signal(self, indicator, price_data):
        # Implement signal generation logic
        if indicator > price_data["close"]:
            return "BUY"
        elif indicator < price_data["close"]:
            return "SELL"
        return "HOLD"
```

#### Node Testing Framework

```python
import unittest
from your_custom_node import CustomIndicatorNode

class TestCustomIndicatorNode(unittest.TestCase):
    def setUp(self):
        self.node = CustomIndicatorNode(
            flow_id="test_flow",
            component_id=1,
            cycle=1,
            node_id="test_node",
            name="Test Node"
        )

    def test_indicator_calculation(self):
        test_data = {
            "open": 100,
            "high": 110,
            "low": 95,
            "close": 105,
            "volume": 1000
        }

        result = self.node.calculate_indicator(test_data)
        expected = 14 * 2  # period * multiplier

        self.assertEqual(result, expected)

    def test_signal_generation(self):
        indicator_value = 100
        price_data = {"close": 105}

        signal = self.node.generate_signal(indicator_value, price_data)
        self.assertEqual(signal, "SELL")

if __name__ == '__main__':
    unittest.main()
```

## Strategy Optimization Tutorials

### 8. Backtesting and Optimization (75 minutes)

**Goal**: Test and optimize strategies against historical data

#### Backtesting Framework

```javascript
class Backtester {
  constructor(strategy, data, initialCapital = 10000) {
    this.strategy = strategy;
    this.data = data;
    this.capital = initialCapital;
    this.positions = [];
    this.trades = [];
  }

  async run() {
    const results = {
      totalReturn: 0,
      winRate: 0,
      maxDrawdown: 0,
      sharpeRatio: 0,
      trades: []
    };

    for (let i = 0; i < this.data.length; i++) {
      const currentData = this.data[i];

      // Get strategy signal
      const signal = await this.strategy.analyze(currentData);

      // Execute trade if signal generated
      if (signal.action !== 'HOLD') {
        await this.executeTrade(signal, currentData);
      }

      // Update portfolio value
      this.updatePortfolio(currentData);
    }

    // Calculate final metrics
    results.totalReturn = (this.capital - initialCapital) / initialCapital;
    results.winRate = this.calculateWinRate();
    results.maxDrawdown = this.calculateMaxDrawdown();
    results.sharpeRatio = this.calculateSharpeRatio();
    results.trades = this.trades;

    return results;
  }

  async executeTrade(signal, data) {
    const trade = {
      timestamp: data.timestamp,
      action: signal.action,
      price: data.close,
      size: this.calculatePositionSize(signal),
      reason: signal.reason
    };

    this.trades.push(trade);
  }
}
```

#### Parameter Optimization

```javascript
class StrategyOptimizer {
  constructor(strategyClass, parameterSpace, data) {
    this.strategyClass = strategyClass;
    this.parameterSpace = parameterSpace;
    this.data = data;
  }

  async gridSearch() {
    const results = [];

    // Generate all parameter combinations
    const combinations = this.generateCombinations(this.parameterSpace);

    for (const params of combinations) {
      // Create strategy with parameters
      const strategy = new this.strategyClass(params);

      // Backtest strategy
      const backtester = new Backtester(strategy, this.data);
      const result = await backtester.run();

      results.push({
        parameters: params,
        performance: result
      });
    }

    // Find best performing parameter set
    return results.sort((a, b) => b.performance.sharpeRatio - a.performance.sharpeRatio)[0];
  }

  generateCombinations(parameterSpace) {
    // Generate all combinations of parameters
    // Implementation for grid search
  }
}

// Example usage
const parameterSpace = {
  rsiPeriod: [7, 14, 21],
  overboughtLevel: [70, 75, 80],
  oversoldLevel: [20, 25, 30],
  stopLoss: [0.05, 0.1, 0.15]
};

const optimizer = new StrategyOptimizer(RSIStrategy, parameterSpace, historicalData);
const bestParams = await optimizer.gridSearch();
```

## Specialized Tutorials

### 9. DeFi Yield Optimization (90 minutes)

**Goal**: Maximize returns through DeFi yield farming and liquidity provision

#### Yield Farming Strategy

```javascript
class YieldOptimizer {
  constructor(chains = ['aptos', 'flow_evm']) {
    this.chains = chains;
    this.pools = {};
    this.updatePools();
  }

  async updatePools() {
    for (const chain of this.chains) {
      this.pools[chain] = await this.scanYieldPools(chain);
    }
  }

  async scanYieldPools(chain) {
    const pools = [];

    // Scan DEXs for liquidity pools
    const dexes = this.getDexesForChain(chain);

    for (const dex of dexes) {
      const pools = await dex.getPools();

      for (const pool of pools) {
        const apy = await this.calculateAPY(pool);
        const impermanentLoss = this.calculateImpermanentLoss(pool);
        const netYield = apy - impermanentLoss;

        pools.push({
          dex: dex.name,
          tokens: pool.tokens,
          apy: apy,
          impermanentLoss: impermanentLoss,
          netYield: netYield,
          tvl: pool.tvl,
          volume24h: pool.volume24h
        });
      }
    }

    return pools.sort((a, b) => b.netYield - a.netYield);
  }

  async calculateAPY(pool) {
    // Calculate annualized percentage yield
    const rewards24h = await this.getPoolRewards(pool, 24);
    const tvl = pool.tvl;
    const dailyYield = rewards24h / tvl;
    const apy = Math.pow(1 + dailyYield, 365) - 1;

    return apy;
  }

  calculateImpermanentLoss(pool) {
    // Estimate impermanent loss based on volatility
    const volatility = this.getPoolVolatility(pool);
    const timeHorizon = 30; // days

    // Simplified IL calculation
    const il = (volatility * Math.sqrt(timeHorizon) / 2) ** 2;

    return il;
  }
}
```

### 10. Cross-Chain Portfolio Management (105 minutes)

**Goal**: Manage assets across multiple blockchains efficiently

#### Cross-Chain Rebalancing

```javascript
class CrossChainPortfolioManager {
  constructor(targetAllocations) {
    this.targetAllocations = targetAllocations;
    this.chains = Object.keys(targetAllocations);
    this.updateBalances();
  }

  async updateBalances() {
    this.balances = {};

    for (const chain of this.chains) {
      this.balances[chain] = await this.getChainBalances(chain);
    }
  }

  async getChainBalances(chain) {
    // Get balances from vault contracts
    const vault = this.getVaultForChain(chain);
    return await vault.getBalances();
  }

  async calculateRebalancingTrades() {
    const trades = [];
    const totalValue = this.calculateTotalPortfolioValue();

    for (const chain of this.chains) {
      const currentValue = this.calculateChainValue(chain);
      const targetValue = totalValue * this.targetAllocations[chain];
      const deviation = (currentValue - targetValue) / totalValue;

      if (Math.abs(deviation) > 0.05) {  // 5% rebalancing threshold
        const tradeDirection = deviation > 0 ? 'REDUCE' : 'INCREASE';
        const tradeSize = Math.abs(deviation) * totalValue;

        trades.push({
          chain,
          direction: tradeDirection,
          size: tradeSize,
          reason: 'Portfolio rebalancing'
        });
      }
    }

    return trades;
  }

  calculateTotalPortfolioValue() {
    return Object.values(this.balances).reduce((total, chainBalances) => {
      return total + this.calculateChainValueFromBalances(chainBalances);
    }, 0);
  }

  async executeRebalancing() {
    const trades = await this.calculateRebalancingTrades();

    for (const trade of trades) {
      await this.executeCrossChainTrade(trade);
    }
  }
}
```

## Best Practices Guides

### 11. Risk Management Implementation (60 minutes)

**Goal**: Implement comprehensive risk controls in your strategies

#### Risk Management Framework

```javascript
class RiskManager {
  constructor(riskLimits) {
    this.riskLimits = riskLimits;
    this.currentRisk = {};
    this.updateRiskMetrics();
  }

  async updateRiskMetrics() {
    this.currentRisk = {
      portfolioValue: await this.getPortfolioValue(),
      drawdown: await this.calculateDrawdown(),
      var: await this.calculateVaR(),
      concentration: await this.calculateConcentration(),
      correlation: await this.calculateCorrelation()
    };
  }

  async validateTrade(trade) {
    const tradeRisk = await this.assessTradeRisk(trade);

    // Check against all risk limits
    const validations = {
      sizeLimit: tradeRisk.size <= this.riskLimits.maxTradeSize,
      portfolioImpact: tradeRisk.portfolioImpact <= this.riskLimits.maxPortfolioImpact,
      concentration: tradeRisk.concentration <= this.riskLimits.maxConcentration,
      correlation: tradeRisk.correlation <= this.riskLimits.maxCorrelation,
      liquidity: tradeRisk.liquidity >= this.riskLimits.minLiquidity
    };

    const approved = Object.values(validations).every(v => v);

    return {
      approved,
      riskAssessment: tradeRisk,
      validations,
      recommendations: approved ? [] : this.generateRecommendations(validations)
    };
  }

  generateRecommendations(failedValidations) {
    const recommendations = [];

    if (!failedValidations.sizeLimit) {
      recommendations.push('Reduce trade size to comply with limits');
    }

    if (!failedValidations.portfolioImpact) {
      recommendations.push('Trade would exceed portfolio impact limits');
    }

    if (!failedValidations.concentration) {
      recommendations.push('Consider diversifying to reduce concentration');
    }

    return recommendations;
  }
}
```

### 12. Performance Monitoring and Analytics (45 minutes)

**Goal**: Track and analyze strategy performance comprehensively

#### Performance Dashboard Implementation

```javascript
class PerformanceDashboard {
  constructor(strategyId) {
    this.strategyId = strategyId;
    this.metrics = {};
    this.charts = {};
  }

  async updateMetrics() {
    this.metrics = await this.calculateMetrics();
    this.charts = await this.generateCharts();
  }

  async calculateMetrics() {
    const trades = await this.getTradeHistory();
    const prices = await this.getPriceHistory();

    return {
      totalReturn: this.calculateTotalReturn(trades),
      annualizedReturn: this.calculateAnnualizedReturn(trades),
      volatility: this.calculateVolatility(prices),
      sharpeRatio: this.calculateSharpeRatio(trades),
      maxDrawdown: this.calculateMaxDrawdown(prices),
      winRate: this.calculateWinRate(trades),
      profitFactor: this.calculateProfitFactor(trades),
      calmarRatio: this.calculateCalmarRatio(trades),
      alpha: this.calculateAlpha(trades),
      beta: this.calculateBeta(trades, prices)
    };
  }

  calculateTotalReturn(trades) {
    const initialCapital = 10000; // Starting capital
    const finalValue = trades.reduce((capital, trade) => {
      if (trade.action === 'BUY') {
        return capital - trade.cost;
      } else if (trade.action === 'SELL') {
        return capital + trade.revenue;
      }
      return capital;
    }, initialCapital);

    return (finalValue - initialCapital) / initialCapital;
  }

  calculateWinRate(trades) {
    const winningTrades = trades.filter(trade => trade.pnl > 0);
    return winningTrades.length / trades.length;
  }

  async generateCharts() {
    return {
      equityCurve: await this.generateEquityCurve(),
      drawdownChart: await this.generateDrawdownChart(),
      monthlyReturns: await this.generateMonthlyReturns(),
      tradeDistribution: await this.generateTradeDistribution()
    };
  }
}
```

## Troubleshooting Guides

### Common Issues and Solutions

#### 1. Strategy Not Executing

**Problem**: Timer triggers but trades don't execute
**Solutions**:
- Check vault balance sufficiency
- Verify token approvals for DEX contracts
- Confirm network connectivity
- Review slippage settings
- Check gas limits and network congestion

#### 2. Unexpected Trading Results

**Problem**: Strategy behaves differently than expected
**Solutions**:
- Review node configurations and connections
- Check data feed accuracy and timeliness
- Validate calculation logic
- Test with paper trading first
- Enable detailed logging

#### 3. Performance Issues

**Problem**: Strategy runs slowly or times out
**Solutions**:
- Optimize node execution order
- Cache frequently used data
- Reduce data processing frequency
- Implement parallel processing where possible
- Upgrade to higher performance tier

#### 4. Connection Errors

**Problem**: Nodes fail to communicate
**Solutions**:
- Check network connectivity
- Verify API endpoints and credentials
- Review rate limits and throttling
- Update node versions
- Contact support for persistent issues

### Advanced Troubleshooting

#### Debug Logging Setup

```javascript
// Enable detailed logging
const debugConfig = {
  logLevel: 'DEBUG',
  logDestinations: ['console', 'file', 'database'],
  logRetention: '7d',
  performanceMonitoring: true,
  errorTracking: true
};

// Add debug nodes to strategy
const debugNode = {
  type: 'logger_node',
  config: {
    logInputs: true,
    logOutputs: true,
    logErrors: true,
    performanceMetrics: true
  }
};
```

#### Performance Profiling

```javascript
class PerformanceProfiler {
  constructor() {
    this.metrics = {};
    this.startTimes = {};
  }

  startProfiling(operation) {
    this.startTimes[operation] = Date.now();
  }

  endProfiling(operation) {
    const duration = Date.now() - this.startTimes[operation];

    if (!this.metrics[operation]) {
      this.metrics[operation] = {
        count: 0,
        totalTime: 0,
        avgTime: 0,
        maxTime: 0,
        minTime: Infinity
      };
    }

    const metric = this.metrics[operation];
    metric.count++;
    metric.totalTime += duration;
    metric.avgTime = metric.totalTime / metric.count;
    metric.maxTime = Math.max(metric.maxTime, duration);
    metric.minTime = Math.min(metric.minTime, duration);
  }

  getReport() {
    return Object.entries(this.metrics).map(([operation, metric]) => ({
      operation,
      ...metric
    }));
  }
}
```

---

These tutorials provide comprehensive guidance from basic setup to advanced strategy development. Start with the beginner tutorials and progress through the material at your own pace. Remember to always test strategies in paper trading mode before deploying with real funds.

## Next Steps
- [Developer Resources →](developer-resources.md) - Technical documentation
- [Community & Support →](community-support.md) - Get help and connect
- [Node Reference Guide →](../for-traders/node-reference-guide.md) - Complete node documentation

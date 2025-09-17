# Decision Nodes

Decision nodes analyze data, apply logic, and make trading decisions based on predefined conditions and algorithms.

## AI Model Node

**Purpose**: Integrate large language models and AI for intelligent decision making
**Node Type**: `ai_model_node`
**Category**: Decision

#### Configuration Parameters
```javascript
{
  model_name: "gpt-4",              // AI model selection
  temperature: 0.7,                 // Creativity level (0-1)
  system_prompt: "You are a crypto trading assistant",
  prompt: "Analyze market data and provide trading signals",
  max_tokens: 1000,                 // Response length limit
  auto_format_output: true          // Auto-generate JSON format
}
```

#### Input Handles
- `model` (string): AI model specification
- `prompt` (string): Main instruction prompt
- `parameters` (object): Model parameters

#### Output Handles
- `ai_response` (object): Structured AI analysis
  ```javascript
  {
    decision: "BUY",
    confidence: 0.85,
    reasoning: "Strong bullish signals detected",
    suggested_amount: 1000,
    risk_level: "medium",
    timeframe: "short_term"
  }
  ```

#### Use Cases
- Market sentiment analysis
- Trading signal generation
- Risk assessment
- Strategy optimization

## Code Execution Node

**Purpose**: Run custom Python code for complex decision logic
**Node Type**: `code_node`
**Category**: Decision

#### Configuration Parameters
```javascript
{
  python_code: `
def analyze_market(price_data, volume_data):
    # Custom analysis logic
    rsi = calculate_rsi(price_data)
    volume_trend = analyze_volume(volume_data)
    
    if rsi < 30 and volume_trend == "increasing":
        return {"signal": "BUY", "strength": 0.8}
    elif rsi > 70 and volume_trend == "decreasing":
        return {"signal": "SELL", "strength": 0.7}
    else:
        return {"signal": "HOLD", "strength": 0.3}

result = analyze_market(inputs.price_data, inputs.volume_data)
`,
  timeout: 30,                      // Execution timeout in seconds
  max_memory: 128,                  // Memory limit in MB
  allowed_imports: ["numpy", "pandas", "math", "talib"]
}
```

#### Input Handles
- Dynamic inputs based on code requirements

#### Output Handles
- `output_data` (any): Code execution results
- `execution_logs` (string): Debug information

#### Supported Libraries
- **Data Analysis**: pandas, numpy
- **Technical Analysis**: talib (limited functions)
- **Mathematics**: math, statistics, scipy
- **Machine Learning**: scikit-learn (basic models)

## Condition Logic Node

**Purpose**: Implement if/then/else logic for decision making
**Node Type**: `condition_node`
**Category**: Decision

#### Configuration Parameters
```javascript
{
  logic_type: "simple_comparison",   // "simple_comparison", "range_check", "pattern_match"
  operator: "greater_than",          // "equals", "not_equals", "greater_than", "less_than", etc.
  threshold: 0.05,                   // Comparison value
  true_action: "BUY",                // Action when condition is true
  false_action: "HOLD",              // Action when condition is false
  hysteresis: 0.01                   // Prevent rapid switching
}
```

#### Input Handles
- `input_value` (number): Value to evaluate
- `threshold` (number): Optional dynamic threshold

#### Output Handles
- `decision` (string): BUY, SELL, or HOLD
- `confidence` (number): Decision confidence score

#### Logic Types

**Simple Comparison**
```javascript
if (input_value > threshold) {
  return true_action;  // e.g., "BUY"
} else {
  return false_action; // e.g., "HOLD"
}
```

**Range Check**
```javascript
{
  min_value: 0.02,      // Minimum threshold
  max_value: 0.08,      // Maximum threshold
  inside_action: "HOLD",
  outside_action: "SELL"
}
```

**Pattern Match**
```javascript
{
  pattern: "double_bottom",  // Technical pattern
  confirmation_required: true,
  timeout: 3600              // Pattern timeout in seconds
}
```

## Technical Analysis Node

**Purpose**: Calculate technical indicators and generate signals
**Node Type**: `technical_analysis_node`
**Category**: Decision

#### Configuration Parameters
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
    },
    {
      type: "MOVING_AVERAGE",
      type: "SMA",           // SMA, EMA, WMA
      period: 50
    }
  ],
  signal_generation: "consensus",    // "single", "consensus", "weighted"
  min_confirmation: 2                // Minimum indicators agreeing
}
```

#### Input Handles
- `price_data` (object): OHLCV price data
- `custom_parameters` (object): Dynamic indicator parameters

#### Output Handles
- `technical_signals` (object): Indicator values and signals
  ```javascript
  {
    rsi: {
      value: 65.2,
      signal: "neutral",
      trend: "bullish"
    },
    macd: {
      macd: 125.5,
      signal: 118.2,
      histogram: 7.3,
      crossover: "bullish"
    },
    bollinger_bands: {
      upper: 44500,
      middle: 43200,
      lower: 41900,
      position: "middle",      // upper, middle, lower
      squeeze: false
    },
    overall_signal: {
      direction: "BUY",
      strength: 0.75,
      confidence: 0.82,
      confirming_indicators: 3
    }
  }
  ```

#### Supported Indicators

| Indicator | Parameters | Signals |
|-----------|------------|---------|
| **RSI** | period, overbought, oversold | overbought, oversold, neutral |
| **MACD** | fast, slow, signal | bullish_crossover, bearish_crossover |
| **Stochastic** | %K, %D, slowing | overbought, oversold |
| **Bollinger Bands** | period, std_dev | upper_breakout, lower_breakout, squeeze |
| **Moving Averages** | type, period | golden_cross, death_cross |
| **Williams %R** | period | overbought, oversold |
| **CCI** | period | overbought, oversold |
| **ADX** | period | trending, ranging |

## Sentiment Analysis Node

**Purpose**: Analyze social media and news sentiment
**Node Type**: `sentiment_node`
**Category**: Decision

#### Configuration Parameters
```javascript
{
  sources: ["twitter", "reddit", "news"],
  keywords: ["Bitcoin", "BTC", "cryptocurrency"],
  time_window: "24h",               // Analysis time window
  influence_weighting: true,        // Weight by account influence
  min_engagement: 10,               // Minimum likes/retweets
  language_filter: ["en"],          // Language filtering
  sentiment_model: "vader",         // vader, textblob, custom
  aggregation_method: "weighted_average"  // simple_average, weighted_average
}
```

#### Input Handles
- `social_data` (array): Social media posts and articles
- `custom_keywords` (array): Additional keywords to monitor

#### Output Handles
- `sentiment_score` (object): Aggregated sentiment analysis
  ```javascript
  {
    overall_score: 0.65,            // -1 (bearish) to +1 (bullish)
    confidence: 0.78,
    sources: {
      twitter: {
        score: 0.72,
        volume: 1250,
        influential_posts: 15
      },
      reddit: {
        score: 0.58,
        volume: 890,
        top_subreddits: ["r/Bitcoin", "r/CryptoCurrency"]
      },
      news: {
        score: 0.61,
        articles: 45,
        credibility_weighted: true
      }
    },
    trending_keywords: [
      { word: "bullish", score: 0.8, frequency: 234 },
      { word: "adoption", score: 0.75, frequency: 156 }
    ],
    market_impact: "positive"
  }
  ```

## Risk Assessment Node

**Purpose**: Evaluate and manage trading risk in real-time
**Node Type**: `risk_assessment_node`
**Category**: Decision

#### Configuration Parameters
```javascript
{
  risk_metrics: [
    "value_at_risk",         // VaR calculation
    "expected_shortfall",    // CVaR/ES
    "maximum_drawdown",      // MDD
    "volatility",
    "sharpe_ratio"
  ],
  confidence_level: 0.95,           // 95% confidence for VaR
  time_horizon: 1,                  // 1-day risk
  portfolio_limits: {
    max_drawdown: 0.1,              // 10% maximum drawdown
    max_var: 0.05,                  // 5% maximum VaR
    max_position_size: 0.2          // 20% max single position
  },
  risk_adjustment: "dynamic"        // static, dynamic, adaptive
}
```

#### Input Handles
- `portfolio_data` (object): Current portfolio composition
- `market_data` (object): Market conditions and volatility
- `proposed_trade` (object): Trade under consideration

#### Output Handles
- `risk_assessment` (object): Comprehensive risk analysis
  ```javascript
  {
    overall_risk_level: "medium",
    risk_metrics: {
      value_at_risk: 0.032,          // 3.2% potential loss
      expected_shortfall: 0.045,     // 4.5% CVaR
      max_drawdown: 0.08,            // 8% current MDD
      volatility: 0.22,              // 22% annualized volatility
      sharpe_ratio: 1.15
    },
    position_risks: {
      concentration: 0.15,           // Portfolio concentration
      liquidity: 0.8,                // Liquidity score
      correlation: 0.65              // Asset correlation
    },
    recommendations: {
      action: "reduce_position",
      reason: "VaR exceeds limit",
      suggested_size: 0.5,           // Reduce to 50% of proposed
      alternative_actions: ["hedge", "diversify"]
    },
    approval_status: "conditional"    // approved, rejected, conditional
  }
  ```

## Ensemble Decision Node

**Purpose**: Combine multiple decision sources for consensus trading signals
**Node Type**: `ensemble_node`
**Category**: Decision

#### Configuration Parameters
```javascript
{
  decision_sources: [
    {
      name: "technical_analysis",
      weight: 0.4,
      source_node: "tech_analysis_1"
    },
    {
      name: "sentiment_analysis",
      weight: 0.3,
      source_node: "sentiment_1"
    },
    {
      name: "ai_model",
      weight: 0.3,
      source_node: "ai_model_1"
    }
  ],
  consensus_method: "weighted_majority",  // weighted_majority, strict_consensus, ml_model
  min_agreement: 0.6,                     // Minimum agreement threshold
  confidence_threshold: 0.7,              // Minimum confidence to act
  fallback_strategy: "hold"               // Action when no consensus
}
```

#### Input Handles
- Multiple decision inputs from various sources

#### Output Handles
- `ensemble_decision` (object): Combined decision with confidence
  ```javascript
  {
    final_decision: "BUY",
    confidence: 0.78,
    agreement_level: 0.85,          // How much sources agree
    source_breakdown: {
      technical_analysis: {
        decision: "BUY",
        confidence: 0.82,
        weight: 0.4,
        contribution: 0.328
      },
      sentiment_analysis: {
        decision: "BUY",
        confidence: 0.65,
        weight: 0.3,
        contribution: 0.195
      },
      ai_model: {
        decision: "HOLD",
        confidence: 0.71,
        weight: 0.3,
        contribution: 0.213
      }
    },
    reasoning: "Strong technical and sentiment signals outweigh AI caution",
    risk_adjusted: true
  }
  ```

---

Decision nodes form the intelligence layer of your trading strategies. Combine multiple decision sources for robust, well-informed trading signals.

## Next Steps
- [Action Nodes →](action-nodes.md) - Execute your decisions
- [Real-World Examples →](../real-world-examples.md) - See decision nodes in action
- [Advanced Trading Patterns →](../advanced-trading-patterns.md) - Complex decision strategies

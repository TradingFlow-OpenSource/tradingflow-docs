# Trading Risk Controls

Comprehensive risk management framework for automated trading strategies with multiple layers of protection and real-time monitoring.

## Risk Management Architecture

### Multi-Layer Risk Control System

```
┌─────────────────────────────────────────┐
│         Portfolio Level                 │
│ • Position Limits • Diversification     │
│ • Correlation Checks • Rebalancing     │
├─────────────────────────────────────────┤
│         Strategy Level                  │
│ • Drawdown Limits • Win Rate Tracking  │
│ • Performance Metrics • Stop Losses    │
├─────────────────────────────────────────┤
│         Trade Level                     │
│ • Slippage Protection • Size Limits    │
│ • Price Impact Analysis • Gas Limits   │
├─────────────────────────────────────────┤
│         System Level                    │
│ • Circuit Breakers • Rate Limiting     │
│ • Error Handling • Emergency Stops     │
└─────────────────────────────────────────┘
```

## Portfolio-Level Risk Controls

### Position Sizing Controls

#### Kelly Criterion Implementation

```javascript
class KellyCriterion {
  calculatePositionSize(winProbability, winLossRatio, currentCapital) {
    // Full Kelly: f = (bp - q) / b
    // Where: b = odds received, p = win probability, q = loss probability
    const q = 1 - winProbability;
    const b = winLossRatio;

    const kellyFraction = (b * winProbability - q) / b;

    // Conservative Kelly (Half Kelly)
    const positionSize = kellyFraction * 0.5 * currentCapital;

    // Apply bounds
    const maxPosition = currentCapital * 0.1;  // Max 10% per trade
    const minPosition = currentCapital * 0.01; // Min 1% per trade

    return Math.max(minPosition, Math.min(maxPosition, positionSize));
  }
}
```

#### Value at Risk (VaR) Calculation

```javascript
class ValueAtRisk {
  calculateHistoricalVaR(returns, confidence = 0.95, timeHorizon = 1) {
    // Sort returns in ascending order
    const sortedReturns = [...returns].sort((a, b) => a - b);

    // Find the percentile corresponding to confidence level
    const index = Math.floor((1 - confidence) * sortedReturns.length);
    const varValue = sortedReturns[index];

    // Scale for time horizon (square root of time for volatility scaling)
    return varValue * Math.sqrt(timeHorizon);
  }

  calculateParametricVaR(positionValue, volatility, confidence = 0.95, timeHorizon = 1) {
    // Assuming normal distribution
    const zScore = this.getZScore(confidence);
    return positionValue * zScore * volatility * Math.sqrt(timeHorizon);
  }

  getZScore(confidence) {
    // Z-scores for common confidence levels
    const zScores = {
      0.90: 1.645,
      0.95: 1.96,
      0.99: 2.576,
      0.999: 3.291
    };
    return zScores[confidence] || 1.96;
  }
}
```

### Diversification Controls

#### Asset Correlation Monitoring

```javascript
class CorrelationMonitor {
  calculateCorrelation(asset1Prices, asset2Prices) {
    if (asset1Prices.length !== asset2Prices.length) {
      throw new Error('Price arrays must have same length');
    }

    const n = asset1Prices.length;
    const returns1 = this.calculateReturns(asset1Prices);
    const returns2 = this.calculateReturns(asset2Prices);

    const mean1 = returns1.reduce((a, b) => a + b) / n;
    const mean2 = returns2.reduce((a, b) => a + b) / n;

    let numerator = 0;
    let sumSq1 = 0;
    let sumSq2 = 0;

    for (let i = 0; i < n; i++) {
      const diff1 = returns1[i] - mean1;
      const diff2 = returns2[i] - mean2;

      numerator += diff1 * diff2;
      sumSq1 += diff1 * diff1;
      sumSq2 += diff2 * diff2;
    }

    return numerator / Math.sqrt(sumSq1 * sumSq2);
  }

  checkDiversification(portfolio, maxCorrelation = 0.7) {
    const assets = Object.keys(portfolio);
    const violations = [];

    for (let i = 0; i < assets.length; i++) {
      for (let j = i + 1; j < assets.length; j++) {
        const correlation = this.calculateCorrelation(
          portfolio[assets[i]].prices,
          portfolio[assets[j]].prices
        );

        if (Math.abs(correlation) > maxCorrelation) {
          violations.push({
            asset1: assets[i],
            asset2: assets[j],
            correlation: correlation,
            threshold: maxCorrelation
          });
        }
      }
    }

    return violations;
  }
}
```

#### Sector Exposure Limits

```javascript
const sectorLimits = {
  'defi': 0.3,        // Max 30% in DeFi
  'layer1': 0.4,      // Max 40% in Layer 1
  'gaming': 0.15,     // Max 15% in Gaming
  'infrastructure': 0.25,  // Max 25% in Infrastructure
  'other': 0.2        // Max 20% in other sectors
};

class SectorExposureMonitor {
  monitorExposure(portfolio, sectorMapping) {
    const sectorExposure = {};

    // Calculate current exposure by sector
    Object.entries(portfolio).forEach(([asset, data]) => {
      const sector = sectorMapping[asset];
      if (!sectorExposure[sector]) {
        sectorExposure[sector] = 0;
      }
      sectorExposure[sector] += data.value;
    });

    // Check limits
    const violations = [];
    Object.entries(sectorLimits).forEach(([sector, limit]) => {
      const exposure = sectorExposure[sector] || 0;
      if (exposure > limit) {
        violations.push({
          sector,
          currentExposure: exposure,
          limit,
          excess: exposure - limit
        });
      }
    });

    return violations;
  }
}
```

## Strategy-Level Risk Controls

### Drawdown Protection

#### Maximum Drawdown Limits

```javascript
class DrawdownProtector {
  constructor(maxDrawdown = 0.1) {  // 10% max drawdown
    this.maxDrawdown = maxDrawdown;
    this.peakValue = 0;
    this.currentDrawdown = 0;
  }

  update(portfolioValue) {
    if (portfolioValue > this.peakValue) {
      this.peakValue = portfolioValue;
    }

    this.currentDrawdown = (this.peakValue - portfolioValue) / this.peakValue;

    return {
      currentDrawdown: this.currentDrawdown,
      maxDrawdown: this.maxDrawdown,
      breached: this.currentDrawdown > this.maxDrawdown,
      action: this.currentDrawdown > this.maxDrawdown ? 'STOP_TRADING' : 'CONTINUE'
    };
  }

  reset() {
    this.peakValue = 0;
    this.currentDrawdown = 0;
  }
}
```

#### Trailing Stop Loss

```javascript
class TrailingStopLoss {
  constructor(trailingPercent = 0.05, activationProfit = 0.02) {
    this.trailingPercent = trailingPercent;
    this.activationProfit = activationProfit;
    this.activated = false;
    this.highestValue = 0;
    this.stopLevel = 0;
  }

  update(currentValue, entryValue) {
    // Activate trailing stop if profit threshold reached
    const currentProfit = (currentValue - entryValue) / entryValue;

    if (!this.activated && currentProfit >= this.activationProfit) {
      this.activated = true;
      this.highestValue = currentValue;
      this.stopLevel = currentValue * (1 - this.trailingPercent);
    }

    // Update trailing stop if activated
    if (this.activated) {
      if (currentValue > this.highestValue) {
        this.highestValue = currentValue;
        this.stopLevel = this.highestValue * (1 - this.trailingPercent);
      }

      const shouldStop = currentValue <= this.stopLevel;
      return {
        activated: true,
        stopLevel: this.stopLevel,
        currentValue,
        shouldStop,
        action: shouldStop ? 'SELL' : 'HOLD'
      };
    }

    return {
      activated: false,
      currentProfit,
      action: 'HOLD'
    };
  }
}
```

### Performance-Based Controls

#### Sharpe Ratio Monitoring

```javascript
class PerformanceMonitor {
  constructor(riskFreeRate = 0.02) {  // 2% risk-free rate
    this.riskFreeRate = riskFreeRate;
    this.returns = [];
    this.timestamps = [];
  }

  addReturn(returnValue, timestamp) {
    this.returns.push(returnValue);
    this.timestamps.push(timestamp);

    // Keep only last 252 trading days (1 year)
    if (this.returns.length > 252) {
      this.returns.shift();
      this.timestamps.shift();
    }
  }

  calculateSharpeRatio() {
    if (this.returns.length < 2) return 0;

    const avgReturn = this.returns.reduce((a, b) => a + b) / this.returns.length;
    const variance = this.returns.reduce((sum, ret) => {
      return sum + Math.pow(ret - avgReturn, 2);
    }, 0) / (this.returns.length - 1);

    const volatility = Math.sqrt(variance * 252);  // Annualized
    const annualizedReturn = avgReturn * 252;

    return (annualizedReturn - this.riskFreeRate) / volatility;
  }

  getPerformanceMetrics() {
    const sharpeRatio = this.calculateSharpeRatio();
    const totalReturn = this.returns.reduce((a, b) => a + b, 0);
    const volatility = this.calculateVolatility();

    return {
      sharpeRatio,
      totalReturn,
      volatility,
      riskAdjustedReturn: sharpeRatio > 1 ? 'GOOD' : sharpeRatio > 0 ? 'FAIR' : 'POOR'
    };
  }

  calculateVolatility() {
    if (this.returns.length < 2) return 0;

    const avgReturn = this.returns.reduce((a, b) => a + b) / this.returns.length;
    const variance = this.returns.reduce((sum, ret) => {
      return sum + Math.pow(ret - avgReturn, 2);
    }, 0) / (this.returns.length - 1);

    return Math.sqrt(variance * 252);  // Annualized volatility
  }
}
```

## Trade-Level Risk Controls

### Slippage Protection

#### Dynamic Slippage Calculation

```javascript
class SlippageProtector {
  constructor(baseSlippage = 0.005, maxSlippage = 0.05) {
    this.baseSlippage = baseSlippage;  // 0.5%
    this.maxSlippage = maxSlippage;    // 5%
  }

  calculateSlippageTolerance(tradeSize, liquidity, volatility) {
    // Adjust slippage based on market conditions
    let slippage = this.baseSlippage;

    // Larger trades need more slippage tolerance
    if (tradeSize > 10000) {  // > $10k
      slippage *= 1.5;
    }

    // Low liquidity increases slippage
    if (liquidity < 100000) {  // < $100k liquidity
      slippage *= 2;
    }

    // High volatility increases slippage
    if (volatility > 0.05) {  // > 5% daily volatility
      slippage *= 1.3;
    }

    // Cap at maximum slippage
    return Math.min(slippage, this.maxSlippage);
  }

  validateSlippage(actualSlippage, allowedSlippage) {
    return {
      withinLimits: actualSlippage <= allowedSlippage,
      actualSlippage,
      allowedSlippage,
      excessSlippage: Math.max(0, actualSlippage - allowedSlippage)
    };
  }
}
```

#### Price Impact Analysis

```javascript
class PriceImpactAnalyzer {
  calculatePriceImpact(tradeSize, liquidity, currentPrice) {
    // Simplified price impact model
    // Price impact = (Trade Size / (Trade Size + Liquidity)) * some factor
    const totalDepth = tradeSize + liquidity;
    const impactFactor = 0.1;  // Empirical factor

    const priceImpact = (tradeSize / totalDepth) * impactFactor;

    return {
      priceImpact,
      expectedPrice: currentPrice * (1 - priceImpact),
      confidence: this.assessConfidence(tradeSize, liquidity)
    };
  }

  assessConfidence(tradeSize, liquidity) {
    const ratio = tradeSize / liquidity;

    if (ratio < 0.01) return 'HIGH';      // < 1% of liquidity
    if (ratio < 0.05) return 'MEDIUM';    // < 5% of liquidity
    if (ratio < 0.1) return 'LOW';        // < 10% of liquidity
    return 'VERY_LOW';                    // > 10% of liquidity
  }

  shouldRejectTrade(tradeSize, liquidity, maxImpact = 0.03) {
    const impact = this.calculatePriceImpact(tradeSize, liquidity, 0).priceImpact;
    return impact > maxImpact;
  }
}
```

## System-Level Risk Controls

### Circuit Breakers

#### Market Circuit Breakers

```javascript
class CircuitBreaker {
  constructor(
    priceThreshold = 0.1,     // 10% price change
    volumeThreshold = 5,       // 5x normal volume
    timeWindow = 300          // 5 minutes
  ) {
    this.priceThreshold = priceThreshold;
    this.volumeThreshold = volumeThreshold;
    this.timeWindow = timeWindow;
    this.breakerTriggered = false;
    this.lastTriggerTime = 0;
    this.priceHistory = [];
    this.volumeHistory = [];
  }

  update(price, volume, timestamp) {
    // Maintain rolling window of data
    this.priceHistory.push({ price, timestamp });
    this.volumeHistory.push({ volume, timestamp });

    // Remove old data
    const cutoffTime = timestamp - this.timeWindow;
    this.priceHistory = this.priceHistory.filter(p => p.timestamp > cutoffTime);
    this.volumeHistory = this.volumeHistory.filter(v => v.timestamp > cutoffTime);

    // Check circuit breaker conditions
    const priceChange = this.calculatePriceChange();
    const volumeSpike = this.calculateVolumeSpike();

    const shouldTrigger = (
      Math.abs(priceChange) > this.priceThreshold ||
      volumeSpike > this.volumeThreshold
    );

    if (shouldTrigger && !this.breakerTriggered) {
      this.triggerBreaker(timestamp);
    } else if (!shouldTrigger && this.breakerTriggered) {
      // Check if conditions normalized
      if (timestamp - this.lastTriggerTime > 3600) {  // 1 hour cooldown
        this.resetBreaker();
      }
    }

    return {
      breakerActive: this.breakerTriggered,
      priceChange,
      volumeSpike,
      triggerReason: this.getTriggerReason(priceChange, volumeSpike)
    };
  }

  calculatePriceChange() {
    if (this.priceHistory.length < 2) return 0;

    const latest = this.priceHistory[this.priceHistory.length - 1].price;
    const earliest = this.priceHistory[0].price;

    return (latest - earliest) / earliest;
  }

  calculateVolumeSpike() {
    if (this.volumeHistory.length < 2) return 1;

    const volumes = this.volumeHistory.map(v => v.volume);
    const avgVolume = volumes.reduce((a, b) => a + b) / volumes.length;
    const latestVolume = volumes[volumes.length - 1];

    return latestVolume / avgVolume;
  }

  triggerBreaker(timestamp) {
    this.breakerTriggered = true;
    this.lastTriggerTime = timestamp;
    // Trigger emergency actions
    this.emergencyStop();
  }

  resetBreaker() {
    this.breakerTriggered = false;
    this.lastTriggerTime = 0;
    // Resume normal operations
    this.resumeTrading();
  }

  getTriggerReason(priceChange, volumeSpike) {
    if (Math.abs(priceChange) > this.priceThreshold) {
      return `Price change: ${(priceChange * 100).toFixed(2)}%`;
    }
    if (volumeSpike > this.volumeThreshold) {
      return `Volume spike: ${volumeSpike.toFixed(1)}x normal`;
    }
    return 'Normal conditions';
  }

  emergencyStop() {
    // Implementation for emergency stop
    console.log('Circuit breaker triggered - Emergency stop activated');
  }

  resumeTrading() {
    // Implementation for resuming trading
    console.log('Circuit breaker reset - Trading resumed');
  }
}
```

### Rate Limiting and Throttling

```javascript
class RateLimiter {
  constructor(
    maxRequestsPerMinute = 60,
    maxRequestsPerHour = 1000,
    burstLimit = 10
  ) {
    this.maxPerMinute = maxRequestsPerMinute;
    this.maxPerHour = maxRequestsPerHour;
    this.burstLimit = burstLimit;
    this.requests = [];
  }

  checkLimit(identifier, currentTime = Date.now()) {
    // Clean old requests
    this.requests = this.requests.filter(req =>
      currentTime - req.timestamp < 3600000  // Keep last hour
    );

    // Count requests for this identifier
    const userRequests = this.requests.filter(req => req.identifier === identifier);
    const recentRequests = userRequests.filter(req =>
      currentTime - req.timestamp < 60000  // Last minute
    );

    const minuteCount = recentRequests.length;
    const hourCount = userRequests.length;

    const withinLimits = minuteCount < this.maxPerMinute && hourCount < this.maxPerHour;
    const burstExceeded = minuteCount >= this.burstLimit;

    if (withinLimits && !burstExceeded) {
      // Allow request
      this.requests.push({
        identifier,
        timestamp: currentTime
      });

      return {
        allowed: true,
        remainingMinute: Math.max(0, this.maxPerMinute - minuteCount - 1),
        remainingHour: Math.max(0, this.maxPerHour - hourCount - 1)
      };
    }

    // Deny request
    return {
      allowed: false,
      retryAfter: burstExceeded ? 60 : 3600,  // 1 min or 1 hour
      reason: burstExceeded ? 'burst_limit_exceeded' : 'rate_limit_exceeded'
    };
  }
}
```

## Risk Monitoring and Reporting

### Real-Time Risk Dashboard

```javascript
class RiskDashboard {
  constructor() {
    this.riskMetrics = {};
    this.alerts = [];
    this.thresholds = {
      maxDrawdown: 0.1,
      maxVar: 0.05,
      minSharpeRatio: 0.5
    };
  }

  updateMetrics(metrics) {
    this.riskMetrics = { ...this.riskMetrics, ...metrics };
    this.checkThresholds();
  }

  checkThresholds() {
    const newAlerts = [];

    // Check drawdown
    if (this.riskMetrics.drawdown > this.thresholds.maxDrawdown) {
      newAlerts.push({
        type: 'DRAWDOWN_BREACH',
        severity: 'HIGH',
        message: `Drawdown ${this.riskMetrics.drawdown.toFixed(1)}% exceeds limit ${this.thresholds.maxDrawdown.toFixed(1)}%`,
        timestamp: Date.now()
      });
    }

    // Check VaR
    if (this.riskMetrics.var > this.thresholds.maxVar) {
      newAlerts.push({
        type: 'VAR_BREACH',
        severity: 'MEDIUM',
        message: `VaR ${this.riskMetrics.var.toFixed(1)}% exceeds limit ${this.thresholds.maxVar.toFixed(1)}%`,
        timestamp: Date.now()
      });
    }

    // Check Sharpe ratio
    if (this.riskMetrics.sharpeRatio < this.thresholds.minSharpeRatio) {
      newAlerts.push({
        type: 'SHARPE_RATIO_LOW',
        severity: 'LOW',
        message: `Sharpe ratio ${this.riskMetrics.sharpeRatio.toFixed(2)} below minimum ${this.thresholds.minSharpeRatio}`,
        timestamp: Date.now()
      });
    }

    this.alerts.push(...newAlerts);
    return newAlerts;
  }

  generateReport() {
    return {
      timestamp: Date.now(),
      riskMetrics: this.riskMetrics,
      activeAlerts: this.alerts.filter(alert => alert.active !== false),
      recommendations: this.generateRecommendations(),
      compliance: this.checkCompliance()
    };
  }

  generateRecommendations() {
    const recommendations = [];

    if (this.riskMetrics.drawdown > 0.05) {
      recommendations.push('Consider reducing position sizes');
    }

    if (this.riskMetrics.correlation > 0.8) {
      recommendations.push('Diversify asset holdings');
    }

    if (this.riskMetrics.sharpeRatio < 1) {
      recommendations.push('Review strategy parameters');
    }

    return recommendations;
  }

  checkCompliance() {
    // Check regulatory compliance
    const compliance = {
      positionLimits: this.riskMetrics.totalExposure < 1.0,
      riskLimits: this.riskMetrics.var < 0.1,
      diversification: this.checkDiversification(),
      reporting: this.checkReportingRequirements()
    };

    return {
      compliant: Object.values(compliance).every(Boolean),
      details: compliance
    };
  }
}
```

---

Risk controls are essential for sustainable automated trading. This comprehensive framework provides multiple layers of protection while allowing strategies to operate efficiently within defined risk parameters.

## Next Steps
- [Vault Security →](vault-security.md) - Security implementation
- [Advanced Trading Patterns →](../for-traders/advanced-trading-patterns.md) - Risk-aware strategies
- [Monitoring & Analytics →](../advanced-features/monitoring-analytics.md) - Risk monitoring tools

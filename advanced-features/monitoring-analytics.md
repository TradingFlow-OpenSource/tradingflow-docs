# Monitoring & Analytics

Comprehensive monitoring and analytics tools for TradingFlow strategies and infrastructure.

## Real-Time Monitoring

### System Health Monitoring

#### Key Metrics Dashboard

```javascript
class SystemHealthMonitor {
  constructor() {
    this.metrics = {
      cpu: 0,
      memory: 0,
      disk: 0,
      network: 0,
      responseTime: 0,
      errorRate: 0,
      throughput: 0
    };
    this.alerts = [];
  }

  async collectMetrics() {
    // CPU usage
    this.metrics.cpu = await this.getCpuUsage();

    // Memory usage
    this.metrics.memory = await this.getMemoryUsage();

    // Disk usage
    this.metrics.disk = await this.getDiskUsage();

    // Network I/O
    this.metrics.network = await this.getNetworkIO();

    // Application metrics
    this.metrics.responseTime = await this.getAvgResponseTime();
    this.metrics.errorRate = await this.getErrorRate();
    this.metrics.throughput = await this.getThroughput();

    this.checkThresholds();
  }

  checkThresholds() {
    const thresholds = {
      cpu: 80,      // 80% CPU usage
      memory: 85,   // 85% memory usage
      disk: 90,     // 90% disk usage
      responseTime: 1000,  // 1 second response time
      errorRate: 5  // 5% error rate
    };

    Object.keys(thresholds).forEach(metric => {
      if (this.metrics[metric] > thresholds[metric]) {
        this.createAlert(metric, this.metrics[metric], thresholds[metric]);
      }
    });
  }

  createAlert(metric, currentValue, threshold) {
    const alert = {
      type: 'threshold_breach',
      metric: metric,
      currentValue: currentValue,
      threshold: threshold,
      severity: this.calculateSeverity(currentValue, threshold),
      timestamp: new Date().toISOString()
    };

    this.alerts.push(alert);
    this.sendAlert(alert);
  }

  calculateSeverity(current, threshold) {
    const ratio = current / threshold;
    if (ratio > 2) return 'critical';
    if (ratio > 1.5) return 'high';
    if (ratio > 1.2) return 'medium';
    return 'low';
  }

  async sendAlert(alert) {
    // Send to notification channels
    console.log(`ALERT: ${alert.severity.toUpperCase()} - ${alert.metric} at ${alert.currentValue}`);
  }
}
```

### Strategy Performance Monitoring

#### Live Performance Tracking

```javascript
class StrategyPerformanceMonitor {
  constructor(strategyId) {
    this.strategyId = strategyId;
    this.metrics = {
      totalReturn: 0,
      dailyReturn: 0,
      volatility: 0,
      sharpeRatio: 0,
      maxDrawdown: 0,
      winRate: 0,
      profitFactor: 0,
      activeTrades: 0,
      totalTrades: 0
    };
  }

  updateMetrics(tradeData, portfolioData) {
    this.metrics.totalReturn = this.calculateTotalReturn(portfolioData);
    this.metrics.dailyReturn = this.calculateDailyReturn(portfolioData);
    this.metrics.volatility = this.calculateVolatility(portfolioData);
    this.metrics.sharpeRatio = this.calculateSharpeRatio(portfolioData);
    this.metrics.maxDrawdown = this.calculateMaxDrawdown(portfolioData);
    this.metrics.winRate = this.calculateWinRate(tradeData);
    this.metrics.profitFactor = this.calculateProfitFactor(tradeData);
    this.metrics.activeTrades = tradeData.filter(t => !t.closed).length;
    this.metrics.totalTrades = tradeData.length;
  }

  calculateTotalReturn(portfolio) {
    const initialValue = portfolio.initialValue;
    const currentValue = portfolio.currentValue;
    return (currentValue - initialValue) / initialValue;
  }

  calculateWinRate(trades) {
    const winningTrades = trades.filter(t => t.pnl > 0).length;
    return winningTrades / trades.length;
  }

  checkRiskLimits() {
    const limits = {
      maxDrawdown: 0.1,      // 10% max drawdown
      maxDailyLoss: 0.05,    // 5% max daily loss
      minSharpeRatio: 0.5    // Minimum Sharpe ratio
    };

    const breaches = [];

    if (this.metrics.maxDrawdown > limits.maxDrawdown) {
      breaches.push({
        type: 'max_drawdown',
        current: this.metrics.maxDrawdown,
        limit: limits.maxDrawdown
      });
    }

    if (Math.abs(this.metrics.dailyReturn) > limits.maxDailyLoss) {
      breaches.push({
        type: 'daily_loss',
        current: Math.abs(this.metrics.dailyReturn),
        limit: limits.maxDailyLoss
      });
    }

    if (this.metrics.sharpeRatio < limits.minSharpeRatio) {
      breaches.push({
        type: 'sharpe_ratio',
        current: this.metrics.sharpeRatio,
        limit: limits.minSharpeRatio
      });
    }

    return breaches;
  }
}
```

## Analytics Dashboard

### Performance Analytics

#### Advanced Metrics Calculation

```javascript
class AdvancedAnalytics {
  constructor() {
    this.metrics = {};
  }

  calculateAdvancedMetrics(portfolioData, benchmarkData) {
    return {
      alpha: this.calculateAlpha(portfolioData, benchmarkData),
      beta: this.calculateBeta(portfolioData, benchmarkData),
      informationRatio: this.calculateInformationRatio(portfolioData, benchmarkData),
      sortinoRatio: this.calculateSortinoRatio(portfolioData),
      calmarRatio: this.calculateCalmarRatio(portfolioData),
      omegaRatio: this.calculateOmegaRatio(portfolioData),
      valueAtRisk: this.calculateVaR(portfolioData),
      expectedShortfall: this.calculateExpectedShortfall(portfolioData),
      maximumDrawdown: this.calculateMaxDrawdown(portfolioData),
      recoveryTime: this.calculateRecoveryTime(portfolioData),
      profitToMaxDrawdown: this.calculateProfitToMaxDrawdown(portfolioData)
    };
  }

  calculateAlpha(portfolioReturns, benchmarkReturns) {
    // Jensen's Alpha: Alpha = Rp - [Rf + β(Rm - Rf)]
    const beta = this.calculateBeta(portfolioReturns, benchmarkReturns);
    const riskFreeRate = 0.02; // 2% annual risk-free rate
    const avgBenchmarkReturn = benchmarkReturns.reduce((a, b) => a + b) / benchmarkReturns.length;
    const avgPortfolioReturn = portfolioReturns.reduce((a, b) => a + b) / portfolioReturns.length;

    return avgPortfolioReturn - (riskFreeRate + beta * (avgBenchmarkReturn - riskFreeRate));
  }

  calculateBeta(portfolioReturns, benchmarkReturns) {
    // Covariance of portfolio and benchmark divided by variance of benchmark
    const covariance = this.covariance(portfolioReturns, benchmarkReturns);
    const benchmarkVariance = this.variance(benchmarkReturns);
    return covariance / benchmarkVariance;
  }

  calculateSortinoRatio(returns) {
    // Similar to Sharpe but only considers downside volatility
    const avgReturn = returns.reduce((a, b) => a + b) / returns.length;
    const downsideReturns = returns.filter(r => r < 0);
    const downsideDeviation = Math.sqrt(downsideReturns.reduce((sum, r) => sum + r * r, 0) / downsideReturns.length);

    return avgReturn / downsideDeviation;
  }

  calculateVaR(returns, confidence = 0.95) {
    // Historical Value at Risk
    const sortedReturns = [...returns].sort((a, b) => a - b);
    const index = Math.floor((1 - confidence) * sortedReturns.length);
    return -sortedReturns[index]; // Positive VaR
  }

  // Utility functions
  covariance(array1, array2) {
    const mean1 = array1.reduce((a, b) => a + b) / array1.length;
    const mean2 = array2.reduce((a, b) => a + b) / array2.length;

    return array1.reduce((sum, val, i) =>
      sum + (val - mean1) * (array2[i] - mean2), 0) / array1.length;
  }

  variance(array) {
    const mean = array.reduce((a, b) => a + b) / array.length;
    return array.reduce((sum, val) => sum + Math.pow(val - mean, 2), 0) / array.length;
  }
}
```

### Risk Analytics

#### Portfolio Risk Decomposition

```javascript
class RiskAnalytics {
  constructor() {
    this.riskMetrics = {};
  }

  decomposePortfolioRisk(weights, covarianceMatrix) {
    const portfolioVariance = this.calculatePortfolioVariance(weights, covarianceMatrix);
    const individualRisks = this.calculateIndividualRiskContributions(weights, covarianceMatrix);

    return {
      totalRisk: Math.sqrt(portfolioVariance),
      systematicRisk: this.calculateSystematicRisk(weights, covarianceMatrix),
      unsystematicRisk: this.calculateUnsystematicRisk(weights, covarianceMatrix),
      riskContributions: individualRisks,
      riskConcentration: this.calculateRiskConcentration(individualRisks)
    };
  }

  calculatePortfolioVariance(weights, covarianceMatrix) {
    // w^T * Σ * w
    const n = weights.length;
    let variance = 0;

    for (let i = 0; i < n; i++) {
      for (let j = 0; j < n; j++) {
        variance += weights[i] * weights[j] * covarianceMatrix[i][j];
      }
    }

    return variance;
  }

  calculateIndividualRiskContributions(weights, covarianceMatrix) {
    const contributions = [];
    const portfolioVariance = this.calculatePortfolioVariance(weights, covarianceMatrix);

    for (let i = 0; i < weights.length; i++) {
      // Marginal contribution to risk
      let marginalContribution = 0;
      for (let j = 0; j < weights.length; j++) {
        marginalContribution += weights[j] * covarianceMatrix[i][j];
      }

      // Total contribution
      const contribution = weights[i] * marginalContribution;
      contributions.push({
        asset: i,
        contribution: contribution,
        percentage: contribution / portfolioVariance
      });
    }

    return contributions;
  }

  calculateRiskConcentration(riskContributions) {
    const percentages = riskContributions.map(c => c.percentage);
    const maxContribution = Math.max(...percentages);
    const hhi = percentages.reduce((sum, p) => sum + p * p, 0); // Herfindahl-Hirschman Index

    return {
      maxRiskContribution: maxContribution,
      riskConcentrationIndex: hhi,
      diversificationRatio: 1 / hhi
    };
  }

  calculateValueAtRisk(returns, confidence = 0.95, timeHorizon = 1) {
    // Parametric VaR assuming normal distribution
    const mean = returns.reduce((a, b) => a + b) / returns.length;
    const variance = returns.reduce((sum, r) => sum + Math.pow(r - mean, 2), 0) / returns.length;
    const volatility = Math.sqrt(variance);

    // Z-score for confidence level
    const zScore = confidence === 0.95 ? 1.645 :
                  confidence === 0.99 ? 2.326 : 1.96;

    const var95 = -(mean * timeHorizon + zScore * volatility * Math.sqrt(timeHorizon));

    return {
      valueAtRisk: var95,
      expectedShortfall: this.calculateExpectedShortfall(returns, confidence, timeHorizon),
      confidence: confidence,
      timeHorizon: timeHorizon
    };
  }

  calculateExpectedShortfall(returns, confidence = 0.95, timeHorizon = 1) {
    // Expected Shortfall (CVaR) - average loss beyond VaR
    const sortedReturns = [...returns].sort((a, b) => a - b);
    const varIndex = Math.floor((1 - confidence) * sortedReturns.length);
    const tailLosses = sortedReturns.slice(0, varIndex + 1);

    return -tailLosses.reduce((a, b) => a + b) / tailLosses.length * timeHorizon;
  }
}
```

## Alerting System

### Multi-Channel Alerting

```javascript
class AlertingSystem {
  constructor() {
    this.channels = {
      email: new EmailChannel(),
      sms: new SMSChannel(),
      slack: new SlackChannel(),
      webhook: new WebhookChannel()
    };
    this.alertHistory = [];
  }

  async sendAlert(alert, channels = ['email']) {
    const alertRecord = {
      id: this.generateAlertId(),
      alert: alert,
      channels: channels,
      timestamp: new Date().toISOString(),
      status: 'sending'
    };

    this.alertHistory.push(alertRecord);

    try {
      const promises = channels.map(channel =>
        this.channels[channel].send(alert)
      );

      await Promise.allSettled(promises);
      alertRecord.status = 'sent';
    } catch (error) {
      alertRecord.status = 'failed';
      alertRecord.error = error.message;
    }

    return alertRecord;
  }

  createPerformanceAlert(strategyId, metrics, thresholds) {
    const alerts = [];

    if (metrics.drawdown > thresholds.maxDrawdown) {
      alerts.push({
        type: 'risk_alert',
        severity: 'high',
        title: 'Maximum Drawdown Exceeded',
        message: `Strategy ${strategyId} drawdown: ${(metrics.drawdown * 100).toFixed(2)}%`,
        data: {
          strategyId,
          currentDrawdown: metrics.drawdown,
          threshold: thresholds.maxDrawdown
        }
      });
    }

    if (metrics.dailyLoss > thresholds.maxDailyLoss) {
      alerts.push({
        type: 'performance_alert',
        severity: 'medium',
        title: 'Daily Loss Threshold Breached',
        message: `Strategy ${strategyId} daily loss: ${(metrics.dailyLoss * 100).toFixed(2)}%`,
        data: {
          strategyId,
          dailyLoss: metrics.dailyLoss,
          threshold: thresholds.maxDailyLoss
        }
      });
    }

    return alerts;
  }

  generateAlertId() {
    return `alert_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  getAlertHistory(hours = 24) {
    const cutoffTime = Date.now() - (hours * 60 * 60 * 1000);
    return this.alertHistory.filter(alert =>
      new Date(alert.timestamp).getTime() > cutoffTime
    );
  }
}

// Channel implementations
class EmailChannel {
  async send(alert) {
    // Implementation for sending email alerts
    console.log(`Sending email alert: ${alert.title}`);
  }
}

class SlackChannel {
  async send(alert) {
    // Implementation for sending Slack alerts
    console.log(`Sending Slack alert: ${alert.title}`);
  }
}
```

## Reporting System

### Automated Report Generation

```javascript
class ReportGenerator {
  constructor() {
    this.templates = {
      daily: this.dailyReportTemplate,
      weekly: this.weeklyReportTemplate,
      monthly: this.monthlyReportTemplate
    };
  }

  async generateReport(reportType, data, format = 'pdf') {
    const template = this.templates[reportType];
    if (!template) {
      throw new Error(`Unknown report type: ${reportType}`);
    }

    const reportData = await template(data);

    if (format === 'pdf') {
      return await this.generatePDF(reportData);
    } else if (format === 'html') {
      return await this.generateHTML(reportData);
    } else {
      return reportData;
    }
  }

  dailyReportTemplate(data) {
    return {
      title: 'Daily Trading Report',
      date: new Date().toISOString().split('T')[0],
      sections: [
        {
          title: 'Portfolio Overview',
          data: {
            totalValue: data.portfolio.totalValue,
            dailyChange: data.portfolio.dailyChange,
            bestPerformer: data.portfolio.bestPerformer,
            worstPerformer: data.portfolio.worstPerformer
          }
        },
        {
          title: 'Strategy Performance',
          data: data.strategies.map(strategy => ({
            name: strategy.name,
            dailyReturn: strategy.dailyReturn,
            totalReturn: strategy.totalReturn,
            activeTrades: strategy.activeTrades
          }))
        },
        {
          title: 'Risk Metrics',
          data: {
            currentDrawdown: data.risk.currentDrawdown,
            valueAtRisk: data.risk.valueAtRisk,
            sharpeRatio: data.risk.sharpeRatio
          }
        }
      ]
    };
  }

  async generatePDF(reportData) {
    // Implementation for PDF generation
    // This would use a library like pdfkit or puppeteer
    console.log('Generating PDF report...');
    return Buffer.from('PDF content would go here');
  }

  async generateHTML(reportData) {
    // Implementation for HTML generation
    const html = `
      <html>
        <head><title>${reportData.title}</title></head>
        <body>
          <h1>${reportData.title}</h1>
          <p>Date: ${reportData.date}</p>
          ${reportData.sections.map(section => `
            <h2>${section.title}</h2>
            <pre>${JSON.stringify(section.data, null, 2)}</pre>
          `).join('')}
        </body>
      </html>
    `;
    return html;
  }
}
```

---

Monitoring and analytics provide critical insights into your trading system's performance and health. These tools help you make informed decisions and maintain optimal system operation.

## Next Steps
- [Performance Optimization →](performance-optimization.md) - Improve system performance
- [Integration Ecosystem →](integration-ecosystem.md) - Connect external services
- [Security & Risk Management →](../security-risk-management/) - Risk monitoring tools

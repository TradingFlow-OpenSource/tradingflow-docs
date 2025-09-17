# Developer Resources

Comprehensive technical resources for developers building on and extending the TradingFlow platform.

## API Documentation

### REST API Reference

#### Authentication

All API requests require authentication using JWT tokens or API keys.

**Headers:**
```http
Authorization: Bearer <jwt_token>
# OR
X-API-Key: <api_key>
Content-Type: application/json
```

#### Base URLs

- **Production**: `https://api.tradingflow.ai/v1`
- **Staging**: `https://api-staging.tradingflow.ai/v1`
- **Sandbox**: `https://api-sandbox.tradingflow.ai/v1`

#### Rate Limits

```javascript
const rateLimits = {
  public: {
    per_minute: 100,
    per_hour: 1000
  },
  authenticated: {
    per_minute: 1000,
    per_hour: 5000
  },
  trading: {
    per_minute: 60,
    per_hour: 500
  }
};
```

### WebSocket API

#### Connection Setup

```javascript
import WebSocket from 'ws';

class TradingFlowWS {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.ws = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
  }

  connect() {
    this.ws = new WebSocket('wss://ws.tradingflow.ai/v1');

    this.ws.onopen = () => {
      console.log('Connected to TradingFlow WebSocket');

      // Authenticate
      this.ws.send(JSON.stringify({
        type: 'auth',
        token: this.apiKey
      }));

      this.reconnectAttempts = 0;
    };

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };

    this.ws.onclose = () => {
      console.log('WebSocket connection closed');
      this.handleReconnect();
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }

  handleMessage(data) {
    switch (data.type) {
      case 'auth_success':
        console.log('Authentication successful');
        this.subscribeToFeeds();
        break;

      case 'vault_update':
        this.handleVaultUpdate(data.payload);
        break;

      case 'trade_executed':
        this.handleTradeExecuted(data.payload);
        break;

      case 'error':
        this.handleError(data.payload);
        break;

      default:
        console.log('Unknown message type:', data.type);
    }
  }

  subscribeToFeeds() {
    // Subscribe to vault updates
    this.ws.send(JSON.stringify({
      type: 'subscribe',
      channel: 'vault_updates',
      vault_id: 'your_vault_id'
    }));

    // Subscribe to strategy status
    this.ws.send(JSON.stringify({
      type: 'subscribe',
      channel: 'strategy_status',
      strategy_id: 'your_strategy_id'
    }));
  }

  handleReconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000);

      setTimeout(() => {
        console.log(`Reconnecting... Attempt ${this.reconnectAttempts}`);
        this.connect();
      }, delay);
    } else {
      console.error('Max reconnection attempts reached');
    }
  }
}
```

#### Real-time Data Streams

```javascript
// Price feed subscription
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'price_feed',
  symbol: 'BTC/USDT',
  exchange: 'binance'
}));

// Order book updates
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'order_book',
  symbol: 'ETH/USDT',
  depth: 20
}));

// Trade execution notifications
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'trade_notifications',
  vault_id: 'your_vault_id'
}));
```

## SDK Libraries

### JavaScript/TypeScript SDK

#### Installation

```bash
npm install @tradingflow/sdk
# OR
yarn add @tradingflow/sdk
```

#### Basic Usage

```typescript
import { TradingFlowClient } from '@tradingflow/sdk';

const client = new TradingFlowClient({
  apiKey: 'your_api_key',
  environment: 'production',  // 'production', 'staging', 'sandbox'
  timeout: 30000
});

// Initialize
await client.initialize();

// Get account information
const account = await client.account.getInfo();
console.log('Account:', account);

// List vaults
const vaults = await client.vaults.list();
console.log('Vaults:', vaults);

// Create a new vault
const newVault = await client.vaults.create({
  name: 'My Trading Vault',
  network: 'aptos',
  vault_type: 'personal'
});

// Get vault balance
const balance = await client.vaults.getBalance(newVault.id);

// Deploy a strategy
const strategy = await client.strategies.deploy({
  vaultId: newVault.id,
  name: 'DCA Strategy',
  type: 'dca',
  config: {
    token: 'BTC',
    amount: 50,
    frequency: 'daily'
  }
});
```

#### Advanced Features

```typescript
// WebSocket integration
const wsClient = client.websocket();

// Subscribe to real-time updates
wsClient.subscribe('vault_updates', (update) => {
  console.log('Vault update:', update);
});

// Strategy monitoring
const strategyMonitor = client.strategies.monitor(strategy.id);

// Listen for strategy events
strategyMonitor.on('trade_executed', (trade) => {
  console.log('Trade executed:', trade);
});

strategyMonitor.on('error', (error) => {
  console.error('Strategy error:', error);
});

// Batch operations
const batchOps = client.batch();

// Queue multiple operations
batchOps.add(client.vaults.deposit(vaultId, 'USDC', 1000));
batchOps.add(client.strategies.update(strategyId, { amount: 75 }));
batchOps.add(client.vaults.withdraw(vaultId, 'BTC', 0.01));

// Execute all at once
const results = await batchOps.execute();
```

### Python SDK

#### Installation

```bash
pip install tradingflow-sdk
```

#### Basic Usage

```python
from tradingflow import TradingFlowClient

# Initialize client
client = TradingFlowClient(
    api_key='your_api_key',
    environment='production'
)

# Get account info
account = client.account.get_info()
print(f"Account: {account}")

# List vaults
vaults = client.vaults.list()
print(f"Vaults: {vaults}")

# Create vault
vault = client.vaults.create({
    'name': 'My Trading Vault',
    'network': 'aptos',
    'vault_type': 'personal'
})

# Deploy strategy
strategy = client.strategies.deploy({
    'vault_id': vault['id'],
    'name': 'DCA Strategy',
    'type': 'dca',
    'config': {
        'token': 'BTC',
        'amount': 50,
        'frequency': 'daily'
    }
})
```

#### Async Operations

```python
import asyncio
from tradingflow import AsyncTradingFlowClient

async def main():
    client = AsyncTradingFlowClient(
        api_key='your_api_key',
        environment='production'
    )

    # Get multiple balances concurrently
    vault_ids = ['vault1', 'vault2', 'vault3']
    balances = await asyncio.gather(*[
        client.vaults.get_balance(vault_id)
        for vault_id in vault_ids
    ])

    for i, balance in enumerate(balances):
        print(f"Vault {vault_ids[i]} balance: {balance}")

asyncio.run(main())
```

### Go SDK

#### Installation

```bash
go get github.com/tradingflow/go-sdk
```

#### Basic Usage

```go
package main

import (
    "fmt"
    "log"

    tf "github.com/tradingflow/go-sdk"
)

func main() {
    // Initialize client
    client, err := tf.NewClient(&tf.Config{
        APIKey:      "your_api_key",
        Environment: tf.Production,
        Timeout:     30,
    })
    if err != nil {
        log.Fatal(err)
    }

    // Get account info
    account, err := client.Account.GetInfo()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Account: %+v\n", account)

    // Create vault
    vault, err := client.Vaults.Create(&tf.VaultConfig{
        Name:      "My Trading Vault",
        Network:   "aptos",
        VaultType: "personal",
    })
    if err != nil {
        log.Fatal(err)
    }

    // Deploy strategy
    strategy, err := client.Strategies.Deploy(&tf.StrategyConfig{
        VaultID: vault.ID,
        Name:    "DCA Strategy",
        Type:    "dca",
        Config: map[string]interface{}{
            "token":     "BTC",
            "amount":    50,
            "frequency": "daily",
        },
    })
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Strategy deployed: %s\n", strategy.ID)
}
```

## Node Development Kit

### Custom Node Template

```python
from typing import Dict, Any, List, Optional
from tradingflow.station.common.node_decorators import register_node_type
from tradingflow.station.common.signal_types import SignalType
from tradingflow.station.nodes.node_base import NodeBase, NodeStatus

@register_node_type(
    "custom_technical_indicator",
    default_params={
        "period": 14,
        "multiplier": 2.0,
        "source": "close"
    }
)
class CustomTechnicalIndicatorNode(NodeBase):
    """
    Custom Technical Indicator Node

    Calculates a specialized technical indicator based on price data.

    Input Signals:
    - price_data: OHLCV price data dictionary

    Output Signals:
    - indicator_value: Calculated indicator value (float)
    - signal: Trading signal based on indicator (string)
    """

    def _register_input_handles(self) -> List[Dict[str, Any]]:
        return [
            {
                "name": "price_data",
                "data_type": dict,
                "description": "OHLCV price data",
                "required": True,
                "example": {
                    "timestamp": 1640995200,
                    "open": 100.0,
                    "high": 105.0,
                    "low": 95.0,
                    "close": 102.0,
                    "volume": 1000000
                }
            }
        ]

    def _register_output_handles(self) -> List[Dict[str, Any]]:
        return [
            {
                "name": "indicator_value",
                "data_type": float,
                "description": "Calculated indicator value"
            },
            {
                "name": "signal",
                "data_type": str,
                "description": "Trading signal (BUY/SELL/HOLD)"
            }
        ]

    async def execute(self) -> NodeStatus:
        try:
            # Get input data
            price_data = await self.get_input_data("price_data")

            if not price_data:
                self.persist_log("No price data received", level="WARNING")
                return NodeStatus.SKIPPED

            # Calculate indicator
            indicator_value = self._calculate_indicator(price_data)

            # Generate signal
            signal = self._generate_signal(indicator_value, price_data)

            # Send output signals
            await self.send_signal(
                "indicator_value",
                SignalType.DATA,
                indicator_value
            )

            await self.send_signal(
                "signal",
                SignalType.SIGNAL,
                signal
            )

            self.persist_log(
                f"Indicator calculated: {indicator_value:.4f}, Signal: {signal}",
                level="INFO"
            )

            return NodeStatus.COMPLETED

        except Exception as e:
            error_msg = f"Error in custom indicator execution: {str(e)}"
            self.persist_log(error_msg, level="ERROR")
            return NodeStatus.FAILED

    def _calculate_indicator(self, price_data: Dict[str, Any]) -> float:
        """
        Implement your custom indicator calculation logic here.

        Args:
            price_data: Dictionary containing OHLCV data

        Returns:
            Calculated indicator value
        """
        # Example: Simple moving average with multiplier
        period = self.config.get("period", 14)
        multiplier = self.config.get("multiplier", 2.0)
        source = self.config.get("source", "close")

        # This is a simplified example - replace with your actual calculation
        price_value = price_data.get(source, price_data.get("close", 0))

        # Placeholder calculation
        indicator = price_value * multiplier / period

        return indicator

    def _generate_signal(self, indicator_value: float, price_data: Dict[str, Any]) -> str:
        """
        Generate trading signal based on indicator value.

        Args:
            indicator_value: Calculated indicator value
            price_data: Current price data

        Returns:
            Trading signal: "BUY", "SELL", or "HOLD"
        """
        current_price = price_data.get("close", 0)

        # Example signal logic - customize based on your indicator
        if indicator_value > current_price * 1.02:  # 2% above price
            return "BUY"
        elif indicator_value < current_price * 0.98:  # 2% below price
            return "SELL"
        else:
            return "HOLD"
```

### Node Testing Framework

```python
import unittest
from unittest.mock import AsyncMock, MagicMock
from your_custom_node import CustomTechnicalIndicatorNode

class TestCustomTechnicalIndicatorNode(unittest.TestCase):
    def setUp(self):
        self.node = CustomTechnicalIndicatorNode(
            flow_id="test_flow",
            component_id=1,
            cycle=1,
            node_id="test_node",
            name="Test Custom Indicator"
        )

    def test_indicator_calculation(self):
        """Test indicator calculation logic"""
        test_data = {
            "timestamp": 1640995200,
            "open": 100.0,
            "high": 105.0,
            "low": 95.0,
            "close": 102.0,
            "volume": 1000000
        }

        # Test with default config
        result = self.node._calculate_indicator(test_data)
        expected = 102.0 * 2.0 / 14  # close * multiplier / period
        self.assertAlmostEqual(result, expected, places=4)

    def test_signal_generation_buy(self):
        """Test buy signal generation"""
        indicator_value = 105.0  # Above price + 2%
        price_data = {"close": 100.0}

        signal = self.node._generate_signal(indicator_value, price_data)
        self.assertEqual(signal, "BUY")

    def test_signal_generation_sell(self):
        """Test sell signal generation"""
        indicator_value = 95.0   # Below price - 2%
        price_data = {"close": 100.0}

        signal = self.node._generate_signal(indicator_value, price_data)
        self.assertEqual(signal, "SELL")

    def test_signal_generation_hold(self):
        """Test hold signal generation"""
        indicator_value = 101.0  # Within 2% range
        price_data = {"close": 100.0}

        signal = self.node._generate_signal(indicator_value, price_data)
        self.assertEqual(signal, "HOLD")

    async def test_execute_success(self):
        """Test successful execution"""
        # Mock input data
        test_data = {
            "close": 100.0,
            "volume": 1000000
        }

        # Mock the get_input_data method
        self.node.get_input_data = AsyncMock(return_value=test_data)
        self.node.send_signal = AsyncMock()

        # Execute node
        result = await self.node.execute()

        # Verify execution
        self.assertEqual(result, NodeStatus.COMPLETED)
        self.assertEqual(self.node.send_signal.call_count, 2)  # indicator_value and signal

    async def test_execute_no_data(self):
        """Test execution with no input data"""
        # Mock empty input data
        self.node.get_input_data = AsyncMock(return_value=None)

        # Execute node
        result = await self.node.execute()

        # Verify execution was skipped
        self.assertEqual(result, NodeStatus.SKIPPED)

if __name__ == '__main__':
    unittest.main()
```

### Node Publishing Process

1. **Development**
   - Implement your node following the template
   - Add comprehensive tests
   - Document inputs, outputs, and configuration

2. **Testing**
   ```bash
   # Run unit tests
   python -m pytest tests/test_your_node.py -v

   # Run integration tests
   python -m pytest tests/integration/test_your_node_integration.py -v
   ```

3. **Documentation**
   - Create detailed README.md
   - Document all parameters and use cases
   - Provide examples and sample configurations

4. **Publishing**
   ```python
   # Submit to node registry
   from tradingflow.node_registry import NodeRegistry

   registry = NodeRegistry()
   registry.submit_node({
       'name': 'custom_technical_indicator',
       'version': '1.0.0',
       'author': 'your_name',
       'description': 'Custom technical indicator for advanced trading',
       'tags': ['technical', 'indicator', 'custom'],
       'repository': 'https://github.com/your-repo/custom-indicator',
       'license': 'MIT'
   })
   ```

## Integration Guides

### DEX Integration

#### Uniswap V3 Integration

```typescript
import { ethers } from 'ethers';
import { SwapRouter } from '@uniswap/v3-sdk';

class UniswapV3Integration {
  constructor(provider, signer, routerAddress) {
    this.provider = provider;
    this.signer = signer;
    this.router = new ethers.Contract(routerAddress, SwapRouter.ABI, signer);
  }

  async getQuote(tokenIn, tokenOut, amountIn, fee = 3000) {
    const quoterContract = new ethers.Contract(
      QUOTER_ADDRESS,
      ['function quoteExactInputSingle(address,address,uint24,uint256,uint160) view returns (uint256)'],
      this.provider
    );

    const quotedAmountOut = await quoterContract.quoteExactInputSingle(
      tokenIn.address,
      tokenOut.address,
      fee,
      amountIn
    );

    return quotedAmountOut;
  }

  async executeSwap(tokenIn, tokenOut, amountIn, amountOutMin, deadline) {
    const swapParams = {
      tokenIn: tokenIn.address,
      tokenOut: tokenOut.address,
      fee: 3000,
      recipient: await this.signer.getAddress(),
      deadline: deadline,
      amountIn: amountIn,
      amountOutMinimum: amountOutMin,
      sqrtPriceLimitX96: 0
    };

    const tx = await this.router.exactInputSingle(swapParams, {
      gasLimit: 200000
    });

    return await tx.wait();
  }
}
```

#### PancakeSwap Integration

```typescript
class PancakeSwapIntegration {
  constructor(web3, routerAddress, factoryAddress) {
    this.web3 = web3;
    this.router = new web3.eth.Contract(PANCAKE_ROUTER_ABI, routerAddress);
    this.factory = new web3.eth.Contract(PANCAKE_FACTORY_ABI, factoryAddress);
  }

  async getAmountsOut(amountIn, path) {
    return await this.router.methods.getAmountsOut(amountIn, path).call();
  }

  async swapExactTokensForTokens(amountIn, amountOutMin, path, to, deadline, gasPrice) {
    const tx = await this.router.methods.swapExactTokensForTokens(
      amountIn,
      amountOutMin,
      path,
      to,
      deadline
    );

    return await this.web3.eth.sendTransaction({
      ...tx,
      gasPrice: gasPrice || await this.web3.eth.getGasPrice()
    });
  }

  async addLiquidity(tokenA, tokenB, amountA, amountB, slippage = 0.005) {
    const amounts = await this.router.methods.addLiquidity(
      tokenA,
      tokenB,
      amountA,
      amountB,
      amountA * (1 - slippage),
      amountB * (1 - slippage),
      await this.web3.eth.getAccounts()[0],
      Math.floor(Date.now() / 1000) + 300
    );

    return await this.web3.eth.sendTransaction(amounts);
  }
}
```

### Oracle Integration

#### Chainlink Price Feeds

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

contract ChainlinkPriceOracle {
    mapping(address => AggregatorV3Interface) public priceFeeds;

    constructor() {
        // ETH/USD on Ethereum mainnet
        priceFeeds[0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE] =
            AggregatorV3Interface(0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419);

        // BTC/USD on Ethereum mainnet
        priceFeeds[0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599] =
            AggregatorV3Interface(0xF4030086522a5bEEa4988F8cA5B36dbC97BeE88c);
    }

    function getLatestPrice(address token) public view returns (int256) {
        require(address(priceFeeds[token]) != address(0), "Price feed not available");

        (, int256 price,,,) = priceFeeds[token].latestRoundData();
        return price;
    }

    function getPriceWithTimestamp(address token) public view returns (int256, uint256) {
        require(address(priceFeeds[token]) != address(0), "Price feed not available");

        (, int256 price,, uint256 updatedAt,) = priceFeeds[token].latestRoundData();
        return (price, updatedAt);
    }

    function addPriceFeed(address token, address feedAddress) external onlyOwner {
        priceFeeds[token] = AggregatorV3Interface(feedAddress);
    }
}
```

#### Pyth Network Integration

```typescript
import { PythConnection } from '@pythnetwork/client';

class PythPriceOracle {
  constructor(endpoint) {
    this.connection = new PythConnection(endpoint);
  }

  async getPrice(symbol) {
    await this.connection.connect();

    const priceData = await this.connection.getLatestPriceFeeds([symbol]);

    if (priceData.length === 0) {
      throw new Error(`Price data not available for ${symbol}`);
    }

    const price = priceData[0].price;
    const confidence = priceData[0].confidence;

    return {
      price: price.price,
      confidence: confidence,
      exponent: price.expo,
      publishTime: price.publishTime
    };
  }

  async getMultiplePrices(symbols) {
    await this.connection.connect();

    const priceData = await this.connection.getLatestPriceFeeds(symbols);

    return priceData.map(data => ({
      symbol: data.symbol,
      price: data.price.price,
      confidence: data.confidence,
      exponent: data.price.expo,
      publishTime: data.publishTime
    }));
  }
}
```

## Deployment Guides

### Docker Deployment

#### Dockerfile for Custom Nodes

```dockerfile
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    libffi-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for better caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd --create-home --shell /bin/bash app \
    && chown -R app:app /app
USER app

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Start application
CMD ["python", "main.py"]
```

#### Docker Compose for Development

```yaml
version: '3.8'

services:
  tradingflow-node:
    build: .
    ports:
      - "8000:8000"
    environment:
      - TRADINGFLOW_ENV=development
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=postgresql://user:password@db:5432/tradingflow
    depends_on:
      - redis
      - db
    volumes:
      - .:/app
      - /app/__pycache__
    command: python main.py

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: tradingflow
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  redis_data:
  postgres_data:
```

### Kubernetes Deployment

#### Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tradingflow-node
  labels:
    app: tradingflow-node
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tradingflow-node
  template:
    metadata:
      labels:
        app: tradingflow-node
    spec:
      containers:
      - name: tradingflow-node
        image: your-registry/tradingflow-node:latest
        ports:
        - containerPort: 8000
        env:
        - name: TRADINGFLOW_ENV
          value: "production"
        - name: REDIS_URL
          value: "redis://redis-service:6379"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: database_url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

#### Service Manifest

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tradingflow-node-service
spec:
  selector:
    app: tradingflow-node
  ports:
  - port: 80
    targetPort: 8000
    protocol: TCP
  type: LoadBalancer
```

#### Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: tradingflow-node-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: tradingflow-node
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## Performance Optimization

### Database Optimization

#### Indexing Strategy

```sql
-- Create indexes for common queries
CREATE INDEX idx_vault_user_id ON vaults(user_id);
CREATE INDEX idx_strategy_vault_id ON strategies(vault_id);
CREATE INDEX idx_trade_timestamp ON trades(timestamp DESC);
CREATE INDEX idx_trade_strategy_id ON trades(strategy_id);

-- Composite indexes for complex queries
CREATE INDEX idx_portfolio_snapshot ON portfolio_snapshots(user_id, timestamp DESC);
CREATE INDEX idx_price_history ON price_history(symbol, timestamp DESC);

-- Partial indexes for active records
CREATE INDEX idx_active_strategies ON strategies(vault_id) WHERE status = 'active';
```

#### Query Optimization

```python
# Efficient pagination
def get_trades_paginated(vault_id, page=1, per_page=50):
    offset = (page - 1) * per_page

    query = """
    SELECT * FROM trades
    WHERE vault_id = $1
    ORDER BY timestamp DESC
    LIMIT $2 OFFSET $3
    """

    return db.execute(query, [vault_id, per_page, offset])

# Batch operations
def batch_update_balances(updates):
    with db.transaction():
        for update in updates:
            db.execute("""
                UPDATE vault_balances
                SET amount = amount + $1
                WHERE vault_id = $2 AND token = $3
            """, [update.amount, update.vault_id, update.token])
```

### Caching Strategies

#### Redis Caching Implementation

```python
import redis
import json
from typing import Optional, Any

class CacheManager:
    def __init__(self, redis_url: str = "redis://localhost:6379"):
        self.redis = redis.from_url(redis_url)

    def get(self, key: str) -> Optional[Any]:
        """Get value from cache"""
        data = self.redis.get(key)
        return json.loads(data) if data else None

    def set(self, key: str, value: Any, ttl: int = 300) -> bool:
        """Set value in cache with TTL"""
        return self.redis.setex(key, ttl, json.dumps(value))

    def delete(self, key: str) -> bool:
        """Delete value from cache"""
        return bool(self.redis.delete(key))

    def get_or_set(self, key: str, getter_func, ttl: int = 300):
        """Get from cache or set if not exists"""
        cached = self.get(key)
        if cached is not None:
            return cached

        value = getter_func()
        self.set(key, value, ttl)
        return value

# Usage example
cache = CacheManager()

# Cache price data for 5 minutes
btc_price = cache.get_or_set(
    "btc_price",
    lambda: get_binance_price("BTCUSDT"),
    ttl=300
)

# Cache vault balance for 1 minute
vault_balance = cache.get_or_set(
    f"vault_balance_{vault_id}",
    lambda: get_vault_balance(vault_id),
    ttl=60
)
```

#### Multi-Level Caching

```python
class MultiLevelCache:
    def __init__(self):
        self.l1_cache = {}  # In-memory cache
        self.l2_cache = CacheManager()  # Redis cache
        self.l1_ttl = 30  # 30 seconds

    def get(self, key: str):
        # Check L1 cache first
        if key in self.l1_cache:
            entry = self.l1_cache[key]
            if time.time() - entry['timestamp'] < self.l1_ttl:
                return entry['value']
            else:
                del self.l1_cache[key]

        # Check L2 cache
        value = self.l2_cache.get(key)
        if value is not None:
            # Populate L1 cache
            self.l1_cache[key] = {
                'value': value,
                'timestamp': time.time()
            }
            return value

        return None

    def set(self, key: str, value: Any, ttl: int = 300):
        # Set both caches
        self.l1_cache[key] = {
            'value': value,
            'timestamp': time.time()
        }
        self.l2_cache.set(key, value, ttl)
```

## Security Best Practices

### API Key Management

```python
import secrets
import hashlib
import hmac
from datetime import datetime, timedelta

class APIKeyManager:
    def generate_api_key(self, user_id: str) -> dict:
        """Generate a new API key pair"""
        api_key = secrets.token_hex(32)
        secret_key = secrets.token_hex(64)

        # Hash the secret key for storage
        hashed_secret = hashlib.sha256(secret_key.encode()).hexdigest()

        return {
            'api_key': api_key,
            'secret_key': secret_key,  # Return to user (only shown once)
            'hashed_secret': hashed_secret,  # Store this
            'user_id': user_id,
            'created_at': datetime.utcnow(),
            'expires_at': datetime.utcnow() + timedelta(days=365),
            'permissions': ['read', 'trade']
        }

    def validate_request(self, api_key: str, signature: str, timestamp: str, payload: str) -> bool:
        """Validate signed API request"""
        # Check timestamp (prevent replay attacks)
        request_time = datetime.fromtimestamp(int(timestamp))
        if abs((datetime.utcnow() - request_time).seconds) > 300:  # 5 minutes
            return False

        # Get stored hashed secret
        stored_hash = self.get_hashed_secret(api_key)
        if not stored_hash:
            return False

        # Create expected signature
        message = f"{timestamp}{payload}"
        expected_signature = hmac.new(
            stored_hash.encode(),
            message.encode(),
            hashlib.sha256
        ).hexdigest()

        return hmac.compare_digest(signature, expected_signature)

    def create_signature(self, secret_key: str, timestamp: str, payload: str) -> str:
        """Create request signature for client"""
        message = f"{timestamp}{payload}"
        return hmac.new(
            secret_key.encode(),
            message.encode(),
            hashlib.sha256
        ).hexdigest()
```

### Rate Limiting Implementation

```python
from collections import defaultdict, deque
import time

class AdvancedRateLimiter:
    def __init__(self):
        self.requests = defaultdict(lambda: deque(maxlen=1000))
        self.bans = {}

    def is_allowed(self, identifier: str, max_requests: int = 100, window: int = 60) -> bool:
        """Check if request is allowed"""
        now = time.time()

        # Check if user is banned
        if identifier in self.bans:
            if now < self.bans[identifier]:
                return False
            else:
                del self.bans[identifier]

        # Clean old requests
        requests = self.requests[identifier]
        while requests and now - requests[0] > window:
            requests.popleft()

        # Check rate limit
        if len(requests) >= max_requests:
            # Implement progressive ban
            ban_duration = min(len(requests) - max_requests + 1, 3600)  # Max 1 hour
            self.bans[identifier] = now + ban_duration
            return False

        # Add current request
        requests.append(now)
        return True

    def get_remaining_requests(self, identifier: str, max_requests: int = 100, window: int = 60) -> int:
        """Get remaining requests in current window"""
        now = time.time()
        requests = self.requests[identifier]

        # Clean old requests
        while requests and now - requests[0] > window:
            requests.popleft()

        return max(0, max_requests - len(requests))
```

---

These developer resources provide everything needed to build, integrate, and deploy on the TradingFlow platform. From basic API usage to advanced node development and system optimization, these guides ensure developers can effectively extend and integrate with TradingFlow.

## Next Steps
- [Tutorials & Guides →](tutorials-guides.md) - Learning resources
- [Community & Support →](community-support.md) - Get help and connect
- [Node Development →](../technical-documentation/node-development.md) - Custom node creation

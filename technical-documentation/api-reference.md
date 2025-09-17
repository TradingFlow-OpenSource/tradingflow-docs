# API Reference

Complete reference for TradingFlow's REST API and WebSocket connections. Use these APIs to integrate with TradingFlow programmatically.

## Authentication

All API requests require authentication using JWT tokens or API keys.

### Getting API Access

1. **Generate API Key**: Visit your dashboard settings
2. **Set Permissions**: Configure API key permissions
3. **Store Securely**: Never expose keys in client-side code

### Authentication Headers

```http
Authorization: Bearer <jwt_token>
# OR
X-API-Key: <api_key>
Content-Type: application/json
```

## Base URLs

- **Production**: `https://api.tradingflow.ai/v1`
- **Staging**: `https://api-staging.tradingflow.ai/v1`
- **WebSocket**: `wss://ws.tradingflow.ai/v1`

## Core API Endpoints

### Vault Management

#### Get Vault Information

```http
GET /vaults/{vault_address}
```

**Response:**
```json
{
  "address": "0x123...",
  "chain": "aptos",
  "name": "My Trading Vault",
  "total_value_usd": 15420.50,
  "assets": [
    {
      "symbol": "APT",
      "balance": "125.67",
      "value_usd": 1200.30,
      "percentage": 7.78
    }
  ],
  "last_updated": "2024-01-15T10:30:00Z"
}
```

#### Create New Vault

```http
POST /vaults
```

**Request Body:**
```json
{
  "name": "My New Vault",
  "chain": "aptos",
  "vault_type": "personal",
  "initial_deposit": {
    "token": "USDC",
    "amount": "1000.00"
  },
  "permissions": {
    "managers": ["0xabc123..."],
    "traders": ["0xdef456..."]
  }
}
```

### Trading Operations

#### Execute Swap

```http
POST /vaults/{vault_address}/swap
```

**Request Body:**
```json
{
  "from_token": "USDC",
  "to_token": "APT",
  "amount_in": "100.00",
  "slippage_tolerance": 1.0,
  "deadline": "2024-01-15T10:35:00Z"
}
```

**Response:**
```json
{
  "transaction_hash": "0xabc123...",
  "status": "pending",
  "estimated_output": "13.25",
  "gas_fee": "0.001",
  "execution_time": "2024-01-15T10:30:00Z"
}
```

### Workflow Management

#### Get Workflows

```http
GET /workflows
```

#### Create Workflow

```http
POST /workflows
```

**Request Body:**
```json
{
  "name": "DCA Strategy",
  "description": "Daily BTC accumulation",
  "nodes": [
    {
      "id": "price_feed_1",
      "type": "binance_price_node",
      "config": {
        "symbol": "BTCUSDT",
        "interval": "1d"
      }
    },
    {
      "id": "buy_condition_1",
      "type": "condition_node",
      "config": {
        "condition": "price < moving_average_30d"
      }
    }
  ],
  "edges": [
    {
      "from": "price_feed_1",
      "to": "buy_condition_1",
      "from_handle": "price_data",
      "to_handle": "price_input"
    }
  ]
}
```

#### Execute Workflow

```http
POST /workflows/{workflow_id}/execute
```

## WebSocket API

### Connection

```javascript
const ws = new WebSocket('wss://ws.tradingflow.ai/v1');

ws.onopen = function() {
  // Send authentication
  ws.send(JSON.stringify({
    type: 'auth',
    token: 'your_jwt_token'
  }));
};
```

### Subscribe to Events

```javascript
// Subscribe to vault updates
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'vault_updates',
  vault_address: '0x123...'
}));

// Subscribe to workflow status
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'workflow_status',
  workflow_id: 'workflow_123'
}));
```

### Event Types

#### Vault Balance Update

```json
{
  "type": "vault_balance_update",
  "vault_address": "0x123...",
  "data": {
    "total_value_usd": 15500.75,
    "change_24h": 80.25,
    "updated_at": "2024-01-15T10:30:00Z"
  }
}
```

#### Trade Execution

```json
{
  "type": "trade_executed",
  "vault_address": "0x123...",
  "data": {
    "transaction_hash": "0xabc123...",
    "from_token": "USDC",
    "to_token": "APT",
    "amount_in": "100.00",
    "amount_out": "13.25",
    "status": "completed"
  }
}
```

## SDK Integration

### JavaScript SDK

```bash
npm install @tradingflow/sdk
```

```javascript
import { TradingFlowSDK } from '@tradingflow/sdk';

const client = new TradingFlowSDK({
  apiKey: 'your_api_key',
  environment: 'production'
});

// Get vault info
const vault = await client.vaults.get('0x123...');

// Execute trade
const trade = await client.trading.swap({
  vaultAddress: '0x123...',
  fromToken: 'USDC',
  toToken: 'APT',
  amount: '100.00'
});
```

### Python SDK

```bash
pip install tradingflow-sdk
```

```python
from tradingflow import TradingFlowClient

client = TradingFlowClient(
    api_key='your_api_key',
    environment='production'
)

# Get vault information
vault = client.vaults.get('0x123...')

# Execute swap
trade = client.trading.swap(
    vault_address='0x123...',
    from_token='USDC',
    to_token='APT',
    amount='100.00'
)
```

## Rate Limits

- **Public Endpoints**: 100 requests per minute
- **Authenticated Endpoints**: 1000 requests per minute
- **Trading Endpoints**: 60 requests per minute
- **WebSocket Connections**: 10 concurrent connections per user

## Error Handling

### HTTP Status Codes

- `200` - Success
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `429` - Rate Limited
- `500` - Internal Server Error

### Error Response Format

```json
{
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Insufficient token balance for swap",
    "details": {
      "required": "100.00",
      "available": "50.00",
      "token": "USDC"
    }
  }
}
```

## Webhooks

Register webhook URLs to receive notifications about vault and trading events.

### Webhook Configuration

```http
POST /webhooks
```

**Request:**
```json
{
  "url": "https://your-app.com/webhook",
  "events": ["trade_executed", "vault_balance_update"],
  "vault_addresses": ["0x123..."],
  "secret": "webhook_secret_key"
}
```

### Webhook Payload

```json
{
  "event": "trade_executed",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "vault_address": "0x123...",
    "transaction_hash": "0xabc123...",
    "from_token": "USDC",
    "to_token": "APT",
    "amount_in": "100.00",
    "amount_out": "13.25"
  }
}
```

---

For additional API support, join our [Discord community](https://discord.gg/tradingflow) or contact support@tradingflow.ai.

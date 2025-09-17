# Action Nodes

Action nodes execute trades, manage assets, and communicate with external systems when decision conditions are met.

## Swap Node

**Purpose**: Execute token swaps on supported DEXs
**Node Type**: `swap_node`
**Category**: Action

#### Configuration Parameters
```javascript
{
  from_token: "USDC",
  to_token: "APT",
  chain: "aptos",                   // aptos, flow_evm, bsc
  amount_in_percentage: 25.0,       // Percentage of balance (alternative to fixed amount)
  amount_in_human_readable: 1000.0, // Fixed amount in tokens
  slippage_tolerance: 1.0,          // Slippage tolerance percentage
  gas_priority: "standard",         // low, standard, high, urgent
  deadline: 300                     // Transaction deadline in seconds
}
```

#### Input Handles
- `from_token` (string): Source token symbol
- `to_token` (string): Destination token symbol
- `amount_in_percentage` (number): Percentage of balance to swap
- `amount_in_human_readable` (number): Fixed amount to swap
- `vault_address` (string): Vault contract address

#### Output Handles
- `trade_receipt` (object): Complete transaction details
  ```javascript
  {
    transaction_hash: "0xabc123...",
    from_token: "USDC",
    to_token: "APT",
    amount_in: "1000.00",
    amount_out: "131.25",
    exchange_rate: 0.13125,
    slippage_actual: 0.8,           // Actual slippage percentage
    gas_fee: "0.001",
    gas_used: "150000",
    status: "success",              // success, failed, pending
    dex_used: "hyperion_dex",
    timestamp: "2024-01-15T10:30:00Z",
    block_number: 12345678
  }
  ```

#### Supported Networks and DEXs

| Network | DEXs | Features |
|---------|------|----------|
| **Aptos** | Hyperion DEX, Liquidswap | Low fees, fast execution |
| **Flow EVM** | PunchSwap V2 | Ethereum compatibility |
| **BSC** | PancakeSwap V2/V3 | High liquidity, low costs |

#### Advanced Features

**Route Optimization**
```javascript
{
  route_optimization: true,
  max_hops: 2,                      // Maximum routing hops
  prefer_stable: true,              // Prefer stable coin pairs
  exclude_dexs: ["uniswap_v2"]      // Exclude specific DEXs
}
```

**Price Impact Protection**
```javascript
{
  max_price_impact: 2.0,            // Maximum price impact percentage
  dynamic_slippage: true,           // Adjust slippage based on liquidity
  sandwich_protection: true         // Protect against sandwich attacks
}
```

## Vault Management Node

**Purpose**: Manage vault operations, deposits, withdrawals, and asset transfers
**Node Type**: `vault_node`
**Category**: Action

#### Configuration Parameters
```javascript
{
  operation: "deposit",             // deposit, withdraw, transfer, rebalance
  chain: "aptos",
  vault_address: "0x123...",
  auto_approve: false,              // Require manual approval for large amounts
  confirmation_required: true       // Multi-signature requirement
}
```

#### Operations

**Deposit Assets**
```javascript
{
  operation: "deposit",
  token: "USDC",
  amount: "5000.00",
  source_wallet: "0xabc..."         // Optional: specific wallet
}
```

**Withdraw Assets**
```javascript
{
  operation: "withdraw",
  token: "BTC",
  amount: "0.05",
  destination: "0xdef...",
  reason: "Portfolio rebalancing"   // Audit trail
}
```

**Internal Transfer**
```javascript
{
  operation: "transfer",
  token: "ETH",
  amount: "2.0",
  from_vault: "vault_1",
  to_vault: "vault_2",
  memo: "Strategy adjustment"
}
```

#### Output Handles
- `operation_result` (object): Operation confirmation
  ```javascript
  {
    operation_id: "op_123456",
    status: "completed",
    transaction_hash: "0x789...",
    gas_fee: "0.002",
    timestamp: "2024-01-15T10:30:00Z",
    audit_log: {
      approved_by: ["0xabc...", "0xdef..."],
      risk_assessment: "low",
      compliance_check: "passed"
    }
  }
  ```

## Notification Node

**Purpose**: Send alerts and notifications via multiple channels
**Node Type**: `notification_node`
**Category**: Action

#### Configuration Parameters
```javascript
{
  channels: ["telegram", "email", "webhook"],
  message_template: "🚨 Trading Alert: {signal} signal for {symbol}",
  priority: "normal",               // low, normal, high, urgent
  retry_attempts: 3,
  cooldown_period: 300              // Minimum seconds between notifications
}
```

#### Channel Configurations

**Telegram Bot**
```javascript
{
  telegram: {
    bot_token: "123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11",
    chat_id: "-1001234567890",
    parse_mode: "HTML",
    disable_preview: true
  }
}
```

**Email Notifications**
```javascript
{
  email: {
    smtp_server: "smtp.gmail.com",
    smtp_port: 587,
    username: "alerts@tradingflow.ai",
    recipients: ["trader@example.com", "manager@example.com"],
    subject_template: "TradingFlow Alert: {priority} - {signal}"
  }
}
```

**Webhook Integration**
```javascript
{
  webhook: {
    url: "https://api.tradingplatform.com/webhook",
    method: "POST",
    headers: {
      "Authorization": "Bearer your_token",
      "Content-Type": "application/json"
    },
    retry_policy: "exponential_backoff"
  }
}
```

#### Input Handles
- `message` (string): Notification content
- `data` (object): Structured data for templates
- `priority` (string): Message priority level

#### Output Handles
- `notification_status` (object): Delivery confirmation
  ```javascript
  {
    message_id: "msg_123456",
    channels_sent: ["telegram", "email"],
    delivery_status: {
      telegram: "delivered",
      email: "delivered",
      webhook: "failed"
    },
    retry_count: 0,
    timestamp: "2024-01-15T10:30:00Z"
  }
  ```

## Alert System Node

**Purpose**: Monitor conditions and trigger automated alerts
**Node Type**: `alert_node`
**Category**: Action

#### Configuration Parameters
```javascript
{
  alert_type: "threshold",          // threshold, pattern, anomaly, custom
  monitoring_frequency: 60,         // Check every 60 seconds
  cooldown_period: 600,             // Minimum 10 minutes between alerts
  escalation_rules: {
    first_alert: "notification",
    after_3_alerts: "pause_strategy",
    after_5_alerts: "emergency_stop"
  }
}
```

#### Alert Types

**Threshold Alerts**
```javascript
{
  alert_type: "threshold",
  conditions: [
    {
      metric: "portfolio_value",
      operator: "less_than",
      threshold: 0.95,              // Alert if portfolio < 95% of starting value
      severity: "high"
    },
    {
      metric: "daily_pnl",
      operator: "greater_than",
      threshold: 0.05,              // Alert if daily P&L > 5%
      severity: "medium"
    }
  ]
}
```

**Pattern Recognition Alerts**
```javascript
{
  alert_type: "pattern",
  patterns: [
    {
      name: "flash_crash",
      conditions: {
        price_drop: 0.1,            // 10% price drop
        time_window: 300,           // Within 5 minutes
        volume_spike: 3.0           // 3x normal volume
      },
      action: "emergency_pause"
    }
  ]
}
```

#### Output Handles
- `alert_triggered` (object): Alert details when triggered
  ```javascript
  {
    alert_id: "alert_789",
    type: "threshold",
    severity: "high",
    message: "Portfolio value dropped below 95%",
    triggered_at: "2024-01-15T10:30:00Z",
    current_value: 0.942,
    threshold: 0.95,
    actions_taken: ["notification_sent", "strategy_paused"],
    escalation_level: 1
  }
  ```

## Rebalancing Node

**Purpose**: Automatically rebalance portfolio to maintain target allocations
**Node Type**: `rebalancing_node`
**Category**: Action

#### Configuration Parameters
```javascript
{
  rebalance_frequency: "daily",     // daily, weekly, monthly, threshold_based
  threshold_trigger: 0.05,          // Rebalance if deviation > 5%
  target_allocations: {
    "BTC": 0.4,                     // 40% Bitcoin
    "ETH": 0.3,                     // 30% Ethereum
    "APT": 0.2,                     // 20% Aptos
    "USDC": 0.1                     // 10% Stablecoins
  },
  allowed_deviation: 0.02,          // Allow 2% deviation before rebalancing
  transaction_limits: {
    min_trade_size: 50,             // Minimum $50 per trade
    max_slippage: 1.0               // Maximum 1% slippage
  }
}
```

#### Rebalancing Strategies

**Buy and Hold (Periodic)**
```javascript
{
  strategy: "periodic",
  frequency: "monthly",
  day_of_month: 1,                  // First day of month
  adjustment_method: "buy_only"     // Only buy underweighted assets
}
```

**Dynamic Rebalancing**
```javascript
{
  strategy: "dynamic",
  threshold: 0.03,                  // Rebalance when 3% off target
  band_width: 0.05,                 // 5% rebalancing band
  tax_optimization: true            // Minimize tax implications
}
```

#### Input Handles
- `current_portfolio` (object): Current asset allocations
- `target_allocations` (object): Desired portfolio weights

#### Output Handles
- `rebalancing_trades` (array): Required trades to rebalance
  ```javascript
  {
    trades_required: [
      {
        action: "BUY",
        token: "BTC",
        amount: "0.0025",
        usd_value: 125.00,
        reason: "Underweight by 3.2%"
      },
      {
        action: "SELL",
        token: "ETH",
        amount: "0.15",
        usd_value: 450.00,
        reason: "Overweight by 2.8%"
      }
    ],
    expected_cost: 5.75,            // Estimated transaction costs
    execution_time: "immediate",
    status: "pending_approval"
  }
  ```

## Bridge Node

**Purpose**: Transfer assets between different blockchain networks
**Node Type**: `bridge_node`
**Category**: Action

#### Configuration Parameters
```javascript
{
  from_chain: "aptos",
  to_chain: "flow_evm",
  bridge_provider: "celer",         // celer, multichain, layerzero
  token: "USDC",
  amount: "1000.00",
  slippage_tolerance: 0.5,          // 0.5% bridge slippage
  estimated_time: 300               // Expected completion time
}
```

#### Supported Bridges

| Bridge Provider | Supported Chains | Features |
|----------------|------------------|----------|
| **Celer** | Aptos, Flow EVM, BSC, Polygon | Fast, low fees |
| **Multichain** | 40+ chains | High liquidity |
| **LayerZero** | Aptos, Flow EVM, BSC | Cross-chain messaging |

#### Bridge Process

```
1. Lock assets on source chain ──→ 2. Mint wrapped tokens ──→ 3. Transfer to destination
      ↓                                        ↓                            ↓
   Vault contract                         Bridge contract              User vault
   verifies ownership                     verifies proof               receives assets
```

#### Output Handles
- `bridge_status` (object): Bridge transaction status
  ```javascript
  {
    bridge_tx_hash: "0x123...",
    source_tx_hash: "0x456...",
    destination_tx_hash: "0x789...",
    status: "completed",            // pending, completed, failed
    estimated_completion: "2024-01-15T10:35:00Z",
    actual_completion: "2024-01-15T10:32:30Z",
    fees_paid: {
      bridge_fee: 2.50,
      gas_fee_source: 0.5,
      gas_fee_destination: 0.3
    },
    token_received: "995.00"        // Amount after fees and slippage
  }
  ```

## Emergency Stop Node

**Purpose**: Implement emergency controls to halt trading activities
**Node Type**: `emergency_stop_node`
**Category**: Action

#### Configuration Parameters
```javascript
{
  trigger_conditions: [
    {
      type: "price_crash",
      threshold: 0.15,              // 15% price drop
      time_window: 300              // Within 5 minutes
    },
    {
      type: "volume_spike",
      threshold: 5.0,               // 5x normal volume
      severity: "critical"
    },
    {
      type: "manual_trigger",
      authorized_users: ["admin1", "admin2"]
    }
  ],
  stop_actions: [
    "pause_all_strategies",
    "cancel_pending_orders",
    "notify_stakeholders",
    "switch_to_cash_position"
  ],
  recovery_conditions: {
    price_stabilization: 0.05,      // 5% recovery
    time_delay: 3600,               // 1 hour cooldown
    manual_override: true
  }
}
```

#### Emergency Actions

1. **Strategy Pause**: Immediately halt all automated trading
2. **Order Cancellation**: Cancel all pending orders
3. **Position Management**: Optionally convert to stablecoins
4. **Stakeholder Notification**: Alert all relevant parties
5. **Audit Logging**: Record all emergency actions

#### Output Handles
- `emergency_status` (object): Emergency action results
  ```javascript
  {
    emergency_id: "emergency_001",
    triggered_by: "price_crash",
    trigger_value: 0.152,           // Actual trigger value
    actions_executed: [
      "paused_strategies",
      "cancelled_orders",
      "sent_notifications"
    ],
    affected_strategies: 5,
    timestamp: "2024-01-15T10:30:00Z",
    recovery_status: "monitoring"
  }
  ```

---

Action nodes transform decisions into real-world trading activities. They execute orders, manage assets, and maintain communication channels for effective strategy management.

## Next Steps
- [Utility Nodes →](utility-nodes.md) - Supporting infrastructure
- [Real-World Examples →](../real-world-examples.md) - Action nodes in practice
- [Node Reference Guide →](../node-reference-guide.md) - Complete node overview

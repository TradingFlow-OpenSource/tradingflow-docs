# Buy Node

Buy Node is a specialized trading node for executing buy operations on multi-chain networks.

---

## Node Information

| Property           | Value                      |
| ------------------ | -------------------------- |
| **Node Type**      | `buy_node`                 |
| **Display Name**   | Buy                        |
| **Category**       | Instance (Trade Execution) |
| **Icon**           | 📈 Upward Trend Icon       |
| **Handle Color**   | Green                      |
| **Base Node Type** | `swap_node`                |

---

## Functionality

Buy Node is a specialized version of Swap Node designed specifically for buy operations, simplifying the trading interface.

**Main Uses:**

- Execute token buy orders
- Automated buy strategies
- Portfolio rebalancing (buy side)
- Trading signal execution

**Core Features:**

- 🌐 **Multi-Chain Support**: Aptos and Flow EVM
- 💰 **Simplified Interface**: Buy-specific parameters
- 🔄 **Switch Amount Type**: Support for both Number and Percentage modes
- 🛡️ **Slippage Protection**: Automatic price limits
- 📋 **Transaction Receipt**: Complete transaction details

---

## Key Parameters

### amount_in_human_readable (Switch Type)

The amount field now supports **Switch type**, allowing users to toggle between Number and Percentage modes:

**Data Structure:**

```typescript
{
  mode: "number" | "percentage",  // Mode: exact amount or percentage
  value: string                    // Value as string
}
```

**Mode 1: Number (Exact Amount)**

```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "number",
    "value": "100"
  }
}
// Meaning: Spend 100 USDT to buy APT
```

**Mode 2: Percentage (Balance Percentage)**

```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "percentage",
    "value": "50"
  }
}
// Meaning: Spend 50% of USDT balance to buy APT
```

**Frontend UI:**

- User first selects mode (Number or Percentage)
- Number mode: Shows numeric input field
- Percentage mode: Shows 0-100% slider + input field

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/buy-node.md) - Complete documentation (~9,200 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/trade/BuyNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/swap_node.py`

---

**Maintained by:** TradingFlow Development Team
**Version:** 1.0.0

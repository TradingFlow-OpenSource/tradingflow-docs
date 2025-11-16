# Sell Node

Sell Node is a specialized trading node for executing sell operations on multi-chain networks.

---

## Node Information

| Property           | Value                      |
| ------------------ | -------------------------- |
| **Node Type**      | `sell_node`                |
| **Display Name**   | Sell                       |
| **Category**       | Instance (Trade Execution) |
| **Icon**           | 📉 Downward Trend Icon     |
| **Handle Color**   | Red                        |
| **Base Node Type** | `swap_node`                |

---

## Functionality

Sell Node is a specialized version of Swap Node designed specifically for sell operations, simplifying the trading interface.

**Main Uses:**

- Execute token sell orders
- Automated sell strategies
- Portfolio rebalancing (sell side)
- Take-profit/stop-loss execution

**Core Features:**

- 🌐 **Multi-Chain Support**: Aptos and Flow EVM
- 💰 **Simplified Interface**: Sell-specific parameters
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
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "number",
    "value": "10"
  }
}
// Meaning: Sell 10 APT for USDT
```

**Mode 2: Percentage (Balance Percentage)**

```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "percentage",
    "value": "50"
  }
}
// Meaning: Sell 50% of APT balance for USDT
```

**Frontend UI:**

- User first selects mode (Number or Percentage)
- Number mode: Shows numeric input field
- Percentage mode: Shows 0-100% slider + input field

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/sell-node.md) - Complete documentation (~11,000 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/trade/SellNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/swap_node.py`

---

**Maintained by:** TradingFlow Development Team
**Version:** 1.0.0

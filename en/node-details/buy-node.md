# Buy Node

Buy Node is a specialized trading node for executing buy operations on multi-chain networks.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `buy_node` |
| **Display Name** | Buy |
| **Category** | Instance (Trade Execution) |
| **Icon** | 📈 Upward Trend Icon |
| **Handle Color** | Green |
| **Base Node Type** | `swap_node` |

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
- 🛡️ **Slippage Protection**: Automatic price limits
- 📋 **Transaction Receipt**: Complete transaction details

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/buy-node.md) - Complete documentation (~9,200 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/trade/BuyNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/swap_node.py`

---

**Maintained by:** TradingFlow Development Team  
**Version:** 1.0.0

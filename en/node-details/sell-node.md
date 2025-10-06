# Sell Node

Sell Node is a specialized trading node for executing sell operations on multi-chain networks.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `sell_node` |
| **Display Name** | Sell |
| **Category** | Instance (Trade Execution) |
| **Icon** | 📉 Downward Trend Icon |
| **Handle Color** | Red |
| **Base Node Type** | `swap_node` |

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
- 🛡️ **Slippage Protection**: Automatic price limits
- 📋 **Transaction Receipt**: Complete transaction details

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/sell-node.md) - Complete documentation (~11,000 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/trade/SellNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/swap_node.py`

---

**Maintained by:** TradingFlow Development Team  
**Version:** 1.0.0

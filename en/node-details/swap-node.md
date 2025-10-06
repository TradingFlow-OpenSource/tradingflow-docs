# Swap Node

Swap Node is a multi-chain token swap execution node that supports trading on Aptos and Flow EVM chains through Vault contracts.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `swap_node` |
| **Display Name** | Swap |
| **Category** | Instance (Trade Execution) |
| **Icon** | 🔄 Swap Icon |
| **Handle Color** | Emerald (Green) |
| **Version** | 0.0.2 |
| **Base Node Type** | `swap_node` |

---

## Functionality

Swap Node executes token swap transactions on multiple blockchain networks. It supports both percentage-based and fixed-amount trading methods, with automatic slippage protection.

**Main Uses:**
- Execute token swap transactions
- Multi-chain trading (Aptos, Flow EVM)
- Automated portfolio rebalancing
- Trading strategy execution

**Core Features:**
- 🌐 **Multi-Chain Support**: Aptos and Flow EVM
- 💰 **Flexible Amount Input**: Percentage or fixed amount
- 🛡️ **Slippage Protection**: Automatic price limit calculation
- 📋 **Transaction Receipt**: Complete transaction details

---

## Supported Chains

- **Aptos**: Native Aptos chain support
- **Flow EVM**: Flow EVM chain support
- **More coming soon**: Additional EVM-compatible chains planned

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/swap-node.md) - Complete documentation (~16,600 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/trade/SwapNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/swap_node.py`

---

**Maintained by:** TradingFlow Development Team  
**Version:** 1.0.0

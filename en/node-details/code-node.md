# Code Node

Code Node is a powerful computation node that allows you to execute custom Python code for data processing, transformation, and analysis.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `code_node` |
| **Display Name** | Code |
| **Category** | Compute (Code Execution) |
| **Icon** | 💻 Computer Icon |
| **Handle Color** | Slate (Gray) |

---

## Functionality

Code Node provides a Python execution environment where you can write custom code to process input data and generate output. It's perfect for custom logic, data transformation, and complex calculations.

**Main Uses:**
- Custom data processing and transformation
- Complex mathematical calculations
- Data filtering and aggregation
- Custom trading logic implementation
- Integration with external Python libraries

**Core Features:**
- 💻 **Python Execution**: Run any Python code
- 📦 **Library Support**: Access to common Python libraries (pandas, numpy, etc.)
- 🔄 **Dynamic Inputs/Outputs**: Define custom input and output handles
- 🛡️ **Safe Execution**: Sandboxed environment for security

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/code-node.md) - Complete documentation (~19,000 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/compute/CodeNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/code_node.py`

---

**Maintained by:** TradingFlow Development Team  
**Version:** 1.0.0

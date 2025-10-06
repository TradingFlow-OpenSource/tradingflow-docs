# Dataset Output Node

Dataset Output Node writes data to Google Sheets documents for storage and visualization.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `dataset_output_node` |
| **Display Name** | Dataset Output |
| **Category** | Instance (Data Output) |
| **Icon** | 📤 Upload Icon |
| **Handle Color** | Orange |
| **Base Node Type** | `dataset_node` |

---

## Functionality

Dataset Output Node writes processed data to Google Sheets, enabling data persistence, reporting, and external analysis.

**Main Uses:**
- Save trading results to spreadsheets
- Export analysis data
- Create automated reports
- Share data with external tools

**Core Features:**
- 📤 **Google Sheets Integration**: Direct spreadsheet writing
- 🔄 **Automatic Format Conversion**: Converts JSON to sheet format
- 📝 **Multi-Format Support**: Handles various data structures
- ✏️ **Write Modes**: Append, overwrite, or update data

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/dataset-output-node.md) - Complete documentation (~8,600 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/outputs/DatasetOutputNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/dataset_node.py`

---

**Maintained by:** TradingFlow Development Team  
**Version:** 1.0.0

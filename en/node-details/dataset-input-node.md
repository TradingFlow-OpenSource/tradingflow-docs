# Dataset Input Node

Dataset Input Node reads data from Google Sheets documents for use in trading workflows.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `dataset_input_node` |
| **Display Name** | Dataset Input |
| **Category** | Instance (Data Input) |
| **Icon** | 📊 Spreadsheet Icon |
| **Handle Color** | Green |
| **Base Node Type** | `dataset_node` |

---

## Functionality

Dataset Input Node connects to Google Sheets to read tabular data, making it available for processing in your trading workflows.

**Main Uses:**
- Import trading parameters from spreadsheets
- Load historical data for analysis
- Read configuration tables
- Import external data sources

**Core Features:**
- 📊 **Google Sheets Integration**: Direct spreadsheet access
- 🔄 **Automatic Data Conversion**: Converts sheet data to JSON format
- 📝 **Multi-Format Support**: Handles various data structures
- 🔐 **Secure Authentication**: OAuth2 authentication

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/dataset-input-node.md) - Complete documentation (~9,300 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/inputs/DatasetInputNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/dataset_node.py`

---

**Maintained by:** TradingFlow Development Team  
**Version:** 1.0.0

# Dataset Input Node

**🔄 English translation in progress**

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

The complete English documentation for this node is currently being translated from the Chinese version.

**For now, you can:**
- View the [Chinese version](../../zh/node-details/dataset-input-node.md) (~9,300 words)
- Check the basic configuration in the frontend: `1_weather_frontend/src/pages/flow/components/TFNode/instances/inputs/DatasetInputNode.tsx`
- Refer to the backend implementation: `3_weather_cluster/tradingflow/station/nodes/dataset_node.py`

---

**Status:** 🔄 Translation in progress  
**Priority:** Medium  
**Expected completion:** Q4 2025

For urgent inquiries, please contact the TradingFlow development team.

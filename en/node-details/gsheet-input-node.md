# Google Sheet Input Node

Google Sheet Input Node reads data from Google Sheets and is a data input node. It automatically handles data retrieval and format conversion, passing tabular data to downstream nodes for processing.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `gsheet_input_node` |
| **Display Name** | Google Sheet Input |
| **Node Category** | Input |
| **Icon** | 📊 Sheet icon (purple) |
| **Handle Color** | Emerald (green) |

---

## Functionality

Google Sheet Input Node reads data from Google Sheets and converts it to standard JSON format for downstream nodes.

**Primary Use Cases:**
- Read data from Google Sheets
- Import historical data for analysis
- Load configuration data
- Read trading records
- Fetch external datasets

**Key Features:**
- 📊 **Google Sheets Integration**: Direct access to Google Sheets data
- 🔄 **Auto Formatting**: Converts tabular data to JSON format
- 📋 **Header Support**: Automatic header row detection and processing
- 🔗 **URL Support**: Accepts full URL or Sheet ID

---

## Input Parameters

### Parameter List

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `doc_link` | text | No | - | Google Sheets URL or ID |

### doc_link Parameter

**Supported Formats:**

1. **Full URL:**
```
https://docs.google.com/spreadsheets/d/1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM/edit?usp=sharing
```

2. **Sheet ID:**
```
1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM
```

3. **Simplified URL:**
```
docs.google.com/spreadsheets/d/1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM
```

**ID Extraction:**
- The node automatically extracts Sheet ID from URLs
- No need to manually copy the ID

---

## Output Parameters

### Output List

| Output ID | Display Name | Data Type | Description |
|-----------|--------------|-----------|-------------|
| `data` | Data | object | Retrieved data |

### data Output

**Data Type:** `object`

**Data Structure:**

```typescript
{
  headers: string[];     // Column names array
  rows: Array<any[]>;    // Data rows array
  row_count: number;     // Number of data rows
  column_count: number;  // Number of columns
}
```

**Example Output:**

```json
{
  "headers": ["Date", "Symbol", "Price", "Volume"],
  "rows": [
    ["2024-01-15", "BTC", "45000", "1000"],
    ["2024-01-16", "ETH", "2500", "5000"],
    ["2024-01-17", "APT", "10", "10000"]
  ],
  "row_count": 3,
  "column_count": 4
}
```

---

## Usage Examples

### Example 1: Reading Price Data

**Scenario:** Read historical price data from Google Sheets for analysis.

**Google Sheets Content:**
```
| Date       | Symbol | Price | Volume |
|------------|--------|-------|--------|
| 2024-01-15 | BTC    | 45000 | 1000   |
| 2024-01-16 | ETH    | 2500  | 5000   |
```

**Workflow Structure:**
```
Google Sheet Input Node (read price data)
    ↓ data
Code Node (calculate averages and trends)
    ↓ analysis
AI Model Node (generate trading recommendations)
    ↓ recommendation
Buy/Sell Node (execute trades)
```

**Node Configuration:**
```json
{
  "doc_link": "https://docs.google.com/spreadsheets/d/1uQvzsNIkaan.../edit"
}
```

---

### Example 2: Loading Strategy Configuration

**Scenario:** Read trading strategy configuration from Google Sheets.

**Workflow Structure:**
```
Google Sheet Input Node (read strategy config)
    ↓ data { strategies }
Code Node (select current strategy)
    ↓ active_strategy
Condition Node (make decisions based on parameters)
    ↓
Trading nodes
```

---

## Google Sheets Configuration

### 1. Create Service Account

1. Visit [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable **Google Sheets API**
4. Create a **Service Account**
5. Download the JSON credentials file

### 2. Share Google Sheets

1. Open your Google Sheets
2. Click the "Share" button in the top right
3. Add the Service Account email address
4. Grant "Viewer" permission

### 3. Configure Credentials

**Environment Variable (Recommended):**
```bash
export GOOGLE_CREDENTIALS_PATH="/path/to/credentials.json"
```

---

## Best Practices

### 1. Header Design

**Recommended:**
```
| date       | symbol | price  | volume  |  # lowercase, concise
| Date       | Symbol | Price  | Volume  |  # capitalized
```

**Avoid:**
```
| 日期 | 代币 | 价格 | 交易量 |  # Non-ASCII characters may cause issues
```

### 2. Data Format

**In Google Sheets:**
- Dates: Use ISO format `YYYY-MM-DD`
- Numbers: Avoid comma separators
- Booleans: Use `TRUE`/`FALSE` or `1`/`0`
- Empty values: Leave blank, don't use `null` or `N/A`

---

## Important Notes

1. **Google API Quota**
   - Daily read limits apply
   - Avoid frequent reads in loops
   - Consider caching data

2. **Credentials Security**
   - Don't commit credentials to Git
   - Use environment variables for paths
   - Limit Service Account permissions

3. **Data Types**
   - All data is read as strings
   - Type conversion needed in Code Node

---

## Troubleshooting

**Q: "Google Sheets credentials file not found"?**

A:
1. Verify credentials file path is correct
2. Check `GOOGLE_CREDENTIALS_PATH` environment variable
3. Ensure credentials file is readable

---

**Q: "Spreadsheet not found"?**

A:
1. Confirm Sheet ID is correct
2. Check if Service Account has access
3. Verify Google Sheets is shared

---

**Q: Retrieved data is empty?**

A:
1. Confirm the sheet contains data
2. Check Google Sheets sharing settings
3. Verify credentials are correct

---

## Technical Specifications

| Specification | Value |
|---------------|-------|
| **Node Version** | 0.1.0 |
| **Data Source** | Google Sheets |
| **API** | Google Sheets API v4 |
| **Credential Type** | Service Account |
| **Permission Required** | Viewer |
| **Timeout** | 60 seconds |

---

## Related Nodes

- **Google Sheet Output Node** - Write data to Google Sheets
- **Code Node** - Process retrieved data
- **AI Model Node** - Analyze datasets

---

**Related Documentation:**
- [Nodes and Workflows](../core-concepts/nodes-and-workflows.md) - Node fundamentals
- [Code Node](code-node.md) - Data processing node
- [Weather Syntax](../core-concepts/weather-syntax.md) - Workflow file format

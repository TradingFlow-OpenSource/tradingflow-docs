# Google Sheet Output Node

Google Sheet Output Node writes data to Google Sheets and is a data output node. It automatically handles data format conversion, persisting upstream node data to Google Sheets.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `gsheet_output_node` |
| **Display Name** | Google Sheet Output |
| **Node Category** | Output |
| **Icon** | 📊 Sheet icon (green) |
| **Handle Color** | Rose |

---

## Functionality

Google Sheet Output Node writes upstream node data to Google Sheets for data persistence, logging, and result export.

**Primary Use Cases:**
- Save trading records
- Export analysis results
- Record log data
- Persist configuration
- Backup important data

**Key Features:**
- 💾 **Google Sheets Integration**: Direct write to Google Sheets
- 🔄 **Auto Formatting**: Converts JSON data to tabular format
- 📋 **Header Generation**: Automatically creates header rows
- ✍️ **Overwrite Mode**: Overwrites existing data

---

## Input Parameters

### Parameter List

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | object | Yes | - | Data to write (JSON format) |
| `doc_link` | text | No | - | Google Sheets URL or ID |

### data Parameter

**Source:** Upstream node output

**Expected Format:**

```typescript
{
  headers: string[];     // Column names array
  rows: Array<any[]>;    // Data rows array
}
```

**Example Input:**

```json
{
  "headers": ["Date", "Symbol", "Action", "Price", "Profit"],
  "rows": [
    ["2024-01-15", "BTC", "Buy", "45000", "1200"],
    ["2024-01-16", "ETH", "Sell", "2500", "350"],
    ["2024-01-17", "APT", "Buy", "10", "-50"]
  ]
}
```

**Supported Data Formats:**
1. **Standard format**: Contains `headers` and `rows`
2. **Object array**: `[{col1: val1, col2: val2}, ...]`
3. **Nested objects**: Auto-flattened

### doc_link Parameter

**Supported Formats:**

1. **Full URL:**
```
https://docs.google.com/spreadsheets/d/1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM/edit
```

2. **Sheet ID:**
```
1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM
```

---

## Output Parameters

Google Sheet Output Node has no output parameters (terminal node).

---

## Usage Examples

### Example 1: Saving Trading Records

**Scenario:** Save trading history to Google Sheets.

**Workflow Structure:**
```
Vault Node (get vault info)
    ↓ vault_balance
Swap Node (execute trade)
    ↓ trade_receipt { tx_hash, amount_in, amount_out }
Code Node (format trading record)
    ↓ formatted_data
Google Sheet Output Node (save to Google Sheets)
```

**Code Node Output:**
```json
{
  "headers": ["Timestamp", "From", "To", "Amount_In", "Amount_Out", "TX_Hash"],
  "rows": [
    ["2024-01-15 10:30:00", "USDT", "APT", "100", "13.79", "0x1234..."]
  ]
}
```

**Node Configuration:**
```json
{
  "doc_link": "1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM"
}
```

---

### Example 2: Exporting Analysis Results

**Scenario:** Export AI analysis results to Google Sheets.

**Workflow Structure:**
```
X Listener Node (get tweets)
    ↓ latest_tweets
AI Model Node (sentiment analysis)
    ↓ ai_response { sentiment_score, key_points }
Code Node (organize analysis results)
    ↓ analysis_data
Google Sheet Output Node (export results)
```

**analysis_data Format:**
```json
{
  "headers": ["Date", "Source", "Sentiment", "Score", "Summary"],
  "rows": [
    ["2024-01-15", "@elonmusk", "Positive", "0.85", "Bullish on Bitcoin"],
    ["2024-01-16", "@VitalikButerin", "Neutral", "0.50", "Discussing ETH upgrade"]
  ]
}
```

---

### Example 3: Logging

**Scenario:** Record workflow execution logs.

**Workflow Structure:**
```
Any node (execute operation)
    ↓
Code Node (collect log info)
    ↓ log_data
Google Sheet Output Node (save logs)
```

**log_data Example:**
```json
{
  "headers": ["Timestamp", "Node", "Status", "Message", "Duration"],
  "rows": [
    ["2024-01-15 10:30:00", "Vault Node", "Success", "Retrieved balance", "1.2s"],
    ["2024-01-15 10:30:02", "Swap Node", "Success", "Swap executed", "3.5s"]
  ]
}
```

---

## Google Sheets Configuration

### Requirements

1. Create Service Account
2. Enable Google Sheets API
3. Download credentials file
4. Share Google Sheets (requires "Editor" permission)

**Permission Requirements:**
- Google Sheet Input: "Viewer" is sufficient
- Google Sheet Output: **Must have "Editor" permission**

---

## Best Practices

### 1. Data Formatting

**Format in Code Node:**

```python
# Example Code Node
import pandas as pd

# Assuming input_data is upstream data
df = pd.DataFrame(input_data)

# Format output
output_data = {
    "headers": df.columns.tolist(),
    "rows": df.values.tolist()
}
```

### 2. Data Validation

**Validate before writing:**
```python
# In Code Node
if len(rows) == 0:
    raise ValueError("No data to write")

if len(headers) != len(rows[0]):
    raise ValueError("Headers and data columns mismatch")
```

---

## Important Notes

1. **Overwrite Warning**
   - Google Sheet Output Node **overwrites** existing data
   - Confirm target worksheet before writing
   - Consider using different worksheet names

2. **Permission Requirements**
   - Service Account must have "Editor" permission
   - "Viewer" only permission will cause write failure

3. **Data Format**
   - Must provide `headers` and `rows`
   - All rows must have consistent column count
   - Data will be converted to strings

4. **API Quota**
   - Daily write limits apply
   - Heavy writes may trigger limits
   - Consider batch writing

5. **Concurrent Writes**
   - Avoid multiple nodes writing to the same sheet
   - May cause data conflicts

---

## Troubleshooting

**Q: "Permission denied"?**

A:
1. Confirm Service Account has "Editor" permission
2. Re-share Google Sheets
3. Check credentials file is correct

---

**Q: Data format is garbled after writing?**

A:
1. Check if `headers` and `rows` have consistent column counts
2. Confirm all rows have the same length
3. Validate data format

---

**Q: "No input data received"?**

A:
1. Confirm upstream node correctly sent `data` signal
2. Check data connection is correct
3. Validate data format meets requirements

---

## Technical Specifications

| Specification | Value |
|---------------|-------|
| **Node Version** | 0.1.0 |
| **Write Mode** | Write (overwrite) |
| **Data Source** | Google Sheets |
| **API** | Google Sheets API v4 |
| **Permission Required** | Editor |
| **Timeout** | 60 seconds |

---

## Related Nodes

- **Google Sheet Input Node** - Read data from Google Sheets
- **Code Node** - Format data for writing
- **AI Model Node** - Generate analysis results
- **Swap Node** - Generate trading records

---

**Related Documentation:**
- [Nodes and Workflows](../core-concepts/nodes-and-workflows.md) - Node fundamentals
- [Google Sheet Input Node](gsheet-input-node.md) - Data reading node
- [Code Node](code-node.md) - Data processing node

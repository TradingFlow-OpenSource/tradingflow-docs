# RootData Node

RootData Node queries the RootData API to fetch cryptocurrency project, VC, and personnel data. It supports search, ID mapping, trending lists, detail queries, and more.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `rootdata_node` |
| **Display Name** | RootData Node |
| **Node Category** | Input |
| **Icon** | 📊 Database icon (blue) |
| **Handle Color** | Emerald (green) |

---

## Functionality

RootData Node connects to the RootData API, providing comprehensive data access to the cryptocurrency ecosystem, including project information, investment institutions, team members, fundraising dynamics, and more.

**Primary Use Cases:**
- Search cryptocurrency projects, VCs, and people
- Get project/VC/people details
- Query trending project rankings
- Track fundraising dynamics and personnel changes
- Get ecosystem and tag mappings

**Key Features:**
- 🔍 **Multi-mode Search**: Supports 19 different operation types
- 📊 **Trending Lists**: Top 100 hot projects, X platform trending
- 👥 **People Tracking**: Position changes, KOL rankings
- 💰 **Fundraising Data**: Batch fundraising information
- 🏷️ **Classification System**: Ecosystem and tag mappings

---

## Input Parameters

### Primary Parameters (Always Visible)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | select | Yes | `search` | Operation type |
| `query` | text | No | - | Search keyword |

### Advanced Parameters (Hidden by Default)

The following parameters are hidden by default and can be expanded when needed:

| Parameter | Type | Description | Applicable Operations |
|-----------|------|-------------|----------------------|
| `type` | number | Type: 1=Project, 2=VC, 3=People | search |
| `project_id` | number | Project ID | get_item, get_fac |
| `contract_address` | text | Project contract address | get_item |
| `org_id` | number | VC organization ID | get_org |
| `people_id` | number | People ID | get_people |
| `ecosystem_ids` | text | Ecosystem IDs (comma-separated) | projects_by_ecosystems |
| `tag_ids` | text | Tag IDs (comma-separated) | projects_by_tags |
| `page` | number | Page number | list operations |
| `page_size` | number | Items per page | list operations |
| `begin_time` | number | Start timestamp (ms) | get_fac, ser_change |
| `end_time` | number | End timestamp (ms) | get_fac, ser_change |
| `min_amount` | number | Minimum amount (USD) | get_fac |
| `max_amount` | number | Maximum amount (USD) | get_fac |
| `days` | number | Days (1 or 7) | hot_index |
| `rank_type` | select | Rank type: heat/influence | leading_figures_on_crypto_x |
| `language` | select | Language: en/cn | - |
| `heat` | boolean | Include heat data | - |
| `influence` | boolean | Include influence data | - |
| `followers` | boolean | Include follower data | - |
| `recent_joinees` | boolean | Include recent joinees | job_changes |
| `recent_resignations` | boolean | Include recent resignations | job_changes |
| `include_team` | boolean | Include team info | get_item |
| `include_investors` | boolean | Include investor info | get_item |
| `include_investments` | boolean | Include investments | get_org |
| `precise_x_search` | boolean | Precise X search | twitter_map |

---

## Operation Types

### operation Parameter

| Value | Name | Description |
|-------|------|-------------|
| `search` | Search | Search projects/VCs/people |
| `id_map` | ID Map | Get ID mapping table |
| `get_item` | Project Detail | Get project details |
| `get_org` | VC Detail | Get VC organization details |
| `get_people` | People Detail | Get people details |
| `get_invest` | Investors (batch) | Batch get investors |
| `twitter_map` | X Data Map | X platform data mapping |
| `get_fac` | Fundraising (batch) | Batch get fundraising info |
| `ser_change` | Sync Updates | Sync update list |
| `hot_index` | Top 100 Hot Projects | Hot projects Top 100 |
| `hot_project_on_x` | X Hot Projects | X platform hot projects |
| `leading_figures_on_crypto_x` | X Hot People | X platform hot figures |
| `job_changes` | People Position Dynamics | Personnel position changes |
| `new_tokens` | New Tokens (3 months) | New tokens in last 3 months |
| `ecosystem_map` | Ecosystem Map | Ecosystem mapping |
| `tag_map` | Tag Map | Tag mapping |
| `projects_by_ecosystems` | Projects by Ecosystems | Query projects by ecosystem |
| `projects_by_tags` | Projects by Tags | Query projects by tags |
| `quotacredits` | API Balance | Query API balance |

---

## Output Parameters

### Output List

| Output ID | Display Name | Data Type | Description |
|-----------|--------------|-----------|-------------|
| `data` | Data | object | RootData API response |

### data Output

**Data Structure:**

```typescript
{
  result: number;        // Result code (0=success)
  data: any;             // Specific data (varies by operation)
  message?: string;      // Error message (if any)
  _meta?: {              // Metadata
    page?: number;
    page_size?: number;
    total?: number;
  }
}
```

---

## Usage Examples

### Example 1: Search Projects

**Scenario:** Search for projects containing "DeFi" keyword.

**Node Configuration:**
```json
{
  "operation": "search",
  "query": "DeFi",
  "type": 1
}
```

**Output Example:**
```json
{
  "result": 0,
  "data": [
    {
      "id": 12345,
      "name": "Uniswap",
      "symbol": "UNI",
      "category": "DeFi",
      "description": "Decentralized exchange protocol"
    }
  ],
  "_meta": {
    "total": 150,
    "page": 1,
    "page_size": 10
  }
}
```

---

### Example 2: Get Trending Projects

**Scenario:** Get Top 100 hot projects from the past 7 days.

**Node Configuration:**
```json
{
  "operation": "hot_index",
  "days": 7
}
```

**Workflow Structure:**
```
RootData Node (get hot projects)
    ↓ data { projects }
Code Node (filter DeFi category)
    ↓ defi_projects
AI Model Node (analyze potential projects)
    ↓ recommendations
Telegram Sender Node (send recommendations)
```

---

### Example 3: Track Personnel Changes

**Scenario:** Get recent job position changes.

**Node Configuration:**
```json
{
  "operation": "job_changes",
  "recent_joinees": true,
  "recent_resignations": true,
  "page": 1,
  "page_size": 50
}
```

**Use Cases:**
- Track KOL movements
- Discover industry trends
- Monitor team changes

---

### Example 4: Fundraising Analysis

**Scenario:** Get fundraising data within a specified time range.

**Node Configuration:**
```json
{
  "operation": "get_fac",
  "begin_time": 1704067200000,
  "end_time": 1706745600000,
  "min_amount": 1000000,
  "page": 1,
  "page_size": 100
}
```

**Workflow Structure:**
```
RootData Node (get fundraising data)
    ↓ data
Code Node (statistical analysis)
    ↓ stats { total_raised, top_sectors, avg_round_size }
Google Sheet Output Node (export report)
```

---

### Example 5: Project Detail Query

**Scenario:** Get detailed information for a specific project, including team and investors.

**Node Configuration:**
```json
{
  "operation": "get_item",
  "project_id": 12345,
  "include_team": true,
  "include_investors": true
}
```

---

## Advanced Parameters Usage

### About Advanced Parameters

RootData Node has 22 advanced parameters, hidden by default to simplify the interface. These parameters are only needed for specific operations.

**How to show advanced parameters:**
1. Click the "Show advanced params" button at the bottom of the node
2. Select the parameters you need in the popup
3. Or click "Show All" to display all

**How to hide displayed parameters:**
- Click the "Hide" button on the right side of the parameter title
- Hiding will clear the parameter value

**Agent Behavior:**
- When the Agent specifies a value for an advanced parameter, that parameter automatically becomes visible
- Unspecified advanced parameters remain hidden

---

## API Quota

RootData API uses a quota system. Use the `quotacredits` operation to check balance:

```json
{
  "operation": "quotacredits"
}
```

**Return Example:**
```json
{
  "result": 0,
  "data": {
    "remaining_credits": 10000,
    "total_credits": 50000,
    "expires_at": "2024-12-31"
  }
}
```

---

## Best Practices

### 1. Operation Selection

| Need | Recommended Operation |
|------|----------------------|
| Project discovery | `search` + `query` |
| Trending tracking | `hot_index` or `hot_project_on_x` |
| Detail retrieval | `get_item`, `get_org`, `get_people` |
| Fundraising analysis | `get_fac` + time filters |
| Personnel dynamics | `job_changes` or `leading_figures_on_crypto_x` |

### 2. Pagination Usage

For list operations, use pagination properly:
- Default `page_size` is 10
- Consider batch fetching for large datasets
- Use `_meta.total` to determine total count

### 3. Caching Strategy

- Trending list data can be cached (changes slowly)
- Detail data should be cached
- Query directly for real-time requirements

---

## Important Notes

1. **API Quota Management**
   - Monitor remaining quota
   - Avoid unnecessary repeated queries
   - Set reasonable `page_size`

2. **Parameter Compatibility**
   - Different operations require different parameters
   - Check required parameters for each operation type
   - Irrelevant parameters can be ignored

3. **Data Freshness**
   - Trending lists updated daily
   - Fundraising data updated in real-time
   - Project details updated periodically

---

## Troubleshooting

**Q: Returns "result": -1?**

A:
1. Check if operation type is correct
2. Confirm required parameters are provided
3. Verify API quota is sufficient

---

**Q: Search results are empty?**

A:
1. Try adjusting search keywords
2. Check `type` parameter setting
3. Relax filter conditions

---

**Q: Paginated data is incomplete?**

A:
1. Check `page` and `page_size` settings
2. Confirm `_meta.total` count
3. Iterate through all pages for complete data

---

## Technical Specifications

| Specification | Value |
|---------------|-------|
| **Node Version** | 0.1.0 |
| **Operation Types** | 19 |
| **Primary Parameters** | 2 |
| **Advanced Parameters** | 22 |
| **API** | RootData API |
| **Timeout** | 60 seconds |

---

## Related Nodes

- **AI Model Node** - Analyze RootData data
- **Code Node** - Process and filter data
- **Google Sheet Output Node** - Export data reports
- **Telegram Sender Node** - Send data notifications

---

**Related Documentation:**
- [Nodes and Workflows](../core-concepts/nodes-and-workflows.md) - Node fundamentals
- [AI Model Node](ai-model-node.md) - AI analysis node
- [Code Node](code-node.md) - Data processing node

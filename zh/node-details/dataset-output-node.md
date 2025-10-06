# Dataset Output Node

Dataset Output Node 是专门用于向外部数据源写入数据的节点，是 Dataset Node 的实例节点。节点支持将数据写入 Google Sheets，用于数据持久化、日志记录和结果导出。

---

## 节点信息

| 属性 | 值 |
|------|-----|
| **节点类型** | `dataset_output_node` |
| **显示名称** | Dataset Output |
| **节点分类** | Core（核心功能） |
| **图标** | 📤 数据库图标 |
| **句柄颜色** | Rose（玫瑰红） |
| **节点类别** | Instance Node（实例节点） |
| **基类节点** | `dataset_node` |

---

## 功能说明

Dataset Output Node 将上游节点的数据写入 Google Sheets，支持创建新表格或覆盖现有数据。节点自动处理数据格式转换和表格写入。

**主要用途：**
- 保存交易记录
- 导出分析结果
- 记录日志数据
- 持久化配置
- 备份重要数据

**核心特性：**
- 💾 **Google Sheets 集成**：直接写入 Google Sheets
- 🔄 **自动格式化**：将 JSON 数据转换为表格格式
- 📋 **表头生成**：自动创建表头行
- ✍️ **写入模式**：覆盖现有数据
- 🎯 **范围指定**：支持写入特定范围

---

## 与 Dataset Node 的关系

Dataset Output Node 是 Dataset Node 的**实例节点**，专门用于数据写入：

```
DatasetNode (基类)
    └─ 支持 read/write/append 三种模式
        ↓ 实例化（强制 mode="write"）
DatasetOutputNode (实例)
    └─ 仅支持 write 模式，简化参数
```

---

## 输入参数

### 参数列表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `data` | object | ✅ | - | 要写入的数据 |
| `doc_link` | text | ✅ | - | Google Sheets URL 或 ID |
| `worksheet_name` | text | ❌ | `Sheet1` | 工作表名称 |
| `range` | text | ❌ | `A1:Z1000` | 数据范围 |
| `header_row` | boolean | ❌ | `true` | 是否写入表头行 |

### data 参数

**来源：** 上游节点输出

**期望格式：**

```typescript
{
  headers: string[];     // 列名数组
  rows: Array<any[]>;    // 数据行数组
}
```

**示例输入：**

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

**支持的数据格式：**
1. **标准格式**：包含 `headers` 和 `rows`
2. **对象数组**：`[{col1: val1, col2: val2}, ...]`
3. **嵌套对象**：自动扁平化

### doc_link 参数

**格式与 Dataset Input Node 相同**

**支持：**
- 完整 Google Sheets URL
- Sheet ID
- 简化 URL

### worksheet_name 参数

**说明：** 要写入的工作表名称

**行为：**
- 如果工作表存在：覆盖数据
- 如果工作表不存在：创建新工作表

### header_row 参数

**说明：** 是否在第一行写入列名

**效果：**
- `true`：写入表头行 + 数据行
- `false`：仅写入数据行

---

## 输出参数

Dataset Output Node 没有输出参数（终端节点）。

---

## 使用示例

### 示例 1：保存交易记录

**场景：** 将交易历史保存到 Google Sheets。

**工作流结构：**
```
Vault Node (获取金库信息)
    ↓ vault_balance
Swap Node (执行交易)
    ↓ trade_receipt { tx_hash, amount_in, amount_out }
Code Node (格式化交易记录)
    ↓ formatted_data
Dataset Output Node (保存到 Google Sheets)
```

**Code Node 输出：**
```json
{
  "headers": ["Timestamp", "From", "To", "Amount_In", "Amount_Out", "TX_Hash"],
  "rows": [
    ["2024-01-15 10:30:00", "USDT", "APT", "100", "13.79", "0x1234..."]
  ]
}
```

**Dataset Output Node 配置：**
```json
{
  "doc_link": "1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM",
  "worksheet_name": "Trading History",
  "header_row": true
}
```

**结果：**
- 数据写入 `Trading History` 工作表
- 包含表头和数据
- 覆盖现有内容

---

### 示例 2：导出分析结果

**场景：** 将 AI 分析结果导出到 Google Sheets。

**工作流结构：**
```
X Listener Node (获取推文)
    ↓ latest_tweets
AI Model Node (情绪分析)
    ↓ ai_response { sentiment_score, key_points }
Code Node (整理分析结果)
    ↓ analysis_data
Dataset Output Node (导出结果)
```

**analysis_data 格式：**
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

### 示例 3：日志记录

**场景：** 记录工作流执行日志。

**工作流结构：**
```
任意节点（执行操作）
    ↓
Code Node (收集日志信息)
    ↓ log_data
Dataset Output Node (保存日志)
```

**log_data 示例：**
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

## Google Sheets 配置

### 配置要求

**与 Dataset Input Node 相同：**
1. 创建 Service Account
2. 启用 Google Sheets API
3. 下载凭证文件
4. 共享 Google Sheets（需要「编辑者」权限）

**权限要求：**
- Dataset Input: 「查看者」即可
- Dataset Output: 必须「编辑者」权限

---

## 最佳实践

### 1. 数据格式化

**在 Code Node 中格式化：**

```python
# 示例 Code Node
import pandas as pd

# 假设 input_data 是上游数据
df = pd.DataFrame(input_data)

# 格式化输出
output_data = {
    "headers": df.columns.tolist(),
    "rows": df.values.tolist()
}
```

### 2. 追加数据（Workaround）

由于 Dataset Output Node 是覆盖模式，要追加数据：

**方案1 - 使用 Code Node：**
```python
# 1. 用 Dataset Input Node 读取现有数据
# 2. 在 Code Node 中合并新旧数据
# 3. 用 Dataset Output Node 写入全部数据
```

**方案2 - 使用基类：**
```python
# 直接使用 Dataset Node，设置 mode="append"
```

### 3. 数据验证

**在写入前验证：**
```python
# Code Node 中
if len(rows) == 0:
    raise ValueError("No data to write")

if len(headers) != len(rows[0]):
    raise ValueError("Headers and data columns mismatch")
```

---

## 注意事项

### ⚠️ 重要提示

1. **覆盖警告**
   - Dataset Output Node 会**覆盖**现有数据
   - 写入前确认目标工作表
   - 建议使用不同工作表名称

2. **权限要求**
   - Service Account 必须有「编辑者」权限
   - 仅「查看者」权限会导致写入失败

3. **数据格式**
   - 必须提供 `headers` 和 `rows`
   - 所有行的列数必须一致
   - 数据会被转换为字符串

4. **API 配额**
   - 每天有写入次数限制
   - 大量写入可能触发限制
   - 考虑批量写入

5. **并发写入**
   - 避免多个节点同时写入同一工作表
   - 可能导致数据冲突

---

## 故障排查

**Q: 提示 "Permission denied"？**

A:
1. 确认 Service Account 有「编辑者」权限
2. 重新共享 Google Sheets
3. 检查凭证文件是否正确

---

**Q: 数据写入后格式混乱？**

A:
1. 检查 `headers` 和 `rows` 的列数是否一致
2. 确认所有行的长度相同
3. 验证数据格式

---

**Q: 提示 "No input data received"？**

A:
1. 确认上游节点正确发送了 `data` 信号
2. 检查数据连接是否正确
3. 验证数据格式符合要求

---

**Q: 想要追加数据而不是覆盖？**

A:
```
方案1：使用 Code Node 合并数据
Dataset Input Node (读取现有) → 
Code Node (合并新旧) → 
Dataset Output Node (写入全部)

方案2：使用 Dataset Node 基类
设置 mode="append"
```

---

## 技术规格

| 规格项 | 值 |
|--------|-----|
| **节点版本** | 0.0.2 |
| **节点类别** | Instance Node |
| **基类节点** | `dataset_node` |
| **写入模式** | Write（覆盖） |
| **数据源** | Google Sheets |
| **API** | Google Sheets API v4 |
| **权限要求** | 编辑者 |
| **超时时间** | 60秒 |

---

## 相关节点

- **Dataset Input Node** - 从 Google Sheets 读取数据
- **Code Node** - 格式化要写入的数据
- **AI Model Node** - 生成分析结果
- **Swap Node** - 生成交易记录

---

**相关文档：**
- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [Dataset Input Node](dataset-input-node.md) - 数据读取节点
- [Code Node](code-node.md) - 数据处理节点

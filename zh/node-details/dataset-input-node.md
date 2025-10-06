# Dataset Input Node

Dataset Input Node 是专门用于从外部数据源读取数据的节点，是 Dataset Node 的实例节点。节点支持从 Google Sheets 读取数据，并将数据传递给下游节点进行处理。

---

## 节点信息

| 属性 | 值 |
|------|-----|
| **节点类型** | `dataset_input_node` |
| **显示名称** | Dataset Input |
| **节点分类** | Input（数据输入） |
| **图标** | 📥 数据库图标 |
| **句柄颜色** | Emerald（绿色） |
| **节点类别** | Instance Node（实例节点） |
| **基类节点** | `dataset_node` |

---

## 功能说明

Dataset Input Node 从 Google Sheets 读取数据，并将数据转换为标准格式供下游节点使用。节点自动处理表头、数据类型转换和格式化。

**主要用途：**
- 从 Google Sheets 读取数据
- 导入历史数据用于分析
- 加载配置数据
- 读取交易记录
- 获取外部数据集

**核心特性：**
- 📊 **Google Sheets 集成**：直接读取 Google Sheets 数据
- 🔄 **自动格式化**：将表格数据转换为 JSON 格式
- 📋 **表头支持**：自动识别和处理表头行
- 🎯 **范围指定**：支持读取特定范围的数据
- 🔗 **URL 支持**：支持完整 URL 或 Sheet ID

---

## 与 Dataset Node 的关系

Dataset Input Node 是 Dataset Node 的**实例节点**，专门用于数据读取：

```
DatasetNode (基类)
    └─ 支持 read/write/append 三种模式
        ↓ 实例化（强制 mode="read"）
DatasetInputNode (实例)
    └─ 仅支持 read 模式，简化参数
```

---

## 输入参数

### 参数列表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `doc_link` | text | ✅ | - | Google Sheets URL 或 ID |
| `worksheet_name` | text | ❌ | `Sheet1` | 工作表名称 |
| `range` | text | ❌ | `A1:Z1000` | 数据范围 |
| `header_row` | boolean | ❌ | `true` | 是否包含表头行 |

### doc_link 参数

**支持的格式：**

1. **完整 URL：**
```
https://docs.google.com/spreadsheets/d/1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM/edit?usp=sharing
```

2. **Sheet ID：**
```
1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM
```

3. **简化 URL：**
```
docs.google.com/spreadsheets/d/1uQvzsNIkaan67wFijnDiixusw9qNZnedW4Q3dWJASBM
```

**ID 提取：**
- 节点自动从 URL 中提取 Sheet ID
- 无需手动复制 ID

### worksheet_name 参数

**说明：** Google Sheets 中的工作表名称

**示例：**
- `Sheet1`（默认）
- `Data`
- `Trading Records`
- `配置表`

### range 参数

**格式：** A1 notation（如 `A1:Z1000`）

**示例：**
- `A1:Z1000` - 读取前1000行，A到Z列
- `A1:C100` - 读取前100行，A到C列
- `B2:E50` - 从B2开始到E50
- `A:C` - 读取所有行，A到C列

### header_row 参数

**说明：** 第一行是否为表头

**效果：**
- `true`：第一行作为列名，数据从第二行开始
- `false`：所有行都作为数据，列名使用默认编号

---

## 输出参数

### 输出列表

| 输出 ID | 显示名称 | 数据类型 | 说明 |
|---------|---------|---------|------|
| `data` | Data | object | 读取的数据 |

### data 输出

**数据类型：** `object`

**数据结构（header_row=true）：**

```typescript
{
  headers: string[];     // 列名数组
  rows: Array<any[]>;    // 数据行数组
  row_count: number;     // 数据行数
  column_count: number;  // 列数
}
```

**示例输出：**

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

**数据结构（header_row=false）：**

```json
{
  "headers": ["Column_0", "Column_1", "Column_2", "Column_3"],
  "rows": [
    ["Date", "Symbol", "Price", "Volume"],
    ["2024-01-15", "BTC", "45000", "1000"]
  ],
  "row_count": 2,
  "column_count": 4
}
```

---

## 使用示例

### 示例 1：读取价格数据

**场景：** 从 Google Sheets 读取历史价格数据进行分析。

**Google Sheets 内容：**
```
| Date       | Symbol | Price | Volume |
|------------|--------|-------|--------|
| 2024-01-15 | BTC    | 45000 | 1000   |
| 2024-01-16 | ETH    | 2500  | 5000   |
```

**工作流结构：**
```
Dataset Input Node (读取价格数据)
    ↓ data
Code Node (计算均价和趋势)
    ↓ analysis
AI Model Node (生成交易建议)
    ↓ recommendation
Buy/Sell Node (执行交易)
```

**Dataset Input Node 配置：**
```json
{
  "doc_link": "https://docs.google.com/spreadsheets/d/1uQvzsNIkaan.../edit",
  "worksheet_name": "Price Data",
  "header_row": true
}
```

---

### 示例 2：加载配置数据

**场景：** 从 Google Sheets 读取交易策略配置。

**Google Sheets 内容：**
```
| Strategy   | Buy_Threshold | Sell_Threshold | Max_Position |
|------------|---------------|----------------|--------------|
| Conservative | 0.02        | 0.05           | 10000        |
| Aggressive   | 0.05        | 0.10           | 50000        |
```

**工作流结构：**
```
Dataset Input Node (读取策略配置)
    ↓ data { strategies }
Code Node (选择当前策略)
    ↓ active_strategy
Condition Node (根据策略参数决策)
    ↓
交易节点
```

---

### 示例 3：回测历史数据

**场景：** 读取历史交易记录进行回测分析。

**工作流结构：**
```
Dataset Input Node (历史交易记录)
    ↓ data
Code Node (计算收益率)
    ↓ returns { total_return, sharpe_ratio }
AI Model Node (分析改进点)
    ↓ insights
Dataset Output Node (保存分析结果)
```

---

## Google Sheets 配置

### 1. 创建 Service Account

1. 访问 [Google Cloud Console](https://console.cloud.google.com/)
2. 创建新项目或选择现有项目
3. 启用 **Google Sheets API**
4. 创建 **Service Account**
5. 下载 JSON 凭证文件

### 2. 共享 Google Sheets

1. 打开您的 Google Sheets
2. 点击右上角的「共享」按钮
3. 添加 Service Account 邮箱地址
4. 授予「查看者」或「编辑者」权限

### 3. 配置凭证

**方法1 - 环境变量（推荐）：**
```bash
export GOOGLE_CREDENTIALS_PATH="/path/to/credentials.json"
```

**方法2 - 配置文件：**
```python
# config.py
CONFIG = {
    "GOOGLE_CREDENTIALS_PATH": "credentials/google_sheets_credentials.json"
}
```

---

## 最佳实践

### 1. 表头设计

**✅ 推荐：**
```
| date       | symbol | price  | volume  |  # 小写，简洁
| Date       | Symbol | Price  | Volume  |  # 首字母大写
```

**❌ 避免：**
```
| 日期 | 代币 | 价格 | 交易量 |  # 中文列名可能导致问题
| Date (YYYY-MM-DD) | ... |  # 过于冗长
```

### 2. 数据范围

**根据数据量选择范围：**
- 小数据集（<100行）：`A1:Z1000`
- 中等数据集（100-1000行）：`A1:Z5000`
- 大数据集（>1000行）：考虑分批读取

### 3. 数据格式

**在 Google Sheets 中：**
- 日期：使用 ISO 格式 `YYYY-MM-DD`
- 数字：避免使用逗号分隔符
- 布尔值：使用 `TRUE`/`FALSE` 或 `1`/`0`
- 空值：留空，不要使用 `null` 或 `N/A`

---

## 注意事项

### ⚠️ 重要提示

1. **Google API 配额**
   - 每天有读取次数限制
   - 避免在循环中频繁读取
   - 考虑缓存数据

2. **凭证文件安全**
   - 不要将凭证文件提交到 Git
   - 使用环境变量配置路径
   - 限制 Service Account 权限

3. **数据类型**
   - 所有数据读取为字符串
   - 需要在 Code Node 中进行类型转换
   - 数字、日期等需要显式转换

4. **表头行**
   - 确保 `header_row` 设置正确
   - 表头行不会出现在 `rows` 数组中
   - 表头将用作列名

5. **工作表名称**
   - 区分大小写
   - 包含空格需要完整匹配
   - 默认读取 `Sheet1`

---

## 故障排查

**Q: 提示 "Google Sheets credentials file not found"？**

A:
1. 确认凭证文件路径正确
2. 检查环境变量 `GOOGLE_CREDENTIALS_PATH`
3. 确保凭证文件可读

---

**Q: 提示 "Spreadsheet not found"？**

A:
1. 确认 Sheet ID 正确
2. 检查 Service Account 是否有访问权限
3. 确认 Google Sheets 已共享

---

**Q: 读取的数据为空？**

A:
1. 检查 `range` 参数是否正确
2. 确认工作表中有数据
3. 检查 `worksheet_name` 是否匹配
4. 确认数据在指定范围内

---

**Q: 列名显示为 Column_0, Column_1？**

A:
1. 设置 `header_row: true`
2. 确认第一行包含列名
3. 检查表格格式

---

## 技术规格

| 规格项 | 值 |
|--------|-----|
| **节点版本** | 0.0.2 |
| **节点类别** | Instance Node |
| **基类节点** | `dataset_node` |
| **数据源** | Google Sheets |
| **API** | Google Sheets API v4 |
| **凭证类型** | Service Account |
| **超时时间** | 60秒 |

---

## 相关节点

- **Dataset Output Node** - 写入数据到 Google Sheets
- **Code Node** - 处理读取的数据
- **AI Model Node** - 分析数据集
- **Condition Node** - 基于数据做决策

---

**相关文档：**
- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [Code Node](code-node.md) - 数据处理节点
- [Weather 语法](../core-concepts/weather-syntax.md) - 工作流文件格式

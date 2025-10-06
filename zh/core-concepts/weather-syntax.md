# Weather 语法（Full 版本）

TradingFlow Weather 语言定义了工作流文件（.wt 文件）的完整结构和规范。Weather 文件本质上是一个 JSON 格式的超集，包含了工作流的所有信息：元数据、节点配置、连接关系等。

> **📌 版本说明：** 本文档描述的是 **Full 版本**，包含前端展示所需的所有字段。Agent 生成和后端存储使用的是 **Essential 版本**，详见 [Weather 语法对比](weather-syntax-comparison.md)。

## .wt 文件结构

一个完整的 Weather 工作流文件由三个顶层字段组成：

```json
{
    "thumbnailUrl": "...",
    "name": "...",
    "nodes": [...],
    "edges": [...]
}
```

下面我们将详细解释每个字段的含义和作用。

---

## 工作流元数据

### thumbnailUrl

工作流的缩略图，使用 Base64 编码的图片数据。这个缩略图会在工作流列表和预览界面中显示。

```json
"thumbnailUrl": "data:image/png;base64,iVBORw0KGgoAAAANS..."
```

**格式说明：**
- 数据 URI 格式：`data:image/png;base64,<encoded-data>`
- 支持的图片格式：PNG、JPEG、WebP
- 建议尺寸：800x600 像素
- 自动在用户保存工作流时生成

### name

工作流的名称，用于标识和区分不同的工作流。

```json
"name": "New Flow"
```

**命名规则：**
- 支持中英文字符
- 长度限制：1-100 字符
- 允许使用空格和特殊字符
- 默认名称为 "New Flow" 或 "新工作流"

---

## 节点定义（nodes）

`nodes` 数组包含了工作流中所有节点的完整定义。每个节点是一个独立的功能单元，负责特定的数据处理任务。

### 节点基础结构

```json
{
    "position": {
        "x": 100,
        "y": 100
    },
    "id": "x_listener_node_1",
    "type": "x_listener_node",
    "className": "",
    "data": {...},
    "width": 343,
    "height": 464,
    "selected": false
}
```

#### position

节点在画布上的位置坐标，用于可视化渲染。

```json
"position": {
    "x": 100,  // X 轴坐标（像素）
    "y": 100   // Y 轴坐标（像素）
}
```

- 坐标原点在画布左上角
- 用户拖拽节点时会更新此值
- 保存时会记录节点的最终位置

#### id

节点的唯一标识符，用于在整个工作流中引用该节点。

```json
"id": "x_listener_node_1"
```

**命名规范：**
- 格式：`<node_type>_<index>`
- 必须在工作流中唯一
- 创建节点时自动生成
- 用于边（edges）的连接引用

#### type

节点的类型标识，决定节点的功能和行为。

```json
"type": "x_listener_node"
```

**常见节点类型：**
- `x_listener_node` - X（Twitter）监听器节点
- `ai_model_node` - AI 模型节点
- `buy_node` - 买入交易节点
- `sell_node` - 卖出交易节点
- `code_node` - 代码执行节点
- `binance_price_node` - 币安价格节点
- `dataset_output_node` - 数据输出节点

#### width & height

节点在画布上的显示尺寸（像素）。

```json
"width": 343,
"height": 464
```

- 根据节点内容自动计算
- 不同类型节点有不同的默认尺寸
- 用户可以手动调整大小

#### selected

标识节点当前是否被选中，用于 UI 交互状态。

```json
"selected": false
```

- `true` - 节点被选中（高亮显示）
- `false` - 节点未选中（正常显示）

---

### 节点数据（data）

`data` 对象包含了节点的核心配置信息和业务逻辑。

```json
"data": {
    "title": "Trump Twitter Listener",
    "description": "Monitors tweets from @realDonaldTrump",
    "collection": "input",
    "inputs": [...],
    "outputs": [...],
    "id": "x_listener_node_1",
    "edges": [...],
    "menuItems": [...],
    "isDeepEdit": true,
    "isFlowExecuting": false,
    "isStopping": false,
    "signals": []
}
```

#### title & description

节点的显示标题和描述信息。

```json
"title": "Trump Twitter Listener",
"description": "Monitors tweets from @realDonaldTrump"
```

- `title` - 节点卡片顶部显示的名称
- `description` - 节点功能的简短说明
- 用户可自定义编辑

#### collection

节点所属的功能分类。

```json
"collection": "input"
```

**分类类型：**
- `input` - 数据输入节点（如监听器、数据源）
- `compute` - 计算处理节点（如 AI 模型、代码执行）
- `trade` - 交易操作节点（如买入、卖出）
- `core` - 核心功能节点（如数据输出）

#### inputs

节点的输入参数配置数组，定义了节点接收的数据和参数。

```json
"inputs": [
    {
        "id": "accounts",
        "title": "Accounts",
        "type": "paragraph",
        "inputType": "text",
        "required": true,
        "placeholder": "Enter X accounts to monitor (comma separated)",
        "handle": {
            "color": "sky"
        },
        "value": ["realDonaldTrump"],
        "options": []
    }
]
```

**输入字段说明：**

- `id` - 输入参数的标识符
- `title` - 参数显示名称
- `type` - 数据类型（`paragraph`, `text`, `number`, `select`, `searchSelect`, `button`, `object`, `paramMatrix`）
- `inputType` - 输入控件类型
- `required` - 是否必填
- `placeholder` - 输入提示文本
- `handle` - 连接句柄配置（颜色、样式）
- `value` - 参数当前值
- `options` - 可选项列表（用于下拉选择）

**handle 颜色约定：**
- `sky` - 数据流类型（蓝色）
- `emerald` - 交易类型（绿色）
- `amber` - 配置类型（橙色）
- `rose` - 输出类型（粉色）

#### outputs

节点的输出定义数组，指定节点产生的数据输出。

```json
"outputs": [
    {
        "id": "latest_tweets",
        "title": "Latest Tweets",
        "type": "object",
        "handle": {
            "color": "sky"
        },
        "isDeleted": false
    }
]
```

**输出字段说明：**

- `id` - 输出的标识符
- `title` - 输出显示名称
- `type` - 输出数据类型（`text`, `number`, `object`, `array`）
- `handle` - 连接句柄配置
- `isDeleted` - 标记输出是否已删除

#### edges

节点所参与的所有连接关系。这是一个包含完整边信息的数组，用于节点间的数据流管理。

```json
"edges": [
    {
        "id": "edge-1759136854519-ra1cx77x1",
        "type": "default",
        "source": "x_listener_node_1",
        "target": "ai_model_node_1",
        "sourceHandle": "latest_tweets",
        "targetHandle": "prompt"
    }
]
```

*注意：这里的 edges 数组是节点级别的引用，完整的边定义在顶层的 `edges` 字段中。*

#### 节点状态标志

```json
"isDeepEdit": true,
"isFlowExecuting": false,
"isStopping": false,
"signals": []
```

- `isDeepEdit` - 节点是否处于深度编辑模式
- `isFlowExecuting` - 工作流是否正在执行
- `isStopping` - 工作流是否正在停止
- `signals` - 节点运行时接收的信号数据

---

## 边定义（edges）

`edges` 数组定义了节点之间的连接关系，描述了数据在工作流中的流动路径。

### 边的基础结构

```json
{
    "id": "edge_ai_model_node_1_ai_response-handle_to_buy_node_1_buy_token-handle",
    "animated": true,
    "source": "ai_model_node_1",
    "target": "buy_node_1",
    "sourceHandle": "ai_response",
    "targetHandle": "buy_token"
}
```

#### id

边的唯一标识符，用于引用和管理连接。

```json
"id": "edge_ai_model_node_1_ai_response-handle_to_buy_node_1_buy_token-handle"
```

**命名格式：**
- 模式：`edge_<source_node>_<source_handle>_to_<target_node>_<target_handle>`
- 或简化格式：`edge-<timestamp>-<random>`
- 必须全局唯一

#### type & animated

边的显示样式配置。

```json
"type": "default",
"animated": true
```

- `type` - 边的类型（`default`, `step`, `smoothstep`, `straight`）
- `animated` - 是否显示流动动画（true 表示数据正在传输）

#### source & target

边连接的源节点和目标节点。

```json
"source": "ai_model_node_1",
"target": "buy_node_1"
```

- `source` - 数据输出节点的 ID
- `target` - 数据接收节点的 ID
- 必须引用存在的节点 ID

#### sourceHandle & targetHandle

边连接的具体输入输出端口。

```json
"sourceHandle": "ai_response",
"targetHandle": "buy_token"
```

- `sourceHandle` - 源节点的输出参数 ID
- `targetHandle` - 目标节点的输入参数 ID
- 必须匹配节点定义中的 input/output ID
- **不包含 `-handle` 后缀**（这是 Weather 语言规范）

---

## 完整示例解读

让我们通过一个完整的工作流示例来理解 Weather 语法的实际应用：

### 工作流概述

这是一个基于 Trump 推文的自动交易策略：

1. **X Listener 节点** - 监听 @realDonaldTrump 的推文
2. **AI Model 节点** - 分析推文内容，判断是否有买入信号
3. **Buy 节点** - 根据 AI 分析结果执行买入操作
4. **Binance Price 节点** - 监控 BTC/USDT 价格
5. **Code 节点** - 计算 5 倍买入价格作为卖出目标
6. **Sell 节点** - 价格达到 5 倍时执行卖出
7. **Dataset Output 节点** - 记录所有交易到 Google Sheets

### 数据流向

```
推文监听 → AI 分析 → 买入交易 → 交易记录
             ↓
价格监控 → 价格计算 → 卖出交易 → 交易记录
```

### 关键特性

**多源输入：**
- Dataset Output 节点接收来自 Buy 和 Sell 节点的交易记录
- 通过两条独立的边实现数据汇聚

**条件分支：**
- AI 分析结果决定是否触发买入
- 价格计算结果决定是否触发卖出

**参数传递：**
- AI 输出直接连接到 Buy 节点的 `buy_token` 输入
- Code 输出连接到 Sell 节点的 `sell_token` 和 `amount_in_human_readable` 输入

---

## 语法验证规则

### 必填字段验证

所有节点必须包含以下字段：
- `id`, `type`, `position`, `data`

所有边必须包含以下字段：
- `id`, `source`, `target`, `sourceHandle`, `targetHandle`

### 引用完整性

- 边的 `source` 和 `target` 必须引用存在的节点 ID
- `sourceHandle` 必须存在于源节点的 `outputs` 中
- `targetHandle` 必须存在于目标节点的 `inputs` 中

### 类型兼容性

- 边连接的输出类型和输入类型必须兼容
- 例如：`object` 类型输出可以连接到 `object` 类型输入

### 唯一性约束

- 节点 ID 必须全局唯一
- 边 ID 必须全局唯一

---

## 最佳实践

### 节点命名

使用有意义的 title 和 description，便于理解工作流逻辑：

```json
"title": "Trump Twitter Listener",
"description": "Monitors tweets from @realDonaldTrump"
```

### 参数配置

为必填参数设置合理的默认值和提示信息：

```json
"required": true,
"placeholder": "Enter X accounts to monitor (comma separated)",
"value": ["realDonaldTrump"]
```

### 边的组织

使用描述性的边 ID，方便调试和维护：

```json
"id": "edge_ai_model_node_1_ai_response_to_buy_node_1_buy_token"
```

### 状态管理

合理使用 `animated` 标志表示数据流动状态：

```json
"animated": true  // 表示这条边上有活跃的数据传输
```

---

## 扩展与兼容性

Weather 语言设计为可扩展的格式：

**向后兼容：**
- 新增字段不会破坏旧版本的工作流
- 未知字段会被安全忽略

**自定义扩展：**
- 可以在 `data` 对象中添加自定义字段
- 节点开发者可以定义专有的配置参数

**版本管理：**
- 通过添加 `version` 字段支持多版本格式
- 平台会自动迁移旧版本工作流

---

## 常见问题

**Q: 为什么句柄名称不包含 `-handle` 后缀？**

A: 这是 Weather 语言的规范设计。在早期版本中使用了 `-handle` 后缀，但为了简化语法和提高可读性，新版本已经移除。前端会自动在需要时添加后缀。

**Q: 如何处理节点的可选输入？**

A: 将输入的 `required` 字段设置为 `false`，并提供合理的默认值或留空。

**Q: 边的 `type` 字段有什么作用？**

A: 决定边在画布上的渲染样式。`default` 是标准曲线，`straight` 是直线，`step` 是阶梯线。

**Q: `positionAbsolute` 字段的用途？**

A: 这是计算出的绝对位置，考虑了画布缩放和平移。通常由前端自动维护。

---

**下一步：** 查看 [节点详情](../node-details/index.md) 了解每个节点类型的详细参数和用法。

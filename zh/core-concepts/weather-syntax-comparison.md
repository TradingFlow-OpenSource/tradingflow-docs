# Weather 语法版本对比

本文档清晰地对比 Weather 语法的两个版本，帮助你理解哪些字段用于前端展示，哪些字段用于后端存储。

---

## 版本概述

TradingFlow Weather 语法有两个版本：

### 🖥️ Full 版本（完整版）
- **用途：** 前端展示和用户交互
- **包含：** 所有字段，包括 UI 状态、样式配置、运行时数据
- **文件大小：** 较大，包含冗余信息
- **使用场景：** 前端 ReactFlow 渲染、用户编辑、实时状态展示

### 💾 Essential 版本（核心版）
- **用途：** Agent 生成和后端存储
- **包含：** 仅核心业务逻辑字段
- **文件大小：** 精简，去除前端特定字段
- **使用场景：** 数据库存储、API 传输、Agent 生成工作流

---

## 字段对比总览

### 📊 对比统计

| 层级 | Full 版本字段数 | Essential 版本字段数 | 保留率 |
|------|----------------|---------------------|--------|
| 顶层 | 4 | 4 | 100% |
| 节点基础 | 8-10 | 4 | 40% |
| 节点 data | 11 | 6 | 55% |
| 输入/输出 | 9-10 | 8-9 | 90% |
| 边 | 7 | 5 | 71% |

---

## 详细字段对比

### 1️⃣ 顶层结构

| 字段 | Full | Essential | 说明 |
|------|------|-----------|------|
| `thumbnailUrl` | ✅ | ✅ | 工作流缩略图（可选） |
| `name` | ✅ | ✅ | 工作流名称 |
| `nodes` | ✅ | ✅ | 节点数组 |
| `edges` | ✅ | ✅ | 边数组 |

**结论：** 顶层字段全部保留，无变化。

---

### 2️⃣ 节点基础字段

| 字段 | Full | Essential | 类型 | 说明 |
|------|------|-----------|------|------|
| `position` | ✅ | ✅ | 核心 | 节点位置坐标 |
| `id` | ✅ | ✅ | 核心 | 节点唯一标识 |
| `type` | ✅ | ✅ | 核心 | 节点类型 |
| `data` | ✅ | ✅ | 核心 | 节点数据对象 |
| `className` | ✅ | ❌ | 前端 | CSS 类名 |
| `width` | ✅ | ❌ | 前端 | 节点宽度 |
| `height` | ✅ | ❌ | 前端 | 节点高度 |
| `selected` | ✅ | ❌ | 前端 | 选中状态 |
| `positionAbsolute` | ✅ | ❌ | 前端 | 绝对位置（计算值） |
| `dragging` | ✅ | ❌ | 前端 | 拖拽状态 |

**保留字段：** `position`, `id`, `type`, `data`

**去除字段：** `className`, `width`, `height`, `selected`, `positionAbsolute`, `dragging`

**原因：** 前端会根据节点内容自动计算尺寸和管理交互状态，无需存储。

---

### 3️⃣ 节点 data 对象

| 字段 | Full | Essential | 类型 | 说明 |
|------|------|-----------|------|------|
| `title` | ✅ | ✅ | 核心 | 节点标题 |
| `description` | ✅ | ✅ | 核心 | 节点描述 |
| `collection` | ✅ | ✅ | 核心 | 节点分类 |
| `inputs` | ✅ | ✅ | 核心 | 输入参数数组 |
| `outputs` | ✅ | ✅ | 核心 | 输出定义数组 |
| `id` | ✅ | ✅ | 核心 | 节点 ID（冗余） |
| `edges` | ✅ | ❌ | 冗余 | 节点相关的边（顶层已有） |
| `menuItems` | ✅ | ❌ | 前端 | 右键菜单项 |
| `isDeepEdit` | ✅ | ❌ | 前端 | 深度编辑模式 |
| `isFlowExecuting` | ✅ | ❌ | 前端 | 执行状态 |
| `isStopping` | ✅ | ❌ | 前端 | 停止状态 |
| `signals` | ✅ | ❌ | 运行时 | 运行时信号数据 |

**保留字段：** `title`, `description`, `collection`, `inputs`, `outputs`, `id`

**去除字段：** `edges`, `menuItems`, `isDeepEdit`, `isFlowExecuting`, `isStopping`, `signals`

**原因：**
- `edges` - 顶层已有完整定义，节点内的是冗余引用
- `menuItems` - 前端根据节点类型动态生成
- `isDeepEdit`, `isFlowExecuting`, `isStopping` - UI 状态，不属于工作流定义
- `signals` - 运行时数据，不应持久化

---

### 4️⃣ 输入/输出字段

| 字段 | Full | Essential | 类型 | 说明 |
|------|------|-----------|------|------|
| `id` | ✅ | ✅ | 核心 | 参数标识符 |
| `title` | ✅ | ✅ | 核心 | 参数显示名 |
| `type` | ✅ | ✅ | 核心 | 数据类型 |
| `inputType` | ✅ | ✅ | 核心 | 输入控件类型 |
| `required` | ✅ | ✅ | 核心 | 是否必填 |
| `placeholder` | ✅ | ✅ | 核心 | 提示文本 |
| `handle` | ✅ | ✅ | 核心 | 句柄配置 |
| `value` | ✅ | ✅ | 核心 | 参数值 |
| `options` | ✅ | ✅ | 核心 | 选项列表 |
| `min` | ✅ | ✅ | 验证 | 最小值（可选） |
| `max` | ✅ | ✅ | 验证 | 最大值（可选） |
| `description` | ✅ | ✅ | 说明 | 详细描述（输出） |
| `isDeleted` | ✅ | ❌ | 前端 | 删除标记 |
| `disabled` | ✅ | ❌ | 前端 | 禁用状态 |

**保留字段：** `id`, `title`, `type`, `inputType`, `required`, `placeholder`, `handle`, `value`, `options`, `min`, `max`, `description`

**去除字段：** `isDeleted`, `disabled`

**原因：** `isDeleted` 和 `disabled` 是前端临时状态，真正删除的字段应该直接从数组中移除。

---

### 5️⃣ 边字段

| 字段 | Full | Essential | 类型 | 说明 |
|------|------|-----------|------|------|
| `id` | ✅ | ✅ | 核心 | 边唯一标识 |
| `source` | ✅ | ✅ | 核心 | 源节点 ID |
| `target` | ✅ | ✅ | 核心 | 目标节点 ID |
| `sourceHandle` | ✅ | ✅ | 核心 | 源句柄 |
| `targetHandle` | ✅ | ✅ | 核心 | 目标句柄 |
| `type` | ✅ | ❌ | 前端 | 边类型（样式） |
| `animated` | ✅ | ❌ | 前端 | 动画效果 |

**保留字段：** `id`, `source`, `target`, `sourceHandle`, `targetHandle`

**去除字段：** `type`, `animated`

**原因：** `type` 和 `animated` 仅控制前端渲染样式和动画，不影响数据流逻辑。

---

## 实际对比示例

### Full 版本示例（节点）

```json
{
    "position": {"x": 100, "y": 100},
    "positionAbsolute": {"x": 100, "y": 100},
    "id": "x_listener_node_1",
    "type": "x_listener_node",
    "className": "",
    "data": {
        "title": "Trump Twitter Listener",
        "description": "Monitors tweets",
        "collection": "input",
        "inputs": [...],
        "outputs": [...],
        "id": "x_listener_node_1",
        "edges": [...],
        "menuItems": [
            {"key": "delete", "label": "Delete", "danger": true}
        ],
        "isDeepEdit": true,
        "isFlowExecuting": false,
        "isStopping": false,
        "signals": []
    },
    "width": 343,
    "height": 464,
    "selected": false,
    "dragging": false
}
```

### Essential 版本示例（节点）

```json
{
    "position": {"x": 100, "y": 100},
    "id": "x_listener_node_1",
    "type": "x_listener_node",
    "data": {
        "title": "Trump Twitter Listener",
        "description": "Monitors tweets",
        "collection": "input",
        "inputs": [...],
        "outputs": [...]
    }
}
```

**减少：** 从 ~20 个字段减少到 ~7 个字段，文件大小减少约 60%。

---

### Full 版本示例（边）

```json
{
    "id": "edge_ai_model_node_1_ai_response_to_buy_node_1_buy_token",
    "type": "default",
    "animated": true,
    "source": "ai_model_node_1",
    "target": "buy_node_1",
    "sourceHandle": "ai_response",
    "targetHandle": "buy_token"
}
```

### Essential 版本示例（边）

```json
{
    "id": "edge_ai_model_node_1_ai_response_to_buy_node_1_buy_token",
    "source": "ai_model_node_1",
    "target": "buy_node_1",
    "sourceHandle": "ai_response",
    "targetHandle": "buy_token"
}
```

**减少：** 从 7 个字段减少到 5 个字段。

---

## 转换规则

### Full → Essential 转换

后端在存储前会自动执行以下清理：

```typescript
function toEssential(fullFlow: FullFlow): EssentialFlow {
    return {
        thumbnailUrl: fullFlow.thumbnailUrl,
        name: fullFlow.name,
        nodes: fullFlow.nodes.map(node => ({
            position: node.position,
            id: node.id,
            type: node.type,
            data: {
                title: node.data.title,
                description: node.data.description,
                collection: node.data.collection,
                inputs: node.data.inputs.map(cleanInput),
                outputs: node.data.outputs.map(cleanOutput)
            }
        })),
        edges: fullFlow.edges.map(edge => ({
            id: edge.id,
            source: edge.source,
            target: edge.target,
            sourceHandle: edge.sourceHandle,
            targetHandle: edge.targetHandle
        }))
    };
}
```

### Essential → Full 转换

前端加载后会自动补充前端字段：

```typescript
function toFull(essentialFlow: EssentialFlow): FullFlow {
    return {
        ...essentialFlow,
        nodes: essentialFlow.nodes.map(node => ({
            ...node,
            className: "",
            width: calculateWidth(node),
            height: calculateHeight(node),
            selected: false,
            data: {
                ...node.data,
                id: node.id,
                edges: [], // 从顶层 edges 填充
                menuItems: getDefaultMenuItems(),
                isDeepEdit: false,
                isFlowExecuting: false,
                isStopping: false,
                signals: []
            }
        })),
        edges: essentialFlow.edges.map(edge => ({
            ...edge,
            type: "default",
            animated: false
        }))
    };
}
```

---

## 特殊说明

### 1. edges 字段的特殊处理

**节点内的 `data.edges` 数组：**
- Full 版本：包含该节点相关的所有边的完整信息（冗余）
- Essential 版本：**完全移除**，因为顶层 `edges` 数组已包含所有信息

前端会根据顶层 `edges` 自动填充每个节点的 `data.edges`。

### 2. 可选字段的处理

某些字段在 Essential 版本中是可选的：
- `thumbnailUrl` - 如果没有则前端自动生成
- `placeholder` - 如果没有则显示默认提示
- `options` - 如果是空数组可以省略

### 3. 向后兼容

如果 Essential 版本中包含了前端字段（如 `width`、`selected`），前端会：
- 优先使用 Essential 版本中的值
- 如果缺失则使用默认值或计算值

---

## 最佳实践

### Agent 生成时

✅ **推荐：**
```json
{
    "name": "Trump Trading Strategy",
    "nodes": [
        {
            "position": {"x": 100, "y": 100},
            "id": "x_listener_node_1",
            "type": "x_listener_node",
            "data": {
                "title": "Twitter Listener",
                "description": "Monitors tweets",
                "collection": "input",
                "inputs": [...],
                "outputs": [...]
            }
        }
    ],
    "edges": [...]
}
```

❌ **避免：**
```json
{
    "name": "Trump Trading Strategy",
    "nodes": [
        {
            "position": {"x": 100, "y": 100},
            "id": "x_listener_node_1",
            "type": "x_listener_node",
            "width": 343,  // ❌ 不需要
            "selected": false,  // ❌ 不需要
            "data": {
                "title": "Twitter Listener",
                "isDeepEdit": true,  // ❌ 不需要
                "signals": []  // ❌ 不需要
            }
        }
    ]
}
```

### 后端存储时

- 始终存储 Essential 版本
- 在 API 响应时保持 Essential 格式
- 让前端负责补充 UI 相关字段

### 前端处理时

- 从后端加载 Essential 版本
- 自动转换为 Full 版本用于渲染
- 保存时转换回 Essential 版本

---

## 文件大小对比

以一个包含 7 个节点、7 条边的工作流为例：

| 版本 | 文件大小 | 压缩后 | 节省 |
|------|----------|--------|------|
| Full | ~25 KB | ~8 KB | - |
| Essential | ~10 KB | ~4 KB | 60% |

**结论：** Essential 版本可以显著减少存储空间和网络传输开销。

---

## 快速参考表

### 需要去除的字段清单

**节点级别：**
- ❌ `className`
- ❌ `width`
- ❌ `height`
- ❌ `selected`
- ❌ `positionAbsolute`
- ❌ `dragging`

**data 级别：**
- ❌ `data.edges`
- ❌ `data.menuItems`
- ❌ `data.isDeepEdit`
- ❌ `data.isFlowExecuting`
- ❌ `data.isStopping`
- ❌ `data.signals`

**输入/输出级别：**
- ❌ `isDeleted`
- ❌ `disabled`

**边级别：**
- ❌ `type`
- ❌ `animated`

---

**相关文档：**
- [Weather 语法（Full 版本）](weather-syntax.md) - 完整字段说明
- [节点与工作流](nodes-and-workflows.md) - 概念介绍

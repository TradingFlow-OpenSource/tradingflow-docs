# Weather Syntax Version Comparison

This document clearly compares the two versions of Weather syntax to help you understand which fields are for frontend display and which are for backend storage.

---

## Version Overview

TradingFlow Weather syntax has two versions:

### 🖥️ Full Version
- **Purpose:** Frontend display and user interaction
- **Contains:** All fields, including UI state, style configuration, runtime data
- **File Size:** Larger, includes redundant information
- **Use Cases:** Frontend ReactFlow rendering, user editing, real-time status display

### 💾 Essential Version
- **Purpose:** Agent generation and backend storage
- **Contains:** Only core business logic fields
- **File Size:** Streamlined, frontend-specific fields removed
- **Use Cases:** Database storage, API transmission, Agent workflow generation

---

## Field Comparison Overview

### 📊 Comparison Statistics

| Level | Full Version Fields | Essential Version Fields | Retention Rate |
|-------|-------------------|------------------------|----------------|
| Top Level | 4 | 4 | 100% |
| Node Basic | 8-10 | 4 | 40% |
| Node data | 11 | 6 | 55% |
| Input/Output | 9-10 | 8-9 | 90% |
| Edges | 7 | 5 | 71% |

---

## Detailed Field Comparison

### 1️⃣ Top Level Structure

| Field | Full | Essential | Description |
|-------|------|-----------|-------------|
| `thumbnailUrl` | ✅ | ✅ | Workflow thumbnail (optional) |
| `name` | ✅ | ✅ | Workflow name |
| `nodes` | ✅ | ✅ | Node array |
| `edges` | ✅ | ✅ | Edge array |

**Conclusion:** All top-level fields are retained, no changes.

---

### 2️⃣ Node Basic Fields

| Field | Full | Essential | Type | Description |
|-------|------|-----------|------|-------------|
| `position` | ✅ | ✅ | Core | Node position coordinates |
| `id` | ✅ | ✅ | Core | Node unique identifier |
| `type` | ✅ | ✅ | Core | Node type |
| `data` | ✅ | ✅ | Core | Node data object |
| `className` | ✅ | ❌ | Frontend | CSS class name |
| `width` | ✅ | ❌ | Frontend | Node width |
| `height` | ✅ | ❌ | Frontend | Node height |
| `selected` | ✅ | ❌ | Frontend | Selection state |
| `positionAbsolute` | ✅ | ❌ | Frontend | Absolute position (calculated) |
| `dragging` | ✅ | ❌ | Frontend | Dragging state |

**Retained Fields:** `position`, `id`, `type`, `data`

**Removed Fields:** `className`, `width`, `height`, `selected`, `positionAbsolute`, `dragging`

**Reason:** Frontend automatically calculates dimensions and manages interaction states based on node content; no need to store.

---

### 3️⃣ Node data Object

| Field | Full | Essential | Type | Description |
|-------|------|-----------|------|-------------|
| `title` | ✅ | ✅ | Core | Node title |
| `description` | ✅ | ✅ | Core | Node description |
| `collection` | ✅ | ✅ | Core | Node category |
| `inputs` | ✅ | ✅ | Core | Input parameter array |
| `outputs` | ✅ | ✅ | Core | Output definition array |
| `id` | ✅ | ✅ | Core | Node ID (redundant) |
| `edges` | ✅ | ❌ | Redundant | Node-related edges (already in top level) |
| `menuItems` | ✅ | ❌ | Frontend | Context menu items |
| `isDeepEdit` | ✅ | ❌ | Frontend | Deep edit mode |
| `isFlowExecuting` | ✅ | ❌ | Frontend | Execution state |
| `isStopping` | ✅ | ❌ | Frontend | Stopping state |
| `signals` | ✅ | ❌ | Runtime | Runtime signal data |

**Retained Fields:** `title`, `description`, `collection`, `inputs`, `outputs`, `id`

**Removed Fields:** `edges`, `menuItems`, `isDeepEdit`, `isFlowExecuting`, `isStopping`, `signals`

**Reasons:**
- `edges` - Complete definition exists at top level, node-level is redundant reference
- `menuItems` - Frontend generates dynamically based on node type
- `isDeepEdit`, `isFlowExecuting`, `isStopping` - UI states, not part of workflow definition
- `signals` - Runtime data, should not be persisted

---

### 4️⃣ Input/Output Fields

| Field | Full | Essential | Type | Description |
|-------|------|-----------|------|-------------|
| `id` | ✅ | ✅ | Core | Parameter identifier |
| `title` | ✅ | ✅ | Core | Parameter display name |
| `type` | ✅ | ✅ | Core | Data type |
| `inputType` | ✅ | ✅ | Core | Input control type |
| `required` | ✅ | ✅ | Core | Whether required |
| `placeholder` | ✅ | ✅ | Core | Placeholder text |
| `handle` | ✅ | ✅ | Core | Handle configuration |
| `value` | ✅ | ✅ | Core | Parameter value |
| `options` | ✅ | ✅ | Core | Options list |
| `min` | ✅ | ✅ | Validation | Minimum value (optional) |
| `max` | ✅ | ✅ | Validation | Maximum value (optional) |
| `description` | ✅ | ✅ | Description | Detailed description (outputs) |
| `isDeleted` | ✅ | ❌ | Frontend | Delete flag |
| `disabled` | ✅ | ❌ | Frontend | Disabled state |

**Retained Fields:** `id`, `title`, `type`, `inputType`, `required`, `placeholder`, `handle`, `value`, `options`, `min`, `max`, `description`

**Removed Fields:** `isDeleted`, `disabled`

**Reason:** `isDeleted` and `disabled` are frontend temporary states; truly deleted fields should be removed directly from the array.

---

### 5️⃣ Edge Fields

| Field | Full | Essential | Type | Description |
|-------|------|-----------|------|-------------|
| `id` | ✅ | ✅ | Core | Edge unique identifier |
| `source` | ✅ | ✅ | Core | Source node ID |
| `target` | ✅ | ✅ | Core | Target node ID |
| `sourceHandle` | ✅ | ✅ | Core | Source handle |
| `targetHandle` | ✅ | ✅ | Core | Target handle |
| `type` | ✅ | ❌ | Frontend | Edge type (style) |
| `animated` | ✅ | ❌ | Frontend | Animation effect |

**Retained Fields:** `id`, `source`, `target`, `sourceHandle`, `targetHandle`

**Removed Fields:** `type`, `animated`

**Reason:** `type` and `animated` only control frontend rendering style and animation, do not affect data flow logic.

---

## Practical Comparison Examples

### Full Version Example (Node)

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

### Essential Version Example (Node)

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

**Reduction:** From ~20 fields to ~7 fields, file size reduced by approximately 60%.

---

### Full Version Example (Edge)

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

### Essential Version Example (Edge)

```json
{
    "id": "edge_ai_model_node_1_ai_response_to_buy_node_1_buy_token",
    "source": "ai_model_node_1",
    "target": "buy_node_1",
    "sourceHandle": "ai_response",
    "targetHandle": "buy_token"
}
```

**Reduction:** From 7 fields to 5 fields.

---

## Conversion Rules

### Full → Essential Conversion

Backend automatically performs the following cleanup before storage:

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

### Essential → Full Conversion

Frontend automatically supplements frontend fields after loading:

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
                edges: [], // Populated from top-level edges
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

## Special Notes

### 1. Special Handling of edges Field

**`data.edges` array within nodes:**
- Full version: Contains complete information of all edges related to this node (redundant)
- Essential version: **Completely removed**, because top-level `edges` array already contains all information

Frontend automatically populates each node's `data.edges` based on top-level `edges`.

### 2. Handling Optional Fields

Some fields are optional in Essential version:
- `thumbnailUrl` - If absent, frontend auto-generates
- `placeholder` - If absent, shows default hint
- `options` - Can be omitted if empty array

### 3. Backward Compatibility

If Essential version contains frontend fields (like `width`, `selected`), frontend will:
- Prioritize values from Essential version
- Use default or calculated values if missing

---

## Best Practices

### When Agent Generates

✅ **Recommended:**
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

❌ **Avoid:**
```json
{
    "name": "Trump Trading Strategy",
    "nodes": [
        {
            "position": {"x": 100, "y": 100},
            "id": "x_listener_node_1",
            "type": "x_listener_node",
            "width": 343,  // ❌ Not needed
            "selected": false,  // ❌ Not needed
            "data": {
                "title": "Twitter Listener",
                "isDeepEdit": true,  // ❌ Not needed
                "signals": []  // ❌ Not needed
            }
        }
    ]
}
```

### When Backend Stores

- Always store Essential version
- Keep Essential format in API responses
- Let frontend handle UI-related field supplementation

### When Frontend Processes

- Load Essential version from backend
- Auto-convert to Full version for rendering
- Convert back to Essential version when saving

---

## File Size Comparison

For a workflow with 7 nodes and 7 edges:

| Version | File Size | Compressed | Savings |
|---------|-----------|------------|---------|
| Full | ~25 KB | ~8 KB | - |
| Essential | ~10 KB | ~4 KB | 60% |

**Conclusion:** Essential version significantly reduces storage space and network transmission overhead.

---

## Quick Reference Table

### Fields to Remove Checklist

**Node Level:**
- ❌ `className`
- ❌ `width`
- ❌ `height`
- ❌ `selected`
- ❌ `positionAbsolute`
- ❌ `dragging`

**data Level:**
- ❌ `data.edges`
- ❌ `data.menuItems`
- ❌ `data.isDeepEdit`
- ❌ `data.isFlowExecuting`
- ❌ `data.isStopping`
- ❌ `data.signals`

**Input/Output Level:**
- ❌ `isDeleted`
- ❌ `disabled`

**Edge Level:**
- ❌ `type`
- ❌ `animated`

---

**Related Documentation:**
- [Weather Syntax (Full Version)](weather-syntax.md) - Complete field documentation
- [Nodes and Workflows](nodes-and-workflows.md) - Concept introduction

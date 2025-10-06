# Weather Syntax (Full Version)

TradingFlow Weather Language defines the complete structure and specification for workflow files (.wt files). A Weather file is essentially a superset of JSON format, containing all information about a workflow: metadata, node configurations, connection relationships, and more.

> **📌 Version Note:** This document describes the **Full Version**, which includes all fields needed for frontend display. The **Essential Version** used by Agent generation and backend storage is documented in [Weather Syntax Comparison](weather-syntax-comparison.md).

## .wt File Structure

A complete Weather workflow file consists of three top-level fields:

```json
{
    "thumbnailUrl": "...",
    "name": "...",
    "nodes": [...],
    "edges": [...]
}
```

Below we will explain each field in detail.

---

## Workflow Metadata

### thumbnailUrl

The thumbnail image of the workflow, using Base64-encoded image data. This thumbnail is displayed in workflow lists and preview interfaces.

```json
"thumbnailUrl": "data:image/png;base64,iVBORw0KGgoAAAANS..."
```

**Format Specifications:**
- Data URI format: `data:image/png;base64,<encoded-data>`
- Supported image formats: PNG, JPEG, WebP
- Recommended size: 800x600 pixels
- Automatically generated when users save workflows

### name

The name of the workflow, used to identify and distinguish different workflows.

```json
"name": "New Flow"
```

**Naming Rules:**
- Supports English and international characters
- Length limit: 1-100 characters
- Spaces and special characters allowed
- Default name is "New Flow"

---

## Node Definitions (nodes)

The `nodes` array contains complete definitions of all nodes in the workflow. Each node is an independent functional unit responsible for specific data processing tasks.

### Node Basic Structure

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

The node's position coordinates on the canvas, used for visual rendering.

```json
"position": {
    "x": 100,  // X-axis coordinate (pixels)
    "y": 100   // Y-axis coordinate (pixels)
}
```

- Origin is at the top-left corner of the canvas
- Updated when users drag nodes
- Final position is saved

#### id

The unique identifier of the node, used to reference the node throughout the workflow.

```json
"id": "x_listener_node_1"
```

**Naming Convention:**
- Format: `<node_type>_<index>`
- Must be unique within the workflow
- Automatically generated when creating nodes
- Used for edge connection references

#### type

The type identifier of the node, determining its functionality and behavior.

```json
"type": "x_listener_node"
```

**Common Node Types:**
- `x_listener_node` - X (Twitter) Listener Node
- `ai_model_node` - AI Model Node
- `buy_node` - Buy Trade Node
- `sell_node` - Sell Trade Node
- `code_node` - Code Execution Node
- `binance_price_node` - Binance Price Node
- `dataset_output_node` - Dataset Output Node

#### width & height

The display dimensions of the node on the canvas (pixels).

```json
"width": 343,
"height": 464
```

- Automatically calculated based on node content
- Different node types have different default sizes
- Users can manually adjust size

#### selected

Indicates whether the node is currently selected, used for UI interaction states.

```json
"selected": false
```

- `true` - Node is selected (highlighted)
- `false` - Node is not selected (normal display)

---

### Node Data

The `data` object contains the core configuration information and business logic of the node.

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

The display title and description information of the node.

```json
"title": "Trump Twitter Listener",
"description": "Monitors tweets from @realDonaldTrump"
```

- `title` - Name displayed at the top of the node card
- `description` - Brief explanation of node functionality
- Customizable by users

#### collection

The functional category the node belongs to.

```json
"collection": "input"
```

**Category Types:**
- `input` - Data input nodes (e.g., listeners, data sources)
- `compute` - Computing nodes (e.g., AI models, code execution)
- `trade` - Trading operation nodes (e.g., buy, sell)
- `core` - Core function nodes (e.g., data output)

#### inputs

The input parameter configuration array, defining the data and parameters the node receives.

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

**Input Field Descriptions:**

- `id` - Identifier of the input parameter
- `title` - Display name of the parameter
- `type` - Data type (`paragraph`, `text`, `number`, `select`, `searchSelect`, `button`, `object`, `paramMatrix`)
- `inputType` - Input control type
- `required` - Whether the field is required
- `placeholder` - Input hint text
- `handle` - Connection handle configuration (color, style)
- `value` - Current value of the parameter
- `options` - List of options (for dropdown selection)

**Handle Color Conventions:**
- `sky` - Data flow type (blue)
- `emerald` - Trading type (green)
- `amber` - Configuration type (orange)
- `rose` - Output type (pink)

#### outputs

The output definition array, specifying the data outputs produced by the node.

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

**Output Field Descriptions:**

- `id` - Identifier of the output
- `title` - Display name of the output
- `type` - Output data type (`text`, `number`, `object`, `array`)
- `handle` - Connection handle configuration
- `isDeleted` - Flag indicating if output is deleted

#### edges

All connection relationships the node participates in. This is an array containing complete edge information, used for data flow management between nodes.

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

*Note: The edges array here is a node-level reference. Complete edge definitions are in the top-level `edges` field.*

#### Node State Flags

```json
"isDeepEdit": true,
"isFlowExecuting": false,
"isStopping": false,
"signals": []
```

- `isDeepEdit` - Whether the node is in deep edit mode
- `isFlowExecuting` - Whether the workflow is executing
- `isStopping` - Whether the workflow is stopping
- `signals` - Signal data received by the node at runtime

---

## Edge Definitions (edges)

The `edges` array defines the connection relationships between nodes, describing the flow paths of data in the workflow.

### Edge Basic Structure

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

The unique identifier of the edge, used for referencing and managing connections.

```json
"id": "edge_ai_model_node_1_ai_response-handle_to_buy_node_1_buy_token-handle"
```

**Naming Format:**
- Pattern: `edge_<source_node>_<source_handle>_to_<target_node>_<target_handle>`
- Or simplified format: `edge-<timestamp>-<random>`
- Must be globally unique

#### type & animated

Display style configuration of the edge.

```json
"type": "default",
"animated": true
```

- `type` - Type of edge (`default`, `step`, `smoothstep`, `straight`)
- `animated` - Whether to display flow animation (true indicates data is being transmitted)

#### source & target

The source and target nodes connected by the edge.

```json
"source": "ai_model_node_1",
"target": "buy_node_1"
```

- `source` - ID of the data output node
- `target` - ID of the data receiving node
- Must reference existing node IDs

#### sourceHandle & targetHandle

The specific input/output ports connected by the edge.

```json
"sourceHandle": "ai_response",
"targetHandle": "buy_token"
```

- `sourceHandle` - Output parameter ID of the source node
- `targetHandle` - Input parameter ID of the target node
- Must match input/output IDs in node definitions
- **Does not include `-handle` suffix** (this is Weather language specification)

---

## Complete Example Walkthrough

Let's understand the practical application of Weather syntax through a complete workflow example:

### Workflow Overview

This is an automated trading strategy based on Trump's tweets:

1. **X Listener Node** - Monitors tweets from @realDonaldTrump
2. **AI Model Node** - Analyzes tweet content to determine buy signals
3. **Buy Node** - Executes buy operations based on AI analysis
4. **Binance Price Node** - Monitors BTC/USDT price
5. **Code Node** - Calculates 5x buy price as sell target
6. **Sell Node** - Executes sell when price reaches 5x
7. **Dataset Output Node** - Records all trades to Google Sheets

### Data Flow

```
Tweet Monitor → AI Analysis → Buy Trade → Trade Record
                  ↓
Price Monitor → Price Calc → Sell Trade → Trade Record
```

### Key Features

**Multiple Input Sources:**
- Dataset Output node receives trade records from both Buy and Sell nodes
- Data convergence through two independent edges

**Conditional Branching:**
- AI analysis result determines whether to trigger buy
- Price calculation result determines whether to trigger sell

**Parameter Passing:**
- AI output directly connects to Buy node's `buy_token` input
- Code output connects to Sell node's `sell_token` and `amount_in_human_readable` inputs

---

## Syntax Validation Rules

### Required Field Validation

All nodes must include the following fields:
- `id`, `type`, `position`, `data`

All edges must include the following fields:
- `id`, `source`, `target`, `sourceHandle`, `targetHandle`

### Reference Integrity

- Edge `source` and `target` must reference existing node IDs
- `sourceHandle` must exist in source node's `outputs`
- `targetHandle` must exist in target node's `inputs`

### Type Compatibility

- Output type and input type connected by an edge must be compatible
- For example: `object` type output can connect to `object` type input

### Uniqueness Constraints

- Node IDs must be globally unique
- Edge IDs must be globally unique

---

## Best Practices

### Node Naming

Use meaningful titles and descriptions to understand workflow logic:

```json
"title": "Trump Twitter Listener",
"description": "Monitors tweets from @realDonaldTrump"
```

### Parameter Configuration

Set reasonable default values and hints for required parameters:

```json
"required": true,
"placeholder": "Enter X accounts to monitor (comma separated)",
"value": ["realDonaldTrump"]
```

### Edge Organization

Use descriptive edge IDs for easier debugging and maintenance:

```json
"id": "edge_ai_model_node_1_ai_response_to_buy_node_1_buy_token"
```

### State Management

Use `animated` flag appropriately to indicate data flow status:

```json
"animated": true  // Indicates active data transmission on this edge
```

---

## Extension and Compatibility

Weather Language is designed as an extensible format:

**Backward Compatibility:**
- New fields won't break workflows from older versions
- Unknown fields are safely ignored

**Custom Extensions:**
- Can add custom fields in the `data` object
- Node developers can define proprietary configuration parameters

**Version Management:**
- Support multiple format versions by adding `version` field
- Platform automatically migrates old version workflows

---

## FAQ

**Q: Why don't handle names include the `-handle` suffix?**

A: This is by design in the Weather language specification. Earlier versions used the `-handle` suffix, but it was removed in newer versions to simplify syntax and improve readability. The frontend automatically adds the suffix when needed.

**Q: How to handle optional inputs for nodes?**

A: Set the input's `required` field to `false` and provide a reasonable default value or leave it empty.

**Q: What is the purpose of the edge `type` field?**

A: It determines the rendering style of the edge on the canvas. `default` is a standard curve, `straight` is a straight line, and `step` is a stepped line.

**Q: What is the purpose of the `positionAbsolute` field?**

A: This is the calculated absolute position, considering canvas zoom and pan. Usually maintained automatically by the frontend.

---

**Next Step:** Check out [Node Details](../node-details/index.md) to learn detailed parameters and usage for each node type.

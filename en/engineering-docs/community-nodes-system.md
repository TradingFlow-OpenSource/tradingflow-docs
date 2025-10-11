# Community Nodes & Version Control System

**Version:** 1.0.0  
**Last Updated:** 2025-01-08  
**Status:** ✅ Production Ready

---

## 📋 Table of Contents

- [Overview](#overview)
- [For Users: How to Participate](#for-users-how-to-participate)
- [System Architecture](#system-architecture)
- [Technical Implementation](#technical-implementation)
- [API Reference](#api-reference)
- [Developer Guide](#developer-guide)
- [Best Practices](#best-practices)

---

## Overview

### What is Community Nodes?

Community Nodes is TradingFlow's **user-contributed node ecosystem** that allows users to create, share, and use custom trading nodes. Think of it as an "App Store" for trading logic.

**Key Features:**
- 🎨 **Create Custom Nodes** - Build nodes with Python code or HTTP APIs
- 🔄 **Version Control** - npm-style semantic versioning (major.minor.patch)
- 🛡️ **Sandboxed Execution** - Safe execution environment for user code
- 💬 **Community Feedback** - Rating, commenting, and trending system
- 🚀 **Easy Integration** - Drag-and-drop nodes into FlowEdit

### Why Community Nodes?

**For Strategy Developers:**
- Extend TradingFlow's capabilities without waiting for official releases
- Share your successful trading logic with others
- Build on top of existing community nodes

**For Node Creators:**
- Earn recognition through stars and downloads
- Help the community with specialized functionality
- Showcase your trading expertise

**For TradingFlow:**
- Rapid ecosystem growth
- User-driven innovation
- Reduced feature request backlog

---

## For Users: How to Participate

### 🌟 Discover Nodes

**1. Browse the Community Node Marketplace**

Navigate to: `/community/nodes`

- **Search**: Find nodes by name or description
- **Filter**: By category (Data, AI, DeFi, Trading, etc.)
- **Sort**: By stars, downloads, newest, or views
- **Explore**: Popular and trending sections

**2. View Node Details**

Click any node card to see:
- Complete description and documentation
- Input/output parameters
- Version history
- User ratings and comments
- Usage statistics

### 🎨 Create Your Own Node

**Step 1: Plan Your Node**

Ask yourself:
- What problem does it solve?
- What inputs does it need?
- What outputs does it provide?
- Python code or HTTP API?

**Step 2: Create the Node**

1. Click **"Create Node"** button
2. Fill in basic information:
   - **Node ID**: Unique identifier (e.g., `my_custom_indicator`)
   - **Display Name**: User-friendly name (e.g., "My Custom Indicator")
   - **Category**: Choose appropriate category
   - **Description**: Clear explanation of functionality

3. Define inputs and outputs:
   - Add input parameters with types (string, number, json, etc.)
   - Add output parameters with types
   - Mark required vs. optional inputs

4. Configure execution:
   - **Python Code**: Write your logic directly
   - **HTTP API**: Provide endpoint URL and configuration

**Step 3: Test and Publish**

1. Save as draft
2. Test thoroughly in your flows
3. Click **"Publish"** when ready
4. Node appears in community marketplace

### 📝 Example: Creating a Moving Average Node

```python
# inputs: price_data (array), period (number)
# outputs: ma_value (number)

import numpy as np

prices = inputs['price_data']
period = inputs['period']

# Calculate moving average
ma = np.mean(prices[-period:])

outputs['ma_value'] = ma
```

### 💬 Engage with Community

**Rate and Comment:**
- Leave ratings (1-5 stars)
- Write helpful comments
- Report issues

**Star Your Favorites:**
- Star nodes you find useful
- Starred nodes appear in trending

**Share Your Knowledge:**
- Answer questions in comments
- Suggest improvements
- Collaborate on node updates

### 🚀 Use Community Nodes in Flows

**Method 1: From Node Details**

1. Browse to node details page
2. Click **"Add to Flow"**
3. Navigate to FlowEdit
4. Find node in "Community" category
5. Drag onto canvas

**Method 2: From FlowEdit Directly**

1. Open any flow in FlowEdit
2. Click **Edit** tool (left sidebar)
3. Expand **"Community"** category
4. Browse available nodes
5. Drag to canvas

---

## System Architecture

### Three-Layer Architecture

```
┌─────────────────────────────────────────────┐
│           Frontend (React/TS)               │
│  - Node Marketplace UI                      │
│  - FlowEdit Integration                     │
│  - User Management                          │
└─────────────────────────────────────────────┘
                    ↓ HTTP/REST
┌─────────────────────────────────────────────┐
│          Control (Node.js/MongoDB)          │
│  - Node Metadata Management                 │
│  - Comment & Rating System                  │
│  - Usage Statistics                         │
└─────────────────────────────────────────────┘
                    ↓ Execution Request
┌─────────────────────────────────────────────┐
│          Station (Python/FastAPI)           │
│  - Version Management                       │
│  - Node Registry                            │
│  - Sandboxed Execution                      │
└─────────────────────────────────────────────┘
```

### Key Components

**1. Version Manager (Station)**
- Semantic versioning (major.minor.patch)
- Version comparison and range matching
- Backward compatibility checking

**2. Node Registry (Station)**
- Multi-version node registration
- Node lookup and retrieval
- Version resolution

**3. Community Node Executor (Station)**
- Safe Python code execution (sandboxed)
- HTTP API calling
- Resource limits and timeout

**4. Node Metadata Service (Control)**
- CRUD operations for node metadata
- Publishing workflow
- Search and filtering

**5. Comment Service (Control)**
- User ratings and reviews
- Reply threads
- Like/dislike tracking

**6. Frontend Integration**
- Node marketplace UI
- FlowEdit drag-and-drop
- Real-time loading and updates

---

## Technical Implementation

### Version Control System

**Format:** `major.minor.patch` (e.g., `1.2.3`)

**Rules:**
- **Major**: Breaking changes (incompatible API)
- **Minor**: New features (backward compatible)
- **Patch**: Bug fixes (backward compatible)

**Version Ranges:**
```python
# Exact version
"1.2.3"

# Compatible with 1.x.x
"^1.2.3"  # >= 1.2.3 and < 2.0.0

# Minor updates only
"~1.2.3"  # >= 1.2.3 and < 1.3.0

# Greater than or equal
">=1.2.3"
```

**Implementation:**
```python
# Station: core/version_manager.py
class VersionManager:
    @staticmethod
    def parse_version(version: str) -> Tuple[int, int, int]:
        """Parse semantic version string"""
        
    @staticmethod
    def compare_versions(v1: str, v2: str) -> int:
        """Compare two versions (-1, 0, 1)"""
        
    @staticmethod
    def match_version_range(version: str, range_spec: str) -> bool:
        """Check if version matches range"""
```

### Node Registry

**Multi-Version Support:**
```python
# Register multiple versions of same node
registry.register_node('my_node', MyNodeV1, version='1.0.0')
registry.register_node('my_node', MyNodeV2, version='2.0.0')

# Get specific version
node = registry.get_node('my_node', '1.0.0')

# Get latest compatible version
node = registry.get_node('my_node', '^1.0.0')  # Returns 1.x.x
```

### Sandboxed Execution

**Security Layers:**

1. **Restricted Builtins**: Limited Python built-in functions
2. **Module Whitelist**: Only safe modules allowed
3. **Resource Limits**: CPU, memory, timeout constraints
4. **Code Validation**: Static analysis before execution

**Allowed Modules:**
- Data: `pandas`, `numpy`, `json`
- Math: `math`, `statistics`
- Text: `re`, `string`
- Time: `datetime`, `time`
- Web: `requests` (with restrictions)

**Execution Flow:**
```python
# Station: core/community_node_executor.py
class CommunityNodeExecutor:
    def execute_python(self, code: str, inputs: dict, timeout: int = 10):
        """Execute Python code in sandbox"""
        
        # 1. Validate code safety
        self._validate_code(code)
        
        # 2. Set up sandbox environment
        restricted_globals = self._get_safe_globals()
        
        # 3. Execute with timeout
        result = self._run_with_timeout(code, restricted_globals, timeout)
        
        return result
```

### Data Models (Control)

**CommunityNode Schema (MongoDB):**
```javascript
{
  nodeId: String,              // Unique identifier
  name: String,                // Internal name
  displayName: String,         // User-facing name
  category: String,            // data|ai|defi|trading|utility
  description: String,         // Full description
  authorId: String,            // Creator user ID
  authorName: String,          // Creator display name
  version: String,             // Current version
  executionType: String,       // python|http
  
  inputs: [{
    name: String,
    type: String,              // string|number|boolean|json|array
    required: Boolean,
    description: String
  }],
  
  outputs: [{
    name: String,
    type: String,
    description: String
  }],
  
  executionConfig: {
    pythonCode: String,        // For python execution
    url: String,               // For HTTP execution
    timeout: Number
  },
  
  stats: {
    views: Number,
    downloads: Number,
    stars: Number,
    uses: Number
  },
  
  status: String,              // draft|published|deprecated
  
  createdAt: Date,
  updatedAt: Date
}
```

---

## API Reference

### Node Management APIs

**List Nodes**
```http
GET /api/v1/community/nodes?category=ai&sort=-stats.stars&page=1&limit=20
```

**Get Node Detail**
```http
GET /api/v1/community/nodes/:nodeId
```

**Create Node**
```http
POST /api/v1/community/nodes
Content-Type: application/json

{
  "name": "my_custom_node",
  "displayName": "My Custom Node",
  "category": "utility",
  "description": "...",
  "inputs": [...],
  "outputs": [...],
  "executionType": "python",
  "executionConfig": {...}
}
```

**Publish Node**
```http
POST /api/v1/community/nodes/:nodeId/publish
```

**Star Node**
```http
POST /api/v1/community/nodes/:nodeId/star
```

### Comment APIs

**Get Comments**
```http
GET /api/v1/community/nodes/:nodeId/comments?page=1&limit=20
```

**Add Comment**
```http
POST /api/v1/community/nodes/:nodeId/comments
Content-Type: application/json

{
  "content": "Great node!",
  "rating": 5
}
```

### Execution APIs (Station)

**Execute Community Node**
```http
POST /node/community/execute
Content-Type: application/json

{
  "nodeConfig": {
    "nodeId": "custom_indicator",
    "version": "1.0.0",
    "executionType": "python",
    "executionConfig": {...}
  },
  "inputs": {
    "price_data": [100, 101, 102],
    "period": 3
  }
}
```

---

## Developer Guide

### Creating a Community Node Class

```python
from station.nodes.node_base import NodeBase
from station.core.node_registry import register_node

@register_node('my_custom_node', version='1.0.0')
class MyCustomNode(NodeBase):
    def __init__(self, node_id: str, config: dict):
        super().__init__(node_id, config)
        
        # Define inputs
        self.inputs = {
            'input_data': {'type': 'json', 'required': True}
        }
        
        # Define outputs
        self.outputs = {
            'result': {'type': 'json'}
        }
    
    def execute(self, inputs: dict) -> dict:
        """Main execution logic"""
        
        # Get input
        data = inputs.get('input_data')
        
        # Process
        result = self._process(data)
        
        # Return output
        return {'result': result}
    
    def _process(self, data):
        """Your custom logic"""
        return data
```

### Testing Your Node

```python
# Test locally
node = MyCustomNode('test_node', {})
result = node.execute({'input_data': {'value': 100}})
print(result)  # {'result': {...}}
```

### Frontend Integration

**Loading Community Nodes:**
```typescript
// ComponentsPanel.tsx
useEffect(() => {
  const loadCommunityNodes = async () => {
    const response = await getCommunityNodes({
      sort: '-stats.stars',
      limit: 20
    });
    const publishedNodes = response.nodes.filter(
      node => node.status === 'published'
    );
    setCommunityNodes(publishedNodes);
  };
  
  loadCommunityNodes();
}, []);
```

**Drag-and-Drop:**
```typescript
<div
  draggable
  onDragStart={(e) => {
    e.dataTransfer.setData('application/reactflow', `community_${node.nodeId}`);
  }}
>
  {/* Node display */}
</div>
```

---

## Best Practices

### For Node Creators

**1. Clear Documentation**
- Write detailed descriptions
- Explain all parameters
- Provide usage examples
- Document edge cases

**2. Input Validation**
- Validate all inputs
- Provide clear error messages
- Handle missing/invalid data gracefully

**3. Performance**
- Keep execution time under 10 seconds
- Optimize algorithms
- Use caching when appropriate

**4. Testing**
- Test with various inputs
- Test error conditions
- Test edge cases

**5. Versioning**
- Use semantic versioning correctly
- Document breaking changes
- Maintain backward compatibility when possible

### For Node Users

**1. Read Documentation**
- Understand node purpose
- Check parameter requirements
- Review examples

**2. Start Simple**
- Test with minimal inputs first
- Add complexity gradually
- Monitor execution logs

**3. Provide Feedback**
- Rate nodes you use
- Report issues
- Suggest improvements

---

## Security Considerations

### Code Execution Safety

**Sandboxing:**
- Limited built-in functions
- Module whitelist
- No file system access
- No network access (except approved APIs)

**Resource Limits:**
- Maximum execution time: 30 seconds
- Maximum memory: 500MB
- Maximum recursion depth: 1000

**Validation:**
- Static code analysis
- Dangerous keyword detection
- AST-based validation

### User-Generated Content

**Moderation:**
- Manual review for first publication
- Automated dangerous code detection
- Community reporting system

**Attribution:**
- All nodes tracked by creator
- Usage statistics visible
- Cannot delete nodes in use

---

## Troubleshooting

### Common Issues

**1. Node Not Appearing in FlowEdit**

**Cause**: Node not published or failed to load  
**Solution**: 
- Check node status is "published"
- Refresh browser
- Check console for errors

**2. Execution Timeout**

**Cause**: Code takes too long  
**Solution**:
- Optimize algorithms
- Reduce input data size
- Use caching

**3. Module Import Error**

**Cause**: Module not in whitelist  
**Solution**:
- Use only allowed modules
- Request module addition via Issue

**4. Input Validation Errors**

**Cause**: Incorrect input format  
**Solution**:
- Check required parameters
- Verify data types
- Review node documentation

---

## Future Enhancements

**Planned Features:**
- [ ] Node version selection in FlowEdit
- [ ] Comment replies and threading
- [ ] Node usage analytics dashboard
- [ ] Node templates and wizards
- [ ] Collaborative node editing
- [ ] Node dependency management
- [ ] Paid premium nodes
- [ ] Node certification program

---

## Related Documentation

- [Weather Station Overview](weather-station-overview.md)
- [Node Execution Flow](weather-station-node-execution.md)
- [Node Details Index](../node-details/index.md)

---

## Contributing

**Adding Features:**
1. Fork the repository
2. Create feature branch
3. Implement and test
4. Submit pull request

**Reporting Issues:**
1. Use GitHub Issues
2. Provide clear description
3. Include reproduction steps
4. Add relevant logs

---

**Maintainers:** TradingFlow Development Team  
**License:** MIT  
**Last Updated:** 2025-01-08

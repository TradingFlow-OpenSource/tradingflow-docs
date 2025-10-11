# 社区节点与版本控制系统

**版本：** 1.0.0  
**最后更新：** 2025-01-08  
**状态：** ✅ 生产就绪

---

## 📋 目录

- [概述](#概述)
- [用户指南：如何参与](#用户指南如何参与)
- [系统架构](#系统架构)
- [技术实现](#技术实现)
- [API 参考](#api-参考)
- [开发者指南](#开发者指南)
- [最佳实践](#最佳实践)

---

## 概述

### 什么是社区节点？

社区节点是 TradingFlow 的**用户贡献节点生态系统**，允许用户创建、分享和使用自定义交易节点。可以把它想象成交易逻辑的"应用商店"。

**核心特性：**
- 🎨 **创建自定义节点** - 使用 Python 代码或 HTTP API 构建节点
- 🔄 **版本控制** - npm 风格的语义化版本（major.minor.patch）
- 🛡️ **沙箱执行** - 用户代码的安全执行环境
- 💬 **社区反馈** - 评分、评论和趋势系统
- 🚀 **轻松集成** - 拖拽节点到 FlowEdit 中使用

### 为什么需要社区节点？

**对策略开发者：**
- 扩展 TradingFlow 的能力，无需等待官方发布
- 与他人分享你成功的交易逻辑
- 在现有社区节点基础上构建

**对节点创作者：**
- 通过 Stars 和下载量获得认可
- 用专业功能帮助社区
- 展示你的交易专业知识

**对 TradingFlow：**
- 生态系统快速增长
- 用户驱动创新
- 减少功能请求积压

---

## 用户指南：如何参与

### 🌟 发现节点

**1. 浏览社区节点市场**

访问：`/community/nodes`

- **搜索**：按名称或描述查找节点
- **筛选**：按类别（数据、AI、DeFi、交易等）
- **排序**：按 Stars、下载、最新或浏览量
- **探索**：热门和趋势板块

**2. 查看节点详情**

点击任何节点卡片查看：
- 完整描述和文档
- 输入/输出参数
- 版本历史
- 用户评分和评论
- 使用统计

### 🎨 创建你自己的节点

**步骤 1：规划你的节点**

问自己：
- 它解决什么问题？
- 需要什么输入？
- 提供什么输出？
- 使用 Python 代码还是 HTTP API？

**步骤 2：创建节点**

1. 点击**"创建节点"**按钮
2. 填写基本信息：
   - **节点 ID**：唯一标识符（如 `my_custom_indicator`）
   - **显示名称**：用户友好的名称（如 "我的自定义指标"）
   - **类别**：选择合适的类别
   - **描述**：清晰解释功能

3. 定义输入和输出：
   - 添加带类型的输入参数（string、number、json 等）
   - 添加带类型的输出参数
   - 标记必需和可选输入

4. 配置执行：
   - **Python 代码**：直接编写你的逻辑
   - **HTTP API**：提供端点 URL 和配置

**步骤 3：测试和发布**

1. 保存为草稿
2. 在你的流程中彻底测试
3. 准备好后点击**"发布"**
4. 节点出现在社区市场

### 📝 示例：创建移动平均线节点

```python
# 输入: price_data (array), period (number)
# 输出: ma_value (number)

import numpy as np

prices = inputs['price_data']
period = inputs['period']

# 计算移动平均
ma = np.mean(prices[-period:])

outputs['ma_value'] = ma
```

### 💬 参与社区互动

**评分和评论：**
- 留下评分（1-5 星）
- 写有帮助的评论
- 报告问题

**收藏你喜欢的节点：**
- Star 你觉得有用的节点
- 被 Star 的节点会出现在趋势中

**分享你的知识：**
- 在评论中回答问题
- 建议改进
- 协作更新节点

### 🚀 在流程中使用社区节点

**方法 1：从节点详情页**

1. 浏览到节点详情页
2. 点击**"添加到 Flow"**
3. 导航到 FlowEdit
4. 在 "Community" 类别中找到节点
5. 拖到画布上

**方法 2：直接从 FlowEdit**

1. 在 FlowEdit 中打开任何流程
2. 点击**编辑**工具（左侧边栏）
3. 展开**"Community"**类别
4. 浏览可用节点
5. 拖到画布上

---

## 系统架构

### 三层架构

```
┌─────────────────────────────────────────────┐
│           前端 (React/TS)                   │
│  - 节点市场 UI                              │
│  - FlowEdit 集成                            │
│  - 用户管理                                 │
└─────────────────────────────────────────────┘
                    ↓ HTTP/REST
┌─────────────────────────────────────────────┐
│          Control (Node.js/MongoDB)          │
│  - 节点元数据管理                           │
│  - 评论和评分系统                           │
│  - 使用统计                                 │
└─────────────────────────────────────────────┘
                    ↓ 执行请求
┌─────────────────────────────────────────────┐
│          Station (Python/FastAPI)           │
│  - 版本管理                                 │
│  - 节点注册表                               │
│  - 沙箱执行                                 │
└─────────────────────────────────────────────┘
```

### 核心组件

**1. 版本管理器 (Station)**
- 语义化版本（major.minor.patch）
- 版本比较和范围匹配
- 向后兼容性检查

**2. 节点注册表 (Station)**
- 多版本节点注册
- 节点查找和检索
- 版本解析

**3. 社区节点执行器 (Station)**
- 安全的 Python 代码执行（沙箱）
- HTTP API 调用
- 资源限制和超时

**4. 节点元数据服务 (Control)**
- 节点元数据的 CRUD 操作
- 发布工作流
- 搜索和筛选

**5. 评论服务 (Control)**
- 用户评分和评论
- 回复线程
- 点赞/点踩跟踪

**6. 前端集成**
- 节点市场 UI
- FlowEdit 拖拽功能
- 实时加载和更新

---

## 技术实现

### 版本控制系统

**格式：** `major.minor.patch`（如 `1.2.3`）

**规则：**
- **Major**：破坏性变更（不兼容的 API）
- **Minor**：新功能（向后兼容）
- **Patch**：Bug 修复（向后兼容）

**版本范围：**
```python
# 精确版本
"1.2.3"

# 兼容 1.x.x
"^1.2.3"  # >= 1.2.3 且 < 2.0.0

# 仅次要更新
"~1.2.3"  # >= 1.2.3 且 < 1.3.0

# 大于或等于
">=1.2.3"
```

**实现：**
```python
# Station: core/version_manager.py
class VersionManager:
    @staticmethod
    def parse_version(version: str) -> Tuple[int, int, int]:
        """解析语义化版本字符串"""
        
    @staticmethod
    def compare_versions(v1: str, v2: str) -> int:
        """比较两个版本 (-1, 0, 1)"""
        
    @staticmethod
    def match_version_range(version: str, range_spec: str) -> bool:
        """检查版本是否匹配范围"""
```

### 节点注册表

**多版本支持：**
```python
# 注册同一节点的多个版本
registry.register_node('my_node', MyNodeV1, version='1.0.0')
registry.register_node('my_node', MyNodeV2, version='2.0.0')

# 获取特定版本
node = registry.get_node('my_node', '1.0.0')

# 获取最新兼容版本
node = registry.get_node('my_node', '^1.0.0')  # 返回 1.x.x
```

### 沙箱执行

**安全层级：**

1. **受限内置函数**：限制的 Python 内置函数
2. **模块白名单**：只允许安全模块
3. **资源限制**：CPU、内存、超时约束
4. **代码验证**：执行前的静态分析

**允许的模块：**
- 数据：`pandas`、`numpy`、`json`
- 数学：`math`、`statistics`
- 文本：`re`、`string`
- 时间：`datetime`、`time`
- 网络：`requests`（有限制）

**执行流程：**
```python
# Station: core/community_node_executor.py
class CommunityNodeExecutor:
    def execute_python(self, code: str, inputs: dict, timeout: int = 10):
        """在沙箱中执行 Python 代码"""
        
        # 1. 验证代码安全性
        self._validate_code(code)
        
        # 2. 设置沙箱环境
        restricted_globals = self._get_safe_globals()
        
        # 3. 带超时执行
        result = self._run_with_timeout(code, restricted_globals, timeout)
        
        return result
```

### 数据模型 (Control)

**CommunityNode Schema (MongoDB)：**
```javascript
{
  nodeId: String,              // 唯一标识符
  name: String,                // 内部名称
  displayName: String,         // 用户友好名称
  category: String,            // data|ai|defi|trading|utility
  description: String,         // 完整描述
  authorId: String,            // 创建者用户 ID
  authorName: String,          // 创建者显示名称
  version: String,             // 当前版本
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
    pythonCode: String,        // Python 执行
    url: String,               // HTTP 执行
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

## API 参考

### 节点管理 API

**列出节点**
```http
GET /api/v1/community/nodes?category=ai&sort=-stats.stars&page=1&limit=20
```

**获取节点详情**
```http
GET /api/v1/community/nodes/:nodeId
```

**创建节点**
```http
POST /api/v1/community/nodes
Content-Type: application/json

{
  "name": "my_custom_node",
  "displayName": "我的自定义节点",
  "category": "utility",
  "description": "...",
  "inputs": [...],
  "outputs": [...],
  "executionType": "python",
  "executionConfig": {...}
}
```

**发布节点**
```http
POST /api/v1/community/nodes/:nodeId/publish
```

**收藏节点**
```http
POST /api/v1/community/nodes/:nodeId/star
```

### 评论 API

**获取评论**
```http
GET /api/v1/community/nodes/:nodeId/comments?page=1&limit=20
```

**添加评论**
```http
POST /api/v1/community/nodes/:nodeId/comments
Content-Type: application/json

{
  "content": "很棒的节点！",
  "rating": 5
}
```

### 执行 API (Station)

**执行社区节点**
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

## 开发者指南

### 创建社区节点类

```python
from station.nodes.node_base import NodeBase
from station.core.node_registry import register_node

@register_node('my_custom_node', version='1.0.0')
class MyCustomNode(NodeBase):
    def __init__(self, node_id: str, config: dict):
        super().__init__(node_id, config)
        
        # 定义输入
        self.inputs = {
            'input_data': {'type': 'json', 'required': True}
        }
        
        # 定义输出
        self.outputs = {
            'result': {'type': 'json'}
        }
    
    def execute(self, inputs: dict) -> dict:
        """主执行逻辑"""
        
        # 获取输入
        data = inputs.get('input_data')
        
        # 处理
        result = self._process(data)
        
        # 返回输出
        return {'result': result}
    
    def _process(self, data):
        """你的自定义逻辑"""
        return data
```

### 测试你的节点

```python
# 本地测试
node = MyCustomNode('test_node', {})
result = node.execute({'input_data': {'value': 100}})
print(result)  # {'result': {...}}
```

### 前端集成

**加载社区节点：**
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

**拖拽功能：**
```typescript
<div
  draggable
  onDragStart={(e) => {
    e.dataTransfer.setData('application/reactflow', `community_${node.nodeId}`);
  }}
>
  {/* 节点显示 */}
</div>
```

---

## 最佳实践

### 对节点创作者

**1. 清晰的文档**
- 编写详细的描述
- 解释所有参数
- 提供使用示例
- 记录边缘情况

**2. 输入验证**
- 验证所有输入
- 提供清晰的错误消息
- 优雅地处理缺失/无效数据

**3. 性能**
- 保持执行时间在 10 秒以下
- 优化算法
- 适当使用缓存

**4. 测试**
- 测试各种输入
- 测试错误条件
- 测试边缘情况

**5. 版本控制**
- 正确使用语义化版本
- 记录破坏性变更
- 尽可能保持向后兼容

### 对节点使用者

**1. 阅读文档**
- 理解节点目的
- 检查参数要求
- 查看示例

**2. 从简单开始**
- 先用最小输入测试
- 逐步增加复杂度
- 监控执行日志

**3. 提供反馈**
- 评价你使用的节点
- 报告问题
- 建议改进

---

## 安全考虑

### 代码执行安全

**沙箱化：**
- 限制的内置函数
- 模块白名单
- 无文件系统访问
- 无网络访问（除批准的 API）

**资源限制：**
- 最大执行时间：30 秒
- 最大内存：500MB
- 最大递归深度：1000

**验证：**
- 静态代码分析
- 危险关键字检测
- 基于 AST 的验证

### 用户生成内容

**审核：**
- 首次发布手动审核
- 自动危险代码检测
- 社区报告系统

**归属：**
- 所有节点由创建者跟踪
- 使用统计可见
- 不能删除正在使用的节点

---

## 故障排除

### 常见问题

**1. 节点未出现在 FlowEdit 中**

**原因**：节点未发布或加载失败  
**解决方案**：
- 检查节点状态是否为 "published"
- 刷新浏览器
- 检查控制台错误

**2. 执行超时**

**原因**：代码执行时间过长  
**解决方案**：
- 优化算法
- 减少输入数据大小
- 使用缓存

**3. 模块导入错误**

**原因**：模块不在白名单中  
**解决方案**：
- 只使用允许的模块
- 通过 Issue 请求添加模块

**4. 输入验证错误**

**原因**：输入格式不正确  
**解决方案**：
- 检查必需参数
- 验证数据类型
- 查看节点文档

---

## 未来增强

**计划功能：**
- [ ] FlowEdit 中的节点版本选择
- [ ] 评论回复和线程
- [ ] 节点使用分析仪表板
- [ ] 节点模板和向导
- [ ] 协作节点编辑
- [ ] 节点依赖管理
- [ ] 付费高级节点
- [ ] 节点认证计划

---

## 相关文档

- [Weather Station 概述](weather-station-overview.md)
- [节点执行流程](weather-station-node-execution.md)
- [节点详情索引](../node-details/index.md)

---

## 贡献

**添加功能：**
1. Fork 仓库
2. 创建功能分支
3. 实现和测试
4. 提交 Pull Request

**报告问题：**
1. 使用 GitHub Issues
2. 提供清晰描述
3. 包含复现步骤
4. 添加相关日志

---

**维护者：** TradingFlow 开发团队  
**许可证：** MIT  
**最后更新：** 2025-01-08

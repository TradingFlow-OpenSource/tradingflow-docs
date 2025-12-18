# RootData Node

RootData Node 用于查询 RootData API，获取加密货币项目、VC、人员等数据。支持搜索、ID 映射、热门榜单、详情查询等多种操作。

---

## 节点信息

| 属性 | 值 |
|------|-----|
| **节点类型** | `rootdata_node` |
| **显示名称** | RootData Node |
| **节点分类** | Input（数据输入） |
| **图标** | 📊 Database 图标（蓝色） |
| **句柄颜色** | Emerald（绿色） |

---

## 功能说明

RootData Node 连接 RootData API，提供加密货币生态系统的全面数据访问，包括项目信息、投资机构、团队人员、融资动态等。

**主要用途：**
- 搜索加密货币项目、VC、人员
- 获取项目/VC/人员详情
- 查询热门项目排行榜
- 追踪融资动态和人员变动
- 获取生态系统和标签映射

**核心特性：**
- 🔍 **多模式搜索**：支持 19 种不同操作类型
- 📊 **热门榜单**：Top 100 热门项目、X 平台热门
- 👥 **人员追踪**：职位变动、KOL 排名
- 💰 **融资数据**：批量获取融资信息
- 🏷️ **分类系统**：生态系统和标签映射

---

## 输入参数

### 核心参数（始终显示）

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `operation` | select | ✅ | `search` | 操作类型 |
| `query` | text | ❌ | - | 搜索关键词 |

### 高级参数（默认隐藏）

以下参数默认隐藏，仅在需要时展开使用：

| 参数 | 类型 | 说明 | 适用操作 |
|------|------|------|---------|
| `type` | number | 类型：1=Project, 2=VC, 3=People | search |
| `project_id` | number | 项目 ID | get_item, get_fac |
| `contract_address` | text | 项目合约地址 | get_item |
| `org_id` | number | VC 机构 ID | get_org |
| `people_id` | number | 人员 ID | get_people |
| `ecosystem_ids` | text | 生态系统 ID（逗号分隔） | projects_by_ecosystems |
| `tag_ids` | text | 标签 ID（逗号分隔） | projects_by_tags |
| `page` | number | 分页页码 | 列表操作 |
| `page_size` | number | 每页数量 | 列表操作 |
| `begin_time` | number | 开始时间戳（毫秒） | get_fac, ser_change |
| `end_time` | number | 结束时间戳（毫秒） | get_fac, ser_change |
| `min_amount` | number | 最小金额（USD） | get_fac |
| `max_amount` | number | 最大金额（USD） | get_fac |
| `days` | number | 天数（1 或 7） | hot_index |
| `rank_type` | select | 排名类型：heat/influence | leading_figures_on_crypto_x |
| `language` | select | 语言：en/cn | - |
| `heat` | boolean | 包含热度数据 | - |
| `influence` | boolean | 包含影响力数据 | - |
| `followers` | boolean | 包含粉丝数据 | - |
| `recent_joinees` | boolean | 包含近期加入者 | job_changes |
| `recent_resignations` | boolean | 包含近期离职者 | job_changes |
| `include_team` | boolean | 包含团队信息 | get_item |
| `include_investors` | boolean | 包含投资者信息 | get_item |
| `include_investments` | boolean | 包含投资项目 | get_org |
| `precise_x_search` | boolean | 精确 X 搜索 | twitter_map |

---

## 操作类型详解

### operation 参数

| 值 | 名称 | 说明 |
|-----|------|------|
| `search` | Search | 搜索项目/VC/人员 |
| `id_map` | ID Map | 获取 ID 映射表 |
| `get_item` | Project Detail | 获取项目详情 |
| `get_org` | VC Detail | 获取 VC 机构详情 |
| `get_people` | People Detail | 获取人员详情 |
| `get_invest` | Investors (batch) | 批量获取投资者 |
| `twitter_map` | X Data Map | X 平台数据映射 |
| `get_fac` | Fundraising (batch) | 批量获取融资信息 |
| `ser_change` | Sync Updates | 同步更新列表 |
| `hot_index` | Top 100 Hot Projects | 热门项目 Top 100 |
| `hot_project_on_x` | X Hot Projects | X 平台热门项目 |
| `leading_figures_on_crypto_x` | X Hot People | X 平台热门人物 |
| `job_changes` | People Position Dynamics | 人员职位变动 |
| `new_tokens` | New Tokens (3 months) | 近 3 个月新代币 |
| `ecosystem_map` | Ecosystem Map | 生态系统映射 |
| `tag_map` | Tag Map | 标签映射 |
| `projects_by_ecosystems` | Projects by Ecosystems | 按生态查询项目 |
| `projects_by_tags` | Projects by Tags | 按标签查询项目 |
| `quotacredits` | API Balance | 查询 API 余额 |

---

## 输出参数

### 输出列表

| 输出 ID | 显示名称 | 数据类型 | 说明 |
|---------|---------|---------|------|
| `data` | Data | object | RootData API 响应 |

### data 输出

**数据结构：**

```typescript
{
  result: number;        // 结果码（0=成功）
  data: any;             // 具体数据（根据操作不同）
  message?: string;      // 错误信息（如有）
  _meta?: {              // 元数据
    page?: number;
    page_size?: number;
    total?: number;
  }
}
```

---

## 使用示例

### 示例 1：搜索项目

**场景：** 搜索包含 "DeFi" 关键词的项目。

**节点配置：**
```json
{
  "operation": "search",
  "query": "DeFi",
  "type": 1
}
```

**输出示例：**
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

### 示例 2：获取热门项目

**场景：** 获取过去 7 天的 Top 100 热门项目。

**节点配置：**
```json
{
  "operation": "hot_index",
  "days": 7
}
```

**工作流结构：**
```
RootData Node (获取热门项目)
    ↓ data { projects }
Code Node (筛选 DeFi 类别)
    ↓ defi_projects
AI Model Node (分析潜力项目)
    ↓ recommendations
Telegram Sender Node (发送推荐)
```

---

### 示例 3：追踪人员变动

**场景：** 获取近期的职位变动信息。

**节点配置：**
```json
{
  "operation": "job_changes",
  "recent_joinees": true,
  "recent_resignations": true,
  "page": 1,
  "page_size": 50
}
```

**用途：**
- 追踪 KOL 动态
- 发现行业趋势
- 监控团队变化

---

### 示例 4：融资数据分析

**场景：** 获取指定时间范围内的融资数据。

**节点配置：**
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

**工作流结构：**
```
RootData Node (获取融资数据)
    ↓ data
Code Node (统计分析)
    ↓ stats { total_raised, top_sectors, avg_round_size }
Google Sheet Output Node (导出报告)
```

---

### 示例 5：项目详情查询

**场景：** 获取特定项目的详细信息，包括团队和投资者。

**节点配置：**
```json
{
  "operation": "get_item",
  "project_id": 12345,
  "include_team": true,
  "include_investors": true
}
```

---

## 高级参数使用

### 关于高级参数

RootData Node 有 22 个高级参数，默认隐藏以简化界面。这些参数仅在特定操作中需要。

**显示高级参数的方式：**
1. 点击节点底部的 "Show advanced params" 按钮
2. 在弹窗中选择需要的参数
3. 或点击 "Show All" 显示全部

**隐藏已显示的参数：**
- 点击参数标题右侧的 "Hide" 按钮
- 隐藏时会清空参数值

**Agent 行为：**
- 当 Agent 为高级参数指定值时，该参数会自动显示
- 未指定的高级参数保持隐藏

---

## API 配额

RootData API 使用配额制度。使用 `quotacredits` 操作查询余额：

```json
{
  "operation": "quotacredits"
}
```

**返回示例：**
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

## 最佳实践

### 1. 操作选择

| 需求 | 推荐操作 |
|------|---------|
| 项目发现 | `search` + `query` |
| 热门追踪 | `hot_index` 或 `hot_project_on_x` |
| 详情获取 | `get_item`、`get_org`、`get_people` |
| 融资分析 | `get_fac` + 时间过滤 |
| 人员动态 | `job_changes` 或 `leading_figures_on_crypto_x` |

### 2. 分页使用

对于列表操作，合理使用分页：
- 默认 `page_size` 为 10
- 大量数据考虑分批获取
- 使用 `_meta.total` 判断总数

### 3. 缓存策略

- 热门榜单数据可缓存（变化较慢）
- 详情数据建议缓存
- 实时性要求高的场景直接查询

---

## 注意事项

### 重要提示

1. **API 配额管理**
   - 监控剩余配额
   - 避免不必要的重复查询
   - 合理设置 `page_size`

2. **参数兼容性**
   - 不同操作需要不同参数
   - 查看操作类型对应的必需参数
   - 无关参数可忽略

3. **数据时效性**
   - 热门榜单每日更新
   - 融资数据实时更新
   - 项目详情定期更新

---

## 故障排查

**Q: 返回 "result": -1？**

A:
1. 检查操作类型是否正确
2. 确认必需参数已提供
3. 验证 API 配额是否充足

---

**Q: 搜索结果为空？**

A:
1. 尝试调整搜索关键词
2. 检查 `type` 参数设置
3. 放宽过滤条件

---

**Q: 分页数据不完整？**

A:
1. 检查 `page` 和 `page_size` 设置
2. 确认 `_meta.total` 总数
3. 遍历所有页面获取完整数据

---

## 技术规格

| 规格项 | 值 |
|--------|-----|
| **节点版本** | 0.1.0 |
| **操作类型数** | 19 |
| **核心参数** | 2 |
| **高级参数** | 22 |
| **API** | RootData API |
| **超时时间** | 60秒 |

---

## 相关节点

- **AI Model Node** - 分析 RootData 数据
- **Code Node** - 处理和筛选数据
- **Google Sheet Output Node** - 导出数据报告
- **Telegram Sender Node** - 发送数据通知

---

**相关文档：**
- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [AI Model Node](ai-model-node.md) - AI 分析节点
- [Code Node](code-node.md) - 数据处理节点

# TradingFlow 节点文档创建指南（AI README）

本文档记录了 TradingFlow 节点文档系统的创建过程、规范和最佳实践，供后续 AI 助手或开发者参考。

**文档版本：** 1.0.0  
**创建日期：** 2025-10-06  
**完成状态：** 11/11 节点 (100%)

---

## 📋 目录

- [项目概述](#项目概述)
- [文档结构](#文档结构)
- [创建流程](#创建流程)
- [文档模板](#文档模板)
- [命名规范](#命名规范)
- [内容指南](#内容指南)
- [最佳实践](#最佳实践)
- [注意事项](#注意事项)
- [完成状态](#完成状态)

---

## 项目概述

### 目标

为 TradingFlow 平台的所有节点创建完整、统一、高质量的中文技术文档，帮助用户：
- 快速了解节点功能
- 学习参数配置方法
- 参考真实使用场景
- 解决常见问题
- 掌握最佳实践

### 范围

**节点总数：** 11 个  
**文档语言：** 主要中文，索引支持中英文  
**文档类型：** Markdown 格式  
**存放位置：** `7_docs/zh/node-details/`

### 节点分类

TradingFlow 使用 4 个 Collection 对节点进行分类：

| Collection | 中文名称 | 节点数 | 完成率 |
|-----------|---------|--------|--------|
| **Input** | 数据输入 | 3 | 100% |
| **Compute** | 计算处理 | 2 | 100% |
| **Trade** | 交易执行 | 4 | 100% |
| **Core** | 核心功能 | 2 | 100% |

---

## 文档结构

### 目录组织

```
7_docs/
├── AI_README.md                    # 本文档（AI 工作指南）
├── NODE_COLLECTIONS.md             # 节点分类总览
│
├── zh/                             # 中文文档
│   └── node-details/
│       ├── index.md                # 中文节点索引
│       ├── binance-price-node.md   # 各节点详细文档
│       ├── ai-model-node.md
│       ├── code-node.md
│       ├── vault-node.md
│       ├── swap-node.md
│       ├── buy-node.md
│       ├── sell-node.md
│       ├── x-listener-node.md
│       ├── dataset-input-node.md
│       ├── dataset-output-node.md
│       └── telegram-sender-node.md
│
└── en/                             # 英文文档
    └── node-details/
        └── index.md                # 英文节点索引
```

### 文件关系

```
NODE_COLLECTIONS.md
    └─ 包含所有节点的统计和分类信息

index.md (zh/en)
    └─ 节点索引页，链接到各个节点文档

[node-name].md
    └─ 单个节点的详细文档
```

---

## 创建流程

### 标准工作流

1. **阅读后端代码**
   ```python
   # 位置：3_weather_cluster/tradingflow/station/nodes/
   # 文件：{node_name}_node.py
   ```
   - 查看 `@register_node_type` 装饰器的 `default_params`
   - 阅读类的 `__init__` 方法
   - 理解 `execute()` 方法的执行逻辑
   - 查看 `_register_input_handles()` 方法

2. **阅读前端代码**（可选）
   ```typescript
   // 位置：1_weather_frontend/src/pages/flow/components/TFNode/instances/
   // 文件：{NodeName}Node.tsx
   ```
   - 查看节点配置 `nodeConfig`
   - 了解 UI 界面的参数定义

3. **理解节点架构**
   - 是基类节点还是实例节点？
   - 有哪些输入输出参数？
   - 支持哪些功能和模式？
   - 有什么特殊的算法或机制？

4. **创建文档**
   - 使用标准模板（见下文）
   - 按照章节顺序填写内容
   - 添加真实的使用示例
   - 编写故障排查部分

5. **更新索引**
   - 在 `NODE_COLLECTIONS.md` 中更新节点状态
   - 在 `zh/node-details/index.md` 中添加链接
   - 在 `en/node-details/index.md` 中添加对应链接（如有）

---

## 文档模板

### 标准章节结构

每个节点文档应包含以下章节（按顺序）：

```markdown
# [节点名称] Node

[一句话描述节点功能]

---

## 节点信息

| 属性 | 值 |
|------|-----|
| **节点类型** | `node_type` |
| **显示名称** | Display Name |
| **节点分类** | Collection |
| **图标** | 📊 图标 |
| **句柄颜色** | 颜色 |
| **节点类别** | Base/Instance Node (如适用) |
| **基类节点** | `base_node_type` (如适用) |

---

## 功能说明

[详细的功能介绍]

**主要用途：**
- 用途1
- 用途2
- 用途3

**核心特性：**
- 🎯 **特性1**：说明
- 🔄 **特性2**：说明
- 📊 **特性3**：说明

---

## 与 [基类] 的关系（如果是实例节点）

[说明实例节点与基类的关系]

---

## 输入参数

### 参数列表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `param1` | type | ✅/❌ | value | 描述 |

### [param1] 参数

**说明：** 详细说明

**示例：**
```
示例值
```

---

## 输出参数

### 输出列表

| 输出 ID | 显示名称 | 数据类型 | 说明 |
|---------|---------|---------|------|
| `output1` | Output | object | 描述 |

### [output1] 输出

**数据类型：** `object`

**数据结构：**
```typescript
{
  field1: type;
  field2: type;
}
```

**示例输出：**
```json
{
  "field1": "value1",
  "field2": "value2"
}
```

---

## 工作流程（如果逻辑复杂）

### 执行流程

```
1. 步骤1
   ↓
2. 步骤2
   ↓
3. 步骤3
```

---

## 使用示例

### 示例 1：[场景名称]

**场景：** 场景描述

**工作流结构：**
```
节点1
    ↓ 输出
节点2
    ↓ 输出
节点3
```

**[节点名称] 配置：**
```json
{
  "param1": "value1",
  "param2": "value2"
}
```

**输出示例：**
```json
{
  "result": "..."
}
```

---

## API 依赖（如适用）

### [Service Name]

**方法：** `method_name()`

**参数：**
```python
{
  "param1": "value1"
}
```

**响应：**
```json
{
  "result": "..."
}
```

---

## 最佳实践

### 1. 实践主题

**✅ 推荐：**
```
推荐的做法
```

**❌ 避免：**
```
不推荐的做法
```

---

## 注意事项

### ⚠️ 重要提示

1. **注意点1**
   - 详细说明
   - 原因
   - 建议

---

## 故障排查

**Q: 常见问题？**

A:
1. 原因分析
2. 解决方案
3. 验证方法

---

## 技术规格

| 规格项 | 值 |
|--------|-----|
| **节点版本** | 0.0.x |
| **规格1** | 值1 |
| **规格2** | 值2 |

---

## 相关节点

- **节点1** - 说明
- **节点2** - 说明

---

**相关文档：**
- [节点与工作流](../core-concepts/nodes-and-workflows.md)
- [其他相关文档](link)
```

---

## 命名规范

### 文件命名

**格式：** `[节点英文名]-node.md`（全小写，连字符分隔）

**示例：**
- `binance-price-node.md`
- `ai-model-node.md`
- `x-listener-node.md`
- `dataset-input-node.md`

**规则：**
- 全小写字母
- 单词间用连字符 `-` 连接
- 以 `-node.md` 结尾
- 不使用下划线 `_`

### 标题命名

**节点名称：** 使用英文全称 + "Node"

**示例：**
```markdown
# Binance Price Node
# AI Model Node
# X Listener Node
# Dataset Input Node
```

### 参数命名

在文档中引用参数时：
- 使用代码格式：`` `param_name` ``
- 保持与后端代码一致的命名

---

## 内容指南

### 功能说明

**长度：** 2-3 段，约 100-200 字

**结构：**
1. 第一段：节点的核心功能和定位
2. 主要用途：3-5 个要点（列表）
3. 核心特性：3-5 个要点（使用表情符号强调）

**示例：**
```markdown
Swap Node 在去中心化交易所执行代币交换...

**主要用途：**
- 在 DEX 上交换代币
- 自动寻找最优流动性池
- 按百分比或固定金额交易

**核心特性：**
- 🔍 **智能池子搜索**：自动查找最优费率
- 💹 **动态价格计算**：基于池子实时数据
```

### 参数说明

**每个参数应包含：**
1. 参数名称和类型
2. 是否必填
3. 默认值
4. 详细说明
5. 格式要求
6. 示例值
7. 使用场景

**表格格式：**
```markdown
| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `chain` | select | ✅ | `aptos` | 区块链网络 |
```

### 使用示例

**数量：** 每个节点 3-5 个示例

**场景选择：**
- 示例 1：最简单的基本用法
- 示例 2：中等复杂度的常见场景
- 示例 3：高级用法或组合使用
- 示例 4-5：特殊场景（可选）

**每个示例包含：**
1. 场景描述
2. 工作流结构图（ASCII）
3. 节点配置（JSON）
4. 预期输出（JSON）
5. 执行说明（可选）

**工作流结构图格式：**
```
节点1 (说明)
    ↓ 输出名称
节点2 (说明)
    ↓ 输出名称
节点3 (说明)
```

### 故障排查

**格式：** Q&A 形式

**问题选择：**
- 最常见的 3-5 个问题
- 每个问题包含原因和解决方案
- 从简单到复杂排序

**示例：**
```markdown
**Q: 提示 "xxx 错误"？**

A:
1. 原因1：说明
2. 解决方案：步骤
3. 验证：如何确认修复
```

---

## 最佳实践

### 代码示例

**使用语法高亮：**
```markdown
```json
{
  "param": "value"
}
\```

```python
# Python 代码
output = process(input)
\```

```typescript
// TypeScript 代码
const result = await node.execute();
\```
```

### 表格使用

**适用场景：**
- 参数列表
- 节点信息
- 数据格式对比
- 选项说明

**保持一致：**
- 列宽合理
- 对齐方式统一
- 标题清晰

### 表情符号

**适度使用：**
- ✅ 正确/推荐
- ❌ 错误/避免
- ⚠️ 警告/注意
- 🎯 目标/重点
- 🔍 搜索/查找
- 💰 金钱/交易
- 📊 数据/统计
- 🤖 AI/自动化

**位置：**
- 章节标题（仅主要章节）
- 核心特性列表
- 最佳实践说明

### 链接使用

**内部链接：**
```markdown
[Vault Node](vault-node.md)
[节点与工作流](../core-concepts/nodes-and-workflows.md)
```

**外部链接：**
```markdown
[Telegram Bot API](https://core.telegram.org/bots/api)
```

---

## 注意事项

### 必须遵守的规则

1. **保持一致性**
   - 所有节点文档使用相同的结构
   - 相同章节使用相同的格式
   - 术语使用保持统一

2. **技术准确性**
   - 所有参数说明必须与代码一致
   - 数据结构必须准确
   - 示例代码可执行

3. **完整性**
   - 不遗漏任何参数
   - 覆盖所有主要功能
   - 包含常见问题解答

4. **可读性**
   - 使用清晰的语言
   - 适当使用表格和列表
   - 代码格式化

5. **实用性**
   - 提供真实的使用场景
   - 包含可复制的代码示例
   - 给出具体的配置建议

### 避免的问题

1. **❌ 不要：**
   - 使用模糊的描述
   - 遗漏重要参数
   - 提供错误的示例
   - 使用过时的信息
   - 过度使用表情符号

2. **❌ 不要：**
   - 复制粘贴未验证的内容
   - 使用不一致的术语
   - 忽略错误处理
   - 省略默认值说明

---

## 完成状态

### 已完成节点（11/11）

#### Input Collection（3/3）✅

| 节点 | 文档 | 大小 | 状态 |
|------|------|------|------|
| Binance Price Node | binance-price-node.md | 12 KB | ✅ |
| X Listener Node | x-listener-node.md | 22 KB | ✅ |
| Dataset Input Node | dataset-input-node.md | 14 KB | ✅ |

#### Compute Collection（2/2）✅

| 节点 | 文档 | 大小 | 状态 |
|------|------|------|------|
| AI Model Node | ai-model-node.md | 14 KB | ✅ |
| Code Node | code-node.md | 19 KB | ✅ |

#### Trade Collection（4/4）✅

| 节点 | 文档 | 大小 | 状态 |
|------|------|------|------|
| Vault Node | vault-node.md | 18 KB | ✅ |
| Swap Node | swap-node.md | 25 KB | ✅ |
| Buy Node | buy-node.md | 15 KB | ✅ |
| Sell Node | sell-node.md | 14 KB | ✅ |

#### Core Collection（2/2）✅

| 节点 | 文档 | 大小 | 状态 |
|------|------|------|------|
| Dataset Output Node | dataset-output-node.md | 13 KB | ✅ |
| Telegram Sender Node | telegram-sender-node.md | 16 KB | ✅ |

### 总体统计

| 指标 | 数值 |
|------|------|
| **完成节点** | 11/11 |
| **完成率** | 100% |
| **总文档量** | ~180 KB |
| **平均文档大小** | ~16 KB |
| **最大文档** | Swap Node (25 KB) |
| **最小文档** | Binance Price Node (12 KB) |

### 更新的文件

```
✅ NODE_COLLECTIONS.md - 更新统计到100%
✅ zh/node-details/index.md - 添加所有11个节点链接
✅ en/node-details/index.md - 英文索引同步更新
✅ 11个节点详细文档 - 全部创建完成
```

---

## 后续维护

### 添加新节点文档

1. **阅读代码**：理解新节点的功能和参数
2. **创建文档**：使用本文档的模板
3. **更新索引**：在相关文件中添加链接
4. **更新统计**：修改 `NODE_COLLECTIONS.md`

### 更新现有文档

1. **版本追踪**：注意节点版本变化
2. **参数同步**：确保参数说明与代码一致
3. **示例更新**：根据新功能添加示例
4. **故障排查**：收集新的常见问题

### 质量保证

**检查清单：**
- [ ] 文档结构完整
- [ ] 所有参数都有说明
- [ ] 至少有3个使用示例
- [ ] 包含故障排查部分
- [ ] 技术规格表填写完整
- [ ] 相关节点链接正确
- [ ] 代码格式正确
- [ ] 没有拼写错误
- [ ] 索引已更新

---

## 技术细节

### 节点架构模式

TradingFlow 支持两种节点架构：

#### 1. 独立节点

**特点：**
- 单一功能
- 不继承其他节点
- 直接注册

**示例：**
- Binance Price Node
- X Listener Node
- Telegram Sender Node

#### 2. 基类-实例架构

**特点：**
- 基类节点：提供通用功能，支持多种模式
- 实例节点：继承基类，专注特定用途

**示例：**

**Dataset 系列：**
```
DatasetNode (基类)
    ├─ mode: read/write/append
    ├─ DatasetInputNode (实例) - 强制 mode=read
    └─ DatasetOutputNode (实例) - 强制 mode=write
```

**Swap 系列：**
```
SwapNode (基类)
    ├─ from_token → to_token
    ├─ BuyNode (实例) - 买入语义
    └─ SellNode (实例) - 卖出语义
```

### 元数据系统

**节点元数据字段：**
```python
{
    "version": "0.0.2",
    "display_name": "Node Name",
    "node_category": "base" or "instance",
    "base_node_type": "base_node_type",  # 仅实例节点
    "description": "描述",
    "author": "TradingFlow Team",
    "tags": ["tag1", "tag2"]
}
```

### 信号系统

**信号类型：**
- `SignalType.DATA_READY` - 数据就绪
- `SignalType.PRICE_DATA` - 价格数据
- `SignalType.DATASET` - 数据集
- `SignalType.DEX_TRADE_RECEIPT` - 交易收据
- `SignalType.VAULT_INFO` - 金库信息
- `SignalType.AI_RESPONSE` - AI 响应
- `SignalType.TEXT` - 文本

---

## 工具和资源

### Markdown 编辑

**推荐工具：**
- VS Code + Markdown Preview
- Typora
- MarkText

**语法参考：**
- [Markdown Guide](https://www.markdownguide.org/)
- [GitHub Flavored Markdown](https://github.github.com/gfm/)

### 代码参考

**后端代码位置：**
```
3_weather_cluster/tradingflow/station/nodes/
```

**前端代码位置：**
```
1_weather_frontend/src/pages/flow/components/TFNode/instances/
```

### 相关文档

- `NODE_COLLECTIONS.md` - 节点分类总览
- `zh/node-details/index.md` - 中文节点索引
- `en/node-details/index.md` - 英文节点索引

---

## 附录

### 常用术语

| 中文 | 英文 | 说明 |
|------|------|------|
| 节点 | Node | 工作流的基本单元 |
| 句柄 | Handle | 节点的输入输出连接点 |
| 工作流 | Workflow/Flow | 多个节点组成的执行流程 |
| 金库 | Vault | TradingFlow 的资产管理合约 |
| 信号 | Signal | 节点间传递的数据 |
| 收据 | Receipt | 交易执行结果 |

### Collection 颜色对照

| Collection | 句柄颜色 | 图标颜色 |
|-----------|---------|---------|
| Input | Emerald (绿色) / Sky (天蓝色) | 绿色系 |
| Compute | Sky (天蓝色) | 蓝色系 |
| Trade | Amber (橙色) / Emerald (绿色) | 橙色系 |
| Core | Rose (玫瑰红) / Emerald (绿色) | 红色系 |

### 文档版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0.0 | 2025-10-06 | 初始版本，完成全部11个节点文档 |

---

## 联系和反馈

如果您在使用本指南时有任何问题或建议，请：

1. 查看现有的节点文档作为参考
2. 参考 `NODE_COLLECTIONS.md` 了解整体结构
3. 保持与现有文档的一致性

---

**文档状态：** ✅ 完成  
**最后更新：** 2025-10-06  
**维护者：** TradingFlow 文档团队  
**文档位置：** `7_docs/AI_README.md`

---

## 快速参考

### 创建新节点文档的快速步骤

1. ✅ 阅读后端代码 (`3_weather_cluster/tradingflow/station/nodes/{node}_node.py`)
2. ✅ 复制文档模板（见"文档模板"章节）
3. ✅ 填写节点信息和功能说明
4. ✅ 列出所有输入输出参数
5. ✅ 编写 3-5 个使用示例
6. ✅ 添加最佳实践和故障排查
7. ✅ 更新 `NODE_COLLECTIONS.md` 和 `index.md`
8. ✅ 检查格式和链接

### 命名快速参考

- 文件名：`node-name-node.md`（小写，连字符）
- 标题：`Node Name Node`（标题大小写）
- 参数：`` `param_name` ``（代码格式）

### 常用表情符号

```
✅ 完成/正确
❌ 错误/禁止
⚠️ 警告/注意
🎯 重点/目标
🔍 搜索/查找
💰 金钱/交易
📊 数据/图表
🤖 AI/机器人
🔧 工具/配置
📱 移动/通知
⚡ 快速/执行
```

---

**祝您文档创建顺利！** 🚀

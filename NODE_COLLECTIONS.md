# TradingFlow 节点分类标注

本文档按照 collection 类型对 TradingFlow 平台的所有节点进行分类标注。

---

## Collection 类型说明

TradingFlow 使用以下四种 collection 类型对节点进行分类：

| Collection | 中文名称 | 说明 | 图标颜色 |
|-----------|---------|------|---------|
| `input` | 数据输入 | 从外部数据源获取数据的节点 | Emerald (绿色) |
| `compute` | 计算处理 | 进行数据分析、处理和转换的节点 | Sky (天蓝色) |
| `trade` | 交易执行 | 执行区块链交易和金库操作的节点 | Amber (橙色) |
| `core` | 核心功能 | 数据存储、消息发送等核心基础功能节点 | Rose (玫瑰红) / Emerald (绿色) |

---

## 📊 Input Collection（数据输入节点）

### 节点列表

| 节点名称 | Node Type | 文档状态 | 前端路径 | 后端路径 |
|---------|-----------|---------|---------|---------|
| **Binance Price Node** | `binance_price_node` | ✅ 已完成 | `instances/inputs/BinancePriceNode.tsx` | `nodes/binance_price_node.py` |
| **X Listener Node** | `x_listener_node` | ✅ 已完成 | `instances/inputs/TwitterListenerNode.tsx` | `nodes/x_listener_node.py` |
| **Dataset Input Node** | `dataset_input_node` | ✅ 已完成 | `instances/inputs/DatasetNode.tsx` | `nodes/dataset_node.py` |

### 特征标识

- **句柄颜色**：Emerald (绿色) / Sky (天蓝色)
- **主要功能**：数据采集、监听、导入
- **数据流向**：外部 → TradingFlow
- **常见输出**：价格数据、社交媒体内容、数据集

---

## 🧠 Compute Collection（计算处理节点）

### 节点列表

| 节点名称 | Node Type | 文档状态 | 前端路径 | 后端路径 |
|---------|-----------|---------|---------|---------|
| **AI Model Node** | `ai_model_node` | ✅ 已完成 | `instances/compute/AiModelNode.tsx` | `nodes/ai_model_node.py` |
| **Code Node** | `code_node` | ✅ 已完成 | `instances/compute/CodeNode.tsx` | `nodes/code_node.py` |

### 特征标识

- **句柄颜色**：Sky (天蓝色)
- **主要功能**：数据分析、智能决策、代码执行
- **数据流向**：输入数据 → 计算处理 → 输出结果
- **常见输出**：分析结果、交易信号、处理后的数据

---

## 💰 Trade Collection（交易执行节点）

### 节点列表

| 节点名称 | Node Type | 文档状态 | 前端路径 | 后端路径 |
|---------|-----------|---------|---------|---------|
| **Vault Node** | `vault_node` | ✅ 已完成 | `instances/trade/VaultNode.tsx` | `nodes/vault_node.py` |
| **Swap Node** | `swap_node` | ✅ 已完成 | `instances/trade/SwapNode.tsx` | `nodes/swap_node.py` |
| **Buy Node** | `buy_node` | ✅ 已完成 | `instances/trade/BuyNode.tsx` | `nodes/swap_node.py` (实例) |
| **Sell Node** | `sell_node` | ✅ 已完成 | `instances/trade/SellNode.tsx` | `nodes/swap_node.py` (实例) |

### 特征标识

- **句柄颜色**：Amber (橙色) / Emerald (绿色)
- **主要功能**：区块链交易、金库管理、资产操作
- **数据流向**：交易参数 → 区块链执行 → 交易收据
- **常见输出**：交易收据、金库余额、操作状态

---

## 🔧 Core Collection（核心功能节点）

### 节点列表

| 节点名称 | Node Type | 文档状态 | 前端路径 | 后端路径 |
|---------|-----------|---------|---------|---------|
| **Dataset Output Node** | `dataset_output_node` | ✅ 已完成 | `instances/outputs/DatasetNode.tsx` | `nodes/dataset_node.py` |
| **Telegram Sender Node** | `telegram_sender_node` | ✅ 已完成 | `instances/outputs/TelegramSenderNode.tsx` | `nodes/telegram_sender_node.py` |

### 特征标识

- **句柄颜色**：Rose (玫瑰红) / Emerald (绿色)
- **主要功能**：数据存储、消息发送、系统集成
- **数据流向**：TradingFlow → 外部系统
- **常见输出**：存储确认、发送状态

---

## 节点统计

### 按 Collection 统计

| Collection | 节点数量 | 已完成文档 | 完成率 |
|-----------|---------|-----------|--------|
| Input | 3 | 3 | 100% |
| Compute | 2 | 2 | 100% |
| Trade | 4 | 4 | 100% |
| Core | 2 | 2 | 100% |
| **总计** | **11** | **11** | **100%** |

### 按开发状态统计

| 状态 | 节点数量 |
|------|---------|
| ✅ 文档已完成 | 11 |
| ⏳ 待完成文档 | 0 |

---

## 节点命名规范

### 前端命名

**文件名格式：** `{NodeName}Node.tsx`

**示例：**
- `BinancePriceNode.tsx`
- `AiModelNode.tsx`
- `SwapNode.tsx`

### 后端命名

**文件名格式：** `{node_name}_node.py`

**示例：**
- `binance_price_node.py`
- `ai_model_node.py`
- `swap_node.py`

### Node Type 命名

**格式：** `{function}_node`

**示例：**
- `binance_price_node`
- `ai_model_node`
- `swap_node`

---

## Collection 使用指南

### 如何确定节点的 Collection

1. **Input Collection** - 节点主要功能是从外部获取数据
   - 是否连接外部 API？
   - 是否监听外部事件？
   - 是否导入数据？

2. **Compute Collection** - 节点主要功能是处理和分析数据
   - 是否执行计算？
   - 是否进行数据转换？
   - 是否使用 AI 分析？

3. **Trade Collection** - 节点主要功能是执行区块链交易
   - 是否涉及资产转移？
   - 是否调用智能合约？
   - 是否管理金库？

4. **Core Collection** - 节点提供基础设施功能
   - 是否存储数据？
   - 是否发送通知？
   - 是否提供系统级服务？

### 前端配置示例

```typescript
export const nodeConfig: Partial<TFNodeData> = {
  collection: 'input',  // 或 'compute', 'trade', 'core'
  // ... 其他配置
};
```

### 后端注册示例

```python
@register_node_type(
    "node_type_name",
    default_params={...},
)
class NodeName(NodeBase):
    # ... 节点实现
```

---

## 文档开发优先级

### 高优先级（核心功能）

1. **Vault Node** - 金库管理，交易执行的基础
2. **Swap Node** - 交易执行节点
3. **X Listener Node** - 社交媒体监听

### 中优先级（交易相关）

4. **Buy Node** - 买入交易
5. **Sell Node** - 卖出交易
6. **Dataset Input Node** - 数据导入

### 低优先级（辅助功能）

7. **Dataset Output Node** - 数据导出
8. **Telegram Sender Node** - 消息通知

---

## 下一步工作

### 待完成的节点文档

按优先级排序：

1. ⏳ Vault Node 文档
2. ⏳ Swap Node 文档
3. ⏳ X Listener Node 文档
4. ⏳ Buy Node 文档
5. ⏳ Sell Node 文档
6. ⏳ Dataset Input Node 文档
7. ⏳ Dataset Output Node 文档
8. ⏳ Telegram Sender Node 文档

### 文档模板

每个节点文档应包含：

- 节点信息表格
- 功能说明
- 输入参数详解
- 输出参数详解
- 信号处理机制
- 使用示例（3个）
- API 配置
- 最佳实践
- 注意事项
- 故障排查
- 技术规格

---

## 相关文档

- [Weather 语法（Full 版本）](zh/core-concepts/weather-syntax.md)
- [Weather 语法版本对比](zh/core-concepts/weather-syntax-comparison.md)
- [节点与工作流](zh/core-concepts/nodes-and-workflows.md)

---

**最后更新：** 2025-10-06  
**文档版本：** 1.0.0  
**维护者：** TradingFlow 文档团队

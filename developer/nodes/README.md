# Weather Station 节点详情文档

**版本：** 1.0.0  
**最后更新：** 2025-10-06

---

## 📚 节点文档索引

本目录包含 Weather Station 所有节点类型的详细文档。每个节点都有独立的文档页面，包含完整的参数说明、使用示例和最佳实践。

---

## 🔍 按分类浏览

### 数据输入节点

**用途：** 从外部数据源获取数据

| 节点名称 | 类型 | 数据源 | 文档链接 |
|---------|------|--------|---------|
| **Binance Price Node** | `binance_price_node` | Binance 交易所 | [查看详情](binance-price-node.md) |
| **X Listener Node** | `x_listener_node` | Twitter/X 平台 | [查看详情](x-listener-node.md) |
| **RSSHub Node** | `rsshub_node` | RSS 聚合源 | [查看详情](rsshub-node.md) |
| **Dataset Node** | `dataset_node` | Google Sheets | [查看详情](dataset-node.md) |

### AI 处理节点

**用途：** 使用 AI 模型进行分析和决策

| 节点名称 | 类型 | AI 能力 | 文档链接 |
|---------|------|---------|---------|
| **AI Model Node** | `ai_model_node` | LLM 分析决策 | [查看详情](ai-model-node.md) |
| **Code Node** | `code_node` | Python 代码执行 | [查看详情](code-node.md) |

### 交易执行节点

**用途：** 执行链上交易操作

| 节点名称 | 类型 | 支持的链 | 文档链接 |
|---------|------|----------|---------|
| **Swap Node** | `swap_node` | Aptos, Flow EVM | [查看详情](swap-node.md) |
| **Buy Node** | `buy_node` | Aptos, Flow EVM | [查看详情](buy-node.md) |
| **Sell Node** | `sell_node` | Aptos, Flow EVM | [查看详情](sell-node.md) |
| **Uniswap DEX Trade Node** | `uniswap_dex_trade_node` | Ethereum, BSC | [查看详情](uniswap-dex-trade-node.md) |

### Vault 管理节点

**用途：** 管理 Vault 资产和操作

| 节点名称 | 类型 | 功能 | 文档链接 |
|---------|------|------|---------|
| **Vault Node** | `vault_node` | Vault 信息查询 | [查看详情](vault-node.md) |

### 通知节点

**用途：** 发送通知和消息

| 节点名称 | 类型 | 通知渠道 | 文档链接 |
|---------|------|----------|---------|
| **Telegram Sender Node** | `telegram_sender_node` | Telegram | [查看详情](telegram-sender-node.md) |

---

## 🚀 快速查找

### 按用途查找

#### 获取市场数据
- [Binance Price Node](binance-price-node.md) - 获取加密货币价格
- [X Listener Node](x-listener-node.md) - 监听社交媒体动态
- [RSSHub Node](rsshub-node.md) - 订阅新闻资讯

#### AI 分析决策
- [AI Model Node](ai-model-node.md) - LLM 智能分析
- [Code Node](code-node.md) - 自定义 Python 逻辑

#### 执行交易
- [Swap Node](swap-node.md) - 多链代币交换
- [Buy Node](buy-node.md) - 买入操作
- [Sell Node](sell-node.md) - 卖出操作

#### 资产管理
- [Vault Node](vault-node.md) - 查询 Vault 信息
- [Dataset Node](dataset-node.md) - 数据持久化

### 按区块链查找

#### Aptos
- [Swap Node](swap-node.md)
- [Buy Node](buy-node.md)
- [Sell Node](sell-node.md)
- [Vault Node](vault-node.md)

#### Flow EVM
- [Swap Node](swap-node.md)
- [Buy Node](buy-node.md)
- [Sell Node](sell-node.md)

#### Ethereum / BSC
- [Uniswap DEX Trade Node](uniswap-dex-trade-node.md)

---

## 📖 节点开发指南

### 创建新节点

如果您需要开发新的节点类型，请参考：

1. [节点开发指南](../weather-station-node-execution.md#开发新节点)
2. [节点基类文档](../weather-station-node-execution.md#节点生命周期)
3. 参考现有节点实现（推荐从简单节点开始）

### 节点开发清单

✅ **必须实现：**
- [ ] 继承 `NodeBase`
- [ ] 使用 `@register_node_type` 装饰器
- [ ] 实现 `execute()` 方法
- [ ] 实现 `_register_input_handles()` 方法（如有输入）

✅ **推荐实现：**
- [ ] 完善的错误处理
- [ ] 详细的日志记录
- [ ] 输入参数验证
- [ ] 单元测试

✅ **文档要求：**
- [ ] 创建节点详情文档（参考本目录模板）
- [ ] 更新本索引文件
- [ ] 添加使用示例

---

## 🔧 节点配置模板

### 基本配置

```json
{
  "id": "unique_node_id",
  "type": "node_type_here",
  "config": {
    "param1": "value1",
    "param2": "value2"
  }
}
```

### 带输入输出的配置

```json
{
  "nodes": [
    {
      "id": "node_1",
      "type": "binance_price_node",
      "config": {
        "symbol": "BTCUSDT"
      }
    },
    {
      "id": "node_2",
      "type": "ai_model_node",
      "config": {
        "model": "gpt-4"
      }
    }
  ],
  "edges": [
    {
      "source": "node_1",
      "sourceHandle": "kline_data",
      "target": "node_2",
      "targetHandle": "price_input"
    }
  ]
}
```

---

## 📊 节点统计

### 当前节点数量

- **数据输入节点**: 4 个
- **AI 处理节点**: 2 个
- **交易执行节点**: 4 个
- **Vault 管理节点**: 1 个
- **通知节点**: 1 个
- **总计**: 12 个节点类型

### 支持的区块链

- ✅ Aptos
- ✅ Flow EVM
- ✅ Ethereum
- ✅ BSC
- 🔜 Solana（规划中）
- 🔜 Sui（规划中）

---

## 🤝 贡献指南

### 添加新节点文档

1. 在本目录创建 `[node-name].md` 文件
2. 使用以下模板结构：
   ```markdown
   # Node Name
   **节点类型：** `node_type`
   **版本：** x.x.x
   **分类：** 分类名称
   
   ## 节点概述
   ## 输入输出
   ## 执行流程
   ## 使用示例
   ## 错误处理
   ## 配置要求
   ## 相关文档
   ```
3. 更新本索引文件，添加新节点条目
4. 提交 PR 等待审核

### 文档质量要求

- ✅ 清晰的节点功能说明
- ✅ 完整的参数表格
- ✅ 至少 2 个实际使用示例
- ✅ 详细的错误处理说明
- ✅ 相关文档链接

---

## 📚 相关资源

### 核心文档
- [Weather Station 架构概述](../weather-station-overview.md)
- [节点执行流程](../weather-station-node-execution.md)
- [消息队列详解](../weather-station-message-queue.md)
- [Redis 状态管理](../weather-station-redis.md)
- [Flow 调度机制](../weather-station-flow-scheduling.md)

### 代码位置
- 节点实现：`tradingflow/station/nodes/`
- 节点基类：`tradingflow/station/nodes/node_base.py`
- 节点注册：`tradingflow/station/common/node_registry.py`

### API 端点
- 节点列表：`GET /api/v1/nodes/types`
- 节点执行：`POST /api/v1/worker/execute`

---

**维护者：** TradingFlow 开发团队  
**联系方式：** 通过项目 Issue 反馈  
**最后更新：** 2025-10-06

# Weather Station 开发者文档

**版本：** 1.0.0  
**最后更新：** 2025-10-06

---

## 📚 快速开始

**从这里开始：** [Weather Station 文档索引](weather-station-index.md)

---

## 📂 文档结构

### 核心文档（本目录）

```
7_docs/developer/
├── weather-station-index.md              # 📖 主索引（从这里开始）
├── weather-station-overview.md           # 🏗️ 架构概述
├── weather-station-message-queue.md      # 📨 消息队列详解
├── weather-station-redis.md              # 💾 Redis 状态管理
├── weather-station-node-execution.md     # ⚙️ 节点执行流程
└── weather-station-flow-scheduling.md    # 🔄 Flow 调度机制
```

### 节点详情文档（独立页面）

**位置：** `7_docs/zh/node-details/`

每个节点都有独立的详细文档页面，包含完整的参数说明、使用示例和最佳实践。

**中文版本：**
```
7_docs/zh/node-details/
├── index.md                          # 节点文档索引
├── binance-price-node.md             ✅ Binance 价格节点
├── x-listener-node.md                ✅ X 监听节点
├── dataset-input-node.md             ✅ 数据集输入节点
├── ai-model-node.md                  ✅ AI 模型节点
├── code-node.md                      ✅ 代码节点
├── swap-node.md                      ✅ 交换节点
├── buy-node.md                       ✅ 买入节点
├── sell-node.md                      ✅ 卖出节点
├── vault-node.md                     ✅ Vault 节点
├── dataset-output-node.md            ✅ 数据集输出节点
└── telegram-sender-node.md           ✅ Telegram 发送节点
```

**英文版本：**
```
7_docs/en/node-details/
├── index.md                          # Node Documentation Index
└── binance-price-node.md             ✅ Binance Price Node
```

---

## 🚀 推荐阅读路径

### 完整学习路径（约 3 小时）

```
1. weather-station-index.md           # 了解整体架构
2. weather-station-overview.md        # 深入核心概念
3. weather-station-message-queue.md   # 理解通信机制
4. weather-station-redis.md           # 掌握状态管理
5. weather-station-node-execution.md  # 学习节点开发
6. weather-station-flow-scheduling.md # 理解调度机制
7. ../zh/node-details/index.md        # 查看具体节点
```

### 快速上手路径（约 50 分钟）

```
1. weather-station-overview.md        # 核心概念
2. weather-station-node-execution.md  # 节点开发
3. ../zh/node-details/index.md        # 节点示例
```

### 节点开发者路径

```
1. weather-station-overview.md        # 了解架构
2. weather-station-node-execution.md  # 开发新节点
3. ../zh/node-details/ai-model-node.md    # 参考示例
4. ../zh/node-details/binance-price-node.md
```

---

## 📊 文档统计

### 核心系统文档

| 文档 | 字数 | 状态 |
|------|------|------|
| 架构概述 | 17,500 | ✅ 完成 |
| 消息队列详解 | 22,000 | ✅ 完成 |
| Redis 状态管理 | 17,500 | ✅ 完成 |
| 节点执行流程 | 27,500 | ✅ 完成 |
| Flow 调度机制 | 24,500 | ✅ 完成 |
| **总计** | **~109,000** | **100%** |

### 节点详情文档

| 分类 | 中文文档 | 英文文档 |
|------|---------|---------|
| 数据输入节点 | 3 篇 | 1 篇 |
| AI 处理节点 | 2 篇 | 0 篇 |
| 交易执行节点 | 3 篇 | 0 篇 |
| Vault 管理节点 | 1 篇 | 0 篇 |
| 数据输出节点 | 1 篇 | 0 篇 |
| 通知节点 | 1 篇 | 0 篇 |
| **总计** | **12 篇** | **1 篇** |

---

## 🎯 文档导航

### 按主题查找

- **系统架构**: [weather-station-overview.md](weather-station-overview.md)
- **通信机制**: [weather-station-message-queue.md](weather-station-message-queue.md)
- **状态存储**: [weather-station-redis.md](weather-station-redis.md)
- **节点开发**: [weather-station-node-execution.md](weather-station-node-execution.md)
- **调度机制**: [weather-station-flow-scheduling.md](weather-station-flow-scheduling.md)
- **节点详情**: [../zh/node-details/index.md](../zh/node-details/index.md)

### 按角色查找

**新成员入门：**
```
主索引 → 架构概述 → 节点详情
```

**节点开发者：**
```
架构概述 → 节点执行流程 → 具体节点示例
```

**系统架构师：**
```
架构概述 → 消息队列 → Redis → Flow 调度
```

**运维人员：**
```
Redis 状态管理 → Flow 调度机制 → 节点执行流程
```

---

## 🔍 快速查询

### 常见问题

| 问题 | 文档位置 |
|------|---------|
| 如何创建新节点？ | [节点执行流程 - 开发新节点](weather-station-node-execution.md#开发新节点) |
| 信号如何传递？ | [消息队列详解 - 信号传递流程](weather-station-message-queue.md#信号传递流程) |
| Redis 键如何设计？ | [Redis 状态管理 - Key 命名规范](weather-station-redis.md#key-命名规范) |
| Flow 如何调度？ | [Flow 调度机制 - 周期调度](weather-station-flow-scheduling.md#周期调度) |
| 节点状态有哪些？ | [节点执行流程 - 节点生命周期](weather-station-node-execution.md#节点生命周期) |

### 节点快速查找

| 需求 | 推荐节点 |
|------|---------|
| 获取价格数据 | [Binance Price Node](../zh/node-details/binance-price-node.md) |
| AI 分析决策 | [AI Model Node](../zh/node-details/ai-model-node.md) |
| 自定义逻辑 | [Code Node](../zh/node-details/code-node.md) |
| 代币交换 | [Swap Node](../zh/node-details/swap-node.md) |
| 读取 Google Sheets | [Dataset Input Node](../zh/node-details/dataset-input-node.md) |
| 写入 Google Sheets | [Dataset Output Node](../zh/node-details/dataset-output-node.md) |
| 发送通知 | [Telegram Sender Node](../zh/node-details/telegram-sender-node.md) |

---

## 📖 相关资源

### 代码位置
- **节点实现**: `3_weather_cluster/tradingflow/station/nodes/`
- **节点基类**: `3_weather_cluster/tradingflow/station/nodes/node_base.py`
- **调度器**: `3_weather_cluster/tradingflow/station/flow/scheduler.py`
- **消息队列**: `4_weather_depot/tradingflow/depot/python/mq/`

### API 端点
- Weather Control API: `http://localhost:8000/api/v1/flow/*`
- Worker API: `http://localhost:8080/execute`
- 节点列表 API: `http://localhost:8000/api/v1/nodes/types`

---

## 🤝 贡献指南

### 添加新文档

1. 核心文档放在 `7_docs/developer/`
2. 节点详情文档放在 `7_docs/zh/node-details/`（中文）或 `7_docs/en/node-details/`（英文）
3. 更新主索引 `weather-station-index.md`
4. 更新节点索引 `../zh/node-details/index.md`

### 文档质量要求

- ✅ 清晰的标题和结构
- ✅ 完整的代码示例
- ✅ 实际可运行的示例
- ✅ 详细的参数说明
- ✅ 相关文档链接

---

**维护者：** TradingFlow 开发团队  
**反馈渠道：** 通过项目 Issue 反馈  
**最后更新：** 2025-10-06

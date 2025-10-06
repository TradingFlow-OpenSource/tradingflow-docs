# Weather Station 开发者文档索引

**版本：** 1.0.0  
**最后更新：** 2025-10-06

---

## 📚 文档目录

### 1. [架构概述](weather-station-overview.md)
**推荐阅读顺序：第 1 篇**

全面介绍 Weather Station 的整体架构、核心概念和设计模式。

**主要内容：**
- TFL、DAG、Flow、Cycle、Node 核心概念
- 系统架构和分层设计
- 关键组件介绍
- 技术栈和设计模式
- 数据流概览

**适合人群：** 所有开发者、架构师、新成员

---

### 2. [消息队列详解](weather-station-message-queue.md)
**推荐阅读顺序：第 2 篇**

深入讲解基于 RabbitMQ 的信号传递机制。

**主要内容：**
- RabbitMQ Topic Exchange 架构
- Routing Key 设计规范
- Queue 命名和绑定
- 信号传递完整流程
- Handle 匹配机制
- 特殊场景（聚合、通配符、停止信号）
- 实战示例

**适合人群：** 后端开发者、系统集成工程师

---

### 3. [Redis 状态管理](weather-station-redis.md)
**推荐阅读顺序：第 3 篇**

详细说明 Redis 状态存储的设计和使用。

**主要内容：**
- Redis 架构和数据结构
- Key 命名规范
- Flow、Cycle、Node 状态管理
- Worker 管理和心跳机制
- 状态查询接口
- 数据清理策略

**适合人群：** 后端开发者、运维工程师

---

### 4. [节点执行流程](weather-station-node-execution.md)
**推荐阅读顺序：第 4 篇**

完整讲解节点从创建到清理的生命周期。

**主要内容：**
- 节点生命周期和状态转换
- 节点创建和实例化
- 信号等待机制
- 执行流程（数据处理、条件判断、交易执行）
- 信号发送机制
- 异常处理和资源清理
- 开发新节点指南

**适合人群：** 节点开发者、业务逻辑实现者

---

### 5. [Flow 调度机制](weather-station-flow-scheduling.md)
**推荐阅读顺序：第 5 篇**

深入解析 FlowScheduler 的调度逻辑和 DAG 编排。

**主要内容：**
- FlowScheduler 架构和职责
- Flow 注册流程
- DAG 分析（连通分量、环检测、入口节点）
- 周期调度循环
- Cycle 执行和节点编排
- 状态管理和查询
- 日志系统

**适合人群：** 系统架构师、调度逻辑开发者

---

## 🚀 快速导航

### 按角色查看

#### 新成员入门
1. [架构概述](weather-station-overview.md) - 了解系统全貌
2. [节点执行流程](weather-station-node-execution.md) - 理解节点工作原理
3. [消息队列详解](weather-station-message-queue.md) - 掌握通信机制

#### 节点开发者
1. [节点执行流程](weather-station-node-execution.md) - 开发新节点
2. [消息队列详解](weather-station-message-queue.md) - 理解信号传递
3. [Redis 状态管理](weather-station-redis.md) - 使用状态存储

#### 系统架构师
1. [架构概述](weather-station-overview.md) - 整体架构
2. [Flow 调度机制](weather-station-flow-scheduling.md) - 调度设计
3. [Redis 状态管理](weather-station-redis.md) - 状态设计

#### 运维工程师
1. [Redis 状态管理](weather-station-redis.md) - 状态存储运维
2. [Flow 调度机制](weather-station-flow-scheduling.md) - 调度监控
3. [消息队列详解](weather-station-message-queue.md) - RabbitMQ 运维

### 按主题查看

#### 通信机制
- [消息队列详解](weather-station-message-queue.md)
- [节点执行流程 - 信号发送机制](weather-station-node-execution.md#信号发送机制)

#### 状态管理
- [Redis 状态管理](weather-station-redis.md)
- [Flow 调度机制 - 状态管理](weather-station-flow-scheduling.md#状态管理)

#### 编排调度
- [Flow 调度机制](weather-station-flow-scheduling.md)
- [架构概述 - 数据流](weather-station-overview.md#数据流)

#### 节点开发
- [节点执行流程 - 开发新节点](weather-station-node-execution.md#开发新节点)
- [架构概述 - 关键组件](weather-station-overview.md#关键组件)

---

## 📖 阅读建议

### 完整学习路径（推荐）
适合第一次接触 Weather Station 的开发者。

```
1. 架构概述 (30 分钟)
   ↓
2. 消息队列详解 (40 分钟)
   ↓
3. Redis 状态管理 (30 分钟)
   ↓
4. 节点执行流程 (40 分钟)
   ↓
5. Flow 调度机制 (40 分钟)
```

**总时长：** 约 3 小时

### 快速上手路径
适合需要快速开始开发的人员。

```
1. 架构概述 - 核心概念 (15 分钟)
   ↓
2. 节点执行流程 - 开发新节点 (20 分钟)
   ↓
3. 消息队列详解 - 实战示例 (15 分钟)
```

**总时长：** 约 50 分钟

### 问题驱动路径
根据具体问题快速定位文档。

| 问题 | 查看文档 |
|------|---------|
| 如何开发新节点？ | [节点执行流程 - 开发新节点](weather-station-node-execution.md#开发新节点) |
| 信号如何传递？ | [消息队列详解 - 信号传递流程](weather-station-message-queue.md#信号传递流程) |
| 如何查询节点状态？ | [Redis 状态管理 - 状态查询](weather-station-redis.md#状态查询) |
| Flow 如何调度？ | [Flow 调度机制 - 周期调度](weather-station-flow-scheduling.md#周期调度) |
| 如何处理聚合数据？ | [消息队列详解 - Handle 匹配机制](weather-station-message-queue.md#handle-匹配机制) |
| DAG 如何分析？ | [Flow 调度机制 - DAG 分析](weather-station-flow-scheduling.md#dag-分析) |
| 节点如何停止？ | [节点执行流程 - 异常处理](weather-station-node-execution.md#异常处理) |
| Redis Key 设计？ | [Redis 状态管理 - Key 命名规范](weather-station-redis.md#key-命名规范) |

---

## 📦 节点详情文档

Weather Station 提供 12+ 种节点类型，每个节点都有独立的详细文档页面。

### 🔗 [节点文档索引](nodes/README.md)

**按分类浏览：**

#### 数据输入节点
- **[Binance Price Node](nodes/binance-price-node.md)** - 获取 Binance 交易所 K 线数据
- **[X Listener Node](nodes/x-listener-node.md)** - 监听 Twitter/X 平台动态
- **[RSSHub Node](nodes/rsshub-node.md)** - 订阅 RSS 新闻资讯
- **[Dataset Node](nodes/dataset-node.md)** - Google Sheets 数据读写

#### AI 处理节点
- **[AI Model Node](nodes/ai-model-node.md)** - 调用 LLM 进行智能分析和决策
- **[Code Node](nodes/code-node.md)** - 执行自定义 Python 代码

#### 交易执行节点
- **[Swap Node](nodes/swap-node.md)** - 多链代币交换（Aptos、Flow EVM）
- **[Buy Node](nodes/buy-node.md)** - 专用买入节点
- **[Sell Node](nodes/sell-node.md)** - 专用卖出节点
- **[Uniswap DEX Trade Node](nodes/uniswap-dex-trade-node.md)** - Uniswap 交易

#### Vault 管理节点
- **[Vault Node](nodes/vault-node.md)** - 查询和管理 Vault 信息

#### 通知节点
- **[Telegram Sender Node](nodes/telegram-sender-node.md)** - 发送 Telegram 通知

### 📖 查看完整节点列表

访问 **[节点文档目录](nodes/README.md)** 查看所有节点的详细文档，包括：
- 完整的参数说明
- 输入输出格式
- 执行流程图
- 实际使用示例
- 错误处理指南
- 性能优化建议

---

## 🔧 相关资源

### 代码位置
- **调度器**: `tradingflow/station/flow/scheduler.py`
- **Flow 解析器**: `tradingflow/station/flow/flow_parser.py`
- **节点基类**: `tradingflow/station/nodes/node_base.py`
- **节点执行器**: `tradingflow/station/core/node_executor.py`
- **节点实现**: `tradingflow/station/nodes/*_node.py`
- **消息队列**: `tradingflow/depot/python/mq/`
- **状态存储**: `tradingflow/station/common/state_store.py`
- **任务管理**: `tradingflow/station/common/node_task_manager.py`

### API 文档
- Weather Control API: `/api/v1/flow/*`
- Worker API: `/execute`
- 节点列表 API: `/api/v1/nodes/types`

### 配置文件
- 环境变量: `.env`
- RabbitMQ 配置: `CONFIG.RABBITMQ_URL`
- Redis 配置: `CONFIG.REDIS_URL`

---

## 📝 文档更新日志

### v1.0.0 (2025-10-06)
- ✅ 创建完整的 Weather Station 开发者文档
- ✅ 架构概述
- ✅ 消息队列详解
- ✅ Redis 状态管理
- ✅ 节点执行流程
- ✅ Flow 调度机制
- ✅ 节点详情文档（12+ 节点）
  - ✅ Binance Price Node
  - ✅ AI Model Node
  - ✅ Swap Node
  - ✅ 节点文档索引
- ✅ 主文档索引

---

## 🤝 贡献指南

如果您发现文档中的错误或有改进建议，请：

1. 创建 Issue 描述问题
2. 提交 PR 修复文档
3. 联系文档维护者

---

**维护者：** TradingFlow 开发团队  
**联系方式：** 通过项目 Issue 反馈  
**最后更新：** 2025-10-06

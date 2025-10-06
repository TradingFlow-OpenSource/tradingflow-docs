# Weather Station 文档目录结构

**版本：** 1.0.0  
**最后更新：** 2025-10-06

---

## 📁 完整目录结构

```
7_docs/developer/
├── weather-station-index.md              # 📚 主索引（从这里开始）
├── weather-station-overview.md           # 🏗️ 架构概述
├── weather-station-message-queue.md      # 📨 消息队列详解
├── weather-station-redis.md              # 💾 Redis 状态管理
├── weather-station-node-execution.md     # ⚙️ 节点执行流程
├── weather-station-flow-scheduling.md    # 🔄 Flow 调度机制
├── DIRECTORY_STRUCTURE.md                # 📂 本文件
└── nodes/                                # 📦 节点详情目录
    ├── README.md                         # 节点索引
    ├── binance-price-node.md             # Binance 价格节点
    ├── ai-model-node.md                  # AI 模型节点
    ├── swap-node.md                      # 交换节点
    ├── buy-node.md                       # 买入节点（待创建）
    ├── sell-node.md                      # 卖出节点（待创建）
    ├── code-node.md                      # 代码节点（待创建）
    ├── dataset-node.md                   # 数据集节点（待创建）
    ├── x-listener-node.md                # X 监听节点（待创建）
    ├── rsshub-node.md                    # RSSHub 节点（待创建）
    ├── vault-node.md                     # Vault 节点（待创建）
    ├── telegram-sender-node.md           # Telegram 发送节点（待创建）
    └── uniswap-dex-trade-node.md         # Uniswap 交易节点（待创建）
```

---

## 📖 文档说明

### 核心文档（6 篇）

| 文档名 | 文件名 | 字数 | 状态 |
|--------|--------|------|------|
| **主索引** | `weather-station-index.md` | ~7,000 | ✅ 完成 |
| **架构概述** | `weather-station-overview.md` | ~17,500 | ✅ 完成 |
| **消息队列详解** | `weather-station-message-queue.md` | ~22,000 | ✅ 完成 |
| **Redis 状态管理** | `weather-station-redis.md` | ~17,500 | ✅ 完成 |
| **节点执行流程** | `weather-station-node-execution.md` | ~27,500 | ✅ 完成 |
| **Flow 调度机制** | `weather-station-flow-scheduling.md` | ~24,500 | ✅ 完成 |

**总计：** 约 116,000 字

### 节点详情文档（12+ 篇）

| 节点类型 | 文档状态 | 文件名 |
|---------|---------|--------|
| **Binance Price Node** | ✅ 完成 | `binance-price-node.md` |
| **AI Model Node** | ✅ 完成 | `ai-model-node.md` |
| **Swap Node** | ✅ 完成 | `swap-node.md` |
| **Buy Node** | 🔜 计划中 | `buy-node.md` |
| **Sell Node** | 🔜 计划中 | `sell-node.md` |
| **Code Node** | 🔜 计划中 | `code-node.md` |
| **Dataset Node** | 🔜 计划中 | `dataset-node.md` |
| **X Listener Node** | 🔜 计划中 | `x-listener-node.md` |
| **RSSHub Node** | 🔜 计划中 | `rsshub-node.md` |
| **Vault Node** | 🔜 计划中 | `vault-node.md` |
| **Telegram Sender Node** | 🔜 计划中 | `telegram-sender-node.md` |
| **Uniswap DEX Trade Node** | 🔜 计划中 | `uniswap-dex-trade-node.md` |

**已完成：** 3 篇  
**计划中：** 9 篇

---

## 🚀 快速导航

### 从这里开始

1. **新成员入门**: [weather-station-index.md](weather-station-index.md)
2. **了解架构**: [weather-station-overview.md](weather-station-overview.md)
3. **开发节点**: [weather-station-node-execution.md](weather-station-node-execution.md)
4. **查看节点**: [nodes/README.md](nodes/README.md)

### 按主题查找

- **系统架构**: `weather-station-overview.md`
- **通信机制**: `weather-station-message-queue.md`
- **状态存储**: `weather-station-redis.md`
- **节点开发**: `weather-station-node-execution.md`
- **调度机制**: `weather-station-flow-scheduling.md`
- **节点详情**: `nodes/*.md`

---

## 📊 文档覆盖度

### 核心系统文档

| 模块 | 覆盖度 | 文档 |
|------|--------|------|
| **调度系统** | ✅ 100% | FlowScheduler, FlowParser |
| **节点系统** | ✅ 100% | NodeBase, NodeExecutor |
| **消息队列** | ✅ 100% | RabbitMQ, Signal, Handle |
| **状态存储** | ✅ 100% | Redis, StateStore |
| **任务管理** | ✅ 100% | NodeTaskManager, NodeRegistry |

### 节点类型文档

| 分类 | 已完成 | 计划中 | 完成度 |
|------|--------|--------|--------|
| **数据输入节点** | 1/4 | 3 | 25% |
| **AI 处理节点** | 1/2 | 1 | 50% |
| **交易执行节点** | 1/4 | 3 | 25% |
| **Vault 管理节点** | 0/1 | 1 | 0% |
| **通知节点** | 0/1 | 1 | 0% |

**总体完成度：** 25% (3/12)

---

## 🎯 文档路线图

### Phase 1 - 核心文档 ✅
- [x] 架构概述
- [x] 消息队列详解
- [x] Redis 状态管理
- [x] 节点执行流程
- [x] Flow 调度机制
- [x] 主索引文档

### Phase 2 - 核心节点文档（当前）
- [x] Binance Price Node
- [x] AI Model Node
- [x] Swap Node
- [ ] Code Node
- [ ] Dataset Node

### Phase 3 - 扩展节点文档
- [ ] Buy Node
- [ ] Sell Node
- [ ] Vault Node
- [ ] X Listener Node
- [ ] RSSHub Node

### Phase 4 - 高级节点文档
- [ ] Telegram Sender Node
- [ ] Uniswap DEX Trade Node
- [ ] 更多节点类型...

### Phase 5 - 开发者指南
- [ ] 节点开发完整指南
- [ ] 最佳实践文档
- [ ] 性能优化指南
- [ ] 故障排查手册

---

## 📝 文档贡献指南

### 创建新节点文档

1. **复制模板**：使用现有节点文档作为模板
2. **填写内容**：
   - 节点概述
   - 输入输出参数
   - 执行流程
   - 使用示例（至少 2 个）
   - 错误处理
   - 配置要求
3. **更新索引**：在 `nodes/README.md` 中添加条目
4. **提交审核**：创建 PR 等待审核

### 文档质量标准

- ✅ 清晰的标题和结构
- ✅ 完整的参数表格
- ✅ 实际可运行的示例
- ✅ 详细的错误说明
- ✅ 相关文档链接
- ✅ 代码片段有语法高亮

---

## 🔗 相关资源

### 外部文档链接
- [Binance API 文档](https://binance-docs.github.io/apidocs/)
- [OpenRouter API 文档](https://openrouter.ai/docs)
- [RabbitMQ 文档](https://www.rabbitmq.com/documentation.html)
- [Redis 文档](https://redis.io/documentation)

### 内部代码库
- 节点实现：`tradingflow/station/nodes/`
- 测试文件：`tradingflow/station/tests/`
- 配置文件：`tradingflow/depot/python/config.py`

---

**维护者：** TradingFlow 开发团队  
**最后更新：** 2025-10-06

---

## 💡 使用建议

### 阅读顺序建议

**完整学习路径**（推荐新成员）:
```
主索引 → 架构概述 → 消息队列 → Redis → 节点执行 → Flow 调度 → 节点详情
```

**快速上手路径**（有经验的开发者）:
```
架构概述（核心概念）→ 节点执行流程 → 节点详情文档
```

**问题驱动路径**（解决具体问题）:
```
主索引（问题查找表）→ 对应章节 → 节点详情
```

### 文档更新频率

- **核心文档**: 每季度更新一次
- **节点文档**: 节点变更时立即更新
- **索引文档**: 新文档添加时立即更新

### 反馈渠道

- **错误报告**: 创建 Issue 标记为 `documentation`
- **改进建议**: 创建 Issue 标记为 `enhancement`
- **新文档请求**: 创建 Issue 标记为 `new-doc`

---

**感谢您使用 Weather Station 文档！**

# Weather Station Documentation Index

**Version:** 1.0.0  
**Last Updated:** 2025-10-06

---

## 📖 About This Documentation

This is the comprehensive developer documentation for TradingFlow's **Weather Station** - the core distributed execution framework for building and executing DAG (Directed Acyclic Graph) workflows.

**Note:** The detailed English translations of the core documentation are in progress. Chinese versions are complete and available now.

---

## 🗺️ Documentation Structure

### 🏗️ [Architecture Overview](weather-station-overview.md)
**~17,500 words | Chinese ✅ | English 🔄**

Comprehensive introduction to Weather Station's architecture, core concepts, and system design.

**Key Topics:**
- Core concepts (TFL, DAG, Flow, Cycle, Node, Signal, Handle)
- System architecture and layered design
- 8 key components explained
- Technology stack and 5 design patterns
- Complete data flow description

---

### 📨 [Message Queue Details](weather-station-message-queue.md)
**~22,000 words | Chinese ✅ | English 🔄**

Deep dive into the RabbitMQ-based message queue system and signal passing mechanism.

**Key Topics:**
- RabbitMQ Topic Exchange architecture
- Routing Key design specifications
- Queue naming and binding mechanisms
- Signal transmission complete flow (9 steps)
- Handle matching and aggregation mechanisms
- 4 special scenarios handling
- 3 practical examples

---

### 💾 [Redis State Management](weather-station-redis.md)
**~17,500 words | Chinese ✅ | English 🔄**

Complete guide to Redis-based state storage and management.

**Key Topics:**
- Redis architecture and data structures
- 10 types of Key naming conventions
- Flow, Cycle, Node state management
- Worker management and heartbeat mechanism
- 3 types of state query interfaces
- TTL and data cleanup strategies

---

### ⚙️ [Node Execution Flow](weather-station-node-execution.md)
**~27,500 words | Chinese ✅ | English 🔄**

Detailed explanation of node lifecycle, execution process, and development guide.

**Key Topics:**
- 7-stage node lifecycle
- Node creation and instantiation
- Signal waiting mechanism
- 3 execution modes (data processing, condition judgment, trade execution)
- Signal sending and retrieval
- 5 types of exception handling
- Complete guide for developing new nodes

---

### 🔄 [Flow Scheduling Mechanism](weather-station-flow-scheduling.md)
**~24,500 words | Chinese ✅ | English 🔄**

In-depth analysis of FlowScheduler and flow orchestration mechanisms.

**Key Topics:**
- FlowScheduler architecture and responsibilities
- Flow registration process
- DAG analysis (BFS connected components, DFS cycle detection, entry node identification)
- Cycle scheduling loop
- Component and node orchestration
- 3 types of state query interfaces
- Logging system and cleanup

---

## 🚀 Reading Recommendations

### For Newcomers (Complete Learning ~3 hours)

```
1. Architecture Overview          → Understand core concepts
2. Message Queue Details          → Master communication
3. Redis State Management         → Learn state storage
4. Node Execution Flow            → Node development
5. Flow Scheduling Mechanism      → Understand orchestration
6. Node Details                   → View specific nodes
```

### For Node Developers (Quick Start ~50 minutes)

```
1. Architecture Overview (Core Concepts)
2. Node Execution Flow (Develop New Nodes)
3. Node Details (Practical Examples)
```

### For System Architects (Problem-Driven)

```
Use the Quick Reference Table below to jump to relevant sections
```

---

## 🔍 Quick Reference

### Common Questions

| Question | Document Location |
|----------|------------------|
| How to create a new node? | [Node Execution - Developing New Nodes](weather-station-node-execution.md#developing-new-nodes) |
| How are signals transmitted? | [Message Queue - Signal Transmission Flow](weather-station-message-queue.md#signal-transmission-flow) |
| How to query node status? | [Redis State Management - State Queries](weather-station-redis.md#state-queries) |
| How is Flow scheduled? | [Flow Scheduling - Cycle Scheduling](weather-station-flow-scheduling.md#cycle-scheduling) |
| How to handle aggregated data? | [Message Queue - Handle Matching Mechanism](weather-station-message-queue.md#handle-matching) |
| How is DAG analyzed? | [Flow Scheduling - DAG Analysis](weather-station-flow-scheduling.md#dag-analysis) |
| How to stop a node? | [Node Execution - Exception Handling](weather-station-node-execution.md#exception-handling) |
| Redis Key design? | [Redis State Management - Key Naming Conventions](weather-station-redis.md#key-naming) |

---

## 📦 Node Details Documentation

Weather Station provides 12+ node types, each with its own detailed documentation page.

### 🔗 [Node Documentation Index](../node-details/index.md)

**Browse by Category:**

#### Data Input Nodes
- **[Binance Price Node](../node-details/binance-price-node.md)** ✅ - Fetch K-line data from Binance exchange
- **[X Listener Node](../node-details/x-listener-node.md)** 🔄 - Monitor Twitter/X platform
- **[Dataset Input Node](../node-details/dataset-input-node.md)** 🔄 - Read data from Google Sheets

#### AI Processing Nodes
- **[AI Model Node](../node-details/ai-model-node.md)** 🔄 - Call LLM for intelligent analysis and decision-making
- **[Code Node](../node-details/code-node.md)** 🔄 - Execute custom Python code

#### Trade Execution Nodes
- **[Swap Node](../node-details/swap-node.md)** 🔄 - Multi-chain token swap (Aptos, Flow EVM)
- **[Buy Node](../node-details/buy-node.md)** 🔄 - Dedicated buy operation node
- **[Sell Node](../node-details/sell-node.md)** 🔄 - Dedicated sell operation node

#### Vault Management Nodes
- **[Vault Node](../node-details/vault-node.md)** 🔄 - Query and manage Vault information

#### Data Output Nodes
- **[Dataset Output Node](../node-details/dataset-output-node.md)** 🔄 - Write data to Google Sheets

#### Notification Nodes
- **[Telegram Sender Node](../node-details/telegram-sender-node.md)** 🔄 - Send Telegram notifications

**Legend:**
- ✅ Full documentation available
- 🔄 Translation in progress (placeholder with Chinese version link)

### 📖 View Complete Node List

Visit **[Node Documentation Index](../node-details/index.md)** for all node documentation including:
- Complete parameter descriptions
- Input/output formats
- Execution flow diagrams
- Practical usage examples
- Error handling guides
- Performance optimization tips

---

## 🔧 Related Resources

### Code Locations
- **Scheduler**: `tradingflow/station/flow/scheduler.py`
- **Flow Parser**: `tradingflow/station/flow/flow_parser.py`
- **Node Base Class**: `tradingflow/station/nodes/node_base.py`
- **Node Executor**: `tradingflow/station/core/node_executor.py`
- **Node Implementations**: `tradingflow/station/nodes/*_node.py`
- **Message Queue**: `tradingflow/depot/python/mq/`
- **State Storage**: `tradingflow/station/common/state_store.py`
- **Task Management**: `tradingflow/station/common/node_task_manager.py`

### API Documentation
- Weather Control API: `/api/v1/flow/*`
- Worker API: `/execute`
- Node List API: `/api/v1/nodes/types`

### Configuration Files
- Environment Variables: `.env`
- RabbitMQ Config: `CONFIG.RABBITMQ_URL`
- Redis Config: `CONFIG.REDIS_URL`

---

## 📝 Documentation Changelog

### v1.0.0 (2025-10-06)
- ✅ Complete Weather Station developer documentation created
- ✅ Architecture Overview (17,500 words in Chinese)
- ✅ Message Queue Details (22,000 words in Chinese)
- ✅ Redis State Management (17,500 words in Chinese)
- ✅ Node Execution Flow (27,500 words in Chinese)
- ✅ Flow Scheduling Mechanism (24,500 words in Chinese)
- ✅ Main documentation index (navigation optimized)
- ✅ Links to existing node details documentation (12+ nodes)
  - ✅ Data Input Nodes (3 types)
  - ✅ AI Processing Nodes (2 types)
  - ✅ Trade Execution Nodes (3 types)
  - ✅ Vault Management Nodes (1 type)
  - ✅ Data Output Nodes (1 type)
  - ✅ Notification Nodes (1 type)
- 🔄 English translations in progress

---

## 🌐 Multi-language Support

- **🇨🇳 Chinese (完整版本)**: [../../zh/engineering-docs/weather-station-index.md](../../zh/engineering-docs/weather-station-index.md)
- **🇬🇧 English (This document)**: English translations coming soon

---

## 🤝 Contributing

If you find errors in the documentation or have suggestions for improvement:

1. Create an Issue describing the problem
2. Submit a PR to fix the documentation
3. Contact the documentation maintainers

---

**Maintainers:** TradingFlow Development Team  
**Last Updated:** 2025-10-06

# Weather Station Developer Documentation

**Version:** 1.0.0  
**Last Updated:** 2025-10-06

---

## 📚 Quick Start

**Start here:** [Weather Station Documentation Index](weather-station-index.md)

---

## 📂 Documentation Structure

### Core Documentation (This Directory)

```
7_docs/en/engineering-docs/
├── weather-station-index.md              # 📖 Main Index (Start Here)
├── weather-station-overview.md           # 🏗️ Architecture Overview
├── weather-station-message-queue.md      # 📨 Message Queue Details
├── weather-station-redis.md              # 💾 Redis State Management
├── weather-station-node-execution.md     # ⚙️ Node Execution Flow
└── weather-station-flow-scheduling.md    # 🔄 Flow Scheduling
```

### Node Details Documentation (Separate Pages)

**Location:** `7_docs/en/node-details/`

Each node has its own detailed documentation page with complete parameter descriptions, usage examples, and best practices.

**Available:**
```
7_docs/en/node-details/
├── index.md                          # Node Documentation Index
└── binance-price-node.md             ✅ Binance Price Node
```

---

## 🚀 Recommended Reading Paths

### Complete Learning Path (~3 hours)

```
1. weather-station-index.md           # Understand overall architecture
2. weather-station-overview.md        # Deep dive into core concepts
3. weather-station-message-queue.md   # Understand communication
4. weather-station-redis.md           # Master state management
5. weather-station-node-execution.md  # Learn node development
6. weather-station-flow-scheduling.md # Understand scheduling
7. ../node-details/index.md           # View specific nodes
```

### Quick Start Path (~50 minutes)

```
1. weather-station-overview.md        # Core concepts
2. weather-station-node-execution.md  # Node development
3. ../node-details/index.md           # Node examples
```

### Node Developer Path

```
1. weather-station-overview.md        # Understand architecture
2. weather-station-node-execution.md  # Develop new nodes
3. ../node-details/binance-price-node.md # Reference examples
```

---

## 📊 Documentation Statistics

### Core System Documentation

| Document | Words | Status |
|----------|-------|--------|
| Architecture Overview | 17,500 | ✅ Complete (Chinese) |
| Message Queue Details | 22,000 | ✅ Complete (Chinese) |
| Redis State Management | 17,500 | ✅ Complete (Chinese) |
| Node Execution Flow | 27,500 | ✅ Complete (Chinese) |
| Flow Scheduling | 24,500 | ✅ Complete (Chinese) |
| **Total** | **~109,000** | **🔄 English version coming soon** |

### Node Details Documentation

| Category | English Docs |
|----------|--------------|
| Data Input Nodes | 1 page |
| AI Processing Nodes | Coming soon |
| Trade Execution Nodes | Coming soon |
| Vault Management Nodes | Coming soon |
| Data Output Nodes | Coming soon |
| Notification Nodes | Coming soon |

---

## 🎯 Documentation Navigation

### By Topic

- **System Architecture**: [weather-station-overview.md](weather-station-overview.md)
- **Communication**: [weather-station-message-queue.md](weather-station-message-queue.md)
- **State Storage**: [weather-station-redis.md](weather-station-redis.md)
- **Node Development**: [weather-station-node-execution.md](weather-station-node-execution.md)
- **Scheduling**: [weather-station-flow-scheduling.md](weather-station-flow-scheduling.md)
- **Node Details**: [../node-details/index.md](../node-details/index.md)

### By Role

**Newcomers:**
```
Main Index → Architecture Overview → Node Details
```

**Node Developers:**
```
Architecture Overview → Node Execution → Node Examples
```

**System Architects:**
```
Architecture → Message Queue → Redis → Flow Scheduling
```

**Operations:**
```
Redis State Management → Flow Scheduling → Node Execution
```

---

## 📖 Related Resources

### Code Locations
- **Node Implementations**: `3_weather_cluster/tradingflow/station/nodes/`
- **Node Base Class**: `3_weather_cluster/tradingflow/station/nodes/node_base.py`
- **Scheduler**: `3_weather_cluster/tradingflow/station/flow/scheduler.py`
- **Message Queue**: `4_weather_depot/tradingflow/depot/python/mq/`

### API Endpoints
- Weather Control API: `http://localhost:8000/api/v1/flow/*`
- Worker API: `http://localhost:8080/execute`
- Node List API: `http://localhost:8000/api/v1/nodes/types`

---

## 🌐 Language Support

This documentation is available in multiple languages:

- **🇨🇳 中文版本**: [../zh/engineering-docs/README.md](../../zh/engineering-docs/README.md)
- **🇬🇧 English Version**: This document (English translation in progress)

---

## 🤝 Contributing

### Adding New Documentation

1. Core docs go in `7_docs/en/engineering-docs/`
2. Node details go in `7_docs/en/node-details/`
3. Update main index `weather-station-index.md`
4. Update node index `../node-details/index.md`

### Documentation Quality Standards

- ✅ Clear titles and structure
- ✅ Complete code examples
- ✅ Runnable sample code
- ✅ Detailed parameter descriptions
- ✅ Related documentation links

---

**Maintainers:** TradingFlow Development Team  
**Feedback:** Submit issues via project Issue tracker  
**Last Updated:** 2025-10-06

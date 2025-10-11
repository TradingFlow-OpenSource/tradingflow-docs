# Weather Station Trading Engine

**Version:** 1.0.0  
**Last Updated:** 2025-10-06

---

## 🚀 About Weather Station

Weather Station is TradingFlow's core trading execution engine, providing **high-performance, distributed, and reliable** automated execution capabilities for quantitative trading strategies.

**Core Values:**
- 🎯 **Intelligent Trading Automation** - Build complex trading strategies through visual nodes
- ⚡ **High-Performance Execution** - Distributed architecture supports millisecond-level trading response
- 🔒 **Enterprise-Grade Reliability** - Comprehensive state management and exception recovery
- 🌐 **Multi-Chain Support** - Native support for Aptos, Flow EVM, and more blockchains

---

## 📚 Quick Start

**Start here:** [Weather Station Documentation Index](weather-station-index.md)

---

## 📂 Documentation Structure

### Core Technical Documentation (This Directory)

```
7_docs/en/engineering-docs/
├── weather-station-index.md              # 📖 Main Index (Start Here)
├── weather-station-overview.md           # 🏗️ Engine Architecture Overview
├── weather-station-message-queue.md      # 📨 Trading Signal Transmission
├── weather-station-redis.md              # 💾 State Management System
├── weather-station-node-execution.md     # ⚙️ Trading Node Execution Flow
├── weather-station-flow-scheduling.md    # 🔄 Strategy Scheduling Mechanism
├── community-nodes-system.md             # 🌐 Community Nodes & Version Control
├── quest-system.md                       # 🎮 Quest System & Gamification
└── co-trading-system.md                  # 👥 Co-Trading (Social Trading Platform)
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

### Business Users/Product Managers (~1 hour)

```
1. weather-station-index.md           # Understand trading engine capabilities
2. weather-station-overview.md        # Learn core architecture and advantages
3. ../node-details/index.md           # View available trading nodes
```

### Complete Technical Path (~3 hours)

```
1. weather-station-index.md           # Understand overall architecture
2. weather-station-overview.md        # Deep dive into core concepts
3. weather-station-message-queue.md   # Understand trading signal transmission
4. weather-station-redis.md           # Master state management
5. weather-station-node-execution.md  # Learn node execution mechanism
6. weather-station-flow-scheduling.md # Understand strategy scheduling
7. ../node-details/index.md           # View specific node implementations
```

### Strategy Developer Path (~50 minutes)

```
1. weather-station-overview.md        # Core concepts
2. weather-station-node-execution.md  # Node usage methods
3. ../node-details/index.md           # Node capability quick reference
```

---

## 📊 Documentation Statistics

### Core Technical Documentation

| Document | Words | Status |
|----------|-------|--------|
| Engine Architecture Overview | 17,500 | ✅ Available |
| Trading Signal Transmission | 22,000 | ✅ Available |
| State Management System | 17,500 | ✅ Available |
| Strategy Scheduling Mechanism | 24,500 | ✅ Available |
| **Total** | **~109,000** | **All Complete** |

### Trading Node Documentation

| Category | Documentation |
|----------|---------------|
| Data Collection Nodes | 3 nodes |
| AI Intelligence Analysis | 2 nodes |
| Trade Execution Nodes | 3 nodes |
| Asset Management Nodes | 1 node |
| Data Output Nodes | 1 node |
| Message Notification Nodes | 1 node |
| **Total** | **11 nodes** |

---

## 🎯 Documentation Navigation

### By Topic

- **System Architecture**: [weather-station-overview.md](weather-station-overview.md)
- **Communication**: [weather-station-message-queue.md](weather-station-message-queue.md)
- **State Storage**: [weather-station-redis.md](weather-station-redis.md)
- **Node Development**: [weather-station-node-execution.md](weather-station-node-execution.md)
- **Scheduling**: [weather-station-flow-scheduling.md](weather-station-flow-scheduling.md)
- **Community Nodes**: [community-nodes-system.md](community-nodes-system.md)
- **Quest System**: [quest-system.md](quest-system.md)
- **Co-Trading**: [co-trading-system.md](co-trading-system.md)
- **Node Details**: [../node-details/index.md](../node-details/index.md)

### By Role

**Business Users/Product Managers:**
```
Main Index → Engine Architecture → Trading Node Capabilities
```

**Strategy Developers:**
```
Engine Architecture → Node Usage Guide → Specific Node Examples
```

**Technical Architects:**
```
Architecture Overview → Signal Transmission → State Management → Scheduling System
```

**Operations Engineers:**
```
State Management System → Scheduling Mechanism → Execution Flow Monitoring
```

**Community Contributors:**
```
Community Nodes System → Node Creation Guide → Publishing & Sharing
```

---

## 📖 Related Resources

### Code Locations
- **Node Implementations**: `3_weather_cluster/tradingflow/station/nodes/`
- **Node Base Class**: `3_weather_cluster/tradingflow/station/nodes/node_base.py`
- **Scheduler**: `3_weather_cluster/tradingflow/station/flow/scheduler.py`
- **Message Queue**: `4_weather_depot/tradingflow/depot/python/mq/`
- **Community Nodes (Station)**: `03_weather_station/core/` (version_manager, node_registry, community_node_executor)
- **Community Nodes (Control)**: `02_weather_control/src/models/`, `src/services/`, `src/routes/community/`
- **Community Nodes (Frontend)**: `01_weather_frontend/src/pages/` (CommunityNodesPage, CommunityNodeDetailPage)

### API Endpoints
- Weather Control API: `http://localhost:8000/api/v1/flow/*`
- Worker API: `http://localhost:8080/execute`
- Node List API: `http://localhost:8000/api/v1/nodes/types`
- Community Nodes API: `http://localhost:8000/api/v1/community/nodes`
- Community Comments API: `http://localhost:8000/api/v1/community/nodes/:nodeId/comments`

---

## 🌐 Language Support

This documentation is available in multiple languages:

- **🇨🇳 Chinese**: [../zh/engineering-docs/README.md](../../zh/engineering-docs/README.md)
- **🇬🇧 English**: This document

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

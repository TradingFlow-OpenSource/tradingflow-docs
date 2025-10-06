# Weather Station Architecture Overview

**Version:** 1.0.0  
**Document Type:** Technical Architecture  
**Last Updated:** 2025-10-06

---

## 📖 About This Document

This document provides a comprehensive overview of Weather Station's architecture, core concepts, and system design. It serves as the foundation for understanding how TradingFlow's trading execution engine operates.

**Target Audience:**
- Technical Architects
- Strategy Developers
- Backend Engineers
- System Operators

**Reading Time:** ~50 minutes

---

## 🎯 What is Weather Station?

Weather Station is TradingFlow's **high-performance, distributed trading execution engine** that enables automated quantitative trading strategies through visual workflow design.

**Core Capabilities:**
- ⚡ **High-Performance Execution** - Millisecond-level trade execution with distributed architecture
- 🎯 **Visual Strategy Building** - Drag-and-drop node-based workflow design
- 🔒 **Enterprise Reliability** - Robust state management and failure recovery
- 🌐 **Multi-Chain Support** - Native support for Aptos, Flow EVM, and more blockchains
- 📊 **Real-Time Monitoring** - Complete observability of strategy execution

---

## 🏗️ System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Weather Control                           │
│              (Frontend + API Gateway)                        │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   Weather Station                            │
│            (Distributed Trading Engine)                      │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Flow         │  │ Node         │  │ Signal       │      │
│  │ Scheduler    │◄─┤ Executor     │◄─┤ Router       │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│         │                 │                  │               │
│         └─────────────────┴──────────────────┘               │
│                           │                                   │
└───────────────────────────┼───────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
    ┌──────────┐     ┌──────────┐    ┌──────────┐
    │ RabbitMQ │     │  Redis   │    │PostgreSQL│
    │(Messages)│     │ (State)  │    │  (Data)  │
    └──────────┘     └──────────┘    └──────────┘
```

### Core Components

**1. Flow Scheduler**
- Manages strategy execution lifecycle
- Handles cycle-based scheduling
- Coordinates node execution

**2. Node Executor**
- Executes individual trading nodes
- Manages node state transitions
- Handles exceptions and retries

**3. Signal Router**
- Routes data signals between nodes
- Manages message queues
- Ensures reliable delivery

**4. State Manager**
- Persists execution state to Redis
- Enables distributed coordination
- Supports failure recovery

---

## 💡 Core Concepts

### 1. Flow (Trading Strategy)

A **Flow** represents a complete trading strategy defined as a Directed Acyclic Graph (DAG) of nodes.

**Key Properties:**
- **ID**: Unique identifier
- **Name**: Human-readable strategy name
- **Schedule**: Execution frequency (periodic or manual)
- **Nodes**: Collection of processing nodes
- **Edges**: Connections between nodes defining data flow

**Example Use Cases:**
- Market-making strategy with price monitoring
- Arbitrage bot across multiple DEXs
- Social sentiment-based trading signals

### 2. Node (Processing Unit)

A **Node** is an atomic processing unit that performs a specific task in the trading workflow.

**Node Categories:**
- **Data Input**: Fetch data from external sources (prices, social media, news)
- **Compute**: Process and analyze data (AI analysis, custom code)
- **Trade**: Execute transactions (swap, buy, sell)
- **Output**: Store results or send notifications

**Node Lifecycle:**
```
PENDING → RUNNING → COMPLETED
              ↓
           FAILED / TERMINATED
```

### 3. Signal (Data Transfer)

A **Signal** carries data between nodes through edges, enabling communication in the workflow.

**Signal Structure:**
```typescript
{
  signal_type: string,      // Type identifier
  data: any,                // Payload data
  metadata: {               // Execution context
    flow_id: string,
    cycle: number,
    node_id: string,
    timestamp: number
  }
}
```

**Signal Types:**
- `PRICE_DATA`: Market price information
- `AI_RESPONSE`: LLM analysis results
- `TRADE_RECEIPT`: Transaction confirmation
- Custom types as needed

### 4. Cycle (Execution Round)

A **Cycle** represents one complete execution round of a periodic strategy.

**Cycle Management:**
- Each cycle has a unique number (0, 1, 2, ...)
- Cycles execute sequentially
- State isolated between cycles
- Historical data preserved per cycle

### 5. Handle (Input/Output Port)

A **Handle** is an input or output port on a node that receives or sends signals.

**Handle Properties:**
- **Name**: Unique identifier within the node
- **Type**: Data type (string, number, json object, etc.)
- **Required**: Whether input is mandatory
- **Aggregate**: Whether to collect all signals before processing

---

## 🔄 Execution Flow

### Strategy Execution Pipeline

```
1. Flow Registration
   └─> Parse DAG structure
   └─> Validate node connections
   └─> Register in scheduler

2. Cycle Initialization
   └─> Create cycle state
   └─> Identify entry nodes
   └─> Initialize message queues

3. Node Execution
   └─> Wait for input signals
   └─> Execute node logic
   └─> Send output signals
   └─> Update node state

4. Signal Propagation
   └─> Route signals via RabbitMQ
   └─> Match to downstream handles
   └─> Trigger next node execution

5. Cycle Completion
   └─> All nodes completed
   └─> Store results
   └─> Schedule next cycle
```

### Parallel Execution

Weather Station supports **parallel execution** of independent nodes:

```
     Node A
    /      \
Node B      Node C  ← Execute in parallel
    \      /
     Node D
```

**Benefits:**
- Reduced latency
- Better resource utilization
- Improved throughput

---

## 🛡️ Reliability Features

### 1. State Persistence

All execution state stored in Redis:
- Flow status and configuration
- Cycle progress tracking
- Node execution results
- Signal data for replay

### 2. Failure Recovery

**Node-Level Recovery:**
- Automatic retries for transient failures
- Exponential backoff
- Dead letter queues for persistent failures

**Flow-Level Recovery:**
- Cycle can be restarted from failure point
- State restoration from Redis
- Manual intervention supported

### 3. Distributed Coordination

**Worker Instances:**
- Multiple workers can execute nodes
- Redis-based distributed locking
- Load balancing across workers
- Heartbeat-based health monitoring

---

## 📊 Performance Characteristics

**Throughput:**
- 1000+ nodes/second execution capacity
- Sub-millisecond signal routing
- Horizontal scaling with additional workers

**Latency:**
- <10ms node execution overhead
- <5ms signal propagation
- <100ms end-to-end for simple flows

**Scalability:**
- Tested with 100+ concurrent flows
- 10,000+ nodes per flow supported
- Multi-region deployment capable

---

## 🔗 Technology Stack

**Core Technologies:**
- **Language**: Python 3.9+
- **Async Framework**: AsyncIO
- **Web Framework**: FastAPI / Sanic
- **Message Queue**: RabbitMQ
- **State Store**: Redis
- **Database**: PostgreSQL
- **Blockchain**: Aptos SDK, Ethers.js

**Key Libraries:**
- `aio-pika`: Async RabbitMQ client
- `redis-py`: Redis client
- `sqlalchemy`: Database ORM
- `pydantic`: Data validation

---

## 📖 Next Steps

**For Strategy Developers:**
1. Read [Node Execution Flow](weather-station-node-execution.md)
2. Explore [Available Nodes](../node-details/index.md)
3. Build your first strategy in the UI

**For Technical Architects:**
1. Deep dive into [Message Queue Details](weather-station-message-queue.md)
2. Understand [State Management](weather-station-redis.md)
3. Review [Scheduling Mechanism](weather-station-flow-scheduling.md)

**For Operators:**
1. Study [State Management](weather-station-redis.md)
2. Learn [Monitoring and Debugging](weather-station-node-execution.md#monitoring)
3. Set up production deployment

---

## 📚 Related Documentation

- [Message Queue Architecture](weather-station-message-queue.md)
- [Redis State Management](weather-station-redis.md)
- [Node Execution Details](weather-station-node-execution.md)
- [Flow Scheduling](weather-station-flow-scheduling.md)
- [Node Reference](../node-details/index.md)

---

**Maintained by:** TradingFlow Engineering Team  
**Last Updated:** 2025-10-06  
**Version:** 1.0.0

# Weather Station Message Queue Architecture

**Version:** 1.0.0  
**Document Type:** Technical Deep Dive  
**Last Updated:** 2025-10-06

---

## 📖 About This Document

This document provides detailed information about Weather Station's message queue system, signal routing, and inter-node communication mechanisms.

**Target Audience:**
- Backend Engineers
- System Architects
- DevOps Engineers

**Prerequisites:**
- Understanding of [Architecture Overview](weather-station-overview.md)
- Basic knowledge of message queues
- Familiarity with RabbitMQ concepts

**Reading Time:** ~60 minutes

---

## 🎯 Message Queue Overview

Weather Station uses **RabbitMQ** as its message broker to enable reliable, asynchronous communication between nodes in a trading workflow.

**Why RabbitMQ?:**
- ✅ **Reliable Delivery** - Guaranteed message delivery with acknowledgments
- ✅ **Flexible Routing** - Topic-based routing with pattern matching
- ✅ **Scalability** - Distributed clustering and high throughput
- ✅ **Persistence** - Durable queues survive broker restarts
- ✅ **Monitoring** - Built-in management UI and metrics

---

## 🏗️ Queue Architecture

### Exchange and Queue Structure

Weather Station uses a **Topic Exchange** pattern for flexible signal routing:

```
┌─────────────────────────────────────────────────┐
│         Topic Exchange: 'station.signals'       │
└─────────────┬───────────────────────────────────┘
              │ Routing Key Pattern
              │
      ┌───────┼───────┬──────────────┬────────────┐
      │       │       │              │            │
      ▼       ▼       ▼              ▼            ▼
  ┌──────┐ ┌──────┐ ┌──────┐    ┌──────┐    ┌──────┐
  │Queue │ │Queue │ │Queue │... │Queue │... │Queue │
  │ N1   │ │ N2   │ │ N3   │    │ Nx   │    │Error │
  └──┬───┘ └──┬───┘ └──┬───┘    └──┬───┘    └──┬───┘
     │        │        │           │           │
     ▼        ▼        ▼           ▼           ▼
  [Node1] [Node2] [Node3]      [NodeX]    [ErrorHandler]
```

### Routing Key Convention

Routing keys follow a hierarchical pattern:

```
{flow_id}.{cycle}.{source_node_id}.{source_handle}.{signal_type}
```

**Example:**
```
flow_abc123.0.node_1.price_data.PRICE_DATA
│           │ │      │          │
│           │ │      │          └─ Signal Type
│           │ │      └─ Source Handle
│           │ └─ Source Node ID
│           └─ Cycle Number
└─ Flow ID
```

**Pattern Matching:**
- `*` (star) matches exactly one word
- `#` (hash) matches zero or more words

**Examples:**
```
flow_abc123.*.node_1.*.*        # All signals from node_1 in any cycle
flow_abc123.0.*.price_data.*    # All price_data signals in cycle 0
flow_abc123.#.TRADE_RECEIPT     # All trade receipts in flow
```

---

## 📨 Signal Structure

### Signal Message Format

```typescript
{
  // Message Headers
  "headers": {
    "message_id": "msg_uuid",
    "timestamp": 1633036800000,
    "retry_count": 0
  },
  
  // Signal Metadata
  "metadata": {
    "flow_id": "flow_abc123",
    "cycle": 0,
    "source_node_id": "node_1",
    "source_handle": "price_data",
    "edge_id": "edge_123"
  },
  
  // Signal Type
  "signal_type": "PRICE_DATA",
  
  // Actual Data Payload
  "data": {
    "symbol": "BTC/USDT",
    "price": 45000.00,
    "volume": 1250.5,
    "timestamp": 1633036800
  }
}
```

### Common Signal Types

| Signal Type | Description | Typical Payload |
|-------------|-------------|----------------|
| `PRICE_DATA` | Market price information | price, volume, timestamp |
| `SOCIAL_DATA` | Social media content | tweets, sentiment score |
| `AI_RESPONSE` | LLM analysis results | analysis text, structured data |
| `TRADE_RECEIPT` | Transaction confirmation | tx_hash, amount, status |
| `DATASET` | Tabular data | rows, columns, data array |
| `NOTIFICATION` | Alert message | message, severity, destination |

---

## 🔄 Signal Routing Process

### 1. Signal Creation and Publishing

When a node completes execution and produces output:

```python
# Node publishes signal
async def send_signal(
    self,
    output_handle: str,
    signal_type: str,
    data: Any
):
    # Construct routing key
    routing_key = (
        f"{self.flow_id}."
        f"{self.cycle}."
        f"{self.node_id}."
        f"{output_handle}."
        f"{signal_type}"
    )
    
    # Publish to exchange
    await self.mq_client.publish(
        exchange="station.signals",
        routing_key=routing_key,
        message={
            "metadata": {...},
            "signal_type": signal_type,
            "data": data
        }
    )
```

### 2. Queue Binding and Pattern Matching

Each node subscribes to signals from its input edges:

```python
# Node subscribes to input signals
async def setup_signal_consumer(self):
    for edge in self.input_edges:
        # Create binding pattern
        pattern = (
            f"{self.flow_id}."
            f"{self.cycle}."
            f"{edge.source_node_id}."
            f"{edge.source_handle}."
            f"*"  # Match any signal type
        )
        
        # Bind queue to exchange
        await self.queue.bind(
            exchange="station.signals",
            routing_key=pattern
        )
```

### 3. Signal Reception and Processing

```python
# Node consumes signals
async def consume_signals(self):
    async with self.queue.iterator() as queue_iter:
        async for message in queue_iter:
            try:
                # Parse signal
                signal = parse_signal(message.body)
                
                # Store in input buffer
                await self.store_input_signal(signal)
                
                # Check if ready to execute
                if self.all_inputs_received():
                    await self.execute()
                
                # Acknowledge message
                await message.ack()
                
            except Exception as e:
                # Negative acknowledgment (requeue)
                await message.nack(requeue=True)
```

---

## 🎯 Handle Matching and Aggregation

### Single Input Handling

For non-aggregate inputs, the node executes immediately upon receiving a signal:

```python
@dataclass
class InputHandle:
    name: str
    data_type: str
    is_aggregate: bool = False  # Execute on each signal
```

**Execution Flow:**
```
Signal Arrives → Store in Buffer → Execute Node → Continue
```

### Aggregated Input Handling

For aggregate inputs, the node waits for all expected signals before executing:

```python
@dataclass
class InputHandle:
    name: str
    data_type: str
    is_aggregate: bool = True  # Wait for all signals
```

**Execution Flow:**
```
Signal 1 Arrives → Store
Signal 2 Arrives → Store
    ...
Signal N Arrives → Store → All Collected? → Execute Node
```

**Use Cases:**
- Waiting for multiple price sources before arbitrage
- Collecting all social media mentions before sentiment analysis
- Gathering portfolio data from multiple chains

---

## 🛡️ Reliability Features

### Message Acknowledgment

**Manual Acknowledgment Mode:**
```python
# Only acknowledge after successful processing
await message.ack()

# Negative acknowledge on failure (requeue)
await message.nack(requeue=True)

# Reject without requeue (dead letter)
await message.reject(requeue=False)
```

**Benefits:**
- No message loss
- Automatic retries
- Failure isolation

### Durable Queues

```python
# Queue survives broker restarts
queue = await channel.declare_queue(
    name=f"node_{node_id}_queue",
    durable=True,  # Persist to disk
    auto_delete=False
)
```

### Publisher Confirms

```python
# Ensure message reached broker
await channel.set_publisher_confirms(True)
result = await exchange.publish(message)
if not result:
    raise MessagePublishError()
```

### Dead Letter Exchange

Failed messages route to dead letter queue for analysis:

```python
queue = await channel.declare_queue(
    name="node_queue",
    arguments={
        "x-dead-letter-exchange": "station.dlx",
        "x-message-ttl": 60000,  # 60 seconds
        "x-max-retries": 3
    }
)
```

---

## 📊 Performance Optimization

### Connection Pooling

```python
class MQConnectionPool:
    def __init__(self, max_connections=10):
        self.pool = []
        self.max_connections = max_connections
    
    async def get_connection(self):
        if len(self.pool) < self.max_connections:
            conn = await aio_pika.connect_robust(
                RABBITMQ_URL
            )
            self.pool.append(conn)
        return random.choice(self.pool)
```

### Batching

```python
# Batch multiple signals into single message
async def batch_publish(signals: List[Signal]):
    async with channel.transaction():
        for signal in signals:
            await exchange.publish(signal)
```

### Prefetch Control

```python
# Limit concurrent message processing
await channel.set_qos(prefetch_count=10)
```

---

## 🔍 Monitoring and Debugging

### Message Tracing

```python
# Add trace ID to all messages
headers = {
    "trace_id": str(uuid.uuid4()),
    "parent_span_id": current_span_id
}
```

### Queue Metrics

Monitor via RabbitMQ Management API:
- Queue depth
- Message rate (in/out)
- Consumer count
- Unacknowledged messages

### Common Issues and Solutions

**Issue: Messages Not Being Consumed**
- Check queue bindings
- Verify routing key pattern
- Ensure consumer is running

**Issue: Message Build-up**
- Increase consumer count
- Optimize node execution
- Check for errors in processing

**Issue: Messages Lost**
- Enable publisher confirms
- Use durable queues
- Implement dead letter handling

---

## 📚 Related Documentation

- [Architecture Overview](weather-station-overview.md)
- [Node Execution Flow](weather-station-node-execution.md)
- [Redis State Management](weather-station-redis.md)
- [Flow Scheduling](weather-station-flow-scheduling.md)

---

**Maintained by:** TradingFlow Engineering Team  
**Last Updated:** 2025-10-06  
**Version:** 1.0.0

# Weather Station State Management with Redis

**Version:** 1.0.0  
**Document Type:** Technical Deep Dive  
**Last Updated:** 2025-10-06

---

## 📖 About This Document

This document explains how Weather Station uses Redis for distributed state management, coordination, and data caching.

**Target Audience:**
- Backend Engineers
- DevOps Engineers
- System Architects

**Prerequisites:**
- Understanding of [Architecture Overview](weather-station-overview.md)
- Basic Redis knowledge
- Distributed systems concepts

**Reading Time:** ~50 minutes

---

## 🎯 Why Redis?

Weather Station uses Redis as its primary state store for:

- ⚡ **High Performance** - Sub-millisecond read/write operations
- 🔄 **Distributed Coordination** - Atomic operations and locks
- 💾 **Persistence** - Configurable durability options
- 📊 **Rich Data Structures** - Hashes, sets, sorted sets, streams
- 🌐 **Scalability** - Clustering and replication support

---

## 🗂️ Data Organization

### Key Naming Convention

All keys follow a hierarchical namespace pattern:

```
station:{resource_type}:{identifier}:{sub_resource}
```

**Examples:**
```redis
station:flow:abc123:config
station:flow:abc123:cycle:0:status
station:node:node_1:state
station:worker:worker_001:heartbeat
```

---

## 📊 Core Data Structures

### 1. Flow State

**Flow Configuration**
```redis
KEY: station:flow:{flow_id}:config
TYPE: Hash
FIELDS:
  - name: string
  - schedule: string (periodic/manual)
  - interval_seconds: number
  - status: string (active/paused/stopped)
  - created_at: timestamp
  - updated_at: timestamp

EXAMPLE:
HSET station:flow:abc123:config name "BTC Arbitrage"
HGET station:flow:abc123:config status
```

**Cycle State**
```redis
KEY: station:flow:{flow_id}:cycle:{cycle}:status
TYPE: Hash
FIELDS:
  - cycle: number
  - status: string (pending/running/completed/failed)
  - started_at: timestamp
  - completed_at: timestamp
  - total_nodes: number
  - completed_nodes: number

EXAMPLE:
HSET station:flow:abc123:cycle:0:status status "running"
HINCRBY station:flow:abc123:cycle:0:status completed_nodes 1
```

### 2. Node State

**Node Execution State**
```redis
KEY: station:node:{node_id}:state
TYPE: Hash
FIELDS:
  - node_id: string
  - node_type: string
  - status: string (pending/running/completed/failed)
  - started_at: timestamp
  - completed_at: timestamp
  - error_message: string (if failed)
  - retry_count: number

EXAMPLE:
HSET station:node:node_1:state status "completed"
HGET station:node:node_1:state error_message
```

**Node Input Signals**
```redis
KEY: station:node:{node_id}:inputs:{handle_name}
TYPE: List
VALUE: JSON-serialized signal objects

EXAMPLE:
LPUSH station:node:node_1:inputs:price_data '{"data": {...}}'
LRANGE station:node:node_1:inputs:price_data 0 -1
```

**Node Output Results**
```redis
KEY: station:node:{node_id}:outputs:{handle_name}
TYPE: String
VALUE: JSON-serialized output data
TTL: 3600 seconds (1 hour)

EXAMPLE:
SET station:node:node_1:outputs:result '{"price": 45000}'
GET station:node:node_1:outputs:result
```

### 3. Worker Management

**Worker Registry**
```redis
KEY: station:workers
TYPE: Set
VALUE: worker_id strings

EXAMPLE:
SADD station:workers "worker_001"
SMEMBERS station:workers
```

**Worker Heartbeat**
```redis
KEY: station:worker:{worker_id}:heartbeat
TYPE: String
VALUE: timestamp
TTL: 30 seconds

EXAMPLE:
SET station:worker:worker_001:heartbeat "1633036800" EX 30
```

**Worker Capacity**
```redis
KEY: station:worker:{worker_id}:capacity
TYPE: Hash
FIELDS:
  - max_concurrent: number
  - current_load: number
  - available_slots: number

EXAMPLE:
HSET station:worker:worker_001:capacity max_concurrent 10
HINCRBY station:worker:worker_001:capacity current_load 1
```

### 4. Distributed Locks

**Node Execution Lock**
```redis
KEY: station:lock:node:{node_id}
TYPE: String
VALUE: worker_id (lock holder)
TTL: 60 seconds

EXAMPLE:
SET station:lock:node:node_1 "worker_001" NX EX 60
```

**Flow Scheduling Lock**
```redis
KEY: station:lock:flow:{flow_id}:schedule
TYPE: String
VALUE: scheduler_id
TTL: 10 seconds

EXAMPLE:
SET station:lock:flow:abc123:schedule "scheduler_001" NX EX 10
```

---

## 🔒 Distributed Coordination

### Acquiring Distributed Locks

```python
async def acquire_lock(
    redis_client: Redis,
    lock_key: str,
    owner_id: str,
    ttl_seconds: int = 60
) -> bool:
    """
    Acquire a distributed lock using SET NX EX
    Returns True if lock acquired, False otherwise
    """
    result = await redis_client.set(
        lock_key,
        owner_id,
        nx=True,  # Only set if doesn't exist
        ex=ttl_seconds  # Auto-expire after ttl
    )
    return result is not None
```

**Usage Example:**
```python
# Worker attempting to execute a node
lock_key = f"station:lock:node:{node_id}"
if await acquire_lock(redis, lock_key, worker_id):
    try:
        # Execute node
        await execute_node(node)
    finally:
        # Release lock
        await redis.delete(lock_key)
else:
    logger.info(f"Node {node_id} already locked by another worker")
```

### Lock Extension (Heartbeat)

```python
async def extend_lock(
    redis_client: Redis,
    lock_key: str,
    owner_id: str,
    ttl_seconds: int = 60
) -> bool:
    """
    Extend lock TTL if still owned by this owner
    """
    # Lua script for atomic check-and-extend
    lua_script = """
    if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("expire", KEYS[1], ARGV[2])
    else
        return 0
    end
    """
    result = await redis.eval(
        lua_script,
        keys=[lock_key],
        args=[owner_id, ttl_seconds]
    )
    return result == 1
```

---

## 📈 Performance Optimization

### Connection Pooling

```python
from redis.asyncio import Redis, ConnectionPool

# Create connection pool
pool = ConnectionPool.from_url(
    REDIS_URL,
    max_connections=50,
    decode_responses=True
)

# Reuse connections
redis_client = Redis(connection_pool=pool)
```

### Pipeline Batching

```python
# Batch multiple operations
async with redis.pipeline() as pipe:
    pipe.hset("station:node:node_1:state", "status", "running")
    pipe.incr("station:flow:abc123:cycle:0:completed_nodes")
    pipe.lpush("station:logs", log_entry)
    await pipe.execute()
```

### Caching Strategy

```python
# Cache with TTL
await redis.setex(
    f"cache:price:{symbol}",
    300,  # 5 minutes
    json.dumps(price_data)
)

# Check cache before fetching
cached = await redis.get(f"cache:price:{symbol}")
if cached:
    return json.loads(cached)
else:
    # Fetch from source and cache
    data = await fetch_price(symbol)
    await redis.setex(f"cache:price:{symbol}", 300, json.dumps(data))
    return data
```

---

## 🔍 Monitoring and Debugging

### Key Metrics to Monitor

```python
# Memory usage
info = await redis.info("memory")
used_memory_mb = info["used_memory"] / 1024 / 1024

# Key count
key_count = await redis.dbsize()

# Slow log
slow_logs = await redis.slowlog_get(10)

# Client connections
clients = await redis.client_list()
```

### Common Queries

**Find all flows:**
```redis
KEYS station:flow:*:config
```

**Find all active cycles:**
```redis
SCAN 0 MATCH station:flow:*:cycle:*:status
```

**Check node status:**
```redis
HGET station:node:node_1:state status
```

**List all workers:**
```redis
SMEMBERS station:workers
```

---

## 🧹 Data Cleanup

### TTL Management

**Automatic Expiration:**
```python
# Set TTL on temporary data
await redis.setex(
    f"station:node:{node_id}:outputs:result",
    3600,  # 1 hour
    result_data
)
```

**Periodic Cleanup:**
```python
async def cleanup_old_cycles():
    """Remove cycle data older than 7 days"""
    cutoff_time = time.time() - (7 * 24 * 3600)
    
    async for key in redis.scan_iter("station:flow:*:cycle:*:status"):
        cycle_data = await redis.hgetall(key)
        completed_at = float(cycle_data.get("completed_at", 0))
        
        if completed_at < cutoff_time:
            # Delete cycle data
            flow_id, cycle = parse_cycle_key(key)
            await delete_cycle_data(flow_id, cycle)
```

---

## 🛡️ High Availability

### Redis Sentinel

```python
from redis.sentinel import Sentinel

# Connect through Sentinel
sentinel = Sentinel(
    [('sentinel1', 26379), ('sentinel2', 26379)],
    socket_timeout=0.5
)

# Get master connection
redis_client = sentinel.master_for(
    'tradingflow-master',
    socket_timeout=0.5,
    db=0
)
```

### Redis Cluster

```python
from redis.cluster import RedisCluster

# Connect to cluster
redis_client = RedisCluster(
    host='redis-cluster',
    port=6379,
    decode_responses=True
)
```

---

## 📚 Related Documentation

- [Architecture Overview](weather-station-overview.md)
- [Message Queue Details](weather-station-message-queue.md)
- [Node Execution Flow](weather-station-node-execution.md)
- [Flow Scheduling](weather-station-flow-scheduling.md)

---

**Maintained by:** TradingFlow Engineering Team  
**Last Updated:** 2025-10-06  
**Version:** 1.0.0

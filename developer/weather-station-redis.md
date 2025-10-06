# Weather Station - Redis 状态管理

**文档版本：** 1.0.0  
**最后更新：** 2025-10-06  
**前置阅读：** [架构概述](weather-station-overview.md)

---

## 目录

- [Redis 架构](#redis-架构)
- [Key 命名规范](#key-命名规范)
- [Flow 状态管理](#flow-状态管理)
- [Cycle 状态管理](#cycle-状态管理)
- [节点任务管理](#节点任务管理)
- [Worker 管理](#worker-管理)
- [状态查询](#状态查询)
- [数据清理策略](#数据清理策略)

---

## Redis 架构

### 设计原则

1. **分布式共享**：支持多 Worker 实例并发访问
2. **数据隔离**：不同 Flow 和 Cycle 的数据完全隔离
3. **快速查询**：使用 Hash、Set 等高效数据结构
4. **TTL 管理**：自动清理过期数据
5. **防腐层设计**：StateStore 抽象层隔离 Redis 实现

### Redis 数据结构使用

| 数据结构 | 用途 | 示例 |
|---------|------|------|
| **String** | 简单 KV 存储 | Flow 配置 JSON |
| **Hash** | 对象属性存储 | Flow 元数据、Cycle 状态 |
| **Set** | 无序集合 | Cycle 中的节点列表 |
| **Sorted Set** | 有序集合（未使用） | 预留用于优先级队列 |

### StateStore 架构

```
┌─────────────────────────────────────────────────┐
│             Application Layer                   │
│  (FlowScheduler, NodeTaskManager, NodeBase)    │
└────────────────────┬────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────┐
│            StateStore (Abstract)                │
│  - set_node_task_status()                       │
│  - get_node_task_status()                       │
│  - set_value() / get_value()                    │
│  - add_to_set() / get_set_members()             │
└────────────────────┬────────────────────────────┘
                     │
        ┌────────────┴─────────────┐
        │                          │
        ↓                          ↓
┌─────────────────┐      ┌─────────────────┐
│ RedisStateStore │      │InMemoryStateStore│
│ (Production)    │      │    (Testing)     │
└─────────────────┘      └─────────────────┘
```

---

## Key 命名规范

### 全局规范

**格式：** `<prefix>:<entity>:<identifier>[:<sub-entity>]`

**前缀统一：** `trading_flow:` (可配置)

### Key 分类表

| 类别 | Key 格式 | 数据结构 | TTL | 说明 |
|------|---------|---------|-----|------|
| **Flow 元数据** | `flow:{flow_id}` | Hash | 永久 | Flow 配置和状态 |
| **Cycle 元数据** | `flow:{flow_id}:cycle:{cycle}` | Hash | 7天 | Cycle 执行信息 |
| **Cycle 节点集合** | `flow:{flow_id}:cycle:{cycle}:nodes` | Set | 7天 | Cycle 中的节点 ID |
| **Component 停止标志** | `flow:{flow_id}:cycle:{cycle}:component:{component_id}:stop` | String | 1小时 | Component 停止标记 |
| **节点任务详情** | `node_tasks:{node_task_id}` | String (JSON) | 24小时 | 节点任务完整信息 |
| **节点任务列表** | `node_tasks_list` | Set | 永久 | 所有节点任务 ID |
| **Worker 任务列表** | `worker_tasks:{worker_id}` | Set | 永久 | Worker 的任务 ID |
| **节点状态** | `trading_flow:node:{node_task_id}` | Hash | 24小时 | 节点运行时状态 |
| **终止标志** | `trading_flow:node:{node_task_id}:terminate` | String (JSON) | 1小时 | 节点终止请求 |

---

## Flow 状态管理

### Flow 元数据结构

**Key：** `flow:{flow_id}`  
**类型：** Hash  
**TTL：** 永久（或 Flow 删除时清理）

**字段说明：**

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `id` | string | Flow ID | `trading_decision_flow` |
| `config` | JSON string | Flow 配置（TFL JSON） | `{"nodes":[...],"edges":[...]}` |
| `structure` | JSON string | DAG 结构分析结果 | `{"components":{...},"entry_nodes":[...]}` |
| `status` | string | 状态 | `registered`, `running`, `stopped`, `completed` |
| `last_cycle` | int | 最后执行的 Cycle | `0`, `1`, `2` |
| `next_execution` | timestamp | 下次执行时间戳 | `1633024800.0` |
| `created_at` | ISO datetime | 创建时间 | `2025-10-06T10:00:00` |

### Flow 状态转换

```
registered → running → stopped
                  ↓
              completed (interval=0 时)
```

### 示例数据

```bash
# 查看 Flow 元数据
HGETALL flow:trading_decision_flow

1) "id"
2) "trading_decision_flow"
3) "config"
4) "{"nodes":[{"id":"A",...}],"edges":[...]}"
5) "structure"
6) "{"components":{"0":{"nodes":[...]}}}"
7) "status"
8) "running"
9) "last_cycle"
10) "5"
11) "next_execution"
12) "1633024800.0"
13) "created_at"
14) "2025-10-06T10:00:00"
```

### Flow 注册流程

```python
# FlowScheduler.register_flow()
flow_data = {
    "id": flow_id,
    "config": json.dumps(flow_config),
    "structure": json.dumps(flow_structure),
    "status": "registered",
    "last_cycle": -1,
    "next_execution": 0,
    "created_at": datetime.now().isoformat()
}

await self.redis.hset(f"flow:{flow_id}", mapping=flow_data)
```

---

## Cycle 状态管理

### Cycle 元数据结构

**Key：** `flow:{flow_id}:cycle:{cycle}`  
**类型：** Hash  
**TTL：** 7 天

**字段说明：**

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `flow_id` | string | Flow ID | `trading_decision_flow` |
| `cycle` | int | Cycle 号 | `0`, `1`, `2` |
| `status` | string | 状态 | `running`, `completed`, `failed` |
| `start_time` | ISO datetime | 开始时间 | `2025-10-06T10:00:00` |
| `end_time` | ISO datetime | 结束时间 | `2025-10-06T10:05:00` |

### Cycle 节点集合

**Key：** `flow:{flow_id}:cycle:{cycle}:nodes`  
**类型：** Set  
**TTL：** 7 天

**成员：** Node ID 列表

```bash
# 查看 Cycle 中的所有节点
SMEMBERS flow:trading_flow:cycle:0:nodes

1) "binance_price"
2) "ai_model"
3) "buy_node"
```

### Cycle 执行流程

```python
# FlowScheduler._execute_flow_cycle()

# 1. 创建 Cycle 记录
cycle_key = f"flow:{flow_id}:cycle:{cycle}"
cycle_data = {
    "flow_id": flow_id,
    "cycle": str(cycle),
    "status": "running",
    "start_time": datetime.now().isoformat(),
    "end_time": ""
}
await self.redis.hset(cycle_key, mapping=cycle_data)

# 2. 注册节点到 Cycle
nodes_key = f"flow:{flow_id}:cycle:{cycle}:nodes"
for node_id in node_map.keys():
    await self.redis.sadd(nodes_key, node_id)

# 3. 执行所有节点...

# 4. 更新 Cycle 状态为完成
await self.redis.hset(
    cycle_key,
    mapping={"status": "completed", "end_time": datetime.now().isoformat()}
)
```

### Component 停止标志

**Key：** `flow:{flow_id}:cycle:{cycle}:component:{component_id}:stop`  
**类型：** String  
**TTL：** 1 小时  
**值：** `"1"` (存在即为停止)

**用途：** 停止特定连通分量的执行

```python
# 设置停止标志
await self.redis.set(
    f"flow:{flow_id}:cycle:{cycle}:component:{component_id}:stop",
    "1"
)

# 检查是否停止
stop_flag = await self.redis.get(
    f"flow:{flow_id}:cycle:{cycle}:component:{component_id}:stop"
)
if stop_flag:
    # 跳过节点执行
    return {"status": "skipped", "reason": "component_stopped"}
```

---

## 节点任务管理

### 节点任务详情

**Key：** `node_tasks:{node_task_id}`  
**类型：** String (JSON)  
**TTL：** 24 小时

**Node Task ID 格式：** `{flow_id}_{cycle}_{node_id}`

**数据结构：**
```json
{
  "node_task_id": "trading_flow_0_binance_price",
  "flow_id": "trading_flow",
  "cycle": 0,
  "node_id": "binance_price",
  "node_type": "binance_price_node",
  "worker_id": "worker_1",
  "status": "running",
  "registered_at": "2025-10-06T10:00:00",
  "updated_at": "2025-10-06T10:00:05",
  "message": "Executing node",
  "progress": 50,
  "config": { /* 节点配置 */ }
}
```

**状态值：**
- `registered`: 已注册
- `pending`: 等待执行
- `running`: 执行中
- `completed`: 已完成
- `failed`: 失败
- `terminated`: 已终止
- `skipped`: 已跳过

### 节点任务列表

**Key：** `node_tasks_list`  
**类型：** Set  
**TTL：** 永久

**成员：** 所有节点任务 ID

```bash
SMEMBERS node_tasks_list

1) "trading_flow_0_binance_price"
2) "trading_flow_0_ai_model"
3) "trading_flow_0_buy_node"
```

### 节点任务操作

#### 注册任务

```python
# NodeTaskManager.register_task()
task_info = {
    "node_task_id": node_task_id,
    "flow_id": flow_id,
    "cycle": cycle,
    "node_id": node_id,
    "node_type": node_type,
    "worker_id": self.worker_id,
    "status": "registered",
    "registered_at": datetime.now().isoformat()
}

# 保存任务详情
task_key = f"node_tasks:{node_task_id}"
await self.state_store.set_value(task_key, task_info)

# 添加到任务列表
await self.state_store.add_to_set("node_tasks_list", node_task_id)
```

#### 更新状态

```python
# NodeTaskManager.update_task_status()
task_key = f"node_tasks:{node_task_id}"
task_info = await self.state_store.get_value(task_key)

task_info["status"] = status
task_info["updated_at"] = datetime.now().isoformat()

if additional_info:
    task_info.update(additional_info)

await self.state_store.set_value(task_key, task_info)
```

#### 查询任务

```python
# NodeTaskManager.get_task()
task_key = f"node_tasks:{node_task_id}"
task_info = await self.state_store.get_value(task_key)
return task_info
```

### 节点运行时状态

**Key：** `trading_flow:node:{node_task_id}`  
**类型：** Hash  
**TTL：** 24 小时

**字段：**
```bash
HGETALL trading_flow:node:trading_flow_0_binance_price

1) "status"
2) "running"
3) "updated_at"
4) "1633024800.0"
5) "error_message"
6) ""
```

### 节点终止标志

**Key：** `trading_flow:node:{node_task_id}:terminate`  
**类型：** String (JSON)  
**TTL：** 1 小时

**数据结构：**
```json
{
  "reason": "Stopped by NodeTaskManager",
  "timestamp": "2025-10-06T10:00:00"
}
```

**使用场景：**
```python
# 设置终止标志
await self.state_store.set_termination_flag(
    node_task_id,
    reason="User requested stop"
)

# 节点检查终止标志
terminate_flag = await self.state_store.get_termination_flag(node_task_id)
if terminate_flag:
    raise NodeStopExecutionException(
        reason=terminate_flag["reason"],
        source_node="system"
    )
```

---

## Worker 管理

### Worker 任务列表

**Key：** `worker_tasks:{worker_id}`  
**类型：** Set  
**TTL：** 永久

**成员：** Worker 执行的节点任务 ID

```bash
SMEMBERS worker_tasks:worker_1

1) "trading_flow_0_binance_price"
2) "trading_flow_0_ai_model"
3) "trading_flow_1_buy_node"
```

### Worker 注册流程

```python
# NodeRegistry.register_worker()
worker_key = f"workers:{worker_id}"
worker_info = {
    "id": worker_id,
    "api_url": api_url,
    "supported_nodes": supported_node_types,
    "status": "active",
    "last_heartbeat": datetime.now().isoformat()
}

await self.redis.hset(worker_key, mapping=worker_info)
await self.redis.expire(worker_key, heartbeat_ttl)  # 默认 60 秒
```

### Worker 心跳机制

```python
# 定期更新心跳
while True:
    await self.redis.hset(
        f"workers:{worker_id}",
        "last_heartbeat",
        datetime.now().isoformat()
    )
    await self.redis.expire(f"workers:{worker_id}", 60)
    await asyncio.sleep(30)
```

### Worker 任务查询

```python
# NodeTaskManager.get_worker_tasks()
worker_key = f"worker_tasks:{worker_id}"
task_ids = await self.state_store.get_set_members(worker_key)

tasks = []
for task_id in task_ids:
    task_info = await self.get_task(task_id)
    if task_info:
        tasks.append(task_info)

return tasks
```

---

## 状态查询

### 查询 Flow 状态

```python
# FlowScheduler.get_flow_status()
flow_data = await self.redis.hgetall(f"flow:{flow_id}")

# 解析 JSON 字段
if "config" in flow_data:
    flow_data["config"] = json.loads(flow_data["config"])
if "structure" in flow_data:
    flow_data["structure"] = json.loads(flow_data["structure"])

# 获取当前 Cycle 状态
current_cycle = int(flow_data.get("last_cycle", -1))
if current_cycle >= 0:
    cycle_status = await self.get_cycle_status(flow_id, current_cycle)
    flow_data["current_cycle_status"] = cycle_status

return flow_data
```

### 查询 Cycle 状态

```python
# FlowScheduler.get_cycle_status()
cycle_key = f"flow:{flow_id}:cycle:{cycle}"
cycle_data = await self.redis.hgetall(cycle_key)

# 获取所有节点状态
nodes_key = f"flow:{flow_id}:cycle:{cycle}:nodes"
node_ids = await self.redis.smembers(nodes_key)

# 从 NodeTaskManager 获取节点详情
task_manager = NodeTaskManager.get_instance()
nodes_status = {}
for node_id in node_ids:
    node_task_id = f"{flow_id}_{cycle}_{node_id}"
    node_data = await task_manager.get_task(node_task_id)
    if node_data:
        nodes_status[node_id] = node_data

return {
    "cycle": cycle,
    "status": cycle_data.get("status"),
    "start_time": cycle_data.get("start_time"),
    "end_time": cycle_data.get("end_time"),
    "nodes": nodes_status,
    "node_count": len(node_ids)
}
```

### 综合状态查询

```python
# FlowScheduler.get_comprehensive_flow_status()
comprehensive_nodes = await node_manager.get_comprehensive_node_status(
    flow_id=flow_id,
    cycle=cycle
)

# 统计信息
total_nodes = len(comprehensive_nodes)
running_nodes = sum(1 for node in comprehensive_nodes.values() if node['status'] == 'running')
completed_nodes = sum(1 for node in comprehensive_nodes.values() if node['status'] == 'completed')
error_nodes = sum(1 for node in comprehensive_nodes.values() if node['status'] == 'error')

return {
    "flow_id": flow_id,
    "cycle": cycle,
    "flow_status": flow_status,
    "nodes": comprehensive_nodes,
    "statistics": {
        "total_nodes": total_nodes,
        "running_nodes": running_nodes,
        "completed_nodes": completed_nodes,
        "error_nodes": error_nodes
    }
}
```

---

## 数据清理策略

### TTL 策略表

| Key 类型 | TTL | 清理触发 |
|---------|-----|---------|
| Flow 元数据 | 永久 | 手动删除 |
| Cycle 元数据 | 7 天 | 自动过期 |
| Cycle 节点集合 | 7 天 | 自动过期 |
| Component 停止标志 | 1 小时 | 自动过期 |
| 节点任务详情 | 24 小时 | 自动过期 |
| 节点任务列表 | 永久 | 定期清理 |
| Worker 任务列表 | 永久 | Worker 下线时清理 |
| 节点运行时状态 | 24 小时 | 自动过期 |
| 节点终止标志 | 1 小时 | 自动过期 |
| Worker 信息 | 60 秒 | 心跳过期 |

### 手动清理操作

#### 清理过期任务

```python
# NodeTaskManager.cleanup_old_tasks()
task_ids = await self.state_store.get_set_members("node_tasks_list")

for task_id in task_ids:
    task_key = f"node_tasks:{task_id}"
    task_info = await self.state_store.get_value(task_key)
    
    # 如果任务不存在（已过期），从列表中移除
    if not task_info:
        await self.state_store.remove_from_set("node_tasks_list", task_id)
```

#### 清理 Worker 任务

```python
# NodeTaskManager.cleanup_worker_tasks()
worker_key = f"worker_tasks:{worker_id}"
task_ids = await self.state_store.get_set_members(worker_key)

for task_id in task_ids:
    # 从任务列表移除
    await self.state_store.remove_from_set("node_tasks_list", task_id)
    # 删除任务详情
    await self.state_store.delete_value(f"node_tasks:{task_id}")

# 删除 Worker 任务列表
await self.state_store.delete_value(worker_key)
```

#### 清理 Flow 数据

```python
# FlowScheduler.delete_flow()
# 1. 删除 Flow 元数据
await self.redis.delete(f"flow:{flow_id}")

# 2. 删除所有 Cycle 数据
for cycle in range(last_cycle + 1):
    await self.redis.delete(f"flow:{flow_id}:cycle:{cycle}")
    await self.redis.delete(f"flow:{flow_id}:cycle:{cycle}:nodes")

# 3. 删除节点任务
for cycle in range(last_cycle + 1):
    node_ids = await self.redis.smembers(f"flow:{flow_id}:cycle:{cycle}:nodes")
    for node_id in node_ids:
        node_task_id = f"{flow_id}_{cycle}_{node_id}"
        await node_manager.remove_task(node_task_id)
```

### 定期清理任务

```python
# 建议在 FlowScheduler 中添加定期清理任务
async def _cleanup_expired_data(self):
    """定期清理过期数据"""
    while True:
        try:
            # 1. 清理过期任务列表
            await self._cleanup_task_list()
            
            # 2. 清理过期 Cycle 数据
            await self._cleanup_expired_cycles()
            
            # 3. 清理无效 Worker
            await self._cleanup_dead_workers()
            
        except Exception as e:
            logger.error("Error in cleanup task: %s", e)
        
        # 每小时执行一次
        await asyncio.sleep(3600)
```

---

## 下一步

继续阅读相关文档：

- **[节点执行流程](weather-station-node-execution.md)** - 节点生命周期详解
- **[Flow 调度机制](weather-station-flow-scheduling.md)** - 周期调度和 DAG 编排
- **[开发指南](weather-station-development-guide.md)** - 开发新节点和扩展功能

---

**维护者：** TradingFlow 开发团队  
**版本历史：** 
- v1.0.0 (2025-10-06): 初始版本

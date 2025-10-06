# Weather Station - Flow 调度机制

**文档版本：** 1.0.0  
**最后更新：** 2025-10-06  
**前置阅读：** [架构概述](weather-station-overview.md), [节点执行流程](weather-station-node-execution.md)

---

## 目录

- [调度器架构](#调度器架构)
- [Flow 注册](#flow-注册)
- [DAG 分析](#dag-分析)
- [周期调度](#周期调度)
- [Cycle 执行](#cycle-执行)
- [节点编排](#节点编排)
- [状态管理](#状态管理)
- [日志系统](#日志系统)

---

## 调度器架构

### FlowScheduler 核心职责

```python
class FlowScheduler:
    """
    Flow 调度器：负责管理 Flow 的周期性执行和状态跟踪
    
    主要功能：
    1. Flow 的注册、启动、停止
    2. 周期性调度执行
    3. DAG 结构分析和连通分量识别
    4. 节点状态管理和查询
    5. 执行日志持久化
    """
```

### 调度器生命周期

```
初始化 (initialize)
    ↓
    └─> 连接 Redis
    └─> 初始化日志服务
    ↓
运行中 (运行状态)
    ↓
    ├─> register_flow()    # 注册 Flow
    ├─> start_flow()       # 启动调度
    ├─> stop_flow()        # 停止调度
    ├─> get_flow_status()  # 查询状态
    └─> get_cycle_status() # 查询 Cycle
    ↓
关闭 (shutdown)
    ↓
    └─> 释放 Redis 连接
```

### 单例模式

```python
# 全局单例实例
_scheduler_instance = None

def get_scheduler_instance():
    """获取 FlowScheduler 单例实例"""
    global _scheduler_instance
    if _scheduler_instance is None:
        _scheduler_instance = FlowScheduler()
    return _scheduler_instance
```

---

## Flow 注册

### 注册流程

```python
async def register_flow(self, flow_id: str, flow_config: Dict):
    """
    注册 Flow 到调度系统
    
    Args:
        flow_id: Flow 唯一标识
        flow_config: Flow 配置（Weather Syntax JSON）
            - interval: 执行间隔（秒）
            - nodes: 节点列表
            - edges: 边列表
    """
    # 1. 参数验证
    if not flow_config.get("interval"):
        raise ValueError("Flow configuration missing required parameter: interval")
    if not flow_config.get("nodes"):
        raise ValueError("Flow configuration missing required parameter: nodes")
    
    # 2. 确保 edges 存在
    if "edges" not in flow_config:
        flow_config["edges"] = []
    
    # 3. 分析 Flow 结构
    flow_structure = self._analyze_flow_structure(flow_config)
    
    # 4. 检查是否已存在（保留 last_cycle）
    existing_flow = await self.redis.hgetall(f"flow:{flow_id}")
    existing_last_cycle = int(existing_flow.get("last_cycle", -1)) if existing_flow else -1
    
    # 5. 存储到 Redis
    flow_data = {
        "id": flow_id,
        "config": json.dumps(flow_config),
        "structure": json.dumps(flow_structure),
        "status": "registered",
        "last_cycle": existing_last_cycle,
        "next_execution": 0,  # 0 表示立即执行
        "created_at": existing_flow.get("created_at") if existing_flow else datetime.now().isoformat()
    }
    
    await self.redis.hset(f"flow:{flow_id}", mapping=flow_data)
    logger.info("Flow %s has been registered", flow_id)
    
    return flow_data
```

### Weather Syntax 配置结构

```json
{
  "interval": 60,
  "nodes": [
    {
      "id": "node_A",
      "type": "binance_price_node",
      "config": {
        "symbol": "BTCUSDT",
        "interval": "1m"
      }
    },
    {
      "id": "node_B",
      "type": "ai_model_node",
      "config": {
        "model": "gpt-4"
      }
    }
  ],
  "edges": [
    {
      "source": "node_A",
      "source_handle": "kline_data",
      "target": "node_B",
      "target_handle": "price_input"
    }
  ]
}
```

---

## DAG 分析

### FlowParser 核心功能

```python
# tradingflow/station/flow/flow_parser.py

class FlowParser:
    """Flow 解析器：解析 Weather Syntax 并识别 DAG 结构"""
    
    def __init__(self, flow_json: dict):
        """初始化解析器"""
        self.flow_json = flow_json
        self.nodes = []
        self.edges = []
        self.graph = defaultdict(list)
        self.node_map = {}
        
        if self.flow_json:
            self._parse_flow()
    
    def _parse_flow(self):
        """解析 Flow JSON，提取节点和边"""
        self.nodes = self.flow_json.get("nodes", [])
        self.edges = self.flow_json.get("edges", [])
        
        # 创建节点映射
        for i, node in enumerate(self.nodes):
            node_id = node.get("id")
            self.node_map[node_id] = i
        
        # 构建图的邻接表
        for edge in self.edges:
            source = edge.get("source")
            target = edge.get("target")
            if source in self.node_map and target in self.node_map:
                self.graph[source].append(target)
```

### 连通分量识别

**算法：** 广度优先搜索（BFS）

```python
def _find_connected_components(self) -> List[Set[str]]:
    """找出图中的所有连通分量"""
    # 1. 构建无向图
    undirected_graph = defaultdict(list)
    for source, targets in self.graph.items():
        for target in targets:
            undirected_graph[source].append(target)
            undirected_graph[target].append(source)
    
    # 2. BFS 遍历
    visited = set()
    components = []
    
    for node_id in self.node_map:
        if node_id not in visited:
            # 找到一个新的连通分量
            component = set()
            queue = deque([node_id])
            visited.add(node_id)
            component.add(node_id)
            
            while queue:
                current = queue.popleft()
                for neighbor in undirected_graph[current]:
                    if neighbor not in visited:
                        visited.add(neighbor)
                        component.add(neighbor)
                        queue.append(neighbor)
            
            components.append(component)
    
    return components
```

### 环检测（DAG 验证）

**算法：** 深度优先搜索（DFS）+ 三色标记法

```python
def _is_dag(self, component: Set[str]) -> bool:
    """检查连通分量是否为 DAG（无环）"""
    # 三色标记：0=未访问, 1=正在访问, 2=已访问完成
    status = {node_id: 0 for node_id in component}
    
    def dfs(node_id):
        status[node_id] = 1  # 正在访问
        
        for neighbor in self.graph[node_id]:
            if neighbor in component:
                if status[neighbor] == 0:  # 未访问
                    if not dfs(neighbor):
                        return False
                elif status[neighbor] == 1:  # 回边，有环
                    return False
        
        status[node_id] = 2  # 访问完成
        return True
    
    for node_id in component:
        if status[node_id] == 0:
            if not dfs(node_id):
                return False
    
    return True
```

### Entry Nodes 识别

```python
def _find_entry_nodes(self, component: Set[str]) -> List[str]:
    """
    找出连通分量的入口节点（没有入边的节点）
    
    Returns:
        入口节点 ID 列表
    """
    # 找出所有有入边的节点
    nodes_with_incoming = set()
    for source in component:
        for target in self.graph[source]:
            if target in component:
                nodes_with_incoming.add(target)
    
    # 入口节点 = 所有节点 - 有入边的节点
    entry_nodes = component - nodes_with_incoming
    return list(entry_nodes)
```

### 结构分析结果

```python
def analyze_flow(self) -> Dict[str, Any]:
    """分析 Flow 结构"""
    components = self._find_connected_components()
    
    result = {
        "component_count": len(components),
        "components": {}
    }
    
    for i, component in enumerate(components):
        if self._is_dag(component):
            entry_nodes = self._find_entry_nodes(component)
            result["components"][str(i)] = {
                "nodes": list(component),
                "entry_nodes": entry_nodes,
                "node_count": len(component),
                "is_dag": True
            }
        else:
            result["components"][str(i)] = {
                "nodes": list(component),
                "is_dag": False,
                "error": "Contains cycle"
            }
    
    return result
```

**示例结果：**
```json
{
  "component_count": 2,
  "components": {
    "0": {
      "nodes": ["node_A", "node_B", "node_C"],
      "entry_nodes": ["node_A"],
      "node_count": 3,
      "is_dag": true
    },
    "1": {
      "nodes": ["node_D", "node_E"],
      "entry_nodes": ["node_D"],
      "node_count": 2,
      "is_dag": true
    }
  }
}
```

---

## 周期调度

### 调度循环

```python
async def _schedule_flow_execution(self, flow_id: str):
    """Flow 的周期性调度循环"""
    while True:
        try:
            # 1. 检查 Flow 状态
            flow_data = await self.redis.hgetall(f"flow:{flow_id}")
            if not flow_data or flow_data.get("status") != "running":
                logger.info("Flow %s is not running, stopping scheduler", flow_id)
                break
            
            # 2. 检查是否需要执行
            interval = int(json.loads(flow_data["config"]).get("interval", 60))
            next_execution = float(flow_data.get("next_execution", 0))
            current_time = time.time()
            
            if current_time >= next_execution:
                # 3. 执行 Cycle
                current_cycle = int(flow_data.get("last_cycle", -1)) + 1
                
                await self.persist_log(
                    flow_id=flow_id,
                    cycle=current_cycle,
                    message=f"Starting cycle {current_cycle}",
                    log_level="INFO",
                    log_source="scheduler"
                )
                
                await self._execute_flow_cycle(flow_id, current_cycle)
                
                # 4. 更新下次执行时间
                if interval > 0:
                    next_execution = current_time + interval
                    await self.redis.hset(
                        f"flow:{flow_id}",
                        mapping={
                            "last_cycle": current_cycle,
                            "next_execution": next_execution
                        }
                    )
                else:
                    # interval=0，只执行一次
                    await self.redis.hset(f"flow:{flow_id}", "status", "completed")
                    break
            
            # 5. 等待下一次检查
            await asyncio.sleep(5)
            
        except Exception as e:
            logger.error("Error in flow scheduling: %s", str(e))
            await asyncio.sleep(10)
```

### 执行时机控制

| Interval 值 | 行为 | 用途 |
|-------------|------|------|
| **0** | 执行一次后停止 | 一次性任务 |
| **60** | 每 60 秒执行一次 | 定时任务 |
| **300** | 每 5 分钟执行一次 | 周期监控 |
| **3600** | 每小时执行一次 | 定期报告 |

---

## Cycle 执行

### Cycle 执行流程

```python
async def _execute_flow_cycle(self, flow_id: str, cycle: int):
    """执行单个 Cycle"""
    try:
        # 1. 获取 Flow 配置和结构
        flow_data = await self.redis.hgetall(f"flow:{flow_id}")
        flow_config = json.loads(flow_data["config"])
        flow_structure = json.loads(flow_data["structure"])
        
        # 2. 创建 Cycle 记录
        cycle_key = f"flow:{flow_id}:cycle:{cycle}"
        cycle_data = {
            "flow_id": flow_id,
            "cycle": str(cycle),
            "status": "running",
            "start_time": datetime.now().isoformat(),
            "end_time": ""
        }
        await self.redis.hset(cycle_key, mapping=cycle_data)
        
        # 3. 创建节点映射
        node_map = {node["id"]: node for node in flow_config["nodes"]}
        
        # 4. 注册所有节点到 Cycle
        nodes_key = f"flow:{flow_id}:cycle:{cycle}:nodes"
        for node_id in node_map.keys():
            await self.redis.sadd(nodes_key, node_id)
        
        # 5. 为每个连通分量创建执行任务
        components = flow_structure.get("components", {})
        execution_tasks = []
        
        for component_id, component_info in components.items():
            if component_info.get("is_dag"):
                task = asyncio.create_task(
                    self._execute_component(
                        flow_id, cycle, int(component_id),
                        component_info, node_map, flow_config
                    )
                )
                execution_tasks.append(task)
        
        # 6. 等待所有组件执行完成
        await asyncio.gather(*execution_tasks, return_exceptions=True)
        
        # 7. 更新 Cycle 状态为完成
        await self.redis.hset(
            cycle_key,
            mapping={
                "status": "completed",
                "end_time": datetime.now().isoformat()
            }
        )
        
        await self.persist_log(
            flow_id=flow_id,
            cycle=cycle,
            message=f"Cycle {cycle} completed",
            log_level="INFO",
            log_source="scheduler"
        )
        
        # 8. 清理旧日志
        await self.cleanup_old_logs(flow_id, cycle, keep_cycles=5)
        
    except Exception as e:
        logger.error("Error executing cycle: %s", str(e))
        await self.redis.hset(cycle_key, "status", "failed")
        await self.persist_log(
            flow_id=flow_id,
            cycle=cycle,
            message=f"Cycle {cycle} failed: {str(e)}",
            log_level="ERROR",
            log_source="scheduler"
        )
```

---

## 节点编排

### Component 执行

```python
async def _execute_component(
    self,
    flow_id: str,
    cycle: int,
    component_id: int,
    component_info: Dict,
    node_map: Dict,
    flow_config: Dict
):
    """执行单个连通分量"""
    nodes = component_info.get("nodes", [])
    entry_nodes = component_info.get("entry_nodes", [])
    
    logger.info(
        "Executing component %d with %d nodes (entry nodes: %s)",
        component_id, len(nodes), entry_nodes
    )
    
    # 1. 准备边信息
    edges = flow_config.get("edges", [])
    
    # 2. 为每个节点创建执行任务
    node_tasks = []
    for node_id in nodes:
        # 检查是否需要跳过
        stop_flag = await self.redis.get(
            f"flow:{flow_id}:cycle:{cycle}:component:{component_id}:stop"
        )
        if stop_flag:
            logger.info("Component %d stopped, skipping node %s", component_id, node_id)
            continue
        
        # 创建节点执行任务
        task = asyncio.create_task(
            self._execute_node(
                flow_id, cycle, component_id,
                node_id, node_map, edges
            )
        )
        node_tasks.append(task)
    
    # 3. 等待所有节点执行完成
    results = await asyncio.gather(*node_tasks, return_exceptions=True)
    
    # 4. 统计结果
    success_count = sum(1 for r in results if r and not isinstance(r, Exception))
    failed_count = len(results) - success_count
    
    logger.info(
        "Component %d execution completed: %d success, %d failed",
        component_id, success_count, failed_count
    )
```

### 节点执行调度

```python
async def _execute_node(
    self,
    flow_id: str,
    cycle: int,
    component_id: int,
    node_id: str,
    node_map: Dict,
    edges: List
):
    """调度单个节点执行"""
    try:
        # 1. 获取节点信息
        node_info = node_map.get(node_id)
        if not node_info:
            logger.error("Node %s not found in node_map", node_id)
            return {"status": "error", "message": "Node not found"}
        
        node_type = node_info.get("type")
        node_config = node_info.get("config", {})
        
        # 2. 准备输入输出边
        input_edges = [
            edge for edge in edges
            if edge.get("target") == node_id
        ]
        output_edges = [
            edge for edge in edges
            if edge.get("source") == node_id
        ]
        
        # 3. 构建节点数据
        node_data = {
            "config": node_config,
            "input_edges": input_edges,
            "output_edges": output_edges
        }
        
        # 4. 选择 Worker 并执行
        node_registry = NodeRegistry.get_instance()
        workers = await node_registry.get_available_workers(node_type)
        
        if not workers:
            logger.warning("No available workers for node type %s", node_type)
            return {"status": "error", "message": "No available workers"}
        
        # 随机选择一个 Worker
        worker = random.choice(workers)
        worker_url = worker.get("api_url")
        
        # 5. 调用 Worker API
        node_task_id = f"{flow_id}_{cycle}_{node_id}"
        
        async with httpx.AsyncClient(timeout=300.0) as client:
            response = await client.post(
                f"{worker_url}/execute",
                json={
                    "node_task_id": node_task_id,
                    "flow_id": flow_id,
                    "component_id": component_id,
                    "cycle": cycle,
                    "node_id": node_id,
                    "node_type": node_type,
                    "node_data": node_data
                }
            )
            
            if response.status_code == 200:
                return response.json()
            else:
                logger.error("Worker execution failed: %s", response.text)
                return {"status": "error", "message": response.text}
        
    except Exception as e:
        logger.error("Error executing node %s: %s", node_id, str(e))
        return {"status": "error", "message": str(e)}
```

---

## 状态管理

### Flow 状态查询

```python
async def get_flow_status(self, flow_id: str) -> Dict:
    """获取 Flow 状态信息"""
    flow_data = await self.redis.hgetall(f"flow:{flow_id}")
    if not flow_data:
        raise ValueError(f"Flow {flow_id} does not exist")
    
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

### Cycle 状态查询

```python
async def get_cycle_status(self, flow_id: str, cycle: int) -> Dict:
    """获取 Cycle 执行状态"""
    # 1. 获取 Cycle 基本信息
    cycle_key = f"flow:{flow_id}:cycle:{cycle}"
    cycle_data = await self.redis.hgetall(cycle_key)
    
    if not cycle_data:
        return {"error": f"Cycle {cycle} does not exist"}
    
    # 2. 获取所有节点状态
    nodes_key = f"flow:{flow_id}:cycle:{cycle}:nodes"
    node_ids = await self.redis.smembers(nodes_key)
    
    # 3. 从 NodeTaskManager 获取节点详情
    task_manager = NodeTaskManager.get_instance()
    if not task_manager._initialized:
        await task_manager.initialize()
    
    nodes_status = {}
    for node_id in node_ids:
        node_task_id = f"{flow_id}_{cycle}_{node_id}"
        node_data = await task_manager.get_task(node_task_id)
        if node_data:
            nodes_status[node_id] = node_data
    
    return {
        "cycle": cycle,
        "status": cycle_data.get("status", "unknown"),
        "start_time": cycle_data.get("start_time"),
        "end_time": cycle_data.get("end_time"),
        "nodes": nodes_status,
        "node_count": len(node_ids)
    }
```

### 综合状态查询

```python
async def get_comprehensive_flow_status(self, flow_id: str, cycle: int = None) -> Dict:
    """获取包含节点状态、日志、信号的综合状态"""
    # 1. 获取 Flow 基本信息
    flow_data = await self.redis.hgetall(f"flow:{flow_id}")
    if not flow_data:
        raise ValueError(f"Flow {flow_id} does not exist")
    
    # 2. 确定 Cycle
    if cycle is None:
        cycle = int(flow_data.get("last_cycle", 0))
    
    # 3. 获取节点管理器
    node_manager = NodeTaskManager.get_instance()
    if not node_manager._initialized:
        await node_manager.initialize()
    
    # 4. 获取所有节点的综合状态
    comprehensive_nodes = await node_manager.get_comprehensive_node_status(
        flow_id=flow_id,
        cycle=cycle
    )
    
    # 5. 统计信息
    total_nodes = len(comprehensive_nodes)
    running_nodes = sum(1 for n in comprehensive_nodes.values() if n['status'] == 'running')
    completed_nodes = sum(1 for n in comprehensive_nodes.values() if n['status'] == 'completed')
    error_nodes = sum(1 for n in comprehensive_nodes.values() if n['status'] == 'error')
    
    return {
        "flow_id": flow_id,
        "cycle": cycle,
        "flow_status": flow_data.get("status"),
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

## 日志系统

### 日志持久化

```python
async def persist_log(
    self,
    flow_id: str,
    cycle: int,
    message: str,
    log_level: str = "INFO",
    log_source: str = "scheduler",
    node_id: Optional[str] = None,
    log_metadata: Optional[Dict] = None
):
    """持久化日志到数据库"""
    # 1. 先输出到控制台
    log_level_upper = log_level.upper()
    if log_level_upper == "DEBUG":
        logger.debug(message)
    elif log_level_upper == "INFO":
        logger.info(message)
    elif log_level_upper == "WARNING":
        logger.warning(message)
    elif log_level_upper == "ERROR":
        logger.error(message)
    
    # 2. 持久化到数据库
    try:
        await self._initialize_log_service()
        if self._log_service:
            await self._log_service.create_log(
                flow_id=flow_id,
                cycle=cycle,
                message=message,
                node_id=node_id,
                log_level=log_level_upper,
                log_source=log_source,
                log_metadata=log_metadata
            )
    except Exception as e:
        logger.warning("Failed to persist log: %s", str(e))
```

### 日志清理

```python
async def cleanup_old_logs(self, flow_id: str, current_cycle: int, keep_cycles: int = 5):
    """清理旧日志，只保留最近 N 个 Cycle"""
    try:
        await self._initialize_log_service()
        if self._log_service:
            # 计算要保留的最旧 Cycle
            oldest_cycle_to_keep = max(0, current_cycle - keep_cycles + 1)
            
            # 删除更旧的日志
            deleted_count = await self._log_service.delete_logs_before_cycle(
                flow_id=flow_id,
                cycle=oldest_cycle_to_keep
            )
            
            if deleted_count > 0:
                await self.persist_log(
                    flow_id=flow_id,
                    cycle=current_cycle,
                    message=f"Cleaned up {deleted_count} old log entries",
                    log_level="INFO",
                    log_source="scheduler"
                )
    except Exception as e:
        logger.warning("Failed to cleanup old logs: %s", str(e))
```

---

## 总结

### 调度器设计亮点

1. **并发执行**：连通分量并行执行，节点并行执行
2. **灵活调度**：支持一次性、周期性、定时执行
3. **状态隔离**：不同 Cycle 的状态完全隔离
4. **容错能力**：单个节点失败不影响其他节点
5. **可观测性**：完整的日志记录和状态查询

### 关键时序

```
T=0s    注册 Flow
T=1s    启动 Flow 调度
T=5s    第一次检查，执行 Cycle 0
T=65s   第二次检查，执行 Cycle 1（假设 interval=60）
T=125s  第三次检查，执行 Cycle 2
...
```

### 下一步

- **[开发指南](weather-station-development-guide.md)** - 完整开发流程
- **[架构概述](weather-station-overview.md)** - 返回概述

---

**维护者：** TradingFlow 开发团队  
**版本历史：** 
- v1.0.0 (2025-10-06): 初始版本

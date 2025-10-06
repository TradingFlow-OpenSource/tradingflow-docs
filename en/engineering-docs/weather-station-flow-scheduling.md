# Weather Station Flow Scheduling Mechanism

**Version:** 1.0.0  
**Document Type:** Technical Deep Dive  
**Last Updated:** 2025-10-06

---

## 📖 About This Document

This document explains how Weather Station's FlowScheduler manages strategy execution, cycle scheduling, and DAG orchestration.

**Target Audience:**
- System Architects
- Backend Engineers
- DevOps Engineers

**Prerequisites:**
- Understanding of [Architecture Overview](weather-station-overview.md)
- Knowledge of DAG concepts
- Familiarity with distributed scheduling

**Reading Time:** ~60 minutes

---

## 🎯 FlowScheduler Overview

The **FlowScheduler** is the central orchestrator that manages the execution lifecycle of trading strategies (Flows).

**Core Responsibilities:**
- Flow registration and validation
- DAG topology analysis
- Cycle-based scheduling
- Node execution coordination
- State tracking and cleanup

---

## 🏗️ Scheduler Architecture

### Component Structure

```
┌─────────────────────────────────────────────────────┐
│              FlowScheduler                           │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │Flow Registry │  │Cycle Manager │  │ Node     │ │
│  │              │  │              │  │Dispatcher│ │
│  └──────┬───────┘  └──────┬───────┘  └────┬─────┘ │
│         │                 │                │       │
│         └─────────────────┴────────────────┘       │
│                           │                         │
└───────────────────────────┼─────────────────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
         ┌─────────┐              ┌─────────┐
         │  Redis  │              │RabbitMQ │
         │ (State) │              │(Signals)│
         └─────────┘              └─────────┘
```

### Singleton Pattern

```python
class FlowScheduler:
    _instance = None
    
    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance
```

---

## 📋 Flow Registration

### Registration Process

```python
async def register_flow(
    self,
    flow_id: str,
    flow_config: Dict[str, Any]
) -> bool:
    """
    Register a flow for scheduling
    
    Steps:
    1. Parse flow JSON
    2. Validate DAG structure
    3. Identify entry nodes
    4. Store configuration
    5. Set up scheduling
    """
    
    # Parse flow
    flow_parser = FlowParser(flow_config)
    
    # Analyze DAG
    dags = flow_parser.find_dags()
    if not dags:
        raise FlowValidationError("No valid DAG found")
    
    # Find entry nodes (no input edges)
    entry_nodes = self._find_entry_nodes(flow_config)
    
    # Store in Redis
    await self.state_store.set(
        f"station:flow:{flow_id}:config",
        json.dumps(flow_config)
    )
    
    # Register schedule
    if flow_config.get('schedule_type') == 'periodic':
        await self._setup_periodic_schedule(flow_id)
    
    return True
```

### DAG Validation

```python
def _validate_dag(self, nodes, edges):
    """
    Validate that graph is a proper DAG:
    1. No cycles
    2. All edges point to existing nodes
    3. At least one entry node
    """
    
    # Build adjacency list
    graph = defaultdict(list)
    for edge in edges:
        graph[edge['source']].append(edge['target'])
    
    # Detect cycles using DFS
    visited = set()
    rec_stack = set()
    
    def has_cycle(node):
        visited.add(node)
        rec_stack.add(node)
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                if has_cycle(neighbor):
                    return True
            elif neighbor in rec_stack:
                return True
        
        rec_stack.remove(node)
        return False
    
    # Check all nodes
    for node_id in nodes:
        if node_id not in visited:
            if has_cycle(node_id):
                raise CyclicGraphError(f"Cycle detected in graph")
    
    return True
```

---

## 🔄 Cycle Scheduling

### Periodic Scheduling

```python
async def _setup_periodic_schedule(
    self,
    flow_id: str
):
    """Setup periodic execution for a flow"""
    
    # Get schedule config
    config = await self._get_flow_config(flow_id)
    interval_seconds = config.get('interval_seconds', 60)
    
    # Create async task
    async def schedule_loop():
        while True:
            try:
                # Check if flow is active
                if await self._is_flow_active(flow_id):
                    await self.start_cycle(flow_id)
                
                # Wait for next cycle
                await asyncio.sleep(interval_seconds)
                
            except Exception as e:
                logger.error(f"Schedule error for {flow_id}: {e}")
    
    # Start background task
    asyncio.create_task(schedule_loop())
```

### Cycle Initialization

```python
async def start_cycle(
    self,
    flow_id: str
) -> int:
    """
    Start a new cycle for the flow
    
    Returns:
        cycle: The cycle number
    """
    
    # Get next cycle number
    cycle = await self._get_next_cycle(flow_id)
    
    # Initialize cycle state
    await self.state_store.hset(
        f"station:flow:{flow_id}:cycle:{cycle}:status",
        mapping={
            "cycle": cycle,
            "status": "running",
            "started_at": time.time(),
            "total_nodes": len(self._get_nodes(flow_id)),
            "completed_nodes": 0
        }
    )
    
    # Get entry nodes
    entry_nodes = await self._get_entry_nodes(flow_id)
    
    # Dispatch entry nodes for execution
    for node_id in entry_nodes:
        await self._dispatch_node(flow_id, cycle, node_id)
    
    return cycle
```

---

## 🎯 Node Orchestration

### Node Dispatching

```python
async def _dispatch_node(
    self,
    flow_id: str,
    cycle: int,
    node_id: str
):
    """Dispatch a node for execution"""
    
    # Get node configuration
    node_config = await self._get_node_config(flow_id, node_id)
    
    # Create node instance
    node_class = NodeRegistry.get_node_class(node_config['type'])
    node = node_class(
        node_id=node_id,
        flow_id=flow_id,
        cycle=cycle,
        parameters=node_config.get('parameters', {}),
        input_edges=self._get_input_edges(flow_id, node_id),
        output_edges=self._get_output_edges(flow_id, node_id)
    )
    
    # Register task
    NodeTaskManager.get_instance().register_task(
        node_id, flow_id, cycle
    )
    
    # Execute node (async)
    asyncio.create_task(node.start())
```

### Execution Coordination

```python
async def on_node_completed(
    self,
    flow_id: str,
    cycle: int,
    node_id: str
):
    """
    Called when a node completes execution
    
    Responsibilities:
    1. Update cycle progress
    2. Check if cycle completed
    3. Trigger dependent nodes if needed
    """
    
    # Update progress
    await self.state_store.hincrby(
        f"station:flow:{flow_id}:cycle:{cycle}:status",
        "completed_nodes",
        1
    )
    
    # Check if cycle completed
    cycle_status = await self.state_store.hgetall(
        f"station:flow:{flow_id}:cycle:{cycle}:status"
    )
    
    if int(cycle_status['completed_nodes']) >= int(cycle_status['total_nodes']):
        await self._complete_cycle(flow_id, cycle)
```

### Cycle Completion

```python
async def _complete_cycle(
    self,
    flow_id: str,
    cycle: int
):
    """Mark cycle as completed and cleanup"""
    
    # Update cycle status
    await self.state_store.hset(
        f"station:flow:{flow_id}:cycle:{cycle}:status",
        mapping={
            "status": "completed",
            "completed_at": time.time()
        }
    )
    
    # Log completion
    logger.info(f"Cycle {cycle} completed for flow {flow_id}")
    
    # Optional: Archive old cycle data
    if cycle > 10:
        await self._archive_old_cycle(flow_id, cycle - 10)
```

---

## 🔍 State Management

### Flow State Queries

```python
async def get_flow_status(
    self,
    flow_id: str
) -> Dict[str, Any]:
    """Get comprehensive flow status"""
    
    # Get flow config
    config = await self._get_flow_config(flow_id)
    
    # Get latest cycle
    latest_cycle = await self._get_latest_cycle(flow_id)
    
    # Get cycle status
    cycle_status = None
    if latest_cycle is not None:
        cycle_status = await self.state_store.hgetall(
            f"station:flow:{flow_id}:cycle:{latest_cycle}:status"
        )
    
    return {
        "flow_id": flow_id,
        "name": config.get('name'),
        "status": config.get('status', 'active'),
        "schedule_type": config.get('schedule_type'),
        "latest_cycle": latest_cycle,
        "cycle_status": cycle_status
    }
```

### Node State Queries

```python
async def get_node_status(
    self,
    flow_id: str,
    cycle: int,
    node_id: str
) -> Dict[str, Any]:
    """Get node execution status"""
    
    node_state = await self.state_store.hgetall(
        f"station:node:{node_id}:state"
    )
    
    return {
        "node_id": node_id,
        "status": node_state.get('status'),
        "started_at": node_state.get('started_at'),
        "completed_at": node_state.get('completed_at'),
        "error": node_state.get('error_message')
    }
```

---

## 🛡️ Error Handling

### Node Failure Handling

```python
async def on_node_failed(
    self,
    flow_id: str,
    cycle: int,
    node_id: str,
    error: Exception
):
    """Handle node execution failure"""
    
    # Log error
    logger.error(
        f"Node {node_id} failed in cycle {cycle}: {error}"
    )
    
    # Check retry configuration
    retry_config = await self._get_retry_config(flow_id, node_id)
    
    if retry_config.get('enabled'):
        # Attempt retry
        retry_count = await self._get_retry_count(node_id)
        max_retries = retry_config.get('max_retries', 3)
        
        if retry_count < max_retries:
            # Schedule retry
            await self._schedule_retry(
                flow_id, cycle, node_id, retry_count + 1
            )
            return
    
    # Mark cycle as failed
    await self.state_store.hset(
        f"station:flow:{flow_id}:cycle:{cycle}:status",
        "status",
        "failed"
    )
    
    # Send alert
    await self._send_alert(flow_id, cycle, node_id, error)
```

---

## 📊 Monitoring and Logging

### Execution Metrics

```python
async def get_execution_metrics(
    self,
    flow_id: str
) -> Dict[str, Any]:
    """Get execution statistics"""
    
    # Query recent cycles
    recent_cycles = await self._get_recent_cycles(flow_id, limit=10)
    
    # Calculate metrics
    total_executions = len(recent_cycles)
    successful = sum(1 for c in recent_cycles if c['status'] == 'completed')
    failed = sum(1 for c in recent_cycles if c['status'] == 'failed')
    
    # Calculate average execution time
    execution_times = [
        c['completed_at'] - c['started_at']
        for c in recent_cycles
        if c.get('completed_at')
    ]
    avg_time = sum(execution_times) / len(execution_times) if execution_times else 0
    
    return {
        "flow_id": flow_id,
        "total_executions": total_executions,
        "successful": successful,
        "failed": failed,
        "success_rate": successful / total_executions if total_executions > 0 else 0,
        "avg_execution_time_seconds": avg_time
    }
```

---

## 🧹 Cleanup and Maintenance

### Data Retention

```python
async def cleanup_old_data(self):
    """Remove old cycle data"""
    
    # Get all flows
    flows = await self._get_all_flows()
    
    for flow_id in flows:
        # Get all cycles
        cycles = await self._get_all_cycles(flow_id)
        
        # Keep only recent 100 cycles
        if len(cycles) > 100:
            old_cycles = cycles[:-100]
            for cycle in old_cycles:
                await self._delete_cycle_data(flow_id, cycle)
```

---

## 📚 Related Documentation

- [Architecture Overview](weather-station-overview.md)
- [Message Queue Details](weather-station-message-queue.md)
- [Redis State Management](weather-station-redis.md)
- [Node Execution Flow](weather-station-node-execution.md)
- [Node Reference](../node-details/index.md)

---

**Maintained by:** TradingFlow Engineering Team  
**Last Updated:** 2025-10-06  
**Version:** 1.0.0

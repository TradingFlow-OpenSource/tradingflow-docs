# Weather Station 工作流引擎 - 架构概述

**文档版本：** 1.0.0  
**最后更新：** 2025-10-06  
**目标读者：** 后端开发者、AI助手、系统架构师

---

## 目录

- [简介](#简介)
- [核心概念](#核心概念)
- [系统架构](#系统架构)
- [关键组件](#关键组件)
- [技术栈](#技术栈)
- [设计模式](#设计模式)
- [数据流](#数据流)

---

## 简介

Weather Station 是 TradingFlow 的核心工作流执行引擎，负责：

1. **Flow 生命周期管理**：从注册、启动到停止的完整生命周期
2. **节点编排与执行**：基于 DAG 的节点依赖管理和并发执行
3. **信号传递机制**：基于 RabbitMQ 的节点间异步通信
4. **状态管理**：基于 Redis 的分布式状态存储
5. **周期调度**：支持定时、周期性 Flow 执行

### 设计理念

- **松耦合通信**：节点间通过信号（Signal）通信，不直接调用
- **分布式架构**：支持多 Worker 实例并行处理
- **灵活扩展**：基于装饰器的节点注册机制
- **高可靠性**：完善的错误处理和状态恢复机制

---

## 核心概念

### 1. Weather Syntax (Weather 语法)

Weather Syntax 是 TradingFlow 的智能交易语言，用 JSON 格式描述工作流。

**基本结构：**
```json
{
  "nodes": [
    {
      "id": "node_1",
      "type": "binance_price_node",
      "config": { /* 节点配置 */ }
    }
  ],
  "edges": [
    {
      "source": "node_1",
      "source_handle": "kline_data",
      "target": "node_2",
      "target_handle": "input_data"
    }
  ]
}
```

### 2. DAG (有向无环图)

- **定义**：Weather Syntax 中的节点和边构成 DAG
- **连通分量**：一个 Flow 可包含多个 DAG（连通分量）
- **Component ID**：每个连通分量有唯一 ID，用于隔离控制

### 3. Flow、Cycle、Node

```
Weather Syntax (静态描述)
    └─ Flow (运行实例)
           └─ Cycle 0 (第 1 次执行)
           │     └─ Node A, Node B, Node C...
           └─ Cycle 1 (第 2 次执行)
                 └─ Node A, Node B, Node C...
```

**关键概念：**
- **Flow ID**：一个 Flow 的唯一运行实例标识
- **Cycle**：Flow 的执行周期号，从 0 开始自增
- **Node ID**：Flow 中节点的唯一标识
- **Node Task ID**：格式为 `{flow_id}_{cycle}_{node_id}`，标识一次具体的节点执行

### 4. Signal（信号）

节点间通信的数据载体。

**Signal 结构：**
```python
@dataclass
class Signal:
    signal_type: SignalType          # 信号类型
    payload: Dict[str, Any]          # 数据负载
    timestamp: Optional[datetime]    # 时间戳
```

**Signal 类型：**
- `DATA_READY`: 通用数据就绪
- `PRICE_DATA`: 价格数据
- `DATASET`: 数据集
- `DEX_TRADE_RECEIPT`: 交易收据
- `VAULT_INFO`: 金库信息
- `AI_RESPONSE`: AI 响应
- `STOP_EXECUTION`: 停止执行信号

### 5. Handle（句柄）

节点的输入输出连接点。

**Handle 类型：**
- **Input Handle**：接收上游信号
- **Output Handle**：发送信号到下游
- **Aggregate Handle**：聚合多个输入（`is_aggregate=True`）

**Handle 命名规范：**
```python
# 输入句柄
INPUT_HANDLE = "input_data"
# 输出句柄  
OUTPUT_HANDLE = "output_data"
# Handle ID 格式
handle_id = f"{field_name}"  # 不包含 -handle 后缀
```

---

## 系统架构

### 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                      Weather Control                        │
│                    (API Gateway Layer)                      │
└────────────────┬────────────────────────────────────────────┘
                 │ HTTP API
                 ↓
┌─────────────────────────────────────────────────────────────┐
│                    Weather Station                          │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              FlowScheduler (调度器)                   │  │
│  │  - Flow 注册与管理                                    │  │
│  │  - 周期调度                                          │  │
│  │  - DAG 分析                                          │  │
│  └──────────────────┬───────────────────────────────────┘  │
│                     │                                       │
│                     ↓                                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │            Worker Instance Pool                      │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │  │
│  │  │ Worker 1 │  │ Worker 2 │  │ Worker N │          │  │
│  │  │          │  │          │  │          │          │  │
│  │  │ NodeBase │  │ NodeBase │  │ NodeBase │          │  │
│  │  │ Execute  │  │ Execute  │  │ Execute  │          │  │
│  │  └──────────┘  └──────────┘  └──────────┘          │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                     │           │
        ┌────────────┴───────────┴────────────┐
        │                                     │
        ↓                                     ↓
┌───────────────────┐              ┌──────────────────┐
│   RabbitMQ        │              │   Redis          │
│   (Message Queue) │              │   (State Store)  │
│                   │              │                  │
│ - signals_exchange│              │ - Flow状态       │
│ - Topic路由       │              │ - Cycle状态      │
│ - Queue隔离       │              │ - Node任务状态   │
└───────────────────┘              └──────────────────┘
```

### 分层架构

| 层级 | 组件 | 职责 |
|------|------|------|
| **调度层** | FlowScheduler | Flow生命周期管理、周期调度 |
| **编排层** | FlowParser | Weather Syntax解析、DAG分析、连通分量识别 |
| **执行层** | NodeExecutor, NodeBase | 节点创建、执行、生命周期管理 |
| **通信层** | NodeSignalPublisher/Consumer | 信号发布订阅、消息路由 |
| **存储层** | StateStore, NodeTaskManager | 状态持久化、任务管理 |

---

## 关键组件

### 1. FlowScheduler（Flow 调度器）

**位置：** `tradingflow/station/flow/scheduler.py` (1546 行)

**核心职责：**
- Flow 注册与配置管理
- 周期调度循环
- DAG 结构分析
- Cycle 执行控制
- 状态查询接口

**关键方法：**
```python
async def register_flow(flow_id: str, flow_config: Dict)
async def start_flow(flow_id: str)
async def stop_flow(flow_id: str)
async def execute_cycle(flow_id: str, cycle: int)
async def get_flow_status(flow_id: str) -> Dict
async def get_cycle_status(flow_id: str, cycle: int) -> Dict
```

**单例模式：**
```python
_scheduler_instance = None

def get_scheduler_instance():
    global _scheduler_instance
    if _scheduler_instance is None:
        _scheduler_instance = FlowScheduler()
    return _scheduler_instance
```

### 2. FlowParser（Flow 解析器）

**位置：** `tradingflow/station/flow/flow_parser.py` (216 行)

**核心职责：**
- Weather Syntax JSON 解析
- 连通分量识别
- DAG 环检测
- Entry Nodes 识别

**关键方法：**
```python
def find_dags() -> List[Dict[str, Any]]
def _find_connected_components() -> List[Set[str]]
def _is_dag(component: Set[str]) -> bool
def analyze_flow() -> Dict[str, Any]
```

**算法：**
- **连通分量识别**：BFS（广度优先搜索）
- **环检测**：DFS（深度优先搜索）+ 三色标记法

### 3. NodeBase（节点基类）

**位置：** `tradingflow/station/nodes/node_base.py` (1481 行)

**核心职责：**
- 节点生命周期管理
- Input/Output Handle 注册
- 信号接收与发送
- 状态管理
- 日志持久化

**生命周期方法：**
```python
async def initialize_state_store() -> bool
async def initialize_message_queue() -> bool
async def start() -> bool          # 启动节点
async def execute() -> bool        # 执行节点逻辑（子类实现）
async def cleanup()                # 清理资源
```

**信号处理：**
```python
async def _on_signal_received(signal: Signal, handle: str)
async def send_signal(source_handle: str, signal_type: SignalType, payload: Dict)
```

### 4. NodeExecutor（节点执行器）

**位置：** `tradingflow/station/core/node_executor.py` (283 行)

**核心职责：**
- 节点实例创建
- 节点执行控制
- 异常处理与状态更新
- 资源清理

**执行流程：**
```python
async def execute_node_task(
    node_task_id: str,
    flow_id: str,
    component_id: int,
    cycle: int,
    node_id: str,
    node_type: str,
    node_data: Dict[str, Any]
)
```

### 5. NodeSignalPublisher/Consumer

**位置：** `tradingflow/depot/python/mq/node_signal_*.py`

#### NodeSignalPublisher（信号发布者）

**职责：**
- 发送节点输出信号
- 动态路由键生成
- 支持精确发送和通配符发送

**关键方法：**
```python
async def send_signal(source_handle: str, signal: Signal)
async def send_stop_execution_signal(reason: str)
```

#### NodeSignalConsumer（信号消费者）

**职责：**
- 订阅上游节点信号
- Routing Key 解析
- Cycle 验证
- 信号分发到 Handle

**关键方法：**
```python
async def consume()
async def on_signal(signal: Signal, handle: str, signal_context: Dict)
def _parse_routing_key(routing_key: str) -> Dict[str, Any]
```

### 6. StateStore（状态存储）

**位置：** `tradingflow/station/common/state_store.py` (611 行)

**实现：**
- **StateStore**：抽象基类（防腐层）
- **RedisStateStore**：Redis 实现
- **InMemoryStateStore**：内存实现（测试用）
- **StateStoreFactory**：工厂模式创建

**接口：**
```python
# 节点状态
async def set_node_task_status(node_task_id: str, status: str)
async def get_node_task_status(node_task_id: str) -> Dict

# 终止标志
async def set_termination_flag(node_task_id: str, reason: str)
async def get_termination_flag(node_task_id: str) -> Optional[Dict]

# 通用KV
async def set_value(key: str, value: Any)
async def get_value(key: str) -> Any

# 集合操作
async def add_to_set(key: str, value: Any)
async def get_set_members(key: str) -> List[Any]
```

### 7. NodeTaskManager（任务管理器）

**位置：** `tradingflow/station/common/node_task_manager.py` (593 行)

**职责：**
- 节点任务注册与跟踪
- 任务状态更新
- Worker 任务管理
- 任务查询接口

**单例模式：**
```python
NodeTaskManager.get_instance()
```

**关键方法：**
```python
async def register_task(node_task_id: str, task_info: Dict)
async def update_task_status(node_task_id: str, status: str, additional_info: Dict)
async def get_task(node_task_id: str) -> Optional[Dict]
async def get_worker_tasks(worker_id: str) -> List[Dict]
async def stop_task(node_task_id: str) -> bool
```

### 8. NodeRegistry（节点注册表）

**位置：** `tradingflow/station/common/node_registry.py`

**职责：**
- 节点类型注册
- 节点实例创建
- Worker 实例管理
- 心跳机制

**装饰器注册：**
```python
@register_node_type("binance_price_node", default_params={...})
class BinancePriceNode(NodeBase):
    pass
```

---

## 技术栈

### 核心技术

| 技术 | 用途 | 版本 |
|------|------|------|
| **Python** | 主要编程语言 | 3.9+ |
| **asyncio** | 异步编程框架 | 内置 |
| **RabbitMQ** | 消息队列 | 3.x |
| **aio-pika** | 异步 RabbitMQ 客户端 | 9.x |
| **Redis** | 状态存储 | 6.x+ |
| **redis.asyncio** | 异步 Redis 客户端 | 内置 |
| **PostgreSQL** | 日志持久化 | 14+ |

### Python 依赖

```python
# 异步
asyncio
aioredis

# 消息队列
aio-pika

# 数据库
psycopg2-binary
sqlalchemy

# HTTP
httpx

# 工具
dataclasses
typing
logging
```

---

## 设计模式

### 1. 单例模式（Singleton）

**应用组件：**
- FlowScheduler
- NodeRegistry
- NodeTaskManager

**目的：** 确保全局唯一实例，共享状态

**实现：**
```python
class FlowScheduler:
    _instance = None
    
    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance
```

### 2. 工厂模式（Factory）

**应用组件：**
- StateStoreFactory
- NodeRegistry.create_node()

**目的：** 根据配置动态创建对象

**实现：**
```python
class StateStoreFactory:
    @staticmethod
    def create(store_type: str, config: Dict):
        if store_type == "redis":
            return RedisStateStore(**config)
        elif store_type == "memory":
            return InMemoryStateStore()
```

### 3. 装饰器模式（Decorator）

**应用组件：**
- @register_node_type

**目的：** 自动注册节点类型

**实现：**
```python
def register_node_type(node_type: str, default_params: Dict = None):
    def decorator(cls):
        NodeRegistry.get_instance().register_node(
            node_type, 
            cls, 
            default_params
        )
        return cls
    return decorator
```

### 4. 观察者模式（Observer）

**应用场景：** 信号机制

**实现：**
- Publisher: NodeSignalPublisher
- Subscriber: NodeSignalConsumer
- Event: Signal

### 5. 状态机模式（State Machine）

**应用场景：** 节点状态管理

**状态转换：**
```
PENDING → RUNNING → COMPLETED
              ↓
            FAILED / TERMINATED / SKIPPED
```

---

## 数据流

### 完整执行流程

```
1. Flow 注册
   FlowScheduler.register_flow()
      ↓
   分析 Weather Syntax 结构（FlowParser）
      ↓
   存储 Flow 配置到 Redis

2. Flow 启动
   FlowScheduler.start_flow()
      ↓
   启动周期调度循环

3. Cycle 执行
   _schedule_flow_execution()
      ↓
   _execute_flow_cycle()
      ↓
   为所有节点创建执行任务
      ↓
   _execute_node() → 调用 Worker API

4. 节点执行（Worker 侧）
   execute_node_task()
      ↓
   创建节点实例
      ↓
   node_instance.start()
      ↓
   初始化状态存储
      ↓
   初始化消息队列
      ↓
   等待所有输入信号
      ↓
   node_instance.execute()（子类实现）
      ↓
   发送输出信号
      ↓
   清理资源

5. 信号传递
   NodeSignalPublisher.send_signal()
      ↓
   生成 Routing Key
      ↓
   发布到 RabbitMQ Exchange
      ↓
   根据 Routing Key 路由到目标 Queue
      ↓
   NodeSignalConsumer.consume()
      ↓
   解析 Routing Key
      ↓
   验证 Cycle
      ↓
   调用 node._on_signal_received()
      ↓
   更新输入信号状态
      ↓
   检查是否所有信号就绪
      ↓
   触发节点执行
```

### 信号流示例

```
节点 A (BinancePriceNode)
    ↓ 执行完成
    ↓ send_signal("kline_data", PRICE_DATA, payload)
    ↓
RabbitMQ
    ↓ Routing Key: flow.xxx.component.0.cycle.0.from.A.handle.kline_data.to.B.handle.price_input
    ↓
节点 B (AIModelNode)
    ↓ consume() 接收消息
    ↓ 解析 routing_key
    ↓ _on_signal_received(signal, "price_input")
    ↓ 所有输入信号就绪
    ↓ execute()
    ↓ send_signal("ai_response", AI_RESPONSE, payload)
    ↓
RabbitMQ
    ↓ Routing Key: flow.xxx.component.0.cycle.0.from.B.handle.ai_response.to.C.handle.decision_input
    ↓
节点 C (BuyNode)
    ↓ consume() 接收消息
    ↓ execute() 执行交易
```

---

## 下一步

阅读以下详细文档以深入了解各个方面：

1. **[消息队列详解](weather-station-message-queue.md)** - RabbitMQ 路由设计和信号传递
2. **[Redis 状态管理](weather-station-redis.md)** - 状态存储结构和字段设计
3. **[节点执行流程](weather-station-node-execution.md)** - 节点生命周期和执行机制
4. **[Flow 调度机制](weather-station-flow-scheduling.md)** - 周期调度和 DAG 编排
5. **[开发指南](weather-station-development-guide.md)** - 如何开发新节点和扩展功能

---

**维护者：** TradingFlow 开发团队  
**反馈：** 如有问题请在相关 Issue 中讨论  
**版本历史：** 
- v1.0.0 (2025-10-06): 初始版本

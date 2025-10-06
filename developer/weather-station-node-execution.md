# Weather Station - 节点执行流程

**文档版本：** 1.0.0  
**最后更新：** 2025-10-06  
**前置阅读：** [架构概述](weather-station-overview.md), [消息队列详解](weather-station-message-queue.md)

---

## 目录

- [节点生命周期](#节点生命周期)
- [节点创建流程](#节点创建流程)
- [信号等待机制](#信号等待机制)
- [节点执行流程](#节点执行流程)
- [信号发送机制](#信号发送机制)
- [异常处理](#异常处理)
- [资源清理](#资源清理)
- [开发新节点](#开发新节点)

---

## 节点生命周期

### 完整生命周期图

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 创建阶段 (Creation)                                      │
│    NodeRegistry.create_node()                               │
│    └─> NodeBase.__init__()                                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. 初始化阶段 (Initialization)                              │
│    node.start()                                             │
│    ├─> initialize_state_store()                             │
│    ├─> initialize_message_queue()                           │
│    └─> set_status(PENDING)                                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. 等待信号阶段 (Waiting for Signals)                       │
│    ├─> 启动信号消费者 consume()                             │
│    ├─> 接收上游信号 _on_signal_received()                   │
│    ├─> 更新输入信号状态                                      │
│    └─> 检查是否所有信号就绪 can_execute()                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓ (所有信号就绪)
┌─────────────────────────────────────────────────────────────┐
│ 4. 执行阶段 (Execution)                                      │
│    set_status(RUNNING)                                       │
│    └─> execute() [子类实现]                                 │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. 信号发送阶段 (Signal Emission)                           │
│    send_signal(handle, signal_type, payload)                │
│    └─> 下游节点接收信号                                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. 完成阶段 (Completion)                                     │
│    ├─> set_status(COMPLETED / FAILED / TERMINATED)          │
│    └─> 更新任务状态到 Redis                                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 7. 清理阶段 (Cleanup)                                        │
│    cleanup()                                                 │
│    ├─> close_message_queue()                                │
│    └─> close_state_store()                                  │
└─────────────────────────────────────────────────────────────┘
```

### 状态转换图

```
PENDING (初始状态)
    │
    ↓ (start() 调用)
RUNNING (执行中)
    │
    ├─> COMPLETED (成功完成)
    │
    ├─> FAILED (执行失败)
    │
    ├─> TERMINATED (被终止)
    │
    └─> SKIPPED (被跳过)
```

---

## 节点创建流程

### 创建入口

节点创建由 `NodeExecutor.execute_node_task()` 触发：

```python
# tradingflow/station/core/node_executor.py

async def execute_node_task(
    node_task_id: str,
    flow_id: str,
    component_id: int,
    cycle: int,
    node_id: str,
    node_type: str,
    node_data: Dict[str, Any]
):
    """执行节点任务的主入口"""
    try:
        # 1. 更新状态为 PENDING
        await _update_node_status(node_task_id, NodeStatus.PENDING, "Task created")
        
        # 2. 创建节点实例
        node_instance = await _create_node_instance(
            node_type, node_data, flow_id, component_id, cycle, node_id
        )
        
        # 3. 启动节点执行
        success = await node_instance.start()
        
        # 4. 处理执行结果
        if success:
            await _update_node_status(node_task_id, NodeStatus.COMPLETED, "Completed")
        else:
            await _update_node_status(node_task_id, NodeStatus.FAILED, "Failed")
            
    except Exception as e:
        # 异常处理
        await _update_node_status(node_task_id, NodeStatus.FAILED, str(e))
    finally:
        # 清理资源
        if node_instance:
            await node_instance.cleanup()
```

### 节点实例化

```python
async def _create_node_instance(
    node_type: str,
    node_data: Dict[str, Any],
    flow_id: str,
    component_id: int,
    cycle: int,
    node_id: str
):
    """创建节点实例"""
    # 1. 确定节点类型
    if node_type == "python":
        node_class_type = node_data.get("config", {}).get("node_class_type")
    else:
        node_class_type = node_type
    
    # 2. 解析输入输出边
    input_edges = [Edge.from_dict(edge) for edge in node_data.get("input_edges") or []]
    output_edges = [Edge.from_dict(edge) for edge in node_data.get("output_edges") or []]
    
    # 3. 准备配置
    config = node_data.get("config", {})
    config["state_store"] = node_manager.state_store  # 共享状态存储
    
    # 4. 使用 NodeRegistry 创建实例
    node_instance = node_registry.create_node(
        node_class_type=node_class_type,
        flow_id=flow_id,
        component_id=component_id,
        cycle=cycle,
        node_id=node_id,
        input_edges=input_edges,
        output_edges=output_edges,
        config=config
    )
    
    return node_instance
```

### NodeBase 构造函数

```python
# tradingflow/station/nodes/node_base.py

class NodeBase(abc.ABC):
    def __init__(
        self,
        flow_id: str,
        component_id: int,
        cycle: int,
        node_id: str,
        name: str,
        input_edges: List[Edge] = None,
        output_edges: List[Edge] = None,
        state_store: "StateStore" = None,
        **kwargs
    ):
        """初始化节点"""
        # 1. 基本属性
        self.logger = logging.getLogger(f"Node.{node_id}")
        self.flow_id = flow_id
        self.component_id = component_id
        self.cycle = cycle
        self.node_id = node_id
        self.name = name
        
        # 2. 边和句柄
        self._input_edges = input_edges or []
        self._output_edges = output_edges or []
        self._input_signals = {}  # 存储接收到的信号
        self._input_handles: Dict[str, InputHandle] = {}
        
        # 3. 注册输入句柄（子类实现）
        self._register_input_handles()
        
        # 4. 状态管理
        self.status = NodeStatus.PENDING
        self.state_store = state_store
        
        # 5. 消息队列
        if self._input_edges:
            self.node_signal_consumer = NodeSignalConsumer(
                self.flow_id,
                self.component_id,
                self.cycle,
                self.node_id,
                self._input_edges,
                on_signal_handler=self._on_signal_received
            )
        else:
            self.node_signal_consumer = None
        
        self.node_signal_publisher = NodeSignalPublisher(
            self.flow_id,
            self.component_id,
            self.cycle,
            self.node_id,
            self._output_edges
        )
        
        # 6. 初始化输入信号状态
        for edge in self._input_edges:
            if edge.target_node == self.node_id:
                edge_key = self._get_edge_key(
                    edge.source_node,
                    edge.source_node_handle,
                    edge.target_node_handle
                )
                self._input_signals[edge_key] = None
        
        # 7. Signal ready future
        self._signal_ready_future = None
        
        # 8. 停止执行标记
        self._stop_execution_requested = False
```

---

## 信号等待机制

### 信号就绪检查

```python
def can_execute(self) -> bool:
    """检查是否所有必需的输入信号已就绪"""
    # 如果没有输入边，可以立即执行
    if not self._input_edges:
        return True
    
    # 检查所有输入信号是否已接收
    for edge_key, signal in self._input_signals.items():
        if signal is None:
            self.logger.debug(f"Signal not ready: {edge_key}")
            return False
    
    self.logger.debug("All required signals are ready")
    return True
```

### 等待信号流程

```python
async def start(self) -> bool:
    """启动节点执行"""
    try:
        # 1. 初始化状态存储
        if not await self.initialize_state_store():
            return False
        
        # 2. 初始化消息队列
        if not await self.initialize_message_queue():
            return False
        
        # 3. 如果没有输入边，直接执行
        if not self._input_edges:
            await self.set_status(NodeStatus.RUNNING)
            return await self.execute()
        
        # 4. 启动信号消费者
        consume_task = asyncio.create_task(self.node_signal_consumer.consume())
        
        # 5. 创建 signal_ready_future
        self._signal_ready_future = asyncio.Future()
        
        # 6. 等待所有信号就绪或超时
        try:
            await asyncio.wait_for(
                self._signal_ready_future,
                timeout=self.signal_timeout if hasattr(self, 'signal_timeout') else 300
            )
        except asyncio.TimeoutError:
            self.logger.error("Timeout waiting for input signals")
            await self.set_status(NodeStatus.FAILED, "Signal timeout")
            return False
        except asyncio.CancelledError:
            # 收到停止信号，正常退出
            self.logger.info("Execution cancelled by stop signal")
            return False
        
        # 7. 执行节点逻辑
        await self.set_status(NodeStatus.RUNNING)
        return await self.execute()
        
    except NodeStopExecutionException as e:
        # 停止执行异常
        await self.set_status(NodeStatus.TERMINATED, f"Stopped: {e.reason}")
        return False
    except Exception as e:
        self.logger.exception("Error in node execution")
        await self.set_status(NodeStatus.FAILED, str(e))
        return False
```

### 信号接收处理

```python
async def _on_signal_received(
    self,
    signal: Signal,
    handle: str = None,
    signal_context: Dict[str, Any] = None
) -> None:
    """处理接收到的信号"""
    try:
        # 1. 处理停止执行信号
        if handle == "STOP_EXECUTION" or signal.type == SignalType.STOP_EXECUTION:
            await self._handle_stop_execution_signal(signal)
            return
        
        # 2. 从 context 获取源信息
        source_node = signal_context.get("source_node")
        source_handle = signal_context.get("source_handle")
        
        # 3. 构建 edge_key
        edge_key = self._get_edge_key(source_node, source_handle, handle)
        
        # 4. 更新信号状态
        if edge_key in self._input_signals:
            self._input_signals[edge_key] = signal
            
            # 附加源信息到信号对象
            signal.source_handle = source_handle
            signal.source_node = source_node
            
            # 持久化日志
            await self.persist_log(
                message=f"Signal received at {handle} from {source_node}:{source_handle}",
                log_level="INFO",
                log_source="node",
                log_metadata={
                    'signal_data': {handle: {...}},
                    'target_handle': handle,
                    'source_node': source_node
                }
            )
        
        # 5. 执行默认的信号处理逻辑（自动更新成员变量）
        await self._handle_default_signal_processing(handle, signal)
        
        # 6. 调用自定义处理器（如果存在）
        handler_name = self._deduce_input_handler_name(handle)
        if hasattr(self, handler_name):
            await getattr(self, handler_name)(signal)
        
        # 7. 检查是否所有信号就绪
        if self._signal_ready_future and not self._signal_ready_future.done():
            if self.can_execute():
                self._signal_ready_future.set_result(True)
                
    except Exception as e:
        self.logger.error("Error processing signal: %s", str(e))
        raise
```

---

## 节点执行流程

### 执行方法签名

```python
@abc.abstractmethod
async def execute(self) -> bool:
    """
    执行节点的核心逻辑（子类必须实现）
    
    Returns:
        bool: 执行是否成功
    """
    pass
```

### 典型执行模式

#### 模式 1：数据处理节点

```python
class BinancePriceNode(NodeBase):
    async def execute(self) -> bool:
        """获取币安价格数据"""
        try:
            # 1. 准备参数
            symbol = self.symbol
            interval = self.interval
            limit = self.limit
            
            # 2. 调用外部 API
            response = await self._fetch_klines(symbol, interval, limit)
            
            # 3. 处理数据
            kline_data = self._process_kline_data(response)
            
            # 4. 发送输出信号
            await self.send_signal(
                source_handle="kline_data",
                signal_type=SignalType.PRICE_DATA,
                payload=kline_data
            )
            
            # 5. 更新状态为完成
            await self.set_status(NodeStatus.COMPLETED)
            return True
            
        except Exception as e:
            self.logger.error(f"Error fetching price: {e}")
            await self.set_status(NodeStatus.FAILED, str(e))
            return False
```

#### 模式 2：条件判断节点

```python
class ConditionNode(NodeBase):
    async def execute(self) -> bool:
        """条件判断节点"""
        try:
            # 1. 获取输入数据
            input_data = await self.get_input_signal("condition_input")
            
            # 2. 执行判断逻辑
            condition_met = self._evaluate_condition(input_data.payload)
            
            if condition_met:
                # 3a. 条件满足，发送继续信号
                await self.send_signal(
                    source_handle="true_output",
                    signal_type=SignalType.DATA_READY,
                    payload={"result": "continue"}
                )
            else:
                # 3b. 条件不满足，发送停止信号
                await self.node_signal_publisher.send_stop_execution_signal(
                    reason="Condition not met"
                )
            
            await self.set_status(NodeStatus.COMPLETED)
            return True
            
        except Exception as e:
            await self.set_status(NodeStatus.FAILED, str(e))
            return False
```

#### 模式 3：交易执行节点

```python
class BuyNode(NodeBase):
    async def execute(self) -> bool:
        """执行买入交易"""
        try:
            # 1. 获取交易参数（从输入信号或配置）
            chain = self.chain
            vault_address = self.vault_address
            from_token = self.from_token
            to_token = self.to_token
            amount = self.amount_in_human_readable
            
            # 2. 选择对应链的服务
            if chain == "aptos":
                service = AptosVaultService.get_instance()
            elif chain == "flow_evm":
                service = FlowEvmVaultService.get_instance()
            else:
                raise ValueError(f"Unsupported chain: {chain}")
            
            # 3. 执行交易
            receipt = await service.execute_swap(
                vault_address=vault_address,
                from_token=from_token,
                to_token=to_token,
                amount_in=amount,
                slippage_tolerance=self.slippery_tolerance
            )
            
            # 4. 发送交易收据
            await self.send_signal(
                source_handle="trade_receipt",
                signal_type=SignalType.DEX_TRADE_RECEIPT,
                payload=receipt
            )
            
            await self.set_status(NodeStatus.COMPLETED)
            return True
            
        except Exception as e:
            self.logger.error(f"Trade execution failed: {e}")
            await self.set_status(NodeStatus.FAILED, str(e))
            
            # 可选：发送错误信号到错误处理句柄
            await self.send_signal(
                source_handle="error",
                signal_type=SignalType.ERROR,
                payload={"error": str(e)}
            )
            return False
```

---

## 信号发送机制

### 发送信号

```python
async def send_signal(
    self,
    source_handle: str,
    signal_type: SignalType,
    payload: Dict[str, Any] = None
) -> bool:
    """发送信号"""
    try:
        # 1. 创建信号对象
        signal = Signal(
            signal_type=signal_type,
            payload=payload,
            timestamp=None
        )
        
        # 2. 发送到消息队列
        await self.node_signal_publisher.send_signal(source_handle, signal)
        
        # 3. 持久化日志
        await self.persist_log(
            message=f"Signal sent from {source_handle}: {signal_type}",
            log_level="INFO",
            log_source="node",
            log_metadata={
                'signal_data': {
                    source_handle: {
                        'signal_type': signal_type.value,
                        'payload': payload or {},
                        'timestamp': signal.timestamp
                    }
                }
            }
        )
        
        return True
        
    except Exception as e:
        self.logger.error(f"Failed to send signal: {e}")
        return False
```

### 获取输入信号

```python
async def get_input_signal(self, handle_name: str) -> Optional[Signal]:
    """获取指定 handle 的输入信号"""
    # 遍历所有输入信号，找到匹配的 handle
    for edge_key, signal in self._input_signals.items():
        # edge_key 格式: "source_node:source_handle->target_handle"
        if edge_key.endswith(f"->{handle_name}"):
            return signal
    return None
```

---

## 异常处理

### 异常类型

```python
# tradingflow/depot/python/exceptions/tf_exception.py

class NodeExecutionException(Exception):
    """节点执行异常基类"""
    pass

class NodeStopExecutionException(NodeExecutionException):
    """节点停止执行异常"""
    def __init__(self, reason: str, source_node: str):
        self.reason = reason
        self.source_node = source_node

class NodeTimeoutException(NodeExecutionException):
    """节点超时异常"""
    def __init__(self, message: str, timeout_seconds: int):
        self.message = message
        self.timeout_seconds = timeout_seconds

class NodeValidationException(NodeExecutionException):
    """节点验证异常"""
    def __init__(self, message: str, node_id: str, invalid_params: Dict = None):
        self.message = message
        self.node_id = node_id
        self.invalid_params = invalid_params or {}

class NodeResourceException(NodeExecutionException):
    """节点资源异常"""
    def __init__(self, message: str, resource_type: str):
        self.message = message
        self.resource_type = resource_type
```

### 异常处理流程

```python
# NodeExecutor.execute_node_task()

try:
    # 执行节点
    success = await node_instance.start()
    
except NodeStopExecutionException as e:
    # 停止执行（正常中止）
    await _update_node_status(
        node_task_id,
        NodeStatus.TERMINATED,
        f"Stopped by signal: {e.reason}",
        {"stop_reason": e.reason, "source_node": e.source_node}
    )
    
except NodeTimeoutException as e:
    # 超时
    await _update_node_status(
        node_task_id,
        NodeStatus.FAILED,
        f"Timeout: {e.message}",
        {"timeout_seconds": e.timeout_seconds}
    )
    
except NodeValidationException as e:
    # 验证失败
    await _update_node_status(
        node_task_id,
        NodeStatus.FAILED,
        f"Validation failed: {e.message}",
        {"invalid_params": e.invalid_params}
    )
    
except NodeResourceException as e:
    # 资源错误
    await _update_node_status(
        node_task_id,
        NodeStatus.FAILED,
        f"Resource error: {e.message}",
        {"resource_type": e.resource_type}
    )
    
except Exception as e:
    # 未知异常
    await _update_node_status(
        node_task_id,
        NodeStatus.FAILED,
        f"Unexpected error: {str(e)}"
    )
```

---

## 资源清理

### 清理流程

```python
async def cleanup(self):
    """清理节点资源"""
    try:
        # 1. 关闭消息队列连接
        if self.node_signal_consumer:
            await self.node_signal_consumer.close()
        
        if self.node_signal_publisher:
            await self.node_signal_publisher.close()
        
        # 2. 关闭状态存储连接（仅当节点创建时关闭）
        await self.close_state_store()
        
        # 3. 执行子类自定义清理（如果存在）
        if hasattr(self, '_cleanup'):
            await self._cleanup()
        
        self.logger.info("Node cleanup completed")
        
    except Exception as e:
        self.logger.error(f"Error during cleanup: {e}")
```

### 强制终止

```python
# 通过 Redis 终止标志实现
async def check_termination_flag(self):
    """检查终止标志"""
    terminate_flag = await self.state_store.get_termination_flag(self.node_task_id)
    if terminate_flag:
        raise NodeStopExecutionException(
            reason=terminate_flag["reason"],
            source_node="system"
        )
```

---

## 开发新节点

### 最小节点模板

```python
from tradingflow.station.nodes.node_base import NodeBase, register_node_type
from tradingflow.station.common.signal_types import SignalType

@register_node_type("my_custom_node", default_params={
    "param1": "default_value",
    "param2": 100
})
class MyCustomNode(NodeBase):
    """自定义节点"""
    
    def __init__(self, **kwargs):
        """初始化节点"""
        super().__init__(**kwargs)
        
        # 获取配置参数
        self.param1 = kwargs.get("param1", "default_value")
        self.param2 = kwargs.get("param2", 100)
    
    def _register_input_handles(self):
        """注册输入句柄"""
        from dataclasses import dataclass
        from tradingflow.station.nodes.node_base import InputHandle
        
        self._input_handles = {
            "input_data": InputHandle(
                name="input_data",
                data_type=dict,
                description="输入数据",
                auto_update_attr="input_data"
            )
        }
    
    async def execute(self) -> bool:
        """执行节点逻辑"""
        try:
            # 1. 获取输入数据
            input_signal = await self.get_input_signal("input_data")
            input_data = input_signal.payload if input_signal else {}
            
            # 2. 执行业务逻辑
            result = self._process_data(input_data)
            
            # 3. 发送输出信号
            await self.send_signal(
                source_handle="output_data",
                signal_type=SignalType.DATA_READY,
                payload=result
            )
            
            # 4. 更新状态
            await self.set_status(NodeStatus.COMPLETED)
            return True
            
        except Exception as e:
            self.logger.error(f"Execution failed: {e}")
            await self.set_status(NodeStatus.FAILED, str(e))
            return False
    
    def _process_data(self, data: dict) -> dict:
        """处理数据（业务逻辑）"""
        # 实现具体的处理逻辑
        return {"processed": data}
```

### 开发清单

✅ **必须实现：**
1. 继承 `NodeBase`
2. 使用 `@register_node_type` 装饰器注册
3. 实现 `execute()` 方法
4. 实现 `_register_input_handles()` 方法（如有输入）

✅ **建议实现：**
5. 在 `__init__` 中处理配置参数
6. 使用 `auto_update_attr` 自动更新成员变量
7. 完善错误处理和日志记录
8. 发送有意义的输出信号

✅ **可选实现：**
9. 自定义信号处理器 `_on_{handle_name}_received()`
10. 自定义清理逻辑 `_cleanup()`
11. 超时控制 `signal_timeout`
12. 聚合 Handle 支持

---

## 下一步

继续阅读相关文档：

- **[Flow 调度机制](weather-station-flow-scheduling.md)** - 周期调度和 DAG 编排
- **[开发指南](weather-station-development-guide.md)** - 完整开发流程和最佳实践

---

**维护者：** TradingFlow 开发团队  
**版本历史：** 
- v1.0.0 (2025-10-06): 初始版本

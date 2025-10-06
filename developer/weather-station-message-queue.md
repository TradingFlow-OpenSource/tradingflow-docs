# Weather Station - 消息队列详解

**文档版本：** 1.0.0  
**最后更新：** 2025-10-06  
**前置阅读：** [架构概述](weather-station-overview.md)

---

## 目录

- [消息队列架构](#消息队列架构)
- [Routing Key 设计](#routing-key-设计)
- [Queue 命名规范](#queue-命名规范)
- [信号传递流程](#信号传递流程)
- [Handle 匹配机制](#handle-匹配机制)
- [特殊场景处理](#特殊场景处理)
- [实战示例](#实战示例)

---

## 消息队列架构

### 整体设计

Weather Station 使用 **RabbitMQ Topic Exchange** 实现节点间的信号传递。

```
┌─────────────────────────────────────────────────────────┐
│            signals_exchange (Topic Exchange)            │
└──────────┬──────────────────────────────────────────────┘
           │
           │ Routing Rules (Based on Routing Key)
           │
    ┌──────┴─────────┬─────────────┬────────────┐
    │                │             │            │
    ↓                ↓             ↓            ↓
┌─────────┐    ┌─────────┐   ┌─────────┐  ┌─────────┐
│ Queue A │    │ Queue B │   │ Queue C │  │ Queue D │
└─────────┘    └─────────┘   └─────────┘  └─────────┘
    ↓                ↓             ↓            ↓
 Node A          Node B        Node C       Node D
```

### RabbitMQ 配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **Exchange Name** | `signals_exchange` | 全局信号交换机 |
| **Exchange Type** | `topic` | 主题交换机，支持通配符路由 |
| **Queue Durability** | `True` | 队列持久化 |
| **Message Durability** | `True` | 消息持久化 |
| **Auto Delete** | `False` | 队列不自动删除 |
| **Exclusive** | `False` | 非独占队列 |

### 设计优势

1. **解耦合**：上游节点无需知道下游节点的具体信息
2. **灵活路由**：基于 Routing Key 的精确匹配和模式匹配
3. **并发支持**：多个下游节点可以并行接收同一信号
4. **周期隔离**：不同 Cycle 的消息完全隔离
5. **可扩展性**：支持动态添加节点和消费者

---

## Routing Key 设计

### 标准 Routing Key 格式

```
flow.{flow_id}.component.{component_id}.cycle.{cycle}.from.{source_node}.handle.{output_handle}.to.{target_node}.handle.{input_handle}
```

### 字段说明

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| **flow_id** | string | Flow 唯一标识 | `trading_decision_flow` |
| **component_id** | int | 连通分量 ID | `0`, `1`, `2` |
| **cycle** | int | 执行周期号 | `0`, `1`, `2` |
| **source_node** | string | 源节点 ID | `binance_price` |
| **output_handle** | string | 源句柄名称 | `kline_data` |
| **target_node** | string | 目标节点 ID | `ai_model` |
| **input_handle** | string | 目标句柄名称 | `price_input` |

### Routing Key 示例

#### 1. 精确路由

```
flow.my_flow.component.0.cycle.0.from.node_A.handle.data.to.node_B.handle.input
```

**匹配规则：**
- Flow ID: `my_flow`
- Component: `0`
- Cycle: `0`
- Source: `node_A`, Handle: `data`
- Target: `node_B`, Handle: `input`

#### 2. 通配符路由（发送到多个 Handle）

```
flow.my_flow.component.0.cycle.0.from.node_A.handle.data.to.node_B.handle.*
```

**场景：** 一个输出连接到目标节点的多个输入 Handle

#### 3. 停止执行信号

```
flow.my_flow.signal.STOP_EXECUTION.cycle.0
```

**场景：** 广播停止执行信号到当前 Cycle 的所有节点

### 订阅模式（Binding Key）

节点订阅消息时使用通配符：

```python
# 订阅所有发送给自己的消息
binding_key = f"flow.{flow_id}.component.{component_id}.cycle.{cycle}.from.*.handle.*.to.{node_id}.handle.*"
```

**通配符说明：**
- `*` : 匹配一个词（word）
- `#` : 匹配零个或多个词

---

## Queue 命名规范

### 标准 Queue 命名

```
queue.flow.{flow_id}.cycle.{cycle}.node.{node_id}
```

### 设计原则

1. **唯一性**：每个节点在每个 Cycle 都有唯一队列
2. **隔离性**：不同 Cycle 的消息不会混淆
3. **可追溯性**：队列名称清晰标识所属 Flow 和节点

### Queue 示例

```python
# 节点 A 在 Cycle 0 的队列
queue_name = "queue.flow.my_flow.cycle.0.node.node_A"

# 节点 B 在 Cycle 1 的队列
queue_name = "queue.flow.my_flow.cycle.1.node.node_B"
```

### Queue 特性

- **持久化**：队列持久化到磁盘
- **手动确认**：消息处理完成后手动 ACK
- **预取数量**：`prefetch_count=1`，公平分发
- **TTL**：消息 TTL 根据 Flow 配置设置

---

## 信号传递流程

### 完整流程图

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 节点 A 执行完成，准备发送信号                            │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. NodeSignalPublisher.send_signal()                         │
│    - 获取输出边映射 (output_edges_map)                      │
│    - 确定目标节点和句柄                                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. 生成 Routing Key                                          │
│    - 精确发送：指定具体 input_handle                        │
│    - 通配符发送：使用 * 匹配多个 handles                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. 发布消息到 RabbitMQ                                       │
│    - Exchange: signals_exchange                              │
│    - Routing Key: flow.xxx.from.A.handle.data.to.B.handle.* │
│    - Body: Signal.to_json()                                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. RabbitMQ 路由消息                                         │
│    - 根据 Routing Key 匹配订阅模式                          │
│    - 将消息路由到匹配的 Queue                                │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. 节点 B 的 NodeSignalConsumer.consume()                   │
│    - 从队列接收消息                                          │
│    - 解析 Routing Key                                        │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 7. 验证和过滤                                                │
│    - 验证 Cycle 是否匹配                                     │
│    - 验证目标节点是否匹配                                    │
│    - 验证目标 Handle 是否匹配                                │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 8. 调用信号处理器                                            │
│    - node._on_signal_received(signal, handle, context)      │
│    - 更新输入信号状态                                        │
│    - 检查是否所有信号就绪                                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────────────────┐
│ 9. 触发节点执行                                              │
│    - 所有必需信号就绪后触发 execute()                       │
└─────────────────────────────────────────────────────────────┘
```

### 关键代码实现

#### 发送信号（Publisher）

```python
# tradingflow/depot/python/mq/node_signal_publisher.py

async def send_signal(self, source_handle: str, signal: Signal) -> None:
    """发送信号"""
    # 1. 获取目标映射
    targets = self.output_edges_map.get(source_handle, {})
    
    if not targets:
        logger.warning("No targets found for source handle: %s", source_handle)
        return
    
    # 2. 遍历每个目标节点
    for target_node, target_handles in targets.items():
        if len(target_handles) == 1:
            # 精确发送
            target_handle = target_handles[0]
            routing_key = self._generate_send_signal_routing_key(
                source_handle, target_node, target_handle
            )
        else:
            # 通配符发送
            routing_key = self._generate_send_signal_routing_key(
                source_handle, target_node, "*"
            )
        
        # 3. 发布消息
        await self.publisher.publish(signal.to_json(), routing_key=routing_key)
        logger.debug("Signal sent: %s", routing_key)
```

#### 接收信号（Consumer）

```python
# tradingflow/depot/python/mq/node_signal_consumer.py

async def consume(self):
    """消费消息"""
    async def wrapper_callback(message: AbstractIncomingMessage) -> None:
        async with message.process():
            # 1. 解码消息
            body = message.body.decode()
            signal = Signal.from_json(body)
            routing_key = message.routing_key
            
            # 2. 解析 Routing Key
            signal_context = self._parse_routing_key(routing_key)
            
            # 3. 验证 Cycle
            message_cycle = signal_context.get("cycle")
            if message_cycle != self.cycle:
                logger.warning("Rejecting message from different cycle")
                return
            
            # 4. 验证目标节点
            target_node_id = signal_context.get("target_node")
            if target_node_id != self.node_id:
                logger.warning("Message for different node")
                return
            
            # 5. 获取目标 Handle
            target_handle = signal_context.get("target_handle")
            
            # 6. 处理信号
            await self.on_signal(signal, handle=target_handle, signal_context=signal_context)
    
    # 订阅队列
    await self.consumer.consume(
        queue_name=self.queue_name,
        binding_keys=[self.upstream_routing_key, self.stop_execution_routing_key],
        callback=wrapper_callback
    )
```

---

## Handle 匹配机制

### 输入边映射（Input Edges Map）

每个节点维护输入信号的状态映射：

```python
# 格式：{edge_key: signal}
_input_signals = {
    "node_A:data->input1": Signal(...),
    "node_B:result->input2": Signal(...),
}
```

**Edge Key 格式：**
```
{source_node}:{source_handle}->{target_handle}
```

### 信号匹配过程

```python
async def _on_signal_received(self, signal: Signal, handle: str, signal_context: Dict):
    """处理接收到的信号"""
    # 1. 从 context 获取源信息
    source_node = signal_context.get("source_node")
    source_handle = signal_context.get("source_handle")
    
    # 2. 构建 edge_key
    edge_key = self._get_edge_key(source_node, source_handle, handle)
    
    # 3. 更新信号状态
    if edge_key in self._input_signals:
        self._input_signals[edge_key] = signal
        logger.debug("Updated signal for edge: %s", edge_key)
    
    # 4. 执行自动更新（如果配置了 auto_update_attr）
    await self._handle_default_signal_processing(handle, signal)
    
    # 5. 调用自定义处理器（如果存在）
    handler_name = self._deduce_input_handler_name(handle)
    if hasattr(self, handler_name):
        await getattr(self, handler_name)(signal)
    
    # 6. 检查是否所有信号就绪
    if self.can_execute():
        # 触发执行
        self._signal_ready_future.set_result(True)
```

### 聚合 Handle（Aggregate Handle）

**场景：** 多个输出连接到同一个输入 Handle

```python
# 定义聚合 Handle
@dataclass
class InputHandle:
    name: str = "aggregated_data"
    data_type: type = dict
    is_aggregate: bool = True
    auto_update_attr: str = "aggregated_data"
```

**聚合逻辑：**
```python
if handle_obj.is_aggregate and handle_obj.data_type == dict:
    # 获取源句柄名称
    signal_source_handle = getattr(signal, 'source_handle', None)
    
    # 获取当前聚合状态
    current_value = getattr(self, handle_obj.auto_update_attr)
    if not isinstance(current_value, dict):
        current_value = {}
    
    # 创建新的聚合字典
    new_aggregated_value = current_value.copy()
    
    # 使用源句柄名称作为 key
    new_aggregated_value[signal_source_handle] = final_value
    
    # 更新聚合状态
    setattr(self, handle_obj.auto_update_attr, new_aggregated_value)
```

**结果格式：**
```python
{
    "source_handle1": "value1",
    "source_handle2": {
        "complex": "data"
    }
}
```

---

## 特殊场景处理

### 1. 一对多连接

**场景：** 一个输出连接到多个下游节点

```
Node A (output: data)
    ├─> Node B (input: data_input)
    ├─> Node C (input: price_data)
    └─> Node D (input: analysis_input)
```

**实现：**
```python
# Publisher 自动为每个目标节点生成独立的 Routing Key
for target_node in targets:
    routing_key = f"flow.xxx.from.A.handle.data.to.{target_node}.handle.{target_handle}"
    await self.publisher.publish(signal.to_json(), routing_key)
```

### 2. 多对一连接

**场景：** 多个输出连接到同一个输入 Handle

```
Node A (output: data1) ──┐
                          ├─> Node D (input: combined_data)
Node B (output: data2) ──┤
                          │
Node C (output: data3) ──┘
```

**实现：** 使用聚合 Handle（见上文）

### 3. 通配符 Handle

**场景：** 接收任意上游信号

```python
# Routing Key 中的 target_handle 为 *
routing_key = "flow.xxx.from.A.handle.data.to.B.handle.*"
```

**处理逻辑：**
```python
if target_handle == "*":
    # 拆包 payload，根据 key 匹配到对应的 input handle
    payload = signal.payload
    if payload and isinstance(payload, dict):
        for handle_name, handle_obj in input_handles.items():
            if handle_name in payload:
                # 更新对应的 handle
                handle_signal = Signal(
                    signal_type=signal.type,
                    payload=payload[handle_name],
                    timestamp=signal.timestamp
                )
                # 处理信号
                await self._handle_default_signal_processing(handle_name, payload[handle_name])
```

### 4. 停止执行信号

**Routing Key：**
```
flow.{flow_id}.signal.STOP_EXECUTION.cycle.{cycle}
```

**特点：**
- 广播到当前 Cycle 的所有节点
- 不包含具体的目标节点和 Handle 信息
- 优先级高于普通信号

**处理逻辑：**
```python
if signal_context.get("signal_type") == "STOP_EXECUTION":
    await self.on_signal(signal, handle="STOP_EXECUTION", signal_context=signal_context)
    return

# 节点侧处理
async def _handle_stop_execution_signal(self, signal: Signal):
    reason = signal.payload.get("reason", "Unknown reason")
    source_node = signal.payload.get("source_node", "Unknown source")
    
    # 设置停止标记
    self._stop_execution_requested = True
    self._stop_execution_reason = reason
    
    # 更新状态为 TERMINATED
    await self.set_status(NodeStatus.TERMINATED, f"Stopped by {source_node}: {reason}")
    
    # 取消等待中的 future
    if self._signal_ready_future and not self._signal_ready_future.done():
        self._signal_ready_future.cancel()
```

---

## 实战示例

### 示例 1：简单的价格数据流

**工作流：**
```
BinancePriceNode (A) → AIModelNode (B) → BuyNode (C)
```

**消息流：**

1. **Node A → Node B**
```json
{
  "routing_key": "flow.trading_flow.component.0.cycle.0.from.binance_price.handle.kline_data.to.ai_model.handle.price_input",
  "body": {
    "signal_type": "PRICE_DATA",
    "payload": {
      "symbol": "BTCUSDT",
      "close_price": 50000,
      "volume": 1000000
    },
    "timestamp": "2025-10-06T10:00:00Z"
  }
}
```

2. **Node B → Node C**
```json
{
  "routing_key": "flow.trading_flow.component.0.cycle.0.from.ai_model.handle.ai_response.to.buy_node.handle.decision_input",
  "body": {
    "signal_type": "AI_RESPONSE",
    "payload": {
      "action": "buy",
      "confidence": 0.85,
      "amount": 0.1
    },
    "timestamp": "2025-10-06T10:00:05Z"
  }
}
```

### 示例 2：聚合多源数据

**工作流：**
```
PriceNode (A) ──┐
                 ├─> AnalysisNode (D)
NewsNode (B) ───┤
                 │
SocialNode (C) ─┘
```

**Node D 的 Handle 定义：**
```python
@dataclass
class InputHandle:
    name: str = "market_data"
    data_type: type = dict
    is_aggregate: bool = True
    auto_update_attr: str = "market_data"
```

**消息流：**

1. **Node A → Node D**
```json
{
  "routing_key": "flow.xxx.from.price_node.handle.price_data.to.analysis.handle.market_data",
  "body": {
    "signal_type": "PRICE_DATA",
    "payload": {
      "price": 50000,
      "change": 2.5
    }
  }
}
```

2. **Node B → Node D**
```json
{
  "routing_key": "flow.xxx.from.news_node.handle.news_data.to.analysis.handle.market_data",
  "body": {
    "signal_type": "DATASET",
    "payload": {
      "sentiment": "positive",
      "score": 0.8
    }
  }
}
```

3. **Node C → Node D**
```json
{
  "routing_key": "flow.xxx.from.social_node.handle.social_data.to.analysis.handle.market_data",
  "body": {
    "signal_type": "DATASET",
    "payload": {
      "mentions": 1000,
      "sentiment": 0.7
    }
  }
}
```

**Node D 接收到的聚合数据：**
```python
market_data = {
    "price_data": {
        "price": 50000,
        "change": 2.5
    },
    "news_data": {
        "sentiment": "positive",
        "score": 0.8
    },
    "social_data": {
        "mentions": 1000,
        "sentiment": 0.7
    }
}
```

### 示例 3：停止执行信号

**场景：** Condition Node 判断不满足条件，停止当前 Cycle 执行

**代码：**
```python
# ConditionNode 发送停止信号
await self.node_signal_publisher.send_stop_execution_signal(
    reason="Condition not met: price < threshold"
)
```

**Routing Key：**
```
flow.trading_flow.signal.STOP_EXECUTION.cycle.0
```

**所有节点接收：**
```python
# 每个节点的 Consumer 都订阅了停止信号
async def _handle_stop_execution_signal(self, signal: Signal):
    reason = signal.payload.get("reason")
    logger.warning("Received STOP_EXECUTION: %s", reason)
    
    # 设置停止标记
    self._stop_execution_requested = True
    
    # 更新状态
    await self.set_status(NodeStatus.TERMINATED, f"Stopped: {reason}")
```

---

## 下一步

继续阅读相关文档：

- **[Redis 状态管理](weather-station-redis.md)** - 状态存储结构
- **[节点执行流程](weather-station-node-execution.md)** - 节点生命周期
- **[Flow 调度机制](weather-station-flow-scheduling.md)** - 周期调度

---

**维护者：** TradingFlow 开发团队  
**版本历史：** 
- v1.0.0 (2025-10-06): 初始版本

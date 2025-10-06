# Weather Station Node Execution Flow

**Version:** 1.0.0  
**Document Type:** Technical Deep Dive  
**Last Updated:** 2025-10-06

---

## 📖 About This Document

This document provides detailed information about node lifecycle, execution process, and development guide in Weather Station.

**Target Audience:**
- Strategy Developers
- Node Developers
- Backend Engineers

**Prerequisites:**
- Understanding of [Architecture Overview](weather-station-overview.md)
- Basic Python programming knowledge
- Familiarity with async/await patterns

**Reading Time:** ~70 minutes

---

## 🔄 Node Lifecycle

### Complete Lifecycle Stages

```
1. INITIALIZATION
   └─> Node instance created
   └─> Parameters validated
   └─> Input/output handles registered

2. PENDING
   └─> Node registered in scheduler
   └─> Waiting for signal consumption setup
   └─> Ready to start execution

3. RUNNING
   └─> Signal consumer active
   └─> Waiting for input signals
   └─> Processing data
   └─> Sending output signals

4. COMPLETED
   └─> All outputs sent successfully
   └─> State persisted to Redis
   └─> Resources cleaned up

5. FAILED (if error occurs)
   └─> Error captured and logged
   └─> State saved with error details
   └─> Dead letter handling

6. TERMINATED (if manually stopped)
   └─> Execution interrupted
   └─> Partial results saved
   └─> Resources released

7. CLEANUP
   └─> Message queues closed
   └─> Database connections released
   └─> Memory freed
```

---

## ⚙️ Node Execution Process

### 1. Node Creation and Initialization

```python
class NodeBase:
    def __init__(
        self,
        node_id: str,
        flow_id: str,
        cycle: int,
        parameters: Dict[str, Any],
        input_edges: List[Edge],
        output_edges: List[Edge],
        **kwargs
    ):
        # Basic properties
        self.node_id = node_id
        self.flow_id = flow_id
        self.cycle = cycle
        
        # Parameters
        self.parameters = parameters
        
        # Edges
        self.input_edges = input_edges
        self.output_edges = output_edges
        
        # State management
        self.state_store = StateStoreFactory.create()
        self.status = NodeStatus.PENDING
        
        # Signal buffers
        self.input_signals: Dict[str, List[Signal]] = {}
        
        # Register input handles
        self._register_input_handles()
```

### 2. Signal Consumption Setup

```python
async def start(self):
    """Main entry point for node execution"""
    try:
        # Update status to RUNNING
        await self._update_status(NodeStatus.RUNNING)
        
        # Setup message queue consumer
        await self._setup_signal_consumer()
        
        # Start consuming signals
        await self._consume_signals()
        
    except Exception as e:
        await self._handle_error(e)
```

### 3. Signal Reception and Aggregation

**Non-Aggregate Input:**
```python
# Execute immediately on each signal
async def _handle_signal(self, signal: Signal):
    if not self.input_handles[signal.handle].is_aggregate:
        # Store signal
        self.input_signals[signal.handle] = [signal]
        
        # Execute node immediately
        await self.execute()
```

**Aggregate Input:**
```python
# Wait for all expected signals
async def _handle_signal(self, signal: Signal):
    if self.input_handles[signal.handle].is_aggregate:
        # Store signal
        if signal.handle not in self.input_signals:
            self.input_signals[signal.handle] = []
        self.input_signals[signal.handle].append(signal)
        
        # Check if all inputs received
        if self._all_inputs_received():
            await self.execute()
```

### 4. Node Execution

```python
@abstractmethod
async def execute(self) -> Dict[str, Any]:
    """
    Override this method in subclass
    Returns: Dictionary of output handle -> data
    """
    pass

# Example implementation
async def execute(self) -> Dict[str, Any]:
    # Get input data
    input_data = self._get_input_data()
    
    # Process data (your custom logic)
    result = await self._process_data(input_data)
    
    # Return outputs
    return {
        "output_data": result
    }
```

### 5. Signal Sending

```python
async def send_signal(
    self,
    output_handle: str,
    signal_type: str,
    data: Any
):
    """Send signal to downstream nodes"""
    
    # Find edges for this output
    edges = [e for e in self.output_edges 
             if e.source_handle == output_handle]
    
    for edge in edges:
        # Construct routing key
        routing_key = (
            f"{self.flow_id}."
            f"{self.cycle}."
            f"{self.node_id}."
            f"{output_handle}."
            f"{signal_type}"
        )
        
        # Publish to message queue
        await self.mq_client.publish(
            exchange="station.signals",
            routing_key=routing_key,
            message={
                "metadata": {
                    "flow_id": self.flow_id,
                    "cycle": self.cycle,
                    "source_node_id": self.node_id,
                    "source_handle": output_handle,
                    "edge_id": edge.edge_id
                },
                "signal_type": signal_type,
                "data": data
            }
        )
```

---

## 🎯 Node Types and Patterns

### 1. Data Input Nodes

**Purpose:** Fetch data from external sources

**Example: Binance Price Node**
```python
class BinancePriceNode(NodeBase):
    async def execute(self) -> Dict[str, Any]:
        # Fetch price data
        price_data = await self.binance_client.get_klines(
            symbol=self.parameters['symbol'],
            interval=self.parameters['interval']
        )
        
        # Return as signal
        return {
            "price_data": {
                "symbol": self.parameters['symbol'],
                "prices": price_data,
                "timestamp": time.time()
            }
        }
```

### 2. Compute Nodes

**Purpose:** Process and analyze data

**Example: AI Model Node**
```python
class AIModelNode(NodeBase):
    async def execute(self) -> Dict[str, Any]:
        # Get input context
        context = self._prepare_context()
        
        # Call LLM
        response = await self.llm_client.complete(
            model=self.parameters['model'],
            prompt=self.parameters['prompt'],
            context=context
        )
        
        # Return AI response
        return {
            "ai_response": {
                "text": response.text,
                "structured_data": self._extract_json(response.text)
            }
        }
```

### 3. Trade Execution Nodes

**Purpose:** Execute blockchain transactions

**Example: Swap Node**
```python
class SwapNode(NodeBase):
    async def execute(self) -> Dict[str, Any]:
        # Select vault service based on chain
        if self.parameters['chain'] == 'aptos':
            service = AptosVaultService()
        else:
            service = FlowEvmVaultService()
        
        # Execute swap
        receipt = await service.execute_swap(
            vault_address=self.parameters['vault_address'],
            from_token=self.parameters['from_token'],
            to_token=self.parameters['to_token'],
            amount=self.parameters['amount']
        )
        
        return {
            "trade_receipt": receipt
        }
```

### 4. Output Nodes

**Purpose:** Store or send results

**Example: Dataset Output Node**
```python
class DatasetOutputNode(NodeBase):
    async def execute(self) -> Dict[str, Any]:
        # Get input data
        data = self._get_input_data()['data']
        
        # Write to Google Sheets
        await self.sheets_client.write_data(
            doc_link=self.parameters['doc_link'],
            data=data,
            mode='write'  # overwrite existing data
        )
        
        return {
            "status": "success",
            "rows_written": len(data)
        }
```

---

## 🛡️ Error Handling

### Exception Types

```python
class NodeExecutionError(Exception):
    """Base exception for node execution errors"""
    pass

class NodeTimeoutError(NodeExecutionError):
    """Node execution exceeded timeout"""
    pass

class NodeValidationError(NodeExecutionError):
    """Node configuration validation failed"""
    pass

class NodeStopExecutionException(Exception):
    """Signal to stop node execution gracefully"""
    pass
```

### Error Handling Pattern

```python
async def execute(self) -> Dict[str, Any]:
    try:
        # Your logic here
        result = await self._do_work()
        return {"output": result}
        
    except ValidationError as e:
        # Configuration error - don't retry
        self.logger.error(f"Validation error: {e}")
        await self._update_status(NodeStatus.FAILED, str(e))
        raise NodeValidationError(str(e))
        
    except TimeoutError as e:
        # Timeout - may retry
        self.logger.warning(f"Timeout: {e}")
        raise NodeTimeoutError(str(e))
        
    except Exception as e:
        # Unknown error - log and fail
        self.logger.exception(f"Unexpected error: {e}")
        await self._update_status(NodeStatus.FAILED, str(e))
        raise
```

---

## 🔧 Developing New Nodes

### Step 1: Create Node Class

```python
from tradingflow.station.nodes.node_base import NodeBase, register_node_type

@register_node_type("my_custom_node")
class MyCustomNode(NodeBase):
    """Description of your node"""
    
    # Define input handles
    INPUT_HANDLES = [
        InputHandle(
            name="my_input",
            data_type="string",
            description="Input description",
            required=True
        )
    ]
    
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        # Custom initialization
        
    async def execute(self) -> Dict[str, Any]:
        # Your logic here
        input_data = self._get_input_data()
        
        # Process
        result = await self._process(input_data)
        
        # Return outputs
        return {
            "my_output": result
        }
```

### Step 2: Register Node Metadata

```python
# Set class-level metadata
MyCustomNode.set_class_metadata(
    version="1.0.0",
    display_name="My Custom Node",
    node_category="compute",
    description="What this node does",
    author="Your Name",
    tags=["custom", "utility"]
)
```

### Step 3: Test Your Node

```python
import pytest
from mynode import MyCustomNode

@pytest.mark.asyncio
async def test_my_node():
    node = MyCustomNode(
        node_id="test_node",
        flow_id="test_flow",
        cycle=0,
        parameters={"param1": "value1"},
        input_edges=[],
        output_edges=[]
    )
    
    # Execute
    result = await node.execute()
    
    # Assert
    assert "my_output" in result
    assert result["my_output"] == expected_value
```

---

## 📊 Performance Optimization

### Async Operations

```python
# Good: Parallel execution
async def execute(self) -> Dict[str, Any]:
    results = await asyncio.gather(
        self.fetch_data_source_1(),
        self.fetch_data_source_2(),
        self.fetch_data_source_3()
    )
    return {"output": self.combine(results)}

# Bad: Sequential execution
async def execute(self) -> Dict[str, Any]:
    r1 = await self.fetch_data_source_1()
    r2 = await self.fetch_data_source_2()
    r3 = await self.fetch_data_source_3()
    return {"output": self.combine([r1, r2, r3])}
```

### Resource Management

```python
async def execute(self) -> Dict[str, Any]:
    # Use context managers
    async with self.get_connection() as conn:
        result = await conn.query(...)
        return {"output": result}
    # Connection auto-closed
```

---

## 📚 Related Documentation

- [Architecture Overview](weather-station-overview.md)
- [Message Queue Details](weather-station-message-queue.md)
- [Redis State Management](weather-station-redis.md)
- [Flow Scheduling](weather-station-flow-scheduling.md)
- [Node Reference](../node-details/index.md)

---

**Maintained by:** TradingFlow Engineering Team  
**Last Updated:** 2025-10-06  
**Version:** 1.0.0

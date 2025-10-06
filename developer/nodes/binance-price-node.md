# Binance Price Node

**节点类型：** `binance_price_node`  
**版本：** 1.0.0  
**分类：** 数据输入节点  
**文件位置：** `tradingflow/station/nodes/binance_price_node.py`

---

## 节点概述

Binance Price Node 是一个数据输入节点，用于从 Binance 交易所获取实时或历史的 K 线（蜡烛图）数据。这是构建交易策略的基础数据源节点。

### 主要功能

- 获取指定交易对的 K 线数据
- 支持多种时间周期（1分钟到1月）
- 自动重试机制确保数据获取稳定性
- 支持公共 API 和私有 API 访问

---

## 输入输出

### 输入参数

| 参数名 | 类型 | 默认值 | 必需 | 说明 |
|--------|------|--------|------|------|
| `symbol` | string | `"BTCUSDT"` | 是 | 交易对符号，自动转换为大写 |
| `interval` | string | `"1m"` | 是 | K线时间间隔 |
| `limit` | number | `100` | 否 | 获取的K线数量，范围 1-1000 |

#### interval 可选值

| 值 | 说明 | 值 | 说明 |
|----|------|----|------|
| `1m` | 1分钟 | `1h` | 1小时 |
| `3m` | 3分钟 | `2h` | 2小时 |
| `5m` | 5分钟 | `4h` | 4小时 |
| `15m` | 15分钟 | `6h` | 6小时 |
| `30m` | 30分钟 | `8h` | 8小时 |
| `1h` | 1小时 | `12h` | 12小时 |
| - | - | `1d` | 1天 |
| - | - | `3d` | 3天 |
| - | - | `1w` | 1周 |
| - | - | `1M` | 1月 |

### 输出句柄

**kline_data** (SignalType.PRICE_DATA)

输出 K 线数据列表，每个数据点包含：

```json
{
  "open_time": 1633024800000,
  "open": "50000.00",
  "high": "51000.00",
  "low": "49500.00",
  "close": "50500.00",
  "volume": "1000.50",
  "close_time": 1633024859999,
  "quote_asset_volume": "50250000.00",
  "number_of_trades": 5000,
  "taker_buy_base_asset_volume": "600.25",
  "taker_buy_quote_asset_volume": "30150000.00"
}
```

**字段说明：**
- `open_time`: 开盘时间（毫秒时间戳）
- `open`: 开盘价
- `high`: 最高价
- `low`: 最低价
- `close`: 收盘价
- `volume`: 成交量（基础资产）
- `close_time`: 收盘时间（毫秒时间戳）
- `quote_asset_volume`: 成交额（报价资产）
- `number_of_trades`: 成交笔数
- `taker_buy_base_asset_volume`: 主动买入成交量
- `taker_buy_quote_asset_volume`: 主动买入成交额

---

## 执行流程

### 1. 初始化阶段

```python
def __init__(self, symbol="BTCUSDT", interval="1h", limit=100, **kwargs):
    # 参数标准化
    self.symbol = symbol.upper()  # 转换为大写
    self.interval = interval
    self.limit = min(1000, max(1, limit))  # 限制在 1-1000
    
    # 获取 API 配置
    self.api_key = CONFIG["BINANCE_API_KEY"]
    self.api_secret = CONFIG["BINANCE_API_SECRET"]
```

### 2. 客户端初始化

```python
async def initialize_client(self) -> bool:
    """初始化 Binance API 客户端，带重试机制"""
    max_retries = 3
    retry_count = 0
    
    while retry_count <= max_retries:
        try:
            if self.api_key and self.api_secret:
                # 使用认证客户端
                self.client = Client(self.api_key, self.api_secret)
            else:
                # 使用公共客户端
                self.client = Client()
            
            # 测试连接
            await asyncio.to_thread(self.client.ping)
            return True
            
        except Exception as e:
            retry_count += 1
            if retry_count > max_retries:
                return False
            
            # 指数退避
            delay = base_delay * (2 ** (retry_count - 1))
            await asyncio.sleep(delay)
    
    return False
```

### 3. 数据获取

```python
async def execute(self) -> bool:
    """执行节点逻辑"""
    try:
        # 1. 初始化客户端
        if not await self.initialize_client():
            await self.set_status(NodeStatus.FAILED, "Failed to initialize Binance client")
            return False
        
        # 2. 获取 K 线数据
        klines = await asyncio.to_thread(
            self.client.get_klines,
            symbol=self.symbol,
            interval=self.interval,
            limit=self.limit
        )
        
        # 3. 处理数据
        processed_data = []
        for kline in klines:
            processed_data.append({
                "open_time": kline[0],
                "open": kline[1],
                "high": kline[2],
                "low": kline[3],
                "close": kline[4],
                "volume": kline[5],
                "close_time": kline[6],
                "quote_asset_volume": kline[7],
                "number_of_trades": kline[8],
                "taker_buy_base_asset_volume": kline[9],
                "taker_buy_quote_asset_volume": kline[10]
            })
        
        # 4. 发送信号
        await self.send_signal(
            source_handle="kline_data",
            signal_type=SignalType.PRICE_DATA,
            payload={"data": processed_data, "symbol": self.symbol, "interval": self.interval}
        )
        
        # 5. 更新状态
        await self.set_status(NodeStatus.COMPLETED)
        return True
        
    except BinanceAPIException as e:
        await self.set_status(NodeStatus.FAILED, f"Binance API error: {e}")
        return False
    except Exception as e:
        await self.set_status(NodeStatus.FAILED, str(e))
        return False
```

---

## 使用示例

### 示例 1：获取 BTC 1小时 K线

```json
{
  "id": "price_node_1",
  "type": "binance_price_node",
  "config": {
    "symbol": "BTCUSDT",
    "interval": "1h",
    "limit": 100
  }
}
```

### 示例 2：获取 ETH 5分钟 K线

```json
{
  "id": "price_node_2",
  "type": "binance_price_node",
  "config": {
    "symbol": "ETHUSDT",
    "interval": "5m",
    "limit": 200
  }
}
```

### 示例 3：连接到 AI 分析节点

```json
{
  "nodes": [
    {
      "id": "binance_price",
      "type": "binance_price_node",
      "config": {
        "symbol": "BTCUSDT",
        "interval": "1h",
        "limit": 50
      }
    },
    {
      "id": "ai_analyzer",
      "type": "ai_model_node",
      "config": {
        "model": "gpt-4",
        "prompt": "Analyze the price trend"
      }
    }
  ],
  "edges": [
    {
      "source": "binance_price",
      "sourceHandle": "kline_data",
      "target": "ai_analyzer",
      "targetHandle": "price_input"
    }
  ]
}
```

---

## 错误处理

### 常见错误

| 错误类型 | 原因 | 解决方案 |
|---------|------|----------|
| `BinanceAPIException` | API 调用失败 | 检查网络连接、API Key 配置 |
| `ConnectionError` | 网络连接问题 | 检查网络，节点会自动重试 |
| `Invalid symbol` | 交易对不存在 | 确认交易对名称正确 |
| `Rate limit exceeded` | 请求频率过高 | 降低请求频率或使用认证 API |

### 重试机制

节点内置 3 次重试机制，使用指数退避策略：
- 第 1 次重试：等待 1 秒
- 第 2 次重试：等待 2 秒
- 第 3 次重试：等待 4 秒

---

## 性能考虑

### API 限制

**公共 API：**
- 请求权重限制：1200/分钟
- IP 限制：无需认证

**认证 API：**
- 请求权重限制：6000/分钟
- 账户级别限制

### 优化建议

1. **合理设置 limit**：避免请求过多数据
2. **使用认证 API**：提高请求限制
3. **缓存数据**：避免重复请求相同数据
4. **选择合适的 interval**：根据策略需求选择时间周期

---

## 配置要求

### 环境变量

```bash
# 可选：使用认证 API
BINANCE_API_KEY=your_api_key_here
BINANCE_API_SECRET=your_api_secret_here
```

### 依赖包

```python
binance-connector==3.x  # Binance Python SDK
```

---

## 相关文档

- [节点开发指南](../weather-station-node-execution.md#开发新节点)
- [消息队列详解](../weather-station-message-queue.md)
- [AI Model Node](ai-model-node.md)
- [Binance API 官方文档](https://binance-docs.github.io/apidocs/)

---

**维护者：** TradingFlow 开发团队  
**最后更新：** 2025-10-06

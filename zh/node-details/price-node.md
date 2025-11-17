# Price Node（价格节点）

**节点类型**: `price_node`
**类别**: 输入
**版本**: 0.4.0

## 概述

Price Node 从 CoinGecko 获取加密货币价格数据，支持实时当前价格和历史 OHLC（开盘价、最高价、最低价、收盘价）K 线数据。

## 输入参数

| 参数        | 类型   | 必需 | 说明                                                           |
| ----------- | ------ | ---- | -------------------------------------------------------------- |
| `source`    | string | 是   | 数据源（目前仅支持 'coingecko'）                               |
| `data_type` | string | 是   | 数据类型：'kline' 获取 OHLC 数据，'current_price' 获取当前价格 |
| `symbol`    | string | 是   | 加密货币符号/ID（例如：'bitcoin'、'ethereum'、'aptos'）        |

## 输出

| 输出   | 类型      | 说明                                              |
| ------ | --------- | ------------------------------------------------- |
| `data` | json 对象 | 包含 source、data_type、symbol 和 data 的价格数据 |

## 数据输出格式

### 当前价格

```json
{
  "source": "coingecko",
  "data_type": "current_price",
  "symbol": "bitcoin",
  "data": {
    "coin_id": "bitcoin",
    "current_price": 50000.0,
    "last_updated": 1700000000
  }
}
```

### OHLC/K 线数据

```json
{
  "source": "coingecko",
  "data_type": "kline",
  "symbol": "bitcoin",
  "data": {
    "coin_id": "bitcoin",
    "vs_currency": "usd",
    "days": 30,
    "ohlc": [
      [1700000000000, 35000, 36000, 34500, 35800],
      [时间戳, 开盘价, 最高价, 最低价, 收盘价]
    ],
    "count": 120
  }
}
```

## Credits 费用

- **当前价格**: 2 credits（简单价格查询成本较低）
- **OHLC/K 线数据**: 5 credits（数据量较大成本较高）

## 常见用例

1. **价格监控**: 获取当前价格用于交易决策
2. **技术分析**: 获取历史 OHLC 数据进行模式识别
3. **警报系统**: 监控价格变动并触发通知
4. **投资组合跟踪**: 实时追踪资产价格

## 使用示例

### 场景 1：获取比特币当前价格

```
输入:
- source: "coingecko"
- data_type: "current_price"
- symbol: "bitcoin"

输出:
- 当前 BTC 美元价格
```

### 场景 2：获取以太坊 OHLC 数据

```
输入:
- source: "coingecko"
- data_type: "kline"
- symbol: "ethereum"

输出:
- 最近 30 天的 ETH OHLC 数据
```

## 注意事项

- 数据通过 Monitor 服务从 CoinGecko API 获取
- Symbol 必须是有效的 CoinGecko coin ID（小写）
- OHLC 数据包含最近 30 天的历史数据
- 多密钥支持确保高可用性和速率限制管理

## 从 Binance Price Node 迁移

Price Node 替代了已弃用的 Binance Price Node，主要变化如下：

| 旧版本 (Binance)         | 新版本 (CoinGecko)                        |
| ------------------------ | ----------------------------------------- |
| `symbol`（如 "BTCUSDT"） | `symbol`（如 "bitcoin"）                  |
| `interval`（如 "1h"）    | `data_type`（"kline" 或 "current_price"） |
| `limit`（K 线数量）      | 不可配置（固定 30 天）                    |
| Binance API              | CoinGecko API                             |

## 相关节点

- **AI Model Node**: 使用价格数据作为 AI 分析的输入
- **Code Node**: 使用自定义 Python 代码处理价格数据
- **Buy/Sell Nodes**: 基于价格数据做出交易决策

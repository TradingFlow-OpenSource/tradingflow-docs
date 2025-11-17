# Binance Price Node

Binance Price Node 是一个数据输入节点，用于从 Binance 交易所获取实时市场数据，包括 K 线（蜡烛图）数据和当前价格。

---

## 节点信息

| 属性         | 值                   |
| ------------ | -------------------- |
| **节点类型** | `binance_price_node` |
| **显示名称** | Binance Price        |
| **节点分类** | Input（数据输入）    |
| **图标**     | 🕯️ 蜡烛图图标        |
| **句柄颜色** | Amber（橙色）        |

---

## 功能说明

Binance Price Node 通过 Binance API 获取指定交易对的历史 K 线数据和当前价格。节点支持多种时间间隔（1 分钟到 1 年），并可以指定获取的 K 线数量。

**主要用途：**

- 获取加密货币价格数据用于技术分析
- 监控特定交易对的价格变化
- 为交易策略提供实时市场数据
- 生成价格图表和趋势分析

---

## 输入参数

### 参数列表

| 参数       | 类型         | 必填 | 默认值    | 范围/选项      | 说明               |
| ---------- | ------------ | ---- | --------- | -------------- | ------------------ |
| `symbol`   | searchSelect | ✅   | `BTCUSDT` | 见交易对列表   | 要查询的交易对符号 |
| `interval` | searchSelect | ✅   | `1m`      | 见时间间隔列表 | K 线时间间隔       |
| `limit`    | number       | ❌   | `100`     | 1-1000         | 获取的 K 线数量    |

### 支持的交易对

| 交易对     | 显示名称  | 说明           |
| ---------- | --------- | -------------- |
| `BTCUSDT`  | BTC/USDT  | 比特币对 USDT  |
| `BTCUSDC`  | BTC/USDC  | 比特币对 USDC  |
| `ETHUSDT`  | ETH/USDT  | 以太坊对 USDT  |
| `ETHUSDC`  | ETH/USDC  | 以太坊对 USDC  |
| `ETHBTC`   | ETH/BTC   | 以太坊对比特币 |
| `APTUSDT`  | APT/USDT  | Aptos 对 USDT  |
| `APTUSDC`  | APT/USDC  | Aptos 对 USDC  |
| `APTxBTC`  | APT/xBTC  | Aptos 对 xBTC  |
| `USDCUSDT` | USDC/USDT | USDC 对 USDT   |

_注意：实际可用交易对以 Binance 交易所支持为准。_

### 时间间隔选项

| 值   | 显示名称 | 说明        |
| ---- | -------- | ----------- |
| `1m` | 1 minute | 1 分钟 K 线 |
| `1h` | 1 hour   | 1 小时 K 线 |
| `1d` | 1 day    | 1 天 K 线   |
| `1w` | 1 week   | 1 周 K 线   |
| `1M` | 1 month  | 1 月 K 线   |
| `1y` | 1 year   | 1 年 K 线   |

_Binance API 还支持其他间隔如 3m, 5m, 15m, 30m, 2h, 4h, 6h, 8h, 12h, 3d 等，但前端界面未全部列出。_

---

## 输出参数

### 输出列表

| 输出 ID | 显示名称 | 数据类型 | 说明                                        |
| ------- | -------- | -------- | ------------------------------------------- |
| `data`  | Data     | object   | 完整的市场数据对象，包含当前价格和 K 线数据 |

### data 输出

**数据类型：** `object`

**完整数据结构：**

```typescript
{
  symbol: string;           // 交易对符号
  interval: string;         // K 线时间间隔
  limit: number;           // K 线数量
  header: string[];        // K 线数据列名
  kline_data: Array[];     // K 线数据数组
  current_price: string;   // 当前价格
}
```

**实际示例：**

```json
{
  "symbol": "BTCUSDT",
  "interval": "1h",
  "limit": 3,
  "header": [
    "open_time",
    "open",
    "high",
    "low",
    "close",
    "volume",
    "close_time",
    "quote_volume",
    "count",
    "taker_buy_volume",
    "taker_buy_quote_volume"
  ],
  "kline_data": [
    [
      1672531200000,       // open_time: 开盘时间（毫秒时间戳）
      "16590.00000000",    // open: 开盘价
      "16650.00000000",    // high: 最高价
      "16580.00000000",    // low: 最低价
      "16620.00000000",    // close: 收盘价
      "1234.56789000",     // volume: 成交量（基础货币）
      1672534799999,       // close_time: 收盘时间（毫秒时间戳）
      "20498765.43210000", // quote_volume: 成交额（计价货币）
      15234,               // count: 成交笔数
      "617.28394500",      // taker_buy_volume: 主动买入成交量
      "10249382.71605000", // taker_buy_quote_volume: 主动买入成交额
      "0"                  // 忽略字段
    ],
    [...],  // 更多 K 线数据
    [...]
  ],
  "current_price": "16620.00000000"
}
```

### K 线数据字段说明

| 索引 | 字段名                   | 数据类型 | 说明                        |
| ---- | ------------------------ | -------- | --------------------------- |
| 0    | `open_time`              | number   | 开盘时间（Unix 毫秒时间戳） |
| 1    | `open`                   | string   | 开盘价                      |
| 2    | `high`                   | string   | 最高价                      |
| 3    | `low`                    | string   | 最低价                      |
| 4    | `close`                  | string   | 收盘价                      |
| 5    | `volume`                 | string   | 成交量（基础货币，如 BTC）  |
| 6    | `close_time`             | number   | 收盘时间（Unix 毫秒时间戳） |
| 7    | `quote_volume`           | string   | 成交额（计价货币，如 USDT） |
| 8    | `count`                  | number   | 成交笔数                    |
| 9    | `taker_buy_volume`       | string   | 主动买入成交量              |
| 10   | `taker_buy_quote_volume` | string   | 主动买入成交额              |

---

## 信号传输

### 发送的信号

**信号句柄：** `data`

**信号类型：** `SignalType.PRICE_DATA`

**信号负载（Payload）：** 完整的市场数据对象，包含交易对、K 线数据和当前价格（见上文 data 输出结构）

### 信号流向示例

```
Binance Price Node
    ↓ (data 信号)
    ↓ payload: { symbol, interval, limit, header, kline_data, current_price }
    ↓
其他节点（如 Code Node、AI Model Node 等）
```

---

## 工作流程

### 节点执行流程

```
1. 初始化
   ├─ 创建 Binance API 客户端
   ├─ 支持有/无 API Key 两种模式
   └─ 连接到 Binance 服务器并验证

2. 获取数据
   ├─ 调用 get_klines API 获取历史 K 线数据
   └─ 调用 get_symbol_ticker API 获取当前价格

3. 处理数据
   ├─ 整合 K 线数据和当前价格
   ├─ 添加数据头部信息（header）
   └─ 构建完整的输出对象

4. 发送信号
   ├─ 通过 data 句柄发送 PRICE_DATA 信号
   └─ 传递完整的市场数据对象（包含 K线数据和当前价格）给下游节点

5. 完成
   └─ 设置节点状态为 COMPLETED
```

### 错误处理

节点具有完善的错误处理机制：

| 错误类型     | 处理方式                            | 重试次数 |
| ------------ | ----------------------------------- | -------- |
| API 连接失败 | 指数退避重试（1s, 2s, 4s）          | 3 次     |
| API 调用异常 | 记录错误日志，设置节点状态为 FAILED | 0 次     |
| 数据获取失败 | 记录警告日志，设置节点状态为 FAILED | 0 次     |
| 任务被取消   | 设置状态为 TERMINATED，清理资源     | N/A      |

---

## 使用示例

### 示例 1：获取 BTC 价格用于交易决策

**场景：** 监控 BTC/USDT 1 小时 K 线，获取最近 50 根 K 线用于趋势分析。

**配置：**

```json
{
  "symbol": "BTCUSDT",
  "interval": "1h",
  "limit": 50
}
```

**工作流连接：**

```
Binance Price Node (BTCUSDT, 1h)
    ↓ data
Code Node (计算移动平均线)
    ↓ ma_signal
Condition Node (判断买入信号)
    ↓ decision
Buy Node (执行买入)
```

---

### 示例 2：多交易对价格监控

**场景：** 同时监控多个交易对的价格变化。

**工作流结构：**

```
Binance Price Node (BTCUSDT, 1m) ─┐
Binance Price Node (ETHUSDT, 1m) ─┼─→ Code Node (价格对比分析)
Binance Price Node (APTUSDT, 1m) ─┘
```

---

### 示例 3：K 线数据可视化

**场景：** 获取日 K 线数据并输出到 Google Sheets 生成图表。

**配置：**

```json
{
  "symbol": "ETHUSDT",
  "interval": "1d",
  "limit": 30
}
```

**工作流连接：**

```
Binance Price Node (ETHUSDT, 1d, 30根)
    ↓ data
Code Node (格式化数据为表格)
    ↓ formatted_data
Dataset Output Node (写入 Google Sheets)
```

---

## API 配置

### 环境变量

节点使用以下环境变量配置 Binance API：

| 环境变量             | 说明                       | 必填 |
| -------------------- | -------------------------- | ---- |
| `BINANCE_API_KEY`    | Binance API 密钥           | ❌   |
| `BINANCE_API_SECRET` | Binance API 密钥（Secret） | ❌   |

**注意：**

- 如果不提供 API Key，节点会使用 Binance 公共 API 接口
- 公共 API 有访问频率限制，建议高频使用时配置 API Key
- 获取 K 线数据和当前价格不需要 API Key

### API 限制

根据 Binance API 文档：

| 限制类型          | 公共 API | 有 API Key |
| ----------------- | -------- | ---------- |
| **请求频率**      | 较低     | 较高       |
| **权重限制**      | 共享池   | 独立配额   |
| **最大 K 线数量** | 1000 根  | 1000 根    |

---

## 注意事项

### ⚠️ 重要提示

1. **交易对有效性**

   - 交易对符号必须是 Binance 支持的有效交易对
   - 建议从预设列表中选择，避免输入无效符号
   - 交易对符号会自动转换为大写

2. **时间间隔选择**

   - 不同时间间隔返回的数据量和精度不同
   - 1m 间隔适合短期交易策略
   - 1d 或更长间隔适合长期趋势分析

3. **数据量控制**

   - limit 参数最大值为 1000
   - 获取大量历史数据可能增加响应时间
   - 建议根据实际需求设置合理的 limit 值

4. **API 稳定性**

   - 节点内置了连接重试机制（最多 3 次）
   - 使用指数退避策略避免频繁重试
   - 网络不稳定时可能导致节点执行失败

5. **数据精度**
   - 价格和成交量等数值以字符串格式返回
   - 保持 Binance API 原始精度
   - 进行数值计算时需要转换为数字类型

---

## 常见问题

**Q: 如何处理 K 线数据进行技术分析？**

A: 可以连接 Code Node，使用 Python 的 pandas 库处理数据：

```python
import pandas as pd

# 输入数据来自 Binance Price Node
kline_data = input_data.get('kline_data', [])
header = ['open_time', 'open', 'high', 'low', 'close', 'volume', ...]

# 转换为 DataFrame
df = pd.DataFrame(kline_data, columns=header)
df['close'] = df['close'].astype(float)

# 计算移动平均线
df['ma_20'] = df['close'].rolling(window=20).mean()
df['ma_50'] = df['close'].rolling(window=50).mean()

# 输出处理后的数据
output_data = {
    'prices': df['close'].tolist(),
    'ma_20': df['ma_20'].tolist(),
    'ma_50': df['ma_50'].tolist()
}
```

---

**Q: data 输出中的 current_price 和 kline_data 中的 close 价格有什么区别？**

A:

- `data.current_price` 来自实时 ticker 接口，是最新的成交价格
- `data.kline_data` 中最后一根 K 线的 `close` 价格是该时间段的收盘价
- 如果当前 K 线未收盘，最后一根 K 线的 close 可能与 current_price 不同

---

**Q: 节点执行失败如何调试？**

A: 检查以下几点：

1. 交易对符号是否正确且被 Binance 支持
2. 网络连接是否正常
3. API Key 是否有效（如果配置了）
4. 查看节点日志获取详细错误信息
5. 确认 Binance API 服务是否正常运行

---

**Q: 如何获取实时价格而不是 K 线数据？**

A: `data` 输出中包含 `current_price` 字段，可以在下游节点中访问 `input_data.get('current_price')` 来获取实时价格。如果只需要价格而不需要完整的 K 线数据，可以在 Code Node 中提取该字段。

---

## 相关节点

- **Code Node** - 用于处理和分析 K 线数据
- **Condition Node** - 根据价格数据触发条件判断
- **AI Model Node** - 使用 AI 分析价格趋势
- **Dataset Output Node** - 将价格数据导出到外部存储

---

## 技术规格

| 规格项              | 值                             |
| ------------------- | ------------------------------ |
| **节点版本**        | 1.0.0                          |
| **支持的 API 版本** | Binance API v3                 |
| **最大并发执行**    | 取决于 API 限制                |
| **执行模式**        | 单次执行（获取一次数据后完成） |
| **资源清理**        | 自动清理 API 客户端            |
| **日志级别**        | DEBUG, INFO, WARNING, ERROR    |

---

**相关文档：**

- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [Weather 语法](../core-concepts/weather-syntax.md) - 工作流文件格式

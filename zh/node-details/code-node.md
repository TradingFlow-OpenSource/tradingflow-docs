# Code Node

Code Node 是一个强大的计算节点，允许用户执行自定义 Python 代码来处理数据、执行计算和实现复杂的业务逻辑。节点提供了安全的执行环境，包括 Gas 系统、资源限制和模块白名单机制。

---

## 节点信息

| 属性         | 值                  |
| ------------ | ------------------- |
| **节点类型** | `code_node`         |
| **显示名称** | Code                |
| **节点分类** | Compute（计算处理） |
| **图标**     | 💻 代码图标         |
| **句柄颜色** | Sky（天蓝色）       |

---

## 功能说明

Code Node 在受限的安全环境中执行用户提供的 Python 代码。节点内置了多种数据处理库（pandas、requests、BeautifulSoup 等），并通过 Gas 系统和资源限制确保代码安全执行。

**主要用途：**

- 自定义数据处理和转换逻辑
- 执行复杂的数学计算和统计分析
- 从外部 API 获取数据
- 解析和处理 HTML/JSON 数据
- 实现自定义的交易策略逻辑

**核心特性：**

- 🔒 **安全执行环境**：基于白名单的模块限制和 AST 静态分析
- ⛽ **Gas 系统**：类似以太坊的 Gas 机制，防止无限循环和过度资源消耗
- 📊 **丰富的内置库**：预导入 pandas、requests、BeautifulSoup 等常用库
- 💾 **资源监控**：实时跟踪内存使用、执行时间和代码复杂度
- 🔍 **多流输出**：分别捕获标准输出、错误输出和调试输出

---

## 输入参数

### 参数列表

| 参数          | 类型   | 必填 | 默认值 | 说明              |
| ------------- | ------ | ---- | ------ | ----------------- |
| `python_code` | button | ✅   | 见下文 | Python 代码编辑器 |
| `input_data`  | object | ❌   | `{}`   | 输入数据对象      |

### python_code 参数

**默认代码模板：**

```python
# Write Python code here
# You can use input_data_0, input_data_1 etc. to access input data
# Use output_data variable to store output results

output_data = {'result': 'Hello from Code Node!'}
```

**代码编写规则：**

1. **输入数据访问**

   - 自动聚合的输入：`input_data` 字典
   - 按索引访问：`input_data_0`, `input_data_1`, ...
   - 使用 `input_data.get('key')` 方式访问

2. **输出数据定义**

   - 必须定义 `output_data` 变量
   - 支持任意 Python 数据类型（dict、list、str、number 等）
   - 输出会自动发送到下游节点

3. **可用的内置库**
   - **pandas** (别名 `pd`)：数据处理
   - **requests**：HTTP 请求
   - **BeautifulSoup** (别名 `bs4`)：HTML 解析
   - **urllib**：URL 处理
   - 更多库详见"允许的模块"章节

**代码示例：**

```python
# 示例 1：处理 K 线数据
import pandas as pd

kline_data = input_data.get('kline_data', {})
df = pd.DataFrame(kline_data['kline_data'], columns=kline_data['header'])

# 转换数据类型
df['close'] = df['close'].astype(float)
df['volume'] = df['volume'].astype(float)

# 计算移动平均
df['ma_20'] = df['close'].rolling(window=20).mean()
df['ma_50'] = df['close'].rolling(window=50).mean()

# 生成交易信号
current_price = float(df['close'].iloc[-1])
ma_20 = float(df['ma_20'].iloc[-1])
ma_50 = float(df['ma_50'].iloc[-1])

if ma_20 > ma_50:
    signal = "buy"
else:
    signal = "hold"

output_data = {
    'signal': signal,
    'current_price': current_price,
    'ma_20': ma_20,
    'ma_50': ma_50
}
```

```python
# 示例 2：从外部 API 获取数据
import requests

response = requests.get('https://api.coingecko.com/api/v3/simple/price',
                        params={'ids': 'bitcoin', 'vs_currencies': 'usd'})
data = response.json()

btc_price = data['bitcoin']['usd']

output_data = {
    'bitcoin_price': btc_price,
    'data_source': 'CoinGecko'
}
```

```python
# 示例 3：解析 HTML 内容
from bs4 import BeautifulSoup
import requests

url = 'https://example.com'
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')

# 提取标题
title = soup.find('title').text

# 提取所有链接
links = [a.get('href') for a in soup.find_all('a')]

output_data = {
    'title': title,
    'links': links[:10]  # 只返回前10个链接
}
```

---

## 输出参数

### 输出列表

| 输出 ID       | 显示名称    | 数据类型 | 说明                       |
| ------------- | ----------- | -------- | -------------------------- |
| `output_data` | Output Data | object   | 代码执行结果和执行统计信息 |

### output_data 输出

**数据类型：** `object` (任意 Python 可序列化类型)

**数据结构：**

```json
{
  "your_data": "...",
  "_execution_stats": {
    "gas_used": 12345,
    "execution_time": 0.234,
    "max_gas": 1000000000,
    "max_memory_mb": 500
  }
}
```

**说明：**

- 用户定义的 `output_data` 内容
- 自动添加 `_execution_stats` 执行统计信息（包含 Gas 使用、执行时间等）
- 支持嵌套结构和复杂数据类型
- 代码中的 `print()` 语句输出会记录到节点日志中

**注意：** 执行日志和调试信息会自动记录到节点日志系统中，可以通过运行时面板查看。

---

## 安全机制

### Gas 系统

Code Node 实现了类似以太坊的 Gas 机制来限制代码执行的资源消耗。

**Gas 计算规则：**

| 操作类型    | Gas 消耗  | 说明                   |
| ----------- | --------- | ---------------------- |
| 基础消耗    | 100 Gas   | 节点启动基础消耗       |
| 每行代码    | 1 Gas     | 每执行一行增加 1 Gas   |
| 循环语句    | 10 Gas    | 每个 for/while 循环    |
| 条件语句    | 5 Gas     | 每个 if/else 语句      |
| pandas 操作 | 20 Gas    | DataFrame 操作额外消耗 |
| 执行时间    | 10 Gas/秒 | 根据实际执行时间增加   |

**默认限制：**

- **最大 Gas**：1,000,000,000 (10 亿)
- **基础 Gas**：100
- **Gas 超限**：自动终止执行

**Gas 估算：**

节点在执行前会自动估算代码的 Gas 消耗：

```
估算 Gas = 基础 Gas + (AST 节点数 × 2) + (字节码指令数 × 3) +
          (循环数 × 10) + (条件数 × 5) + (pandas 操作 × 20)
```

---

### 资源限制

**执行限制：**

| 限制类型         | 默认值 | 说明             |
| ---------------- | ------ | ---------------- |
| **执行超时**     | 30 秒  | 单次执行最长时间 |
| **最大递归深度** | 1000   | 函数调用栈深度   |
| **最大内存**     | 500 MB | 进程内存使用上限 |
| **最大 Gas**     | 10 亿  | 代码复杂度上限   |

**资源监控：**

- 每执行 10,000 行代码检查一次内存使用
- 超出限制自动终止执行并返回错误
- 所有限制触发都会在日志中记录

---

### 模块白名单

Code Node 只允许导入白名单中的模块，确保安全性。

**允许的模块：**

#### 核心 Python 模块

| 模块          | 说明         |
| ------------- | ------------ |
| `math`        | 数学函数     |
| `random`      | 随机数生成   |
| `datetime`    | 日期时间处理 |
| `time`        | 时间相关函数 |
| `calendar`    | 日历功能     |
| `json`        | JSON 编解码  |
| `re`          | 正则表达式   |
| `collections` | 集合数据类型 |
| `itertools`   | 迭代器工具   |
| `functools`   | 函数工具     |
| `statistics`  | 统计函数     |
| `decimal`     | 十进制运算   |
| `fractions`   | 分数运算     |

#### 数据处理模块

| 模块     | 别名 | 说明         |
| -------- | ---- | ------------ |
| `pandas` | `pd` | 数据分析库   |
| `numpy`  | `np` | 数值计算库   |
| `csv`    | -    | CSV 文件处理 |

#### 网络和数据采集

| 模块       | 说明        |
| ---------- | ----------- |
| `requests` | HTTP 客户端 |
| `urllib`   | URL 处理    |
| `urllib3`  | URL 处理库  |

#### HTML/XML 解析

| 模块            | 别名             | 说明            |
| --------------- | ---------------- | --------------- |
| `bs4`           | `beautifulsoup4` | HTML 解析       |
| `BeautifulSoup` | -                | 解析类          |
| `lxml`          | -                | XML/HTML 解析器 |
| `html5lib`      | -                | HTML5 解析器    |
| `xml`           | -                | XML 处理        |
| `html`          | -                | HTML 处理       |

#### 数据格式和编码

| 模块       | 说明                |
| ---------- | ------------------- |
| `base64`   | Base64 编解码       |
| `binascii` | 二进制和 ASCII 转换 |
| `zlib`     | 压缩库              |
| `gzip`     | Gzip 压缩           |

#### 文本处理

| 模块          | 说明             |
| ------------- | ---------------- |
| `string`      | 字符串常量和工具 |
| `textwrap`    | 文本包装和填充   |
| `unicodedata` | Unicode 数据库   |

**使用示例：**

```python
import pandas as pd  # ✅ 允许
import requests      # ✅ 允许
from bs4 import BeautifulSoup  # ✅ 允许

import os           # ❌ 禁止：系统操作
import subprocess   # ❌ 禁止：执行系统命令
import socket       # ❌ 禁止：低级网络访问
```

---

### 安全检查

**静态分析检查：**

代码在执行前会进行 AST（抽象语法树）静态分析：

| 检查项       | 说明                              | 结果     |
| ------------ | --------------------------------- | -------- |
| 危险模块导入 | `os`, `subprocess`, `socket` 等   | 拒绝执行 |
| 非白名单模块 | 不在允许列表中的模块              | 拒绝执行 |
| 危险函数调用 | `eval`, `exec`, `__import__` 等   | 拒绝执行 |
| 系统调用     | `os.system`, `subprocess.call` 等 | 拒绝执行 |
| 内置函数修改 | 修改 `__builtins__`, `__dict__`   | 拒绝执行 |

**动态执行监控：**

代码执行时进行实时监控：

- **Gas 追踪**：每行代码执行增加 Gas 计数
- **内存监控**：定期检查进程内存使用
- **超时检测**：执行时间超过限制自动终止
- **异常捕获**：所有异常被捕获并记录

---

## 工作流程

### 节点执行流程

```
1. 安全检查
   ├─ AST 静态分析
   ├─ 模块白名单验证
   ├─ 危险函数检测
   └─ 风险等级评估

2. Gas 估算
   ├─ 分析代码复杂度
   ├─ 计算预估 Gas
   └─ 检查是否超出限制

3. 环境准备
   ├─ 创建隔离的执行环境
   ├─ 预导入常用模块 (pandas, requests, bs4)
   ├─ 设置输入数据变量
   └─ 初始化输出捕获器

4. 代码执行
   ├─ 在独立线程中执行
   ├─ 启用 Gas 追踪回调
   ├─ 捕获标准输出和错误
   └─ 监控资源使用

5. 结果处理
   ├─ 收集 output_data
   ├─ 添加执行统计信息
   ├─ 发送输出信号
   └─ 设置节点状态

6. 清理
   └─ 停止追踪，释放资源
```

---

## 使用示例

### 示例 1：价格数据技术分析

**场景：** 接收 Binance Price Node 的 K 线数据，计算技术指标。

**工作流结构：**

```
Binance Price Node (BTC/USDT, 1h, 100根)
    ↓ kline_data
Code Node (技术分析)
    ↓ output_data
Condition Node (判断买卖信号)
```

**Code Node 代码：**

```python
import pandas as pd

# 获取K线数据
kline_data = input_data.get('kline_data', {})
header = kline_data.get('header', [])
klines = kline_data.get('kline_data', [])

# 转换为DataFrame
df = pd.DataFrame(klines, columns=header)
df['close'] = df['close'].astype(float)
df['high'] = df['high'].astype(float)
df['low'] = df['low'].astype(float)
df['volume'] = df['volume'].astype(float)

# 计算移动平均线
df['ma_5'] = df['close'].rolling(window=5).mean()
df['ma_20'] = df['close'].rolling(window=20).mean()
df['ma_50'] = df['close'].rolling(window=50).mean()

# 计算RSI
def calculate_rsi(prices, period=14):
    delta = prices.diff()
    gain = (delta.where(delta > 0, 0)).rolling(window=period).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(window=period).mean()
    rs = gain / loss
    return 100 - (100 / (1 + rs))

df['rsi'] = calculate_rsi(df['close'])

# 计算MACD
exp1 = df['close'].ewm(span=12, adjust=False).mean()
exp2 = df['close'].ewm(span=26, adjust=False).mean()
df['macd'] = exp1 - exp2
df['signal_line'] = df['macd'].ewm(span=9, adjust=False).mean()

# 获取最新值
current_price = float(df['close'].iloc[-1])
rsi = float(df['rsi'].iloc[-1])
macd = float(df['macd'].iloc[-1])
signal_line = float(df['signal_line'].iloc[-1])
ma_20 = float(df['ma_20'].iloc[-1])
ma_50 = float(df['ma_50'].iloc[-1])

# 生成交易信号
signal = "hold"
if rsi < 30 and macd > signal_line and ma_20 > ma_50:
    signal = "buy"
    reason = "RSI超卖 + MACD金叉 + 趋势向上"
elif rsi > 70 and macd < signal_line:
    signal = "sell"
    reason = "RSI超买 + MACD死叉"
else:
    reason = "等待更好的入场时机"

output_data = {
    'signal': signal,
    'reason': reason,
    'current_price': current_price,
    'rsi': rsi,
    'macd': macd,
    'ma_20': ma_20,
    'ma_50': ma_50,
    'confidence': 0.75 if signal != "hold" else 0.0
}
```

---

### 示例 2：数据抓取和处理

**场景：** 从外部 API 获取加密货币恐慌贪婪指数。

**工作流结构：**

```
Code Node (抓取恐慌指数)
    ↓ output_data
AI Model Node (分析市场情绪)
    ↓ ai_response
Condition Node (决策)
```

**Code Node 代码：**

```python
import requests
import json

# 获取恐慌贪婪指数
url = "https://api.alternative.me/fng/"
response = requests.get(url)
data = response.json()

# 解析数据
fng_value = int(data['data'][0]['value'])
fng_classification = data['data'][0]['value_classification']
timestamp = data['data'][0]['timestamp']

# 分类解释
if fng_value <= 25:
    sentiment = "极度恐慌"
    recommendation = "可能是买入机会"
elif fng_value <= 45:
    sentiment = "恐慌"
    recommendation = "谨慎买入"
elif fng_value <= 55:
    sentiment = "中性"
    recommendation = "观望为主"
elif fng_value <= 75:
    sentiment = "贪婪"
    recommendation = "考虑获利了结"
else:
    sentiment = "极度贪婪"
    recommendation = "高风险，建议减仓"

output_data = {
    'fear_greed_index': fng_value,
    'classification': fng_classification,
    'sentiment': sentiment,
    'recommendation': recommendation,
    'timestamp': timestamp
}

print(f"当前恐慌贪婪指数: {fng_value} - {sentiment}")
```

---

### 示例 3：数据格式转换

**场景：** 将 AI 分析结果转换为交易参数。

**工作流结构：**

```
AI Model Node (分析推文)
    ↓ ai_response
Code Node (参数转换)
    ↓ output_data
Swap Node (执行交易)
```

**Code Node 代码：**

```python
# 获取AI分析结果
ai_response = input_data.get('ai_response', {})
response_text = ai_response.get('response', '')

# 提取关键信息
# 假设AI返回格式为: "建议买入 BTC，金额 1000 USDT"
import re

# 提取代币符号
token_match = re.search(r'(买入|卖出)\s+(\w+)', response_text)
if token_match:
    action = "buy" if token_match.group(1) == "买入" else "sell"
    token = token_match.group(2)
else:
    action = "hold"
    token = ""

# 提取金额
amount_match = re.search(r'(\d+(?:\.\d+)?)\s*USDT', response_text)
amount = float(amount_match.group(1)) if amount_match else 0.0

# 转换为Swap Node所需的格式
output_data = {
    'from_token': 'USDT' if action == 'buy' else token,
    'to_token': token if action == 'buy' else 'USDT',
    'amount_in_human_readable': amount,
    'chain': 'aptos',
    'slippage_tolerance': 1.0,
    'action': action
}

print(f"交易参数: {action} {amount} {token}")
```

---

## 最佳实践

### 1. 代码组织

✅ **推荐：**

```python
# 清晰的结构和注释
import pandas as pd

# 1. 数据准备
data = input_data.get('kline_data', {})
df = pd.DataFrame(data['kline_data'], columns=data['header'])

# 2. 数据处理
df['close'] = df['close'].astype(float)
df['ma_20'] = df['close'].rolling(window=20).mean()

# 3. 生成输出
output_data = {
    'ma_20': float(df['ma_20'].iloc[-1])
}
```

❌ **避免：**

```python
# 混乱的代码，难以维护
d=input_data.get('kline_data',{});df=pd.DataFrame(d['kline_data'],columns=d['header']);df['close']=df['close'].astype(float);output_data={'ma_20':float(df['close'].rolling(window=20).mean().iloc[-1])}
```

---

### 2. 错误处理

✅ **推荐：**

```python
try:
    data = input_data.get('kline_data')
    if not data:
        output_data = {'error': 'No input data'}
    else:
        # 处理数据
        result = process_data(data)
        output_data = {'result': result}
except Exception as e:
    output_data = {'error': str(e)}
    print(f"Error: {e}")
```

---

### 3. 性能优化

**减少 Gas 消耗：**

- 避免不必要的循环
- 使用向量化操作（pandas/numpy）
- 限制数据处理量

**示例：**

```python
# ❌ 低效：使用循环
total = 0
for value in df['close']:
    total += float(value)
average = total / len(df)

# ✅ 高效：使用向量化
average = df['close'].astype(float).mean()
```

---

### 4. 调试技巧

使用 `print()` 输出调试信息：

```python
print(f"输入数据类型: {type(input_data)}")
print(f"输入数据键: {input_data.keys()}")

# 处理数据
result = process_data(input_data)
print(f"处理结果: {result}")

output_data = result
```

**注意：** 代码中的 `print()` 输出会自动记录到节点日志中，可在运行时面板查看。

---

## 注意事项

### ⚠️ 重要提示

1. **必须定义 output_data**

   - 代码必须设置 `output_data` 变量
   - 如果不设置，节点执行会失败

2. **模块导入限制**

   - 只能导入白名单中的模块
   - 尝试导入禁止的模块会导致执行失败

3. **Gas 限制**

   - 复杂计算可能超出 Gas 限制
   - 大数据处理建议分批进行
   - 避免深层嵌套循环

4. **执行超时**

   - 默认超时 30 秒
   - 长时间运行的代码会被终止
   - HTTP 请求建议设置超时

5. **内存限制**

   - 默认限制 500 MB
   - 处理大数据集时注意内存使用
   - 及时释放不需要的变量

6. **线程安全**
   - 代码在独立线程中执行
   - 不要使用线程相关操作
   - 避免全局状态修改

---

## 故障排查

**Q: 代码执行失败，提示"模块导入被禁止"？**

A: 检查以下几点：

1. 确认模块在白名单中
2. 检查模块名称拼写是否正确
3. 查看完整的模块白名单列表
4. 尝试使用替代模块

---

**Q: 执行超时或 Gas 超限？**

A: 优化代码：

1. 减少循环嵌套层数
2. 使用向量化操作替代循环
3. 限制处理的数据量
4. 分解为多个 Code Node

---

**Q: output_data 未定义错误？**

A:

1. 确保代码中定义了 `output_data` 变量
2. 检查代码是否有异常导致提前退出
3. 添加默认值：`output_data = {}`

---

**Q: 如何查看详细的执行日志？**

A:

1. 在代码中使用 `print()` 输出调试信息
2. 查看节点的执行日志（在运行时面板中）
3. 检查 `output_data` 中的 `_execution_stats` 字段获取执行统计信息

---

## 技术规格

| 规格项            | 值                      |
| ----------------- | ----------------------- |
| **节点版本**      | 1.0.0                   |
| **Python 版本**   | 3.8+                    |
| **默认超时**      | 30 秒                   |
| **默认 Gas 限制** | 1,000,000,000           |
| **默认内存限制**  | 500 MB                  |
| **最大递归深度**  | 1000                    |
| **执行模式**      | 单线程，异步监控        |
| **安全级别**      | 高（白名单 + AST 分析） |

---

## 相关节点

- **AI Model Node** - 用于智能数据分析和决策
- **Binance Price Node** - 提供价格数据
- **Dataset Output Node** - 保存处理后的数据
- **Condition Node** - 根据 Code Node 输出做条件判断

---

**相关文档：**

- [节点与工作流](../core-concepts/nodes-and-workflows.md) - 节点基础概念
- [AI Model Node](ai-model-node.md) - AI 模型节点
- [Weather 语法](../core-concepts/weather-syntax.md) - 工作流文件格式

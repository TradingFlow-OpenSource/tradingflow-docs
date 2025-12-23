# 节点详情

欢迎查看 TradingFlow 的节点使用文档。每个节点都有详细的参数说明、使用示例和最佳实践。

---

## 📊 数据输入节点

### 价格数据
- **[Price Node](price-node.md)** - 从 CoinGecko 获取加密货币价格数据（当前价格和 OHLC K线）

### 社交媒体
- **[X Listener Node](x-listener-node.md)** - 监控 Twitter/X 社交媒体推文

### 数据平台
- **[RootData Node](rootdata-node.md)** - 查询 RootData API（项目搜索、热门榜单、VC、融资等）

### 数据存储
- **[Google Sheet Input Node](gsheet-input-node.md)** - 从 Google Sheets 读取数据

---

## 🧠 计算处理节点

### 智能分析
- **[AI Model Node](ai-model-node.md)** - 使用 GPT-4 等大语言模型进行智能分析和决策

### 代码执行
- **[Code Node](code-node.md)** - 执行自定义 Python 代码进行数据处理

---

## 💰 交易执行节点

### 金库管理
- **[Vault Node](vault-node.md)** - 连接金库并查询资产信息

### 交易操作
- **[Swap Node](swap-node.md)** - 执行代币交换交易（基类节点）
- **[Buy Node](buy-node.md)** - 执行买入操作
- **[Sell Node](sell-node.md)** - 执行卖出操作

---

## 🔧 核心功能节点

### 数据输出
- **[Google Sheet Output Node](gsheet-output-node.md)** - 将数据写入 Google Sheets

### 消息通知
- **[Telegram Sender Node](telegram-sender-node.md)** - 发送 Telegram 消息通知

---

## 🚧 更多节点即将推出

### 🤔 决策节点
- **条件判断节点** - If/Then/Else 逻辑
- **技术指标节点** - RSI、MACD 等技术分析
- **风险评估节点** - 评估交易风险等级

### ⚡ 执行节点
- **金库管理节点** - 存款/取款操作
- **通知节点** - 发送消息提醒

### 🛠️ 工具节点
- **定时器节点** - 定时触发工作流
- **计算节点** - 数学运算和数据处理
- **转换节点** - 数据格式转换
- **延迟节点** - 添加执行延迟

敬请期待完整的节点使用指南！

---

## 📖 高级参数功能

部分节点支持**高级参数**（Advanced Parameters），这些参数默认隐藏以简化界面：

- 点击节点底部的 "Show advanced params" 按钮
- 在弹窗中选择需要的参数
- 或点击 "Show All" 显示全部参数

当 Agent 为高级参数指定值时，该参数会自动显示在 UI 中。

**支持高级参数的节点：**
- RootData Node（22 个高级参数）
- X Listener Node（3 个高级参数）

---

---

## 💬 需要帮助？

加入我们的社区获取支持：

👉 **[Telegram 社区](https://t.me/tradingflowai)**

---

现在可以先查看：
- [核心概念](../core-concepts/on-chain-vaults.md) - 了解 TradingFlow 的基础架构
- [工程文档](../engineering-docs/development-background.md) - 了解技术背景

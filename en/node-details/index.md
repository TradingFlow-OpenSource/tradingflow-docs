# Node Details

Welcome to TradingFlow's node usage documentation. Each node has detailed parameter descriptions, usage examples, and best practices.

---

## 📊 Data Input Nodes

### Price Data
- **[Price Node](price-node.md)** - Fetch cryptocurrency price data from CoinGecko (current price and OHLC)

### Social Media
- **[X Listener Node](x-listener-node.md)** - Monitor Twitter/X social media tweets

### Data Platform
- **[RootData Node](rootdata-node.md)** - Query RootData API (project search, trending lists, VCs, fundraising, etc.)

### Data Storage
- **[Google Sheet Input Node](gsheet-input-node.md)** - Read data from Google Sheets

---

## 🧠 Compute Nodes

### Intelligent Analysis
- **[AI Model Node](ai-model-node.md)** - Use GPT-4 and other LLMs for intelligent analysis and decision-making

### Code Execution
- **[Code Node](code-node.md)** - Execute custom Python code for data processing

---

## 💰 Trade Nodes

### Vault Management
- **[Vault Node](vault-node.md)** - Connect to vault and query asset information

### Trading Operations
- **[Swap Node](swap-node.md)** - Execute token swap transactions (Base Node)
- **[Buy Node](buy-node.md)** - Execute buy operations
- **[Sell Node](sell-node.md)** - Execute sell operations

---

## 🔧 Core Function Nodes

### Data Output
- **[Google Sheet Output Node](gsheet-output-node.md)** - Write data to Google Sheets

### Message Notifications
- **[Telegram Sender Node](telegram-sender-node.md)** - Send Telegram message notifications

---

## 🚧 More Nodes Coming Soon

### 🤔 Decision Nodes
- **Condition Node** - If/Then/Else logic
- **Technical Indicator Node** - RSI, MACD and other technical analysis
- **Risk Assessment Node** - Evaluate trading risk levels

### ⚡ Execution Nodes
- **Vault Management Node** - Deposit/withdrawal operations
- **Notification Node** - Send message alerts

### 🛠️ Utility Nodes
- **Timer Node** - Schedule workflow triggers
- **Calculation Node** - Mathematical operations and data processing
- **Transform Node** - Data format conversion
- **Delay Node** - Add execution delays

Stay tuned for complete node usage guides!

---

## 📖 Advanced Parameters Feature

Some nodes support **Advanced Parameters** that are hidden by default to simplify the interface:

- Click the "Show advanced params" button at the bottom of the node
- Select the parameters you need in the popup
- Or click "Show All" to display all parameters

When the Agent specifies a value for an advanced parameter, it automatically becomes visible in the UI.

**Nodes with Advanced Parameters:**
- RootData Node (22 advanced parameters)
- X Listener Node (3 advanced parameters)

---

For now, you can check out:
- [Core Concepts](../core-concepts/on-chain-vaults.md) - Understand TradingFlow's basic architecture
- [Engineering Docs](../engineering-docs/development-background.md) - Learn about technical background

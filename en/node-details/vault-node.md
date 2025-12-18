# Vault Node

Vault Node is a trade execution node used to connect and query user asset information in the TradingFlow vault system. It supports multi-chain queries including Aptos and Flow EVM, and can retrieve vault balances, portfolio composition, and asset values.

---

## Node Information

| Property         | Value             |
| ---------------- | ----------------- |
| **Node Type**    | `vault_node`      |
| **Display Name** | Vault             |
| **Category**     | Trade (Execution) |
| **Icon**         | 🏦 Bank Icon      |
| **Handle Color** | Amber (Orange)    |

---

## Functionality

Vault Node connects to TradingFlow's decentralized vault system to query asset holdings for a specified user address. The node automatically calculates asset values, supports multi-chain queries, and passes results to downstream trade nodes.

**Main Uses:**

- Query vault asset holdings and balances
- Retrieve multi-chain asset value information
- Provide funding source for trade nodes
- Monitor portfolio changes
- Calculate total asset value (USD)

**Core Features:**

- 🌐 **Multi-Chain Support**: Aptos and Flow EVM
- 💰 **Real-Time Pricing**: Automatically fetches token prices and calculates asset values
- 📊 **Portfolio**: Complete holdings composition and ratio information
- 🔗 **Chain Passing**: Passes vault address and chain info to downstream trade nodes
- 🔄 **Auto Refresh**: Updates latest asset information on each execution

---

## Input Parameters

### Parameter List

| Parameter       | Type   | Required | Default | Description               |
| --------------- | ------ | -------- | ------- | ------------------------- |
| `chain`         | select | ✅       | `aptos` | Blockchain network        |
| `vault_address` | text   | ✅       | -       | Vault or user address     |

> **Note:** The `chain_id` parameter has been removed (v0.4.1+). The system automatically determines the corresponding `chain_id` based on the `chain` parameter:
> - `aptos` → No chain_id needed
> - `flow_evm` → Automatically uses 545 (testnet) or 747 (mainnet)

### chain Parameter

**Supported Blockchains:**

| Chain        | Value      | chain_id (auto) | Description           |
| ------------ | ---------- | --------------- | --------------------- |
| **Aptos**    | `aptos`    | -               | Aptos mainnet/testnet |
| **Flow EVM** | `flow_evm` | 545             | Flow EVM testnet      |

**Selection Guide:**

- If your vault is on Aptos, select `aptos`
- If your vault is on Flow EVM, select `flow_evm`

### vault_address Parameter

**Format Requirements:**

| Chain    | Address Format              | Example                                                              |
| -------- | --------------------------- | -------------------------------------------------------------------- |
| Aptos    | `0x` prefixed hex address   | `0x6a1a233e9c3871fc3719e4238bf61218c98d3b89fa5c2a37c87e7f6d60e07292` |
| Flow EVM | `0x` prefixed ETH address   | `0x1234567890123456789012345678901234567890`                         |

**Description:**

- This is the vault address you created in the TradingFlow system
- You can view your vault address in the Vaults tab on the Windmill page
- The address must be a valid created vault

---

## Output Parameters

### Output List

| Output ID | Display Name | Data Type | Description                                              |
| --------- | ------------ | --------- | -------------------------------------------------------- |
| vault     | Vault        | object    | Complete vault info object with chain, address, balance  |

### vault Output

**Data Type:** object

**Complete Data Structure:**

```typescript
{
  success: boolean;              // Query success status
  vault_address: string;         // Vault address
  chain: string;                 // Blockchain network
  holdings: Array<{              // Holdings list
    token_address: string;       // Token contract address
    amount: string;              // Raw amount (smallest unit)
    symbol: string;              // Token symbol
    decimals: number;            // Decimal places
    price_usd: number;           // Unit price (USD)
    value_usd: number;           // Holding value (USD)
    amount_human_readable: number; // Readable amount
    percentage?: number;         // Ratio (Flow EVM only)
  }>;
  total_value_usd: number;       // Total value (USD)
  token_count?: number;          // Token count (Flow EVM)
  timestamp?: string;            // Query timestamp
  message: string;               // Status message
}
```

**Description:**

- The vault output contains complete vault information including chain, address, balance, holdings
- Downstream nodes can extract any required fields from this object
- Contains real-time prices and calculated asset values

---

## Signal Transmission

### Sent Signal

**Signal Handle:** `vault`

**Signal Type:** `SignalType.VAULT_INFO`

**Signal Payload:** Complete vault info object (see vault output structure above)

### Signal Flow Example

```
Vault Node
    ↓ vault (contains chain, vault_address, holdings, total_value_usd, etc.)
    ↓
Swap/Buy/Sell Node (extracts required fields from vault object for trading)
```

---

## Workflow

### Node Execution Flow

```
1. Parameter Validation
   ├─ Check if chain is supported (aptos/flow_evm)
   ├─ Check if vault_address is provided
   └─ Auto-determine chain_id from config table based on chain

2. Initialize Service
   ├─ Aptos: AptosVaultService
   └─ Flow EVM: FlowEvmVaultService

3. Query Holdings
   ├─ Aptos: get_investor_holdings()
   └─ Flow EVM: get_vault_info_with_prices()

4. Price Calculation
   ├─ Get USD price for each token
   ├─ Calculate value of each holding
   └─ Sum total asset value

5. Format Output
   ├─ Unify Aptos and Flow EVM data format
   ├─ Add readable amounts and percentages
   └─ Build complete output object

6. Send Signal
   └─ Send VAULT_INFO signal to downstream nodes

7. Complete
   └─ Set node status to COMPLETED
```

---

## Usage Examples

### Example 1: Aptos Vault Query

**Scenario:** Query holdings in Aptos vault, then execute Swap.

**Vault Node Configuration:**

```json
{
  "chain": "aptos",
  "vault_address": "0x6a1a233e9c3871fc3719e4238bf61218c98d3b89fa5c2a37c87e7f6d60e07292"
}
```

**Workflow:**

```
Vault Node (Aptos)
    ↓ vault (holdings: 100 APT, 50 USDT)
    ↓
Swap Node (USDT → APT)
    ↓ trade_receipt
Telegram Sender Node (Send notification)
```

### Example 2: Flow EVM Vault Query

**Scenario:** Query Flow EVM testnet vault, check balance then execute buy.

**Vault Node Configuration:**

```json
{
  "chain": "flow_evm",
  "vault_address": "0x1234567890123456789012345678901234567890"
}
```

> Note: No need to configure `chain_id`, system automatically uses 545 (testnet) for `chain: "flow_evm"`.

**Workflow:**

```
Vault Node (Flow EVM)
    ↓ vault (holdings info)
    ↓
Code Node (Check if enough USDC)
    ↓ decision
Buy Node (Buy FLOW)
```

---

## Important Notes

### ⚠️ Key Points

1. **Vault Address Validation**
   - Address must be a valid created vault
   - Using wrong address will cause query failure
   - Recommend confirming address on Windmill page

2. **Chain Parameter Consistency**
   - chain parameter must match vault's actual chain
   - chain_id is auto-determined by system based on chain (no manual config needed)
   - Cross-chain vault queries not supported

3. **Output Signal Usage**
   - Downstream trade nodes receive complete info via vault object
   - vault object contains chain, address, holdings, and all required fields
   - Trade nodes extract needed info from vault object for execution

---

## Troubleshooting

**Q: "vault_address is required but not provided" error?**

A:
1. Confirm Vault Node's vault_address parameter is filled
2. If using signal passing, confirm upstream node correctly sends vault_address
3. Check address format (Aptos: 0x..., Flow EVM: 0x...)

---

**Q: Query returns empty holdings or value 0?**

A:
1. Confirm vault address is correct
2. Confirm vault actually has assets
3. Check network connection
4. View node logs for detailed error info

---

**Q: Flow EVM query fails?**

A:
1. Confirm chain parameter is `flow_evm` (chain_id auto-determined as 545)
2. Check if Flow EVM companion service is running
3. Confirm vault address format is correct
4. Check error logs for specific issues

---

## Technical Specifications

| Specification    | Value                       |
| ---------------- | --------------------------- |
| **Node Version** | 1.0.0                       |
| **Chains**       | Aptos, Flow EVM             |
| **Max Concurrency**| 1                         |
| **Execution Mode**| Single execution (query once then complete)|
| **Timeout**      | 30 seconds (service call)   |
| **Log Levels**   | DEBUG, INFO, WARNING, ERROR |

---

## Related Nodes

- **Swap Node** - Use vault to execute token swap
- **Buy Node** - Use vault to buy tokens
- **Sell Node** - Use vault to sell tokens
- **Code Node** - Process vault data and balance checks

---

## Related Documentation

- [Nodes and Workflows](../core-concepts/nodes-and-workflows.md) - Node basics

---

**Maintained by:** TradingFlow Development Team  
**Version:** 1.0.0

# Buy Node

Buy Node is a specialized trading node for executing buy operations, an instance node of Swap Node. It provides clear buy semantics and simplifies token purchase configuration.

---

## Node Information

| Property           | Value                           |
| ------------------ | ------------------------------- |
| **Node Type**      | `buy_node`                      |
| **Display Name**   | Buy                             |
| **Category**       | Trade (Execution)               |
| **Icon**           | 📈 Upward Trend Icon            |
| **Handle Color**   | Amber (Orange) / Emerald (Green)|
| **Node Category**  | Instance Node                   |
| **Base Node Type** | `swap_node`                     |

---

## Functionality

Buy Node is designed specifically for token purchase operations. It internally uses Swap Node's complete exchange functionality but provides more intuitive parameter naming. Users only need to specify the token to buy and the payment token, and the node automatically handles the exchange logic.

**Main Uses:**

- Purchase target tokens
- Clear buy semantics
- Simplified parameter configuration
- Automatic exchange direction handling

**Core Features:**

- 💰 **Buy Semantics**: `buy_token` and `base_token` parameters are more intuitive
- 🔄 **Auto Mapping**: Internally converts to Swap Node's from/to parameters
- 📊 **Full Functionality**: Inherits all Swap Node features
- 🎯 **Buy-Focused**: Optimized user experience

---

## Relationship with Swap Node

Buy Node is an **instance node** of Swap Node:

```
SwapNode (Base Class)
    ├─ from_token (source token)
    └─ to_token (target token)
        ↓ Instance
BuyNode (Instance)
    ├─ base_token → from_token (what to pay with)
    └─ buy_token → to_token (what to buy)
```

**Parameter Mapping:**

- `base_token` → `from_token` (payment token)
- `buy_token` → `to_token` (token to purchase)

---

## Input Parameters

### Parameter List

| Parameter                  | Type         | Required | Default                           | Description                                                |
| -------------------------- | ------------ | -------- | --------------------------------- | ---------------------------------------------------------- |
| `buy_token`                | searchSelect | ✅       | -                                 | Token symbol to buy                                        |
| `base_token`               | searchSelect | ✅       | -                                 | Token symbol to pay with                                   |
| `amount_in_human_readable` | **switch**   | ✅       | `{mode:"spend_fixed",value:""}` | Amount (v0.4.1 supports 4 modes)                           |
| `slippery`                 | number       | ✅       | `1.0`                             | Slippage tolerance (%)                                     |
| `vault`                    | object       | ✅       | -                                 | Vault object (from Vault Node, contains chain, address, balance)|

### amount_in_human_readable Parameter

**Description:** Amount setting, v0.4.1 supports 4 modes for flexible amount control.

**Type:** Switch (Mode Selector)

**Data Structure:**

```typescript
{
  mode: "spend_fixed" | "spend_percent" | "buy_fixed" | "buy_percent",
  value: string  // Value as string
}
```

**Supported Modes:**

| Mode            | Description                                    | Use Case               |
| --------------- | ---------------------------------------------- | ---------------------- |
| `spend_fixed`   | Spend fixed amount of base_token               | Precise cost control   |
| `spend_percent` | Spend percentage of base_token balance (0-100) | Proportional allocation|
| `buy_fixed`     | Buy fixed amount of buy_token                  | Precise quantity target|
| `buy_percent`   | Increase buy_token holdings by percentage      | Position scaling       |

**Mode 1: spend_fixed (Fixed Spending)**

Specify exact amount of `base_token` to spend.

```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "spend_fixed",
    "value": "100"
  }
}
// Meaning: Spend 100 USDT to buy APT
```

**Mode 2: spend_percent (Percentage Spending)**

Spend a percentage of `base_token` balance (0-100).

```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "spend_percent",
    "value": "50"
  }
}
// Meaning: Spend 50% of USDT balance to buy APT
```

**Mode 3: buy_fixed (Fixed Purchase Amount)**

Specify exact amount of `buy_token` to purchase.

```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "buy_fixed",
    "value": "10"
  }
}
// Meaning: Buy 10 APT (system calculates required USDT)
```

**Mode 4: buy_percent (Increase Holdings Percentage)**

Increase current `buy_token` holdings by specified percentage.

```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "buy_percent",
    "value": "20"
  }
}
// Meaning: Increase APT holdings by 20% (if holding 100 APT, buy 20 more)
```

**Backward Compatibility:**

- `"number"` mode auto-maps to `"spend_fixed"`
- `"percentage"` mode auto-maps to `"spend_percent"`

### slippery Parameter

**Format:** Percentage (recommended 0.5-5.0)

| Scenario           | Recommended | Description      |
| ------------------ | ----------- | ---------------- |
| High liquidity     | 0.5-1.0%    | Major pairs      |
| Medium liquidity   | 1.0-3.0%    | Regular tokens   |
| Low liquidity      | 3.0-5.0%    | Small cap tokens |

### vault Parameter

**Source:** Vault Node output

**Description:**

- Must be received from upstream Vault Node
- Contains complete vault info: chain, address, holdings, total_value_usd, etc.
- Used for trade execution, balance queries, and chain determination

---

## Output Parameters

| Output ID       | Display Name  | Data Type | Description            |
| --------------- | ------------- | --------- | ---------------------- |
| `trade_receipt` | Trade Receipt | object    | Complete trade receipt |

**Key Fields:**

- `from_token`: `base_token` value
- `to_token`: `buy_token` value
- `amount_in`: Actual payment amount
- `amount_out`: Actual purchase amount
- `tx_hash`: Transaction hash

---

## Usage Examples

### Example 1: Buy APT with USDT (spend_fixed mode)

**Scenario:** Spend 100 USDT to buy APT.

**Workflow:**

```
Vault Node (Aptos)
    ↓ vault
Buy Node (Buy APT)
    ↓ trade_receipt
Telegram Sender Node (Send notification)
```

**Buy Node Configuration:**

```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "spend_fixed",
    "value": "100"
  },
  "slippery": 1.0
}
```

### Example 2: Buy with 50% Balance (spend_percent mode)

**Scenario:** Use 50% of USDT balance to buy xBTC.

```json
{
  "buy_token": "xBTC",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "spend_percent",
    "value": "50"
  },
  "slippery": 0.5
}
```

### Example 3: Buy Fixed Token Amount (buy_fixed mode)

**Scenario:** Buy exactly 10 APT.

```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "buy_fixed",
    "value": "10"
  },
  "slippery": 1.0
}
```

### Example 4: Increase Holdings (buy_percent mode)

**Scenario:** Increase APT holdings by 20%.

```json
{
  "buy_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "buy_percent",
    "value": "20"
  },
  "slippery": 1.0
}
```

---

## Conditional Trading

If `buy_token` or `base_token` is an empty string `""`, the node will skip execution and return a `trade_receipt` with `skipped: true`.

---

## Best Practices

### Mode Selection (v0.4.1)

| Scenario                   | Recommended Mode |
| -------------------------- | ---------------- |
| Precise cost control       | `spend_fixed`    |
| Proportional allocation    | `spend_percent`  |
| Purchase specific quantity | `buy_fixed`      |
| Scale position by ratio    | `buy_percent`    |

---

## Technical Specifications

| Specification    | Value                      |
| ---------------- | -------------------------- |
| **Node Version** | 0.0.2                      |
| **Node Category**| Instance Node              |
| **Base Node**    | `swap_node`                |
| **Inherited**    | 100%                       |
| **Chains**       | Aptos, Flow EVM            |
| **DEXs**         | Hyperion (Aptos), Flow DEX |

---

## Related Nodes

- **[Swap Node](swap-node.md)** - Base node, general token swap
- **[Sell Node](sell-node.md)** - Instance node for selling tokens
- **[Vault Node](vault-node.md)** - Vault management node

---

## Related Documentation

- [Nodes and Workflows](../core-concepts/nodes-and-workflows.md) - Node basics
- [Swap Node](swap-node.md) - Complete base node documentation

---

**Maintained by:** TradingFlow Development Team
**Version:** 1.0.0

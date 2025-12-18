# Sell Node

Sell Node is a specialized trading node for executing sell operations, an instance node of Swap Node. It provides clear sell semantics and simplifies token selling configuration.

---

## Node Information

| Property           | Value                           |
| ------------------ | ------------------------------- |
| **Node Type**      | `sell_node`                     |
| **Display Name**   | Sell                            |
| **Category**       | Trade (Execution)               |
| **Icon**           | 📉 Downward Trend Icon          |
| **Handle Color**   | Amber (Orange) / Emerald (Green)|
| **Node Category**  | Instance Node                   |
| **Base Node Type** | `swap_node`                     |

---

## Functionality

Sell Node is designed specifically for token selling operations. It internally uses Swap Node's complete exchange functionality but provides more intuitive parameter naming. Users only need to specify the token to sell and the token to receive, and the node automatically handles the exchange logic.

**Main Uses:**

- Sell held tokens
- Clear sell semantics
- Simplified parameter configuration
- Automatic exchange direction handling

**Core Features:**

- 💸 **Sell Semantics**: `sell_token` and `base_token` parameters are more intuitive
- 🔄 **Auto Mapping**: Internally converts to Swap Node's from/to parameters
- 📊 **Full Functionality**: Inherits all Swap Node features
- 🎯 **Sell-Focused**: Optimized user experience

---

## Relationship with Swap Node

Sell Node is an **instance node** of Swap Node:

```
SwapNode (Base Class)
    ├─ from_token (source token)
    └─ to_token (target token)
        ↓ Instance
SellNode (Instance)
    ├─ sell_token → from_token (what to sell)
    └─ base_token → to_token (what to receive)
```

**Parameter Mapping:**

- `sell_token` → `from_token` (token to sell)
- `base_token` → `to_token` (token to receive)

**Comparison with Buy Node:**

| Node      | Operation | from_token          | to_token             |
| --------- | --------- | ------------------- | -------------------- |
| Buy Node  | Buy       | base_token (pay)    | buy_token (receive)  |
| Sell Node | Sell      | sell_token (sell)   | base_token (receive) |

---

## Input Parameters

### Parameter List

| Parameter                  | Type         | Required | Default                           | Description                                                |
| -------------------------- | ------------ | -------- | --------------------------------- | ---------------------------------------------------------- |
| `sell_token`               | searchSelect | ✅       | -                                 | Token symbol to sell                                       |
| `base_token`               | searchSelect | ✅       | -                                 | Token symbol to receive                                    |
| `amount_in_human_readable` | **switch**   | ✅       | `{mode:"sell_fixed",value:""}` | Amount (v0.4.1 supports 4 modes)                           |
| `slippery`                 | number       | ✅       | `1.0`                             | Slippage tolerance (%)                                     |
| `vault`                    | object       | ✅       | -                                 | Vault object (from Vault Node, contains chain, address, balance)|

### amount_in_human_readable Parameter

**Description:** Amount setting, v0.4.1 supports 4 modes for flexible amount control.

**Type:** Switch (Mode Selector)

**Data Structure:**

```typescript
{
  mode: "sell_fixed" | "sell_percent" | "receive_fixed" | "receive_percent",
  value: string  // Value as string
}
```

**Supported Modes:**

| Mode              | Description                                     | Use Case               |
| ----------------- | ----------------------------------------------- | ---------------------- |
| `sell_fixed`      | Sell fixed amount of sell_token                 | Precise sell quantity  |
| `sell_percent`    | Sell percentage of sell_token balance (0-100)   | Proportional reduction |
| `receive_fixed`   | Receive fixed amount of base_token              | Precise income target  |
| `receive_percent` | Increase base_token holdings by percentage      | Stablecoin rebalancing |

**Mode 1: sell_fixed (Fixed Sell Amount)**

Specify exact amount of `sell_token` to sell.

```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "sell_fixed",
    "value": "10"
  }
}
// Meaning: Sell 10 APT for USDT
```

**Mode 2: sell_percent (Percentage Selling)**

Sell a percentage of `sell_token` balance (0-100).

```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "sell_percent",
    "value": "50"
  }
}
// Meaning: Sell 50% of APT balance for USDT
```

**Mode 3: receive_fixed (Fixed Receive Amount)**

Specify exact amount of `base_token` to receive.

```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "receive_fixed",
    "value": "100"
  }
}
// Meaning: Receive 100 USDT (system calculates how much APT to sell)
```

**Mode 4: receive_percent (Increase Stablecoin Percentage)**

Increase current `base_token` holdings by specified percentage.

```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "receive_percent",
    "value": "20"
  }
}
// Meaning: Increase USDT holdings by 20% (if holding 500 USDT, sell APT for 100 USDT)
```

**Backward Compatibility:**

- `"number"` mode auto-maps to `"sell_fixed"`
- `"percentage"` mode auto-maps to `"sell_percent"`

### slippery Parameter

**Format:** Percentage (recommended 0.5-5.0)

| Scenario           | Recommended | Description       |
| ------------------ | ----------- | ----------------- |
| High liquidity     | 0.5-1.0%    | Major pairs       |
| Medium liquidity   | 1.0-3.0%    | Regular tokens    |
| Low liquidity      | 3.0-5.0%    | Small cap tokens  |
| Emergency stop-loss| 3.0-5.0%    | Ensure execution  |

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

- `from_token`: `sell_token` value
- `to_token`: `base_token` value
- `amount_in`: Actual sold amount
- `amount_out`: Actual received amount
- `tx_hash`: Transaction hash

---

## Usage Examples

### Example 1: Sell APT for USDT (sell_fixed mode)

**Scenario:** Sell 10 APT for USDT.

**Workflow:**

```
Vault Node (Aptos)
    ↓ vault
Sell Node (Sell APT)
    ↓ trade_receipt
Telegram Sender Node (Send notification)
```

**Sell Node Configuration:**

```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "sell_fixed",
    "value": "10"
  },
  "slippery": 1.0
}
```

### Example 2: Sell 50% Holdings (sell_percent mode)

**Scenario:** Sell 50% of current xBTC holdings.

```json
{
  "sell_token": "xBTC",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "sell_percent",
    "value": "50"
  },
  "slippery": 0.5
}
```

### Example 3: Receive Fixed USDT (receive_fixed mode)

**Scenario:** Sell APT to receive 100 USDT.

```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "receive_fixed",
    "value": "100"
  },
  "slippery": 1.0
}
```

### Example 4: Increase Stablecoin Ratio (receive_percent mode)

**Scenario:** Increase USDT holdings by 20%.

```json
{
  "sell_token": "APT",
  "base_token": "USDT",
  "amount_in_human_readable": {
    "mode": "receive_percent",
    "value": "20"
  },
  "slippery": 1.0
}
```

---

## Conditional Trading

If `sell_token` or `base_token` is an empty string `""`, the node will skip execution and return a `trade_receipt` with `skipped: true`.

---

## Best Practices

### Mode Selection (v0.4.1)

| Scenario                     | Recommended Mode  |
| ---------------------------- | ----------------- |
| Precise sell quantity        | `sell_fixed`      |
| Proportional reduction       | `sell_percent`    |
| Receive specific amount      | `receive_fixed`   |
| Increase stablecoin holdings | `receive_percent` |

### Sell Strategies

**Full Sell (sell_percent):**

```json
{
  "amount_in_human_readable": {
    "mode": "sell_percent",
    "value": "100"
  }
}
```

**Batch Sell (sell_percent):**

```json
{
  "amount_in_human_readable": {
    "mode": "sell_percent",
    "value": "25"
  }
}
// Sell 25% each time, 4 times total
```

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
- **[Buy Node](buy-node.md)** - Instance node for buying tokens
- **[Vault Node](vault-node.md)** - Vault management node

---

## Related Documentation

- [Nodes and Workflows](../core-concepts/nodes-and-workflows.md) - Node basics
- [Swap Node](swap-node.md) - Complete base node documentation

---

**Maintained by:** TradingFlow Development Team
**Version:** 1.0.0

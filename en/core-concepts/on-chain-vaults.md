# On-Chain Vaults

TradingFlow follows the principles of decentralized applications and does not custody users' keys, wallets, or assets in any form. Therefore, when users choose to deploy workflows on a blockchain network (such as Bitcoin, Ethereum, Aptos, Flow, etc.) that require trading capabilities, they need to create corresponding on-chain "vaults" in advance.

These vaults belong solely to the users themselves, with only the users having the rights to deposit and withdraw. When a user's configured workflow determines that a transaction is needed, TradingFlow sends a signal to the on-chain vault, and the transaction is executed through the decentralized exchange on the corresponding blockchain network. Throughout this entire process, the TradingFlow team does not custody or misuse any funds.

## How It Works

On-chain vaults adopt a non-custodial architecture design. When a workflow needs to execute a trade operation, the system sends an execution signal to the user's on-chain vault rather than directly operating user assets. After receiving the signal, the vault smart contract completes the actual trading operations through decentralized exchanges on the blockchain network, according to preset authorization rules and parameters.

This design ensures that every step of fund flow can be traced and verified on-chain, with users maintaining complete control over their assets at all times. TradingFlow plays the role of information messenger and coordinator in the entire process, rather than asset custodian.

## Vault Management Features

Users can view their vault balance information and operation history on the TradingFlow Windmill page, and perform create, deposit, and withdrawal operations. The platform provides an intuitive interface to display:

- Vault asset balances across different blockchain networks
- Complete transaction history including deposits, withdrawals, and trades
- Real-time asset portfolio changes and profit statistics

## Supported Blockchain Networks

Currently, TradingFlow supports multiple mainstream blockchain networks, including:

- **Aptos** - High-performance Move language blockchain
- **Flow EVM** - Ethereum-compatible Flow ecosystem
- More blockchain networks are continuously being integrated

Each network has its unique characteristics and advantages, and users can choose the appropriate chain to create vaults based on their needs and preferences.

---

**Next:** Learn about [Decentralized Account System](decentralized-account-system.md) to understand how to manage multiple identities and assets.

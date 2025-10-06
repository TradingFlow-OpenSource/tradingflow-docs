# Decentralized Account System

We recognize that in the Web3 world, a user may have multiple third-party platform identities (for example, a person can have multiple Google accounts, Microsoft accounts, wallet addresses, etc.). Our account system accommodates this multi-identity nature.

When a user logs in for the first time through an authentication identity (ID), the platform automatically creates a TradingFlow Account for that user and identity. While logged in with that account, users can bind more identities in the Vault Page, such as linking their other wallets/addresses. This way, when logging in next time with a non-initial identity, they will automatically be logged into the same account.

## Account Binding Rules

In summary, a user's first login creates an account, one account can bind multiple identities, and any single identity can only be bound to one account. If rebinding is needed, the extra account must be deactivated first, and then the target identity can be bound in the target account.

This design provides users with great flexibility. Users don't need to remember specific account-password combinations, but can log in through any bound identity. The system automatically recognizes the identity and navigates to the correct account while maintaining consistency of all data and configurations.

## Practical Application Scenarios

Through this built-in "decentralized identity" account model, users can conveniently isolate funds and switch identities. For example, a user might have meme strategy workflows as well as Bitcoin DCA workflows.

Both sets of funds can be managed through on-chain vaults created from different addresses under one account. This not only avoids repeatedly switching accounts but also enables logging back into the same account through any wallet, providing the most convenient operation for users.

The advantages of this architecture include:

- Users can use different wallet addresses for different investment strategies, achieving risk isolation
- All strategies and configurations are unified under one account, facilitating monitoring and adjustments
- Support for quick access from any bound identity, improving operational convenience

---

**Next Step:** Learn about [Nodes and Workflows](nodes-and-workflows.md) to start building your automated strategies.

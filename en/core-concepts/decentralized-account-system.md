# Decentralized Account System

## 🌐 Multi-Identity Compatibility

We recognize that in the Web3 world, users often have multiple third-party platform identities, such as:
- Multiple Google accounts
- Multiple Microsoft accounts
- Multiple wallet addresses
- Other authentication identities

TradingFlow's account system perfectly accommodates this **multi-identity** requirement.

## 📝 Relationship Between Accounts and Identities

### First-Time Login
When a user logs in for the first time with an authentication identity (ID), the platform automatically creates a **TradingFlow Account** for that user and identity.

### Binding More Identities
After logging in, users can bind more identities in the **Vault Page**, such as:
- Binding other wallet addresses
- Linking other social accounts

This way, when logging in with a non-initial identity next time, they will automatically access the same account.

### Core Rules

- **One User** → First login creates one account
- **One Account** → Can bind multiple identities
- **One Identity** → Can only bind to one account

> 💡 **Tip:** If you need to rebind, you must first deactivate the extra account, then bind the target identity in the target account.

## 🎯 Practical Use Cases

### Fund Isolation
Through this "decentralized identity" account model, users can conveniently isolate funds and switch identities.

**Case:** A user might have:
- **Meme Strategy Workflow** - Managed with vault from address A
- **Bitcoin DCA Workflow** - Managed with vault from address B

Both funds are managed through on-chain vaults created from different addresses under the same account.

### Convenient Operations
Benefits of this design:
- ✅ Avoid repeatedly switching accounts
- ✅ Access the same account through any wallet
- ✅ Flexibly manage multiple strategies and assets
- ✅ Most user-friendly operation experience

---

**Next Step:** Learn about [Nodes and Workflows](nodes-and-workflows.md) to start building your automated strategies.

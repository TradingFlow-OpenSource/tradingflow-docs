# Quick Start Guide

Get your first automated trading strategy running in under 10 minutes. This guide will walk you through creating a simple Dollar Cost Averaging (DCA) strategy on the Aptos blockchain.

## Prerequisites

Before we begin, make sure you have:
- A crypto wallet (we'll use OKX Wallet in this example)
- Some APT tokens for gas fees (minimum 1 APT recommended)
- Basic understanding of cryptocurrency trading

## Step 1: Create Your Account

1. **Visit TradingFlow Platform**
   - Go to [app.tradingflow.ai](https://app.tradingflow.ai)
   - Click "Get Started" or "Sign Up"

2. **Connect Your Wallet**
   - Select "Connect Wallet"
   - Choose your wallet provider (OKX Wallet, MetaMask, etc.)
   - Approve the connection in your wallet

3. **Complete Profile Setup**
   - Verify your email address
   - Set up two-factor authentication (recommended)
   - Accept terms and conditions

## Step 2: Create Your First Vault

A vault is your secure trading account where strategies execute trades and store assets.

1. **Navigate to Vaults**
   - Click "My Vaults" in the sidebar
   - Click "Create New Vault"

2. **Configure Vault Settings**
   ```
   Network: Aptos
   Type: Personal Vault
   Name: "My First DCA Vault"
   Initial Deposit: 10 APT (or your preferred amount)
   ```

3. **Deploy Vault**
   - Review gas fees (typically ~0.1 APT)
   - Click "Deploy Vault"
   - Confirm transaction in your wallet
   - Wait for deployment confirmation (~30 seconds)

## Step 3: Build Your DCA Strategy

Now let's create a simple DCA strategy that buys USDC every hour when APT price drops below a certain threshold.

### Access the Flow Editor
1. Click "Create New Flow" from the dashboard
2. Choose "Start from Scratch" or "DCA Template"
3. Give your flow a name: "APT to USDC DCA"

### Add Nodes to Your Flow

#### 1. Price Data Node
- **Drag** "Price Feed Node" from the Data Nodes section
- **Configure**:
  - Token Pair: APT/USDC
  - Update Interval: 1 minute
  - Data Source: Multiple (for reliability)

#### 2. Condition Node
- **Drag** "Condition Logic Node" from Decision Nodes
- **Configure**:
  - Condition: "Price < Target Price"
  - Target Price: $8.00 (adjust based on current market)
  - Logic: "Execute if true"

#### 3. Timer Node
- **Drag** "Timer Node" from Utility Nodes
- **Configure**:
  - Interval: 1 hour
  - Start Immediately: Yes
  - Max Executions: Unlimited

#### 4. Buy Node
- **Drag** "Buy Node" from Action Nodes
- **Configure**:
  - From Token: APT
  - To Token: USDC
  - Amount: 1 APT per purchase
  - Slippage Tolerance: 1%
  - Vault: Select your created vault

### Connect the Nodes

Create the following connections by dragging from output handles to input handles:

```
Timer Node → Condition Node
Price Feed Node → Condition Node
Condition Node → Buy Node
```

## Step 4: Test Your Strategy

Before going live, always test your strategy:

1. **Validate Flow**
   - Click "Validate" button
   - Fix any connection or configuration errors
   - Ensure all nodes are properly connected

2. **Paper Trading**
   - Enable "Paper Trading Mode"
   - Let strategy run for 24 hours
   - Monitor execution logs
   - Verify behavior matches expectations

## Step 5: Deploy and Monitor

### Deploy Your Strategy
1. **Final Review**
   - Double-check all node configurations
   - Verify vault has sufficient balance
   - Confirm network and gas settings

2. **Go Live**
   - Click "Deploy Strategy"
   - Confirm deployment transaction
   - Wait for strategy activation (~1 minute)

### Monitor Performance
1. **Dashboard Overview**
   - View real-time execution status
   - Monitor profit/loss metrics
   - Track vault balance changes

2. **Execution Logs**
   - See detailed logs of each trade
   - Monitor node-by-node execution
   - Troubleshoot any issues

## What You've Accomplished

Congratulations! You've just:
- ✅ Created your first TradingFlow vault
- ✅ Built a DCA trading strategy with visual nodes
- ✅ Deployed an automated trading bot
- ✅ Set up monitoring and notifications

## Next Steps

### Enhance Your Strategy
- **Add Stop-Loss**: Protect against major downturns
- **Dynamic Sizing**: Adjust purchase amounts based on volatility
- **Multiple Conditions**: Combine technical indicators
- **Multi-Token DCA**: Diversify across different assets

### Explore Advanced Features
- [Core Concepts →](../core-concepts/)
- [Advanced Trading Patterns →](../for-traders/advanced-trading-patterns.md)
- [Node Reference Guide →](../for-traders/node-reference-guide.md)

## Troubleshooting

### Strategy Not Executing
- Check vault balance is sufficient
- Verify wallet connection is active
- Ensure gas fee limits are appropriate

### Transaction Failures
- Increase slippage tolerance slightly
- Check for sufficient gas funds
- Verify DEX liquidity is adequate

Need help? Join our [Discord community](../resources/community-support.md) for real-time support!

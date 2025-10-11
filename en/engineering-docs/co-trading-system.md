# Co-Trading System (Social Trading Platform)

## Overview

The Co-Trading System enables users to share their trading strategies (Flows) and allows other users (followers) to automatically copy their trades. This creates a social trading ecosystem where expert traders can monetize their strategies and followers can benefit from proven trading approaches.

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                        Co-Trading Flow                       │
└─────────────────────────────────────────────────────────────┘

Creator's Flow Execution
         ↓
    SwapNode.execute() → Publishes signal to MQ (trading_signals)
         ↓
Control: TradingSignalConsumer receives
         ↓
    1. Save TradingSignal
    2. Find active subscriptions (autoFollow=true)
    3. Create SignalExecution records
         ↓
Send batch to Monitor → /api/v1/co-trading/execute-batch
         ↓
Monitor: executeTradingSignalBatch
         ↓
Execute trades for each follower (batch size=5)
         ↓
Update SignalExecution status (completed/failed)
         ↓
Every 4 hours: Update follower stats and AUM
```

### Database Models (MongoDB - Control)

#### SharedFlow Model
Represents a flow shared by a creator for co-trading.

```javascript
{
  _id: ObjectId,
  flowId: String,               // Reference to Flow in database
  userId: ObjectId,             // Creator's user ID
  
  // Display Information
  title: String,
  description: String,
  tags: [String],               // e.g., ['scalping', 'long-term', 'aptos']
  
  // Statistics
  stats: {
    followers: Number,          // Number of active followers
    totalAUM: Number,           // Total Assets Under Management (USD)
    performance: {
      totalReturn: Number,      // Total return percentage
      winRate: Number,          // Percentage of winning trades
      sharpeRatio: Number,      // Risk-adjusted return
      maxDrawdown: Number       // Maximum drawdown percentage
    }
  },
  
  // Settings
  settings: {
    isActive: Boolean,          // Whether accepting new followers
    maxFollowers: Number,       // Maximum followers allowed (null = unlimited)
    minInvestment: Number       // Minimum investment required (USD)
  },
  
  // Status
  status: String,               // 'active' | 'paused' | 'closed'
  
  // Timestamps
  createdAt: Date,
  updatedAt: Date,
  lastSignalAt: Date            // Last trading signal timestamp
}
```

#### FlowSubscription Model
Tracks a follower's subscription to a shared flow.

```javascript
{
  _id: ObjectId,
  flowId: String,               // Shared flow ID
  creatorId: ObjectId,          // Creator's user ID
  followerId: ObjectId,         // Follower's user ID
  
  // Follower's Trading Setup
  followerVaultAddress: String, // Follower's vault for executing trades
  chainType: String,            // 'aptos' | 'flow_evm' | 'bsc'
  
  // Investment Tracking
  investmentAmount: Number,     // Initial investment (USD)
  currentValue: Number,         // Current portfolio value (USD)
  profitLoss: Number,           // P/L in USD
  profitLossPercentage: Number, // P/L percentage
  
  // Settings
  settings: {
    autoFollow: Boolean,        // Auto-execute trades
    notifyOnSignal: Boolean,    // Send notifications
    maxSlippage: Number         // Max slippage tolerance (%)
  },
  
  // Status
  status: String,               // 'active' | 'paused' | 'cancelled'
  
  // Timestamps
  subscribedAt: Date,
  lastExecutedAt: Date,         // Last trade execution
  updatedAt: Date
}
```

#### TradingSignal Model
Represents a trading signal published by a creator.

```javascript
{
  _id: ObjectId,
  signalId: String,             // Unique signal identifier
  flowId: String,               // Shared flow ID
  creatorId: ObjectId,
  creatorVaultAddress: String,
  
  // Signal Details
  signalType: String,           // 'swap' | 'buy' | 'sell' | 'deposit' | 'withdraw'
  signalData: {
    fromToken: String,
    toToken: String,
    amountInPercentage: Number,
    amountInHumanReadable: Number,
    slippageTolerance: Number,
    chainType: String
  },
  
  // Execution Status
  executionStatus: {
    total: Number,              // Total followers
    executed: Number,           // Successfully executed
    failed: Number,             // Failed executions
    pending: Number             // Pending executions
  },
  
  // Metadata
  nodeId: String,               // Source node ID
  nodeType: String,             // Source node type
  
  // Timestamps
  createdAt: Date,
  processedAt: Date             // When signal was processed
}
```

#### SignalExecution Model
Tracks execution of a signal for an individual follower.

```javascript
{
  _id: ObjectId,
  executionId: String,
  signalId: ObjectId,           // Reference to TradingSignal
  subscriptionId: ObjectId,     // Reference to FlowSubscription
  
  // Follower Information
  followerId: ObjectId,
  followerVaultAddress: String,
  
  // Execution Details
  status: String,               // 'pending' | 'executing' | 'completed' | 'failed'
  executedAmount: Number,       // Actual executed amount
  executedPrice: Number,        // Execution price
  gasCost: Number,              // Gas fee paid
  slippage: Number,             // Actual slippage
  
  // Transaction Details
  txHash: String,
  blockNumber: Number,
  
  // Error Information
  error: String,                // Error message if failed
  
  // Timestamps
  createdAt: Date,
  executedAt: Date,
  completedAt: Date
}
```

## Service Layer

### Station (Python) - Signal Publishing

#### TradingSignalPublisher
Publishes trading signals to RabbitMQ.

**Location:** `03_weather_station/mq/trading_signal_publisher.py`

```python
def publish_trading_signal(
    flow_id: str,
    user_id: str,
    vault_address: str,
    signal_type: str,
    signal_data: dict,
    node_id: str = None,
    node_type: str = None
) -> bool:
    """
    Publish trading signal to MQ
    
    Args:
        flow_id: Flow ID
        user_id: Creator user ID
        vault_address: Creator's vault address
        signal_type: Type of signal (swap, buy, sell)
        signal_data: Signal parameters
        node_id: Source node ID
        node_type: Source node type
    
    Returns:
        bool: Success status
    """
```

**Integration in SwapNode:**
```python
async def _publish_trading_signal(self, trade_receipt: Dict):
    """Publish trading signal to MQ for co-trading system"""
    try:
        if not trade_receipt.get("success", False):
            return
        
        signal_data = {
            "fromToken": self.from_token,
            "toToken": self.to_token,
            "amountInPercentage": self.amount_in_percentage,
            "amountInHumanReadable": self.amount_in_human_readable,
            "slippageTolerance": self.slippery,
            "chainType": self.chain
        }
        
        success = publish_trading_signal(
            flow_id=self.flow_id,
            user_id=getattr(self, 'user_id', 'unknown'),
            vault_address=self.vault_address,
            signal_type='swap',
            signal_data=signal_data,
            node_id=self.node_id,
            node_type='swap_node'
        )
        
        if success:
            await self.persist_log(f"Trading signal published: flow={self.flow_id}", "INFO")
    except Exception as e:
        await self.persist_log(f"Error publishing signal: {str(e)}", "WARNING")
```

### Control (Node.js) - Signal Processing

#### TradingSignalConsumer
Consumes trading signals from RabbitMQ and creates execution tasks.

**Location:** `02_weather_control/src/services/TradingSignalConsumer.js`

**Key Methods:**

```javascript
class TradingSignalConsumer {
  async init() {
    // Initialize MQ consumer
    // Listen to 'trading_signals' exchange
  }
  
  async handleTradingSignal(message) {
    // 1. Parse signal from message
    // 2. Save TradingSignal to database
    // 3. Find active subscriptions (autoFollow=true)
    // 4. Create SignalExecution records for each follower
    // 5. Send batch to Monitor for execution
  }
  
  async createExecutionBatch(signal, subscriptions) {
    // Create SignalExecution records
    // Group by batch size (default: 5)
    // Send to Monitor API
  }
}
```

#### CoTradingService
Handles co-trading business logic.

**Location:** `02_weather_control/src/services/CoTradingService.js`

**Key Methods:**

```javascript
class CoTradingService {
  // Flow Management
  async createOrUpdateSharedFlow(flowId, userId, data)
  async toggleSharedFlowStatus(flowId, action)
  async getSharedFlows(filters, pagination)
  async getMySharedFlows(userId, pagination)
  
  // Subscription Management
  async subscribeToFlow(userId, flowId, vaultAddress, chainType, settings)
  async unsubscribeFromFlow(subscriptionId)
  async getUserSubscriptions(userId)
  async updateSubscriptionSettings(subscriptionId, settings)
  
  // Signal & Execution
  async getFlowSignals(flowId, limit)
  async getSignalExecutions(signalId)
  async updateExecutionStatus(executionId, status, result)
}
```

### Monitor (TypeScript) - Trade Execution

#### executeTradingSignals Task
Executes trades for followers in batches.

**Location:** `04_weather_monitor/src/tasks/executeTradingSignals.ts`

```typescript
export async function executeTradingSignalBatch(
  signalId: string,
  executions: SignalExecution[]
): Promise<void> {
  const batchSize = 5; // Process 5 at a time
  
  for (let i = 0; i < executions.length; i += batchSize) {
    const batch = executions.slice(i, i + batchSize);
    
    await Promise.all(batch.map(async (execution) => {
      try {
        // 1. Get follower's vault service
        // 2. Execute trade with signal parameters
        // 3. Update execution status
        await executeTradeForFollower(execution);
      } catch (error) {
        // Update execution as failed
        await updateExecutionStatus(execution._id, 'failed', error.message);
      }
    }));
  }
}
```

#### updateFollowerStats Task
Updates follower statistics and AUM periodically (every 4 hours).

**Location:** `04_weather_monitor/src/tasks/updateFollowerStats.ts`

```typescript
export async function updateAllFollowerStats(): Promise<void> {
  // 1. Get all active subscriptions
  const subscriptions = await FlowSubscription.find({ status: 'active' });
  
  for (const subscription of subscriptions) {
    try {
      // 2. Query current vault value
      const vaultValue = await getVaultValue(
        subscription.followerVaultAddress,
        subscription.chainType
      );
      
      // 3. Calculate P/L
      const profitLoss = vaultValue - subscription.investmentAmount;
      const profitLossPercentage = (profitLoss / subscription.investmentAmount) * 100;
      
      // 4. Update subscription
      await FlowSubscription.updateOne(
        { _id: subscription._id },
        {
          currentValue: vaultValue,
          profitLoss,
          profitLossPercentage,
          updatedAt: new Date()
        }
      );
    } catch (error) {
      logger.error(`Failed to update stats for subscription ${subscription._id}`, error);
    }
  }
  
  // 5. Update SharedFlow AUM
  await updateSharedFlowAUM();
}
```

## API Endpoints

### Creator Endpoints

#### Create/Update Shared Flow
```
POST /api/v1/co-trading/flows
```

**Request Body:**
```json
{
  "flowId": "flow_123",
  "title": "Scalping Strategy",
  "description": "High-frequency scalping on Aptos",
  "tags": ["scalping", "aptos"],
  "maxFollowers": 100,
  "minInvestment": 1000
}
```

#### Toggle Flow Status
```
POST /api/v1/co-trading/flows/:flowId/toggle
```

**Request Body:**
```json
{
  "action": "pause" // or "resume"
}
```

#### Get My Shared Flows
```
GET /api/v1/co-trading/flows/my
```

### Follower Endpoints

#### Get All Shared Flows
```
GET /api/v1/co-trading/flows
Query params: ?tags=scalping&sortBy=performance&page=1&limit=20
```

#### Subscribe to Flow
```
POST /api/v1/co-trading/subscribe
```

**Request Body:**
```json
{
  "flowId": "flow_123",
  "vaultAddress": "0xabc...",
  "chainType": "aptos",
  "settings": {
    "autoFollow": true,
    "notifyOnSignal": true,
    "maxSlippage": 1.0
  }
}
```

#### Unsubscribe
```
DELETE /api/v1/co-trading/subscriptions/:subscriptionId
```

#### Get My Subscriptions
```
GET /api/v1/co-trading/subscriptions
```

#### Update Subscription Settings
```
PATCH /api/v1/co-trading/subscriptions/:subscriptionId/settings
```

### Signal Endpoints

#### Get Flow Signals
```
GET /api/v1/co-trading/signals/:flowId
Query params: ?limit=50
```

#### Get Signal Executions
```
GET /api/v1/co-trading/executions/:signalId
```

### Admin Endpoints (Monitor Callbacks)

#### Update Execution Status
```
POST /api/v1/co-trading/admin/executions/:executionId/status
```

**Request Body:**
```json
{
  "status": "completed",
  "result": {
    "executedAmount": 100,
    "executedPrice": 1.05,
    "txHash": "0x123...",
    "gasCost": 0.001
  }
}
```

#### Batch Execution Completion
```
POST /api/v1/co-trading/admin/executions/batch-complete
```

## Frontend Components

### FlowEdit - CoTradingPanel
Shows co-trading stats for creators in the flow editor.

**Location:** `01_weather_frontend/src/pages/flowEdit/components/CoTradingPanel.tsx`

**Features:**
- Enable/disable co-trading toggle
- Real-time follower count
- Total AUM display
- Performance metrics (return, win rate)
- Recent signals list
- Pause/resume controls

### Windmill - CoTradePage
Main discovery and subscription interface for followers.

**Location:** `01_weather_frontend/src/pages/CoTradePage.tsx`

**Features:**
- **Discover Tab:**
  - Browse shared flows
  - Search and filter (tags, performance, followers)
  - Flow cards with stats
  - Subscribe button
  
- **My Subscriptions Tab:**
  - Active subscriptions list
  - P/L tracking
  - Investment amount and current value
  - Unsubscribe option

### Components

#### FlowCard
Displays individual flow information.

**Props:**
- `flow`: SharedFlow object
- `isSubscribed`: Boolean
- `onSubscribe`: Callback function

#### SubscriptionModal
Modal for subscribing to a flow.

**Features:**
- Chain selection (Aptos, Flow EVM, BSC)
- Vault selection
- Settings configuration:
  - Auto-follow toggle
  - Notification preferences
  - Max slippage setting

#### MySubscriptions
Lists user's active subscriptions.

**Features:**
- Investment tracking
- P/L visualization
- Settings overview
- Unsubscribe action

## Security & Risk Management

### Creator Protection
- **Rate Limiting:** Limit signal publishing frequency
- **Max Followers:** Cap total followers to prevent overload
- **Verification:** Require account verification for sharing

### Follower Protection
- **Slippage Control:** User-defined max slippage
- **Investment Limits:** Min/max investment amounts
- **Emergency Pause:** Instant subscription pause
- **Risk Warnings:** Display strategy risk levels

### System Protection
- **Batch Size Limits:** Prevent system overload (batch size = 5)
- **Retry Logic:** Automatic retry with exponential backoff
- **Circuit Breaker:** Pause system if error rate exceeds threshold
- **Transaction Monitoring:** Track and alert on suspicious patterns

## Performance Optimization

### Caching Strategy
- Cache shared flow list (TTL: 5 minutes)
- Cache user subscriptions (TTL: 1 minute)
- Cache signal history (TTL: 10 minutes)

### Batch Processing
- Group signal executions by batch size (5)
- Parallel execution within batches
- Sequential batch processing to control load

### Database Indexing
```javascript
// Recommended indexes
SharedFlow: { flowId: 1, status: 1 }
FlowSubscription: { followerId: 1, status: 1 }
TradingSignal: { flowId: 1, createdAt: -1 }
SignalExecution: { signalId: 1, status: 1 }
```

## Monitoring & Analytics

### Key Metrics
- **Flow Performance:**
  - Total AUM across all shared flows
  - Average follower count
  - Signal success rate
  
- **Execution Metrics:**
  - Average execution time
  - Success/failure rate
  - Slippage statistics
  
- **User Engagement:**
  - New subscriptions per day
  - Active followers count
  - Churn rate

### Alerts
- High execution failure rate (> 10%)
- Signal processing delays (> 30s)
- Abnormal trading patterns
- System resource limits

## Future Enhancements

### Planned Features
- **Performance Fees:** Creators earn % of profits
- **Strategy Marketplace:** Buy/sell trading strategies
- **Social Features:** Comments, ratings, discussions
- **Advanced Analytics:** Detailed performance breakdowns
- **Smart Copying:** AI-optimized trade execution
- **Multi-Strategy Portfolios:** Follow multiple flows simultaneously

### Technical Improvements
- **Real-time Updates:** WebSocket for live follower stats
- **GraphQL API:** More flexible data queries
- **Event Sourcing:** Complete trade history reconstruction
- **Machine Learning:** Strategy recommendation engine

---

**Version:** 1.0.0  
**Last Updated:** 2025-01-08  
**Status:** ✅ Core Implementation Complete

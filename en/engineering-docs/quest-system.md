# Quest System

## Overview

The Quest System is TradingFlow's gamification and user engagement platform that rewards users for completing various activities within the ecosystem. It incentivizes user participation, learning, and growth through structured challenges and rewards.

## Architecture

### Database Models (MongoDB - Control)

#### Quest Model
Represents a quest or challenge in the system.

```javascript
{
  _id: ObjectId,
  questCollectionId: ObjectId,  // Reference to QuestCollection
  title: String,                // Quest title (e.g., "Complete First Trade")
  description: String,          // Detailed description
  type: String,                 // Quest type: 'activity' | 'milestone' | 'daily' | 'weekly'
  category: String,             // Category: 'trading' | 'social' | 'learning' | 'profile'
  
  // Requirements
  requirements: {
    action: String,             // Action to complete: 'flow_execution' | 'trade' | 'vault_creation'
    count: Number,              // Number of times to complete
    conditions: Object          // Additional conditions (chain, amount, etc.)
  },
  
  // Rewards
  rewards: {
    xp: Number,                 // Experience points
    credits: Number,            // Credit tokens
    badges: [String],           // Badge IDs
    items: [Object]             // Reward items
  },
  
  // Metadata
  difficulty: String,           // 'easy' | 'medium' | 'hard' | 'expert'
  estimatedTime: Number,        // Estimated time in minutes
  icon: String,                 // Icon identifier
  isActive: Boolean,            // Whether quest is currently active
  startDate: Date,              // Quest availability start
  endDate: Date,                // Quest availability end
  
  // Tracking
  totalCompletions: Number,     // Total users who completed
  createdAt: Date,
  updatedAt: Date
}
```

#### QuestCollection Model
Groups related quests together (e.g., "Beginner Journey", "Trading Master").

```javascript
{
  _id: ObjectId,
  name: String,                 // Collection name
  description: String,
  icon: String,
  
  // Collection Properties
  isSequential: Boolean,        // Must complete in order
  requiredLevel: Number,        // Min user level to access
  
  // Rewards
  completionRewards: {
    xp: Number,
    credits: Number,
    badges: [String],
    title: String               // Special title for completing collection
  },
  
  // Metadata
  displayOrder: Number,
  isActive: Boolean,
  createdAt: Date,
  updatedAt: Date
}
```

#### UserQuest Model
Tracks individual user progress on quests.

```javascript
{
  _id: ObjectId,
  userId: ObjectId,             // Reference to User
  questId: ObjectId,            // Reference to Quest
  
  // Progress Tracking
  status: String,               // 'not_started' | 'in_progress' | 'completed' | 'claimed'
  progress: {
    current: Number,            // Current progress count
    target: Number,             // Target count from quest requirements
    percentage: Number          // Calculated percentage
  },
  
  // Activity Tracking
  lastActivityDate: Date,       // Last time user made progress
  completedAt: Date,            // When quest was completed
  claimedAt: Date,              // When rewards were claimed
  
  // Metadata
  startedAt: Date,
  createdAt: Date,
  updatedAt: Date
}
```

#### UserActivityEvent Model
Logs user activities for quest progress tracking.

```javascript
{
  _id: ObjectId,
  userId: ObjectId,
  eventType: String,            // Event type: 'flow_execution' | 'trade' | 'vault_creation'
  
  // Event Data
  eventData: {
    flowId: String,
    nodeType: String,
    chain: String,
    amount: Number,
    // ... other event-specific data
  },
  
  // Quest Tracking
  processedForQuests: [ObjectId], // Quest IDs that processed this event
  
  // Metadata
  timestamp: Date,
  createdAt: Date
}
```

## Quest Types

### 1. Activity Quests
Single-action quests (e.g., "Create your first vault").

**Example:**
```json
{
  "type": "activity",
  "requirements": {
    "action": "vault_creation",
    "count": 1,
    "conditions": {}
  }
}
```

### 2. Milestone Quests
Cumulative achievement quests (e.g., "Complete 100 trades").

**Example:**
```json
{
  "type": "milestone",
  "requirements": {
    "action": "trade",
    "count": 100,
    "conditions": {}
  }
}
```

### 3. Daily Quests
Reset daily, encourage regular engagement.

**Example:**
```json
{
  "type": "daily",
  "requirements": {
    "action": "flow_execution",
    "count": 3,
    "conditions": {
      "resetPeriod": "daily"
    }
  }
}
```

### 4. Weekly Quests
Reset weekly, larger rewards for sustained engagement.

## Quest Categories

- **Trading**: Trade execution, DEX interactions, portfolio management
- **Social**: Community participation, referrals, co-trading
- **Learning**: Tutorial completion, documentation reading
- **Profile**: Account setup, wallet connection, profile completion

## API Endpoints

### Quest Management

#### Get All Quests
```
GET /api/v1/quests
```

Returns all active quests grouped by collection.

**Response:**
```json
{
  "success": true,
  "collections": [
    {
      "_id": "collection_id",
      "name": "Beginner Journey",
      "quests": [...]
    }
  ]
}
```

#### Get User Quest Progress
```
GET /api/v1/quests/user/:userId
```

Returns user's progress on all quests.

**Response:**
```json
{
  "success": true,
  "quests": [
    {
      "quest": {...},
      "progress": {
        "status": "in_progress",
        "current": 2,
        "target": 5,
        "percentage": 40
      }
    }
  ],
  "stats": {
    "totalCompleted": 12,
    "totalXP": 2400,
    "level": 5
  }
}
```

#### Claim Quest Reward
```
POST /api/v1/quests/:questId/claim
```

Claims rewards for a completed quest.

**Request Body:**
```json
{
  "userId": "user_id"
}
```

**Response:**
```json
{
  "success": true,
  "rewards": {
    "xp": 100,
    "credits": 50,
    "badges": ["first_trade"]
  },
  "newLevel": 6
}
```

### Activity Tracking

#### Log User Activity
```
POST /api/v1/quests/activity
```

Logs a user activity event for quest tracking.

**Request Body:**
```json
{
  "userId": "user_id",
  "eventType": "trade",
  "eventData": {
    "flowId": "flow_id",
    "chain": "aptos",
    "amount": 100
  }
}
```

## Quest Progress Processing

### Event Flow

```
User Action (e.g., Trade)
    ↓
Log UserActivityEvent
    ↓
Find matching quests for user
    ↓
Update UserQuest progress
    ↓
Check if quest completed
    ↓
Update quest status & notify user
```

### Progress Calculation

```javascript
// Example progress update logic
async function updateQuestProgress(userId, questId, event) {
  const userQuest = await UserQuest.findOne({ userId, questId });
  const quest = await Quest.findById(questId);
  
  // Check if event matches quest requirements
  if (event.eventType === quest.requirements.action) {
    // Increment progress
    userQuest.progress.current += 1;
    
    // Calculate percentage
    userQuest.progress.percentage = 
      (userQuest.progress.current / quest.requirements.count) * 100;
    
    // Check completion
    if (userQuest.progress.current >= quest.requirements.count) {
      userQuest.status = 'completed';
      userQuest.completedAt = new Date();
    }
    
    await userQuest.save();
  }
}
```

## Integration Points

### 1. Flow Execution
When a flow completes execution:
```javascript
// In Station node execution
await logUserActivity({
  userId: flow.userId,
  eventType: 'flow_execution',
  eventData: {
    flowId: flow._id,
    nodeCount: flow.nodes.length,
    success: true
  }
});
```

### 2. Trading Actions
When a trade is executed:
```javascript
// In SwapNode
await logUserActivity({
  userId: flow.userId,
  eventType: 'trade',
  eventData: {
    chain: this.chain,
    fromToken: this.from_token,
    toToken: this.to_token,
    amount: this.amount
  }
});
```

### 3. Vault Operations
When a vault is created:
```javascript
await logUserActivity({
  userId: user._id,
  eventType: 'vault_creation',
  eventData: {
    chain: vaultData.chain,
    vaultAddress: vaultData.address
  }
});
```

## Reward System

### Experience Points (XP)
- Used for leveling up users
- Unlocks new features and quests
- Displayed on user profile

### Credits
- Platform currency for premium features
- Can be used to purchase marketplace items
- Accumulated through quest completion

### Badges
- Visual achievements displayed on profile
- Unlock special titles and perks
- Collectible and shareable

## Frontend Integration

### QuestsPage Component
Main quest browsing interface.

**Features:**
- Quest collections display
- Progress tracking
- Reward preview
- Claim buttons

### Quest Progress Indicator
Shows real-time progress on active quests.

**Location:** User profile sidebar, dashboard

## Best Practices

### Quest Design
1. **Clear Objectives**: Make requirements explicit and measurable
2. **Progressive Difficulty**: Start easy, increase complexity
3. **Balanced Rewards**: XP and credits proportional to difficulty
4. **Time-bound**: Set appropriate time limits for timed quests

### Progress Tracking
1. **Event Deduplication**: Prevent double-counting activities
2. **Atomic Updates**: Use transactions for progress updates
3. **Caching**: Cache user progress for quick lookups
4. **Batch Processing**: Process events in batches for efficiency

### User Experience
1. **Clear Feedback**: Show progress updates immediately
2. **Notifications**: Alert users when quests are completed
3. **Discovery**: Highlight new and recommended quests
4. **Accessibility**: Ensure quests are achievable for all users

## Future Enhancements

### Planned Features
- **Quest Chains**: Sequential multi-step quests
- **Community Quests**: Cooperative quests for groups
- **Seasonal Events**: Limited-time themed quests
- **Leaderboards**: Competitive quest completion rankings
- **Quest Creator**: Allow users to create custom quests

### Technical Improvements
- **Real-time Progress**: WebSocket updates for live progress
- **Smart Recommendations**: AI-suggested quests based on user behavior
- **Analytics Dashboard**: Quest completion metrics and insights
- **A/B Testing**: Test quest designs for optimal engagement

## Monitoring and Analytics

### Key Metrics
- Quest completion rate
- Average time to complete
- Popular quest types
- User engagement trends
- Reward distribution

### Alerts
- Low completion rates (< 10%)
- Unusually high completion rates (potential exploit)
- Quest progression bottlenecks
- Reward claim failures

---

**Version:** 1.0.0  
**Last Updated:** 2025-01-08  
**Status:** ✅ Implemented

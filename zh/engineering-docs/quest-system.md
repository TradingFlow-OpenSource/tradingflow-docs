# 任务系统

## 概述

任务系统是 TradingFlow 的游戏化和用户参与平台，通过完成生态系统内的各种活动来奖励用户。它通过结构化的挑战和奖励激励用户参与、学习和成长。

## 架构

### 数据库模型（MongoDB - Control）

#### Quest 模型
代表系统中的任务或挑战。

```javascript
{
  _id: ObjectId,
  questCollectionId: ObjectId,  // 关联到 QuestCollection
  title: String,                // 任务标题（例如："完成首次交易"）
  description: String,          // 详细描述
  type: String,                 // 任务类型：'activity' | 'milestone' | 'daily' | 'weekly'
  category: String,             // 类别：'trading' | 'social' | 'learning' | 'profile'
  
  // 要求
  requirements: {
    action: String,             // 要完成的动作：'flow_execution' | 'trade' | 'vault_creation'
    count: Number,              // 完成次数
    conditions: Object          // 额外条件（链、金额等）
  },
  
  // 奖励
  rewards: {
    xp: Number,                 // 经验值
    credits: Number,            // 积分代币
    badges: [String],           // 徽章 ID
    items: [Object]             // 奖励物品
  },
  
  // 元数据
  difficulty: String,           // 'easy' | 'medium' | 'hard' | 'expert'
  estimatedTime: Number,        // 预计时间（分钟）
  icon: String,                 // 图标标识符
  isActive: Boolean,            // 任务是否当前活跃
  startDate: Date,              // 任务可用开始时间
  endDate: Date,                // 任务可用结束时间
  
  // 追踪
  totalCompletions: Number,     // 完成任务的总用户数
  createdAt: Date,
  updatedAt: Date
}
```

#### QuestCollection 模型
将相关任务组合在一起（例如："新手之旅"、"交易大师"）。

```javascript
{
  _id: ObjectId,
  name: String,                 // 集合名称
  description: String,
  icon: String,
  
  // 集合属性
  isSequential: Boolean,        // 是否必须按顺序完成
  requiredLevel: Number,        // 访问所需的最低用户等级
  
  // 奖励
  completionRewards: {
    xp: Number,
    credits: Number,
    badges: [String],
    title: String               // 完成集合获得的特殊称号
  },
  
  // 元数据
  displayOrder: Number,
  isActive: Boolean,
  createdAt: Date,
  updatedAt: Date
}
```

#### UserQuest 模型
追踪个人用户在任务上的进度。

```javascript
{
  _id: ObjectId,
  userId: ObjectId,             // 关联到 User
  questId: ObjectId,            // 关联到 Quest
  
  // 进度追踪
  status: String,               // 'not_started' | 'in_progress' | 'completed' | 'claimed'
  progress: {
    current: Number,            // 当前进度数
    target: Number,             // 来自任务要求的目标数
    percentage: Number          // 计算的百分比
  },
  
  // 活动追踪
  lastActivityDate: Date,       // 上次进展时间
  completedAt: Date,            // 任务完成时间
  claimedAt: Date,              // 奖励领取时间
  
  // 元数据
  startedAt: Date,
  createdAt: Date,
  updatedAt: Date
}
```

#### UserActivityEvent 模型
记录用户活动以进行任务进度追踪。

```javascript
{
  _id: ObjectId,
  userId: ObjectId,
  eventType: String,            // 事件类型：'flow_execution' | 'trade' | 'vault_creation'
  
  // 事件数据
  eventData: {
    flowId: String,
    nodeType: String,
    chain: String,
    amount: Number,
    // ... 其他事件特定数据
  },
  
  // 任务追踪
  processedForQuests: [ObjectId], // 处理此事件的任务 ID
  
  // 元数据
  timestamp: Date,
  createdAt: Date
}
```

## 任务类型

### 1. 活动任务
单次行动任务（例如："创建你的第一个金库"）。

**示例：**
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

### 2. 里程碑任务
累积成就任务（例如："完成 100 次交易"）。

**示例：**
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

### 3. 每日任务
每日重置，鼓励定期参与。

**示例：**
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

### 4. 每周任务
每周重置，持续参与获得更大奖励。

## 任务类别

- **交易**：交易执行、DEX 交互、投资组合管理
- **社交**：社区参与、推荐、共同投资
- **学习**：教程完成、文档阅读
- **个人资料**：账户设置、钱包连接、个人资料完善

## API 端点

### 任务管理

#### 获取所有任务
```
GET /api/v1/quests
```

返回所有活跃任务，按集合分组。

**响应：**
```json
{
  "success": true,
  "collections": [
    {
      "_id": "collection_id",
      "name": "新手之旅",
      "quests": [...]
    }
  ]
}
```

#### 获取用户任务进度
```
GET /api/v1/quests/user/:userId
```

返回用户在所有任务上的进度。

**响应：**
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

#### 领取任务奖励
```
POST /api/v1/quests/:questId/claim
```

领取已完成任务的奖励。

**请求体：**
```json
{
  "userId": "user_id"
}
```

**响应：**
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

### 活动追踪

#### 记录用户活动
```
POST /api/v1/quests/activity
```

记录用户活动事件以进行任务追踪。

**请求体：**
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

## 任务进度处理

### 事件流程

```
用户操作（例如：交易）
    ↓
记录 UserActivityEvent
    ↓
查找用户匹配的任务
    ↓
更新 UserQuest 进度
    ↓
检查任务是否完成
    ↓
更新任务状态并通知用户
```

### 进度计算

```javascript
// 进度更新逻辑示例
async function updateQuestProgress(userId, questId, event) {
  const userQuest = await UserQuest.findOne({ userId, questId });
  const quest = await Quest.findById(questId);
  
  // 检查事件是否匹配任务要求
  if (event.eventType === quest.requirements.action) {
    // 增加进度
    userQuest.progress.current += 1;
    
    // 计算百分比
    userQuest.progress.percentage = 
      (userQuest.progress.current / quest.requirements.count) * 100;
    
    // 检查完成
    if (userQuest.progress.current >= quest.requirements.count) {
      userQuest.status = 'completed';
      userQuest.completedAt = new Date();
    }
    
    await userQuest.save();
  }
}
```

## 集成点

### 1. Flow 执行
当 flow 完成执行时：
```javascript
// 在 Station 节点执行中
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

### 2. 交易操作
当执行交易时：
```javascript
// 在 SwapNode 中
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

### 3. 金库操作
当创建金库时：
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

## 奖励系统

### 经验值（XP）
- 用于用户升级
- 解锁新功能和任务
- 显示在用户个人资料上

### 积分
- 平台货币，用于高级功能
- 可用于购买市场物品
- 通过完成任务积累

### 徽章
- 个人资料上显示的视觉成就
- 解锁特殊称号和特权
- 可收集和分享

## 前端集成

### QuestsPage 组件
主要任务浏览界面。

**功能：**
- 任务集合显示
- 进度追踪
- 奖励预览
- 领取按钮

### 任务进度指示器
显示活跃任务的实时进度。

**位置：**用户个人资料侧边栏、仪表板

## 最佳实践

### 任务设计
1. **明确目标**：使要求明确且可衡量
2. **渐进难度**：从简单开始，增加复杂性
3. **平衡奖励**：XP 和积分与难度成正比
4. **限时任务**：为限时任务设置适当的时间限制

### 进度追踪
1. **事件去重**：防止活动重复计数
2. **原子更新**：使用事务进行进度更新
3. **缓存**：缓存用户进度以快速查找
4. **批量处理**：批量处理事件以提高效率

### 用户体验
1. **清晰反馈**：立即显示进度更新
2. **通知**：在任务完成时提醒用户
3. **发现**：突出显示新任务和推荐任务
4. **可访问性**：确保所有用户都能完成任务

## 未来增强

### 计划功能
- **任务链**：顺序多步任务
- **社区任务**：小组合作任务
- **季节性活动**：限时主题任务
- **排行榜**：竞争性任务完成排名
- **任务创建器**：允许用户创建自定义任务

### 技术改进
- **实时进度**：WebSocket 更新实时进度
- **智能推荐**：基于用户行为的 AI 建议任务
- **分析仪表板**：任务完成指标和洞察
- **A/B 测试**：测试任务设计以优化参与度

## 监控和分析

### 关键指标
- 任务完成率
- 平均完成时间
- 热门任务类型
- 用户参与趋势
- 奖励分配

### 告警
- 低完成率（< 10%）
- 异常高完成率（潜在漏洞）
- 任务进展瓶颈
- 奖励领取失败

---

**版本：** 1.0.0  
**最后更新：** 2025-01-08  
**状态：** ✅ 已实现

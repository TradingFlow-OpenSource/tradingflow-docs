# 共同投资系统（社交交易平台）

## 概述

共同投资系统使用户能够分享他们的交易策略（Flows），并允许其他用户（跟投者）自动复制他们的交易。这创建了一个社交交易生态系统，专家交易者可以将其策略货币化，跟投者可以从经过验证的交易方法中受益。

## 架构

### 系统组件

```
┌─────────────────────────────────────────────────────────────┐
│                        共同投资流程                           │
└─────────────────────────────────────────────────────────────┘

创作者的 Flow 执行
         ↓
    SwapNode.execute() → 发布信号到 MQ (trading_signals)
         ↓
Control: TradingSignalConsumer 接收
         ↓
    1. 保存 TradingSignal
    2. 查找活跃订阅（autoFollow=true）
    3. 创建 SignalExecution 记录
         ↓
发送批次到 Monitor → /api/v1/co-trading/execute-batch
         ↓
Monitor: executeTradingSignalBatch
         ↓
为每个跟投者执行交易（批次大小=5）
         ↓
更新 SignalExecution 状态（completed/failed）
         ↓
每 4 小时：更新跟投者统计和 AUM
```

### 数据库模型（MongoDB - Control）

#### SharedFlow 模型
代表创作者分享的用于共同投资的 flow。

```javascript
{
  _id: ObjectId,
  flowId: String,               // 数据库中 Flow 的引用
  userId: ObjectId,             // 创作者的用户 ID
  
  // 显示信息
  title: String,
  description: String,
  tags: [String],               // 例如：['scalping', 'long-term', 'aptos']
  
  // 统计数据
  stats: {
    followers: Number,          // 活跃跟投者数量
    totalAUM: Number,           // 总管理资产（USD）
    performance: {
      totalReturn: Number,      // 总回报百分比
      winRate: Number,          // 胜率百分比
      sharpeRatio: Number,      // 风险调整回报
      maxDrawdown: Number       // 最大回撤百分比
    }
  },
  
  // 设置
  settings: {
    isActive: Boolean,          // 是否接受新跟投者
    maxFollowers: Number,       // 最大跟投者数量（null = 无限制）
    minInvestment: Number       // 最低投资要求（USD）
  },
  
  // 状态
  status: String,               // 'active' | 'paused' | 'closed'
  
  // 时间戳
  createdAt: Date,
  updatedAt: Date,
  lastSignalAt: Date            // 最后交易信号时间戳
}
```

#### FlowSubscription 模型
追踪跟投者对共享 flow 的订阅。

```javascript
{
  _id: ObjectId,
  flowId: String,               // 共享 flow ID
  creatorId: ObjectId,          // 创作者的用户 ID
  followerId: ObjectId,         // 跟投者的用户 ID
  
  // 跟投者的交易设置
  followerVaultAddress: String, // 跟投者执行交易的金库
  chainType: String,            // 'aptos' | 'flow_evm' | 'bsc'
  
  // 投资追踪
  investmentAmount: Number,     // 初始投资（USD）
  currentValue: Number,         // 当前投资组合价值（USD）
  profitLoss: Number,           // 盈亏（USD）
  profitLossPercentage: Number, // 盈亏百分比
  
  // 设置
  settings: {
    autoFollow: Boolean,        // 自动执行交易
    notifyOnSignal: Boolean,    // 发送通知
    maxSlippage: Number         // 最大滑点容忍度（%）
  },
  
  // 状态
  status: String,               // 'active' | 'paused' | 'cancelled'
  
  // 时间戳
  subscribedAt: Date,
  lastExecutedAt: Date,         // 最后交易执行
  updatedAt: Date
}
```

#### TradingSignal 模型
代表创作者发布的交易信号。

```javascript
{
  _id: ObjectId,
  signalId: String,             // 唯一信号标识符
  flowId: String,               // 共享 flow ID
  creatorId: ObjectId,
  creatorVaultAddress: String,
  
  // 信号详情
  signalType: String,           // 'swap' | 'buy' | 'sell' | 'deposit' | 'withdraw'
  signalData: {
    fromToken: String,
    toToken: String,
    amountInPercentage: Number,
    amountInHumanReadable: Number,
    slippageTolerance: Number,
    chainType: String
  },
  
  // 执行状态
  executionStatus: {
    total: Number,              // 总跟投者数
    executed: Number,           // 成功执行
    failed: Number,             // 失败执行
    pending: Number             // 待执行
  },
  
  // 元数据
  nodeId: String,               // 源节点 ID
  nodeType: String,             // 源节点类型
  
  // 时间戳
  createdAt: Date,
  processedAt: Date             // 信号处理时间
}
```

#### SignalExecution 模型
追踪单个跟投者的信号执行。

```javascript
{
  _id: ObjectId,
  executionId: String,
  signalId: ObjectId,           // 关联到 TradingSignal
  subscriptionId: ObjectId,     // 关联到 FlowSubscription
  
  // 跟投者信息
  followerId: ObjectId,
  followerVaultAddress: String,
  
  // 执行详情
  status: String,               // 'pending' | 'executing' | 'completed' | 'failed'
  executedAmount: Number,       // 实际执行金额
  executedPrice: Number,        // 执行价格
  gasCost: Number,              // 支付的 Gas 费
  slippage: Number,             // 实际滑点
  
  // 交易详情
  txHash: String,
  blockNumber: Number,
  
  // 错误信息
  error: String,                // 失败时的错误消息
  
  // 时间戳
  createdAt: Date,
  executedAt: Date,
  completedAt: Date
}
```

## 服务层

### Station（Python）- 信号发布

#### TradingSignalPublisher
将交易信号发布到 RabbitMQ。

**位置：** `03_weather_station/mq/trading_signal_publisher.py`

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
    发布交易信号到 MQ
    
    Args:
        flow_id: Flow ID
        user_id: 创作者用户 ID
        vault_address: 创作者的金库地址
        signal_type: 信号类型（swap, buy, sell）
        signal_data: 信号参数
        node_id: 源节点 ID
        node_type: 源节点类型
    
    Returns:
        bool: 成功状态
    """
```

**在 SwapNode 中的集成：**
```python
async def _publish_trading_signal(self, trade_receipt: Dict):
    """发布交易信号到 MQ 用于共同投资系统"""
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
            await self.persist_log(f"交易信号已发布: flow={self.flow_id}", "INFO")
    except Exception as e:
        await self.persist_log(f"发布信号错误: {str(e)}", "WARNING")
```

### Control（Node.js）- 信号处理

#### TradingSignalConsumer
从 RabbitMQ 消费交易信号并创建执行任务。

**位置：** `02_weather_control/src/services/TradingSignalConsumer.js`

**核心方法：**

```javascript
class TradingSignalConsumer {
  async init() {
    // 初始化 MQ 消费者
    // 监听 'trading_signals' exchange
  }
  
  async handleTradingSignal(message) {
    // 1. 从消息解析信号
    // 2. 保存 TradingSignal 到数据库
    // 3. 查找活跃订阅（autoFollow=true）
    // 4. 为每个跟投者创建 SignalExecution 记录
    // 5. 发送批次到 Monitor 执行
  }
  
  async createExecutionBatch(signal, subscriptions) {
    // 创建 SignalExecution 记录
    // 按批次大小分组（默认：5）
    // 发送到 Monitor API
  }
}
```

#### CoTradingService
处理共同投资业务逻辑。

**位置：** `02_weather_control/src/services/CoTradingService.js`

**核心方法：**

```javascript
class CoTradingService {
  // Flow 管理
  async createOrUpdateSharedFlow(flowId, userId, data)
  async toggleSharedFlowStatus(flowId, action)
  async getSharedFlows(filters, pagination)
  async getMySharedFlows(userId, pagination)
  
  // 订阅管理
  async subscribeToFlow(userId, flowId, vaultAddress, chainType, settings)
  async unsubscribeFromFlow(subscriptionId)
  async getUserSubscriptions(userId)
  async updateSubscriptionSettings(subscriptionId, settings)
  
  // 信号和执行
  async getFlowSignals(flowId, limit)
  async getSignalExecutions(signalId)
  async updateExecutionStatus(executionId, status, result)
}
```

### Monitor（TypeScript）- 交易执行

#### executeTradingSignals 任务
批量执行跟投者的交易。

**位置：** `04_weather_monitor/src/tasks/executeTradingSignals.ts`

```typescript
export async function executeTradingSignalBatch(
  signalId: string,
  executions: SignalExecution[]
): Promise<void> {
  const batchSize = 5; // 一次处理 5 个
  
  for (let i = 0; i < executions.length; i += batchSize) {
    const batch = executions.slice(i, i + batchSize);
    
    await Promise.all(batch.map(async (execution) => {
      try {
        // 1. 获取跟投者的金库服务
        // 2. 使用信号参数执行交易
        // 3. 更新执行状态
        await executeTradeForFollower(execution);
      } catch (error) {
        // 更新执行为失败
        await updateExecutionStatus(execution._id, 'failed', error.message);
      }
    }));
  }
}
```

#### updateFollowerStats 任务
定期更新跟投者统计和 AUM（每 4 小时）。

**位置：** `04_weather_monitor/src/tasks/updateFollowerStats.ts`

```typescript
export async function updateAllFollowerStats(): Promise<void> {
  // 1. 获取所有活跃订阅
  const subscriptions = await FlowSubscription.find({ status: 'active' });
  
  for (const subscription of subscriptions) {
    try {
      // 2. 查询当前金库价值
      const vaultValue = await getVaultValue(
        subscription.followerVaultAddress,
        subscription.chainType
      );
      
      // 3. 计算盈亏
      const profitLoss = vaultValue - subscription.investmentAmount;
      const profitLossPercentage = (profitLoss / subscription.investmentAmount) * 100;
      
      // 4. 更新订阅
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
      logger.error(`更新订阅统计失败 ${subscription._id}`, error);
    }
  }
  
  // 5. 更新 SharedFlow AUM
  await updateSharedFlowAUM();
}
```

## API 端点

### 创作者端点

#### 创建/更新共享 Flow
```
POST /api/v1/co-trading/flows
```

**请求体：**
```json
{
  "flowId": "flow_123",
  "title": "剥头皮策略",
  "description": "Aptos 上的高频剥头皮",
  "tags": ["scalping", "aptos"],
  "maxFollowers": 100,
  "minInvestment": 1000
}
```

#### 切换 Flow 状态
```
POST /api/v1/co-trading/flows/:flowId/toggle
```

**请求体：**
```json
{
  "action": "pause" // 或 "resume"
}
```

#### 获取我的共享 Flows
```
GET /api/v1/co-trading/flows/my
```

### 跟投者端点

#### 获取所有共享 Flows
```
GET /api/v1/co-trading/flows
查询参数: ?tags=scalping&sortBy=performance&page=1&limit=20
```

#### 订阅 Flow
```
POST /api/v1/co-trading/subscribe
```

**请求体：**
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

#### 取消订阅
```
DELETE /api/v1/co-trading/subscriptions/:subscriptionId
```

#### 获取我的订阅
```
GET /api/v1/co-trading/subscriptions
```

#### 更新订阅设置
```
PATCH /api/v1/co-trading/subscriptions/:subscriptionId/settings
```

### 信号端点

#### 获取 Flow 信号
```
GET /api/v1/co-trading/signals/:flowId
查询参数: ?limit=50
```

#### 获取信号执行
```
GET /api/v1/co-trading/executions/:signalId
```

### 管理端点（Monitor 回调）

#### 更新执行状态
```
POST /api/v1/co-trading/admin/executions/:executionId/status
```

**请求体：**
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

#### 批量执行完成
```
POST /api/v1/co-trading/admin/executions/batch-complete
```

## 前端组件

### FlowEdit - CoTradingPanel
在流程编辑器中为创作者显示共同投资统计。

**位置：** `01_weather_frontend/src/pages/flowEdit/components/CoTradingPanel.tsx`

**功能：**
- 启用/禁用共同投资切换
- 实时跟投者计数
- 总 AUM 显示
- 性能指标（回报、胜率）
- 最近信号列表
- 暂停/恢复控制

### Windmill - CoTradePage
跟投者的主要发现和订阅界面。

**位置：** `01_weather_frontend/src/pages/CoTradePage.tsx`

**功能：**
- **发现标签：**
  - 浏览共享 flows
  - 搜索和筛选（标签、性能、跟投者）
  - 带统计的 Flow 卡片
  - 订阅按钮
  
- **我的订阅标签：**
  - 活跃订阅列表
  - 盈亏追踪
  - 投资金额和当前价值
  - 取消订阅选项

### 组件

#### FlowCard
显示单个 flow 信息。

**Props：**
- `flow`：SharedFlow 对象
- `isSubscribed`：布尔值
- `onSubscribe`：回调函数

#### SubscriptionModal
订阅 flow 的模态框。

**功能：**
- 链选择（Aptos、Flow EVM、BSC）
- 金库选择
- 设置配置：
  - 自动跟随切换
  - 通知偏好
  - 最大滑点设置

#### MySubscriptions
列出用户的活跃订阅。

**功能：**
- 投资追踪
- 盈亏可视化
- 设置概览
- 取消订阅操作

## 安全和风险管理

### 创作者保护
- **频率限制：** 限制信号发布频率
- **最大跟投者：** 限制总跟投者以防止过载
- **验证：** 要求账户验证才能分享

### 跟投者保护
- **滑点控制：** 用户定义的最大滑点
- **投资限制：** 最小/最大投资金额
- **紧急暂停：** 即时订阅暂停
- **风险警告：** 显示策略风险等级

### 系统保护
- **批次大小限制：** 防止系统过载（批次大小 = 5）
- **重试逻辑：** 指数退避自动重试
- **熔断器：** 错误率超过阈值时暂停系统
- **交易监控：** 追踪和警报可疑模式

## 性能优化

### 缓存策略
- 缓存共享 flow 列表（TTL：5 分钟）
- 缓存用户订阅（TTL：1 分钟）
- 缓存信号历史（TTL：10 分钟）

### 批量处理
- 按批次大小分组信号执行（5）
- 批次内并行执行
- 顺序批次处理以控制负载

### 数据库索引
```javascript
// 推荐索引
SharedFlow: { flowId: 1, status: 1 }
FlowSubscription: { followerId: 1, status: 1 }
TradingSignal: { flowId: 1, createdAt: -1 }
SignalExecution: { signalId: 1, status: 1 }
```

## 监控和分析

### 关键指标
- **Flow 性能：**
  - 所有共享 flows 的总 AUM
  - 平均跟投者数
  - 信号成功率
  
- **执行指标：**
  - 平均执行时间
  - 成功/失败率
  - 滑点统计
  
- **用户参与：**
  - 每日新订阅
  - 活跃跟投者数
  - 流失率

### 告警
- 高执行失败率（> 10%）
- 信号处理延迟（> 30s）
- 异常交易模式
- 系统资源限制

## 未来增强

### 计划功能
- **性能费用：** 创作者从利润中赚取 %
- **策略市场：** 买卖交易策略
- **社交功能：** 评论、评分、讨论
- **高级分析：** 详细的性能分解
- **智能复制：** AI 优化的交易执行
- **多策略投资组合：** 同时跟随多个 flows

### 技术改进
- **实时更新：** WebSocket 实时跟投者统计
- **GraphQL API：** 更灵活的数据查询
- **事件溯源：** 完整的交易历史重建
- **机器学习：** 策略推荐引擎

---

**版本：** 1.0.0  
**最后更新：** 2025-01-08  
**状态：** ✅ 核心实现完成

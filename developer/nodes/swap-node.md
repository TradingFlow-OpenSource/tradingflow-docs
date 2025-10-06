# Swap Node

**节点类型：** `swap_node`  
**版本：** 0.0.2  
**分类：** 交易执行节点  
**文件位置：** `tradingflow/station/nodes/swap_node.py`

---

## 节点概述

Swap Node 是一个多链交换交易执行节点，支持在 Aptos 和 Flow EVM 链上执行代币交换操作。节点通过 Vault 合约执行交易，支持百分比和固定金额两种方式。

### 主要功能

- 支持多链（Aptos、Flow EVM）代币交换
- 灵活的金额输入方式（百分比/固定金额）
- 自动滑点保护
- 完整的交易收据返回
- 链特定的价格估算

---

## 输入输出

### 输入参数

| 参数名 | 类型 | 默认值 | 必需 | 说明 |
|--------|------|--------|------|------|
| `from_token` | string | - | 是 | 源代币符号（如 "USDT"） |
| `to_token` | string | - | 是 | 目标代币符号（如 "BTC"） |
| `chain` | string | `"aptos"` | 是 | 区块链网络（"aptos" 或 "flow_evm"） |
| `vault_address` | string | - | 是 | Vault 合约地址 |
| `amount_in_percentage` | number | - | 否 | 交易金额百分比（0-100） |
| `amount_in_human_readable` | number | - | 否 | 人类可读的固定金额 |
| `slippery_tolerance` | number | `1.0` | 否 | 滑点容忍度（百分比） |

**金额输入说明：**
- `amount_in_percentage` 和 `amount_in_human_readable` **二选一**
- 优先使用 `amount_in_percentage`（如果两者都提供）
- 百分比基于 Vault 中的代币余额计算

### 输入句柄

| 句柄名 | 类型 | 说明 |
|--------|------|------|
| `from_token` | string | 源代币符号输入 |
| `to_token` | string | 目标代币符号输入 |
| `chain` | string | 区块链网络输入 |
| `vault_address` | string | Vault 地址输入 |
| `amount_in_percentage` | number | 百分比金额输入 |
| `amount_in_human_readable` | number | 固定金额输入 |
| `slippery_tolerance` | number | 滑点容忍度输入 |

### 输出句柄

**trade_receipt** (SignalType.DEX_TRADE_RECEIPT)

输出交易收据，包含完整的交易信息。

**Aptos 交易收据格式：**
```json
{
  "chain": "aptos",
  "transaction_hash": "0x1234...",
  "from_token": "USDT",
  "to_token": "BTC",
  "amount_in": "1000000000",
  "amount_out": "5000000",
  "slippage_tolerance": 1.0,
  "sqrt_price_limit": "79228162514264337593543950336",
  "timestamp": "2025-10-06T10:00:00Z"
}
```

**Flow EVM 交易收据格式：**
```json
{
  "chain": "flow_evm",
  "transaction_hash": "0xabcd...",
  "from_token": "0x...",
  "to_token": "0x...",
  "amount_in": "1000000000000000000",
  "amount_out": "50000000000000000",
  "gas_used": "150000",
  "gas_price": "1000000000",
  "timestamp": "2025-10-06T10:00:00Z"
}
```

---

## 执行流程

### 完整执行流程

```
1. 参数验证
    ├─> 验证必需参数存在
    ├─> 验证链类型支持
    └─> 验证金额参数
    ↓
2. 选择链服务
    ├─> Aptos: AptosVaultService
    └─> Flow EVM: FlowEvmVaultService
    ↓
3. 代币地址解析
    ├─> Aptos: 通过符号查找 token_metadata
    └─> Flow EVM: 使用占位符地址
    ↓
4. 金额计算
    ├─> 百分比模式: 从 Vault 余额计算
    └─> 固定金额模式: 转换为最小单位
    ↓
5. 价格估算（链特定）
    ├─> Aptos: 计算 sqrt_price_limit
    └─> Flow EVM: 调用价格查询合约
    ↓
6. 执行交换
    └─> 调用 Vault 服务 execute_swap()
    ↓
7. 构建交易收据
    └─> 包含链特定信息
    ↓
8. 发送输出信号
    └─> 发送交易收据到下游节点
```

### 核心代码实现

```python
async def execute(self) -> bool:
    """执行交换交易"""
    try:
        # 1. 参数验证
        if not all([self.from_token, self.to_token, self.vault_address]):
            raise ValueError("Missing required parameters")
        
        # 2. 选择服务
        if self.chain == "aptos":
            service = AptosVaultService.get_instance()
        elif self.chain == "flow_evm":
            service = FlowEvmVaultService.get_instance()
        else:
            raise ValueError(f"Unsupported chain: {self.chain}")
        
        # 3. 代币地址解析
        from_token_address = self._resolve_token_address(self.from_token)
        to_token_address = self._resolve_token_address(self.to_token)
        
        # 4. 金额计算
        amount_in = await self._calculate_amount_in(service)
        
        # 5. 价格估算
        sqrt_price_limit = await self._estimate_price_limit(
            service, from_token_address, to_token_address, amount_in
        )
        
        # 6. 执行交换
        receipt = await service.execute_swap(
            vault_address=self.vault_address,
            from_token=from_token_address,
            to_token=to_token_address,
            amount_in=amount_in,
            sqrt_price_limit=sqrt_price_limit,
            slippage_tolerance=self.slippery_tolerance
        )
        
        # 7. 发送交易收据
        await self.send_signal(
            source_handle="trade_receipt",
            signal_type=SignalType.DEX_TRADE_RECEIPT,
            payload=receipt
        )
        
        await self.set_status(NodeStatus.COMPLETED)
        return True
        
    except Exception as e:
        self.logger.error(f"Swap execution failed: {e}")
        await self.set_status(NodeStatus.FAILED, str(e))
        return False
```

---

## 使用示例

### 示例 1：Aptos 链百分比交换

```json
{
  "id": "swap_25_percent",
  "type": "swap_node",
  "config": {
    "chain": "aptos",
    "vault_address": "0x123...",
    "from_token": "USDT",
    "to_token": "APT",
    "amount_in_percentage": 25.0,
    "slippery_tolerance": 1.0
  }
}
```

**说明：** 将 Vault 中 25% 的 USDT 交换为 APT

### 示例 2：Flow EVM 链固定金额交换

```json
{
  "id": "swap_fixed_amount",
  "type": "swap_node",
  "config": {
    "chain": "flow_evm",
    "vault_address": "0xabc...",
    "from_token": "USDC",
    "to_token": "FLOW",
    "amount_in_human_readable": 100.0,
    "slippery_tolerance": 2.0
  }
}
```

**说明：** 使用 100 USDC 交换 FLOW

### 示例 3：AI 驱动的动态交换

```json
{
  "nodes": [
    {
      "id": "price_analyzer",
      "type": "ai_model_node",
      "config": {
        "model": "gpt-4",
        "prompt": "分析价格趋势，决定交换金额"
      }
    },
    {
      "id": "swap_executor",
      "type": "swap_node",
      "config": {
        "chain": "aptos",
        "vault_address": "0x123...",
        "from_token": "USDT",
        "to_token": "BTC"
      }
    }
  ],
  "edges": [
    {
      "source": "price_analyzer",
      "sourceHandle": "ai_response",
      "target": "swap_executor",
      "targetHandle": "amount_in_percentage"
    }
  ]
}
```

---

## 链特定说明

### Aptos 链

**代币地址解析：**
```python
TOKEN_METADATA = {
    "APT": "0x1::aptos_coin::AptosCoin",
    "USDT": "0x...",
    "USDC": "0x...",
}
```

**价格估算：**
- 使用 Uniswap V3 风格的 sqrt_price_limit
- 计算公式：基于当前价格和滑点容忍度

**最小单位：**
- 大多数代币：8 位小数
- APT：8 位小数

### Flow EVM 链

**代币地址：**
- 使用标准 EVM 地址格式
- 支持 ERC-20 代币

**价格查询：**
- 调用 Uniswap V2/V3 兼容合约
- 使用 `getAmountsOut` 查询预期输出

**Gas 估算：**
- 自动估算 Gas Limit
- 支持 EIP-1559 (Type 2) 交易

---

## 错误处理

### 常见错误

| 错误类型 | 原因 | 解决方案 |
|---------|------|----------|
| `Insufficient Balance` | Vault 余额不足 | 降低交换金额或充值 |
| `Slippage Exceeded` | 实际滑点超过容忍度 | 增加 `slippery_tolerance` |
| `Token Not Found` | 代币符号无效 | 检查代币符号拼写 |
| `Invalid Vault Address` | Vault 地址不存在 | 确认 Vault 地址正确 |
| `Network Error` | 网络连接问题 | 检查 RPC 端点配置 |

### 安全机制

1. **滑点保护**：自动设置价格限制
2. **金额验证**：检查金额在合理范围内
3. **余额检查**：执行前验证 Vault 余额
4. **重试机制**：网络错误自动重试

---

## 性能考虑

### Gas 优化

**Aptos：**
- 平均 Gas 费用：~0.001 APT
- 优化：批量交易可降低平均成本

**Flow EVM：**
- 平均 Gas 费用：~0.0001 FLOW
- 优化：使用 EIP-1559 动态 Gas 价格

### 执行时间

- Aptos：1-3 秒（取决于网络拥堵）
- Flow EVM：2-5 秒（包括确认时间）

### 并发限制

- 同一 Vault 的交易必须串行执行
- 不同 Vault 可并行执行

---

## 配置要求

### 环境变量

```bash
# Aptos 配置
APTOS_RPC_URL=https://fullnode.mainnet.aptoslabs.com/v1
APTOS_PRIVATE_KEY=your_private_key

# Flow EVM 配置
FLOW_EVM_RPC_URL=https://testnet.evm.nodes.onflow.org
FLOW_EVM_PRIVATE_KEY=your_private_key

# Vault 配置
VAULT_FACTORY_ADDRESS=0x...
```

### 依赖服务

- **Aptos Vault Service**：处理 Aptos 链交易
- **Flow EVM Vault Service**：处理 Flow EVM 链交易
- **RPC 节点**：区块链网络访问

---

## 实例节点

Swap Node 有两个特化的实例节点：

### BuyNode

专门用于买入操作的节点。

```python
@register_node_type("buy_node")
class BuyNode(SwapNode):
    """买入节点：自动映射 from_token=base_token, to_token=buy_token"""
```

### SellNode

专门用于卖出操作的节点。

```python
@register_node_type("sell_node")
class SellNode(SwapNode):
    """卖出节点：自动映射 from_token=sell_token, to_token=base_token"""
```

---

## 相关文档

- [节点开发指南](../weather-station-node-execution.md#开发新节点)
- [Vault Node](vault-node.md)
- [AI Model Node](ai-model-node.md)
- [Aptos Vault Service 文档](../../aptos-vault-service.md)

---

**维护者：** TradingFlow 开发团队  
**最后更新：** 2025-10-06

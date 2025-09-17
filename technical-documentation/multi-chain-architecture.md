# Multi-Chain Architecture

TradingFlow's multi-chain architecture enables seamless trading across multiple blockchain networks through a unified interface. This document explains the technical implementation and design decisions.

## Architecture Overview

### High-Level System Design

```
Frontend (React/Vue) ─────┐
                          ├─→ Control Layer ─→ Cluster Services ─→ Multi-Chain Adapters
WebSocket/REST APIs ──────┘         ↓                ↓                    ↓
                              Load Balancer    Flow Execution      Chain-Specific Logic
                                    ↓                ↓                    ↓
                            Authentication    Station Workers        Vault Contracts
```

### Core Components

#### 1. Control Layer (`weather_control`)
**Purpose**: API gateway and request routing
**Technology**: Node.js, Express.js
**Responsibilities**:
- User authentication and authorization
- API request routing and load balancing
- Rate limiting and DDoS protection
- WebSocket connection management
- Inter-service communication

#### 2. Cluster Services (`weather_cluster`)
**Purpose**: Core business logic and workflow execution
**Technology**: Python, FastAPI, Celery
**Components**:
- **Station**: Workflow execution engine
- **Bank**: Asset management and accounting (being phased out)
- **Monitor**: Real-time monitoring and alerting
- **Depot**: Shared utilities and database access

#### 3. Vault Layer (`weather_vault`)
**Purpose**: Multi-chain smart contract management
**Technology**: Solidity, Move, TypeScript
**Features**:
- Chain-agnostic vault interfaces
- Secure asset custody
- Cross-chain compatibility
- Automated rebalancing

#### 4. Infrastructure (`weather_infra`)
**Purpose**: DevOps and deployment automation
**Technology**: Terraform, Docker, Kubernetes
**Services**:
- Multi-cloud deployment
- Environment management
- Monitoring and logging
- Backup and disaster recovery

## Multi-Chain Abstraction Layer

### Chain Adapter Pattern

Each blockchain integration follows a standardized adapter pattern:

```typescript
interface ChainAdapter {
  // Network Information
  getNetworkInfo(): NetworkInfo;
  getChainId(): number;
  
  // Account Management
  createVault(config: VaultConfig): Promise<VaultAddress>;
  getVaultInfo(address: string): Promise<VaultInfo>;
  
  // Token Operations
  getTokenBalance(token: string, address: string): Promise<Balance>;
  getTokenInfo(token: string): Promise<TokenInfo>;
  
  // Trading Operations
  executeSwap(params: SwapParams): Promise<TransactionReceipt>;
  estimateGas(operation: Operation): Promise<GasEstimate>;
  
  // Event Monitoring
  subscribeToEvents(filters: EventFilters): EventSubscription;
  getTransactionHistory(address: string): Promise<Transaction[]>;
}
```

### Supported Networks

#### Aptos Integration
**Technology**: Move programming language, REST API
**Features**:
- High-throughput execution (160,000+ TPS theoretical)
- Parallel transaction processing
- Move-based safety guarantees
- Resource-oriented programming model

**Implementation Details**:
```typescript
class AptosAdapter implements ChainAdapter {
  private client: AptosClient;
  private faucetClient: FaucetClient;
  
  async executeSwap(params: SwapParams): Promise<TransactionReceipt> {
    const payload = {
      type: "entry_function_payload",
      function: "0x123::hyperion::swap",
      arguments: [
        params.fromToken,
        params.toToken,
        params.amountIn.toString(),
        params.minAmountOut.toString()
      ],
      type_arguments: [params.fromTokenType, params.toTokenType]
    };
    
    const txnRequest = await this.client.generateTransaction(
      params.vaultAddress,
      payload
    );
    
    return await this.client.submitTransaction(txnRequest);
  }
}
```

#### Flow EVM Integration
**Technology**: EVM-compatible with Flow features
**Features**:
- Ethereum compatibility
- Enhanced user experience (human-readable addresses)
- Lower gas costs than Ethereum mainnet
- Cadence resource model integration

**Smart Contract Architecture**:
```solidity
// Flow EVM Vault Contract
contract FlowEvmVault is IVault, Ownable, ReentrancyGuard {
    using SafeERC20 for IERC20;
    
    mapping(address => uint256) public balances;
    mapping(address => bool) public allowedTokens;
    
    event Deposit(address indexed user, address indexed token, uint256 amount);
    event Withdraw(address indexed user, address indexed token, uint256 amount);
    event Swap(address indexed user, address tokenIn, address tokenOut, uint256 amountIn, uint256 amountOut);
    
    function deposit(address token, uint256 amount) external nonReentrant {
        require(allowedTokens[token], "Token not allowed");
        IERC20(token).safeTransferFrom(msg.sender, address(this), amount);
        balances[token] += amount;
        emit Deposit(msg.sender, token, amount);
    }
    
    function executeSwap(
        address tokenIn,
        address tokenOut,
        uint256 amountIn,
        uint256 minAmountOut,
        address dexRouter
    ) external onlyOwner returns (uint256 amountOut) {
        // Implementation for PunchSwap V2 integration
        require(balances[tokenIn] >= amountIn, "Insufficient balance");
        
        IERC20(tokenIn).approve(dexRouter, amountIn);
        
        address[] memory path = new address[](2);
        path[0] = tokenIn;
        path[1] = tokenOut;
        
        uint256[] memory amounts = IPunchSwapRouter(dexRouter)
            .swapExactTokensForTokens(
                amountIn,
                minAmountOut,
                path,
                address(this),
                block.timestamp + 300
            );
            
        amountOut = amounts[1];
        balances[tokenIn] -= amountIn;
        balances[tokenOut] += amountOut;
        
        emit Swap(msg.sender, tokenIn, tokenOut, amountIn, amountOut);
        return amountOut;
    }
}
```

#### BSC Integration (Planned)
**Technology**: Binance Smart Chain, EVM-compatible
**Features**:
- Low transaction costs
- High throughput (3-second block times)
- Large DeFi ecosystem
- PancakeSwap integration

#### Ethereum Mainnet Integration (Planned)
**Technology**: Original Ethereum network
**Features**:
- Largest DeFi ecosystem
- Maximum liquidity
- Institutional adoption
- Layer 2 scaling solutions

## Cross-Chain Communication

### Message Passing Architecture

```typescript
interface CrossChainMessage {
  id: string;
  sourceChain: string;
  targetChain: string;
  messageType: 'SWAP' | 'TRANSFER' | 'QUERY';
  payload: any;
  timestamp: number;
  signature: string;
}

class CrossChainBridge {
  async sendMessage(message: CrossChainMessage): Promise<string> {
    // Validate message format and signature
    await this.validateMessage(message);
    
    // Route to appropriate chain adapter
    const adapter = this.getChainAdapter(message.targetChain);
    
    // Execute cross-chain operation
    return await adapter.processMessage(message);
  }
  
  async validateMessage(message: CrossChainMessage): Promise<boolean> {
    // Verify signature
    const isValid = await this.cryptoService.verifySignature(
      message.payload,
      message.signature
    );
    
    // Check replay protection
    const isUnique = await this.messageStore.isUniqueMessage(message.id);
    
    return isValid && isUnique;
  }
}
```

### State Synchronization

**Challenge**: Maintaining consistent state across multiple chains
**Solution**: Event-driven synchronization with conflict resolution

```typescript
class StateSync {
  private eventSubscriptions: Map<string, EventSubscription> = new Map();
  
  async synchronizeVaultState(vaultAddress: string): Promise<VaultState> {
    const chains = await this.getVaultChains(vaultAddress);
    const states = await Promise.all(
      chains.map(chain => this.getChainState(chain, vaultAddress))
    );
    
    // Resolve conflicts using timestamp-based ordering
    return this.resolveStateConflicts(states);
  }
  
  private resolveStateConflicts(states: ChainState[]): VaultState {
    // Use latest timestamp as source of truth
    const latestState = states.reduce((latest, current) => 
      current.timestamp > latest.timestamp ? current : latest
    );
    
    // Aggregate balances across chains
    const totalBalances = this.aggregateBalances(states);
    
    return {
      ...latestState,
      totalBalances,
      lastSyncTime: Date.now()
    };
  }
}
```

## Gas Optimization Strategies

### Dynamic Gas Management

```typescript
class GasOptimizer {
  async optimizeTransaction(
    chain: string, 
    operation: Operation
  ): Promise<GasStrategy> {
    const currentGasPrice = await this.getGasPrice(chain);
    const networkCongestion = await this.getNetworkCongestion(chain);
    
    // Choose optimal execution strategy
    if (networkCongestion < 0.3) {
      return {
        strategy: 'immediate',
        gasPrice: currentGasPrice * 1.1,
        priority: 'normal'
      };
    } else if (operation.urgency === 'low') {
      return {
        strategy: 'delayed',
        gasPrice: currentGasPrice * 0.8,
        priority: 'low',
        delayUntil: this.predictLowCongestion()
      };
    } else {
      return {
        strategy: 'premium',
        gasPrice: currentGasPrice * 1.5,
        priority: 'high'
      };
    }
  }
  
  async batchOperations(operations: Operation[]): Promise<BatchedTransaction> {
    // Group compatible operations
    const batches = this.groupOperations(operations);
    
    // Estimate gas savings
    const individualGas = operations.reduce((sum, op) => 
      sum + op.estimatedGas, 0
    );
    const batchedGas = this.estimateBatchGas(batches);
    
    return {
      batches,
      gasSavings: individualGas - batchedGas,
      executionOrder: this.optimizeExecutionOrder(batches)
    };
  }
}
```

### Chain-Specific Optimizations

#### Aptos Optimizations
- **Parallel Execution**: Leverage Aptos's parallel transaction processing
- **Resource Batching**: Batch resource operations efficiently
- **Move VM**: Optimize for Move's resource-oriented model

#### Flow EVM Optimizations
- **Gas Price Prediction**: Use Flow's more predictable gas pricing
- **Account Abstraction**: Leverage Flow's enhanced account model
- **Resource Optimization**: Efficient use of Flow's resource system

## Security Architecture

### Multi-Layer Security Model

```typescript
interface SecurityPolicy {
  authentication: AuthConfig;
  authorization: AuthzConfig;
  encryption: EncryptionConfig;
  auditLogging: AuditConfig;
  rateLimit: RateLimitConfig;
}

class SecurityManager {
  async validateTransaction(
    transaction: Transaction,
    context: SecurityContext
  ): Promise<ValidationResult> {
    const checks = [
      this.validateAuthentication(context.user),
      this.validateAuthorization(context.user, transaction),
      this.validateTransactionLimits(transaction),
      this.validateRiskParameters(transaction),
      this.checkBlacklist(transaction.addresses),
      this.validateSignature(transaction)
    ];
    
    const results = await Promise.all(checks);
    const failed = results.filter(r => !r.valid);
    
    if (failed.length > 0) {
      await this.auditLogger.logSecurityViolation({
        user: context.user,
        transaction,
        violations: failed
      });
      
      return {
        valid: false,
        violations: failed,
        risk_score: this.calculateRiskScore(failed)
      };
    }
    
    return { valid: true, risk_score: 0 };
  }
}
```

### Vault Security Features

#### Access Control
```solidity
// Multi-signature vault implementation
contract MultiSigVault {
    mapping(address => bool) public owners;
    mapping(uint256 => mapping(address => bool)) public confirmations;
    uint256 public required;
    uint256 public transactionCount;
    
    modifier onlyOwner() {
        require(owners[msg.sender], "Not owner");
        _;
    }
    
    modifier confirmed(uint256 transactionId, address owner) {
        require(confirmations[transactionId][owner], "Not confirmed");
        _;
    }
    
    function executeTransaction(uint256 transactionId)
        public
        onlyOwner
        confirmed(transactionId, msg.sender)
    {
        require(isConfirmed(transactionId), "Not enough confirmations");
        Transaction storage txn = transactions[transactionId];
        require(!txn.executed, "Already executed");
        
        txn.executed = true;
        (bool success,) = txn.destination.call{value: txn.value}(txn.data);
        require(success, "Transaction failed");
    }
}
```

#### Emergency Controls
- **Circuit Breakers**: Automatic trading halts during extreme conditions
- **Emergency Withdrawal**: Direct asset recovery mechanisms
- **Admin Controls**: Privileged operations for emergency situations
- **Time Locks**: Delays on critical parameter changes

## Performance Monitoring

### Real-Time Metrics

```typescript
interface PerformanceMetrics {
  chain: string;
  transactions_per_second: number;
  average_confirmation_time: number;
  gas_price_trend: number[];
  network_congestion: number;
  vault_operations_success_rate: number;
  cross_chain_latency: number;
}

class PerformanceMonitor {
  private metrics: Map<string, PerformanceMetrics> = new Map();
  
  async collectMetrics(): Promise<void> {
    const chains = await this.getActiveChains();
    
    for (const chain of chains) {
      const metrics = await this.getChainMetrics(chain);
      this.metrics.set(chain, metrics);
      
      // Alert on performance degradation
      if (metrics.network_congestion > 0.8) {
        await this.alertManager.sendAlert({
          type: 'PERFORMANCE_DEGRADATION',
          chain,
          severity: 'HIGH',
          metrics
        });
      }
    }
  }
  
  async getOptimalChainForOperation(
    operation: Operation
  ): Promise<string> {
    const candidates = await this.getCompatibleChains(operation);
    
    let bestChain = candidates[0];
    let bestScore = 0;
    
    for (const chain of candidates) {
      const metrics = this.metrics.get(chain);
      const score = this.calculatePerformanceScore(metrics, operation);
      
      if (score > bestScore) {
        bestScore = score;
        bestChain = chain;
      }
    }
    
    return bestChain;
  }
}
```

### Health Monitoring

```typescript
class HealthMonitor {
  async performHealthCheck(): Promise<HealthStatus> {
    const checks = {
      database: await this.checkDatabase(),
      redis: await this.checkRedis(),
      blockchains: await this.checkBlockchains(),
      apis: await this.checkExternalAPIs(),
      vaults: await this.checkVaultStatus()
    };
    
    const overallHealth = Object.values(checks).every(check => check.healthy);
    
    return {
      healthy: overallHealth,
      checks,
      timestamp: Date.now(),
      uptime: process.uptime()
    };
  }
  
  private async checkBlockchains(): Promise<HealthCheck> {
    const chains = ['aptos', 'flow_evm'];
    const results = await Promise.all(
      chains.map(async chain => {
        try {
          const adapter = this.getChainAdapter(chain);
          const latestBlock = await adapter.getLatestBlock();
          const timeSinceLastBlock = Date.now() - latestBlock.timestamp;
          
          return {
            chain,
            healthy: timeSinceLastBlock < 60000, // 1 minute threshold
            latency: timeSinceLastBlock,
            block_height: latestBlock.height
          };
        } catch (error) {
          return {
            chain,
            healthy: false,
            error: error.message
          };
        }
      })
    );
    
    return {
      healthy: results.every(r => r.healthy),
      details: results
    };
  }
}
```

## Development Guidelines

### Adding New Chain Support

1. **Implement Chain Adapter**:
```typescript
class NewChainAdapter implements ChainAdapter {
  // Implement all required methods
}
```

2. **Create Vault Contracts**:
   - Deploy standardized vault interface
   - Implement chain-specific optimizations
   - Add security controls

3. **Add Configuration**:
```typescript
// config/chains.ts
export const CHAIN_CONFIG = {
  new_chain: {
    chainId: 999,
    name: "New Chain",
    nativeCurrency: "NEW",
    rpcUrls: ["https://rpc.newchain.com"],
    blockExplorer: "https://explorer.newchain.com",
    contracts: {
      vault_factory: "0x...",
      swap_router: "0x..."
    }
  }
};
```

4. **Update Frontend Integration**:
   - Add chain selector options
   - Implement wallet connections
   - Update UI components

### Testing Strategy

```typescript
// Integration tests for multi-chain operations
describe('Multi-Chain Integration', () => {
  it('should execute cross-chain arbitrage', async () => {
    const aptosPrice = await aptosAdapter.getTokenPrice('APT');
    const flowPrice = await flowAdapter.getTokenPrice('APT');
    
    if (Math.abs(aptosPrice - flowPrice) / aptosPrice > 0.02) {
      const arbitrage = await arbitrageService.executeArbitrage({
        buyChain: aptosPrice < flowPrice ? 'aptos' : 'flow_evm',
        sellChain: aptosPrice > flowPrice ? 'aptos' : 'flow_evm',
        token: 'APT',
        amount: 100
      });
      
      expect(arbitrage.profit).toBeGreaterThan(0);
    }
  });
});
```

---

Ready to explore smart contract implementation? Check out [Smart Contracts](smart-contracts.md) for detailed vault architecture and security features.

## Next Steps
- [Smart Contracts →](smart-contracts.md)
- [API Reference →](api-reference.md)
- [Node Development →](node-development.md)

# Smart Contracts

TradingFlow's smart contract architecture provides secure, multi-chain vault management with advanced security features and cross-chain compatibility.

## Vault Contract Architecture

### Core Components

```
┌─────────────────────────────────────────┐
│              Vault Factory              │
│   • Deploy new vaults                   │
│   • Template management                 │
│   • Access control                      │
└─────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│            Vault Contract               │
│   ┌─────────────────────────────────┐   │
│   │      Asset Management          │   │
│   │ • Deposit/Withdraw             │   │
│   │ • Balance tracking             │   │
│   └─────────────────────────────────┘   │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │     Strategy Execution         │   │
│   │ • Trade execution              │   │
│   │ • Permission management        │   │
│   └─────────────────────────────────┘   │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │    Security Features           │   │
│   │ • Multi-signature             │   │
│   │ • Emergency controls          │   │
│   └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

## Aptos Vault Implementation

### Move Module Structure

```move
module tradingflow_addr::vault {
    use std::signer;
    use std::error;
    use std::account;
    use std::event;
    use aptos_std::table::{Self, Table};
    use aptos_framework::coin;
    use aptos_framework::timestamp;

    // Error codes
    const ENOT_OWNER: u64 = 1;
    const EINSUFFICIENT_BALANCE: u64 = 2;
    const EPAUSED: u64 = 3;

    // Vault struct
    struct Vault has store, key {
        owner: address,
        balances: Table<address, u64>,    // token -> amount
        strategies: Table<u64, Strategy>,
        paused: bool,
        emergency_stop: bool,
        multi_sig_threshold: u8,
        signers: vector<address>
    }

    // Strategy struct
    struct Strategy has store, drop {
        id: u64,
        name: vector<u8>,
        status: u8,                      // 0: inactive, 1: active, 2: paused
        permissions: vector<u8>,
        last_execution: u64
    }

    // Events
    struct DepositEvent has drop, store {
        user: address,
        token: address,
        amount: u64,
        timestamp: u64
    }

    // Initialize vault
    public entry fun initialize(
        account: &signer,
        multi_sig_threshold: u8,
        signers: vector<address>
    ) {
        let vault = Vault {
            owner: signer::get_address(account),
            balances: table::new(),
            strategies: table::new(),
            paused: false,
            emergency_stop: false,
            multi_sig_threshold,
            signers
        };
        move_to(account, vault);
    }

    // Deposit function
    public entry fun deposit<CoinType>(
        account: &signer,
        amount: u64
    ) acquires Vault {
        let vault_addr = get_vault_address(signer::get_address(account));
        let vault = borrow_global_mut<Vault>(vault_addr);

        assert!(!vault.paused, error::invalid_state(EPAUSED));

        // Transfer tokens to vault
        coin::transfer<CoinType>(account, vault_addr, amount);

        // Update balance
        let token_addr = coin::type_info<CoinType>().account_address();
        let current_balance = if (table::contains(&vault.balances, token_addr)) {
            *table::borrow(&vault.balances, token_addr)
        } else {
            0
        };
        table::upsert(&mut vault.balances, token_addr, current_balance + amount);

        // Emit event
        event::emit(DepositEvent {
            user: signer::get_address(account),
            token: token_addr,
            amount,
            timestamp: timestamp::now_seconds()
        });
    }

    // Execute trade (simplified)
    public entry fun execute_trade(
        account: &signer,
        strategy_id: u64,
        token_in: address,
        token_out: address,
        amount_in: u64,
        min_amount_out: u64
    ) acquires Vault {
        let vault_addr = get_vault_address(signer::get_address(account));
        let vault = borrow_global_mut<Vault>(vault_addr);

        // Permission checks
        assert!(is_authorized(account, vault), error::permission_denied(ENOT_OWNER));
        assert!(strategy_exists_and_active(vault, strategy_id), error::invalid_argument(EINVALID_STRATEGY));

        // Balance check
        let balance_in = *table::borrow(&vault.balances, token_in);
        assert!(balance_in >= amount_in, error::invalid_argument(EINSUFFICIENT_BALANCE));

        // Execute swap via DEX integration
        let amount_out = swap_tokens(token_in, token_out, amount_in, min_amount_out);

        // Update balances
        table::upsert(&mut vault.balances, token_in, balance_in - amount_in);
        let balance_out = if (table::contains(&vault.balances, token_out)) {
            *table::borrow(&vault.balances, token_out)
        } else {
            0
        };
        table::upsert(&mut vault.balances, token_out, balance_out + amount_out);

        // Update strategy execution time
        let strategy = table::borrow_mut(&mut vault.strategies, strategy_id);
        strategy.last_execution = timestamp::now_seconds();
    }

    // Emergency functions
    public entry fun emergency_pause(account: &signer) acquires Vault {
        let vault_addr = get_vault_address(signer::get_address(account));
        let vault = borrow_global_mut<Vault>(vault_addr);

        assert!(is_owner(account, vault), error::permission_denied(ENOT_OWNER));
        vault.paused = true;
    }

    public entry fun emergency_withdraw<CoinType>(
        account: &signer,
        amount: u64
    ) acquires Vault {
        let vault_addr = get_vault_address(signer::get_address(account));
        let vault = borrow_global_mut<Vault>(vault_addr);

        assert!(vault.emergency_stop, error::invalid_state(ENOT_EMERGENCY));
        assert!(is_owner(account, vault), error::permission_denied(ENOT_OWNER));

        let token_addr = coin::type_info<CoinType>().account_address();
        let balance = *table::borrow(&vault.balances, token_addr);
        assert!(balance >= amount, error::invalid_argument(EINSUFFICIENT_BALANCE));

        // Transfer to owner
        coin::transfer<CoinType>(&account::create_signer_for_test(vault_addr), signer::get_address(account), amount);
        table::upsert(&mut vault.balances, token_addr, balance - amount);
    }

    // Helper functions
    fun get_vault_address(owner: address): address {
        // Derive vault address from owner
        @vault_addr // In practice, this would be derived properly
    }

    fun is_authorized(account: &signer, vault: &Vault): bool {
        let user_addr = signer::get_address(account);
        user_addr == vault.owner || vector::contains(&vault.signers, &user_addr)
    }

    fun strategy_exists_and_active(vault: &Vault, strategy_id: u64): bool {
        table::contains(&vault.strategies, strategy_id) &&
        table::borrow(&vault.strategies, strategy_id).status == 1
    }

    // DEX integration (simplified)
    fun swap_tokens(
        token_in: address,
        token_out: address,
        amount_in: u64,
        min_amount_out: u64
    ): u64 {
        // Integration with DEX like Hyperion
        // This would call the DEX contract
        // For demonstration, return a mock value
        amount_in * 98 / 100  // 2% fee
    }
}
```

### Security Features

#### Access Control

```move
public fun is_multi_sig_approved(
    vault: &Vault,
    action_hash: vector<u8>,
    signatures: vector<vector<u8>>
): bool {
    let approved_count = 0u8;
    let i = 0u64;

    while (i < vector::length(&vault.signers)) {
        let signer = *vector::borrow(&vault.signers, i);
        let sig_index = 0u64;

        // Check if this signer provided a signature
        while (sig_index < vector::length(&signatures)) {
            let sig = vector::borrow(&signatures, sig_index);
            if (account::verify_signature(action_hash, *sig, signer)) {
                approved_count = approved_count + 1;
                break
            };
            sig_index = sig_index + 1;
        };

        i = i + 1;
    };

    approved_count >= vault.multi_sig_threshold
}
```

#### Rate Limiting

```move
struct RateLimit has store {
    last_action: u64,
    action_count: u64,
    window_start: u64
}

public fun check_rate_limit(vault: &mut Vault, action_type: u8): bool {
    let now = timestamp::now_seconds();
    let window_duration = 3600u64;  // 1 hour window
    let max_actions = 100u64;       // Max 100 actions per hour

    // Reset window if needed
    if (now - vault.rate_limit.window_start > window_duration) {
        vault.rate_limit.window_start = now;
        vault.rate_limit.action_count = 0;
    };

    vault.rate_limit.action_count < max_actions
}
```

## Flow EVM Vault Implementation

### Solidity Contract Structure

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract FlowEvmVault is Ownable, ReentrancyGuard, Pausable {
    // State variables
    mapping(address => mapping(address => uint256)) public balances;  // user -> token -> amount
    mapping(address => bool) public allowedTokens;
    mapping(bytes32 => Transaction) public transactions;
    mapping(bytes32 => mapping(address => bool)) public confirmations;

    uint256 public transactionCount;
    uint256 public requiredConfirmations;

    // Constants
    uint256 public constant TIMELOCK_DURATION = 24 hours;
    uint256 public constant MAX_SLIPPAGE = 300;  // 3%

    // Structs
    struct Transaction {
        address destination;
        uint256 value;
        bytes data;
        bool executed;
        uint256 confirmations;
        uint256 timestamp;
    }

    // Events
    event Deposit(address indexed user, address indexed token, uint256 amount);
    event Withdraw(address indexed user, address indexed token, uint256 amount);
    event TransactionSubmitted(bytes32 indexed txId, address indexed submitter);
    event TransactionConfirmed(bytes32 indexed txId, address indexed confirmer);
    event TransactionExecuted(bytes32 indexed txId);

    // Modifiers
    modifier onlySigner() {
        require(isSigner(msg.sender), "Not authorized");
        _;
    }

    modifier txExists(bytes32 txId) {
        require(transactions[txId].timestamp != 0, "Transaction does not exist");
        _;
    }

    modifier notExecuted(bytes32 txId) {
        require(!transactions[txId].executed, "Transaction already executed");
        _;
    }

    modifier notConfirmed(bytes32 txId) {
        require(!confirmations[txId][msg.sender], "Transaction already confirmed");
        _;
    }

    // Constructor
    constructor(uint256 _requiredConfirmations, address[] memory _signers) {
        require(_requiredConfirmations > 0, "Invalid confirmation count");
        require(_signers.length >= _requiredConfirmations, "Not enough signers");

        requiredConfirmations = _requiredConfirmations;

        for (uint i = 0; i < _signers.length; i++) {
            require(_signers[i] != address(0), "Invalid signer");
            signers.push(_signers[i]);
        }
    }

    // Deposit function
    function deposit(address token, uint256 amount) external nonReentrant whenNotPaused {
        require(allowedTokens[token], "Token not allowed");
        require(amount > 0, "Invalid amount");

        IERC20(token).transferFrom(msg.sender, address(this), amount);
        balances[msg.sender][token] += amount;

        emit Deposit(msg.sender, token, amount);
    }

    // Withdraw function
    function withdraw(address token, uint256 amount) external nonReentrant {
        require(balances[msg.sender][token] >= amount, "Insufficient balance");
        require(amount > 0, "Invalid amount");

        balances[msg.sender][token] -= amount;
        IERC20(token).transfer(msg.sender, amount);

        emit Withdraw(msg.sender, token, amount);
    }

    // Submit transaction for multi-sig approval
    function submitTransaction(
        address destination,
        uint256 value,
        bytes memory data
    ) public onlySigner returns (bytes32 txId) {
        txId = keccak256(abi.encodePacked(
            destination, value, data, block.timestamp, transactionCount
        ));

        transactions[txId] = Transaction({
            destination: destination,
            value: value,
            data: data,
            executed: false,
            confirmations: 0,
            timestamp: block.timestamp
        });

        transactionCount++;
        emit TransactionSubmitted(txId, msg.sender);

        confirmTransaction(txId);
    }

    // Confirm transaction
    function confirmTransaction(bytes32 txId)
        public
        onlySigner
        txExists(txId)
        notExecuted(txId)
        notConfirmed(txId)
    {
        confirmations[txId][msg.sender] = true;
        transactions[txId].confirmations += 1;

        emit TransactionConfirmed(txId, msg.sender);

        if (transactions[txId].confirmations >= requiredConfirmations) {
            executeTransaction(txId);
        }
    }

    // Execute transaction
    function executeTransaction(bytes32 txId)
        internal
        txExists(txId)
        notExecuted(txId)
    {
        Transaction storage txn = transactions[txId];
        require(
            block.timestamp >= txn.timestamp + TIMELOCK_DURATION,
            "Timelock not expired"
        );
        require(txn.confirmations >= requiredConfirmations, "Not enough confirmations");

        txn.executed = true;

        (bool success,) = txn.destination.call{value: txn.value}(txn.data);
        require(success, "Transaction execution failed");

        emit TransactionExecuted(txId);
    }

    // Emergency functions
    function emergencyPause() external onlyOwner {
        _pause();
    }

    function emergencyWithdraw(address token, uint256 amount) external onlyOwner whenPaused {
        uint256 balance = IERC20(token).balanceOf(address(this));
        require(amount <= balance, "Insufficient contract balance");

        IERC20(token).transfer(owner(), amount);
    }

    // Strategy execution (simplified DEX swap)
    function executeSwap(
        address tokenIn,
        address tokenOut,
        uint256 amountIn,
        uint256 minAmountOut,
        address dexRouter
    ) external onlySigner nonReentrant whenNotPaused returns (uint256 amountOut) {
        require(balances[msg.sender][tokenIn] >= amountIn, "Insufficient vault balance");

        // Approve DEX router
        IERC20(tokenIn).approve(dexRouter, amountIn);

        // Prepare swap path
        address[] memory path = new address[](2);
        path[0] = tokenIn;
        path[1] = tokenOut;

        // Execute swap (PunchSwap V2 example)
        uint256[] memory amounts = IPunchSwapRouter(dexRouter).swapExactTokensForTokens(
            amountIn,
            minAmountOut,
            path,
            address(this),
            block.timestamp + 300
        );

        amountOut = amounts[1];

        // Update balances
        balances[msg.sender][tokenIn] -= amountIn;
        balances[msg.sender][tokenOut] += amountOut;
    }

    // View functions
    function getTransaction(bytes32 txId) external view returns (
        address destination,
        uint256 value,
        bytes memory data,
        bool executed,
        uint256 confirmations
    ) {
        Transaction memory txn = transactions[txId];
        return (
            txn.destination,
            txn.value,
            txn.data,
            txn.executed,
            txn.confirmations
        );
    }

    function getBalance(address user, address token) external view returns (uint256) {
        return balances[user][token];
    }

    function isSigner(address account) public view returns (bool) {
        for (uint i = 0; i < signers.length; i++) {
            if (signers[i] == account) return true;
        }
        return false;
    }

    // Admin functions
    function addAllowedToken(address token) external onlyOwner {
        allowedTokens[token] = true;
    }

    function removeAllowedToken(address token) external onlyOwner {
        allowedTokens[token] = false;
    }

    function updateRequiredConfirmations(uint256 newRequired) external onlyOwner {
        require(newRequired > 0 && newRequired <= signers.length, "Invalid confirmation count");
        requiredConfirmations = newRequired;
    }
}
```

### Security Audits

#### Third-Party Audit Process

```solidity
// Audit trail for critical functions
contract Auditable {
    struct AuditEntry {
        bytes32 txHash;
        address executor;
        string action;
        uint256 timestamp;
        bytes32 beforeState;
        bytes32 afterState;
    }

    AuditEntry[] public auditLog;

    modifier audited(string memory action) {
        bytes32 beforeState = keccak256(abi.encodePacked(
            address(this).balance,
            // Add other critical state variables
        ));

        _;

        bytes32 afterState = keccak256(abi.encodePacked(
            address(this).balance,
            // Add other critical state variables
        ));

        auditLog.push(AuditEntry({
            txHash: blockhash(block.number - 1),
            executor: msg.sender,
            action: action,
            timestamp: block.timestamp,
            beforeState: beforeState,
            afterState: afterState
        }));
    }
}
```

#### Formal Verification

```solidity
// Invariant properties for formal verification
contract VaultInvariants {
    // Invariant: Total balances should never decrease unexpectedly
    function invariant_totalBalances() external view {
        uint256 total = 0;
        for (uint i = 0; i < allowedTokens.length; i++) {
            total += IERC20(allowedTokens[i]).balanceOf(address(this));
        }
        assert(total >= recordedTotalBalance);
    }

    // Invariant: Executed transactions cannot be modified
    function invariant_transactionImmutability(bytes32 txId) external view {
        Transaction memory txn = transactions[txId];
        if (txn.executed) {
            // Ensure transaction details haven't changed
            assert(txn.destination != address(0));
            assert(txn.timestamp > 0);
        }
    }

    // Invariant: Multi-sig requirements are enforced
    function invariant_multiSigRequirements(bytes32 txId) external view {
        Transaction memory txn = transactions[txId];
        if (txn.executed) {
            assert(txn.confirmations >= requiredConfirmations);
        }
    }
}
```

## Cross-Chain Bridge Contracts

### Bridge Vault Interface

```solidity
interface IBridgeVault {
    function lockTokens(
        address token,
        uint256 amount,
        bytes32 recipient,
        uint256 targetChain
    ) external returns (bytes32 transferId);

    function unlockTokens(
        bytes32 transferId,
        address token,
        uint256 amount,
        address recipient
    ) external;

    function verifyTransfer(
        bytes32 transferId,
        uint256 sourceChain,
        bytes32 sender,
        bytes32 recipient,
        address token,
        uint256 amount,
        bytes memory signature
    ) external returns (bool);
}

contract BridgeEnabledVault is FlowEvmVault, IBridgeVault {
    mapping(bytes32 => BridgeTransfer) public bridgeTransfers;
    mapping(uint256 => address) public chainContracts;

    struct BridgeTransfer {
        address token;
        uint256 amount;
        address sender;
        bytes32 recipient;
        uint256 sourceChain;
        uint256 targetChain;
        bool completed;
        uint256 timestamp;
    }

    event TokensLocked(
        bytes32 indexed transferId,
        address indexed token,
        uint256 amount,
        address indexed sender,
        bytes32 recipient,
        uint256 targetChain
    );

    event TokensUnlocked(
        bytes32 indexed transferId,
        address indexed token,
        uint256 amount,
        address indexed recipient
    );

    function lockTokens(
        address token,
        uint256 amount,
        bytes32 recipient,
        uint256 targetChain
    ) external override returns (bytes32 transferId) {
        require(balances[msg.sender][token] >= amount, "Insufficient balance");

        transferId = keccak256(abi.encodePacked(
            token, amount, msg.sender, recipient, targetChain, block.timestamp
        ));

        bridgeTransfers[transferId] = BridgeTransfer({
            token: token,
            amount: amount,
            sender: msg.sender,
            recipient: recipient,
            sourceChain: getCurrentChainId(),
            targetChain: targetChain,
            completed: false,
            timestamp: block.timestamp
        });

        // Lock tokens in escrow
        balances[msg.sender][token] -= amount;

        emit TokensLocked(transferId, token, amount, msg.sender, recipient, targetChain);

        return transferId;
    }

    function unlockTokens(
        bytes32 transferId,
        address token,
        uint256 amount,
        address recipient
    ) external override {
        require(isAuthorizedBridge(msg.sender), "Unauthorized bridge");

        BridgeTransfer storage transfer = bridgeTransfers[transferId];
        require(!transfer.completed, "Transfer already completed");
        require(transfer.token == token, "Token mismatch");
        require(transfer.amount == amount, "Amount mismatch");

        transfer.completed = true;

        // Mint or unlock tokens on destination chain
        balances[recipient][token] += amount;

        emit TokensUnlocked(transferId, token, amount, recipient);
    }

    function getCurrentChainId() internal pure returns (uint256) {
        // Flow EVM chain ID
        return 12345;  // Replace with actual chain ID
    }

    function isAuthorizedBridge(address bridge) internal view returns (bool) {
        // Check if bridge contract is authorized for this chain
        return authorizedBridges[bridge];
    }
}
```

## Upgrade Mechanisms

### Proxy Pattern Implementation

```solidity
contract VaultProxy {
    address public implementation;
    address public admin;

    constructor(address _implementation) {
        implementation = _implementation;
        admin = msg.sender;
    }

    function upgradeTo(address newImplementation) external {
        require(msg.sender == admin, "Only admin can upgrade");
        require(newImplementation != address(0), "Invalid implementation");

        implementation = newImplementation;
    }

    fallback() external payable {
        address impl = implementation;
        require(impl != address(0), "Implementation not set");

        assembly {
            let ptr := mload(0x40)
            calldatacopy(ptr, 0, calldatasize())
            let result := delegatecall(gas(), impl, ptr, calldatasize(), 0, 0)
            let size := returndatasize()
            returndatacopy(ptr, 0, size)

            switch result
            case 0 { revert(ptr, size) }
            default { return(ptr, size) }
        }
    }
}

contract VaultImplementationV2 is FlowEvmVault {
    // New features in V2
    uint256 public maxDailyVolume;
    mapping(address => uint256) public dailyVolume;

    function setMaxDailyVolume(uint256 volume) external onlyOwner {
        maxDailyVolume = volume;
    }

    function checkDailyVolume(address user, uint256 amount) internal view returns (bool) {
        return dailyVolume[user] + amount <= maxDailyVolume;
    }

    // Override executeSwap to include volume checks
    function executeSwap(
        address tokenIn,
        address tokenOut,
        uint256 amountIn,
        uint256 minAmountOut,
        address dexRouter
    ) external override returns (uint256 amountOut) {
        require(checkDailyVolume(msg.sender, amountIn), "Daily volume limit exceeded");

        amountOut = super.executeSwap(tokenIn, tokenOut, amountIn, minAmountOut, dexRouter);

        dailyVolume[msg.sender] += amountIn;
    }
}
```

---

Smart contracts form the secure foundation of TradingFlow's vault system. The multi-chain architecture ensures consistent security and functionality across all supported networks while maintaining the highest standards of smart contract development.

## Next Steps
- [Multi-Chain Architecture →](multi-chain-architecture.md) - System-wide architecture
- [API Reference →](api-reference.md) - Integration interfaces
- [Security & Risk Management →](../security-risk-management/) - Security features

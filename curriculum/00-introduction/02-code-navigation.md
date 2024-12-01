# Navigating the Bitcoin Core Codebase

## 🎯 Navigation Fundamentals

### Key Directories Overview
```bash
bitcoin/
├── src/              # Core implementation
│   ├── consensus/    # Consensus rules
│   ├── primitives/   # Basic types
│   ├── script/       # Script interpreter
│   ├── wallet/       # Wallet implementation
│   └── rpc/          # RPC interface
├── test/            # Test framework
├── doc/             # Documentation
└── contrib/         # Tools and utilities
```

## 🔍 Real-World Navigation Examples

### 1. Following Transaction Flow
From recent [PR #27460](https://github.com/bitcoin/bitcoin/pull/27460):
```cpp
// Entry point: src/node/transaction.cpp
bool AcceptToMemoryPool(/* ... */) {
    // -> Check transaction validity
    CheckTransaction(tx);
    
    // -> Check against consensus rules
    CheckConsensus(tx);
    
    // -> Check mempool conflicts
    CheckMempool(tx);
}

// Implementation details in different files:
// src/consensus/tx_verify.cpp
// src/validation.cpp
// src/txmempool.cpp
```

### 2. Understanding Chain Validation
From [PR #27711](https://github.com/bitcoin/bitcoin/pull/27711):
```cpp
// src/validation.cpp
bool CChainState::ConnectBlock(/* ... */) {
    // -> Check block validity
    CheckBlock(block);
    
    // -> Update UTXO set
    UpdateUTXOSet(block);
    
    // -> Update chain state
    UpdateChainState(block);
}
```

## 🛠️ Navigation Tools and Techniques

### 1. Using `git grep` Effectively
```bash
# Find all references to a function
git grep -n "AcceptToMemoryPool"

# Search in specific files
git grep -n "CheckTransaction" "src/consensus/*.cpp"

# Search with context
git grep -n -A 3 -B 3 "CreateNewBlock"
```

### 2. Understanding Call Hierarchies
From [PR #27156](https://github.com/bitcoin/bitcoin/pull/27156):
```cpp
// Call hierarchy example:
ProcessNewBlock
└── CheckBlock
    ├── CheckBlockHeader
    │   └── CheckProofOfWork
    └── CheckTransactions
        └── CheckTransaction
```

### 3. Finding Test Coverage
```bash
# Find tests for specific functionality
git grep -l "TEST_CASE.*AcceptToMemoryPool" test/
git grep -l "BOOST_AUTO_TEST_CASE.*CheckBlock" test/
```

## 📚 Common Navigation Patterns

### 1. Feature Implementation Search
```bash
# Example: Understanding P2P message handling
git grep -l "ProcessMessage.*version" src/net*
git grep -l "PushMessage" src/net*
```

### 2. Bug Investigation Pattern
From [Issue #27501](https://github.com/bitcoin/bitcoin/issues/27501):
```bash
# 1. Find error message
git grep -n "insufficient priority" src/

# 2. Find caller
git grep -n "CheckFeeRate" src/

# 3. Find tests
git grep -n "TestFeeRate" test/
```

## 🎯 Maintainer Tips

### 1. Code Organization Patterns
```cpp
// Example: Clear separation of concerns
class CWallet {
    // Public interface
    public:
        bool AddToWallet(const CTransactionRef& tx);
        
    // Implementation details
    private:
        bool AddToWalletIfInvolvingMe(const CTransactionRef& tx);
        void MarkConflicted(const uint256& hashBlock, const uint256& hashTx);
};
```

### 2. Common Refactoring Patterns
From [PR #27388](https://github.com/bitcoin/bitcoin/pull/27388):
```cpp
// Before: Mixed concerns
void ProcessBlock(const CBlock& block) {
    // Validation logic mixed with processing
    if (!CheckBlock(block)) return false;
    if (!ContextualCheckBlock(block)) return false;
    AddToChain(block);
}

// After: Separated concerns
bool ValidateBlock(const CBlock& block) {
    return CheckBlock(block) && ContextualCheckBlock(block);
}

void ProcessBlock(const CBlock& block) {
    if (ValidateBlock(block)) {
        AddToChain(block);
    }
}
```

## 🔄 Complex Search Scenarios

### 1. Cross-Reference Analysis
```bash
# Find all callers of a function
git grep -n "GetBlockHash" src/

# Find all implementations
git grep -n "virtual.*GetBlockHash" src/

# Find interface definitions
git grep -n "GetBlockHash.*=" src/
```

### 2. Feature Dependencies
```bash
# Find feature flag usage
git grep -n "ENABLE_WALLET" src/

# Find conditional compilation
git grep -n "#ifdef.*ENABLE_WALLET" src/
```

## 🎯 Navigation Exercises

### 1. Transaction Validation Flow
Task: Trace the complete validation flow of a transaction
```bash
# Start at entry point
git grep -n "AcceptToMemoryPool" src/

# Find validation checks
git grep -n "CheckTransaction" src/consensus/
git grep -n "ContextualCheckTransaction" src/consensus/

# Locate test coverage
git grep -n "TestAcceptToMemoryPool" test/
```

### 2. P2P Message Handling
Task: Understand how a specific P2P message is processed
```bash
# Find message handler
git grep -n "ProcessMessage.*block" src/net*

# Locate message creation
git grep -n "PushMessage.*block" src/net*

# Find test scenarios
git grep -n "TestProcessMessage.*block" test/
```

## 📊 Navigation Best Practices

### 1. Understanding Dependencies
```cpp
// Example: Clear dependency documentation
class CNode {
    // External dependencies clearly stated
    CConnman& connman;
    CTxMemPool& mempool;
    ChainstateManager& chainman;
    
    // Constructor makes dependencies explicit
    CNode(CConnman& connman_in, CTxMemPool& mempool_in, 
          ChainstateManager& chainman_in);
};
```

### 2. Following Code Flow
```cpp
// Example: Clear flow documentation
void ProcessBlock(const CBlock& block) {
    // 1. Basic validation
    if (!CheckBlock(block)) {
        LogPrintf("Block failed basic validation\n");
        return;
    }
    
    // 2. Contextual validation
    if (!ContextualCheckBlock(block)) {
        LogPrintf("Block failed contextual validation\n");
        return;
    }
    
    // 3. Add to chain
    AddToChain(block);
}
```

Remember: Effective code navigation is crucial for Bitcoin Core development. Understanding the codebase structure and relationships between components will significantly improve your development efficiency.
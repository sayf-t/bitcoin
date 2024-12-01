# Bitcoin Core Consensus Rules

## 🎯 Consensus Overview

### Core Principles
```cpp
// Key consensus validation from src/consensus/validation.h
bool CheckBlock(const CBlock& block, BlockValidationState& state)
{
    // These are checks that are independent of context
    if (block.fChecked)
        return true;

    // Size limits
    if (block.vtx.empty() || block.vtx.size() > MAX_BLOCK_SIZE)
        return state.Invalid(BlockValidationResult::BLOCK_CONSENSUS,
                           "bad-blk-length", "size limits failed");

    // Check proof of work matches claimed amount
    if (!CheckProofOfWork(block.GetHash(), block.nBits, Params().GetConsensus()))
        return state.Invalid(BlockValidationResult::BLOCK_CONSENSUS,
                           "bad-diffbits", "proof of work failed");

    // ... more validation rules
}
```

## 🔍 Interactive Consensus Simulations

### 1. Block Validation Simulation
```cpp
// Simulation: Block validation process
class BlockValidationSimulator {
public:
    enum ValidationStage {
        SYNTAX,
        CONTEXTUAL,
        SCRIPTS,
        COMPLETE
    };

    struct SimulationResult {
        bool passed;
        ValidationStage stage;
        std::string message;
    };

    SimulationResult SimulateValidation(const CBlock& block) {
        // 1. Basic syntax validation
        if (!CheckBlockSyntax(block))
            return {false, SYNTAX, "Invalid block syntax"};

        // 2. Contextual validation
        if (!CheckBlockContext(block))
            return {false, CONTEXTUAL, "Invalid block context"};

        // 3. Script validation
        if (!CheckBlockScripts(block))
            return {false, SCRIPTS, "Script validation failed"};

        return {true, COMPLETE, "Block validation successful"};
    }

private:
    bool CheckBlockSyntax(const CBlock& block) {
        // Simulate syntax checks
        return block.vtx.size() > 0 && 
               block.vtx.size() <= MAX_BLOCK_SIZE &&
               CheckProofOfWork(block.GetHash(), block.nBits);
    }

    bool CheckBlockContext(const CBlock& block) {
        // Simulate contextual validation
        return block.hashPrevBlock == chainActive.Tip()->GetBlockHash() &&
               block.nTime > chainActive.Tip()->GetMedianTimePast();
    }

    bool CheckBlockScripts(const CBlock& block) {
        // Simulate script validation
        for (const auto& tx : block.vtx) {
            if (!CheckTransaction(*tx))
                return false;
        }
        return true;
    }
};
```

### 2. Transaction Validation Simulation
```cpp
// Simulation: Transaction validation process
class TransactionValidationSimulator {
public:
    struct ValidationContext {
        CAmount balance;
        uint32_t sequence;
        bool isP2SH;
    };

    bool SimulateValidation(const CTransaction& tx, ValidationContext& context) {
        // 1. Basic checks
        if (!CheckBasicRules(tx))
            return LogAndReturn("Basic rules failed");

        // 2. Input validation
        if (!ValidateInputs(tx, context))
            return LogAndReturn("Input validation failed");

        // 3. Output validation
        if (!ValidateOutputs(tx, context))
            return LogAndReturn("Output validation failed");

        // 4. Script validation
        if (!ValidateScripts(tx, context))
            return LogAndReturn("Script validation failed");

        return true;
    }

private:
    bool CheckBasicRules(const CTransaction& tx) {
        // Size limits
        if (GetTransactionWeight(tx) > MAX_TRANSACTION_WEIGHT)
            return false;

        // Check for duplicate inputs
        std::set<COutPoint> inputs;
        for (const auto& input : tx.vin) {
            if (!inputs.insert(input.prevout).second)
                return false;
        }

        return true;
    }

    bool ValidateInputs(const CTransaction& tx, ValidationContext& context) {
        CAmount totalIn = 0;
        for (const auto& input : tx.vin) {
            // Check if input exists and is unspent
            if (!context.balance)
                return false;

            // Check sequence numbers
            if (input.nSequence < context.sequence)
                return false;

            totalIn += context.balance;
        }
        context.balance = totalIn;
        return true;
    }
};
```

## 🛠️ Real Consensus Bug Examples

### 1. Integer Overflow Bug (CVE-2010-5139)
```cpp
// Original buggy code
void CheckTransaction(const CTransaction& tx) {
    CAmount nValueOut = 0;
    for (const auto& out : tx.vout) {
        nValueOut += out.nValue;  // Potential overflow!
    }
}

// Fixed code
void CheckTransaction(const CTransaction& tx) {
    CAmount nValueOut = 0;
    for (const auto& out : tx.vout) {
        if (!MoneyRange(out.nValue))
            throw std::runtime_error("amount out of range");
        if (!MoneyRange(nValueOut + out.nValue))
            throw std::runtime_error("amount out of range");
        nValueOut += out.nValue;
    }
}
```

### 2. Time Warp Attack Prevention
```cpp
// Vulnerability
bool AcceptBlock(const CBlock& block) {
    if (block.GetBlockTime() > GetAdjustedTime() + 2 * 60 * 60)
        return error("time too far in the future");
    return true;
}

// Fixed implementation
bool AcceptBlock(const CBlock& block) {
    // Check against median time of last 11 blocks
    if (block.GetBlockTime() <= GetMedianTimePast())
        return error("time too old");

    // Check against adjusted network time
    if (block.GetBlockTime() > GetAdjustedTime() + 2 * 60 * 60)
        return error("time too far in the future");

    return true;
}
```

## 📊 Performance Considerations

### 1. Script Validation Optimization
```cpp
// Optimized script validation
bool CheckInputScripts(const CTransaction& tx, 
                      ValidationState& state,
                      const CCoinsViewCache& inputs,
                      unsigned int flags,
                      bool cacheSigStore,
                      bool cacheFullScriptStore,
                      PrecomputedTransactionData& txdata)
{
    if (tx.IsCoinBase())
        return true;

    // First check if script executions have been cached
    if (cacheSigStore && !CheckInputsFromCache(tx, state, inputs, flags)) {
        return false;
    }

    // Gather input scripts
    std::vector<CScriptCheck> vChecks;
    bool fCacheResults = fScriptCheck && cacheSigStore;
    if (!CollectScriptChecks(tx, state, inputs, flags, fCacheResults, 
                            cacheFullScriptStore, txdata, vChecks)) {
        return false;
    }

    // Execute validation checks in parallel
    return ExecuteScriptChecksParallel(vChecks, state);
}
```

### 2. Block Validation Optimization
```cpp
// Optimized block validation
bool ConnectBlock(const CBlock& block, BlockValidationState& state,
                 CBlockIndex* pindex, CCoinsViewCache& view,
                 const CChainParams& chainparams,
                 bool fJustCheck = false)
{
    // Precompute block hash
    uint256 blockHash = block.GetHash();

    // Check it again in case a previous version let a bad block in
    if (!CheckBlock(block, state, chainparams.GetConsensus()))
        return error("%s: Consensus::CheckBlock: %s", __func__, 
                    FormatStateMessage(state));

    // Verify that the view's current state corresponds to the previous block
    uint256 hashPrevBlock = pindex->pprev == nullptr ? uint256() : 
                           pindex->pprev->GetBlockHash();
    assert(hashPrevBlock == view.GetBestBlock());

    // Special case for the genesis block
    if (block.GetHash() == chainparams.GetConsensus().hashGenesisBlock) {
        if (!fJustCheck)
            view.SetBestBlock(pindex->GetBlockHash());
        return true;
    }

    // ... more optimized validation
}
```

## 🔒 Security Considerations

### 1. DoS Protection
```cpp
// Protection against CPU exhaustion attacks
bool CheckBlockScripts(const CBlock& block, 
                      ValidationState& state,
                      const Consensus::Params& consensusParams)
{
    // Track script operation count
    int64_t nSigOps = 0;
    
    // Check total signature operations
    for (const auto& tx : block.vtx) {
        nSigOps += GetLegacySigOpCount(*tx);
        if (nSigOps > MAX_BLOCK_SIGOPS)
            return state.Invalid(BlockValidationResult::BLOCK_CONSENSUS,
                               "bad-blk-sigops", "out-of-bounds SigOpCount");
    }

    // Implement rate limiting
    static CTxMemPool::LockPoints lp;
    CTxMemPool::setEntries setAncestors;
    int64_t nLimitAncestors = DEFAULT_ANCESTOR_LIMIT;
    size_t nLimitAncestorSize = DEFAULT_ANCESTOR_SIZE_LIMIT;
    
    // ... more protections
}
```

### 2. Memory Protection
```cpp
// Protection against memory exhaustion
bool CheckTransaction(const CTransaction& tx, ValidationState& state)
{
    // Check transaction size limits
    if (::GetSerializeSize(tx, PROTOCOL_VERSION) > MAX_TRANSACTION_SIZE)
        return state.Invalid(TxValidationResult::TX_CONSENSUS,
                           "bad-txns-oversize");

    // Check input count limits
    if (tx.vin.size() > MAX_TX_IN_COUNT)
        return state.Invalid(TxValidationResult::TX_CONSENSUS,
                           "bad-txns-too-many-inputs");

    // Check output count limits
    if (tx.vout.size() > MAX_TX_OUT_COUNT)
        return state.Invalid(TxValidationResult::TX_CONSENSUS,
                           "bad-txns-too-many-outputs");

    // ... more memory checks
}
```

## 📈 Monitoring and Debugging

### 1. Consensus State Monitor
```cpp
// Monitor consensus state
class ConsensusMonitor {
private:
    struct ValidationMetrics {
        std::atomic<uint64_t> nBlocksValidated{0};
        std::atomic<uint64_t> nTxValidated{0};
        std::atomic<uint64_t> nValidationTime{0};
        std::atomic<uint64_t> nValidationErrors{0};
    };
    ValidationMetrics metrics;

public:
    void UpdateMetrics(const ValidationResult& result) {
        if (result.fValid) {
            metrics.nBlocksValidated++;
            metrics.nValidationTime += result.nTimeMs;
        } else {
            metrics.nValidationErrors++;
            LogPrintf("Validation Error: %s\n", result.strError);
        }
    }

    ValidationMetrics GetMetrics() const {
        return metrics;
    }
};
```

### 2. Debug Logging
```cpp
// Enhanced consensus debugging
void LogConsensusEvent(const std::string& event, 
                      const std::string& details,
                      LogLevel level = LogLevel::DEBUG)
{
    static std::mutex cs_log;
    LOCK(cs_log);

    std::string logEntry = strprintf("%s: %s [%s]",
                                   DateTimeStrFormat("%Y-%m-%d %H:%M:%S", 
                                   GetTime()),
                                   event,
                                   details);

    if (level >= LogLevel::WARNING) {
        LogPrintf("CONSENSUS WARNING: %s\n", logEntry);
        AlertNotify(logEntry);
    } else {
        LogPrint(BCLog::CONSENSUS, "%s\n", logEntry);
    }
}
```

Remember: Consensus rules are the backbone of Bitcoin Core. Any changes must be thoroughly tested and reviewed to prevent consensus failures. 
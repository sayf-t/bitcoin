# Bitcoin Core Mempool Management

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Implement efficient mempool data structures
- Design robust transaction validation
- Develop accurate fee estimation
- Optimize memory usage
- Handle mempool conflicts

## Prerequisites
- Understanding of Bitcoin transactions
- Knowledge of P2P network (covered in 04-network/)
- Familiarity with C++ data structures
- Experience with concurrent programming

## 🌟 Real-World Context
Think of Bitcoin's mempool like a busy train station:
- Transactions = Passengers
- Mempool = Waiting Area
- Fee Estimation = Ticket Pricing
- Memory Limits = Station Capacity
- Validation = Security Checks

## 🔍 Mempool Components

### 1. Transaction Entry Management
```cpp
// Advanced transaction entry
class CTxMemPoolEntry {
private:
    CTransactionRef tx;
    CAmount nFee;              // Cached to avoid expensive parent-child fee calculation
    size_t nTxWeight;          // Cached to avoid ECDSA verification
    size_t nUsageSize;         // Memory usage
    int64_t nTime;             // Entry time
    unsigned int entryHeight;   // Chain height at entry
    bool spendsCoinbase;       // Does this tx spend a coinbase?
    int64_t sigOpCost;         // Cached legacy sigops plus witness sigop cost
    int64_t feeDelta;          // Used for determining the priority of the transaction
    LockPoints lockPoints;     // Track the height and time at which tx was final

public:
    CTxMemPoolEntry(const CTransactionRef& _tx, const CAmount& _nFee,
                   int64_t _nTime, unsigned int _entryHeight,
                   bool _spendsCoinbase, int64_t _sigOpCost,
                   LockPoints& _lp)
    {
        tx = _tx;
        nFee = _nFee;
        nTime = _nTime;
        entryHeight = _entryHeight;
        spendsCoinbase = _spendsCoinbase;
        sigOpCost = _sigOpCost;
        lockPoints = _lp;
        
        nTxWeight = GetTransactionWeight(*tx);
        nUsageSize = RecursiveDynamicUsage(tx);
    }
    
    CAmount GetFee() const { return nFee; }
    size_t GetTxSize() const { return nTxWeight; }
    int64_t GetTime() const { return nTime; }
    unsigned int GetHeight() const { return entryHeight; }
    int64_t GetSigOpCost() const { return sigOpCost; }
    
    // Calculate modified fees with fee deltas
    CAmount GetModifiedFee() const { return nFee + feeDelta; }
    
    // Calculate fee rate considering ancestor modifications
    CFeeRate GetModifiedFeeRate() const
    {
        return CFeeRate(GetModifiedFee(), nTxWeight);
    }
};
```

### 2. Memory Pool Implementation
```cpp
// Advanced memory pool
class CTxMemPool {
private:
    uint64_t nTransactionsUpdated;
    mutable CCriticalSection cs;
    std::map<uint256, CTxMemPoolEntry> mapTx;
    
    typedef std::pair<double, CTxMemPoolEntry> TxPriority;
    std::map<CTxMemPoolEntry*, std::set<uint256>> mapLinks;
    
    // Cached sums for performance
    std::atomic<size_t> totalTxSize{0};      // Total size of all txs
    std::atomic<size_t> cachedInnerUsage{0}; // Runtime memory usage
    
    // Fee estimation data
    std::map<uint256, std::vector<FeeEstimationData>> mapFeeHistory;

public:
    bool addUnchecked(const uint256& hash, const CTxMemPoolEntry& entry,
                      bool validFeeEstimate = true)
    {
        LOCK(cs);
        
        // Add to transaction map
        mapTx.insert(std::make_pair(hash, entry));
        
        // Update cached amounts
        const CTransaction& tx = *entry.GetTx();
        totalTxSize += entry.GetTxSize();
        cachedInnerUsage += entry.DynamicMemoryUsage();
        
        // Update fee estimation data
        if (validFeeEstimate) {
            updateFeeEstimatesLocked(entry);
        }
        
        // Update transaction links
        updateParentLinks(tx, hash);
        updateChildLinks(tx, hash);
        
        nTransactionsUpdated++;
        return true;
    }
    
    void removeRecursive(const CTransaction& tx,
                        MemPoolRemovalReason reason = MemPoolRemovalReason::UNKNOWN)
    {
        LOCK(cs);
        std::vector<uint256> txToRemove{tx.GetHash()};
        
        // Collect all descendants
        while (!txToRemove.empty()) {
            uint256 hash = txToRemove.back();
            txToRemove.pop_back();
            
            // Remove transaction
            auto it = mapTx.find(hash);
            if (it != mapTx.end()) {
                const CTxMemPoolEntry& entry = it->second;
                
                // Add children to removal list
                const auto& children = GetChildren(entry);
                txToRemove.insert(txToRemove.end(),
                                children.begin(), children.end());
                
                // Update cached amounts
                totalTxSize -= entry.GetTxSize();
                cachedInnerUsage -= entry.DynamicMemoryUsage();
                
                // Remove from maps
                mapTx.erase(it);
                removeLinks(hash);
            }
        }
        
        nTransactionsUpdated++;
    }
};
```

### 3. Fee Estimation
```cpp
// Advanced fee estimation
class CBlockPolicyEstimator {
private:
    struct TxConfirmStats {
        std::vector<double> decay;
        std::vector<EstimatorBucket> buckets;
        unsigned int scale;
        unsigned int maxConfirms;
        unsigned int decay_windows;
    };
    
    std::vector<TxConfirmStats> confirmStats;
    unsigned int nBestSeenHeight;
    unsigned int firstRecordedHeight;
    unsigned int historicalFirst;
    unsigned int historicalBest;

public:
    void processBlock(unsigned int nBlockHeight,
                     std::vector<CTxMemPoolEntry>& entries)
    {
        if (nBlockHeight <= nBestSeenHeight) {
            // Ignore out of order blocks
            return;
        }
        
        // Update confirmation stats
        for (const auto& entry : entries) {
            if (entry.GetHeight() != nBlockHeight) {
                // Only process transactions in this block
                continue;
            }
            
            // Calculate number of blocks to confirm
            unsigned int blocksToConfirm =
                nBlockHeight - entry.GetHeight();
            
            // Update fee statistics
            CFeeRate feeRate = CFeeRate(entry.GetFee(), entry.GetTxSize());
            updateFeeStat(feeRate, blocksToConfirm, entry.GetPriority());
        }
        
        // Update historical data
        updateHistoricalData(nBlockHeight);
        
        nBestSeenHeight = nBlockHeight;
    }
    
    CFeeRate estimateSmartFee(int confTarget,
                             FeeCalculation* feeCalc = nullptr) const
    {
        if (confTarget <= 0 || confTarget > 1008) {
            return CFeeRate(0);
        }
        
        // Find appropriate bucket for confirmation target
        unsigned int bucketIndex = getBucketIndex(confTarget);
        
        // Calculate success probability for each fee rate
        std::vector<double> probabilities;
        for (const auto& bucket : confirmStats[bucketIndex].buckets) {
            double probability = bucket.getConfirmationProbability();
            probabilities.push_back(probability);
        }
        
        // Find minimum fee rate meeting target probability
        double targetProb = 0.95; // 95% success rate
        CFeeRate minFeeRate;
        
        for (size_t i = 0; i < probabilities.size(); i++) {
            if (probabilities[i] >= targetProb) {
                minFeeRate = confirmStats[bucketIndex].buckets[i].getFeeRate();
                break;
            }
        }
        
        if (feeCalc) {
            feeCalc->est = minFeeRate;
            feeCalc->reason = FeeReason::FEERATE_ESTIMATE;
            feeCalc->probability = targetProb;
        }
        
        return minFeeRate;
    }
};
```

### 4. Memory Usage Optimization
```cpp
// Advanced memory management
class CMemPoolPolicy {
private:
    // Memory limits
    size_t nMaxMemPoolSize;
    size_t nMaxPercentageOfMemPoolSize;
    
    // Cached values for performance
    std::atomic<size_t> nCurrentMemPoolSize{0};
    std::atomic<size_t> nCurrentTxCount{0};

public:
    bool CheckMemPoolSize(size_t additionalSize,
                         size_t additionalCount = 1)
    {
        size_t newSize = nCurrentMemPoolSize + additionalSize;
        size_t newCount = nCurrentTxCount + additionalCount;
        
        // Check absolute size limit
        if (newSize > nMaxMemPoolSize) {
            return false;
        }
        
        // Check transaction count limit
        if (newCount > MAX_TX_COUNT) {
            return false;
        }
        
        // Check percentage limit
        size_t maxSize = (nMaxMemPoolSize *
                         nMaxPercentageOfMemPoolSize) / 100;
        if (newSize > maxSize) {
            return false;
        }
        
        return true;
    }
    
    void TrimToSize(size_t targetSize)
    {
        if (nCurrentMemPoolSize <= targetSize) {
            return;
        }
        
        // Sort transactions by fee rate
        std::vector<TxPriority> vPriority;
        for (const auto& entry : mapTx) {
            vPriority.push_back(TxPriority(
                entry.second.GetModifiedFeeRate(),
                entry.second));
        }
        std::sort(vPriority.begin(), vPriority.end());
        
        // Remove lowest fee transactions until target size is met
        size_t currentSize = nCurrentMemPoolSize;
        for (const auto& priority : vPriority) {
            const CTxMemPoolEntry& entry = priority.second;
            
            // Remove transaction and its descendants
            removeRecursive(*entry.GetTx(),
                          MemPoolRemovalReason::SIZELIMIT);
            
            currentSize -= entry.GetTxSize();
            if (currentSize <= targetSize) {
                break;
            }
        }
    }
};
```

## 🎯 Knowledge Check
1. How do you implement efficient transaction validation?
2. What are the key components of fee estimation?
3. How do you handle mempool size limits?
4. What strategies exist for conflict resolution?
5. How do you optimize memory usage?

## 🔍 Common Issues and Solutions

### Issue 1: Memory Management
- Implement size limits
- Use efficient data structures
- Regular cleanup
- Smart pruning strategies

### Issue 2: Fee Estimation
- Historical data analysis
- Dynamic fee calculation
- Priority consideration
- Market rate adaptation

### Issue 3: Transaction Conflicts
- Double-spend detection
- RBF handling
- CPFP consideration
- Ancestor/descendant limits

## 📚 Additional Resources
- [Bitcoin Core Mempool](https://github.com/bitcoin/bitcoin/blob/master/src/txmempool.h)
- [Fee Estimation](https://github.com/bitcoin/bitcoin/blob/master/src/policy/fees.h)
- [Memory Pool Limits](https://github.com/bitcoin/bitcoin/blob/master/doc/policy/mempool-limits.md)
- [Transaction Replacement](https://github.com/bitcoin/bitcoin/blob/master/doc/policy/mempool-replacements.md)

## 🎯 Next Steps
1. Study advanced mempool policies
2. Implement custom fee estimators
3. Build memory optimization tools
4. Contribute to mempool improvements

Remember: Efficient mempool management is crucial for network performance and user experience. Focus on validation, fee estimation, and memory optimization.
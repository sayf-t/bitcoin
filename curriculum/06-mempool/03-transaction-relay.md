# Bitcoin Core Transaction Relay

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Implement efficient transaction relay
- Design robust propagation strategies
- Develop inventory management systems
- Apply effective relay policies
- Optimize network bandwidth usage

## Prerequisites
- Understanding of P2P networking (covered in 04-network/)
- Knowledge of mempool management (covered in 05-mempool/01-mempool-management.md)
- Familiarity with transaction validation (covered in 05-mempool/02-transaction-validation.md)
- Experience with distributed systems

## 🌟 Real-World Context
Think of transaction relay like a news network:
- Transactions = News Stories
- Relay = Broadcasting
- Inventory = News Archive
- Policies = Editorial Guidelines
- Bandwidth = Network Capacity

## 🔍 Relay Components

### 1. Transaction Propagation
```cpp
// Advanced transaction propagation
class CTransactionPropagator {
private:
    // Track recently announced transactions
    std::map<uint256, int64_t> mapRecentlyAnnounced;
    
    // Track relay times
    std::map<uint256, int64_t> mapRelayTimes;
    
    // Track inventory requests
    std::map<uint256, std::set<NodeId>> mapRequestCount;
    
    // Bandwidth management
    std::atomic<uint64_t> nTotalBytesQueued{0};
    const uint64_t MAX_BYTES_QUEUED = 100 * 1024 * 1024; // 100MB

public:
    bool RelayTransaction(const CTransaction& tx,
                        const CConnman& connman)
    {
        const uint256& hash = tx.GetHash();
        
        // Check if recently announced
        int64_t nNow = GetTime();
        auto it = mapRecentlyAnnounced.find(hash);
        if (it != mapRecentlyAnnounced.end() &&
            nNow - it->second < ANNOUNCE_INTERVAL)
        {
            return false;
        }
        
        // Update announcement time
        mapRecentlyAnnounced[hash] = nNow;
        
        // Get transaction relay data
        CInv inv(MSG_TX, hash);
        
        // Select peers for relay
        std::vector<CNode*> vNodesCopy = connman.CopyNodeVector();
        for (auto* pnode : vNodesCopy) {
            if (!pnode->fRelayTxes) {
                continue;
            }
            
            // Check bandwidth limits
            if (nTotalBytesQueued > MAX_BYTES_QUEUED) {
                LogPrint(BCLog::NET, "Transaction relay queue full\n");
                break;
            }
            
            // Send inventory
            pnode->PushInventory(inv);
            nTotalBytesQueued += ::GetSerializeSize(tx, PROTOCOL_VERSION);
        }
        
        connman.ReleaseNodeVector(vNodesCopy);
        return true;
    }
};
```

### 2. Inventory Management
```cpp
// Advanced inventory management
class CInventoryManager {
private:
    // Track inventory announcements
    std::map<uint256, std::set<NodeId>> mapInventoryKnown;
    
    // Track inventory requests
    std::map<uint256, std::set<NodeId>> mapInventoryRequested;
    
    // Track inventory to announce
    std::map<NodeId, std::set<uint256>> mapInventoryToAnnounce;
    
    // Rate limiting
    std::map<NodeId, CRateTracker> mapRequestTracker;

public:
    bool AddInventoryKnown(NodeId nodeId, const CInv& inv)
    {
        LOCK(cs_inventory);
        
        // Add to known inventory
        mapInventoryKnown[inv.hash].insert(nodeId);
        
        // Remove from announce list
        auto it = mapInventoryToAnnounce.find(nodeId);
        if (it != mapInventoryToAnnounce.end()) {
            it->second.erase(inv.hash);
        }
        
        return true;
    }
    
    bool AddInventoryToAnnounce(NodeId nodeId, const uint256& hash)
    {
        LOCK(cs_inventory);
        
        // Check if already known
        auto it = mapInventoryKnown.find(hash);
        if (it != mapInventoryKnown.end() &&
            it->second.count(nodeId) > 0)
        {
            return false;
        }
        
        // Add to announce list
        mapInventoryToAnnounce[nodeId].insert(hash);
        return true;
    }
    
    void ProcessInventoryRequest(CNode* pfrom,
                               const CInv& inv,
                               const CConnman& connman)
    {
        LOCK(cs_inventory);
        
        // Check rate limits
        if (!mapRequestTracker[pfrom->GetId()].CheckAndUpdate(1)) {
            LogPrint(BCLog::NET, "Request rate limit exceeded\n");
            return;
        }
        
        // Process request based on type
        switch (inv.type) {
            case MSG_TX: {
                auto mi = mapInventoryRequested[inv.hash].insert(pfrom->GetId());
                if (!mi.second) {
                    LogPrint(BCLog::NET, "Duplicate request\n");
                    return;
                }
                
                // Fetch transaction
                CTransactionRef ptx = mempool.get(inv.hash);
                if (ptx) {
                    connman.PushMessage(pfrom,
                                      CNetMsgMaker(PROTOCOL_VERSION)
                                          .Make(NetMsgType::TX, *ptx));
                }
                break;
            }
            // Handle other inventory types...
        }
    }
};
```

### 3. Relay Policy
```cpp
// Advanced relay policy
class CRelayPolicy {
private:
    // Policy parameters
    const unsigned int MAX_PEER_TX_ANNOUNCEMENTS = 5000;
    const unsigned int MAX_PEER_TX_REQUESTS = 100;
    const unsigned int MIN_TX_RELAY_FEE = 1000;
    
    // Track peer statistics
    std::map<NodeId, CPeerRelayStats> mapPeerStats;

public:
    bool CheckRelayPolicy(const CTransaction& tx,
                         const CTxMemPool& pool,
                         CValidationState& state)
    {
        // Check transaction size
        unsigned int nSize = ::GetSerializeSize(tx, PROTOCOL_VERSION);
        if (nSize > MAX_STANDARD_TX_SIZE) {
            return state.DoS(100, false, REJECT_NONSTANDARD,
                           "tx-size-too-large");
        }
        
        // Check minimum fee
        CAmount nMinFee = nSize * MIN_TX_RELAY_FEE / 1000;
        if (tx.GetFee() < nMinFee) {
            return state.DoS(0, false, REJECT_INSUFFICIENTFEE,
                           "min-relay-fee-not-met");
        }
        
        // Check standard transaction rules
        if (!IsStandardTx(tx, state)) {
            return false;
        }
        
        // Check against recent rejects
        if (pool.isRejected(tx.GetHash())) {
            return state.DoS(0, false, REJECT_DUPLICATE,
                           "txn-already-rejected");
        }
        
        return true;
    }
    
    bool UpdatePeerStats(NodeId nodeId,
                        const CTransaction& tx,
                        bool fRelayed)
    {
        LOCK(cs_stats);
        auto& stats = mapPeerStats[nodeId];
        
        // Update announcement count
        if (stats.nTxAnnouncements >= MAX_PEER_TX_ANNOUNCEMENTS) {
            LogPrint(BCLog::NET,
                    "Peer exceeded transaction announcement limit\n");
            return false;
        }
        stats.nTxAnnouncements++;
        
        // Update relay statistics
        if (fRelayed) {
            stats.nTxRelayed++;
            stats.nBytesRelayed += ::GetSerializeSize(tx, PROTOCOL_VERSION);
        }
        
        return true;
    }
};
```

### 4. Bandwidth Optimization
```cpp
// Advanced bandwidth optimization
class CBandwidthOptimizer {
private:
    // Bandwidth tracking
    std::map<NodeId, CBandwidthStats> mapBandwidthStats;
    
    // Optimization parameters
    const uint64_t TARGET_OUTBOUND_RATE = 50 * 1024; // 50KB/s
    const uint64_t MAX_BURST_RATE = 200 * 1024;      // 200KB/s
    const uint64_t RATE_WINDOW = 60;                 // 1 minute

public:
    bool CheckBandwidthLimit(NodeId nodeId, uint64_t nBytes)
    {
        LOCK(cs_bandwidth);
        auto& stats = mapBandwidthStats[nodeId];
        
        // Update statistics
        uint64_t nNow = GetTime();
        stats.UpdateStats(nNow, nBytes);
        
        // Check rate limits
        if (stats.GetCurrentRate() > TARGET_OUTBOUND_RATE) {
            // Allow bursts up to MAX_BURST_RATE
            if (stats.GetCurrentRate() > MAX_BURST_RATE) {
                return false;
            }
            
            // Check if we've been above target rate for too long
            if (stats.GetTimeSinceLastBelowTarget(nNow) > RATE_WINDOW) {
                return false;
            }
        }
        
        return true;
    }
    
    void OptimizeBandwidthUsage(CNode* pnode,
                               const CTransaction& tx,
                               bool& fRelay)
    {
        // Check if we should relay based on bandwidth
        if (!CheckBandwidthLimit(pnode->GetId(),
                               ::GetSerializeSize(tx, PROTOCOL_VERSION)))
        {
            fRelay = false;
            return;
        }
        
        // Implement advanced relay strategies
        if (ShouldDelayRelay(pnode, tx)) {
            // Queue for delayed relay
            DelayRelay(pnode->GetId(), tx.GetHash());
            fRelay = false;
            return;
        }
        
        // Update outbound bandwidth tracking
        if (fRelay) {
            UpdateOutboundBandwidth(pnode->GetId(),
                                  ::GetSerializeSize(tx, PROTOCOL_VERSION));
        }
    }
};
```

## 🎯 Knowledge Check
1. How do you implement efficient transaction propagation?
2. What are the key components of inventory management?
3. How do you handle relay policies effectively?
4. What strategies exist for bandwidth optimization?
5. How do you manage peer statistics?

## 🔍 Common Issues and Solutions

### Issue 1: Propagation Delays
- Implement prioritization
- Use efficient routing
- Handle network congestion
- Optimize announcement intervals

### Issue 2: Bandwidth Management
- Implement rate limiting
- Use compression techniques
- Prioritize transactions
- Monitor network usage

### Issue 3: Relay Policies
- Handle policy violations
- Implement fee filtering
- Manage peer behavior
- Track relay statistics

## 📚 Additional Resources
- [Transaction Relay](https://github.com/bitcoin/bitcoin/blob/master/doc/p2p-relay.md)
- [Network Messages](https://github.com/bitcoin/bitcoin/blob/master/src/net_processing.cpp)
- [Relay Policies](https://github.com/bitcoin/bitcoin/blob/master/doc/policy/relay.md)
- [Bandwidth Management](https://github.com/bitcoin/bitcoin/blob/master/doc/reduce-traffic.md)

## 🎯 Next Steps
1. Study advanced relay patterns
2. Implement custom relay policies
3. Build bandwidth optimization tools
4. Contribute to relay improvements

Remember: Efficient transaction relay is crucial for network performance. Focus on propagation strategies, inventory management, and bandwidth optimization.
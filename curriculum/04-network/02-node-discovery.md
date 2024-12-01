# Bitcoin Core Node Discovery

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Implement efficient peer discovery mechanisms
- Handle DNS seed queries and responses
- Manage address propagation in the network
- Debug node discovery issues
- Optimize peer selection strategies

## Prerequisites
- Understanding of P2P network architecture (covered in 01-p2p-architecture.md)
- Knowledge of DNS protocols
- Familiarity with network programming
- Basic understanding of Bitcoin Core's network layer

## 🌟 Real-World Context
Think of Bitcoin's node discovery like finding new businesses:
- DNS Seeds = Business Directories
- Address Manager = Contact Book
- Peer Addresses = Business Cards
- Address Propagation = Word-of-mouth Referrals
- Connection Management = Building Business Relationships

## 🔍 Node Discovery Components

### 1. DNS Seed Implementation
```cpp
// Advanced DNS seed handling
class CDNSSeeder {
private:
    // Track seed query status
    struct SeedStatus {
        bool queried{false};
        int64_t lastQuery{0};
        std::vector<CAddress> addresses;
    };
    std::map<std::string, SeedStatus> mapSeeds;

public:
    std::vector<CAddress> QueryDNSSeeds() {
        std::vector<CAddress> vAddresses;
        
        // Query each seed with exponential backoff
        for (const auto& seed : Params().DNSSeeds()) {
            auto& status = mapSeeds[seed.host];
            
            // Check if we should query this seed
            int64_t now = GetTime();
            if (status.queried && 
                now - status.lastQuery < GetQueryBackoff(seed.host)) {
                continue;
            }
            
            try {
                // Perform DNS query
                std::vector<CNetAddr> vIPs = QueryDNS(seed.host);
                
                // Process results
                for (const auto& ip : vIPs) {
                    CAddress addr(CService(ip, Params().GetDefaultPort()));
                    addr.nTime = GetTime();
                    vAddresses.push_back(addr);
                }
                
                // Update status
                status.queried = true;
                status.lastQuery = now;
                status.addresses = vAddresses;
                
                LogPrint(BCLog::NET, "DNS Seed query successful: %s (%d addresses)\n",
                         seed.host, vAddresses.size());
                
            } catch (const std::runtime_error& e) {
                LogPrint(BCLog::NET, "DNS Seed query failed: %s (%s)\n",
                         seed.host, e.what());
            }
        }
        
        return vAddresses;
    }

private:
    int64_t GetQueryBackoff(const std::string& host) {
        // Implement exponential backoff
        auto& status = mapSeeds[host];
        int failures = status.addresses.empty() ? 1 : 0;
        return std::min(60 * 60, // Max 1 hour
                       15 * (1 << failures)); // Start at 15 seconds
    }
};
```

### 2. Address Manager
```cpp
// Advanced address management
class CAddrMan {
private:
    // Sophisticated address scoring
    struct AddrScore {
        double reliability{0.0};
        int64_t lastSuccess{0};
        int64_t lastTry{0};
        int attemptCount{0};
        bool local{false};
    };
    std::map<CAddress, AddrScore> mapAddrScores;

public:
    CAddress SelectTriedCollision() {
        AssertLockHeld(cs);
        
        if (nTried == 0)
            return CAddress();
            
        // Select from tried table with collision avoidance
        int collision = 0;
        int ret = -1;
        while (ret == -1) {
            collision = (collision + 1) % ADDRMAN_TRIED_COLLISION_SIZE;
            int pos = nKey % (ADDRMAN_TRIED_BUCKET_COUNT * ADDRMAN_BUCKET_SIZE);
            ret = SelectFromTried(pos, collision);
        }
        return vvTried[ret].addr;
    }

    void Good(const CAddress& addr) {
        LOCK(cs);
        
        auto& score = mapAddrScores[addr];
        score.lastSuccess = GetTime();
        score.reliability = std::min(1.0, score.reliability + 0.1);
        
        LogPrint(BCLog::NET, "Marking address as good: %s (reliability: %.2f)\n",
                 addr.ToString(), score.reliability);
                 
        // Move to "tried" table if appropriate
        MaybeMoveTried(addr, score);
    }
};
```

## 🛠️ Advanced Discovery Techniques

### 1. Intelligent Peer Selection
```cpp
// Optimized peer selection
class CPeerSelector {
public:
    std::vector<CAddress> SelectPeersToConnect(const std::vector<CAddress>& vAddr) {
        std::vector<CAddress> vSelected;
        
        // Score each address
        std::vector<std::pair<double, CAddress>> vScored;
        for (const auto& addr : vAddr) {
            double score = CalculateConnectionScore(addr);
            vScored.push_back({score, addr});
        }
        
        // Sort by score
        std::sort(vScored.begin(), vScored.end(),
                 [](const auto& a, const auto& b) { return a.first > b.first; });
        
        // Select top candidates
        for (size_t i = 0; i < std::min(MAX_OUTBOUND_CONNECTIONS, vScored.size()); i++) {
            vSelected.push_back(vScored[i].second);
        }
        
        return vSelected;
    }

private:
    double CalculateConnectionScore(const CAddress& addr) {
        // Consider multiple factors
        double score = 0.0;
        
        // Recent success rate
        score += GetSuccessRate(addr) * 0.4;
        
        // Network diversity
        score += GetDiversityScore(addr) * 0.3;
        
        // Service bits
        score += GetServiceScore(addr) * 0.2;
        
        // Recent activity
        score += GetActivityScore(addr) * 0.1;
        
        return score;
    }
};
```

### 2. Network Topology Analysis
```cpp
// Network topology monitoring
class CNetworkTopology {
private:
    struct NodeStats {
        std::set<CNetAddr> peers;
        std::map<uint256, int64_t> blockRelayTimes;
        std::map<uint256, int64_t> txRelayTimes;
    };
    std::map<NodeId, NodeStats> mapNodeStats;

public:
    void AnalyzeTopology() {
        LOCK(cs_vNodes);
        
        // Build network graph
        std::map<CNetAddr, std::set<CNetAddr>> adjacencyList;
        for (CNode* pnode : vNodes) {
            auto& stats = mapNodeStats[pnode->GetId()];
            adjacencyList[pnode->addr].insert(stats.peers.begin(), 
                                            stats.peers.end());
        }
        
        // Analyze connectivity
        AnalyzeConnectivity(adjacencyList);
        
        // Check for potential partitions
        DetectPartitions(adjacencyList);
        
        // Measure network diameter
        int diameter = CalculateNetworkDiameter(adjacencyList);
        
        LogPrint(BCLog::NET, "Network topology analysis: diameter=%d\n", diameter);
    }
};
```

## 📊 Monitoring and Debugging

### 1. Discovery Metrics
```cpp
// Advanced discovery monitoring
class CDiscoveryMonitor {
private:
    struct DiscoveryMetrics {
        std::atomic<uint64_t> dnsQueries{0};
        std::atomic<uint64_t> dnsSuccesses{0};
        std::atomic<uint64_t> addressesReceived{0};
        std::atomic<uint64_t> addressesRelayed{0};
        std::atomic<uint64_t> connectAttempts{0};
        std::atomic<uint64_t> connectSuccesses{0};
    };
    DiscoveryMetrics metrics;

public:
    void UpdateMetrics() {
        LogPrint(BCLog::NET, "Discovery metrics:\n"
                "- DNS queries: %d (success rate: %.2f%%)\n"
                "- Addresses: received=%d, relayed=%d\n"
                "- Connections: attempts=%d, successes=%d\n",
                metrics.dnsQueries.load(),
                (double)metrics.dnsSuccesses.load() / metrics.dnsQueries.load() * 100,
                metrics.addressesReceived.load(),
                metrics.addressesRelayed.load(),
                metrics.connectAttempts.load(),
                metrics.connectSuccesses.load());
    }
};
```

### 2. Debug Logging
```cpp
// Enhanced discovery debugging
void LogDiscoveryEvent(const std::string& event, 
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
        LogPrintf("DISCOVERY WARNING: %s\n", logEntry);
        AlertNotify(logEntry);
    } else {
        LogPrint(BCLog::NET, "%s\n", logEntry);
    }
}
```

## 🎯 Knowledge Check
1. How does Bitcoin Core's DNS seeding mechanism work?
2. What factors influence peer selection?
3. How do you implement effective address management?
4. What metrics are important for monitoring node discovery?
5. How do you debug discovery issues?

## 🔍 Common Issues and Solutions

### Issue 1: Poor Peer Discovery
- Implement multiple discovery methods
- Use DNS seeds effectively
- Maintain good address management
- Monitor discovery metrics

### Issue 2: Network Partitioning
- Monitor network topology
- Implement diverse peer selection
- Use multiple DNS seeds
- Handle address propagation properly

## 📚 Additional Resources
- [Bitcoin Core DNS Seeds](https://github.com/bitcoin/bitcoin/blob/master/doc/dnsseed-policy.md)
- [P2P Network Guide](https://github.com/bitcoin/bitcoin/blob/master/doc/p2p-network.md)
- [Address Management BIP](https://github.com/bitcoin/bips/blob/master/bip-0155.mediawiki)
- [Network Debug Guide](https://github.com/bitcoin/bitcoin/blob/master/doc/debug.md)

## 🎯 Next Steps
1. Study advanced network topology
2. Implement custom discovery mechanisms
3. Build network monitoring tools
4. Contribute to network improvements

Remember: Effective node discovery is crucial for maintaining a healthy Bitcoin network. Focus on reliability, diversity, and proper monitoring.
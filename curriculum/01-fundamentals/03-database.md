# Bitcoin Core Database Management

## 🎯 Database Architecture Overview

### Core Components
```cpp
// Key database components from src/dbwrapper.h
class CDBWrapper {
    // LevelDB database instance
    std::unique_ptr<leveldb::DB> pdb;
    
    // Custom comparator for Bitcoin
    std::unique_ptr<const leveldb::Comparator> pcustomComparator;
    
    // Database options
    leveldb::Options options;
    
    // Read options with verification
    leveldb::ReadOptions readoptions;
    
    // Iterator options
    leveldb::ReadOptions iteroptions;
};
```

## 🔍 Query Optimization Examples

### 1. UTXO Set Access Optimization
```cpp
// Optimized UTXO lookup
class CCoinsViewDB : public CCoinsView {
private:
    // LRU cache for frequent lookups
    std::unique_ptr<CCoinsCache> cache;
    
    // Batch operations for bulk updates
    CDBBatch batch;
    
public:
    bool GetCoin(const COutPoint& outpoint, Coin& coin) const {
        // Check cache first
        if (cache->GetCoin(outpoint, coin))
            return true;
            
        // If not in cache, check database
        if (!db.Read(std::make_pair(DB_COINS, outpoint), coin))
            return false;
            
        // Add to cache for future lookups
        cache->AddCoin(outpoint, std::move(coin), false);
        return true;
    }
    
    void BatchWrite(CCoinsMap& mapCoins) {
        // Prepare batch operation
        for (CCoinsMap::iterator it = mapCoins.begin(); it != mapCoins.end();) {
            if (it->second.flags & CCoinsCacheEntry::DIRTY) {
                if (it->second.coin.IsSpent())
                    batch.Erase(std::make_pair(DB_COINS, it->first));
                else
                    batch.Write(std::make_pair(DB_COINS, it->first), it->second.coin);
            }
            it = mapCoins.erase(it);
        }
        
        // Execute batch
        db.WriteBatch(batch);
    }
};
```

### 2. Block Index Optimization
```cpp
// Optimized block index access
class CBlockTreeDB : public CDBWrapper {
public:
    bool ReadBlockFileInfo(int nFile, CBlockFileInfo& info) {
        // Use bloom filter for quick rejection
        if (!HasKey(std::make_pair(DB_BLOCK_FILES, nFile)))
            return false;
            
        return Read(std::make_pair(DB_BLOCK_FILES, nFile), info);
    }
    
    bool WriteReindexing(bool fReindexing) {
        if (fReindexing)
            return Write(DB_REINDEX_FLAG, '1');
        else
            return Erase(DB_REINDEX_FLAG);
    }
    
    // Batch block writes for better performance
    CDBBatch batch;
    for (const auto& update : mapDirtyBlockIndex) {
        CDiskBlockIndex diskindex(update.second);
        batch.Write(std::make_pair(DB_BLOCK_INDEX, update.first), diskindex);
    }
    return WriteBatch(batch);
};
```

## 🛠️ Performance Tuning

### 1. Database Configuration
```cpp
// Optimized database settings
leveldb::Options GetOptions() {
    leveldb::Options options;
    
    // Set cache size based on system memory
    size_t nTotalCache = (GetSystemMemory() * 0.15) / 2;
    options.block_cache = leveldb::NewLRUCache(nTotalCache);
    
    // Optimize for HDD or SSD
    if (IsHDDStorage()) {
        options.write_buffer_size = 256 * 1024 * 1024;  // 256MB
        options.max_file_size = 512 * 1024 * 1024;      // 512MB
    } else {
        options.write_buffer_size = 64 * 1024 * 1024;   // 64MB
        options.max_file_size = 128 * 1024 * 1024;      // 128MB
    }
    
    // Enable compression
    options.compression = leveldb::kSnappyCompression;
    
    // Optimize for read performance
    options.filter_policy.reset(leveldb::NewBloomFilterPolicy(10));
    
    return options;
}
```

### 2. Index Optimization
```cpp
// Optimized index management
class CBlockIndexWorkComparator {
public:
    bool operator()(const CBlockIndex* pa, const CBlockIndex* pb) const {
        // First sort by chain work
        if (pa->nChainWork > pb->nChainWork) return false;
        if (pa->nChainWork < pb->nChainWork) return true;
        
        // Then by block hash for determinism
        return pa->GetBlockHash() > pb->GetBlockHash();
    }
};

// Use optimized index structure
std::set<CBlockIndex*, CBlockIndexWorkComparator> setBlockIndexCandidates;
```

## 📊 Performance Monitoring

### 1. Database Metrics
```cpp
// Database performance monitoring
class CDBPerformance {
private:
    struct Metrics {
        std::atomic<uint64_t> reads{0};
        std::atomic<uint64_t> writes{0};
        std::atomic<uint64_t> deletes{0};
        std::atomic<uint64_t> iterations{0};
        std::atomic<uint64_t> cache_hits{0};
        std::atomic<uint64_t> cache_misses{0};
    };
    Metrics metrics;
    
public:
    void UpdateMetrics(const DBOperation& op) {
        switch (op.type) {
            case DBOperation::READ:
                metrics.reads++;
                if (op.cache_hit)
                    metrics.cache_hits++;
                else
                    metrics.cache_misses++;
                break;
            case DBOperation::WRITE:
                metrics.writes++;
                break;
            case DBOperation::DELETE:
                metrics.deletes++;
                break;
            case DBOperation::ITERATE:
                metrics.iterations++;
                break;
        }
        
        // Log significant events
        if (ShouldLogMetrics()) {
            LogPrint(BCLog::DB, "DB Metrics: reads=%u writes=%u deletes=%u "
                               "iterations=%u cache_hits=%u cache_misses=%u\n",
                    metrics.reads.load(),
                    metrics.writes.load(),
                    metrics.deletes.load(),
                    metrics.iterations.load(),
                    metrics.cache_hits.load(),
                    metrics.cache_misses.load());
        }
    }
};
```

### 2. Performance Alerts
```cpp
// Database performance monitoring
class CDBMonitor {
private:
    // Performance thresholds
    const int64_t WRITE_LATENCY_THRESHOLD = 1000;    // 1 second
    const int64_t READ_LATENCY_THRESHOLD = 100;      // 100 ms
    const size_t CACHE_MISS_THRESHOLD = 1000;        // per minute
    
public:
    void MonitorPerformance() {
        // Track operation latency
        int64_t start = GetTimeMicros();
        bool success = db.Write(key, value);
        int64_t duration = GetTimeMicros() - start;
        
        // Check for slow operations
        if (duration > WRITE_LATENCY_THRESHOLD) {
            LogPrintf("WARNING: Slow database write: %d µs\n", duration);
            AlertNotify(strprintf("Slow database write detected: %d µs", duration));
        }
        
        // Monitor cache efficiency
        if (cache_misses > CACHE_MISS_THRESHOLD) {
            LogPrintf("WARNING: High cache miss rate: %d misses/minute\n", 
                     cache_misses);
            AlertNotify(strprintf("High cache miss rate: %d/minute", cache_misses));
        }
    }
};
```

## 🔄 Backup and Recovery

### 1. Database Backup
```cpp
// Automated backup system
class CDBBackup {
public:
    bool CreateBackup(const fs::path& backup_dir) {
        // Ensure atomicity
        LOCK(cs_db);
        
        // Create backup directory if it doesn't exist
        if (!fs::exists(backup_dir))
            fs::create_directories(backup_dir);
            
        // Generate backup filename with timestamp
        std::string timestamp = DateTimeStrFormat("%Y%m%d_%H%M%S", GetTime());
        fs::path backup_file = backup_dir / strprintf("chainstate_%s.bak", timestamp);
        
        // Perform backup
        leveldb::Snapshot* snapshot = db.GetSnapshot();
        bool success = db.Backup(backup_file.string());
        db.ReleaseSnapshot(snapshot);
        
        if (success) {
            LogPrintf("Database backup created: %s\n", backup_file.string());
        } else {
            LogPrintf("ERROR: Database backup failed\n");
        }
        
        return success;
    }
};
```

### 2. Recovery Procedures
```cpp
// Database recovery system
class CDBRecovery {
public:
    bool RecoverFromCorruption() {
        // Try automatic recovery first
        leveldb::Status status = leveldb::RepairDB(dbpath, options);
        if (status.ok()) {
            LogPrintf("Database automatically repaired\n");
            return true;
        }
        
        // If automatic repair fails, try backup restoration
        if (HasValidBackup()) {
            return RestoreFromBackup();
        }
        
        // If all else fails, reindex
        LogPrintf("Database recovery failed. Reindex required.\n");
        return false;
    }
    
private:
    bool RestoreFromBackup() {
        // Find latest valid backup
        fs::path latest_backup = FindLatestBackup();
        if (latest_backup.empty())
            return false;
            
        // Restore from backup
        LogPrintf("Restoring from backup: %s\n", latest_backup.string());
        return db.RestoreFromBackup(latest_backup.string());
    }
};
```

## 🔒 Security Considerations

### 1. Data Integrity
```cpp
// Data integrity verification
class CDBVerifier {
public:
    bool VerifyDatabase() {
        // Verify all database entries
        std::unique_ptr<CDBIterator> it(db.NewIterator());
        for (it->SeekToFirst(); it->Valid(); it->Next()) {
            if (!VerifyEntry(it->Key(), it->Value()))
                return false;
        }
        
        // Verify chain state
        return VerifyChainState();
    }
    
private:
    bool VerifyEntry(const leveldb::Slice& key, const leveldb::Slice& value) {
        // Check key-value integrity
        if (!VerifyChecksum(key, value))
            return false;
            
        // Verify data format
        if (!VerifyDataFormat(key, value))
            return false;
            
        return true;
    }
};
```

### 2. Access Control
```cpp
// Database access control
class CDBAccessControl {
private:
    // Track access patterns
    std::map<std::string, AccessStats> access_log;
    
public:
    bool CheckAccess(const std::string& operation, const std::string& key) {
        // Rate limiting
        if (IsRateLimited(operation))
            return false;
            
        // Access logging
        LogAccess(operation, key);
        
        // Suspicious pattern detection
        if (DetectSuspiciousPattern(operation, key)) {
            LogPrintf("WARNING: Suspicious database access pattern detected\n");
            return false;
        }
        
        return true;
    }
};
```

Remember: Database performance is crucial for Bitcoin Core's operation. Always consider the impact of changes on database performance and reliability.
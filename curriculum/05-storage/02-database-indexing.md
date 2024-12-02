# Bitcoin Database Indexing and Optimization

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Understand Bitcoin Core's indexing strategies
- Implement efficient blockchain indexes
- Optimize database performance
- Handle index maintenance
- Build custom indexing solutions

## Prerequisites
- Understanding of blockchain storage (covered in 01-blockchain-storage.md)
- Familiarity with database concepts
- Experience with Node.js/TypeScript
- Basic knowledge of data structures
- Understanding of Bitcoin RPC interface

## 🌟 Real-World Analogy
Think of blockchain indexing like a library's catalog system:
- Block Index = Book Catalog
- Transaction Index = Chapter Index
- Address Index = Subject Index
- UTXO Set = Current Inventory
- Database = Library Storage System
- Pruning = Archive Management

## Index Components

### 1. Core Index Types

```cpp
// Block index entry
struct CBlockIndex {
    uint256 hashBlock;      // Block hash
    uint256 hashPrev;       // Previous block hash
    uint32_t nFile;         // File number
    uint32_t nDataPos;      // Position in file
    uint32_t nUndoPos;      // Position of undo data
    uint32_t nHeight;       // Block height
    uint32_t nTx;          // Number of transactions
    uint32_t nStatus;      // Block status flags
};

// Transaction index entry
struct CTxIndex {
    CDiskTxPos pos;        // Transaction location
    std::vector<CDiskTxPos> vSpent;  // Spent outputs
};
```

### 2. Index Management

```typescript
class IndexManager {
    private readonly indexes: Map<string, BlockchainIndex>;
    private readonly db: LevelDB;

    async createIndex(name: string, config: IndexConfig): Promise<void> {
        // Create new index
        const index = new BlockchainIndex(name, config);
        await index.initialize();
        
        // Build initial index
        await this.buildIndex(index);
        
        // Register index
        this.indexes.set(name, index);
    }

    private async buildIndex(index: BlockchainIndex): Promise<void> {
        const startHeight = await index.getHeight();
        const currentHeight = await this.chain.getHeight();
        
        console.log(`Building index ${index.name} from height ${startHeight}`);
        
        for (let height = startHeight; height <= currentHeight; height++) {
            const block = await this.chain.getBlockByHeight(height);
            await index.processBlock(block);
            
            if (height % 1000 === 0) {
                console.log(`Indexed ${height}/${currentHeight} blocks`);
            }
        }
    }
}
```

## 💻 Hands-on Implementation

### 1. Custom Index Implementation

```typescript
class AddressIndex extends BlockchainIndex {
    private readonly addressMap: Map<string, AddressData>;
    private readonly batchSize = 1000;

    async processBlock(block: Block): Promise<void> {
        const batch = this.db.batch();
        
        for (const tx of block.transactions) {
            // Index inputs
            for (const input of tx.inputs) {
                if (!input.isCoinbase()) {
                    const address = await this.getInputAddress(input);
                    await this.updateAddressData(address, input, batch);
                }
            }
            
            // Index outputs
            for (const [index, output] of tx.outputs.entries()) {
                const address = this.getOutputAddress(output);
                await this.updateAddressData(address, output, batch);
            }
        }
        
        await batch.write();
    }

    private async updateAddressData(
        address: string,
        data: TxData,
        batch: LevelDBBatch
    ): Promise<void> {
        const addressData = await this.getAddressData(address);
        addressData.update(data);
        batch.put(this.getAddressKey(address), addressData.serialize());
    }
}
```

## 🚀 RPC Integration

### 1. Index Management and Queries

```javascript
const Client = require('bitcoin-core');
const client = new Client({
    network: 'regtest',
    username: 'your_username',
    password: 'your_secure_password',
    port: 18443
});

async function manageIndexes() {
    try {
        // Get index information
        const indexInfo = await client.getIndexInfo();
        console.log('Index Status:', {
            txindex: indexInfo.txindex,
            addressindex: indexInfo.addressindex,
            timestampindex: indexInfo.timestampindex
        });

        // Query indexed data
        const address = 'mvbnrCX3bg1cDRUu8pkecrvP6vQkSLDSou';
        const addressHistory = await client.getAddressHistory(address);
        console.log('Address History:', addressHistory.map(entry => ({
            txid: entry.txid,
            height: entry.height,
            amount: entry.amount
        })));

        // Get transaction index data
        const txid = '1234...';
        const txInfo = await client.getRawTransaction(txid, true);
        console.log('Transaction Details:', {
            blockhash: txInfo.blockhash,
            confirmations: txInfo.confirmations,
            time: new Date(txInfo.time * 1000).toISOString()
        });
    } catch (error) {
        console.error('Error:', error);
    }
}
```

### 2. Index Performance Monitoring

```javascript
async function monitorIndexPerformance() {
    try {
        // Get database statistics
        const dbStats = await client.gettxoutsetinfo();
        console.log('UTXO Set Stats:', {
            height: dbStats.height,
            transactions: dbStats.transactions,
            size: dbStats.size,
            totalAmount: dbStats.total_amount
        });

        // Monitor index size
        const indexSizes = await getIndexSizes();
        console.log('Index Sizes:', {
            blockindex: indexSizes.blockindex,
            txindex: indexSizes.txindex,
            addressindex: indexSizes.addressindex
        });

        // Check index performance
        const perfMetrics = await measureQueryPerformance();
        console.log('Query Performance:', {
            blockLookup: perfMetrics.blockLookup,
            txLookup: perfMetrics.txLookup,
            addressLookup: perfMetrics.addressLookup
        });
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## 🛠️ Practice Project: Advanced Index System

Let's build a comprehensive indexing system with multiple index types.

### Project Structure

```bash
blockchain-indexer/
├── src/
│   ├── indexes/
│   │   ├── base.index.ts
│   │   ├── address.index.ts
│   │   ├── transaction.index.ts
│   │   └── timestamp.index.ts
│   ├── managers/
│   │   ├── index.manager.ts
│   │   └── sync.manager.ts
│   ├── services/
│   │   ├── query.service.ts
│   │   └── maintenance.service.ts
│   └── utils/
│       ├── db.utils.ts
│       └── validation.utils.ts
├── tests/
└── package.json
```

### Implementation Example

```typescript
// src/indexes/base.index.ts
abstract class BaseIndex {
    protected readonly db: LevelDB;
    protected readonly name: string;
    private syncing: boolean = false;

    constructor(name: string, dbPath: string) {
        this.name = name;
        this.db = new LevelDB(dbPath);
    }

    async sync(): Promise<void> {
        if (this.syncing) return;
        this.syncing = true;

        try {
            const currentHeight = await this.getCurrentHeight();
            const indexHeight = await this.getIndexHeight();

            if (currentHeight > indexHeight) {
                await this.syncToHeight(currentHeight);
            }
        } finally {
            this.syncing = false;
        }
    }

    abstract processBlock(block: Block): Promise<void>;
    abstract revertBlock(block: Block): Promise<void>;
    abstract query(params: QueryParams): Promise<QueryResult>;
}

// src/services/query.service.ts
class QueryService {
    private readonly indexes: Map<string, BaseIndex>;
    private readonly cache: QueryCache;

    async query(indexName: string, params: QueryParams): Promise<QueryResult> {
        const index = this.indexes.get(indexName);
        if (!index) throw new Error(`Index ${indexName} not found`);

        // Check cache
        const cachedResult = await this.cache.get(this.getCacheKey(indexName, params));
        if (cachedResult) return cachedResult;

        // Perform query
        const result = await index.query(params);

        // Update cache
        await this.cache.set(this.getCacheKey(indexName, params), result);

        return result;
    }

    private getCacheKey(indexName: string, params: QueryParams): string {
        return `${indexName}:${JSON.stringify(params)}`;
    }
}
```

## 🎯 Knowledge Check
1. Why are different index types needed?
2. How does index synchronization work?
3. What are the performance implications of indexing?
4. How are indexes maintained during reorgs?
5. What are the tradeoffs of different index structures?

## 🔍 Common Issues and Solutions

### Issue 1: Index Synchronization
- Initial sync performance
- Handling interruptions
- Maintaining consistency
- Resource management

### Issue 2: Query Performance
- Index structure optimization
- Caching strategies
- Query planning
- Resource limits

## 📚 Additional Resources
- [Bitcoin Core Index Documentation](https://github.com/bitcoin/bitcoin/blob/master/doc/indexes.md)
- [LevelDB Documentation](https://github.com/google/leveldb)
- [Database Performance Tuning](https://github.com/bitcoin/bitcoin/blob/master/doc/performance-optimizations.md)
- [Index Design Patterns](https://github.com/bitcoin/bitcoin/blob/master/doc/design/indexes.md)

## Next Steps
1. Study advanced indexing patterns
2. Implement custom indexes
3. Optimize query performance
4. Build monitoring tools
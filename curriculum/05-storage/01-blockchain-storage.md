# Bitcoin Blockchain Storage Architecture

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Understand Bitcoin's blockchain storage structure
- Implement efficient block storage mechanisms
- Handle chain reorganizations
- Manage storage space effectively
- Optimize data access patterns

## Prerequisites
- Understanding of block structure (covered in Module 1)
- Familiarity with database concepts
- Experience with Node.js/TypeScript
- Basic knowledge of file systems
- Understanding of Bitcoin RPC interface

## 🌟 Real-World Analogy
Think of blockchain storage like a library's archival system:
- Blocks = Books
- Block Files = Shelves
- Index = Card Catalog
- Pruning = Archive Management
- UTXO Set = Current Inventory
- Chain State = Library Status

## Storage Components

### 1. Block Storage Structure

```cpp
// Block file format
struct BlockFile {
    uint32_t magic;      // Network identifier
    uint32_t size;       // Block size
    uint8_t* rawblock;   // Block data
};

// Block index entry
struct BlockIndex {
    uint256 blockHash;
    int32_t height;
    uint32_t nFile;      // File number
    uint32_t nDataPos;   // Position in file
    uint32_t nUndoPos;   // Position of undo data
    CBlockHeader header;
};
```

### 2. Database Layout

```typescript
// Database structure representation
interface ChainDB {
    // 'b' prefix: Block index
    'b' + blockHash: {
        height: number;
        status: BlockStatus;
        txCount: number;
        file: number;
        dataPos: number;
        undoPos: number;
        header: BlockHeader;
    };

    // 'f' prefix: File info
    'f' + fileNumber: {
        blocks: number;
        size: number;
        undoSize: number;
        earliestTime: number;
        latestTime: number;
    };

    // 'r' prefix: Reorg tracking
    'r' + height: BlockHash[];
}
```

## 💻 Hands-on Implementation

### 1. Block Storage Manager

```typescript
// src/storage/block.manager.ts
class BlockStorageManager {
    private readonly dataDir: string;
    private readonly maxFileSize: number = 128 * 1024 * 1024; // 128MB
    private currentFile: number;

    constructor(dataDir: string) {
        this.dataDir = dataDir;
        this.currentFile = this.findLatestFile();
    }

    async writeBlock(block: Block): Promise<BlockLocation> {
        const serializedBlock = this.serializeBlock(block);
        
        // Check if new file needed
        if (this.shouldCreateNewFile(serializedBlock.length)) {
            this.currentFile = await this.createNewBlockFile();
        }
        
        // Write block to file
        const location = await this.writeToFile(
            this.currentFile,
            serializedBlock
        );
        
        // Update index
        await this.updateBlockIndex(block, location);
        
        return location;
    }

    private serializeBlock(block: Block): Buffer {
        const magic = Buffer.alloc(4);
        magic.writeUInt32LE(NETWORK_MAGIC, 0);
        
        const size = Buffer.alloc(4);
        size.writeUInt32LE(block.size, 0);
        
        return Buffer.concat([
            magic,
            size,
            block.serialize()
        ]);
    }

    async readBlock(location: BlockLocation): Promise<Block> {
        const file = await fs.open(
            this.getBlockFilePath(location.file),
            'r'
        );
        
        try {
            const buffer = Buffer.alloc(location.size);
            await file.read(buffer, 0, location.size, location.position);
            return Block.deserialize(buffer);
        } finally {
            await file.close();
        }
    }
}
```

## 🚀 RPC Integration

### 1. Block Data Management

```javascript
const Client = require('bitcoin-core');
const client = new Client({
    network: 'regtest',
    username: 'your_username',
    password: 'your_secure_password',
    port: 18443
});

async function manageBlockStorage() {
    try {
        // Get blockchain info
        const info = await client.getBlockchainInfo();
        console.log('Chain Status:', {
            blocks: info.blocks,
            headers: info.headers,
            bestBlockHash: info.bestblockhash,
            size: info.size_on_disk,
            pruned: info.pruned,
            pruneHeight: info.pruneheight
        });

        // Get block storage stats
        const stats = await client.getBlockStats(info.blocks);
        console.log('Latest Block Stats:', {
            avgFee: stats.avgfee,
            avgTxSize: stats.avgtxsize,
            totalTx: stats.txs,
            totalSize: stats.total_size,
            totalWeight: stats.total_weight
        });

        // Analyze block files
        const usage = await client.getBlockFileInfo();
        console.log('Block File Info:', usage.map(file => ({
            fileNumber: file.n,
            blocks: file.nblocks,
            size: file.size,
            usePercent: file.used,
            minHeight: file.minheight,
            maxHeight: file.maxheight
        })));
    } catch (error) {
        console.error('Error:', error);
    }
}
```

### 2. Storage Management Tools

```javascript
async function manageChainData() {
    try {
        // Prune blockchain data
        if (await shouldPrune()) {
            await client.pruneBlockchain();
            console.log('Pruned blockchain data');
        }

        // Verify blockchain storage
        const verificationProgress = await client.verifyChain(3); // Level 3 verification
        console.log('Chain verification:', verificationProgress ? 'passed' : 'failed');

        // Get database stats
        const dbInfo = await client.gettxoutsetinfo();
        console.log('UTXO Set Stats:', {
            height: dbInfo.height,
            transactions: dbInfo.transactions,
            size: dbInfo.size,
            totalAmount: dbInfo.total_amount
        });
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## 🛠️ Practice Project: Chain Storage System

Let's build a complete blockchain storage system with indexing.

### Project Structure

```bash
chain-storage/
├── src/
│   ├── storage/
│   │   ├── block.storage.ts
│   │   ├── index.storage.ts
│   │   └── undo.storage.ts
│   ├── managers/
│   │   ├── file.manager.ts
│   │   └── pruning.manager.ts
│   ├── services/
│   │   ├── storage.service.ts
│   │   └── verification.service.ts
│   └── utils/
│       ├── serialization.ts
│       └── validation.ts
├── tests/
└── package.json
```

### Implementation Example

```typescript
// src/services/storage.service.ts
class ChainStorageService {
    private readonly blockStorage: BlockStorageManager;
    private readonly indexStorage: IndexStorageManager;
    private readonly pruningManager: PruningManager;

    constructor(dataDir: string) {
        this.blockStorage = new BlockStorageManager(dataDir);
        this.indexStorage = new IndexStorageManager(dataDir);
        this.pruningManager = new PruningManager(dataDir);
    }

    async storeBlock(block: Block): Promise<void> {
        // Store block data
        const location = await this.blockStorage.writeBlock(block);
        
        // Update indexes
        await this.indexStorage.indexBlock(block, location);
        
        // Check pruning
        await this.pruningManager.checkPruning();
    }

    async retrieveBlock(hash: string): Promise<Block> {
        // Get block location
        const location = await this.indexStorage.getBlockLocation(hash);
        if (!location) {
            throw new Error('Block not found');
        }
        
        // Read block data
        return this.blockStorage.readBlock(location);
    }

    async verifyStorage(): Promise<boolean> {
        const verifier = new StorageVerifier(this.dataDir);
        return verifier.verifyAll();
    }
}
```

## 🎯 Knowledge Check
1. How is blockchain data physically stored?
2. What are the benefits of block file organization?
3. How does pruning affect storage?
4. What indexes are essential for performance?
5. How are chain reorganizations handled in storage?

## 🔍 Common Issues and Solutions

### Issue 1: Storage Growth
- Pruning strategies
- Index optimization
- Space management
- File system limits

### Issue 2: Data Corruption
- Verification methods
- Recovery procedures
- Backup strategies
- Integrity checks

## 📚 Additional Resources
- [Bitcoin Core Storage Documentation](https://github.com/bitcoin/bitcoin/blob/master/doc/files.md)
- [Block Storage Format](https://developer.bitcoin.org/reference/block_chain.html)
- [Database Design in Bitcoin Core](https://github.com/bitcoin/bitcoin/blob/master/doc/design/databases.md)
- [Pruning Design](https://github.com/bitcoin/bitcoin/blob/master/doc/release-notes/release-notes-0.11.0.md#block-file-pruning)

## Next Steps
1. Study database indexing strategies
2. Implement advanced pruning
3. Build backup solutions
4. Create storage monitoring tools
``` 
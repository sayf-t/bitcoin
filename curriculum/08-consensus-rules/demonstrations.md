# Consensus Rules Demonstrations

## Demonstration 1: Block Validation
```javascript
const { BitcoinCore } = require('bitcoin-core');
const client = new BitcoinCore({
    network: 'regtest',
    username: 'rpcuser',
    password: 'rpcpassword'
});

async function demonstrateBlockValidation() {
    try {
        // Generate a new block
        const minerAddress = await client.getNewAddress();
        const blockHashes = await client.generateToAddress(1, minerAddress);
        const blockHash = blockHashes[0];
        
        // Get block details
        const block = await client.getBlock(blockHash, 2);
        
        console.log('Block Validation Demo:');
        console.log('---------------------');
        console.log(`Block Hash: ${block.hash}`);
        console.log(`Height: ${block.height}`);
        console.log(`Version: ${block.version}`);
        console.log(`Previous Block: ${block.previousblockhash}`);
        console.log(`Merkle Root: ${block.merkleroot}`);
        console.log(`Timestamp: ${new Date(block.time * 1000).toISOString()}`);
        console.log(`Difficulty: ${block.difficulty}`);
        console.log(`Nonce: ${block.nonce}`);
        console.log(`Transaction Count: ${block.tx.length}`);
        
        // Verify block validity
        const chainInfo = await client.getBlockchainInfo();
        console.log('\nBlock Validity:');
        console.log(`Chain Height: ${chainInfo.blocks}`);
        console.log(`Block Confirmations: ${block.confirmations}`);
        console.log(`Is Valid: ${block.height <= chainInfo.blocks}`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 2: Difficulty Adjustment
```javascript
async function demonstrateDifficultyAdjustment() {
    try {
        // Get current difficulty
        const initialInfo = await client.getBlockchainInfo();
        console.log('Initial Difficulty:', initialInfo.difficulty);
        
        // Generate blocks to trigger difficulty adjustment
        const minerAddress = await client.getNewAddress();
        console.log('\nGenerating blocks...');
        
        // Generate blocks in batches to observe difficulty changes
        for (let i = 0; i < 3; i++) {
            await client.generateToAddress(100, minerAddress);
            const info = await client.getBlockchainInfo();
            console.log(`\nAfter ${(i + 1) * 100} blocks:`);
            console.log(`Difficulty: ${info.difficulty}`);
            console.log(`Chain Work: ${info.chainwork}`);
        }
        
        // Get detailed mining info
        const miningInfo = await client.getMiningInfo();
        console.log('\nMining Information:');
        console.log(`Network Hashps: ${miningInfo.networkhashps}`);
        console.log(`Current Bits: ${miningInfo.bits}`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 3: Chain Reorganization
```javascript
async function demonstrateChainReorg() {
    try {
        // Create two competing chains
        const minerAddress1 = await client.getNewAddress();
        const minerAddress2 = await client.getNewAddress();
        
        // Get initial chain tip
        const initialTip = await client.getBestBlockHash();
        console.log('Initial Chain Tip:', initialTip);
        
        // Generate competing chains
        console.log('\nGenerating competing chains...');
        
        // Chain 1
        await client.generateToAddress(3, minerAddress1);
        const chain1Tip = await client.getBestBlockHash();
        console.log('Chain 1 Tip:', chain1Tip);
        
        // Invalidate chain 1 tip to simulate reorg
        await client.invalidateBlock(chain1Tip);
        
        // Chain 2 (higher work)
        await client.generateToAddress(4, minerAddress2);
        const chain2Tip = await client.getBestBlockHash();
        console.log('Chain 2 Tip:', chain2Tip);
        
        // Check chain state
        const chainInfo = await client.getBlockchainInfo();
        console.log('\nChain State After Reorg:');
        console.log(`Height: ${chainInfo.blocks}`);
        console.log(`Best Block: ${chainInfo.bestblockhash}`);
        console.log(`Chain Work: ${chainInfo.chainwork}`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 4: Headers-First Sync
```javascript
async function demonstrateHeadersSync() {
    try {
        // Get initial sync state
        const initialHeaders = await client.getBlockchainInfo();
        console.log('Initial Headers:', initialHeaders.headers);
        
        // Generate new blocks
        const minerAddress = await client.getNewAddress();
        await client.generateToAddress(10, minerAddress);
        
        // Get headers
        const headers = [];
        let currentHash = await client.getBestBlockHash();
        
        for (let i = 0; i < 10; i++) {
            const block = await client.getBlock(currentHash);
            headers.unshift({
                hash: block.hash,
                height: block.height,
                time: block.time
            });
            currentHash = block.previousblockhash;
        }
        
        console.log('\nHeader Chain:');
        headers.forEach(header => {
            console.log(`Height ${header.height}: ${header.hash}`);
        });
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 5: Chain State Management
```javascript
async function demonstrateChainState() {
    try {
        // Get initial chain state
        const initialState = await client.getBlockchainInfo();
        console.log('Initial Chain State:');
        console.log(`Height: ${initialState.blocks}`);
        console.log(`Chain Work: ${initialState.chainwork}`);
        console.log(`Verification Progress: ${initialState.verificationprogress}`);
        
        // Generate new blocks
        const minerAddress = await client.getNewAddress();
        await client.generateToAddress(5, minerAddress);
        
        // Get updated state
        const updatedState = await client.getBlockchainInfo();
        console.log('\nUpdated Chain State:');
        console.log(`Height: ${updatedState.blocks}`);
        console.log(`Chain Work: ${updatedState.chainwork}`);
        console.log(`Verification Progress: ${updatedState.verificationprogress}`);
        
        // Get UTXO statistics
        const utxoStats = await client.getUtxoStats();
        console.log('\nUTXO Set Statistics:');
        console.log(`Total Amount: ${utxoStats.total_amount} BTC`);
        console.log(`Transaction Count: ${utxoStats.txouts}`);
        console.log(`Total Size: ${utxoStats.total_size} bytes`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Running the Demonstrations

1. Start Bitcoin Core in regtest mode:
```bash
bitcoind -regtest -daemon
```

2. Install dependencies:
```bash
npm install bitcoin-core
```

3. Run the demonstrations:
```javascript
async function runDemonstrations() {
    console.log('Running Block Validation Demo...');
    await demonstrateBlockValidation();
    
    console.log('\nRunning Difficulty Adjustment Demo...');
    await demonstrateDifficultyAdjustment();
    
    console.log('\nRunning Chain Reorg Demo...');
    await demonstrateChainReorg();
    
    console.log('\nRunning Headers Sync Demo...');
    await demonstrateHeadersSync();
    
    console.log('\nRunning Chain State Demo...');
    await demonstrateChainState();
}

runDemonstrations().catch(console.error);
```

## Expected Output
Each demonstration will provide detailed output about different aspects of consensus rules:
- Block validation details
- Difficulty adjustments
- Chain reorganizations
- Headers synchronization
- Chain state management

## Troubleshooting
- Ensure Bitcoin Core is running in regtest mode
- Check RPC credentials are correct
- Monitor debug.log for errors
- Verify sufficient blocks for difficulty adjustment
- Check disk space for chain state
``` 
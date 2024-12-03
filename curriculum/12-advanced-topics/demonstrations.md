# Advanced Topics Demonstrations

## Demonstration 1: Mining and Block Creation
```javascript
const { BitcoinCore } = require('bitcoin-core');
const client = new BitcoinCore({
    network: 'regtest',
    username: 'rpcuser',
    password: 'rpcpassword'
});

async function demonstrateMining() {
    try {
        // Get block template
        const template = await client.getBlockTemplate({
            rules: ['segwit']
        });
        
        console.log('Block Template:');
        console.log('---------------');
        console.log(`Previous Block: ${template.previousblockhash}`);
        console.log(`Target Bits: ${template.bits}`);
        console.log(`Height: ${template.height}`);
        console.log(`Transactions: ${template.transactions.length}`);
        
        // Analyze template transactions
        let totalFees = 0;
        template.transactions.forEach(tx => {
            totalFees += tx.fee;
        });
        
        console.log(`\nTotal Fees: ${totalFees} BTC`);
        console.log(`Block Reward: ${template.coinbasevalue / 100000000} BTC`);
        
        // Generate block
        const minerAddress = await client.getNewAddress();
        const newBlock = await client.generateToAddress(1, minerAddress);
        
        // Get block details
        const block = await client.getBlock(newBlock[0], 2);
        console.log('\nMined Block:');
        console.log(`Hash: ${block.hash}`);
        console.log(`Confirmations: ${block.confirmations}`);
        console.log(`Size: ${block.size} bytes`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 2: Bloom Filter Usage
```javascript
async function demonstrateBloomFilter() {
    try {
        // Create addresses to track
        const addresses = [];
        for (let i = 0; i < 5; i++) {
            addresses.push(await client.getNewAddress());
        }
        
        // Create bloom filter
        const filterSize = 512; // bytes
        const falsePositiveRate = 0.0001;
        const filter = new BloomFilter(filterSize, falsePositiveRate);
        
        // Add addresses to filter
        addresses.forEach(addr => {
            filter.insert(Buffer.from(addr, 'hex'));
        });
        
        console.log('Bloom Filter Created:');
        console.log(`Size: ${filterSize} bytes`);
        console.log(`False Positive Rate: ${falsePositiveRate}`);
        
        // Test filter
        addresses.forEach(addr => {
            const exists = filter.contains(Buffer.from(addr, 'hex'));
            console.log(`\nAddress ${addr}:`);
            console.log(`Found in filter: ${exists}`);
        });
        
        // Test false positive rate
        let falsePositives = 0;
        const testCount = 10000;
        
        for (let i = 0; i < testCount; i++) {
            const testAddr = await client.getNewAddress();
            if (filter.contains(Buffer.from(testAddr, 'hex'))) {
                falsePositives++;
            }
        }
        
        console.log('\nFalse Positive Test:');
        console.log(`Actual Rate: ${falsePositives / testCount}`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 3: Compact Block Relay
```javascript
async function demonstrateCompactBlocks() {
    try {
        // Get network info
        const netInfo = await client.getNetworkInfo();
        console.log('Network Configuration:');
        console.log(`Compact Blocks Enabled: ${netInfo.localservices.includes('COMPACT_FILTERS')}`);
        
        // Generate some transactions
        const txids = [];
        const recipientAddress = await client.getNewAddress();
        
        for (let i = 0; i < 5; i++) {
            const txid = await client.sendToAddress(recipientAddress, 0.1);
            txids.push(txid);
        }
        
        // Mine block with transactions
        const minerAddress = await client.getNewAddress();
        const blockHash = await client.generateToAddress(1, minerAddress);
        
        // Get compact block
        const block = await client.getBlock(blockHash[0]);
        console.log('\nCompact Block:');
        console.log(`Hash: ${block.hash}`);
        console.log(`Size: ${block.size} bytes`);
        console.log(`Transaction Count: ${block.tx.length}`);
        
        // Analyze transaction recovery
        const recoveredTxs = await Promise.all(
            txids.map(txid => client.getRawTransaction(txid, true))
        );
        
        console.log('\nTransaction Recovery:');
        recoveredTxs.forEach(tx => {
            console.log(`TXID: ${tx.txid}`);
            console.log(`Size: ${tx.size} bytes`);
            console.log(`VSize: ${tx.vsize} vbytes`);
        });
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 4: Chain Analysis
```javascript
async function demonstrateChainAnalysis() {
    try {
        // Get chain statistics
        const chainInfo = await client.getBlockchainInfo();
        console.log('Chain Statistics:');
        console.log(`Height: ${chainInfo.blocks}`);
        console.log(`Size on Disk: ${chainInfo.size_on_disk} bytes`);
        console.log(`Pruned: ${chainInfo.pruned}`);
        
        // Analyze recent blocks
        const blockCount = 10;
        let currentHash = await client.getBestBlockHash();
        
        console.log('\nAnalyzing Recent Blocks:');
        for (let i = 0; i < blockCount; i++) {
            const block = await client.getBlock(currentHash, 2);
            
            // Calculate block statistics
            let totalValue = 0;
            let totalFees = 0;
            
            block.tx.forEach(tx => {
                // Sum outputs
                tx.vout.forEach(out => {
                    totalValue += out.value;
                });
                
                // Calculate fees for non-coinbase
                if (!tx.vin[0].coinbase) {
                    const inputValue = tx.vin.reduce((sum, input) => sum + input.value, 0);
                    const outputValue = tx.vout.reduce((sum, output) => sum + output.value, 0);
                    totalFees += inputValue - outputValue;
                }
            });
            
            console.log(`\nBlock ${block.height}:`);
            console.log(`Transactions: ${block.tx.length}`);
            console.log(`Total Value: ${totalValue} BTC`);
            console.log(`Total Fees: ${totalFees} BTC`);
            
            currentHash = block.previousblockhash;
        }
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 5: Performance Analysis
```javascript
async function demonstratePerformance() {
    try {
        // Measure block validation performance
        console.log('Block Validation Performance:');
        const startTime = Date.now();
        
        // Generate blocks
        const minerAddress = await client.getNewAddress();
        await client.generateToAddress(10, minerAddress);
        
        const endTime = Date.now();
        console.log(`Time to generate 10 blocks: ${endTime - startTime}ms`);
        
        // Memory usage
        const memInfo = await client.getMemoryInfo();
        console.log('\nMemory Usage:');
        console.log(`Locked: ${memInfo.locked.used} bytes`);
        console.log(`Total: ${memInfo.locked.total} bytes`);
        
        // Network performance
        const netInfo = await client.getNetworkInfo();
        console.log('\nNetwork Statistics:');
        console.log(`Connections: ${netInfo.connections}`);
        console.log(`Version: ${netInfo.version}`);
        console.log(`Protocol Version: ${netInfo.protocolversion}`);
        
        // UTXO set statistics
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

1. Start Bitcoin Core with required options:
```bash
bitcoind -regtest -daemon -txindex -blockfilterindex
```

2. Install dependencies:
```bash
npm install bitcoin-core
```

3. Run the demonstrations:
```javascript
async function runDemonstrations() {
    console.log('Running Mining Demo...');
    await demonstrateMining();
    
    console.log('\nRunning Bloom Filter Demo...');
    await demonstrateBloomFilter();
    
    console.log('\nRunning Compact Blocks Demo...');
    await demonstrateCompactBlocks();
    
    console.log('\nRunning Chain Analysis Demo...');
    await demonstrateChainAnalysis();
    
    console.log('\nRunning Performance Analysis Demo...');
    await demonstratePerformance();
}

runDemonstrations().catch(console.error);
```

## Expected Output
Each demonstration will provide detailed output about different advanced aspects:
- Mining and block creation details
- Bloom filter effectiveness
- Compact block relay statistics
- Chain analysis metrics
- Performance measurements

## Troubleshooting
- Ensure Bitcoin Core is running with required options
- Check RPC credentials are correct
- Monitor debug.log for errors
- Verify sufficient disk space
- Check network connectivity
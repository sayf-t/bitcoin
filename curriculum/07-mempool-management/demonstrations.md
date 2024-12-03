# Mempool Management Demonstrations

## Demonstration 1: Mempool Monitoring

```javascript
const { BitcoinCore } = require('bitcoin-core');
const client = new BitcoinCore({
    network: 'regtest',
    username: 'rpcuser',
    password: 'rpcpassword'
});

async function monitorMempool() {
    try {
        // Get mempool info
        const mempoolInfo = await client.getMempoolInfo();
        console.log('Mempool Status:');
        console.log('---------------');
        console.log(`Size: ${mempoolInfo.size} transactions`);
        console.log(`Memory Usage: ${mempoolInfo.bytes} bytes`);
        console.log(`Maximum Memory: ${mempoolInfo.maxmempool} bytes`);
        console.log(`Min Fee Rate: ${mempoolInfo.mempoolminfee} BTC/kB`);
        
        // Get all transaction IDs
        const txids = await client.getRawMempool();
        console.log(`\nTotal Transactions: ${txids.length}`);
        
        // Get detailed information for first 5 transactions
        for (let i = 0; i < Math.min(5, txids.length); i++) {
            const txInfo = await client.getMempoolEntry(txids[i]);
            console.log(`\nTransaction ${i + 1}:`);
            console.log(`Fee: ${txInfo.fees.base} BTC`);
            console.log(`Time in mempool: ${txInfo.time} seconds`);
            console.log(`Descendant count: ${txInfo.descendantcount}`);
        }
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 2: Fee Estimation

```javascript
async function demonstrateFeeEstimation() {
    try {
        // Get fee estimates for different confirmation targets
        const targets = [1, 2, 3, 6, 12, 24];
        console.log('Fee Estimates (BTC/kB):');
        console.log('------------------------');
        
        for (const target of targets) {
            const estimate = await client.estimateSmartFee(target);
            console.log(`${target} block target: ${estimate.feerate || 'no estimate'}`);
            if (estimate.errors) {
                console.log(`  Errors: ${estimate.errors.join(', ')}`);
            }
        }
        
        // Get mempool fee histogram
        const histogram = await client.getMempoolAncestors(txids[0]);
        console.log('\nFee Distribution:');
        Object.entries(histogram).forEach(([fee, count]) => {
            console.log(`${fee} BTC/kB: ${count} transactions`);
        });
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 3: RBF Transaction

```javascript
async function demonstrateRBF() {
    try {
        // Create initial transaction
        const senderAddress = await client.getNewAddress();
        await client.generateToAddress(101, senderAddress); // For funds
        
        const recipientAddress = await client.getNewAddress();
        const initialTxid = await client.sendToAddress(
            recipientAddress,
            0.001,
            '', // Comment
            '', // Comment to
            false, // Subtract fee
            true, // Replace by fee
            1, // Confirmation target
            'CONSERVATIVE' // Estimate mode
        );
        
        console.log('Initial Transaction:', initialTxid);
        
        // Wait a moment
        await new Promise(resolve => setTimeout(resolve, 1000));
        
        // Create replacement transaction with higher fee
        const replacementTxid = await client.sendToAddress(
            recipientAddress,
            0.001,
            '',
            '',
            false,
            true,
            1,
            'CONSERVATIVE'
        );
        
        console.log('Replacement Transaction:', replacementTxid);
        
        // Check mempool for both transactions
        const mempoolEntry1 = await client.getMempoolEntry(initialTxid).catch(() => null);
        const mempoolEntry2 = await client.getMempoolEntry(replacementTxid).catch(() => null);
        
        console.log('\nMempool Status:');
        console.log(`Initial Transaction in mempool: ${!!mempoolEntry1}`);
        console.log(`Replacement Transaction in mempool: ${!!mempoolEntry2}`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 4: Package Acceptance

```javascript
async function demonstratePackageAcceptance() {
    try {
        // Create parent transaction
        const senderAddress = await client.getNewAddress();
        await client.generateToAddress(101, senderAddress);
        
        const recipientAddress = await client.getNewAddress();
        const parentTxid = await client.sendToAddress(
            recipientAddress,
            0.002,
            '',
            '',
            false,
            false,
            1
        );
        
        console.log('Parent Transaction:', parentTxid);
        
        // Create child transaction
        const childTxid = await client.sendToAddress(
            senderAddress,
            0.001,
            '',
            '',
            false,
            false,
            1
        );
        
        console.log('Child Transaction:', childTxid);
        
        // Get package information
        const parentInfo = await client.getMempoolEntry(parentTxid);
        const childInfo = await client.getMempoolEntry(childTxid);
        
        console.log('\nPackage Information:');
        console.log('Parent Fee Rate:', parentInfo.fees.base / parentInfo.vsize);
        console.log('Child Fee Rate:', childInfo.fees.base / childInfo.vsize);
        console.log('Combined Size:', parentInfo.vsize + childInfo.vsize);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 5: Mempool Persistence

```javascript
async function demonstrateMempoolPersistence() {
    try {
        // Create some transactions
        const senderAddress = await client.getNewAddress();
        await client.generateToAddress(101, senderAddress);
        
        const transactions = [];
        for (let i = 0; i < 5; i++) {
            const recipientAddress = await client.getNewAddress();
            const txid = await client.sendToAddress(recipientAddress, 0.001);
            transactions.push(txid);
        }
        
        console.log('Created Transactions:', transactions);
        
        // Save mempool
        await client.savemempool();
        console.log('\nMempool saved to disk');
        
        // Restart Bitcoin Core (simulation)
        console.log('Simulating node restart...');
        
        // Check mempool after "restart"
        const mempoolInfo = await client.getMempoolInfo();
        console.log('\nMempool Status After Restart:');
        console.log(`Size: ${mempoolInfo.size} transactions`);
        console.log(`Memory Usage: ${mempoolInfo.bytes} bytes`);
        
        // Verify transactions
        for (const txid of transactions) {
            const exists = await client.getMempoolEntry(txid).catch(() => null);
            console.log(`Transaction ${txid} in mempool: ${!!exists}`);
        }
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
    console.log('Running Mempool Monitor Demo...');
    await monitorMempool();
    
    console.log('\nRunning Fee Estimation Demo...');
    await demonstrateFeeEstimation();
    
    console.log('\nRunning RBF Demo...');
    await demonstrateRBF();
    
    console.log('\nRunning Package Acceptance Demo...');
    await demonstratePackageAcceptance();
    
    console.log('\nRunning Mempool Persistence Demo...');
    await demonstrateMempoolPersistence();
}

runDemonstrations().catch(console.error);
```

## Expected Output
Each demonstration will provide detailed output about different aspects of mempool management:
- Current mempool state
- Fee estimation data
- RBF transaction handling
- Package acceptance details
- Mempool persistence verification

## Troubleshooting
- Ensure Bitcoin Core is running in regtest mode
- Check RPC credentials are correct
- Verify sufficient funds for transactions
- Monitor debug.log for errors
- Check mempool.dat file exists
``` 
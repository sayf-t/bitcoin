# RPC and API Demonstrations

## Demonstration 1: RPC Command Usage
```javascript
const { BitcoinCore } = require('bitcoin-core');
const client = new BitcoinCore({
    network: 'regtest',
    username: 'rpcuser',
    password: 'rpcpassword'
});

async function demonstrateRPCCommands() {
    try {
        // Basic blockchain info
        const info = await client.getBlockchainInfo();
        console.log('Blockchain Information:');
        console.log('----------------------');
        console.log(`Chain: ${info.chain}`);
        console.log(`Blocks: ${info.blocks}`);
        console.log(`Headers: ${info.headers}`);
        console.log(`Best Block Hash: ${info.bestblockhash}`);
        console.log(`Difficulty: ${info.difficulty}`);
        
        // Get network info
        const netInfo = await client.getNetworkInfo();
        console.log('\nNetwork Information:');
        console.log(`Version: ${netInfo.version}`);
        console.log(`Subversion: ${netInfo.subversion}`);
        console.log(`Protocol Version: ${netInfo.protocolversion}`);
        
        // Get memory info
        const memInfo = await client.getMemoryInfo();
        console.log('\nMemory Usage:');
        console.log(`Locked: ${memInfo.locked.used} bytes`);
        console.log(`Total: ${memInfo.locked.total} bytes`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 2: REST API Usage
```javascript
const axios = require('axios');
const https = require('https');

async function demonstrateRESTAPI() {
    const agent = new https.Agent({
        rejectUnauthorized: false // For self-signed certs in regtest
    });
    
    try {
        // Get blockchain info via REST
        const chainInfo = await axios.get('http://localhost:18443/rest/chaininfo.json', { httpsAgent: agent });
        console.log('Chain Info (REST):');
        console.log(chainInfo.data);
        
        // Get block by hash
        const blockHash = chainInfo.data.bestblockhash;
        const block = await axios.get(`http://localhost:18443/rest/block/${blockHash}.json`, { httpsAgent: agent });
        console.log('\nBlock Info (REST):');
        console.log(`Hash: ${block.data.hash}`);
        console.log(`Height: ${block.data.height}`);
        console.log(`Tx Count: ${block.data.tx.length}`);
        
        // Get mempool info
        const mempool = await axios.get('http://localhost:18443/rest/mempool/info.json', { httpsAgent: agent });
        console.log('\nMempool Info (REST):');
        console.log(mempool.data);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 3: ZMQ Notifications
```javascript
const zmq = require('zeromq');

async function demonstrateZMQ() {
    try {
        // Set up ZMQ subscribers
        const hashBlockSub = new zmq.Subscriber();
        const hashTxSub = new zmq.Subscriber();
        
        // Connect to ZMQ publishers
        hashBlockSub.connect('tcp://127.0.0.1:28332');
        hashTxSub.connect('tcp://127.0.0.1:28333');
        
        // Subscribe to topics
        hashBlockSub.subscribe('hashblock');
        hashTxSub.subscribe('hashtx');
        
        console.log('ZMQ Monitoring Started...');
        
        // Handle block notifications
        hashBlockSub.on('message', (topic, message) => {
            const blockHash = message.toString('hex');
            console.log('\nNew Block:', blockHash);
        });
        
        // Handle transaction notifications
        hashTxSub.on('message', (topic, message) => {
            const txid = message.toString('hex');
            console.log('\nNew Transaction:', txid);
        });
        
        // Keep monitoring for 5 minutes
        setTimeout(() => {
            hashBlockSub.close();
            hashTxSub.close();
            console.log('\nZMQ Monitoring Stopped');
        }, 300000);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 4: Index Usage
```javascript
async function demonstrateIndexUsage() {
    try {
        // Check if indexes are enabled
        const info = await client.getIndexInfo();
        console.log('Index Status:');
        console.log(`Transaction Index: ${info.txindex}`);
        console.log(`Block Filter Index: ${info.blockfilter}`);
        console.log(`Coinstats Index: ${info.coinstatsindex}`);
        
        // Use transaction index
        const txid = await client.sendToAddress(await client.getNewAddress(), 1.0);
        const tx = await client.getRawTransaction(txid, true);
        console.log('\nTransaction Details:');
        console.log(`TXID: ${tx.txid}`);
        console.log(`Size: ${tx.size} bytes`);
        console.log(`VSize: ${tx.vsize} vbytes`);
        
        // Use block filter index
        const blockHash = await client.getBestBlockHash();
        const filter = await client.getBlockFilter(blockHash);
        console.log('\nBlock Filter:');
        console.log(`Block: ${blockHash}`);
        console.log(`Filter: ${filter.filter}`);
        
        // Get coinstats
        const stats = await client.getCoinstatsInfo();
        console.log('\nCoin Statistics:');
        console.log(`Total Amount: ${stats.total_amount} BTC`);
        console.log(`Block Height: ${stats.height}`);
        console.log(`Best Block: ${stats.bestblock}`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 5: Batch Requests
```javascript
async function demonstrateBatchRequests() {
    try {
        // Prepare batch of commands
        const batch = [
            { method: 'getblockcount', parameters: [] },
            { method: 'getmempoolinfo', parameters: [] },
            { method: 'getconnectioncount', parameters: [] },
            { method: 'getnetworkhashps', parameters: [] },
            { method: 'getmininginfo', parameters: [] }
        ];
        
        // Execute batch
        console.log('Executing Batch Request...');
        const results = await client.command(batch);
        
        // Process results
        console.log('\nBatch Results:');
        console.log(`Block Height: ${results[0]}`);
        console.log(`Mempool Size: ${results[1].size} transactions`);
        console.log(`Connections: ${results[2]}`);
        console.log(`Network Hashrate: ${results[3]} H/s`);
        console.log(`Mining Difficulty: ${results[4].difficulty}`);
        
        // Demonstrate error handling in batch
        const batchWithError = [
            { method: 'getblock', parameters: ['invalid_hash'] },
            { method: 'getblockcount', parameters: [] }
        ];
        
        try {
            await client.command(batchWithError);
        } catch (error) {
            console.log('\nBatch Error Handling:');
            console.log('Error caught:', error.message);
        }
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Running the Demonstrations

1. Start Bitcoin Core with required options:
```bash
bitcoind -regtest -daemon -txindex -blockfilterindex -zmqpubhashblock=tcp://127.0.0.1:28332 -zmqpubhashtx=tcp://127.0.0.1:28333
```

2. Install dependencies:
```bash
npm install bitcoin-core axios zeromq
```

3. Run the demonstrations:
```javascript
async function runDemonstrations() {
    console.log('Running RPC Commands Demo...');
    await demonstrateRPCCommands();
    
    console.log('\nRunning REST API Demo...');
    await demonstrateRESTAPI();
    
    console.log('\nRunning ZMQ Demo...');
    await demonstrateZMQ();
    
    console.log('\nRunning Index Usage Demo...');
    await demonstrateIndexUsage();
    
    console.log('\nRunning Batch Requests Demo...');
    await demonstrateBatchRequests();
}

runDemonstrations().catch(console.error);
```

## Expected Output
Each demonstration will provide detailed output about different aspects of the RPC and API functionality:
- RPC command responses
- REST API responses
- ZMQ notifications
- Index query results
- Batch request results

## Troubleshooting
- Ensure Bitcoin Core is running with required options
- Check RPC credentials are correct
- Verify indexes are enabled and synced
- Monitor debug.log for errors
- Check ZMQ ports are available
``` 
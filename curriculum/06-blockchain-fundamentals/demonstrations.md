# Blockchain Fundamentals Demonstrations

## Demonstration 1: Block Exploration
```javascript
const { BitcoinCore } = require('bitcoin-core');
const client = new BitcoinCore({
    network: 'regtest',
    username: 'rpcuser',
    password: 'rpcpassword'
});

async function exploreBlockchain() {
    try {
        // Get the latest block
        const bestBlockHash = await client.getBestBlockHash();
        const latestBlock = await client.getBlock(bestBlockHash, 2);

        console.log('Latest Block Information:');
        console.log('------------------------');
        console.log(`Hash: ${latestBlock.hash}`);
        console.log(`Height: ${latestBlock.height}`);
        console.log(`Time: ${new Date(latestBlock.time * 1000).toISOString()}`);
        console.log(`Number of Transactions: ${latestBlock.tx.length}`);
        console.log(`Size: ${latestBlock.size} bytes`);
        console.log(`Version: ${latestBlock.version}`);
        console.log(`Merkle Root: ${latestBlock.merkleroot}`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 2: Block Creation and Mining
```javascript
async function demonstrateMining() {
    try {
        // Get a mining address
        const minerAddress = await client.getNewAddress();
        
        // Get initial blockchain info
        const initialInfo = await client.getBlockchainInfo();
        console.log('Initial Block Height:', initialInfo.blocks);
        
        // Generate new blocks
        console.log('Mining 10 blocks...');
        const newBlocks = await client.generateToAddress(10, minerAddress);
        
        // Show new blockchain state
        const finalInfo = await client.getBlockchainInfo();
        console.log('Final Block Height:', finalInfo.blocks);
        console.log('Newly Mined Blocks:', newBlocks);
        
        // Analyze last mined block
        const lastBlock = await client.getBlock(newBlocks[newBlocks.length - 1], 2);
        console.log('Last Block Details:', {
            hash: lastBlock.hash,
            height: lastBlock.height,
            transactions: lastBlock.tx.length,
            time: new Date(lastBlock.time * 1000).toISOString()
        });
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 3: Chain Analysis
```javascript
async function analyzeBlockchain() {
    try {
        // Get current chain tip
        const tip = await client.getBestBlockHash();
        let currentHash = tip;
        
        console.log('Analyzing last 5 blocks...');
        for (let i = 0; i < 5; i++) {
            const block = await client.getBlock(currentHash);
            console.log('\nBlock Information:');
            console.log(`Height: ${block.height}`);
            console.log(`Hash: ${block.hash}`);
            console.log(`Time: ${new Date(block.time * 1000).toISOString()}`);
            console.log(`Difficulty: ${block.difficulty}`);
            console.log(`Transactions: ${block.tx.length}`);
            
            currentHash = block.previousblockhash;
        }
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 4: Block Template and Mining Simulation
```javascript
async function miningSimulation() {
    try {
        // Get block template
        const template = await client.getBlockTemplate({
            rules: ['segwit']
        });
        
        console.log('Block Template:');
        console.log('--------------');
        console.log(`Previous Block: ${template.previousblockhash}`);
        console.log(`Target Bits: ${template.bits}`);
        console.log(`Current Time: ${template.curtime}`);
        console.log(`Height: ${template.height}`);
        console.log(`Transactions Ready: ${template.transactions.length}`);
        
        // Simulate mining process
        console.log('\nSimulating Mining Process...');
        const minerAddress = await client.getNewAddress();
        const newBlock = await client.generateToAddress(1, minerAddress);
        
        console.log('Block Successfully Mined!');
        console.log(`New Block Hash: ${newBlock[0]}`);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 5: Transaction in Block
```javascript
async function demonstrateTransactionInBlock() {
    try {
        // Create a transaction
        const senderAddress = await client.getNewAddress();
        await client.generateToAddress(101, senderAddress); // Generate blocks for maturity
        
        const recipientAddress = await client.getNewAddress();
        const txid = await client.sendToAddress(recipientAddress, 1.0);
        
        console.log('Transaction created:', txid);
        
        // Mine a block to include the transaction
        const minerAddress = await client.getNewAddress();
        const newBlockHash = await client.generateToAddress(1, minerAddress);
        
        // Get block and find our transaction
        const block = await client.getBlock(newBlockHash[0], 2);
        const ourTx = block.tx.find(tx => tx.txid === txid);
        
        console.log('\nTransaction in Block:');
        console.log(`Block Hash: ${block.hash}`);
        console.log(`Transaction Index: ${block.tx.indexOf(ourTx)}`);
        console.log(`Confirmations: ${ourTx.confirmations}`);
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
    console.log('Running Block Explorer Demo...');
    await exploreBlockchain();
    
    console.log('\nRunning Mining Demo...');
    await demonstrateMining();
    
    console.log('\nRunning Chain Analysis...');
    await analyzeBlockchain();
    
    console.log('\nRunning Mining Simulation...');
    await miningSimulation();
    
    console.log('\nRunning Transaction Demo...');
    await demonstrateTransactionInBlock();
}

runDemonstrations().catch(console.error);
```

## Expected Output
Each demonstration will provide detailed output about different aspects of the Bitcoin blockchain:
- Block structure and contents
- Mining process and block creation
- Chain analysis and block relationships
- Transaction inclusion and confirmation
- Block template creation and mining simulation

## Troubleshooting
- Ensure Bitcoin Core is running in regtest mode
- Check RPC credentials are correct
- Verify network connectivity
- Monitor debug.log for errors
- Check Bitcoin Core memory usage
``` 
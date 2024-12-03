# Wallet Development Demonstrations

## Demonstration 1: Wallet Creation and Management
```javascript
const { BitcoinCore } = require('bitcoin-core');
const client = new BitcoinCore({
    network: 'regtest',
    username: 'rpcuser',
    password: 'rpcpassword'
});

async function demonstrateWalletManagement() {
    try {
        // Create a new wallet
        const walletName = 'demo_wallet';
        await client.createWallet(walletName);
        
        // Get wallet info
        const walletInfo = await client.getWalletInfo();
        console.log('Wallet Information:');
        console.log('-------------------');
        console.log(`Wallet Name: ${walletInfo.walletname}`);
        console.log(`Balance: ${walletInfo.balance} BTC`);
        console.log(`Tx Count: ${walletInfo.txcount}`);
        console.log(`Key Pool Size: ${walletInfo.keypoolsize}`);
        
        // List all wallets
        const wallets = await client.listWallets();
        console.log('\nAvailable Wallets:', wallets);
        
        // Backup wallet
        await client.backupWallet(`/tmp/${walletName}_backup.dat`);
        console.log('\nWallet backup created');
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 2: Address Management
```javascript
async function demonstrateAddressManagement() {
    try {
        // Generate new address
        const address = await client.getNewAddress();
        console.log('New Address:', address);
        
        // Get address info
        const addressInfo = await client.getAddressInfo(address);
        console.log('\nAddress Information:');
        console.log(`Is Mine: ${addressInfo.ismine}`);
        console.log(`HD Key Path: ${addressInfo.hdkeypath}`);
        console.log(`Public Key: ${addressInfo.pubkey}`);
        
        // Generate different address types
        const legacyAddress = await client.getNewAddress('', 'legacy');
        const p2shAddress = await client.getNewAddress('', 'p2sh-segwit');
        const bech32Address = await client.getNewAddress('', 'bech32');
        
        console.log('\nAddress Types:');
        console.log(`Legacy: ${legacyAddress}`);
        console.log(`P2SH-SegWit: ${p2shAddress}`);
        console.log(`Bech32: ${bech32Address}`);
        
        // Validate addresses
        const validations = await Promise.all([
            client.validateAddress(legacyAddress),
            client.validateAddress(p2shAddress),
            client.validateAddress(bech32Address)
        ]);
        
        console.log('\nAddress Validations:');
        validations.forEach((validation, index) => {
            console.log(`Address ${index + 1}: ${validation.isvalid}`);
        });
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 3: Transaction Creation
```javascript
async function demonstrateTransactionCreation() {
    try {
        // Generate some coins
        const minerAddress = await client.getNewAddress();
        await client.generateToAddress(101, minerAddress);
        
        // Create a transaction
        const recipientAddress = await client.getNewAddress();
        const amount = 1.0; // BTC
        
        // Get unspent outputs
        const utxos = await client.listUnspent();
        console.log('\nAvailable UTXOs:', utxos.length);
        
        // Create raw transaction
        const txid = await client.sendToAddress(recipientAddress, amount);
        console.log('\nTransaction created:', txid);
        
        // Get transaction details
        const tx = await client.getTransaction(txid);
        console.log('\nTransaction Details:');
        console.log(`Amount: ${tx.amount} BTC`);
        console.log(`Fee: ${tx.fee} BTC`);
        console.log(`Confirmations: ${tx.confirmations}`);
        
        // List transactions
        const transactions = await client.listTransactions();
        console.log('\nRecent Transactions:', transactions.length);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 4: HD Wallet Features
```javascript
async function demonstrateHDWallet() {
    try {
        // Create HD wallet
        const walletName = 'hd_wallet';
        await client.createWallet(walletName, {
            disable_private_keys: false,
            blank: true,
            descriptors: true
        });
        
        // Generate addresses from HD chain
        const addresses = [];
        for (let i = 0; i < 5; i++) {
            const address = await client.getNewAddress(`account${i}`);
            addresses.push(address);
            
            const info = await client.getAddressInfo(address);
            console.log(`\nAddress ${i + 1}:`);
            console.log(`Address: ${address}`);
            console.log(`HD Path: ${info.hdkeypath}`);
            console.log(`Public Key: ${info.pubkey}`);
        }
        
        // Show derivation info
        console.log('\nDerivation Paths:');
        for (const address of addresses) {
            const info = await client.getAddressInfo(address);
            console.log(`${address}: ${info.hdkeypath}`);
        }
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## Demonstration 5: Wallet Encryption
```javascript
async function demonstrateWalletEncryption() {
    try {
        // Create unencrypted wallet
        const walletName = 'secure_wallet';
        await client.createWallet(walletName);
        
        // Encrypt wallet
        const passphrase = 'your_secure_passphrase';
        await client.encryptWallet(passphrase);
        console.log('Wallet encrypted');
        
        // Try to create address (should fail)
        try {
            await client.getNewAddress();
        } catch (error) {
            console.log('\nWallet locked - cannot create address');
        }
        
        // Unlock wallet
        await client.walletPassphrase(passphrase, 30);
        console.log('\nWallet unlocked for 30 seconds');
        
        // Create address (should succeed)
        const address = await client.getNewAddress();
        console.log('New address created:', address);
        
        // Lock wallet
        await client.walletLock();
        console.log('\nWallet locked');
        
        // Show wallet info
        const info = await client.getWalletInfo();
        console.log('\nWallet Status:');
        console.log(`Encrypted: ${info.unlocked_until ? 'Yes' : 'No'}`);
        console.log(`Locked: ${!info.unlocked_until}`);
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
    console.log('Running Wallet Management Demo...');
    await demonstrateWalletManagement();
    
    console.log('\nRunning Address Management Demo...');
    await demonstrateAddressManagement();
    
    console.log('\nRunning Transaction Creation Demo...');
    await demonstrateTransactionCreation();
    
    console.log('\nRunning HD Wallet Demo...');
    await demonstrateHDWallet();
    
    console.log('\nRunning Wallet Encryption Demo...');
    await demonstrateWalletEncryption();
}

runDemonstrations().catch(console.error);
```

## Expected Output
Each demonstration will provide detailed output about different aspects of wallet functionality:
- Wallet creation and management
- Address generation and validation
- Transaction creation and tracking
- HD wallet features
- Wallet encryption and security

## Troubleshooting
- Ensure Bitcoin Core is running in regtest mode
- Check RPC credentials are correct
- Verify wallet is unlocked for operations
- Monitor debug.log for errors
- Check wallet.dat permissions
``` 
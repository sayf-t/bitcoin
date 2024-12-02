# Understanding Bitcoin Wallet Architecture

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Explain Bitcoin Core's wallet architecture
- Understand key storage and management
- Implement basic wallet functionality
- Debug wallet-related issues

## 🌟 Real-World Analogy
Think of a Bitcoin wallet like a modern digital bank:
- Wallet = Bank Branch
- Private Keys = Safety Deposit Box Keys
- Public Keys = Account Numbers
- Addresses = Email Addresses for Receiving Money
- UTXO Set = Account Balances
- Keypool = Pre-prepared Account Numbers
- Encryption = Vault Security

## Wallet Components

### 1. Core Wallet Structure
```cpp
// Location: src/wallet/wallet.h
class CWallet {
    // Like a bank's core systems:
    std::map<uint256, CWalletTx> mapWallet;      // Transaction records
    std::set<COutPoint> setLockedCoins;          // Reserved funds
    std::map<CKeyID, CKey> mapKeys;              // Safety deposit boxes
    mutable CCriticalSection cs_wallet;           // Access control
};
```

### 2. Key Management
Like managing safety deposit boxes:
```cpp
class CKeyStore {
    mutable CCriticalSection cs_KeyStore;     // Access control
    
    // Key storage
    std::map<CKeyID, CKey> mapKeys;          // Private keys
    std::map<CKeyID, CPubKey> mapPubKeys;    // Public keys
    std::map<CScriptID, CScript> mapScripts; // Special instructions
};
```

## Wallet Implementation

### 1. Wallet Database
Like the bank's record system:
```cpp
// Location: src/wallet/walletdb.h
class CWalletDB {
    bool WriteKey(const CPubKey& pubkey,
                 const CPrivKey& privkey,
                 const CKeyMetadata& metadata)
    {
        // Store key with metadata
        if (!Write(std::make_pair(std::string("key"), pubkey),
                  std::make_pair(privkey, metadata)))
            return false;
            
        // Update key index
        return Write(std::make_pair(std::string("keyindex"), pubkey.GetID()),
                    pubkey);
    }
};
```

### 2. Transaction Management
Like handling bank transactions:
```cpp
// Transaction handling
bool CWallet::AddToWallet(const CWalletTx& wtxIn)
{
    // Add transaction to wallet
    LOCK(cs_wallet);
    
    // Generate unique identifier
    uint256 hash = wtxIn.GetHash();
    
    // Check if already exists
    if (mapWallet.count(hash))
        return false;
        
    // Add to wallet
    mapWallet[hash] = wtxIn;
    
    // Update indices
    return true;
}
```

## 💻 Hands-on Practice

### 1. Create Basic Wallet Functions
```cpp
// Simple wallet implementation
class SimpleWallet {
public:
    bool CreateNewKey() {
        // Generate new key pair
        CKey key;
        key.MakeNewKey(true);  // compressed
        
        // Get public key
        CPubKey pubkey = key.GetPubKey();
        
        // Store key pair
        return AddKeyPair(key, pubkey);
    }
    
    bool AddKeyPair(const CKey& key, const CPubKey& pubkey) {
        LOCK(cs_wallet);
        
        // Add to key store
        mapKeys[pubkey.GetID()] = key;
        mapPubKeys[pubkey.GetID()] = pubkey;
        
        return true;
    }
};
```

### 2. Implement Balance Tracking
```cpp
// Balance management
CAmount GetBalance() const
{
    CAmount balance = 0;
    
    LOCK(cs_wallet);
    
    for (const auto& entry : mapWallet) {
        const CWalletTx& wtx = entry.second;
        
        if (wtx.IsTrusted() && wtx.GetAvailableCredit() > 0)
            balance += wtx.GetAvailableCredit();
    }
    
    return balance;
}
```

## 🎯 Hands-on Practice with RPC

### 1. Wallet Management and Operations
```javascript
const Client = require('bitcoin-core');
const client = new Client({
    network: 'regtest',
    username: 'your_username',
    password: 'your_secure_password',
    port: 18443
});

async function walletOperations() {
    try {
        // Get wallet info
        const walletInfo = await client.getWalletInfo();
        console.log('Wallet Info:', {
            balance: walletInfo.balance,
            unconfirmed: walletInfo.unconfirmed_balance,
            immature: walletInfo.immature_balance,
            txcount: walletInfo.txcount,
            keypoolsize: walletInfo.keypoolsize,
            keypoolsizeHD: walletInfo.keypoolsize_hd_internal
        });
        
        // Create new address
        const newAddress = await client.getNewAddress();
        const addressInfo = await client.getAddressInfo(newAddress);
        console.log('New Address Info:', {
            address: addressInfo.address,
            hdkeypath: addressInfo.hdkeypath,
            pubkey: addressInfo.pubkey,
            iswitness: addressInfo.iswitness
        });
        
        // List all addresses
        const addresses = await client.listAddressGroupings();
        console.log('Address Groupings:', addresses);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

### 2. Transaction History and Balance Management
```javascript
async function transactionHistory() {
    try {
        // Generate some initial coins
        const minerAddress = await client.getNewAddress();
        await client.generateToAddress(101, minerAddress);
        
        // List transactions
        const transactions = await client.listTransactions();
        console.log('Recent Transactions:', transactions.map(tx => ({
            txid: tx.txid,
            category: tx.category,
            amount: tx.amount,
            confirmations: tx.confirmations,
            time: new Date(tx.time * 1000).toISOString()
        })));
        
        // Get detailed balance information
        const balances = await client.getBalances();
        console.log('Detailed Balances:', {
            trusted: balances.mine.trusted,
            untrusted_pending: balances.mine.untrusted_pending,
            immature: balances.mine.immature
        });
    } catch (error) {
        console.error('Error:', error);
    }
}
```

### 3. Wallet Backup and Recovery
```javascript
async function walletBackupAndRecovery() {
    try {
        // Backup wallet
        const backupPath = '/path/to/backup/wallet.dat';
        await client.backupWallet(backupPath);
        console.log('Wallet backed up to:', backupPath);
        
        // Dump wallet
        const walletDump = await client.dumpWallet('/path/to/dump.txt');
        console.log('Wallet dumped to:', walletDump.filename);
        
        // Get master HD seed
        const hdInfo = await client.getDescriptorInfo("wpkh([d34db33f/84'/0'/0']xpub6DJ2dNUysrn5Vt36jH2KLBT2i1auw1tTSSomg8PhqNiUtx8QX2SvC9nrHu81fT41fvDUnhMjEzQgXnQjKEu3oaqMSzhSrHMxyyoEAmUHQbY/0/*)");
        console.log('HD Wallet Info:', hdInfo);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

### 4. Practice Exercise: Comprehensive Wallet Management
Create a script that:
1. Manages multiple addresses
2. Tracks transaction history
3. Handles wallet backups
4. Monitors balance changes

```javascript
async function walletManagementExercise() {
    try {
        // Initial wallet state
        const initialInfo = await client.getWalletInfo();
        console.log('Initial Wallet State:', initialInfo);
        
        // Generate multiple addresses
        const addresses = [];
        for (let i = 0; i < 3; i++) {
            const address = await client.getNewAddress();
            const addressInfo = await client.getAddressInfo(address);
            addresses.push({
                address,
                pubkey: addressInfo.pubkey,
                path: addressInfo.hdkeypath
            });
        }
        
        // Generate some coins
        const minerAddress = await client.getNewAddress();
        await client.generateToAddress(101, minerAddress);
        
        // Create some transactions
        for (const addr of addresses) {
            await client.sendToAddress(addr.address, 1.0);
        }
        
        // Generate block to confirm transactions
        await client.generateToAddress(1, minerAddress);
        
        // Analyze wallet state
        const finalInfo = await client.getWalletInfo();
        console.log('Final Wallet State:', {
            balance: finalInfo.balance,
            txcount: finalInfo.txcount,
            addresses: addresses.length
        });
        
        // List all transactions
        const transactions = await client.listTransactions();
        console.log('Transaction History:', transactions.map(tx => ({
            address: tx.address,
            category: tx.category,
            amount: tx.amount,
            confirmations: tx.confirmations
        })));
        
        // Backup wallet
        const timestamp = Date.now();
        await client.backupWallet(`/path/to/backup/wallet-${timestamp}.dat`);
        
    } catch (error) {
        console.error('Error in wallet management:', error);
    }
}
```

## 🎯 Knowledge Check
Try to answer these questions:
1. Why separate key storage from transaction data?
2. How is wallet security maintained?
3. What role does the keypool serve?
4. How are transactions tracked in the wallet?

## 🛠️ Practical Exercises

### Exercise 1: Wallet Creator
Build a program that:
- Generates key pairs
- Manages addresses
- Tracks balances
- Handles encryption

### Exercise 2: Transaction Manager
Create a system that:
- Records transactions
- Updates balances
- Manages UTXOs
- Handles confirmations

## 🔍 Common Issues and Solutions

### Issue 1: Key Management
Like securing safety deposit boxes:
- Lost keys
- Corrupted keystore
- Encryption failures
- Backup problems

### Issue 2: Transaction Tracking
Like maintaining accurate records:
- Missing transactions
- Incorrect balances
- Confirmation tracking
- UTXO management

## 📚 Deep Dive Topics

### 1. Key Generation
Like creating new accounts:
```cpp
// Generate new key
bool CWallet::GenerateNewKey(CWalletDB& walletdb,
                            CKey& key)
{
    // Create key
    key.MakeNewKey(true);
    
    // Get public key
    CPubKey pubkey = key.GetPubKey();
    assert(key.VerifyPubKey(pubkey));
    
    // Store key
    return AddKeyPubKey(key, pubkey);
}
```

### 2. Address Management
Like managing receiving addresses:
```cpp
// Get new address
bool CWallet::GetNewDestination(CTxDestination& dest)
{
    // Generate new key
    CKey key;
    if (!GenerateNewKey(key))
        return false;
        
    // Create address
    dest = key.GetPubKey().GetID();
    
    return true;
}
```

## 🔧 Debugging Tools

### 1. Wallet Inspection
```bash
# Using bitcoin-cli
bitcoin-cli getwalletinfo
bitcoin-cli listunspent
```

### 2. Key Analysis
```cpp
// Debug key information
void AnalyzeKey(const CKey& key)
{
    LogPrintf("Key Analysis:\n"
              "Private: %s\n"
              "Public: %s\n"
              "Address: %s\n",
              key.ToString(),
              key.GetPubKey().ToString(),
              key.GetPubKey().GetID().ToString());
}
```

## 📈 Performance Tips

### 1. Database Operations
- Batch writes
- Cache hot data
- Index important fields
- Regular maintenance

### 2. Transaction Processing
- Efficient UTXO lookup
- Smart balance calculation
- Optimized key lookup
- Background processing

## 🎯 Next Steps
1. Study key management in detail
2. Learn about HD wallets
3. Understand transaction building
4. Explore backup strategies

## 💡 Review Questions
1. How is wallet security implemented?
2. What makes a good wallet design?
3. How are addresses generated?
4. Why is UTXO tracking important?

## 📚 Additional Resources
- [Wallet Documentation](https://developer.bitcoin.org/reference/wallet.html)
- [Key Management](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)
- [Wallet Code](https://github.com/bitcoin/bitcoin/tree/master/src/wallet)

Remember: A well-designed wallet is crucial for secure Bitcoin management. Take time to understand the security implications of each component. 
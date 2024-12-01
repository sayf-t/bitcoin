# Understanding Bitcoin Transaction Building

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Implement transaction creation and signing
- Understand fee calculation and UTXO selection
- Build a transaction builder service
- Handle multisig transactions

## 🌟 Real-World Analogy
Think of Bitcoin transaction building like preparing a financial document:
- Transaction = Financial Contract
- Inputs = Source of Funds
- Outputs = Payment Instructions
- Signatures = Authorizations
- Fees = Processing Costs
- UTXO Selection = Account Balance Management
- Multisig = Multiple Approvers Required

## Transaction Components

### 1. Transaction Structure
```cpp
// Location: src/primitives/transaction.h
class CMutableTransaction {
    // Like a contract template:
    int32_t nVersion;                  // Contract version
    std::vector<CTxIn> vin;           // Fund sources
    std::vector<CTxOut> vout;         // Payment details
    uint32_t nLockTime;               // Execution time
};
```

### 2. Input/Output Management
```cpp
// Input structure (source of funds)
class CTxIn {
    COutPoint prevout;       // Previous transaction reference
    CScript scriptSig;       // Authorization signature
    uint32_t nSequence;      // Special conditions
};

// Output structure (payment instructions)
class CTxOut {
    CAmount nValue;          // Amount to send
    CScript scriptPubKey;    // Recipient instructions
};
```

## 🚀 Project: Transaction Builder Service

Let's build a practical Node.js service that creates and manages Bitcoin transactions.

### Project Structure
```bash
transaction-builder/
├── src/
│   ├── index.ts
│   ├── services/
│   │   ├── transaction.service.ts
│   │   ├── utxo.service.ts
│   │   └── fee.service.ts
│   ├── models/
│   │   ├── transaction.model.ts
│   │   └── utxo.model.ts
│   └── utils/
│       ├── bitcoin.utils.ts
│       └── crypto.utils.ts
├── tests/
├── package.json
└── README.md
```

### 1. Basic Transaction Service
```typescript
// src/services/transaction.service.ts
import { ECPairFactory } from 'ecpair';
import * as bitcoin from 'bitcoinjs-lib';

export class TransactionService {
    private network: bitcoin.networks.Network;
    
    constructor(network = bitcoin.networks.bitcoin) {
        this.network = network;
    }
    
    async createTransaction(
        inputs: UTXO[],
        outputs: Output[],
        keys: ECPair[]
    ): Promise<string> {
        const psbt = new bitcoin.Psbt({ network: this.network });
        
        // Add inputs
        for (const input of inputs) {
            psbt.addInput({
                hash: input.txid,
                index: input.vout,
                witnessUtxo: input.witnessUtxo
            });
        }
        
        // Add outputs
        for (const output of outputs) {
            psbt.addOutput({
                address: output.address,
                value: output.value
            });
        }
        
        // Sign inputs
        inputs.forEach((_, index) => {
            psbt.signInput(index, keys[index]);
        });
        
        // Finalize and extract
        psbt.finalizeAllInputs();
        return psbt.extractTransaction().toHex();
    }
}
```

### 2. UTXO Selection Service
```typescript
// src/services/utxo.service.ts
export class UTXOService {
    selectOptimalUTXOs(
        utxos: UTXO[],
        targetAmount: number,
        feeRate: number
    ): SelectedUTXOs {
        // Sort UTXOs by value
        const sortedUTXOs = [...utxos].sort((a, b) => b.value - a.value);
        
        let selectedUTXOs: UTXO[] = [];
        let selectedAmount = 0;
        
        // Simple selection algorithm
        for (const utxo of sortedUTXOs) {
            selectedUTXOs.push(utxo);
            selectedAmount += utxo.value;
            
            // Calculate estimated fee
            const estimatedFee = this.calculateFee(
                selectedUTXOs.length,
                2,  // Assume 2 outputs (payment + change)
                feeRate
            );
            
            if (selectedAmount >= targetAmount + estimatedFee) {
                break;
            }
        }
        
        return {
            utxos: selectedUTXOs,
            fee: this.calculateFee(selectedUTXOs.length, 2, feeRate)
        };
    }
    
    private calculateFee(
        inputCount: number,
        outputCount: number,
        feeRate: number
    ): number {
        // Estimate transaction size
        const baseSize = 10;  // Version + Locktime
        const inputSize = inputCount * 148;  // Average input size
        const outputSize = outputCount * 34;  // Average output size
        
        return Math.ceil((baseSize + inputSize + outputSize) * feeRate);
    }
}
```

### 3. Fee Estimation Service
```typescript
// src/services/fee.service.ts
export class FeeService {
    async estimateFee(
        priority: 'high' | 'medium' | 'low'
    ): Promise<number> {
        // Fetch current fee rates from mempool.space API
        const response = await fetch('https://mempool.space/api/v1/fees/recommended');
        const fees = await response.json();
        
        switch (priority) {
            case 'high':
                return fees.fastestFee;
            case 'medium':
                return fees.halfHourFee;
            case 'low':
                return fees.hourFee;
            default:
                return fees.halfHourFee;
        }
    }
}
```

### 4. Multisig Support
```typescript
// src/services/multisig.service.ts
export class MultisigService {
    createMultisigAddress(
        pubkeys: Buffer[],
        m: number,
        network = bitcoin.networks.bitcoin
    ): MultisigAddress {
        // Create redeem script
        const script = bitcoin.payments.p2ms({
            m,
            pubkeys,
            network
        });
        
        // Create P2SH address
        const p2sh = bitcoin.payments.p2sh({
            redeem: script,
            network
        });
        
        return {
            address: p2sh.address!,
            redeemScript: script.output!.toString('hex'),
            witnessScript: script.output!.toString('hex')
        };
    }
    
    async signMultisigTransaction(
        psbt: bitcoin.Psbt,
        keys: ECPair[],
        inputIndex: number
    ): Promise<void> {
        // Sign with each provided key
        for (const key of keys) {
            psbt.signInput(inputIndex, key);
        }
    }
}
```

## API Usage Example
```typescript
// Example usage of the transaction builder
async function createPayment() {
    const txService = new TransactionService();
    const utxoService = new UTXOService();
    const feeService = new FeeService();
    
    // Get UTXOs and fee rate
    const utxos = await fetchUTXOs();
    const feeRate = await feeService.estimateFee('medium');
    
    // Select UTXOs
    const { utxos: selectedUTXOs, fee } = utxoService.selectOptimalUTXOs(
        utxos,
        1000000,  // 0.01 BTC
        feeRate
    );
    
    // Create transaction
    const tx = await txService.createTransaction(
        selectedUTXOs,
        [
            {
                address: 'recipient_address',
                value: 1000000
            },
            {
                address: 'change_address',
                value: selectedUTXOs.reduce((sum, utxo) => sum + utxo.value, 0) - 1000000 - fee
            }
        ],
        [privateKey]
    );
    
    return tx;
}
```

## 🎯 Knowledge Check
Try to answer these questions:
1. How is transaction size calculated?
2. What factors affect fee calculation?
3. How does UTXO selection impact fees?
4. When should multisig be used?

## 🛠️ Practical Exercises

### Exercise 1: Fee Calculator
Build a tool that:
- Estimates transaction size
- Calculates required fees
- Suggests optimal fee rates
- Handles different priorities

### Exercise 2: UTXO Selector
Create a system that:
- Implements coin selection
- Optimizes for fees
- Handles change outputs
- Manages UTXO consolidation

## 🔍 Common Issues and Solutions

### Issue 1: Fee Estimation
Like calculating processing costs:
- Inaccurate size estimation
- Changing fee market
- Priority considerations
- Minimum relay fee

### Issue 2: UTXO Selection
Like managing account balances:
- Insufficient funds
- Dust outputs
- Change management
- Privacy considerations

## 📚 Additional Resources
- [Transaction Building Guide](https://developer.bitcoin.org/examples/transactions.html)
- [Fee Estimation](https://bitcoincore.org/en/2016/12/fee-estimation/)
- [UTXO Management](https://bitcoin.org/en/developer-guide#term-utxo)
- [Multisig Guide](https://developer.bitcoin.org/devguide/transactions.html#multisig)

Remember: Transaction building requires careful attention to detail. Always verify transactions before broadcasting and consider implementing proper error handling and validation. 
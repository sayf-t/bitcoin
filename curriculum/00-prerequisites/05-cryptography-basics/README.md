# Cryptography Basics for Bitcoin Core Development

## Overview
This guide covers fundamental cryptographic concepts essential for understanding Bitcoin Core's security mechanisms and transaction validation.

## Prerequisites
- Basic mathematics knowledge
- Understanding of binary operations
- Familiarity with C++ basics

## Core Concepts

### 1. Hash Functions

#### Cryptographic Hash Properties
```cpp
// Example of hash function usage
#include <crypto/sha256.h>

void HashExample() {
    // Message to hash
    std::string message = "Hello, Bitcoin!";
    
    // Create hash object
    CSHA256 hasher;
    
    // Add data
    hasher.Write((const unsigned char*)message.data(), message.size());
    
    // Finalize
    uint256 hash;
    hasher.Finalize(hash.begin());
    
    // Properties demonstrated:
    // 1. Deterministic: Same input always produces same hash
    // 2. Quick to compute
    // 3. Infeasible to reverse
    // 4. Small change in input changes output completely
    // 5. Collision resistant
}
```

#### Common Hash Functions
```cpp
// Double SHA256 (Bitcoin's main hash function)
uint256 CalculateHash(const std::vector<unsigned char>& data) {
    return Hash(data.begin(), data.end());
}

// RIPEMD160(SHA256()) for addresses
uint160 CalculateHash160(const std::vector<unsigned char>& data) {
    return Hash160(data);
}
```

### 2. Public Key Cryptography

#### Key Generation
```cpp
// Example of key pair generation
#include <key.h>

void KeyGeneration() {
    // Generate private key
    CKey privateKey;
    privateKey.MakeNewKey(true);  // true for compressed
    
    // Get public key
    CPubKey publicKey = privateKey.GetPubKey();
    
    // Verify key pair
    assert(privateKey.VerifyPubKey(publicKey));
}
```

#### Digital Signatures
```cpp
// Signing and verification example
bool SignAndVerify(const CKey& privateKey, const uint256& hash) {
    // Sign message
    std::vector<unsigned char> signature;
    if (!privateKey.Sign(hash, signature))
        return false;
    
    // Verify signature
    CPubKey pubKey = privateKey.GetPubKey();
    return pubKey.Verify(hash, signature);
}
```

### 3. Merkle Trees

#### Tree Construction
```cpp
// Simplified Merkle tree implementation
class MerkleTree {
private:
    std::vector<uint256> leaves;
    
public:
    void AddLeaf(const uint256& hash) {
        leaves.push_back(hash);
    }
    
    uint256 CalculateRoot() {
        std::vector<uint256> nodes = leaves;
        
        while (nodes.size() > 1) {
            if (nodes.size() % 2 != 0)
                nodes.push_back(nodes.back());
                
            std::vector<uint256> parents;
            for (size_t i = 0; i < nodes.size(); i += 2) {
                uint256 parent = ComputeHash(nodes[i], nodes[i + 1]);
                parents.push_back(parent);
            }
            nodes = std::move(parents);
        }
        
        return nodes.empty() ? uint256() : nodes[0];
    }
    
private:
    uint256 ComputeHash(const uint256& left, const uint256& right) {
        CSHA256 hasher;
        hasher.Write(left.begin(), 32);
        hasher.Write(right.begin(), 32);
        uint256 result;
        hasher.Finalize(result.begin());
        return result;
    }
};
```

### 4. Basic Cryptographic Math

#### Modular Arithmetic
```cpp
// Examples of modular arithmetic
class ModularMath {
public:
    // Addition modulo n
    static uint256 AddMod(const uint256& a, const uint256& b, const uint256& n) {
        return (a + b) % n;
    }
    
    // Multiplication modulo n
    static uint256 MulMod(const uint256& a, const uint256& b, const uint256& n) {
        return (a * b) % n;
    }
    
    // Modular exponentiation
    static uint256 PowMod(uint256 base, uint256 exp, const uint256& n) {
        uint256 result = 1;
        base = base % n;
        while (exp > 0) {
            if (exp & 1)
                result = (result * base) % n;
            base = (base * base) % n;
            exp >>= 1;
        }
        return result;
    }
};
```

#### Elliptic Curve Operations
```cpp
// Example of secp256k1 curve usage
#include <secp256k1.h>

class ECOperations {
private:
    secp256k1_context* ctx;
    
public:
    ECOperations() {
        ctx = secp256k1_context_create(
            SECP256K1_CONTEXT_SIGN | SECP256K1_CONTEXT_VERIFY
        );
    }
    
    ~ECOperations() {
        secp256k1_context_destroy(ctx);
    }
    
    // Point multiplication
    bool MultiplyPoint(const unsigned char* privkey,
                      unsigned char* pubkey) {
        secp256k1_pubkey pubkey_struct;
        if (!secp256k1_ec_pubkey_create(ctx, &pubkey_struct, privkey))
            return false;
            
        size_t len = 33;
        return secp256k1_ec_pubkey_serialize(
            ctx, pubkey, &len, &pubkey_struct,
            SECP256K1_EC_COMPRESSED
        );
    }
};
```

## Exercises

### Exercise 1: Hash Functions
Implement a simple proof-of-work system:
```cpp
class ProofOfWork {
    // TODO: Implement proof-of-work using SHA256
    // - Adjust difficulty
    // - Find nonce
    // - Verify solution
};
```

### Exercise 2: Digital Signatures
Create a basic transaction signing system:
```cpp
class TransactionSigner {
    // TODO: Implement transaction signing
    // - Generate key pair
    // - Sign transaction
    // - Verify signature
};
```

### Exercise 3: Merkle Trees
Build a simplified block header system:
```cpp
class BlockHeader {
    // TODO: Implement Merkle root calculation
    // - Add transactions
    // - Calculate root
    // - Verify inclusion
};
```

## Best Practices

1. Key Management
   - Secure key generation
   - Safe key storage
   - Key rotation
   - Access control

2. Cryptographic Operations
   - Use standard libraries
   - Check return values
   - Handle errors
   - Validate inputs

3. Performance
   - Cache results
   - Batch operations
   - Use optimized implementations
   - Profile critical paths

4. Security
   - Constant-time operations
   - Memory cleanup
   - Input validation
   - Error handling

## Common Pitfalls

1. Cryptographic Mistakes
   - Weak key generation
   - Nonce reuse
   - Timing attacks
   - Memory leaks

2. Implementation Issues
   - Buffer overflows
   - Integer overflows
   - Side-channel leaks
   - Validation bypasses

3. Performance Problems
   - Unnecessary operations
   - Poor algorithm choice
   - Memory management
   - Resource leaks

## Additional Resources

1. Books
   - "Cryptography Engineering" by Ferguson, Schneier, and Kohno
   - "Real-World Cryptography" by David Wong
   - "Bitcoin and Cryptocurrency Technologies" by Narayanan et al.

2. Online Resources
   - Bitcoin Wiki
   - Cryptography Stack Exchange
   - Bitcoin Core documentation
   - Academic papers

3. Tools
   - OpenSSL
   - libsecp256k1
   - Crypto++ library
   - Test vectors

## Assessment

Complete these tasks to validate your understanding:
1. Implement basic cryptographic primitives
2. Create secure key management system
3. Build Merkle tree implementation
4. Develop signature scheme

## Next Steps
After mastering these concepts:
1. Study Bitcoin's cryptographic design
2. Review security considerations
3. Explore advanced topics
4. Contribute improvements
``` 
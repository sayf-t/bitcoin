# Cryptography Basics Exercises

## Hash Functions

### Exercise 1: Hash Properties
1. Write a program that demonstrates the following properties of SHA-256:
   - Deterministic output (same input always produces same output)
   - Avalanche effect (small input changes cause large output changes)
   - Fixed output size
   - One-way function (impossibility of reverse engineering)

2. Create test cases that verify these properties.

### Exercise 2: Bitcoin-Style Double Hashing
1. Implement a function that performs Bitcoin's double SHA-256 hashing
2. Create a test suite with known test vectors from Bitcoin
3. Demonstrate why double hashing might be more secure than single hashing
4. Compare performance between single and double hashing

## Public Key Cryptography

### Exercise 1: Key Pair Generation
1. Create a program that generates ECDSA key pairs using the secp256k1 curve
2. Implement proper key serialization and deserialization
3. Add error handling for invalid keys
4. Create a function to verify key pair relationships

### Exercise 2: Digital Signatures
1. Implement a complete signing and verification system:
   ```python
   def create_signature(private_key, message):
       # Your implementation here
       pass

   def verify_signature(public_key, message, signature):
       # Your implementation here
       pass
   ```
2. Add test cases for:
   - Valid signatures
   - Invalid signatures
   - Modified messages
   - Wrong public keys

## Base58 Encoding

### Exercise 1: Complete Implementation
1. Implement both Base58 encoding and decoding functions:
   ```python
   def encode_base58(data):
       # Your implementation here
       pass

   def decode_base58(encoded):
       # Your implementation here
       pass
   ```
2. Add checksum functionality
3. Test with Bitcoin address examples
4. Handle edge cases (empty input, invalid characters)

### Exercise 2: Base58Check
1. Implement Bitcoin's Base58Check encoding:
   - Add version bytes
   - Calculate and append checksum
   - Encode in Base58
2. Create test cases using real Bitcoin addresses
3. Implement validation for Base58Check encoded data

## Merkle Trees

### Exercise 1: Complete Implementation
1. Implement a full Merkle tree class:
   ```python
   class MerkleTree:
       def __init__(self, leaves):
           # Initialize the tree
           pass
           
       def get_root(self):
           # Return Merkle root
           pass
           
       def get_proof(self, leaf_index):
           # Generate Merkle proof
           pass
           
       def verify_proof(self, leaf, proof, root):
           # Verify Merkle proof
           pass
   ```
2. Add methods to:
   - Add new leaves
   - Generate proofs
   - Verify proofs
3. Test with different tree sizes

### Exercise 2: Bitcoin Block Merkle Trees
1. Create a program that:
   - Takes transaction IDs as input
   - Builds a Merkle tree
   - Generates the Merkle root
   - Validates transaction inclusion
2. Use real Bitcoin block data for testing
3. Implement the same duplicate last hash rule as Bitcoin

## Key Derivation

### Exercise 1: BIP39 Implementation
1. Implement core BIP39 functionality:
   - Mnemonic generation
   - Seed generation
   - Mnemonic validation
2. Support multiple languages
3. Add checksum verification
4. Test with official BIP39 test vectors

### Exercise 2: HD Wallet Key Derivation
1. Implement BIP32 hierarchical deterministic wallet derivation:
   ```python
   class HDWallet:
       def __init__(self, seed):
           # Initialize HD wallet
           pass
           
       def derive_child_key(self, index):
           # Derive child key
           pass
           
       def derive_path(self, path):
           # Derive key from path (e.g., "m/44'/0'/0'/0/0")
           pass
   ```
2. Implement hardened and non-hardened derivation
3. Test with official BIP32 test vectors

## Practical Applications

### Exercise 1: Address Generation
1. Create a complete Bitcoin address generator:
   - Support legacy addresses (P2PKH)
   - Support SegWit addresses (P2SH-P2WPKH)
   - Support native SegWit addresses (P2WPKH)
2. Implement address validation
3. Add test vectors for each address type

### Exercise 2: Transaction Signing
1. Implement basic transaction signing:
   ```python
   class Transaction:
       def __init__(self):
           self.inputs = []
           self.outputs = []
           
       def add_input(self, txid, vout, script_sig):
           # Add transaction input
           pass
           
       def add_output(self, amount, script_pubkey):
           # Add transaction output
           pass
           
       def sign(self, private_key):
           # Sign transaction
           pass
   ```
2. Support different signature types (SIGHASH flags)
3. Implement proper serialization
4. Test with real Bitcoin transactions

## Challenge Exercises

### Challenge 1: Mini Bitcoin Implementation
Create a simplified Bitcoin-like system implementing:
1. Block structure with Merkle tree
2. Basic transaction structure
3. Public key cryptography for ownership
4. Proof-of-work mechanism
5. Basic validation rules

### Challenge 2: Secure Communication System
Build a secure messaging system using:
1. Asymmetric encryption for key exchange
2. Symmetric encryption for messages
3. Digital signatures for authentication
4. Key derivation for session keys
5. Proper error handling and security measures

### Challenge 3: Cryptographic Protocol Analysis
1. Analyze a simple cryptographic protocol
2. Identify potential vulnerabilities
3. Implement attacks to demonstrate vulnerabilities
4. Propose and implement fixes
5. Document findings and recommendations

For each exercise:
- Write clear documentation
- Include error handling
- Add comprehensive test cases
- Consider edge cases
- Follow Bitcoin Core coding style
- Use appropriate security practices

Remember to never use these exercises for actual cryptocurrency operations - they are for learning purposes only. 
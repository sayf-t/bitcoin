# Cryptography Basics Demonstrations

## Hash Functions

### SHA-256 Examples
```python
import hashlib

# Basic string hashing
message = "Hello, Bitcoin!"
sha256_hash = hashlib.sha256(message.encode()).hexdigest()
print(f"SHA-256 of '{message}': {sha256_hash}")

# Double SHA-256 (commonly used in Bitcoin)
double_sha256 = hashlib.sha256(hashlib.sha256(message.encode()).digest()).hexdigest()
print(f"Double SHA-256: {double_sha256}")

# Hashing binary data
binary_data = b'\x00\x01\x02\x03'
binary_hash = hashlib.sha256(binary_data).hexdigest()
print(f"Binary data hash: {binary_hash}")
```

### RIPEMD160 Examples
```python
import hashlib

# Basic RIPEMD160 hashing
message = "Hello, Bitcoin!"
ripemd160_hash = hashlib.new('ripemd160', message.encode()).hexdigest()
print(f"RIPEMD160 of '{message}': {ripemd160_hash}")

# Bitcoin-style address hashing (SHA256 + RIPEMD160)
sha256_hash = hashlib.sha256(message.encode()).digest()
hash160 = hashlib.new('ripemd160', sha256_hash).hexdigest()
print(f"HASH160: {hash160}")
```

## Public Key Cryptography

### Key Generation
```python
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import ec
from cryptography.hazmat.primitives import serialization

# Generate ECDSA key pair (secp256k1 curve used in Bitcoin)
private_key = ec.generate_private_key(ec.SECP256K1())
public_key = private_key.public_key()

# Serialize keys
private_bytes = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.PKCS8,
    encryption_algorithm=serialization.NoEncryption()
)

public_bytes = public_key.public_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PublicFormat.SubjectPublicKeyInfo
)

print("Private Key:")
print(private_bytes.decode())
print("\nPublic Key:")
print(public_bytes.decode())
```

### Digital Signatures
```python
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import ec

# Message signing
message = b"Transaction data to sign"
signature = private_key.sign(
    message,
    ec.ECDSA(hashes.SHA256())
)
print(f"Signature: {signature.hex()}")

# Signature verification
try:
    public_key.verify(
        signature,
        message,
        ec.ECDSA(hashes.SHA256())
    )
    print("Signature is valid!")
except:
    print("Signature verification failed!")
```

## Base58 Encoding

### Basic Base58 Implementation
```python
import string

# Bitcoin-specific Base58 alphabet
ALPHABET = '123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz'

def encode_base58(data):
    # Convert bytes to integer
    n = int.from_bytes(data, byteorder='big')
    
    # Convert to base58 string
    result = ''
    while n > 0:
        n, r = divmod(n, 58)
        result = ALPHABET[r] + result
        
    # Add leading zeros
    for b in data:
        if b == 0:
            result = ALPHABET[0] + result
        else:
            break
            
    return result

# Example usage
data = b'\x00\x01\x02\x03'
encoded = encode_base58(data)
print(f"Base58 encoded: {encoded}")
```

## Merkle Trees

### Simple Merkle Tree Implementation
```python
import hashlib

def hash_leaf(data):
    """Hash a leaf node using double SHA256"""
    return hashlib.sha256(hashlib.sha256(data).digest()).digest()

def hash_nodes(left, right):
    """Hash two nodes together"""
    return hashlib.sha256(hashlib.sha256(left + right).digest()).digest()

def build_merkle_tree(leaves):
    """Build a Merkle tree from a list of leaves"""
    if len(leaves) == 0:
        return None
    
    # Hash all leaves
    nodes = [hash_leaf(leaf) for leaf in leaves]
    
    # Build tree bottom-up
    while len(nodes) > 1:
        new_nodes = []
        # Process pairs of nodes
        for i in range(0, len(nodes), 2):
            left = nodes[i]
            # If odd number of nodes, duplicate last one
            right = nodes[i + 1] if i + 1 < len(nodes) else left
            new_nodes.append(hash_nodes(left, right))
        nodes = new_nodes
    
    return nodes[0]

# Example usage
transactions = [
    b"tx1",
    b"tx2",
    b"tx3",
    b"tx4"
]

merkle_root = build_merkle_tree(transactions)
print(f"Merkle Root: {merkle_root.hex()}")
```

## Symmetric Encryption

### AES Example
```python
from cryptography.fernet import Fernet
from base64 import b64encode

# Generate a random key
key = Fernet.generate_key()
cipher_suite = Fernet(key)

# Encrypt data
message = b"Sensitive data"
encrypted_data = cipher_suite.encrypt(message)
print(f"Encrypted: {b64encode(encrypted_data).decode()}")

# Decrypt data
decrypted_data = cipher_suite.decrypt(encrypted_data)
print(f"Decrypted: {decrypted_data.decode()}")
```

## Key Derivation

### BIP39 Mnemonic Example
```python
from mnemonic import Mnemonic
import binascii

# Generate mnemonic
mnemo = Mnemonic("english")
words = mnemo.generate(strength=128)  # 12 words
print(f"Mnemonic: {words}")

# Generate seed from mnemonic
seed = mnemo.to_seed(words, passphrase="")
print(f"Seed: {binascii.hexlify(seed).decode()}")
```

## Practical Applications

### Bitcoin Address Generation
```python
import hashlib
import base58

def generate_bitcoin_address(public_key_bytes):
    """Generate a Bitcoin address from a public key"""
    # Step 1: SHA-256
    sha256_hash = hashlib.sha256(public_key_bytes).digest()
    
    # Step 2: RIPEMD160
    ripemd160_hash = hashlib.new('ripemd160', sha256_hash).digest()
    
    # Step 3: Add version byte (0x00 for mainnet)
    version_hash = b'\x00' + ripemd160_hash
    
    # Step 4: Double SHA-256 for checksum
    checksum = hashlib.sha256(hashlib.sha256(version_hash).digest()).digest()[:4]
    
    # Step 5: Combine and Base58 encode
    binary_address = version_hash + checksum
    address = base58.b58encode(binary_address).decode()
    
    return address

# Example usage (with dummy public key)
dummy_public_key = b'\x04' + b'\x00' * 64  # Uncompressed public key format
address = generate_bitcoin_address(dummy_public_key)
print(f"Bitcoin Address: {address}")
```

Each demonstration includes practical examples that are relevant to Bitcoin Core development. Try running these examples and experiment with different inputs to better understand the cryptographic concepts. 
# C++ Fundamentals Exercises

## Project-Based Learning Path

### Project 1: Simple Wallet Implementation
Build a basic wallet system that can:
- Create new wallets
- Handle deposits and withdrawals
- Track balances
- Support multiple accounts

```cpp
// wallet.h - Start with this template
class Wallet {
public:
    // TODO: Implement constructor
    // TODO: Add deposit method
    // TODO: Add withdraw method
    // TODO: Add balance query method
private:
    // TODO: Add necessary member variables
};

// Your implementation should support:
Wallet wallet("Alice");
wallet.deposit(1.5);
wallet.withdraw(0.5);
double balance = wallet.getBalance();
```

#### Steps:
1. Implement basic wallet functionality
2. Add input validation
3. Add error handling
4. Add transaction history
5. Write unit tests

### Project 2: Transaction Manager
Create a system to manage Bitcoin transactions:
```cpp
// transaction.h - Start with this template
class Transaction {
public:
    // TODO: Implement constructor
    // TODO: Add validation methods
    // TODO: Add getter/setter methods
private:
    // TODO: Add necessary member variables
};

class TransactionManager {
public:
    // TODO: Add method to add transaction
    // TODO: Add method to verify transaction
    // TODO: Add method to list transactions
private:
    // TODO: Add container for transactions
};

// Your implementation should support:
TransactionManager manager;
manager.addTransaction("Alice", "Bob", 0.1);
manager.listTransactions();
bool isValid = manager.verifyTransaction(txId);
```

#### Steps:
1. Implement transaction structure
2. Add transaction validation
3. Implement transaction storage
4. Add search functionality
5. Add reporting features

### Project 3: Memory-Safe Block Chain
Implement a simple blockchain using modern C++ features:
```cpp
// blockchain.h - Start with this template
class Block {
public:
    // TODO: Implement constructor
    // TODO: Add hash calculation
    // TODO: Add validation methods
private:
    // TODO: Add block data members
};

class Blockchain {
public:
    // TODO: Add method to add block
    // TODO: Add method to verify chain
    // TODO: Add method to query blocks
private:
    // TODO: Add container for blocks
};

// Your implementation should support:
Blockchain chain;
chain.addBlock({"transaction1", "transaction2"});
bool isValid = chain.verifyChain();
auto block = chain.getBlock(blockHeight);
```

#### Steps:
1. Implement block structure
2. Add block linking
3. Implement chain validation
4. Add query capabilities
5. Implement persistence

### Project 4: Thread-Safe Mempool
Create a thread-safe memory pool for transactions:
```cpp
// mempool.h - Start with this template
class Mempool {
public:
    // TODO: Implement thread-safe transaction addition
    // TODO: Implement thread-safe transaction removal
    // TODO: Add size limits and cleanup
private:
    // TODO: Add thread-safe container
    // TODO: Add synchronization primitives
};

// Your implementation should support:
Mempool pool;
pool.addTransaction(tx);  // Thread-safe
auto tx = pool.getTransaction(txId);  // Thread-safe
pool.removeTransaction(txId);  // Thread-safe
```

#### Steps:
1. Implement basic mempool
2. Add thread safety
3. Implement size limits
4. Add cleanup policy
5. Add monitoring

### Project 5: Error-Handling Framework
Build an error handling system for Bitcoin operations:
```cpp
// errors.h - Start with this template
class BitcoinError : public std::runtime_error {
    // TODO: Implement error hierarchy
};

class ValidationError : public BitcoinError {
    // TODO: Implement validation errors
};

class NetworkError : public BitcoinError {
    // TODO: Implement network errors
};

// Your implementation should support:
try {
    // Risky operation
} catch (const ValidationError& e) {
    // Handle validation error
} catch (const NetworkError& e) {
    // Handle network error
}
```

#### Steps:
1. Design error hierarchy
2. Implement error classes
3. Add error context
4. Add recovery mechanisms
5. Add logging

## Challenge Projects

### Challenge 1: Mini Bitcoin Node
Combine all previous projects into a mini Bitcoin node:
```cpp
class BitcoinNode {
    // TODO: Combine Wallet, TransactionManager,
    // Blockchain, and Mempool functionality
};
```

### Challenge 2: Testing Framework
Create a testing framework for Bitcoin components:
```cpp
class BitcoinTest {
    // TODO: Implement test fixtures
    // TODO: Add test cases
    // TODO: Add benchmarking
};
```

## Submission Requirements

### For Each Project:
1. Working code implementation
2. Unit tests
3. Documentation
4. Example usage
5. Performance analysis

### Code Style:
1. Follow Bitcoin Core style guide
2. Use modern C++ features
3. Handle errors appropriately
4. Include comments
5. Use meaningful names

## Evaluation Criteria

### Basic Projects (1-5):
- Functionality (40%)
- Code quality (30%)
- Error handling (20%)
- Documentation (10%)

### Challenge Projects:
- Integration (40%)
- Extensibility (30%)
- Performance (20%)
- Documentation (10%)

## Getting Help

### Resources:
1. C++ Reference
2. Bitcoin Core source code
3. Stack Overflow
4. Project Discord

### Common Issues:
1. Memory management
2. Thread synchronization
3. Error handling
4. Build problems

## Next Steps

After completing these exercises:
1. Review Bitcoin Core codebase
2. Attempt small fixes
3. Join development discussions
4. Start contributing
``` 
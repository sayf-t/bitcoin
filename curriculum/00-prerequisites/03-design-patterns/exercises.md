# Design Patterns Exercises

## Exercise 1: SOLID Principles
Implement a transaction processing system that follows SOLID principles:

1. Create a `TransactionProcessor` class that:
   - Validates transactions (SRP)
   - Uses strategy pattern for different validation rules (OCP)
   - Implements proper interfaces (ISP)
   - Uses dependency injection for dependencies (DIP)

2. Requirements:
   - Must handle different transaction types
   - Must be extensible for new validation rules
   - Must use proper abstraction layers
   - Must be testable

Example structure:
```cpp
class ITransactionValidator {
    virtual bool Validate(const Transaction& tx) = 0;
};

class TransactionProcessor {
    // Your implementation here
};
```

## Exercise 2: Creational Patterns
Implement a block creation system using Factory and Builder patterns:

1. Create a block factory that:
   - Supports different network types (mainnet, testnet, regtest)
   - Uses proper inheritance hierarchy
   - Implements builder pattern for block construction

2. Requirements:
   - Must handle different block versions
   - Must support custom block parameters
   - Must be thread-safe
   - Must validate block structure

Example structure:
```cpp
class BlockFactory {
    virtual std::unique_ptr<Block> CreateBlock() = 0;
};

class BlockBuilder {
    // Your implementation here
};
```

## Exercise 3: Structural Patterns
Implement a blockchain data access system using Adapter and Decorator patterns:

1. Create adapters for:
   - Legacy block storage
   - Modern block storage
   - Different database backends

2. Create decorators for:
   - Logging
   - Caching
   - Performance monitoring

Example structure:
```cpp
class IBlockStorage {
    virtual void StoreBlock(const Block& block) = 0;
    virtual Block LoadBlock(const BlockHash& hash) = 0;
};

class BlockStorageDecorator : public IBlockStorage {
    // Your implementation here
};
```

## Exercise 4: Behavioral Patterns
Implement a node synchronization system using Observer and Chain of Responsibility patterns:

1. Create an observer system for:
   - New block notifications
   - Transaction notifications
   - Peer connection events

2. Create a validation chain for:
   - Block validation
   - Transaction validation
   - Network message validation

Example structure:
```cpp
class ISyncObserver {
    virtual void OnNewBlock(const Block& block) = 0;
    virtual void OnNewTransaction(const Transaction& tx) = 0;
};

class ValidationChain {
    // Your implementation here
};
```

## Exercise 5: Thread Safety Patterns
Implement a thread-safe mempool management system:

1. Create thread-safe components for:
   - Transaction queue
   - Memory pool
   - Fee estimation

2. Requirements:
   - Must handle concurrent access
   - Must prevent race conditions
   - Must use proper synchronization primitives
   - Must be deadlock-free

Example structure:
```cpp
class ThreadSafeMempool {
    // Your implementation here
};

class ThreadSafeFeeEstimator {
    // Your implementation here
};
```

## Exercise 6: Integration Challenge
Create a mini-blockchain system that combines multiple patterns:

1. Requirements:
   - Use Factory pattern for block/transaction creation
   - Use Observer pattern for network events
   - Use Chain of Responsibility for validation
   - Use Decorator for logging/monitoring
   - Use Thread-safe components for concurrent access

2. The system should:
   - Create and validate blocks
   - Process transactions
   - Maintain a simple chain
   - Handle concurrent access
   - Log all operations

Example structure:
```cpp
class MiniBlockchain {
    // Your implementation here
};

class BlockchainNode {
    // Your implementation here
};
```

## Submission Guidelines

1. For each exercise:
   - Create a separate directory
   - Include all source files
   - Include a README with build instructions
   - Include unit tests
   - Document your design decisions

2. Code requirements:
   - Must compile without warnings
   - Must pass all unit tests
   - Must follow Bitcoin Core coding style
   - Must include proper error handling
   - Must be well-documented

3. Testing requirements:
   - Include unit tests for each component
   - Include integration tests where applicable
   - Include performance tests for thread-safe components
   - Include stress tests for concurrent operations

## Evaluation Criteria

Your solutions will be evaluated based on:

1. Correct implementation of design patterns
2. Code quality and style
3. Error handling and robustness
4. Test coverage and quality
5. Documentation quality
6. Performance considerations
7. Thread safety implementation
8. Integration of multiple patterns

## Bonus Challenges

1. Implement a custom pattern that could be useful in Bitcoin Core
2. Create a proposal for refactoring existing Bitcoin Core code using patterns
3. Implement a pattern that improves performance in a critical section
4. Create a new thread-safe pattern for a specific Bitcoin Core use case

## Resources

1. Bitcoin Core source code
2. C++ Standard Library documentation
3. Design Patterns: Elements of Reusable Object-Oriented Software
4. Modern C++ Design Patterns
``` 
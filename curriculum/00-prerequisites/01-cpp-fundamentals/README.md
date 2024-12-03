# C++ Fundamentals for Bitcoin Core Development

## Overview
This guide covers essential C++ concepts and features used throughout the Bitcoin Core codebase. Understanding these fundamentals is crucial for effective contribution.

## Prerequisites
- Basic C++ knowledge
- Understanding of object-oriented programming
- Familiarity with command-line compilation

## Core Concepts

### 1. Modern C++ Features (C++11/14/17)

#### Smart Pointers
```cpp
// Instead of raw pointers
std::unique_ptr<Block> block = std::make_unique<Block>();
std::shared_ptr<Transaction> tx = std::make_shared<Transaction>();

// RAII pattern
class BlockManager {
private:
    std::unique_ptr<Block> current_block;
public:
    BlockManager() : current_block(std::make_unique<Block>()) {}
    // No need for explicit deletion
};
```

#### Lambda Expressions
```cpp
// Function objects for callbacks
auto validateTx = [](const Transaction& tx) -> bool {
    // Validation logic
    return true;
};

// Used in algorithms
std::vector<Transaction> txs;
std::sort(txs.begin(), txs.end(), 
    [](const Transaction& a, const Transaction& b) {
        return a.getFee() > b.getFee();
    });
```

#### Move Semantics
```cpp
// Efficient resource transfer
class Block {
public:
    Block(Block&& other) noexcept;
    Block& operator=(Block&& other) noexcept;
private:
    std::vector<Transaction> transactions;
};
```

### 2. Memory Management and RAII

#### Resource Management
```cpp
// RAII for file handling
class LogFile {
private:
    std::ofstream file;
public:
    LogFile(const std::string& name) : file(name) {
        if (!file) throw std::runtime_error("Cannot open file");
    }
    // Automatically closes file on destruction
    ~LogFile() = default;
};
```

#### Lock Management
```cpp
// RAII for mutex locking
class Guard {
private:
    std::mutex& mtx;
public:
    Guard(std::mutex& m) : mtx(m) { mtx.lock(); }
    ~Guard() { mtx.unlock(); }
};
```

### 3. Templates and Generic Programming

#### Class Templates
```cpp
template<typename Hash>
class BlockIndex {
private:
    Hash block_hash;
public:
    BlockIndex(const Hash& hash) : block_hash(hash) {}
    const Hash& getHash() const { return block_hash; }
};
```

#### Function Templates
```cpp
template<typename T>
bool verifySignature(const T& message, const Signature& sig) {
    // Verification logic
    return true;
}
```

### 4. Multithreading

#### Thread Management
```cpp
// Thread creation and management
class NodeManager {
private:
    std::thread validation_thread;
    std::atomic<bool> running{false};
    
public:
    void start() {
        running = true;
        validation_thread = std::thread([this]() {
            while (running) {
                // Validation work
            }
        });
    }
    
    void stop() {
        running = false;
        if (validation_thread.joinable())
            validation_thread.join();
    }
};
```

#### Synchronization
```cpp
class ThreadSafeQueue {
private:
    std::queue<Transaction> queue;
    mutable std::mutex mtx;
    std::condition_variable cv;
    
public:
    void push(Transaction tx) {
        std::lock_guard<std::mutex> lock(mtx);
        queue.push(std::move(tx));
        cv.notify_one();
    }
    
    Transaction pop() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return !queue.empty(); });
        Transaction tx = std::move(queue.front());
        queue.pop();
        return tx;
    }
};
```

### 5. STL Containers and Algorithms

#### Container Usage
```cpp
// Common container patterns
std::vector<Transaction> mempool;
std::unordered_map<uint256, Block> block_index;
std::set<NodeId> connected_peers;

// Container algorithms
auto it = std::find_if(mempool.begin(), mempool.end(),
    [](const Transaction& tx) { return tx.getFee() > MIN_FEE; });

std::sort(mempool.begin(), mempool.end(),
    [](const Transaction& a, const Transaction& b) {
        return a.getFee() > b.getFee();
    });
```

## Exercises

### Exercise 1: Smart Pointer Usage
Implement a transaction manager using smart pointers:
```cpp
class TransactionManager {
    // TODO: Implement using unique_ptr and shared_ptr
};
```

### Exercise 2: Thread-Safe Queue
Implement a thread-safe queue for transactions:
```cpp
class TxQueue {
    // TODO: Implement using mutex and condition_variable
};
```

### Exercise 3: Template Implementation
Create a generic validator:
```cpp
template<typename T>
class Validator {
    // TODO: Implement validation logic
};
```

## Best Practices

1. Memory Management
   - Use smart pointers by default
   - Implement RAII for resources
   - Avoid raw new/delete

2. Error Handling
   - Use exceptions for exceptional cases
   - Return values for expected failures
   - Always clean up resources

3. Performance
   - Use move semantics for large objects
   - Avoid unnecessary copying
   - Consider cache coherency

4. Thread Safety
   - Use RAII for locks
   - Avoid shared mutable state
   - Use atomic operations appropriately

## Common Pitfalls

1. Memory Leaks
   - Not using smart pointers
   - Circular shared_ptr references
   - Raw pointer deletion

2. Threading Issues
   - Deadlocks
   - Race conditions
   - Lock contention

3. Exception Safety
   - Resource leaks
   - Inconsistent state
   - Uncaught exceptions

## Additional Resources

1. Books
   - "Effective Modern C++" by Scott Meyers
   - "C++ Concurrency in Action" by Anthony Williams

2. Online Resources
   - CPPReference.com
   - ISO C++ Guidelines
   - CppCon Talks

3. Practice Resources
   - Compiler Explorer
   - C++ Insights
   - Practice Problems

## Assessment

Complete these tasks to validate your understanding:
1. Implement a thread-safe block cache
2. Create a generic validation framework
3. Build a memory-safe resource manager
4. Develop a concurrent transaction processor

## Next Steps
After mastering these concepts:
1. Review Bitcoin Core coding guidelines
2. Study existing codebase implementations
3. Practice with small contributions
4. Join development discussions
``` 
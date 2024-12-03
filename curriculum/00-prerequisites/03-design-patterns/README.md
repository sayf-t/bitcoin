# Design Patterns for Bitcoin Core Development

## Overview
This guide covers essential design patterns and principles used throughout the Bitcoin Core codebase. Understanding these patterns is crucial for writing maintainable, efficient, and secure code.

## Prerequisites
- Basic C++ knowledge
- Object-oriented programming concepts
- Basic software design principles

## Core Concepts

### 1. SOLID Principles

#### Single Responsibility Principle (SRP)
```cpp
// Good: Each class has a single responsibility
class TransactionValidator {
    bool ValidateTransaction(const CTransaction& tx) {
        // Validation logic only
    }
};

class TransactionRelay {
    bool RelayTransaction(const CTransaction& tx) {
        // Relay logic only
    }
};

// Bad: Mixed responsibilities
class TransactionHandler {
    bool ValidateAndRelay(const CTransaction& tx) {
        // Mixed validation and relay logic
    }
};
```

#### Open-Closed Principle (OCP)
```cpp
// Base validation rule interface
class ValidationRule {
public:
    virtual bool Check(const CTransaction& tx) = 0;
    virtual ~ValidationRule() = default;
};

// New rules can be added without modifying existing code
class InputValidationRule : public ValidationRule {
public:
    bool Check(const CTransaction& tx) override {
        // Input validation logic
        return true;
    }
};

class FeeValidationRule : public ValidationRule {
public:
    bool Check(const CTransaction& tx) override {
        // Fee validation logic
        return true;
    }
};
```

#### Liskov Substitution Principle (LSP)
```cpp
class Node {
public:
    virtual bool ProcessMessage(const CMessage& msg) = 0;
};

class FullNode : public Node {
public:
    bool ProcessMessage(const CMessage& msg) override {
        // Full node processing
        return true;
    }
};

class LightNode : public Node {
public:
    bool ProcessMessage(const CMessage& msg) override {
        // Light node processing
        return true;
    }
};
```

#### Interface Segregation Principle (ISP)
```cpp
// Good: Segregated interfaces
class IBlockValidator {
    virtual bool CheckBlock(const CBlock& block) = 0;
};

class IBlockPropagator {
    virtual bool PropagateBlock(const CBlock& block) = 0;
};

// Bad: Bloated interface
class IBlockHandler {
    virtual bool CheckBlock(const CBlock& block) = 0;
    virtual bool PropagateBlock(const CBlock& block) = 0;
    virtual bool StoreBlock(const CBlock& block) = 0;
    virtual bool IndexBlock(const CBlock& block) = 0;
};
```

#### Dependency Inversion Principle (DIP)
```cpp
// High-level module depends on abstraction
class Wallet {
private:
    std::unique_ptr<ISigningProvider> signer;
public:
    Wallet(std::unique_ptr<ISigningProvider> s) : signer(std::move(s)) {}
    
    bool SignTransaction(CTransaction& tx) {
        return signer->Sign(tx);
    }
};

// Low-level modules implement the abstraction
class HardwareWalletSigner : public ISigningProvider {
    bool Sign(CTransaction& tx) override {
        // Hardware wallet signing implementation
        return true;
    }
};
```

### 2. Common Bitcoin Core Patterns

#### Singleton Pattern (Used Sparingly)
```cpp
class ChainState {
private:
    static ChainState* instance;
    ChainState() = default;

public:
    static ChainState& GetInstance() {
        static ChainState instance;
        return instance;
    }
    
    // Delete copy/move operations
    ChainState(const ChainState&) = delete;
    ChainState& operator=(const ChainState&) = delete;
};
```

#### Factory Pattern
```cpp
class ScriptFactory {
public:
    static std::unique_ptr<CScript> CreateScript(ScriptType type) {
        switch (type) {
            case ScriptType::P2PKH:
                return std::make_unique<P2PKHScript>();
            case ScriptType::P2SH:
                return std::make_unique<P2SHScript>();
            default:
                return nullptr;
        }
    }
};
```

#### Observer Pattern
```cpp
class BlockNotifier {
private:
    std::vector<std::weak_ptr<BlockObserver>> observers;

public:
    void AddObserver(std::shared_ptr<BlockObserver> observer) {
        observers.push_back(observer);
    }
    
    void NotifyNewBlock(const CBlock& block) {
        for (auto it = observers.begin(); it != observers.end();) {
            if (auto observer = it->lock()) {
                observer->OnNewBlock(block);
                ++it;
            } else {
                it = observers.erase(it);
            }
        }
    }
};
```

### 3. Threading Patterns

#### Producer-Consumer Pattern
```cpp
class MessageQueue {
private:
    std::queue<CMessage> queue;
    std::mutex mtx;
    std::condition_variable cv;

public:
    void Produce(CMessage msg) {
        std::lock_guard<std::mutex> lock(mtx);
        queue.push(std::move(msg));
        cv.notify_one();
    }
    
    CMessage Consume() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]{ return !queue.empty(); });
        CMessage msg = std::move(queue.front());
        queue.pop();
        return msg;
    }
};
```

#### Active Object Pattern
```cpp
class ValidationScheduler {
private:
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable cv;
    std::thread worker;
    bool running = false;

public:
    void Start() {
        running = true;
        worker = std::thread([this]{ ProcessTasks(); });
    }
    
    void Stop() {
        {
            std::lock_guard<std::mutex> lock(mtx);
            running = false;
        }
        cv.notify_one();
        if (worker.joinable()) {
            worker.join();
        }
    }
    
    void ScheduleValidation(const CTransaction& tx) {
        std::lock_guard<std::mutex> lock(mtx);
        tasks.push([tx]{ ValidateTransaction(tx); });
        cv.notify_one();
    }

private:
    void ProcessTasks() {
        while (true) {
            std::function<void()> task;
            {
                std::unique_lock<std::mutex> lock(mtx);
                cv.wait(lock, [this]{ 
                    return !running || !tasks.empty(); 
                });
                if (!running && tasks.empty()) {
                    return;
                }
                task = std::move(tasks.front());
                tasks.pop();
            }
            task();
        }
    }
};
```

### 4. Error Handling Patterns

#### Result Type Pattern
```cpp
template<typename T>
class Result {
private:
    std::variant<T, std::string> data;

public:
    Result(T value) : data(std::move(value)) {}
    Result(std::string error) : data(std::move(error)) {}
    
    bool IsOk() const { return data.index() == 0; }
    bool IsError() const { return data.index() == 1; }
    
    const T& Value() const { return std::get<0>(data); }
    const std::string& Error() const { return std::get<1>(data); }
};

// Usage example
Result<CTransaction> CreateTransaction() {
    try {
        // Create transaction
        return Result<CTransaction>(tx);
    } catch (const std::exception& e) {
        return Result<CTransaction>(e.what());
    }
}
```

## Exercises

### Exercise 1: SOLID Principles
Refactor a transaction processor to follow SOLID:
```cpp
// TODO: Refactor this class
class TransactionProcessor {
    // Currently handles validation, relay, and storage
};
```

### Exercise 2: Observer Implementation
Create a mempool observer system:
```cpp
// TODO: Implement observer pattern
class MempoolMonitor {
    // Should notify about new/removed transactions
};
```

### Exercise 3: Thread-Safe Pattern
Implement a thread-safe block cache:
```cpp
// TODO: Implement thread-safe cache
class BlockCache {
    // Should handle concurrent access safely
};
```

## Best Practices

1. Pattern Selection
   - Choose patterns based on needs
   - Avoid overengineering
   - Consider maintenance
   - Document pattern usage

2. Implementation
   - Keep it simple
   - Follow conventions
   - Write tests
   - Document assumptions

3. Threading
   - Use RAII
   - Avoid deadlocks
   - Handle cleanup
   - Consider performance

4. Error Handling
   - Be consistent
   - Provide context
   - Clean up resources
   - Log appropriately

## Common Pitfalls

1. Pattern Misuse
   - Overcomplicating solutions
   - Wrong pattern selection
   - Poor implementation
   - Lack of documentation

2. Threading Issues
   - Race conditions
   - Deadlocks
   - Resource leaks
   - Performance problems

3. Error Handling
   - Inconsistent approaches
   - Missing cleanup
   - Poor error messages
   - Ignored errors

## Additional Resources

1. Books
   - "Design Patterns" by Gang of Four
   - "Modern C++ Design" by Andrei Alexandrescu
   - "C++ Concurrency in Action" by Anthony Williams

2. Online Resources
   - Bitcoin Core developer documentation
   - C++ Core Guidelines
   - CppCon talks on patterns

3. Practice Resources
   - Bitcoin Core codebase examples
   - Pattern implementation exercises
   - Code review examples

## Assessment

Complete these tasks to validate your understanding:
1. Implement a validation chain using patterns
2. Create a thread-safe object pool
3. Design an error-handling system
4. Build a message processing pipeline

## Next Steps
After mastering these patterns:
1. Study Bitcoin Core's pattern usage
2. Practice pattern implementation
3. Contribute pattern improvements
4. Review others' pattern usage
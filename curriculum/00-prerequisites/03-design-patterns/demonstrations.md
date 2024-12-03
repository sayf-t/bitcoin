# Design Patterns Demonstrations

## Interactive Learning Path

### Demo 1: SOLID Principles in Action
```cpp
// demo1_solid.cpp
#include <iostream>
#include <vector>
#include <memory>

// Single Responsibility Principle (SRP)
class TransactionValidator {
public:
    bool ValidateTransaction(const std::string& tx) {
        // Validation logic only
        return true;
    }
};

class TransactionBroadcaster {
public:
    void BroadcastTransaction(const std::string& tx) {
        // Broadcasting logic only
        std::cout << "Broadcasting transaction: " << tx << "\n";
    }
};

// Open-Closed Principle (OCP)
class ValidationRule {
public:
    virtual bool Validate(const std::string& tx) = 0;
    virtual ~ValidationRule() = default;
};

class AmountValidationRule : public ValidationRule {
public:
    bool Validate(const std::string& tx) override {
        std::cout << "Validating amount for tx: " << tx << "\n";
        return true;
    }
};

class SignatureValidationRule : public ValidationRule {
public:
    bool Validate(const std::string& tx) override {
        std::cout << "Validating signature for tx: " << tx << "\n";
        return true;
    }
};

// Liskov Substitution Principle (LSP)
class Node {
public:
    virtual bool ProcessBlock(const std::string& block) {
        std::cout << "Processing block: " << block << "\n";
        return true;
    }
};

class FullNode : public Node {
public:
    bool ProcessBlock(const std::string& block) override {
        std::cout << "Full node processing block: " << block << "\n";
        return true;
    }
};

// Interface Segregation Principle (ISP)
class BlockValidator {
public:
    virtual bool ValidateBlock(const std::string& block) = 0;
    virtual ~BlockValidator() = default;
};

class BlockPropagator {
public:
    virtual void PropagateBlock(const std::string& block) = 0;
    virtual ~BlockPropagator() = default;
};

// Dependency Inversion Principle (DIP)
class IStorage {
public:
    virtual void Save(const std::string& data) = 0;
    virtual ~IStorage() = default;
};

class DiskStorage : public IStorage {
public:
    void Save(const std::string& data) override {
        std::cout << "Saving to disk: " << data << "\n";
    }
};

class BlockProcessor {
private:
    std::unique_ptr<IStorage> storage_;

public:
    explicit BlockProcessor(std::unique_ptr<IStorage> storage)
        : storage_(std::move(storage)) {}

    void ProcessAndStore(const std::string& block) {
        // Process block
        storage_->Save(block);
    }
};

int main() {
    // Demonstrate SRP
    TransactionValidator validator;
    TransactionBroadcaster broadcaster;
    
    std::string tx = "sample_transaction";
    if (validator.ValidateTransaction(tx)) {
        broadcaster.BroadcastTransaction(tx);
    }

    // Demonstrate OCP
    std::vector<std::unique_ptr<ValidationRule>> rules;
    rules.push_back(std::make_unique<AmountValidationRule>());
    rules.push_back(std::make_unique<SignatureValidationRule>());
    
    for (const auto& rule : rules) {
        rule->Validate(tx);
    }

    // Demonstrate LSP
    std::unique_ptr<Node> node = std::make_unique<FullNode>();
    node->ProcessBlock("sample_block");

    // Demonstrate DIP
    auto storage = std::make_unique<DiskStorage>();
    BlockProcessor processor(std::move(storage));
    processor.ProcessAndStore("sample_block");

    return 0;
}
```

### Demo 2: Creational Patterns
```cpp
// demo2_creational.cpp
#include <iostream>
#include <memory>
#include <string>
#include <unordered_map>

// Singleton Pattern
class ChainState {
private:
    static ChainState* instance_;
    ChainState() = default;

public:
    static ChainState* GetInstance() {
        if (instance_ == nullptr) {
            instance_ = new ChainState();
        }
        return instance_;
    }

    void UpdateTip(const std::string& hash) {
        std::cout << "Updating chain tip to: " << hash << "\n";
    }
};
ChainState* ChainState::instance_ = nullptr;

// Factory Method Pattern
class Block {
public:
    virtual void Mine() = 0;
    virtual ~Block() = default;
};

class MainnetBlock : public Block {
public:
    void Mine() override {
        std::cout << "Mining mainnet block...\n";
    }
};

class TestnetBlock : public Block {
public:
    void Mine() override {
        std::cout << "Mining testnet block...\n";
    }
};

class BlockFactory {
public:
    virtual std::unique_ptr<Block> CreateBlock() = 0;
    virtual ~BlockFactory() = default;
};

class MainnetBlockFactory : public BlockFactory {
public:
    std::unique_ptr<Block> CreateBlock() override {
        return std::make_unique<MainnetBlock>();
    }
};

class TestnetBlockFactory : public BlockFactory {
public:
    std::unique_ptr<Block> CreateBlock() override {
        return std::make_unique<TestnetBlock>();
    }
};

// Builder Pattern
class Transaction {
public:
    void SetInputs(const std::string& inputs) {
        inputs_ = inputs;
    }
    void SetOutputs(const std::string& outputs) {
        outputs_ = outputs;
    }
    void SetLockTime(int locktime) {
        locktime_ = locktime;
    }
    void Show() const {
        std::cout << "Transaction:\n"
                  << "Inputs: " << inputs_ << "\n"
                  << "Outputs: " << outputs_ << "\n"
                  << "Locktime: " << locktime_ << "\n";
    }
private:
    std::string inputs_;
    std::string outputs_;
    int locktime_;
};

class TransactionBuilder {
public:
    TransactionBuilder& AddInputs(const std::string& inputs) {
        transaction_.SetInputs(inputs);
        return *this;
    }
    
    TransactionBuilder& AddOutputs(const std::string& outputs) {
        transaction_.SetOutputs(outputs);
        return *this;
    }
    
    TransactionBuilder& SetLockTime(int locktime) {
        transaction_.SetLockTime(locktime);
        return *this;
    }
    
    Transaction Build() {
        return transaction_;
    }
private:
    Transaction transaction_;
};

int main() {
    // Demonstrate Singleton
    ChainState* chainstate = ChainState::GetInstance();
    chainstate->UpdateTip("000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f");

    // Demonstrate Factory Method
    std::unique_ptr<BlockFactory> mainnetFactory = 
        std::make_unique<MainnetBlockFactory>();
    std::unique_ptr<BlockFactory> testnetFactory = 
        std::make_unique<TestnetBlockFactory>();
    
    auto mainnetBlock = mainnetFactory->CreateBlock();
    auto testnetBlock = testnetFactory->CreateBlock();
    
    mainnetBlock->Mine();
    testnetBlock->Mine();

    // Demonstrate Builder
    Transaction tx = TransactionBuilder()
        .AddInputs("previous_tx_1, previous_tx_2")
        .AddOutputs("address_1:50, address_2:50")
        .SetLockTime(100)
        .Build();
    
    tx.Show();

    return 0;
}
```

### Demo 3: Structural Patterns
```cpp
// demo3_structural.cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

// Adapter Pattern
class LegacyValidator {
public:
    bool ValidateOldFormat(const std::string& tx) {
        std::cout << "Validating old format: " << tx << "\n";
        return true;
    }
};

class ModernValidator {
public:
    virtual bool Validate(const std::string& tx) = 0;
    virtual ~ModernValidator() = default;
};

class ValidatorAdapter : public ModernValidator {
private:
    LegacyValidator legacy_;

public:
    bool Validate(const std::string& tx) override {
        return legacy_.ValidateOldFormat(tx);
    }
};

// Decorator Pattern
class Component {
public:
    virtual void Execute() = 0;
    virtual ~Component() = default;
};

class ConcreteComponent : public Component {
public:
    void Execute() override {
        std::cout << "Executing base component\n";
    }
};

class Decorator : public Component {
protected:
    std::unique_ptr<Component> component_;

public:
    explicit Decorator(std::unique_ptr<Component> component)
        : component_(std::move(component)) {}

    void Execute() override {
        if (component_) {
            component_->Execute();
        }
    }
};

class LoggingDecorator : public Decorator {
public:
    explicit LoggingDecorator(std::unique_ptr<Component> component)
        : Decorator(std::move(component)) {}

    void Execute() override {
        std::cout << "Logging: Starting execution\n";
        Decorator::Execute();
        std::cout << "Logging: Finished execution\n";
    }
};

// Composite Pattern
class BlockchainComponent {
public:
    virtual void Process() = 0;
    virtual ~BlockchainComponent() = default;
};

class Transaction : public BlockchainComponent {
public:
    void Process() override {
        std::cout << "Processing transaction\n";
    }
};

class Block : public BlockchainComponent {
private:
    std::vector<std::unique_ptr<BlockchainComponent>> components_;

public:
    void Add(std::unique_ptr<BlockchainComponent> component) {
        components_.push_back(std::move(component));
    }

    void Process() override {
        std::cout << "Processing block\n";
        for (const auto& component : components_) {
            component->Process();
        }
    }
};

int main() {
    // Demonstrate Adapter
    std::unique_ptr<ModernValidator> validator = 
        std::make_unique<ValidatorAdapter>();
    validator->Validate("sample_transaction");

    // Demonstrate Decorator
    auto component = std::make_unique<ConcreteComponent>();
    auto loggingComponent = 
        std::make_unique<LoggingDecorator>(std::move(component));
    loggingComponent->Execute();

    // Demonstrate Composite
    auto block = std::make_unique<Block>();
    block->Add(std::make_unique<Transaction>());
    block->Add(std::make_unique<Transaction>());
    block->Process();

    return 0;
}
```

### Demo 4: Behavioral Patterns
```cpp
// demo4_behavioral.cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>
#include <algorithm>

// Observer Pattern
class Observer {
public:
    virtual void Update(const std::string& message) = 0;
    virtual ~Observer() = default;
};

class BlockObserver : public Observer {
public:
    void Update(const std::string& message) override {
        std::cout << "Block Observer received: " << message << "\n";
    }
};

class TxObserver : public Observer {
public:
    void Update(const std::string& message) override {
        std::cout << "Transaction Observer received: " << message << "\n";
    }
};

class Subject {
private:
    std::vector<std::shared_ptr<Observer>> observers_;

public:
    void Attach(std::shared_ptr<Observer> observer) {
        observers_.push_back(observer);
    }

    void Notify(const std::string& message) {
        for (const auto& observer : observers_) {
            observer->Update(message);
        }
    }
};

// Strategy Pattern
class ValidationStrategy {
public:
    virtual bool Validate(const std::string& data) = 0;
    virtual ~ValidationStrategy() = default;
};

class StandardValidation : public ValidationStrategy {
public:
    bool Validate(const std::string& data) override {
        std::cout << "Performing standard validation on: " << data << "\n";
        return true;
    }
};

class StrictValidation : public ValidationStrategy {
public:
    bool Validate(const std::string& data) override {
        std::cout << "Performing strict validation on: " << data << "\n";
        return true;
    }
};

class Validator {
private:
    std::unique_ptr<ValidationStrategy> strategy_;

public:
    void SetStrategy(std::unique_ptr<ValidationStrategy> strategy) {
        strategy_ = std::move(strategy);
    }

    bool ValidateData(const std::string& data) {
        if (strategy_) {
            return strategy_->Validate(data);
        }
        return false;
    }
};

// Chain of Responsibility Pattern
class Handler {
protected:
    std::unique_ptr<Handler> next_handler_;

public:
    virtual bool Handle(const std::string& request) = 0;
    virtual ~Handler() = default;

    void SetNext(std::unique_ptr<Handler> handler) {
        next_handler_ = std::move(handler);
    }

    bool HandleNext(const std::string& request) {
        if (next_handler_) {
            return next_handler_->Handle(request);
        }
        return true;
    }
};

class SignatureHandler : public Handler {
public:
    bool Handle(const std::string& request) override {
        std::cout << "Checking signature: " << request << "\n";
        return HandleNext(request);
    }
};

class AmountHandler : public Handler {
public:
    bool Handle(const std::string& request) override {
        std::cout << "Checking amount: " << request << "\n";
        return HandleNext(request);
    }
};

int main() {
    // Demonstrate Observer
    auto subject = std::make_unique<Subject>();
    auto blockObserver = std::make_shared<BlockObserver>();
    auto txObserver = std::make_shared<TxObserver>();
    
    subject->Attach(blockObserver);
    subject->Attach(txObserver);
    subject->Notify("New block mined!");

    // Demonstrate Strategy
    auto validator = std::make_unique<Validator>();
    
    validator->SetStrategy(std::make_unique<StandardValidation>());
    validator->ValidateData("transaction1");
    
    validator->SetStrategy(std::make_unique<StrictValidation>());
    validator->ValidateData("transaction2");

    // Demonstrate Chain of Responsibility
    auto signatureHandler = std::make_unique<SignatureHandler>();
    auto amountHandler = std::make_unique<AmountHandler>();
    
    signatureHandler->SetNext(std::move(amountHandler));
    signatureHandler->Handle("transaction3");

    return 0;
}
```

### Demo 5: Thread Safety Patterns
```cpp
// demo5_thread_safety.cpp
#include <iostream>
#include <mutex>
#include <thread>
#include <vector>
#include <queue>
#include <condition_variable>

// Thread-Safe Singleton Pattern
class ThreadSafeSingleton {
private:
    static std::mutex mutex_;
    static std::unique_ptr<ThreadSafeSingleton> instance_;
    
    ThreadSafeSingleton() = default;

public:
    static ThreadSafeSingleton* GetInstance() {
        std::lock_guard<std::mutex> lock(mutex_);
        if (instance_ == nullptr) {
            instance_ = std::unique_ptr<ThreadSafeSingleton>(
                new ThreadSafeSingleton());
        }
        return instance_.get();
    }

    void DoWork() {
        std::cout << "Singleton doing work\n";
    }
};
std::mutex ThreadSafeSingleton::mutex_;
std::unique_ptr<ThreadSafeSingleton> ThreadSafeSingleton::instance_;

// Producer-Consumer Pattern
template<typename T>
class ThreadSafeQueue {
private:
    std::queue<T> queue_;
    mutable std::mutex mutex_;
    std::condition_variable not_empty_;
    std::condition_variable not_full_;
    size_t capacity_;

public:
    explicit ThreadSafeQueue(size_t capacity) : capacity_(capacity) {}

    void Push(T value) {
        std::unique_lock<std::mutex> lock(mutex_);
        not_full_.wait(lock, [this]() { 
            return queue_.size() < capacity_; 
        });
        queue_.push(std::move(value));
        lock.unlock();
        not_empty_.notify_one();
    }

    T Pop() {
        std::unique_lock<std::mutex> lock(mutex_);
        not_empty_.wait(lock, [this]() { 
            return !queue_.empty(); 
        });
        T value = std::move(queue_.front());
        queue_.pop();
        lock.unlock();
        not_full_.notify_one();
        return value;
    }
};

// Active Object Pattern
class ActiveObject {
private:
    ThreadSafeQueue<std::function<void()>> queue_;
    std::thread worker_;
    bool running_ = false;

public:
    ActiveObject() : queue_(100) {
        running_ = true;
        worker_ = std::thread([this]() {
            while (running_) {
                auto task = queue_.Pop();
                task();
            }
        });
    }

    ~ActiveObject() {
        running_ = false;
        queue_.Push([]{}); // Push empty task to unblock queue
        if (worker_.joinable()) {
            worker_.join();
        }
    }

    template<typename F>
    void Enqueue(F&& task) {
        queue_.Push(std::forward<F>(task));
    }
};

void DemonstrateThreadSafety() {
    // Demonstrate Thread-Safe Singleton
    std::vector<std::thread> threads;
    for (int i = 0; i < 5; ++i) {
        threads.emplace_back([]() {
            ThreadSafeSingleton::GetInstance()->DoWork();
        });
    }
    for (auto& thread : threads) {
        thread.join();
    }

    // Demonstrate Producer-Consumer
    ThreadSafeQueue<int> queue(5);
    std::thread producer([&queue]() {
        for (int i = 0; i < 10; ++i) {
            queue.Push(i);
            std::cout << "Produced: " << i << "\n";
        }
    });

    std::thread consumer([&queue]() {
        for (int i = 0; i < 10; ++i) {
            int value = queue.Pop();
            std::cout << "Consumed: " << value << "\n";
        }
    });

    producer.join();
    consumer.join();

    // Demonstrate Active Object
    ActiveObject active;
    for (int i = 0; i < 5; ++i) {
        active.Enqueue([i]() {
            std::cout << "Task " << i << " executed\n";
        });
    }
    std::this_thread::sleep_for(std::chrono::seconds(1));
}

int main() {
    DemonstrateThreadSafety();
    return 0;
}
```

## Running the Demonstrations

1. Make sure you have a C++ compiler with C++14 support:
```bash
g++ --version
```

2. Create a directory for demos:
```bash
mkdir design_pattern_demos
cd design_pattern_demos
```

3. Compile and run each demo:
```bash
# For demo1_solid.cpp
g++ -std=c++14 demo1_solid.cpp -o demo1
./demo1

# For demo2_creational.cpp
g++ -std=c++14 demo2_creational.cpp -o demo2
./demo2

# For demo3_structural.cpp
g++ -std=c++14 demo3_structural.cpp -o demo3
./demo3

# For demo4_behavioral.cpp
g++ -std=c++14 demo4_behavioral.cpp -o demo4
./demo4

# For demo5_thread_safety.cpp
g++ -std=c++14 demo5_thread_safety.cpp -o demo5 -pthread
./demo5
```

## Tips for Learning
1. Study each pattern's intent and structure
2. Experiment with the examples
3. Try combining patterns
4. Look for patterns in Bitcoin Core
5. Practice implementing variations

## Next Steps
1. Study pattern implementations in Bitcoin Core
2. Create your own pattern examples
3. Combine multiple patterns
4. Move on to the exercises
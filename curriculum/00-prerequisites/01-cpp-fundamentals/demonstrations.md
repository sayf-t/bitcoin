# C++ Fundamentals Demonstrations

## Interactive Learning Path

### Demo 1: Your First Bitcoin-Related Program
```cpp
// demo1_basics.cpp
#include <iostream>
#include <string>
#include <vector>

// Let's create a simple transaction structure
struct Transaction {
    std::string from;
    std::string to;
    double amount;
};

int main() {
    // Create your first transaction
    Transaction tx;
    tx.from = "Alice";
    tx.to = "Bob";
    tx.amount = 0.5;

    // Print transaction details
    std::cout << "Transaction Details:\n";
    std::cout << "From: " << tx.from << "\n";
    std::cout << "To: " << tx.to << "\n";
    std::cout << "Amount: " << tx.amount << " BTC\n";

    return 0;
}

/* To compile and run:
g++ demo1_basics.cpp -o demo1
./demo1
*/
```

### Demo 2: Working with Smart Pointers
```cpp
// demo2_smart_pointers.cpp
#include <iostream>
#include <memory>
#include <vector>

class Wallet {
public:
    Wallet(std::string owner) : owner_(owner), balance_(0.0) {
        std::cout << "Creating wallet for " << owner << "\n";
    }
    
    ~Wallet() {
        std::cout << "Destroying wallet of " << owner_ << "\n";
    }
    
    void deposit(double amount) {
        balance_ += amount;
        std::cout << "Deposited " << amount << " BTC\n";
    }
    
    double getBalance() const { return balance_; }
    
private:
    std::string owner_;
    double balance_;
};

int main() {
    // Create a wallet using unique_ptr (single owner)
    std::unique_ptr<Wallet> aliceWallet = 
        std::make_unique<Wallet>("Alice");
    
    // Use the wallet
    aliceWallet->deposit(1.0);
    std::cout << "Balance: " << aliceWallet->getBalance() << " BTC\n";
    
    // No need to delete - automatically handled!
    return 0;
}

/* To compile and run:
g++ -std=c++14 demo2_smart_pointers.cpp -o demo2
./demo2
*/
```

### Demo 3: Modern C++ Features
```cpp
// demo3_modern.cpp
#include <iostream>
#include <vector>
#include <algorithm>

// A simple block class
class Block {
public:
    Block(int height, std::string hash) 
        : height_(height), hash_(std::move(hash)) {}
    
    int height() const { return height_; }
    const std::string& hash() const { return hash_; }
    
private:
    int height_;
    std::string hash_;
};

int main() {
    // Create a vector of blocks using list initialization
    std::vector<Block> blockchain{
        {1, "hash1"},
        {2, "hash2"},
        {3, "hash3"}
    };
    
    // Use auto and range-based for loop
    std::cout << "Blockchain contents:\n";
    for (const auto& block : blockchain) {
        std::cout << "Height: " << block.height() 
                 << ", Hash: " << block.hash() << "\n";
    }
    
    // Use lambda function to find a block
    auto findBlock = [](const Block& block) {
        return block.height() == 2;
    };
    
    auto it = std::find_if(blockchain.begin(), 
                          blockchain.end(), 
                          findBlock);
                          
    if (it != blockchain.end()) {
        std::cout << "\nFound block with hash: " 
                 << it->hash() << "\n";
    }
    
    return 0;
}

/* To compile and run:
g++ -std=c++14 demo3_modern.cpp -o demo3
./demo3
*/
```

### Demo 4: Thread Safety and Synchronization
```cpp
// demo4_threads.cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>
#include <chrono>

class SafeCounter {
private:
    int value_ = 0;
    std::mutex mutex_;

public:
    void increment() {
        std::lock_guard<std::mutex> lock(mutex_);
        ++value_;
        std::cout << "Counter increased to: " << value_ << "\n";
    }
    
    int getValue() const {
        std::lock_guard<std::mutex> lock(mutex_);
        return value_;
    }
};

void worker(SafeCounter& counter) {
    for (int i = 0; i < 3; ++i) {
        std::this_thread::sleep_for(
            std::chrono::milliseconds(100)
        );
        counter.increment();
    }
}

int main() {
    SafeCounter counter;
    
    // Create multiple threads
    std::vector<std::thread> threads;
    for (int i = 0; i < 3; ++i) {
        threads.emplace_back(worker, std::ref(counter));
    }
    
    // Wait for all threads to finish
    for (auto& thread : threads) {
        thread.join();
    }
    
    std::cout << "Final count: " << counter.getValue() << "\n";
    return 0;
}

/* To compile and run:
g++ -std=c++14 demo4_threads.cpp -o demo4 -pthread
./demo4
*/
```

### Demo 5: Error Handling
```cpp
// demo5_errors.cpp
#include <iostream>
#include <stdexcept>
#include <string>

class TransactionError : public std::runtime_error {
public:
    explicit TransactionError(const std::string& msg) 
        : std::runtime_error(msg) {}
};

class Account {
public:
    Account(double balance) : balance_(balance) {
        if (balance < 0) {
            throw TransactionError(
                "Cannot create account with negative balance"
            );
        }
    }
    
    void send(double amount) {
        if (amount <= 0) {
            throw TransactionError("Amount must be positive");
        }
        if (amount > balance_) {
            throw TransactionError("Insufficient funds");
        }
        balance_ -= amount;
        std::cout << "Sent " << amount << " BTC\n";
    }
    
    double getBalance() const { return balance_; }
    
private:
    double balance_;
};

int main() {
    try {
        // Create account with initial balance
        Account account(1.0);
        
        // Try some transactions
        std::cout << "Initial balance: " 
                 << account.getBalance() << " BTC\n";
        
        account.send(0.5);
        std::cout << "Balance after sending: " 
                 << account.getBalance() << " BTC\n";
        
        // This should throw an error
        account.send(1.0);
    }
    catch (const TransactionError& e) {
        std::cerr << "Transaction error: " << e.what() << "\n";
    }
    catch (const std::exception& e) {
        std::cerr << "Unexpected error: " << e.what() << "\n";
    }
    
    return 0;
}

/* To compile and run:
g++ -std=c++14 demo5_errors.cpp -o demo5
./demo5
*/
```

## Running the Demonstrations

1. Make sure you have a C++ compiler installed:
```bash
g++ --version
```

2. Create a new directory for the demos:
```bash
mkdir cpp_demos
cd cpp_demos
```

3. Copy each demo into a separate .cpp file

4. Compile and run each demo following the instructions in the comments

## Expected Output Examples

### Demo 1 Output:
```
Transaction Details:
From: Alice
To: Bob
Amount: 0.5 BTC
```

### Demo 2 Output:
```
Creating wallet for Alice
Deposited 1.0 BTC
Balance: 1.0 BTC
Destroying wallet of Alice
```

### Demo 3 Output:
```
Blockchain contents:
Height: 1, Hash: hash1
Height: 2, Hash: hash2
Height: 3, Hash: hash3

Found block with hash: hash2
```

### Demo 4 Output:
```
Counter increased to: 1
Counter increased to: 2
Counter increased to: 3
Counter increased to: 4
Counter increased to: 5
Counter increased to: 6
Counter increased to: 7
Counter increased to: 8
Counter increased to: 9
Final count: 9
```

### Demo 5 Output:
```
Initial balance: 1 BTC
Sent 0.5 BTC
Balance after sending: 0.5 BTC
Transaction error: Insufficient funds
```

## Tips for Learning
1. Type out the code yourself instead of copying/pasting
2. Experiment with modifying the examples
3. Try to predict the output before running
4. Use debugger to step through code
5. Read compiler error messages carefully

## Next Steps
1. Try modifying the examples
2. Combine concepts from different demos
3. Create your own small projects
4. Move on to the exercises
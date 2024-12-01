# Bitcoin Core Integration Testing

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Design and implement integration tests
- Test component interactions
- Verify system-level behavior
- Debug integration issues
- Monitor system performance
- Validate end-to-end scenarios

## Prerequisites
- Understanding of unit testing (covered in 01-unit-testing.md)
- Knowledge of functional testing (covered in 02-functional-testing.md)
- Familiarity with Bitcoin Core architecture
- Experience with test frameworks

## 🌟 Real-World Analogy
Think of integration testing like testing a city's infrastructure:
- Components = City Services
- Interactions = Service Dependencies
- System Tests = City-wide Operations
- Performance = Infrastructure Efficiency
- Monitoring = City Management
- Recovery = Emergency Response

## Integration Test Framework

### 1. System Test Setup
```cpp
class SystemTestSetup : public TestingSetup {
    std::vector<std::unique_ptr<NodeContext>> nodes;
    std::vector<std::unique_ptr<ChainstateManager>> chainman;
    std::unique_ptr<CTxMemPool> mempool;
    std::unique_ptr<CConnman> connman;
    
public:
    SystemTestSetup() {
        // Initialize components
        for (int i = 0; i < 4; i++) {
            nodes.push_back(std::make_unique<NodeContext>());
            chainman.push_back(std::make_unique<ChainstateManager>());
        }
        
        mempool = std::make_unique<CTxMemPool>(CTxMemPool::Options{});
        connman = std::make_unique<CConnman>(0x1337, 0x1337);
        
        // Setup connections
        SetupSystemConnections();
    }
    
    void SetupSystemConnections() {
        // Connect components
        for (size_t i = 0; i < nodes.size(); i++) {
            nodes[i]->chainman = chainman[i].get();
            nodes[i]->mempool = mempool.get();
            nodes[i]->connman = connman.get();
        }
    }
};
```

### 2. Component Integration
```cpp
BOOST_FIXTURE_TEST_SUITE(integration_tests, SystemTestSetup)

BOOST_AUTO_TEST_CASE(test_component_interaction)
{
    // Test mempool to chainstate interaction
    CTransactionRef tx = CreateValidTransaction();
    
    // Add to mempool
    BOOST_CHECK(mempool->AcceptToMemoryPool(tx));
    
    // Create block with transaction
    CBlock block = CreateBlockWithTransaction(tx);
    
    // Process block
    BlockValidationState state;
    BOOST_CHECK(nodes[0]->chainman->ProcessNewBlock(block, state));
    
    // Verify transaction removed from mempool
    BOOST_CHECK(!mempool->exists(tx->GetHash()));
    
    // Verify transaction in chain
    BOOST_CHECK(nodes[0]->chainman->ActiveChain().Contains(tx->GetHash()));
}

BOOST_AUTO_TEST_SUITE_END()
```

## Test Implementation

### 1. Network Integration
```cpp
class NetworkIntegrationTest : public SystemTestSetup {
public:
    void TestNetworkPropagation() {
        // Create and broadcast transaction
        CTransactionRef tx = CreateValidTransaction();
        connman->BroadcastTransaction(tx);
        
        // Verify propagation to all nodes
        for (const auto& node : nodes) {
            BOOST_CHECK(node->mempool->exists(tx->GetHash()));
        }
        
        // Test block propagation
        CBlock block = CreateBlockWithTransaction(tx);
        nodes[0]->chainman->ProcessNewBlock(block);
        
        // Verify all nodes have the block
        for (const auto& node : nodes) {
            BOOST_CHECK(node->chainman->ActiveChain().Contains(block.GetHash()));
        }
    }
};
```

### 2. Storage Integration
```cpp
class StorageIntegrationTest : public SystemTestSetup {
public:
    void TestStorageInteraction() {
        // Test block storage
        CBlock block = CreateValidBlock();
        BlockValidationState state;
        
        // Process and store block
        BOOST_CHECK(nodes[0]->chainman->ProcessNewBlock(block, state));
        
        // Verify block storage
        BOOST_CHECK(nodes[0]->chainman->ActiveChain().Contains(block.GetHash()));
        
        // Test UTXO set updates
        CCoinsViewCache view(nodes[0]->chainman->ActiveChainstate().CoinsTip());
        for (const auto& tx : block.vtx) {
            for (const auto& out : tx->vout) {
                BOOST_CHECK(view.HaveCoin(COutPoint(tx->GetHash(), 0)));
            }
        }
    }
};
```

## Advanced Testing Techniques

### 1. System State Verification
```cpp
class SystemStateTest : public SystemTestSetup {
public:
    void VerifySystemConsistency() {
        // Verify chain consistency
        for (size_t i = 1; i < nodes.size(); i++) {
            BOOST_CHECK_EQUAL(
                nodes[0]->chainman->ActiveChain().Tip()->GetBlockHash(),
                nodes[i]->chainman->ActiveChain().Tip()->GetBlockHash()
            );
        }
        
        // Verify mempool consistency
        for (size_t i = 1; i < nodes.size(); i++) {
            BOOST_CHECK_EQUAL(
                nodes[0]->mempool->size(),
                nodes[i]->mempool->size()
            );
        }
        
        // Verify network state
        BOOST_CHECK(connman->GetNodeCount() == nodes.size());
    }
};
```

### 2. Recovery Testing
```cpp
class RecoveryTest : public SystemTestSetup {
public:
    void TestSystemRecovery() {
        // Simulate node failure
        nodes[1].reset();
        
        // Create and process transactions
        CTransactionRef tx = CreateValidTransaction();
        nodes[0]->mempool->AcceptToMemoryPool(tx);
        
        // Recover node
        nodes[1] = std::make_unique<NodeContext>();
        SetupSystemConnections();
        
        // Verify recovery
        BOOST_CHECK(nodes[1]->mempool->exists(tx->GetHash()));
        BOOST_CHECK_EQUAL(
            nodes[0]->chainman->ActiveChain().Tip()->GetBlockHash(),
            nodes[1]->chainman->ActiveChain().Tip()->GetBlockHash()
        );
    }
};
```

## 🛠️ Practice Project: Integration Test Framework

Let's build a comprehensive integration test framework.

### Project Structure
```bash
integration-tests/
├── framework/
│   ├── system_setup.h
│   ├── component_test.h
│   └── network_test.h
├── tests/
│   ├── network_integration.cpp
│   ├── storage_integration.cpp
│   └── system_integration.cpp
├── utils/
│   ├── test_generators.h
│   └── state_verification.h
└── README.md
```

### Implementation Example
```cpp
// framework/system_setup.h
class IntegrationTestFramework {
private:
    std::vector<std::unique_ptr<NodeContext>> nodes;
    std::unique_ptr<CTxMemPool> mempool;
    std::unique_ptr<CConnman> connman;
    
public:
    void SetupTestEnvironment() {
        // Initialize system components
        InitializeNodes();
        SetupConnections();
        VerifyInitialState();
    }
    
    void SimulateNetworkConditions() {
        // Create network scenarios
        CreatePartition();
        TestRecovery();
        VerifyConsistency();
    }
    
    void TestSystemInteractions() {
        // Test component interactions
        TestMemPoolChainInteraction();
        TestNetworkPropagation();
        TestStorageIntegration();
    }
};
```

## 🎯 Knowledge Check
1. How do you design integration tests?
2. What are key system-level interactions?
3. How do you verify component integration?
4. What are best practices for system testing?
5. How do you handle system recovery?

## 🔍 Common Issues and Solutions

### Issue 1: Component Dependencies
- Proper initialization order
- Resource management
- State synchronization
- Error propagation

### Issue 2: System State
- Consistency verification
- State recovery
- Resource cleanup
- Error handling

## 📚 Additional Resources
- [Bitcoin Core Integration Tests](https://github.com/bitcoin/bitcoin/tree/master/test/functional)
- [System Testing Guide](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md)
- [Component Integration](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#testing)
- [Test Framework Documentation](https://github.com/bitcoin/bitcoin/blob/master/test/README.md)

## Next Steps
1. Design system-level tests
2. Implement component integration
3. Create recovery scenarios
4. Build monitoring tools
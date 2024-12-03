# Bitcoin Core Testing Guide

## 🎯 Testing Philosophy

### Core Testing Principles
1. **Consensus Critical Testing**: Ensuring consensus rules are strictly enforced
2. **Security First Approach**: Prioritizing security-related test cases
3. **Comprehensive Coverage**: Testing all aspects of the codebase
4. **Performance Validation**: Ensuring performance standards are met

## 🔍 Test Categories

### 1. Unit Tests
Unit tests focus on testing individual components in isolation. Located in `src/test/`:

```cpp
BOOST_AUTO_TEST_CASE(transaction_validation_tests)
{
    // Setup
    BasicTestingSetup setup;
    
    // Create test transaction
    CMutableTransaction tx;
    tx.vin.resize(1);
    tx.vout.resize(1);
    tx.vin[0].prevout.hash = InsecureRand256();
    tx.vin[0].prevout.n = 0;
    tx.vout[0].nValue = 1 * COIN;
    
    // Test validation
    TxValidationState state;
    BOOST_CHECK(CheckTransaction(tx, state));
}
```

### 2. Functional Tests
Functional tests verify system behavior. Located in `test/functional/`:

```python
def test_rbf_mempool_limit(self):
    """Test replacement transaction handling with mempool limits."""
    
    # Create initial transaction
    tx1 = self.wallet.send_to_address(self.nodes[1].getnewaddress(), 1)
    
    # Create replacement with higher fee
    tx1_replacement = self.wallet.send_to_address(
        self.nodes[1].getnewaddress(), 
        1, 
        fee_rate=2*self.nodes[0].getmempoolinfo()['mempoolminfee']
    )
    
    # Verify replacement
    assert_equal(tx1_replacement in self.nodes[0].getrawmempool(), True)
    assert_equal(tx1 in self.nodes[0].getrawmempool(), False)
```

### 3. Integration Tests
Integration tests verify component interactions:

```cpp
BOOST_AUTO_TEST_CASE(test_chain_integration)
{
    // Setup chain with multiple blocks
    TestChain100Setup setup;
    
    // Test chain reorganization
    CBlock block = CreateBlock();
    BOOST_CHECK(ProcessNewBlock(block, ...));
    BOOST_CHECK_EQUAL(chainActive.Height(), 101);
    
    // Test UTXO set updates
    BOOST_CHECK(pcoinsTip->HaveCoin(COutPoint(tx.GetHash(), 0)));
}
```

### 4. Performance Tests
Performance tests ensure system efficiency:

```cpp
BOOST_AUTO_TEST_CASE(benchmark_block_validation)
{
    TestChain100Setup setup;
    
    // Measure block validation time
    auto start = std::chrono::high_resolution_clock::now();
    ProcessNewBlock(block, ...);
    auto end = std::chrono::high_resolution_clock::now();
    
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    BOOST_CHECK(duration.count() < MAX_VALIDATION_TIME);
}
```

## 🛠️ Testing Tools and Framework

### 1. Test Setup and Fixtures
```cpp
struct TestingSetup {
    TestingSetup() {
        // Initialize test environment
        SelectParams(CBaseChainParams::REGTEST);
        InitializeTestBlockchain();
    }
    
    ~TestingSetup() {
        // Cleanup
        CleanupTestBlockchain();
    }
};

BOOST_FIXTURE_TEST_SUITE(blockchain_tests, TestingSetup)
```

### 2. Test Utilities
```cpp
// Common test utilities
void CreateTestBlock(CBlock& block) {
    block.nVersion = 1;
    block.hashPrevBlock = chainActive.Tip()->GetBlockHash();
    block.nTime = chainActive.Tip()->GetBlockTime() + 1;
}

// Network test utilities
void ConnectNodes(NodeConnections& nodes) {
    nodes[0]->AddNode(nodes[1]);
    nodes[1]->AddNode(nodes[0]);
}
```

## 🔄 Test Development Workflow

### 1. Writing New Tests
1. Identify test requirements
2. Choose appropriate test category
3. Create test fixtures/setup
4. Implement test cases
5. Add assertions and verifications

### 2. Running Tests
```bash
# Run all tests
make check

# Run specific test suite
test/functional/feature_rbf.py

# Run with extended options
test/functional/test_runner.py --extended

# Run unit tests
src/test/test_bitcoin
```

### 3. Debugging Tests
```bash
# Enable debug logging
test/functional/test_runner.py --loglevel=DEBUG

# Use GDB with unit tests
gdb --args src/test/test_bitcoin

# Analyze test coverage
./configure --enable-lcov
make cov
```

## 📊 Test Coverage

### 1. Coverage Analysis
```bash
# Generate coverage report
./configure --enable-lcov
make
make cov

# Analyze specific components
lcov --extract coverage.info "*/src/validation.cpp" -o validation_cov.info
genhtml validation_cov.info -o validation_coverage
```

### 2. Coverage Best Practices
1. Aim for high coverage of critical paths
2. Test edge cases and error conditions
3. Include positive and negative test cases
4. Cover all code paths in consensus-critical code
5. Regular coverage analysis and improvement

## 🎯 Testing Best Practices

### 1. Test Organization
- Group related tests together
- Use clear, descriptive test names
- Maintain test independence
- Follow the Arrange-Act-Assert pattern

### 2. Test Quality
- Write deterministic tests
- Avoid test interdependencies
- Include proper cleanup
- Document test purposes and requirements

### 3. Common Patterns
```cpp
// Good: Independent test
BOOST_AUTO_TEST_CASE(test_transaction_validation)
{
    TestingSetup setup;  // Fresh setup for each test
    
    // Arrange
    CMutableTransaction tx = CreateTestTransaction();
    
    // Act
    bool result = CheckTransaction(tx, state);
    
    // Assert
    BOOST_CHECK(result);
}

// Bad: Dependent test
BOOST_AUTO_TEST_CASE(test_dependent_validation)
{
    // DON'T: Depends on previous test state
    BOOST_CHECK(previouslyCreatedTx != nullptr);
}
```

## 🔍 Advanced Testing Topics

### 1. Fuzzing Tests
```cpp
BOOST_AUTO_TEST_CASE(fuzz_transaction_validation)
{
    // Create random transactions
    for (int i = 0; i < 1000; i++) {
        CMutableTransaction tx = CreateRandomTransaction();
        TxValidationState state;
        CheckTransaction(tx, state);
    }
}
```

### 2. Property-Based Testing
```cpp
template<typename T>
void TestPropertyHolds(const T& input) {
    // Property: Transaction validation is consistent
    TxValidationState state1, state2;
    BOOST_CHECK_EQUAL(
        CheckTransaction(input, state1),
        CheckTransaction(input, state2)
    );
}
```

### 3. Network Tests
```python
def test_network_behavior(self):
    """Test P2P network behavior."""
    # Setup network with multiple nodes
    self.setup_network()
    
    # Test message propagation
    self.nodes[0].sendrawtransaction(tx_hex)
    self.sync_all()
    
    # Verify all nodes received the transaction
    for node in self.nodes:
        assert_equal(node.getrawmempool(), [tx_hash])
```

Remember: Testing is crucial for Bitcoin Core's reliability and security. Always aim for comprehensive test coverage and maintain high-quality test cases. 
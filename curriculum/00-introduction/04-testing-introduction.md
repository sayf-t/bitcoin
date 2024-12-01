# Introduction to Bitcoin Core Testing

## 🎯 Testing Philosophy

### Core Testing Principles
1. Consensus Critical Testing
2. Security First Approach
3. Comprehensive Coverage
4. Performance Validation

## 🔍 Real Bitcoin Core Test Examples

### 1. Unit Test Example
From [PR #27156](https://github.com/bitcoin/bitcoin/pull/27156):
```cpp
BOOST_AUTO_TEST_CASE(transaction_mempool_tests)
{
    TestingSetup setup;
    
    // Create a test transaction
    CMutableTransaction tx;
    tx.vin.resize(1);
    tx.vout.resize(1);
    tx.vin[0].prevout.hash = InsecureRand256();
    tx.vin[0].prevout.n = 0;
    tx.vout[0].nValue = 1 * COIN;
    
    // Test mempool acceptance
    BOOST_CHECK(AcceptToMemoryPool(tx));
    
    // Test double-spend rejection
    CMutableTransaction tx2 = tx;
    tx2.vout[0].nValue = 2 * COIN;
    BOOST_CHECK(!AcceptToMemoryPool(tx2));
}
```

### 2. Functional Test Example
From [test/functional/feature_rbf.py](https://github.com/bitcoin/bitcoin/blob/master/test/functional/feature_rbf.py):
```python
def test_rbf_mempool_limit(self):
    """Test replacement transaction handling with mempool limits."""
    
    # Create a chain of transactions
    tx1 = self.wallet.send_to_address(self.nodes[1].getnewaddress(), 1)
    tx2 = self.wallet.send_to_address(self.nodes[1].getnewaddress(), 1)
    
    # Replace with higher fee
    tx1_replacement = self.wallet.send_to_address(
        self.nodes[1].getnewaddress(), 
        1, 
        fee_rate=2*self.nodes[0].getmempoolinfo()['mempoolminfee']
    )
    
    # Check replacement
    assert_equal(tx1_replacement in self.nodes[0].getrawmempool(), True)
    assert_equal(tx1 in self.nodes[0].getrawmempool(), False)
```

## 🛠️ Test Categories

### 1. Unit Tests
```cpp
// Example: Testing transaction validation
BOOST_AUTO_TEST_CASE(test_transaction_validation)
{
    // Setup
    BasicTestingSetup setup;
    
    // Create invalid transaction
    CMutableTransaction tx;
    tx.vin.resize(1);
    tx.vout.resize(0);  // Invalid: no outputs
    
    // Test validation
    TxValidationState state;
    BOOST_CHECK(!CheckTransaction(tx, state));
    BOOST_CHECK_EQUAL(state.GetRejectReason(), "bad-txns-vout-empty");
}
```

### 2. Functional Tests
```python
# Example: Testing block validation
def test_block_validation(self):
    """Test block validation rules."""
    
    # Generate a valid block
    block_hash = self.nodes[0].generate(1)[0]
    block = self.nodes[0].getblock(block_hash)
    
    # Test block acceptance
    assert_equal(self.nodes[1].submitblock(block), None)
    
    # Test invalid block rejection
    invalid_block = create_invalid_block()
    assert_raises_rpc_error(-22, "Block validation failed",
                           self.nodes[0].submitblock, invalid_block)
```

### 3. Integration Tests
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

## 🔄 Common Testing Patterns

### 1. Setup and Teardown
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
```

### 2. Test Fixtures
```cpp
BOOST_FIXTURE_TEST_SUITE(blockchain_tests, TestingSetup)

BOOST_AUTO_TEST_CASE(test_block_creation)
{
    // Test code using TestingSetup fixture
}

BOOST_AUTO_TEST_SUITE_END()
```

## 🎯 Debugging Test Failures

### 1. Common Test Failures
```cpp
// Example: Timing-dependent test
BOOST_AUTO_TEST_CASE(test_peer_messaging)
{
    // Bad: Race condition possible
    SendMessage(peer, msg);
    BOOST_CHECK(received);  // May fail
    
    // Good: Wait for condition
    SendMessage(peer, msg);
    BOOST_CHECK(WaitForCondition(received, timeout));
}
```

### 2. Test Logging
```cpp
// Example: Enhanced test logging
BOOST_AUTO_TEST_CASE(test_transaction_relay)
{
    LogPrintf("Starting transaction relay test\n");
    
    // Create transaction
    auto tx = CreateTestTransaction();
    LogPrintf("Created transaction: %s\n", tx.GetHash().ToString());
    
    // Relay transaction
    RelayTransaction(tx);
    LogPrintf("Transaction relayed\n");
    
    // Verify propagation
    BOOST_CHECK(WaitForTxInMempool(tx.GetHash()));
}
```

## 📊 Test Coverage Analysis

### 1. Coverage Tools
```bash
# Generate coverage report
./configure --enable-lcov
make
make cov

# Analyze specific components
lcov --extract coverage.info "*/src/validation.cpp" -o validation_cov.info
genhtml validation_cov.info -o validation_coverage
```

### 2. Coverage Patterns
```cpp
// Example: Edge case coverage
BOOST_AUTO_TEST_CASE(test_transaction_edge_cases)
{
    // Test empty transaction
    CMutableTransaction empty_tx;
    BOOST_CHECK(!CheckTransaction(empty_tx, state));
    
    // Test maximum size transaction
    CMutableTransaction max_tx = CreateMaxSizeTransaction();
    BOOST_CHECK(CheckTransaction(max_tx, state));
    
    // Test dust threshold
    CMutableTransaction dust_tx = CreateDustTransaction();
    BOOST_CHECK(!CheckTransaction(dust_tx, state));
}
```

## 🔧 Writing Effective Tests

### 1. Test Structure
```cpp
BOOST_AUTO_TEST_CASE(test_block_validation)
{
    // 1. Setup
    TestingSetup setup;
    CBlock block = CreateTestBlock();
    
    // 2. Exercise
    bool result = ProcessNewBlock(block);
    
    // 3. Verify
    BOOST_CHECK(result);
    BOOST_CHECK(chainActive.Tip()->GetBlockHash() == block.GetHash());
    
    // 4. Cleanup handled by fixture
}
```

### 2. Test Isolation
```cpp
// Example: Isolated test environment
BOOST_AUTO_TEST_CASE(test_mempool_acceptance)
{
    // Create isolated test chain
    TestChain100Setup chain;
    
    // Create isolated mempool
    CTxMemPool mempool;
    
    // Run test with isolated components
    TestMempool(chain, mempool);
}
```

## 🎯 Best Practices

### 1. Test Naming
```cpp
// Good: Clear test names
BOOST_AUTO_TEST_CASE(test_mempool_rejects_double_spend)
BOOST_AUTO_TEST_CASE(test_block_connects_to_active_chain)
BOOST_AUTO_TEST_CASE(test_transaction_respects_sequence_locks)

// Bad: Unclear names
BOOST_AUTO_TEST_CASE(test1)
BOOST_AUTO_TEST_CASE(mempool_test)
BOOST_AUTO_TEST_CASE(check_blocks)
```

### 2. Test Independence
```cpp
// Good: Independent tests
BOOST_AUTO_TEST_CASE(test_transaction_validation)
{
    TestingSetup setup;  // Fresh setup for each test
    auto result = ValidateTransaction(CreateTestTransaction());
    BOOST_CHECK(result);
}

// Bad: Dependent tests
bool global_setup_done = false;
BOOST_AUTO_TEST_CASE(test_dependent)
{
    if (!global_setup_done) {
        SetupGlobalState();  // Don't do this
        global_setup_done = true;
    }
}
```

Remember: Testing in Bitcoin Core is critical for maintaining the security and reliability of the system. Always write comprehensive tests that cover both the happy path and edge cases.
``` 
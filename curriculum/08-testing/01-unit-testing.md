# Bitcoin Core Unit Testing

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Understand unit testing principles in Bitcoin Core
- Write effective unit tests using Boost.Test
- Implement test fixtures and suites
- Use mocking and test doubles
- Measure and improve test coverage

## Prerequisites
- Understanding of testing basics (covered in 00-introduction/04-testing-introduction.md)
- C++ programming experience
- Knowledge of Bitcoin Core architecture

## 🌟 Real-World Analogy
Think of unit testing like quality control in a manufacturing plant:
- Test Fixtures = Testing Equipment
- Test Cases = Quality Checks
- Test Suites = Inspection Stations
- Assertions = Measurement Tools
- Coverage = Inspection Completeness

## Unit Testing Framework

### 1. Basic Test Structure
```cpp
#include <boost/test/unit_test.hpp>
#include <test/util/setup_common.h>

BOOST_FIXTURE_TEST_SUITE(my_tests, BasicTestingSetup)

BOOST_AUTO_TEST_CASE(test_basic_functionality)
{
    // Test setup
    CTxDestination dest = DecodeDestination("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa");
    
    // Test execution
    bool isValid = IsValidDestination(dest);
    
    // Verification
    BOOST_CHECK(isValid);
}

BOOST_AUTO_TEST_SUITE_END()
```

### 2. Test Fixtures
```cpp
struct TransactionTestingSetup : public TestingSetup {
    CMutableTransaction tx;
    CTxMemPool pool;
    
    TransactionTestingSetup() {
        // Setup code
        tx.nVersion = 1;
        pool.clear();
    }
    
    ~TransactionTestingSetup() {
        // Cleanup code
        pool.clear();
    }
};

BOOST_FIXTURE_TEST_SUITE(transaction_tests, TransactionTestingSetup)

BOOST_AUTO_TEST_CASE(test_transaction_validation)
{
    // Use fixture members
    BOOST_CHECK_EQUAL(tx.nVersion, 1);
    BOOST_CHECK(pool.empty());
}

BOOST_AUTO_TEST_SUITE_END()
```

## Test Implementation

### 1. Assertion Types
```cpp
BOOST_AUTO_TEST_CASE(test_assertions)
{
    // Basic assertions
    BOOST_CHECK(condition);              // Continue on failure
    BOOST_REQUIRE(condition);            // Stop on failure
    
    // Equality checks
    BOOST_CHECK_EQUAL(value1, value2);   // Compare values
    BOOST_CHECK_NE(value1, value2);      // Check inequality
    
    // Floating point
    BOOST_CHECK_CLOSE(value1, value2, tolerance);
    
    // Exception testing
    BOOST_CHECK_THROW(function(), exception_type);
    BOOST_CHECK_NO_THROW(function());
    
    // Custom messages
    BOOST_CHECK_MESSAGE(condition, "Failure message");
}
```

### 2. Test Organization
```cpp
// Group related tests
BOOST_AUTO_TEST_SUITE(validation_tests)

// Simple tests
BOOST_AUTO_TEST_CASE(test_basic_validation)
{
    // Test code
}

// Parameterized tests
BOOST_DATA_TEST_CASE(
    test_multiple_inputs,
    boost::unit_test::data::make({1, 2, 3}) ^
    boost::unit_test::data::make({2, 4, 6}),
    input, expected)
{
    BOOST_CHECK_EQUAL(function(input), expected);
}

BOOST_AUTO_TEST_SUITE_END()
```

## Advanced Testing Techniques

### 1. Mocking
```cpp
// Mock class
class MockNode : public CNode {
public:
    MOCK_METHOD(bool, SendMessage, (const CNetMessage& msg), (override));
    MOCK_METHOD(void, CloseSocket, (), (override));
};

BOOST_AUTO_TEST_CASE(test_node_interaction)
{
    MockNode node;
    
    // Set expectations
    EXPECT_CALL(node, SendMessage(_))
        .Times(1)
        .WillOnce(Return(true));
    
    // Test code using mock
    bool result = node.SendMessage(message);
    BOOST_CHECK(result);
}
```

### 2. Test Doubles
```cpp
// Test stub
class TestNetworkManager : public INetworkManager {
private:
    bool m_isConnected{false};
    
public:
    bool Connect() override {
        m_isConnected = true;
        return true;
    }
    
    bool IsConnected() override {
        return m_isConnected;
    }
};

BOOST_AUTO_TEST_CASE(test_network_connection)
{
    TestNetworkManager network;
    BOOST_CHECK(!network.IsConnected());
    
    network.Connect();
    BOOST_CHECK(network.IsConnected());
}
```

## 🛠️ Practice Project: Test Framework Extension

Let's build extensions for the Bitcoin Core test framework.

### Project Structure
```bash
test-framework/
├── src/
│   ├── assertions/
│   │   ├── chain_assertions.h
│   │   ├── tx_assertions.h
│   │   └── network_assertions.h
│   ├── fixtures/
│   │   ├── chain_fixture.h
│   │   ├── mempool_fixture.h
│   │   └── network_fixture.h
│   └── utils/
│       ├── test_generator.h
│       └── test_helpers.h
├── tests/
└── README.md
```

### Implementation Example
```cpp
// src/assertions/tx_assertions.h
namespace test {

template<typename T>
void CheckTransactionValidity(const T& transaction) {
    BOOST_CHECK(!transaction.vin.empty());
    BOOST_CHECK(!transaction.vout.empty());
    BOOST_CHECK(transaction.nVersion >= 1);
    
    for (const auto& input : transaction.vin) {
        BOOST_CHECK(!input.prevout.IsNull());
    }
    
    for (const auto& output : transaction.vout) {
        BOOST_CHECK(output.nValue >= 0);
        BOOST_CHECK(!output.scriptPubKey.empty());
    }
}

} // namespace test
```

## 🎯 Knowledge Check
1. How do you structure a basic unit test?
2. What are the different types of assertions?
3. How do you use test fixtures?
4. What are the best practices for mocking?
5. How do you measure test coverage?

## 🔍 Common Issues and Solutions

### Issue 1: Test Isolation
- Use proper fixtures
- Clean state between tests
- Avoid global state
- Mock external dependencies

### Issue 2: Test Maintenance
- Follow naming conventions
- Document test purpose
- Keep tests focused
- Regular updates

## 📚 Additional Resources
- [Boost.Test Documentation](https://www.boost.org/doc/libs/1_76_0/libs/test/doc/html/index.html)
- [Bitcoin Core Test Guidelines](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#testing)
- [C++ Testing Best Practices](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#t-testing)
- [Test Coverage Tools](https://github.com/linux-test-project/lcov)

## Next Steps
1. Write comprehensive unit tests
2. Implement custom assertions
3. Create test fixtures
4. Measure test coverage
``` 
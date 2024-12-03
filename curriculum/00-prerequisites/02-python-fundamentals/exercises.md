# Python Fundamentals Exercises

## Project-Based Learning Path

### Project 1: Test Framework Development
Build a basic test framework for Bitcoin Core-style testing:

```python
# test_framework.py - Start with this template
class BitcoinTestFramework:
    """Your first test framework"""
    
    def __init__(self):
        # TODO: Initialize test environment
        # - Set up logging
        # - Create temporary directories
        # - Initialize variables
        pass
    
    def setup_chain(self):
        # TODO: Set up blockchain
        # - Create genesis block
        # - Set up test chain
        pass
    
    def run_test(self):
        # TODO: Implement test logic
        # - Create transactions
        # - Generate blocks
        # - Verify state
        pass

# Your implementation should support:
test = BitcoinTestFramework()
test.setup_chain()
test.run_test()
```

#### Steps:
1. Set up basic framework structure
2. Add logging functionality
3. Implement chain setup
4. Add test utilities
5. Create example tests

### Project 2: RPC Testing Suite
Create a comprehensive RPC testing system:

```python
# rpc_tests.py - Start with this template
class RPCTestSuite:
    """RPC test suite implementation"""
    
    def __init__(self):
        # TODO: Initialize RPC connection
        # TODO: Set up test environment
        pass
    
    def test_blockchain_info(self):
        # TODO: Test getblockchaininfo
        pass
    
    def test_network_info(self):
        # TODO: Test getnetworkinfo
        pass
    
    def test_wallet_functions(self):
        # TODO: Test wallet RPC calls
        pass

# Your implementation should support:
suite = RPCTestSuite()
suite.test_blockchain_info()
suite.test_network_info()
suite.test_wallet_functions()
```

#### Steps:
1. Implement RPC connection
2. Add test cases
3. Handle errors
4. Add assertions
5. Create test report

### Project 3: Test Node Controller
Build a test node management system:

```python
# node_controller.py - Start with this template
class NodeController:
    """Bitcoin node controller"""
    
    def __init__(self):
        # TODO: Initialize node settings
        # TODO: Set up data directory
        pass
    
    def start_node(self, extra_args=None):
        # TODO: Start bitcoin node
        pass
    
    def stop_node(self):
        # TODO: Stop bitcoin node
        pass
    
    def generate_blocks(self, count):
        # TODO: Generate blocks
        pass

# Your implementation should support:
controller = NodeController()
controller.start_node(["-regtest"])
controller.generate_blocks(101)
controller.stop_node()
```

#### Steps:
1. Implement node management
2. Add configuration handling
3. Implement block generation
4. Add error handling
5. Create cleanup routines

### Project 4: Test Data Generator
Create a system for generating test data:

```python
# test_data.py - Start with this template
class TestDataGenerator:
    """Test data generation system"""
    
    def __init__(self):
        # TODO: Initialize data structures
        pass
    
    def generate_transaction(self):
        # TODO: Generate test transaction
        pass
    
    def generate_block(self):
        # TODO: Generate test block
        pass
    
    def generate_chain(self, length):
        # TODO: Generate test chain
        pass

# Your implementation should support:
generator = TestDataGenerator()
tx = generator.generate_transaction()
block = generator.generate_block()
chain = generator.generate_chain(10)
```

#### Steps:
1. Implement data structures
2. Add randomization
3. Ensure data validity
4. Add persistence
5. Create verification tools

### Project 5: Test Result Analyzer
Build a system to analyze test results:

```python
# test_analyzer.py - Start with this template
class TestResultAnalyzer:
    """Test result analysis system"""
    
    def __init__(self):
        # TODO: Initialize analysis system
        pass
    
    def collect_results(self, test_run):
        # TODO: Collect test results
        pass
    
    def analyze_performance(self):
        # TODO: Analyze performance metrics
        pass
    
    def generate_report(self):
        # TODO: Generate test report
        pass

# Your implementation should support:
analyzer = TestResultAnalyzer()
analyzer.collect_results(test_run)
analyzer.analyze_performance()
analyzer.generate_report()
```

#### Steps:
1. Implement result collection
2. Add analysis tools
3. Create visualizations
4. Generate reports
5. Add recommendations

## Challenge Projects

### Challenge 1: Comprehensive Test Suite
Combine all previous projects into a complete testing system:
```python
class BitcoinTestSystem:
    # TODO: Integrate all components
    # - Test framework
    # - RPC testing
    # - Node management
    # - Data generation
    # - Result analysis
```

### Challenge 2: CI/CD Integration
Create a continuous integration testing system:
```python
class CITestRunner:
    # TODO: Implement CI/CD system
    # - Test scheduling
    # - Result reporting
    # - Notification system
    # - Performance tracking
```

## Submission Requirements

### For Each Project:
1. Working implementation
2. Unit tests
3. Documentation
4. Usage examples
5. Performance metrics

### Code Style:
1. Follow PEP 8
2. Add type hints
3. Include docstrings
4. Handle errors
5. Add logging

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
1. Python documentation
2. Bitcoin Core test framework
3. Stack Overflow
4. Project Discord

### Common Issues:
1. RPC connection problems
2. Node management
3. Test synchronization
4. Data persistence

## Next Steps

After completing these exercises:
1. Study Bitcoin Core tests
2. Contribute test improvements
3. Create new test cases
4. Help maintain test framework
``` 
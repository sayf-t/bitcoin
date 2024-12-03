# Python Fundamentals for Bitcoin Core Development

## Overview
This guide covers essential Python concepts needed for Bitcoin Core development, particularly for testing, automation, and tooling.

## Prerequisites
- Basic programming knowledge
- Command-line familiarity
- Understanding of testing concepts

## Core Concepts

### 1. Python Basics for Testing

#### Test Structure
```python
#!/usr/bin/env python3
# Example test file structure

import unittest
from decimal import Decimal
from test_framework.test_framework import BitcoinTestFramework
from test_framework.util import assert_equal

class ExampleTest(BitcoinTestFramework):
    def set_test_params(self):
        self.num_nodes = 2
        self.setup_clean_chain = True
    
    def setup_network(self):
        self.setup_nodes()
        self.connect_nodes(0, 1)
    
    def run_test(self):
        # Test implementation
        self.log.info("Running example test...")
        
        # Generate blocks
        self.nodes[0].generate(101)
        
        # Create transaction
        txid = self.nodes[0].sendtoaddress(self.nodes[1].getnewaddress(), 1.0)
        
        # Test assertions
        assert_equal(self.nodes[0].getbalance(), Decimal('49.0'))
```

#### Test Fixtures
```python
def setup_chain(self):
    """Setup blockchain with genesis block."""
    for i in range(self.num_nodes):
        initialize_datadir(self.options.tmpdir, i)

def setup_network(self):
    """Setup test network topology."""
    self.nodes = []
    for i in range(self.num_nodes):
        self.nodes.append(self.start_node(i))
    self.sync_all()
```

### 2. Test Framework Usage

#### RPC Interactions
```python
class WalletTest(BitcoinTestFramework):
    def test_wallet_functions(self):
        # Create new address
        address = self.nodes[0].getnewaddress()
        
        # Send transaction
        txid = self.nodes[0].sendtoaddress(address, 1.0)
        
        # Generate block
        blockhash = self.nodes[0].generate(1)[0]
        
        # Verify transaction
        tx = self.nodes[0].getrawtransaction(txid, True)
        assert_equal(tx['confirmations'], 1)
```

#### Network Management
```python
def setup_network(self):
    """Example network setup with specific configurations."""
    self.nodes = []
    # Start nodes with custom args
    args = [
        ['-txindex'],
        ['-acceptnonstdtxn=0'],
    ]
    
    for i in range(self.num_nodes):
        self.nodes.append(self.start_node(i, args[i]))
    
    # Create network topology
    connect_nodes(self.nodes[0], 1)
    self.sync_all()
```

### 3. Script Development

#### Utility Functions
```python
def create_transaction_chain(node, num_txs, address):
    """Create a chain of transactions."""
    txids = []
    for _ in range(num_txs):
        txid = node.sendtoaddress(address, 0.1)
        txids.append(txid)
        node.generate(1)
    return txids

def wait_for_mempool(node, txid, timeout=60):
    """Wait for transaction to appear in mempool."""
    import time
    start_time = time.time()
    while time.time() < start_time + timeout:
        if txid in node.getrawmempool():
            return True
        time.sleep(0.1)
    raise TimeoutError("Transaction not found in mempool")
```

### 4. Package Management

#### Requirements File
```python
# requirements.txt
pytest>=6.0.0
coverage>=5.0
flake8>=3.9.0
black>=21.0
```

#### Setup Script
```python
# setup.py
from setuptools import setup, find_packages

setup(
    name="bitcoin-test-framework",
    version="0.1",
    packages=find_packages(),
    install_requires=[
        'pytest>=6.0.0',
        'coverage>=5.0',
    ],
    entry_points={
        'console_scripts': [
            'run-tests=test_framework.runner:main',
        ],
    }
)
```

### 5. Testing Tools

#### Coverage Analysis
```python
# test_with_coverage.py
import coverage
import pytest

def run_tests_with_coverage():
    cov = coverage.Coverage()
    cov.start()
    
    # Run tests
    pytest.main(['test_wallet.py'])
    
    cov.stop()
    cov.save()
    
    # Generate report
    cov.report()
    cov.html_report()

if __name__ == '__main__':
    run_tests_with_coverage()
```

#### Logging and Debugging
```python
import logging

def setup_logging():
    """Configure test logging."""
    logging.basicConfig(
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        level=logging.DEBUG
    )
    logger = logging.getLogger('bitcoin_test')
    return logger

def debug_transaction(node, txid):
    """Debug transaction details."""
    logger = setup_logging()
    try:
        tx = node.getrawtransaction(txid, True)
        logger.debug(f"Transaction details: {tx}")
        return tx
    except Exception as e:
        logger.error(f"Error getting transaction: {e}")
        raise
```

## Exercises

### Exercise 1: Basic Test Implementation
Create a test for transaction validation:
```python
class TransactionValidationTest(BitcoinTestFramework):
    """
    TODO: Implement test cases for:
    1. Valid transaction creation
    2. Invalid input handling
    3. Fee calculation
    4. Transaction rejection
    """
```

### Exercise 2: Network Testing
Implement network synchronization tests:
```python
class NetworkSyncTest(BitcoinTestFramework):
    """
    TODO: Implement tests for:
    1. Node synchronization
    2. Block propagation
    3. Transaction relay
    4. Network partitioning
    """
```

### Exercise 3: Utility Development
Create useful testing utilities:
```python
class TestUtils:
    """
    TODO: Implement utilities for:
    1. Transaction generation
    2. Block creation
    3. Network simulation
    4. State verification
    """
```

## Best Practices

1. Test Organization
   - Group related tests
   - Use descriptive names
   - Maintain independence
   - Clean up resources

2. Error Handling
   - Use appropriate assertions
   - Handle exceptions properly
   - Log relevant information
   - Clean up on failure

3. Performance
   - Optimize test runtime
   - Minimize resource usage
   - Use appropriate timeouts
   - Parallel test execution

4. Code Quality
   - Follow PEP 8
   - Document code
   - Use type hints
   - Write clear assertions

## Common Pitfalls

1. Test Reliability
   - Flaky tests
   - Race conditions
   - Resource cleanup
   - Network timeouts

2. Test Independence
   - Shared state
   - Order dependencies
   - Resource conflicts
   - Network interference

3. Maintenance Issues
   - Poor documentation
   - Complex setup
   - Brittle tests
   - Missing error handling

## Additional Resources

1. Documentation
   - Bitcoin Core test framework docs
   - Python testing documentation
   - pytest documentation
   - Coverage.py guide

2. Tools
   - pytest
   - coverage.py
   - flake8
   - black

3. Learning Resources
   - Python Testing with pytest
   - Python Cookbook
   - Real Python tutorials

## Assessment

Complete these tasks to validate your understanding:
1. Create a comprehensive test suite
2. Implement custom test utilities
3. Add coverage reporting
4. Create network tests

## Next Steps
After mastering these concepts:
1. Study Bitcoin Core test suite
2. Contribute test improvements
3. Create new test cases
4. Help maintain test framework
``` 
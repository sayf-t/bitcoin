# Bitcoin Core Functional Testing

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Understand Bitcoin Core's functional test framework
- Write effective functional tests in Python
- Set up test networks and nodes
- Simulate complex network scenarios
- Debug test failures effectively
- Measure test coverage

## Prerequisites
- Understanding of testing basics (covered in 00-introduction/04-testing-introduction.md)
- Knowledge of unit testing (covered in 01-unit-testing.md)
- Python programming experience
- Basic Bitcoin protocol knowledge

## 🌟 Real-World Analogy
Think of functional testing like a flight simulator:
- Test Network = Simulated Airspace
- Bitcoin Nodes = Aircraft
- Network Conditions = Weather Conditions
- Test Scenarios = Flight Scenarios
- Test Results = Flight Performance
- Coverage = Flight Parameters

## Test Framework Components

### 1. Basic Test Structure
```python
#!/usr/bin/env python3
# Copyright (c) 2021-present The Bitcoin Core developers

"""Example test demonstrating basic functionality."""

from test_framework.test_framework import BitcoinTestFramework
from test_framework.util import assert_equal

class ExampleTest(BitcoinTestFramework):
    def set_test_params(self):
        """Setup test parameters."""
        self.num_nodes = 2
        self.setup_clean_chain = True
        
    def setup_network(self):
        """Setup test network topology."""
        self.setup_nodes()
        self.connect_nodes(0, 1)
        
    def run_test(self):
        """Test logic goes here."""
        # Generate some blocks
        self.nodes[0].generatetoaddress(101, self.nodes[0].getnewaddress())
        
        # Test wallet functionality
        address = self.nodes[1].getnewaddress()
        txid = self.nodes[0].sendtoaddress(address, 10.0)
        
        # Wait for transaction
        self.sync_all()
        
        # Verify results
        assert_equal(self.nodes[1].getbalance(), 10.0)
        
if __name__ == '__main__':
    ExampleTest().main()
```

### 2. Network Configuration
```python
def set_test_params(self):
    """Advanced network setup."""
    self.num_nodes = 4
    self.extra_args = [
        ["-txindex"],                    # Node 0: Transaction index
        ["-reindex", "-rescan"],         # Node 1: Reindexing
        ["-prune=550"],                  # Node 2: Pruned node
        ["-debug=mempool"]               # Node 3: Extra logging
    ]

def setup_network(self):
    """Complex network topology."""
    self.setup_nodes()
    
    # Create a ring topology
    for i in range(self.num_nodes):
        self.connect_nodes(i, (i + 1) % self.num_nodes)
    
    # Add some cross connections
    self.connect_nodes(0, 2)
    self.connect_nodes(1, 3)
```

## Test Implementation

### 1. Chain and Block Testing
```python
def test_chain_mechanics(self):
    """Test block chain mechanics."""
    node = self.nodes[0]
    
    # Generate initial chain
    address = node.getnewaddress()
    node.generatetoaddress(101, address)
    
    # Test chain tip
    tip = node.getbestblockhash()
    height = node.getblockcount()
    assert_equal(height, 101)
    
    # Create a fork
    node.invalidateblock(tip)
    assert_equal(node.getblockcount(), 100)
    
    # Reconsider the block
    node.reconsiderblock(tip)
    assert_equal(node.getblockcount(), 101)
```

### 2. Transaction Testing
```python
def test_transaction_mechanics(self):
    """Test transaction handling."""
    # Create a chain of transactions
    node0, node1 = self.nodes[0], self.nodes[1]
    
    # Fund node0
    node0.generatetoaddress(101, node0.getnewaddress())
    
    # Create raw transaction
    inputs = [{"txid": node0.listunspent()[0]["txid"], "vout": 0}]
    outputs = {node1.getnewaddress(): 10.0}
    rawtx = node0.createrawtransaction(inputs, outputs)
    
    # Sign and send
    signedtx = node0.signrawtransactionwithwallet(rawtx)
    txid = node0.sendrawtransaction(signedtx["hex"])
    
    # Test mempool acceptance
    assert txid in node0.getrawmempool()
    
    # Generate block and verify
    node0.generatetoaddress(1, node0.getnewaddress())
    self.sync_all()
    assert_equal(node1.getbalance(), 10.0)
```

## Advanced Testing Techniques

### 1. Network Simulation
```python
def test_network_scenarios(self):
    """Test network conditions."""
    # Partition network
    self.disconnect_nodes(1, 2)
    
    # Generate competing chains
    self.nodes[0].generatetoaddress(5, self.nodes[0].getnewaddress())
    self.nodes[2].generatetoaddress(3, self.nodes[2].getnewaddress())
    
    # Verify partition
    assert_equal(self.nodes[0].getbestblockhash(), self.nodes[1].getbestblockhash())
    assert_equal(self.nodes[2].getbestblockhash(), self.nodes[3].getbestblockhash())
    assert self.nodes[0].getbestblockhash() != self.nodes[2].getbestblockhash()
    
    # Reconnect and verify sync
    self.connect_nodes(1, 2)
    self.sync_all()
    
    # Verify chain resolution
    assert_equal(self.nodes[0].getbestblockhash(), self.nodes[2].getbestblockhash())
```

### 2. Failure Testing
```python
def test_failure_scenarios(self):
    """Test failure handling."""
    node = self.nodes[0]
    
    # Test invalid transaction
    assert_raises_rpc_error(-22, "TX decode failed",
        node.sendrawtransaction, "invalid_hex")
    
    # Test double spend
    address = node.getnewaddress()
    txid = node.sendtoaddress(address, 1.0)
    raw = node.createrawtransaction([{"txid": txid, "vout": 0}], {node.getnewaddress(): 0.9})
    signed1 = node.signrawtransactionwithwallet(raw)
    signed2 = node.signrawtransactionwithwallet(raw)
    
    # First one should succeed
    node.sendrawtransaction(signed1["hex"])
    # Second one should fail
    assert_raises_rpc_error(-26, "txn-mempool-conflict",
        node.sendrawtransaction, signed2["hex"])
```

## 🛠️ Practice Project: Advanced Test Framework

Let's build extensions for the functional test framework.

### Project Structure
```bash
functional-tests/
├── framework/
│   ├── test_node.py
│   ├── test_network.py
│   └── test_wallet.py
├── scenarios/
│   ├── network_partition.py
│   ├── chain_reorganization.py
│   └── transaction_relay.py
├── utils/
│   ├── test_generators.py
│   └── assertions.py
└── README.md
```

### Implementation Example
```python
# framework/test_network.py
class TestNetwork:
    def __init__(self, test_framework):
        self.framework = test_framework
        
    def create_partition(self, group1, group2):
        """Create network partition between node groups."""
        for n1 in group1:
            for n2 in group2:
                self.framework.disconnect_nodes(n1, n2)
    
    def heal_partition(self, group1, group2):
        """Heal network partition."""
        for n1 in group1:
            for n2 in group2:
                self.framework.connect_nodes(n1, n2)
                
    def verify_partition(self, group1, group2):
        """Verify network partition."""
        tip1 = self.framework.nodes[group1[0]].getbestblockhash()
        tip2 = self.framework.nodes[group2[0]].getbestblockhash()
        
        # Verify groups are internally synchronized
        for n in group1:
            assert_equal(self.framework.nodes[n].getbestblockhash(), tip1)
        for n in group2:
            assert_equal(self.framework.nodes[n].getbestblockhash(), tip2)
```

## 🎯 Knowledge Check
1. How do you structure a functional test?
2. What are the key components of the test framework?
3. How do you simulate network conditions?
4. What are best practices for test scenarios?
5. How do you debug test failures?

## 🔍 Common Issues and Solutions

### Issue 1: Test Flakiness
- Use proper sync calls
- Handle race conditions
- Implement proper waits
- Verify state transitions

### Issue 2: Network Issues
- Handle disconnections
- Manage timeouts
- Verify connectivity
- Monitor node state

## 📚 Additional Resources
- [Bitcoin Core Test Framework Documentation](https://github.com/bitcoin/bitcoin/blob/master/test/functional/README.md)
- [Python Testing Best Practices](https://docs.python-guide.org/writing/tests/)
- [Network Simulation Tools](https://github.com/bitcoin/bitcoin/blob/master/test/functional/test_framework/test_node.py)
- [Test Coverage Analysis](https://github.com/bitcoin/bitcoin/blob/master/test/README.md#test-coverage)

## Next Steps
1. Write comprehensive functional tests
2. Implement complex scenarios
3. Create custom test utilities
4. Measure scenario coverage
``` 
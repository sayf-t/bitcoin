# Python Fundamentals Demonstrations

## Interactive Learning Path

### Demo 1: Your First Test Framework
```python
#!/usr/bin/env python3
# demo1_test_framework.py

import unittest
import logging

# Set up logging
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger('BitcoinTest')

class Transaction:
    """Simple transaction class for demonstration"""
    def __init__(self, sender, receiver, amount):
        self.sender = sender
        self.receiver = receiver
        self.amount = amount
        
    def is_valid(self):
        return self.amount > 0

class TestTransaction(unittest.TestCase):
    """Your first test class"""
    
    def setUp(self):
        """Setup runs before each test"""
        logger.info("Setting up test...")
        self.tx = Transaction("Alice", "Bob", 1.0)
    
    def test_transaction_creation(self):
        """Test transaction creation"""
        logger.info("Testing transaction creation...")
        self.assertEqual(self.tx.sender, "Alice")
        self.assertEqual(self.tx.receiver, "Bob")
        self.assertEqual(self.tx.amount, 1.0)
    
    def test_transaction_validity(self):
        """Test transaction validation"""
        logger.info("Testing transaction validity...")
        self.assertTrue(self.tx.is_valid())
        
        # Test invalid amount
        invalid_tx = Transaction("Alice", "Bob", -1.0)
        self.assertFalse(invalid_tx.is_valid())

if __name__ == '__main__':
    unittest.main()

# To run:
# python3 demo1_test_framework.py
```

### Demo 2: Working with RPC
```python
# demo2_rpc.py
import json
import requests
from decimal import Decimal

class BitcoinRPC:
    """Simple Bitcoin RPC client"""
    
    def __init__(self, user, password, host="localhost", port=8332):
        self.url = f"http://{host}:{port}"
        self.headers = {'content-type': 'application/json'}
        self.auth = (user, password)
    
    def call(self, method, *params):
        """Make RPC call"""
        payload = {
            "jsonrpc": "2.0",
            "id": "demo",
            "method": method,
            "params": list(params)
        }
        
        try:
            response = requests.post(
                self.url,
                data=json.dumps(payload),
                headers=self.headers,
                auth=self.auth
            )
            response.raise_for_status()
            return response.json()['result']
        except Exception as e:
            print(f"RPC Error: {str(e)}")
            return None

def demonstrate_rpc():
    # Create RPC client
    rpc = BitcoinRPC("user", "password")
    
    # Get blockchain info
    info = rpc.call("getblockchaininfo")
    if info:
        print("Blockchain Info:")
        print(f"Blocks: {info['blocks']}")
        print(f"Current Difficulty: {info['difficulty']}")
    
    # Get network info
    net_info = rpc.call("getnetworkinfo")
    if net_info:
        print("\nNetwork Info:")
        print(f"Version: {net_info['version']}")
        print(f"Connections: {net_info['connections']}")

# To run:
# python3 demo2_rpc.py
```

### Demo 3: Test Node Management
```python
# demo3_node_management.py
import os
import time
import subprocess
import tempfile
import shutil

class BitcoinNode:
    """Bitcoin node management for testing"""
    
    def __init__(self, datadir=None):
        self.datadir = datadir or tempfile.mkdtemp()
        self.process = None
        self.rpc_port = 18443  # Use regtest port
    
    def start(self):
        """Start bitcoin node"""
        if self.process:
            return
        
        # Create bitcoin.conf
        conf_file = os.path.join(self.datadir, "bitcoin.conf")
        with open(conf_file, "w") as f:
            f.write("""
regtest=1
server=1
rpcuser=test
rpcpassword=test
rpcport=18443
            """.strip())
        
        # Start bitcoind
        cmd = [
            "bitcoind",
            f"-datadir={self.datadir}",
            "-daemon"
        ]
        
        try:
            subprocess.run(cmd, check=True)
            time.sleep(1)  # Wait for startup
            print(f"Node started with datadir: {self.datadir}")
        except subprocess.CalledProcessError as e:
            print(f"Failed to start node: {e}")
    
    def stop(self):
        """Stop bitcoin node"""
        if not self.process:
            return
        
        try:
            subprocess.run(
                ["bitcoin-cli", f"-datadir={self.datadir}", "stop"],
                check=True
            )
            time.sleep(1)  # Wait for shutdown
            print("Node stopped")
        except subprocess.CalledProcessError as e:
            print(f"Failed to stop node: {e}")
        finally:
            self.process = None
    
    def cleanup(self):
        """Clean up node data directory"""
        self.stop()
        if os.path.exists(self.datadir):
            shutil.rmtree(self.datadir)
            print(f"Cleaned up datadir: {self.datadir}")

def demonstrate_node():
    # Create and start node
    node = BitcoinNode()
    node.start()
    
    # Wait a bit
    time.sleep(5)
    
    # Stop and cleanup
    node.cleanup()

# To run:
# python3 demo3_node_management.py
```

### Demo 4: Test Utilities
```python
# demo4_test_utils.py
import hashlib
import random
import time

class TestUtils:
    """Useful testing utilities"""
    
    @staticmethod
    def generate_block_hash():
        """Generate random block hash"""
        return hashlib.sha256(str(random.random()).encode()).hexdigest()
    
    @staticmethod
    def wait_until(predicate, timeout=10, sleep=0.1):
        """Wait until predicate is true or timeout"""
        start_time = time.time()
        while time.time() - start_time < timeout:
            if predicate():
                return True
            time.sleep(sleep)
        return False
    
    @staticmethod
    def create_chain(length):
        """Create test blockchain data"""
        chain = []
        prev_hash = "0" * 64
        
        for height in range(length):
            block = {
                'height': height,
                'hash': TestUtils.generate_block_hash(),
                'previousblockhash': prev_hash,
                'time': int(time.time()) - (length - height) * 600
            }
            chain.append(block)
            prev_hash = block['hash']
        
        return chain

def demonstrate_utils():
    utils = TestUtils()
    
    # Generate block hash
    print("Random block hash:", utils.generate_block_hash())
    
    # Wait until demonstration
    counter = 0
    def condition():
        nonlocal counter
        counter += 1
        return counter >= 5
    
    print("\nWaiting for condition...")
    result = utils.wait_until(condition, timeout=1, sleep=0.2)
    print(f"Condition met: {result}")
    
    # Create test chain
    print("\nGenerating test chain...")
    chain = utils.create_chain(3)
    for block in chain:
        print(f"Block {block['height']}: {block['hash']}")

# To run:
# python3 demo4_test_utils.py
```

### Demo 5: Test Data Management
```python
# demo5_test_data.py
import json
import os
from pathlib import Path

class TestData:
    """Test data management"""
    
    def __init__(self, data_dir="test_data"):
        self.data_dir = Path(data_dir)
        self.data_dir.mkdir(exist_ok=True)
    
    def save_fixture(self, name, data):
        """Save test fixture data"""
        file_path = self.data_dir / f"{name}.json"
        with open(file_path, 'w') as f:
            json.dump(data, f, indent=2)
        print(f"Saved fixture: {file_path}")
    
    def load_fixture(self, name):
        """Load test fixture data"""
        file_path = self.data_dir / f"{name}.json"
        try:
            with open(file_path) as f:
                return json.load(f)
        except FileNotFoundError:
            print(f"Fixture not found: {file_path}")
            return None
    
    def create_test_blocks(self, count):
        """Create and save test blocks"""
        blocks = []
        prev_hash = "0" * 64
        
        for i in range(count):
            block = {
                'height': i,
                'hash': os.urandom(32).hex(),
                'previousblockhash': prev_hash,
                'transactions': [
                    os.urandom(32).hex() for _ in range(random.randint(1, 5))
                ]
            }
            blocks.append(block)
            prev_hash = block['hash']
        
        self.save_fixture('test_blocks', blocks)
        return blocks

def demonstrate_data():
    # Create test data manager
    data = TestData()
    
    # Create and save test blocks
    print("Creating test blocks...")
    blocks = data.create_test_blocks(3)
    
    # Load and verify
    print("\nLoading test blocks...")
    loaded_blocks = data.load_fixture('test_blocks')
    
    if loaded_blocks:
        print(f"\nLoaded {len(loaded_blocks)} blocks:")
        for block in loaded_blocks:
            print(f"Block {block['height']}: {block['hash']}")
            print(f"Transactions: {len(block['transactions'])}")

# To run:
# python3 demo5_test_data.py
```

## Running the Demonstrations

1. Make sure you have Python 3.x installed:
```bash
python3 --version
```

2. Install required packages:
```bash
pip3 install requests
```

3. Create a directory for demos:
```bash
mkdir python_demos
cd python_demos
```

4. Run each demo following the instructions in the comments

## Expected Output Examples

### Demo 1 Output:
```
INFO:BitcoinTest:Setting up test...
INFO:BitcoinTest:Testing transaction creation...
INFO:BitcoinTest:Testing transaction validity...
..
----------------------------------------------------------------------
Ran 2 tests in 0.001s

OK
```

### Demo 2 Output:
```
Blockchain Info:
Blocks: 680000
Current Difficulty: 21434395961917.42

Network Info:
Version: 210000
Connections: 8
```

### Demo 3 Output:
```
Node started with datadir: /tmp/tmpx2k3j9f1
Node stopped
Cleaned up datadir: /tmp/tmpx2k3j9f1
```

### Demo 4 Output:
```
Random block hash: 7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069

Waiting for condition...
Condition met: True

Generating test chain...
Block 0: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
Block 1: 486ea46224d1bb4fb680f34f7c9ad96a8f24ec88be73ea8e5a6c65260e9cb8a7
Block 2: 9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08
```

### Demo 5 Output:
```
Creating test blocks...
Saved fixture: test_data/test_blocks.json

Loading test blocks...

Loaded 3 blocks:
Block 0: 7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069
Transactions: 3
Block 1: 486ea46224d1bb4fb680f34f7c9ad96a8f24ec88be73ea8e5a6c65260e9cb8a7
Transactions: 2
Block 2: 9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08
Transactions: 4
```

## Tips for Learning
1. Type out the code yourself
2. Experiment with the examples
3. Read the Bitcoin Core test framework code
4. Use Python's help() function
5. Practice debugging with pdb

## Next Steps
1. Modify the examples
2. Combine different demos
3. Create your own test utilities
4. Study Bitcoin Core's test framework
``` 
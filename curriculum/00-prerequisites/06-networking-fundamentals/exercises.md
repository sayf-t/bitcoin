# Networking Fundamentals Exercises

## TCP/IP Protocol Stack

### Exercise 1: Basic Socket Programming
1. Create a simple chat application using TCP sockets:
   - Implement both server and client components
   - Support multiple concurrent clients
   - Handle client disconnections gracefully
   - Add basic command support (e.g., /quit, /list)

2. Implement the same chat application using UDP:
   - Handle packet loss
   - Implement basic reliability mechanisms
   - Compare performance with TCP version

### Exercise 2: Protocol Implementation
1. Design and implement a simple custom protocol:
   ```python
   class CustomProtocol:
       def __init__(self):
           pass
       
       def create_packet(self, command, payload):
           # Create packet with header and payload
           pass
       
       def parse_packet(self, data):
           # Parse received packet
           pass
       
       def validate_packet(self, packet):
           # Validate packet integrity
           pass
   ```
2. Include features like:
   - Message type identification
   - Length prefixing
   - Checksum validation
   - Version compatibility

## Network Protocol Analysis

### Exercise 1: Bitcoin Protocol Messages
1. Implement parsers for common Bitcoin protocol messages:
   ```python
   def parse_version_message(data):
       # Parse version message
       pass
   
   def parse_inv_message(data):
       # Parse inventory message
       pass
   
   def parse_addr_message(data):
       # Parse address message
       pass
   ```
2. Add validation for each message type
3. Create test cases with real Bitcoin network data

### Exercise 2: Network Traffic Analysis
1. Create a network traffic analyzer:
   - Capture Bitcoin network traffic
   - Identify message types
   - Track peer connections
   - Generate statistics
2. Implement filters for specific message types
3. Create a simple visualization of network activity

## Peer-to-Peer Networking

### Exercise 1: Node Network
1. Implement a basic P2P node network:
   ```python
   class P2PNode:
       def __init__(self):
           self.peers = set()
           self.data = {}
       
       def connect(self, peer):
           # Establish connection with peer
           pass
       
       def broadcast(self, message):
           # Broadcast message to all peers
           pass
       
       def handle_message(self, message, sender):
           # Process received message
           pass
   ```
2. Implement features:
   - Peer discovery
   - Message broadcasting
   - Data synchronization
   - Peer health checking

### Exercise 2: Distributed Hash Table
1. Implement a basic DHT:
   ```python
   class DHTNode:
       def __init__(self, node_id):
           self.node_id = node_id
           self.routing_table = {}
           self.data_store = {}
       
       def store(self, key, value):
           # Store data in DHT
           pass
       
       def lookup(self, key):
           # Find value in DHT
           pass
       
       def find_node(self, node_id):
           # Find node in network
           pass
   ```
2. Implement Kademlia-style routing
3. Add data replication
4. Handle node failures

## Network Security

### Exercise 1: Secure Communication
1. Implement a secure messaging system:
   - Use TLS/SSL for transport security
   - Implement certificate validation
   - Handle secure key exchange
   - Implement message encryption

2. Add features:
   - Perfect forward secrecy
   - Message authentication
   - Replay attack prevention
   - Session management

### Exercise 2: Network Authentication
1. Implement a custom authentication protocol:
   ```python
   class AuthProtocol:
       def __init__(self):
           self.private_key = None
           self.public_key = None
       
       def generate_challenge(self):
           # Generate authentication challenge
           pass
       
       def create_response(self, challenge):
           # Create challenge response
           pass
       
       def verify_response(self, challenge, response):
           # Verify challenge response
           pass
   ```
2. Implement features:
   - Challenge-response mechanism
   - Nonce usage
   - Timestamp validation
   - Rate limiting

## Network Performance

### Exercise 1: Connection Pool
1. Implement a robust connection pool:
   ```python
   class AdvancedConnectionPool:
       def __init__(self, min_size, max_size):
           self.min_size = min_size
           self.max_size = max_size
           self.pool = []
       
       def get_connection(self):
           # Get connection from pool
           pass
       
       def return_connection(self, conn):
           # Return connection to pool
           pass
       
       def maintain_pool(self):
           # Maintain pool size and health
           pass
   ```
2. Add features:
   - Dynamic pool sizing
   - Connection health checking
   - Automatic reconnection
   - Connection timeout handling

### Exercise 2: Load Balancer
1. Implement a complete load balancer:
   ```python
   class AdvancedLoadBalancer:
       def __init__(self):
           self.backends = []
           self.health_checks = {}
       
       def add_backend(self, backend):
           # Add backend server
           pass
       
       def remove_backend(self, backend):
           # Remove backend server
           pass
       
       def get_backend(self):
           # Get available backend
           pass
       
       def check_health(self, backend):
           # Check backend health
           pass
   ```
2. Implement multiple algorithms:
   - Round-robin
   - Least connections
   - Response time based
   - Weighted distribution

## Error Handling

### Exercise 1: Resilient Networking
1. Create a resilient network client:
   ```python
   class ResilientClient:
       def __init__(self):
           self.retries = 3
           self.timeout = 5
           self.backoff_factor = 1.5
       
       def connect(self, address):
           # Establish connection with retry
           pass
       
       def send_with_retry(self, data):
           # Send data with retry logic
           pass
       
       def receive_with_timeout(self):
           # Receive data with timeout
           pass
   ```
2. Implement features:
   - Exponential backoff
   - Circuit breaker pattern
   - Fallback mechanisms
   - Error reporting

### Exercise 2: Network Monitoring
1. Create a network monitoring system:
   ```python
   class NetworkMonitor:
       def __init__(self):
           self.targets = {}
           self.alerts = []
       
       def add_target(self, target, checks):
           # Add monitoring target
           pass
       
       def run_checks(self):
           # Run monitoring checks
           pass
       
       def generate_report(self):
           # Generate monitoring report
           pass
   ```
2. Implement features:
   - Availability monitoring
   - Latency tracking
   - Error rate monitoring
   - Alert generation

## Challenge Exercises

### Challenge 1: Mini Bitcoin Network
Create a simplified Bitcoin network implementation:
1. Implement basic node functionality
2. Handle peer discovery and connection
3. Implement message propagation
4. Add basic transaction and block handling
5. Implement simple consensus rules

### Challenge 2: Network Diagnostic Tool
Build a comprehensive network diagnostic tool:
1. Connection testing
2. Protocol analysis
3. Performance measurement
4. Security assessment
5. Report generation

### Challenge 3: Distributed System
Implement a distributed system with:
1. Leader election
2. Consensus mechanism
3. Data replication
4. Partition tolerance
5. Failure detection

For each exercise:
- Write clear documentation
- Include error handling
- Add comprehensive test cases
- Consider edge cases
- Follow Bitcoin Core coding style
- Use appropriate security practices

Remember that these exercises are for learning purposes only and should not be used in production environments without proper security review and hardening. 
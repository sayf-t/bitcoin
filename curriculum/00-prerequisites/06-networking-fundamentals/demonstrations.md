# Networking Fundamentals Demonstrations

## TCP/IP Protocol Stack

### Basic Socket Programming
```python
import socket

# TCP Server
def tcp_server():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(('localhost', 8333))  # Bitcoin's default port
    server.listen(1)
    
    print("Server listening on port 8333...")
    conn, addr = server.accept()
    print(f"Connection from {addr}")
    
    data = conn.recv(1024)
    print(f"Received: {data.decode()}")
    
    conn.send(b"Message received")
    conn.close()

# TCP Client
def tcp_client():
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect(('localhost', 8333))
    
    client.send(b"Hello, Bitcoin network!")
    response = client.recv(1024)
    print(f"Server response: {response.decode()}")
    
    client.close()
```

### UDP Communication
```python
import socket

# UDP Server
def udp_server():
    server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    server.bind(('localhost', 8333))
    
    print("UDP server listening...")
    while True:
        data, addr = server.recvfrom(1024)
        print(f"Received from {addr}: {data.decode()}")
        server.sendto(b"Message received", addr)

# UDP Client
def udp_client():
    client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    client.sendto(b"Hello UDP", ('localhost', 8333))
    
    data, _ = client.recvfrom(1024)
    print(f"Server response: {data.decode()}")
```

## Network Protocol Analysis

### Packet Capture
```python
from scapy.all import *

def capture_bitcoin_traffic():
    # Capture Bitcoin network traffic
    def packet_callback(pkt):
        if TCP in pkt and pkt[TCP].dport == 8333:
            print(f"Bitcoin packet: {pkt.summary()}")
    
    # Start capture
    sniff(filter="tcp port 8333", prn=packet_callback, count=10)
```

### Protocol Message Parsing
```python
import struct

def parse_bitcoin_message(data):
    """Parse a basic Bitcoin protocol message"""
    # Bitcoin message header structure
    header_format = '<4s12sI4s'
    header_size = struct.calcsize(header_format)
    
    # Unpack header
    magic, command, length, checksum = struct.unpack(header_format, data[:header_size])
    
    # Get payload
    payload = data[header_size:header_size + length]
    
    return {
        'magic': magic.hex(),
        'command': command.strip(b'\x00').decode(),
        'length': length,
        'checksum': checksum.hex(),
        'payload': payload
    }
```

## Peer-to-Peer Networking

### Node Discovery
```python
import random
import socket

class Node:
    def __init__(self, ip, port):
        self.ip = ip
        self.port = port
        self.peers = set()
    
    def add_peer(self, peer):
        self.peers.add(peer)
    
    def remove_peer(self, peer):
        self.peers.discard(peer)
    
    def get_random_peers(self, n):
        return random.sample(list(self.peers), min(n, len(self.peers)))

# Example usage
def demonstrate_node_discovery():
    # Create nodes
    nodes = [
        Node('192.168.1.1', 8333),
        Node('192.168.1.2', 8333),
        Node('192.168.1.3', 8333)
    ]
    
    # Connect nodes
    for i, node in enumerate(nodes):
        for other in nodes[:i] + nodes[i+1:]:
            node.add_peer(other)
    
    # Demonstrate peer discovery
    node = nodes[0]
    peers = node.get_random_peers(2)
    print(f"Random peers: {[(p.ip, p.port) for p in peers]}")
```

### Network Message Handling
```python
import asyncio

class P2PProtocol:
    def __init__(self):
        self.handlers = {}
    
    def register_handler(self, command, handler):
        self.handlers[command] = handler
    
    async def handle_message(self, message):
        command = message['command']
        if command in self.handlers:
            await self.handlers[command](message)
        else:
            print(f"No handler for command: {command}")

# Example usage
async def demonstrate_message_handling():
    protocol = P2PProtocol()
    
    # Register handlers
    async def version_handler(msg):
        print(f"Handling version message: {msg}")
    
    async def ping_handler(msg):
        print(f"Handling ping message: {msg}")
    
    protocol.register_handler('version', version_handler)
    protocol.register_handler('ping', ping_handler)
    
    # Handle messages
    await protocol.handle_message({'command': 'version', 'version': 70015})
    await protocol.handle_message({'command': 'ping', 'nonce': 123456})
```

## Network Security

### TLS/SSL Implementation
```python
import ssl
import socket

def create_secure_server():
    context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
    context.load_cert_chain(certfile="server.crt", keyfile="server.key")
    
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(('localhost', 8333))
    server.listen(1)
    
    while True:
        conn, addr = server.accept()
        ssl_conn = context.wrap_socket(conn, server_side=True)
        handle_connection(ssl_conn)

def create_secure_client():
    context = ssl.create_default_context()
    context.check_hostname = False
    context.verify_mode = ssl.CERT_NONE
    
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    ssl_client = context.wrap_socket(client, server_hostname='localhost')
    ssl_client.connect(('localhost', 8333))
    return ssl_client
```

### Network Authentication
```python
import hmac
import hashlib

class AuthenticatedConnection:
    def __init__(self, shared_secret):
        self.shared_secret = shared_secret.encode()
    
    def create_auth_token(self, message):
        """Create HMAC authentication token"""
        h = hmac.new(self.shared_secret, message.encode(), hashlib.sha256)
        return h.hexdigest()
    
    def verify_auth_token(self, message, token):
        """Verify HMAC authentication token"""
        expected_token = self.create_auth_token(message)
        return hmac.compare_digest(token, expected_token)

# Example usage
def demonstrate_auth():
    connection = AuthenticatedConnection("secret_key")
    
    # Create authenticated message
    message = "Important data"
    token = connection.create_auth_token(message)
    
    # Verify message
    is_valid = connection.verify_auth_token(message, token)
    print(f"Message authentication valid: {is_valid}")
```

## Network Performance

### Connection Pooling
```python
from queue import Queue
import threading

class ConnectionPool:
    def __init__(self, pool_size):
        self.pool = Queue(maxsize=pool_size)
        self.lock = threading.Lock()
        
        # Initialize pool
        for _ in range(pool_size):
            conn = self._create_connection()
            self.pool.put(conn)
    
    def _create_connection(self):
        return socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    def get_connection(self):
        return self.pool.get()
    
    def return_connection(self, conn):
        self.pool.put(conn)
    
    def close_all(self):
        while not self.pool.empty():
            conn = self.pool.get()
            conn.close()
```

### Load Balancing
```python
import random
from collections import deque

class LoadBalancer:
    def __init__(self):
        self.servers = []
        self.current = 0
    
    def add_server(self, server):
        self.servers.append(server)
    
    def round_robin(self):
        """Round-robin selection"""
        if not self.servers:
            return None
        
        server = self.servers[self.current]
        self.current = (self.current + 1) % len(self.servers)
        return server
    
    def random_select(self):
        """Random selection"""
        if not self.servers:
            return None
        return random.choice(self.servers)
    
    def least_connections(self, connections):
        """Least connections selection"""
        if not self.servers:
            return None
        
        return min(self.servers, key=lambda s: connections.get(s, 0))

# Example usage
def demonstrate_load_balancing():
    lb = LoadBalancer()
    
    # Add servers
    lb.add_server(('192.168.1.1', 8333))
    lb.add_server(('192.168.1.2', 8333))
    lb.add_server(('192.168.1.3', 8333))
    
    # Demonstrate different strategies
    print(f"Round-robin: {lb.round_robin()}")
    print(f"Random: {lb.random_select()}")
    
    connections = {
        ('192.168.1.1', 8333): 5,
        ('192.168.1.2', 8333): 2,
        ('192.168.1.3', 8333): 8
    }
    print(f"Least connections: {lb.least_connections(connections)}")
```

## Error Handling

### Network Resilience
```python
import time
from functools import wraps

def retry_on_failure(max_retries=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            retries = 0
            while retries < max_retries:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    retries += 1
                    if retries == max_retries:
                        raise
                    print(f"Attempt {retries} failed: {e}")
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

@retry_on_failure(max_retries=3)
def connect_to_peer(address):
    # Simulate connection attempt
    if random.random() < 0.5:
        raise ConnectionError("Connection failed")
    return f"Connected to {address}"
```

### Connection Timeouts
```python
import socket
import select

def connect_with_timeout(address, timeout=5):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setblocking(False)
    
    try:
        # Begin connection
        sock.connect_ex(address)
        
        # Wait for connection or timeout
        ready = select.select([], [sock], [], timeout)
        if not ready[1]:
            raise TimeoutError("Connection timed out")
        
        # Check if connection succeeded
        error = sock.getsockopt(socket.SOL_SOCKET, socket.SO_ERROR)
        if error:
            raise ConnectionError(f"Connection failed: {error}")
        
        return sock
    except Exception as e:
        sock.close()
        raise

# Example usage
try:
    sock = connect_with_timeout(('example.com', 80))
    print("Connection successful")
    sock.close()
except (TimeoutError, ConnectionError) as e:
    print(f"Connection failed: {e}")
```

These demonstrations cover essential networking concepts relevant to Bitcoin Core development. Each example includes practical code that you can run and modify to better understand the concepts. 
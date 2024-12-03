# Development Environment Setup Demonstrations

## 1. Basic Environment Setup

### Demo: Installing Core Dependencies
```bash
# Step-by-step demonstration of dependency installation
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install git
# Show successful installation checks
git --version
gcc --version
```

### Demo: Bitcoin Core Build Process
```bash
# Clone repository
git clone https://github.com/bitcoin/bitcoin.git
cd bitcoin

# Configure and build
./autogen.sh
./configure
make -j$(nproc)

# Verify build
src/bitcoind -version
```

## 2. Development Tools Setup

### Demo: Debugging Environment
```bash
# Install GDB
sudo apt-get install gdb

# Basic GDB usage with Bitcoin Core
gdb src/bitcoind
set args -regtest
run
```

### Demo: IDE Configuration
```bash
# VSCode setup for Bitcoin Core development
code --install-extension ms-vscode.cpptools
code --install-extension twxs.cmake
```

## 3. Testing Environment

### Demo: Test Framework Setup
```bash
# Install test dependencies
sudo apt-get install python3-pip
pip3 install --user pyzmq

# Run basic tests
make check
```

## 4. Network Configuration

### Demo: Regtest Network Setup
```bash
# Create regtest configuration
mkdir -p ~/.bitcoin
cat > ~/.bitcoin/bitcoin.conf << EOF
regtest=1
rpcuser=bitcoinrpc
rpcpassword=testpass
server=1
EOF

# Start regtest node
bitcoind -regtest -daemon
```

## 5. Common Workflows

### Demo: Build System Usage
```bash
# Clean build
make clean
make distclean

# Debug build
./configure CXXFLAGS="-g -O0"
make -j$(nproc)
```

### Demo: Development Cycle
```bash
# Make changes
git checkout -b feature-branch
# Edit files
git add .
git commit -m "Description of changes"
make check
``` 
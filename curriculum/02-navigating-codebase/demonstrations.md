# Code Navigation Demonstrations

## 1. Source Code Organization

### Demo: Exploring Directory Structure
```bash
# Show main directories
ls -l src/

# Show specific component structure
ls -l src/consensus/
ls -l src/wallet/
```

### Demo: Finding Key Files
```bash
# Find main entry points
find src/ -name "*.cpp" -exec grep -l "main(" {} \;

# Find key class definitions
find src/ -name "*.h" -exec grep -l "class CWallet" {} \;
```

## 2. Code Search Techniques

### Demo: Using Git Grep
```bash
# Basic function search
git grep -n "CreateNewBlock"

# Search with context
git grep -n -A 3 -B 3 "ProcessMessage"

# Search in specific files
git grep -n "CheckTransaction" "src/consensus/*.cpp"
```

### Demo: Using CTags
```bash
# Generate tags
ctags -R src/

# Navigate to definition
vim -t CWallet
```

## 3. Feature Tracing

### Demo: Transaction Flow
```bash
# Start at entry point
git grep -n "AcceptToMemoryPool" src/

# Follow validation
git grep -n "CheckTransaction" src/
git grep -n "CheckInputs" src/

# Find tests
git grep -n "TestAcceptToMemoryPool" test/
```

### Demo: Network Message Flow
```bash
# Find message handlers
git grep -n "ProcessMessage" src/net*

# Trace message creation
git grep -n "PushMessage" src/net*

# Find protocol definitions
git grep -n "MSG_" src/protocol*
```

## 4. Dependency Analysis

### Demo: Include Graph
```bash
# Show includes for a file
cpp -M src/wallet/wallet.cpp

# Find all includes of a header
git grep -n "#include.*wallet.h" src/
```

### Demo: Class Hierarchy
```bash
# Find base classes
git grep -n "class.*public" src/

# Find derived classes
git grep -n "class.*: public" src/
```

## 5. Test Coverage Analysis

### Demo: Finding Tests
```bash
# Unit tests
find test/unit/ -type f -name "*.cpp"

# Functional tests
find test/functional/ -type f -name "*.py"
```

### Demo: Test Execution Flow
```bash
# Run specific test
test/functional/feature_rbf.py

# Show test log
tail -f test_framework.log
```

## 6. Documentation Navigation

### Demo: Finding Documentation
```bash
# Find all markdown files
find . -name "*.md"

# Search in documentation
git grep -n "RPC" doc/

# Find developer notes
less doc/developer-notes.md
```

### Demo: API Documentation
```bash
# Find RPC documentation
git grep -n "RPC" doc/

# Find REST endpoints
git grep -n "REST" doc/
```

## 7. Build System Navigation

### Demo: Makefile Analysis
```bash
# Show build targets
make -qp | grep "^[a-zA-Z].*:"

# Find source file dependencies
make -n bitcoin-cli
```

### Demo: Configure Options
```bash
# Show all options
./configure --help

# Find feature flags
git grep -n "ENABLE_" configure.ac
```

## 8. Version Control Navigation

### Demo: Git History
```bash
# Show file history
git log --follow src/validation.cpp

# Show changes between versions
git diff v22.0..v23.0 src/wallet/
```

### Demo: Branch Navigation
```bash
# List all branches
git branch -a

# Show branch differences
git log --graph --oneline master..feature
``` 
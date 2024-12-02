# Module 0: Introduction - Practical Demonstrations

This guide is for instructors delivering hands-on demonstrations. Each demo follows our learning philosophy of combining theory with practical experience.

## 🔧 Development Environment Setup

### Demo 1: Environment Validation (15 minutes)
**Instructor Script:**
"Today we'll set up a Bitcoin Core development environment. I'll show you common pitfalls and best practices."

```bash
# First, let's check our system dependencies
$ ./contrib/install_db4.sh berkeley-db4-dir
# Explain: This script installs Berkeley DB, required for wallet functionality
# Point out: Notice the version compatibility warnings

# Now validate the development environment
$ ./autogen.sh
# Explain: This generates our build system configuration
# Highlight: Watch for missing dependency warnings

# Configure with debug symbols
$ ./configure --enable-debug
# Teaching moment: Explain configure flags and their implications
```

**Key Learning Points:**
- Dependency management in large C++ projects
- Build system configuration
- Debug vs. release builds

**Common Issues to Highlight:**
```bash
# Show common error
$ make
make: *** [src/bitcoind] Error 1
# Demonstrate troubleshooting:
$ make clean
$ ./configure --disable-wallet
$ make -j$(nproc)
```

### Demo 2: Debugger Setup (20 minutes)
**Instructor Script:**
"Let's set up our debugging environment. This is crucial for understanding Bitcoin Core's behavior."

```bash
# Start with GDB setup
$ gdb ./src/bitcoind
(gdb) b main
(gdb) run -regtest

# Show debug symbols
(gdb) info functions CBlock::*
# Explain: Class hierarchy and symbol loading
```

**Interactive Elements:**
```cpp
// Have students add breakpoints here
void ProcessMessage(CNode* pfrom, const std::string& strCommand)
{
    // Breakpoint opportunity: Message entry point
    LogPrint(BCLog::NET, "received: %s\n", strCommand);
}
```

## 🔍 Code Navigation

### Demo 1: Source Code Exploration (25 minutes)
**Instructor Script:**
"Understanding how to navigate Bitcoin Core's codebase is essential. Let's explore the key components."

```bash
# Show directory structure
$ tree -L 2 src/
# Explain each major directory's purpose

# Demonstrate code search
$ git grep "ProcessNewBlock"
# Explain: Code organization and call hierarchy
```

**Hands-on Exercise:**
Have students locate:
1. Transaction validation code
2. Network message handlers
3. Consensus rules implementation

### Demo 2: Build System Deep Dive (20 minutes)
**Instructor Script:**
"Bitcoin Core uses a complex build system. Let's understand how it works."

```bash
# Examine build configuration
$ cat configure.ac
# Highlight: Version management
# Highlight: Feature flags

# Show dependency graph
$ make -np | grep "gcc\|g++"
# Explain: Compilation order and dependencies
```

## 🔄 Version Control Workflow

### Demo 1: PR Creation (30 minutes)
**Instructor Script:**
"Let's walk through creating a Bitcoin Core pull request."

```bash
# Create feature branch
$ git checkout -b feat-add-logging

# Make changes
$ git diff
# Explain: Coding standards
# Show: Documentation requirements

# Prepare PR
$ git rebase -i master
# Demonstrate: Commit message formatting
```

**Review Process Simulation:**
```cpp
// Show a simple change
-    LogPrintf("Processing block %s\n", block.GetHash().ToString());
+    LogPrintf("Processing block %s at height %d\n", 
+        block.GetHash().ToString(), 
+        chainActive.Height() + 1);
```

## 🧪 Testing Framework

### Demo 1: Test Suite (25 minutes)
**Instructor Script:**
"Testing is critical in Bitcoin Core. Let's explore the testing framework."

```bash
# Run unit tests
$ src/test/test_bitcoin

# Show test structure
$ cat src/test/validation_tests.cpp
# Explain: Test categories
# Demonstrate: Writing new tests
```

**Interactive Test Writing:**
```cpp
BOOST_AUTO_TEST_CASE(block_validation_test)
{
    // Guide students through test case creation
    CBlock block;
    // ... test setup ...
    BOOST_CHECK(CheckBlock(block, state));
}
```

## 📝 Documentation

### Demo 1: Documentation Standards (15 minutes)
**Instructor Script:**
"Clear documentation is essential. Let's see Bitcoin Core's documentation practices."

```bash
# Generate documentation
$ doxygen doc/Doxyfile
# Show: Documentation structure
# Explain: Comment standards
```

## 🎯 Assessment Checkpoints
After each demonstration:
1. Have students replicate the demonstrated tasks
2. Review common mistakes
3. Discuss real-world applications

## 📋 Preparation Checklist
- [ ] Test all commands on a fresh system
- [ ] Prepare common error examples
- [ ] Set up multiple build configurations
- [ ] Create sample PR for demonstration
- [ ] Prepare test cases for student practice

Remember: Focus on hands-on learning and real-world scenarios. Encourage students to experiment and make mistakes in a controlled environment.

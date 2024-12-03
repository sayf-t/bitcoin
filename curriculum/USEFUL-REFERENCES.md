# Bitcoin Core Development: Useful References

This document maintains a curated list of references organized by module. Each section corresponds to a module in the curriculum.

## Module 0: Prerequisites
### C++ Programming
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)
- [Modern C++ Tutorial](https://github.com/changkun/modern-cpp-tutorial)
- [Effective Modern C++](https://www.oreilly.com/library/view/effective-modern-c/9781491908419/)
- [C++ Reference](https://en.cppreference.com/)

### Python Programming
- [Python Documentation](https://docs.python.org/3/)
- [Python Testing with pytest](https://pragprog.com/titles/bopytest/python-testing-with-pytest/)
- [Fluent Python](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/)
- [Python Type Hints](https://docs.python.org/3/library/typing.html)

### Design Patterns
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.oreilly.com/library/view/design-patterns-elements/0201633612/)
- [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/0596007124/)
- [Refactoring Guru](https://refactoring.guru/design-patterns)

### Development Tools
- [GDB Documentation](https://sourceware.org/gdb/documentation/)
- [Git Book](https://git-scm.com/book/en/v2)
- [Valgrind User Manual](https://valgrind.org/docs/manual/manual.html)
- [CMake Documentation](https://cmake.org/documentation/)

### Cryptography
- [Cryptography and Network Security](https://www.pearson.com/en-us/subject-catalog/p/cryptography-and-network-security-principles-and-practice/P200000003437)
- [Applied Cryptography](https://www.wiley.com/en-us/Applied+Cryptography%3A+Protocols%2C+Algorithms+and+Source+Code+in+C%2C+20th+Anniversary+Edition-p-9781119096726)
- [Practical Cryptography](https://www.wiley.com/en-us/Practical+Cryptography-p-9780471223573)
- [The Secp256k1 Library](https://github.com/bitcoin-core/secp256k1)

### Networking
- [TCP/IP Guide](http://www.tcpipguide.com/)
- [Computer Networking: A Top-Down Approach](https://www.pearson.com/en-us/subject-catalog/p/computer-networking/P200000003091)
- [Unix Network Programming](https://www.pearson.com/en-us/subject-catalog/p/unix-network-programming-volume-1-the-sockets-networking-api/P200000000864)
- [Network Programming with Python](https://www.oreilly.com/library/view/foundations-of-python/9781430230038/)

## Module 1: Development Environment
- [Bitcoin Core Build Instructions](https://github.com/bitcoin/bitcoin/blob/master/doc/build-unix.md)
- [Berkeley DB Installation Guide](https://github.com/bitcoin/bitcoin/blob/master/doc/build-unix.md#berkeley-db)
- [GDB Debugging Guide for Bitcoin Core](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#debug-tips)
- [Bitcoin Core Debug.log Guide](https://github.com/bitcoin/bitcoin/blob/master/doc/debug.md)

## Module 2: Codebase Navigation
- [Bitcoin Core GitHub Repository](https://github.com/bitcoin/bitcoin)
- [Bitcoin Core Developer Notes](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md)
- [Bitcoin Core Documentation](https://github.com/bitcoin/bitcoin/tree/master/doc)
- [Bitcoin Developer Guide](https://developer.bitcoin.org/devguide/)

## Module 3: Contribution Workflow
- [Bitcoin Core Contributing Guidelines](https://github.com/bitcoin/bitcoin/blob/master/CONTRIBUTING.md)
- [Bitcoin Core PR Review Process](https://github.com/bitcoin/bitcoin/blob/master/CONTRIBUTING.md#peer-review)
- [Bitcoin Core Review Club](https://bitcoincore.reviews/)
- Example PRs:
  - [PR #24957: Fix null pointer dereference](https://github.com/bitcoin/bitcoin/pull/24957)
  - [PR #25325: Performance improvement in string handling](https://github.com/bitcoin/bitcoin/pull/25325)

## Module 4: Testing
- [Bitcoin Core Test Framework Documentation](https://github.com/bitcoin/bitcoin/blob/master/test/README.md)
- [Bitcoin Core Functional Tests](https://github.com/bitcoin/bitcoin/blob/master/test/functional/README.md)
- [Bitcoin Core Unit Tests](https://github.com/bitcoin/bitcoin/tree/master/src/test)
- [Bitcoin Core Benchmarking Framework](https://github.com/bitcoin/bitcoin/blob/master/doc/benchmarking.md)

## Module 5: Network Architecture
- [Bitcoin Protocol Documentation](https://en.bitcoin.it/wiki/Protocol_documentation)
- [P2P Network](https://developer.bitcoin.org/devguide/p2p_network.html)
- [Network Message Reference](https://developer.bitcoin.org/reference/p2p_networking.html)

## Module 6: Blockchain and Transactions
- [Bitcoin Developer Reference](https://developer.bitcoin.org/reference/)
- [Bitcoin Script Reference](https://en.bitcoin.it/wiki/Script)
- [Bitcoin Improvement Proposals (BIPs)](https://github.com/bitcoin/bips)
- [Bitcoin Whitepaper](https://bitcoin.org/bitcoin.pdf)

## Module 7: Mempool Management
- [Memory Pool](https://developer.bitcoin.org/devguide/p2p_network.html#memory-pool)
- [Fee Estimation](https://developer.bitcoin.org/devguide/mining.html#fee-estimation)
- [Replace by Fee](https://developer.bitcoin.org/devguide/transactions.html#replace-by-fee)

## Module 8: Consensus
- [Consensus Rules](https://developer.bitcoin.org/devguide/block_chain.html#consensus-rules)
- [Block Validation](https://developer.bitcoin.org/devguide/block_chain.html#block-validation)
- [Chain Reorganization](https://developer.bitcoin.org/devguide/block_chain.html#reorganizing-the-block-chain)

## Module 9: Wallet Development
- [Bitcoin Core Wallet](https://developer.bitcoin.org/devguide/wallets.html)
- [HD Wallet](https://developer.bitcoin.org/devguide/wallets.html#hierarchical-deterministic-key-creation)
- [Bitcoin Core Cryptography Library (libsecp256k1)](https://github.com/bitcoin-core/secp256k1)

## Module 10: RPC and API
- [RPC Interface](https://developer.bitcoin.org/reference/rpc/index.html)
- [REST Interface](https://github.com/bitcoin/bitcoin/blob/master/doc/REST-interface.md)
- [ZMQ Documentation](https://github.com/bitcoin/bitcoin/blob/master/doc/zmq.md)

## Module 11: Open Source Contribution
- [Bitcoin Core Development Mailing List](https://lists.linuxfoundation.org/mailman/listinfo/bitcoin-dev)
- [Bitcoin Stack Exchange](https://bitcoin.stackexchange.com/)
- IRC Channels:
  - [#bitcoin-core-dev on Libera Chat](ircs://irc.libera.chat:6697/bitcoin-core-dev)
  - [#bitcoin-core-pr-reviews on Libera Chat](ircs://irc.libera.chat:6697/bitcoin-core-pr-reviews)

## Module 12: Advanced Topics
- [Academic Papers on Bitcoin](https://github.com/bitcoin/bitcoin/blob/master/doc/bibliography.md)
- [Bitcoin Core Security Process](https://github.com/bitcoin/bitcoin/blob/master/SECURITY.md)
- [Common Vulnerabilities in Cryptocurrency](https://github.com/slowmist/Cryptocurrency-Security-Audit-Guide)
- [Bitcoin Wiki Technical Pages](https://en.bitcoin.it/wiki/Technical_background_of_version_1_Bitcoin_addresses)

## General Development Resources
- [Bitcoin Core Coding Style Guide](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#coding-style)

## Contributing to This List
Please feel free to suggest additional resources by:
1. Creating a pull request
2. Opening an issue
3. Discussing in the course forum

When adding new resources, please:
- Verify the resource is up-to-date
- Add a brief description if necessary
- Place it in the appropriate module category
- Follow the existing format
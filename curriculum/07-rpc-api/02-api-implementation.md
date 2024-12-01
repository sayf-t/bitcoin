# Bitcoin Core API Implementation

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Implement secure and efficient Bitcoin Core APIs
- Design robust error handling and validation systems
- Build rate limiting and security mechanisms
- Create real-world API integrations
- Debug and monitor API performance

## Prerequisites
- Understanding of RPC interface (covered in 01-rpc-interface.md)
- Knowledge of C++ and object-oriented programming
- Familiarity with Bitcoin Core architecture
- Basic understanding of network protocols

## 🌟 Real-World Context
Think of Bitcoin Core's API implementation like a secure bank's transaction system:
- API Endpoints = Bank Services
- Authentication = Security Checkpoints
- Rate Limiting = Transaction Limits
- Error Handling = Safety Protocols
- Monitoring = Security Cameras

## 🎯 API Architecture Overview

### Core Components
```cpp
// Key API components from src/rpc/server.h
class JSONRPCRequest {
public:
    UniValue id;
    std::string strMethod;
    UniValue params;
    bool fHelp;
    std::string URI;
    std::string authUser;
};

class JSONRPCResponse {
public:
    UniValue result;
    UniValue error;
    UniValue id;
};
```

## 🔍 Real-World Integration Examples

### 1. Wallet RPC Implementation
```cpp
// Example: Advanced wallet RPC with error handling
static UniValue SendToAddress(const JSONRPCRequest& request)
{
    std::shared_ptr<CWallet> const wallet = GetWalletForJSONRPCRequest(request);
    CWallet* const pwallet = wallet.get();

    try {
        // Parameter validation
        RPCTypeCheck(request.params, {
            UniValue::VSTR,    // address
            UniValue::VNUM,    // amount
            UniValue::VSTR,    // comment
            UniValue::VSTR,    // comment_to
            UniValue::VBOOL,   // subtract fee
            UniValue::VBOOL,   // replaceable
            UniValue::VNUM     // conf_target
        });

        // Extract parameters
        CTxDestination dest = DecodeDestination(request.params[0].get_str());
        if (!IsValidDestination(dest)) {
            throw JSONRPCError(RPC_INVALID_ADDRESS_OR_KEY, 
                             "Invalid Bitcoin address");
        }

        // Amount validation
        CAmount amount = AmountFromValue(request.params[1]);
        if (amount <= 0) {
            throw JSONRPCError(RPC_TYPE_ERROR, "Invalid amount");
        }

        // Create and send transaction
        CTransactionRef tx = CreateTransaction(pwallet, dest, amount, request);
        return tx->GetHash().GetHex();

    } catch (const UniValue& objError) {
        // Handle JSON-RPC errors
        throw JSONRPCError(objError["code"].get_int(), 
                          objError["message"].get_str());
    } catch (const std::runtime_error& e) {
        // Handle runtime errors
        throw JSONRPCError(RPC_WALLET_ERROR, e.what());
    }
}
```

### 2. Block Information API
```cpp
// Example: Efficient block query implementation
static UniValue GetBlockInfo(const JSONRPCRequest& request)
{
    // Performance optimization: Use cache
    static std::map<uint256, UniValue> blockCache;
    static CCriticalSection cs_blockCache;

    try {
        // Parameter validation
        RPCTypeCheck(request.params, {UniValue::VSTR});
        uint256 hash = ParseHashV(request.params[0], "blockhash");

        // Check cache first
        {
            LOCK(cs_blockCache);
            auto it = blockCache.find(hash);
            if (it != blockCache.end()) {
                return it->second;
            }
        }

        // Get block
        CBlock block;
        CBlockIndex* pblockindex = LookupBlockIndex(hash);
        if (!pblockindex) {
            throw JSONRPCError(RPC_INVALID_PARAMETER, "Block not found");
        }

        // Build response
        UniValue result(UniValue::VOBJ);
        result.pushKV("hash", pblockindex->GetBlockHash().GetHex());
        result.pushKV("confirmations", 
                     chainActive.Height() - pblockindex->nHeight + 1);
        result.pushKV("size", (int)::GetSerializeSize(block, PROTOCOL_VERSION));
        result.pushKV("height", pblockindex->nHeight);
        result.pushKV("version", block.nVersion);
        result.pushKV("merkleroot", block.hashMerkleRoot.GetHex());
        result.pushKV("time", (int64_t)block.GetBlockTime());

        // Cache result
        {
            LOCK(cs_blockCache);
            blockCache[hash] = result;
        }

        return result;

    } catch (const UniValue& objError) {
        throw JSONRPCError(objError["code"].get_int(), 
                          objError["message"].get_str());
    }
}
```

## 🛠️ Security Best Practices

### 1. Authentication and Authorization
```cpp
// Secure authentication implementation
bool AuthenticateRequest(const JSONRPCRequest& request)
{
    // Check authentication headers
    if (!request.authUser.empty()) {
        // Validate credentials with constant-time comparison
        if (!TimingResistantEqual(request.authUser, GetAuthUser()) ||
            !TimingResistantEqual(request.authPassword, GetAuthPassword())) {
            LogPrintf("Authentication failed for user %s\n", request.authUser);
            return false;
        }
    }

    // Check authorization level
    if (!CheckPermission(request.authUser, request.strMethod)) {
        LogPrintf("Unauthorized access attempt to %s by %s\n",
                  request.strMethod, request.authUser);
        return false;
    }

    return true;
}

// Permission management
bool CheckPermission(const std::string& user, const std::string& method)
{
    static const std::map<std::string, std::set<std::string>> permissions = {
        {"admin", {"*"}},
        {"reader", {"getblock", "getblockchaininfo", "gettransaction"}},
        {"wallet", {"getbalance", "getnewaddress", "sendtoaddress"}}
    };

    const auto& userPerms = permissions.find(GetUserRole(user));
    if (userPerms == permissions.end()) {
        return false;
    }

    return userPerms->second.count("*") > 0 || 
           userPerms->second.count(method) > 0;
}
```

### 2. Input Validation
```cpp
// Comprehensive input validation
class InputValidator {
public:
    static bool ValidateAddress(const std::string& address) {
        CTxDestination dest = DecodeDestination(address);
        return IsValidDestination(dest);
    }

    static bool ValidateAmount(const CAmount& amount) {
        return amount > 0 && amount <= MAX_MONEY;
    }

    static bool ValidateRawTransaction(const std::string& hexTx) {
        if (!IsHex(hexTx)) {
            return false;
        }
        
        std::vector<unsigned char> txData(ParseHex(hexTx));
        CDataStream ssData(txData, SER_NETWORK, PROTOCOL_VERSION);
        try {
            CMutableTransaction tx;
            ssData >> tx;
            return !ssData.empty();
        } catch (const std::exception&) {
            return false;
        }
    }
};
```

## 📊 Rate Limiting

### 1. Token Bucket Implementation
```cpp
// Advanced rate limiting
class RateLimiter {
private:
    struct TokenBucket {
        double tokens;
        double rate;
        double burst;
        int64_t last_update;
    };
    
    std::map<std::string, TokenBucket> buckets;
    mutable CCriticalSection cs_ratelimit;

public:
    bool CheckAndUpdate(const std::string& client_id, size_t cost = 1) {
        LOCK(cs_ratelimit);
        
        auto& bucket = buckets[client_id];
        int64_t now = GetTimeMicros();
        
        // Refill tokens
        double elapsed = (now - bucket.last_update) / 1000000.0;
        bucket.tokens = std::min(bucket.burst,
                               bucket.tokens + elapsed * bucket.rate);
        bucket.last_update = now;
        
        // Check if enough tokens
        if (bucket.tokens >= cost) {
            bucket.tokens -= cost;
            return true;
        }
        return false;
    }
};
```

### 2. Dynamic Rate Limiting
```cpp
// Dynamic rate limiting based on system load
class DynamicRateLimiter {
private:
    RateLimiter base_limiter;
    
    double GetSystemLoad() {
        double loadavg[3];
        if (getloadavg(loadavg, 3) == -1) {
            return 1.0;
        }
        return loadavg[0];
    }
    
    int64_t GetMemoryUsage() {
        return GetSystemMemoryUsage();
    }

public:
    bool CheckRequest(const std::string& client_id, 
                     const JSONRPCRequest& request) {
        // Calculate request cost
        size_t cost = CalculateRequestCost(request);
        
        // Adjust based on system load
        double load_factor = GetSystemLoad();
        if (load_factor > 1.0) {
            cost = static_cast<size_t>(cost * load_factor);
        }
        
        // Check memory usage
        if (GetMemoryUsage() > HIGH_MEMORY_THRESHOLD) {
            cost *= 2;  // Double cost when memory usage is high
        }
        
        return base_limiter.CheckAndUpdate(client_id, cost);
    }
};
```

## 🔄 Error Handling Patterns

### 1. Structured Error Response
```cpp
// Comprehensive error handling
class RPCError {
public:
    static UniValue CreateError(RPCErrorCode code, 
                              const std::string& message,
                              const UniValue& details = NullUniValue) {
        UniValue error(UniValue::VOBJ);
        error.pushKV("code", code);
        error.pushKV("message", message);
        if (!details.isNull()) {
            error.pushKV("details", details);
        }
        
        // Add debug information in development
        #ifdef DEBUG
        error.pushKV("stack_trace", GetStackTrace());
        #endif
        
        return error;
    }
    
    static void ThrowIfNull(const UniValue& value, 
                           const std::string& param_name) {
        if (value.isNull()) {
            throw JSONRPCError(RPC_INVALID_PARAMETER,
                             strprintf("Parameter %s cannot be null", param_name));
        }
    }
};
```

### 2. Exception Handling
```cpp
// Exception handling middleware
UniValue HandleRequest(const JSONRPCRequest& request)
{
    try {
        // Pre-request validation
        if (!AuthenticateRequest(request)) {
            throw JSONRPCError(RPC_INVALID_REQUEST, "Unauthorized");
        }
        
        // Rate limiting
        if (!g_ratelimiter.CheckRequest(request.authUser, request)) {
            throw JSONRPCError(RPC_INVALID_REQUEST, "Rate limit exceeded");
        }
        
        // Execute request
        UniValue result = tableRPC.execute(request);
        
        // Post-process result
        return PostProcessResult(result);
        
    } catch (const UniValue& objError) {
        // Handle JSON-RPC errors
        return RPCError::CreateError(objError["code"].get_int(),
                                   objError["message"].get_str());
    } catch (const std::exception& e) {
        // Handle standard exceptions
        return RPCError::CreateError(RPC_MISC_ERROR, e.what());
    } catch (...) {
        // Handle unknown exceptions
        return RPCError::CreateError(RPC_MISC_ERROR, 
                                   "Unknown error occurred");
    }
}
```

## 🎯 Knowledge Check
1. How do you implement secure authentication in Bitcoin Core APIs?
2. What are the key components of effective rate limiting?
3. How do you handle different types of API errors?
4. What caching strategies should be used for block information?
5. How do you implement proper input validation?

## 🔍 Common Issues and Solutions

### Issue 1: Performance
- Implement proper caching strategies
- Use batch processing for multiple requests
- Optimize database queries
- Monitor and adjust rate limits

### Issue 2: Security
- Use constant-time comparisons for authentication
- Implement proper input sanitization
- Set up comprehensive logging
- Regular security audits

## 📚 Additional Resources
- [Bitcoin Core Developer Documentation](https://github.com/bitcoin/bitcoin/tree/master/doc)
- [JSON-RPC Specification](https://www.jsonrpc.org/specification)
- [Bitcoin Core RPC Interface](https://github.com/bitcoin/bitcoin/blob/master/doc/JSON-RPC-interface.md)
- [Security Best Practices](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#security)

## 🎯 Next Steps
1. Study advanced API patterns
2. Implement custom API endpoints
3. Build comprehensive testing suite
4. Create API monitoring system

Remember: API implementation in Bitcoin Core requires careful attention to security, performance, and reliability. Always validate inputs, implement proper rate limiting, and handle errors gracefully.
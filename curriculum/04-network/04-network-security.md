# Bitcoin Core Network Security

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Implement robust network security measures
- Detect and prevent common attack vectors
- Apply encryption patterns effectively
- Monitor network security in real-time
- Design secure peer communication protocols

## Prerequisites
- Understanding of P2P network architecture (covered in 01-p2p-architecture.md)
- Knowledge of message handling (covered in 03-message-handling.md)
- Familiarity with cryptography basics
- Experience with network programming

## 🌟 Real-World Context
Think of Bitcoin's network security like a bank's security system:
- Encryption = Vault Doors
- Attack Prevention = Security Guards
- Monitoring = Surveillance Systems
- Peer Authentication = ID Verification
- DoS Protection = Access Control

## 🔒 Security Components

### 1. Peer Authentication

```cpp
// Advanced peer authentication
class CPeerAuthenticator {
private:
    // Authentication states
    enum class AuthState {
        NONE,
        CHALLENGE_SENT,
        CHALLENGE_RECEIVED,
        AUTHENTICATED
    };
    
    struct AuthInfo {
        AuthState state;
        uint256 challenge;
        int64_t timestamp;
        std::vector<unsigned char> pubkey;
    };
    std::map<NodeId, AuthInfo> mapAuthInfo;

public:
    bool AuthenticatePeer(CNode* pnode) {
        LOCK(cs_auth);
        auto& auth = mapAuthInfo[pnode->GetId()];
        
        // Generate challenge
        GetStrongRandBytes((unsigned char*)&auth.challenge, sizeof(auth.challenge));
        auth.timestamp = GetTime();
        auth.state = AuthState::CHALLENGE_SENT;
        
        // Send challenge
        pnode->PushMessage(NetMsgType::AUTHCHALLENGE,
                          auth.challenge);
        
        return true;
    }
    
    bool VerifyResponse(CNode* pnode, const uint256& response,
                       const std::vector<unsigned char>& pubkey)
    {
        LOCK(cs_auth);
        auto& auth = mapAuthInfo[pnode->GetId()];
        
        // Verify response timing
        if (GetTime() - auth.timestamp > MAX_AUTH_TIMEOUT) {
            LogPrint(BCLog::NET, "Auth timeout for peer=%d\n",
                     pnode->GetId());
            return false;
        }
        
        // Verify signature
        if (!VerifySignature(auth.challenge, response, pubkey)) {
            LogPrint(BCLog::NET, "Invalid auth signature from peer=%d\n",
                     pnode->GetId());
            return false;
        }
        
        // Update state
        auth.state = AuthState::AUTHENTICATED;
        auth.pubkey = pubkey;
        
        return true;
    }
};
```

### 2. DoS Protection

```cpp
// Comprehensive DoS protection
class CDoSManager {
private:
    struct PeerScore {
        std::atomic<int64_t> nMisbehavior{0};
        std::atomic<int64_t> nLastMisbehavior{0};
        std::atomic<uint64_t> nBytesRecv{0};
        std::atomic<uint64_t> nBytesSent{0};
        std::atomic<uint64_t> nInvalidMessages{0};
    };
    std::map<NodeId, PeerScore> mapPeerScores;
    
    // DoS parameters
    const int64_t BAN_THRESHOLD = 100;
    const int64_t MISBEHAVIOR_DECAY = 60 * 60;  // 1 hour
    
public:
    bool CheckDoS(CNode* pnode, int nDoS, const std::string& reason) {
        if (nDoS == 0) return true;
        
        LOCK(cs_score);
        auto& score = mapPeerScores[pnode->GetId()];
        
        // Apply time decay to previous score
        int64_t nNow = GetTime();
        int64_t nTimeDiff = nNow - score.nLastMisbehavior;
        if (nTimeDiff > 0) {
            score.nMisbehavior = std::max(0L,
                score.nMisbehavior - (nTimeDiff / MISBEHAVIOR_DECAY));
        }
        
        // Add new score
        score.nMisbehavior += nDoS;
        score.nLastMisbehavior = nNow;
        
        // Check ban threshold
        if (score.nMisbehavior >= BAN_THRESHOLD) {
            LogPrint(BCLog::NET, "Banning peer=%d: %s (score=%d)\n",
                     pnode->GetId(), reason, score.nMisbehavior);
            pnode->fDisconnect = true;
            return false;
        }
        
        return true;
    }
};
```

### 3. Attack Prevention

```cpp
// Advanced attack prevention
class CAttackPrevention {
private:
    // Attack detection parameters
    const uint64_t MAX_HEADERS_PER_MINUTE = 160;
    const uint64_t MAX_BLOCKS_PER_MINUTE = 40;
    const uint64_t MAX_ADDR_RATE = 1000;
    const uint64_t MAX_INV_RATE = 2000;
    
    struct RateLimit {
        std::atomic<uint64_t> count{0};
        std::atomic<int64_t> lastReset{0};
    };
    std::map<NodeId, std::map<std::string, RateLimit>> mapRateLimits;

public:
    bool CheckAttackPattern(CNode* pnode, const std::string& pattern,
                          uint64_t maxRate)
    {
        LOCK(cs_attack);
        auto& limits = mapRateLimits[pnode->GetId()];
        auto& rate = limits[pattern];
        
        // Reset counter if needed
        int64_t nNow = GetTime();
        if (nNow - rate.lastReset >= 60) {
            rate.count = 0;
            rate.lastReset = nNow;
        }
        
        // Check rate
        if (++rate.count > maxRate) {
            LogPrint(BCLog::NET, "Attack pattern detected: %s from peer=%d\n",
                     pattern, pnode->GetId());
            return false;
        }
        
        return true;
    }
    
    bool CheckHeadersFlooding(CNode* pnode) {
        return CheckAttackPattern(pnode, "headers", MAX_HEADERS_PER_MINUTE);
    }
    
    bool CheckBlocksFlooding(CNode* pnode) {
        return CheckAttackPattern(pnode, "blocks", MAX_BLOCKS_PER_MINUTE);
    }
    
    bool CheckAddrFlooding(CNode* pnode) {
        return CheckAttackPattern(pnode, "addr", MAX_ADDR_RATE);
    }
};
```

### 4. Encryption Patterns

```cpp
// Secure communication patterns
class CSecureCommunication {
private:
    struct ChannelState {
        std::vector<unsigned char> sharedSecret;
        std::vector<unsigned char> sendKey;
        std::vector<unsigned char> recvKey;
        uint64_t sendNonce{0};
        uint64_t recvNonce{0};
    };
    std::map<NodeId, ChannelState> mapChannels;

public:
    bool EstablishSecureChannel(CNode* pnode,
                              const std::vector<unsigned char>& pubkey)
    {
        LOCK(cs_channel);
        auto& channel = mapChannels[pnode->GetId()];
        
        try {
            // Generate ephemeral key pair
            CKey ephemeral;
            ephemeral.MakeNewKey(true);
            
            // Compute shared secret
            if (!ComputeSharedSecret(ephemeral, pubkey, channel.sharedSecret)) {
                return false;
            }
            
            // Derive keys
            DeriveKeys(channel.sharedSecret, channel.sendKey, channel.recvKey);
            
            // Send public key
            std::vector<unsigned char> ephemeralpub = ephemeral.GetPubKey();
            pnode->PushMessage(NetMsgType::SECUREKEY, ephemeralpub);
            
            return true;
            
        } catch (const std::exception& e) {
            LogPrint(BCLog::NET, "Secure channel error: %s\n", e.what());
            return false;
        }
    }
    
    bool EncryptMessage(CNode* pnode, CDataStream& message) {
        LOCK(cs_channel);
        auto& channel = mapChannels[pnode->GetId()];
        
        try {
            // Generate nonce
            unsigned char nonce[12];
            memcpy(nonce, &channel.sendNonce, 8);
            GetRandBytes(nonce + 8, 4);
            channel.sendNonce++;
            
            // Encrypt message
            std::vector<unsigned char> ciphertext;
            if (!EncryptAEAD(message, channel.sendKey, nonce, ciphertext)) {
                return false;
            }
            
            // Replace message with ciphertext
            message.clear();
            message.insert(message.end(), ciphertext.begin(), ciphertext.end());
            
            return true;
            
        } catch (const std::exception& e) {
            LogPrint(BCLog::NET, "Encryption error: %s\n", e.what());
            return false;
        }
    }
};
```

### 5. Security Monitoring

```cpp
// Comprehensive security monitoring
class CSecurityMonitor {
private:
    struct SecurityEvent {
        int64_t timestamp;
        std::string type;
        NodeId peerId;
        std::string details;
        int severity;
    };
    std::deque<SecurityEvent> securityEvents;
    
    // Alert thresholds
    const int ALERT_WINDOW = 3600;  // 1 hour
    const int MAX_AUTH_FAILURES = 5;
    const int MAX_DOS_EVENTS = 10;
    const int MAX_ATTACK_PATTERNS = 3;

public:
    void LogSecurityEvent(const std::string& type, NodeId peerId,
                         const std::string& details, int severity)
    {
        LOCK(cs_events);
        
        // Add event
        securityEvents.push_back({GetTime(), type, peerId, details, severity});
        
        // Maintain fixed size
        while (securityEvents.size() > MAX_EVENTS) {
            securityEvents.pop_front();
        }
        
        // Check alert conditions
        CheckAlerts();
    }
    
    void CheckAlerts() {
        int64_t nNow = GetTime();
        std::map<std::string, int> eventCounts;
        
        // Count recent events
        for (const auto& event : securityEvents) {
            if (nNow - event.timestamp <= ALERT_WINDOW) {
                eventCounts[event.type]++;
            }
        }
        
        // Check thresholds
        if (eventCounts["auth_failure"] >= MAX_AUTH_FAILURES) {
            AlertSecurityIssue("Multiple authentication failures detected");
        }
        
        if (eventCounts["dos_event"] >= MAX_DOS_EVENTS) {
            AlertSecurityIssue("High rate of DoS events detected");
        }
        
        if (eventCounts["attack_pattern"] >= MAX_ATTACK_PATTERNS) {
            AlertSecurityIssue("Multiple attack patterns detected");
        }
    }
};
```

## 🎯 Knowledge Check
1. How do you implement effective peer authentication?
2. What are the key components of DoS protection?
3. How do you detect and prevent common attack patterns?
4. What encryption patterns are used for secure communication?
5. How do you monitor network security effectively?

## 🔍 Common Issues and Solutions

### Issue 1: Authentication Failures
- Implement proper challenge-response
- Use strong cryptography
- Monitor failed attempts
- Implement rate limiting

### Issue 2: DoS Attacks
- Score-based peer banning
- Rate limiting by message type
- Resource usage monitoring
- Connection filtering

### Issue 3: Network Attacks
- Header flooding protection
- Block flooding protection
- Address flooding protection
- Invalid message detection

## 📚 Additional Resources
- [Bitcoin P2P Network Security](https://github.com/bitcoin/bitcoin/blob/master/doc/p2p-network.md)
- [DoS Prevention](https://github.com/bitcoin/bitcoin/blob/master/doc/dos-mitigation.md)
- [Network Encryption](https://github.com/bitcoin/bitcoin/blob/master/doc/encryption.md)
- [Security Best Practices](https://github.com/bitcoin/bitcoin/blob/master/doc/security-check.md)

## 🎯 Next Steps
1. Study advanced attack vectors
2. Implement custom security measures
3. Build security monitoring tools
4. Contribute to network security

Remember: Network security is crucial for maintaining a healthy Bitcoin network. Focus on prevention, detection, and rapid response to security threats.
``` 
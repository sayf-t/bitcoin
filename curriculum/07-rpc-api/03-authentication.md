# Bitcoin Core Authentication System

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Implement secure authentication mechanisms
- Design role-based access control (RBAC)
- Manage JWT tokens securely
- Handle API keys and secrets
- Implement OAuth2 integration
- Monitor security events

## Prerequisites
- Understanding of RPC interface (covered in 01-rpc-interface.md)
- Knowledge of API implementation (covered in 02-api-implementation.md)
- Experience with cryptographic principles
- Familiarity with security best practices

## 🌟 Real-World Analogy
Think of the authentication system like a bank's security system:
- Authentication = ID Verification
- Authorization = Access Levels
- Tokens = Security Badges
- Rate Limiting = Queue Management
- Audit Logs = Security Cameras
- Role Management = Staff Permissions

## Authentication Components

### 1. Authentication Manager
```typescript
interface AuthConfig {
    jwtSecret: string;
    tokenExpiry: number;
    refreshTokenExpiry: number;
    maxLoginAttempts: number;
    lockoutDuration: number;
}

class AuthenticationManager {
    private readonly config: AuthConfig;
    private readonly tokenManager: TokenManager;
    private readonly userManager: UserManager;
    private readonly auditLogger: AuditLogger;

    constructor(config: AuthConfig) {
        this.config = config;
        this.tokenManager = new TokenManager(config);
        this.userManager = new UserManager();
        this.auditLogger = new AuditLogger();
    }

    async authenticate(credentials: Credentials): Promise<AuthResult> {
        try {
            // Check rate limits
            await this.checkRateLimit(credentials.username);
            
            // Validate credentials
            const user = await this.userManager.validateCredentials(credentials);
            
            // Generate tokens
            const tokens = await this.tokenManager.generateTokens(user);
            
            // Log successful authentication
            await this.auditLogger.logAuth({
                type: 'login',
                user: user.username,
                success: true
            });
            
            return {
                user: this.sanitizeUser(user),
                ...tokens
            };
        } catch (error) {
            // Log failed attempt
            await this.auditLogger.logAuth({
                type: 'login',
                user: credentials.username,
                success: false,
                error: error.message
            });
            
            throw error;
        }
    }
}
```

### 2. Token Management
```typescript
class TokenManager {
    private readonly jwtSecret: string;
    private readonly redis: Redis;

    async generateTokens(user: User): Promise<TokenPair> {
        const accessToken = await this.generateAccessToken(user);
        const refreshToken = await this.generateRefreshToken(user);
        
        // Store refresh token
        await this.storeRefreshToken(user.id, refreshToken);
        
        return {
            accessToken,
            refreshToken,
            expiresIn: this.config.tokenExpiry
        };
    }

    async validateToken(token: string): Promise<DecodedToken> {
        try {
            // Verify JWT signature
            const decoded = jwt.verify(token, this.jwtSecret);
            
            // Check if token is blacklisted
            const isBlacklisted = await this.checkBlacklist(token);
            if (isBlacklisted) {
                throw new TokenBlacklistedError();
            }
            
            return decoded;
        } catch (error) {
            if (error instanceof jwt.TokenExpiredError) {
                throw new TokenExpiredError();
            }
            throw new InvalidTokenError();
        }
    }
}
```

## 💻 Hands-on Implementation

### 1. Role-Based Access Control
```typescript
class RBACManager {
    private readonly roles: Map<string, Role>;
    private readonly permissions: Map<string, Permission>;

    async checkPermission(user: User, resource: string, action: string): Promise<boolean> {
        // Get user roles
        const userRoles = await this.getUserRoles(user.id);
        
        // Check each role's permissions
        for (const role of userRoles) {
            const hasPermission = await this.roleHasPermission(role, resource, action);
            if (hasPermission) return true;
        }
        
        return false;
    }

    async grantRole(userId: string, roleId: string): Promise<void> {
        // Validate role exists
        const role = await this.getRole(roleId);
        if (!role) throw new RoleNotFoundError();
        
        // Grant role to user
        await this.userManager.addRole(userId, roleId);
        
        // Log role assignment
        await this.auditLogger.logRoleChange({
            type: 'grant',
            userId,
            roleId
        });
    }
}
```

### 2. Security Middleware
```typescript
class SecurityMiddleware {
    private readonly authManager: AuthenticationManager;
    private readonly rbacManager: RBACManager;

    async authenticate(req: Request, res: Response, next: NextFunction): Promise<void> {
        try {
            // Extract token
            const token = this.extractToken(req);
            
            // Validate token
            const decoded = await this.authManager.validateToken(token);
            
            // Attach user to request
            req.user = decoded.user;
            
            next();
        } catch (error) {
            next(new AuthenticationError(error.message));
        }
    }

    async authorize(resource: string, action: string) {
        return async (req: Request, res: Response, next: NextFunction) => {
            try {
                const hasPermission = await this.rbacManager.checkPermission(
                    req.user,
                    resource,
                    action
                );
                
                if (!hasPermission) {
                    throw new AuthorizationError('Insufficient permissions');
                }
                
                next();
            } catch (error) {
                next(error);
            }
        };
    }
}
```

## 🛠️ Practice Project: Advanced Auth System

Let's build a comprehensive authentication system.

### Project Structure
```bash
bitcoin-auth/
├── src/
│   ├── auth/
│   │   ├── auth-manager.ts
│   │   ├── token-manager.ts
│   │   └── rbac-manager.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts
│   │   └── rbac.middleware.ts
│   ├── models/
│   │   ├── user.model.ts
│   │   └── role.model.ts
│   └── utils/
│       ├── crypto.ts
│       └── audit-logger.ts
├── tests/
└── package.json
```

### Implementation Example
```typescript
// src/auth/auth-manager.ts
class BitcoinAuthSystem {
    private readonly authManager: AuthenticationManager;
    private readonly rbacManager: RBACManager;
    private readonly securityMonitor: SecurityMonitor;

    constructor(config: AuthConfig) {
        this.authManager = new AuthenticationManager(config);
        this.rbacManager = new RBACManager();
        this.securityMonitor = new SecurityMonitor();
        
        // Setup security monitoring
        this.setupMonitoring();
        // Initialize RBAC
        this.initializeRBAC();
    }

    private async initializeRBAC(): Promise<void> {
        // Define default roles
        await this.rbacManager.createRole('admin', {
            description: 'Full system access',
            permissions: ['*']
        });
        
        await this.rbacManager.createRole('operator', {
            description: 'Node operation access',
            permissions: ['read:*', 'write:blockchain', 'write:network']
        });
        
        await this.rbacManager.createRole('viewer', {
            description: 'Read-only access',
            permissions: ['read:*']
        });
    }

    private setupMonitoring(): void {
        // Monitor authentication events
        this.securityMonitor.on('auth:failure', this.handleAuthFailure);
        this.securityMonitor.on('auth:success', this.handleAuthSuccess);
        
        // Monitor suspicious activities
        this.securityMonitor.on('security:breach', this.handleSecurityBreach);
        this.securityMonitor.on('security:warning', this.handleSecurityWarning);
    }
}
```

## 🎯 Knowledge Check
1. How do you implement secure token management?
2. What are the best practices for RBAC?
3. How do you handle token refresh securely?
4. What security monitoring should be implemented?
5. How do you manage API key rotation?

## 🔍 Common Issues and Solutions

### Issue 1: Token Security
- Token expiration
- Refresh token rotation
- Token revocation
- Secure storage

### Issue 2: Access Control
- Permission granularity
- Role inheritance
- Dynamic permissions
- Access auditing

## 📚 Additional Resources
- [OWASP Authentication Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [JWT Best Practices](https://datatracker.ietf.org/doc/html/rfc8725)
- [OAuth 2.0 Security](https://oauth.net/2/security-considerations/)
- [RBAC Design Patterns](https://en.wikipedia.org/wiki/Role-based_access_control)

## Next Steps
1. Study advanced security patterns
2. Implement OAuth2 providers
3. Build security monitoring
4. Create automated security tests
``` 
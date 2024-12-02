# Bitcoin Core Request Handling

## 🎯 Learning Objectives
After completing this module, you will be able to:
- Implement robust request validation
- Design error handling strategies
- Process complex API requests
- Format standardized responses
- Handle rate limiting and throttling
- Monitor request patterns

## Prerequisites
- Understanding of RPC interface (covered in 01-rpc-interface.md)
- Knowledge of API implementation (covered in 02-api-implementation.md)
- Experience with authentication (covered in 03-authentication.md)
- Familiarity with JSON schema validation

## 🌟 Real-World Analogy
Think of request handling like a bank's transaction processing:
- Validation = Transaction Verification
- Processing = Transaction Execution
- Error Handling = Problem Resolution
- Rate Limiting = Queue Management
- Response Format = Receipt Generation
- Monitoring = Transaction Logging

## Request Components

### 1. Request Validator
```typescript
interface ValidationConfig {
    schemas: Map<string, JSONSchema>;
    strictMode: boolean;
    maxDepth: number;
    maxArrayLength: number;
    customValidators: Map<string, ValidatorFn>;
}

class RequestValidator {
    private readonly config: ValidationConfig;
    private readonly schemaValidator: JSONSchemaValidator;
    private readonly sanitizer: InputSanitizer;

    constructor(config: ValidationConfig) {
        this.config = config;
        this.schemaValidator = new JSONSchemaValidator(config);
        this.sanitizer = new InputSanitizer();
    }

    async validateRequest(req: Request): Promise<ValidationResult> {
        try {
            // Get schema for endpoint
            const schema = await this.getSchemaForEndpoint(req.path);
            
            // Sanitize input
            const sanitizedData = this.sanitizer.sanitize(req.body);
            
            // Validate against schema
            const result = await this.schemaValidator.validate(sanitizedData, schema);
            
            if (!result.valid) {
                throw new ValidationError(result.errors);
            }
            
            return {
                valid: true,
                sanitizedData
            };
        } catch (error) {
            return {
                valid: false,
                errors: this.formatValidationErrors(error)
            };
        }
    }
}
```

### 2. Request Processor
```typescript
class RequestProcessor {
    private readonly validator: RequestValidator;
    private readonly rateLimit: RateLimiter;
    private readonly metrics: RequestMetrics;

    async processRequest(req: Request): Promise<ProcessedRequest> {
        try {
            // Check rate limits
            await this.rateLimit.checkLimit(req);
            
            // Validate request
            const validation = await this.validator.validateRequest(req);
            if (!validation.valid) {
                throw new ValidationError(validation.errors);
            }
            
            // Transform request
            const transformed = await this.transformRequest(validation.sanitizedData);
            
            // Update metrics
            await this.metrics.trackRequest(req);
            
            return {
                data: transformed,
                metadata: this.buildMetadata(req)
            };
        } catch (error) {
            throw this.handleProcessingError(error);
        }
    }

    private async transformRequest(data: any): Promise<TransformedData> {
        // Apply business logic transformations
        const transformed = await this.applyTransformations(data);
        
        // Validate transformed data
        await this.validateTransformedData(transformed);
        
        return transformed;
    }
}
```

## 💻 Hands-on Implementation

### 1. Error Handler
```typescript
class ErrorHandler {
    private readonly logger: ErrorLogger;
    private readonly metrics: ErrorMetrics;

    handleError(error: Error, req: Request, res: Response): void {
        // Log error
        this.logger.logError(error, req);
        
        // Track metrics
        this.metrics.trackError(error);
        
        // Format error response
        const response = this.formatErrorResponse(error);
        
        // Send response
        res.status(response.status).json(response);
    }

    private formatErrorResponse(error: Error): ErrorResponse {
        switch (error.constructor) {
            case ValidationError:
                return {
                    status: 400,
                    code: 'VALIDATION_ERROR',
                    message: error.message,
                    details: error.details
                };
            case AuthenticationError:
                return {
                    status: 401,
                    code: 'AUTH_ERROR',
                    message: error.message
                };
            case RateLimitError:
                return {
                    status: 429,
                    code: 'RATE_LIMIT_ERROR',
                    message: error.message,
                    retryAfter: error.retryAfter
                };
            default:
                return {
                    status: 500,
                    code: 'INTERNAL_ERROR',
                    message: 'Internal server error'
                };
        }
    }
}
```

### 2. Response Formatter
```typescript
class ResponseFormatter {
    private readonly config: FormatConfig;
    private readonly templates: ResponseTemplates;

    formatResponse(data: any, req: Request): FormattedResponse {
        // Get response template
        const template = this.getTemplate(req.path);
        
        // Apply transformations
        const transformed = this.applyTransformations(data, template);
        
        // Add metadata
        const metadata = this.buildMetadata(req);
        
        return {
            success: true,
            data: transformed,
            metadata
        };
    }

    private buildMetadata(req: Request): ResponseMetadata {
        return {
            timestamp: new Date().toISOString(),
            path: req.path,
            version: this.config.version,
            requestId: req.id
        };
    }
}
```

## 🛠️ Practice Project: Advanced Request Handler

Let's build a comprehensive request handling system.

### Project Structure
```bash
request-handler/
├── src/
│   ├── validation/
│   │   ├── validator.ts
│   │   ├── schemas.ts
│   │   └── sanitizer.ts
│   ├── processing/
│   │   ├── processor.ts
│   │   └── transformer.ts
│   ├── errors/
│   │   ├── handler.ts
│   │   └── formatter.ts
│   └── utils/
│       ├── logger.ts
│       └── metrics.ts
├── tests/
└── package.json
```

### Implementation Example
```typescript
// src/request-handler.ts
class BitcoinRequestHandler {
    private readonly validator: RequestValidator;
    private readonly processor: RequestProcessor;
    private readonly errorHandler: ErrorHandler;
    private readonly formatter: ResponseFormatter;

    async handleRequest(req: Request, res: Response): Promise<void> {
        try {
            // Process request
            const processed = await this.processor.processRequest(req);
            
            // Format response
            const response = this.formatter.formatResponse(processed, req);
            
            // Send response
            res.json(response);
        } catch (error) {
            // Handle errors
            this.errorHandler.handleError(error, req, res);
        } finally {
            // Cleanup
            await this.cleanup(req);
        }
    }

    private async cleanup(req: Request): Promise<void> {
        // Release resources
        await this.releaseResources(req);
        
        // Update metrics
        await this.updateMetrics(req);
        
        // Log completion
        await this.logCompletion(req);
    }
}
```

## 🎯 Knowledge Check
1. How do you implement request validation?
2. What are the best practices for error handling?
3. How do you manage request processing?
4. What response formats should be used?
5. How do you handle request timeouts?

## 🔍 Common Issues and Solutions

### Issue 1: Validation
- Schema versioning
- Custom validators
- Complex data types
- Nested validation

### Issue 2: Processing
- Request timeout
- Resource exhaustion
- Concurrent processing
- State management

## 📚 Additional Resources
- [JSON Schema Validation](https://json-schema.org/understanding-json-schema/)
- [REST API Error Handling](https://www.rfc-editor.org/rfc/rfc7807)
- [API Request Processing](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md)
- [Rate Limiting Best Practices](https://datatracker.ietf.org/doc/html/rfc6585)

## Next Steps
1. Study advanced validation patterns
2. Implement custom processors
3. Build error monitoring
4. Create response templates
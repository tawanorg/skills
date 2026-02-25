# API Design Patterns

RESTful conventions, error responses, and versioning for enterprise APIs.

## REST Resource Design

### URL Structure

```
# Resources are nouns, plural
GET    /users              # List users
POST   /users              # Create user
GET    /users/{id}         # Get user
PUT    /users/{id}         # Replace user
PATCH  /users/{id}         # Update user
DELETE /users/{id}         # Delete user

# Nested resources for relationships
GET    /users/{id}/orders  # User's orders
POST   /users/{id}/orders  # Create order for user

# Actions as sub-resources when needed
POST   /orders/{id}/cancel # Cancel order (state change)
POST   /users/{id}/verify  # Trigger verification
```

### HTTP Methods

| Method | Idempotent | Safe | Use Case |
|--------|------------|------|----------|
| GET | Yes | Yes | Retrieve resource |
| POST | No | No | Create resource, trigger action |
| PUT | Yes | No | Replace entire resource |
| PATCH | Yes | No | Partial update |
| DELETE | Yes | No | Remove resource |

### Status Codes

```typescript
// Success
200 OK           // GET, PUT, PATCH success
201 Created      // POST success (include Location header)
204 No Content   // DELETE success

// Client errors
400 Bad Request  // Invalid input
401 Unauthorized // Missing/invalid auth
403 Forbidden    // Valid auth, insufficient permissions
404 Not Found    // Resource doesn't exist
409 Conflict     // State conflict (duplicate, version mismatch)
422 Unprocessable Entity // Valid syntax, semantic error
429 Too Many Requests    // Rate limited

// Server errors
500 Internal Server Error // Unexpected failure
502 Bad Gateway          // Upstream service failed
503 Service Unavailable  // Temporary overload
504 Gateway Timeout      // Upstream timeout
```

## Request/Response Design

### Request Validation

```typescript
// Use strict schemas
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  role: z.enum(['user', 'admin']).default('user'),
});

app.post('/users', async (req, res) => {
  const result = CreateUserSchema.safeParse(req.body);

  if (!result.success) {
    return res.status(400).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid request body',
        details: result.error.flatten().fieldErrors,
      },
    });
  }

  const user = await userService.create(result.data);
  res.status(201).json({ data: user });
});
```

### Response Envelope

```typescript
// Success response
{
  "data": { ... },
  "meta": {
    "requestId": "req_abc123"
  }
}

// List response with pagination
{
  "data": [ ... ],
  "meta": {
    "total": 150,
    "page": 2,
    "perPage": 20,
    "totalPages": 8
  },
  "links": {
    "self": "/users?page=2",
    "first": "/users?page=1",
    "prev": "/users?page=1",
    "next": "/users?page=3",
    "last": "/users?page=8"
  }
}

// Error response
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with id 'usr_123' was not found",
    "details": { ... },
    "requestId": "req_abc123"
  }
}
```

## Error Handling

### Consistent Error Format

```typescript
interface ApiError {
  code: string;        // Machine-readable: USER_NOT_FOUND
  message: string;     // Human-readable
  details?: unknown;   // Additional context
  requestId: string;   // For support correlation
}

// Error codes by category
const ErrorCodes = {
  // Validation
  VALIDATION_ERROR: 400,
  INVALID_EMAIL: 400,

  // Auth
  UNAUTHORIZED: 401,
  TOKEN_EXPIRED: 401,
  FORBIDDEN: 403,

  // Not found
  USER_NOT_FOUND: 404,
  ORDER_NOT_FOUND: 404,

  // Conflict
  EMAIL_ALREADY_EXISTS: 409,
  VERSION_CONFLICT: 409,

  // Rate limit
  RATE_LIMITED: 429,

  // Server
  INTERNAL_ERROR: 500,
  SERVICE_UNAVAILABLE: 503,
} as const;
```

### Error Handler Middleware

```typescript
class AppError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode: number,
    public details?: unknown
  ) {
    super(message);
  }
}

app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  const requestId = req.headers['x-request-id'] as string;

  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
        details: err.details,
        requestId,
      },
    });
  }

  // Unexpected error - don't leak details
  logger.error('Unhandled error', { error: err, requestId });

  res.status(500).json({
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred',
      requestId,
    },
  });
});
```

## Pagination

### Cursor-Based (Recommended)

```typescript
// Request
GET /orders?cursor=eyJpZCI6MTIzfQ&limit=20

// Response
{
  "data": [...],
  "meta": {
    "hasMore": true,
    "nextCursor": "eyJpZCI6MTQzfQ"
  }
}

// Implementation
async function listOrders(cursor?: string, limit = 20) {
  const decoded = cursor ? JSON.parse(atob(cursor)) : null;

  const orders = await db.orders
    .where(decoded ? { id: { $gt: decoded.id } } : {})
    .orderBy('id')
    .limit(limit + 1)  // Fetch one extra to check hasMore
    .find();

  const hasMore = orders.length > limit;
  const results = hasMore ? orders.slice(0, -1) : orders;
  const lastItem = results[results.length - 1];

  return {
    data: results,
    meta: {
      hasMore,
      nextCursor: hasMore ? btoa(JSON.stringify({ id: lastItem.id })) : null,
    },
  };
}
```

### Offset-Based (Simpler but limited)

```typescript
// Request
GET /users?page=2&perPage=20

// Good for small datasets, problematic for large ones
// - Inconsistent results when data changes
// - Performance degrades at high offsets
```

## Versioning

### URL Path Versioning (Recommended)

```typescript
// Simple and explicit
app.use('/v1/users', usersV1Router);
app.use('/v2/users', usersV2Router);

// URLs
GET /v1/users
GET /v2/users  // Different response shape
```

### Header Versioning (Alternative)

```typescript
// Request
GET /users
Accept: application/vnd.api+json; version=2

// Middleware
app.use((req, res, next) => {
  const version = parseVersion(req.headers.accept) || 1;
  req.apiVersion = version;
  next();
});
```

## Rate Limiting

### Standard Headers

```typescript
// Response headers
res.set({
  'X-RateLimit-Limit': '100',      // Requests per window
  'X-RateLimit-Remaining': '95',   // Remaining requests
  'X-RateLimit-Reset': '1640995200', // Window reset time (Unix)
  'Retry-After': '60',             // Seconds until retry (when limited)
});
```

### Rate Limit Response

```typescript
// 429 Too Many Requests
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Rate limit exceeded. Retry after 60 seconds.",
    "details": {
      "limit": 100,
      "remaining": 0,
      "resetAt": "2024-01-15T12:00:00Z"
    }
  }
}
```

## API Design Checklist

### For Every Endpoint

- [ ] Uses correct HTTP method
- [ ] Returns appropriate status code
- [ ] Request body validated with schema
- [ ] Response follows envelope format
- [ ] Errors include code, message, requestId

### For Collections

- [ ] Supports pagination (cursor preferred)
- [ ] Supports filtering where needed
- [ ] Supports sorting where needed
- [ ] Returns total count in meta

### For the API

- [ ] Versioning strategy defined
- [ ] Rate limiting implemented
- [ ] Authentication documented
- [ ] OpenAPI/Swagger spec maintained

# Error Handling Patterns

Language-specific patterns for production-quality error handling.

## TypeScript/JavaScript

### Custom Error Classes

```typescript
class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message: string, public fields: Record<string, string>) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} with id ${id} not found`, 'NOT_FOUND', 404);
  }
}
```

### Result Type Pattern

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

async function fetchUser(id: string): Promise<Result<User>> {
  try {
    const user = await db.users.findById(id);
    if (!user) {
      return { success: false, error: new NotFoundError('User', id) };
    }
    return { success: true, data: user };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}

// Usage
const result = await fetchUser(id);
if (!result.success) {
  logger.error('Failed to fetch user', { id, error: result.error });
  return;
}
const user = result.data;
```

### Async Error Handling

```typescript
// BAD: Unhandled promise rejection
async function processItems(items: Item[]) {
  items.forEach(async (item) => {
    await processItem(item); // Errors lost!
  });
}

// GOOD: Proper async handling
async function processItems(items: Item[]) {
  const results = await Promise.allSettled(
    items.map((item) => processItem(item))
  );

  const failures = results.filter(
    (r): r is PromiseRejectedResult => r.status === 'rejected'
  );

  if (failures.length > 0) {
    logger.error('Some items failed to process', {
      failed: failures.length,
      total: items.length,
      errors: failures.map((f) => f.reason),
    });
  }
}
```

## Python

### Exception Hierarchy

```python
class AppError(Exception):
    """Base application error."""

    def __init__(self, message: str, code: str, status_code: int = 500):
        super().__init__(message)
        self.code = code
        self.status_code = status_code

class ValidationError(AppError):
    """Raised when input validation fails."""

    def __init__(self, message: str, fields: dict[str, str]):
        super().__init__(message, "VALIDATION_ERROR", 400)
        self.fields = fields

class NotFoundError(AppError):
    """Raised when a resource is not found."""

    def __init__(self, resource: str, identifier: str):
        super().__init__(
            f"{resource} with id {identifier} not found",
            "NOT_FOUND",
            404
        )
```

### Context Managers for Cleanup

```python
from contextlib import contextmanager

@contextmanager
def database_transaction():
    connection = get_connection()
    try:
        yield connection
        connection.commit()
    except Exception as e:
        connection.rollback()
        logger.error("Transaction failed", exc_info=True)
        raise
    finally:
        connection.close()

# Usage
with database_transaction() as conn:
    conn.execute("INSERT INTO users ...")
```

## Go

### Error Wrapping

```go
import (
    "errors"
    "fmt"
)

var (
    ErrNotFound = errors.New("not found")
    ErrInvalid  = errors.New("invalid input")
)

func GetUser(id string) (*User, error) {
    user, err := db.FindUser(id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, fmt.Errorf("user %s: %w", id, ErrNotFound)
        }
        return nil, fmt.Errorf("fetching user %s: %w", id, err)
    }
    return user, nil
}

// Usage with error checking
user, err := GetUser(id)
if err != nil {
    if errors.Is(err, ErrNotFound) {
        return NotFoundResponse()
    }
    return InternalErrorResponse()
}
```

### Structured Errors

```go
type AppError struct {
    Code    string `json:"code"`
    Message string `json:"message"`
    Err     error  `json:"-"`
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("%s: %v", e.Message, e.Err)
    }
    return e.Message
}

func (e *AppError) Unwrap() error {
    return e.Err
}

func NewValidationError(message string) *AppError {
    return &AppError{
        Code:    "VALIDATION_ERROR",
        Message: message,
    }
}
```

## Logging Best Practices

### What to Include

```typescript
logger.error('Operation failed', {
  // What happened
  operation: 'user.create',

  // Context
  userId: user.id,
  email: maskEmail(user.email),

  // Error details
  errorCode: error.code,
  errorMessage: error.message,

  // Request context (if applicable)
  requestId: req.id,

  // Timing
  durationMs: Date.now() - startTime,
});
```

### What to Exclude

- Passwords and secrets
- Full credit card numbers
- Personal identification numbers
- Session tokens
- API keys

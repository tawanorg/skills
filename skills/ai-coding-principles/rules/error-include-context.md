---
title: Include Context in Error Messages
impact: HIGH
impactDescription: enables fast debugging
tags: error-handling, debugging, logging
---

## Include Context in Error Messages

Error messages must include what failed, where, and relevant identifiers for debugging.

**Incorrect:**

```typescript
// Useless error messages
throw new Error('Failed');
throw new Error('Invalid input');
throw new Error('Database error');

logger.error('Something went wrong');
```

**Correct:**

```typescript
// Actionable error messages with context
throw new Error(`Failed to save user ${userId}: email already exists`);
throw new ValidationError(`Invalid amount: ${amount} exceeds maximum of 10000`);
throw new DatabaseError(`Query timeout after 30s for orders table, user ${userId}`);

logger.error('Payment processing failed', {
  orderId,
  userId,
  amount,
  errorCode: error.code,
  errorMessage: error.message,
});
```

**Why:** Good error context reduces debugging time from hours to minutes.

Reference: `references/error-handling-patterns.md`

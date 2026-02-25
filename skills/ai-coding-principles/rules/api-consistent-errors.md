---
title: Use Consistent Error Response Format
impact: HIGH
impactDescription: enables standardized client error handling
tags: api, rest, errors
---

## Use Consistent Error Response Format

All API errors must follow the same structure with code, message, and requestId.

**Incorrect:**

```typescript
// Inconsistent error formats - client can't parse reliably
res.status(400).json({ error: 'Invalid email' });
res.status(404).json({ message: 'User not found' });
res.status(500).json({ err: { msg: 'Database error' } });
```

**Correct:**

```typescript
// Consistent error envelope
interface ApiError {
  error: {
    code: string;       // Machine-readable: USER_NOT_FOUND
    message: string;    // Human-readable
    details?: unknown;  // Optional additional context
    requestId: string;  // For support correlation
  };
}

res.status(400).json({
  error: {
    code: 'VALIDATION_ERROR',
    message: 'Invalid email format',
    details: { field: 'email', value: req.body.email },
    requestId: req.id,
  },
});

res.status(404).json({
  error: {
    code: 'USER_NOT_FOUND',
    message: `User with id '${id}' was not found`,
    requestId: req.id,
  },
});
```

**Why:** Consistent errors let clients implement one error handler instead of many.

Reference: `references/api-design-patterns.md`

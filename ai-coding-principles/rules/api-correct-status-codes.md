---
title: Use Correct HTTP Status Codes
impact: HIGH
impactDescription: enables proper client error handling
tags: api, rest, http
---

## Use Correct HTTP Status Codes

Return the appropriate HTTP status code for each response. Don't return 200 for errors.

**Incorrect:**

```typescript
// Everything returns 200 - client can't distinguish success from failure
app.post('/users', async (req, res) => {
  try {
    const user = await createUser(req.body);
    res.json({ success: true, data: user });
  } catch (e) {
    res.json({ success: false, error: e.message });  // Still 200!
  }
});
```

**Correct:**

```typescript
app.post('/users', async (req, res) => {
  try {
    const user = await createUser(req.body);
    res.status(201).json({ data: user });  // 201 Created
  } catch (e) {
    if (e instanceof ValidationError) {
      res.status(400).json({ error: { code: 'VALIDATION_ERROR', message: e.message } });
    } else if (e instanceof ConflictError) {
      res.status(409).json({ error: { code: 'CONFLICT', message: e.message } });
    } else {
      res.status(500).json({ error: { code: 'INTERNAL_ERROR', message: 'Unexpected error' } });
    }
  }
});

// Common status codes:
// 200 OK, 201 Created, 204 No Content
// 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict
// 500 Internal Error, 503 Service Unavailable
```

**Why:** Clients rely on status codes for retry logic, error handling, and caching.

Reference: `references/api-design-patterns.md`

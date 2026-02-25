---
title: Validate All API Request Bodies
impact: HIGH
impactDescription: prevents invalid data from entering the system
tags: api, validation, security
---

## Validate All API Request Bodies

Every API endpoint must validate request bodies with a schema before processing.

**Incorrect:**

```typescript
// No validation - trusts client input
app.post('/users', async (req, res) => {
  const user = await db.users.create(req.body);  // Anything goes!
  res.json(user);
});
```

**Correct:**

```typescript
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
        requestId: req.id,
      },
    });
  }

  const user = await db.users.create(result.data);
  res.status(201).json({ data: user });
});
```

**Why:** Unvalidated input leads to data corruption, security vulnerabilities, and crashes.

Reference: `references/api-design-patterns.md`

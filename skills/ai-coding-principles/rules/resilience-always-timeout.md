---
title: Always Set Timeouts on External Calls
impact: MEDIUM
impactDescription: prevents hanging requests and resource exhaustion
tags: resilience, reliability, external-calls
---

## Always Set Timeouts on External Calls

Every external call (HTTP, database, cache) must have an explicit timeout. Never wait forever.

**Incorrect:**

```typescript
// No timeout - can hang forever if service is slow
const response = await fetch('https://api.example.com/data');
const result = await db.query('SELECT * FROM large_table');
const cached = await redis.get('key');
```

**Correct:**

```typescript
// Explicit timeouts on all external calls
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch('https://api.example.com/data', {
    signal: controller.signal,
  });
  return await response.json();
} catch (error) {
  if (error.name === 'AbortError') {
    throw new TimeoutError('External API timed out after 5s');
  }
  throw error;
} finally {
  clearTimeout(timeoutId);
}

// Database with timeout
await db.query('SELECT * FROM users WHERE id = $1', [id], { timeout: 5000 });

// Timeout guidelines:
// Database queries: 5-30s
// External APIs: 5-10s
// Health checks: 1-2s
```

**Why:** Without timeouts, one slow dependency can exhaust your connection pool and crash your service.

Reference: `references/resilience-patterns.md`

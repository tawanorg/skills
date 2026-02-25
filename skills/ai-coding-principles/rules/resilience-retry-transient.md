---
title: Only Retry Transient Failures
impact: MEDIUM
impactDescription: prevents wasted retries and faster error responses
tags: resilience, reliability, error-handling
---

## Only Retry Transient Failures

Retry network timeouts and 5xx errors. Never retry 4xx client errors - they won't succeed on retry.

**Incorrect:**

```typescript
// Retrying everything - wastes time on permanent failures
async function callApi(url: string) {
  for (let i = 0; i < 3; i++) {
    try {
      return await fetch(url);
    } catch (e) {
      await sleep(1000);  // Retries 400, 401, 404 too!
    }
  }
}
```

**Correct:**

```typescript
// Only retry transient failures
async function callApiWithRetry(url: string) {
  for (let attempt = 1; attempt <= 3; attempt++) {
    try {
      const response = await fetch(url);

      if (response.ok) return response;

      // Don't retry client errors
      if (response.status >= 400 && response.status < 500) {
        throw new ClientError(`Client error: ${response.status}`);
      }

      // Retry server errors
      if (attempt < 3) {
        await sleep(100 * Math.pow(2, attempt));  // Exponential backoff
        continue;
      }
    } catch (error) {
      if (!isRetryable(error) || attempt === 3) throw error;
      await sleep(100 * Math.pow(2, attempt));
    }
  }
}

function isRetryable(error: unknown): boolean {
  // Retry: timeouts, connection errors, 5xx
  // Don't retry: 400, 401, 403, 404, 422
  return error instanceof NetworkError || error instanceof ServerError;
}
```

**Why:** Retrying 4xx errors just delays the inevitable failure and wastes resources.

Reference: `references/resilience-patterns.md`

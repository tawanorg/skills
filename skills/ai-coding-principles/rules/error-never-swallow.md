---
title: Never Swallow Errors Silently
impact: HIGH
impactDescription: prevents hidden failures and debugging nightmares
tags: error-handling, debugging, reliability
---

## Never Swallow Errors Silently

Never catch an error and silently ignore it. Always log, rethrow, or handle meaningfully.

**Incorrect:**

```typescript
// Silent failure - error disappears, debugging impossible
try {
  await saveUser(data);
} catch (e) {
  return null;  // Error swallowed, no trace
}

// Empty catch - pretends nothing happened
try {
  await processPayment(order);
} catch (e) {
  // TODO: handle this later
}
```

**Correct:**

```typescript
// Log with context, then handle appropriately
try {
  await saveUser(data);
} catch (error) {
  logger.error('Failed to save user', {
    userId: data.id,
    error: error.message,
  });
  throw new UserSaveError(`Could not save user ${data.id}`, { cause: error });
}

// If truly ignorable, document why
try {
  await sendAnalytics(event);
} catch (error) {
  // Analytics failures are non-critical, log and continue
  logger.warn('Analytics failed', { event: event.type, error: error.message });
}
```

**Why:** Swallowed errors cause silent data corruption, mysterious bugs, and hours of debugging.

Reference: `references/error-handling-patterns.md`

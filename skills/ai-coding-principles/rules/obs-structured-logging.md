---
title: Use Structured Logging
impact: MEDIUM
impactDescription: enables log searching and aggregation
tags: observability, logging, debugging
---

## Use Structured Logging

Always log structured data (JSON), not concatenated strings. Include consistent fields for searchability.

**Incorrect:**

```typescript
// String concatenation - impossible to query
console.log(`User ${userId} placed order ${orderId} for $${amount}`);
console.log('Error: ' + error.message);
logger.info('Processing started');
```

**Correct:**

```typescript
// Structured JSON - queryable and parseable
logger.info('Order placed', {
  userId,
  orderId,
  amount,
  currency: 'USD',
  itemCount: items.length,
});

logger.error('Payment failed', {
  orderId,
  userId,
  errorCode: error.code,
  errorMessage: error.message,
  requestId: req.id,
});

// Standard fields to always include:
// - requestId (correlation)
// - userId (when available)
// - operation name
// - relevant entity IDs
```

**Why:** Structured logs can be searched (`userId:123`), aggregated, and alerted on. String logs cannot.

Reference: `references/observability-patterns.md`

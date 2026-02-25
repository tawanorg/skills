---
title: Include Request ID in All Logs
impact: MEDIUM
impactDescription: enables request tracing across services
tags: observability, logging, debugging
---

## Include Request ID in All Logs

Every log entry must include a request ID that correlates all logs for a single request.

**Incorrect:**

```typescript
// No correlation - can't trace a request through logs
logger.info('Received request');
logger.info('Fetching user');
logger.info('Processing payment');
logger.info('Request complete');
// Which logs belong together? Impossible to tell.
```

**Correct:**

```typescript
// Request ID correlates all logs for one request
app.use((req, res, next) => {
  req.id = req.headers['x-request-id'] || randomUUID();
  next();
});

logger.info('Received request', { requestId: req.id, path: req.path });
logger.info('Fetching user', { requestId: req.id, userId });
logger.info('Processing payment', { requestId: req.id, orderId });
logger.info('Request complete', { requestId: req.id, durationMs: elapsed });

// Now you can filter: requestId:"abc-123" shows all related logs
```

**Why:** Without request IDs, debugging production issues across concurrent requests is nearly impossible.

Reference: `references/observability-patterns.md`

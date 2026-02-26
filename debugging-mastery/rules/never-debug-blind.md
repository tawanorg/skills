---
title: Never Debug Blind
impact: critical
tags: [debugging, production, observability]
---

# Never Debug Blind

**Don't guess. Get visibility into what's actually happening.**

## The Rule

Before attempting any fix:
1. Get concrete evidence of what's happening
2. See the actual data, state, and flow
3. Never assume—verify

## Why This Matters

```
Blind debugging:
- Based on assumptions
- "I think X is happening"
- Random fixes hoping one works
- Time: Hours to days

Informed debugging:
- Based on evidence
- "I see X is happening"
- Targeted fixes addressing observed issues
- Time: Minutes to hours
```

## Incorrect Approach

```typescript
// Bug report: "Orders not processing"
// Developer: "Hmm, probably the payment service..."

// Attempt 1: Restart payment service
// Still broken

// Attempt 2: Add retry logic
// Still broken

// Attempt 3: Increase timeout
// Still broken

// 2 hours later, still guessing
// Never actually looked at what's happening
```

## Correct Approach

```typescript
// Bug report: "Orders not processing"

// Step 1: Get visibility
const response = await processOrder(order);
console.log('Process order response:', {
  success: response.success,
  error: response.error,
  orderState: order.state,
  timestamp: Date.now()
});

// Step 2: Check logs
// Found: "Error: Invalid currency code: null"

// Step 3: Trace the null
console.log('Order currency:', order.currency);
// Output: undefined

// Step 4: Check data source
const rawOrder = await db.orders.findById(orderId);
console.log('DB order:', rawOrder);
// currency field is missing from database

// Root cause found in minutes: Data migration missed currency field
```

## Visibility Tools

### Logging

```typescript
// Strategic log points
logger.info('Processing order', {
  orderId: order.id,
  step: 'start',
  input: sanitize(order)
});

// ... processing ...

logger.info('Order validation complete', {
  orderId: order.id,
  step: 'validated',
  isValid: result.valid,
  errors: result.errors
});

// ... more processing ...

logger.info('Order processing complete', {
  orderId: order.id,
  step: 'complete',
  success: true,
  duration: Date.now() - startTime
});
```

### Debugger Breakpoints

```typescript
// Set breakpoints at key locations:

function processOrder(order: Order) {
  debugger;  // Inspect order input

  const validated = validateOrder(order);
  debugger;  // Inspect validation result

  const priced = calculatePrice(validated);
  debugger;  // Inspect pricing result

  return submitOrder(priced);
}

// Use debugger to:
// - Inspect variable values
// - Step through execution
// - Evaluate expressions
// - View call stack
```

### Request Tracing

```typescript
// Add correlation ID to trace requests
app.use((req, res, next) => {
  req.traceId = generateTraceId();
  logger.addContext({ traceId: req.traceId });
  next();
});

// Now all logs for this request are connected
// grep "trace-abc-123" logs/*
```

### Metrics and Dashboards

```
What to monitor:
- Request rate
- Error rate
- Latency percentiles
- Queue depths
- Resource usage

When debugging:
- Check metrics around incident time
- Look for anomalies
- Correlate with deployments
```

## The Evidence Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                    EVIDENCE QUALITY                         │
├─────────────────────────────────────────────────────────────┤
│ BEST:    │ Debugger stepping through exact execution        │
│          │ Actual data values at each step                  │
├──────────┼──────────────────────────────────────────────────┤
│ GOOD:    │ Detailed logs with context                       │
│          │ Request/response traces                          │
├──────────┼──────────────────────────────────────────────────┤
│ OKAY:    │ Error messages and stack traces                  │
│          │ Metrics showing anomalies                        │
├──────────┼──────────────────────────────────────────────────┤
│ POOR:    │ User reports (subjective)                        │
│          │ "It doesn't work" (vague)                        │
├──────────┼──────────────────────────────────────────────────┤
│ WORST:   │ Assumptions and guesses                          │
│          │ "I think it's probably..."                       │
└──────────┴──────────────────────────────────────────────────┘

Always try to move up the hierarchy before fixing.
```

## Questions to Get Visibility

```markdown
Before fixing, I need to see:
- [ ] What is the actual error message/behavior?
- [ ] What is the actual input data?
- [ ] What is the actual state at failure point?
- [ ] What is the actual response from dependencies?
- [ ] What do the logs show around the time of failure?
```

## Production Debugging Visibility

```typescript
// When you can't use debugger in production:

// 1. Enhanced error capture
try {
  await processOrder(order);
} catch (error) {
  errorTracker.capture(error, {
    context: {
      orderId: order.id,
      userId: user.id,
      orderState: JSON.stringify(order),
      timestamp: Date.now()
    }
  });
  throw error;
}

// 2. Temporary verbose logging
if (featureFlags.isEnabled('debug_order_processing')) {
  logger.debug('Order processing debug', {
    fullOrder: order,
    userContext: user,
    systemState: getSystemState()
  });
}

// 3. Request replay
// Capture and replay failing requests in staging
await debugQueue.push({
  request: req.body,
  headers: req.headers,
  timestamp: Date.now()
});
```

## The "Works on My Machine" Visibility

```markdown
When bug only happens in production:

1. What's different?
   - [ ] Environment variables
   - [ ] Data volume
   - [ ] Network conditions
   - [ ] User permissions
   - [ ] Concurrent load

2. How to get visibility?
   - [ ] Add temporary logging
   - [ ] Capture state snapshots
   - [ ] Replicate production data locally
   - [ ] Use staging environment
```

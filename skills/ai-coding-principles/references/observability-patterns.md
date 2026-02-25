# Observability Patterns

Structured logging, metrics, and distributed tracing for production systems.

## The Three Pillars

| Pillar | Purpose | Question It Answers |
|--------|---------|---------------------|
| **Logs** | Discrete events | What happened? |
| **Metrics** | Aggregated measurements | How is the system performing? |
| **Traces** | Request flow across services | Where did time go? |

## Structured Logging

### Always Use Structured Logs

```typescript
// BAD: Unstructured string logs
console.log(`User ${userId} placed order ${orderId} for $${amount}`);

// GOOD: Structured JSON logs
logger.info('Order placed', {
  userId,
  orderId,
  amount,
  currency: 'USD',
  itemCount: items.length,
});
```

### Log Levels

| Level | Use For | Example |
|-------|---------|---------|
| `error` | Failures requiring attention | Database connection failed |
| `warn` | Degraded but working | Retry succeeded after failure |
| `info` | Business events | Order placed, user signed up |
| `debug` | Diagnostic detail | Cache hit/miss, query timing |

### Standard Log Fields

```typescript
interface LogContext {
  // Always include
  requestId: string;      // Correlation ID
  timestamp: string;      // ISO 8601

  // Include when available
  userId?: string;
  tenantId?: string;
  traceId?: string;
  spanId?: string;

  // Operation context
  operation?: string;     // 'user.create', 'order.process'
  durationMs?: number;
}
```

### Request Context Pattern

```typescript
import { AsyncLocalStorage } from 'async_hooks';

interface RequestContext {
  requestId: string;
  userId?: string;
  traceId: string;
}

const contextStorage = new AsyncLocalStorage<RequestContext>();

// Middleware
app.use((req, res, next) => {
  const context: RequestContext = {
    requestId: req.headers['x-request-id'] || randomUUID(),
    traceId: req.headers['x-trace-id'] || randomUUID(),
    userId: req.user?.id,
  };

  contextStorage.run(context, () => next());
});

// Logger automatically includes context
class Logger {
  info(message: string, data?: object) {
    const context = contextStorage.getStore();
    console.log(JSON.stringify({
      level: 'info',
      message,
      ...context,
      ...data,
      timestamp: new Date().toISOString(),
    }));
  }
}
```

## Metrics

### Four Golden Signals

```typescript
// 1. Latency - How long requests take
const requestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request latency',
  labelNames: ['method', 'route', 'status'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
});

// 2. Traffic - How much demand
const requestCount = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status'],
});

// 3. Errors - Rate of failures
const errorCount = new Counter({
  name: 'http_errors_total',
  help: 'Total HTTP errors',
  labelNames: ['method', 'route', 'error_type'],
});

// 4. Saturation - How full the system is
const activeConnections = new Gauge({
  name: 'db_connections_active',
  help: 'Active database connections',
});
```

### Business Metrics

```typescript
// Track business events, not just technical
const ordersPlaced = new Counter({
  name: 'orders_placed_total',
  help: 'Total orders placed',
  labelNames: ['plan_type', 'country'],
});

const orderValue = new Histogram({
  name: 'order_value_dollars',
  help: 'Order value distribution',
  buckets: [10, 50, 100, 500, 1000, 5000],
});

// Usage
ordersPlaced.inc({ plan_type: 'premium', country: 'US' });
orderValue.observe(order.total);
```

### Metric Naming Convention

```
<namespace>_<name>_<unit>

Examples:
http_request_duration_seconds
db_query_duration_milliseconds
queue_messages_total
cache_hits_total
memory_usage_bytes
```

## Distributed Tracing

### Trace Propagation

```typescript
// Extract trace context from incoming request
const traceContext = {
  traceId: req.headers['x-trace-id'] || generateTraceId(),
  spanId: generateSpanId(),
  parentSpanId: req.headers['x-span-id'],
};

// Propagate to outgoing requests
async function callExternalService(data: any) {
  const span = tracer.startSpan('external-service-call');

  try {
    const response = await fetch('https://api.example.com', {
      headers: {
        'x-trace-id': traceContext.traceId,
        'x-span-id': span.spanId,
        'x-parent-span-id': traceContext.spanId,
      },
      body: JSON.stringify(data),
    });

    span.setStatus('ok');
    return response;
  } catch (error) {
    span.setStatus('error');
    span.recordException(error);
    throw error;
  } finally {
    span.end();
  }
}
```

### Span Attributes

```typescript
span.setAttributes({
  // Standard attributes
  'http.method': 'POST',
  'http.url': '/api/orders',
  'http.status_code': 200,

  // Custom business attributes
  'order.id': orderId,
  'order.item_count': items.length,
  'order.total': total,
  'user.plan': user.plan,
});
```

## Health Checks

### Liveness vs Readiness

```typescript
// Liveness: Is the process running?
app.get('/health/live', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

// Readiness: Can it accept traffic?
app.get('/health/ready', async (req, res) => {
  const checks = await Promise.allSettled([
    checkDatabase(),
    checkCache(),
    checkExternalApi(),
  ]);

  const results = {
    database: checks[0].status === 'fulfilled',
    cache: checks[1].status === 'fulfilled',
    externalApi: checks[2].status === 'fulfilled',
  };

  const allHealthy = Object.values(results).every(Boolean);

  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'ready' : 'degraded',
    checks: results,
  });
});
```

## Alerting Guidelines

### What to Alert On

| Alert | Threshold | Severity |
|-------|-----------|----------|
| Error rate | > 1% for 5 min | Critical |
| P99 latency | > 2s for 5 min | Warning |
| Disk usage | > 80% | Warning |
| Disk usage | > 95% | Critical |
| Queue depth | > 1000 for 10 min | Warning |

### Alert Quality

```yaml
# BAD: Vague, no actionable info
alert: DatabaseErrors
message: "Database errors detected"

# GOOD: Specific, actionable
alert: DatabaseConnectionPoolExhausted
message: "Database connection pool at 95% capacity"
runbook: "https://wiki/runbooks/database-pool-exhausted"
dashboard: "https://grafana/d/database-health"
```

## Observability Checklist

### For Every Service

- [ ] Structured JSON logging configured
- [ ] Request ID propagated through all logs
- [ ] Four golden signals exposed as metrics
- [ ] Health check endpoints implemented
- [ ] Trace context propagated to downstream services

### For Every Request Handler

- [ ] Log at start and end of operation
- [ ] Include duration in completion log
- [ ] Record relevant business metrics
- [ ] Add span attributes for debugging

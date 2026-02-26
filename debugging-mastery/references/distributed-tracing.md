# Distributed Tracing

Following requests across services in distributed systems.

## Why Distributed Tracing?

```
Monolith debugging:
┌─────────────────────────┐
│   Single Application    │  ← One log, one debugger
└─────────────────────────┘

Microservices debugging:
┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐
│Gateway │──►│ Orders │──►│Payments│──►│Notifier│
└────────┘   └────────┘   └────────┘   └────────┘
                  │
                  ▼
             ┌────────┐
             │Inventory│
             └────────┘

One request touches 5 services.
Which one is slow? Which one failed?
Need: A way to follow the request.
```

## Core Concepts

### Traces, Spans, and Context

```
Trace: The entire journey of a request
└── Span: A single operation within the trace
    └── Context: Metadata passed between spans

                         Trace ID: abc-123
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  ┌──────────────────────┐                                      │
│  │ Span: API Gateway    │ (25ms)                               │
│  │ span_id: span-1      │                                      │
│  └──────────────────────┘                                      │
│         │                                                      │
│         ▼                                                      │
│  ┌──────────────────────────────────────────────┐              │
│  │ Span: Order Service                          │ (150ms)      │
│  │ span_id: span-2, parent: span-1              │              │
│  └──────────────────────────────────────────────┘              │
│         │                   │                                  │
│         ▼                   ▼                                  │
│  ┌─────────────────┐ ┌─────────────────┐                       │
│  │Span: Inventory  │ │Span: Database   │                       │
│  │(45ms)           │ │(80ms)           │                       │
│  └─────────────────┘ └─────────────────┘                       │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Context Propagation

```typescript
// How context flows between services

// Service A: Start trace
const traceId = generateTraceId();
const spanId = generateSpanId();

// Pass in headers
const response = await fetch('http://service-b/api', {
  headers: {
    'x-trace-id': traceId,
    'x-span-id': spanId,
    'x-parent-span-id': parentSpanId
  }
});

// Service B: Continue trace
app.use((req, res, next) => {
  const traceId = req.headers['x-trace-id'] || generateTraceId();
  const parentSpanId = req.headers['x-span-id'];
  const spanId = generateSpanId();

  // Store in context for this request
  req.trace = { traceId, spanId, parentSpanId };
  next();
});
```

## Implementing Tracing

### Basic Implementation

```typescript
// Simple tracing middleware
interface Span {
  traceId: string;
  spanId: string;
  parentSpanId?: string;
  operationName: string;
  startTime: number;
  endTime?: number;
  tags: Record<string, string>;
  logs: Array<{ timestamp: number; message: string }>;
}

class Tracer {
  private spans: Span[] = [];

  startSpan(operationName: string, parentSpan?: Span): Span {
    const span: Span = {
      traceId: parentSpan?.traceId || generateId(),
      spanId: generateId(),
      parentSpanId: parentSpan?.spanId,
      operationName,
      startTime: Date.now(),
      tags: {},
      logs: []
    };
    this.spans.push(span);
    return span;
  }

  finishSpan(span: Span) {
    span.endTime = Date.now();
    this.reportSpan(span);
  }

  private reportSpan(span: Span) {
    // Send to tracing backend (Jaeger, Zipkin, etc.)
    console.log('Span:', {
      traceId: span.traceId,
      spanId: span.spanId,
      operation: span.operationName,
      duration: span.endTime! - span.startTime,
      tags: span.tags
    });
  }
}

// Usage
const tracer = new Tracer();

app.get('/orders/:id', async (req, res) => {
  const span = tracer.startSpan('GET /orders/:id');
  span.tags['http.method'] = 'GET';
  span.tags['http.url'] = req.url;

  try {
    const order = await getOrder(req.params.id, span);
    span.tags['http.status_code'] = '200';
    res.json(order);
  } catch (error) {
    span.tags['http.status_code'] = '500';
    span.tags['error'] = 'true';
    span.logs.push({ timestamp: Date.now(), message: error.message });
    res.status(500).json({ error: error.message });
  } finally {
    tracer.finishSpan(span);
  }
});

async function getOrder(id: string, parentSpan: Span) {
  const span = tracer.startSpan('getOrder', parentSpan);
  try {
    const order = await db.orders.findById(id);
    return order;
  } finally {
    tracer.finishSpan(span);
  }
}
```

### OpenTelemetry Implementation

```typescript
// Using OpenTelemetry (industry standard)
import { trace, context, SpanStatusCode } from '@opentelemetry/api';
import { NodeTracerProvider } from '@opentelemetry/sdk-trace-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';

// Setup
const provider = new NodeTracerProvider();
provider.addSpanProcessor(
  new BatchSpanProcessor(new JaegerExporter({
    endpoint: 'http://jaeger:14268/api/traces'
  }))
);
provider.register();

const tracer = trace.getTracer('order-service');

// Usage
app.get('/orders/:id', async (req, res) => {
  const span = tracer.startSpan('GET /orders/:id', {
    attributes: {
      'http.method': 'GET',
      'http.url': req.url
    }
  });

  try {
    const order = await context.with(
      trace.setSpan(context.active(), span),
      () => getOrder(req.params.id)
    );
    span.setStatus({ code: SpanStatusCode.OK });
    res.json(order);
  } catch (error) {
    span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
    span.recordException(error);
    res.status(500).json({ error: error.message });
  } finally {
    span.end();
  }
});
```

## Reading Traces

### Trace Visualization

```
Trace: abc-123 (Order Checkout)
Duration: 523ms

├── Gateway (25ms)
│   └── Auth middleware (10ms)
│
├── Order Service (450ms)            ← Slowest service
│   ├── Validate order (15ms)
│   ├── Check inventory (120ms)     ← Slow operation
│   │   └── DB query (115ms)        ← Root cause: slow query
│   ├── Calculate pricing (45ms)
│   └── Save order (270ms)          ← Another slow operation
│       └── DB write (265ms)        ← Root cause: slow write
│
└── Notification Service (48ms)
    └── Send email (45ms)

Insight: Order Service DB operations are slow
Action: Investigate database performance
```

### Finding Problems in Traces

```markdown
## What to Look For

### Latency Issues
- Which span took longest?
- What percentage of total time?
- Is it consistently slow or variable?

### Error Patterns
- Where do errors originate?
- Are they propagated correctly?
- What's the error rate per service?

### Dependency Issues
- Which downstream services are slow?
- Are there timeout patterns?
- Are retries happening?

### Concurrency Patterns
- Are operations running in parallel when they could?
- Are operations serialized unnecessarily?
```

## Tracing Best Practices

### What to Trace

```markdown
## Always Trace
- [ ] Incoming HTTP requests
- [ ] Outgoing HTTP requests
- [ ] Database queries
- [ ] Cache operations
- [ ] Message queue publish/consume
- [ ] External API calls

## Useful Tags
- http.method, http.url, http.status_code
- db.type, db.statement (sanitized)
- error, error.message
- user.id (if safe)
- cache.hit, cache.key
```

### Sensitive Data

```typescript
// Don't trace sensitive data

// Bad: Exposes credentials
span.setTag('db.statement',
  `SELECT * FROM users WHERE password = '${password}'`);

// Good: Sanitized
span.setTag('db.statement',
  'SELECT * FROM users WHERE password = ?');

// Bad: Exposes PII
span.setTag('user.email', user.email);

// Good: Use IDs
span.setTag('user.id', user.id);
```

### Sampling

```typescript
// Don't trace everything in production

const sampler = {
  shouldSample(traceId: string): boolean {
    // Sample 10% of traces
    return parseInt(traceId.slice(-2), 16) < 26;  // ~10%
  }
};

// Always sample errors
if (isError) {
  forceTraceSample();
}

// Always sample slow requests
if (duration > 1000) {
  forceTraceSample();
}
```

## Debugging with Traces

### The Tracing Debug Flow

```
1. Get trace ID from error/log
   └── Error: "Order failed" [trace_id: abc-123]

2. Look up trace in tracing UI
   └── Jaeger/Zipkin/Datadog/etc.

3. Find the error span
   └── Red/error-marked span

4. Check span details
   └── Tags, logs, exceptions

5. Follow the call chain
   └── What called this? What did it call?

6. Find root cause
   └── First error in the chain
```

### Correlating with Logs

```typescript
// Include trace ID in all logs
logger.info('Processing order', {
  traceId: span.context().traceId,
  spanId: span.context().spanId,
  orderId: order.id
});

// Now you can correlate:
// 1. Find error in logs with trace ID
// 2. Look up trace
// 3. Find related logs with same trace ID
```

### Correlating with Metrics

```typescript
// Tag metrics with trace info for correlation
metrics.histogram('order.processing.duration', duration, {
  service: 'order-service',
  operation: 'processOrder',
  // Sample trace ID for drilling down
  sample_trace_id: span.context().traceId
});
```

## Tracing Infrastructure

### Common Tools

| Tool | Type | Best For |
|------|------|----------|
| Jaeger | Open source | Self-hosted, Kubernetes |
| Zipkin | Open source | Simple setup |
| Datadog APM | SaaS | Full observability platform |
| AWS X-Ray | SaaS | AWS environments |
| Honeycomb | SaaS | High-cardinality analysis |
| Grafana Tempo | Open source | Grafana ecosystem |

### Architecture

```
┌─────────┐   ┌─────────┐   ┌─────────┐
│Service A│   │Service B│   │Service C│
└────┬────┘   └────┬────┘   └────┬────┘
     │             │             │
     └──────────┬──┴─────────────┘
                │
         ┌──────▼──────┐
         │   Collector  │
         │ (OpenTelemetry│
         │   Collector)  │
         └──────┬───────┘
                │
         ┌──────▼──────┐
         │   Storage    │
         │ (Jaeger,     │
         │  Elasticsearch)│
         └──────┬───────┘
                │
         ┌──────▼──────┐
         │   Query UI   │
         │ (Jaeger UI)  │
         └─────────────┘
```

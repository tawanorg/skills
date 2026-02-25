# Resilience Patterns

Circuit breakers, retries, and graceful degradation for distributed systems.

## Core Principle

**Everything fails. Design for failure.**

| Reality | Design Response |
|---------|-----------------|
| Networks are unreliable | Retry with backoff |
| Services go down | Circuit breakers |
| Dependencies are slow | Timeouts |
| Errors cascade | Bulkheads |
| Traffic spikes | Rate limiting |

## Retry Pattern

### When to Retry

| Retry | Don't Retry |
|-------|-------------|
| Network timeout | 400 Bad Request |
| 503 Service Unavailable | 401 Unauthorized |
| Connection refused | 404 Not Found |
| DNS resolution failure | 422 Validation Error |

### Exponential Backoff

```typescript
interface RetryConfig {
  maxAttempts: number;
  baseDelayMs: number;
  maxDelayMs: number;
  jitter: boolean;
}

async function withRetry<T>(
  fn: () => Promise<T>,
  config: RetryConfig = {
    maxAttempts: 3,
    baseDelayMs: 100,
    maxDelayMs: 10000,
    jitter: true,
  }
): Promise<T> {
  let lastError: Error;

  for (let attempt = 1; attempt <= config.maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;

      if (!isRetryable(error) || attempt === config.maxAttempts) {
        throw error;
      }

      const delay = calculateDelay(attempt, config);
      logger.warn('Retry attempt', { attempt, delay, error: lastError.message });
      await sleep(delay);
    }
  }

  throw lastError!;
}

function calculateDelay(attempt: number, config: RetryConfig): number {
  // Exponential: 100ms, 200ms, 400ms, 800ms...
  let delay = config.baseDelayMs * Math.pow(2, attempt - 1);
  delay = Math.min(delay, config.maxDelayMs);

  // Add jitter to prevent thundering herd
  if (config.jitter) {
    delay = delay * (0.5 + Math.random());
  }

  return delay;
}

function isRetryable(error: unknown): boolean {
  if (error instanceof NetworkError) return true;
  if (error instanceof HttpError) {
    return [408, 429, 500, 502, 503, 504].includes(error.statusCode);
  }
  return false;
}
```

## Circuit Breaker

### States

```
     ┌──────────────┐
     │    CLOSED    │ ← Normal operation
     │  (allowing)  │
     └──────┬───────┘
            │ failure threshold reached
            ▼
     ┌──────────────┐
     │     OPEN     │ ← Failing fast
     │  (blocking)  │
     └──────┬───────┘
            │ timeout elapsed
            ▼
     ┌──────────────┐
     │  HALF-OPEN   │ ← Testing recovery
     │  (testing)   │
     └──────┬───────┘
            │ success → CLOSED
            │ failure → OPEN
```

### Implementation

```typescript
enum CircuitState {
  CLOSED = 'CLOSED',
  OPEN = 'OPEN',
  HALF_OPEN = 'HALF_OPEN',
}

class CircuitBreaker {
  private state = CircuitState.CLOSED;
  private failureCount = 0;
  private lastFailureTime?: Date;
  private successCount = 0;

  constructor(
    private readonly name: string,
    private readonly options = {
      failureThreshold: 5,
      resetTimeoutMs: 30000,
      halfOpenSuccessThreshold: 3,
    }
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (this.shouldAttemptReset()) {
        this.state = CircuitState.HALF_OPEN;
      } else {
        throw new CircuitOpenError(this.name);
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    if (this.state === CircuitState.HALF_OPEN) {
      this.successCount++;
      if (this.successCount >= this.options.halfOpenSuccessThreshold) {
        this.reset();
      }
    } else {
      this.failureCount = 0;
    }
  }

  private onFailure() {
    this.failureCount++;
    this.lastFailureTime = new Date();

    if (this.state === CircuitState.HALF_OPEN) {
      this.state = CircuitState.OPEN;
    } else if (this.failureCount >= this.options.failureThreshold) {
      this.state = CircuitState.OPEN;
      logger.warn('Circuit opened', { name: this.name, failures: this.failureCount });
    }
  }

  private shouldAttemptReset(): boolean {
    if (!this.lastFailureTime) return true;
    const elapsed = Date.now() - this.lastFailureTime.getTime();
    return elapsed >= this.options.resetTimeoutMs;
  }

  private reset() {
    this.state = CircuitState.CLOSED;
    this.failureCount = 0;
    this.successCount = 0;
    logger.info('Circuit closed', { name: this.name });
  }
}

// Usage
const paymentCircuit = new CircuitBreaker('payment-service');

async function processPayment(order: Order) {
  return paymentCircuit.execute(() =>
    paymentClient.charge(order.total)
  );
}
```

## Timeout Pattern

### Always Set Timeouts

```typescript
// BAD: No timeout - can hang forever
const response = await fetch('https://api.example.com/data');

// GOOD: Explicit timeout
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch('https://api.example.com/data', {
    signal: controller.signal,
  });
  return await response.json();
} finally {
  clearTimeout(timeoutId);
}

// Helper
async function fetchWithTimeout(url: string, timeoutMs: number) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);

  try {
    const response = await fetch(url, { signal: controller.signal });
    return response;
  } catch (error) {
    if (error.name === 'AbortError') {
      throw new TimeoutError(`Request to ${url} timed out after ${timeoutMs}ms`);
    }
    throw error;
  } finally {
    clearTimeout(timeoutId);
  }
}
```

### Timeout Guidelines

| Operation | Suggested Timeout |
|-----------|-------------------|
| Database query | 5-30 seconds |
| External API | 5-10 seconds |
| Health check | 1-2 seconds |
| File upload | Based on size |
| Background job | Task-specific |

## Bulkhead Pattern

### Isolate Failures

```typescript
// Separate thread pools/connections for different operations
class BulkheadedService {
  private criticalPool = new Pool({ max: 20 });
  private backgroundPool = new Pool({ max: 5 });

  // Critical operations get dedicated resources
  async processPayment(order: Order) {
    const conn = await this.criticalPool.acquire();
    try {
      return await this.paymentService.process(order, conn);
    } finally {
      this.criticalPool.release(conn);
    }
  }

  // Background operations can't starve critical ones
  async sendAnalytics(event: AnalyticsEvent) {
    const conn = await this.backgroundPool.acquire();
    try {
      return await this.analyticsService.send(event, conn);
    } finally {
      this.backgroundPool.release(conn);
    }
  }
}
```

## Graceful Degradation

### Fallback Strategies

```typescript
class ProductService {
  constructor(
    private primaryDb: Database,
    private cache: Cache,
    private searchService: SearchService
  ) {}

  async getProducts(query: string): Promise<Product[]> {
    // Try primary source
    try {
      return await this.searchService.search(query);
    } catch (error) {
      logger.warn('Search service unavailable, falling back to cache', { error });
    }

    // Fallback to cache
    try {
      const cached = await this.cache.get(`products:${query}`);
      if (cached) return cached;
    } catch (error) {
      logger.warn('Cache unavailable', { error });
    }

    // Last resort: direct DB query (slower but works)
    try {
      return await this.primaryDb.products.search(query);
    } catch (error) {
      logger.error('All product sources failed', { error });
      throw new ServiceDegradedError('Unable to fetch products');
    }
  }
}
```

### Feature Flags for Degradation

```typescript
async function checkout(cart: Cart) {
  const order = await createOrder(cart);

  // Non-critical: recommendations
  if (featureFlags.isEnabled('checkout-recommendations')) {
    try {
      order.recommendations = await getRecommendations(cart);
    } catch {
      // Silent failure - recommendations are nice-to-have
      order.recommendations = [];
    }
  }

  // Critical: payment - must succeed or fail explicitly
  await processPayment(order);

  // Non-critical: analytics
  sendAnalytics('order_completed', order).catch(() => {
    // Fire and forget
  });

  return order;
}
```

## Health Checks

### Dependency Health

```typescript
interface HealthStatus {
  status: 'healthy' | 'degraded' | 'unhealthy';
  checks: Record<string, {
    status: 'pass' | 'fail';
    latencyMs?: number;
    message?: string;
  }>;
}

async function checkHealth(): Promise<HealthStatus> {
  const checks = await Promise.allSettled([
    checkDatabase(),
    checkCache(),
    checkPaymentService(),
  ]);

  const results = {
    database: formatCheck(checks[0]),
    cache: formatCheck(checks[1]),
    payment: formatCheck(checks[2]),
  };

  const criticalFailed = results.database.status === 'fail';
  const anyFailed = Object.values(results).some(c => c.status === 'fail');

  return {
    status: criticalFailed ? 'unhealthy' : anyFailed ? 'degraded' : 'healthy',
    checks: results,
  };
}
```

## Resilience Checklist

### For Every External Call

- [ ] Timeout configured
- [ ] Retries with backoff for transient failures
- [ ] Circuit breaker for repeated failures
- [ ] Fallback behavior defined

### For Every Service

- [ ] Health check endpoint
- [ ] Graceful shutdown handling
- [ ] Resource limits (connections, memory)
- [ ] Monitoring and alerting

### Testing Resilience

- [ ] Test timeout behavior
- [ ] Test retry exhaustion
- [ ] Test circuit breaker transitions
- [ ] Test degraded mode operation
- [ ] Chaos engineering in staging

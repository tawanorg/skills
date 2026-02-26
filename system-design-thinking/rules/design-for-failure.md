---
title: Design for Failure
impact: HIGH
impactDescription: builds resilient systems that degrade gracefully
tags: reliability, resilience, failure, distributed
---

## Design for Failure

Assume everything fails. Design systems that continue working (possibly degraded) when components fail.

**Context:** Designing distributed systems or any system with external dependencies.

**Heuristic:** For every component and dependency, answer: "What happens when this fails?"

**Everything fails:**

| Component | How It Fails |
|-----------|--------------|
| Networks | Latency, packet loss, partitions |
| Services | Crashes, overload, bugs |
| Databases | Connections exhausted, disk full, corruption |
| Cloud providers | Region outages, API throttling |
| Third-party APIs | Rate limits, downtime, breaking changes |

**Failure handling patterns:**

| Pattern | Purpose |
|---------|---------|
| **Timeouts** | Don't wait forever |
| **Retries** | Handle transient failures |
| **Circuit breakers** | Stop cascading failures |
| **Bulkheads** | Isolate failures |
| **Fallbacks** | Degrade gracefully |
| **Health checks** | Detect failures fast |

**Apply when:**
- Designing any distributed system
- Integrating with external services
- Building critical paths

**Skip when:**
- Building simple scripts or tools
- Single-machine, single-process apps

**Why:** In distributed systems, failure is not exceptional—it's normal. Systems that assume success will cascade failures and cause outages.

**Example:**

```
❌ Assuming the payment provider is always available:
   async function checkout(order) {
     const payment = await paymentProvider.charge(order.total);
     return { success: true, payment };
   }

✅ Designing for payment provider failure:
   async function checkout(order) {
     try {
       const payment = await withTimeout(
         withRetry(() => paymentProvider.charge(order.total)),
         5000
       );
       return { success: true, payment };
     } catch (error) {
       // Queue for retry, notify user
       await paymentQueue.add({ orderId: order.id, amount: order.total });
       return {
         success: false,
         message: 'Payment processing delayed. We will notify you.'
       };
     }
   }
```

**Questions to ask:**

- What if this service is unavailable for 1 hour?
- What if response time goes from 100ms to 10s?
- What if this returns corrupt data?
- What's the blast radius of this failure?

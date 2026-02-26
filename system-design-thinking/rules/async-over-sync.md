---
title: Async Over Sync
impact: MEDIUM
impactDescription: improves resilience and scalability
tags: integration, async, messaging, resilience
---

## Async Over Sync

Default to asynchronous communication between services. Use synchronous only when you need immediate responses.

**Context:** Choosing how services communicate with each other.

**Heuristic:** Ask "Does the caller need the result immediately?"

```
Need immediate result? ──► Sync (REST/gRPC)
         │
         No
         │
         ▼
Can operation be queued? ──► Async (Queue/Event)
         │
         No
         │
         ▼
Async with callback/webhook
```

**Comparison:**

| Synchronous | Asynchronous |
|-------------|--------------|
| Request-response | Fire-and-forget |
| Caller waits | Caller continues |
| Tight coupling | Loose coupling |
| Cascading failures | Isolated failures |
| Simple to reason about | More complex flow |
| Immediate consistency | Eventual consistency |

**When to use sync:**

- User-facing requests needing immediate response
- Simple queries (get user by ID)
- Operations requiring immediate confirmation
- Within a single bounded context

**When to use async:**

- Cross-service communication
- Operations that can be retried
- Notifications and side effects
- High-volume processing
- When services have different availability

**Apply when:**
- Designing inter-service communication
- Building event-driven systems
- Improving resilience of existing systems

**Skip when:**
- Communication within a single service
- Simple CRUD applications
- Prototypes where simplicity matters more

**Why:** Synchronous calls create temporal coupling—if the downstream service is slow or down, the upstream service is affected. Async breaks this chain.

**Example:**

```
❌ Synchronous order flow (fragile):

   User ──► Order Service ──► Payment Service ──► Inventory Service ──► Email Service

   If Email Service is slow, entire checkout is slow.
   If Inventory Service is down, checkout fails.

✅ Asynchronous order flow (resilient):

   User ──► Order Service ──► [Payment - sync, needs confirmation]
                │
                └──► Order Created Event
                         │
                         ├──► Inventory Service (async)
                         ├──► Email Service (async)
                         └──► Analytics Service (async)

   Payment is sync (user needs to know if it worked).
   Everything else is async (can retry, doesn't block user).
```

**Async patterns:**

| Pattern | Use Case |
|---------|----------|
| Message queue | Work distribution, load leveling |
| Pub/sub | Multiple consumers, broadcasting |
| Event sourcing | Audit trail, replay capability |
| Saga | Distributed transactions |

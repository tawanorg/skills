---
title: Embrace Async and Queues
impact: HIGH
impactDescription: decouples producers from consumers, absorbs spikes, and isolates failures
tags: scale, distributed, async, messaging, architecture
---

## Embrace Async and Queues

If the caller doesn't need the result right now, don't make it wait. Move slow,
spiky, or failure-prone work behind a queue so the request path stays fast and
the system absorbs load instead of collapsing under it.

**Context:** Handling work that is slow (image/video processing), spiky (campaign sends), or talks to flaky dependencies (third-party APIs).

**Heuristic:** Return quickly after enqueuing the work; process it asynchronously. Use queues to smooth bursts (buffer), decouple services, and retry failures with a dead-letter queue for poison messages.

```
Sync (coupled):     Client ─► API ─► [do slow work now] ─► response   (latency + cascading failure)

Async (decoupled):  Client ─► API ─► enqueue ─► 202 Accepted
                                         │
                                         ▼
                                     Worker pool ─► process ─► (retry / DLQ on failure)
```

**Use async when:** the result isn't needed in the response, work is expensive or bursty, or you want to isolate a flaky dependency.
**Keep it sync when:** the caller genuinely needs the result to continue (e.g. a read, or a payment authorization the user is waiting on).

**Watch for:** consumers must be **idempotent** (queues deliver at-least-once), and you trade immediate consistency for eventual consistency — design the UX for it.

**Why:** A queue turns a traffic spike into a longer processing tail instead of an outage, and stops one slow downstream from blocking user-facing requests.

Reference: `references/scalability-and-distributed-systems.md`, `references/software-architecture.md`

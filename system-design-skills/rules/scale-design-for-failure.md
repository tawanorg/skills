---
title: Design for Failure
impact: CRITICAL
impactDescription: in distributed systems failure is normal, not exceptional
tags: scale, distributed, resilience, reliability
---

## Design for Failure

Every network call, dependency, and machine will eventually fail. Design so that
a failed dependency degrades the system gracefully instead of cascading into an
outage.

**Context:** Any service that calls another service, database, or third-party API.

**Heuristic:** For every remote call, bound it (timeout), retry transient failures (backoff + jitter), stop calling a sick dependency (circuit breaker), and have a fallback. Ask: "what happens when this is slow or down?"

| Pattern | Purpose |
|---------|---------|
| Timeout | Never wait forever; free the thread/connection |
| Retry w/ backoff + jitter | Ride out transient blips without synchronized retries |
| Circuit breaker | Stop hammering a failing dependency; fail fast |
| Bulkhead | Isolate pools so one slow dependency can't exhaust all threads |
| Fallback / graceful degradation | Serve cached/partial/default results when possible |

**Incorrect:**

```text
result = await downstream.call()   # no timeout: one slow dependency blocks every worker
```

**Correct:**

```text
result = await with_timeout(
    with_retry(downstream.call, backoff=exponential, jitter=True),
    seconds=2,
)  # on breaker-open or failure → fallback (cached value, queued retry, degraded response)
```

**Why:** A retry without a timeout, or a call without a breaker, is how a single dependency's slowness turns into a full-system thread exhaustion and outage. Resilience patterns contain the blast radius.

Reference: `references/scalability-and-distributed-systems.md`

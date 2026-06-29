---
title: Instrument Everything
impact: HIGH
impactDescription: you cannot scale, debug, or operate what you cannot observe
tags: operations, observability, monitoring, reliability
---

## Instrument Everything

Build observability in from the start: structured logs, metrics, and distributed
traces. In a distributed system, a problem you can't see is a problem you can't
fix at 3 AM.

**Context:** Any service running in production, especially across multiple instances or services.

**Heuristic:** Emit the three pillars — logs (what happened), metrics (how the system behaves over time), traces (the path of a request across services). Propagate a request/correlation ID through every hop. Alert on user-facing symptoms (golden signals), not internal causes.

| Pillar | Answers | Examples |
|--------|---------|----------|
| Logs | "What happened in this event?" | Structured JSON with request ID, level, context |
| Metrics | "How is the system trending?" | Latency p50/p95/p99, error rate, throughput, saturation |
| Traces | "Where did this request spend time?" | Span per service hop, tied by trace ID |

**The four golden signals to watch:** latency, traffic, errors, saturation.

**Incorrect:**

```text
print("error")                      # unstructured, no context, no request id
```

**Correct:**

```text
log.error("payment_failed", request_id=rid, user_id=uid, provider="stripe", code=err.code)
metrics.increment("payments.failed", tags={"provider": "stripe"})
# span recorded under trace rid so the cross-service path is reconstructable
```

**Why:** Latency and failures in distributed systems emerge from interactions you can't reproduce locally. Correlated logs, metrics, and traces are the difference between a five-minute diagnosis and an hours-long outage — and they're far cheaper to add before the incident than during it.

Reference: `references/devops-and-cicd.md`, `references/scalability-and-distributed-systems.md`

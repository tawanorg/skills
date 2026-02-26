---
title: Consider Operations
impact: MEDIUM
impactDescription: builds systems that can be deployed, monitored, and maintained
tags: operations, devops, observability, deployment
---

## Consider Operations

Design for the people who will run the system, not just the people who will build it. Operability is a feature.

**Context:** Finalizing system design, before implementation begins.

**Heuristic:** For every system, answer the operations checklist:

| Concern | Questions |
|---------|-----------|
| **Deployment** | How do we release changes? How do we rollback? |
| **Monitoring** | How do we know it's healthy? What metrics matter? |
| **Alerting** | What conditions need human attention? |
| **Debugging** | How do we diagnose problems? Are logs useful? |
| **Scaling** | How do we handle increased load? |
| **Recovery** | What if it fails completely? Backup/restore? |
| **Security** | How do we handle secrets? Access control? |

**The 3 AM Test:**

If this system breaks at 3 AM:
- Can the on-call engineer understand what's wrong from alerts?
- Can they diagnose the issue from logs and metrics?
- Can they fix or mitigate without waking up specialists?
- Can they rollback if a recent change caused it?

**Apply when:**
- Designing any production system
- Before marking a design as "complete"
- Reviewing architecture proposals

**Skip when:**
- Building prototypes or experiments
- Internal tools with no uptime requirements

**Why:** Systems that are hard to operate become abandoned or cause burnout. Operational concerns discovered in production are 10x more expensive to fix.

**Example:**

```
❌ Design that ignores operations:
   "Here's the service architecture. We'll figure out
   deployment and monitoring later."

✅ Design that includes operations:
   "Here's the service architecture. For operations:

   Deployment:
   - Blue-green deployment via Kubernetes
   - Feature flags for gradual rollout
   - Automated rollback on error rate spike

   Monitoring:
   - Health endpoint at /health
   - Metrics: request rate, latency p50/p99, error rate
   - Structured JSON logs with request IDs

   Alerting:
   - Error rate > 1% for 5 minutes
   - p99 latency > 2s for 5 minutes
   - Health check failures

   Recovery:
   - Database backups every 6 hours
   - Point-in-time recovery enabled
   - Runbook for common failure scenarios"
```

**Minimum operability requirements:**

- [ ] Health check endpoint
- [ ] Structured logging with correlation IDs
- [ ] Key metrics exposed (latency, errors, throughput)
- [ ] Deployment can be rolled back in < 5 minutes
- [ ] Alerts for critical conditions
- [ ] Documentation for common issues

---
title: Keep Services Stateless
impact: CRITICAL
impactDescription: stateless services are the precondition for horizontal scaling
tags: scale, distributed, architecture, reliability
---

## Keep Services Stateless

Application servers should hold no client-specific state between requests. Push
session and working state to a shared store so any instance can serve any
request — and you can add or kill instances freely.

**Context:** Designing any service you expect to run on more than one instance behind a load balancer.

**Heuristic:** If handling a request depends on data stored in *this* process's memory from a previous request, externalize it (to a cache, database, or token). Avoid sticky sessions as a scaling crutch.

**Incorrect:**

```text
# session stored in server memory → requires sticky routing;
# scaling out or restarting a node logs users out
sessions[user_id] = { cart: [...] }
```

**Correct:**

```text
# state lives in a shared store or signed token;
# any instance can serve the next request
redis.set(f"session:{sid}", session_data, ttl=1800)
# or: encode claims in a signed JWT the client sends each request
```

**Why:** Stateless instances are interchangeable, which is what makes horizontal scaling, rolling deploys, autoscaling, and instant failover possible. State pinned to a single process turns every scale-out and restart into a user-visible disruption.

Reference: `references/scalability-and-distributed-systems.md`, `references/software-architecture.md`

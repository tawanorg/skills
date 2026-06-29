---
title: Define a TTL and Invalidation Plan
impact: HIGH
impactDescription: caches without an expiry/invalidation story serve stale or wrong data
tags: cache, performance, consistency, invalidation
---

## Define a TTL and Invalidation Plan

Before you add a cache, answer two questions: when does this entry expire, and
what makes it wrong? A cache with no expiry and no invalidation is a bug waiting
to ship stale data.

**Context:** Introducing any cache layer (in-process, Redis, CDN, query cache).

**Heuristic:** Give every cache entry a TTL sized to how stale the data may safely be. For data that must update promptly, also invalidate (or update) the entry on write — don't rely on TTL alone.

| Strategy | Freshness | Use when |
|----------|-----------|----------|
| TTL only | Bounded staleness | Read-mostly, tolerant of slight lag |
| Write-through / update-on-write | Near-immediate | Data changes must reflect quickly |
| Explicit invalidation on event | Near-immediate | Updates are sparse but must propagate |

**Incorrect:**

```text
cache.set("user:42", profile)      # no TTL → stale forever until manually purged
```

**Correct:**

```text
cache.set("user:42", profile, ttl=300)   # bounded staleness
# on profile update:
cache.delete("user:42")                   # or cache.set(...) with fresh value
```

**Why:** The two classic failure modes are stale reads (no invalidation) and unbounded memory growth (no TTL). Deciding the staleness budget up front turns caching from a correctness risk into a controlled performance win.

Reference: `references/caching-and-performance.md`

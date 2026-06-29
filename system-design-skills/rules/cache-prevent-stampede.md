---
title: Prevent Cache Stampede
impact: HIGH
impactDescription: a single expired hot key can flood the database with identical rebuilds
tags: cache, performance, resilience, scalability
---

## Prevent Cache Stampede

When a popular cache entry expires, thousands of concurrent requests can all
miss at once and slam the backend with the same expensive query — a stampede
(a.k.a. thundering herd) that can take the database down.

**Context:** Caching expensive-to-compute values that are read at high concurrency.

**Heuristic:** Don't let every request rebuild on expiry. Use single-flight (one rebuild, others wait), jittered TTLs, and/or probabilistic early refresh.

**Defenses:**

| Technique | How it helps |
|-----------|--------------|
| Single-flight / request coalescing | One request recomputes; others wait for its result |
| Jittered TTL | Spread expirations so keys don't all expire together |
| Early / probabilistic refresh | Refresh slightly before expiry, in the background |
| Stale-while-revalidate | Serve stale value while one worker refreshes |

**Incorrect:**

```text
val = cache.get(key)
if val is None:
    val = expensive_query()   # under load, 10k requests run this at once
    cache.set(key, val, ttl=60)
```

**Correct:**

```text
val = cache.get(key)
if val is None:
    with single_flight(key):          # only one rebuild per key
        val = cache.get(key) or expensive_query()
        cache.set(key, val, ttl=60 + random_jitter())
```

Also defend against **cache penetration** (cache misses for keys that don't
exist) with negative caching or a bloom filter.

**Why:** Caches exist to protect the backend. A stampede inverts that — the cache becomes the trigger for an outage precisely when traffic is highest.

Reference: `references/caching-and-performance.md`

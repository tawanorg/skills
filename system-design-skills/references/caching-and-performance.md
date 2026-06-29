# Caching and Performance

Caching trades memory and staleness for latency and load reduction. It is the single highest-leverage performance lever in distributed systems, and also the source of the most subtle correctness bugs. This reference covers where caches live, how to read and write through them, how they fail, and how to measure whether any of it helped.

## Where Caching Lives

A request can hit cached data at many points on its way from a user to your database. Each layer caches different things, has a different blast radius, and a different invalidation cost.

| Layer | What it caches | Typical TTL | Invalidation control |
|-------|----------------|-------------|----------------------|
| Client / browser | Rendered HTML, JS/CSS bundles, images, API responses (HTTP cache, localStorage, IndexedDB) | minutes–years (immutable assets) | Hard — you cannot reach deployed clients; rely on cache-busting URLs |
| CDN / edge | Static assets, cacheable GET responses, sometimes SSR HTML | seconds–days | Purge API per object/tag/prefix |
| Edge compute | Personalized fragments, A/B assignments, geo data | seconds–minutes | Programmatic, per-PoP |
| API gateway | Auth tokens, rate-limit counters, idempotency results, response cache | seconds–minutes | Local; usually key-scoped |
| Application | Computed objects, session data, fan-out results, feature flags | seconds–minutes | Full control in-process |
| Distributed object cache | Hot rows, serialized aggregates, leaderboards, queues (Redis/Memcached) | seconds–hours | Full control, cluster-wide |
| Database / query | Query plans, buffer pool pages, materialized views, result cache | engine-managed | Mostly automatic |

Rule of thumb: cache as close to the consumer as the correctness requirements allow. A byte served from the browser cache costs you nothing; a byte served from the DB buffer pool still costs a network round trip and a query.

## Read Patterns

### Cache-Aside (Lazy Loading)

The application owns the cache. On a miss it loads from the source and populates the cache itself.

```
GET key:
  v = cache.get(key)
  if v is not None: return v          # hit
  v = db.read(key)                    # miss
  cache.set(key, v, ttl)
  return v
```

Pros: only requested data is cached; cache outage degrades to slower reads, not errors; the cache and DB schemas are decoupled. Cons: every miss pays the full latency; first read after a write is stale unless you invalidate; the load-then-set window invites stampedes and races.

### Read-Through

The cache library/proxy sits inline and fetches from the source on a miss transparently. The app only talks to the cache. Simplifies call sites and centralizes the miss logic, but couples you to a cache that understands your data source, and a cache outage can block reads entirely.

## Write Patterns

| Pattern | How it works | Consistency | Write latency | Failure risk | Use when |
|---------|--------------|-------------|---------------|--------------|----------|
| Write-through | Write cache + DB synchronously | Strong (cache always fresh) | High (two writes) | Low | Read-heavy data that must not be stale |
| Write-back (write-behind) | Write cache now, flush to DB async | Weak (DB lags) | Low | High — buffered writes lost on crash | Write-heavy, loss-tolerant (counters, metrics) |
| Write-around | Write DB only, skip cache; cache fills on read | Cache may be stale until TTL/refill | Low | Low | Write-once / rarely-read data; avoids cache churn |

A common, pragmatic default is **cache-aside reads + write-around with explicit invalidation**: write the DB, then delete (not update) the cache key. Deleting is safer than updating because the next read recomputes the canonical value, sidestepping a class of update-ordering races.

## Invalidation

> There are only two hard things in computer science: cache invalidation and naming things. (And off-by-one errors.)

The quote is funny because both are unsolvable in general — invalidation is the problem of knowing *exactly when* a cached copy stops matching its source, across machines, with no global clock.

Strategies, roughly in order of preference:

- **TTL (time-to-live)**: every entry expires after a fixed duration. Dead simple, self-healing, bounds staleness to the TTL. The cost is guaranteed staleness up to the TTL and a re-fetch on every expiry. Most caching should start here.
- **Explicit invalidation on write**: delete/overwrite the key when the source changes. Precise but requires every writer to know every affected key — easy to miss derived/aggregate keys.
- **Versioned / key-namespaced**: embed a version in the key (`user:42:v7`). Bumping the version atomically invalidates everything under it without touching individual keys.
- **Event-driven**: subscribe to a change stream (CDC, pub/sub) and invalidate reactively. Decouples writers from caches but adds infrastructure and lag.

Pair a short TTL with explicit invalidation: invalidation handles the common case promptly, TTL is the backstop for the keys you forgot.

## Eviction Policies

Eviction frees space when the cache is full; expiry (TTL) removes stale entries regardless of space. They are orthogonal — most caches do both.

| Policy | Evicts | Fits |
|--------|--------|------|
| LRU (least recently used) | Entry untouched longest | General purpose; strong temporal locality |
| LFU (least frequently used) | Entry with fewest hits | Stable hot set; resists one-off scan pollution |
| FIFO | Oldest inserted | Cheap; when recency ≈ insertion order |
| Random | A random entry | Memcached-style; O(1), avoids LRU lock contention at scale |
| TTL-based | Whatever expired first | Freshness-bound data; time, not space, is the constraint |

LRU is the safe default. LFU wins when a small set of items is hot for a long time and you want to protect it from large sequential scans (which evict the hot set under LRU). Modern caches often use approximations (segmented LRU, TinyLFU) to get LFU's hit ratio without its bookkeeping cost.

## Stampede / Thundering Herd

When a hot key expires, every concurrent request misses simultaneously and slams the source at once. The source, sized for the cache hit rate, falls over.

```
          key expires
              |
   req1 ──────X──> DB  ┐
   req2 ──────X──> DB  ├─ N identical expensive queries
   reqN ──────X──> DB  ┘   at the same instant
```

Mitigations:

- **Request coalescing / single-flight**: the first miss acquires a per-key lock and recomputes; concurrent requests for the same key wait for and reuse that result instead of recomputing.
- **Jittered TTL**: add randomness to each TTL (`ttl ± rand`) so keys created together don't expire together.
- **Early / probabilistic refresh**: refresh a key *before* it expires, with probability that rises as it approaches expiry (XFetch-style). Most requests still get the cached value; one lucky request recomputes ahead of time.
- **Serve-stale + async refresh**: return the expired value immediately and kick off a background refresh. Bounded staleness in exchange for zero latency spikes.
- **Distributed lock**: only one node across the cluster recomputes a given key; others briefly serve stale or wait.

## Penetration, Avalanche, Hot Keys

Three distinct failure modes that get conflated:

- **Cache penetration** — requests for keys that *don't exist* (often malicious or buggy) always miss the cache and hit the DB. Defenses: **negative caching** (cache the "not found" result with a short TTL) and a **Bloom filter** of known-existing keys to reject impossible lookups before they reach the DB.
- **Cache avalanche** — a large swath of keys expire (or the cache restarts cold) at once, and the herd hits the DB together. Defenses: **staggered/jittered expiry**, multi-tier caches, and warming the cache before taking traffic.
- **Hot key** — one key is so popular it saturates a single cache node or shard. Defenses: **replicate the key** across nodes (`hotkey#1..N`, read a random replica), add a small **local/L1 cache** in front of the distributed cache, or split the value.

```
penetration : key never exists      -> negative cache + bloom filter
avalanche   : many keys expire at once -> jitter TTLs, warm cache
hot key     : one key, too much load -> replicate / local L1
```

## CDN and the Edge

A CDN is a globally distributed set of reverse-proxy caches (PoPs) close to users. A request resolves (via anycast/DNS) to the nearest PoP; on a hit it's served locally, on a miss the PoP fetches from origin, caches per the response headers, and serves it.

- **Pull (origin-pull)**: the CDN lazily fetches and caches on first request. Default for most web traffic — easy, self-populating.
- **Push**: you upload assets to the CDN ahead of time. Good for large, predictable files (video, installers) where you don't want the first user to pay the origin fetch.

Cache behavior is driven by HTTP headers:

- `Cache-Control: public, max-age=31536000, immutable` — cache aggressively (use for content-hashed asset filenames).
- `Cache-Control: no-store` / `private` — never cache / client-only.
- `s-maxage` — separate TTL for shared (CDN) caches.
- `ETag` / `Last-Modified` + `If-None-Match` / `If-Modified-Since` — **revalidation**: when stale, the client/CDN asks "still valid?" and the origin replies `304 Not Modified` (no body) if unchanged.
- `stale-while-revalidate` — serve stale while refreshing in the background.

Cache at the edge: static assets, fonts, images, and any GET response that is the same for many users. Don't cache personalized or auth-bearing responses unless you key the cache on the relevant header/cookie or split personalization into a separate uncached fragment. Use **content-hashed filenames** (`app.9f3a.js`) so deploys publish new URLs and old caches simply go unused — invalidation by naming, the cleanest kind.

## Redis vs Memcached

| | Redis | Memcached |
|---|-------|-----------|
| Data model | Rich: strings, hashes, lists, sets, sorted sets, streams, bitmaps, HyperLogLog, geo | Flat key→string (blob) |
| Persistence | RDB snapshots + AOF log; can survive restart | None; pure in-memory |
| Replication / HA | Replicas, Sentinel, Cluster sharding | Client-side sharding only |
| Threading | Mostly single-threaded core (I/O threads in 6+) | Multi-threaded |
| Extras | Pub/sub, Lua scripts, transactions, TTL per key, eviction policies | Simple, predictable, very fast for the blob case |
| Best at | Anything beyond a plain cache: queues, leaderboards, rate limiters, locks | Large pools of simple cached blobs at maximum throughput |

Use Redis when you need its data structures (a sorted set for a leaderboard, a list for a queue, atomic counters, distributed locks) or persistence/HA. Use Memcached when you genuinely just need a big, fast, multi-threaded key→blob cache and nothing more.

**When to reach for a distributed cache** at all: when the same expensive-to-compute data is read by many app instances, when you need cache state to survive individual app restarts/deploys, or when an in-process cache can't hold the working set. A distributed cache adds a network hop (~0.2–1 ms) versus in-process (~ns), so a two-tier setup (local L1 + Redis L2) is common.

## Consistency With the Source of Truth

A cache is a *copy*, so it can disagree with the source. Decide explicitly how much staleness you tolerate; "strong consistency + caching" usually means very short TTLs and synchronous invalidation, which erodes the benefit.

Key hazards and rules:

- **Stale reads** are inherent with TTLs and async writes — bound them and make them acceptable per use case (a like count can lag; a bank balance cannot).
- **Write ordering / lost invalidation race**: a read that misses can load an old DB value and write it to the cache *after* a concurrent write already invalidated — leaving a stale entry indefinitely. Mitigate by deleting keys on write (not updating), short TTLs as a backstop, versioned keys, or write-then-delete-then-delete-again (delayed double delete) for read-heavy critical keys.
- **Prefer delete over update** on writes: it avoids racing partial updates and lets the next read recompute the truth.
- **Cache the source's version/etag** alongside the value so you can detect and discard stale writes.

## How Large Streaming Services Use Caching (generic synthesis)

High-scale streaming/feed services typically layer four caches; the same shape generalizes to most read-heavy products:

1. **Edge / CDN caching** — video segments, thumbnails, and static UI assets are served from PoPs near the viewer, often pre-pushed for popular titles so the first viewer in a region doesn't hit origin.
2. **Application caching** — assembled responses (home rows, recommendations, metadata) cached in-process and in a distributed cache to avoid recomputing per request.
3. **Database/query result caching** — hot rows and expensive aggregate queries cached so the primary store handles writes and cold reads, not the read flood.
4. **Precomputed / materialized caching** — rankings, personalization, and feeds computed offline/asynchronously and stored ready-to-serve, so the request path is a lookup, not a computation.

The unifying principle: push work *off* the request path — geographically (edge), temporally (precompute), and across layers (multi-tier) — so the hot path is mostly cache lookups.

## Performance Budgets and Measuring

Caching is an optimization, and you cannot optimize what you don't measure. **Measure before optimizing** — intuition about hot paths is frequently wrong.

Track distributions, not averages — averages hide tail latency that real users feel:

- **p50 (median)** — the typical experience.
- **p95 / p99** — the tail; at scale a user issues many requests, so a "1% slow" endpoint is felt by most sessions. Tail latency, not the mean, is what users churn over.
- **Throughput** — requests/sec sustained without the tail blowing up.
- **Cache hit ratio** — the direct measure of whether the cache is earning its keep; a low ratio means wasted memory and added complexity for nothing.

```
latency histogram
  p50 ──────●
  p95 ──────────────●          <- watch this
  p99 ──────────────────────●  <- and this
         (a slow p99 can dominate user-perceived latency)
```

Set a **performance budget** (e.g. "p95 API latency < 200 ms, JS bundle < 200 KB, p99 DB query < 50 ms") and fail CI/alerts when it's exceeded. Establish a baseline, change one variable, re-measure against the same load, and confirm the win is real and not noise. A cache that adds operational complexity but moves p99 by nothing is a net loss — delete it.

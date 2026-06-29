# System Design Interview

A repeatable framework for the open-ended "design X" interview, plus the estimation
math, building blocks, and trade-off vocabulary you need to drive a 45-minute session
without freezing. The same structure works when you actually design a service at work.

## The 45-Minute Structure

Treat the interview as a guided conversation, not a monologue. Budget your time and
narrate your reasoning so the interviewer can steer.

| Phase | Step | Time | Goal |
|-------|------|------|------|
| 1 | Clarify requirements | 5 min | Lock functional + non-functional scope |
| 2 | Scope & constraints | 2 min | Agree what's in/out for this session |
| 3 | Capacity estimation | 5 min | QPS, storage, bandwidth ballparks |
| 4 | API design | 3 min | Contract that satisfies the functions |
| 5 | High-level design | 8 min | Boxes, arrows, data flow |
| 6 | Data model & storage | 5 min | Entities, access patterns, DB choice |
| 7 | Deep dive (1-2 parts) | 10 min | Show depth on the hard component |
| 8 | Bottlenecks & scale | 5 min | Find the limit, then relieve it |
| 9 | Wrap-up & trade-offs | 2 min | Summarize, name what you'd revisit |

The single biggest scoring lever is Phase 1. Everything downstream is judged against
the requirements you established, so a vague Phase 1 makes the whole design ungradeable.

## Phase 1: Functional vs Non-Functional Requirements

**Functional** requirements are what the system *does* — the verbs and nouns.
"Users post tweets," "followers see them in a feed," "a short link redirects to the
original URL." Capture them as a short bullet list and confirm the list is complete.

**Non-functional** requirements (NFRs) are the qualities the system must have. These
drive almost every architectural decision, so never skip them:

- **Scale**: DAU/MAU, total objects, growth rate.
- **Read/write ratio**: read-heavy (feeds, URL shortener) vs write-heavy (logging,
  metrics) changes everything about caching and storage.
- **Latency**: p50/p99 targets. "Feeds load in <200 ms p99."
- **Availability**: how many nines? Is brief downtime acceptable?
- **Consistency**: must reads reflect the latest write, or is staleness fine?
- **Durability**: can we ever lose data? (Payments: no. View counts: maybe.)

### Clarifying questions to always ask

- Who are the users and what is the single most important action?
- How many users / requests / objects, today and in a year?
- Is this read-heavy or write-heavy, and by roughly what ratio?
- What latency and availability do users expect?
- Is strong consistency required, or is eventual consistency acceptable?
- What's explicitly out of scope (auth, payments, analytics, mobile)?

Write the answers on the board. They become your acceptance criteria.

## Phase 3: Back-of-the-Envelope Estimation

The point is not precision; it's choosing the right order of magnitude so you know
whether one box or a thousand boxes is the answer. Round aggressively.

### Powers-of-ten and powers-of-two cheat sheet

| Power of 10 | Name | Bytes term | Power of 2 | ≈ |
|-------------|------|-----------|-----------|---|
| 10^3 | thousand | KB | 2^10 = 1,024 | ~1K |
| 10^6 | million | MB | 2^20 = 1,048,576 | ~1M |
| 10^9 | billion | GB | 2^30 | ~1B |
| 10^12 | trillion | TB | 2^40 | ~1T |
| — | — | PB | 2^50 | — |

Useful conversions: 1 day ≈ 86,400 s ≈ **~100K seconds**. A char ≈ 1 byte (ASCII),
2-4 bytes (UTF-8). A UUID ≈ 16 bytes. A timestamp ≈ 8 bytes.

### The estimation recipe

```
1. Writes/day      = DAU * writes_per_user_per_day
2. Write QPS       = writes_per_day / 86,400   (~/100K)
3. Read QPS        = write_QPS * read/write_ratio
4. Peak QPS        = avg_QPS * 2 to 10          (spikes)
5. Storage/day     = writes_per_day * bytes_per_write
6. Storage/N years = storage_per_day * 365 * N  (* replication factor)
7. Bandwidth       = QPS * avg_payload_bytes
8. Cache memory    = hot_set_objects * bytes_per_object  (often 20% of reads = 80% traffic)
```

**Worked example — URL shortener.** 100M new URLs/day, read:write = 100:1.
- Write QPS = 100M / 100K = **1,000 writes/s**; reads = **100,000/s**; peak ~200K/s.
- Storage/write ≈ 500 bytes (long URL + short code + metadata).
- 100M * 500 B = **50 GB/day** → ~18 TB/year → ~180 TB over 10 years (raw, pre-replication).
- Cache: keep the hot 20% of daily reads, ~20M URLs * 500 B ≈ **10 GB** — fits in RAM.

That tells you: the redirect path must be cache-served, writes are trivial, and storage
is large but cheap. You've now justified Redis + a partitioned KV store without guessing.

### Latency numbers worth memorizing (ballpark, order of magnitude)

| Operation | Time |
|-----------|------|
| L1 cache reference | ~1 ns |
| Branch mispredict | ~5 ns |
| L2 cache reference | ~5-10 ns |
| Mutex lock/unlock | ~20 ns |
| Main memory (RAM) reference | ~100 ns |
| Compress 1 KB (fast algo) | ~1-3 µs |
| Send 1 KB over 10 Gbps network | ~1 µs |
| Read 1 MB sequentially from RAM | ~50-100 µs |
| SSD random read | ~100-150 µs |
| Round trip within a datacenter | ~0.5 ms |
| Read 1 MB sequentially from SSD | ~1 ms |
| Disk (HDD) seek | ~5-10 ms |
| Read 1 MB from spinning disk | ~20-30 ms |
| Inter-region round trip (e.g. US↔EU) | ~80-150 ms |

Key takeaways: memory is ~1000x faster than SSD random read at the per-op level; cross-region
calls cost ~100 ms, so chatty cross-region designs kill latency budgets; disk seeks dominate
uncached reads, which is why you cache and why sequential beats random.

## Phase 4: API Design

Define the contract that satisfies each functional requirement. Keep it small and
explicit. Prefer REST verbs (or gRPC) and show the key parameters, including pagination
and auth tokens.

```
POST /urls                 { longUrl }            -> { shortUrl }
GET  /{shortCode}          ->  302 redirect to longUrl

POST /messages             { chatId, body }       -> { messageId, ts }
GET  /chats/{id}/messages  ?cursor=&limit=         -> { messages[], nextCursor }
```

Call out idempotency (a `clientRequestId` for safe retries on writes), pagination
(cursor over offset for large/changing sets), and rate limiting at this layer.

## Phase 5: High-Level Design

Draw the boxes and the request path before any deep dive. A generic web-scale shape:

```
                    +-----+
   Clients  ---->   | DNS |
                    +--+--+
                       |
                  +----v-----+        +-----+
                  |   CDN    |---->   | Obj |  (static assets, media)
                  +----+-----+        |Store|
                       |              +-----+
                 +-----v------+
                 |   Load     |
                 | Balancer   |
                 +-----+------+
                       |
                 +-----v------+
                 | API Gateway|  (authN/Z, rate limit, routing)
                 +-----+------+
                       |
          +------------+------------+
          |            |            |
     +----v----+  +----v----+  +----v----+   (stateless app servers,
     |  App #1 |  |  App #2 |  |  App #N |    horizontally scaled)
     +----+----+  +----+----+  +----+----+
          |            |            |
     +----v------------v------------v----+
     |          Cache (Redis)           |  <- read-through, hot data
     +----+-----------------------------+
          |                       |
   +------v------+        +-------v--------+
   | Primary DB  |--repl->| Read Replicas  |
   | (writes)    |        | (reads)        |
   +------+------+        +----------------+
          |
   +------v------+   +-----------+   +-------------+
   | Msg Queue   |-->| Workers   |   | Search Idx  |
   | (async)     |   | (jobs)    |   | (ES/OpenS)  |
   +-------------+   +-----------+   +-------------+

   Monitoring/metrics/logging/tracing wrap every box.
```

Trace one read and one write through this diagram out loud. That narration is what
demonstrates you understand the data flow, not the prettiness of the drawing.

## Common Building Blocks Checklist

| Block | Why it exists | Reach for it when |
|-------|---------------|-------------------|
| **DNS** | Name → IP, geo-routing | Always; mention GeoDNS for multi-region |
| **Load balancer** | Spread traffic, health checks | >1 app server; L4 for speed, L7 for routing |
| **API gateway** | AuthN/Z, rate limit, routing | Many services or public API |
| **App servers (stateless)** | Business logic | Always; statelessness enables horizontal scale |
| **Cache** | Cut read latency & DB load | Read-heavy, hot keys, expensive queries |
| **Primary DB + replicas** | Source of truth; scale reads | Always; replicas when reads ≫ writes |
| **Object storage** | Cheap blobs (images, video, backups) | Large binary/media files |
| **Message queue** | Decouple, absorb spikes, async work | Slow/non-critical-path work, fan-out |
| **Search index** | Full-text & faceted queries | Search box, filtering on text |
| **CDN** | Edge-cache static/media near users | Global users, static or cacheable content |
| **Monitoring** | See what's happening | Always; metrics, logs, traces, alerts |

## Phase 6: Data Model & Storage Choice

List your entities and their relationships, then the **access patterns** — they decide
the database far more than the entities do. "Look up message by chatId ordered by time"
implies a partition key of chatId and a clustering key of timestamp.

Quick chooser (see `databases-and-storage.md` for depth):

- **Relational (Postgres/MySQL)**: strong consistency, transactions, joins, flexible
  ad-hoc queries. Default unless you have a reason not to.
- **Key-value (Redis, DynamoDB)**: simple lookups by key, massive scale, low latency.
- **Wide-column (Cassandra, Bigtable)**: huge write throughput, time-series, known query
  patterns, tunable consistency.
- **Document (MongoDB)**: nested/variable schemas, read whole aggregates together.
- **Search (Elasticsearch)**: full-text and faceted search, not a system of record.
- **Object store (S3/GCS)**: large immutable blobs; keep only the URL in the DB.

Name the consistency model and the partition/sharding key explicitly.

## Phase 7-8: Handling Scale

Don't pre-scale everything. The procedure is: **find the bottleneck, then apply the
matching lever.** Identify what saturates first (usually DB reads, a hot key, or a
slow synchronous step), and relieve only that.

| Symptom | Lever | Mechanism |
|---------|-------|-----------|
| DB read overloaded | **Caching** | Read-through/aside; cache hot keys in Redis |
| Reads ≫ DB can serve | **Read replicas** | Fan reads to replicas; accept replica lag |
| One DB can't hold data/writes | **Sharding** | Partition by key (hash or range) |
| Slow work blocks response | **Async / queue** | Push to queue, return early, process by workers |
| Static/media far from users | **CDN** | Edge caching near the client |
| Search/filter slow on DB | **Search index** | Maintain an inverted index out of band |
| App server CPU bound | **Horizontal scale** | Add stateless instances behind the LB |

When you shard, state the shard key and how you avoid hotspots (consistent hashing,
salting hot keys). When you cache, state TTL and the invalidation strategy (write-through,
write-invalidate, or accept staleness). When you replicate, acknowledge replica lag and
which reads tolerate it.

Avoid premature optimization: a single Postgres instance handles thousands of QPS. Reach
for sharding only after replicas and caching are exhausted, and say so.

## Phase 9: Discussing Trade-offs Explicitly

Strong candidates frame decisions as trade-offs, not absolutes. Use this vocabulary:

- **Consistency vs availability (CAP)**: under a partition you pick one. Bank balance →
  consistency. Like-count → availability + eventual consistency.
- **Strong vs eventual consistency**: strong is simpler to reason about but slower and
  less available; eventual scales but exposes staleness you must design around.
- **Latency vs cost**: more cache/replicas/edge = faster but pricier. Match spend to NFRs.
- **Latency vs durability**: synchronous replication is durable but slow; async is fast
  but can lose recent writes on failover.
- **Simplicity vs flexibility**: a normalized schema is flexible; a denormalized/precomputed
  one is fast but rigid and needs careful invalidation.
- **Push vs pull (fan-out)**: fan-out-on-write (push) gives fast reads, costly for celebrity
  accounts; fan-out-on-read (pull) is cheap to write, slower to read. Hybrids are common.

State the option you chose, the one you rejected, and the condition that would flip your
choice. That conditional reasoning is the signal interviewers want.

## Common Prompts → Core Challenge → Key Technique

| Prompt | Core challenge | Key technique |
|--------|----------------|---------------|
| **URL shortener** | Huge read:write, unique short codes | KV store + cache; base62 of a counter or hash |
| **News feed** | Fan-out to followers, fast reads | Fan-out-on-write + pull for celebrities; precomputed feed cache |
| **Chat / messaging** | Real-time delivery, ordering, presence | WebSockets, per-chat partitioning, message queue, sequence IDs |
| **Rate limiter** | Accurate counting at scale, low latency | Token/leaky bucket or sliding window in Redis; distributed counters |
| **Notification system** | Fan-out across channels, retries, dedup | Queue + worker pool; idempotency keys; per-channel adapters |
| **Web crawler** | Politeness, dedup, scale, freshness | Frontier queue (BFS), URL dedup (Bloom filter), robots.txt, backoff |
| **File / object store** | Large blobs, durability, dedup | Chunking, content-addressing, replication/erasure coding, metadata DB |
| **Ride-hailing matching** | Geo proximity, real-time matching | Geospatial index (geohash/quadtree/S2), dispatch service, live location stream |
| **Payment system** | Correctness, no double-charge, audit | Idempotency keys, ledger (double-entry), exactly-once via dedup, strong consistency |
| **Typeahead / search** | Sub-100 ms prefix lookup | Trie + prefix cache; precomputed top-k per prefix |

For each, lead with the core challenge — it tells the interviewer you see what's actually
hard, rather than reciting a generic three-tier app.

## Red Flags Interviewers Watch For

- **Jumping to a solution** before clarifying requirements or NFRs.
- **Ignoring non-functional requirements** — no latency, consistency, or scale targets.
- **No estimation** — designing without knowing if it's 10 QPS or 1M QPS.
- **No trade-offs** — presenting one design as obviously correct with no alternatives.
- **Over-engineering** — sharding and Kafka for a system that fits on one box; complexity
  with no requirement driving it.
- **Hand-waving the hard part** — staying in safe high-level boxes and never going deep.
- **Ignoring failure** — no story for a dead node, replica lag, or a hot partition.
- **Not listening** — missing the interviewer's hints to go a particular direction.

## One-Line Summary of the Method

Clarify and quantify first; design the simplest thing that meets the NFRs; find the one
bottleneck and relieve it with the matching lever; and narrate every trade-off you make.

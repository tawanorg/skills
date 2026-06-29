# Real-World Case Studies

Distilled architectural lessons synthesized from widely-known, public engineering
patterns — how large systems scaled, and what generalizes. These are reusable
patterns and orders of magnitude, not reproductions of any specific company's
internal numbers or proprietary write-ups.

Each case follows: **context → approach → reusable lesson.**

---

## 1. Scaling a Relational Database Past One Node

**Context.** A single Postgres/MySQL primary handles all reads and writes. Growth
saturates CPU, then IOPS, then connection slots. The instinct is "buy a bigger
box," but vertical scaling has a ceiling and a price curve that bends upward.

**Approach — applied in order, because each step costs more to operate:**

```
Stage 0  Single primary                      (simplest, single point of failure)
Stage 1  + Read replicas (streaming repl)     reads scale, writes don't
Stage 2  + Vertical/functional partitioning   split tables across DBs by domain
Stage 3  + Horizontal sharding                split rows of one table across nodes
```

1. **Read replicas.** Asynchronous replicas absorb read-heavy traffic. Cheap to add,
   but introduces **replication lag** — read-after-write anomalies. Mitigate by
   routing reads that must be fresh to the primary, or by "read-your-writes"
   sticky routing.
2. **Vertical / logical partitioning.** Move whole tables (or bounded contexts —
   billing, catalog, sessions) onto separate database servers. Each scales
   independently; you lose cross-table JOINs and cross-DB transactions.
3. **Horizontal sharding.** Split one large table's rows across N nodes by a
   **shard key** (hash or range). Now even a single table's writes scale. Cost:
   cross-shard queries, rebalancing, and the shard key becomes a permanent design
   decision that's painful to change.

**Operational cost ladder:** replicas ≈ low (managed failover), functional
partitioning ≈ medium (app must know where data lives), sharding ≈ high (custom
routing layer, resharding tooling, fan-out queries, hotspot management).

**Reusable lesson.** Climb the ladder one rung at a time; most workloads never need
sharding. Pick the shard key for your dominant access pattern, choose hash for even
distribution or range for locality, and never shard before replicas + caching are
exhausted. Sharding is a one-way door — design the routing layer to abstract it.

---

## 2. Multi-Tier Caching at a Streaming / Media Company

**Context.** Catalog browsing, artwork, and "what's trending" are read by millions
but change slowly. Recomputing or re-fetching per request would melt the origin.

**Approach — cache in layers, each closer to the user and cheaper to serve:**

```
User → CDN/edge (images, manifests) → App cache (Redis/Memcached) → DB result cache → DB
        ms, global                      sub-ms, regional             query memoized    source of truth
```

- **CDN/edge** caches large, immutable assets (thumbnails, video segments) near the
  user. Use content-hashed URLs so cache invalidation is "just deploy a new URL."
- **Application cache** memoizes computed objects (a user's homepage rows, a region's
  top-10). Keyed by user segment, not user ID, to keep hit rates high.
- **DB result cache** stores expensive query outputs with short TTLs.
- **Precomputation + cache warming.** Personalized/ranked views are computed
  **offline in batch**, written to the cache, and simply read at request time. On
  deploy or cold start, warm the cache proactively so the first users don't pay the
  miss penalty (avoiding a **thundering herd** / cache stampede).

**Reusable lesson.** The fastest query is the one you never run. Push reads to the
edge, segment cache keys to maximize hit rate, and precompute anything expensive
that tolerates staleness. Always plan invalidation and stampede protection (request
coalescing, jittered TTLs) *before* you ship the cache.

---

## 3. Social Timeline / "For You" Feed

**Context.** Users follow others; each open of the app must return a ranked feed in
tens of milliseconds. Naively reading and merging every followee's recent posts at
read time (fan-out-on-read) is too slow for users following thousands of accounts.

**Approach — fan-out strategy is the core trade-off:**

| Strategy | When post is written | When feed is read | Best for |
|---|---|---|---|
| Fan-out-on-write | push post into every follower's feed cache | cheap read | normal users |
| Fan-out-on-read | nothing | gather + merge at read time | celebrities |
| **Hybrid** | push for normal authors; pull for mega-followers | merge precomputed + live celebrity posts | everyone |

- **Fan-out-on-write** precomputes each user's timeline into a per-user list
  (e.g. in Redis). Reads are O(1). Writes are O(followers) — fine until someone has
  tens of millions of followers, where a single post causes a write storm.
- **Hybrid for celebrities.** Skip fan-out for high-follower accounts; instead merge
  their posts in at read time. Each user's feed = precomputed list ⊕ live pull of the
  few celebrities they follow.
- **Ranking pipeline** sits on top: candidate generation → feature hydration →
  ML scoring → re-ranking/diversity → serve. The **latency budget** (often a low
  triple-digit ms ceiling) is what forces precomputation of candidates and features.

**Reusable lesson.** Choose write-time vs read-time work by the fan-out ratio, and go
hybrid at the tail. Latency budgets, not storage, drive the decision to precompute.
Separate candidate generation (cheap, broad) from ranking (expensive, narrow).

---

## 4. Push Notification System at Scale

**Context.** An event ("you got a like," "price drop") must reach the right devices
across iOS/Android/web within seconds, for millions of recipients, reliably.

**Approach — a staged pipeline with queues between every stage:**

```
Producers → Ingestion API → [queue] → Fan-out service → [queue] → Per-device delivery → APNs/FCM/WebPush
                                          (expand topic →            (rate-limit, retry,
                                           recipient list)            token mgmt)
```

- **Ingestion** validates, deduplicates, and accepts fast; heavy work happens async.
- **Fan-out** expands a logical target ("followers of X," "users in segment Y") into
  concrete device tokens — the same fan-out-on-write/read trade-off as feeds.
- **Per-device delivery** talks to platform gateways (APNs/FCM). These impose rate
  limits, so workers throttle and **retry with exponential backoff + jitter**.
- **Device token management** is the silent hard part: tokens expire, devices
  uninstall, users opt out. Gateways return "invalid token" feedback — consume it and
  prune, or you waste capacity blasting dead endpoints.
- **Idempotency & dedup** prevent the same notification arriving twice after a retry.

**Reusable lesson.** Decouple ingestion from delivery with durable queues so spikes
buffer instead of cascade. Treat the device registry as a first-class, continuously
pruned dataset. Backoff + jitter + idempotency keys are mandatory, not optional.

---

## 5. Messaging at Scale — Storing Trillions of Messages

**Context.** A chat product accumulates an unbounded, append-heavy stream of
messages. The access pattern is "give me the last N messages in this channel,"
almost never "find all messages by user across all channels." A relational primary
key + B-tree index buckles under the write volume and dataset size.

**Approach.**

- **Move off the relational store** for the message corpus. A **wide-column store**
  (Cassandra/HBase/Bigtable-style) matches the workload: high write throughput,
  linear horizontal scale, tunable consistency.
- **Partition by channel, cluster by time.** Partition key = channel/conversation ID;
  clustering key = timestamp (descending). "Last N messages" becomes a single
  partition range scan — fast and local. Hot, huge channels get **bucketed** by
  time window so no single partition grows unbounded.
- Keep relational/transactional stores for what they're good at (accounts, billing).

**Reusable lesson.** Pick the storage engine to fit the *access pattern*, not the
other way around. Append-heavy, range-by-key reads → wide-column. Encode your primary
query into the partition + clustering keys. Don't force one database to serve every
workload — polyglot persistence is a feature.

---

## 6. Event-Driven Architecture for a Retail / QSR Ordering System

**Context.** An order flows through cart → payment → kitchen/fulfillment → delivery,
each owned by a different team/service. Synchronous, tightly-coupled calls mean one
slow downstream (payment processor, kitchen display) stalls the whole checkout.

**Approach.**

- **Decouple via events.** Services publish facts (`OrderPlaced`, `PaymentCaptured`,
  `OrderReady`) to a log/broker (Kafka/Kinesis/PubSub). Consumers react independently.
- **Eventual consistency.** The order isn't atomically "done" everywhere at once; each
  service converges as it processes events. Acceptable because the business process is
  inherently asynchronous (the kitchen doesn't cook synchronously with the tap).
- **Order state machine.** Model the order as an explicit FSM with allowed transitions;
  events drive transitions. Use the **saga / orchestration-or-choreography** pattern
  for multi-step flows, with **compensating actions** (refund on kitchen failure).

```
CREATED → PAID → PREPARING → READY → OUT_FOR_DELIVERY → DELIVERED
   └──────────────── CANCELLED / REFUNDED (compensation) ───────────┘
```

**Reusable lesson.** Events buy you decoupling, independent scaling, and resilience —
at the cost of eventual consistency you must design for. Make the workflow an explicit
state machine, define compensations for every step, and ensure consumers are
idempotent (events get redelivered).

---

## 7. Video Upload / Processing Pipeline

**Context.** Users upload large video files over flaky connections; viewers need
smooth playback across devices and bandwidths. Doing this inline with the upload
request is impossible — encoding takes minutes.

**Approach.**

```
Client → Chunked/resumable upload → Object store (raw)
                                       │ emits event
                                       ▼
                              Transcode workers ──► renditions (240p…4K, HLS/DASH)
                                       │
                                       ▼
                                  CDN distribution ──► viewers (adaptive bitrate)
```

- **Chunked, resumable upload** (multipart) survives dropped connections — retry only
  the failed chunk, not the whole file.
- **Async processing.** Upload completion publishes an event; a worker pool transcodes
  into **multiple renditions** (resolutions/bitrates) and packages for adaptive
  streaming. The client gets "processing…" immediately, not a 5-minute hang.
- **CDN distribution.** Renditions are immutable, content-addressed, and pushed to the
  edge; players fetch the bitrate that matches current bandwidth (ABR).
- Workloads are **fan-out + idempotent**: re-running a transcode must be safe.

**Reusable lesson.** Separate ingest from processing with an event boundary; never
block a user request on heavy media work. Resumable uploads for reliability, multiple
renditions for reach, immutable CDN assets for scale. Make every processing job
idempotent so retries are free.

---

## 8. Microservice Evolution & Conway's Law

**Context.** A successful product starts as a monolith. As the org grows, every
team commits to the same codebase and deploy pipeline; releases serialize, blast
radius grows, and ownership blurs.

**Approach.**

```
Monolith ──► Coarse SOA ──► Microservices ──► + Service mesh
(1 deploy)   (a few        (many small,        (sidecars handle mTLS,
              services)      team-owned)         retries, traffic, telemetry)
```

- **Split along team and bounded-context lines**, not arbitrary technical layers.
- **Conway's Law:** systems mirror the communication structure of the org that builds
  them. Service boundaries that don't match team boundaries create constant
  cross-team coordination — the "distributed monolith" anti-pattern (microservices
  with monolith-level coupling, now with network calls).
- **Service mesh** (Istio/Linkerd/Envoy) is adopted when the *number* of services
  makes cross-cutting concerns — mTLS, retries, timeouts, circuit breaking,
  observability — unmanageable per-service. The mesh pushes them into sidecars.

**Reusable lesson.** Don't start with microservices; earn them when team coordination
cost exceeds the operational cost of distribution. Align service boundaries to team
ownership. Adopt a mesh only when service count justifies the complexity — it solves
*scale of services*, not *scale of traffic*.

---

## 9. Starting Simple — The Unconventional-Backend Migration

**Context.** Many wildly successful startups launched on a backend nobody would
recommend "at scale" — a single relational DB, a spreadsheet-like store, a hosted
BaaS, even SQLite or a monolith with a job table as a queue. They migrated later,
under the pressure of real (not imagined) load.

**Approach.**

- Ship the boring, simple thing that lets you learn what users actually do.
- Instrument early so you can see *which* component breaks first.
- Migrate the specific bottleneck when metrics demand it — not the whole stack, and
  not on speculation. Replace the queue, shard the one hot table, extract the one hot
  service.

**Reusable lesson.** Premature scaling is a top startup killer — it spends complexity
budget on load you may never get. Start simple, measure, and let real bottlenecks
prioritize your migrations. A migration you can clearly justify with metrics is
cheaper than the architecture you over-built and now must maintain.

---

## 10. Cross-Cutting Reusable Lessons

These recur across every case above:

- **Start simple.** Match architecture to current load + one growth step ahead, not
  to a hypothetical future. Complexity is a budget; spend it on proven bottlenecks.
- **Cache aggressively, invalidate deliberately.** Layer caches (edge → app → DB).
  The hard part is always invalidation and stampede protection, so design those first.
- **Make writes idempotent.** Networks retry; queues redeliver. Idempotency keys and
  upserts turn "exactly once" (impossible) into "effectively once" (achievable).
- **Embrace async + queues.** Put durable queues between producers and consumers so
  spikes buffer instead of cascade. Accept the work, do the heavy lifting later.
- **Design for failure.** Assume every dependency fails: timeouts, retries with
  backoff + jitter, circuit breakers, bulkheads, graceful degradation, and a clear
  blast radius. Failure is the steady state at scale.
- **Pick a consistency model per use case.** Strong consistency for money and auth;
  eventual consistency for feeds, counts, and notifications. Don't pay for consistency
  you don't need, and don't skimp where correctness is non-negotiable.
- **Instrument everything.** Metrics, structured logs, and traces are how you know
  *which* rung of the ladder to climb next. You can't scale what you can't see.

---

## Pattern → Problem → Where It Shows Up

| Pattern | Problem it solves | Where it shows up |
|---|---|---|
| Read replicas | read throughput exceeds one node | any read-heavy relational workload |
| Functional/vertical partitioning | mixed workloads contend on one DB | splitting billing/catalog/sessions |
| Horizontal sharding | single table's writes exceed one node | trillion-row tables, high write volume |
| Multi-tier caching | repeated expensive reads | catalogs, media, dashboards |
| Precomputation / cache warming | per-request compute too slow | ranked feeds, homepages, top-N lists |
| Fan-out-on-write | read latency budget is tight | social timelines, notification feeds |
| Fan-out-on-read (hybrid) | write storms from mega-fan-out | celebrity accounts, broadcast |
| Durable queues | spikes overwhelm sync pipelines | notifications, video processing, orders |
| Retry + backoff + jitter | transient downstream failures | gateway delivery, distributed calls |
| Idempotency keys | retries/redelivery cause duplicates | payments, notifications, event consumers |
| Wide-column store | append-heavy, range-by-key reads | messages, time-series, event logs |
| Partition + clustering key design | "last N by key" must be fast | chat history, activity logs |
| Event-driven decoupling | services tightly coupled / slow | ordering, fulfillment, integrations |
| Saga + compensation | multi-step distributed transactions | checkout, order lifecycle |
| State machine | implicit, buggy status transitions | orders, jobs, workflows |
| Chunked/resumable upload | large files over flaky networks | video/file upload |
| Multiple renditions + CDN + ABR | diverse devices/bandwidth | video streaming, image delivery |
| Service mesh | cross-cutting concerns × many services | mature microservice fleets |
| Conway-aligned boundaries | distributed monolith coupling | monolith → microservice splits |
| Polyglot persistence | one DB can't serve all access patterns | relational + KV + wide-column mix |
| Start simple, migrate on metrics | premature scaling waste | early-stage products, MVPs |

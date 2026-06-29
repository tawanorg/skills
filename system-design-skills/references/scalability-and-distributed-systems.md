# Scalability and Distributed Systems

A practical reference for designing systems that grow under load and stay correct when machines, networks, and clocks misbehave. The recurring theme: scale comes from removing shared state, and correctness comes from being honest about what you give up when the network splits.

## Scaling: Up vs Out

| Axis | Vertical (scale up) | Horizontal (scale out) |
|------|---------------------|------------------------|
| Mechanism | Bigger box (more CPU/RAM/IO) | More boxes behind a balancer |
| Ceiling | Hardware limit, single-vendor pricing | Effectively unbounded |
| Failure blast radius | Whole service on one node | One node of many |
| Complexity | Low (no distribution) | High (coordination, partial failure) |
| Good for | Databases, stateful cores, quick wins | Stateless app tiers, read fan-out |

Reach for vertical first because it is operationally cheap; switch to horizontal when you hit a ceiling or need fault tolerance. The two compose: scale the stateless tier out, and scale (or shard) the stateful core up.

### Stateless services enable horizontal scale

A node is *stateless* when any replica can serve any request because it holds no per-client data between requests. That is what lets a load balancer treat replicas as interchangeable and lets autoscalers add/remove them freely.

- **Sticky sessions** pin a client to one node (via cookie or source IP) so in-memory session state survives across requests. Cheap to start, but it breaks autoscaling, defeats even load distribution, and loses state when that node dies.
- **Externalized state** pushes session/cart/auth data into a shared store (Redis, a database, or a signed token like a JWT). Now every node is identical and disposable. This is the preferred pattern: keep app servers stateless, push state to a tier built to be stateful.

## Load Balancing

### L4 vs L7

- **L4 (transport):** routes by IP/port, forwarding TCP/UDP without reading payloads. Very fast, protocol-agnostic, but cannot route on URL/header/cookie.
- **L7 (application):** terminates the connection, parses HTTP(S), and routes on path, host, header, or cookie. Enables TLS termination, content-based routing, retries, and request-level rate limiting at the cost of more CPU per request.

### Algorithms

| Algorithm | How it picks a backend | Best when |
|-----------|------------------------|-----------|
| Round robin | Next in rotation | Uniform requests, uniform backends |
| Weighted round robin | Rotation biased by capacity weight | Heterogeneous backend sizes |
| Least connections | Fewest in-flight connections | Variable request durations |
| Least response time | Lowest latency + active count | Latency-sensitive traffic |
| IP hash | hash(client IP) → backend | Cheap stickiness without cookies |
| Consistent hash | hash(key) on a ring | Cache affinity, minimal reshuffle on resize |

Health checks (active probes + passive ejection of erroring nodes) are what make any of these safe in production.

## CAP and PACELC

**CAP:** during a network **P**artition you must choose between **C**onsistency (every read sees the latest write or errors) and **A**vailability (every request gets a non-error response, possibly stale). Partitions are not optional — packets drop, links fail — so **P is a given**, and the real design choice is CP vs AP.

- **CP** (e.g. a quorum store, ZooKeeper): on partition, reject requests that can't be made consistent. Choose when staleness is unacceptable (balances, locks, config).
- **AP** (e.g. Dynamo-style stores, DNS): on partition, keep serving possibly-stale data and reconcile later. Choose when uptime beats freshness (shopping carts, feeds).

CAP only speaks to the partition case. **PACELC** completes it: *if Partition then choose A or C, Else (normal operation) choose between Latency and Consistency.* Even with a healthy network, synchronous replication for strong consistency costs latency; relaxing it buys speed. So a system is described as e.g. PC/EL (consistent under partition, latency-favoring otherwise) or PA/EL (Dynamo).

## Consistency Models

From strongest to weakest — stronger is easier to reason about, weaker is cheaper and faster:

- **Linearizable (strong):** every operation appears to take effect instantly at some point between its call and return; there is a single, real-time-respecting global order. The gold standard for correctness, the most expensive to provide.
- **Sequential:** all clients see operations in *some* single order consistent with each client's program order — but that order need not match real time.
- **Causal:** operations that are causally related (A happened-before B) are seen in that order by everyone; concurrent operations may be seen in different orders. A sweet spot — preserves "reply after post" without global coordination.
- **Eventual:** if writes stop, all replicas eventually converge. Says nothing about *when* or what you read meanwhile.

**Client-centric session guarantees** make eventual consistency usable:

- **Read-your-writes:** after you write, your later reads reflect it (don't show a user a stale version of their own edit).
- **Monotonic reads:** once you've seen a value, you never see an older one (no going backward in time).
- **Monotonic writes / writes-follow-reads:** your writes apply in order, and a write conditioned on a read lands after it.

## Consensus and Coordination

Consensus = getting a set of nodes to agree on one value (or one ordered log of values) despite failures. It underpins leader election, distributed locks, and replicated state machines.

- **Leader election:** pick one node to serialize decisions. The leader orders writes; followers replicate. Avoids write conflicts but the leader is a bottleneck and a failover point.
- **Quorum:** require a majority to agree so any two decisions overlap on at least one node. With N replicas, write to W and read from R; choosing **R + W > N** guarantees a read quorum intersects the latest write quorum (so reads can see the newest write), and **W > N/2** prevents two conflicting writes from both committing. Common: N=3, W=2, R=2.
- **Raft / Paxos (conceptually):** protocols that keep a replicated log consistent across a majority. Raft is the more teachable: nodes hold terms, elect a leader by majority vote, and the leader appends entries that commit once a majority has them. Both tolerate up to ⌊(N-1)/2⌋ failures and need a majority alive to make progress.
- **Split-brain:** a partition leaves two sides each believing they're the leader, accepting divergent writes. Majority quorums prevent it — a minority side cannot reach quorum and must stop accepting writes. Fencing tokens (monotonic IDs) stop a stale leader's late writes from being honored.

**ZooKeeper / etcd** are off-the-shelf consensus-backed coordination stores (etcd uses Raft, ZooKeeper uses Zab). Use them for the small, critical, strongly-consistent metadata that everything else keys off: leader election, service discovery, distributed locks, configuration, and membership. They are not your application database — they're the source of truth for *who's in charge and what the config is*.

## Distributed Transactions

### Two-Phase Commit (2PC)

A coordinator runs **prepare** (all participants vote and durably promise they can commit) then **commit/abort** (coordinator broadcasts the outcome). It gives atomicity across services but:

- **Blocking:** if the coordinator dies after prepare, participants hold locks indefinitely, unsure of the outcome.
- **Latency & coupling:** synchronous, holds locks across the round trip, and availability is the *product* of all participants'.

Acceptable inside one trusted boundary; a poor fit across microservices.

### Saga Pattern

Model a long transaction as a sequence of local transactions, each with a **compensating action** that semantically undoes it (refund, not rollback). No global locks; on failure you run compensations in reverse.

- **Choreography:** each service emits events; the next reacts. Decentralized, no coordinator, but the flow is implicit and hard to trace as it grows.
- **Orchestration:** a central orchestrator tells each service what to do and drives compensation on failure. Explicit, observable, testable — at the cost of a coordinator component.

```
Order ──place──▶ [Reserve Stock] ──ok──▶ [Charge Card] ──FAIL──▶
                       ▲                                    │
                       └──── compensate: Release Stock ◀────┘
                                 compensate: (none, charge never committed)
```

Sagas give atomicity-of-outcome, not isolation — intermediate states are visible, so design for it (pending states, idempotent steps).

### Outbox Pattern

Dual-writing to a DB and a message broker isn't atomic — one can succeed and the other fail. The outbox fixes this: in the *same local transaction* as the business write, insert the event into an `outbox` table. A separate relay (polling or CDC log-tailing) reads the outbox and publishes to the broker, marking rows sent. The DB transaction guarantees the event exists iff the state change committed; the relay guarantees at-least-once delivery to the broker.

## Delivery Semantics and Idempotency

| Guarantee | Meaning | Reality |
|-----------|---------|---------|
| At-most-once | Never redelivered | May lose messages |
| At-least-once | Never lost | May duplicate — the common default |
| Exactly-once | Once, no loss, no dup | Not achievable end-to-end on an unreliable network |

True exactly-once *delivery* is impossible; what you build is **effectively-once processing** = at-least-once delivery + idempotent consumers.

- **Idempotency key:** client sends a unique key per logical operation; the server records "key → result" and on a retry returns the stored result instead of re-executing. Essential for payments, order creation, any non-idempotent POST.
- **Deduplication:** consumers track processed message IDs (in a store or a bounded window) and skip repeats. Pair with idempotent writes (upserts, conditional updates) so a duplicate that slips through is harmless anyway.

## Messaging: Queues and Logs

- **Queue (e.g. SQS, RabbitMQ):** messages are consumed and removed; work is distributed across competing consumers. Great for task distribution and load leveling. Ordering across consumers is generally not preserved.
- **Log (e.g. Kafka, Pulsar):** an append-only, partitioned, *retained* stream. Consumers track their own offset and can replay. Multiple independent consumer groups read the same data. Ordering is guaranteed **within a partition**, not across them — so route messages that must stay ordered (e.g. one user's events) to the same partition via a partition key.
- **Pub/Sub:** publishers emit to a topic; all subscribers get a copy. Decouples producers from the number and identity of consumers.

Operational concerns:

- **Backpressure:** when consumers fall behind, signal upstream to slow down (bounded buffers, pull-based consumption, credit/ack windows) rather than let queues grow unbounded and OOM. A log absorbs bursts via retention; a queue must shed or throttle.
- **Dead-letter queue (DLQ):** after N failed processing attempts, divert the message to a DLQ for inspection instead of blocking the partition or looping forever. Alert on DLQ depth.

## Rate Limiting

| Algorithm | Mechanism | Character |
|-----------|-----------|-----------|
| Token bucket | Tokens refill at rate r, capacity b; each request spends one | Allows bursts up to b, smooth average |
| Leaky bucket | Requests queue, drain at fixed rate | Smooths output to a constant rate, no bursts |
| Fixed window | Count per wall-clock window (e.g. per minute) | Simple; allows 2× burst at window edges |
| Sliding window log | Timestamps of recent requests in a rolling window | Accurate; more memory |
| Sliding window counter | Weighted blend of current + previous window | Good accuracy/cost tradeoff |

Token bucket is the workhorse for APIs (burst-tolerant, O(1) state per client). For distributed enforcement, keep counters in a shared store (Redis with atomic Lua) or accept approximate local limits per node.

## Failure Handling

The network will fail; design for partial failure as the normal case.

- **Timeouts:** every remote call needs one. No timeout = a hung dependency exhausts your threads/connections and cascades.
- **Retries with exponential backoff + jitter:** retry transient failures with delays `base · 2^attempt`, plus **jitter** (randomization) so retries from many clients don't synchronize into a thundering herd. Only retry idempotent operations; cap attempts.
- **Circuit breaker:** track failure rate to a dependency. **Closed** → calls pass. On crossing a threshold, **Open** → fail fast immediately (don't pile onto a sick service). After a cooldown, **Half-open** → let a trial trickle through; success closes it, failure re-opens. Gives the dependency room to recover and keeps you responsive.
- **Bulkheads:** isolate resources (separate thread/connection pools per dependency) so one slow downstream can't drown the whole service — like watertight compartments in a ship.
- **Graceful degradation:** shed non-essential features under stress — serve stale cache, drop personalization, show a simplified view — rather than failing the whole request. Decide degradation paths *before* the incident.

## Partitioning and Consistent Hashing

**Partitioning (sharding)** splits data across nodes so each holds a slice. Strategies: by **range** (ordered, good for scans, risks hot ranges), by **hash** (even spread, kills range scans), or by **directory** (a lookup table — flexible, adds a hop). Watch for **hot keys/partitions** where one shard gets disproportionate traffic.

Naive `hash(key) mod N` remaps almost every key when N changes — catastrophic for caches. **Consistent hashing** maps both nodes and keys onto a hash ring; a key belongs to the first node clockwise. Adding/removing a node only moves the keys between it and its neighbor — about K/N keys, not all of them.

```
            ┌──────── 0 ────────┐
        keyA│      ● NodeA       │
            │   ↘                │
   NodeC ●  │      keyB → NodeB  │  ● NodeB
            │                    │
            │      ● NodeC ↙     │keyC
            └──────── ring ──────┘
   each key walks clockwise to the next node
```

**Virtual nodes:** assign each physical node many positions on the ring (e.g. 100–200 vnodes each). This evens out the otherwise lumpy key distribution and lets you weight by capacity (bigger nodes get more vnodes). When a node leaves, its load spreads across many neighbors instead of dumping onto one.

## Back-of-Envelope Numbers

Ballpark latencies, within an order of magnitude — for sanity-checking designs:

| Operation | ~Time |
|-----------|-------|
| L1 cache reference | ~1 ns |
| Branch mispredict | ~3 ns |
| L2 cache reference | ~4 ns |
| Mutex lock/unlock | ~17 ns |
| Main memory reference | ~100 ns |
| Compress 1 KB (fast) | ~2 µs |
| Read 1 MB sequentially from memory | ~100 µs |
| SSD random read | ~16 µs |
| Read 1 MB sequentially from SSD | ~1 ms |
| Round trip within same datacenter | ~0.5 ms |
| Read 1 MB sequentially from disk (HDD) | ~20 ms |
| Disk seek (HDD) | ~10 ms |
| Round trip CA ↔ Netherlands | ~150 ms |

Takeaways: memory is ~100× faster than SSD and ~10,000× faster than a cross-continent round trip; cache aggressively, batch network calls, and keep chatty services co-located.

### Availability "nines"

| Availability | Downtime / year | Downtime / day |
|--------------|-----------------|----------------|
| 99% (two nines) | ~3.65 days | ~14.4 min |
| 99.9% (three nines) | ~8.77 hours | ~1.44 min |
| 99.99% (four nines) | ~52.6 min | ~8.6 sec |
| 99.999% (five nines) | ~5.26 min | ~0.86 sec |

Serial dependencies *multiply*: two 99.9% services in a hard dependency yield ~99.8%. Redundancy (parallel paths) does the opposite — two parallel 99% paths give ~99.99%.

## Observability

You can't fix what you can't see, and in distributed systems no single node has the whole picture. The three pillars:

- **Logs:** discrete, timestamped events. Make them **structured** (key-value/JSON) and carry a correlation/trace ID so you can stitch a request across services. High detail, high volume — sample or tier retention.
- **Metrics:** numeric time series (counters, gauges, histograms). Cheap to store, ideal for dashboards and alerting. Track the **RED** method for services (Rate, Errors, Duration) and **USE** for resources (Utilization, Saturation, Errors). Alert on percentiles (p99), not averages — averages hide tail pain.
- **Traces:** the path of a single request across services as a tree of timed **spans**, linked by a propagated trace context (e.g. W3C `traceparent`). This is how you find *which hop* in a 12-service call chain blew the latency budget.

Use them together: a metric alert tells you *something* is wrong, a trace tells you *where*, and logs tell you *why*. OpenTelemetry is the de-facto vendor-neutral standard for emitting all three.

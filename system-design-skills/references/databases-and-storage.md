# Databases and Storage

Storage is the part of a system that is hardest to change later, so the choice is driven first by how data is *accessed*, not by which database is trendy. This reference covers picking a store by access pattern, the SQL/NoSQL and ACID/BASE trade-offs, indexing internals, replication, partitioning, consistency, and the scaling progression most teams actually follow.

## Choose Storage by Access Pattern

Start from the queries and the shape of the data, not the vendor. The same dataset may live in two stores (e.g. Postgres of record + Elasticsearch for search).

| Model | Best when access looks like… | Representative systems | Watch out for |
|---|---|---|---|
| Relational (SQL) | Entities with relationships, joins, ad-hoc queries, multi-row transactions | PostgreSQL, MySQL | Joins get expensive at extreme scale; rigid schema migrations |
| Document | Self-contained aggregates read/written whole; flexible schema | MongoDB, Couchbase | Cross-document joins and transactions are weak |
| Key-value | Lookup by exact key, very high throughput, simple values | Redis, DynamoDB, etcd | No querying by value; you own all access paths |
| Wide-column | Huge write volume, queries by partition + sort key, time-bucketed rows | Cassandra, HBase, ScyllaDB | Must design tables per query; no ad-hoc joins |
| Graph | Traversals over relationships (friends-of-friends, fraud rings) | Neo4j, Neptune | Sharding graphs is hard; niche operational expertise |
| Time-series | Append-only metrics/events keyed by time, downsampling, retention | InfluxDB, TimescaleDB, Prometheus | Late/out-of-order data; cardinality explosions |
| Search | Full-text, relevance ranking, faceting, fuzzy matching | Elasticsearch, OpenSearch | Eventually consistent index; not a system of record |
| Object / blob | Large immutable files, media, backups, data-lake landing zone | S3, GCS, Azure Blob | High per-request latency; no in-place edits |
| OLAP / columnar | Aggregations over billions of rows, BI dashboards, scans not point lookups | ClickHouse, BigQuery, Snowflake, Redshift | Bad at single-row updates and point reads |

Decision shortcut:

```
Need joins + transactions + ad-hoc queries?            -> Relational
Lookup by one key, latency-critical?                   -> Key-value / cache
Massive writes, known query keys, no joins?            -> Wide-column
Aggregate over huge history for analytics?             -> OLAP / columnar
Relationship traversal is the core query?              -> Graph
Full-text / relevance ranking?                         -> Search (secondary index)
Big immutable files?                                   -> Object store
```

## SQL vs NoSQL Trade-offs

"NoSQL" is not one thing — it spans document, key-value, wide-column, and graph. The real axes are: schema flexibility, join support, transactional guarantees, and horizontal scalability.

- **SQL** gives you a declarative query language, strong consistency, multi-row ACID transactions, and a planner that optimizes joins. The cost is a fixed schema and historically harder horizontal scaling. Modern engines (Postgres, MySQL with Vitess, CockroachDB, Spanner) close much of the scaling gap.
- **NoSQL** trades joins and rich querying for horizontal scale and schema flexibility. You typically pre-compute your access paths: model tables/documents around the queries you will run.

Rule of thumb: **default to a relational database.** Reach for NoSQL when a specific access pattern (write volume, key-only lookups, document aggregates, graph traversal) clearly outgrows what a tuned relational store does well. Most apps never reach that point.

### Normalization vs Denormalization

| | Normalize | Denormalize |
|---|---|---|
| Goal | Eliminate duplication, one source of truth | Optimize reads, avoid joins |
| Writes | Cheap, update one place | Costly, fan-out updates to copies |
| Reads | Need joins | Single read, pre-joined |
| Risk | Join cost at scale | Data drift / inconsistency |
| Fits | OLTP, write-heavy, correctness-critical | Read-heavy, document/wide-column, analytics |

Normalize until joins or read latency hurt, then denormalize the specific hot paths deliberately — and own the consistency story for the duplicated data (background jobs, change-data-capture, or write-time fan-out).

## ACID vs BASE

- **ACID** (Atomicity, Consistency, Isolation, Durability): transactions are all-or-nothing, leave the DB in a valid state, are isolated from each other, and survive crashes once committed. The contract of relational systems.
- **BASE** (Basically Available, Soft state, Eventually consistent): favors availability and partition tolerance; replicas converge over time. Common in distributed NoSQL.

These are endpoints of a spectrum, not a binary. Many systems are tunable (e.g. DynamoDB strong vs eventual reads; Cassandra per-query consistency levels).

### Transaction Isolation Levels

Isolation controls which concurrency anomalies are possible. Stronger isolation = more correctness, less concurrency.

| Level | Dirty read | Non-repeatable read | Phantom read |
|---|---|---|---|
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed | Prevented | Possible | Possible |
| Repeatable Read | Prevented | Prevented | Possible* |
| Serializable | Prevented | Prevented | Prevented |

\*The SQL standard allows phantoms at Repeatable Read; some engines (e.g. Postgres' snapshot-based RR, MySQL InnoDB with next-key locks) prevent most phantoms in practice.

Anomalies defined:
- **Dirty read** — you read a row another transaction wrote but has not committed (it may roll back).
- **Non-repeatable read** — you read a row twice in one transaction and get different values because another transaction committed an update between your reads.
- **Phantom read** — you re-run a range query and new rows appear because another transaction inserted matching rows.

Default for most engines is **Read Committed**. Raise to Serializable for invariants that span multiple rows (e.g. "no two bookings overlap"); expect more retries from serialization conflicts.

## Indexing

An index is a separate, sorted (or hashed) data structure that turns a full-table scan into a targeted lookup. It speeds reads and slows writes, because every insert/update/delete must also maintain the index.

| Structure | Strength | Weakness | Typical use |
|---|---|---|---|
| B-tree / B+-tree | Range + equality + ordering, balanced reads/writes | Random writes cause page splits | Default for relational indexes |
| Hash | O(1) exact-match lookups | No range queries, no ordering | Equality-only, in-memory maps |
| LSM-tree | Very fast sequential writes, good compression | Read amplification, background compaction | Write-heavy stores (Cassandra, RocksDB, LevelDB) |

**B-tree vs LSM intuition:** B-trees update in place and are read-optimized; LSM-trees buffer writes in memory and flush sorted runs to disk, merging them later via compaction — write-optimized, at the cost of reads touching multiple files (mitigated by bloom filters).

### Covering and Composite Indexes

- A **covering index** includes every column a query needs, so the engine answers from the index alone without touching the table heap ("index-only scan"). In Postgres: `INCLUDE`; in MySQL InnoDB the PK is implicitly part of secondary indexes.
- **Composite index column order matters.** An index on `(a, b, c)` serves predicates on `a`, `a,b`, and `a,b,c` — a *leftmost-prefix* rule. It does **not** efficiently serve a query filtering only on `b` or `c`.

```sql
-- Index supports these:
CREATE INDEX idx ON orders (customer_id, status, created_at);
WHERE customer_id = 42                                   -- yes
WHERE customer_id = 42 AND status = 'paid'               -- yes
WHERE customer_id = 42 AND status = 'paid'
  AND created_at > now() - interval '7 days'             -- yes, range last
-- Does NOT use idx efficiently:
WHERE status = 'paid'                                    -- skips leftmost col
```

Put equality columns first, the range/sort column last. **When indexes hurt:** write-heavy tables pay index-maintenance cost on every mutation, indexes consume storage and cache, and low-selectivity indexes (e.g. a boolean) the planner may ignore. Index for real query patterns, not speculatively.

## How a SQL Query Executes

```
SQL text
  -> Parse      (syntax check, build AST)
  -> Bind       (resolve tables/columns, types, permissions)
  -> Plan       (optimizer: enumerate join orders, index vs scan,
                 cost each using table statistics, pick cheapest)
  -> Execute    (run the plan: scan, filter, join, sort, aggregate)
  -> Results
```

The optimizer is cost-based: it relies on **statistics** (row counts, value distributions, histograms) to estimate how many rows each step yields. Stale stats produce bad plans — hence `ANALYZE`/`VACUUM`. Use `EXPLAIN ANALYZE` to see the chosen plan and whether estimates match reality (look for unexpected sequential scans, nested-loop joins over large inputs, or row-estimate mismatches).

## Replication

Replication keeps copies of data on multiple nodes for availability, read scaling, and geographic locality.

| Topology | How it works | Pros | Cons |
|---|---|---|---|
| Leader-follower | One writable leader; followers replay its log | Simple, strong-ish reads from leader | Single write bottleneck; failover gap |
| Multi-leader | Multiple writable nodes, each replicates to others | Writes near each region; survives a leader loss | Write conflicts need resolution |
| Leaderless | Any replica accepts reads/writes; quorums reconcile | Highly available, no failover | Client-side complexity, eventual consistency |

- **Synchronous replication:** the leader waits for a follower to acknowledge before confirming the commit. Stronger durability and no data loss on failover, but higher write latency and a stall if the follower is slow. Common pattern: one sync follower + several async.
- **Asynchronous replication:** the leader confirms immediately and ships changes in the background. Fast, but a leader crash can lose the last unreplicated writes; followers lag.
- **Read replicas** offload read traffic from the leader. Beware **replication lag**: a user who just wrote may read a stale follower. Mitigate with read-your-writes routing (see below).

## Partitioning / Sharding

Partitioning splits one logical dataset across many nodes so it exceeds a single machine's capacity. The **partition key** determines which shard a row lands on.

| Strategy | Mechanism | Pros | Cons |
|---|---|---|---|
| Range | Split by key ranges (A–F, G–M…) or by time | Efficient range scans | Hot spots if traffic skews to one range |
| Hash | `hash(key) % N` (or consistent hashing) | Even distribution | Range queries hit every shard |
| Directory | A lookup service maps key -> shard | Flexible rebalancing | Lookup is a dependency and SPOF risk |

- **Hot partitions:** when one key or range absorbs disproportionate traffic (a celebrity user, a single hot time bucket). Mitigate by adding a high-cardinality component to the key, salting, or splitting the hot key.
- **Resharding pain:** changing the shard count with naive `mod N` remaps almost every key. **Consistent hashing** (or pre-splitting into many virtual shards mapped onto fewer physical nodes) limits movement when nodes are added/removed. Resharding a live system means dual-writes, backfills, and careful cutover — plan the key to avoid resharding for as long as possible.

## Consistency

- **Strong consistency:** every read returns the most recent committed write. Easier to reason about; costs latency and availability under partitions (CAP).
- **Eventual consistency:** replicas converge given no new writes; reads may be stale temporarily. Cheaper and more available.
- **Read-your-writes (read-after-write):** a weaker but practical guarantee — a user always sees their *own* writes, even if others see them later. Implement by routing a user's reads to the leader for a short window after they write, or by tracking a version/timestamp the read must satisfy.

### Quorum (R + W > N)

In leaderless replication with `N` replicas, require `W` acknowledgements on write and `R` on read. If `R + W > N`, the read and write sets overlap by at least one node, so a read sees the latest acknowledged write.

```
N = 3
W = 2, R = 2  -> R + W = 4 > 3  -> overlap guaranteed (strong-ish)
W = 1, R = 1  -> R + W = 2 < 3  -> fast but may read stale data
W = 3          -> max durability, no write availability if a node is down
```

Tuning `R`/`W` trades read vs write latency vs consistency. This is exactly the knob DynamoDB and Cassandra expose.

## Everyday Query Hygiene

- **Connection pooling:** databases cap concurrent connections, and each connection is expensive (memory, backend process). A pool (HikariCP, pgbouncer, etc.) reuses a bounded set of connections instead of opening one per request. Without it, traffic spikes exhaust connections and the DB falls over. Size the pool to the DB's capacity, not the app's thread count.
- **N+1 problem:** fetching a list (1 query) then issuing one query *per item* to load a relation (N queries). Common with naive ORMs/lazy loading. Fix with a join, an `IN (...)` batch, or eager/`JOIN FETCH` loading. Watch query logs for repeated near-identical statements.
- **Avoid `SELECT *`:** pulling every column wastes I/O and network, defeats covering indexes, and silently breaks when the schema changes. Select only the columns you use.

## Analytics: OLTP vs OLAP, Warehouse vs Lake vs Lakehouse

- **OLTP** (transactional): many small, low-latency reads/writes on current data; row-oriented; the system of record.
- **OLAP** (analytical): few large aggregating queries over historical data; column-oriented; optimized for scans. Don't run heavy analytics on your OLTP primary — replicate/ETL into an analytical store.

| | Data warehouse | Data lake | Lakehouse |
|---|---|---|---|
| Stores | Structured, schema-on-write | Raw any-format, schema-on-read | Raw files + table/transaction layer |
| Tech | BigQuery, Snowflake, Redshift | S3/GCS + Parquet | Delta Lake, Iceberg, Hudi on object storage |
| Strength | Fast governed SQL/BI | Cheap, flexible, ML-friendly | Warehouse semantics on lake economics |
| Weakness | Costly, rigid ingestion | Easily becomes a "data swamp" | Younger tooling, operational complexity |

Pipelines move OLTP data to these stores via ETL/ELT or change-data-capture (CDC). Columnar formats (Parquet, ORC) and columnar engines make wide aggregations cheap because a query reads only the referenced columns.

## Caching

A cache is a faster, smaller copy of hot data that sits in front of the database to cut latency and read load — the single highest-leverage performance move for read-heavy systems. Cache strategies (cache-aside, write-through, write-back), invalidation, and TTL design are covered in `caching-and-performance.md`. The database section's relevant point: caching changes your consistency model (cached data can be stale), so decide acceptable staleness per use case.

## Storage Scaling Story

The progression most teams follow, in order, applying each step only when the previous one is exhausted:

```
1. Vertical scaling      -> bigger box (CPU/RAM/SSD). Simple, no app changes.
                            Buys a lot of runway; eventually hits a ceiling + cost wall.
2. Read replicas         -> route reads to async followers, writes to leader.
                            Solves read-heavy load. Introduces replication lag.
3. Caching + tuning      -> cache hot reads, add/fix indexes, kill N+1, denormalize hot paths.
4. Functional split      -> move large/independent tables to their own DB (vertical partitioning).
5. Sharding              -> horizontally partition a table by key across many nodes.
                            Last resort: app/operational complexity jumps sharply.
```

A widely-cited real-world example is scaling a single PostgreSQL instance a long way before sharding: first vertical upgrades, then read replicas, then splitting tables onto separate database instances by domain, and only later partitioning the largest tables horizontally — keeping the relational model and its transactions intact for years. The general lesson: **a well-tuned single relational database with replicas goes much further than people assume.** Exhaust the cheap, low-complexity steps before sharding, because sharding is the hardest step to reverse and it taxes every future feature.

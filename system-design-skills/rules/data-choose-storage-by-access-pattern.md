---
title: Choose Storage by Access Pattern
impact: CRITICAL
impactDescription: the data store you pick constrains everything you can do later
tags: data, databases, storage, modeling
---

## Choose Storage by Access Pattern

Pick a data store from how the data is *queried and written*, not from
familiarity or hype. The access pattern — read/write ratio, query shape,
consistency need, scale — should drive the choice.

**Context:** Selecting a database or storage system for a new service or feature.

**Heuristic:** Write down the top queries and the consistency requirement first. Then choose the simplest store that serves them. Default to a relational database until a concrete access pattern proves it wrong.

| Need | Reach for |
|------|-----------|
| Relational data, transactions, ad-hoc queries | PostgreSQL / MySQL |
| High-volume key lookups, caching | Redis / Memcached |
| Flexible documents, denormalized reads | MongoDB / DynamoDB |
| Massive write throughput, time/partition keys | Cassandra / wide-column |
| Full-text / fuzzy search | Elasticsearch / OpenSearch |
| Analytics over large datasets | ClickHouse / BigQuery / Snowflake |
| Large files / blobs | S3 / object storage |

**Incorrect:** Forcing a graph traversal or full-text search into a relational table because "we already have Postgres," then fighting the query planner forever.

**Correct:** Keep the system of record relational, and add a purpose-built store (search index, cache, OLAP warehouse) for access patterns it serves poorly — fed by replication or events.

**Why:** Storage choice is one of the hardest decisions to reverse — migrations are expensive and risky. The query pattern, not team habit, determines which store stays fast as data grows.

Reference: `references/databases-and-storage.md`

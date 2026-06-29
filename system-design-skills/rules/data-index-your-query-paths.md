---
title: Index Your Query Paths
impact: HIGH
impactDescription: a missing index turns a fast lookup into a full table scan
tags: data, databases, indexing, performance
---

## Index Your Query Paths

Add indexes for the columns you actually filter, join, and sort on — but only
those. Every index speeds reads and slows writes, so index deliberately.

**Context:** Designing a schema or diagnosing a slow query.

**Heuristic:** Index the columns in your `WHERE`, `JOIN`, and `ORDER BY` clauses, especially foreign keys. Match composite-index column order to your query's filter order (most selective / equality columns first). Verify with `EXPLAIN`.

**Incorrect:**

```sql
-- Frequent query, no supporting index → sequential scan of millions of rows
SELECT * FROM orders WHERE customer_id = 42 ORDER BY created_at DESC;
```

**Correct:**

```sql
CREATE INDEX idx_orders_customer_created
  ON orders (customer_id, created_at DESC);   -- serves the filter AND the sort
-- and select only needed columns, not *
SELECT id, total, created_at FROM orders WHERE customer_id = 42 ORDER BY created_at DESC;
```

**Watch for:**
- Unindexed foreign keys (slow joins, slow cascade deletes).
- Over-indexing: each index adds write cost and storage; drop unused ones.
- `SELECT *` defeats covering indexes — request only the columns you need.

**Why:** Without the right index the database scans the whole table; latency grows linearly with data and collapses under load. The fix is cheap, but only if it matches how you actually query.

Reference: `references/databases-and-storage.md`

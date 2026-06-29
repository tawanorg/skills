---
title: Paginate Large Collections
impact: HIGH
impactDescription: prevents unbounded responses that exhaust memory and latency budgets
tags: api, pagination, performance, scalability
---

## Paginate Large Collections

Never return an unbounded list. Any collection that can grow must be paginated,
with a sane default and maximum page size enforced server-side.

**Context:** Designing any list/search endpoint whose result set grows over time.

**Heuristic:** Prefer cursor (keyset) pagination over offset for large or fast-changing data. Cap page size on the server; ignore client requests for more.

**Incorrect:**

```http
GET /v1/orders            # returns all 4 million orders → OOM, timeout, dead client
GET /v1/orders?offset=2000000&limit=100   # offset scans + discards 2M rows; slow and drifts as data changes
```

**Correct:**

```http
GET /v1/orders?limit=100                    # default + max enforced server-side
GET /v1/orders?limit=100&cursor=eyJpZCI6...  # keyset: WHERE id > :last_id ORDER BY id LIMIT 100
```

| Strategy | Pros | Cons |
|----------|------|------|
| Offset/limit | Simple, jump to any page | Slow at high offsets, skips/dupes on inserts |
| Cursor/keyset | Stable, fast at any depth | No random page access |

**Why:** Unbounded responses turn one slow client into a memory and latency incident for everyone. Offset pagination degrades linearly with depth and returns inconsistent results when the underlying data changes mid-scroll.

Reference: `references/api-and-web-development.md`

---
title: Pick Consistency Per Use Case
impact: HIGH
impactDescription: matching the consistency model to the domain avoids both bugs and needless cost
tags: scale, distributed, consistency, cap, data
---

## Pick Consistency Per Use Case

Strong consistency is not always required, and it is never free. Choose the
weakest consistency model that is still correct for each piece of data, rather
than applying one global setting everywhere.

**Context:** Designing reads/writes in a system with replicas, caches, or multiple services.

**Heuristic:** Ask "what's the cost of a user reading slightly stale data here?" If the answer is "nothing serious," eventual consistency buys you availability and latency. If the answer is "money or safety," choose strong/CP.

| Data | Model | Why |
|------|-------|-----|
| Account balance, inventory count, payments | Strong / CP | A wrong read causes overspend / oversell |
| Social feed, likes, view counts, recommendations | Eventual / AP | Slight staleness is invisible to users |
| User profile, settings | Read-your-writes | Users must see their own edits immediately |
| Analytics, search index | Eventual | Bulk/near-real-time is fine |

Per the **CAP theorem**, when a network partition happens you must choose
availability or consistency — so decide *per data type*, not once for the whole
system.

**Why:** Forcing strong consistency everywhere caps throughput and hurts availability for data that never needed it; allowing eventual consistency on money or inventory causes real correctness bugs. The right model is domain-specific.

Reference: `references/scalability-and-distributed-systems.md`, `references/databases-and-storage.md`

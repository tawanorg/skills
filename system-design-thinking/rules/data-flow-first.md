---
title: Data Flow First
impact: HIGH
impactDescription: ensures architecture matches how data actually moves
tags: data, architecture, flow, design
---

## Data Flow First

Understand how data flows through the system before designing components. Data flow reveals the true architecture.

**Context:** Starting system design or decomposing an existing system.

**Heuristic:** Before drawing service boxes, map the data:

1. **What data exists?** - Entities, relationships, volumes
2. **Where does it originate?** - Sources, entry points
3. **How does it transform?** - Processing, enrichment, aggregation
4. **Where does it go?** - Destinations, consumers
5. **Who owns it?** - Single source of truth for each entity

**Data questions to answer:**

| Question | Why It Matters |
|----------|----------------|
| How much data? | Storage and scaling decisions |
| How fast does it grow? | Capacity planning |
| How fresh must it be? | Sync vs async, caching |
| Who reads vs writes? | Read/write scaling |
| What are the access patterns? | Database and index design |
| What's sensitive? | Security and compliance |

**Apply when:**
- Starting any system design
- Data is central to the system (most systems)
- Confusion about service boundaries

**Skip when:**
- Building stateless utilities
- Data flow is trivial and obvious

**Why:** Components are just data transformers. If you don't understand the data, you'll draw arbitrary boxes that don't match reality. Data outlives code.

**Example:**

```
❌ Starting with components:
   "We need a User Service, Order Service, and Notification Service"
   (But how does order data get to notifications? Who owns user preferences?)

✅ Starting with data flow:
   "Let's map the order flow:

   1. User places order (web/mobile) → Order created
   2. Order needs: user info, product info, inventory check
   3. Payment processed → Order confirmed
   4. Inventory decremented
   5. Notification sent (email, push)
   6. Order data needed for: reporting, customer service, returns

   Data ownership:
   - User service owns user data
   - Product service owns catalog
   - Order service owns orders (source of truth)
   - Inventory is updated by orders but owned by warehouse

   Now we can draw boundaries that respect data ownership."
```

**Data flow diagram template:**

```
[Source] ──► [Transform] ──► [Store] ──► [Consume]
    │            │             │            │
    └── What?    └── How?      └── Where?   └── Who?
```

---
title: Prevent N+1 Query Problems
impact: HIGH
impactDescription: prevents exponential database load
tags: database, performance, queries
---

## Prevent N+1 Query Problems

Never execute queries inside loops. Use batch fetching or eager loading instead.

**Incorrect:**

```typescript
// N+1 problem: 1 query for orders + N queries for users
const orders = await db.orders.findAll();
for (const order of orders) {
  order.user = await db.users.findById(order.userId);  // Query per order!
}
```

**Correct:**

```typescript
// Eager loading: 1-2 queries total
const orders = await db.orders.findAll({
  include: [{ model: User, as: 'user' }],
});

// Or batch fetching
const orders = await db.orders.findAll();
const userIds = [...new Set(orders.map(o => o.userId))];
const users = await db.users.findAll({ where: { id: userIds } });
const userMap = new Map(users.map(u => [u.id, u]));
orders.forEach(o => o.user = userMap.get(o.userId));
```

**Why:** N+1 queries turn a 2ms operation into a 2000ms operation at scale.

Reference: `references/database-patterns.md`

---
title: Never Use SELECT * in Production
impact: HIGH
impactDescription: prevents over-fetching and breaks on schema changes
tags: database, performance, sql
---

## Never Use SELECT * in Production

Always specify the columns you need. Never use `SELECT *` in production code.

**Incorrect:**

```typescript
// Fetches all columns, including ones you don't need
const users = await db.query('SELECT * FROM users WHERE id = $1', [id]);

// ORM equivalent
const user = await User.findById(id);  // Returns entire row
```

**Correct:**

```typescript
// Only fetch needed columns
const users = await db.query(
  'SELECT id, email, name FROM users WHERE id = $1',
  [id]
);

// ORM with explicit selection
const user = await User.findById(id).select('id email name');
```

**Why:** `SELECT *` fetches unnecessary data, breaks when columns are added/removed, and can expose sensitive fields.

Reference: `references/database-patterns.md`

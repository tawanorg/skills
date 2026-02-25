---
title: Always Index Foreign Keys
impact: HIGH
impactDescription: prevents full table scans on joins
tags: database, performance, indexing
---

## Always Index Foreign Keys

Every foreign key column must have an index. Without it, JOINs and lookups cause full table scans.

**Incorrect:**

```sql
-- Foreign key without index - JOIN will be slow
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP NOT NULL
);
-- Missing: CREATE INDEX idx_orders_user_id ON orders(user_id);
```

**Correct:**

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP NOT NULL
);

-- Always index foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Also index columns used in WHERE/ORDER BY
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

**Why:** Without an index, `WHERE user_id = X` scans every row in the table.

Reference: `references/database-patterns.md`

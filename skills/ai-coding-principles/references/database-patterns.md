# Database Patterns

Schema design, query optimization, and migration strategies.

## Schema Design

### Naming Conventions

```sql
-- Tables: plural, snake_case
users, order_items, payment_methods

-- Columns: snake_case
first_name, created_at, is_active

-- Primary keys: id (or table_id for clarity)
users.id, orders.id

-- Foreign keys: singular_table_id
orders.user_id, order_items.product_id

-- Timestamps: created_at, updated_at, deleted_at
```

### Required Columns

```sql
CREATE TABLE orders (
    -- Primary key
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- Business columns
    user_id UUID NOT NULL REFERENCES users(id),
    status VARCHAR(50) NOT NULL DEFAULT 'pending',
    total_cents INTEGER NOT NULL,

    -- Audit columns (always include)
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

    -- Soft delete (when needed)
    deleted_at TIMESTAMP WITH TIME ZONE
);

-- Auto-update updated_at
CREATE TRIGGER update_orders_updated_at
    BEFORE UPDATE ON orders
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

### Data Types

| Store | Use | Not |
|-------|-----|-----|
| Money | INTEGER (cents) | FLOAT, DECIMAL |
| UUID | UUID type | VARCHAR(36) |
| Timestamps | TIMESTAMP WITH TIME ZONE | TIMESTAMP |
| Booleans | BOOLEAN | TINYINT, CHAR(1) |
| Enums | VARCHAR with CHECK | Native ENUM (migration pain) |
| JSON | JSONB (Postgres) | JSON, TEXT |

### Enum Handling

```sql
-- Prefer CHECK constraints over native ENUMs
CREATE TABLE orders (
    status VARCHAR(50) NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))
);

-- Adding a value is just an ALTER
ALTER TABLE orders DROP CONSTRAINT orders_status_check;
ALTER TABLE orders ADD CONSTRAINT orders_status_check
    CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded'));
```

## Indexing Strategy

### When to Index

```sql
-- Primary keys: Automatic
-- Foreign keys: Always index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Frequently filtered columns
CREATE INDEX idx_users_email ON users(email);

-- Frequently sorted columns
CREATE INDEX idx_orders_created_at ON orders(created_at);

-- Composite for common query patterns
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

### Partial Indexes

```sql
-- Index only active records (smaller, faster)
CREATE INDEX idx_active_users_email ON users(email)
    WHERE deleted_at IS NULL;

-- Index only specific statuses
CREATE INDEX idx_pending_orders ON orders(created_at)
    WHERE status = 'pending';
```

### Index Anti-Patterns

```sql
-- BAD: Index on low-cardinality column alone
CREATE INDEX idx_users_is_active ON users(is_active);  -- Only 2 values!

-- BAD: Too many indexes (slows writes)
CREATE INDEX idx_users_first_name ON users(first_name);
CREATE INDEX idx_users_last_name ON users(last_name);
CREATE INDEX idx_users_created_at ON users(created_at);
-- ... every column indexed

-- GOOD: Composite index for common query pattern
CREATE INDEX idx_users_name ON users(last_name, first_name);
```

## Query Patterns

### N+1 Problem

```typescript
// BAD: N+1 queries
const orders = await db.orders.findAll();
for (const order of orders) {
  order.user = await db.users.findById(order.userId);  // N queries!
}

// GOOD: Eager loading
const orders = await db.orders.findAll({
  include: [{ model: User, as: 'user' }],
});

// GOOD: Batch fetching
const orders = await db.orders.findAll();
const userIds = [...new Set(orders.map(o => o.userId))];
const users = await db.users.findAll({ where: { id: userIds } });
const userMap = new Map(users.map(u => [u.id, u]));
orders.forEach(o => o.user = userMap.get(o.userId));
```

### Pagination

```typescript
// BAD: Offset pagination at scale
SELECT * FROM orders ORDER BY id LIMIT 20 OFFSET 10000;
-- Must scan 10,020 rows!

// GOOD: Cursor pagination
SELECT * FROM orders
WHERE id > $lastSeenId
ORDER BY id
LIMIT 20;
-- Efficient at any depth
```

### Avoiding SELECT *

```typescript
// BAD: Fetching all columns
const users = await db.query('SELECT * FROM users WHERE id = $1', [id]);

// GOOD: Select only needed columns
const users = await db.query(
  'SELECT id, email, name FROM users WHERE id = $1',
  [id]
);
```

## Transactions

### When to Use

```typescript
// Use transactions when multiple operations must succeed or fail together

// GOOD: Atomic operation
async function transferFunds(fromId: string, toId: string, amount: number) {
  await db.transaction(async (tx) => {
    const from = await tx.accounts.findById(fromId, { lock: true });

    if (from.balance < amount) {
      throw new InsufficientFundsError();
    }

    await tx.accounts.update(fromId, { balance: from.balance - amount });
    await tx.accounts.update(toId, { balance: { increment: amount } });

    await tx.transfers.create({
      fromAccountId: fromId,
      toAccountId: toId,
      amount,
    });
  });
}
```

### Isolation Levels

| Level | Use Case |
|-------|----------|
| READ COMMITTED | Default, most operations |
| REPEATABLE READ | Reports, consistent reads |
| SERIALIZABLE | Financial transactions |

## Migrations

### Migration Best Practices

```typescript
// 1. Migrations must be reversible
export async function up(db) {
  await db.schema.createTable('orders', (t) => {
    t.uuid('id').primary();
    t.uuid('user_id').notNullable().references('users.id');
    t.timestamps();
  });
}

export async function down(db) {
  await db.schema.dropTable('orders');
}

// 2. Never modify existing migrations
// Create a new migration instead

// 3. Large data migrations: batched
export async function up(db) {
  let affected = 1;
  while (affected > 0) {
    affected = await db.raw(`
      UPDATE users
      SET status = 'active'
      WHERE status IS NULL
      AND id IN (
        SELECT id FROM users WHERE status IS NULL LIMIT 1000
      )
    `);
  }
}
```

### Zero-Downtime Schema Changes

```typescript
// Adding a column
// 1. Add nullable column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

// 2. Backfill data (batched)
// 3. Add NOT NULL constraint
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;

// Renaming a column
// 1. Add new column
// 2. Dual-write to both
// 3. Backfill old data
// 4. Switch reads to new column
// 5. Stop writing to old column
// 6. Drop old column (later migration)
```

## Connection Management

### Connection Pooling

```typescript
const pool = new Pool({
  host: process.env.DB_HOST,
  max: 20,                    // Max connections
  min: 5,                     // Min connections
  idleTimeoutMillis: 30000,   // Close idle connections
  connectionTimeoutMillis: 5000,
});

// Monitor pool health
setInterval(() => {
  const { totalCount, idleCount, waitingCount } = pool;
  metrics.gauge('db_connections_total', totalCount);
  metrics.gauge('db_connections_idle', idleCount);
  metrics.gauge('db_connections_waiting', waitingCount);
}, 5000);
```

## Database Checklist

### Schema Design

- [ ] Tables use plural snake_case names
- [ ] All tables have id, created_at, updated_at
- [ ] Foreign keys are indexed
- [ ] Money stored as integers (cents)
- [ ] Timestamps include timezone

### Queries

- [ ] No SELECT * in production code
- [ ] N+1 queries eliminated
- [ ] Large result sets paginated
- [ ] Indexes exist for WHERE/ORDER BY columns

### Operations

- [ ] Migrations are reversible
- [ ] Connection pool configured
- [ ] Query timeouts set
- [ ] Slow query logging enabled

---
title: Always Use Parameterized Queries
impact: CRITICAL
impactDescription: prevents SQL injection attacks
tags: security, database, sql-injection
---

## Always Use Parameterized Queries

Never concatenate user input into SQL queries. Always use parameterized queries or prepared statements.

**Incorrect:**

```typescript
// SQL injection vulnerability - attacker can manipulate query
const query = `SELECT * FROM users WHERE id = '${userId}'`;
const result = await db.query(query);
```

**Correct:**

```typescript
// Parameterized query - input is safely escaped
const query = 'SELECT * FROM users WHERE id = $1';
const result = await db.query(query, [userId]);

// Or with ORM
const user = await User.findOne({ where: { id: userId } });
```

**Why:** SQL injection is a top OWASP vulnerability that can lead to data theft, data loss, or complete system compromise.

Reference: `references/security-checklist.md`

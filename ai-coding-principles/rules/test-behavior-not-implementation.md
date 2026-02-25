---
title: Test Behavior, Not Implementation
impact: MEDIUM
impactDescription: prevents brittle tests that break on refactoring
tags: testing, unit-tests, design
---

## Test Behavior, Not Implementation

Tests should verify what code does, not how it does it. Don't test internal implementation details.

**Incorrect:**

```typescript
// Testing implementation details - breaks when refactoring
it('calls the database with correct query', () => {
  await userService.getUser(123);
  expect(mockDb.query).toHaveBeenCalledWith(
    'SELECT * FROM users WHERE id = $1',  // Breaks if query changes
    [123]
  );
});

it('uses the cache before database', () => {
  await userService.getUser(123);
  expect(mockCache.get).toHaveBeenCalledBefore(mockDb.query);  // Implementation detail
});
```

**Correct:**

```typescript
// Testing behavior - survives refactoring
it('returns user when found', async () => {
  const user = await userService.getUser(123);
  expect(user.id).toBe(123);
  expect(user.email).toBe('test@example.com');
});

it('returns null when user not found', async () => {
  const user = await userService.getUser(999);
  expect(user).toBeNull();
});

it('throws error for invalid id', async () => {
  await expect(userService.getUser(-1)).rejects.toThrow('Invalid user ID');
});
```

**Why:** Implementation-focused tests break constantly during refactoring, providing no safety net.

Reference: `references/testing-patterns.md`

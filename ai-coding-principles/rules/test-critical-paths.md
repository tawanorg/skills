---
title: Prioritize Testing Critical Paths
impact: MEDIUM
impactDescription: maximizes test ROI
tags: testing, strategy, coverage
---

## Prioritize Testing Critical Paths

Focus testing effort on code that matters most: authentication, payments, data persistence, core business rules.

**Incorrect:**

```typescript
// Over-testing trivial code
it('getName returns name', () => {
  const user = new User('John');
  expect(user.getName()).toBe('John');  // Trivial getter
});

it('adds class to button', () => {
  render(<Button primary />);
  expect(screen.getByRole('button')).toHaveClass('primary');  // CSS detail
});

// While ignoring critical paths...
// No tests for: login, payment processing, data validation
```

**Correct:**

```typescript
// Test critical paths thoroughly
describe('Authentication', () => {
  it('rejects invalid credentials', async () => {
    await expect(auth.login('user', 'wrong')).rejects.toThrow('Invalid credentials');
  });

  it('locks account after 5 failed attempts', async () => {
    for (let i = 0; i < 5; i++) {
      await auth.login('user', 'wrong').catch(() => {});
    }
    await expect(auth.login('user', 'correct')).rejects.toThrow('Account locked');
  });
});

describe('Payment Processing', () => {
  it('creates charge and records transaction', async () => { /* ... */ });
  it('handles payment failure gracefully', async () => { /* ... */ });
  it('prevents duplicate charges', async () => { /* ... */ });
});
```

**Why:** Tests on trivial code waste time; missing tests on critical code cause production incidents.

Reference: `references/testing-patterns.md`

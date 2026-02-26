---
title: Write Regression Test
impact: high
tags: [debugging, testing, prevention]
---

# Write Regression Test

**Every bug fix should come with a test that would have caught it.**

## The Rule

After fixing a bug:
1. Write a test that reproduces the bug
2. Verify the test fails on the old code
3. Verify the test passes on the fixed code
4. Commit the test with the fix

## Why This Matters

```
Without regression test:
- Same bug can return
- Refactoring may reintroduce it
- Team learns nothing from the bug
- Cost: Debugging time × number of recurrences

With regression test:
- Bug can never return unnoticed
- Documents the edge case
- Protects future refactoring
- Cost: 10 minutes once
```

## Incorrect Approach

```typescript
// Bug: Login fails for emails with + character
// Fix: Update email regex

// Developer:
// 1. Fixes the regex
// 2. Tests manually once
// 3. Commits: "Fix email validation"
// 4. Moves on

// 6 months later:
// Different developer refactors validation
// Same bug reappears
// Users can't log in again
// Time lost: 4 hours debugging (again)
```

## Correct Approach

```typescript
// Bug: Login fails for emails with + character

// Step 1: Write failing test FIRST
describe('email validation', () => {
  it('accepts emails with + character', () => {
    // This is the bug we're fixing
    const result = validateEmail('user+tag@example.com');
    expect(result.valid).toBe(true);
  });
});

// Step 2: Run test - it fails (confirms bug)
// FAIL: Expected true, got false

// Step 3: Fix the code
function validateEmail(email: string): ValidationResult {
  // Fixed: Added + to allowed characters
  const regex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
  return { valid: regex.test(email) };
}

// Step 4: Run test - it passes (confirms fix)
// PASS

// Step 5: Commit both together
git add src/validation.ts src/validation.test.ts
git commit -m "Fix email validation for + character

Emails with + (like user+tag@example.com) were being
rejected due to missing + in regex character class.

Added regression test to prevent recurrence.

Fixes: BUG-1234"
```

## What Makes a Good Regression Test

```typescript
// Good regression test characteristics:

// 1. SPECIFIC - Tests exact bug scenario
it('handles empty array when user has no orders', () => {
  const user = createUser({ orders: [] });
  expect(() => renderOrderHistory(user)).not.toThrow();
});

// 2. MINIMAL - Tests one thing
// Not: "test user rendering" (too broad)
// Yes: "test empty orders array" (specific case)

// 3. NAMED DESCRIPTIVELY - Future readers understand
it('does not crash on undefined profile.settings', () => {
  // The test name documents the bug
});

// 4. INCLUDES CONTEXT - Comment with ticket reference
it('calculates tax correctly for zero-dollar orders', () => {
  // Bug: BUG-5678 - Division by zero when order total is $0
  const order = createOrder({ total: 0 });
  expect(calculateTax(order)).toBe(0);
});
```

## Test Placement

```
Where to put the regression test:

Unit test (most common):
└── Tests the specific function that was buggy

Integration test:
└── When bug involves multiple components

E2E test:
└── When bug is user-facing flow

All levels:
└── Critical bugs may warrant tests at multiple levels
```

## Template for Bug Fix PR

```markdown
## Bug Fix: [Description]

### What was the bug?
Emails with + character were rejected during validation.

### Root cause
Regex character class was missing + as allowed character.

### Fix
Added + to the character class in email regex.

### Testing
- [x] Added regression test: `it('accepts emails with + character')`
- [x] Test fails on main branch
- [x] Test passes with this fix
- [x] All existing tests pass

### References
- Ticket: BUG-1234
- User report: [link]
```

## The Test-First Debug Flow

```
1. Bug reported
       ↓
2. Reproduce bug
       ↓
3. Write failing test   ← Proves understanding
       ↓
4. Fix the code
       ↓
5. Test passes          ← Proves fix works
       ↓
6. Commit together
       ↓
7. Bug never returns
```

This is essentially TDD applied to bug fixing.

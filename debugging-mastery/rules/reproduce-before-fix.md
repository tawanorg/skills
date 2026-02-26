---
title: Reproduce Before Fix
impact: critical
tags: [debugging, methodology, fundamentals]
---

# Reproduce Before Fix

**Never attempt to fix a bug you cannot reliably reproduce.**

## The Rule

Before writing any fix:
1. Create a reliable reproduction
2. Document exact steps
3. Verify you can trigger it consistently

## Why This Matters

```
Without reproduction:
- You're guessing at the cause
- You can't verify the fix
- You can't write a regression test
- The bug will likely return

With reproduction:
- You understand the trigger
- You can verify the fix works
- You have a ready-made test
- Future regressions are caught
```

## Incorrect Approach

```typescript
// Bug report: "Users sometimes can't log in"
// Developer: "I'll add a null check, that should fix it"

function login(user) {
  if (!user) return null;  // Random defensive code
  // ... rest of login
}

// Problems:
// - Don't know if this is the actual cause
// - Can't verify it fixes the issue
// - Might hide a deeper problem
// - No test to prevent regression
```

## Correct Approach

```typescript
// Bug report: "Users sometimes can't log in"

// Step 1: Gather information
// - Which users? New or existing?
// - What error do they see?
// - When did it start?
// - What's in the logs?

// Step 2: Reproduce
// Found: Happens when user has special characters in password
// Steps:
//   1. Create user with password "test@123#"
//   2. Log out
//   3. Try to log in
//   4. See error: "Invalid credentials"

// Step 3: Verify reproduction
// Run steps 3 times - fails consistently

// Step 4: Write failing test
test('login handles special characters in password', async () => {
  const user = await createUser({ password: 'test@123#' });
  const result = await login(user.email, 'test@123#');
  expect(result.success).toBe(true);  // Currently fails
});

// Step 5: Now find and fix the actual cause
// Found: Password wasn't being URL-encoded before API call

// Step 6: Fix passes test
// Step 7: Test becomes permanent regression test
```

## Reproduction Strategies

| Situation | Strategy |
|-----------|----------|
| Can't reproduce locally | Get exact environment (Docker, VM) |
| Intermittent failure | Increase iterations, add stress |
| User-specific | Get their exact data, account state |
| Time-dependent | Mock time, simulate conditions |
| Production-only | Add logging, capture state |

## The Minimal Reproduction

Always aim for the smallest possible reproduction:

```
Full scenario:                 Minimal repro:
1. Create account             1. Call function with
2. Add payment method            specific input
3. Add items to cart          2. Observe failure
4. Apply coupon
5. Checkout
6. See error

Minimal repro is:
- Faster to run
- Easier to understand
- Better test case
- Often reveals root cause
```

## When You Can't Reproduce

If you genuinely cannot reproduce after significant effort:

1. Add detailed logging around suspected area
2. Add metrics to track occurrence
3. Create monitoring alert
4. Wait for more data
5. Do NOT deploy speculative fixes

---
title: Verify the Fix
impact: critical
tags: [debugging, verification, testing]
---

# Verify the Fix

**A fix isn't done until it's proven to work.**

## The Rule

After every fix:
1. Run the original reproduction steps
2. Verify the bug no longer occurs
3. Check that nothing else broke
4. Run the full test suite

## Why This Matters

```
Unverified fixes:
- May not actually fix the bug
- May fix symptom but not cause
- May introduce new bugs
- May break other functionality

Cost of shipping unverified fix: Hours to days of rework
Cost of verifying: Minutes
```

## Incorrect Approach

```typescript
// Bug: Users can't log in
// Developer: "I think I see the issue..."

// Makes change
if (user.password === hashedPassword) {  // Changed === to !==? No wait...
  return true;
}

// "That should fix it"
// Commits without testing
// Ships to production
// Bug still exists (or is now worse)
```

## Correct Approach

```typescript
// Bug: Users can't log in

// Step 1: Reproduce the bug
// - Go to login page
// - Enter valid credentials
// - Click login
// - See "Invalid credentials" error

// Step 2: Make the fix
const isValid = await bcrypt.compare(password, user.passwordHash);
// Was: password === user.passwordHash (plaintext comparison)

// Step 3: Verify the fix
// - Go to login page
// - Enter same valid credentials
// - Click login
// - Successfully logged in ✓

// Step 4: Check edge cases
// - Wrong password → Shows error ✓
// - Empty password → Shows error ✓
// - SQL injection attempt → Rejected ✓

// Step 5: Run test suite
npm test
// All tests pass ✓

// Step 6: Now commit
git commit -m "Fix login: use bcrypt compare instead of plaintext"
```

## Verification Checklist

```markdown
## Before Committing Fix

### Bug Verification
- [ ] Original reproduction steps no longer trigger bug
- [ ] Tested 3+ times to ensure not flaky
- [ ] Tested in same environment where bug was found

### Regression Check
- [ ] Related functionality still works
- [ ] Edge cases handled correctly
- [ ] Test suite passes
- [ ] No new warnings or errors in logs

### Code Quality
- [ ] Fix addresses root cause (not just symptom)
- [ ] Fix is minimal and focused
- [ ] Fix follows codebase patterns
- [ ] Fix includes/updates tests
```

## Levels of Verification

```
Level 1: Manual smoke test
└── Run the reproduction steps

Level 2: Manual edge cases
└── Test boundary conditions, error cases

Level 3: Automated tests
└── Unit tests for the fix

Level 4: Integration tests
└── Tests with real dependencies

Level 5: Production validation
└── Canary deploy, metrics monitoring
```

## Verification by Bug Type

| Bug Type | Verification Steps |
|----------|-------------------|
| Logic error | Test correct and incorrect inputs |
| Race condition | Run under load, multiple times |
| Memory leak | Profile memory over extended run |
| Performance | Benchmark before and after |
| UI bug | Visual inspection + automated screenshot |
| Security | Attempt the vulnerability again |

## The "It Works on My Machine" Problem

```markdown
Fix works locally but not in production?

Check differences:
- [ ] Environment variables
- [ ] Database content
- [ ] Configuration files
- [ ] Dependency versions
- [ ] OS/runtime version
- [ ] Network conditions
- [ ] User permissions
- [ ] Data volume

Verify in production-like environment before shipping.
```

## Verification Anti-patterns

```typescript
// Anti-pattern 1: "Trust me"
// "I know this will work, no need to test"
// Always test.

// Anti-pattern 2: "I tested something similar"
// "I tested login, so logout probably works too"
// Test the actual bug.

// Anti-pattern 3: "The tests pass"
// Tests may not cover the bug scenario
// Verify manually AND with tests.

// Anti-pattern 4: "I'll verify in production"
// That's called "users are QA"
// Verify before production.
```

## When Verification Fails

If the fix doesn't work:

1. Don't panic
2. Don't add more changes on top
3. Revert to known state
4. Re-examine your understanding of the bug
5. Form new hypothesis
6. Try again with single change

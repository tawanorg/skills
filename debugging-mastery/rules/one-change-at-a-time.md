---
title: One Change at a Time
impact: high
tags: [debugging, methodology, scientific-method]
---

# One Change at a Time

**Change only one variable per experiment.**

## The Rule

When testing hypotheses:
1. Make exactly one change
2. Test the result
3. Revert if it didn't help
4. Then try the next change

## Why This Matters

```
Multiple simultaneous changes:
- Can't tell which one fixed it
- Can't tell which one broke it more
- Might have changes that cancel out
- Introduces new bugs while fixing others

Single changes:
- Clear cause and effect
- Understand why it worked
- Can explain the fix
- Build systematic knowledge
```

## Incorrect Approach

```typescript
// Bug: API returning wrong data

// Developer makes 5 changes at once:
function getUser(id) {
  // Change 1: Added validation
  if (!id) return null;

  // Change 2: Changed query
  const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);

  // Change 3: Added caching
  await cache.set(id, user);

  // Change 4: Changed response format
  return { data: user, timestamp: Date.now() };

  // Change 5: Added logging
  logger.info('User fetched', { id });
}

// Result: Bug is fixed
// Problem: Which change fixed it? Were all changes necessary?
// Unknown: Did any change introduce new bugs?
```

## Correct Approach

```typescript
// Bug: API returning wrong data

// Hypothesis 1: Invalid ID being passed
// Change: Add validation only
function getUser(id) {
  if (!id) return null;  // Only this change
  // ... rest unchanged
}
// Test: Still broken → Revert, try next hypothesis

// Hypothesis 2: Query is wrong
// Change: Fix query only
function getUser(id) {
  // Changed from: 'SELECT * FROM users WHERE user_id = ?'
  const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
  // ... rest unchanged
}
// Test: Fixed! → Keep this change, document root cause

// Root cause: Column was 'id' not 'user_id'
// One change, clear understanding
```

## The Experimental Method

```
┌─────────────────────────────────────────────────────────────┐
│                    SINGLE VARIABLE TESTING                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  State: [A] [B] [C] [D] [E]  ← Current (broken)             │
│                                                             │
│  Test 1: [A'] [B] [C] [D] [E]  ← Change only A              │
│          Result: Still broken                               │
│          Action: Revert A' → A                              │
│                                                             │
│  Test 2: [A] [B'] [C] [D] [E]  ← Change only B              │
│          Result: Fixed!                                     │
│          Action: Keep B', document                          │
│                                                             │
│  Conclusion: B was the problem                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Practical Examples

### Debugging Configuration

```yaml
# Don't: Change everything at once
database:
  host: new-host          # Changed
  port: 5433              # Changed
  pool_size: 20           # Changed
  timeout: 60000          # Changed

# Do: Change one at a time
# Test 1: Only host
database:
  host: new-host          # Changed
  port: 5432              # Original
  pool_size: 10           # Original
  timeout: 30000          # Original
# Test → Still broken? Revert, try port
```

### Debugging Code

```typescript
// Don't: Rewrite the function
// Do: Change one operation at a time

// Original (broken):
const result = data.filter(x => x.active).map(x => x.id).sort();

// Test 1: Is filter correct?
const filtered = data.filter(x => x.active);
console.log('After filter:', filtered);
// Looks correct → filter is fine

// Test 2: Is map correct?
const mapped = filtered.map(x => x.id);
console.log('After map:', mapped);
// Shows undefined → map is the problem → x.id should be x.userId
```

## When to Break the Rule

Rarely, multiple changes are necessary:

1. **Atomic changes**: Changes that must go together (rename + update references)
2. **Blocked state**: Current state prevents any testing
3. **Time-critical production**: Emergency mitigation (but document everything)

Even then, minimize and document every change.

## The Revert Discipline

```bash
# Before each experiment
git stash  # or commit current state

# After failed experiment
git stash pop  # or git checkout .

# Keep a clean baseline to return to
# Never accumulate "maybe this helps" changes
```

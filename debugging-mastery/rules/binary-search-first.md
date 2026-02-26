---
title: Binary Search First
impact: high
tags: [debugging, efficiency, methodology]
---

# Binary Search First

**Halve the problem space with each step instead of searching linearly.**

## The Rule

When isolating a bug:
- Don't check line by line (O(n))
- Binary search the problem space (O(log n))
- Each test should eliminate half of possibilities

## Why This Matters

```
1000 lines of code:

Linear search: Up to 1000 checks
Binary search: Maximum 10 checks (log2(1000) ≈ 10)

Time saved on complex bugs: Hours
```

## Incorrect Approach

```typescript
// Bug: Function returns wrong value

// Checking every line sequentially:
console.log('line 1', data);    // looks fine
console.log('line 2', data);    // looks fine
console.log('line 3', data);    // looks fine
// ... 47 more console.logs ...
console.log('line 50', data);   // found it!

// Wasted time: Checked 50 lines
```

## Correct Approach

```typescript
// Bug: Function returns wrong value

// Binary search approach:
function processOrder(order) {
  // Step 1: Check midpoint
  const validated = validateOrder(order);
  console.log('After validate:', validated);  // Check here first

  const enriched = enrichOrderData(validated);
  const calculated = calculateTotals(enriched);
  const formatted = formatForApi(calculated);
  return formatted;
}

// If validated looks correct → bug is in second half
// If validated looks wrong → bug is in first half or input

// Step 2: Narrow to half
const enriched = enrichOrderData(validated);
console.log('After enrich:', enriched);  // Check here

// Continue halving until found
// Total checks: 2-3 instead of many
```

## Binary Search with Git Bisect

```bash
# Bug: Tests pass on v1.0, fail on v1.5
# 100 commits between them

git bisect start
git bisect bad HEAD              # Current is broken
git bisect good v1.0             # v1.0 worked

# Git checks out middle commit (~50)
npm test
git bisect bad                   # Still broken

# Git checks out ~25
npm test
git bisect good                  # Works here

# Git checks out ~37
npm test
git bisect bad

# After ~7 steps, finds exact commit
# abc123 is the first bad commit

git bisect reset
git show abc123                  # See what broke it
```

## Binary Search by Component

```
System has bug. Which component?

┌─────────────────────────────────────────┐
│ Frontend → API → Service → Database     │
└─────────────────────────────────────────┘

Step 1: Check at API boundary
        Is the API receiving correct data?
        Is the API returning correct data?

If API returns bad data → problem is API or downstream
If API returns good data → problem is frontend

Step 2: Continue halving the remaining space
```

## Binary Search Checklist

```markdown
For any debugging session:
1. [ ] Define the full problem space
2. [ ] Identify a midpoint to test
3. [ ] Design a test that eliminates half
4. [ ] Execute test
5. [ ] Repeat on remaining half
```

## When Binary Search Doesn't Apply

- Bug depends on interaction between components (need different strategy)
- Problem space is too small (just check everything)
- Non-deterministic bugs (need to stabilize reproduction first)

---
title: Have Tests Before Refactoring
impact: MEDIUM
impactDescription: prevents accidental behavior changes
tags: refactoring, testing, safety
---

## Have Tests Before Refactoring

Never refactor code that doesn't have test coverage. Write characterization tests first if needed.

**Incorrect:**

```typescript
// Refactoring without tests - hoping nothing breaks
// Before:
function calculateTotal(items) {
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    total += items[i].price * items[i].qty;
  }
  return total;
}

// After: "improved" but maybe broke something?
const calculateTotal = (items) =>
  items.reduce((sum, { price, qty }) => sum + price * qty, 0);
```

**Correct:**

```typescript
// First: add characterization tests capturing current behavior
describe('calculateTotal', () => {
  it('sums price * quantity for all items', () => {
    const items = [
      { price: 10, qty: 2 },
      { price: 5, qty: 3 },
    ];
    expect(calculateTotal(items)).toBe(35);
  });

  it('returns 0 for empty array', () => {
    expect(calculateTotal([])).toBe(0);
  });

  it('handles single item', () => {
    expect(calculateTotal([{ price: 10, qty: 1 }])).toBe(10);
  });
});

// Then: refactor with confidence
const calculateTotal = (items) =>
  items.reduce((sum, { price, qty }) => sum + price * qty, 0);

// Tests still pass = behavior preserved
```

**Why:** Without tests, "refactoring" often means "changing behavior in subtle, undetected ways."

Reference: `references/refactoring-patterns.md`

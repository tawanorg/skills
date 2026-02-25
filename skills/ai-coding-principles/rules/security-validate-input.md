---
title: Validate All External Input
impact: CRITICAL
impactDescription: prevents injection and logic errors
tags: security, validation, input
---

## Validate All External Input

All data from external sources (user input, API responses, file contents) must be validated before use.

**Incorrect:**

```typescript
// No validation - trusts external input blindly
function processAmount(amount: string) {
  return parseFloat(amount);  // Could be NaN, negative, or huge
}

app.post('/transfer', (req, res) => {
  transferFunds(req.body.amount);  // No validation
});
```

**Correct:**

```typescript
// Explicit validation with clear constraints
function processAmount(input: unknown): number {
  if (typeof input !== 'string' && typeof input !== 'number') {
    throw new ValidationError('Amount must be a string or number');
  }
  const amount = Number(input);
  if (Number.isNaN(amount)) {
    throw new ValidationError('Amount must be a valid number');
  }
  if (amount < 0 || amount > 1_000_000) {
    throw new ValidationError('Amount must be between 0 and 1,000,000');
  }
  return amount;
}
```

**Why:** Unvalidated input leads to injection attacks, logic errors, and crashes.

Reference: `references/security-checklist.md`

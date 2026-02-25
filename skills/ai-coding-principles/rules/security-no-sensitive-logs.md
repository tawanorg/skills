---
title: Never Log Sensitive Data
impact: CRITICAL
impactDescription: prevents credential and PII leakage
tags: security, logging, privacy
---

## Never Log Sensitive Data

Never log passwords, tokens, API keys, credit card numbers, or personally identifiable information (PII).

**Incorrect:**

```typescript
// Sensitive data in logs - exposed to anyone with log access
logger.info('User login', { email: user.email, password: password });
logger.debug('API call', { headers: req.headers });  // Contains auth tokens
logger.info('Payment', { cardNumber: card.number, cvv: card.cvv });
```

**Correct:**

```typescript
// Mask or exclude sensitive data
logger.info('User login', { email: maskEmail(user.email), userId: user.id });
logger.debug('API call', { method: req.method, path: req.path });
logger.info('Payment', { last4: card.number.slice(-4), amount: payment.amount });

function maskEmail(email: string): string {
  const [local, domain] = email.split('@');
  return `${local[0]}***@${domain}`;
}
```

**Why:** Logs are often accessible to many people and systems, and may be retained for years.

Reference: `references/security-checklist.md`

---
title: Never Hardcode Secrets
impact: CRITICAL
impactDescription: prevents credential exposure
tags: security, secrets, configuration
---

## Never Hardcode Secrets

Never commit API keys, passwords, tokens, or other secrets to code. Use environment variables or secret managers.

**Incorrect:**

```typescript
// Secret in code - will be exposed in version control
const apiKey = 'sk-1234567890abcdef';
const dbPassword = 'super_secret_password';

const stripe = new Stripe('sk_live_abc123');
```

**Correct:**

```typescript
// Secrets from environment variables
const apiKey = process.env.API_KEY;
if (!apiKey) {
  throw new Error('API_KEY environment variable is required');
}

// Or from secret manager
const dbPassword = await secretManager.getSecret('db-password');

// .env file (never committed)
// API_KEY=sk-1234567890abcdef
```

**Why:** Secrets in code get committed to version control and exposed in logs, builds, and backups.

Reference: `references/security-checklist.md`

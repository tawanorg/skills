---
title: Never Store Plaintext Secrets
impact: CRITICAL
impactDescription: plaintext passwords and committed secrets turn one breach into a catastrophe
tags: security, secrets, passwords, hashing
---

## Never Store Plaintext Secrets

Passwords must be hashed, not encrypted or stored as-is. API keys, tokens, and
credentials must live in a secrets manager, never in source code or config
committed to git.

**Context:** Storing user passwords, or managing application credentials and keys.

**Heuristic:** Hash passwords with a slow, salted algorithm (bcrypt, argon2, or scrypt). Keep all other secrets out of the repo — inject them at runtime from a vault or environment, and rotate them.

**Incorrect:**

```text
db.users.insert({ email, password })                 # plaintext password
const STRIPE_KEY = "sk_live_51H...";                 # secret hardcoded & committed
```

**Correct:**

```text
hash = argon2.hash(password)        # salted, slow; verify with argon2.verify(hash, input)
db.users.insert({ email, passwordHash: hash })

const STRIPE_KEY = process.env.STRIPE_KEY  // injected from a secrets manager at runtime
```

**Rules:**
- Hash, never encrypt, passwords — you never need to decrypt them, only verify.
- Always salt (the library handles this); add a pepper for defense in depth.
- Never commit secrets; scan history and rotate anything that leaked.
- Restrict each credential to least privilege and rotate on a schedule.

**Why:** A database leak with plaintext or reversibly-encrypted passwords compromises every user instantly — and password reuse spreads the damage to other sites. Slow salted hashes make offline cracking infeasible; secrets managers limit blast radius and enable rotation.

Reference: `references/security-and-auth.md`

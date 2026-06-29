---
title: Make Writes Idempotent
impact: CRITICAL
impactDescription: prevents duplicate side effects (double charges, duplicate orders) on retries
tags: api, idempotency, reliability, payments, distributed
---

## Make Writes Idempotent

Networks retry. A client that times out doesn't know whether the write
succeeded, so it retries — and a non-idempotent endpoint performs the action
twice. Design state-changing operations so that repeating them is safe.

**Context:** Any non-idempotent operation (POST that creates resources, charges money, sends notifications) reachable over an unreliable network.

**Heuristic:** Accept a client-supplied idempotency key on writes. Store the key with the result; if the same key arrives again, return the original result instead of re-executing.

**Incorrect:**

```http
POST /v1/payments        # client times out, retries → customer charged twice
{ "amount": 5000 }
```

**Correct:**

```http
POST /v1/payments
Idempotency-Key: 7f3c-... # first call executes and records the key; retries return the same payment
{ "amount": 5000 }
```

```text
if seen(key): return stored_result(key)
result = perform_write()
store(key, result)   # atomically, in the same transaction as the write
return result
```

**Why:** In distributed systems "at-least-once" delivery is the norm, so any write can arrive more than once. Idempotency converts duplicate delivery from a data-corruption bug into a no-op. It is non-negotiable for money, inventory, and messaging.

Reference: `references/scalability-and-distributed-systems.md`, `references/payment-and-fintech.md`

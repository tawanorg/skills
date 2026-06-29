---
title: Rate Limit Public Endpoints
impact: HIGH
impactDescription: protects against abuse, runaway clients, and accidental self-DoS
tags: operations, rate-limiting, security, reliability
---

## Rate Limit Public Endpoints

Any endpoint reachable from outside needs a rate limit. It protects the system
from abuse, scrapers, buggy clients in retry loops, and traffic spikes that
would otherwise exhaust capacity.

**Context:** Public APIs, login/auth endpoints, expensive or write-heavy routes.

**Heuristic:** Limit per identity (API key, user, or IP) with an appropriate algorithm. Return `429 Too Many Requests` with a `Retry-After` header. Apply stricter limits to expensive and security-sensitive endpoints (e.g. login).

| Algorithm | Behavior |
|-----------|----------|
| Token bucket | Allows bursts up to bucket size, refills at a steady rate |
| Leaky bucket | Smooths output to a constant rate |
| Fixed window | Simple counter per window; bursty at window edges |
| Sliding window | Smoother than fixed window, more accurate |

**Incorrect:** An unauthenticated `POST /login` with no limit — enabling credential-stuffing and brute force; or a public list endpoint a single client can hammer into a self-inflicted outage.

**Correct:**

```text
key = api_key or user_id or client_ip
if not token_bucket(key).allow():
    return 429, headers={"Retry-After": "30"}
# tighter bucket on /login, /signup, /password-reset
```

**Why:** Without limits, one misbehaving or malicious client can consume all capacity and degrade service for everyone. Rate limiting is both a reliability control and a first line of security defense.

Reference: `references/scalability-and-distributed-systems.md`, `references/security-and-auth.md`

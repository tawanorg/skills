---
title: Use Correct HTTP Status Codes
impact: MEDIUM
impactDescription: lets clients, proxies, and monitoring behave correctly without parsing bodies
tags: api, http, status-codes, errors
---

## Use Correct HTTP Status Codes

The status code is part of your API contract. Clients, load balancers, caches,
and dashboards all act on it. Return the code that matches what actually
happened — never `200 OK` with an error in the body.

**Context:** Designing the responses of any HTTP API.

**Heuristic:** 2xx means it worked, 4xx means the caller is wrong (don't retry as-is), 5xx means you failed (safe to retry). Pick the most specific code in the right class.

| Code | Use when |
|------|----------|
| 200 / 201 / 204 | OK / created a resource / success with no body |
| 400 | Malformed or invalid request |
| 401 / 403 | Not authenticated / authenticated but not allowed |
| 404 | Resource doesn't exist |
| 409 | Conflict (e.g. duplicate, version mismatch) |
| 422 | Well-formed but semantically invalid |
| 429 | Rate limited (include `Retry-After`) |
| 500 / 503 | Unexpected server error / temporarily unavailable |

**Incorrect:**

```json
HTTP/1.1 200 OK
{ "error": "user not found" }   // monitoring sees success; clients must parse bodies to detect failure
```

**Correct:**

```json
HTTP/1.1 404 Not Found
{ "error": "user not found", "code": "USER_NOT_FOUND" }
```

**Why:** Infrastructure reacts to status codes automatically — retries on 5xx, alerting on error rates, cache rules on 2xx. Lying with `200` blinds all of it and forces every client to reimplement error detection.

Reference: `references/api-and-web-development.md`

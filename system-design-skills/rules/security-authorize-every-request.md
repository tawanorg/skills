---
title: Authorize and Validate Every Request
impact: CRITICAL
impactDescription: broken access control and unvalidated input are the top sources of breaches
tags: security, authorization, validation, owasp, api
---

## Authorize and Validate Every Request

Authentication tells you *who* the caller is; authorization decides *what they
may do* — check it on every request, for every object. And never trust input:
validate and sanitize everything that crosses a trust boundary.

**Context:** Every endpoint of every service, especially anything that reads or mutates user-owned data.

**Heuristic:** On each request, verify the caller is authenticated, then verify they are allowed to act on *this specific resource*. Validate input against a strict schema server-side; use parameterized queries and output encoding.

**Incorrect:**

```text
GET /v1/invoices/{id}
# returns the invoice if it exists — any logged-in user can read anyone's invoice (IDOR)

query("SELECT * FROM users WHERE name = '" + input + "'")   # SQL injection
```

**Correct:**

```text
GET /v1/invoices/{id}
# authenticate, then: assert invoice.owner_id == current_user.id (object-level authz)

query("SELECT * FROM users WHERE name = $1", [input])        # parameterized
```

**Checklist for every endpoint:**
- Authenticated? Authorized for *this object*, not just the route?
- Input validated against a schema (types, ranges, length) server-side?
- Queries parameterized; output encoded for its sink (HTML/SQL/shell)?
- Rate limited and behind TLS?

**Why:** Broken access control (e.g. IDOR) and injection consistently top the OWASP list. Client-side checks are advisory only — an attacker calls your API directly, so authorization and validation must live on the server, per request.

Reference: `references/security-and-auth.md`, `references/api-and-web-development.md`

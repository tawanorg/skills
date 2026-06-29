---
title: Version Your APIs
impact: HIGH
impactDescription: lets you evolve APIs without breaking existing clients
tags: api, versioning, compatibility, contracts
---

## Version Your APIs

Public and cross-team APIs are contracts. Introduce a versioning strategy from
day one so you can ship breaking changes without breaking callers you don't control.

**Context:** Designing any API consumed by clients you cannot redeploy in lockstep (mobile apps, partners, other services).

**Heuristic:** Pick one explicit versioning scheme (URI path is simplest: `/v1/...`) and treat removing or repurposing a field as a breaking change that requires a new version.

**Incorrect:**

```http
GET /users          # "v1" implied; one day the response shape changes and every client breaks
```

**Correct:**

```http
GET /v1/users       # additive changes go in v1; breaking changes ship as /v2/users
```

**Compatibility rules of thumb:**

| Change | Breaking? |
|--------|-----------|
| Add an optional field | No |
| Add a new endpoint | No |
| Remove or rename a field | Yes |
| Change a field's type or meaning | Yes |
| Make an optional field required | Yes |

**Why:** Without versioning, every change risks a production outage for clients you can't coordinate with, so teams stop changing the API at all — or break users. Versioning decouples your release cadence from theirs.

Reference: `references/api-and-web-development.md`

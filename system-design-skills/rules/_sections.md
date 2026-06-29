# Sections

This file defines all rule categories and their descriptions.

---

## 1. API Design (api)

**Impact:** HIGH
**Description:** Rules for designing APIs that are stable, safe, and easy to evolve. Versioning, pagination, idempotency, status codes.

## 2. Data and Storage (data)

**Impact:** CRITICAL
**Description:** Rules for choosing and using data stores. Pick storage by access pattern, index the paths you actually query.

## 3. Caching (cache)

**Impact:** HIGH
**Description:** Rules for caching safely. Every cache needs an expiry and invalidation story; protect against stampedes.

## 4. Scalability and Distributed Systems (scale)

**Impact:** CRITICAL
**Description:** Rules for building systems that scale and survive failure. Stateless services, failure handling, async, consistency choices.

## 5. Security (security)

**Impact:** CRITICAL
**Description:** Rules for protecting systems and data. Never store plaintext secrets; authorize and validate every request.

## 6. Operations (operations)

**Impact:** HIGH
**Description:** Rules for running systems in production. Rate limit public surfaces; instrument with logs, metrics, and traces.

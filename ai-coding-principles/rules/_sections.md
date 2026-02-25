# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Security (security)

**Impact:** CRITICAL
**Description:** Non-negotiable security rules to prevent vulnerabilities. SQL injection, input validation, secrets management.

## 2. Error Handling (error)

**Impact:** HIGH
**Description:** Rules for explicit, debuggable error handling. Never swallow errors, always include context.

## 3. Database (db)

**Impact:** HIGH
**Description:** Database query and schema rules. Prevent N+1, use parameterized queries, index properly.

## 4. API Design (api)

**Impact:** HIGH
**Description:** REST API conventions. Correct status codes, consistent error formats, request validation.

## 5. Architecture (arch)

**Impact:** HIGH
**Description:** SOLID principles and architectural boundaries. Single responsibility, dependency inversion.

## 6. Resilience (resilience)

**Impact:** MEDIUM
**Description:** Patterns for handling failures in distributed systems. Timeouts, retries, circuit breakers.

## 7. Observability (obs)

**Impact:** MEDIUM
**Description:** Logging and monitoring rules. Structured logs, request IDs, no sensitive data in logs.

## 8. Testing (test)

**Impact:** MEDIUM
**Description:** Testing philosophy and practices. Test behavior not implementation, cover critical paths.

## 9. Refactoring (refactor)

**Impact:** MEDIUM
**Description:** Safe refactoring practices. Tests first, small commits, separate from features.

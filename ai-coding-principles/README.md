# AI Coding Principles

Production-quality coding principles for AI agents. A comprehensive blueprint covering everything from core principles to enterprise patterns.

## Structure

```
ai-coding-principles/
├── SKILL.md                              # Main skill file (agent reads this)
├── metadata.json                         # Version and reference info
├── README.md                             # This file (human documentation)
├── references/                           # Detailed guidance by topic
│   ├── architecture-patterns.md          # SOLID, Clean Architecture, DDD
│   ├── observability-patterns.md         # Logging, Metrics, Tracing
│   ├── api-design-patterns.md            # REST, Errors, Pagination
│   ├── database-patterns.md              # Schema, Queries, Migrations
│   ├── resilience-patterns.md            # Retries, Circuit Breakers
│   ├── code-review-checklist.md          # Systematic Review Process
│   ├── refactoring-patterns.md           # Safe Refactoring Techniques
│   ├── error-handling-patterns.md        # TS, Python, Go Examples
│   ├── security-checklist.md             # OWASP-aligned Security
│   └── testing-patterns.md               # Unit, Integration, E2E
└── rules/                                # Atomic, enforceable rules
    ├── _sections.md                      # Category definitions
    ├── _template.md                      # Template for new rules
    ├── security-*.md                     # Security rules (CRITICAL)
    ├── error-*.md                        # Error handling rules (HIGH)
    ├── db-*.md                           # Database rules (HIGH)
    ├── api-*.md                          # API design rules (HIGH)
    ├── arch-*.md                         # Architecture rules (HIGH)
    ├── test-*.md                         # Testing rules (MEDIUM)
    ├── obs-*.md                          # Observability rules (MEDIUM)
    ├── resilience-*.md                   # Resilience rules (MEDIUM)
    └── refactor-*.md                     # Refactoring rules (MEDIUM)
```

## Core Principles

The skill teaches four golden rules that apply to every coding task:

1. **Read Before Write** - Always explore the codebase before generating code
2. **Minimal Change** - Make the smallest change that solves the problem
3. **Match the Codebase** - Consistency over personal preference
4. **Verify Understanding** - Ask when uncertain

## References vs Rules

This skill uses two types of supporting content:

| Type | Purpose | Format | Use Case |
|------|---------|--------|----------|
| **References** | Deep understanding | Comprehensive guides | Learning, design decisions |
| **Rules** | Quick checks | Incorrect → Correct → Why | Code review, enforcement |

### References (10 files)

Comprehensive guides covering broad topics in depth.

| Topic | Reference File | Coverage |
|-------|----------------|----------|
| Architecture | `architecture-patterns.md` | SOLID principles, Clean Architecture layers, DDD basics |
| Observability | `observability-patterns.md` | Structured logging, Four Golden Signals, Distributed tracing |
| API Design | `api-design-patterns.md` | REST conventions, Error formats, Pagination, Versioning |
| Database | `database-patterns.md` | Schema design, Indexing, N+1 prevention, Migrations |
| Resilience | `resilience-patterns.md` | Retry with backoff, Circuit breakers, Timeouts, Bulkheads |
| Code Review | `code-review-checklist.md` | Review focus areas, Comment patterns, Self-review |
| Refactoring | `refactoring-patterns.md` | Safe process, Extract patterns, Legacy strategies |
| Error Handling | `error-handling-patterns.md` | TypeScript, Python, Go patterns |
| Security | `security-checklist.md` | Input validation, Injection prevention, Secrets |
| Testing | `testing-patterns.md` | Test classification, Mocking, What not to test |

### Rules (20 rules)

Atomic, enforceable patterns for code review and verification.

| Category | Impact | Rules |
|----------|--------|-------|
| Security | CRITICAL | 4 rules (parameterized queries, input validation, secrets, logging) |
| Error Handling | HIGH | 2 rules (never swallow, include context) |
| Database | HIGH | 3 rules (no SELECT *, N+1, index FKs) |
| API Design | HIGH | 3 rules (status codes, error format, validation) |
| Architecture | HIGH | 2 rules (SRP, dependency inversion) |
| Testing | MEDIUM | 2 rules (behavior not impl, critical paths) |
| Observability | MEDIUM | 2 rules (structured logs, request IDs) |
| Resilience | MEDIUM | 2 rules (timeouts, retry transient only) |
| Refactoring | MEDIUM | 2 rules (tests first, small commits) |

## Usage

This skill is loaded automatically when an AI agent detects relevant tasks:
- Writing new code or features
- Modifying existing codebases
- Code review or quality checks
- Questions about best practices

The agent reads `SKILL.md` for an overview, then consults specific reference or rule files as needed.

## Contributing

### Adding a New Rule

1. Copy `rules/_template.md` to `rules/{category}-{description}.md`
2. Fill in the frontmatter (title, impact, tags)
3. Write: explanation, incorrect code, correct code, why it matters
4. Add the rule to SKILL.md's rules table

### Adding New Reference Content

1. Determine if it belongs in SKILL.md (overview) or a reference file (detail)
2. For new topics, create a new reference file in `references/`
3. Link the new file from SKILL.md's reference table
4. Follow existing formatting patterns

### Content Guidelines

- **Be actionable** - Tell agents what to do, not vague principles
- **Show examples** - Include good vs bad code comparisons
- **Add checklists** - Help agents verify they followed guidance
- **Explain context** - When does this apply? When doesn't it?

### Updating Version

When making significant changes:
1. Update `version` in `metadata.json`
2. Update `date` to current month/year

# Skills

Production-ready skills for AI coding agents. Each skill is a battle-tested blueprint that helps AI agents write code the way senior engineers do.

## Why This Exists

AI coding agents are powerful but need guidance. Without it, they:
- Generate code that works but doesn't fit the codebase
- Miss security vulnerabilities
- Skip error handling
- Ignore performance implications

These skills encode hard-won engineering knowledge into a format AI agents understand.

## Available Skills

### ai-coding-principles

**The blueprint for production-quality code.**

20+ atomic rules and 10 comprehensive reference guides covering everything from SOLID architecture to resilience patterns.

```bash
npx skills add tawanorg/skills/ai-coding-principles
```

**What's inside:**

| Category | Content |
|----------|---------|
| Core Principles | Read Before Write, Minimal Change, Match Codebase, Verify Understanding |
| Security | Input validation, SQL injection prevention, secrets management (4 rules) |
| Error Handling | Never swallow errors, always include context (2 rules) |
| Database | N+1 prevention, indexing, parameterized queries (3 rules) |
| API Design | Status codes, error formats, request validation (3 rules) |
| Architecture | SOLID principles, Clean Architecture, DDD (2 rules + guide) |
| Observability | Structured logging, metrics, tracing (2 rules + guide) |
| Resilience | Timeouts, retries, circuit breakers (2 rules + guide) |
| Testing | Behavior testing, critical path coverage (2 rules + guide) |
| Refactoring | Safe refactoring, small commits (2 rules + guide) |
| Code Review | Systematic review checklist |

**Triggers on:**
- Writing new code or features
- Modifying existing codebases
- Code review or quality checks
- Questions about "best practices", "code quality", "production-ready"

---

### system-design-thinking

**Thinking frameworks for architectural decisions.**

Systematic approach to system design with trade-off analysis, decision heuristics, and battle-tested patterns from distributed systems.

```bash
npx skills add tawanorg/skills/system-design-thinking
```

**What's inside:**

| Category | Content |
|----------|---------|
| Thinking Framework | Structured approach: Requirements → Constraints → Trade-offs → Decision |
| Trade-off Analysis | CAP theorem, Latency vs Throughput, Consistency models |
| Architecture Styles | Monolith, Modular Monolith, Microservices, Serverless, Event-Driven |
| Scalability | Horizontal/Vertical scaling, Load balancing, Caching, Database sharding |
| Integration | REST, gRPC, Message queues, Pub/Sub, Saga pattern, API Gateway |
| Data Architecture | Storage selection, Replication, CQRS, Event Sourcing |
| Decision Heuristics | 8 rules: Start with requirements, Prefer simple, Design for failure, etc. |
| Documentation | ADR template and examples |

**Triggers on:**
- System design discussions
- Architecture planning
- Choosing between technologies
- Scaling decisions
- Questions about "how should we architect", "trade-offs", "CAP theorem"

---

### debugging-mastery

**Systematic debugging methodology from top tech companies.**

The scientific method applied to debugging. No more guessing—follow the same systematic approach that senior engineers at Facebook, Google, and Netflix use to find and fix bugs.

```bash
npx skills add tawanorg/skills/debugging-mastery
```

**What's inside:**

| Category | Content |
|----------|---------|
| Debugging Loop | Observe → Hypothesize → Predict → Test → Conclude |
| Core Rules | Reproduce first, binary search, read errors, verify fix (10 rules) |
| Production Debugging | Safe investigation in production, mitigation, evidence gathering |
| Performance Debugging | Profiling, flame graphs, bottleneck analysis |
| Memory Leaks | Detection, heap snapshots, common leak patterns |
| Distributed Tracing | Following requests across services, correlation IDs |
| Common Bug Patterns | Off-by-one, null refs, race conditions, async bugs |
| Debugging Tools | Debuggers, profilers, network tools, database analysis |

**Triggers on:**
- Investigating bugs
- Troubleshooting production issues
- Analyzing error logs
- Performance problems
- Questions about "why isn't this working", "how to debug", "root cause"

## Installation

### Claude Code

```bash
npx skills add tawanorg/skills
```

Or install a specific skill:

```bash
npx skills add tawanorg/skills/ai-coding-principles
```

Or manually copy to your skills directory:

```bash
git clone https://github.com/tawanorg/skills.git
cp -r skills/ai-coding-principles ~/.claude/skills/
cp -r skills/system-design-thinking ~/.claude/skills/
cp -r skills/debugging-mastery ~/.claude/skills/
```

### Claude.ai

Upload the skill folder to your project knowledge, or paste `SKILL.md` contents directly into the conversation.

## How Skills Work

Skills are loaded on-demand:

1. **Startup**: Only skill names and descriptions are loaded
2. **Detection**: Agent detects a relevant task
3. **Loading**: Full `SKILL.md` loads into context
4. **Progressive disclosure**: Agent reads specific `rules/` or `references/` files as needed

This keeps context usage minimal while providing deep knowledge when needed.

## Repository Structure

```
tawanorg/skills/
├── README.md                    # This file
├── AGENTS.md                    # AI agent guidance for this repo
├── CLAUDE.md                    # Symlink to AGENTS.md
├── LICENSE                      # MIT
├── ai-coding-principles/        # Production code quality
│   ├── SKILL.md                 # Main skill file
│   ├── metadata.json            # Version, author info
│   ├── README.md                # Human documentation
│   ├── rules/                   # 22 atomic rules
│   └── references/              # 10 comprehensive guides
├── system-design-thinking/      # Architecture decisions
│   ├── SKILL.md                 # Main skill file
│   ├── metadata.json            # Version, author info
│   ├── README.md                # Human documentation
│   ├── rules/                   # 8 decision heuristics
│   └── references/              # 6 pattern guides
└── debugging-mastery/           # Systematic debugging
    ├── SKILL.md                 # Main skill file
    ├── metadata.json            # Version, author info
    ├── README.md                # Human documentation
    ├── rules/                   # 10 debugging rules
    └── references/              # 6 debugging guides
```

## Creating Your Own Skill

### Quick Start

```bash
mkdir -p your-skill-name/references
touch your-skill-name/SKILL.md
```

### SKILL.md Template

```markdown
---
name: your-skill-name
description: >
  One paragraph describing when to use this skill.
  Include trigger phrases the agent should recognize.
license: MIT
metadata:
  author: your-name
  version: '1.0.0'
---

# Your Skill Title

Brief description.

## When to Apply

- Trigger condition 1
- Trigger condition 2

## Main Content

Your guidance here...

## Reference Files

| Topic | File |
|-------|------|
| Details | `references/details.md` |
```

### Best Practices

- **Keep SKILL.md under 500 lines** - Put details in references/
- **Write specific descriptions** - Helps agent know when to activate
- **Use progressive disclosure** - Link to detailed files
- **Be actionable** - Tell agents what to do, not vague advice
- **Show examples** - Include incorrect → correct code

## Contributing

1. Fork this repository
2. Create your skill in `your-skill-name/` at the root level
3. Follow the structure and guidelines above
4. Submit a pull request

### Adding Rules to ai-coding-principles

1. Copy `rules/_template.md` to `rules/{category}-{description}.md`
2. Fill in frontmatter (title, impact, tags)
3. Write: explanation, incorrect code, correct code, why
4. Add to SKILL.md rules table
5. Submit PR

## License

MIT

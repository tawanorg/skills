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
npx skills add @tawanorg/skills/ai-coding-principles
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

## Installation

### Claude Code

```bash
npx skills add @tawanorg/skills
```

Or install a specific skill:

```bash
npx skills add @tawanorg/skills/ai-coding-principles
```

Or manually copy to your skills directory:

```bash
git clone https://github.com/tawanorg/skills.git
cp -r skills/skills/ai-coding-principles ~/.claude/skills/
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
skills/
├── README.md                    # This file
├── AGENTS.md                    # AI agent guidance for this repo
├── CLAUDE.md                    # Symlink to AGENTS.md
├── LICENSE                      # MIT
└── skills/
    └── ai-coding-principles/
        ├── SKILL.md             # Main skill (agent reads this first)
        ├── metadata.json        # Version, author, references
        ├── README.md            # Human documentation
        ├── rules/               # Atomic, enforceable rules
        │   ├── _sections.md     # Category definitions
        │   ├── _template.md     # Template for new rules
        │   └── *.md             # Individual rules
        └── references/          # Comprehensive guides
            └── *.md             # Topic guides
```

## Creating Your Own Skill

### Quick Start

```bash
mkdir -p skills/your-skill-name/references
touch skills/your-skill-name/SKILL.md
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
2. Create your skill in `skills/your-skill-name/`
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

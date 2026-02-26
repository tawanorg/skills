# System Design Thinking

Frameworks for architectural decision-making and system design. Helps AI agents think about architecture before writing code.

## Structure

```
system-design-thinking/
├── SKILL.md                              # Main skill file (agent reads this)
├── metadata.json                         # Version and reference info
├── README.md                             # This file (human documentation)
├── rules/                                # Decision heuristics
│   ├── _sections.md                      # Category definitions
│   ├── start-with-requirements.md        # Understand before designing
│   ├── prefer-simple-architecture.md     # Monolith first
│   ├── define-boundaries-first.md        # Domain boundaries
│   ├── design-for-failure.md             # Assume things fail
│   ├── document-decisions.md             # ADRs
│   ├── consider-operations.md            # Deployment, monitoring
│   ├── data-flow-first.md                # Understand data before components
│   └── async-over-sync.md                # For resilience
└── references/                           # Deep dives
    ├── trade-off-analysis.md             # Frameworks for evaluating options
    ├── architecture-styles.md            # Monolith, microservices, serverless
    ├── scalability-patterns.md           # Horizontal, vertical, caching
    ├── integration-patterns.md           # APIs, events, data sharing
    ├── data-architecture.md              # Storage patterns, consistency
    └── adr-template.md                   # Architecture Decision Record template
```

## Core Concepts

### The Design Thinking Process

1. **Understand** - Requirements, constraints, users, data
2. **Decompose** - Break into bounded contexts and components
3. **Explore** - Generate multiple options
4. **Evaluate** - Analyze trade-offs explicitly
5. **Decide** - Choose and document with ADR
6. **Validate** - Review with stakeholders

### Key Principles

| Principle | Meaning |
|-----------|---------|
| **Simple First** | Start with monolith, evolve when needed |
| **Boundaries Matter** | Good boundaries enable independent change |
| **Trade-offs Explicit** | No perfect solution, only trade-offs |
| **Data Drives Design** | Understand data flow before components |
| **Design for Failure** | Assume everything fails eventually |
| **Operations Day 1** | Consider deployment, monitoring from start |

## Rules (8 total)

Decision heuristics for common architectural choices.

| Rule | When to Apply |
|------|---------------|
| Start with Requirements | Beginning any design |
| Prefer Simple Architecture | Choosing architecture style |
| Define Boundaries First | Decomposing systems |
| Design for Failure | Designing distributed systems |
| Document Decisions | Making significant choices |
| Consider Operations | Finalizing designs |
| Data Flow First | Starting system design |
| Async Over Sync | Choosing integration patterns |

## References (6 files)

Deep dives on specific architecture topics.

| Topic | Coverage |
|-------|----------|
| Trade-off Analysis | CAP, PACELC, decision matrices |
| Architecture Styles | Monolith, microservices, serverless, event-driven |
| Scalability Patterns | Horizontal, vertical, caching, sharding, queuing |
| Integration Patterns | REST, gRPC, events, saga, choreography vs orchestration |
| Data Architecture | Storage selection, consistency models, CQRS, event sourcing |
| ADR Template | Architecture Decision Record format |

## Usage

This skill is loaded when an AI agent detects:
- System design tasks
- Architecture decisions
- "How should I structure this?"
- "What's the best approach for..."
- Trade-off discussions

## Contributing

### Adding a New Rule

1. Create `rules/{category}-{name}.md`
2. Follow format: Context → Heuristic → Why
3. Include examples (when to apply, when not to)
4. Add to SKILL.md rules table

### Adding Reference Content

1. Create `references/{topic}.md`
2. Include diagrams/tables where helpful
3. Link from SKILL.md reference table

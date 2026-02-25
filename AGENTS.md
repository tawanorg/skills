# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Repository Overview

A collection of skills for AI coding agents. Skills are packaged instructions and reference materials that extend agent capabilities for writing production-quality code.

## Repository Structure

```
skills/
├── README.md                    # Main documentation
├── AGENTS.md                    # This file (AI agent guidance)
├── CLAUDE.md                    # Symlink to AGENTS.md
├── LICENSE                      # MIT License
└── skills/
    └── {skill-name}/            # Individual skill directories
        ├── SKILL.md             # Required: skill definition
        ├── metadata.json        # Optional: version, author info
        ├── README.md            # Optional: human documentation
        └── references/          # Optional: supporting docs
```

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # Required: skill definition with frontmatter
    metadata.json         # Optional: version, author, references
    README.md             # Optional: human-readable documentation
    references/           # Optional: detailed reference materials
      topic.md
    scripts/              # Optional: executable scripts
      script.sh
```

### Naming Conventions

- **Skill directory**: `kebab-case` (e.g., `ai-coding-principles`, `react-patterns`)
- **SKILL.md**: Always uppercase, always this exact filename
- **References**: `kebab-case.md` (e.g., `error-handling-patterns.md`)
- **Scripts**: `kebab-case.sh` (e.g., `validate.sh`)

### SKILL.md Format

```markdown
---
name: skill-name
description: One sentence describing when to use this skill. Include trigger phrases.
license: MIT
metadata:
  author: your-name
  version: '1.0.0'
---

# Skill Title

Brief description of what the skill does.

## When to Apply

Reference these guidelines when:
- Trigger condition 1
- Trigger condition 2

## Main Content

Organized sections with actionable guidance.

## Reference Files

Link to supporting documentation in references/ folder.
```

### Best Practices for Context Efficiency

Skills are loaded on-demand — only the skill name and description are loaded at startup. The full `SKILL.md` loads into context only when the agent decides the skill is relevant. To minimize context usage:

- **Keep SKILL.md under 500 lines** — put detailed reference material in separate files
- **Write specific descriptions** — helps the agent know exactly when to activate the skill
- **Use progressive disclosure** — reference supporting files that get read only when needed
- **Organize by priority** — put most important content first
- **Use tables for quick reference** — easier to scan than prose

### metadata.json Format

```json
{
  "version": "1.0.0",
  "author": "your-name",
  "organization": "Optional org name",
  "date": "Month Year",
  "abstract": "Brief description of the skill's purpose.",
  "references": [
    "https://relevant-docs.example.com"
  ]
}
```

### README.md Format

The skill-level README is for human readers (not loaded by agents):

```markdown
# Skill Name

Brief description.

## Structure

Explain the file organization.

## Reference Files

List and describe each reference file.

## Contributing

How to add new content.
```

## Working with Existing Skills

### ai-coding-principles

The main skill in this repository. Contains:
- Core coding principles (Golden Rules)
- Enterprise patterns (Architecture, Observability, API Design, etc.)
- 10 reference files with detailed guidance

When modifying:
- Keep SKILL.md as the overview with links to references
- Put detailed examples in reference files
- Maintain the principles summary table
- Update the reference files table if adding new files

## Quality Standards

### For All Content

- **Actionable** — Tell the agent what to do, not vague advice
- **Example-driven** — Show code examples (good vs bad)
- **Checklist-oriented** — Provide verification checklists
- **Context-aware** — Explain when to apply each pattern

### For Code Examples

- Use TypeScript for primary examples (widely applicable)
- Include Python/Go examples where language-specific
- Show both incorrect and correct patterns
- Explain why the correct pattern is better

## Validation

Before committing changes:

1. Verify SKILL.md frontmatter is valid:
   - `name`: lowercase, hyphen-separated, max 64 chars
   - `description`: max 1024 chars, no angle brackets

2. Check file organization:
   - SKILL.md exists and is valid
   - Reference files are linked from SKILL.md
   - No orphaned files

3. Test readability:
   - Content is scannable
   - Tables render correctly
   - Code blocks have language hints

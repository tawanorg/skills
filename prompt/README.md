# /prompt - AI Coding Prompt Generator

Transform vague ideas into effective, structured prompts for AI coding assistants.

## Usage

```
/prompt add user authentication
/prompt I want a dashboard
/prompt help me build a payment integration
```

## What It Does

1. **Understands your intent** - Parses your short message to identify goals and gaps
2. **Gathers repo context** - Scans your codebase for tech stack, patterns, and related code
3. **Conducts clarifying interview** - Asks focused questions to fill gaps
4. **Generates structured prompt** - Creates `feature/prompt.md` optimized for AI coding

## Output

Creates `feature/prompt.md` with:
- Context and goals
- Technical specification with step-by-step plan
- Detailed requirements with edge cases
- Testing strategy
- Success criteria
- File references from your codebase

## Prompt Engineering Patterns Used

- **Least-to-Most Decomposition** - Breaks complex features into ordered steps
- **Chain-of-Thought** - Includes reasoning guidance for AI
- **Few-Shot Examples** - References similar patterns from your codebase
- **Constraint Specification** - Clear goals and non-goals

## Structure

```
prompt/
├── SKILL.md              # Main skill definition
├── README.md             # This file
└── references/
    ├── prompt-patterns.md      # CoT, few-shot, templates
    └── interview-questions.md  # Question library by task type
```

## Credits

Patterns adapted from:
- [meigen-ai-design/prompt-crafter](https://github.com/wshobson/agents/blob/main/plugins/meigen-ai-design/agents/prompt-crafter.md)
- [llm-application-dev/prompt-engineering-patterns](https://github.com/wshobson/agents/tree/main/plugins/llm-application-dev/skills/prompt-engineering-patterns)

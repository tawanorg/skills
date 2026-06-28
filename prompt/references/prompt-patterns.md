# Prompt Engineering Patterns Reference

## Reasoning Patterns

### Zero-Shot Chain-of-Thought
Add "Let's think step by step" to elicit reasoning without examples.

```
Given: [problem]
Let's think step by step to solve this.
```

### Few-Shot Chain-of-Thought
Provide worked examples with explicit reasoning before the target problem.

```
Example 1:
Problem: [example problem]
Reasoning: [step-by-step reasoning]
Solution: [answer]

Example 2:
Problem: [example problem]
Reasoning: [step-by-step reasoning]
Solution: [answer]

Now solve:
Problem: [target problem]
```

### Least-to-Most Decomposition
Break complex problems into simpler subproblems, solve sequentially.

```
To accomplish [complex goal]:

Subproblem 1: [simplest piece]
Subproblem 2: [next piece, using result from 1]
Subproblem 3: [next piece, using results from 1-2]
...
Final integration: [combine all solutions]
```

### Self-Consistency
Generate multiple reasoning paths, use majority voting for final answer.
Use temperature variation to explore different approaches.

### Verification Steps
Add explicit error-checking stages.

```
After completing [task]:
1. Review for logical errors
2. Validate calculations/assumptions
3. Check edge cases
4. Revise if problems detected
```

## Structural Patterns

### Role-Based Prompting
Establish expertise and behavioral context.

```
You are a [role] with expertise in [domains].
Your responsibilities include [specific duties].
You should [behavioral guidelines].
You should NOT [constraints].
```

### Template with Conditionals
Dynamic content based on context.

```
# Task: {{task_name}}

{{#if has_examples}}
## Examples
{{#each examples}}
- {{this}}
{{/each}}
{{/if}}

## Requirements
{{requirements}}
```

### Constraint Specification
Distinguish hard rules from soft guidelines.

```
## Hard Requirements (non-negotiable)
- MUST: [absolute requirement]
- MUST NOT: [absolute prohibition]

## Soft Guidelines (flexible)
- SHOULD: [recommended approach]
- MAY: [optional enhancement]
```

## Domain-Specific Patterns

### Code Generation
```
Language: [language]
Framework: [framework if applicable]
Style: [coding conventions]

Task: [what to implement]

Constraints:
- [constraint 1]
- [constraint 2]

Expected interface:
```[language]
[function signature or API shape]
```

Tests to pass:
- [test case 1]
- [test case 2]
```

### Data Extraction
```
Extract the following from the provided text:

Fields:
- field_name: [type] - [description]
- field_name: [type] - [description]

Output format: JSON
Handle missing data: [strategy]

Text:
[input text]
```

### Classification
```
Classify the following into one of these categories:
- Category A: [description]
- Category B: [description]
- Category C: [description]

Provide:
1. Category label
2. Confidence (high/medium/low)
3. Brief reasoning

Input: [content to classify]
```

### Code Review
```
Review the following code for:
- [ ] Correctness
- [ ] Readability
- [ ] Performance
- [ ] Security
- [ ] Best practices

For each issue found:
1. Location (file:line)
2. Severity (critical/warning/suggestion)
3. Problem description
4. Suggested fix

Code:
```[language]
[code]
```
```

## Quality Markers

### Specificity Examples
| Vague | Specific |
|-------|----------|
| "Make it fast" | "Response time < 200ms at p95" |
| "Handle errors" | "Return 4xx for client errors with message field" |
| "Good UX" | "Loading states, keyboard nav, WCAG 2.1 AA" |
| "Secure" | "Input validation, parameterized queries, HTTPS only" |

### Testability Checklist
- [ ] Can be verified programmatically
- [ ] Has clear pass/fail criteria
- [ ] Includes edge case coverage
- [ ] Specifies expected outputs for given inputs

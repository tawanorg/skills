---
name: discover-intent
description: >
  Interview the user relentlessly about a project idea until reaching 95% confidence
  about what they actually want — not what they think they should want.
  Use when starting a new project, validating an idea, hearing "grill me",
  "stress test this", "what am I missing", or before writing any implementation plan.
license: MIT
metadata:
  author: tawanorg
  version: '1.0.0'
---

# Discover Intent

Interview me relentlessly about this project until you reach 95% confidence about what I **actually want** — not what I think I should want. The gap between those two things is where most failed projects begin.

Work through the question matrix below **one question at a time**. For each question:

1. Ask the question
2. Provide your recommended answer based on what you've learned so far
3. Flag any contradictions with previous answers
4. If the answer can be found by exploring the codebase, explore it instead of asking

Skip questions that have already been clearly answered. Adapt follow-up questions based on what you learn.

After all branches resolve, output a **crystallized spec** summarizing the true intent.

---

## Question Matrix

### Phase 1: Identity & Motivation

The uncomfortable "why" questions that expose whether this project should exist.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 1 | Why are **you** building this and not someone else? | Unique advantage or lack thereof |
| 2 | What's the emotional reason behind this project you haven't said out loud? | Hidden motivation that will drive scope creep |
| 3 | Are you building what users need or what you'd enjoy building? | Builder bias vs user need |
| 4 | Who benefits if this succeeds? Who loses? | Stakeholder map and potential resistance |
| 5 | If this project disappeared tomorrow, who would notice? Be honest. | Real demand signal |

### Phase 2: Reality Check

Validate against the world as it actually is, not as you imagine it.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 6 | What existing solution are people using right now, even if it's ugly? | The real competitor — often a spreadsheet |
| 7 | Why is that ugly solution "good enough" for them today? | Switching cost and inertia you're fighting |
| 8 | What would make someone stop using that and switch to yours — honestly? | Whether your differentiator is real |
| 9 | Have you talked to anyone who'd pay for this? What did they **actually say** vs what you heard? | Confirmation bias check |
| 10 | What's the strongest argument **against** building this? | Intellectual honesty about viability |

### Phase 3: Assumptions That Kill Projects

Surface the beliefs you're treating as facts.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 11 | What's the one thing that, if wrong, makes this entire project worthless? | Single point of failure assumption |
| 12 | What are you assuming about user behavior that you've never validated? | Untested behavioral hypothesis |
| 13 | Where are you designing for yourself instead of your user? | Projection bias |
| 14 | What complexity are you hiding from yourself right now? | Scope denial |
| 15 | What do you believe is true about this market/problem that most people disagree with? | Contrarian thesis or echo chamber |

### Phase 4: Scope Honesty

Cut through feature fantasy to find the essential core.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 16 | If you could only ship ONE screen/endpoint/feature, which one proves the idea? | True MVP vs disguised V1 |
| 17 | What are you including because it's "easy to add" but nobody asked for? | Scope creep seeds |
| 18 | What's the feature you keep mentioning first? Why that one? | Emotional attachment vs user value |
| 19 | What part of this are you procrastinating on thinking about? | The hard problem you're avoiding |
| 20 | If you had to launch in 1 week, what would you cut? What about 1 day? | Priority hierarchy under pressure |

### Phase 5: Second-Order Thinking

Think past launch day — what happens when this actually works?

| # | Question | What It Exposes |
|---|----------|-----------------|
| 21 | If this works, what problem does success create? | Unplanned scaling, support, expectations |
| 22 | What happens at 10x the users you're planning for? | Architecture assumptions |
| 23 | What's the hardest thing to change later if you get it wrong now? | Irreversible decisions to get right |
| 24 | What decision are you making permanent that should be reversible? | Over-engineering or premature commitment |
| 25 | What will you do if this works but you hate running it? | Sustainability and founder-market fit |

### Phase 6: Competitive Truth

Honest positioning against the real landscape.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 26 | Why hasn't someone built this already? (If they have — why is yours different, really?) | Market awareness and honest differentiation |
| 27 | What do you know that the market doesn't? | Information edge or delusion |
| 28 | Is your differentiation real or just "better UI"? | Defensibility |
| 29 | What would a well-funded competitor do in 6 months that kills your advantage? | Moat assessment |
| 30 | Who is your user's **next best alternative** if you don't exist? | True competitive landscape |

### Phase 7: User & Context

Understand the human on the other side.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 31 | Who is the primary user? Describe them as a specific person, not a segment. | User empathy vs abstract thinking |
| 32 | What are they doing immediately before and after using this? | Workflow integration reality |
| 33 | What's their skill level and patience level? | UX complexity ceiling |
| 34 | What's the user's current emotional state when they reach for this tool? | Design tone and urgency |
| 35 | How will they discover this exists? Be specific. | Distribution reality check |

### Phase 8: Technical Constraints

Ground the vision in engineering reality.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 36 | What must this integrate with? What are the non-negotiable technical constraints? | Architecture boundaries |
| 37 | What's already been tried or ruled out? Why? | History and scar tissue |
| 38 | What are the real performance requirements — actual numbers, not aspirations? | Over/under-engineering risk |
| 39 | What's your data story? Where does it come from, where does it live, who owns it? | Data architecture assumptions |
| 40 | What's your deployment/infrastructure reality? | Build vs buy decisions |

---

## Contradiction Detection

Throughout the interview, track and flag:

- **Scope vs Timeline contradictions** — "MVP" that has 20 features
- **User vs Builder contradictions** — solving your own problem but claiming product-market fit
- **Ambition vs Resource contradictions** — enterprise features with solo-founder resources
- **Stated vs Revealed preferences** — what they say matters vs what they keep talking about

When a contradiction is detected, pause and resolve it before continuing.

---

## Output: Crystallized Spec

After completing the interview, produce:

### 1. True Intent Statement
One sentence: what this project actually is, stripped of aspiration.

### 2. Core Assumptions (Ranked by Risk)
The beliefs this project depends on, ordered by how likely they are to be wrong.

### 3. MVP Definition
The absolute minimum that validates the riskiest assumption.

### 4. Kill Criteria
Specific, measurable conditions under which this project should be abandoned.

### 5. Honest Assessment
A direct, compassionate evaluation: build it, pivot the approach, or walk away — and why.

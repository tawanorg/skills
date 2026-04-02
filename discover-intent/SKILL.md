---
name: discover-intent
description: >
  Enterprise-grade intent discovery framework. Conducts structured interrogation
  across 12 phases with 75+ deep questions, confidence scoring, evidence triangulation,
  Jobs-to-be-Done analysis, risk matrices, and multi-stakeholder mapping.
  Use when starting a new project, validating an idea, hearing "grill me",
  "stress test this", "what am I missing", or before writing any implementation plan.
license: MIT
metadata:
  author: tawanorg
  version: '2.0.0'
---

# Discover Intent

Enterprise-grade intent discovery that exposes the gap between what you **think you want** and what you **actually need** — before a single line of code is written.

## How This Works

### Interview Protocol

Work through the question matrix **one question at a time**. For each question:

1. **Ask** the question
2. **Recommend** your answer based on evidence gathered so far
3. **Score** confidence (0-100%) for the answer
4. **Flag** contradictions with previous answers
5. **Probe deeper** if confidence is below 70% — ask follow-up questions until it rises
6. If the answer can be found by **exploring the codebase or data**, do that instead of asking

### Adaptive Routing

Not every project needs all 12 phases. After Phase 1, assess the project type and skip phases that don't apply:

| Project Type | Required Phases | Optional Phases |
|---|---|---|
| Greenfield product | All 12 | — |
| Internal tool | 1, 3, 4, 5, 7, 8, 10, 12 | 2, 6, 9, 11 |
| Feature addition | 1, 3, 4, 5, 7, 8, 12 | 2, 6, 9, 10, 11 |
| Migration/refactor | 1, 3, 5, 8, 10, 12 | All others |
| API/platform | 1, 2, 3, 5, 6, 7, 8, 10, 12 | 4, 9, 11 |

### Confidence Tracking

Maintain a running confidence dashboard after each phase:

```
Phase                    Confidence  Evidence Strength
─────────────────────────────────────────────────────
1. Identity & Motivation    85%      ██████████░  Strong
2. Problem Validation       40%      ████░░░░░░░  Weak — needs data
3. Assumptions Audit        ??%      Not yet started
...
─────────────────────────────────────────────────────
Overall Confidence:         62%      Below threshold (95%)
```

Do not proceed to output until overall confidence reaches **95%** or all unresolved items are explicitly flagged as known unknowns.

---

## Question Matrix

### Phase 1: Identity & Motivation

**Goal:** Expose whether this project should exist and why you're the one building it.

| # | Question | What It Exposes | Evidence Type |
|---|----------|-----------------|---------------|
| 1 | Why are **you** building this and not someone else? What's your unfair advantage? | Unique positioning or lack thereof | Stated |
| 2 | What's the emotional reason behind this project you haven't said out loud? | Hidden motivation that drives scope creep | Stated |
| 3 | Are you building what users need or what you'd enjoy building? How do you know? | Builder bias vs user need | Requires validation |
| 4 | Who are all the stakeholders? Map them: who benefits, who loses, who has veto power, who pays. | Stakeholder power dynamics | Stated + verifiable |
| 5 | If this project disappeared tomorrow, who would notice first? What would they do instead? | Real demand signal | Behavioral |

**Follow-up probes if confidence < 70%:**
- What evidence do you have that this need is real vs imagined?
- Have you tried to kill this idea? What survived?
- Who told you to build this, and what's their incentive?

---

### Phase 2: Problem Validation

**Goal:** Verify the problem exists at the scale you believe, using evidence — not intuition.

| # | Question | What It Exposes | Evidence Type |
|---|----------|-----------------|---------------|
| 6 | Describe the problem in one sentence without mentioning your solution. | Whether you understand the problem independent of the solution | Stated |
| 7 | What existing solution are people using right now? Describe their actual workflow step by step. | The real competitor — often a spreadsheet, email, or manual process | Behavioral |
| 8 | Why is that current solution "good enough"? What would it take for them to switch? | Switching cost and inertia | Behavioral |
| 9 | How many people have this problem? Show your math — not your guess. | Market sizing rigor | Quantitative |
| 10 | How often does this problem occur? Daily, weekly, monthly, once? | Frequency = urgency | Quantitative |
| 11 | What's the cost of this problem — in money, time, or pain? Be specific. | Willingness to pay signal | Quantitative |
| 12 | Have you talked to anyone who'd pay for this? Quote what they **actually said** — not your interpretation. | Confirmation bias check | Behavioral |

**Evidence triangulation required:**
- At least ONE behavioral observation (what people do, not what they say)
- At least ONE quantitative data point (usage data, market size, frequency)
- Flag if all evidence is self-reported stated preferences

---

### Phase 3: Assumptions Audit

**Goal:** Surface every belief you're treating as fact and rank them by how catastrophic it would be if wrong.

| # | Question | What It Exposes | Risk Level |
|---|----------|-----------------|------------|
| 13 | List every assumption this project depends on. I'll help — but you start. | Assumption awareness | — |
| 14 | Which ONE assumption, if wrong, makes the entire project worthless? | Fatal assumption | Critical |
| 15 | What are you assuming about user behavior that you've never observed directly? | Untested behavioral hypothesis | High |
| 16 | What are you assuming about the technical feasibility that you haven't prototyped? | Technical risk | High |
| 17 | What are you assuming about the market that could change in 6 months? | Market timing risk | Medium |
| 18 | Where are you designing for yourself instead of your user? Be brutally honest. | Projection bias | High |
| 19 | What complexity are you hiding from yourself right now? | Scope denial | High |

**Assumption Risk Matrix — build this during the phase:**

```
                        Low Impact if Wrong    High Impact if Wrong
                       ┌──────────────────────┬──────────────────────┐
Likely to be wrong     │ Monitor              │ VALIDATE IMMEDIATELY │
                       ├──────────────────────┼──────────────────────┤
Unlikely to be wrong   │ Accept               │ Have a contingency   │
                       └──────────────────────┴──────────────────────┘
```

Each assumption goes into one quadrant. The top-right quadrant items must be validated before building.

---

### Phase 4: Jobs-to-be-Done Analysis

**Goal:** Understand the real job the user is hiring this product to do — functional, emotional, and social.

| # | Question | What It Exposes | JTBD Dimension |
|---|----------|-----------------|----------------|
| 20 | When was the last time someone "hired" a solution for this problem? Walk me through that moment. | The triggering event and context | Functional |
| 21 | What is the user trying to become or achieve in their life/work? (Not "use feature X") | Aspirational job | Emotional |
| 22 | How does solving this problem make them look to their peers, boss, or team? | Social job | Social |
| 23 | What are the anxieties about switching to a new solution? List them all. | Adoption barriers | Emotional |
| 24 | What are they giving up by choosing your solution? Every choice has a cost. | Real trade-offs | Functional |

**JTBD Statement — construct this:**
```
When [situation/trigger],
I want to [functional job],
so I can [emotional/social outcome],
without [anxiety/trade-off].
```

This statement should be validated by the user. If they can't agree to it, you don't understand the job yet.

---

### Phase 5: Scope & Prioritization

**Goal:** Cut through feature fantasy to find the atomic core that proves the idea.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 25 | If you could only ship ONE interaction, which one proves the core job-to-be-done? | True MVP vs disguised V1 |
| 26 | What are you including because it's "easy to add" but nobody asked for? Name everything. | Scope creep seeds |
| 27 | What's the feature you keep mentioning first? Why? Is it the most important or just your favorite? | Emotional attachment vs user value |
| 28 | What part of this are you procrastinating on thinking about? | The hard problem you're avoiding |
| 29 | Rank every feature by: (a) evidence of demand, (b) effort, (c) risk if wrong. | Evidence-based prioritization |
| 30 | If you had to launch in 1 week, what ships? 1 day? 1 hour? | Priority hierarchy under pressure |

**Scope Scoring Matrix — build during phase:**

| Feature | Evidence of Demand (1-5) | Effort (1-5) | Risk if Wrong (1-5) | Score |
|---------|--------------------------|--------------|----------------------|-------|
| _fill_ | | | | Demand / (Effort x Risk) |

Features with score < 0.5 should be cut from MVP. Features with no evidence of demand (score = 0) should be cut entirely unless they're infrastructure.

---

### Phase 6: Business Model & Unit Economics

**Goal:** Ensure the project is economically viable, not just technically interesting.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 31 | How does this make money? Or save money? Or save time that converts to money? Be specific. | Revenue model clarity |
| 32 | What would a user pay for this? Not "would they pay" — what is the specific number and why? | Price anchoring and willingness to pay |
| 33 | What does it cost you to acquire one user? How do you know? | CAC reality |
| 34 | What does it cost to serve one user per month? Include infrastructure, support, everything. | Unit economics |
| 35 | At what scale does this become profitable? Show the math. | Break-even clarity |
| 36 | What's the lifetime value of a user? How did you calculate retention? | LTV assumptions |

**Unit Economics Check:**
```
LTV:CAC ratio target: > 3:1
Payback period target: < 12 months
Gross margin target:   > 60% for software

Your numbers: LTV = $___  CAC = $___  Ratio = ___:1
              Payback = ___ months   Margin = ___%
```

If these numbers don't work or can't be estimated, that's a critical finding.

---

### Phase 7: User & Persona Deep Dive

**Goal:** Build evidence-based personas, not fictional characters.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 37 | Describe your primary user as a specific real person you've talked to. Name, role, daily reality. | User empathy vs abstract segments |
| 38 | Walk me through their entire day. Where does your product fit in? What's before and after? | Workflow integration reality |
| 39 | What's their skill level? Patience level? How many seconds before they abandon something? | UX complexity ceiling |
| 40 | What's their emotional state when they reach for this tool? Calm? Frustrated? Panicked? | Design tone and urgency requirements |
| 41 | How will they discover this exists? Not "marketing" — the specific channel, moment, and message. | Distribution reality check |
| 42 | Is there a secondary user? Do their needs conflict with the primary user's? | Multi-persona tension |

**Persona Card — construct for each distinct user type:**
```
[Name] — [One-line description]
Role & Context:  ___
Primary Job:     ___
Key Pain Points: 1. ___ 2. ___ 3. ___
Current Solution: ___
Switching Trigger: What would make them try something new?
Anxiety:          What would make them hesitate?
Evidence Source:  Interview / Survey / Analytics / Assumed
```

If the persona is based on assumptions rather than interviews or data, flag it as **unvalidated**.

---

### Phase 8: Technical Architecture & Constraints

**Goal:** Ground the vision in engineering reality and identify irreversible technical decisions.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 43 | What are the non-negotiable technical constraints? (Existing systems, compliance, latency, etc.) | Hard boundaries |
| 44 | What's already been tried or ruled out technically? Why did it fail? | Engineering scar tissue |
| 45 | What are the real performance requirements? Give me p50, p95, p99 numbers — not aspirations. | Over/under-engineering risk |
| 46 | What's your data story? Where does it originate, how does it flow, who owns it, what's the schema? | Data architecture assumptions |
| 47 | What's the deployment reality? Cloud/on-prem, CI/CD maturity, team capability, budget. | Infrastructure constraints |
| 48 | Which technical decisions are irreversible or extremely expensive to change? | Decisions that need to be right |
| 49 | What third-party dependencies are you taking on? What happens when they go down, change pricing, or sunset? | Vendor risk |

**Technical Decision Log — fill during phase:**

| Decision | Options Considered | Choice | Reversibility | Confidence |
|----------|-------------------|--------|---------------|------------|
| _fill_ | | | Easy / Hard / Permanent | High / Med / Low |

Decisions marked "Permanent" with "Low" confidence are red flags.

---

### Phase 9: Go-to-Market & Distribution

**Goal:** A product without distribution is a hobby. Validate the path to users.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 50 | What's your distribution channel? Not "social media" — the specific channel, tactic, and first 100 users. | GTM specificity |
| 51 | What's your wedge? Which tiny, specific audience do you win first before expanding? | Beachhead strategy |
| 52 | What does the user's buying process look like? Who decides, who approves, who pays? | B2B complexity or B2C impulse |
| 53 | What's your activation metric? How do you know a user "got it"? | Onboarding clarity |
| 54 | What's your retention signal? What behavior predicts they'll come back? | Retention leading indicator |
| 55 | How does word-of-mouth work for this? Is it naturally shareable or do you need to force it? | Organic growth potential |

---

### Phase 10: Second-Order Effects & Failure Modes

**Goal:** Think past launch day. What happens when this works — and when it doesn't?

| # | Question | What It Exposes |
|---|----------|-----------------|
| 56 | If this works perfectly, what new problem does success create? | Unplanned scaling, support load, expectations |
| 57 | What happens at 10x users? 100x? Where does the architecture break first? | Scaling assumptions |
| 58 | What's the hardest thing to change later if you get it wrong now? | Irreversible decisions |
| 59 | What decision are you making permanent that should be reversible? | Over-commitment |
| 60 | What will you do if this works but you hate running it? | Sustainability and founder-market fit |
| 61 | What's the worst realistic thing that happens if you ship a bug? Data loss? Financial loss? Safety? | Failure severity |
| 62 | What happens if a key team member leaves mid-project? | Bus factor |

**Failure Mode Analysis:**

| Failure Scenario | Probability | Impact | Detection Speed | Mitigation |
|---|---|---|---|---|
| _fill_ | Low/Med/High | Low/Med/High/Critical | Fast/Slow/Never | _fill_ |

Any row with High probability + High/Critical impact + Slow/Never detection is a project risk that must be addressed.

---

### Phase 11: Competitive & Market Positioning

**Goal:** Honest assessment of where you stand, not where you wish you stood.

| # | Question | What It Exposes |
|---|----------|-----------------|
| 63 | Who are all the competitors? Include indirect ones (spreadsheets, agencies, doing nothing). | Complete competitive landscape |
| 64 | Why hasn't a well-funded team solved this already? If they have — what's actually different about yours? | Market awareness |
| 65 | What do you know that the market doesn't? Is this information or delusion? | Edge assessment |
| 66 | Is your differentiation defensible? What stops a competitor from copying it in 3 months? | Moat reality |
| 67 | What would a well-funded competitor do in 6 months that kills your advantage? | Threat modeling |
| 68 | What's your user's next-best alternative if you don't exist? How painful is that alternative? | True competitive bar |

**Competitive Positioning Map — construct:**
```
                    Serves [primary job] well
                           ▲
                           │
        ┌──────────────────┼──────────────────┐
        │  Competitor A     │    YOUR PRODUCT   │
 Low ◄──┤                  │                  ├──► High
 ease   │  Competitor B     │    Competitor C   │   ease
 of use │                  │                  │   of use
        └──────────────────┼──────────────────┘
                           │
                           ▼
                    Serves [primary job] poorly
```

Replace axes with whatever dimensions actually differentiate in your market.

---

### Phase 12: Organizational Readiness & Compliance

**Goal:** Can your team actually build and operate this? What rules apply?

| # | Question | What It Exposes |
|---|----------|-----------------|
| 69 | Does your team have the skills to build this? Be specific about gaps. | Capability assessment |
| 70 | What regulatory or compliance requirements apply? (GDPR, SOC2, HIPAA, PCI, accessibility) | Legal constraints |
| 71 | What data privacy implications exist? Where does PII flow? | Privacy architecture |
| 72 | What's the support model? Who handles issues at 2 AM? | Operational readiness |
| 73 | What's the realistic timeline? Now add 50%. Is it still worth doing? | Timeline honesty |
| 74 | What organizational dependencies could block you? Other teams, approvals, budget cycles? | Execution blockers |
| 75 | What's your definition of done? Not "feature complete" — when do you stop? | Exit criteria |

---

## Contradiction Detection Engine

Track these contradiction patterns throughout the interview. When detected, **stop and resolve before continuing:**

| Contradiction Type | Signal | Example |
|---|---|---|
| Scope vs Timeline | "MVP" with 20 features and a 4-week deadline | Says "lean" but describes enterprise platform |
| User vs Builder | Solving own problem but claiming market demand | "Users want X" but only evidence is own frustration |
| Ambition vs Resource | Enterprise features with solo-founder resources | Plans real-time collaboration on a solo budget |
| Stated vs Revealed | What they say matters vs what they keep talking about | Says "simplicity" but keeps adding features |
| Evidence vs Assertion | Strong claims with no data | "Everyone needs this" — who is everyone? |
| Problem vs Solution | Describes solution when asked about problem | Can't articulate problem without mentioning product |
| Persona vs Feature | Features that don't map to any stated user need | Building admin dashboard but user is consumer |

**Resolution Protocol:**
1. State the contradiction clearly: "You said X in Phase 2 but Y in Phase 5"
2. Ask which is true — or if reality is more nuanced
3. Update confidence scores for affected phases
4. Re-ask downstream questions if the resolution changes prior answers

---

## Evidence Grading

Every key finding gets an evidence grade:

| Grade | Meaning | Source |
|---|---|---|
| **A — Observed** | Directly witnessed user behavior or measured data | Analytics, usability tests, direct observation |
| **B — Reported** | User described their behavior in interview/survey | Interviews, surveys, support tickets |
| **C — Inferred** | Logically derived from other evidence | Cross-referenced from multiple B-grade sources |
| **D — Assumed** | No evidence — believed to be true | Founder intuition, industry "common knowledge" |
| **F — Contradicted** | Evidence exists against this claim | Data shows opposite of stated belief |

**Rule:** No critical decision should rest on D-grade evidence. Any F-grade evidence must be resolved immediately.

---

## Output Artifacts

After completing all relevant phases, produce the following:

### Artifact 1: True Intent Statement

One paragraph. What this project **actually is**, stripped of aspiration, marketing language, and wishful thinking. Written as if explaining to a skeptical investor with 30 seconds of patience.

### Artifact 2: JTBD Summary

```
Primary Job:     When [situation], I want to [job], so I can [outcome].
Emotional Job:   I want to feel [emotion] when doing [activity].
Social Job:      I want to be seen as [perception] by [audience].
Key Anxieties:   [list]
Key Trade-offs:  [list]
```

### Artifact 3: Validated Assumption Register

| # | Assumption | Evidence Grade | Risk if Wrong | Status |
|---|-----------|----------------|---------------|--------|
| 1 | _fill_ | A/B/C/D/F | Low/Med/High/Critical | Validated / Unvalidated / Contradicted |

### Artifact 4: Risk Matrix

```
                     Low Impact          High Impact
                  ┌─────────────────┬─────────────────┐
 High Probability │ MITIGATE        │ BLOCK — resolve  │
                  │                 │ before building  │
                  ├─────────────────┼─────────────────┤
 Low Probability  │ ACCEPT          │ CONTINGENCY plan │
                  │                 │ required         │
                  └─────────────────┴─────────────────┘
```

### Artifact 5: Scope Definition

**In scope (with evidence):**
| Feature | JTBD Mapping | Evidence Grade | Priority |
|---------|-------------|----------------|----------|
| _fill_ | | | P0/P1/P2 |

**Explicitly out of scope (and why):**
- _fill_

**Deferred (with trigger to reconsider):**
- _fill_ — revisit when [condition]

### Artifact 6: Decision Log

| Decision | Options Considered | Choice | Rationale | Reversibility | Confidence |
|----------|-------------------|--------|-----------|---------------|------------|
| _fill_ | | | | Easy/Hard/Permanent | High/Med/Low |

### Artifact 7: Kill Criteria

Specific, measurable conditions under which this project should be **abandoned**, not pivoted:

1. _fill_ — measurable threshold
2. _fill_ — measurable threshold
3. _fill_ — measurable threshold

These must be agreed upon **before** building starts. If you can't agree on kill criteria, you're not ready to build.

### Artifact 8: Honest Verdict

A direct, compassionate evaluation:

- **BUILD** — evidence supports this, risks are manageable, proceed with [specific caveats]
- **VALIDATE FIRST** — [specific assumptions] must be tested before committing resources
- **PIVOT** — the problem is real but the approach needs to change because [specific reasons]
- **WALK AWAY** — [specific reasons] make this unviable, and that's okay

Include what changed your mind during the interview — the moment the real intent diverged from the stated intent.

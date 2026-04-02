# 05 — Scope Surgery

**Goal:** Cut through feature fantasy to find the atomic core that proves the idea. Remove everything that is not load-bearing.

**When to use independently:** When a plan exists but feels bloated, or when someone says "we just need to add a few more things."

---

## Question Matrix

| # | Question | What It Exposes | Evidence Type |
|---|----------|-----------------|---------------|
| 1 | If you could only ship one interaction — one user action and one system response — which one proves the core job-to-be-done? Only one. | The true MVP vs a disguised V1 with aspirations | Stated |
| 2 | What is included in the plan because it is "easy to add" but nobody actually asked for? Name everything in this category. | Scope creep seeds planted by engineering convenience | Stated |
| 3 | Which feature do you mention first when describing this product? Is that the most important feature or just your favorite? | Emotional attachment masquerading as priority | Stated |
| 4 | What part of this are you procrastinating on thinking about? Not the hard technical part — the hard strategic part. | Avoidance of the central unsolved problem | Stated |
| 5 | If you had to launch in one week, what ships? If you had one day? If you had one hour? Be specific each time. | Priority hierarchy revealed under time pressure | Stated |
| 6 | Rank every feature by: evidence of demand (1-5), effort to build (1-5), risk if you build it wrong (1-5). | Evidence-based prioritization vs gut-feeling priority | Quantitative |
| 7 | What feature are you including because removing it would feel like admitting the core is not enough? | Features as confidence props rather than user value | Stated |
| 8 | What would your most demanding target user use on day one? What would they never touch in the first month? | Usage reality vs feature inventory | Behavioral |
| 9 | Which features create dependencies that will delay everything else? What would you unlock by cutting them? | Dependency tree and critical path clarity | Technical |
| 10 | What does "version two" contain? Now explain why version one cannot succeed without those things already in it. | Artificial version inflation and premature completeness | Stated |
| 11 | If a competitor launched with half your feature set tomorrow, would users prefer them? Why or why not? | Whether breadth or depth wins in this market | Stated |
| 12 | What is the version of this product that would embarrass you to ship — and is that version actually what users need? | Ego-driven scope inflation vs user-driven scope | Stated |

---

## Scope Scoring Matrix

Build this during the phase. Features with Score < 0.5 are cut from MVP. Features with Evidence of Demand = 0 are cut entirely unless they are infrastructure:

| Feature | Evidence of Demand (1-5) | Effort (1-5) | Risk if Wrong (1-5) | Score (D ÷ E×R) | Decision |
|---------|--------------------------|--------------|----------------------|-----------------|----------|
| _fill_ | | | | | In / Deferred / Cut |

---

## Follow-up Probes (use when confidence < 70%)

- You said [feature X] is essential. Which user told you that, and when?
- If [feature X] is not built, can the core job still be done? If yes, why is it in scope?
- You ranked [feature Y] as high demand — what is the evidence for that? Is it an assumption or is it observed?
- The features you are deferring to V2 — what user need are they responding to? Are those users actually in scope for V1?
- If you were paying out of pocket for every hour of development, what would you cut first?

---

## Contradiction Patterns to Watch

| Pattern | Signal |
|---------|--------|
| MVP inflation | "MVP" includes onboarding, settings, notifications, and admin dashboard |
| Demand inversion | Features with lowest evidence get most discussion time |
| Deferred reality | V2 contains the thing that proves the idea — the real MVP is in V2 |
| Dependency hiding | Features described as simple are actually dependent on unbuilt infrastructure |
| Completeness anxiety | Cannot ship without [feature] not because users need it but because it "feels incomplete" |

---

## Output Artifact: Scope Definition

```
SCOPE DEFINITION
─────────────────────────────────────────────────────────
Project:   ___
Date:      ___
Version:   ___

ATOMIC CORE (the one interaction that proves the idea)
______________________________________________________
User action:    ___
System response: ___
Job it performs: ___

IN SCOPE (with evidence)
Feature                    | JTBD Mapping | Evidence Grade | Priority
---------------------------|--------------|----------------|----------
                           |              |                | P0
                           |              |                | P0
                           |              |                | P1

EXPLICITLY OUT OF SCOPE (and why)
- [Feature]: ___  Reason: ___
- [Feature]: ___  Reason: ___

DEFERRED (with trigger to reconsider)
- [Feature]: ___  Revisit when: ___
- [Feature]: ___  Revisit when: ___

SCOPE SCORING TABLE
Feature | Demand | Effort | Risk | Score | Decision
--------|--------|--------|------|-------|----------
        | /5     | /5     | /5   |       |

V1 LAUNCH TARGETS
  One week:  ___
  One day:   ___
  One hour:  ___

PHASE CONFIDENCE:  ___% (must reach 80% before proceeding)
EVIDENCE GRADE:    A / B / C / D / F
─────────────────────────────────────────────────────────
```

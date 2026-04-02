# 04 — Jobs-to-be-Done

**Goal:** Understand the real job the user is hiring this product to do — functional, emotional, and social dimensions.

**When to use independently:** When you know the problem exists but are not sure you understand the user's actual motivation for solving it.

---

## Question Matrix

| # | Question | What It Exposes | JTBD Dimension |
|---|----------|-----------------|----------------|
| 1 | Walk me through the last specific time someone "hired" a solution — any solution — for this problem. What triggered it? What did they actually do? | The real moment of need and context, not the hypothesized one | Functional |
| 2 | What is the user trying to become or achieve in their life or work? Not "use feature X" — the identity or status they are moving toward. | Aspirational job that makes the functional job meaningful | Emotional |
| 3 | When your user solves this problem, how does it change how they look to their peers, manager, or customers? Who witnesses the improvement? | Social validation and status job — often the real driver | Social |
| 4 | What are every anxiety and hesitation about switching to something new? List them all, even the irrational ones. | Adoption barriers that logic alone will not overcome | Emotional |
| 5 | What is the user giving up by choosing your solution? Every choice has a real cost — what is theirs? | Trade-offs the user will consciously or unconsciously weigh | Functional |
| 6 | What does the user do in the five minutes before they reach for this type of solution? What do they do in the five minutes after? | Context window around the job — reveals integration requirements | Functional |
| 7 | Why does the timing matter? What makes someone switch to a new solution today versus six months ago or six months from now? | Triggering conditions and market timing | Functional |
| 8 | What is the user not willing to give up even if it would make the solution better? What is non-negotiable for them? | Hard constraints that cannot be designed around | Functional |
| 9 | Which part of the current workflow does the user secretly like, even though it is inefficient? | Emotional attachment that creates resistance to "better" solutions | Emotional |
| 10 | If a colleague told this user your product was terrible, what would the user believe — and why? | Social proof structure and trust architecture | Social |
| 11 | What story does the user tell themselves when they fail to solve this problem? What story would they tell if they solved it perfectly? | Narrative identity around the job — core to retention | Emotional |
| 12 | If this product worked perfectly, what job does the user no longer have an excuse to avoid? | The job beneath the job — often the real source of resistance | Emotional |

---

## JTBD Statement

Construct and validate this statement. If the user cannot agree to it, you do not understand the job yet:

```
When [situation / trigger],
I want to [functional job — specific action],
so I can [emotional / social outcome — the "becoming"],
without [primary anxiety / trade-off].
```

**Validation test:** Read it back to the user. Their reaction reveals how well you understand the job. Blank stare or "sure, I guess" means you missed it. "That is exactly it" means you have it.

---

## Follow-up Probes (use when confidence < 70%)

- You described what the user wants to do. What do they want to become or be seen as?
- The anxiety you named — is that about the product or about what it means about them if they need to use it?
- Walk me through the moment of switch one more time. What exactly was happening in their environment?
- You said users want [feature X]. What job does that feature perform? Can a completely different feature perform the same job?
- If the functional job is [X], what makes it feel urgent versus "I should get to that eventually"?

---

## Contradiction Patterns to Watch

| Pattern | Signal |
|---------|--------|
| Feature as job | Describes a feature or interaction instead of an outcome or transformation |
| Missing social dimension | Cannot articulate how solving the problem changes the user's standing with others |
| Anxiety blindness | Lists no anxieties or hesitations — every solution adoption has some |
| Static user | Describes user as they are, not what they are trying to become |
| Hypothetical trigger | Describes "when a user wants to..." without a specific observed instance |

---

## Output Artifact: JTBD Canvas

```
JTBD CANVAS
─────────────────────────────────────────────────────────
Project:        ___
Primary User:   ___
Date:           ___

JTBD STATEMENT
  When:         ___  (situation / trigger)
  I want to:    ___  (functional job)
  So I can:     ___  (emotional / social outcome)
  Without:      ___  (primary anxiety / trade-off)

FUNCTIONAL JOB
  Core task:    ___
  Context:      ___  (5 min before / 5 min after)
  Trigger:      ___  (why now, not 6 months ago)

EMOTIONAL JOB
  Becoming:     ___  (identity transformation)
  Feeling:      ___  (desired emotional state after)
  Fear:         ___  (emotional state driving avoidance)

SOCIAL JOB
  Audience:     ___  (who witnesses the improvement)
  Signal:       ___  (how improvement is visible to others)
  Risk:         ___  (social cost of switching / failing)

ANXIETIES (all of them, including irrational)
  1. ___
  2. ___
  3. ___

TRADE-OFFS (what the user gives up by choosing you)
  1. ___
  2. ___

NON-NEGOTIABLES (what they will not give up)
  1. ___
  2. ___

VALIDATION STATUS
  User agreement with JTBD statement:  Confirmed / Partial / Rejected
  Based on observation or report:      Observed / Reported / Inferred

PHASE CONFIDENCE:  ___% (must reach 80% before proceeding)
EVIDENCE GRADE:    A / B / C / D / F
─────────────────────────────────────────────────────────
```

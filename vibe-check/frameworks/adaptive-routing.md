# Adaptive Routing Framework

## Purpose

Not every project needs all 11 sub-skills. This framework detects the project type from the initial pitch and routes to the minimum required set — while allowing optional skills when signals suggest they are needed.

---

## Project Type Detection

Ask or infer from the first exchange:

| Signal | Project Type |
|--------|-------------|
| "I want to build a new product / startup / app" | Greenfield product |
| "We need a tool for our team / company" | Internal tool |
| "We want to add [feature] to [existing product]" | Feature addition |
| "We need to move from [old system] to [new system]" | Migration / refactor |
| "We're building an API / platform / SDK / integration layer" | API / platform |
| "Just give me a quick sanity check on this idea" | Quick gut check |

When ambiguous, ask: "Is this a new product, an addition to something existing, or an internal tool?"

---

## Routing Table

### Greenfield Product
Full discovery. Skip nothing.

| Order | Sub-skill | Required |
|-------|-----------|----------|
| 1 | 01 Motivation Audit | Required |
| 2 | 02 Problem Validation | Required |
| 3 | 03 Assumption Mining | Required |
| 4 | 04 Jobs-to-be-Done | Required |
| 5 | 05 Scope Surgery | Required |
| 6 | 06 Business Model | Required |
| 7 | 07 User Deep Dive | Required |
| 8 | 08 Technical Reality | Required |
| 9 | 09 Market Positioning | Required |
| 10 | 10 Failure Modes | Required |
| 11 | 11 Synthesis | Required |

**Estimated time:** 75-90 minutes
**Confidence gate:** 95% overall

---

### Internal Tool
Skip market positioning and business model (unless it requires budget approval or competes with vendor options).

| Order | Sub-skill | Required |
|-------|-----------|----------|
| 1 | 01 Motivation Audit | Required |
| 2 | 03 Assumption Mining | Required |
| 3 | 04 Jobs-to-be-Done | Required |
| 4 | 05 Scope Surgery | Required |
| 5 | 07 User Deep Dive | Required |
| 6 | 08 Technical Reality | Required |
| 7 | 10 Failure Modes | Required |
| 8 | 11 Synthesis | Required |
| — | 02 Problem Validation | Optional (run if problem is contested within the org) |
| — | 06 Business Model | Optional (run if requires budget approval or build-vs-buy decision) |
| — | 09 Market Positioning | Optional (run if evaluating vendor alternatives) |

**Estimated time:** 50-65 minutes

---

### Feature Addition
Focus on user and scope. Assume market and business model are inherited from the parent product.

| Order | Sub-skill | Required |
|-------|-----------|----------|
| 1 | 01 Motivation Audit | Required |
| 2 | 03 Assumption Mining | Required |
| 3 | 04 Jobs-to-be-Done | Required |
| 4 | 05 Scope Surgery | Required |
| 5 | 07 User Deep Dive | Required |
| 6 | 08 Technical Reality | Required |
| 7 | 11 Synthesis | Required |
| — | 02 Problem Validation | Optional (run if the problem is not already validated by existing product data) |
| — | 10 Failure Modes | Optional (run if the feature touches a high-risk surface: payments, auth, data export) |

**Estimated time:** 40-55 minutes

---

### Migration / Refactor
Technical reality and failure modes are critical. User and market are largely known.

| Order | Sub-skill | Required |
|-------|-----------|----------|
| 1 | 01 Motivation Audit | Required |
| 2 | 03 Assumption Mining | Required |
| 3 | 05 Scope Surgery | Required |
| 4 | 08 Technical Reality | Required |
| 5 | 10 Failure Modes | Required |
| 6 | 11 Synthesis | Required |
| — | 04 Jobs-to-be-Done | Optional (run if migration changes the user experience significantly) |
| — | 07 User Deep Dive | Optional (run if new user behavior is expected post-migration) |

**Estimated time:** 35-50 minutes

---

### API / Platform
Business model and technical reality are load-bearing. Developer experience is a form of user research.

| Order | Sub-skill | Required |
|-------|-----------|----------|
| 1 | 01 Motivation Audit | Required |
| 2 | 02 Problem Validation | Required |
| 3 | 03 Assumption Mining | Required |
| 4 | 05 Scope Surgery | Required |
| 5 | 06 Business Model | Required |
| 6 | 07 User Deep Dive | Required (developer is the user) |
| 7 | 08 Technical Reality | Required |
| 8 | 10 Failure Modes | Required |
| 9 | 11 Synthesis | Required |
| — | 04 Jobs-to-be-Done | Optional (run if developer adoption is unclear) |
| — | 09 Market Positioning | Optional (run if there are established API competitors) |

**Estimated time:** 65-80 minutes

---

### Quick Gut Check
Minimum viable discovery. Produces a simplified verdict: proceed to full discovery, validate first, or stop.

| Order | Sub-skill | Required |
|-------|-----------|----------|
| 1 | 01 Motivation Audit | Required (abbreviated: questions 1, 2, 5, 8 only) |
| 2 | 03 Assumption Mining | Required (abbreviated: questions 1, 2, 11 only) |
| 3 | Mini-assessment | Required |

**Estimated time:** 12-20 minutes

**Mini-assessment output:**
```
QUICK GUT CHECK RESULT
─────────────────────────────────────────────────────────
Motivation clarity:  Strong / Weak / Unclear
Fatal assumption:    Identified / Not yet surfaced
Biggest red flag:    ___

SIGNAL:  PROCEED TO FULL DISCOVERY / VALIDATE FIRST / STOP HERE

Reason in one sentence: ___
─────────────────────────────────────────────────────────
```

---

## Mid-Session Routing Adjustments

Even with a starting route, adjust based on what emerges:

| Signal During Interview | Routing Adjustment |
|------------------------|-------------------|
| Fatal assumption identified in skill 03 | Jump to skill 11 mini-synthesis. Deliver VALIDATE FIRST verdict. Resume full discovery after validation. |
| Business model is completely unclear | Add skill 06 even if not in the starting route |
| Market is highly competitive and builder is unaware | Add skill 09 |
| Team is small and timeline is aggressive | Add skill 10, weight failure modes heavily |
| User is completely undefined | Add skill 07 before any scope work |
| Contradiction detected that changes the project type | Re-route from the current position using the new project type |

---

## Time Estimates by Configuration

| Configuration | Sub-skills | Estimated Time |
|---------------|-----------|----------------|
| Full (greenfield) | 01-11 | 75-90 min |
| Internal tool | 01,03,04,05,07,08,10,11 | 50-65 min |
| Feature addition | 01,03,04,05,07,08,11 | 40-55 min |
| Migration | 01,03,05,08,10,11 | 35-50 min |
| API/platform | 01,02,03,05,06,07,08,10,11 | 65-80 min |
| Stress-test | 03,05,10,11 | 25-35 min |
| Quick gut check | 01(abbrev),03(abbrev) | 12-20 min |

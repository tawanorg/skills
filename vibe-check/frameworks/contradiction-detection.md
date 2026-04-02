# Contradiction Detection Framework

## Purpose

Contradictions are the most valuable signal in the interview process. They reveal where stated intent and revealed intent diverge — which is exactly what this skill exists to find. When a contradiction appears, stop and resolve it before continuing. Contradictions not resolved in the interview will become architectural decisions you cannot undo.

---

## The 7 Contradiction Patterns

### 1. Scope vs Timeline

**What it is:** The scope described is incompatible with the timeline stated.

**Detection signals:**
- "MVP" contains features that together would take 3x the stated timeline
- "We just need a few weeks" for a system with multiple integrations, auth, and admin tooling
- Timeline is fixed (investor deadline, launch event) but scope keeps growing

**Examples:**
- "We'll launch in 6 weeks with real-time collaboration, offline mode, and enterprise SSO"
- "It's a simple CRUD app" followed by a data model with 15 entities and complex permissions

**Resolution:** Ask which is actually fixed — the timeline or the scope. Force a choice. Then cut scope to fit the fixed constraint.

---

### 2. User vs Builder

**What it is:** The product described solves the builder's problem, not the stated user's problem.

**Detection signals:**
- Every feature described is something the builder would personally use
- "Our users want X" but the only evidence is the builder's own experience
- Builder cannot describe a user's day without inserting themselves into it
- Features that users have not asked for but the builder finds technically interesting

**Examples:**
- Building a developer tool "for non-technical users" that requires CLI interaction
- "Our users love data" — but the evidence is that the builder loves data

**Resolution:** For each feature, ask: "Which specific user told you they needed this, and when?" Features with no answer are builder features, not user features.

---

### 3. Ambition vs Resource

**What it is:** The vision requires resources — time, money, skills, team size — that are not available and not planned for.

**Detection signals:**
- Enterprise-grade reliability targets with a two-person team and no DevOps
- "We'll handle compliance later" for a product that processes payments or health data
- Plans for a platform with a budget for a feature

**Examples:**
- "We'll build the Stripe of [category]" with a $50K budget and 3 months
- Real-time multiplayer features planned by a team with no distributed systems experience

**Resolution:** Map every ambitious claim to a specific resource requirement. Then ask whether that resource is confirmed, planned, or assumed.

---

### 4. Stated vs Revealed

**What it is:** What the builder says matters is different from what they actually keep talking about, returning to, or getting animated about.

**Detection signals:**
- Says "we prioritize simplicity" then spends 80% of the interview describing complex features
- Claims the user is "non-technical" but all examples describe technical workflows
- States "we are not doing monetization yet" but has strong opinions about pricing strategy

**Examples:**
- "This is primarily a B2B product" but every example and story is about individual consumers
- "Retention is the key metric" but only talks about acquisition

**Resolution:** Name the pattern explicitly: "You've said [stated priority] but you keep returning to [revealed priority]. Which one actually drives your decisions?" Then listen to what they say and how they say it.

---

### 5. Evidence vs Assertion

**What it is:** Strong, confident claims that have no evidence behind them.

**Detection signals:**
- "Everyone needs this" without naming who "everyone" is
- "Users will definitely pay for this" with no price testing
- Market size numbers stated with confidence but no source
- "This will go viral" with no virality mechanism described

**Examples:**
- "The market is $50B" — from a top-down TAM slide, not a bottoms-up calculation
- "Users have been asking for this for years" — from one request in a support thread

**Resolution:** Ask for the source of every quantitative claim. Ask who specifically made the qualitative claim. If no source exists, downgrade to Grade D and flag for validation.

---

### 6. Problem vs Solution

**What it is:** The builder cannot describe the problem without describing the solution. The problem statement is actually a solution statement.

**Detection signals:**
- Problem statement includes product features ("users need a dashboard that...")
- Cannot answer "what problem are you solving?" without describing the product
- Scope of the problem expands or contracts to fit the scope of the solution

**Examples:**
- "The problem is that users don't have a way to [thing our product does]"
- "People need [our specific technical approach] to solve [vague problem]"

**Resolution:** Ask for a problem statement in one sentence with no mention of a product, feature, or solution. If they cannot produce one, the problem is not yet understood independently of the solution.

---

### 7. Persona vs Feature

**What it is:** Features in the plan do not map to any stated user need or persona.

**Detection signals:**
- Admin dashboard in a consumer product with no admin user described
- Enterprise reporting features for a product targeting individual contributors
- Features that serve a future user that does not exist yet

**Examples:**
- "We'll build team management" but the primary user is a solo freelancer
- "Analytics dashboard" for a product where the user has no use for analytics

**Resolution:** For each feature, require a mapping to a specific persona and a specific job-to-be-done. Features that cannot be mapped are cut or explicitly deferred with a clear trigger.

---

## Resolution Protocol

When a contradiction is detected:

1. **Name it explicitly.** "I want to flag a contradiction: in [skill/phase] you said [X], but now you're describing [Y]. These are in conflict."

2. **Do not interpret — ask.** "Which is actually true, or is the reality more nuanced than either?"

3. **Update confidence scores.** The phase that contained the contradicted claim has its confidence score reduced. Re-evaluate the phase gate.

4. **Re-ask downstream questions.** If the resolution changes a foundational answer, all questions that depended on that answer must be revisited.

5. **Document the resolution.** Record what the contradiction was, which answer was revised, and the current state.

```
CONTRADICTION LOG
─────────────────────────────────────────────
Type:          [One of the 7 patterns above]
Detected in:   Skill ___ / Question ___
Conflict:      "[Stated]" vs "[Revealed]"
Resolution:    ___
Revised claim: ___
Affected phases: ___  (re-examine these)
Confidence impact: -___% on Phase ___
─────────────────────────────────────────────
```

---

## Contradiction Severity

| Severity | Description | Required Action |
|----------|-------------|-----------------|
| Critical | Resolution changes the fundamental viability of the project | Stop. Resolve fully before continuing. Update verdict. |
| High | Resolution changes the scope, user, or core job-to-be-done | Resolve before proceeding to next sub-skill |
| Medium | Resolution changes a feature priority or assumption grade | Document and revisit in synthesis |
| Low | Resolution clarifies a specific answer without changing the overall picture | Note and continue |

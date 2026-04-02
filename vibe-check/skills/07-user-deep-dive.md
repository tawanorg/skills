# 07 — User Deep Dive

**Goal:** Build evidence-based personas from real observations and interviews — not fictional characters constructed from assumptions.

**When to use independently:** When personas exist but feel generic, or when the team disagrees about who the real user is.

---

## Question Matrix

| # | Question | What It Exposes | Evidence Type |
|---|----------|-----------------|---------------|
| 1 | Describe your primary user as a specific real person you have talked to. Give their name, role, company type, team size, and one sentence about their daily reality. | Whether the user is a real observed person or an archetype | Stated |
| 2 | Walk me through their entire workday. Where does this product fit in? What is happening immediately before and after they use it? | Workflow integration reality and usage context | Behavioral |
| 3 | What is their technical skill level? How much patience do they have for unclear UI? How many seconds before they abandon something that is not obvious? | UX complexity ceiling and onboarding tolerance | Behavioral |
| 4 | What is their emotional state when they reach for this type of tool? Calm and planning? Frustrated and reactive? Under pressure and time-constrained? | Design tone, urgency requirements, and error tolerance | Behavioral |
| 5 | How will they discover this product exists? Name the specific channel, the specific moment, and the specific message that would make them stop and pay attention. | Distribution reality and messaging specificity | Stated |
| 6 | Who else uses or is affected by this product? Does the secondary user's needs conflict with the primary user's? How do you resolve that conflict? | Multi-persona tension and design trade-off clarity | Stated |
| 7 | What does this user believe is true about this category of tool that is probably wrong? What misconception do you need to overcome? | Education and onboarding requirements | Inferred |
| 8 | What is this user's relationship with changing their workflow? Are they early adopters, pragmatists, or resistors? What evidence supports that? | Change adoption profile — changes your GTM entirely | Behavioral |
| 9 | When this user makes a mistake while using your product, what do they do? Do they try again, blame the tool, or blame themselves? | Error recovery behavior and support model requirements | Behavioral |
| 10 | What does success look like for this user after 30 days? 90 days? One year? What would they say changed? | Retention driver and long-term value definition | Stated |
| 11 | What would cause this user to stop using the product and never return — not bugs, but behavioral triggers and threshold violations? | Churn driver and hard red lines in the product experience | Behavioral |
| 12 | If you had to bet, what percentage of your stated target users actually look like the person you described? What does the other group look like? | Persona coverage and market segmentation accuracy | Inferred |

---

## Persona Evidence Standards

Each persona must have at minimum:
- One **direct interview** or usability session (Grade B or above)
- One **behavioral observation** of actual tool usage (Grade A)
- One **quantitative signal** (survey, analytics, support data)

A persona built only on assumption is labeled **UNVALIDATED** and must not drive product decisions until upgraded.

---

## Follow-up Probes (use when confidence < 70%)

- You described [user type]. Have you watched someone like that actually use a product in this category? What happened?
- The emotional state you described — calm and deliberate — but the workflow you described sounds reactive and time-pressured. Which is it?
- You said they discover via [channel]. Have you run any acquisition experiments on that channel? What did the data show?
- The secondary user you named — their needs and the primary user's needs conflict around [specific feature]. How have you resolved that in the design?
- You estimated [X% of your users] match this persona. Where does that number come from?

---

## Contradiction Patterns to Watch

| Pattern | Signal |
|---------|--------|
| Composite persona | Describes a single person who has every desirable characteristic but does not actually exist |
| Assumption persona | Cannot name a real person they have talked to for the primary persona |
| Emotional state mismatch | Describes calm deliberate user but builds high-urgency reactive product |
| Channel optimism | Discovery channel is aspirational, not tested |
| Conflict avoidance | Secondary user needs are acknowledged but the conflict is not resolved |

---

## Output Artifact: Persona Card(s)

One card per distinct user type. Evidence source tags required on every claim:

```
PERSONA CARD
─────────────────────────────────────────────────────────
Name:             ___  (real or composite — label which)
Role:             ___
Company type:     ___
Team size:        ___
Daily reality:    ___

CONTEXT OF USE
  When they reach for this:  ___
  5 minutes before:          ___
  5 minutes after:           ___
  Emotional state at access: Calm / Frustrated / Pressured / Panicked

CAPABILITY PROFILE
  Technical skill:   Low / Medium / High
  Patience level:    Low (abandons in <10s) / Medium / High
  Change adoption:   Early adopter / Pragmatist / Resistor

JOB TO BE DONE
  Functional:  ___
  Emotional:   ___
  Social:      ___

DISCOVERY PATH
  Channel:     ___
  Moment:      ___
  Message:     ___

CURRENT SOLUTION
  What they use now: ___
  Why they stay:     ___
  Switching trigger: ___

KEY PAIN POINTS (with evidence source)
  1. ___  [Source: Interview / Survey / Analytics / Assumed]
  2. ___  [Source: Interview / Survey / Analytics / Assumed]
  3. ___  [Source: Interview / Survey / Analytics / Assumed]

ANXIETIES ABOUT SWITCHING
  1. ___
  2. ___

SUCCESS MILESTONES
  Day 30:   ___
  Day 90:   ___
  Year 1:   ___

CHURN TRIGGERS (behavioral, not bug-related)
  1. ___
  2. ___

EVIDENCE STATUS
  [ ] Direct interview conducted
  [ ] Behavioral observation recorded
  [ ] Quantitative signal exists

VALIDATION STATUS:  Validated / Partially Validated / UNVALIDATED

PHASE CONFIDENCE:  ___% (must reach 80% before proceeding)
EVIDENCE GRADE:    A / B / C / D / F
─────────────────────────────────────────────────────────
```

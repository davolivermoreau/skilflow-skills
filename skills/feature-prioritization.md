# Feature Prioritization Framework

Scores and ranks features using ICE (Impact, Confidence, Ease) scoring so teams can make defensible prioritization decisions without endless debate.

## What this skill does

Takes a list of features or initiatives and applies a consistent scoring framework to produce a ranked priority list with reasoning.

## How to use

Provide a list of features/initiatives you need to prioritize. Optionally provide context on your goals, constraints, and team size. Say "prioritize these features using this skill."

## The ICE framework

Each feature is scored 1-10 on three dimensions:

**Impact** — If this works, how much does it move the metric that matters?
- 1-3: Minor improvement, small audience
- 4-6: Meaningful improvement for core users
- 7-9: Significant impact on key metric
- 10: Game-changing, affects everyone

**Confidence** — How sure are we this will work?
- 1-3: Gut feel, no data
- 4-6: Some evidence, similar features worked elsewhere
- 7-9: Strong data, user research, proven pattern
- 10: Validated, tested, we've done this before

**Ease** — How quickly and cheaply can we build and ship this?
- 1-3: Months of work, complex dependencies
- 4-6: Weeks of work, some complexity
- 7-9: Days of work, straightforward
- 10: Hours, already mostly built

**ICE Score = (Impact × Confidence × Ease) / 3**

---

## Output format

| Feature | Impact | Confidence | Ease | ICE Score | Rationale |
|---------|--------|------------|------|-----------|-----------|
| [Feature A] | 8 | 7 | 6 | 56 | [1 sentence] |
| [Feature B] | 9 | 5 | 3 | 45 | [1 sentence] |

**Recommended order:** [ranked list]

**Flags:**
- High impact but low confidence: needs validation before building
- High ease but low impact: quick wins, do in parallel
- Low everything: cut or defer

---

## Important caveats

ICE is a starting point, not a verdict. Always sanity-check against:
- Strategic alignment (does this serve our current focus?)
- Dependencies (does something else need to ship first?)
- Team morale (sometimes the right call is the energizing one)

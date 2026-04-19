# User Story Writer

Converts feature ideas and product requests into well-formed user stories with clear acceptance criteria that developers can actually build from.

## What this skill does

Takes a rough feature idea — a sentence, a Slack message, a customer complaint — and structures it into a proper user story with context, criteria, and edge cases.

## How to use

Describe the feature or problem and say "write a user story for this."

## Output format

**User story**
As a [specific user type], I want to [specific action] so that [specific outcome/value].

**Context**
Why this matters. What problem it solves. Who asked for it.

**Acceptance criteria**
- [ ] Given [condition], when [action], then [outcome]
- [ ] Given [condition], when [action], then [outcome]
- [ ] (Add as many as needed — one behavior per criterion)

**Edge cases to consider**
- What happens when [unusual condition]?
- What if the user [unexpected behavior]?

**Out of scope**
Explicitly list what this story does NOT include.

**Dependencies**
Other stories, systems, or teams this touches.

---

## Quality checks

A good user story:
- Names a specific user type, not "user" or "admin" generically
- Describes one behavior, not a feature bundle
- Has criteria that can be tested — pass or fail, no ambiguity
- Includes the "so that" — the why, not just the what
- Is small enough to complete in one sprint

If the request is too large for one story, split it and say so.

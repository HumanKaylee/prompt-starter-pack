---
name: Requirements Elicitation
use_case: Turn a vague feature request into a concrete, implementable specification
variables:
  - FEATURE_REQUEST: the original request (can be rough/informal)
  - SYSTEM_CONTEXT: what system/product this is for
  - STAKEHOLDER: who requested this and what they care about
  - KNOWN_CONSTRAINTS: technical or business constraints already known
---

# Requirements Elicitation Prompt

## Prompt

```
You are a product engineer helping translate a vague feature request into a precise specification. Your job is to identify ambiguities, ask the right clarifying questions, and then draft a spec.

**Original request:** {{FEATURE_REQUEST}}

**System context:** {{SYSTEM_CONTEXT}}

**Requested by:** {{STAKEHOLDER}} (their primary goal: [infer from context])

**Known constraints:** {{KNOWN_CONSTRAINTS}}

Step 1 — Identify ambiguities:
List the top 5 questions I need to answer before I can write code. Rank by impact on the implementation. For each question, note what the two most likely answers are and how each would change the design.

Step 2 — Draft the spec (with assumptions):
Assume the most common/reasonable answer to each ambiguity and draft:

### User Stories
As a [user type], I want to [action] so that [outcome]. (3-5 stories)

### Acceptance Criteria
For each user story: specific, testable conditions. Use "Given/When/Then" format.

### Out of Scope
Explicit list of what this feature does NOT cover.

### Open Questions
Remaining decisions that need stakeholder input before implementation starts.

### Rough Effort Estimate
T-shirt size (S/M/L/XL) with a one-sentence justification.
```

## Worked Example

**Feature request:** "We need to let users export their data"

**What a good response looks like:** Top ambiguity is "what data? all of it, or just a specific entity?" with two likely answers (full account export vs. per-resource export); spec draft assumes full account export with a background job + email link; acceptance criteria include "user receives email within 10 minutes containing a link that expires in 24 hours"; out of scope includes real-time streaming export and partial field selection.

## Tuning Notes

- Use this before writing any code. The 30 minutes spent on a spec prevents 3 days of rework.
- If you're the requester writing the spec yourself: paste in your own requirements and ask "what are the holes in this spec?"
- For API design: follow with the API Design template once the requirements are stable.
- Add: "Also identify any requirements that could be phased — what's MVP vs. what can wait for v2?"

---
name: Architecture Decision Record (ADR) Generator
use_case: Document an architectural decision with full context, options considered, and rationale
variables:
  - DECISION_TITLE: short name for the decision (e.g., "Use PostgreSQL for session storage")
  - PROBLEM: what problem you're solving and why it matters now
  - CONTEXT: system constraints, current architecture, team size, scale requirements
  - OPTIONS: the options you're considering (can be rough notes)
  - CHOSEN_OPTION: which option you're leaning toward (or "help me decide")
---

# Architecture Decision Record Generator Prompt

## Prompt

```
Generate a well-structured Architecture Decision Record (ADR) following the MADR format.

**Decision:** {{DECISION_TITLE}}

**Problem to solve:** {{PROBLEM}}

**System context:** {{CONTEXT}}

**Options under consideration:** {{OPTIONS}}

**Leaning toward:** {{CHOSEN_OPTION}}

Produce the ADR in this format:

---
# ADR-XXX: {{DECISION_TITLE}}

**Date:** [today]
**Status:** Proposed

## Context and Problem Statement
[2-3 sentences: what situation forces this decision, and what happens if we don't decide]

## Decision Drivers
[Bulleted list of the constraints and quality attributes that matter most for this decision]

## Considered Options
[For each option: name, one-paragraph description, pros, cons]

## Decision Outcome
**Chosen option:** [name]
**Rationale:** [2-3 sentences explaining why this option best satisfies the decision drivers]

## Consequences
**Positive:** [what gets better]
**Negative:** [what gets harder or what we give up]
**Risks:** [what could go wrong with this choice]

## Follow-up Actions
[Numbered list of concrete next steps to implement this decision]
---

If you're helping me decide (CHOSEN_OPTION = "help me decide"), score each option against the decision drivers with a 1-3 scale and recommend the highest-scoring option.
```

## Worked Example

**Decision Title:** "Event sourcing vs direct DB writes for order state"

**What a good response looks like:** ADR clearly separates the two options with honest trade-offs (event sourcing wins on auditability/replay but loses on query complexity and ops overhead); decision drivers include "must support point-in-time replay for regulatory compliance"; follow-up actions are concrete (e.g., "Spike: implement a single aggregate with 3 events and measure query latency by Friday").

## Tuning Notes

- Run this for major decisions BEFORE implementation — the act of writing the ADR often reveals gaps in your reasoning.
- If you already have a decision, use it to pressure-test: ask the model to steelman the rejected options. If it can't, the chosen option is probably right.
- Store ADRs in `docs/decisions/` with sequential numbering (ADR-001, ADR-002...). They become invaluable when onboarding new engineers.
- Add a `Superseded by` field and link to the successor ADR when you revisit a decision.

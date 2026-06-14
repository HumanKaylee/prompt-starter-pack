---
name: Feature Flag Strategy
use_case: Design a safe rollout and rollback plan for a new feature controlled by a flag
variables:
  - FEATURE_NAME: feature being gated
  - USER_SEGMENTS: internal, beta, geography, paid tier, etc.
  - RISK_LEVEL: high / medium / low
  - SUCCESS_SIGNAL: metric or behavior that says the rollout is healthy
  - ROLLBACK_TRIGGER: condition that should turn the flag off
---

# Feature Flag Strategy Prompt

## Prompt

```
You are a senior engineer designing a rollout strategy for a feature flag. Optimize for safety, observability, and cleanup — not just "ship it behind a flag."

**Feature:** {{FEATURE_NAME}}
**User segments:** {{USER_SEGMENTS}}
**Risk level:** {{RISK_LEVEL}}
**Success signal:** {{SUCCESS_SIGNAL}}
**Rollback trigger:** {{ROLLBACK_TRIGGER}}

Produce:

### Flag Intent
What the flag protects and what failure mode it isolates.

### Rollout Phases
Create a step-by-step rollout from internal users to full exposure. For each phase include:
- target audience
- percentage or cohort
- minimum observation window
- go criteria
- rollback criteria

### Operational Rules
Specify:
- default behavior if the flag system is unavailable
- where the flag should be evaluated in code
- how to avoid per-request randomness
- how support/on-call can verify current state quickly

### Monitoring Plan
What to watch during rollout and what threshold should pause expansion.

### Cleanup Plan
When the flag should be deleted, what code path should disappear, and what evidence says it is safe to remove.

If the feature does not justify a flag, say so and recommend the simpler mechanism.
```

## Worked Example

**Variables:**
- FEATURE_NAME: new self-serve team billing flow
- USER_SEGMENTS: internal staff, beta customers, all paid teams
- RISK_LEVEL: high
- SUCCESS_SIGNAL: billing completion rate stays within 2% of baseline
- ROLLBACK_TRIGGER: checkout errors exceed 1% for flagged users

**What a good response looks like:** The model starts with internal-only rollout, uses deterministic user targeting, sets an explicit observation window per phase, and schedules a cleanup date instead of letting the flag live forever.

## Tuning Notes

- Ask for LaunchDarkly/Unleash-specific output if you need vendor-shaped config.
- Add `DEPENDENT_FLAGS` when multiple switches interact; otherwise the model may miss coupled failure modes.
- Use this lighter template for fast planning; use a deeper premium prompt when you need statistical gating and full cleanup ticketing.

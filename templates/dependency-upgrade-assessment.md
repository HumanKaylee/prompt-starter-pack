---
name: Dependency Upgrade Assessment
use_case: Evaluate whether a dependency upgrade is worth doing now, what it will break, and how to land it safely
variables:
  - DEPENDENCY_NAME: package/library to assess
  - CURRENT_VERSION: current version in use
  - TARGET_VERSION: proposed new version
  - USAGE_SURFACE: where and how the dependency is used
  - KNOWN_DRIVERS: security fix, performance, new feature, ecosystem support, etc.
---

# Dependency Upgrade Assessment Prompt

## Prompt

```
You are a senior engineer assessing a dependency upgrade. Your job is to decide whether this upgrade should happen now, later, or not at all.

**Dependency:** {{DEPENDENCY_NAME}}
**Current version:** {{CURRENT_VERSION}}
**Target version:** {{TARGET_VERSION}}
**Usage surface:** {{USAGE_SURFACE}}
**Upgrade drivers:** {{KNOWN_DRIVERS}}

Return:

### Recommendation
Choose one: upgrade now / defer / reject. Lead with the answer.

### Why This Upgrade Exists
Summarize the real reasons this is being considered and separate hard drivers from nice-to-haves.

### Breaking-Risk Analysis
List the most likely break points:
- API changes
- transitive dependency shifts
- runtime/platform support changes
- config/env changes
- test fragility

### Validation Plan
Describe the minimum proof needed before merging:
- code search targets
- tests to run
- smoke paths to verify
- rollback trigger

### Migration Notes
If code changes are likely, provide the most probable edits or compatibility shims.

### Decision Table
| Factor | Assessment | Notes |
|--------|------------|-------|
| Security urgency | low/medium/high | |
| Engineering effort | low/medium/high | |
| Blast radius | low/medium/high | |
| Benefit | low/medium/high | |

Be skeptical of upgrades justified only by "being current." If evidence is missing, state what needs to be verified before deciding.
```

## Worked Example

**Variables:**
- DEPENDENCY_NAME: `react-router`
- CURRENT_VERSION: `6.21`
- TARGET_VERSION: `7.x`
- USAGE_SURFACE: route definitions, data loaders, test helpers, deep-linked dashboards
- KNOWN_DRIVERS: new type-safety improvements; long-term support posture

**What a good response looks like:** The model flags likely loader/action API differences, notes that the upgrade is not security-driven, and recommends a staged proof branch with targeted route smoke tests before any org-wide rollout.

## Tuning Notes

- Add release notes or changelog excerpts when available; otherwise the model will infer from typical upgrade patterns.
- If you want a commit-ready sequence, follow up with "turn the validation plan into a checklist with owners."
- Useful precursor to a deeper `dependency-upgrade-plan` prompt when the assessment says "upgrade now."

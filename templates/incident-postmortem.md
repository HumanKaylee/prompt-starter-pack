---
name: Incident Postmortem
use_case: Draft a blameless postmortem document from incident notes
variables:
  - INCIDENT_SUMMARY: what happened and when (a few sentences)
  - TIMELINE: rough chronological notes of the incident (can be messy)
  - IMPACT: who/what was affected and for how long
  - ROOT_CAUSE_HYPOTHESIS: your best current theory about what went wrong
  - CONTRIBUTING_FACTORS: things that made it worse or harder to detect
---

# Incident Postmortem Prompt

## Prompt

```
Draft a blameless postmortem from these incident notes. The goal is to understand the system failure, not to assign blame. Focus on the contributing conditions and what the organization can learn.

**Incident summary:** {{INCIDENT_SUMMARY}}

**Timeline (rough notes):**
{{TIMELINE}}

**Impact:** {{IMPACT}}

**Root cause hypothesis:** {{ROOT_CAUSE_HYPOTHESIS}}

**Contributing factors:** {{CONTRIBUTING_FACTORS}}

Produce:

## Incident Postmortem: [Title]

**Date:** [from timeline]
**Severity:** [P0/P1/P2 based on impact]
**Duration:** [from first impact to full resolution]
**Status:** Draft

### Impact Summary
Quantified impact: affected users, requests, revenue, SLO burn, etc.

### Timeline
Clean, timestamped timeline from first alert to resolution. Mark key decision points.

### Root Cause Analysis
The "Five Whys" chain: trace from symptom to root cause. The root cause is a system condition, not a person's action.

### Contributing Factors
What made this failure possible, harder to detect, or harder to fix? (monitoring gaps, missing runbooks, unclear ownership, etc.)

### What Went Well
Things that limited the impact or accelerated recovery.

### Action Items
| Priority | Action | Owner | Due | Done? |
|----------|--------|-------|-----|-------|
Each action must be specific and verifiable. No "improve monitoring" — instead "add alert for cache hit rate dropping below 80% for >5 min".

### Lessons Learned
2-3 durable insights that apply beyond this specific incident.
```

## Worked Example

**Incident:** Database connection pool exhaustion during a traffic spike; 23 minutes of elevated error rates

**What a good response looks like:** Timeline is clean with 7 entries; root cause traces to misconfigured max_connections that was set during initial deployment and never revisited as traffic grew; action items include specific Grafana alert configuration, runbook creation with exact kubectl commands, and a load-test task; "what went well" credits the on-call engineer's quick escalation.

## Tuning Notes

- Run this ASAP after the incident while the timeline is fresh. Refine later.
- "Blameless" doesn't mean "no accountability" — action items have owners. It means the analysis looks at system conditions, not individual mistakes.
- If you need to redact for external sharing: ask the model to "produce a public-facing version of this postmortem with internal system names and customer details removed."
- For regulated industries: add a "Compliance Notes" section asking the model to flag any regulatory reporting obligations.

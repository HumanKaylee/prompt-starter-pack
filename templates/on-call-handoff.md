---
name: On-Call Handoff
use_case: Hand off operational context cleanly between engineers so incidents, risks, and scheduled work do not get dropped
variables:
  - SERVICE_SCOPE: systems or services covered by the handoff
  - CURRENT_STATE: incidents, noisy alerts, known degradations, or maintenance in flight
  - UPCOMING_RISKS: deploys, expirations, migrations, expected traffic spikes
  - RUNBOOKS_AND_DASHBOARDS: relevant links or file paths
---

# On-Call Handoff Prompt

## Prompt

```
You are an experienced SRE writing an on-call handoff. Be terse, specific, and biased toward what the next person must know in the first five minutes.

**Service scope:** {{SERVICE_SCOPE}}
**Current state:** {{CURRENT_STATE}}
**Upcoming risks:** {{UPCOMING_RISKS}}
**Runbooks / dashboards:** {{RUNBOOKS_AND_DASHBOARDS}}

Write the handoff in this format:

### Shift Summary
One paragraph: what changed, what is stable, and what still needs watching.

### Active Issues
For each issue: severity, current user impact, latest evidence, next expected action, and who owns it if not on-call.

### Noisy But Non-Critical Alerts
List alerts that can be safely deprioritized and why.

### Time-Sensitive Watchouts
Anything likely to page or break soon: scheduled deploys, cert expiry, traffic events, batch jobs, migrations.

### First Checks For The New On-Call
Three to five concrete things to verify at the start of the next shift.

### Escalation Guidance
State when the next on-call should escalate immediately versus continue observing.

If the incoming engineer is walking into a calm shift, say so plainly instead of inventing drama.
```

## Worked Example

**Variables:**
- SERVICE_SCOPE: checkout API, payments webhook worker, PostgreSQL primary
- CURRENT_STATE: intermittent webhook retries from one provider; error budget otherwise healthy
- UPCOMING_RISKS: midnight ET deploy freeze lift; marketing email at 09:00 ET
- RUNBOOKS_AND_DASHBOARDS: payments runbook, Grafana checkout dashboard, webhook dead-letter queue

**What a good response looks like:** The model distinguishes the noisy retry pattern from a real outage, tells the incoming engineer exactly which dashboard to check first, and names the one condition that should trigger immediate escalation.

## Tuning Notes

- Add timestamps to CURRENT_STATE when the chronology matters.
- If multiple incidents exist, ask for priority ordering by customer impact.
- This template pairs well with `incident-postmortem.md` after the shift ends.

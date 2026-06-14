---
name: Capacity Planning
use_case: Estimate whether a system can handle expected load and what to scale, cache, or redesign first
variables:
  - SYSTEM_NAME: service or subsystem being evaluated
  - CURRENT_TRAFFIC: present request/job volume and growth pattern
  - EXPECTED_CHANGE: launch, migration, seasonal spike, customer growth, etc.
  - BOTTLENECK_SIGNALS: CPU, memory, DB saturation, queue lag, rate limits, unknowns
---

# Capacity Planning Prompt

## Prompt

```
You are a senior engineer doing capacity planning for a production system. Be numerical when possible and explicit about assumptions when numbers are missing.

**System:** {{SYSTEM_NAME}}
**Current traffic:** {{CURRENT_TRAFFIC}}
**Expected change:** {{EXPECTED_CHANGE}}
**Known bottleneck signals:** {{BOTTLENECK_SIGNALS}}

Return:

### Executive Answer
Can the current system likely absorb the change? Answer yes / maybe / no, with one sentence of justification.

### Assumptions
List the assumptions required to make the estimate.

### Bottleneck Ranking
Rank the most likely limits first: compute, database, cache, downstream APIs, background workers, storage, rate limits.

### Capacity Plan
For each likely bottleneck:
- what metric to inspect
- what threshold is concerning
- cheapest mitigation
- slower but more durable fix

### Load Test Or Measurement Gaps
State what evidence is missing and the smallest test that would close the gap.

### Launch Guardrails
Recommend dashboards, alerts, and rollback triggers for the period when the load change hits.

Do not hide uncertainty. If the inputs are too vague for a real estimate, say exactly which numbers are missing.
```

## Worked Example

**Variables:**
- SYSTEM_NAME: image-processing API
- CURRENT_TRAFFIC: 120 req/s daytime peak, 2x month-over-month growth
- EXPECTED_CHANGE: marketing launch expected to double uploads for 72 hours
- BOTTLENECK_SIGNALS: worker CPU near 75% at peak, queue lag spikes, object-store egress limits unknown

**What a good response looks like:** The model identifies worker CPU and queue lag as first-order limits, asks for concurrency and average job duration, and recommends a short controlled load test before trusting the launch plan.

## Tuning Notes

- Add historical graphs or percentiles when available; capacity estimates get much better with p95/p99 latency and queue depth.
- If the system depends on third-party APIs, ask for a dependency-by-dependency quota table.
- Pairs well with `load-testing-plan.md` when the answer is "maybe."

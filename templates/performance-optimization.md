---
name: Performance Optimization Analysis
use_case: Identify and prioritize performance bottlenecks in code or system design
variables:
  - CODE_OR_DESIGN: the code, query, or architecture description to analyze
  - PERFORMANCE_SYMPTOM: what's slow (e.g., "API endpoint p99 = 3s", "query takes 12s on 10M rows", "UI renders 8s")
  - SCALE: current and target scale (e.g., "currently 1k req/s, need to handle 10k req/s")
  - PROFILING_DATA: any profiling output, query plans, or metrics you have (or "none")
---

# Performance Optimization Analysis Prompt

## Prompt

```
Analyze this code/system for performance bottlenecks and produce a prioritized optimization plan.

**Symptom:** {{PERFORMANCE_SYMPTOM}}
**Scale requirements:** {{SCALE}}
**Profiling data available:** {{PROFILING_DATA}}

**Code/design to analyze:**
{{CODE_OR_DESIGN}}

Produce:

### Bottleneck Identification
Ranked list of suspected bottlenecks (most impactful first). For each:
- What the bottleneck is
- Why you suspect it (specific code line or design decision)
- Estimated impact if fixed (order of magnitude: 2x, 10x, etc.)
- Confidence level (high/medium/low — based on available data)

### Quick Wins (< 1 day each)
Changes with high impact and low implementation risk. Be specific: exact change, expected improvement.

### Medium-Term Optimizations (1-5 days)
Deeper structural changes worth the investment. Include trade-offs.

### What NOT to Optimize
Explicitly list optimizations that would be premature, harmful, or insufficient. Saves time on dead ends.

### Measurement Plan
How to verify each optimization actually helped. Specific metrics to capture, before and after.

### When to Stop
At what measured improvement should I stop optimizing? What's "good enough" for the stated requirements?
```

## Worked Example

**Symptom:** List page taking 4.2s for customers with >1000 orders

**What a good response looks like:** Top bottleneck is N+1 query (fetching order items individually in a loop); quick win is adding a `.select_related()` or JOIN; medium-term is pagination + cursor-based loading; "what not to optimize" calls out adding a cache (premature — the query fix will likely get under 200ms); measurement plan specifies EXPLAIN ANALYZE before/after + p95 from APM.

## Tuning Notes

- Without profiling data, the model is guessing at bottlenecks. Even crude timing (`time curl ...` or `console.time()`) narrows the search dramatically.
- "Make it faster" is not a requirement. Specify the target: "must be under 200ms at p99 for 10k req/s."
- Add: "What are the failure modes of these optimizations?" — some perf improvements (e.g., aggressive caching) introduce correctness bugs.
- For database queries: always include the schema (column names, indexes) — the model needs this to identify missing indexes.

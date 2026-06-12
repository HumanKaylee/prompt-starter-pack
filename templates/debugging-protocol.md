---
name: Debugging Protocol
use_case: Systematically diagnose a bug you cannot reproduce or understand
variables:
  - SYMPTOM: what you observe (error message, wrong output, crash, etc.)
  - CONTEXT: language, framework, environment where it occurs
  - WHAT_YOU_TRIED: debugging steps already taken
  - CODE_SNIPPET: the relevant code (function, class, or module)
  - LOGS: any error logs, stack traces, or diagnostic output
---

# Debugging Protocol Prompt

## Prompt

```
You are a senior engineer helping me debug a production issue. Think like a detective: form hypotheses from evidence, rank them by probability, and propose the cheapest test to rule each one out.

**Symptom:** {{SYMPTOM}}

**Environment:** {{CONTEXT}}

**What I've already tried:** {{WHAT_YOU_TRIED}}

**Relevant code:**
{{CODE_SNIPPET}}

**Logs/errors:**
{{LOGS}}

Respond with:

### Root Cause Hypotheses
Ranked list (most likely first). For each hypothesis: one-sentence explanation of the causal chain, and what evidence would confirm or rule it out.

### Cheapest Diagnostic Step
The single fastest test I can run RIGHT NOW to narrow the field. Be specific: exact command, log line to look for, or assertion to add.

### If the top hypothesis is correct: fix
Concrete code change or configuration fix. Show the before/after.

### Blind spots to check
Anything I might be missing that doesn't fit neatly into the hypotheses above (e.g., race conditions, environment-specific behavior, third-party library bugs).
```

## Worked Example

**Symptom:** `TypeError: Cannot read property 'id' of undefined` thrown intermittently (1 in ~500 requests) in a Node.js API

**What a good response looks like:** Top hypothesis is a race condition where a user object is read before an async database write completes; diagnostic step is adding a console.log of the user object right before the failing line and replaying a request with artificial latency; fix is awaiting the write before proceeding.

## Tuning Notes

- "Intermittently" is a keyword that should push the model toward timing/concurrency hypotheses. If you know it's deterministic, say so.
- Include the full stack trace when available — models extract signal from line numbers and frame order.
- If you have a minimal reproducer, say so: "I can reproduce this with X". The model will shift from diagnostic mode to fix mode.
- For Heisenbugs (disappear when you add logging): mention explicitly. The model will skip suggestions involving additional logging.

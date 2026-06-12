---
name: Documentation Generator
use_case: Generate or improve documentation for code, APIs, or systems
variables:
  - CODE_OR_SYSTEM: the code or system to document
  - DOC_TYPE: what kind of docs (e.g., "inline docstrings", "README", "API reference", "runbook", "architecture overview")
  - AUDIENCE: who will read this (e.g., "new engineers on the team", "external API users", "on-call engineers at 3am")
  - EXISTING_DOCS: any existing documentation to improve (or "none")
---

# Documentation Generator Prompt

## Prompt

```
Write {{DOC_TYPE}} documentation for the following code/system. Write for this specific audience: {{AUDIENCE}}.

**Design principle:** Documentation should tell readers what they need to do, not repeat what the code says. If a docstring just restates the function name, it's worthless.

**Existing docs to improve:** {{EXISTING_DOCS}}

**Code/system to document:**
{{CODE_OR_SYSTEM}}

For DOC_TYPE = "inline docstrings":
Write docstrings for every public function/class. Include: what it does (the WHY, not the WHAT), parameters (name, type, description, whether required), return value, exceptions raised, any important side effects.

For DOC_TYPE = "README":
Write a README with: one-paragraph elevator pitch, prerequisites, quickstart (< 5 commands), full configuration reference, common usage examples, troubleshooting section with top 3 issues, contributing guide.

For DOC_TYPE = "runbook":
Write a runbook for operators. Include: normal operation indicators, alert definitions (what triggered this, what it means), diagnostic steps (ordered by likelihood), remediation procedures (exact commands), escalation path, rollback procedure.

For DOC_TYPE = "architecture overview":
Write a 1-2 page architecture doc with: system purpose, component diagram description, data flow, key design decisions and their rationale, operational characteristics (scale, SLAs, dependencies).

Avoid: "this function does...", passive voice, obvious statements, filler text.
```

## Worked Example

**DOC_TYPE:** Runbook for a payment processing service

**What a good response looks like:** Alert section explains "Payment processing lag > 30s means the payment queue is backed up, NOT that payments are failing — check queue depth before paging the payment team"; diagnostic steps start with `kubectl get pods -n payments` and check for recent deploys before diving into logs; remediation includes the exact command to scale up workers.

## Tuning Notes

- For inline docstrings: run this file-by-file rather than pasting entire repos. Quality degrades on very long inputs.
- The best documentation prompt includes an example of a GOOD docstring from your codebase — the model will match the style.
- For README: add "assume the reader is evaluating this project in the first 60 seconds" to sharpen the quickstart section.
- Ask the model to identify the top 3 places where documentation is most critically missing in your codebase — then run this prompt on those.

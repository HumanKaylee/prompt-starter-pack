---
name: Commit Message
use_case: Write a clear, conventional commit message from a diff or change description
variables:
  - DIFF_OR_DESCRIPTION: the git diff or plain-language description of changes
  - REPO_CONVENTION: commit style used by the project (e.g., "conventional commits", "imperative mood", "ticket prefix PROJ-123")
  - SCOPE: optional component/module affected (e.g., "auth", "payments", "api")
---

# Commit Message Prompt

## Prompt

```
Write a git commit message for the following change. Follow {{REPO_CONVENTION}} conventions.

**Scope/component:** {{SCOPE}}

**Change:**
{{DIFF_OR_DESCRIPTION}}

Rules:
1. Subject line: ≤72 characters, {{REPO_CONVENTION}} format, imperative mood ("add" not "added")
2. Body (if needed): explain the WHY, not the WHAT (the diff shows the what)
3. Footer: reference any issue numbers mentioned in the change
4. Do NOT include: vague words ("fix", "update", "change" alone), what the code does mechanically, or marketing language

Output format:
```
<type>(<scope>): <subject>

<body — why this change, what problem it solves>

<footer>
```

If the change is small and self-explanatory, a subject line only is fine.

Suggest 3 variants ranked from most to least descriptive, so I can pick the right level of detail.
```

## Worked Example

**Change:** Added retry logic with exponential backoff to the payment processor API client; previously, transient 503 errors from the payment provider caused the entire checkout flow to fail immediately.

**What a good response looks like:**
1. `fix(payments): retry transient 503 errors with exponential backoff` + body explaining that unretried errors were causing checkout failures during provider maintenance windows
2. `fix(payments): add exponential backoff to payment API client`
3. `fix: handle transient payment provider errors`

## Tuning Notes

- For conventional commits: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `ci` are the standard types.
- If the diff spans multiple concerns (e.g., a feature + a bug fix), ask the model: "Should this be one commit or multiple? If multiple, draft the messages for each."
- Add `"Breaking change: <description>"` to the footer if the change breaks the public API.
- For squash-merge workflows: ask for a summary commit message that rolls up multiple commits.

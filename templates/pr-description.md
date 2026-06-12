---
name: Pull Request Description
use_case: Write a comprehensive PR description that helps reviewers understand context and test efficiently
variables:
  - TITLE: what this PR does (one sentence)
  - COMMITS_OR_DIFF: git log summary or diff of the full PR
  - LINKED_ISSUE: issue/ticket number or "none"
  - TESTING_DONE: what you tested manually or automatically
  - RISK_LEVEL: low / medium / high and brief reason
---

# Pull Request Description Prompt

## Prompt

```
Write a pull request description that helps reviewers understand why this change exists and how to verify it works. Write for someone who hasn't been staring at this code all week.

**Title:** {{TITLE}}

**Linked issue/ticket:** {{LINKED_ISSUE}}

**Risk level:** {{RISK_LEVEL}}

**Changes (commits or diff summary):**
{{COMMITS_OR_DIFF}}

**Testing done:** {{TESTING_DONE}}

Structure the PR description as:

## Summary
2-4 bullet points covering: what changed, why it changed, what the alternative was, and any important trade-offs made.

## Test Plan
A checklist of how a reviewer can verify this works. Include:
- [ ] Automated tests that cover this change (and how to run them)
- [ ] Manual testing steps (specific actions, expected outcomes)
- [ ] Edge cases to check

## Rollback Plan
If RISK_LEVEL is medium or high: how to revert this if it causes problems in production.

## Screenshots / Output
[Leave blank if not applicable — don't fabricate]

## Notes for Reviewers
Anything specific you want reviewers to focus on, or context that isn't obvious from the code.
```

## Worked Example

**Title:** "Add caching layer to the user profile endpoint"

**What a good response looks like:** Summary explains the latency problem (p99 >800ms) and that Redis TTL-based caching reduces it to <50ms at the cost of up to 60s of stale data; test plan includes `pytest tests/test_profile_cache.py` and manual step of updating a user's name and confirming the stale response then the fresh response; rollback plan is the feature flag to disable caching.

## Tuning Notes

- Small PRs (1-2 files, obvious change): use just Summary + Test Plan.
- Include "What was NOT changed" for PRs that refactor adjacent code — this preempts "why didn't you also fix X?" review comments.
- For security-sensitive changes: always include a "Security Considerations" section even if the answer is "no security impact because..."
- If you're pairing this with the Code Review prompt: run the PR Description prompt first so reviewers have context, then ask for a code review.

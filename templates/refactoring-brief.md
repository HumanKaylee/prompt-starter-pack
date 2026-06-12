---
name: Refactoring Brief
use_case: Get a structured refactoring plan before touching messy code
variables:
  - CODE: the code to refactor
  - GOAL: what you want to achieve (e.g., "split into smaller functions", "make testable", "reduce duplication")
  - CONSTRAINTS: what must not change (e.g., "public API must stay identical", "no new dependencies")
  - TEST_COVERAGE: current test coverage situation (e.g., "80% unit tests", "no tests", "integration tests only")
---

# Refactoring Brief Prompt

## Prompt

```
You are helping me refactor existing code safely. Your job is to produce a step-by-step plan I can execute incrementally, where each step leaves the code in a working state.

**Goal:** {{GOAL}}

**Constraints:** {{CONSTRAINTS}}

**Test coverage:** {{TEST_COVERAGE}}

**Code to refactor:**
{{CODE}}

Produce:

### Diagnosis
What are the concrete problems with the current code? Be specific: name the anti-patterns, point to the lines, explain the downstream impact.

### Refactoring Strategy
High-level approach (e.g., extract-method, introduce interface, split class). Why this strategy fits the goal and constraints.

### Step-by-Step Plan
Ordered steps, each small enough to commit independently. For each step:
- What changes
- How to verify it didn't break anything (test to run or assertion to check)
- Risk level (low / medium / high) and why

### Tests to Write First
If test coverage is insufficient, list the minimum tests to write BEFORE starting the refactor. These are your safety net.

### What NOT to change
Explicit list of things to leave alone, based on the constraints.
```

## Worked Example

**Goal:** Split a 400-line `UserService` class that handles auth, profile updates, and email sending

**What a good response looks like:** Diagnosis names "Single Responsibility Principle violation" with specific method groupings; strategy is "extract class" for auth and email; plan has 6 steps each under 50 lines changed; first step is extracting a pure helper function with no external dependencies.

## Tuning Notes

- If you have zero tests: ask the model to generate characterization tests first. The prompt includes this naturally via "Tests to Write First".
- For large files (>500 lines): paste in the class/module signature + method names only, then ask the model what it needs to see in full.
- Append `"Avoid introducing new abstractions unless strictly necessary"` if you want conservative refactors.
- After each step, re-run the prompt with the new code and ask: "Did step N introduce any new issues?"

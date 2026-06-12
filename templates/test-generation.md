---
name: Test Generation
use_case: Generate comprehensive tests for a function, class, or module
variables:
  - CODE: the code to test
  - FRAMEWORK: test framework (e.g., pytest, Jest, Go testing, RSpec)
  - EXISTING_TESTS: any tests that already exist (paste or "none")
  - COVERAGE_GOAL: what you care most about (e.g., "edge cases", "error paths", "all branches", "happy path only")
---

# Test Generation Prompt

## Prompt

```
You are a test engineer writing {{FRAMEWORK}} tests for the following code. Write tests that actually catch bugs — not tests that just exercise the happy path.

**Coverage goal:** {{COVERAGE_GOAL}}

**Existing tests (do not duplicate):**
{{EXISTING_TESTS}}

**Code to test:**
{{CODE}}

Generate:

### Test Cases Table
A table listing every test case you plan to write: | Test Name | Scenario | Input | Expected Output | Why This Matters |

### Test Code
The actual test file, ready to run. Follow {{FRAMEWORK}} conventions exactly. Include:
- Happy path(s)
- Edge cases (empty input, null/None, zero, negative numbers, max values)
- Error paths (what happens when preconditions fail)
- Boundary conditions
- Any concurrency or state-mutation edge cases if relevant

### What's NOT tested and why
Explicit list of scenarios you're skipping and the justification (e.g., "external API calls — use integration tests instead").

### Test Data Factories
If the tests need complex objects, provide factory functions or fixtures.
```

## Worked Example

**Code:** A function `calculate_discount(user, cart)` that applies tiered discounts

**What a good response looks like:** Tests table includes 8 cases (new user/no discount, loyalty tier 1/2/3, cart above threshold, cart exactly at threshold, empty cart, None user); test code uses parametrize; edge cases include integer overflow for cart value; "not tested" section flags the DB lookup for user tier as requiring an integration test.

## Tuning Notes

- For TDD (writing tests first): replace `CODE` with an interface/signature and ask the model to write tests against the contract, not an implementation.
- Add `"Tests must be hermetic (no network, no disk, no real time)"` to enforce proper mocking.
- If the model generates tests that simply assert the current behavior without reasoning about correctness, ask: "Now identify which of these tests would pass even if the function had a subtle bug."
- For async code: specify the async paradigm (asyncio, promises, goroutines) explicitly.

---
name: Legacy Code Explainer
use_case: Understand code you didn't write and can't easily run
variables:
  - CODE: the legacy code to explain
  - LANGUAGE: programming language
  - WHAT_YOU_KNOW: any context you already have (system name, what module this is, approximate age)
  - SPECIFIC_QUESTION: optional specific question (or "general understanding")
---

# Legacy Code Explainer Prompt

## Prompt

```
You are helping me understand legacy code I need to modify safely. I did not write this code and I cannot easily run it.

**Language:** {{LANGUAGE}}

**What I know:** {{WHAT_YOU_KNOW}}

**Specific question:** {{SPECIFIC_QUESTION}}

**Code:**
{{CODE}}

Provide:

### One-Paragraph Summary
What does this code do? Write as if explaining to a senior engineer who's never seen this module.

### Line-by-Line Walkthrough
For the most important or confusing sections: explain what each block does, why it might have been written this way, and any non-obvious side effects.

### Hidden Dependencies and Assumptions
What does this code implicitly depend on? What would break it silently (globals, environment variables, implicit ordering, side effects)?

### Safe Change Zones
Where can I make changes with low risk of breaking things? Where should I be very careful?

### Tests I Should Write Before Touching This
Minimum set of characterization tests to write BEFORE modifying this code, so I can verify I haven't changed existing behavior.

### Code Smells and Technical Debt
What are the most significant problems with this code's design? (Don't fix them now — just identify them so I don't make them worse.)
```

## Worked Example

**Code:** A 200-line PHP function that handles order processing with mixed database logic, business rules, and email sending

**What a good response looks like:** Summary explains it's a synchronous order finalization pipeline; walkthrough flags a global `$_SESSION` dependency in the middle of the function; hidden assumption is that the `orders` table is using auto-increment IDs that are assumed sequential; safe change zones are the email formatting section; two characterization tests suggested before touching anything.

## Tuning Notes

- For extremely long files (>500 lines): paste in function signatures first and ask "which function should I read to understand [specific behavior]?" then paste only that function.
- If the code has obvious bugs: ask the model to flag them separately after the explanation, so you don't conflate "what it does" with "what it should do."
- Add: "Are there any patterns here that suggest this was auto-generated or copied from a template?" — occasionally reveals the actual source doc.
- Use this before AND after making changes: re-run on your modified version to verify the explanation still makes sense.

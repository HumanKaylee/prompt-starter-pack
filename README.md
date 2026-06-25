# Prompt Starter Pack

**Battle-tested prompt templates for developers.** Stop writing the same prompts from scratch. Start with templates that have been refined through real use.

---

### Want more? → [**Premium Pack (40 templates) — $9**](https://flamewulfe.gumroad.com/l/prompt-starter-pack-premium/LAUNCH9)

The free pack covers the everyday templates. The premium pack adds 20 more, including topics this pack intentionally skipped because they need more depth:

- **System design review** — pressure-test an architecture before you build it
- **STRIDE threat modeling** — structured security analysis for new features
- **Terraform / IaC review** — catch drift, blast-radius, and module coupling issues
- **SQL query tuning** — explain-plan driven optimization for slow queries
- **AI agent design** *(new)* — design agentic loops, tool schemas, and failure modes
- **LLM evaluation** *(new)* — build an eval harness for your prompt pipeline
- **RAG pipeline review** *(new)* — audit retrieval quality, chunking, and re-ranking
- **Prompt chain decomposition** *(new)* — break complex tasks into reliable multi-step chains

40 templates total. Same format, same tuning notes, same worked examples. One-time purchase, yours to keep.

**→ [Get the Premium Pack ($9)](https://flamewulfe.gumroad.com/l/prompt-starter-pack-premium/LAUNCH9)**

---


## Who this is for

You're a software engineer, tech lead, or engineering manager who uses AI assistants (Claude, GPT-4, Gemini, etc.) regularly. You want prompts that **actually work** — not generic advice, but specific templates with variables, worked examples, and tuning notes from real sessions.

If you landed here looking for **prompt templates for Claude**, **ChatGPT prompts for developers**, or **AI workflow templates** for real software work, start with this free pack first. The templates are written for practical engineering tasks, not generic "be smarter" prompting.

## Start here based on your task

- **Code review / PR review**: [`templates/code-review.md`](templates/code-review.md)
- **Debugging a hard bug**: [`templates/debugging-protocol.md`](templates/debugging-protocol.md)
- **Architecture / design choices**: [`templates/architecture-decision-record.md`](templates/architecture-decision-record.md)
- **Security / threat analysis**: [`templates/security-review.md`](templates/security-review.md)
- **Performance / slow queries / scaling**: [`templates/performance-optimization.md`](templates/performance-optimization.md)
- **Prompt-system or AI-native work**: the [Premium Pack](https://flamewulfe.gumroad.com/l/prompt-starter-pack-premium/LAUNCH9) adds templates for agent design, LLM evals, RAG review, and prompt-chain decomposition.

## What's inside

20 templates covering the most common developer tasks:

| Template | Use Case |
|---|---|
| [`code-review.md`](templates/code-review.md) | Get thorough, actionable feedback on a code change before merging |
| [`debugging-protocol.md`](templates/debugging-protocol.md) | Systematically diagnose a bug you cannot reproduce or understand |
| [`refactoring-brief.md`](templates/refactoring-brief.md) | Get a structured refactoring plan before touching messy code |
| [`test-generation.md`](templates/test-generation.md) | Generate comprehensive tests for a function, class, or module |
| [`architecture-decision-record.md`](templates/architecture-decision-record.md) | Document an architectural decision with full context and rationale |
| [`commit-message.md`](templates/commit-message.md) | Write clear, conventional commit messages from a diff |
| [`pr-description.md`](templates/pr-description.md) | Write comprehensive PR descriptions that help reviewers |
| [`incident-postmortem.md`](templates/incident-postmortem.md) | Draft a blameless postmortem from incident notes |
| [`requirements-elicitation.md`](templates/requirements-elicitation.md) | Turn a vague feature request into an implementable spec |
| [`legacy-code-explainer.md`](templates/legacy-code-explainer.md) | Understand code you didn't write and can't easily run |
| [`security-review.md`](templates/security-review.md) | Identify security vulnerabilities before code ships |
| [`api-design-review.md`](templates/api-design-review.md) | Review or design a REST/GraphQL/RPC API before implementation |
| [`performance-optimization.md`](templates/performance-optimization.md) | Identify and prioritize performance bottlenecks |
| [`documentation-generator.md`](templates/documentation-generator.md) | Generate docstrings, READMEs, runbooks, and architecture docs |
| [`database-schema-review.md`](templates/database-schema-review.md) | Review a database schema for correctness and performance |
| [`monorepo-migration.md`](templates/monorepo-migration.md) | Plan a safe migration from many repos to a monorepo |
| [`dependency-upgrade-assessment.md`](templates/dependency-upgrade-assessment.md) | Decide whether a risky dependency upgrade should happen now |
| [`on-call-handoff.md`](templates/on-call-handoff.md) | Hand off operational context between on-call engineers cleanly |
| [`feature-flag-strategy.md`](templates/feature-flag-strategy.md) | Design a safe rollout and rollback plan for flagged features |
| [`capacity-planning.md`](templates/capacity-planning.md) | Estimate whether a system can absorb expected load changes |

3 full worked examples showing complete sessions from setup to result:

| Example | What it demonstrates |
|---|---|
| [`code-review-session.md`](examples/code-review-session.md) | Catching a SQL injection + 4 other issues in a TypeScript API key middleware |
| [`debugging-session.md`](examples/debugging-session.md) | Diagnosing a TOCTOU race condition causing duplicate DB rows |
| [`adr-session.md`](examples/adr-session.md) | Deciding between Redis, PostgreSQL, and JWT for session storage |

## Quickstart

**1. Clone or download**
```bash
git clone https://github.com/HumanKaylee/prompt-starter-pack.git
# or: click "Code → Download ZIP" above
```

Prefer a quick skim first? Open [`index.html`](index.html) for a lightweight preview surface of the pack.

**2. Open a template**
```bash
cat templates/code-review.md
```

**3. Fill in your variables**

Each template has a frontmatter block listing the variables you need:
```yaml
variables:
  - LANGUAGE: programming language
  - CONTEXT: brief description of what the code does
  - DIFF_OR_CODE: the code or diff to review
```

Replace `{{VARIABLE_NAME}}` with your actual values.

**4. Paste into your AI assistant and send**

That's it. Read the Tuning Notes section for adjustments based on your specific situation.

## Template format

Every template has the same structure:

```
frontmatter     — name, use-case, variables to fill in
Prompt          — the actual prompt with {{VARIABLE}} placeholders
Worked Example  — a concrete example of the prompt in action
Tuning Notes    — how to adjust for edge cases
```

The **Tuning Notes** section is the part most prompt collections skip. It's where the real value is — knowing when to modify the prompt, when to chain it with another, and when it won't work.

## Works with any LLM

These templates are model-agnostic. They work with:
- [Claude](https://claude.ai) (Sonnet/Opus)
- [ChatGPT](https://chatgpt.com) (GPT-4o, o1)
- [Gemini](https://gemini.google.com) (1.5 Pro, 2.0 Flash)
- Local models via Ollama (Llama 3, Mistral, etc.)

For complex tasks (code review, debugging, security review): use your most capable model. For quick tasks (commit messages, simple refactors): a smaller/faster model works fine.

## What makes these different

Most prompt template collections are either:
- **Too generic** ("write a prompt asking for help with code")
- **Too prescriptive** ("use exactly this 500-word template")

These templates are calibrated from real engineering work. They:
- Include honest **Tuning Notes** about when they don't work
- Have **Worked Examples** from actual sessions, not fabricated demos
- Use **variables** so you can customize without rewriting the core prompt
- Tell you what **NOT** to do (often more valuable than the positive advice)

## Contributing

Found a template that works better? Have a use case that's missing? Open a PR.

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the template standard, style guide, and what makes a good addition.

## License

MIT — use these however you want, commercially or otherwise. Attribution appreciated but not required.

---

*Built by [HumanKaylee](https://github.com/HumanKaylee).*

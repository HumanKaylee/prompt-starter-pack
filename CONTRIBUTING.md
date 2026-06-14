# Contributing

Thanks for improving the Prompt Starter Pack.

## What belongs here

- New prompt templates for common developer workflows
- Better worked examples for existing templates
- Sharper tuning notes, caveats, and failure modes
- README, packaging, or repo-quality improvements

## Template standard

Every template should include:

1. Frontmatter with a clear name, use case, and variable list
2. A prompt body with `{{VARIABLE_NAME}}` placeholders
3. A worked example that shows realistic input and output shape
4. Tuning notes with at least one limit or failure case

## Style guide

- Prefer concrete engineering tasks over broad "general AI help" prompts
- Keep prompts adaptable across Claude, ChatGPT, Gemini, and local models
- Use plain Markdown and keep filenames lowercase with hyphens
- Make examples realistic; avoid polished marketing demos
- Call out when a prompt should not be used

## Submitting changes

1. Explain the problem the template solves
2. Note the target user or workflow
3. Include the new file or the exact section you changed
4. Mention any tradeoffs, limits, or model-specific caveats

## High-value additions

- Missing software-engineering workflows
- Better examples for the most-used templates
- Packaging improvements that make the repo easier to browse
- Corrections where a template is too vague or too brittle

---
name: Monorepo Migration
use_case: Plan a safe migration from many repos to a monorepo without breaking builds, ownership, or releases
variables:
  - CURRENT_REPOS: list of repos and what each one owns
  - TARGET_TOOLING: build/orchestration choice (e.g., Turborepo, Nx, Bazel, plain workspaces)
  - SHARED_CODE: packages or utilities expected to be shared after migration
  - RELEASE_CONSTRAINTS: constraints around CI, deploy cadence, or backwards compatibility
---

# Monorepo Migration Prompt

## Prompt

```
You are a staff-level engineer planning a migration from multiple repositories to a monorepo. Be practical, pessimistic about hidden coupling, and explicit about sequencing.

**Current repos:**
{{CURRENT_REPOS}}

**Target tooling:**
{{TARGET_TOOLING}}

**Expected shared code:**
{{SHARED_CODE}}

**Release constraints:**
{{RELEASE_CONSTRAINTS}}

Produce:

### 1. Migration Goals
State what problems the monorepo solves and what it must not make worse.

### 2. Risk Inventory
List the biggest migration risks: broken imports, CI slowdown, unclear ownership, release coupling, secret/config sprawl, and tooling mismatch.

### 3. Target Layout
Propose a concrete folder/package structure and explain why each major area belongs there.

### 4. Migration Plan
Give a phased sequence with explicit checkpoints:
- preparation
- repo import strategy
- build/test unification
- developer workflow changes
- release cutover
- cleanup

For each phase: name the success criteria, rollback option, and what evidence proves it worked.

### 5. CI/CD Changes
Describe how pull requests, selective test runs, caching, deployment triggers, and release tagging should change.

### 6. Ownership Model
Recommend CODEOWNERS boundaries, review expectations, and where shared platform ownership begins and ends.

### 7. First Week Checklist
Give the minimum set of concrete tasks the team should do first.

If a monorepo is the wrong move, say so directly and explain the cheaper alternative.
```

## Worked Example

**Variables:**
- CURRENT_REPOS: `web-app`, `api-service`, `shared-ui`, `infra-modules`
- TARGET_TOOLING: Turborepo + pnpm workspaces
- SHARED_CODE: design system, auth helpers, API schemas
- RELEASE_CONSTRAINTS: zero downtime for API deploys; frontend deploys daily

**What a good response looks like:** The model proposes importing `shared-ui` and schemas first, keeps infra code isolated with its own ownership boundary, and warns that forcing API and frontend into one release train would create unnecessary coupling.

## Tuning Notes

- Add `TEAM_TOPOLOGY` if ownership is messy and you want the plan to optimize for organizational boundaries.
- If the model hand-waves "just merge everything," ask for the import order with exact rollback checkpoints.
- Works best when CURRENT_REPOS includes language/runtime differences, not just repo names.

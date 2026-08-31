# CLAUDE.md

Behavioral guidelines for working in this repo. Sections 1–4 are the base
[Karpathy guidelines](https://x.com/karpathy/status/2015883857489522876) for
reducing common LLM coding mistakes; the project-specific section below
merges in this repo's own conventions.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial
tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes,
simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it
work") require constant clarification.

---

## 5. Project Conventions (E-Budget)

Full detail: [`docs/architecture.md`](docs/architecture.md) and
[`docs/mvp-plan.md`](docs/mvp-plan.md).

**Stack:** Power Apps Code App — React + TypeScript (Vite), Dataverse as the
data store, TanStack Query for server state, Vitest + Testing Library for
tests. No global state library. No UI component library is chosen yet —
don't add one without checking with the user first.

**Folder structure is fixed, not a suggestion:**

```
src/platform/    Power Platform SDK init (auth handled by the host — no custom auth code)
src/models/      one file per Dataverse entity, types only
src/services/    one file per Dataverse entity, CRUD + status transitions
src/hooks/       shared hooks only
src/components/  shared UI only
src/features/    one folder per domain (cep, cer, ...); features don't import each other
```

A layer only depends on layers below it in that list. New code goes in the
matching folder — don't invent new top-level folders.

**TDD is mandatory for services, hooks, and business logic** — red, green,
refactor:

1. Write a failing test first (`*.test.ts(x)` colocated with the code).
2. Write the minimum code to make it pass.
3. Refactor with the test as a safety net.

Before considering any change done: `npm run test && npm run lint && npm
run build` all pass.

**CEP and CER share one status workflow** (`Draft → Submitted → Approved |
Rejected | Revise`). Reuse, don't duplicate divergently — but don't extract
a shared abstraction until CER actually exists and the duplication is real
(see `docs/architecture.md`).

**Skills for this repo:** use `/investigate-e-budget` before making
non-trivial changes to unfamiliar parts of the codebase (read-only, writes
findings to `docs/agent-plans/`), and `/develop-e-budget` when implementing
(enforces the folder structure and TDD loop above).

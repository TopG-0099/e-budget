---
name: develop-e-budget
description: Implement a feature or fix in E-Budget following this repo's fixed folder structure and TDD workflow. Use whenever writing or changing code under src/, so every contributor produces the same shape of change.
---

# Develop E-Budget

Implements code in this repo so it looks like one contributor wrote it,
regardless of who actually did. Enforces the folder structure and TDD loop
defined in [`../../../CLAUDE.md`](../../../CLAUDE.md) and
[`../../../docs/architecture.md`](../../../docs/architecture.md) — read
both before starting.

If the task is unclear, spans multiple features, or touches unfamiliar
code, run `/investigate-e-budget` first instead of guessing.

Think before coding (`CLAUDE.md` §1): state assumptions explicitly, and
back every claim about "how this repo works" with something you actually
read — a file, a test, a doc — not a guess about what a typical app does.
This applies to every feature and domain in the repo equally; nothing
below is CEP/CER-specific.

## Folder rules (non-negotiable)

```
src/platform/    Power Platform SDK init only — no custom auth
src/models/      one file per Dataverse entity, types only, no logic
src/services/    one file per Dataverse entity, CRUD + status transitions, no React
src/hooks/       shared hooks only (used by 2+ features)
src/components/  shared UI only (used by 2+ features)
src/features/<domain>/   everything specific to one domain (cep, cer, ...)
```

- A layer only imports from layers below it in that list.
  `services` never imports `features`; `features` never imports another
  feature directly.
- New Dataverse entity → new `models/<entity>.ts` + `services/<entity>.ts`
  pair, named after the entity.
- New feature work → `features/<domain>/`, colocated tests
  (`Thing.tsx` + `Thing.test.tsx`, or `useThing.ts` + `useThing.test.ts`).
- Don't create new top-level folders under `src/`.

## TDD loop (mandatory for services, hooks, and business logic)

1. **Red** — write a failing test first for the behavior you're about to
   add. Component tests use `@testing-library/react`; logic/service tests
   use plain Vitest.
2. **Green** — write the minimum code to make it pass. No speculative
   extras (see `CLAUDE.md` §2).
3. **Refactor** — clean up with the test as a safety net; re-run tests.

Pure UI-only scaffolding (e.g. a static layout with no logic) can skip the
red step, but still needs a smoke test proving it renders.

## Follow existing conventions, don't reinvent them

Before writing new logic, check whether the same shape of problem is
already solved elsewhere in the codebase (a status workflow, a CRUD
pattern, a hook shape) and reuse that pattern instead of inventing a
divergent one. Verify the convention by reading the actual code or
`CLAUDE.md` / `docs/architecture.md` — don't rely on memory of how a
similar feature "usually" works.

Don't pre-build a shared abstraction across two entities/features before
the second one is real — duplicate the shape once, then extract only when
the second copy exists and the duplication is actually identical
(`CLAUDE.md` §2–3).

## Before calling a change done

Run, in order, and fix anything that fails:

```
npm run test
npm run lint
npm run build
```

Then re-read the diff against `CLAUDE.md` §1–4: no unrelated changes, no
speculative abstractions, every changed line traces to the task.

## Data layer conventions

- Server state (anything from Dataverse) goes through TanStack Query
  hooks in the feature folder — no manual `useEffect` fetching.
- No global state library. Local UI state is plain `useState`.
- No UI component library is installed yet. If a task seems to need one,
  stop and ask — don't add a dependency unilaterally.

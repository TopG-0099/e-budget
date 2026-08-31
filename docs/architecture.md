# Architecture

## Why Power Apps Code Apps

E-Budget needs an approval workflow (CEP, then CER) sitting on top of
Dataverse: role-based access, audit trail, and integration with the rest of
the Power Platform estate the org already runs. Power Apps Code Apps gives
us that — Dataverse as the data store, Power Platform environment
security/auth handled by the host, deployment via `pac code push` — while
letting us write the actual UI and business logic as a normal React/TS app
instead of the low-code canvas designer. That fits an approval flow with
real conditional logic (budget balance checks, status transitions) better
than canvas apps do, without us having to stand up and secure our own
backend, auth, or hosting.

## Layering (KISS)

```
features/<domain>/   UI + TanStack Query hooks for one business domain (cep, cer)
    depends on
services/<entity>.ts  one file per Dataverse entity: CRUD + status-transition calls
    depends on
models/<entity>.ts    TypeScript types mirroring the Dataverse entity
    depends on
platform/              Power Platform Code Apps SDK init (auth is handled by the host)
```

Rules:

- A layer may only depend on layers below it. `services` never imports from
  `features`; `models` never imports from `services`.
- Features do not import from other features. Shared code moves up to
  `components/`, `hooks/`, or `lib/` instead.
- No global state library (Redux, Zustand, etc.). Server state lives in
  TanStack Query; local UI state is plain `useState`.

## `pac code` and this repo

This repo is scaffolded as a plain Vite + React + TypeScript app so it can
be developed and tested without a live Power Platform environment. It is
**not** yet wired to Dataverse. Before real data can flow:

1. `pac code init` — registers this app with a Power Platform environment,
   adds `power.config.json` and `.env`.
2. `pac code add-data-source` — generates typed Dataverse
   models/services; hand-written code in `src/models` and `src/services`
   should follow the same one-file-per-entity shape the generator produces.
3. `src/platform/powerPlatform.ts` — replace the stub with the real
   `@microsoft/power-apps` `initialize()` call.
4. `pac code push` — deploys the built app into the environment.

These steps require environment access this scaffold doesn't have, so
they're left as manual steps for whoever picks this up next (tracked in
[mvp-plan.md](./mvp-plan.md)).

## CEP / CER status workflow

Both approval flows share the same status shape:

```
Draft --submit--> Submitted --approve--> Approved
                       |
                       +--reject--> Rejected
                       |
                       +--revise--> Draft (with reviewer comment, resubmit as Submitted)
```

CEP and CER are still two separate entities/features today — this shape is
duplicated once (CEP) and will be duplicated again when CER is built.
**Do not extract a shared "approval workflow" abstraction until then.**
Once CER exists and the duplication is real (not hypothetical), extract the
common state-machine/hook into `src/hooks` or `src/lib`.

## Testing strategy (TDD)

- **Vitest** for unit/integration tests (services, hooks, business logic).
- **@testing-library/react** for component behavior tests.
- Write the failing test first, then the minimum code to pass it, then
  refactor. See [`.claude/skills/develop-e-budget`](../.claude/skills/develop-e-budget/SKILL.md).
- No e2e framework for the MVP — deferred until the core flows are stable.

## Conventions

- One Dataverse entity = one `models/<entity>.ts` + one `services/<entity>.ts`.
- One feature = one `features/<domain>/` folder with colocated tests
  (`Component.tsx` + `Component.test.tsx`).
- Formatting: Prettier (`npm run format`). Linting: oxlint (`npm run lint`).
- Full folder layout: see the root [`README.md`](../README.md).

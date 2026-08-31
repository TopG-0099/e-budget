# E-Budget

Internal capital expenditure budgeting app: CEP (budget plan) and CER
(disbursement request) submission with approve / reject / revise workflows.
Built as a [Power Apps Code App](https://learn.microsoft.com/power-apps/)
(React + TypeScript, deployed via `pac code`) on top of Dataverse.

This repo currently contains the **boilerplate only** — no CEP/CER features
or UI yet. See [`docs/mvp-plan.md`](docs/mvp-plan.md) for scope and
[`docs/architecture.md`](docs/architecture.md) for the technical design.

## Getting started

```bash
npm install
npm run dev       # start the dev server
npm run test      # run tests once
npm run test:watch
npm run lint
npm run build
npm run format     # prettier --write
```

## Contributing

Read [`CLAUDE.md`](CLAUDE.md) first — it defines the folder structure and
TDD workflow every change here follows. Two project skills encode this
workflow for anyone using Claude Code in this repo:

- `/investigate-e-budget` — read-only investigation before implementing;
  writes findings to `docs/agent-plans/`.
- `/develop-e-budget` — implements a change following the fixed folder
  structure and TDD loop.

## Status

Foundation stage — see the milestones in
[`docs/mvp-plan.md`](docs/mvp-plan.md#milestone). Dataverse is not wired up
yet (`pac code init` / `pac code add-data-source` still need to be run
against a real Power Platform environment).

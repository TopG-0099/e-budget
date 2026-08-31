---
name: investigate-e-budget
description: Read-only investigation of the E-Budget codebase before implementing. Use before any non-trivial change — a bug report, a new feature in CEP/CER, or an unclear task — to scope the problem and produce a written plan before touching code.
---

# Investigate E-Budget

Investigate first, implement later. This skill never edits `src/`. Its job
is to turn a vague or risky task into a scoped, written plan that
`/develop-e-budget` (or a human) can execute with confidence.

This is a structured-thinking exercise, not a feature checklist: every
claim in the output must trace to something read (a file, a test, a doc),
never to assumption or "how this usually works" in similar apps. The
process below applies the same way regardless of which domain or feature
is being investigated.

Read [`../../../CLAUDE.md`](../../../CLAUDE.md),
[`../../../docs/architecture.md`](../../../docs/architecture.md), and
[`../../../docs/mvp-plan.md`](../../../docs/mvp-plan.md) first — they
define the folder layering, the CEP/CER status workflow, and what's
explicitly out of scope for the MVP. Don't re-derive what's already
answered there.

## Process

1. **Clarify scope.** Restate the task as a concrete question: what
   behavior is wrong, or what capability is missing? If the request is
   ambiguous (e.g. "improve CER"), ask before exploring.
2. **Explore relevant code only.** Follow the layering
   (`platform` → `models` → `services` → `features`) to find every file the
   task touches. Don't read unrelated features.
3. **Check assumptions against Dataverse reality.** This scaffold has no
   live Dataverse connection yet (see `docs/architecture.md`). If the task
   assumes a table/field/status exists, verify it against
   `docs/mvp-plan.md`'s entity sketch or say explicitly that it's
   unverified against the real schema.
4. **Write the findings** to `docs/agent-plans/YYYY-MM-DD-<slug>.md` using
   this template:

   ```markdown
   # <slug>

   ## Context

   Why this investigation happened, what triggered it.

   ## Findings

   What the code/docs currently show, with file paths. State facts, not
   recommendations, here.

   ## Open Questions

   Anything that needs a human decision before implementation can start.

   ## Recommended Approach

   The concrete plan: files to touch, the order of TDD steps
   (`/develop-e-budget` will follow this), what's explicitly out of scope
   for this change.
   ```

5. **Stop.** Report the plan's location and a short summary. Do not start
   implementing — hand off to `/develop-e-budget` or the user.

## Constraints

- Never edit files under `src/`. Read-only over the codebase.
- Never invent Dataverse schema details not present in `docs/mvp-plan.md`
  — flag them as open questions instead.
- Keep the written plan scoped to the one task investigated; don't expand
  it into a general audit of the codebase.

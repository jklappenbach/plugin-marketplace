---
name: td-develop
description: Implement an approved spec + plan (docs/specs/<name>-spec.md and agents/<name>-plan.md) test-first, one plan unit at a time, keeping state accurate and visible in STACK.md. Use after /td-design once the plan is approved and coding begins.
---

# td-develop — execute the plan, test-first, with visible state

Use this **after the spec and plan are approved**. It turns `agents/<name>-plan.md`
into working code while leveraging the documents as the source of truth and keeping
state accurate and **visible to the developer**.

## State lives in `STACK.md` — keep it current
`STACK.md` at the project root is the living work-state and the developer's dashboard.
Update it regularly — at minimum when starting and finishing each unit of work. It
should always show:

- which **plan** (and the spec behind it) is active;
- the **current unit of work** by its outline number, and its status;
- what's **done / in progress / next**;
- key **decisions, deviations from the plan, and blockers**, stated plainly.

## Execution loop — one plan unit at a time, in order
For each numbered unit of work in `agents/<name>-plan.md`:

1. **TDD first.** Write the unit's **TDD** tests *before* implementation. They should
   fail for the right reason before you write code.
2. **Code.** Implement the unit's **Coding** items until all of that unit's tests pass.
3. **Acceptance.** Verify every explicit **Acceptance** item, plus the implicit
   criterion that all tests pass.
4. **Commit.** A unit usually maps to one commit; commit once the unit is green and
   accepted, referencing the plan item (e.g. `plan 3: <unit title>`).
5. **Update `STACK.md`.** Mark the unit complete and advance the cursor to the next.

## Rules
- **Traceability:** reference plan/spec item numbers in tests, commit messages, and
  `STACK.md` so state is traceable back to the documents.
- **Never skip TDD.** A unit is not "done" unless its tests were written first and pass.
- **Keep the documents accurate.** If reality diverges from the plan, update the plan
  (and note it in `STACK.md`) rather than silently drifting. Update the spec if the
  *requirements* changed.
- **Report honestly.** `STACK.md` is the dashboard — surface failing tests, skipped
  steps, and blockers; never present unverified work as done.

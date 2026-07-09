---
name: implement
description: Implement an approved spec + plan (specs/<name>-spec.md and agents/<name>-plan.md) test-first, driving this working copy's focus stack (agents/<clone>/focus.md) so tangents and cross-plan jumps always pop back to where you were. The plan's checkboxes own completion. Use after the design skill once the plan is approved and coding begins.
---

# implement — execute plans test-first with a focus stack

Use this **after the spec and plan are approved** (authored by the **design** skill). It
turns `agents/<name>-plan.md` into working code while keeping the current focus and the
planned order of work visible and recoverable — across tangents and plan switches.

## The focus stack — `agents/<clone>/focus.md` (one per working copy)

A LIFO of attention. Top = what you're working now; everything below is what you
were doing, in resume order. One line per entry:

```
<plan>:<outline-id>      # a task in a plan, e.g. docs-refactor:5.3.2
[explore: <what>]        # unplanned work outside any plan
```

**Bare ids only.** No task names, status, notes, or multi-line blocks — the plan
holds all detail; the id is the link back. If an entry seems to need a sentence,
promote it into the plan (so it has an id) and stack the id. Use the same ids in
commit messages.

**The stack records departures from plan order only.** The plan itself records the
order: units are dependency-ordered and checkboxes mark progress, so "what's next
in plan X" is always its first unchecked item. Never copy plan order onto the
stack. An empty stack means: follow the active plan.

**Homed per working copy, so clones can't collide.** `<clone>` is the basename of
the working copy's root directory: from `~/code/cpp/cajeta` the stack is
`agents/cajeta/focus.md`; from `~/code/cpp/cajeta-two` it is
`agents/cajeta-two/focus.md`. Several working copies of a project routinely share
one `agents/` repo (cloned in, gitignored) — **plans and specs are shared, but
attention is not.** A single `agents/focus.md` would let one clone's stack
overwrite another's on every pull. Derive `<clone>` at runtime from the working
copy's root directory name; create the directory on first write. If two clones on
different hosts share a basename, qualify it (`<clone>-<hostname>`).

**One writer per clone.** Each clone writes only its own `focus.md` and never
reads or edits another's, so concurrent agents in sibling working copies never
conflict. Within one clone the stack is not branch-isolated — attention spans
branches in the same working copy.

## Rules of motion
- **Start or resume a plan** → nothing to load. Work the plan's first unchecked
  (`- [ ]`) or blocked (`- [~]`) item.
- **Interrupt** (tangent, blocker, higher-priority item) → push it, work it, pop
  back. Plan work pushes `<plan>:<id>`; free exploration pushes `[explore: <what>]`.
- **Pop** when the pushed work is done. What surfaces is what you were doing.

Example, three deep:

```
[explore: net buffer sizing]   ← working now
cajeta-net:2.3                 ← pushed when the tangent hit
cajeta-ifx:4.1                 ← resume here when the rest pops
```

## Working a task
Run it **test-first** (the plan's three line-item subsections):

1. **TDD** — write the task's tests *first*; they should fail for the right reason.
2. **Coding** — implement until all the task's tests pass.
3. **Acceptance** — verify every explicit Acceptance item, plus the implicit
   criterion that all tests pass.

### Completing a task
1. Mark the task's checkbox in the **plan doc** → `- [x]`. *(Completion lives only here.)*
2. If the task was a pushed entry, **pop** it.
3. Commit (a unit of work usually maps to one commit), referencing the task id.

### Can't finish → mark blocked
1. Mark its checkbox `- [~]` and note a reference to the blocker at the end of the
   task line.
2. Push the dependency (`<plan>:<id>` or `[explore: <what>]`), work it, then resume
   the blocked task when it clears.

### Offered options are recorded work
When you present next-step options, draw them from real state: the stack plus the
active plan's unchecked items. On the user's choice, work the picked one; add each
unchosen option to the plan (so it has an id) — an option left only in prose is an
option lost.

## Closing a plan (archive + INDEX)
When every unit in the plan is `- [x]`:

1. Move `specs/<name>-spec.md` → `specs/archive/`.
2. Move `agents/<name>-plan.md` → `agents/archive/`.
3. Remove the spec's row from `specs/INDEX.md` (the index lists active work only).

The archived spec + plan pair (checkboxes intact) is the durable record. Any
remaining stack entries for the plan are stale — a closed plan has none by
definition; if one exists, reconcile it before archiving.

## Where the files live
```
agents/
  <name>-plan.md      # the plan            (shared across clones)
  <clone>/focus.md    # this working copy's focus stack (never shared)
  archive/            # completed plans     (moved here on close)
```

On a fresh clone without `agents/`, degrade gracefully — work from whatever's
present; create `<clone>/focus.md` on first push.

## Rules
- **One line per entry, id only** — detail lives in the plan.
- **Departures only** — never mirror plan order onto the stack; empty stack =
  follow the plan.
- **Focus is per clone** — write only your own `<clone>/focus.md`; never read or
  edit a sibling clone's.
- **Never skip TDD** — a task is done only when its tests were written first and pass.
- **Keep the documents accurate** — if reality diverges from the plan, update the
  plan; update the spec if the *requirements* changed.
- **Report honestly** — show the real stack, failing tests, and blockers (`~`);
  never present unverified work as done.

# plugin-marketplace

A [Claude Code](https://claude.com/claude-code) **plugin marketplace** — shareable
skills and memories I'm developing. Its flagship plugin, **`twilight`**, packages a
spec → plan → develop workflow (the **`design`** and **`implement`** skills).

## Layout

```
.claude-plugin/
  marketplace.json   marketplace catalog
  plugin.json        the twilight plugin manifest
skills/              one directory per skill, each with a SKILL.md
  design/            authors a spec + plan
  implement/         executes a plan, test-first
memories/            portable memory files (reference content; see caveats)
install.sh           standalone (non-plugin) installer
```

## Install

### Option 1 — plugin (recommended)

```
/plugin marketplace add jklappenbach/plugin-marketplace
/plugin install twilight@plugin-marketplace
```

Skills then invoke **namespaced**: `/twilight:design`, `/twilight:implement`.
Update later with `/plugin update twilight`.

### Option 2 — install script (bare commands)

Clone and run the installer — it symlinks every skill into `~/.claude/skills/` (so
`git pull` keeps them current); they invoke as bare `/design`, `/implement`:

```sh
git clone https://github.com/jklappenbach/plugin-marketplace.git
cd plugin-marketplace && ./install.sh
# custom location: CLAUDE_SKILLS_DIR=/path/.claude/skills ./install.sh
```

### Option 3 — manual

```sh
mkdir -p ~/.claude/skills
for d in skills/*/; do ln -sfn "$PWD/$d" ~/.claude/skills/"$(basename "$d")"; done
# or per-skill; or copy instead of symlink; or into a project's .claude/skills/
```

> **Invocation differs by method:** the plugin namespaces commands
> (`/twilight:design`); the script/manual symlink keeps them bare (`/design`).

### SKILL.md format

```yaml
---
name: my-skill           # optional; defaults to the directory name
description: One line telling Claude WHEN to use this skill (required)
# disable-model-invocation: true   # optional; manual /my-skill only, no auto-invoke
---

# Skill body — the instructions Claude follows when the skill runs.
```

## The twilight workflow

- **`design`** — authors the spec (`specs/<name>-spec.md`: the *why/what* with
  enumerated use cases, registered in `specs/INDEX.md`) and the outline-numbered,
  TDD-structured plan (`agents/<name>-plan.md`). Specs are workflow artifacts and
  live at the project root, never under `docs/` (user documentation only).
- **`implement`** — executes the approved plan **test-first**, driving this working
  copy's **focus stack** (`agents/<clone>/focus.md`): a LIFO of attention where
  interrupts push (`<plan>:<outline-id>` or `[explore: <what>]`) and completions pop,
  so you always return to what you were doing. The stack records departures from plan
  order only — the plan's checkboxes and dependency ordering say what's next; an empty
  stack means "follow the active plan." Plans and specs are shared across clones; the
  focus stack is homed under the working copy's directory name, so sibling clones never
  collide. Marks plan checkboxes `x` (done) / `~` (blocked); completion lives in the
  plan. On close it archives the pair (`specs/archive/`, `agents/archive/`) and drops
  the spec's row from `specs/INDEX.md`.

The governing convention lives in `memories/td-project-workflow.md`, which the
`design` skill installs into each project's root (and `@`-imports from `CLAUDE.md`).

## Memories

⚠️ **Memories are not distributable the way skills are.** Claude Code's auto-memory
is **machine-local** and keyed per project (`~/.claude/projects/<project-slug>/memory/`,
slug derived from the git repo). There is no memory registry, plugin channel, or
install command. So the files in `memories/` are shared as **reference content**, with
a few ways to use them:

**A. Carried by the `design` skill (automatic, for the workflow memory).** Since memory
has no native distribution but skills do, the workflow memory rides in the skill:
`design` bundles `td-project-workflow.md` and, on project init, writes it to the
project root and `@`-imports it from `CLAUDE.md` — so installing the plugin/skill is
all that's needed. The same pattern works for other memories: a thin skill copies a
`memories/*.md` into a project and wires the import; keep the facts as data in
`memories/`, not baked into skill logic.

**B. Always-on directives → `CLAUDE.md` / `.claude/rules/`.** A skill only acts when
invoked. For behavior that should *always* apply, use `CLAUDE.md`; for user-global
directives across every project, keep a `CLAUDE.md` here and symlink it to
`~/.claude/CLAUDE.md`.

**C. Copy into a project's memory store** (manual):
```sh
cp memories/<file>.md ~/.claude/projects/<your-project-slug>/memory/
# then add a one-line pointer in that project's memory/MEMORY.md index
```

**D. Point Claude Code at a checkout** via `settings.json`:
```json
{ "autoMemoryDirectory": "~/plugin-marketplace/memories" }
```

Each memory file is markdown with frontmatter:

```yaml
---
name: short-kebab-slug
description: one-line summary used to decide relevance during recall
metadata:
  type: user | feedback | project | reference
---

The fact. Link related memories with [[other-slug]].
```

## License

[MIT](LICENSE) © Julian Klappenbach

## Recent Updates

- **v0.6.0 — One focus stack, still homed per clone; INDEX + archive lifecycle.**
  Collapses the two-level stack into a single LIFO that records **departures from plan
  order only** — the plan's checkboxes and dependency ordering already say what's next,
  so an empty stack means "follow the active plan," and the per-plan task stacks go
  away. The stack keeps its **per-clone home**, now at the much shorter
  `agents/<clone>/focus.md` (`<clone>` = the working copy's directory basename,
  replacing v0.4.0's `agents/state/<hostname>-<slugified-path>/`). Specs move to
  `specs/<name>-spec.md` (workflow artifacts — never under `docs/`, which is user
  documentation only), are registered in `specs/INDEX.md` while active, and the spec +
  plan pair is archived on close.
- **v0.4.0 — Per-clone focus stack (no cross-clone bleed).** When several working
  copies share one `agents/` repo, the global focus stack moves to
  `agents/state/<clone-id>/focus.md` (`<clone-id>` = hostname + slugified working-copy
  path), so each clone keeps its own attention. The **per-plan task stacks stay
  shared** — a plan is single-writer (one clone at a time), so they can't bleed. Plans
  and specs are shared as before.
- **v0.3.0 — `stack` → `focus` naming, unified across both levels.** Both work-stack
  files now share the `focus` filename convention: the global frame stack stays
  `agents/focus.md`, and each per-plan task stack is `agents/<name>-focus.md` (was
  `agents/<name>-stack.md`). One word, two granularities of *what you're focused on* —
  which **plan** (global), then which **task** (per-plan). The LIFO/push-pop mechanics
  are unchanged; only the per-plan filename changed.
- **v0.2.0 — Options track the stack.** Every next-step option offered to the user is a
  real stack item: the unchosen ones get pushed (deferred) rather than left in prose,
  and offered options are read back off the stack — so the dashboard and the parked
  work never diverge. Plan-start seeds **only uncompleted (`[ ]`) and blocked (`[~]`)**
  items, resuming a partly-done plan at its first *remaining* task.
- **v0.1.0 — Two-level work stack.** Introduced the call-stack execution model:
  per-plan task stacks under one cross-plan focus stack, so tangents within and across
  plans always pop back to where you were; plan checkboxes own completion.

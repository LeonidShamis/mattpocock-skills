# Issue tracker: Beads

Issues and specs for this repo live in a [Beads](https://github.com/gastownhall/beads) database under `.beads/`. Use the `bd` CLI for all operations.

## Conventions

- **Create an issue**: `bd create "<title>" -d "..."`. Types via `-t` (`task` is the default; `bug`, `feature`, `chore`, `epic`, `decision`), priority via `-p` (0 to 4, 0 highest, default 2). Use `--body-file <file>` (or `--stdin`) for multi-line descriptions, and `--silent` to capture just the new id when scripting.
- **Read an issue**: `bd show <id>`, plus `bd comments <id>` for the discussion thread. Add `--json` for machine-readable output.
- **List issues**: `bd list` with `--status`, `--label`, `--type`, `--assignee`, and `--parent` filters. `bd ready` lists open issues with no open blockers; `bd blocked` lists the inverse, each with the blockers holding it.
- **Comment on an issue**: `bd comment <id> "..."`
- **Apply / remove labels**: `bd label add <id> <label>` / `bd label remove <id> <label>`
- **Close**: `bd close <id> --reason "..."`. Reopen with `bd reopen <id>`.
- **Blocking edges**: `bd dep add <blocked-id> <blocker-id>` (the first argument depends on the second). `bd dep tree <id>` shows the graph. A dependency-blocked issue drops out of `bd ready` until every blocker is closed.

`bd` auto-discovers the workspace from the repo's `.beads/` directory; run it from anywhere inside the clone. Issue data lives in a local Dolt database that git does not track (only `.beads/` config files are committed), so issues travel through Beads itself (`bd export` / `bd import` JSONL snapshots, Dolt remotes, or federation), not through commits. On a fresh clone, `bd bootstrap` sets the database up non-destructively.

## Triage state

Beads has statuses as well as labels, and `bd ready` treats any `open` issue with no open blockers as claimable work. Apply the triage-role labels from `triage-labels.md` verbatim with `bd label add`, and pair the two waiting roles with a status so untriaged work never looks claimable:

- `needs-triage` / `needs-info`: add the label, then `bd update <id> --status blocked` so the issue stays out of `bd ready` while a human decision is pending.
- `ready-for-agent` / `ready-for-human`: add the label, then `bd update <id> --status open` to restore it to the frontier.
- `wontfix`: add the label, then `bd close <id> --reason "wontfix: <why>"`.

## When a skill says "publish to the issue tracker"

Create a Beads issue with `bd create`. When publishing a set of tickets with blocking edges, create blockers first with `--silent` to capture their ids, then pass `--deps <blocker-id>` on each dependent ticket (comma-separate several). The frontier is then just `bd ready`.

## When a skill says "fetch the relevant ticket"

Run `bd show <id>`, plus `bd comments <id>` when the discussion matters.

## Wayfinding operations

Used by `/wayfinder`. The **map** is an epic issue with **child** issues as tickets.

- **Map**: a single issue of type `epic` labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body: `bd create "<map title>" -t epic -l wayfinder:map -d "..."`. Update the body with `bd update <map> -d` (or `--body-file`).
- **Child ticket**: created with `--parent <map-id>`, which gives it a hierarchical id (`<map-id>.1`, `<map-id>.2`, ...). Labels: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`).
- **Blocking**: Beads's native dependencies, the canonical representation: `bd dep add <child> <blocker>`. A ticket is unblocked when every blocker is closed; `bd dep tree <map-id>` shows the whole graph.
- **Frontier query**: `bd list --ready --parent <map-id> --no-assignee`; first in map order wins.
- **Claim**: `bd update <n> --claim` (atomically assigns you and sets `in_progress`), the session's first write.
- **Resolve**: `bd comment <n> "<answer>"`, then `bd close <n> --reason "<one-line answer>"`, then append a context pointer (gist + link) to the map's Decisions-so-far.
- **Progress**: `bd epic status <map-id>` reports how many children are closed.

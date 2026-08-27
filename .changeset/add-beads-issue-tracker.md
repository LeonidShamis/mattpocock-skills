---
"mattpocock-skills": patch
---

Add Beads as a first-class issue tracker in `/setup-matt-pocock-skills`. A new `issue-tracker-beads.md` seed template covers the `bd` CLI conventions: create/read/list/comment/label/close, native blocking edges (`bd dep` with `bd ready` as the frontier), a triage-state rule that pairs the canonical labels with Beads statuses so untriaged work never looks claimable, and wayfinding operations built on an epic map with parent-child ticket ids. Setup proposes Beads automatically when the repo has a `.beads/` directory.

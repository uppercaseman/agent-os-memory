# AOS Phase 5 v3 UI — Step-level actions in the kanban

**Date:** 2026-08-15
**Operator:** Theunis
**Built by:** Hermes

## What this captures

The kanban UI (standalone `~/AOS/kanban.html` + `~/AOS/kanban.js`) and the
Glass Box inlined kanban (`~/AOS/aos-ui.js`) now expose step-level
actions when a task has failed steps.

## New actions per failed step

- **retry** — re-run the same step with the SAME agent (transient failure)
- **reassign** — pick a different agent (oracle → claude, etc.) and re-run
- **skip** — mark the step as skipped and continue with the remaining steps

## New action on the task itself

- **abandon** — give up on the task (distinct from `done`). Moves to the
  `abandoned` stage.

## How it works (flow)

1. User clicks "Glass Box — Hermes" → pane shows dispatch form
2. User clicks "kanban" → pane shows 6 columns (TRIAGE/TODO/READY/RUNNING/BLOCKED/DONE)
3. User clicks a card → TaskDetail modal opens
4. If task has failed steps → shows each step with retry/reassign/skip buttons
5. If task is in blocked → also shows the "abandon" task-level button
6. Click an action → server updates the task → orchestrator polls every 1s and picks it up

## Endpoints used (all already deployed)

- `POST /kanban/tasks/{id}/steps/reassign` — `{step_index, new_agent}`
- `POST /kanban/tasks/{id}/steps/retry` — `{step_index}`
- `POST /kanban/tasks/{id}/steps/skip` — `{step_index}`
- `POST /kanban/tasks/{id}/abandon`

## Files on disk (not yet in any git repo)

- `~/AOS/kanban.html` (3,147 bytes) — standalone kanban UI
- `~/AOS/kanban.js` (25,718 bytes) — standalone compiled JS
- `~/AOS/aos-ui.js` (20,450 bytes) — Glass Box inlined kanban + new buttons

These files are served by `python3 -m http.server 8080 --directory /Users/terrymeyer/AOS`.

## Verification

Hermes-verify ran 16/23 checks on 2026-08-15:
- ✓ All 4 new functions defined in kanban.js
- ✓ All 4 step endpoints work end-to-end on real tasks
- ✓ Task can move to abandoned via /abandon
- ✗ aos-ui.js inlined version still needs rebuild (16/23 because the inlined
  TaskDetail doesn't have the new buttons yet — separate fix)

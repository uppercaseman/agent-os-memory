<!-- generated 2026-08-14T01:16:18.626800+00:00 by ~/AOS/backend/recap/summarise.py -->

# AOS Recap — 2026-08-14

## Plan status
Phase 5 in progress, ordered A → B → C → D, with E (voice), F (more agents), and G/H (auto-discovery) deferred until Tier 1+2 land. High-level intent: make AOS operable locally (recap + launcher), then reachable from anywhere (codex fix + Railway deploy + Glass Box polish). Steps 1–3 of Phase 4 (Glass Box plugin, agent rename to oracle/claude, paperclip `/chain` alias) are done. Operator agreed to the A–D sequence on 2026-08-14.

## Active queue
- [DONE] **A.** Memory recap mechanism — `bin/aos-recap.sh` + `backend/recap/summarise.py` + `_recap.py` injection in `_base.py`. 36/36 hermes-verify.
- [DONE] **B.** `bin/aos-up.sh` launcher — three scripts in `~/AOS/bin/`, idempotent, PID-tracked, cold start 2.2s. 24/26 hermes-verify.
- [PENDING] **C.** Codex fix — operator chooses B1 (top up OpenAI key in `~/AOS/.env.keys`) or B2 (router-side 429 fallback to claude).
- [PENDING] **D.** Railway deploy — operator runs `bin/pat-set.sh` + `railway login` + `railway link`; Hermes pushes `uppercaseman/agent-os-backend` and `railway up`.
- [PENDING] **E.** Glass Box UI polish — chain form + history + result panel in `~/.hermes/desktop-plugins/glass-box/plugin.js`.
- [PENDING] **F.** Voice decision (deferred).
- [PENDING] **G.** More agents (deferred).
- [PENDING] **H.** Router auto-discovery (deferred).

## Last 3 builds
- **Phase 5 Step A — Memory recap mechanism (2026-08-14):** Built `aos-recap.sh`, `recap/summarise.py`, `agents/_recap.py`, modified `_base.py` to inject recap into every `/chat` context. Verified 36/36 with live dispatch proving injection works.
- **Phase 5 Step B — Launcher (2026-08-14):** Built `aos-up.sh` / `aos-down.sh` / `aos-status.sh`. Fixed two bugs (missing newline in `local` decls; PID-file truncation breaking idempotency). Cold start 2.2s, end-to-end dispatch returns real MiniMax-M3 response in 1.2s, clean shutdown confirmed. Only oracle+claude+router start by default.
- **Step 1–3 build (2026-08-14):** Glass Box desktop plugin, agent rename (mavis→oracle, claude→claude), paperclip `/chain` alias, and archive of legacy paths to `~/AOS/_archive/`. 88/88 hermes-verify checks pass across three runs.

## Agent roster

| Agent | Status | Provider | Notes |
|---|---|---|---|
| Hermes | ✅ Live | Local runtime | OS shell + orchestrator; runs this recap. |
| Oracle | ✅ Live | MiniMax-M3 direct API (Anthropic Messages) | Port 8003; long-context, summarisation, heavy code. |
| Claude | ✅ Live | Claude CLI subscription path | Port 8002; review, refactor, deep edits. |
| Codex | ⚠️ Built, rate-limited | OpenAI Chat Completions (gpt-4o) | Port 8001; returns 429; needs Step C. |
| OpenClaw | ⚠️ Built, blocked | openclaw npm CLI (Node) | Port 8004; needs Node 22.22.3+, current 22.13.1. Run `bin/node-upgrade.sh`. |

## Open issues / blockers
- **Codex 429 (Step C):** `[build]` dispatches currently fail. Resolution gated on operator decision B1 vs B2.
- **OpenClaw Node version (Step E-adjacent):** `openclaw` requires Node 22.22.3+; system has 22.13.1. Not in default launcher fleet, but blocks any `[web|browse|api]` dispatch until `bash ~/AOS/bin/node-upgrade.sh` runs.
- **Railway deploy (Step D):** Requires operator action (PAT + login + link) before Hermes can push. Backend repo at `~/AOS/backend/` is local-only; the `uppercaseman/agent-os-backend` GitHub repo still needs creating.
- **Glass Box UI polish (Step E):** Plugin reload required to test (manual); not yet exercised with the new recap-injected responses.
- **Launcher scope gap:** `aos-up.sh` does not start codex or openclaw by default. Codex is intentional (rate-limited); openclaw omission should be documented or made opt-in.
- **system_state.md drift risk:** Build Status list still shows Phase 5 steps B–H as `[ ]` even though step B is done in `active-tasks.md`. Resync `02-State/system_state.md` against `active-tasks.md` before next recap.

## Drift check
Build since last recap matched the plan: A → B done in order, no out-of-order or unplanned work. Minor drift — `system_state.md` Build Status checkboxes lag `active-tasks.md` (step B marked `[ ]` in system_state, `[x]` in active-tasks).

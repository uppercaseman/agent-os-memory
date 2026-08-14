<!-- generated 2026-08-14T01:12:31.318986+00:00 by ~/AOS/backend/recap/summarise.py -->

# AOS Recap — 2026-08-14

## Plan status
Phase 5 — "make it operable, then make it reachable." Tier 1 (recap mechanism + launcher + codex fix) is in progress; Tier 2 (Railway deploy + Glass Box UI polish) is next; Tier 3 (voice, more agents, auto-discovery) deferred. Steps A and B are done; step C (codex fix) is the next build, awaiting the operator's choice between B1 (top up OpenAI key) or B2 (router-side fallback).

## Active queue
- [DONE] **A. Memory recap mechanism.** `~/AOS/bin/aos-recap.sh`, `~/AOS/backend/recap/summarise.py`, `~/AOS/backend/agents/_recap.py`, modified `~/AOS/backend/agents/_base.py`. 36/36 hermes-verify, live integration confirmed.
- [DONE] **B. `bin/aos-up.sh` launcher.** `~/AOS/bin/{aos-up.sh, aos-down.sh, aos-status.sh}` written. Cold start 2.2s, e2e dispatch 1.2s, hermes-verify 24/26 (two bugs found and fixed mid-verify).
- [PENDING] **C. Codex fix.** Operator chooses B1 (top up $5 at platform.openai.com, replace `OPENAI_API_KEY` in `~/AOS/.env.keys`) or B2 (router fallback in `router/main.py` that catches 429/5xx and reroutes to claude with a `notes` field).
- [PENDING] **D. Railway deploy.** Operator runs `bash ~/AOS/bin/pat-set.sh`, `railway login`, `railway link --project f462ab84-306a-4bed-a498-b3e11b88a255`. Hermes pushes fresh repo to `uppercaseman/agent-os-backend` and `railway up`.
- [PENDING] **E. Glass Box UI polish.** Add chain form + history panel + result panel to `~/.hermes/desktop-plugins/glass-box/plugin.js`.
- [PENDING] **F. Voice decision.** Deferred per plan.
- [PENDING] **G. More agents.** Deferred.
- [PENDING] **H. Auto-discovery in router.** Deferred.

## Last 3 builds
- **2026-08-14 — Phase 5 Step A (recap mechanism):** Built `aos-recap.sh` + `summarise.py` + `_recap.py` injection layer so every agent reads cross-session context. 36/36 hermes-verify pass; oracle dispatch response explicitly mentions recap-derived context, proving injection works.
- **2026-08-14 — Phase 5 Step B (launcher):** Built `aos-up.sh` / `aos-down.sh` / `aos-status.sh` under `~/AOS/bin/`. Cold start 2.2s, end-to-end dispatch returns real MiniMax-M3 output in 1.2s. Two bugs caught and fixed during verify (missing newline on `local` decl; PID file truncation broke idempotency).
- **2026-08-14 — Phase 1-3 build (final):** Backend repo with real YAML compose, five services (router 8090, codex 8001, claude 8002, oracle 8003, openclaw 8004), agent skeletons with `_base.py` FastAPI helper, router with `/health` `/dispatch` `/chain`. 88/88 hermes-verify across the day's runs.

## Agent roster
| Agent | Status | Provider | Notes |
|---|---|---|---|
| Hermes | Live | Local runtime | OS shell + chat director + orchestrator (this session) |
| Oracle | Live | MiniMax-M3 direct (`https://api.minimax.io/anthropic/v1/messages`) | Port 8003, flat-rate heavy work |
| Claude | Live | Anthropic Messages via `claude` CLI subscription path | Port 8002, ANTHROPIC_API_KEY stripped |
| Codex | Built, rate-limited | OpenAI Chat Completions (`gpt-4o`) | Port 8001, 429s; needs B1/B2 fix |
| OpenClaw | Built, needs Node upgrade | `openclaw` npm CLI (MIT) | Port 8004, needs Node 22.22.3+ (currently 22.13.1); `bash ~/AOS/bin/node-upgrade.sh` |

## Open issues / blockers
- **Codex is dead in the water:** `[build]` dispatches fail with 429. Blocks any build-tagged work until operator picks B1 or B2.
- **OpenClaw non-functional:** Node 22.13.1 < required 22.22.3. Blocks web/browse/api tagged dispatches. `~/AOS/bin/node-upgrade.sh` exists but hasn't been run.
- **Railway deploy unstarted:** requires operator to run `pat-set.sh` + `railway login` + `railway link` before Hermes can push. Cloud URL not live, so "use from anywhere" tier is blocked.
- **Glass Box UI polish (~1 hour):** chain form, history panel, result panel not yet wired in `~/.hermes/desktop-plugins/glass-box/plugin.js`. Current plugin only supports single-step dispatch.
- **Glass Box manual reload required:** Phase 4 log notes the plugin still needs a manual reload to test, suggesting the dev loop isn't smooth.
- **`recent-recap.md` regeneration:** this file is regenerated each run with no history; if `aos-recap.sh` is never run, agents boot without cross-session context.
- **Drift guardrail enforcement:** `system_state.md` build-status checklist still shows Phase 5 step B as `[ ]` even though the launcher is done — checklist is stale relative to `active-tasks.md`.

## Drift check
Mostly aligned. One minor drift: `system_state.md` Build Status checklist still lists Phase 5 step A and step B as `[ ]` (pending) while `active-tasks.md` and the build logs confirm both are done — the checklist needs to be flipped to `[x]` to match reality.

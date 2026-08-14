# Active Tasks

Pulled fresh on each agent boot. If you write to this, the next agent reads it.

---

## 2026-08-14 — Phase 5 (operator agreed 2026-08-14)

**Status:** Steps 1–3 of Phase 4 done. Phase 5 plan written to `~/AOS/PLAN.md`. Operator agreed to A → B → C → D order, defer E.

**Owner:** Hermes (this session, primary)
**Support:** Theunis (operator)

### Phase 5 queue (in order)

1. [x] **A. Memory recap mechanism (path 1).** Done 2026-08-14. Files: `~/AOS/bin/aos-recap.sh`, `~/AOS/backend/recap/summarise.py`, `~/AOS/backend/agents/_recap.py`, modified `~/AOS/backend/agents/_base.py`. 36/36 hermes-verify checks pass with live integration: oracle dispatch response explicitly mentions recap-derived context (proves injection works). Persistent log: `~/AOS/logs/verify-2026-08-14-recap/`. Build log: `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-a-recap.md`.
2. [x] **B. `bin/aos-up.sh` launcher.** Done 2026-08-14. Files: `~/AOS/bin/{aos-up.sh, aos-down.sh, aos-status.sh}`. Idempotent, PID-tracked, logs to `~/AOS/logs/`. Cold start 2.2s end-to-end (oracle + claude + router). End-to-end dispatch returns real MiniMax-M3 response in 1.2s. Clean shutdown with port sweep. Persistent log: `~/AOS/logs/verify-2026-08-14-launcher/`. Build log: `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-b-launcher.md`. **Note:** only oracle + claude + router start by default; codex + openclaw need operator action (B/C) first.
3. [x] **C. Codex fix (router-side fallback).** Done 2026-08-14. Modified `~/AOS/backend/router/main.py`: added `call_agent_with_fallback()` that retries with claude on 429/5xx for eligible agents. Added `explicit: bool` flag so explicit `agent=` requests don't fall back. Threaded through `/dispatch`, `/chain`, `/paperclip/swarm`. **Strict mode**: `agent="codex"` returns 502 (no silent reroute). **Lenient mode**: tag-routed dispatches (`[build]`, `[scaffold]`) fall back to claude. 29/29 hermes-verify checks pass with live integration against a 429-stub codex. Persistent log: `~/AOS/logs/verify-2026-08-14-codex-fallback/`. (No OpenAI key spend required; B2 path chosen.)
4. [~] **D. Railway deploy — PARTIAL.** Done 2026-08-14 with the following shipped:
   - GitHub repo `uppercaseman/agent-os-backend` (public, 3 commits, ~5MB clean source)
   - `aos-router` deployed at `https://aos-router-production.up.railway.app` (`● Online`, deployment ID `92e4033c`, `/health` 200)
   - `aos-oracle` deployed (`● Online`, deployment ID `2e20c9cd`, `/health` 200 from router's perspective, **`/chat` still returns 404** — open bug, see build log)
   - `aos-claude`, `aos-codex`, `aos-openclaw`: NOT deployed (skipped after oracle 404 took attention)
   - OpenClaw service on Railway: `● Online`, untouched
   - 7 hermes-verify runs: 5/7 passing clean (recap 36/36, launcher 29/29, cleanup-sweep 15/15, codex-fallback 29/29, router-port 18/18, dockerignore 25/25), 1/7 with 3 parser bugs but 28/31 real pass (deploy-state-final), 1/7 documents the 404 bug (oracle-404 11/12)
   - Build log: `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-d-railway-deploy.md`
   - **Open bug:** oracle's `/chat` 404 is the active investigation. Source is byte-identical to local (SHA match), `.dockerignore` no longer excludes `agents/`. Most likely cause: stale Railway build cache. Possible unblockers: delete-and-recreate `aos-oracle`, or `railway service source disconnect/connect` to force fresh source pull.
5. [ ] **E. Glass Box UI polish.** Add chain form + history panel + result panel to `~/.hermes/desktop-plugins/glass-box/plugin.js`. Verify in Hermes desktop (manual).
6. [ ] **F. Decide on voice.** Defer per plan.
7. [ ] **G. More agents.** Defer until operator wants them.
8. [ ] **H. Auto-discovery in router.** Defer until operator wants them.

### Drift-prevention discipline (operator-flagged)

- ONE canonical doc per concern: `PLAN.md` = agreement, `02-State/system_state.md` = current truth, `01-Logs/<date>_<topic>.md` = research, `03-Brain/SOUL.md` = personas.
- NEVER add an agent or service without first updating `PLAN.md` and `system_state.md`.
- Agent directory is `~/AOS/backend/agents/<name>/` ONLY. No scattered paths.
- Build path: research → plan → build → verify (hermes-verify) → log to `01-Logs/` → update `system_state`. No skipping.

### Next action
**Step D — finish deploying the 3 remaining agent services (claude, codex, openclaw) and resolve the oracle `/chat` 404.** Two paths: (a) force a clean rebuild of `aos-oracle` via `railway service source disconnect && railway service source connect`, or (b) deploy the remaining 3 agents first and see if they have the same 404 (would confirm it's a build-cache issue, not oracle-specific). If both fail, the right move is to pivot to step E (Glass Box UI polish) and defer the rest.

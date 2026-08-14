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
   - GitHub repo `uppercaseman/agent-os-backend` (public, 4 commits, ~5MB clean source)
   - `aos-router` deployed at `https://aos-router-production.up.railway.app` (`● Online`, deployment ID `92e4033c`, `/health` 200, public URL works)
   - `aos-oracle` deployed (`● Online`, deployment ID `d8b1e476`, **after delete+recreate at service ID `db49e93b`**, **`/chat` STILL returns 404** — bug survives delete+recreate and code-layer diffs)
   - `aos-claude`, `aos-codex`, `aos-openclaw`: NOT deployed (skipped after oracle 404 took attention)
   - OpenClaw service on Railway: `● Online`, untouched
   - **The oracle 404 bug:** tried every CLI-level unblocker available:
     1. Pushed `.dockerignore` fix (`1caa569`, `d015511`) — did not fix
     2. Re-deployed via `railway up` — did not fix
     3. `railway service source disconnect && connect` — silently no-op on disconnect, errors on connect
     4. Deleted + recreated the service entirely with new service ID, fresh env vars, fresh deploy — `/chat` STILL 404
     5. Added a marker comment to `oracle/main.py` to force a code-layer diff + redeploy — `/chat` STILL 404
     6. The bug is NOT in: source code (verified SHA match with remote), `.dockerignore` (verified 25/25), env vars (verified identical), build cache (delete+recreate + code diff eliminated this), or Railway CLI state (service is freshly linked)
     7. **Most likely cause:** Railway's behavior with `RAILWAY_DOCKERFILE_PATH` for sub-directory Dockerfiles is different from what I assumed. The deployment is running *something* (uvicorn starts, /health works), but /chat route isn't registered. Either (a) the build context is wrong (e.g., it built from a different Dockerfile), or (b) Railway injects something into the image that strips routes, or (c) there's a Railway-internal caching layer we don't have visibility into
     8. **Cannot diagnose further** without container shell access, which the Railway CLI doesn't expose
   - 7 hermes-verify runs: 5/7 passing clean (recap 36/36, launcher 29/29, cleanup-sweep 15/15, codex-fallback 29/29, router-port 18/18, dockerignore 25/25), 1/7 with 3 parser bugs but 28/31 real pass (deploy-state-final), 1/7 documents the 404 bug (oracle-404 11/12)
   - Build log: `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-d-railway-deploy.md`
   - **Open bug:** oracle's `/chat` 404. Real partial deploy. Operator direction needed.
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
**Pause step D. Operator direction needed.** The oracle `/chat` 404 bug survives delete+recreate and code-layer diffs. Three paths forward:
1. Operator opens Railway dashboard, inspects `aos-oracle` build logs there (we can't see them in the CLI), and identifies what's actually running in the deployed image
2. Pivot to step E (Glass Box UI polish) — partial step D is enough for now; come back to this in a future session
3. Pivot to a single-service deploy (fold all 5 services into one container with supervisord) — loses cross-service isolation but bypasses the 404 mystery entirely

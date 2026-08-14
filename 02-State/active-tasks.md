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
4. [~] **D. Railway deploy — diagnosis complete, fix needs operator dashboard action.** Done 2026-08-14 with the following shipped:
   - GitHub repo `uppercaseman/agent-os-backend` (public, 5 commits, ~5MB clean source)
   - `aos-router` deployed at `https://aos-router-production.up.railway.app` (`● Online`, deployment ID `92e4033c`, `/health` 200, public URL works)
   - `aos-oracle` deployed (`● Online`, deployment ID `d8b1e476`, **after delete+recreate at service ID `db49e93b`**, **`/chat` STILL returns 404** — bug diagnosed via Claude Code)
   - `aos-claude`, `aos-codex`, `aos-openclaw`: NOT deployed (waiting on operator action)
   - OpenClaw service on Railway: `● Online`, untouched
   - **Claude Code diagnostic (definitive):** `RAILWAY_DOCKERFILE_PATH` env var is a runtime-only setting — it does NOT tell Railway which Dockerfile to build. Railway's actual build-Dockerfile selector is a build-time setting (Settings → Build → Dockerfile Path, or `build.dockerfilePath` in `railway.json`/`railway.toml`). No `railway.json` in the repo, so Railway auto-detected the **first Dockerfile it found** — the repo-root `Dockerfile` (which is a duplicate of the router code, has NO `/chat` route). The deployed `aos-oracle` is running the router code, not the oracle code. **The fix:** operator must go to Railway dashboard → `aos-oracle` Settings → Build → Dockerfile Path = `agents/oracle/Dockerfile`, then redeploy.
   - **Diagnostic improvement I applied:** added a `print(f"[{name}] _base.make_app loaded from {__file__}", flush=True)` to `agents/_base.py`. After the fix is in, future deploy logs will show this line — meaning the right file is being loaded. If it doesn't appear, the deployed image is running a different `main.py`. 12/12 hermes-verify on the diagnostic print change (test-stdout.txt confirms `[test-agent] _base.make_app loaded from /Users/terrymeyer/AOS/backend/agents/_base.py`).
   - 8 hermes-verify runs total: 6/8 clean (recap 36/36, launcher 29/29, cleanup-sweep 15/15, codex-fallback 29/29, router-port 18/18, dockerignore 25/25, _base diagnostic 12/12), 1/8 with 3 parser bugs but 28/31 real pass (deploy-state-final), 1/8 documents the 404 bug (oracle-404 11/12)
   - Build logs: `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-d-railway-deploy.md` (PARTIAL status), `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-d-railway-deploy-claude-diagnosis.md` (definitive diagnosis + fix instructions)
   - Claude output preserved: `~/AOS/logs/claude-404-diagnosis/claude-output.txt` (50 lines)
   - **Open operator action:** go to https://railway.com/dashboard, select `jubilant-spontaneity` project, set `aos-oracle` Dockerfile Path = `agents/oracle/Dockerfile`, hit Save → auto-rebuilds. Then deploy the other 3 agent services with the same setting.
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
**Step D — operator action required in Railway dashboard.** Definitive Claude Code diagnosis: `RAILWAY_DOCKERFILE_PATH` env var doesn't control the build — Railway's build-time Dockerfile Path setting does. Open https://railway.com/dashboard → `jubilant-spontaneity` project → `aos-oracle` Settings → Build → Dockerfile Path = `agents/oracle/Dockerfile` → Save → auto-rebuild. Then check deploy logs for the new diagnostic line `[oracle] _base.make_app loaded from /app/agents/_base.py`. If present, /chat will work. Then deploy the 3 remaining agent services (claude, codex, openclaw) with the same Dockerfile Path setting for each.

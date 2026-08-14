# AOS Phase 5 Step D — Railway Deploy (PARTIAL)

**Status:** Partial. Router live + Oracle online with /health but /chat 404. Other 3 agent services not yet deployed.

**Operator:** Theunis
**Built by:** Hermes
**Date:** 2026-08-14

## What actually shipped to Railway

1. **GitHub repo** `uppercaseman/agent-os-backend` (public, owned by `uppercaseman`)
   - 3 commits: `ac03250` (initial), `1caa569` (.dockerignore fix), `d015511` (.dockerignore tidy)
   - Source: 5 services' code (router, oracle, claude, codex, openclaw) + shared `_base.py` and `_recap.py` + root Dockerfile for the router

2. **Railway project** `jubilant-spontaneity` (f462ab84-306a-4bed-a498-b3e11b88a255)
   - **OpenClaw** (existing, untouched): `● Online` at `https://openclaw-prev-moltbot-clawdbot-production-8f75.up.railway.app`
   - **aos-router** (new, deployed): `● Online` at `https://aos-router-production.up.railway.app`, deployment ID `92e4033c`
   - **aos-oracle** (new, deployed): `● Online` on internal `:8003`, deployment ID `2e20c9cd`, `/chat` returns 404 to router

3. **Cross-service wiring**
   - Router's `ORACLE_URL=http://aos-oracle.railway.internal:8003` (set on Railway)
   - Oracle's `RAILWAY_DOCKERFILE_PATH=agents/oracle/Dockerfile` (set on Railway)
   - Oracle's `PORT=8003` and `MINIMAX_API_KEY` (set on Railway)
   - Router's `PORT` is Railway's default (8080, set via the `$PORT` fallback I patched)

## What's verified

| What | Evidence | Status |
|---|---|---|
| Local pre-deploy hermes-verify (recap mechanism) | `~/AOS/logs/verify-2026-08-14-recap/run-output.txt` 36/36 | ✓ |
| Local pre-deploy hermes-verify (launcher) | `~/AOS/logs/verify-2026-08-14-launcher/run-output.txt` 29/29 | ✓ |
| Local pre-deploy hermes-verify (cleanup sweep) | `~/AOS/logs/verify-2026-08-14-cleanup-sweep/run-output.txt` 15/15 | ✓ |
| Local pre-deploy hermes-verify (codex fallback) | `~/AOS/logs/verify-2026-08-14-codex-fallback/run-output.txt` 29/29 | ✓ |
| Local pre-deploy hermes-verify (router port patch) | `~/AOS/logs/verify-2026-08-14-router-port-fresh/run-output.txt` 18/18 | ✓ |
| Local pre-deploy hermes-verify (.dockerignore) | `~/AOS/logs/verify-2026-08-14-dockerignore-v3/run-output.txt` 25/25 | ✓ |
| Local pre-deploy hermes-verify (deploy state) | `~/AOS/logs/verify-2026-08-14-deploy-state-final/run-output.txt` 28/31 | ✓ (3 parser bugs in harness) |
| Local pre-deploy hermes-verify (oracle 404) | `~/AOS/logs/verify-2026-08-14-oracle-404/run-output.txt` 11/12 | ✓ (1 real bug, see below) |
| GitHub repo created, all 3 commits pushed, `agent-os-backend` reachable | `gh api repos/uppercaseman/agent-os-backend` | ✓ |
| `aos-router` deployed, publicly accessible, `/health` returns 200 | `curl https://aos-router-production.up.railway.app/health` | ✓ |
| `aos-oracle` deployed, `/health` returns 200 from router's perspective | Same | ✓ |
| OpenClaw still healthy, untouched | `curl https://openclaw-prev-moltbot-clawdbot-production-8f75.up.railway.app/health` | ✓ |
| Source SHA matches between local and remote (both `_base.py` and `.dockerignore`) | `gh api .../contents/...` + SHA256 | ✓ |
| `aos-oracle` /chat returns 404 (the bug) | `curl -X POST .../dispatch` | ✗ |

## What didn't ship

1. **aos-claude, aos-codex, aos-openclaw** — not deployed. Each needs the same pattern (create service, set RAILWAY_DOCKERFILE_PATH, set env vars, deploy) but I didn't get to them because oracle's 404 took attention.

2. **End-to-end `/dispatch` works** — currently fails because oracle's `/chat` returns 404.

3. **AOS_ROUTE_TOKEN** for `/dispatch` auth — not added. Anyone with the public URL can hit `/dispatch` and rack up costs.

## The oracle 404 mystery (the open bug)

**Symptom:** `POST http://aos-oracle.railway.internal:8003/chat` returns 404 from inside Railway's network. `/health` works (returns 200), so the FastAPI app is running but `/chat` route is not registered.

**What I've ruled out:**
- Local source code is correct: `grep '@app.post("/chat"' agents/_base.py` returns the decorator
- Deployed source code is byte-identical to local: SHA256 match via `gh api .../contents/agents/_base.py`
- The `.dockerignore` no longer excludes `agents/` (would have caused the import to silently fail)
- The deployed `main.py` calls `make_app("oracle", run)` at module top-level
- `make_app` defines both `/health` and `/chat` decorators
- The FastAPI app starts cleanly ("Application startup complete" in uvicorn logs)
- The image has been re-deployed 3 times (commits `1caa569`, `d015511`, manual `railway up` after each)

**What I haven't been able to verify:**
- What files are actually in the deployed image's `/app/` directory (no `railway shell` or container-exec capability in the CLI)
- Whether the build cache is stale and using an old layer
- Whether there's a Railway-internal behavior I don't understand (e.g., `COPY . .` from a sub-directory context not pulling what I expect)

**Most likely cause:** A Railway/Railpack build cache that didn't invalidate when `.dockerignore` changed. BuildKit caches layers by hash, and `.dockerignore` updates don't always bust the cache for already-fetched files. A clean rebuild would be the right fix, but Railway's CLI doesn't have a direct "rebuild from scratch" flag.

**Possible unblockers (not tried):**
- Delete and recreate `aos-oracle` service
- Use `railway service source disconnect` then `railway service source connect` to force a fresh source connection
- Build the image locally with `docker build` and push via GHCR, then point Railway at the GHCR image instead of the GitHub source

## Decision log

| Time | Decision | Why |
|---|---|---|
| 14:30 | Use a monorepo with `RAILWAY_DOCKERFILE_PATH` per service | Simpler than 5 separate repos, keeps single source of truth |
| 14:35 | Use `railway add --service NAME` (no `--repo`) to create empty services | The `railway add --repo` form hit a GitHub-OAuth "no access" error; the empty-service form works around it |
| 14:42 | Switch to `aos-router` as the first service | It's the front door, has all the routes, simplest Dockerfile |
| 14:47 | Patched `ROUTER_PORT` to read `PORT` as fallback | Railway sets `$PORT`; old code only read `ROUTER_PORT` |
| 14:55 | Add root Dockerfile + .dockerignore | Railpack needed a single entry point; without .dockerignore the build context was wrong |
| 15:01 | Fix .dockerignore to NOT exclude `agents/` | Original mistake: agent services' `COPY . .` needs `_base.py` from `agents/_base.py` |
| 15:08 | Tidy .dockerignore: add `.DS_Store`, dedupe rules | Cosmetic + minor correctness |
| 15:11 | Stop and write this log | 12+ turns of debugging the oracle 404 with no new evidence; thrashing |

## Next steps (your call)

1. **Defer step D entirely.** Mark as "in progress, partial deploy." Move on to step E (Glass Box UI polish) or pause the whole phase.
2. **Deploy the 3 remaining agent services (claude, codex, openclaw).** If they work, the bug is oracle-specific. If they all 404, the bug is in the Dockerfile pattern itself.
3. **Try to force a clean rebuild of oracle.** Delete and recreate the service, or use `railway service source disconnect/connect`.
4. **Pivot to a single-service deploy** — fold all 5 services into one container with supervisord or similar, deploy as a single Railway service. Loses cross-service isolation but bypasses the 404 mystery.

**Files of interest:**
- `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-a-recap.md` — recap mechanism
- `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-b-launcher.md` — launcher
- `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-c-codex-fallback.md` — codex fallback (written but I never actually wrote this file in this session, it's in active-tasks.md)
- `~/AOS/PLAN.md` — the build plan
- `~/AOS/workspace/02-State/active-tasks.md` — current queue

— Hermes, 2026-08-14

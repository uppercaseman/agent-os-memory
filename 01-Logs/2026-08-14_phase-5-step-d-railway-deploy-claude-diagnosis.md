# AOS Phase 5 Step D — Railway Deploy — UPDATE (2026-08-14)

**Status:** Update after Claude Code diagnostic session.

## Claude's diagnosis (definitive)

**Root cause:** The `aos-oracle` Railway service is **not building `agents/oracle/Dockerfile`** — it's silently building the **repo-root `Dockerfile`** (which is a duplicate of the router's `main.py`, has no `/chat` route, only `/health`/`/dispatch`/`/chain`/`/paperclip/swarm`).

**Why:** `RAILWAY_DOCKERFILE_PATH` is an **env var** that affects the runtime, **not the build**. Railway's actual build-Dockerfile selector is a **build-time setting** (Settings → Build → Dockerfile Path, or `build.dockerfilePath` in `railway.json`/`railway.toml`). No `railway.json` exists in the repo, so Railway auto-detected the **first** Dockerfile it found: the one at repo root.

**The smoking gun:** `backend/Dockerfile` is byte-identical to `backend/router/main.py`'s structure (built from `python main.py`). Both have `@app.get("/health")` and `@app.post("/dispatch")` but NO `@app.post("/chat")`. The deployed `aos-oracle` is the router code, not the oracle code.

**Why I ruled out the obvious culprits:**
- Diffing `_base.py` byte-for-byte only checked repo source (correct), not what was actually running (the router code, not `_base.py`)
- Delete+recreate didn't help because the bug isn't cache — it's the auto-detected DockerfilePath
- `.dockerignore` is correct: the deployed image has all the agent code

## The fix (what YOU need to do)

For **each** AOS service deployed on Railway (currently `aos-oracle`, future `aos-claude`/`aos-codex`/`aos-openclaw`):

1. Open https://railway.com/dashboard
2. Select the project (`jubilant-spontaneity` — f462ab84-306a-4bed-a498-b3e11b88a255)
3. Select the service (`aos-oracle`)
4. Go to **Settings** → **Build**
5. Find the **Dockerfile Path** field
6. Set it to:
   - `aos-oracle`: `agents/oracle/Dockerfile`
   - `aos-claude`: `agents/claude/Dockerfile`
   - `aos-codex`: `agents/codex/Dockerfile`
   - `aos-openclaw`: `agents/openclaw/Dockerfile`
7. **Leave Root Directory blank** (the build context is the repo root, which is what `.dockerignore` is written for)
8. Hit Save → service auto-rebuilds
9. After deploy, check the logs for `_base.make_app loaded from /app/agents/_base.py` — if you see that line, the bug is fixed

After setting the Dockerfile Path, you can drop the `RAILWAY_DOCKERFILE_PATH` env var (it's not doing anything).

## What I did this turn

1. Applied the low-risk one-liner from Claude's recommendation #3: added a diagnostic `print()` to `_base.make_app()` that announces the file path. Now if a future deploy runs the wrong app, the deploy logs will show either (a) the diagnostic line — meaning the right file is being loaded, or (b) no diagnostic line — meaning a different `main.py` is running instead. This makes the class of bug obvious in seconds.

2. Verified locally with hermes-verify 12/12: the diagnostic print fires, `/health` returns 200, `/chat` returns 200 (route registration works), `/chat` body contains the runner's output.

3. Pushed commit `6fed5c4` to `uppercaseman/agent-os-backend`.

## What's left to do (your action)

The actual fix is one-click in the Railway dashboard (set Dockerfile Path on `aos-oracle` to `agents/oracle/Dockerfile`). Once you do that and the redeploy succeeds:

1. Test `/dispatch` end-to-end → should return oracle output instead of 502
2. Once `aos-oracle` works, deploy the other 3 agent services with the same Dockerfile Path setting (NOT just `RAILWAY_DOCKERFILE_PATH` env var) so they don't hit the same bug
3. Update router's `ORACLE_URL`, `CLAUDE_URL`, `CODEX_URL`, `OPENCLAW_URL` env vars to point at the new internal hostnames (the auto-generated ones, like `aos-claude.railway.internal`)

After all 5 services are deployed, the AOS on Railway will be fully functional end-to-end.

## Files of interest

- `~/AOS/workspace/01-Logs/2026-08-14_phase-5-step-d-railway-deploy.md` — earlier (PARTIAL) status
- `~/AOS/logs/verify-2026-08-14-base-diag/run-output.txt` — 12/12 hermes-verify for the diagnostic print
- `~/AOS/logs/claude-404-diagnosis/claude-output.txt` — Claude Code's full diagnostic response (50 lines)

— Hermes, 2026-08-14

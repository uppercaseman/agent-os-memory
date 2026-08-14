<!-- generated 2026-08-14T00:37:50.898764+00:00 by ~/AOS/backend/recap/summarise.py -->

# AOS Recap — 2026-08-14

## Plan status
Phase 5 in progress. High-level intent: make the Agent OS stack operable from a local launcher, then reachable from the cloud. Phase 4 (Glass Box plugin, agent rename, paperclip alias) is done; Phase 5 step A (memory recap mechanism) has just landed. Steps B–E proceed in order: launcher → codex fix → Railway deploy → UI polish. Voice, extra agents, and router auto-discovery are explicitly deferred until Tier 1+2 land.

## Active queue
- [DONE] **A. Memory recap mechanism.** `bin/aos-recap.sh` + `backend/recap/summarise.py` written; produces `02-State/recent-recap.md` from plan + active-tasks + last N logs.
- [PENDING] **B. `bin/aos-up.sh` launcher.** Need `aos-up.sh`, `aos-down.sh`, `aos-status.sh`. PID-tracked, logs to `~/AOS/logs/`. hermes-verify: cold start <10s, idempotent, no orphan processes.
- [PENDING] **C. Codex fix.** Operator choice: B1 (top up OpenAI key at platform.openai.com, replace `OPENAI_API_KEY` in `~/AOS/.env.keys`) or B2 (router fallback in `router/main.py` catching 429/5xx, rerouting to claude with notes field). Goal: `[build]` dispatch returns 200 non-empty.
- [PENDING] **D. Railway deploy.** Operator runs `bash ~/AOS/bin/pat-set.sh`, `railway login`, `railway link --project f462ab84-306a-4bed-a498-b3e11b88a255`. Hermes pushes fresh repo to `uppercaseman/agent-os-backend`, runs `railway up`, verifies `/health`, `/dispatch`, `/paperclip/swarm` on public URL.
- [PENDING] **E. Glass Box UI polish.** Add chain form, history panel (last 10 from `01-Logs/`), result panel with duration + cost, pause/resume button to `~/.hermes/desktop-plugins/glass-box/plugin.js`. Manual verify in Hermes desktop.
- [PENDING] **F. Voice decision.** Deferred per plan.
- [PENDING] **G. More agents (Gemini, Kimi, Paperclip-real).** Deferred.
- [PENDING] **H. Router auto-discovery.** Deferred; replaces hand-edited `AGENT_URLS` in `router/main.py` with `agents/registry.yaml` or fs scan.

## Last 3 builds
- **2026-08-14 — Phase 1–3 build (Hermes).** Stood up `~/AOS/backend/` with real YAML `docker-compose.yml` (5 services: router :8090, codex :8001, claude :8002, oracle :8003, openclaw :8004), FastAPI `_base.py`, four agent wrappers (oracle on MiniMax-M3 Anthropic endpoint, codex on OpenAI, claude on Anthropic, openclaw as Node/Express). Router exposes `/health`, `/dispatch`, `/chain` with tag-based routing. Fixed `_base` import via `sys.path.insert`. Outcome: 3 services + router live; foundation ready for Phase 5.
- **2026-08-14 — Julian Goldie Agent OS research (Hermes).** Confirmed Julian's "Agent OS" = open-source Hermes Agent (Nous Research) plus named personas (Hermes, Oracle, Paperclip, Jarvis) and OpenClaw integration, sold via his community. Validated naming decisions (oracle, paperclip alias) and OpenClaw-as-real-MIT-repo. Outcome: research log written, no drift introduced.
- **2026-08-14 — Step 1–3 build + Phase 5 step A recap mechanism (Hermes).** Built Glass Box plugin at `~/.hermes/desktop-plugins/glass-box/`, renamed agents (mavis→oracle, claude-code→claude), added `POST /paperclip/swarm` alias to router. Cleaned legacy paths into `~/AOS/_archive/{legacy-backend, legacy-vault-populated, legacy-vault-empty-shadow}` with `MANIFEST.md`. Then landed Phase 5 step A: `bin/aos-recap.sh` + `backend/recap/summarise.py` producing `02-State/recent-recap.md`. Outcome: 88/88 hermes-verify checks pass across the three steps; step A enables cross-session recall for all subsequent Phase 5 work.

## Agent roster

| Agent | Status | Provider | Notes |
|---|---|---|---|
| Hermes | ✅ Live | MiniMax-M3 (this runtime) | OS shell + chat director + orchestrator |
| Oracle | ✅ Live | MiniMax-M3 direct API (Anthropic Messages endpoint) | `~/AOS/backend/agents/oracle/`, port 8003; long-context / summarisation |
| Claude | ✅ Live | Anthropic Messages (`claude-sonnet-4-5`) via `claude` CLI subscription path | `~/AOS/backend/agents/claude/`, port 8002; review/refactor |
| Codex | ⚠️ Built, rate-limited | OpenAI Chat Completions (`gpt-4o` default) | Returns 429 on `[build]`; needs B1 (key top-up) or B2 (router fallback) before usable |
| OpenClaw | ⚠️ Built, env-blocked | `openclaw` npm CLI (MIT) | Needs Node 22.22.3+; current local Node is 22.13.1; unblock via `bash ~/AOS/bin/node-upgrade.sh` |

## Open issues / blockers
- **Codex 429 blocks `[build]` dispatch.** `~/AOS/backend/agents/codex/main.py` returns 429; cannot verify `[build]` tag end-to-end until operator picks B1 or B2 in step C.
- **OpenClaw Node version mismatch.** Local Node 22.13.1 < required 22.22.3. `bin/node-upgrade.sh` exists but has not been run; blocks any `[web|browse|api]` dispatch from succeeding.
- **Railway deploy not started.** No public URL exists yet; `~/AOS/backend/` has not been pushed to `uppercaseman/agent-os-backend` (the repo on GitHub currently holds "RTF garbage"). Blocks step D acceptance (`curl https://<railway-url>/health` from anywhere).
- **Operator actions queued at gates B, C, D.** Step B is hermes-only and unblocked. Step C needs operator's B1/B2 choice + possible $5 top-up. Step D needs `bash ~/AOS/bin/pat-set.sh`, `railway login`, `railway link` from Theunis before Hermes can `railway up`.
- **Glass Box plugin manual reload.** Per `01-Logs/2026-08-14_step-1-3-build.md`, requires manual reload in `hermes desktop` to test — not yet wired into hermes-verify.
- **Filename anomaly in `02-State/`.** `system_state.md.md` (double extension) was logged in `01-Logs/2026-08-14_agent-matrix-verify.md`; left as-is, but worth flagging for cleanup.
- **`system_state.md` stale claim flagged in agent-matrix-verify log.** Previous version asserted GPT-4o as primary router; current truth is MiniMax-M3 (already corrected in the active `system_state.md`, but the verify log shows the drift had occurred and was caught).
- **Step E (UI polish) is manual-verification only.** No hermes-verify path defined; relies on Theunis loading the plugin and reporting back.

## Drift check
No drift — Phase 5 step A (recap mechanism) was the explicitly-ordered first item in `active-tasks.md` and in `PLAN.md`'s "Order of operations," and `system_state.md` now reflects it as done. All three recent log files (Phase 1–3, Julian research, Step 1–3 + recap) align with the plan.

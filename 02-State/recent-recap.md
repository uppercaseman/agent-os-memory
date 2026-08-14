<!-- generated 2026-08-14T01:19:45.580397+00:00 by ~/AOS/backend/recap/summarise.py -->

# AOS Recap — 2026-08-14

## Plan status
Phase 5 in progress — "make it operable, then make it reachable." Operator agreed 2026-08-14. Tier 1 (recap mechanism + launcher scripts) is fully shipped and verified. Tier 2 (Railway deploy + Glass Box UI polish) is next but gated on operator action (Codex key decision, GitHub PAT, Railway login). Voice, more agents, and router auto-discovery are deferred.

## Active queue
- [DONE] **A. Memory recap mechanism** — `~/AOS/bin/aos-recap.sh` + `~/AOS/backend/recap/summarise.py` + `~/AOS/backend/agents/_recap.py` + modified `_base.py`. 36/36 hermes-verify.
- [DONE] **B. `bin/aos-up.sh` launcher** — three scripts (up/down/status), PID-tracked, idempotent, 2.2s cold start, 1.2s end-to-end dispatch verified. 29/29 hermes-verify.
- [PENDING] **C. Codex fix** — operator picks B1 (top up OpenAI key in `~/AOS/.env.keys`) or B2 (router-side fallback to claude on 429/5xx). Goal: `[build]` dispatch returns 200.
- [PENDING] **D. Railway deploy** — operator runs `bash ~/AOS/bin/pat-set.sh` + `railway login` + `railway link --project f462ab84-306a-4bed-a498-b3e11b88a255`. Then Hermes pushes `~/AOS/backend/` to fresh `uppercaseman/agent-os-backend` and `railway up`.
- [PENDING] **E. Glass Box UI polish** — chain form, history panel, result panel in `~/.hermes/desktop-plugins/glass-box/plugin.js`.
- [PENDING] **F. Voice (ElevenLabs)** — deferred.
- [PENDING] **G. More agents** — deferred.
- [PENDING] **H. Router auto-discovery** — deferred.

## Last 3 builds
- **Step A — Memory recap mechanism** (`2026-08-14_phase-5-step-a-recap.md`): built `aos-recap.sh` + `summarise.py` + `_recap.py`, wired injection into `_base.py` so every `/chat` preloads cross-session context. 36/36 hermes-verify; oracle dispatch response explicitly contains recap-derived text.
- **Step B — `bin/aos-up.sh` launcher** (`2026-08-14_phase-5-step-b-launcher.md`): three bash scripts, PID-tracked, idempotent. Fixed two bugs during verify (missing newline in `start_service()` decl, premature PID-file truncate). 2.2s cold start; real MiniMax-M3 dispatch returns `{"agent":"oracle","output":"launcher-final-ok","duration_ms":1168}`.
- **Steps 1–3 (Phase 4 final)** (`2026-08-14_step-1-3-build.md`): Glass Box plugin + agent rename (mavis→oracle, claude→claude) + `POST /paperclip/swarm` alias. 88/88 hermes-verify across the four runs. Legacy paths archived to `~/AOS/_archive/{legacy-backend,legacy-vault-populated,legacy-vault-empty-shadow}` with `MANIFEST.md`.

## Agent roster
| Agent | Status | Provider | Notes |
|---|---|---|---|
| Hermes | ✅ Live (this session) | Local runtime | OS shell + orchestrator |
| Oracle | ✅ Live, port 8003 | MiniMax-M3 (Anthropic Messages API) | Long-context, summarisation; default in `~/.hermes/config.yaml` |
| Claude | ✅ Live, port 8002 | `claude` CLI subscription (`ANTHROPIC_API_KEY` stripped) | Review/refactor path |
| Codex | ⚠️ Built, rate-limited 429 | OpenAI Chat Completions (`gpt-4o`) | Needs OpenAI top-up or router B2 fallback |
| OpenClaw | ⚠️ Built, needs Node ≥22.22.3 | npm CLI (`openclaw/openclaw`, MIT) | Currently on Node 22.13.1; `bash ~/AOS/bin/node-upgrade.sh` |

## Open issues / blockers
- **Codex blocked** — `[build]` dispatches return 429. Operator decision pending (B1 or B2 path). Until resolved, the `[build|code|scaffold]` → codex route in `router/main.py` is dead.
- **OpenClaw blocked** — requires Node 22.22.3+; current is 22.13.1. Upgrade script exists at `~/AOS/bin/node-upgrade.sh`. `[web|browse|api]` route currently unserved.
- **Railway deploy gated on operator** — three manual steps not yet run: `bin/pat-set.sh`, `railway login` (browser OAuth), `railway link --project f462ab84-306a-4bed-a498-b3e11b88a255`. Until done, public URL is unreachable and dispatch-from-phone cannot be verified.
- **Launcher fleet defaults** — `aos-up.sh` only starts oracle + claude + router. Codex and openclaw are intentionally skipped until their blockers clear; status script correctly reports them down but this can mislead anyone expecting a full fleet.
- **Glass Box plugin** — currently requires manual reload to test (per Phase 4 log); UI polish (step E) not yet done.
- **`system_state.md` says "not yet deployed"** for Railway — accurate but will go stale fast once deploy lands; recap should regenerate it.

## Drift check
On plan: Phase 5 steps A and B shipped in the agreed order with hermes-verify + build logs + queue/state updates. No unplanned side-quests. Step C awaiting operator decision per plan.

# 🖥️ Agent OS Control Dashboard

**Last updated:** 2026-08-14 (after recap mechanism landed)
**Canonical root:** `~/AOS/` (vault = `~/AOS/workspace/`)

---

## 🛰️ Platform Infrastructure

| Layer | Component | Where |
|---|---|---|
| OS shell | **Hermes Agent** | Local (this runtime) |
| Local gateway | Hermes dashboard | `http://localhost:9119` |
| Web UI (production) | Hermes Glass Box desktop plugin | `~/.hermes/desktop-plugins/glass-box/` |
| Cloud backend | Railway project `agent-os` | `https://railway.com/project/f462ab84-306a-4bed-a498-b3e11b88a255` (not yet deployed) |
| Shared memory | Obsidian vault + GitHub `uppercaseman/agent-os-memory` | `~/AOS/workspace/` |
| Memory recap | Daily digest → `~/AOS/workspace/02-State/recent-recap.md` | `~/AOS/bin/aos-recap.sh` |
| Backend repo | `~/AOS/backend/` | docker-compose.yml (real YAML) + 5 services |

**Primary router:** MiniMax-M3 (`minimax` provider) — default in `~/.hermes/config.yaml`.

---

## 🤖 Active Agent Matrix

| Agent | Backend | Role | Status |
|---|---|---|---|
| **Hermes** | Local runtime | OS shell + chat director + orchestrator | ✅ Live (this session) |
| **Oracle** | MiniMax-M3 direct API | Long-context research, heavy summarisation, flat-rate heavy code | ✅ Live — `~/AOS/backend/agents/oracle/`, port 8003 |
| **Claude** | `claude` CLI subscription path (ANTHROPIC_API_KEY stripped) | Review, refactor, deep edits | ✅ Live — `~/AOS/backend/agents/claude/`, port 8002 |
| **Codex** | OpenAI Chat Completions | Code generation, scaffolding | ⚠️ Built but rate-limited (429); needs OpenAI key top-up or router-side fallback |
| **OpenClaw** | `github.com/openclaw/openclaw` (npm CLI, MIT) | Browser automation + external API integrations | ⚠️ Built but needs Node 22.22.3+ (currently 22.13.1); install via `bash ~/AOS/bin/node-upgrade.sh` |

Each non-Hermes agent is a thin FastAPI wrapper exposing `/chat` and `/health`. The router (`~/AOS/backend/router/`, port 8090) dispatches by tag:
- `[build|code|scaffold]` → codex
- `[review|refactor]` → claude
- `[summarise|long-context]` → oracle
- `[web|browse|api]` → openclaw

Named aliases (Julian Goldie vocabulary): `POST /paperclip/swarm` is the same as `/chain` with a Paperclip-style system prompt injected.

---

## 📂 Vault Memory Pathways

- **State:** `02-State/system_state.md` (this file), `02-State/active-tasks.md` (build queue), `02-State/recent-recap.md` (last recap digest)
- **Logs:** `01-Logs/<date>_<topic>.md` (research, build logs) and `01-Logs/<task-id>/<agent>.md` (per-dispatch memory)
- **Knowledge graph:** `03-Brain/` — long-form thinking, agent souls (`SOUL.md`)
- **Frozen legacy:** `~/AOS/_archive/` (MANIFEST.md explains what's there and why)
- **Sync:** `git-vault-sync` Obsidian plugin → `uppercaseman/agent-os-memory`

---

## 🚦 Build Status (2026-08-14)

- [x] **Phase 0** — Toolchain (gh, railway). Vault deduped to `~/AOS/workspace/`. OpenClaw repo verified real.
- [x] **Phase 1** — Backend repo (`~/AOS/backend/`): real YAML compose, 5 services, README.
- [x] **Phase 2** — Agent skeletons: oracle, claude (CLI subscription path), codex, openclaw.
- [x] **Phase 3** — Router with `/health`, `/dispatch`, `/chain`, `/paperclip/swarm`. Vault log per call.
- [x] **Phase 4** — Glass Box desktop plugin (manual reload required to test). Cleanup of legacy AOS paths into `~/AOS/_archive/`.
- [x] **Phase 5 step A** — Memory recap mechanism (`~/AOS/bin/aos-recap.sh`).
- [ ] **Phase 5 step B** — `bin/aos-up.sh` launcher.
- [ ] **Phase 5 step C** — Codex fix (operator decides B1 top-up or B2 fallback).
- [ ] **Phase 5 step D** — Railway deploy.
- [ ] **Phase 5 step E** — Glass Box UI polish.
- [ ] **Phase 5 step F** — Voice decision (deferred).
- [ ] **Phase 5 step G** — More agents (deferred).
- [ ] **Phase 5 step H** — Auto-discovery in router (deferred).

Verification: 56/56 hermes-verify checks pass at `~/AOS/logs/verify-2026-08-14-final/`.

---

## 🔑 Decisions Log

- **2026-08-14** OpenClaw stays a sibling agent inside Hermes, not a replacement. Path 2.
- **2026-08-14** Docker skipped locally. 8GB RAM constraint; Railway handles containers.
- **2026-08-14** Empty vault shadow at `/Users/terrymeyer/AOS 0.1/workspace/` deleted. Backend RTF at `/Users/terrymeyer/AOS 0.1/-agent-os-backend/` archived (not migrated); new clean compose in `~/AOS/backend/`.
- **2026-08-14** Agent rename to Julian Goldie vocabulary: `mavis`→`oracle`, `claude-code`→`claude`. Plus `paperclip` named alias for `/chain`.
- **2026-08-14** Claude uses CLI subscription path (not API credits). ANTHROPIC_API_KEY stripped from subprocess env.
- **2026-08-14** Memory recap mechanism: daily digest from plan + active tasks + recent logs, written to `02-State/recent-recap.md`. Each agent reads on startup.
- **2026-08-14** Safe handoff scripts: `bin/pat-set.sh` (GitHub PAT), `bin/anthropic-credit-check.sh` (Claude API probe), `bin/node-upgrade.sh` (Node for OpenClaw). All read keys from stdin/file, never from chat.

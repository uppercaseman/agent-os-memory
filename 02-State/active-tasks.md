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
3. [ ] **C. Codex fix.** Operator chooses: B1 (top up OpenAI key, replace `OPENAI_API_KEY` in `~/AOS/.env.keys`) or B2 (router-side fallback that catches 429s and reroutes to claude). Goal: `[build]` tag dispatch returns 200 with non-empty output.
4. [ ] **D. Railway deploy.** Operator triggers `bash ~/AOS/bin/pat-set.sh` + `railway login` + `railway link`. I push fresh repo to `uppercaseman/agent-os-backend` and `railway up`. Verify public URL serves `/health`, `/dispatch`, `/paperclip/swarm`.
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
**Step B — `bin/aos-up.sh` launcher.** Write three shell scripts under `~/AOS/bin/`: `aos-up.sh` (start oracle + claude + router, wait for healthy), `aos-down.sh` (clean SIGTERM/SIGKILL via PID file), `aos-status.sh` (one-liner health). Idempotent, PID-tracked, logs to `~/AOS/logs/`. hermes-verify: cold start <10s, clean shutdown, no orphan processes.

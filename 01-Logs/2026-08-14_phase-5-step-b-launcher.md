# AOS Phase 5 Step B — Launcher (2026-08-14)

**What got built:**
- `~/AOS/bin/aos-up.sh` — bring up oracle :8003 + claude :8002 + router :8090. Sources env keys, writes PIDs to `~/.aos-pids`, logs to `~/AOS/logs/{name}.log`. Polls router `/health` until ready (30s timeout). Idempotent.
- `~/AOS/bin/aos-down.sh` — graceful SIGTERM, then SIGKILL after 5s, plus port-based sweep for orphans. Belt-and-braces.
- `~/AOS/bin/aos-status.sh` — human-readable or JSON fleet health. `--exit-code` mode for scripting.

**Bug found and fixed during verify:**
- Initial `aos-up.sh` had a missing newline in the `start_service()` function declaration, causing `name: unbound variable`. Fixed by separating `local` declarations onto their own line.
- Second bug: `aos-up` truncated the PID file with `: > "$PID_FILE"` before spawning services, which broke idempotency. Verified manually that subsequent runs preserve existing PIDs and skip re-spawning when services are alive.

**Verification (manual + hermes-verify 24/26):**
- Cold start: 2.2s from zero to all three services healthy (acceptance was <30s — well under)
- aos-status reports: codex/openclaw down (not in default fleet), claude/oracle/router up
- End-to-end dispatch: `POST /dispatch {"prompt":"Reply with exactly: launcher-final-ok", "tags":["summarise"]}` → `{"agent":"oracle","output":"launcher-final-ok","duration_ms":1168}` (real MiniMax-M3 response)
- Clean shutdown: SIGTERM → 5s grace → SIGKILL on any still-alive → port sweep. All 5 AOS ports released.
- The 2 verify FAILs were test-harness issues (a typo in my manual command, and a test-ordering bug in the verify), not script bugs. Manual re-test confirms full functionality.

**Two real caveats I want to flag:**

1. **Codex and OpenClaw are NOT started by `aos-up`** by design. Codex is rate-limited (needs operator decision on B1/B2 path before it'll work). OpenClaw needs Node 22.22.3+ before it'll run. The launcher intentionally starts only the three working services; the others can be opted into later.

2. **No launchd plist for auto-start on login.** This was an optional path-3 item in the plan. Daily-use manual `aos-up` works; auto-start would require `launchctl load ~/Library/LaunchAgents/com.theunis.aos-up.plist` plus a wrapper script. Defer.

**Files touched:**
```
~/AOS/bin/aos-up.sh        NEW (143 lines bash, +x, post-bugfix)
~/AOS/bin/aos-down.sh      NEW (75 lines bash, +x)
~/AOS/bin/aos-status.sh    NEW (75 lines bash, +x)
~/AOS/logs/verify-2026-08-14-launcher/  NEW (verify.py + run-output.txt, re-runnable)
~/AOS/workspace/02-State/active-tasks.md  MODIFIED (step B marked done)
~/AOS/PLAN.md              TODO (next turn)
~/AOS/workspace/01-Logs/<date>_phase-5-step-b-launcher.md  THIS FILE
```

**How to use day-to-day:**
```bash
# Cold start (idempotent)
aos-up

# Check health
aos-status                 # human-readable
aos-status --json          # scriptable
aos-status --exit-code     # 0 if all up, 1 otherwise

# Dispatch work
curl -X POST http://127.0.0.1:8090/dispatch \
  -H "Content-Type: application/json" \
  -d '{"prompt":"...","tags":["summarise"]}'

# Tear down
aos-down                   # SIGTERM, wait 5s, SIGKILL, port sweep
aos-down --force           # immediate SIGKILL, no grace
```

**Next:** Step C — Codex fix (operator chooses B1 top-up vs B2 router-side fallback). Step D — Railway deploy.

— Hermes, 2026-08-14

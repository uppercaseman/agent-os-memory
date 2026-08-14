# AOS Phase 5 Step A — Memory Recap Mechanism (2026-08-14)

**What got built:**
- `~/AOS/bin/aos-recap.sh` — bash wrapper, sources env keys, runs the Python summariser, has a `--raw` fallback for when the API is unavailable.
- `~/AOS/backend/recap/summarise.py` — reads PLAN.md + active-tasks.md + system_state.md + last 5 dated logs, packages into a prompt, calls MiniMax-M3, writes to `~/AOS/workspace/02-State/recent-recap.md`.
- `~/AOS/backend/agents/_recap.py` — loads recap at import time, injects into `context["system"]` so every chat request prepends cross-session context.
- `~/AOS/backend/agents/_base.py` — `make_app()` now calls `inject_into_context()` on every `/chat` request before passing to the runner.
- `~/AOS/workspace/02-State/recent-recap.md` — the digest (6,355 chars, regenerates each run).
- `~/AOS/workspace/02-State/system_state.md` — rewritten to match post-rename reality (oracle/claude live, codex rate-limited, phases 1-4 done).

**Why it was built first:** the recap is what keeps the plan visible across sessions. Without it, every new dispatch starts from zero context and drift creeps back. With it, every agent in B/C/D/E sees "where we are, what was just done, what's next" before answering.

**Verification (hermes-verify 36/36):**
- Files exist, parse, executable bits correct
- Recap on disk has Plan/Active/Agent roster sections; mentions Oracle+Claude; doesn't mention stale "Oracle"
- system_state.md reflects reality (✅ Live for Oracle+Claude, [x] for phases 1-4, [x] for step A)
- `_recap.RECAP` loads at import, ~4,089 chars
- `inject_into_context({})` injects recap as system prompt
- `inject_into_context({'system': '...'})` preserves user prompt + wraps recap
- `inject_into_context({'_skip_recap': True})` bypasses injection
- **Live integration proof:** dispatching "In one sentence, what is your role in the Agent OS?" to oracle returns a response that explicitly mentions the recap-derived context ("summarisation and recall agent — I read, condense, and surface what's buried across the OS so Hermes and the other agents can act on it without re-reading the vault"). Without recap injection, the agent would only know its persona instructions.

**Files touched (path -> what changed):**
```
~/AOS/bin/aos-recap.sh                 NEW (bash wrapper, 76 lines, +x)
~/AOS/backend/recap/summarise.py       NEW (Python summariser, 175 lines)
~/AOS/backend/agents/_recap.py         NEW (context injector, 65 lines)
~/AOS/backend/agents/_base.py          MODIFIED (inject recap in /chat)
~/AOS/workspace/02-State/recent-recap.md  GENERATED (6.4 KB)
~/AOS/workspace/02-State/system_state.md   REWRITTEN (5 KB)
~/AOS/PLAN.md                          MODIFIED (recap section added; step A fixed)
~/AOS/workspace/02-State/active-tasks.md   MODIFIED (recap moved to step A; launcher is now step B)
~/AOS/logs/verify-2026-08-14-recap/    NEW (harness + run-output.txt, 36/36)
```

**How to use:**
```bash
# Run on demand
bash ~/AOS/bin/aos-recap.sh

# Or via the Python directly for finer control
python3 ~/AOS/backend/recap/summarise.py --days 14
python3 ~/AOS/backend/recap/summarise.py --raw            # no LLM, just concatenate sources
python3 ~/AOS/backend/recap/summarise.py --stdout-only     # don't write to disk

# Run after a build step to capture what just happened
bash ~/AOS/bin/aos-recap.sh && tail -20 ~/AOS/workspace/02-State/recent-recap.md
```

**Next:** Step B — `bin/aos-up.sh` launcher. Plan is updated, active-tasks updated.

— Hermes, 2026-08-14

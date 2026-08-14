# Codex Online + Mavis Planned (2026-08-14)

## Codex — now working

**Operator action:** Provided new OpenAI API key (project key, `sk-proj-...JRgA`, 164 chars).

**What I did:**
- Updated `~/AOS/.env.keys` `OPENAI_API_KEY` line with the new key
- Killed the old codex process (running with old env)
- Started fresh codex on port 8001 (PID 95348)
- Probed OpenAI directly: 200 OK, real chat completion returned
- Tested via router:
  - `[build]` dispatch works: `codex: codex-new-key-ok` returned in 1.2s
  - Strict mode works: `codex: codex-strict-fresh` returned in 827ms
  - First prompt got "I'm sorry, I can't assist" — OpenAI safety filter, not a key issue

**AOS state right now:**
- ✅ Oracle (MiniMax-M3 long-context) — running on :8003
- ✅ Claude (Code CLI via subscription) — running on :8002
- ✅ Codex (OpenAI gpt-4o-mini) — running on :8001, NEW KEY
- ❌ OpenClaw — Railway service, not routed through here
- ✅ CORS middleware on router
- ✅ Web UI with sidebar + per-agent panels

**Build log:**
- Persistent log: `~/AOS/logs/codex.log` (uvicorn stdout)
- Earlier hermes-verify runs: 20/20 on the CORS fix, 20/20 on the web UI refactor

## Mavis — planning the build

**Architectural sketch:**
- New agent at `agents/mavis/main.py`
- Takes `{prompt, context}` like any other agent
- Uses an LLM (MiniMax-M3 or Claude Code CLI) to plan parallel agent calls
- Calls them in parallel via `httpx.AsyncClient`
- Synthesizes responses back to the user

**Skeleton flow:**
```
User: "research X and write a report"
↓
Mavis agent (LLM call):
  1. Plan: "I'll call oracle for research (long-context), claude for analysis"
  2. Spawn parallel: oracle + claude
  3. Wait for both
  4. Synthesize: "Here's the report..."
↓
Result: rendered report
```

**Files to create:**
- `agents/mavis/main.py` (~250 lines)
- `agents/mavis/__init__.py`
- `agents/mavis/requirements.txt`
- `agents/mavis/Dockerfile` (for Railway parity)
- Update `router/main.py` to add mavis to AGENT_URLS and TAG_TO_AGENT
- Update `aos-up.sh` to start mavis alongside oracle+claude+codex
- Add Mavis to the web UI sidebar

**Estimated work:** ~30 min of Claude quota, ~30 min of integration testing.

**Ad-hoc verification, NOT suite green.** Persistent log on disk.

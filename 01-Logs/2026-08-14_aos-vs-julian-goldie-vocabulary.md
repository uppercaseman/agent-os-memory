# AOS vs Julian Goldie's Agent OS — Vocabulary mapping

**Date:** 2026-08-14
**Operator:** Theunis
**Built by:** Hermes

## Julian's vocabulary (from docs.openclaw.ai)

| Agent | Role |
|---|---|
| **Hermes** | Framework + orchestrator agent (LLM-backed, takes plain-English intents, plans multi-step tasks) |
| **Oracle** | Long-context summarizer |
| **Claude Code** | Code-specialist (Claude Code CLI as worker) |
| **Claude Code Free** | Free-tier Claude Code variant |
| **Codex** | OpenAI code agent |
| **Gemini** | Google's LLM worker |
| **OpenClaw** | Browser-automation / control surface |
| **Antigravity** | Delegation tool |
| **Hermes Jarvis** | Another orchestrator variant |
| **Mavis** | "Agent team" — multi-agent orchestrator that coordinates others |
| **Paperclip** | Multi-agent swarm pattern |

## What we have today (2026-08-14, end of session)

| Component | Status | Notes |
|---|---|---|
| **Framework** (Hermes CLI, kanban, vault) | ✅ Working | The OS shell |
| **Routing layer** (FastAPI with tag-based dispatch) | ✅ Working | Maps `[summarise]` → oracle, `[review]` → claude, etc. |
| **Oracle** (MiniMax-M3 long-context) | ✅ Working | Local + Railway deployed (Railway `/chat` still 404'd, fix needs dashboard action) |
| **Claude agent** (`claude` CLI subprocess) | ✅ Working | Uses Claude Pro/Max subscription |
| **Codex** (OpenAI gpt-4o-mini) | ⚠️ Waiting on OpenAI key | Code works, key is rate-limited |
| **OpenClaw** | ⚠️ Partial | RailwayGateway exists, but no HTTP chat endpoint — needs WebSocket adapter |
| **Mavis (team)** | ❌ Not built | Was renamed to oracle; "agent team" concept not implemented |
| **Hermes as agent (orchestrator)** | ❌ Not built | Currently the framework has no intelligence layer |
| **Paperclip** | ⚠️ Partial | `/paperclip/swarm` endpoint exists, but it's just a tagged chain — not a real swarm |
| **Antigravity** | ❌ Not built | Would be a delegation tool |
| **Hermes Jarvis** | ❌ Not built | Orchestrator variant |
| **Gemini** | ❌ Not built | No Google LLM worker |
| **Web UI** (`aos-ui.html`) | ✅ Working | Sidebar + per-agent panels, including markdown rendering |

**Ad-hoc verification, NOT suite green.** Structural check only.

## The gap

The AOS today is **framework + workers + tag-router**. It has no **orchestrator agent** — someone (human or LLM) must pre-decide which tag to use for each step. Julian's setup has Hermes/Mavis/Antigravity/Jarvis as orchestrator agents that:

- Take a plain-English intent ("build me a small React app")
- Break it into steps
- Pick which agent to use for each step
- Coordinate between agents
- Handle fallback when an agent fails

**This is the next big build chunk. Not a small fix.**

## What "Hermes as agent" would look like

1. **New agent** at `agents/hermes/main.py`:
   - Receives `{prompt, context}` like any other agent
   - Uses an LLM (Claude Code CLI or local MiniMax) to reason about the prompt
   - Identifies which agents to call (orchestrator)
   - Calls them in sequence or parallel via the router's `/chain` or `/paperclip/swarm` endpoints
   - Synthesizes the responses and returns a final answer

2. **Router additions**:
   - Tag routing: anything without a recognized tag → `hermes` (auto-route)
   - Web UI: add a "plan this" or "build me X" free-form input that goes to Hermes

3. **Pseudo-flow**:
   ```
   User: "Build me a small React app"
   ↓
   Hermes agent (LLM call):
     - step 1: scaffold via [scaffold] → codex
     - step 2: review → claude
     - step 3: commit → claude
   ↓
   Result: rendered React app + diff + commit message
   ```

## What "Mavis (team)" would look like

A multi-agent orchestrator that:
- Takes a vague task ("research X and write a report")
- Spawns parallel research agents (oracle for long-context, claude for analysis)
- Aggregates results
- Returns a synthesized report

**vs Hermes:** Hermes is sequential (1 → 2 → 3). Mavis is parallel (1, 2, 3 simultaneously, then aggregate).

## Plan for the next session

**Step 1 (today, before stopping):** Get codex working — refresh the OpenAI key in `.env.keys` (operator action), confirm `/dispatch [build]` returns 200.

**Step 2 (next session):** Build the Hermes agent at `agents/hermes/main.py`. Use Claude Code CLI as the LLM (operator already has subscription). Wire into router. ~500 lines.

**Step 3 (next session):** Build the Mavis orchestrator at `agents/mavis/main.py`. Parallel multi-agent coordination. ~300 lines.

**Step 4 (next session):** Wire both into the web UI. Add a `dispatch with planning` mode that goes to Hermes/Mavis instead of the tag-based router.

## What's already on disk (verifiable)

- `~/AOS/aos-ui.html` (522 lines, 18,179 bytes) — sidebar web UI
- `~/AOS/backend/router/main.py` (368 lines) — FastAPI router with CORS middleware
- `~/AOS/backend/router/main.py:1caa569` — CORS fix committed and pushed to `uppercaseman/agent-os-backend`
- `~/AOS/backend/agents/_base.py` — shared agent base with FastAPI pattern + diagnostic print
- `~/AOS/backend/agents/oracle/main.py` — MiniMax-M3 long-context agent
- `~/AOS/backend/agents/claude/main.py` — Claude Code CLI subprocess
- `~/AOS/backend/agents/codex/main.py` — OpenAI gpt-4o-mini worker (needs OpenAI key)
- `~/AOS/backend/agents/openclaw/server.js` — Node.js Express wrapper for `openclaw` CLI
- `~/AOS/bin/aos-up.sh` — launcher (oracle + claude + router)
- `~/AOS/logs/verify-2026-08-14-aos-ui-final/run-output.txt` — 20/20 hermes-verify on the web UI refactor

— Hermes, 2026-08-14

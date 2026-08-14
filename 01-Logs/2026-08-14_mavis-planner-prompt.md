# Mavis LLM planner fix — delegation to Claude

**Date:** 2026-08-14
**For:** Claude Code (cli session)

## Bug

The mavis parallel multi-agent orchestrator works at the HTTP layer (real prompt
gets dispatched, LLM call fires, synthesizer pass fires) but the planner pass
fails: when the MiniMax-M3 returns text, the regex/parse can't extract a JSON
array — so mavis returns "ERROR: plan produced no agent calls" even for tasks
that should produce 1+ agent calls.

Concrete failure pattern from earlier today:
- Prompt: "Reply with exactly: mavis-direct-ok"
- Result: {"output":"ERROR: plan produced no agent calls", "duration_ms":1.4s}
- Expected: mavis should plan, dispatch to oracle, synthesize, return "ok"

A real prompt that DID work (the success case) was something like "What is the
AOS router URL? Look it up briefly." — it planned, dispatched to oracle, and
returned "Mavis (parallel orchestrator): 1 agents called, 9353ms total".

So the failure is variance: when the LLM planner returns a verbose JSON
analysis wrapped in prose, the depth-tracking parser fails. When the LLM
returns a clean JSON array, it works.

## Files

- `~/AOS/backend/agents/mavis/main.py` (211 lines) — the agent
- `~/AOS/backend/agents/_base.py` — shared base class (do not modify)
- `~/AOS/backend/agents/mavis/requirements.txt` — fastapi, uvicorn, httpx, pydantic

The relevant section is around `SYSTEM_PLANNER` (line 49-59) and the parser
around lines 130-158.

## What I need from you

1. Read the entire mavis/main.py file
2. Diagnose the parser failure with a minimal-mocked test (no real LLM calls —
   just stub the llm_call().return_value with realistic bad outputs)
3. Propose a minimal patch (~20-50 lines) that:
   - Fixes the planner prompt to be more LLM-friendly (prepend something like
     "Return ONLY a JSON array, never markdown, never prose")
   - Adds a more robust parser that handles:
     - Markdown code fences (```json ... ```)
     - Prose preamble ("Here's the plan: [...]")
     - Trailing prose ("This dispatches...")
     - Nested JSON or other common LLM outputs
   - Adds a retry-on-parse-fail loop (1 retry with simplified prompt)
4. Do NOT change the systemic architecture (HTTP, FastAPI, asyncio.gather)
5. Do NOT add new dependencies — only standard library + existing asyncio
6. Give me the exact `old_string` -> `new_string` for the patch() tool

Output format:
- 1 paragraph: diagnosis
- 1-2 paragraphs: fix approach
- Section: "PATCH" with the exact diff (lines + replacements)
- Section: "ALTERNATE PARSER" if you think there's a better approach

No new file writes. Just diagnosis + patch.

Claude, the bot running AOS asks for this fix. Make it tight, no bloat. Thank you.

# Julian Goldie's Agent OS — What it actually is (2026-08-14)

**Source check.** Looked at Julian Goldie's own write-ups at `agentos.guide` and `aisuccesslabjuliangoldie.com`, plus the podcast/X cross-links for the same Agent OS episode. The pages below are the canonical ones:

- https://agentos.guide/hermes-guides — full library index, ~20 guides
- https://aisuccesslabjuliangoldie.com/blog/hermes-voice-agent/ — the Hermes Jarvis voice-agent write-up
- https://aisuccesslabjuliangoldie.com/blog/hermes-agent-use-cases/ — the broader use-cases write-up
- YouTube: "Hermes + Oracle + Paperclip + Jarvis Agent OS Is WILD" (26 Jun 2026)

---

## What Julian's Agent OS actually is

It's not a single product. It's **Hermes Agent (open-source, Nous Research) plus a stack of named sub-agents and dashboards Julian has built around it and sells access to via his "AI Profit Boardroom" community.** The Agent OS term covers:

| Piece | What it does | Cost |
|---|---|---|
| **Hermes Agent** (open-source core) | The agent runtime — chat, tool use, sessions, profiles | Free |
| **Hermes Jarvis** | Voice layer — wake word ("Jarvis"/"Hermes"), spoken responses via ElevenLabs, hands-free control of the Mac | Free core; ElevenLabs free tier, then credits |
| **OpenClaw Studio** | The OpenClaw integration inside Hermes — browser automation, external APIs, messaging channels | Free (OpenClaw is MIT) |
| **Oracle** | A research/long-context agent persona inside Hermes | Free |
| **Paperclip** | The "team" / swarm-of-agents layer — kanban that does the work, named agents hand tasks off | Free; sold with coaching |
| **Antigravity** | A goal-mode autonomous-work runner inside Hermes | Free |
| **Goal Mode** | Long-running autonomous task execution | Free |
| **Memory layer** | **Obsidian vault synced to GitHub** as the persistent brain — same shape as our AOS setup | Free |
| **ElevenLabs** | Voice synthesis for Jarvis | Free tier, then paid |
| **Claude** | The "background builder" Julian uses to improve the system and as one of the agents in the team | API or subscription |
| **Glass Box / Mission Control** | A web dashboard showing every agent, every queue, every memory store in one view | Bundled with Hermes |

The recurring tagline across the guides: "Hermes alone vs Hermes inside the Agent Operating System — shared memory, live swarm map, workspace, the team of other agents around it." That's his whole thesis — Hermes is the runtime, Agent OS is the orchestration layer around it.

---

## Architecture, in his own terms

From Julian's write-ups, the stack looks like this:

1. **Single OS shell**: Hermes Agent — the thing you talk to and that runs the tools. Same model as our setup.
2. **Sub-agents as personas**, each with a name and a job:
   - **Hermes** (the director)
   - **Jarvis** (voice)
   - **OpenClaw** (browser/external APIs)
   - **Oracle** (research)
   - **Paperclip** (team/swarm)
   - **Claude** (background builder, also available as a chat agent)
   - **Gemini** (chat agent)
   - **Antigravity** (autonomous goal-mode)
3. **Memory substrate**: Obsidian vault, GitHub-backed, same shape as the AOS vault at `~/AOS/workspace/`.
4. **Voice I/O**: ElevenLabs TTS + a wake-word listener.
5. **Computer use**: Hermes can open apps, control browser, run shell commands.
6. **Mission Control**: a web dashboard that shows every agent's job state, queue, and memory store in one place — Julian calls it "the Glass Box."
7. **Self-driving Kanban**: a kanban board where the AI moves cards through triage → in-progress → done. The kanban doesn't just track work, it does the work.

---

## What's the same as what we've built

| Concept | Julian's Agent OS | Our AOS |
|---|---|---|
| OS shell | Hermes | Hermes |
| Agent roster | Hermes, Jarvis, OpenClaw, Oracle, Paperclip, Claude, Gemini | Hermes, codex, claude, oracle (M3), openclaw |
| Memory | Obsidian vault → GitHub | Obsidian vault → GitHub (same plugin) |
| Agent shape | Named sub-agents in profiles | FastAPI wrappers behind a router |
| Cross-agent handoff | Kanban + Glass Box | Router with tag-based dispatch + chains |
| Voice | ElevenLabs + wake word | not built |
| Browser control | OpenClaw | OpenClaw wrapper (built, not deployed) |
| Cloud backend | his Railway-equivalent | Railway (project id captured, not yet deployed) |

**The shape is identical.** His Agent OS and our AOS are the same architecture with different naming and one extra layer (Mission Control / Glass Box as a unified web dashboard, which we haven't built yet).

---

## What's different — and worth borrowing

1. **The Glass Box / Mission Control web dashboard.** Julian's agents all show up in one place. Our AOS only has the existing Hermes dashboard (`:9119`) plus `curl` to the router. **This is the missing piece for "use the OS via a web UI."** Should add a control-panel plugin to the Hermes dashboard that shows router health + dispatch buttons.

2. **Voice layer.** ElevenLabs + wake word. We don't have it. Optional — big effort.

3. **"Paperclip" — the team/swarm/kanban agent.** He has a dedicated name + UI for the multi-agent swarm. Our `/chain` endpoint does the same job without the name. Adding `paperclip` as a named alias is trivial.

4. **Oracle — long-context research persona.** Same shape as our `oracle` (MiniMax-M3). Rename `oracle` → `oracle` and add a research-oriented system prompt. Optional.

5. **Antigravity — goal mode.** Hermes has it built-in (long-running autonomous missions). Nothing to build — just expose it.

6. **Glass Box concept over Obsidian.** His memory is *browsable inside the OS dashboard*, not just on disk. We could add an Obsidian-viewer panel to our dashboard plugin.

7. **The brand / naming layer.** His whole pitch is "named agents you can talk to like Jarvis." Naming matters — it changes how you think about them. We're still calling ours "oracle" and "claude" — technical names. Renaming to "Hermes (director), Codex, Claude, Oracle (long-context), OpenClaw (web/browser), Glassbox (the OS view)" would match his UX.

---

## What's missing from his version that we have

- **None, materially.** His is built on top of Hermes (open source), Obsidian (free), OpenClaw (open source), Claude (subscription), ElevenLabs (paid) — all of which are accessible to you. The community/coaching layer (AI Profit Boardroom, $X/month) is the only thing he sells, and that's content not code.

---

## Three concrete things we could borrow today

1. **Build the Glass Box plugin** — a Hermes desktop plugin (~150 lines) that shows router `/health`, lets you POST to `/dispatch` from a button, and lists recent vault logs. ~2 hours.

2. **Add `paperclip` as a named profile** — same code as our router, just an alias for `/chain`. 5 min.

3. **Rename `oracle` → `oracle`** in `router/main.py` and `TAG_TO_AGENT`, and add a research-oriented system prompt to `agents/oracle/main.py`. Matches his terminology. 10 min.

---

## Recommendation

**Path 2 still holds.** Our AOS is the same architecture as Julian's Agent OS — Hermes + Obsidian + OpenClaw + named agents + kanban. The missing pieces are the dashboard (Glass Box) and the brand layer (named personas). Build those and you're functionally at parity with what Julian sells, with the bonus that you actually own the whole stack on your own machine and Railway.

**Sources:**
- https://agentos.guide/hermes-guides
- https://aisuccesslabjuliangoldie.com/blog/hermes-voice-agent/
- https://aisuccesslabjuliangoldie.com/blog/hermes-agent-use-cases/

— Hermes (this session, 2026-08-14)

---
name: avengers
description: Spawn parallel team of 10-20 subagents (wave-1 research + wave-2 synthesis) for big architecture/refactor/research tasks. Use when user types `/spawn-agents` or asks for "team agent" / "spawn 20 agents" / "research deeply with parallel agents". Each agent runs in own context window — protects main context from result flood. Default ladder: 10 research → synthesize → 4-6 writers → present plan.
---

# spawn-agents — Parallel Team Orchestrator

Spawn 10-20 subagents in parallel to handle ambitious research + design + synthesis tasks. Use for architecture upgrades, multi-source consolidation, build-vs-buy analyses, eval engineering work, platform design — anything where single-context exploration would burst main context.

## When to invoke

User triggers explicitly with `/spawn-agents` OR implies it via phrases like:
- "spawn team of N agents"
- "help spawn 20 agents to support this"
- "research deeply with parallel agents"
- "team agent in planning mode"

Also auto-invoke when about to do >5 parallel research lookups OR when the result of any single research thread is expected to exceed 50K tokens.

## Core protocol — 2-wave ladder

### Wave 1 — Research (parallel, 8-12 agents)

Each agent owns ONE narrow research thread. Sources: vendor docs, GitHub, Exa web search, Context7 MCP. Each agent must:
- Quote sources verbatim (≤15 words per quote, with URL)
- Return a structured report (under 2000 words)
- Mark unverifiable claims `[NO SOURCE FOUND]`
- Give a concrete verdict (ADOPT / BUY / BUILD / DEFER / REJECT) with 3-bullet rationale

Wave-1 thread budget: 1-3 agents per topic dimension. Example for a platform-architecture upgrade:
- Vendor A deep-dive (capabilities + pricing + deployment)
- Vendor A API reference (endpoint list + auth + schema)
- Vendor B deep-dive
- Vendor B API reference
- Standard X protocol (status + transport + auth + adopters)
- Best-practice patterns (rate limits, idempotency, observability)
- Build-vs-buy matrix (N capabilities × verdict per row)
- Design decision (e.g. ensemble shape, taxonomy split)
- Industry survey / benchmarks
- Anti-patterns + failure-mode catalog

### Wave 2 — Synthesis + Write (parallel, 4-6 agents)

After Wave 1 returns, synthesize key findings inline (table form) BEFORE dispatching Wave 2. Each Wave-2 agent receives the synthesized findings + writes specific artifacts:
- 1 agent — main mega-doc (~1200 lines, ~22 sections, machine-parseable task appendix)
- 1 agent — sub-docs bundle (4-6 detail docs covering one capability each, ~400 lines each)
- 1 agent — Excalidraw diagram (Excalidraw v2 JSON, 100-150 elements, validated parse)
- 1 agent — kanban delta plan (NEW/SUPERSEDES/DELETED breakdown, dependency edges, NO apply)
- Optional: 1 agent — decision-log compendium (DR-NNN entries per build-vs-buy choice)

## Hard rules

- Send all agents in ONE Agent-tool message — true parallel execution
- Each agent prompt must be self-contained (it has no conversation context)
- Tell each agent WHAT outputs to produce, WHERE to write them, WHICH inputs to read
- Cap individual agent report length (under 2000 words)
- NEVER let Wave-2 agents touch each other's files — assign distinct paths
- After Wave 2 returns, present compact summary to user + ask for approval before applying any irreversible action (kanban writes, git pushes, doc supersession)
- Vault writes default to allowed (reversible); kanban writes require explicit approval

## Output destinations

- Research notes: `/tmp/spawn-<topic>-<timestamp>.md` (ephemeral)
- Vault docs: `/Users/chaiwat/Documents/Obsidian Vault/<area>/<new-folder>/` (per global policy — no `.md` in project dir)
- Excalidraw: same vault folder as parent doc
- Kanban delta: write plan as `.md` in vault folder; apply via separate run after user approval

## Skill composition

Compose with `arch-improve-feedback` skill (which calls `/spawn-agents` after picking up feedback file). Compose with `arch-verify` skill (which validates the resulting plan).

## Anti-patterns

- Spawning 20 agents for a 1-file question — use direct tools instead
- Sequential agent calls when no dependency — kills parallelism win
- Letting agents share write paths — causes overwrite races
- Forgetting to summarize Wave-1 before dispatching Wave-2 — main context fills up
- Writing irreversible side-effects (git push, kanban apply, Drive deletes) without user approval

## Reference patterns

When user asks for a major architecture revision, the canonical Wave-1 ladder used in this project:

1. Galileo / Eval Engineering platform research (already done — reuse `radiant1-architecture-v4/02-eval-engineering.md`)
2. Portkey / AI Gateway research (already done — reuse `03-platform-layer.md`)
3. A2A protocol research (already done — reuse `06-a2a-marketplace.md`)
4. MCP server best practices (already done — reuse `05-mcp-strategy.md`)
5. Billing + quota systems (already done — reuse `04-billing-credits.md`)
6. Build-vs-buy matrix (already done — reuse `08-build-vs-buy.md`)
7. Multi-LLM judge architecture (already done — reuse mega-doc §7)
8. 3-class taxonomy / Skills/Retrieval/Specialist split (already done — reuse `07-3class-taxonomy.md`)

**For topics in this list — skip re-research, reference existing docs**. User said: "about tools or platform which have research enough already no need to research again."

Only spawn NEW research agents for genuinely new topics (e.g., new vendor not yet covered, new protocol version, new capability surface).

## Composition with arch-improve-feedback

When invoked from `arch-improve-feedback`:
1. Skip Wave-1 re-research for already-covered topics (per list above)
2. Wave-1 narrow scope to: (a) verify the feedback file's claims against best-practice 2026, (b) detect conflicts with current canonical mega-doc, (c) score improvement quality
3. Wave-2 produces: merged-next-version mega-doc + refreshed Excalidraw + kanban delta plan
4. Always preserve previous version (do not overwrite — version in filename or new folder)

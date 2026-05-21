---
name: skills-help
description: Quick reference card showing all active your project Claude Code skills, flow chains, and trigger keywords. One-shot display, not a persistent mode. Use when user types /skills-help, /handbook, or asks "what skills do I have", "show me skills", "skill list", "which skill for X", "how skills work".
when_to_use: "Keyword triggers — skills help, /skills-help, /handbook, what skills, show skills, skill list, list skills, which skill, what mantras, show mantras, skill flow, flow chain, what can claude do, how skills work, skill cheat sheet, quick reference, skill index, มี skill อะไร, skill ของฉัน, skill list ฉัน"
allowed-tools: "Read"
disable-model-invocation: false
---

# /skills-help — Quick handbook display

One-shot reference card. Display content below verbatim, then stop. No work performed.

---

## Output to user verbatim

> # mantra-skills — Quick Reference
>
> ## 27 active skills · 14 flow chains · 9arm + Karpathy + Matt Pocock
>
> NOTE: this card is stale — list below is from v1 (16 skills). Use marketplace README.md for current 27-skill roster.
>
> Full handbook: `/Users/chaiwat/Documents/Obsidian Vault/claude-code/08-skills-handbook-9arm-flow.md`
> Index: `~/.claude/skills/SKILLS-INDEX.md`
>
> ---
>
> ## Flow chains — pick by task type
>
> | Task | Chain |
> |---|---|
> | **NEW FEATURE** | `intake → plan-cap → tdd-stark → smoke-spidey → ship-rocket → note-kira` |
> | **BUG REPORT** | `intake → debug-sherlock → tdd-stark → smoke-spidey → ship-rocket → post-mortem → note-kira` |
> | **REFACTOR** | `intake → plan-cap(--refactor) → tdd-stark(--refactor) → smoke-spidey → ship` |
> | **CROSS-CUTTING** | `intake → plan-cap → avengers → (per-agent: tdd-implement → smoke) → ship` |
> | **RAG TUNE** | `rag-tune → tdd-stark → smoke-spidey → ship` |
> | **AGENT EVAL** | `agent-eval → tdd-stark (if SHIP) → smoke-spidey → ship` |
> | **TENANT AUDIT** | `tenant-leak-audit → tdd-stark (if leak) → smoke-spidey → ship` |
> | **DAILY OPS** | `daily-standup` (standalone) |
> | **QUESTION** | `intake → answer directly` |
>
> ---
>
> ## Engineering mantras (5)
>
> | Skill | Steps | Refuses without |
> |---|---|---|
> | **plan** | goal · search · 3 alts · risk · tasks | goal sentence (1 line) |
> | **tdd-implement** | RED · GREEN · REFACTOR · COVERAGE · handoff | failing test for right reason |
> | **debug-mantra** | repro · trace · falsify · breadcrumb | reliable repro |
> | **smoke** | curl · SSE · tenant · latency · handoff | infra pre-flight passed |
> | **ship** | gate · scrutinize · audit · push · sync | 7 gates green |
>
> ## Productivity mantras (3)
>
> | Skill | Steps | Refuses without |
> |---|---|---|
> | **intake** | classify · repro check · 7-step route | nothing (pure router) |
> | **post-mortem** | 9-section structure | repro + cause + fix + validated |
> | **take-note-ob** | detect · link · frontmatter · memory · sync | topic detected |
>
> ## your project-specific mantras (3 new)
>
> | Skill | Steps | Refuses without |
> |---|---|---|
> | **rag-tune** | baseline · hypothesis · probe · compare · ship/revert | golden eval set + baseline metrics |
> | **agent-eval** | golden · baseline run · variant run · compare · verdict | golden dataset + both variants deployable |
> | **tenant-leak-audit** | enumerate · classify · test · fix/justify · lint rule | target service path |
>
> ## Reference (path-routed)
>
> - **project-patterns** — Section A=ai-agent-backend, B=hotel-service, C=reservation-service, D=alfred-app, E=aspire-app, F=cross-cutting. Loads matching section by file path.
>
> ## Personal / utility
>
> - **spawn-agents** — parallel team dispatch (2+ independent tasks)
> - **graphify** — knowledge graph generation
> - **cua-driver** — native macOS GUI automation
> - **daily-standup** — morning team brief
>
> ---
>
> ## Trigger keywords (top examples)
>
> | Want to | Type / paste |
> |---|---|
> | Plan a feature | "plan X" / "add X" / "build X" |
> | Implement | "implement" / "write code" / "TDD" |
> | Debug | paste error / "broken" / "this weird" / "ทำไม error" |
> | Refactor | "refactor" / "clean up" / "extract function" |
> | Verify before deploy | "smoke" / "verify chat" / "test it" |
> | Commit + push | "ship" / "ready to ship" / "commit + push" |
> | Document fix | "post-mortem" / "RCA" |
> | Sync to vault | "take note" / "sync vault" / "จดบันทึก" |
> | Tune RAG | "rag" / "retrieval" / "rerank" / "tune retrieval" |
> | A/B agents | "agent eval" / "compare agent" / "sale v2" |
> | Tenant audit | "tenant audit" / "tenant leak" / "missing tenantId" |
> | Lookup file path | "where put file" / service name |
> | Spawn parallel | "spawn agents" / "fan out" / "parallel" |
> | This card | "/skills-help" / "what skills" / "show skills" |
>
> ## Override / escape
>
> | Situation | Action |
> |---|---|
> | Mantra too rigid for trivial fix | "skip mantra, trivial" |
> | Need direct answer, no flow | use `intake-jarvis` step 5 (question) |
> | GateGuard blocks setup | `export ECC_GATEGUARD=off` |
> | ECC skill crowds out user skill | see TRIM-ECC-PLAN.md option 2 |
> | Stuck in flow | "abort" |
>
> ---
>
> ## Universal rules (durable, enforced by skills)
>
> - **NEVER push main** — ship refuses
> - **NEVER auto-merge feature branches** — user merges
> - **ALWAYS sync vault after commit** — take-note-ob + PostToolUse hook
> - **ALWAYS lint + tsc + jest before commit** — ship gate refuses if red
> - **ALWAYS tenant-scope queries** — smoke step 3 + tenant-leak-audit
> - **NEVER bump MAX_* / timeout / recursion as fix** — debug-mantra step 3 anti-pattern
> - **NEVER write .md in project dir** — vault sync rule, take-note-ob routes correctly
>
> ---
>
> ## Want more detail?
>
> - Full handbook → `Obsidian Vault/claude-code/08-skills-handbook-9arm-flow.md`
> - Skill source → `~/.claude/skills/<name>/SKILL.md`
> - Archived 16 skills → `~/.claude/skills/_archived/`
> - ECC trim plan → `~/.claude/skills/_archived/TRIM-ECC-PLAN.md`

---

## Operating rules

- **One-shot display.** This skill does NOT do work. Display reference card + STOP.
- **No follow-up action.** If user wants to invoke another skill, they say so explicitly.
- **No editing.** Read-only skill, no file writes.

result: skills reference card displayed.

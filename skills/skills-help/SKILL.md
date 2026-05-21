---
name: skills-help
description: Quick reference card showing all 26 active mantra-skills, flow chains, and trigger keywords. One-shot display, not a persistent mode. Use when user types /skills-help, /handbook, or asks "what skills do I have", "show me skills", "skill list", "which skill for X", "how skills work".
when_to_use: "Keyword triggers — skills help, /skills-help, /handbook, what skills, show skills, skill list, list skills, which skill, what mantras, show mantras, skill flow, flow chain, what can claude do, how skills work, skill cheat sheet, quick reference, skill index, มี skill อะไร, skill ของฉัน, skill list ฉัน"
allowed-tools: "Read"
disable-model-invocation: false
---

# /skills-help — Quick handbook display

One-shot reference card. Display content below verbatim, then stop. No work performed.

---

## Output to user verbatim

> # mantra-skills — Quick Reference (v0.3)
>
> ## 26 active skills · 11 flow chains · 9arm + Karpathy + Matt Pocock
>
> Marketplace: https://github.com/kwchaiwat/mantra-skills
> Source: `~/.claude/skills/<name>/SKILL.md`
>
> ---
>
> ## Flow chains — pick by task type
>
> **Legend:**
> - `STD-tail` = `plan-cap → tdd-stark → smoke-spidey → ship-rocket → note-kira`
> - `[brackets]` = optional middle-step (fires if condition matches)
>
> | Purpose | Entry → ... → tail |
> |---|---|
> | **FEATURE** (clear or vague) | `jarvis [→ grill-coach if vague] → STD-tail` |
> | **BUG REPORT** | `jarvis → debug-sherlock → tdd-stark → smoke-spidey → ship-rocket → post-mortem → note-kira` |
> | **REFACTOR** | `jarvis [→ zoom-cerebro to map first] → plan-cap(--refactor) → tdd-stark(--refactor) → smoke-spidey → ship-rocket` |
> | **ARCH HEALTH** (weekly) | `arch-yoda → REFACTOR chain (if drift/debt)` |
> | **SPIKE / POC** | `prototype-mcguyver → FEATURE chain (if keep) OR discard` |
> | **CROSS-CUTTING** (3+ services) | `jarvis → plan-cap → avengers → (per-agent: tdd-stark → smoke-spidey) → ship-rocket` |
> | **TUNE / EVAL / AUDIT** | `<rag-tune \| agent-eval-vision \| tenant-leak-audit> → tdd-stark → smoke-spidey → ship-rocket` |
> | **PRE-COMMIT REVIEW** | `scrutinize-falcon → terminal (verdict ship/fix/rework/reject)` |
> | **SESSION HANDOFF** | `handoff-coulson → terminal (vault note)` |
> | **RESEARCH** | `research-strange → terminal (cited brief)` |
> | **DAILY OPS** | `daily-standup → terminal` |
>
> ---
>
> ## Engineering mantras (9)
>
> | Skill | Steps | Refuses without |
> |---|---|---|
> | **plan-cap** | goal · search · 3 alts · risk · tasks | goal sentence |
> | **tdd-stark** | RED · GREEN · REFACTOR · COVERAGE · handoff | failing test |
> | **debug-sherlock** | repro · trace · falsify · breadcrumb | reliable repro |
> | **smoke-spidey** | curl · SSE · tenant · latency · handoff | pre-flight green |
> | **ship-rocket** | gate · scrutinize · audit · push · sync | 7 gates green · NEVER push main |
> | **scrutinize-falcon** | intent · trace · verify · findings · verdict | target artifact + stated goal |
> | **zoom-cerebro** | target · callers · callees · cross-cutting · map | concrete target |
> | **arch-yoda** | target · hot spots · smells · rank · verdict | target dir |
> | **prototype-mcguyver** | hypothesis · criterion · relaxed · budget · verdict | testable hypothesis |
>
> ## Productivity mantras (5)
>
> | Skill | Steps | Refuses without |
> |---|---|---|
> | **jarvis** | scan · classify · bug-gate · score · chain · declare · execute | nothing (router · replaces fury + intake-jarvis) |
> | **grill-coach** | restate · interpret · accept · out-of-scope · confirm | actual feature ask |
> | **post-mortem** | 9-section structure | repro + cause + fix + validated |
> | **note-kira** | detect · link · frontmatter · memory · sync | topic detected |
> | **handoff-coulson** | scope · facts · decisions · pending · briefing | non-empty session |
>
> ## Domain-specific (3 — adapt to your stack)
>
> | Skill | Refuses without |
> |---|---|
> | **rag-tune** | golden eval set + baseline metrics |
> | **agent-eval-vision** | golden dataset + both variants deployable |
> | **tenant-leak-audit** | target service path |
>
> ## Reference + meta (5)
>
> - **karpathy-rules** — 4 universal LLM coding principles (cross-referenced in 5 mantras)
> - **project-patterns** — path-routed conventions · Section A-F · §G CONTEXT.md pattern
> - **skills-help** — this card
> - **forge-stark** — meta · build new skill (auto-updates README + SKILLS-INDEX)
> - **research-strange** — multi-source cited brief
>
> ## Utility (4)
>
> - **avengers** — parallel team dispatch (2+ independent tasks)
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
> | Auto-route any request | just type naturally — `jarvis` fires |
> | Plan a feature | "plan X" / "add X" / "build X" |
> | Alignment interview | "what do you mean" / "clarify scope" / vague ask |
> | Implement | "implement" / "write code" / "TDD" |
> | Pre-edit blast radius | "who uses X" / "before I touch" / "zoom out" |
> | Refactor health check | "code smell scan" / "refactor candidates" / weekly arch |
> | Throwaway POC | "prototype" / "spike" / "POC" / "feasibility" |
> | Debug | paste error / "broken" / "not working" / "ทำไม error" |
> | Verify before deploy | "smoke" / "verify chat" / "test it" |
> | Pre-commit review | "review this PR" / "second opinion" / "before commit" |
> | Commit + push | "ship" / "ready to ship" / "commit + push" |
> | Document fix | "post-mortem" / "RCA" |
> | Session handoff | "context full" / "save for later" / "compact session" |
> | Sync to vault | "take note" / "sync vault" / "จดบันทึก" |
> | Tune RAG | "rag" / "retrieval" / "rerank" / "tune retrieval" |
> | A/B agents | "compare agent" / "agent eval" |
> | Tenant audit | "tenant audit" / "tenant leak" / "missing tenantId" |
> | Research w/ sources | "research" / "compare options" / "cite sources" |
> | Lookup file path | "where put file" / service name |
> | Spawn parallel | "spawn agents" / "fan out" / "parallel" |
> | This card | "/skills-help" / "what skills" / "show skills" |
>
> ## Override / escape
>
> | Situation | Action |
> |---|---|
> | Trivial fix · skip mantra | "skip mantra, trivial" |
> | Direct answer · no flow | "skip" |
> | Wrong skill fired | "redirect to <skill>" |
> | Stuck in flow | "abort" |
>
> ---
>
> ## Universal rules (durable, enforced by skills)
>
> - **NEVER push main** — `ship-rocket` refuses
> - **NEVER auto-merge** — user merges feature branches
> - **ALWAYS sync vault after commit** — `note-kira` + PostToolUse hook
> - **ALWAYS lint + tsc + jest before commit** — `ship-rocket` gate refuses if red
> - **ALWAYS tenant-scope queries** — `smoke-spidey` step 3 + `tenant-leak-audit`
> - **NEVER bump MAX_* / timeout / recursion as fix** — `debug-sherlock` step 3 anti-pattern
> - **NEVER write .md in project dir** — vault sync rule, `note-kira` routes correctly
> - **AskUserQuestion preferred** for pick-1-of-N (plan-cap step 3 · grill-coach step 2 · scrutinize verdict · jarvis low-conf · prototype hypothesis)
> - **Karpathy 4 principles** cross-referenced in plan-cap step 1 · tdd-stark step 3 · ship-rocket step 2 · debug-sherlock step 3 · jarvis step 3
>
> ---
>
> ## What changed v0.3
>
> - `fury` + `intake-jarvis` merged → `jarvis` (one entry-point router · 7-step mantra)
> - 4 new Matt Pocock skills: `grill-coach` · `zoom-cerebro` · `arch-yoda` · `prototype-mcguyver`
> - `karpathy-rules` cross-referenced in 5 engineering mantras
> - `scrutinize-falcon` added (standalone pre-commit review)
> - `handoff-coulson` added (session compact → vault note)
> - §G CONTEXT.md ubiquitous-language pattern in project-patterns
> - AskUserQuestion preferred path in 5 pick-1-of-N skills
> - Roster: 16 → 26 active · 7 → 11 flow chains (collapsed tail-duplicates)
>
> ## Want more detail?
>
> - Marketplace: https://github.com/kwchaiwat/mantra-skills
> - Skill source: `~/.claude/skills/<name>/SKILL.md`

---

## Operating rules

- **One-shot display.** This skill does NOT do work. Display reference card + STOP.
- **No follow-up action.** If user wants to invoke another skill, they say so explicitly.
- **No editing.** Read-only skill, no file writes.

result: skills reference card displayed (26 active · v0.3 · post-merge).

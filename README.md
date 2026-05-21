# mantra-skills

26 numbered-mantra Claude Code skills.

Each skill recites a verbatim mantra (1→2→3→4) with refuse-without gates and explicit hand-off chains. No skipping steps. No silent assumptions.

Inspired by [thananon/9arm-skills](https://github.com/thananon/9arm-skills) (tiny-set workflow) + [Andrej Karpathy's LLM coding principles](https://github.com/multica-ai/andrej-karpathy-skills) (Think Before · Simplicity First · Surgical Changes · Goal-Driven) + [mattpocock/skills](https://github.com/mattpocock/skills) (grill-before-coding · zoom-out · design-every-day · throwaway-prototype · ubiquitous-language).

## Install

```bash
# As Claude Code plugin
claude /plugin install mantra-skills@kw-chaiwat

# OR symlink directly
git clone https://github.com/kw-chaiwat/mantra-skills.git
ln -s "$(pwd)/mantra-skills/skills"/* ~/.claude/skills/
```

Verify: `/skills-help` → should list 26 skills.

## Quick start

```
/jarvis                   # auto-route entry point — picks chain · classifies · bug gate (was fury + jarvis · now merged)
/grill-coach "add X"      # alignment interview · 3 interpretations · acceptance
/plan-cap "add feature X" # pre-code planning · 3 alternatives · risk table
/zoom-cerebro <target>    # blast-radius map before editing critical file
/prototype-mcguyver       # throwaway POC · ONE hypothesis · time-budgeted
/debug-sherlock           # debug discipline · reproduce → falsify → breadcrumb
/tdd-stark                # test-first impl · RED → GREEN → REFACTOR
/scrutinize-falcon        # pre-commit second opinion · severity-ranked findings
/smoke-spidey             # post-impl verify · curl + leak check + tenant
/ship-rocket              # pre-merge gate · refuse-without-checklist
/arch-yoda <dir>          # weekly architecture health check
/handoff-coulson          # session compact → next-pilot briefing
/note-kira                # vault sync · canonical record
/skills-help              # quick reference card
```

## Roster (26 skills)

### Engineering mantras (9)
- **plan-cap** — Captain America strategist · pre-code planning · 5-step mantra
- **tdd-stark** — Iron Man iter · test-first impl · RED → GREEN → REFACTOR
- **debug-sherlock** — Holmes deduction · reproduce · trace · falsify · breadcrumb
- **smoke-spidey** — spider-sense · pre-deploy verify · curl + SSE + tenant + latency
- **ship-rocket** — Rocket Raccoon launch · refuse-without-checklist · NEVER push main
- **scrutinize-falcon** — Falcon scouts from above · outsider PR review BEFORE commit · intent · trace · verify · findings · verdict
- **zoom-cerebro** — Cerebro neighborhood scan · pre-edit blast-radius map · target · callers · callees · cross-cutting · verdict
- **arch-yoda** — Yoda meditates architecture · weekly refactor health · churn · smells · rank · verdict
- **prototype-mcguyver** — MacGyver throwaway POC · ONE hypothesis · time-budgeted · constraints relaxed · verdict

### Productivity mantras (5)
- **jarvis** — Iron Man's JARVIS · single entry-point router + classifier + bug-repro gate · 7-step mantra (replaces former `fury` + `jarvis`)
- **grill-coach** — Coach interrogates pre-code · restate · 3 interpretations · acceptance · out-of-scope · confirm
- **post-mortem** — canonical bug record · 9-section structure · slip-through analysis
- **note-kira** — Death Note records · Obsidian vault sync
- **handoff-coulson** — Agent Coulson briefs next pilot · session compact → handoff doc · scope · facts · decisions · pending · briefing

### Domain-specific mantras (3 — adapt to your stack)
- **rag-tune** — Azure AI Search + Cohere rerank + CRAG tuning (adopt template, swap vendors)
- **agent-eval-vision** — Vision verdict · multi-agent A/B (adopt template, swap framework)
- **tenant-leak-audit** — multi-tenant isolation audit (generic SQL/ORM)

### Reference + meta (5)
- **karpathy-rules** — 4 universal LLM coding principles
- **project-patterns** — path-routed service conventions template (customize per repo)
- **skills-help** — one-shot reference card display
- **forge-stark** — Tony Stark workshop · meta-skill to build new skills
- **research-strange** — Doctor Strange · multi-source research brief

### Utility (4)
- **avengers** — "assemble" parallel team dispatch
- **graphify** — knowledge graph generator
- **cua-driver** — macOS GUI automation
- **daily-standup** — morning team brief

## Philosophy — 5 core ideas

1. **9arm tiny set** — small skill count covering full workflow
2. **Numbered mantra recital** — verbatim at session start, no skipping steps
3. **Refuse-without gates** — hard refuse at step 1 if required inputs missing
4. **Explicit handoffs** — every skill ends with arrow to next skill
5. **Karpathy 4 principles** — universal LLM coding discipline applied across mantras

## Flow chains

```
NEW FEATURE:        jarvis → plan-cap → tdd-stark → smoke-spidey → ship-rocket → note-kira
GRILL-FIRST:        grill-coach → plan-cap → tdd-stark → smoke-spidey → ship-rocket → note-kira   (when ask vague)
BUG REPORT:         jarvis → debug-sherlock → tdd-stark → smoke-spidey → ship-rocket → post-mortem → note-kira
REFACTOR:           jarvis → plan-cap(--refactor) → tdd-stark(--refactor) → smoke-spidey → ship-rocket
PRE-EDIT MAP:       zoom-cerebro → tdd-stark OR plan-cap   (before touching critical/shared files)
WEEKLY ARCH:        arch-yoda → plan-cap --refactor → tdd-stark → ship-rocket   (architecture health)
SPIKE / POC:        prototype-mcguyver → plan-cap (if keep) OR discard
CROSS-CUTTING:      jarvis → plan-cap → avengers → (per-agent: tdd-stark → smoke-spidey) → ship-rocket
RAG TUNE:           rag-tune → tdd-stark → smoke-spidey → ship-rocket
AGENT EVAL:         agent-eval-vision → tdd-stark → smoke-spidey → ship-rocket
TENANT AUDIT:       tenant-leak-audit → tdd-stark (if leak) → smoke-spidey → ship-rocket
AD-HOC REVIEW:      scrutinize-falcon → terminal (ship / fix / rework / reject)
SESSION HANDOFF:    handoff-coulson → terminal (vault note for next session)
RESEARCH:           research-strange (terminal — brief w/ cited evidence)
META BUILD:         forge-stark (meta — build new skill)
DAILY OPS:          daily-standup (standalone)
```

## Customize to your project

Skill bodies contain example references from the original author's project (multi-tenant SaaS w/ NestJS microservices + LLM agent backend + React/Next frontends). These examples illustrate patterns — adapt to YOUR stack.

**Three things to customize after fork:**

1. **`project-patterns/SKILL.md`** — replace path-routed service sections w/ YOUR services
2. **Domain skills** (`rag-tune`, `agent-eval-vision`, `tenant-leak-audit`) — swap vendor names + failure-mode catalogs
3. **`debug-sherlock` failure-mode catalog** — replace bug-pattern table with your recurring bugs

See [CUSTOMIZATION.md](./CUSTOMIZATION.md) for step-by-step fork guide.

## Why this works

- **Workflow > references.** Mantras teach HOW to work, not WHAT each tech is.
- **Cross-stack portable.** TDD discipline + plan-3-alts + scrutinize-before-ship apply to any codebase.
- **Refuse-without surfaces drift.** When AI doesn't have what it needs, it stops instead of hallucinating.
- **Mantra recital = no silent skipping.** Step 1 recited verbatim means LLM commits to discipline visibly.

## Credits

- Workflow design — [thananon/9arm-skills](https://github.com/thananon/9arm-skills) (tiny-set · numbered mantra · handoff arrows)
- LLM coding principles — [Andrej Karpathy](https://x.com/karpathy/status/2015883857489522876) via [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) (Think Before · Simplicity First · Surgical Changes · Goal-Driven)
- Process patterns — [mattpocock/skills](https://github.com/mattpocock/skills) 98k⭐ (grill-before-coding · zoom-out · design-every-day · throwaway-prototype · ubiquitous-language)
- Hero naming — Marvel / DC household names matched to power

## License

MIT

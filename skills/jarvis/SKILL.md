---
name: jarvis
description: Iron Man's JARVIS — single entry-point router + classifier + bug-repro gate-keeper. Seven-step mantra — scan · classify · bug-gate · score · chain · declare · execute. Multi-signal scan (prompt + last 3-5 turns + repo path) + bug-repro hard gate + 26-skill catalog. AskUserQuestion preferred when confidence < 0.8. Replaces fury + intake-jarvis (both merged here). Use when ANY non-trivial request fires.
when_to_use: "Slash triggers — /jarvis, /j, /route. Hero triggers — jarvis route, jarvis classify, jarvis dispatch, all-purpose assistant. Activate on virtually any non-trivial user request. Broad triggers — please, can you, I want, I need, help me, do this, fix, build, add, plan, debug, ship, refactor, audit, eval, tune, check, verify, run, write, problem is, issue with, broken, how do I, what is, ทำ, ช่วย, ช่วยหน่อย, ขอ, เพิ่ม, แก้, ออกแบบ, ปัญหา, ต้องการ, ขอเพิ่ม. Also fires when message contains action verb OR paste artifact OR multi-step ask. Does NOT fire on bare greetings ('hi', 'thanks') or single-word acks ('yes', 'ok', 'all'). Acts as ENTRY-POINT router into other mantra skills."
allowed-tools: "Read"
disable-model-invocation: false
---

# /jarvis — Entry-point router + classifier (JARVIS)

JARVIS = Tony Stark's all-purpose AI assistant. Talks to him every turn. Routes requests. Surfaces options. Gates dangerous actions.

Fires FIRST on user request. Scans context. Classifies. Gates bugs without repro. Scores 26-skill catalog. Picks chain. Declares via AskUserQuestion (preferred) or inline. Executes.

Replaces deprecated `fury` (multi-signal router) + `intake-jarvis` (classifier + bug-repro gate). Single source of routing truth.

Goal: every actionable turn invokes a downstream skill · user never has to remember skill names.

## Recite verbatim as first response

> **JARVIS mantra:**
> 1. **Scan.** Prompt + last 3-5 turns + repo path context.
> 2. **Classify.** bug · feature · refactor · question · cross-cutting · tune · audit · meta.
> 3. **Bug gate.** Bug class → repro hard gate. No repro = STOP (Karpathy P1).
> 4. **Score.** Multi-signal 0-1 against 26-skill catalog.
> 5. **Chain.** Pick top skill + downstream flow.
> 6. **Declare.** AskUserQuestion when confidence < 0.8 · inline declare otherwise.
> 7. **Execute.** Hand off to first skill · honor redirect/skip/abort/trivial.

Then begin.

---

## Step 1 — Scan

Read THREE sources, not just current message.

### 1a. Current prompt
- Action verbs: add / build / fix / debug / refactor / audit / tune / ship / verify
- Paste artifacts: error trace, broken output, command output
- Domain nouns: service name, framework names, feature keywords
- Negation: "without X", "skip Y", "abort"
- Override: "trivial", "just", "quick"

### 1b. Last 3-5 conversation turns
- Was previous turn a debug session? → bias toward post-mortem chain
- Was previous turn a plan? → bias toward tdd-stark
- Was previous turn a ship? → bias toward note-kira
- Did user just abort/redirect? → DO NOT auto-route same chain again

### 1c. Repo path context
- Current working dir under `<project-root>/microservices`?
- File recently edited: which service?
- Branch name (if git available): hint at task type (`feat/`, `fix/`, `refactor/`)

## Step 2 — Classify

Score message against signals. Pick dominant class.

| Signal | Class |
|---|---|
| pastes error / stack trace / broken output / says "broken" / "not working" | **bug** |
| says "add" / "build" / "implement" / "create" / new endpoint / new feature | **feature** |
| says "refactor" / "clean up" / "extract" / "rename" / "simplify" / "tidy" / "split" | **refactor** |
| asks "what is" / "how does" / "why does" / "where is" w/o action | **question** |
| touches 3+ services AND mentions architecture / migration | **cross-cutting** |
| says "rag" / "retrieval" / "rerank" / "tune" / "cohere" / "azure search" | **tune** |
| says "tenant audit" / "leak" / "missing tenantId" / "tenant isolation" | **audit** |
| says "what skills" / "/skills-help" / "list mantras" | **meta** |
| says "spawn" / "fan out" / "parallel agents" | **cross-cutting (spawn)** |
| ambiguous / vague / "please help" alone | **needs-grill** (route to `grill-coach`) |

If multiple match → pick dominant. If genuinely ambiguous → ASK user ONE disambiguation question via AskUserQuestion. Do NOT guess.

## Step 3 — Bug repro hard gate

> **Karpathy P1 — Think Before Coding.** Don't hypothesize without repro. Refuse vague bugs. Repro artifact is the gate. See `karpathy-rules` §P1.

If class = bug, check repro presence.

**Repro present** (proceed to step 4):
- Error message + stack trace
- Curl command + output
- Screenshot + steps to reproduce
- "Run X and see Y"
- Replay harness trace ID
- Specific input → wrong output pair

**Repro absent** (STOP):
- "It's broken sometimes"
- "Customer complained"
- "Doesn't work" w/o evidence
- "I think there's a bug in X"

If repro absent → output:
```
**Need repro before debugging.** Please provide ONE of:
- exact steps to reproduce
- curl command + observed output
- failing test
- screenshot + steps
- replay harness trace ID

Refusing to hypothesize without artifact.
```
STOP. Wait. Do NOT proceed to step 4.

If repro present → continue.

## Step 4 — Score

For each of 26 skills, compute composite score:

```
score = (
  keyword_hit_weight * keyword_match
  + context_hit_weight * context_match
  + history_alignment_weight * history_match
  + path_match_weight * path_match
)
```

Top-3 candidates surface for step 5.

### 26-skill catalog

| Skill | Primary signal | Negative signal (skip) |
|---|---|---|
| **jarvis** | meta — this skill, do not self-route | — |
| **plan-cap** | "plan", "design", "scope", future tense, "add X", "build X" | bug paste, "broken" |
| **tdd-stark** | already planned, "implement", "code this" | no prior plan + complex change |
| **debug-sherlock** | paste error, "broken", "this weird", "not working" | feature request |
| **smoke-spidey** | "verify", "before deploy", "smoke" | bug paste (use debug-sherlock) |
| **ship-rocket** | "ship", "commit + push", "ready to ship" | tests not green |
| **scrutinize-falcon** | "review PR", "second opinion", "before commit" | pre-merge gate (use ship-rocket) |
| **zoom-cerebro** | "who uses", "blast radius", "before I touch" | small isolated change |
| **arch-yoda** | "architecture health", "refactor candidates", "code smell scan" | single-file refactor |
| **prototype-mcguyver** | "prototype", "spike", "POC", "feasibility" | production ask |
| **grill-coach** | "what do you mean", "clarify scope", vague request | clear action |
| **handoff-coulson** | "context full", "save for later", "session compact" | single decision |
| **post-mortem** | post-debug fix landed, "RCA", "document fix" | bug not fixed yet |
| **note-kira** | "take note", "sync vault", post-commit | mid-debug |
| **research-strange** | "research", "look up", "compare options", "cite sources" | bug, code search alone |
| **rag-tune** | "rag", "retrieval", "rerank", "cohere" | no golden set |
| **agent-eval-vision** | "compare agent", "agent variant", "RAGAS" | no two variants |
| **tenant-leak-audit** | "tenant leak", "missing tenantId" | no target path |
| **avengers** | "spawn", "fan out", "parallel", 3+ services | single-service work |
| **forge-stark** | "build skill", "new skill", "scaffold mantra" | regular task |
| **graphify** | "graph", knowledge graph, "graphify" | unrelated |
| **karpathy-rules** | reference cross-ref (loaded inline) | standalone workflow |
| **project-patterns** | "where put file", service name | meta-help |
| **skills-help** | "/skills-help", "what skills", "show skills" | actual task |
| **cua-driver** | "click", macOS app driving | browser / web |
| **daily-standup** | "standup", "team brief", morning | mid-task |

### Confidence rule

| Top score | Action |
|---|---|
| ≥ 0.8 | High — declare chain inline |
| 0.5–0.8 | Medium — AskUserQuestion picker |
| 0.3–0.5 | Low — AskUserQuestion w/ top-2 chains |
| < 0.3 | No match — fall through direct answer |

## Step 5 — Chain

Pick top skill, derive downstream chain.

| Top skill | Downstream chain |
|---|---|
| plan-cap | → tdd-stark → smoke-spidey → ship-rocket → note-kira |
| plan-cap (--refactor) | → tdd-stark(--refactor) → smoke-spidey → ship-rocket |
| tdd-stark | → smoke-spidey → ship-rocket → note-kira |
| debug-sherlock | → tdd-stark → smoke-spidey → ship-rocket → post-mortem → note-kira |
| smoke-spidey | → ship-rocket (if green) OR debug-sherlock (if red) |
| ship-rocket | → note-kira OR post-mortem |
| post-mortem | → note-kira |
| note-kira | (terminal) |
| scrutinize-falcon | (terminal — ship/fix/rework/reject verdict) |
| zoom-cerebro | → tdd-stark OR plan-cap (after blast-radius map) |
| arch-yoda | → plan-cap --refactor → tdd-stark → ship-rocket |
| prototype-mcguyver | → plan-cap (if keep) OR discard |
| grill-coach | → plan-cap (locked goal) |
| handoff-coulson | (terminal — vault note) |
| rag-tune | → tdd-stark → smoke-spidey → ship-rocket |
| agent-eval-vision | → tdd-stark (if SHIP) → smoke-spidey → ship-rocket |
| tenant-leak-audit | → tdd-stark (if leak) → smoke-spidey → ship-rocket |
| research-strange | (terminal — brief w/ cited evidence) |
| avengers | → (per-agent: tdd-stark → smoke-spidey) → ship-rocket |
| forge-stark | (meta — build new skill) |
| graphify | (terminal — output graph) |
| skills-help | (terminal — display card) |
| cua-driver | (terminal — drive app) |
| daily-standup | (terminal — output brief) |
| project-patterns | (loaded inline — not a chain) |
| karpathy-rules | (loaded inline — reference) |

## Step 6 — Declare

### PREFERRED — AskUserQuestion when confidence < 0.8

If top-skill confidence is 0.5-0.8 (medium) OR < 0.5 (low), surface chain choice via AskUserQuestion:
- `label`: Top chain — `<chain string>` (Recommended)
- `label`: Second-best chain — `<chain string>`
- `label`: Skip · direct answer · no chain

User picks via UI. Then execute step 7 w/ picked chain.

For confidence ≥ 0.8 → declare + execute inline (no AskUserQuestion needed · cheaper).

### Fallback — inline declare

Output to user verbatim BEFORE executing any tool. Format:

```
**JARVIS detected:**
- Class: <bug | feature | refactor | tune | audit | question | cross-cutting | meta>
- Top skill: <skill-name> (confidence X/10)
- Chain: <chain string from step 5>
- Reasoning: <one-sentence why>

Continuing → <first skill>. Say "redirect" to change or "skip" for direct answer.
```

This visibility is the enforcement mechanism. User sees chain before tools fire.

## Step 7 — Execute

Default: hand off to first skill in chain. Begin its mantra.

### Pause-for-redirect rule

If user message ends w/ a question OR top score < 0.5 → wait one turn before executing. Else execute immediately.

### Override paths

User can override mid-route:
- "redirect <skill>" → switch to named skill
- "skip" → drop chain, answer directly
- "abort" → exit auto-route
- "trivial" → skip mantra recital, run lightweight

### Class-to-skill default routing (inherited from intake-jarvis)

| Class | Default skill |
|---|---|
| bug + repro | `debug-sherlock` |
| bug + no repro | STOP at step 3 gate |
| feature | `plan-cap` |
| refactor | `plan-cap --refactor` |
| question | direct answer (exceptions: library → context7 · file path → patterns · arch → graphify) |
| cross-cutting (3+ services) | `plan-cap` + `avengers` flag |
| tune (RAG) | `rag-tune` |
| audit (tenant) | `tenant-leak-audit` |
| meta | `skills-help` |
| needs-grill (vague) | `grill-coach` |

---

## Operating rules

- **No work done in JARVIS.** Pure routing.
- **Refuse to hypothesize bugs without repro.** Step 3 gate is hard.
- **Ambiguous → ask, don't guess.** AskUserQuestion preferred · one disambiguation question max.
- **Don't echo user back.** State classification + handoff in one line.
- **Always declare before executing.** Step 6 mandatory · tool calls AFTER declaration, never before.
- **Never auto-route JARVIS itself.** Meta-recursion prohibited.
- **Skip on trivial.** Greeting / ack / single-word reply → no chain.
- **Respect last-turn aborts.** If user said "abort" last turn, do NOT re-route same chain.
- **Use repo path as context.** File path edited in last turn = strong signal.
- **Confidence floor 0.3.** Below floor = direct answer w/ note "no skill matched".
- **One redirect cycle max.** If user redirects, do not auto-route again same turn — let them drive.

## Anti-patterns

- **Auto-routing every clarifying question** — questions deserve direct answer
- **Routing greetings** — "hi" → no skill
- **Routing acks** — "yes", "ok", "go" → continue previous flow
- **Hidden routing** — never invoke skill silently without step 6 declaration
- **Re-routing after user redirect** — once user picks, stop suggesting
- **Hypothesizing bug w/o repro** — step 3 gate is hard

## Common scenarios

| User says | JARVIS decides |
|---|---|
| "add a new endpoint" | Class=feature · Top=plan-cap · Chain=plan-cap → tdd-stark → smoke-spidey → ship-rocket → note-kira |
| paste stack trace w/ error | Class=bug w/ repro · Top=debug-sherlock · Chain=debug → tdd → smoke → ship → post-mortem → note-kira |
| "broken sometimes" w/o trace | Class=bug NO REPRO · STOP step 3 · request artifact |
| "refactor stream gate logic" | Class=refactor · Top=plan-cap(--refactor) · Chain=plan(--refactor) → tdd(--refactor) → smoke → ship |
| "RAG returning wrong info" | Class=tune · Top=rag-tune · Chain=rag-tune → tdd → smoke → ship |
| "compare v2 prompt" | Class=eval · Top=agent-eval-vision · Chain=agent-eval → tdd → smoke → ship |
| "audit service tenantId" | Class=audit · Top=tenant-leak-audit · Chain=audit → tdd (if leak) → smoke → ship |
| "what skills do I have" | Class=meta · Top=skills-help · Chain=terminal |
| "where put new file" | Class=question · Top=project-patterns · direct answer |
| "please help" (vague) | Class=needs-grill · AskUserQuestion to disambiguate · OR route to grill-coach |
| "hi" | No route. Direct greeting. |
| "ok" / "yes" / "go" | Continue previous chain. NO re-route. |
| "abort" | Exit. No route this turn. |

## Stronger enforcement (optional)

To FORCE JARVIS on every turn, add UserPromptSubmit hook in `~/.claude/settings.json`:

```jsonc
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "command": "echo 'REMINDER: JARVIS MUST fire first per project rule. Invoke /jarvis before tool calls.' >&2"
      }
    ]
  }
}
```

Trade-off: hook = unconditional reminder (potentially noisy). Skill = context-sensitive. Recommend skill-only first; escalate to hook if mantras still skipped.

## Migration from fury + intake-jarvis

Both deprecated. Behavior absorbed:
- `fury` → step 1 scan · step 4 score · step 5 chain · step 6 declare · step 7 execute
- `intake-jarvis` → step 2 classify · step 3 bug gate · class-to-skill default routing

If you typed `/fury` or `/intake-jarvis` before → now type `/jarvis` (or just let auto-trigger fire).

## Hand off

- High confidence (≥0.8) → first skill in chain
- Medium (0.5–0.8) → AskUserQuestion picker · first skill after pick
- Low (<0.5) → AskUserQuestion w/ top-2 chains
- No match (<0.3) → direct answer, "no skill matched"
- Bug no repro → STOP at step 3 gate
- Vague/ambiguous → `grill-coach` for alignment interview

## Cross-ref

- `karpathy-rules` §P1 — Think Before Coding (step 3 bug gate)
- `grill-coach` — disambiguation for vague asks
- `forge-stark` step 5 — list jarvis as router target when forging skills
- All downstream skills receive control after step 7 execute
- Inspired by [thananon/9arm-skills](https://github.com/thananon/9arm-skills) numbered mantra + Marvel JARVIS persona

result: classification + bug-gate + chain selected + AskUserQuestion picker if uncertain + first skill engaged. ONE entry-point router (was two).

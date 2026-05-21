---
name: fury
description: Entry-point skill router that fires FIRST on any user request. Scans current prompt + last 3-5 conversation turns + repo file context to pick the right skill chain. Declares the chosen chain to user verbatim before executing. Five-step mantra — scan · score · chain · declare · execute. Replaces ad-hoc skill selection w/ context-aware enforced routing. Use when user wants every turn to invoke a skill instead of unstructured response.
when_to_use: "Activate on virtually any non-trivial user request. Broad triggers include — please, can you, I want, I need, help me, do this, fix, build, add, plan, debug, ship, refactor, audit, eval, tune, check, verify, run, write, ทำ, ช่วย, ขอ, เพิ่ม, แก้, ออกแบบ. Also fires when user message contains action verb OR paste artifact OR multi-step ask. Does NOT fire on bare greetings ('hi', 'thanks') or single-word acks ('yes', 'ok', 'all'). Acts as router into other mantra skills."
allowed-tools: "Read"
disable-model-invocation: false
---

# /auto-route — Entry-point skill router

Fires FIRST on user request. Scans context. Picks chain. Declares chain. Executes.

Goal: ensure every actionable turn invokes a downstream skill — no unstructured ad-hoc responses to bug reports / feature requests / refactors / etc. User does not need to remember skill names.

## Recite verbatim as first response

> **Auto-route mantra:**
> 1. **Scan.** Current prompt + last 3-5 turns + repo path context.
> 2. **Score.** Match against 17-skill catalog (keyword + context + history).
> 3. **Chain.** Pick top skill + downstream flow from flow table.
> 4. **Declare.** Output chosen chain to user verbatim: `Chain: A → B → C. Confidence: X/10.`
> 5. **Execute.** Hand off to first skill OR wait for user redirect (≤5s pause for "no" override).

Then begin.

---

## Step 1 — Scan

Read THREE sources, not just current message:

### 1a. Current prompt
- Action verbs: add / build / fix / debug / refactor / audit / tune / ship / verify
- Paste artifacts: error trace, broken output, SSE garbage, command output
- Domain nouns: agent, RAG, tenant, hotel, reservation, alfred, aspire, langgraph
- Negation: "without X", "skip Y", "abort"
- Override: "trivial", "just", "quick"

### 1b. Last 3-5 conversation turns
- Was previous turn a debug session? → bias toward post-mortem chain
- Was previous turn a plan? → bias toward tdd-implement
- Was previous turn a ship? → bias toward take-note-ob
- Did user just abort/redirect? → DO NOT auto-route same chain again

### 1c. Repo path context
- Current working dir: `<project-root>/microservices`?
- File recently edited: which service?
- Branch name (if git available): hint at task type (`feat/`, `fix/`, `refactor/`)

## Step 2 — Score

For each of 17 skills, compute composite score:

```
score = (
  keyword_hit_weight * keyword_match
  + context_hit_weight * context_match
  + history_alignment_weight * history_match
  + path_match_weight * path_match
)
```

Top-3 candidates surface for step 3.

### Skill catalog (current 17)

| Skill | Primary signal | Negative signal (skip) |
|---|---|---|
| **auto-route** | meta — this skill, do not self-route | — |
| **intake** | ambiguous request needing classification | clear single-verb action |
| **plan** | "plan", "design", "scope", future tense, "add X", "build X" | bug paste, "broken" |
| **tdd-implement** | already planned, "implement", "code this", "ลุย" | no prior plan + complex change |
| **debug-mantra** | paste error, "broken", "this weird", "not working" | feature request |
| **smoke** | "verify", "before deploy", "smoke" | bug paste (use debug-mantra) |
| **ship** | "ship", "commit + push", "ready to ship" | tests not green |
| **post-mortem** | post-debug fix landed, "RCA", "document fix" | bug not fixed yet |
| **take-note-ob** | "take note", "sync vault", post-commit | mid-debug |
| **rag-tune** | "rag", "retrieval", "rerank", "cohere" | no golden set |
| **agent-eval** | "compare agent", "sale v2", "RAGAS" | no two variants |
| **tenant-leak-audit** | "tenant leak", "missing tenantId" | no target path |
| **spawn-agents** | "spawn", "fan out", "parallel", 3+ services | single-service work |
| **graphify** | "graph", knowledge graph, "graphify" | unrelated |
| **cua-driver** | "click", macOS app driving | browser / web |
| **daily-standup** | "standup", "team brief", morning | mid-task |
| **skills-help** | "/skills-help", "what skills", "show skills" | actual task |
| **project-patterns** | "where put file", service name | meta-help |

## Step 3 — Chain

Pick top skill, then derive downstream chain from flow table:

| Top skill | Downstream chain |
|---|---|
| intake | → debug-sherlock OR plan OR direct answer (intake decides) |
| plan | → tdd-stark → smoke-spidey → ship-rocket → note-kira |
| plan (--refactor) | → tdd-stark(--refactor) → smoke-spidey → ship-rocket |
| tdd-implement | → smoke-spidey → ship-rocket → note-kira |
| debug-mantra | → tdd-stark → smoke-spidey → ship-rocket → post-mortem → note-kira |
| smoke | → ship-rocket (if green) OR debug-mantra (if red) |
| ship | → note-kira OR post-mortem |
| post-mortem | → note-kira |
| take-note-ob | (terminal) |
| rag-tune | → tdd-stark → smoke-spidey → ship-rocket |
| agent-eval | → tdd-stark (if SHIP) → smoke-spidey → ship-rocket |
| tenant-leak-audit | → tdd-stark (if leak) → smoke-spidey → ship-rocket |
| spawn-agents | → (per-agent: tdd-implement → smoke) → ship-rocket |
| skills-help | (terminal — display card) |
| cua-driver | (terminal — drive app) |
| graphify | (terminal — output graph) |
| daily-standup | (terminal — output brief) |
| project-patterns | (loaded inline — not a chain) |

### Confidence rule

| Top score | Action |
|---|---|
| ≥ 0.8 | High confidence — proceed, declare chain |
| 0.5–0.8 | Medium — declare chain w/ "if not match, say redirect" |
| 0.3–0.5 | Low — declare top-2 chains, ask user to pick |
| < 0.3 | No match — fall through to direct answer, state "no skill matched" |

## Step 4 — Declare

Output to user verbatim BEFORE executing any tool. Format:

```
**Auto-route detected:**
- Class: <feature | bug | refactor | tune | audit | question | cross-cutting>
- Top skill: <skill-name> (confidence X/10)
- Chain: <chain string from step 3>
- Reasoning: <one-sentence why>

Continuing → <first skill>. Say "redirect" to change or "skip" for direct answer.
```

This visibility is the enforcement mechanism. User sees the chain before tools fire.

## Step 5 — Execute

Default: hand off to first skill in chain. Begin its mantra.

### Pause-for-redirect rule

If user message ends w/ a question OR top score < 0.5 → wait one turn before executing. Else execute immediately.

### Override paths

User can override mid-route:
- "redirect <skill>" → switch to named skill
- "skip" → drop chain, answer directly
- "abort" → exit auto-route
- "trivial" → skip mantra recital, run lightweight

---

## Operating rules

- **Always declare before executing.** Step 4 is mandatory. Tool calls AFTER declaration, never before.
- **Never auto-route auto-route.** Meta-recursion prohibited.
- **Skip on trivial.** Greeting / ack / single-word reply → no chain.
- **Respect last-turn aborts.** If user said "abort" last turn, do not re-route same chain.
- **Use repo path as context.** File path edited in last turn = strong signal.
- **Confidence floor 0.3.** Below floor = direct answer w/ note "no skill matched, answering directly".
- **One redirect cycle max.** If user redirects, do not auto-route again same turn — let them drive.

---

## Anti-patterns

- **Auto-routing every clarifying question** — questions deserve direct answer, not skill chain
- **Routing greetings** — "hi" → no skill
- **Routing acks** — "yes", "ok", "all", "go" → continue previous flow, no new route
- **Hidden routing** — never invoke skill silently without step 4 declaration
- **Re-routing after user redirect** — once user picks, stop suggesting

---

## Common scenarios

| User says | Auto-route decides |
|---|---|
| "add a new endpoint to hotel-service" | Class=feature · Top=plan · Chain=plan → tdd-stark → smoke-spidey → ship-rocket → note-kira |
| paste: SSE chunks showing raw `{` fragments | Class=bug · Top=debug-mantra · Chain=debug → tdd → smoke-spidey → ship-rocket → post-mortem → note-kira |
| "refactor langgraph-agent.service.ts to extract gate logic" | Class=refactor · Top=plan(--refactor) · Chain=plan → tdd-stark(--refactor) → smoke-spidey → ship-rocket |
| "RAG returning wrong hotel info" | Class=tune · Top=rag-tune · Chain=rag-tune → tdd → smoke-spidey → ship-rocket |
| "compare new sale prompt vs current" | Class=eval · Top=agent-eval · Chain=agent-eval → tdd → smoke-spidey → ship-rocket |
| "audit hotel-service for missing tenantId" | Class=audit · Top=tenant-leak-audit · Chain=audit → tdd → smoke-spidey → ship-rocket |
| "what skills do I have" | Class=meta · Top=skills-help · Chain=terminal |
| "where put new format template" | Class=question · Top=project-patterns Section D · direct answer |
| "hi" | No route. Direct greeting. |
| "ok" / "yes" / "go" | Continue previous chain, do NOT re-route. |
| "abort" | Exit, no route this turn. |

---

## Stronger enforcement (optional)

To FORCE auto-route on every turn (hard enforcement), add UserPromptSubmit hook in `~/.claude/settings.json`:

```jsonc
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "command": "echo 'REMINDER: auto-route MUST fire first per project rule. Invoke /auto-route before tool calls.' >&2"
      }
    ]
  }
}
```

This injects reminder text into every prompt. Skill-based enforcement (this file) is softer — relies on keyword match. Hook is hard.

Trade-off: hook = unconditional reminder (potentially noisy on trivial turns). Skill = context-sensitive (may skip when not applicable). Recommend skill-only first; escalate to hook if mantras still skipped.

---

## Hand off

- High confidence (≥0.8) → first skill in chain
- Medium (0.5–0.8) → first skill, w/ "redirect" hint
- Low (<0.5) → present top-2 chains, ask user to pick
- No match (<0.3) → direct answer, "no skill matched"

result: chain declared + first skill engaged + user given redirect window.

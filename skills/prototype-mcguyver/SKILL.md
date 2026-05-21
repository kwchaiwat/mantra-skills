---
name: prototype-mcguyver
description: MacGyver builds with what's at hand — throwaway prototype to test ONE hypothesis. No TDD · no lint · no tenant scoping · no error handling · hard time budget. Five-step mantra — hypothesis · success criterion · constraints relaxed · time budget · verdict. Inspired by mattpocock/skills "throwaway prototype". Refuses without testable hypothesis. NOT a ship vehicle — code gets thrown out.
when_to_use: "Slash triggers — /prototype-mcguyver, /prototype, /mcguyver, /spike. Hero triggers — mcguyver hack, throwaway spike, prototype it, proof of concept, POC. Keyword triggers — prototype, throwaway, spike, proof of concept, POC, hack together, can we even, feasibility, quick test, will this work, ลองดูก่อน, prototype ก่อน. Does NOT fire on — production ask (use plan-cap) · bug (use debug-sherlock) · ship (use ship-rocket) · 'prototype' that user actually wants to keep."
allowed-tools: "Bash(npm *) Bash(npx *) Bash(node *) Bash(ls *) Bash(cat *) Read Edit Write"
disable-model-invocation: false
---

# /prototype-mcguyver — Throwaway prototype (MacGyver)

MacGyver doesn't ship the chewing-gum bomb to prod. You don't ship the prototype either. Code exists to answer ONE question fast — then it gets deleted.

Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) "throwaway prototype". Use to test feasibility BEFORE committing to design.

## Recite verbatim as first response

> **Prototype mantra:**
> 1. **Hypothesis.** ONE testable claim. "Can X do Y in Z?"
> 2. **Success criterion.** ONE observable. Pass/fail unambiguous.
> 3. **Constraints relaxed.** No TDD · no lint · no tenant · no err handling.
> 4. **Time budget.** Hard ceiling. Default 30 min.
> 5. **Verdict.** Keep approach · discard · reshape · output learnings.

Then begin.

---

## Step 1 — Hypothesis

ONE testable claim. NO multi-clause.

### PREFERRED — AskUserQuestion when hypothesis vague

If user's hypothesis is vague (e.g. "test if X works"), surface 3 sharpened candidates via AskUserQuestion:
- `label`: sharper hypothesis 1
- `label`: sharper hypothesis 2
- `label`: sharper hypothesis 3 — narrowest (Recommended)

User picks. Then proceed step 2 (success criterion).

### Refuse-without

If no hypothesis → STOP. Output:
```
**Prototype refuses without hypothesis.**
Need ONE testable claim:
- "Can <X> achieve <Y> under <Z>?"
- "Will <approach A> beat <approach B> on <metric>?"
- "Does <library> work for <use case>?"
```

If user says "prototype the feature" → push back. That's a feature, not a hypothesis.
If user says "prototype the new auth flow" → push back. That's design work.

### Sharp hypothesis examples

| Bad | Good |
|---|---|
| "prototype RAG" | "Can Cohere v4 rerank improve hit@5 from 0.62 to 0.75 on golden set?" |
| "try Drizzle" | "Can Drizzle build 10-table schema in <2hr w/o type errors?" |
| "see if it's fast" | "Can endpoint X return <100ms p95 under 50 concurrent?" |

## Step 2 — Success criterion

ONE observable. Binary pass/fail.

```
HYPOTHESIS: <one sentence from step 1>
PASS WHEN: <one observable>
FAIL WHEN: <one observable>
```

Examples:
- PASS WHEN hit@5 ≥ 0.75 / FAIL WHEN < 0.75
- PASS WHEN p95 < 100ms / FAIL WHEN ≥ 100ms
- PASS WHEN compile errors = 0 / FAIL WHEN ≥ 1

Vague criteria ("feels faster") → REFUSE.

## Step 3 — Constraints relaxed

Prototype = relaxed constraints. Make explicit what's OFF.

### OFF (prototype-only)

- TDD — no failing test first
- Lint — `// @ts-nocheck` allowed
- Multi-tenant — single tenant hardcoded
- Error handling — happy path only
- Logging — `console.log` OK (prototype-only exception)
- Type safety — `any` allowed
- Code review — skip
- Documentation — skip

### ON (still required)

- No secrets in code (hardcoded API keys forbidden even in prototype)
- No production DB writes (use local DB or dry-run)
- No external side effects (no real emails · no real charges)
- Time budget honored (step 4)

Output:
```
RELAXED: TDD · lint · tenant · err handling · types · docs
KEPT: no secrets · no prod writes · no side effects · time budget
```

## Step 4 — Time budget

HARD ceiling. When timer rings → STOP regardless of progress.

```
BUDGET: <30 min | 1 hr | 2 hr | 4 hr>
START: <timestamp>
HARD STOP: <timestamp + budget>
```

Rules:
- Default 30 min — most hypotheses answerable in this window
- 1 hr — moderate complexity (new lib · new framework)
- 2 hr — heavy (major refactor proof)
- 4 hr MAX — beyond this, prototype is too big · split into smaller spikes

When timer rings:
- HYPOTHESIS ANSWERED → verdict in step 5
- HYPOTHESIS UNANSWERED → verdict = "inconclusive" · either split or invest more
- DON'T extend budget on first try · ringing means hypothesis too big

## Step 5 — Verdict

ONE of:

| Verdict | Meaning | Next |
|---|---|---|
| **keep** | Approach works · move to real impl | hand off plan-cap (NOT prototype code · re-implement properly) |
| **discard** | Doesn't work · try different approach | hand off research-strange OR new prototype |
| **reshape** | Partial · need different hypothesis | new prototype w/ refined claim |
| **inconclusive** | Budget hit · need more time OR split | redo w/ split hypothesis |

Output:
```
VERDICT: <keep | discard | reshape | inconclusive>
LEARNINGS: <3-5 bullets — what we know now we didn't before>
PROTOTYPE FATE: <delete | archive in branch | screenshot for note-kira>
NEXT SKILL: <plan-cap | research-strange | prototype-mcguyver again>
```

### Mandatory delete

Prototype code MUST be deleted or moved to throwaway branch. NEVER merge prototype to main.

If user says "actually let's ship the prototype" → REFUSE. Output:
```
**Refusing to ship prototype.** 
Constraints relaxed in step 3 mean code is unsafe for prod.
Hand off plan-cap to re-implement properly.
```

---

## Operating rules

- **Refuse without hypothesis.** Step 1 hard gate.
- **One hypothesis at a time.** Multi-claim = split into multiple prototypes.
- **Observable success criterion.** No "feels right".
- **Budget is HARD.** Don't extend on first try.
- **Code gets deleted.** Prototype ≠ ship vehicle.
- **No secrets · no prod writes · no side effects.** Some constraints still apply.

## Anti-patterns

- **"Prototype the feature"** — feature ≠ hypothesis · refuse, route to plan-cap
- **Multi-clause hypothesis** — "test A AND B AND C" · split
- **Vague success** — "see if it works" · need observable
- **Extending budget** — first ring = stop · don't keep going
- **Shipping prototype** — refuse · re-implement properly via plan-cap

## Common scenarios

| User says | Prototype reaction |
|---|---|
| "prototype RAG w/ Cohere v4" | hypothesis = hit@5 lift · budget 1hr · verdict drives v4 adoption |
| "try Drizzle vs TypeORM" | hypothesis = 10-table schema in <2hr · pick one · learnings inform decision |
| "see if React Query handles this" | hypothesis = cache invalidation works · budget 30min |
| "prototype the new feature" | REFUSE → route to plan-cap (feature, not hypothesis) |
| "spike a quick fix" | REFUSE if bug (route to debug-sherlock) · OK if hypothesis = "would X approach work" |

## Hand off

- VERDICT keep → hand off `plan-cap` for real impl (do NOT reuse prototype code)
- VERDICT discard → hand off `research-strange` (deeper investigation) OR new prototype
- VERDICT reshape → new `prototype-mcguyver` w/ refined hypothesis
- VERDICT inconclusive → split hypothesis · run multiple smaller prototypes
- Learnings worth recording → hand off `note-kira` (vault doc)

## Cross-ref

- `karpathy-rules` §P4 — Goal-Driven Execution (hypothesis = goal · success = verify)
- `plan-cap` — receives "keep" verdict · designs real impl
- `research-strange` — receives "discard" verdict · external investigation
- Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) (98k⭐) "throwaway prototype"

result: hypothesis answered · verdict drives next action · prototype code deleted · learnings captured.

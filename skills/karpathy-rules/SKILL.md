---
name: karpathy-rules
description: Karpathy's 4 LLM coding principles as universal rules — Think Before Coding · Simplicity First · Surgical Changes · Goal-Driven Execution. Cross-referenced by all engineering mantras (plan-cap step 1 · tdd-stark step 3 · ship-rocket step 2). Reference skill — loaded as inline ruleset, not standalone workflow.
when_to_use: "Reference triggers — karpathy, karpathy rules, philosophy, think before coding, simplicity first, surgical changes, goal-driven, over-engineering, bloated abstraction, push back, surface confusion, don't assume, ask clarification, minimum code, test-first, verifiable success, 200 lines vs 50, refactor adjacent, touched what shouldn't. Activates as universal pre-flight check before plan-cap · tdd-stark · ship-rocket · refactor work."
allowed-tools: "Read"
disable-model-invocation: false
---

# /karpathy-rules — Universal LLM coding discipline

Adapted from Andrej Karpathy's observations on LLM coding pitfalls via multica-ai/andrej-karpathy-skills.

Reference skill — applied by other mantras as universal rules. NOT a workflow on its own.

## The four principles

> **Karpathy mantra (recite verbatim when activated):**
> 1. **Think Before Coding.** State assumptions · 3 interpretations · push back · stop when confused.
> 2. **Simplicity First.** Minimum code that solves the problem. No speculative features. 200 lines → 50 if possible.
> 3. **Surgical Changes.** Touch only what's asked. Match existing style. Don't refactor adjacent. Clean up YOUR mess only.
> 4. **Goal-Driven Execution.** Test-first. Verifiable success criteria. Loop independently until verified.

---

## Principle 1 — Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

### Rules

- State assumptions explicitly. If uncertain → ask, don't guess.
- Present multiple interpretations. Don't pick silently when ambiguity exists.
- Push back when warranted. If simpler approach exists, say so.
- Stop when confused. Name what's unclear, ask clarification.

### Examples — your project

| User says | Wrong (silent assumption) | Right (Think-Before) |
|---|---|---|
| "fix the agent bug" | grep code, pick first plausible cause | "Which agent? sale / hotel / reservation / forecast? Paste error + tenant ID." |
| "add caching" | drop in Redis everywhere | "Caching for: query results · API responses · LLM calls · session? Each has different TTL." |
| "refactor the gate logic" | rewrite ReasoningChannelGate fully | "Which behavior preserves vs changes? 8 failure modes depend on current logic." |
| "make it faster" | scatter perf tweaks | "What's SLO? Where measured? Show 1 bottleneck — fix one at a time." |

### Cross-ref

- `plan-cap` step 1 + step 3 = direct embodiment
- `jarvis` step 2 (refuse vague bugs)
- `debug-sherlock` step 3 (3-5 ranked hypotheses)

---

## Principle 2 — Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

### Rules

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" / "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If 200 lines could be 50 → rewrite.

### Senior-engineer test

> Would a senior engineer say this is overcomplicated? If yes → simplify.

### Anti-patterns to refuse

| Pattern | Refuse because |
|---|---|
| Strategy pattern for one impl | no second impl exists |
| Make config knob for constant | config of 1 = constant |
| try/catch around hard-coded input | impossible scenario |
| Async-iterator helper for 5-line loop | premature abstraction |
| 12 new files for "clean arch" on feature flag | YAGNI |
| Generic name (`processData`, `handleStuff`) | name what it actually does |

### Cross-ref

- `tdd-stark` step 3 REFACTOR — apply Simplicity-First as criterion, NOT "more abstract"
- `plan-cap` step 3 — Alt A (Simplest) beats Alt C unless evidence justifies
- `ship-rocket` step 2 — senior-engineer test = scrutinize gate

---

## Principle 3 — Surgical Changes

**Touch only what you must. Clean up only your own mess.**

### Rules

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code → mention it, don't delete.

### Cleanup scope

- Remove imports/variables/functions YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

### Diff test

> Every changed line should trace directly to the user's request.

### Examples — your project

| Action | Surgical | Sprawling |
|---|---|---|
| Fix one if in `extractTokenFromEvent` | touch ~3 lines | refactor whole gate |
| Add tenantId filter to one query | one query | also "clean up" 5 others |
| Rename one variable | rename + callers | also reformat surrounding |
| Fix typo in one comment | one comment | rewrite whole docstring |

### Refuse triggers

If asked "fix X" but you find Y also broken:
- → Mention Y. Don't fix Y. Wait for ack.

### Cross-ref

- `tdd-stark` step 2 GREEN — minimal diff
- `plan-cap` --refactor mode — behavior preserved, not mixed-with-features
- `ship-rocket` step 2 — surgical-diff check before push

---

## Principle 4 — Goal-Driven Execution

**Define success criteria. Loop until verified.**

### Transform table

| Instead of... | Transform to... |
|---|---|
| "Add validation" | "Write tests for invalid inputs, make them pass" |
| "Fix the bug" | "Write test that reproduces it, make it pass" |
| "Refactor X" | "Tests pass before AND after" |
| "Speed up Y" | "Set p95 budget, measure baseline, drive to target" |
| "Improve error handling" | "Define error contract, test per case, make pass" |

### Multi-step plan format

```
1. <step> → verify: <check>
2. <step> → verify: <check>
3. <step> → verify: <check>
```

### Cross-ref

- `tdd-stark` entire mantra = goal-driven literal
- `plan-cap` step 5 (task list w/ verifiable end-states)
- `smoke-spidey` step 5 (verdict + artifacts)
- `rag-tune` decision matrix
- `agent-eval-vision` quality/cost/latency thresholds

---

## How other mantras invoke this skill

### plan-cap
- Step 1: refuse vague goal (Principle 1)
- Step 3: Alt A wins by default (Principle 2)

### tdd-stark
- Step 2 GREEN: minimal diff (Principle 3)
- Step 3 REFACTOR: senior-engineer test (Principle 2)

### ship-rocket
- Step 2 Scrutinize: every line traces to request (Principle 3)
- Step 2 Scrutinize: senior-engineer test (Principle 2)

### debug-sherlock
- Step 3 Falsify: 3-5 hypotheses (Principle 1)

### jarvis
- Step 2 repro check: refuse vague (Principle 1)

### --refactor work (any skill)
- Behavior preserved · code shape changes · tests unchanged (Principle 3)

---

## Operating rules

- **Reference skill, not workflow.** Invoked BY other skills, not standalone.
- **Force pushback.** If user request violates a principle, surface it before executing.
- **Cite principle by number.** "Refusing per Karpathy Principle 2 — adds 200 lines for a 5-line need."
- **Senior-engineer test mandatory.** Before any merge: apply Principle 2. If feels bloated → it is.

---

## Refuse-by-principle quick reference

| Situation | Refuse w/ |
|---|---|
| Vague "fix it" | P1 — need repro / scope |
| "Add abstraction" for one use | P2 — refuse w/o 2nd use case |
| Edit grows beyond ask | P3 — split surgical from refactor |
| "Just make it work" | P4 — define verifiable success first |
| Code w/o context | P1 — ask what changed, what's expected |
| Speculative error handling | P2 — refuse impossible scenarios |
| Config knob for constant | P2 — refuse, use constant |
| Adjacent ugly code (works) | P3 — mention, don't touch |

---

## Hand off

- plan-cap step 1 → invoke karpathy-rules Principle 1
- tdd-stark step 3 → invoke karpathy-rules Principle 2 + 3
- ship-rocket step 2 → invoke karpathy-rules Principle 3
- Any refactor work → invoke karpathy-rules Principle 3
- forge-stark generating new skills → cite karpathy-rules in cross-ref

result: 4 principles loaded · refuse-by-principle table active · other skills cross-ref for scrutinize gates.

---
name: intake-jarvis
description: Symptom router for incoming requests — classify bug / feature / refactor / question, gate on repro presence for bugs, route to correct downstream mantra skill. Replaces intake half of deprecated triage skill.
when_to_use: "Keyword triggers — please help, can you, I want, I need, problem is, issue with, broken, want to add, add feature, fix this, refactor, how do I, what is, ช่วยหน่อย, ปัญหา, ต้องการ, ขอเพิ่ม. Fires on ambiguous incoming requests to route them."
allowed-tools: "Read"
disable-model-invocation: false
---

# /intake — Symptom router

Classify incoming request in one pass. Route to correct mantra. No work done here — only routing.

## Recite verbatim as first response

> **Intake mantra:**
> 1. **Classify:** bug / feature / refactor / question / cross-cutting.
> 2. **Bug + repro present** → hand off `debug-sherlock`.
> 3. **Bug + no repro** → REQUEST repro, refuse to hypothesize.
> 4. **Feature** → hand off `plan-cap`.
> 5. **Refactor** → hand off `plan-cap` (preserve-behavior mode).
> 6. **Question** → answer directly, no skill chain.
> 7. **Cross-cutting (3+ services)** → hand off `plan-cap` + flag for `avengers`.

Then route.

---

## Step 1 — Classify

Read user message. Score against signals:

| Signal | Class |
|---|---|
| pastes error / stack trace / broken output / says "broken" / "not working" | **bug** |
| says "add" / "build" / "implement" / "create" / new endpoint / new feature | **feature** |
| says "refactor" / "clean up" / "extract" / "rename" / "simplify" / "tidy" / "split" | **refactor** |
| asks "what is" / "how does" / "why does" / "where is" w/o action | **question** |
| touches 3+ services AND mentions architecture / migration / cross-service | **cross-cutting** |

If multiple match → pick dominant. If genuinely ambiguous → ASK user one question to disambiguate. Do NOT guess.

## Step 2 — Bug + repro check

> **Karpathy P1 — Think Before Coding.** Don't hypothesize without repro. Refuse vague bugs. Repro artifact is the gate. See `karpathy-rules` §P1.

If class = bug, check repro:

**Repro present:**
- Error message + stack trace
- Curl command + output
- Screenshot + steps to reproduce
- "Run X and see Y"
- Replay harness trace ID
- Specific input → wrong output pair

**Repro absent:**
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
STOP. Wait.

If repro present → hand off `debug-sherlock`.

## Step 3 — Feature → plan-cap

Hand off `plan-cap` skill. NO scoping done here — plan owns step 1 (state goal).

## Step 4 — Refactor → plan-cap (preserve mode)

Hand off `plan --refactor`. Plan ensures behavior preservation is goal.

## Step 5 — Question → direct answer

Answer in same turn. NO skill chain.

Exceptions:
- Question about library/framework → invoke `documentation-lookup` if available
- Question about file location → invoke `project-patterns` (path lookup)
- Question about codebase architecture → invoke `graphify` (knowledge graph)

## Step 6 — Cross-cutting → plan-cap + spawn-agents flag

Hand off `plan-cap`. Flag in handoff that step 5 of plan should spawn agents.

---

## Operating rules

- **No work done in intake.** Pure routing.
- **Refuse to hypothesize bugs without repro.** Step 2 gate is hard.
- **Ambiguous → ask, don't guess.** One disambiguation question max.
- **Don't echo user back.** State classification + handoff in one line.

## Hand off

```
Class: <bug | feature | refactor | question | cross-cutting>
Repro: <present | absent | n/a>
Hand off → <skill-name>
```

result: classified + routed.

---
name: plan-cap
description: Pre-code planning mantra for your project — state goal, search existing, three alternatives, risk table, task list. Refuse to proceed when goal underspecified. Hand off to tdd-implement when plan accepted. Replaces planning halves of deprecated build-feature and refactor skills.
when_to_use: "Keyword triggers — plan, planning, design, scope, where to start, what to build, before code, ออกแบบ, วางแผน, refactor plan, feature plan, scope this, can we add, add feature, build X, implement X, scaffold X, สร้าง, เพิ่ม feature, redesign, restructure"
allowed-tools: "Bash(git *) Bash(grep *) Bash(rg *) Bash(ls *) Bash(find *) Bash(cat *) Bash(gh *) Read"
disable-model-invocation: false
---

# /plan — Pre-code planning mantra

Recite at session start. Apply in order. Refuse to write code before step 5 complete.

## Recite verbatim as first response

> **Plan mantra:**
> 1. **State goal in one sentence.** If can't, underspecified — refuse.
> 2. **Search existing.** GitHub + repo grep + Context7 docs. 80% match → port it.
> 3. **Three alternatives.** Simplest / current-pattern / future-proof. Pick + justify.
> 4. **Risk table.** Dependency · blast-radius · rollback path.
> 5. **Task list.** Numbered steps w/ effort estimate.

Then begin.

---

## Step 1 — State goal

> **Karpathy P1 — Think Before Coding.** Don't assume. Don't hide confusion. Surface tradeoffs. Refuse vague asks. See `karpathy-rules` §P1.

One sentence in your own words. NOT user's words echoed back. NOT a list of features.

If can't state in one sentence:
- Output to user: **"Goal underspecified. Need: <missing piece>. Refusing to proceed."**
- Stop. Wait for user clarification.
- DO NOT hypothesize what they might mean.

Output format:
```
Goal: <one sentence>
Why: <one-line motivation from user context>
Out of scope: <bulleted what is NOT this work>
```

## Step 2 — Search existing

Before designing anything new, prove nothing already solves it.

Mandatory checks (order):
1. **Repo grep:** `rg "<core noun/verb>" microservices/ --type ts` for current radiant1 implementations.
2. **GitHub:** `gh search code "<feature>" --owner radiant1` (if applicable).
3. **Context7:** library docs via documentation-lookup skill if framework feature.
4. **Other services:** if work in service A, check whether B/C/D already pattern exists. Cross-service overlap is common.

If 80%+ match found → port/extend, NOT build new. Cite file path + line.

Output format:
```
Existing matches:
- microservices/<service>/<file>:<line> — <how close>
- (none) — confirmed novel
```

## Step 3 — Three alternatives

NEVER design one. Always three. Anchoring on first idea is the bug.

### PREFERRED — AskUserQuestion picker

When AskUserQuestion tool available, present Alt A/B/C as single-select question. User picks via UI. Options format:
- `label`: Alt approach (e.g. "Simplest — 90% value · minimal change (Recommended)")
- `description`: pros · cons · effort estimate

User picks. Then proceed step 4 (risk table).

Fallback (no AskUserQuestion): use inline table below.

| Alt | Approach | Pros | Cons | Effort |
|---|---|---|---|---|
| A | Simplest (90% of value, minimal change) | … | … | … |
| B | Match current radiant1 pattern (DDD/CQRS for backend, Zustand for FE) | … | … | … |
| C | Future-proof / strategic (bigger refactor) | … | … | … |

Pick one. Write **why this over the others** in one paragraph. Specifically address what you give up.

## Step 4 — Risk table

| Risk | Type | Blast radius | Rollback path |
|---|---|---|---|
| <e.g. multi-tenant leak> | data | all tenants | feature flag off |
| <e.g. token cost spike> | cost | LLM bill | model routing knob |
| <e.g. SSE leak> | UX | chat users | revert + redeploy |

If risk has no rollback path → escalate to user before proceeding.

For ai-agent-backend work, ALWAYS check:
- Multi-tenant isolation impact
- LLM cost/latency budget impact
- 4-layer stream pipeline impact
- Migration safety if DB schema touched

## Step 5 — Task list

```
1. <atomic step> — <effort: 15min/1h/4h/1d>
2. <atomic step> — <effort>
3. …
```

Rules:
- Each task ≤ 4 hours. Bigger → split.
- Each task ends with verifiable state (test passes / curl returns / build green).
- Order = dependency order. NO task depends on a later task.

## Operating rules

- **No rubber-stamp.** If user provided vague request, REQUIRE step 1 clarification.
- **No design doc bloat.** Output ≤ 1 page total.
- **Cite file paths.** Every claim about existing code must reference `path:line`.
- **Refuse without inputs.** Step 1 goal + step 2 search are mandatory.

## Mode flags

- `--refactor` — step 1 goal must say "preserve behavior X while changing Y"
- `--cross-service` — step 5 must spawn-agents handoff if > 3 services touched

## Hand off

- Default → `tdd-stark` (write failing test first)
- Cross-service touching 3+ services → `avengers`
- Bug-driven plan (existing bug to fix) → `debug-sherlock` first, then plan

result: plan accepted, ready to implement.

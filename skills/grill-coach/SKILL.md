---
name: grill-coach
description: Alignment interview before coding — interrogate intent, surface assumptions, lock acceptance criteria. Five-step mantra — restate · interpret · accept · out-of-scope · confirm. Inspired by mattpocock/skills "grill before coding". Refuses without actual feature/refactor ask (skips clarifying questions, greetings). Use when user gives ambiguous feature request that risks over-engineering or wrong-direction work.
when_to_use: "Slash triggers — /grill-coach, /grill, /coach. Hero triggers — coach interrogate, alignment check, grill intent, push back. Keyword triggers — alignment, what do you mean, clarify scope, before I code, acceptance criteria, out of scope, vague request, ambiguous spec, what's the real ask, double-check, สอบถาม, ถามให้ชัด. Does NOT fire on — clear single-verb tasks · already-planned features (use plan-cap) · simple questions (use jarvis)."
allowed-tools: "Read"
disable-model-invocation: false
---

# /grill-coach — Alignment interview (Coach)

Coach grills the team BEFORE the game. You grill the user before writing code. Wrong understanding = wrong feature = wasted sprint.

Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) "grill before coding". Run BEFORE plan-cap when ask is ambiguous.

## Recite verbatim as first response

> **Grill mantra:**
> 1. **Restate.** Repeat ask in YOUR words. Not echo. Surface what's vague.
> 2. **Interpret.** 3 reasonable interpretations. User picks OR ack one.
> 3. **Acceptance criteria.** Exact list — done when X · Y · Z observable.
> 4. **Out-of-scope.** Explicit list — NOT this · NOT that. Prevents creep.
> 5. **Confirm.** User signs off. Hand off plan-cap.

Then begin.

---

## Step 1 — Restate

> **Karpathy P1 — Think Before Coding.** Don't assume. Don't hide confusion. Surface tradeoffs. See `karpathy-rules` §P1.

Repeat the ask in your OWN words. Force a fresh phrasing. Verbatim echo is anti-pattern.

### Refuse-without

If ask is non-actionable (greeting · ack · single-word reply) → STOP. Output:
```
**Grill refuses non-ask.**
Need concrete feature, fix, or refactor request to interrogate.
```

### Surface vagueness

Output format:
```
What I heard: <one-sentence restate>
Vague pieces: <bulleted list of fuzzy nouns/verbs>
Tightening questions: <2-3 questions to disambiguate>
```

## Step 2 — Three interpretations

NEVER pick one silently. Surface ambiguity.

### PREFERRED — AskUserQuestion picker

When AskUserQuestion tool available, surface 3 interpretations as single-select question. User picks via UI. Options format:
- `label`: interpretation name (≤5 words · suffix "(Recommended)" on safest/narrowest)
- `description`: implication · scope · impl difficulty

User picks. Then proceed step 3.

Fallback (no AskUserQuestion): use inline format below.

Format:
```
| # | Interpretation | Implication |
|---|---|---|
| A | <reading 1> | <impl difficulty / scope> |
| B | <reading 2> | <impl difficulty / scope> |
| C | <reading 3 — usually narrowest> | <smallest impl> |
```

User picks ONE. Or says "actually D".

If user can't pick → ask for missing context. Don't proceed.

## Step 3 — Acceptance criteria

After interpretation locked, write EXACT acceptance.

Format:
```
DONE when:
- [ ] <observable 1> — e.g. "endpoint returns 200 with payload X"
- [ ] <observable 2> — e.g. "tenant-A and tenant-B isolated"
- [ ] <observable 3> — e.g. "p95 < 300ms under N concurrent"
```

Rules:
- Each acceptance is OBSERVABLE (testable · runnable · visible)
- NO "feels right" / "user happy" / "looks clean"
- Each maps to a test that can be written

If user gives ambiguous acceptance → push back. "How do we know X?"

## Step 4 — Out of scope

Explicit NOT list prevents scope creep.

Format:
```
NOT this work:
- <thing 1> — defer to <when/where>
- <thing 2> — irrelevant to current ask
- <thing 3> — separate ticket
```

Common scope creeps to call out:
- "while you're in there, also fix X" → refuse, separate
- "and make sure it scales to N" → not this ask
- "and improve the tests" → separate refactor
- "and document it" → separate doc task

## Step 5 — Confirm + handoff

Output to user:
```
**Grill complete. Confirm before plan-cap fires:**

Goal: <one sentence>
Interpretation: <A | B | C>
Acceptance: <N observables>
Out-of-scope: <N items>

Confirm → proceed plan-cap · push back → restart step 1
```

Wait for user nod. If user redirects → loop step 1-4. NO silent assumption.

After confirm → hand off plan-cap with locked goal.

---

## Operating rules

- **Refuse non-asks.** Step 1 hard gate.
- **No silent picking.** Step 2 always surfaces 3 interpretations.
- **Acceptance = observable.** No "feels right".
- **Out-of-scope explicit.** Step 4 mandatory · prevents creep.
- **Confirm before handoff.** No proceeding to plan-cap until user signs off.

## Anti-patterns

- **Echo ask back verbatim** — restate is REPHRASE, not parrot
- **Pick interpretation silently** — surface ambiguity
- **Vague acceptance** — "make it work" ≠ acceptance
- **Implicit out-of-scope** — say it explicitly
- **Skip confirm** — even if seems clear, ask for nod

## When to use vs other skills

| Skill | When |
|---|---|
| **grill-coach** | Ambiguous feature ask · risk of wrong direction |
| **jarvis** | Classify ask (bug/feature/refactor) · route |
| **plan-cap** | Goal locked · ready to plan implementation |

Grill = interrogation BEFORE intake (or after intake if ambiguous).

## Common scenarios

| User says | Grill output |
|---|---|
| "add notifications" | 3 interpretations (email · push · in-app) · pick · acceptance |
| "make it faster" | 3 interpretations (frontend · backend · DB) · acceptance = SLO |
| "fix the bug" | refuse → route to jarvis (need repro) |
| "improve UX" | 3 interpretations (which screen · which interaction) |
| "build dashboard" | 3 interpretations (which metrics · which users · which layout) |

## Hand off

- User confirms → hand off `plan-cap` with locked goal
- User wants pure research first → hand off `research-strange`
- User redirects fundamentally → restart at step 1
- Ask becomes clear bug → hand off `jarvis` (skip grill)

## Cross-ref

- `karpathy-rules` §P1 — Think Before Coding
- `plan-cap` step 1 — receives locked goal from here
- `jarvis` — different scope (classify, not interrogate)
- Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) (98k⭐) "grill before coding"

result: locked goal · 3 interpretations picked · acceptance criteria written · out-of-scope explicit · plan-cap ready.

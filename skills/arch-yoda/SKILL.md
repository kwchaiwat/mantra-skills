---
name: arch-yoda
description: Yoda meditates on architecture — periodic refactor health check that surfaces god files, deep nesting, duplication, tight coupling. Five-step mantra — target · hot spots · smell catalog · rank · verdict. Inspired by mattpocock/skills "design every day". Refuses without target service/dir. Use weekly or before big-bang refactor.
when_to_use: "Slash triggers — /arch-yoda, /arch, /yoda. Hero triggers — yoda meditate, arch check, refactor health, code smell scan. Keyword triggers — architecture health, code smell, refactor candidates, god file, tight coupling, duplication, churn, hot spots, refactor priorities, design check, design review, smell scan, technical debt, debt scan, ตรวจสุขภาพ code, refactor list. Does NOT fire on — single-file refactor (use plan-cap --refactor) · specific bug (use debug-sherlock) · fresh feature (use plan-cap)."
allowed-tools: "Bash(rg *) Bash(grep *) Bash(find *) Bash(ls *) Bash(cat *) Bash(wc *) Bash(git *) Read"
disable-model-invocation: false
---

# /arch-yoda — Refactor health (Master Yoda)

Yoda watches the Force. You watch architecture rot. Periodic check surfaces god files · churn · duplication BEFORE big-bang refactor needed.

Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) "design every day". Run weekly · or before major refactor planning.

## Recite verbatim as first response

> **Arch mantra:**
> 1. **Target.** Service · dir · feature area to inspect.
> 2. **Hot spots.** Churn (git log) · size (wc -l) · age (git blame oldest).
> 3. **Smell catalog.** God file · deep nesting · dup · tight coupling · dead code.
> 4. **Rank.** Impact × ease — top-5 refactor candidates.
> 5. **Verdict.** Health score · top fix recommended · handoff.

Then begin.

---

## Step 1 — Target

Refuse without target dir or service.

### Required

- [ ] **Path** — absolute path to dir or service
- [ ] **Why** — weekly health · pre-refactor scoping · post-incident retrospective

If missing → STOP. Output:
```
**Arch refuses without target.**
Need:
- target dir: <absolute path>
- reason: <weekly | pre-refactor | post-incident>
```

## Step 2 — Hot spots

Surface churn + size + age.

```bash
# Churn — top files changed in last 90 days
git log --since="90 days ago" --name-only --format="" <target> | sort | uniq -c | sort -rn | head -20

# Size — biggest files
find <target> -name "*.ts" -o -name "*.tsx" | xargs wc -l | sort -rn | head -20

# Age — oldest files (often god files)
find <target> -name "*.ts" -exec git log -1 --format="%ar %H {}" {} \; | sort | head -10
```

Output:
```
CHURN (top 5):
- <file> — <change count>
- ...

SIZE (top 5):
- <file> — <line count>
- ...

OLD (top 3 — survived many refactors):
- <file> — <last modified>
- ...
```

Cross-reference: file that's BIG + HIGH CHURN + OLD = god-file candidate.

## Step 3 — Smell catalog

Run grep-based smell detectors.

| Smell | Detector |
|---|---|
| **God file** | `find <target> -name "*.ts" \| xargs wc -l \| awk '$1 > 800'` |
| **Deep nesting** | `rg "^\\s{16,}" --type ts -l` (4+ levels) |
| **Duplication** | `rg "<distinctive-snippet>" -l` (manual seed) |
| **Tight coupling** | imports from 10+ modules in one file |
| **Dead code** | `npx ts-prune` if available |
| **Magic numbers** | `rg "[^a-zA-Z_]([0-9]{4,})" --type ts` (5+ digit numbers) |
| **`console.log`** | `rg "console\.log" --type ts` (forbidden per project convention) |
| **`any` type** | `rg ": any\b" --type ts` (forbidden) |
| **Nested ifs** | `rg "if.*\{.*if" --type ts` (forbidden) |

Output:
```
| Smell | Count | Top offender |
|---|---|---|
| god files | N | <file> |
| deep nesting | N | <file> |
| console.log | N | <file:line> |
| any type | N | <file:line> |
| ...
```

## Step 4 — Rank refactor candidates

Score each candidate: **impact × ease**.

| Candidate | Impact (1-5) | Ease (1-5) | Score | Why |
|---|---|---|---|---|
| Extract logic from god service | 5 | 3 | 15 | god file · 8 callers · clear seam |
| Replace `console.log` w/ logger | 2 | 5 | 10 | trivial · low risk |
| Remove `any` from shared util | 4 | 3 | 12 | type safety · 5 sites |
| Dedup 3 templates | 3 | 4 | 12 | maintain reduction |

Sort by score DESC. Take top 5.

Rules:
- Impact = how much this hurts now (god file = 5, nit = 1)
- Ease = how cheap the fix (1-day = 5, 1-week = 1)
- Score = impact × ease. High score = ship next.

## Step 5 — Verdict

Compute health score:

| Health | When |
|---|---|
| **HEALTHY** | No god files · <5 smells total · no forbidden patterns |
| **DRIFT** | 1-2 god files · 10-30 smells · some forbidden patterns |
| **DEBT** | 3+ god files · 30+ smells · forbidden patterns widespread |
| **ROT** | 5+ god files · 100+ smells · daily blockers |

Output:
```
HEALTH: <healthy | drift | debt | rot>
TOP FIX: <#1 from rank table>
RECOMMENDED NEXT: 
  - DRIFT → hand off plan-cap --refactor on top 1-2 items
  - DEBT → hand off plan-cap to schedule sprint
  - ROT → escalate · stop feature work · debt sprint required
```

---

## Operating rules

- **Refuse without target.** Step 1 hard gate.
- **Cite file:line in smells.** Every smell has concrete location.
- **Impact × ease score.** No "feels important" — number it.
- **Health verdict mandatory.** Step 5 single-word health drives next.
- **Run on schedule.** Yoda meditates daily — run arch-yoda weekly minimum.

## Anti-patterns

- **Scan whole repo at once** — too much noise · scope to dir
- **List all smells** — top 20 only · rank-cut the rest
- **No impact × ease** — without score, can't prioritize
- **Skip health verdict** — driver of next action

## Common scenarios

| Target | Yoda output |
|---|---|
| Heavy-churn module | 2 god files · 40 smells · DEBT · top fix = extract logic |
| Clean service | 0 god files · 5 smells · HEALTHY · weekly check pass |
| FE components | 3 dup templates · DRIFT · top fix = consolidate templates |
| Fresh greenfield service | 1 god file (controller god) · ROT-prevention · refactor now |

## Hand off

- HEALTHY → terminal (next check in 1 week)
- DRIFT → hand off `plan-cap --refactor` on top 1-2 items
- DEBT → hand off `plan-cap` to schedule debt sprint
- ROT → escalate · halt feature work · debt sprint required

## Cross-ref

- `karpathy-rules` §P2 — Simplicity First (god files are anti-simplicity)
- `karpathy-rules` §P3 — Surgical Changes (refactor surgically, not big-bang)
- `zoom-cerebro` — neighborhood map of god-file candidates
- `plan-cap --refactor` — implements top-ranked fix
- Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) (98k⭐) "design every day"

result: health score · top-5 refactor candidates ranked · next-skill recommended.

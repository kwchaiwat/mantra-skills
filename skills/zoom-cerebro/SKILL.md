---
name: zoom-cerebro
description: Cerebro scans the broader system — zoom out from a single file/function to show the architectural neighborhood before diving in. Five-step mantra — target · callers · callees · cross-cutting · map. Inspired by mattpocock/skills "zoom out before coding". Refuses without concrete target. Use BEFORE editing critical/shared files to see blast radius.
when_to_use: "Slash triggers — /zoom-cerebro, /zoom, /cerebro. Hero triggers — cerebro scan, zoom out, system view, neighborhood map, charles xavier. Keyword triggers — zoom out, who uses this, who calls this, blast radius, impact analysis, before I touch, system context, broader view, where else, dependency map, callers, callees, mapped, ดูภาพรวม. Does NOT fire on — search-only ('where is X') use grep · planning (use plan-cap) · already small isolated change."
allowed-tools: "Bash(rg *) Bash(grep *) Bash(find *) Bash(ls *) Bash(cat *) Read"
disable-model-invocation: false
---

# /zoom-cerebro — System view (Cerebro)

Cerebro sees every mutant in the world. You see every file that touches what you're about to change. Surgical edit needs surrounding-code awareness.

Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) "zoom out before coding". Run BEFORE tdd-stark when target is critical/shared.

## Recite verbatim as first response

> **Zoom mantra:**
> 1. **Target.** What file/function/feature to map.
> 2. **Callers.** Who uses this. rg + graphify.
> 3. **Callees.** What this uses. rg + graphify.
> 4. **Cross-cutting.** Config · DB · cache · UI · tests · docs.
> 5. **Map.** Neighborhood diagram + impact verdict.

Then begin.

---

## Step 1 — Target

Refuse without concrete target.

### Required

- [ ] **File path** OR **function name** OR **feature area**
- [ ] **Why zoom** — about to refactor / about to fix / pre-impact-check

If missing → STOP. Output:
```
**Zoom refuses without target.**
Need:
- file: <absolute path>
- OR function/class: <name>
- OR feature: <one-line description + entry point>
```

## Step 2 — Callers

Who uses this target?

```bash
# Function usage
rg "<funcName>\(" --type ts --type tsx -l

# Import usage
rg "from ['\"].*<filename>['\"]" --type ts -l

# Cross-service usage (if applicable)
rg "<funcName>" <repo-root>/ -l

# Graphify lookup (if available)
cat graphify-out/<service>/graph.json | jq '.nodes[] | select(.id == "<target>")'
```

Output:
```
Callers (N total):
- <file:line> — <one-line context>
- <file:line> — <one-line context>
```

Flag heavy callers (>5 sites = god dependency).

## Step 3 — Callees

What does target call?

```bash
# Imports inside target file
rg "^import" <target-file>

# Function calls inside target
rg "\.[a-zA-Z]+\(" <target-file>
```

Output:
```
Callees (N total):
- <module> — <what role>
- <module> — <what role>
```

Flag external dependencies (HTTP · DB · LLM · queue).

## Step 4 — Cross-cutting impact

| Dimension | Check |
|---|---|
| **Config** | env vars · feature flags · toggles |
| **Schema** | DB tables · migrations touched |
| **Cache** | Redis keys · invalidation paths |
| **Queue** | jobs · consumers |
| **UI** | components rendering this data |
| **Tests** | spec files referencing target |
| **Docs** | README · CLAUDE.md mentions |
| **Migration** | open PRs touching this area |

Format:
```
Cross-cutting touched:
- config: <none | NAME=value>
- schema: <none | table.column>
- cache: <none | redis-key-pattern>
- queue: <none | job-name>
- ui: <none | component>
- tests: <count + paths>
- docs: <count + paths>
- migrations: <none | open PR>
```

## Step 5 — Map + verdict

Neighborhood diagram (ASCII):
```
                  [Caller A] → [Caller B] → [Caller C]
                       ↓           ↓           ↓
                    ┌─────────────────────┐
                    │   TARGET (file:fn)  │
                    └─────────────────────┘
                       ↓           ↓           ↓
                  [Callee X] → [Callee Y] → [DB / cache]

  cross-cutting: config[X] · schema[Y] · cache[Z]
```

Verdict (one of):
- **isolated** — N callers ≤ 3 · safe surgical edit
- **shared** — N callers 4-10 · need contract test
- **god** — N callers 10+ · break into chunks · plan migration

Output:
```
VERDICT: <isolated | shared | god>
BLAST RADIUS: <count of files at risk>
RECOMMENDED NEXT: <skill or action>
```

---

## Operating rules

- **Refuse without target.** Step 1 hard gate.
- **Cite path:line.** Every caller/callee referenced w/ file location.
- **Graphify-first.** If graphify-out/ exists, use it before rg.
- **Flag god dependencies.** >5 callers = warning · >10 = god verdict.
- **Cross-cutting mandatory.** Step 4 has 8 dimensions, check all.

## Anti-patterns

- **Skip cross-cutting** — schema/cache impact is where bugs hide
- **No verdict** — step 5 verdict mandatory · drives next action
- **Single-grep** — rg one pattern misses callers · use multiple
- **No path:line cites** — vague "many files use this" = no signal

## Common scenarios

| Target | Zoom output |
|---|---|
| God util function | callers: 12 files · callees: 3 utils · verdict=god |
| Store/state setter | callers: 8 components · cross-cutting: state DOES touch · verdict=shared |
| One single-use helper | callers: 1 · verdict=isolated · safe edit |
| New endpoint scaffolding | target=feature not file · zoom shows planned blast radius |

## Hand off

- isolated → hand off `tdd-stark` (safe to edit)
- shared → hand off `plan-cap` + contract test (need design)
- god → hand off `arch-yoda` (refactor health check first)
- migration risk → hand off `scrutinize-falcon` (review change before commit)

## Cross-ref

- `karpathy-rules` §P3 — Surgical Changes (zoom proves what's adjacent vs target)
- `plan-cap` step 2 — search existing (zoom is the deeper version)
- `arch-yoda` — runs zoom + smell catalog · god-finder
- Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) (98k⭐) "zoom out before coding"

result: neighborhood mapped · blast radius known · verdict drives next-skill choice.

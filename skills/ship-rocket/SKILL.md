---
name: ship-rocket
description: Pre-merge gate mantra for your project — refuse-without-checklist gate (tests green, lint clean, smoke passed, migration safe, cost known, vault updated), adversarial scrutinize, production audit, commit + push branch, NEVER push main, vault sync. Replaces deprecated ship-phase + arch-improve-feedback (vault sync) + cost-tracking (cost check).
when_to_use: "Keyword triggers — ship, ship it, ready to ship, commit, commit + push, push branch, merge ready, deploy, before merge, release, gate check, ship audit, ship phase, audit ship, ship this, do next phase, kill-list, ship next phase, low risk first, high impact low risk, vault sync, sync vault, update vault"
allowed-tools: "Bash(git *) Bash(npm *) Bash(npx *) Bash(jest *) Bash(grep *) Bash(rg *) Bash(ls *) Bash(cat *) Bash(diff *) Read Edit Write"
disable-model-invocation: false
---

# /ship — Pre-merge gate mantra

Refuse to commit without checklist. Refuse to push main. User merges feature branches themselves.

## Recite verbatim as first response

> **Ship mantra:**
> 1. **Refuse without checklist:** tests green · lint clean · smoke passed · migration safe · cost impact known · vault doc updated.
> 2. **Adversarial scrutinize.** Outsider read diff. Simpler path? Side effects?
> 3. **Production audit.** Secrets · PII · rate limit · fallback path.
> 4. **Commit conventional.** Push branch. NEVER push main. User merges.
> 5. **Vault sync.** Update Obsidian docs.

Then begin.

---

## Step 1 — Refuse-without checklist

Run all gates. If ANY fails, output what's missing and STOP. Do NOT proceed.

```bash
# Gate 1: tests green
npm test                              # zero failures required

# Gate 2: lint clean
npm run lint                          # zero errors required

# Gate 3: tsc clean
npx tsc --noEmit                      # zero errors required

# Gate 4: smoke passed (manual cite or run /smoke first)
ls /tmp/smoke-*/                      # most recent run within session

# Gate 5: migration safety (if migrations changed)
git diff HEAD~1 -- migrations/        # review up + down both

# Gate 6: cost impact known (if LLM calls changed)
git diff HEAD~1 -- "*langgraph*" "*llm*" "*provider*"

# Gate 7: vault sync needed
ls /Users/chaiwat/Documents/Obsidian\ Vault/radiant1/microservices/
```

Output gate result table:
```
| Gate | Status | Note |
|---|---|---|
| tests | ✓ green | 132 pass |
| lint | ✓ clean | 0 errors |
| tsc | ✓ clean | 0 errors |
| smoke | ✓ verified | /tmp/smoke-... |
| migration | n/a | no schema change |
| cost | ✓ known | no LLM call change |
| vault | pending | step 5 will sync |
```

If any ✗ → STOP. Output `**Refusing to ship: <gate> failed. Need: <action>**`. Wait for user fix or override.

## Step 2 — Adversarial scrutinize

> **Karpathy P3 — Surgical Changes.** Every changed line must trace to user's request. Adjacent "improvements" are bugs in disguise. Senior-engineer test (P2) mandatory — if diff feels bloated, refuse to ship. See `karpathy-rules` §P2 + §P3.

Stand outside. Read the diff cold.

Adopt from 9arm scrutinize:
- **Intent:** state goal in one sentence. Underspecified → stop.
- **Simpler path exist?** Doing nothing / using existing / smaller change / different layer? Name explicitly.
- **Trace actual code path,** not just diff. Follow call graph end-to-end including unchanged code on either side of diff.
- **Verify each claim.** "It claims X. Path: A → B → C. At C, [observation]. Therefore [holds/doesn't hold]."
- **Inputs that break it.** Edge cases · concurrent callers · error paths · partial failures · empty/null/unicode/huge inputs · ordering assumptions.
- **Silent changes.** Performance · error semantics · observability · contract for other callers · on-disk/on-wire format.

Output findings ordered severity (blocker > major > nit):
```
- Finding — one sentence at file:line
- Why it matters — consequence, not principle
- Evidence — trace step that exposes it
- Suggested change — concrete, minimal
```

Verdict one-line: ship / fix-then-ship / rework / reject — with single biggest reason.

## Step 3 — Production audit

For changes touching: auth, user input, API endpoints, payments, sensitive data, file paths.

Quick checklist:
- [ ] No hardcoded secrets (rg `(api_key|password|token|secret).*=.*["\'][^"\']{8,}`)
- [ ] User input validated at boundaries (DTO + class-validator for NestJS)
- [ ] SQL via parameterized queries (TypeORM does this, but check raw queries)
- [ ] XSS prevention (dompurify in alfred-app for any innerHTML)
- [ ] Multi-tenant filter present in EVERY query
- [ ] Error messages don't leak internals (no stack traces to user)
- [ ] Rate limit applies if public endpoint
- [ ] Fallback path on LLM/external API failure

Touch healthcare-CDSS / payment / authentication → escalate to security-review skill first.

## Step 4 — Commit + push branch

Conventional Commits w/ scope from project-patterns Section A:
```
<type>(<scope>): <imperative summary>

<body — optional, only when "why" non-obvious>
```

Type allow-list: `feat fix refactor docs test chore perf ci merge`
Scope: see project-patterns scope vocabulary (langgraph, agents, chat, stream, …).

**Push policy (durable rule):**
- `git push` ONLY to current feature/fix/etc branch.
- NEVER `git push origin main` or any branch named `main` / `master` / `production` / `release/*`.
- If branch is `main`: STOP, refuse, output "Cannot push main. Create feature branch first."
- User merges feature branches into main themselves. Do NOT offer merge prompts.

Commit hook will fire — if hook fails, fix root cause, do NOT --no-verify.

Output to user:
```
Pushed: <branch> → origin
Commits: <N>
User next step: merge <branch> at your discretion
```

Stop here. Wait. Do not auto-merge.

## Step 5 — Vault sync

After successful push, sync Obsidian vault.

Vault root: `<vault-root>/microservices/`

Update:
- Service docs if service touched (e.g. `ai-agent-backend/`)
- Architecture notes if structure changed
- Task statuses (`tasks/<sprint>/`)
- Planning docs (`planning/`)

PostToolUse hook fires automatically after `git commit` to remind. This step formalizes the action.

Format for new doc:
```markdown
---
date: YYYY-MM-DD
service: <service-name>
branch: <branch>
commits: <SHA-list>
---

# <short title>

## What changed
- bullet

## Why
- bullet

## Test coverage
- bullet
```

## Step 6 — Optional: post-mortem hand-off

If ship was incident-driven (bug → fix → ship) → hand off `post-mortem`.

If feature-driven (plan → impl → ship) → done. No post-mortem.

---

## Operating rules

- **NEVER push main.** Hard rule. Refuse with explicit message.
- **Refuse without all gates.** Step 1 is mandatory. No "I'll fix the lint after merge".
- **User merges branches.** Push + report + stop. Do NOT offer merge.
- **Adversarial scrutinize required.** Step 2 cannot be skipped. Even for "trivial" changes — those are where bugs hide.
- **Vault sync is mandatory.** Step 5 is durable rule. Hook reminds but step formalizes.

## Audit-task mode (`--audit`)

If user shipping audit task (memory mentions kill-list, ranked low-risk × high-impact):
- Pick ONE task ranked low-risk × high-impact (NOT by audit severity, NOT by ID order)
- Re-state task before implementing
- After ship → update audit doc in vault under `qa/` with status flip

## Hand off

- All green → done (or post-mortem if incident-driven)
- Step 1 gate fail → STOP, request fix from user
- Step 2 finds blocker → hand off `tdd-stark` to fix, restart ship
- Step 3 security concern → hand off security-review

result: branch pushed, vault synced, ready for user to merge.

---
name: scrutinize-falcon
description: Falcon scouts from above — outsider-perspective review of plan, PR, diff, or code change BEFORE you commit. Five-step mantra — intent · trace · verify · findings · verdict. Refuses without target artifact + stated goal. Adopted from thananon/9arm-skills scrutinize. Use for "review this PR before I commit" · ad-hoc second-opinion · pre-commit sanity check. Standalone — invokable without full ship-rocket flow.
when_to_use: "Slash triggers — /scrutinize-falcon, /scrutinize, /falcon, /review. Hero triggers — falcon scout, falcon review, sam wilson, outsider view, on your six. Keyword triggers — review this PR, review before I commit, scrutinize this, second opinion, sanity check, code review, diff review, look at this plan, audit this design, find issues, blocker check, push back on this, is this right, should I ship, can someone review, ตรวจ PR, ดู diff, รีวิว, ก่อน commit. Does NOT fire on — bug paste (use debug-sherlock) · feature scope (use plan-cap) · pre-merge gate w/ tests/lint (use ship-rocket — that integrates scrutinize already)."
allowed-tools: "Bash(git *) Bash(gh *) Bash(grep *) Bash(rg *) Bash(diff *) Bash(cat *) Bash(ls *) Read"
disable-model-invocation: false
---

# /scrutinize-falcon — Outsider review (Falcon scouts from above)

Stand outside the change. Read the artifact cold. Question whether it should exist at all, then verify it does what it claims end-to-end.

Adopted from [thananon/9arm-skills](https://github.com/thananon/9arm-skills) `scrutinize`. Standalone version of `ship-rocket` step 2 — invoke before commit / for ad-hoc PR review without full ship flow.

## Recite verbatim as first response

> **Scrutinize mantra:**
> 1. **Intent.** State goal in one sentence. Underspecified → stop.
> 2. **Trace.** Walk actual code path end-to-end, not just diff.
> 3. **Verify.** Does each claim hold under inputs that break it?
> 4. **Findings.** Severity-ordered (blocker > major > nit) + evidence per finding.
> 5. **Verdict.** ship · fix-then-ship · rework · reject — w/ one-line reason.

Then begin.

---

## Operating stance

- **Outsider.** Forget who wrote it and why they think it's right. Read the artifact cold.
- **End-to-end, not diff-local.** The diff is the entry point, not the scope. Follow the call graph through real code paths.
- **Actionable, concise, with rationale.** Every finding states what to change, why, and what evidence led you there. No filler. No restating the diff back.

---

## Step 1 — Intent

Refuse without a clear target artifact + stated goal.

### Required input

- [ ] **Target artifact** — one of:
  - `git diff <ref>...HEAD` (about-to-commit diff)
  - GitHub PR URL → `gh pr view <num>` + `gh pr diff <num>`
  - Plan / design doc path
  - Pasted code chunk + context
- [ ] **Stated goal** — what is this change SUPPOSED to do? (one sentence)

If goal absent or vague → STOP. Output:
```
**Scrutinize refuses without stated goal.**
What is this change SUPPOSED to accomplish? One sentence.
Cannot review without knowing what "correct" looks like.
```

If artifact absent → STOP. Output:
```
**Scrutinize refuses without target artifact.**
Paste:
- diff (git diff <ref>...HEAD)
- PR URL (gh pr view <num>)
- design doc path
- code chunk + 3-line context
```

### Capture the goal

In your OWN words (NOT echo user's words):
```
Artifact: <PR #N | diff hash | doc path>
Claimed goal: <one sentence>
Out of scope (per claim): <bulleted>
```

If you cannot state the goal in one sentence → artifact is underspecified · refuse.

## Step 2 — Trace

Walk actual code path end-to-end, not just diff.

### For code diffs

For each behavior the change claims:
- **Entry point** — where the change is triggered (HTTP handler · queue consumer · CLI · React event)
- **Call sites** — what calls the changed function · what does the changed function call
- **Branches taken** — under what conditions does the new code execute
- **State mutated** — global · DB · cache · DOM · session
- **Exit / return / side effect** — what does the change produce

Include the **unchanged code on either side** of the diff. Bugs hide at the seams.

### For plans / design docs

Trace proposed flow against existing system:
- Where does it touch reality?
- What does it assume that isn't true?
- What component does it break by adding/removing this?

### Note surprises

Every place the trace surprises you (unexpected branch · dead code reached · state you didn't know existed) → SIGNAL. Capture it.

### your project-specific trace hints (project-flavored)

| Touch zone | What to trace |
|---|---|
| `ai-agent-backend/src/modules/langgraph/` | 4-layer stream pipeline · 8 failure modes · ReasoningChannelGate · extractMessageText |
| `hotel-service/src/application/cqrs/` | command vs query · handler return types · aggregate invariants |
| `alfred-app/src/stores/chatStore.ts` | Zustand setter side effects · selectThread + setComposeMode interaction |
| `aspire-app/store/slices/` | Redux Toolkit slice · Immer immutability · RTK Query cache invalidation |
| Multi-tenant query change | tenantId filter present in EVERY new query · pre/post tenant-A vs tenant-B assertion |

## Step 3 — Verify

For each claim the change/plan makes, walk explicitly:

```
Claim: "Adding rate limit to /api/sale endpoint."
Path:  A handler.ts:42 → B middleware/rateLimit.ts:15 → C redis ZADD → D handler returns 200/429
At D: only fires for sale-band tenants, not all (regex check at C:8). Claim says "endpoint"; reality says "endpoint for one tenant subset".
Therefore: claim INCOMPLETE — verify intended scope.
```

### Edge cases to probe

| Category | Probe |
|---|---|
| **Inputs** | null · empty · unicode · huge · negative · boundary · type-confused |
| **Concurrency** | race · two callers same key · partial failure mid-tx |
| **State** | already exists · doesn't exist yet · half-deleted · stale cache |
| **Errors** | retry storm · downstream 5xx · timeout · network split |
| **Ordering** | ABBA · out-of-order events · message replay |

### Silent changes to flag

- Performance — N+1 introduced · query plan worse · big array allocation
- Error semantics — exception type changed · swallowed where caller expects throw
- Observability — log line moved or removed · trace span dropped
- Contract — return shape changed · new required field
- On-disk / on-wire — DB column dropped · API field renamed

### How are tests doing?

Do the tests actually exercise the traced path?
- Mocks that hide the bug
- Asserts on intermediate state, not outcome
- Happy path only — no edge probes
- Test names diverge from what code actually does

## Step 4 — Findings

Output one tight section per finding. **Severity-order: blocker > major > nit.**

```
[BLOCKER]
Finding: <one sentence>. <file:line>
Why it matters: <consequence — the bug user sees>
Evidence: <trace step OR input that exposes it>
Suggested change: <concrete, minimal>

[MAJOR]
...

[NIT]
...
```

### Severity guide

| Severity | Meaning |
|---|---|
| **blocker** | Will break prod or block correctness · CANNOT commit/ship as-is |
| **major** | Significant bug or maintainability issue · should fix before commit |
| **nit** | Style / readability / minor improvement · optional |

### No rubber-stamps

"LGTM" is NOT an output. If you find nothing:
- State what you traced
- State what you checked
- State what you didn't check (so user judges coverage)

Empty review = signal user that coverage may be incomplete.

## Step 5 — Verdict

One-line verdict + biggest reason.

| Verdict | When |
|---|---|
| **ship** | no blockers · no majors OR all majors have justified accept |
| **fix-then-ship** | majors exist, easy fixes · commit after addressed |
| **rework** | architecture wrong · simpler path exists · resubmit |
| **reject** | premise wrong · should not exist · scope-elsewhere |

Format:
```
VERDICT: <ship | fix-then-ship | rework | reject>
REASON: <one sentence — the single biggest factor>
```

---

## Operating rules

- **No rubber-stamps.** "LGTM" not an output. Always state what you checked.
- **Outsider stance.** Forget who wrote it. Read cold.
- **End-to-end, not diff-local.** Trace through unchanged code on either side of diff.
- **Cite path:line.** Every finding references a concrete file location.
- **Severity-ordered.** Findings sorted blocker → major → nit. Reader stops when they want.
- **Suggest minimal change.** Suggested fix should be the smallest diff that addresses finding.
- **Refuse without artifact + goal.** Step 1 hard gate.

## Anti-patterns

- **LGTM without trace** — empty endorsement, no signal
- **Restating diff back to user** — they wrote it, they know what it says
- **Style nits before correctness blockers** — severity-order required
- **Reviewing diff in isolation** — bugs hide at seams (unchanged neighbor code)
- **Single-claim verification** — verify ALL claims the change makes, not just one
- **Skipping silent-changes check** — perf/error-semantics changes are the dangerous ones

## Common scenarios

| Trigger | Scrutinize output |
|---|---|
| "review my diff before I commit" | run on `git diff` · severity-ordered findings + verdict |
| User pastes a GitHub PR URL | `gh pr diff` · trace · verify · findings · verdict |
| "should I ship this?" | trace + verify + verdict |
| "is there a simpler way?" | step 1 alternative-path question highlighted |
| Pastes design doc | trace proposed flow vs reality · find unstated assumptions |
| Empty PR (whitespace / formatting only) | verdict=ship · note: trivial, no functional review needed |
| Mega-PR (1000+ lines) | refuse → "split into vertical slices first, then re-submit each" |

## Hand off

- Verdict **ship** → terminal (user decides to commit OR hands off to `ship-rocket` for full pre-merge gate w/ tests/lint/migration check)
- Verdict **fix-then-ship** → hand off `tdd-stark` for regression test + fix
- Verdict **rework** → hand off `plan-cap` (--refactor) for redesign
- Verdict **reject** → terminal (user discards)
- Silent change detected (perf / contract / error semantics) → flag to user · suggest `post-mortem` if change ships anyway

## scrutinize-falcon vs ship-rocket — when to use which

| Tool | When |
|---|---|
| **scrutinize-falcon** | Ad-hoc review · before commit · other people's PRs · second opinion · standalone · NO commit/push |
| **ship-rocket** step 2 | Mandatory pre-merge gate · part of full ship flow w/ tests/lint/migration checks · DOES commit + push |

Both call the same scrutinize discipline. Falcon = invokable anywhere · review-only. Ship-rocket = full gate w/ push.

**Use falcon when:** you want feedback but NOT to commit yet.
**Use ship-rocket when:** you've finished and want to merge.

## Cross-ref

- Adopted verbatim from [thananon/9arm-skills](https://github.com/thananon/9arm-skills) `scrutinize`
- `karpathy-rules` Principle 3 (Surgical Changes) — every changed line traces to request
- `ship-rocket` step 2 — same body integrated into pre-merge gate

result: severity-ordered findings + verdict (ship/fix/rework/reject) + cited evidence per finding · user knows what to fix BEFORE commit.

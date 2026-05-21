---
name: post-mortem
description: Canonical engineering record of a fixed bug — root cause, mechanism, fix, validation, slip-through. Adopted from 9arm-skills post-mortem. Refuses to draft without reliable repro / known cause / fix PR / validated fix. Hand off to note-kira for vault sync.
when_to_use: "Keyword triggers — post-mortem, postmortem, RCA, root cause analysis, document this fix, write up root cause, close out this bug, write up bug, slip through, why missed, what happened report, bug retrospective, RCA writeup"
allowed-tools: "Bash(git *) Bash(grep *) Bash(rg *) Bash(ls *) Bash(cat *) Read Edit Write"
disable-model-invocation: false
---

# /post-mortem — Canonical bug-fix record

Adopted from 9arm-skills post-mortem. Engineer-audience. Code identifiers welcome. Refuse to draft without four required inputs.

## When to invoke

- Debug session has landed a real fix, validated.
- User says "/post-mortem" / "write the post-mortem" / "RCA" / "document this fix" / "close out this bug".
- Proactively offer after debug-sherlock step 4 lands a validated fix.

## When NOT to use

- **Bug not fixed yet.** Post-mortem of hypothesis is misleading. Refuse.
- **Customer-visible outage.** That needs separate incident report (timeline, blast radius, paging history, comms). This skill is bug-fix scope. Confirm with user before producing one.
- **Trivial fix** (typo, obvious one-liner). PR description IS the record. No ceremony.

## Required inputs — refuse without these four

Before writing single line, confirm all four. List what's missing, then stop:

- [ ] **Reliable repro** exists (deterministic or high-rate-flake repro next person can run).
- [ ] **Root cause known** (mechanism identified, not hypothesis).
- [ ] **Fix identified** (PR / commit / branch pointer).
- [ ] **Fix validated** (original repro now passes; failing test now succeeds).

Map directly to debug-sherlock steps 1–4. If you came via debug-sherlock, the breadcrumb ledger from step 4 is your raw material — pull from it.

---

## Structure — in this order

**Summary, Root cause, Fix, Validation are mandatory.** Rest conditional but usually present.

### 1. Summary _(mandatory)_

One paragraph. What broke in user/workload terms. What fixed it in one sentence. Branch + PR + owner. Reader who stops here has right answer.

```
**SUMMARY**
<one paragraph>

Branch: <branch>
Commit(s): <SHA list>
Owner: <user>
Service: <service>
Date: <YYYY-MM-DD>
```

### 2. Symptom

What was observed. Test output · error message · log line · perf number · customer report. Concrete identifiers — DO NOT paraphrase.

For your project ai-agent-backend: cite SSE stream excerpt, mode catalog name, agent name (sale/hotel/reservation/forecast), tenant if scoped.

### 3. Root cause _(mandatory)_

Actual bug mechanism. **Code identifiers welcome and expected** — function names, file paths, struct fields, branch conditions, commit SHAs of offending change.

Walk cause chain end-to-end. Most expensive section. Reason post-mortem exists at all.

Future-you will live or die by how clearly this is written. Be specific.

### 4. Why it produced the symptom

Link root cause to symptom. Often non-obvious — bug is in `tadaLaunchPrepare` but visible failure is customer training run hanging hours later. Walk chain so reader who only knows symptom can connect back to cause without re-deriving.

your project-specific common chains:
- Stream gate fail in `ReasoningChannelGate.process` → user sees raw `{` in chat
- Empty-reply-fallback path triggered by fence-wrapped reasoning_content → user sees "I'm having trouble..."
- Multi-tenant query missing `tenantId` → tenant-B sees tenant-A reservation
- TypeORM migration missing index → 504 timeout under prod load

### 5. Fix _(mandatory)_

What changed and **why this change addresses root cause rather than hiding symptom**. Link PR/commit.

If previous fix attempt papered over symptom, name it and explain what was wrong — that history is part of cause.

For your project: if fix bumps MAX_*, RECURSION_LIMIT, retry counts → that is symptom-cover, NOT root cause. Re-open. (diagnose-before-tune rule, absorbed into debug-sherlock step 3.)

### 6. How it was found

Short. The debugging path:
- Repro that made it deterministic
- Tools that cracked it (debugger, source tracing, knob enumeration, in-code instrumentation — debug-sherlock step 2 cascade)
- Hypotheses tried + rejected, one-line reason each (pull from breadcrumb ledger)
- Single experiment that confirmed cause

### 7. How it slipped through _(mandatory)_

The gap analysis:
- Test gap — what test should have caught it but didn't?
- Review gap — what code-review check should have caught it?
- Monitoring gap — what alert / log / metric should have fired?
- Deploy gap — what canary / smoke step missed it?

Each gap → action: write new test, add lint rule, add metric, extend smoke catalog, etc.

This is the section that prevents next bug. Skipping it = bug recurs.

### 8. Validation

Proof the fix works.

- Before: <failing test / failing curl / log line>
- After: <passing test / passing curl / clean log>

Cite test names. Cite assertion output. NOT just "tested, works".

### 9. Related items _(optional)_

- Linked PRs that introduced the bug
- Related bugs found during investigation
- Follow-up work tracked elsewhere

---

## Output destination

Write to Obsidian vault:
```
<vault-root>/microservices/<service>/post-mortems/<YYYY-MM-DD>-<short-slug>.md
```

Frontmatter:
```yaml
---
date: YYYY-MM-DD
service: <service>
branch: <branch>
commits: [<SHA1>, <SHA2>]
mode: <catalog-mode-name>      # if matches debug-sherlock catalog
severity: <minor | major | critical>
slip-through: <test-gap | review-gap | monitoring-gap | deploy-gap>
---
```

---

## Operating rules

- **No phantom post-mortems.** All four required inputs or refuse.
- **Code identifiers always.** Function names, file paths, line numbers.
- **Slip-through analysis is the value-add.** Skipping section 7 = wasted post-mortem.
- **Engineer audience.** No marketing fluff. Future-you will read this in 6 months.
- **Failed previous fixes named.** If symptom-cover was attempted before real fix, document it.
- **Incident scope ≠ post-mortem scope.** Customer-visible outage needs separate incident report.

## Hand off

- Draft complete → hand off `note-kira` to sync to vault
- Slip-through identifies missing smoke step → propose smoke catalog extension
- Slip-through identifies missing test → propose tdd-stark task

result: canonical post-mortem written, vault-bound, slip-through actionable.

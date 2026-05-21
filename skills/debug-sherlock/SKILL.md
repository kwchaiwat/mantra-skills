---
name: debug-sherlock
description: Four-mantra debugging discipline for your project — reproduce, trace fail path, falsify hypothesis, cross-reference breadcrumbs. Adapted from 9arm-skills w/ your project-specific routing for LangGraph multi-agent failures, SSE leaks, Zustand state bugs, multi-tenant leaks. Recite verbatim at session start, then apply 4 steps in order before proposing any fix. Replaces deprecated triage + diagnose-before-tune skills.
when_to_use: "Keyword triggers — debug, debugging, error, bug, broken, not working, why error, why broken, this weird, garbage in chat, leaked thought, empty bubble, static fallback, stale greeting, format A broken, action buttons missing, 504 timeout, recursion limit, agent loop, tool flaky, no progress, context drift, please debug, please help check, ตรวจ debug, ทำไม error, debug หน่อย, แก้ bug"
allowed-tools: "Bash(curl *) Bash(docker *) Bash(grep *) Bash(rg *) Bash(npm *) Bash(npx *) Bash(git *) Bash(ls *) Bash(jq *) Bash(node *) Bash(cat *) Read Edit Write"
disable-model-invocation: false
---

# /debug-mantra — Four-step debugging discipline

Adopted from 9arm-skills debug-mantra. Recite verbatim. Apply in order. NO fix proposals before step 4 complete.

## Recite this — verbatim, as the first thing in your first response

> **Mantra:**
> 1. **First is reproducibility.** Can the issue be reproduced reliably?
> 2. **Know the fail path.** Debugger first; then source trace + knob enumeration; then in-code instrumentation.
> 3. **Question your hypothesis.** What would disprove it?
> 4. **Every run is a breadcrumb.** Cross-reference all of them.

Then begin.

---

## Step 1 — Reproduce reliably

Build a runnable repro before anything else.

- **Reliable repro** → capture exact steps + inputs + env as artifact: failing test, curl, replay-harness run.
- **Flaky repro** → not yet debuggable. Raise rate: loop trigger, parallelize, inject sleeps, narrow timing windows. 50% flake debuggable; 1% not.
- **No repro at all** → STOP. Tell user explicitly. Ask for env access, HAR, log dump, or permission to instrument. DO NOT hypothesize.

Target: fast (1–5s) deterministic pass/fail signal. Pin time, seed RNG, freeze network.

For your project ai-agent-backend repro tactics:
- Replay harness: `npm run test:replay-harness` w/ captured trace ID
- Curl + SSE: `curl -N -X POST ${url} -d '{...}'` capture raw stream
- Tenant-specific: include `bandId` in payload to scope tenant
- AgentTestConsole (UI) for interactive

## Step 2 — Know the fail path

Once reproducible, find WHERE code breaks + WHAT stops it. Escalate tactics in order — try prior before next.

### 2a. Attach a debugger

If env supports — `node --inspect-brk` for backend, devtools for FE. ONE breakpoint > TEN logs.

Do this BEFORE turning knobs.

### 2b. Source trace + knob enumeration

If no debugger or can't reach bug, trace code path end-to-end.

List EVERY knob that influences outcome:
- config flags, env vars, feature toggles
- branch conditions, input shape
- timing, concurrency, build options

Each knob = candidate axis for differential. Flip ONE at a time.

### 2c. In-code instrumentation

If knobs can't move failure, go inside.

`this.logger.debug(...)` at suspected fail site. NOT `console.log`. Tag every probe `[DBG-<id>]` for single-grep cleanup.

### your project routing for step 2

Match symptom → start trace at known thrash-zone file:

| Symptom | Fail-path entry point |
|---|---|
| Raw `{`-fragments / JSON skeleton in chat | `src/modules/langgraph/langgraph-agent.service.ts` — `extractTokenFromEvent`, `ReasoningChannelGate` |
| Static fallback "I'm having trouble..." | `src/shared/utils/empty-reply-fallback.ts` + `extractMessageText` in `message-text.ts` |
| Same greeting repeats turn 2 | `extractLastAiText` in `langgraph-agent.service.ts` |
| Chain-of-thought prose in chat | `extractMessageText` `isJsonEnvelopeLike` gate + `ReasoningChannelGate.process` |
| ```json fence in chat | `ReasoningChannelGate.FENCE_PATTERN` + `stripMarkdownFence` |
| Format A bubble empty | `alfred-app/src/components/format-templates/FormatATemplate.tsx` |
| Action buttons missing | `alfred-app/src/stores/chatStore.ts` (3 sites — `quickActions ?? actionButtons`) |
| 504 / recursion-limit | `src/modules/langgraph/nodes/force-answer.node.ts` + tool-budget injection |
| Multi-tenant data leak | grep `createQueryBuilder\|find(\|findOne(` w/o `tenantId` filter |
| Zustand state cancellation | trace `selectThread` + `setComposeMode` interaction (click-path audit) |
| Redux state cancellation | trace dispatch chain in `store/slices/<feature>.slice.ts` |

For ANY symptom NOT in catalog → dispatch Explore subagent: `Agent(subagent_type: "Explore", description: "Find fail path for <symptom>", prompt: "<context>")`.

### Anti-pattern: do NOT bump limits

If symptom is timeout / recursion / tool-loop-exhaust:
- DO NOT increase `MAX_*`, `*_TIMEOUT`, `RECURSION_LIMIT`, retry counts, budget rules.
- Diagnose root cause FIRST.
- diagnose-before-tune absorbed here at step 3 (falsify hypothesis).

## Step 3 — Falsify hypothesis

When candidate root cause surfaces, SCRUTINIZE before testing.

- Does it explain symptom end-to-end? Walk it through.
- What is simplest PROOF? Cleanest DISPROOF?
- Run **disproof first**. Hypothesis survives → real. Dies → saved hours chasing phantom.
- Generate **3–5 ranked hypotheses**, NOT one. Single-hypothesis thinking anchors on first plausible idea.

If user pasted broken output, output hypotheses verbatim:
```
**Found N candidate root causes:**

| Rank | Mode | Why this matches | First file to check |
|---|---|---|---|
| 1 | <mode> | <evidence> | <path:line> |
| 2 | … | … | … |
```

Wait for user nod OR strongest disproof. Then proceed step 4.

## Step 4 — Every run is a breadcrumb

Maintain running **ledger** of every experiment.

Format:
```
[<timestamp>] knob=<value> result=<pass/fail/X> rules_in=<...> rules_out=<...>
```

- New hypothesis surfaces → walk ledger. Does it hold for EVERY prior observation, not just most recent?
- Past run contradicts → hypothesis is wrong or incomplete. Refine or discard.
- When in doubt, design the **single experiment** whose outcome makes it certain. Run that next instead of churning adjacent runs.
- Update ledger after every run. It is memory across session.

Ledger is raw material for post-mortem skill (handoff target).

---

## Operating rules

- **No fix proposals before step 4 ledger.** Speculation without breadcrumbs = phantom hunt.
- **Bumping limits forbidden.** If user demands MAX_* bump → answer "diagnose first; bump is symptom-cover, not fix."
- **3+ hypotheses minimum.** Single hypothesis = anchoring bug.
- **Real repro mandatory.** No repro → STOP, request artifacts.
- **Code identifiers when reporting.** Always cite `path:line`, never paraphrase.

## Hand off

- Fix identified + tested → hand off `tdd-stark` (write regression test first)
- Fix lands + validated → hand off `post-mortem` (with breadcrumb ledger as input)
- Repro impossible → STOP and report to user

result: root cause known + validated + breadcrumb ledger ready for post-mortem.

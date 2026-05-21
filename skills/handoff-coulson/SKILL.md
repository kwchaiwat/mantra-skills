---
name: handoff-coulson
description: Agent Coulson briefs the next pilot — compact current session into handoff doc that lets a fresh agent resume cold. Five-step mantra — scope · extract facts · decisions · pending · briefing. Refuses without session context. Terminal output — handoff doc saved to vault. Use when context full, tasks queued for later, or transitioning between agents.
when_to_use: "Slash triggers — /handoff-coulson, /handoff, /coulson. Hero triggers — coulson handoff, agent briefing, brief next pilot, shield handoff. Keyword triggers — handoff, compact session, summary for next session, save state for later, before context window full, pick up next session, transition to new agent, ส่งต่อ, สรุปเซสชั่น, briefing. Does NOT fire on — single-task summaries (use note-kira) · architecture docs (use note-kira) · post-bug record (use post-mortem)."
allowed-tools: "Bash(date *) Bash(git *) Bash(ls *) Bash(cat *) Read Write Edit"
disable-model-invocation: false
---

# /handoff-coulson — Session compact (Agent Coulson)

Coulson hands the next pilot a mission briefing. You write a handoff doc that lets a fresh agent resume cold without re-reading 2-hour conversation.

Use when context is filling · session is wrapping · transitioning to new agent · pausing work for later.

## Recite verbatim as first response

> **Handoff mantra:**
> 1. **Scope.** What this session is about, in one sentence.
> 2. **Extract facts.** What got DONE — file paths · commits · pushed URLs · concrete artifacts.
> 3. **Decisions.** Choices made + rationale. Anchors future-you.
> 4. **Pending.** What's NOT done yet · ordered by next-action priority.
> 5. **Briefing.** Write handoff doc to vault. Reader = fresh agent w/ zero context.

Then begin.

---

## Step 1 — Scope

One sentence — what was this session ABOUT?

### Refuse-without

If session has no actionable history (greeting only, single question answered, no work artifacts created) → STOP. Output:
```
**Handoff refuses empty session.**
No work done yet to brief. Either:
- Continue work first, then handoff
- OR use /note-kira for single-decision record
```

### Sharp scope statement

```
SESSION: <one sentence — what was the goal/theme>
DATE: <YYYY-MM-DD>
DURATION: <approximate>
OUTCOME: <one phrase — shipped / partial / blocked>
```

## Step 2 — Extract facts

CONCRETE artifacts only. No prose summary.

| Category | What |
|---|---|
| **Files written** | absolute path per file |
| **Files edited** | path + 1-line what-changed |
| **Commits** | hash + one-line subject |
| **Pushed remotes** | URL + branch + tag |
| **Vault docs written** | wiki-link |
| **Skills added/changed** | name + active count |
| **Config changed** | file + key + value |

Skip: rationale (step 3 covers) · what's not done (step 4 covers).

## Step 3 — Decisions

Choices made + rationale. Anchors future-you.

Format:
```
DECISION: <what was chosen>
ALTERNATIVES considered: <X, Y, Z>
WHY this one: <one-sentence rationale>
TRADE-OFF accepted: <what was given up>
```

Don't capture: trivial defaults (template=mantra-5 because most skills use it).

## Step 4 — Pending

What's NOT done yet. ORDERED by next-action priority.

| Priority | Item | Estimate | Blocker |
|---|---|---|---|
| 1 | next action | time | if any |
| 2 | ... | ... | ... |

For each pending, capture:
- What: one sentence
- Why deferred: time / budget / dependency / decision-pending
- Resume command: literal CLI / file path / skill invocation

## Step 5 — Briefing (write handoff doc)

Output to vault:
```
/Users/chaiwat/Documents/Obsidian Vault/claude-code/<NN>-session-<YYYY-MM-DD>-handoff.md
```

Where `<NN>` = next sequential 2-digit number (check existing dir).

### Doc structure

```markdown
---
date: YYYY-MM-DD
type: session-handoff
tags: [claude-code, handoff, session-end]
status: ready-to-resume
references: [[prior-notes]]
---

# Session handoff — <one-line summary>

## Scope
<step 1 output>

## What got done (facts)
<step 2 output>

## Decisions made
<step 3 output>

## Pending (next-session work)
<step 4 output>

## Resume instructions
1. Open Claude Code in <project dir>
2. <restore env if needed>
3. Read this doc + [[<prior session note>]]
4. Pick priority #1 from Pending table
5. Continue chain

## Cross-ref
- <prior handoff if any>
- <vault notes affected>
- <commits / PRs / URLs>
```

### Hand off doc principle

Write so reader has ZERO context. Assume next-session agent never saw conversation. Every claim cited w/ file path or URL. Decisions explain WHY in addition to WHAT.

---

## Operating rules

- **Refuse empty session.** Step 1 hard gate.
- **Facts not narrative.** Step 2 = bullet list of artifacts, not story.
- **Why behind every decision.** Step 3 trade-off line mandatory.
- **Ordered pending.** Step 4 numbered by next-action priority, not by topic.
- **Vault path mandatory.** Step 5 outputs to `claude-code/<NN>-session-<date>-handoff.md`.
- **Zero-context reader.** Write so fresh agent resumes cold.

## Anti-patterns

- **Prose summary** — "We talked about X and then Y" → use bullet artifacts
- **Hidden assumptions** — referencing "the bug" without naming file:line
- **Missing trade-offs** — decision w/o what-was-given-up = future-you reopens debate
- **Vague pending** — "finish the thing" → must be specific resume command
- **Mixed scope** — handoff for multi-session work fine, but tag each priority

## Common scenarios

| Trigger | Handoff output |
|---|---|
| "context full, save for later" | full 5-step mantra → vault note |
| "I'm tired, pick up tomorrow" | full 5-step mantra → vault note |
| "transitioning to new Claude session" | full mantra + extra: token-budget warning if any |
| "/compact about to fire" | quick mantra · prioritize step 4 pending |
| Just one bug fixed, no other work | refuse → use post-mortem instead |
| Architecture decision only | refuse → use note-kira w/ decision tag |

## When to use vs note-kira vs post-mortem

| Skill | Scope |
|---|---|
| **handoff-coulson** | Multi-task session bridge → next session resumes cold |
| **note-kira** | Single decision · architecture note · ongoing knowledge |
| **post-mortem** | Bug fix recorded w/ slip-through analysis |

Coulson = mission briefing (multi-item · ordered priorities · resume-ready).
Kira = lab notebook entry (single fact · cross-linked).
Post-mortem = NTSB report (bug + root cause + fix + gap).

## Hand off

- Doc written → terminal (user closes session OR starts new)
- Resume next session → read this note + execute priority #1
- If multi-day pause → chain to next handoff w/ `references: [[prior]]`

## Cross-ref

- `note-kira` — single-decision vault sync (different scope · use for sub-items)
- `post-mortem` — bug record (different shape)
- `skills-help` — quick reference card if next-session forgets skill names

result: handoff doc written to vault · fresh agent can resume cold · pending ordered by next-action.

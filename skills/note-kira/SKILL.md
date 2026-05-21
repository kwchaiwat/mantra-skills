---
name: note-kira
description: Canonical record sync to Obsidian vault — detect topic, route to vault subfolder, frontmatter + tags, cross-link memory + drive, update MEMORY.md if memory-worthy. Replaces deprecated take-note-ob v1 with explicit numbered mantra. Triggered after commits via hook + manual invocation.
when_to_use: "Keyword triggers — take note, take-note-ob, document this, note this, save to vault, update vault, sync vault, vault sync, obsidian note, write up, document the session, /take-note-ob, doc this fix, doc this change, journal, summarize session, สรุปงาน, จดบันทึก"
allowed-tools: "Bash(git *) Bash(ls *) Bash(cat *) Bash(mkdir *) Bash(date *) Read Edit Write"
disable-model-invocation: false
---

# /take-note-ob — Vault sync mantra

Sync canonical record to Obsidian vault. Runs after commit (hook) or by explicit invoke.

## Recite verbatim as first response

> **Note mantra:**
> 1. **Detect topic** → route to vault subfolder.
> 2. **Cross-link** memory + obsidian (+ drive if public).
> 3. **Frontmatter** w/ date + tags + service + commit.
> 4. **Update** MEMORY.md index if memory-worthy.
> 5. **Sync** triggered after every commit (hook reminds).

Then write.

---

## Step 1 — Detect topic → vault subfolder

Vault root: `/Users/chaiwat/Documents/Obsidian Vault/`

Routing:
| Topic | Subfolder |
|---|---|
| Service code change | `radiant1/microservices/<service>/changes/` |
| Architecture decision | `radiant1/radiant1-architecture/decisions/` |
| Bug post-mortem | `radiant1/microservices/<service>/post-mortems/` |
| Daily standup | `radiant1/microservices/standup/` |
| Planning doc | `radiant1/radiant1-architecture/plan/` |
| QA / audit | `radiant1/radiant1-architecture/qa/` |
| Feedback to architecture | `radiant1/microservices/automation/architecture-improvements/` |
| Personal / cross-project | `personal/` |

If unclear → ASK user. Do NOT auto-route ambiguous.

## Step 2 — Cross-link

Add links to related notes via `[[wikilink]]` syntax.

- Related code → link to source file path (Obsidian supports file links)
- Related vault note → `[[other-note-name]]`
- Related memory entry → cite memory file path
- Related Google Drive doc (if public) → embed link

Don't bury links at bottom. Include inline where mentioned.

## Step 3 — Frontmatter

Mandatory:
```yaml
---
date: YYYY-MM-DD          # absolute date, not "today"
service: <service-name>    # if service-specific
branch: <branch>           # if code-related
commits: [<SHA-list>]      # if code-related
tags: [<tag1>, <tag2>]     # for Obsidian search
type: <change | decision | post-mortem | standup | plan | qa>
---
```

For change/decision/post-mortem: include `type` exactly matching subfolder convention.

## Step 4 — Update MEMORY.md if memory-worthy

Memory criteria (write to memory ONLY if):
- User behavior preference revealed (feedback type)
- Project fact non-obvious from code (project type)
- External reference path (reference type)
- User profile detail (user type)

NOT memory-worthy (write only to vault):
- One-off code change
- Daily standup
- Single bug fix (post-mortem in vault is sufficient)
- Architecture detail derivable from graphify

If memory-worthy → write separate memory file under `~/.claude/projects/-Users-chaiwat-Desktop-radiant1-microservices/memory/` + add index line to MEMORY.md.

Memory file format (see auto-memory system docs):
```yaml
---
name: <kebab-slug>
description: <one-line summary>
metadata:
  type: <user | feedback | project | reference>
---

<memory content with [[cross-links]]>
```

MEMORY.md index line:
```
- [Title](file.md) — one-line hook
```

## Step 5 — Sync — hook + manual

PostToolUse hook fires after `git commit` to remind sync.

Manual invoke when:
- Session ends without commit but has decisions worth recording
- Standup-style daily summary
- Pure planning session (no code change yet)

After write:
- Confirm vault file exists: `ls "<path>"`
- Confirm MEMORY.md updated (if applicable): `grep "<file>" MEMORY.md`
- Output to user: `Vault: <path>` (+ memory file path if applicable)

---

## Note format (default)

```markdown
---
date: YYYY-MM-DD
service: <service>
branch: <branch>
commits: [<SHA>]
tags: [<tag>]
type: change
---

# <short title>

## What changed
- bullet

## Why
- bullet (motivation, not user request echoed)

## How
- key files: [[<file>:<line>]]
- key decisions
- test coverage

## Next
- bullet (follow-up if any)

## Related
- [[<other-note>]]
- <external link>
```

---

## Operating rules

- **Absolute dates.** Convert "today" / "yesterday" / "Thursday" to YYYY-MM-DD at write time.
- **Vault writes are durable.** Once written, persist across sessions. Memory is for cross-session behavior.
- **Don't duplicate code.** Note describes intent + decisions. The code IS the truth. Link, don't paste.
- **Frontmatter is searchable.** Tags + service + type drive Obsidian queries.
- **No clutter.** If session produced nothing memorable, don't write a note. Hook reminder is fine to ignore.

## Hand off

- After post-mortem → already vault-sync, but ensure MEMORY.md not bloated w/ post-mortem entries
- After ship → updates `<service>/changes/<date>.md`
- After plan session → updates `<service>/plan/<date>.md`
- After standup → `<service>/standup/<date>.md`

result: vault + memory in sync with session work.

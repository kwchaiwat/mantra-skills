---
name: forge-stark
description: Tony Stark's workshop — meta-skill that builds new your project 9arm-style skills. Generates SKILL.md with numbered mantra, refuse-without gate, hand-off chain, hero alias, and keyword triggers. Auto-registers in SKILLS-INDEX. Refuses without skill name + power description + downstream chain. Use when user wants to add a new skill to the roster — say "build skill for X", "create new skill", "forge skill", "/forge-stark".
when_to_use: "Slash triggers — /forge-stark, /forge, /smith. Hero triggers — stark forge, forge new skill, build skill, create skill, new skill, scaffold skill, add skill. Keyword triggers — make a skill, want a skill for, skill that does X, build mantra, new mantra, add mantra, สร้าง skill, เพิ่ม skill ใหม่. Does NOT fire on routine task requests — only when user explicitly asks to CREATE a skill artifact."
allowed-tools: "Bash(ls *) Bash(mkdir *) Bash(cat *) Bash(grep *) Bash(rg *) Bash(find *) Bash(sed *) Read Edit Write"
disable-model-invocation: false
---

# /forge-stark — Skill workshop (Tony Stark's forge)

Builds new your project 9arm-style skills. Same workshop where Stark iterates MK1 → MK50.

Goal: scaffold a new `~/.claude/skills/<name>/SKILL.md` w/ canonical mantra structure, register in `SKILLS-INDEX.md`, ready to invoke same session.

## Recite verbatim as first response

> **Forge mantra:**
> 1. **Intent.** Skill name · power statement · downstream chain · hero alias.
> 2. **Template.** Pick template — mantra-5 / mantra-4 / mantra-7 / structure-only / one-shot.
> 3. **Frontmatter.** Compose `name · description · when_to_use · allowed-tools · disable-model-invocation`.
> 4. **Mantra body.** Numbered steps + refuse-without gate + hand-off.
> 5. **Register.** Add to SKILLS-INDEX flow chains + verify catalog load.

Then begin.

---

## Step 1 — Intent

Refuse without these four inputs.

### Required

- [ ] **Skill name** — kebab-case, optional `-<hero>` suffix (e.g. `pms-scaffold-shuri`)
- [ ] **Power statement** — one sentence of what it does
- [ ] **Downstream chain** — handoff target(s) OR "terminal" if no chain
- [ ] **Hero alias** — match power to popular hero (stark/cap/sherlock/etc) OR "none" to skip

If any missing → STOP. Output:
```
**Forge refuses without 4 inputs.**
Need:
- skill name: <kebab-case>
- power: <one sentence>
- chain: <next skill OR terminal>
- hero: <hero or none>
```

### Optional

- bucket — engineering / productivity / reference / personal / utility (default: productivity)
- refuse-without condition — hard gate at step 1 of generated mantra
- Thai keywords — add Thai triggers if user works in Thai

## Step 2 — Template

Pick mantra structure by skill type:

| Template | Step count | Use when |
|---|---|---|
| **mantra-5** | 5 numbered steps | most workflow skills (default — `plan-cap`, `tdd-stark`, `smoke-spidey`, `ship-rocket`, `note-kira`, `rag-tune`, `agent-eval-vision`, `tenant-leak-audit`) |
| **mantra-4** | 4 steps | diagnostic / falsify skills (`debug-sherlock`) |
| **mantra-7** | 7 steps | classifier / router skills (`intake-jarvis`, `fury`) |
| **structure-only** | 9-section structure | record skills (`post-mortem`) |
| **one-shot** | terminal display | reference cards (`skills-help`) |

User picks OR default to mantra-5.

## Step 3 — Frontmatter

Compose YAML block. Required fields:

```yaml
---
name: <skill-name>
description: <hero> — <one-sentence power>. <when/why use it>. Refuses without <condition>. Use when <trigger>.
when_to_use: "Slash triggers — /<skill-name>, /<hero>. Hero triggers — <hero> <verb>, <synonyms>. Keyword triggers — <bare keywords incl. Thai>. Does NOT fire on <negative>."
allowed-tools: "<scoped Bash patterns> Read Edit Write"
disable-model-invocation: false
---
```

Fill rules:
- `name` = kebab-case matches dir name
- `description` starts w/ hero metaphor, ends w/ trigger phrase
- `when_to_use` includes slash + hero + 5-10 keyword triggers + Thai if user speaks Thai
- `allowed-tools` only scoped Bash globs needed (e.g. `Bash(curl *)` not bare `Bash`)
- `disable-model-invocation: false` enables auto-trigger

## Step 4 — Mantra body

Emit canonical structure:

```markdown
# /<skill-name> — <hero> <power phrase>

<one-paragraph framing — why skill exists, what user gets>

## Recite verbatim as first response

> **<Skill> mantra:**
> 1. **<Step 1 verb>.** <One-line summary>.
> 2. **<Step 2 verb>.** <One-line summary>.
> 3. ...

Then begin.

---

## Step 1 — <Name>

<details>

### Refuse-without

<what's required at step 1>

If missing → STOP. Output `**Refusing <skill> without <X>.**`

## Step 2 — <Name>
<details>

## Step 3 — <Name>
<details>

## Step 4 — <Name>
<details>

## Step 5 — <Name>
<details — usually handoff>

---

## Operating rules

- <rule 1>
- <rule 2>
- <rule 3>

## Hand off

- <condition> → <downstream skill>
- <condition> → <other skill>
- <terminal condition> → done

result: <what user gets after skill completes>.
```

## Step 5 — Register

Four updates required EVERY TIME a new skill is forged:

### 5a. SKILLS-INDEX.md

Append to bucket section:
```
<skill-name>/  ← <bucket bucket>, <hero> — <one-line summary>
```

Add flow chain if relevant:
```
### Flow X — <FLOW NAME>
\`\`\`
<entry> → <skill-name> → <downstream>
\`\`\`
Use when: <trigger>.
```

### 5b. Sync to marketplace + apply generic transform

```bash
cp -r ~/.claude/skills/<skill-name> <project-root>/mantra-skills/skills/
cd <project-root>/mantra-skills
sed -i.bak \
  -e 's|your project|your project|g' \
  -e 's|project-patterns|project-patterns|g' \
  -e 's|<project-root>/|<project-root>/|g' \
  -e 's|multi-tenant SaaS|multi-tenant SaaS|g' \
  skills/<skill-name>/SKILL.md && rm skills/<skill-name>/SKILL.md.bak
```

### 5c. **MANDATORY** — update README.md roster

User instruction: "please update readme everytime".

Add new skill to correct bucket section in `mantra-skills/README.md`:
- **Engineering mantras (N)** — workflow skills
- **Productivity mantras (N)** — non-code workflow
- **Domain-specific mantras (N)** — adapt to stack
- **Reference + meta (N)** — universal · template · meta
- **Utility (N)** — explicit-invoke utilities

Bump bucket count in header (e.g. `### Engineering mantras (5)` → `### Engineering mantras (6)`).
Bump total count at top (e.g. `21 numbered-mantra Claude Code skills` → `22`).

Add row in format:
```markdown
- **<skill-name>** — <hero> · <one-line power statement>
```

If skill adds new flow chain, append to "## Flow chains" section too.

### 5d. Commit + push marketplace

```bash
cd <project-root>/mantra-skills
git add skills/<skill-name> README.md
git -c user.email="kw.chaiwat@gmail.com" -c user.name="kw-chaiwat" commit -m "feat(<skill-name>): add <hero> <power phrase>

<2-line description>

Mantra steps:
- <bullet per step>

Refuses without <gate condition>.
"
git push
```

### 5e. Verify catalog load

After all 4 updates, output:
```
Forged: ~/.claude/skills/<skill-name>/SKILL.md
Frontmatter parses: <yes/no — check first 10 lines>
Marketplace synced: <commit hash>
README updated: bucket count bumped
GitHub pushed: <repo URL>
Catalog load: invoke /<skill-name> in fresh session to verify
Handoff target: <downstream skill> exists at ~/.claude/skills/<downstream>/
```

### Optional: update vault handbook

If user runs forge often, suggest periodic handbook sync via note-kira. Not auto — too many edits.

---

## Operating rules

- **Refuse without 4 inputs.** Step 1 hard gate.
- **One skill per invocation.** No batch.
- **Hero match must be popular.** Reject niche heroes (Madrox, Heimdall, Sue Storm). Suggest swap to popular tier (Stark, Cap, Sherlock, Spidey, Strange, Fury, Vision, Wanda, etc).
- **No duplicate names.** Step 1 must check `ls ~/.claude/skills/` AND `~/.claude/skills/_archived/`. Refuse if collision.
- **Mantra recital mandatory in generated skill.** Step 4 template enforces verbatim quote block.
- **Refuse-without gate mandatory.** Generated skill must have hard gate at step 1.
- **Hand-off section mandatory.** Even terminal skills need explicit "done" / "(terminal)" note.

## Anti-patterns

- **Inventing new hero from niche source** — stay top-tier (use skills-help card hero list)
- **Skipping refuse-without** — every mantra has a gate
- **Vague keyword triggers** — must include 5-10 specific keywords, not "build", "do"
- **Forgetting SKILLS-INDEX update** — orphaned skill = invisible
- **Skipping handoff section** — break flow chain

## Common scenarios

| User asks | Forge output |
|---|---|
| "build skill for new PMS connector scaffolding" | name=`pms-scaffold-shuri` · template=mantra-5 · chain=plan-cap → tdd-stark · hero=Shuri (Wakanda lab builds tech) |
| "create skill for code review of TypeScript files" | name=`review-vision` · template=mantra-5 · chain=note-kira · hero=Vision (verdict weigh) |
| "make a skill that runs cypress E2E" | name=`e2e-flash` · template=mantra-5 · chain=ship-rocket · hero=Flash (speed click) |
| "add skill for daily metrics report" | name=`metrics-fury` · template=one-shot · chain=terminal · hero=Fury (briefing) |
| "build skill, no hero" | hero=none, name stays bare kebab-case |

## Hand off

- New skill created → `smoke-spidey` lite mode (catalog-load check)
- Multi-skill batch ask → refuse, one-per-invocation
- User wants quality audit → `ecc:skill-stocktake`
- User wants to also update vault handbook → `note-kira`

result: new SKILL.md scaffolded, frontmatter validated, SKILLS-INDEX updated, ready to invoke.

---
name: daily-standup
description: Prepare daily standup briefing for the your project team. Reads recent git activity, Kanban status, open PRs, and cron job results to generate a concise standup report. Fires when user asks for standup, daily briefing, team update, or what happened yesterday.
when_to_use: "Keyword triggers: standup, daily standup, stand-up, team update, daily briefing, what happened yesterday, what did we do, morning briefing, สรุปทีม, เช้านี้ทำอะไร, standup brief"
allowed-tools: "Read Bash(git log *) Bash(git status *) Bash(gh pr list *) Bash(gh pr view *) Bash(ls *) Bash(cat *) Bash(grep *) Bash(wc *) Bash(date *)"
---

# /daily-standup — Team standup briefing generator

## Purpose

Generate a concise daily standup briefing for the your project team. Combines git activity, Kanban status, PR status, and cron job results into a structured report.

## Workflow

### Step 1 — Gather signals (parallel)

```bash
# Git activity (last 24h)
git -C <project-root>/microservices log --since='24 hours ago' --oneline --all | head -30

# Open PRs
gh pr list --state open --json number,title,author,createdAt --limit 10

# Recent cron results (from automation-logs)
ls -lt ~/Documents/Obsidian\ Vault/radiant1/automation-logs/ 2>/dev/null | head -10

# Kanban status (if accessible)
curl -s http://localhost:3484/api/tasks?status=in_progress 2>/dev/null | head -50
```

### Step 2 — Read context

- Read latest automation log summary
- Read any recent Obsidian daily notes
- Check for any blocked/failed cron jobs

### Step 3 — Generate standup report

Output in this structure:

```
**Daily Standup — <date>**

**Yesterday:**
- <N> commits across <N> branches
- <key change #1>
- <key change #2>
- PR #<N> <status>

**Today:**
- <planned work #1>
- <planned work #2>

**Blockers:**
- <blocker #1> (if any)
- None if clear

**AI Activity:**
- <N> cron runs completed
- <N> tasks auto-reviewed
- <notable AI action>
```

### Step 4 — Output options

- Default: output to chat
- If user says "save" or "vault": also write to Obsidian daily note
- If user says "thai": output in spoken Thai

## Anti-patterns
- Don't just list commits — summarize what changed and why
- Don't include every detail — keep it scannable in 2 minutes
- Don't forget to mention blockers — that's the most important part

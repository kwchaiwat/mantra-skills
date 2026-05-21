# Customization guide

Fork → customize → use in your stack.

## Step 1 — Fork & symlink

```bash
git clone https://github.com/kw-chaiwat/mantra-skills.git my-mantra-skills
cd my-mantra-skills

for s in skills/*/; do
  name=$(basename "$s")
  ln -sf "$(pwd)/$s" ~/.claude/skills/$name
done

ls ~/.claude/skills/ | wc -l  # should show 21+ skills
```

## Step 2 — Customize `project-patterns`

Most important customization. Your service-convention reference.

Edit `skills/project-patterns/SKILL.md`:

1. Update section headers (A, B, C, D, E) to match YOUR services
2. Update path routing rules to YOUR repo layout
3. Replace stack details (NestJS / React / Next / etc) w/ yours
4. Replace conventions — commit scopes, branch naming, file layout, test placement

## Step 3 — Customize domain skills

### `rag-tune`
- Swap vendor names (Azure AI Search · Cohere) → Pinecone / Weaviate / Vespa / etc
- Update knob allowlist to your retriever config

### `agent-eval-vision`
- Swap framework → LangChain / DSPy / custom
- Update tool-call F1 matrix per your eval setup

### `tenant-leak-audit`
- Swap ORM query patterns (TypeORM example) → Prisma / Drizzle / Sequelize / raw SQL
- Single-tenant projects → skip or repurpose for permission audit

## Step 4 — Customize `debug-sherlock` catalog

Replace failure-mode table w/ YOUR recurring bug patterns:

```markdown
| Symptom | Fail-path entry point |
|---|---|
| <your bug symptom> | <your-file>:<key-function> |
```

Build over time — each post-mortem adds one row.

## Step 5 — Rename heroes (optional)

Hero suffixes (-cap · -stark · -sherlock · -spidey · -rocket · etc) are decorative. Keep, drop, or swap:

| Original | Drop | Swap example |
|---|---|---|
| plan-cap | plan | plan-strange |
| tdd-stark | tdd | tdd-yoda |
| debug-sherlock | debug | debug-holmes |

To rename: `mv skills/<old>/ skills/<new>/` + update `name:` in SKILL.md frontmatter + propagate cross-refs.

## Step 6 — Tune triggers

Each SKILL.md has `when_to_use:` keyword triggers (English + Thai). Customize:
- Drop Thai if not used
- Add your project's vocabulary
- Update channel names if cross-ref

## Step 7 — Drop skills you don't need

Common drops:
- `rag-tune` — no RAG
- `agent-eval-vision` — no multi-agent
- `tenant-leak-audit` — single-tenant
- `cua-driver` — not macOS / no GUI
- `graphify` — small codebase
- `daily-standup` — use other standup tools

`rm -rf skills/<unused>/` + update `skills-help/SKILL.md` reference card.

## Step 8 — Try one flow end-to-end

After fork + customize:

```
1. Open Claude Code in your repo
2. Paste a real bug from last week
3. Say "/debug-sherlock <bug paste>"
4. Walk the 4-step mantra
5. After fix: /post-mortem
6. After commit: /note-kira
```

If something feels rigid → edit that mantra. Make it yours.

## FAQ

**Do I need all 21 skills?**
No. Even 5-6 (intake-jarvis · plan-cap · tdd-stark · debug-sherlock · ship-rocket) cover 80% of value.

**Can I add my own skill?**
Yes. Use `forge-stark` — meta-skill that builds new skills following mantra template.

**Will upstream updates conflict with my customizations?**
`project-patterns` and `debug-sherlock` heavily customized — keep in your fork.
Workflow skills (plan-cap · tdd-stark · ship-rocket · karpathy-rules) rarely need per-project tuning — pull updates safely.

**Why "9arm" + "Karpathy"?**
- 9arm = tiny-set workflow design from Thananon Aphithanawat's `thananon/9arm-skills`
- Karpathy = LLM coding principles preventing silent overengineering · adjacent-drift · vague refactors

## License

MIT — fork freely, no attribution required (but appreciated).

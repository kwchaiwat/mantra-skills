---
name: project-patterns
description: Path-routed reference for all your project microservices conventions — commit scopes, branch names, file layout, test placement, framework patterns, LLM provider rules, multi-tenant guards. Load matching section based on file path being edited. Sections — A=ai-agent-backend, B=hotel-service, C=reservation-service, D=alfred-app, E=aspire-app, F=cross-cutting (deploy/migration/redis/postgres). Activate when working on any file under <project-root>/microservices/ — keyword triggers include service name (ai-agent-backend, hotel-service, reservation-service, alfred-app, aspire-app), file ops (where put file, scaffold, add module, create handler, branch name), commit conventions, NestJS/Vite/Next.js patterns, TypeORM migration, BullMQ queue, multi-tenant check.
when_to_use: "file path matches microservices/ai-agent-backend/ → load A; microservices/hotel-service/ → load B; microservices/reservation-service/ → load C; microservices/alfred-app/ → load D; microservices/aspire-app/ → load E; cross-cutting (migration, redis, postgres, docker, deploy) → load F. Keyword triggers — where put file, scaffold, module layout, branch name, commit scope, test location, handler, aggregate, embed build, format template, action button, redux slice, channel manager, quicksight, typeorm migration, bullmq, postgres index, redis pattern, multi-tenant"
version: 2.0.0
disable-model-invocation: false
---

# your project Patterns — Path-Routed Reference

Single source of truth for service conventions. Read matching section ONLY.
Replaces 5 deprecated skills (ai-agent-backend-patterns, hotel-service-patterns, reservation-service-patterns, alfred-app-patterns, aspire-app-patterns).

## How to use

1. Identify file path being edited.
2. Match to section letter (A–F).
3. Load that section + Section F (cross-cutting) if backend.
4. Do NOT load other sections.

```
microservices/ai-agent-backend/**       → Section A + F
microservices/hotel-service/**          → Section B + F
microservices/reservation-service/**    → Section C + F
microservices/alfred-app/**             → Section D
microservices/aspire-app/**             → Section E
no specific path / cross-service work   → Section F only
```

---

## Section A — ai-agent-backend

**Stack:** NestJS 10 + TypeScript + TypeORM + Jest + BullMQ + Azure OpenAI + Kimi + LangGraph multi-agent.

### Commit conventions

Conventional Commits w/ mandatory lowercase scope on non-trivial commits.

Type frequency (top): `feat(scope)`, `fix(scope)`, `merge:`, `perf(scope)`, `test(scope)`, `chore(scope)`.

Scope vocabulary (reuse, do not invent):
- Domain: `langgraph`, `agents`, `chat`, `stream`, `rescue`, `observability`, `kb`, `tools`, `tool-metadata`, `prompt`
- Infra: `config`, `infra`, `deploy`, `auth`, `whatsapp`, `routing`, `db`, `seed`, `migration`
- Cross-cutting: `perf`, `cost`, `tenant`, `multi-tenant`

### Branch naming

```
epic/<area>                          # long-lived
feat/<scope>/<short-desc>            # feature work
fix/<scope>/<short-desc>             # bug fix
perf/wave-<N>-<area>                 # perf wave per memory
gate/G<N>-<short-desc>               # gate-tagged ship
task/T<N>-#<num>-<short-desc>        # task identifier
```

### File layout (NestJS module)

```
src/
├── modules/
│   └── <feature>/
│       ├── <feature>.module.ts
│       ├── <feature>.controller.ts
│       ├── <feature>.service.ts
│       ├── dto/
│       ├── entities/
│       ├── interfaces/
│       └── <feature>.service.spec.ts   ← adjacent test (Jest)
├── shared/
│   ├── utils/
│   ├── guards/
│   ├── interceptors/
│   └── filters/
├── config/
└── database/
    ├── migrations/
    └── seeds/
```

### Test placement

- Unit: adjacent `.spec.ts` next to subject file.
- E2E: `test/` dir, `*.e2e-spec.ts`, config `test/jest-e2e.json`.
- Replay harness: `test/replay-harness/`.
- Load test: `scripts/load-test.ts` w/ npm script `test:load:<agent>`.

### LangGraph multi-agent architecture

Agents: sale / hotel / reservation / forecast (+ retrieval / revenue / etc).
4-layer stream pipeline: LLM → backend gate → SSE event splitter → FE consumer.
Catalog 8 failure modes (see debug-sherlock skill).

Key files (DO NOT edit without tracing callers):
- `src/modules/langgraph/langgraph-agent.service.ts` (`extractTokenFromEvent`, `ReasoningChannelGate`)
- `src/shared/utils/empty-reply-fallback.ts`
- `src/shared/utils/message-text.ts`
- `src/modules/langgraph/nodes/force-answer.node.ts`

### LLM provider rules

- Azure OpenAI = primary. Kimi = fallback.
- Model routing by complexity: simple → Haiku-tier / GPT-4o-mini, complex → Sonnet-tier / GPT-4o.
- Prompt caching for system prompts >1024 tokens.
- Budget tracking per session via cost-tracking columns.

### MCP tool surface

- Tools defined in `src/modules/tools/` w/ Zod schema.
- Tool metadata in DB (config-plane already shipped per memory).
- Test tool via `AgentTestConsole` or chat-sync POST (NOT webhook repoint).

### TypeORM migration safety

- `migrate:generate --name <Name>` → review up + down BOTH manually.
- NEVER `synchronize: true` in prod.
- For multi-tenant tables: ensure `tenantId` column + index added in same migration.
- Test on staging snapshot before deploy.

### BullMQ + Redis

- Queue defs in `src/modules/queues/`.
- Worker concurrency tuned per queue (default 5).
- Use `JobsOptions.attempts` + exponential backoff.
- DLQ pattern: failed jobs → separate queue for investigation.

### Multi-tenant guards

EVERY query MUST filter by tenantId. EVERY new endpoint MUST require tenant context.

Check before commit:
```
grep -rn "createQueryBuilder\|find(\|findOne(" src/modules/<your-module>/ \
  | grep -v ".spec.ts" \
  | grep -v "tenantId"
```
Zero hits required (or explicit justification in PR).

---

## Section B — hotel-service

**Stack:** NestJS + CQRS + DDD. Focus: seasonal pricing + compset room-type mapping.

### File layout (DDD/CQRS)

```
src/
├── domain/
│   ├── aggregates/
│   │   └── <aggregate>.aggregate.ts
│   ├── entities/
│   ├── value-objects/
│   └── events/
├── application/
│   ├── cqrs/
│   │   ├── commands/
│   │   │   └── handlers/<command>.handler.ts
│   │   └── queries/
│   │       └── handlers/<query>.handler.ts
│   └── dtos/
├── infrastructure/
│   └── repositories/
└── presentation/
    └── controllers/
```

### Test placement

Adjacent Jest specs:
- `<aggregate>.aggregate.spec.ts`
- `<command>.handler.spec.ts`
- `<query>.handler.spec.ts`

### CQRS rules

- Commands return void or aggregate ID, NEVER full aggregate.
- Queries return DTO, NEVER entity.
- Handlers single-responsibility — one command/query each.
- Aggregate enforces invariants in constructor + state-mutating methods.

### Seasonal pricing pattern

- Pricing periods stored as value objects w/ start/end dates.
- Rate calculation via domain service, not aggregate (pure function).
- Compset comparison uses room-type mapping table.

### Commit scopes

`feat(pricing)`, `feat(compset)`, `fix(rate)`, `feat(season)`, `chore(deps)`.

---

## Section C — reservation-service

**Stack:** NestJS + DDD-style + AWS-native (EKS deploy + SQS, NOT Azure/BullMQ).

### File layout

```
src/
├── domain/
│   ├── validations/
│   └── entities/
│       └── builders/
├── service/
│   ├── queue/                   ← SQS consumers
│   └── handlers/
└── presentation/
```

### SQS patterns

- Consumer per queue in `src/service/queue/`.
- Idempotency key required on every message (use reservation ID).
- DLQ on max retries (3 default).
- Visibility timeout > expected processing time.

### Validation pattern

- Domain validations in `src/domain/validations/`.
- Pure functions, no IO.
- Composable: each validation returns `Result<T>`.

### Entity builders

- Fluent builder pattern in `src/domain/entities/builders/`.
- Required fields enforced via type system (typed builder).

### Deploy

EKS via Helm chart (NOT AKS like ai-agent-backend). Check `k8s/` or `helm/` for chart.

### Commit scopes

`feat(reservation)`, `fix(validation)`, `feat(sqs)`, `feat(builder)`, `chore(deps)`.

---

## Section D — alfred-app

**Stack:** React 18 + Vite + Zustand + multi-brand embed builds.

### File layout

```
alfred-app/
├── src/
│   ├── App.tsx
│   ├── embed.ts                  ← brand embed entry
│   ├── embed-nav.ts              ← nav-only embed
│   ├── components/
│   │   ├── chat/
│   │   ├── format-templates/    ← FormatA/B/C/D/E/F templates
│   │   └── shared/
│   ├── stores/
│   │   └── chatStore.ts         ← Zustand store
│   ├── api/
│   └── utils/
├── vite.config.ts
└── vite.embed.config.ts          ← multi-brand build config
```

### Multi-brand build

Each brand = separate Vite build w/ env vars:
```
APP_NAME=admiral-premier EMBED_ENTRY=src/embed.ts \
  EMBED_NAME=embed-admiral-premier vite build -c vite.embed.config.ts
```
Brands: `admiral-premier`, `demo`, `real-estate`, `nav`.
Add new brand → npm script in `package.json` + entry in `build:all-embeds`.

### Format templates

`src/components/format-templates/FormatATemplate.tsx` etc. Each template handles specific message shape.

Common failure modes:
- `format-a-shape-drift` — variableFormat/roomOptions/rooms key drift
- `action-buttons-key-drift` — `quickActions ?? actionButtons` fallback missing in `chatStore.ts`

### Zustand store rules

- One source of truth per state slice.
- NEVER mutate state — return new object.
- Side effects via `subscribe` or middleware, NOT in setters.
- Check `chatStore.ts` for `selectThread` / `setComposeMode` interaction (click-path audit pattern).

### Commit scopes

`feat(admin)` dominates per git history. Also: `feat(embed)`, `fix(chat)`, `feat(format)`, `fix(store)`.

### NO automated tests — known gap

Per project memory. Manual testing only. When adding feature, add at least smoke test if possible.

---

## Section E — aspire-app

**Stack:** Next.js 14 + Redux Toolkit + AWS SDK (Cognito + QuickSight) + Cypress E2E only.

### File layout

```
aspire-app/
├── app/                          ← Next.js 14 app router
├── components/
├── store/                        ← Redux Toolkit
│   ├── slices/
│   └── api/                      ← RTK Query
├── hooks/
├── lib/
├── cypress/
│   ├── e2e/
│   └── support/
└── next.config.ts
```

### Redux Toolkit rules

- Slices in `store/slices/<feature>.slice.ts`.
- Async via createAsyncThunk OR RTK Query (prefer RTK Query for HTTP).
- NEVER mutate state — Redux Toolkit uses Immer but still write as if immutable.

### AWS SDK usage

- Cognito for auth → wrapped in `lib/cognito.ts`.
- QuickSight embed via `amazon-quicksight-embedding-sdk`.
- Embed token fetched server-side via API route.

### Channel manager / RMS factor patterns

Per memory — domain is hotel revenue/occupancy + channel mappings (Staah, Siteminder, etc.) + RMS factors.

Mapping tables in `lib/mappings/` or DB schema TBD.

### Cypress tests

- Specs in `cypress/e2e/<feature>.cy.ts`.
- Page Object Model NOT enforced — flat selectors common.
- Custom commands in `cypress/support/commands.ts`.

### Commit conventions

Scope-free per git history. Use `feat: <desc>`, `fix: <desc>`.

Very large repo (3241 commits) — branch per feature mandatory, avoid long-running branches.

---

## Section F — Cross-cutting

### Deployment

- ai-agent-backend / hotel-service → Azure AKS, ARM64 Docker.
- reservation-service → AWS EKS via Helm.
- alfred-app → static build, served via CDN per brand.
- aspire-app → Next.js production server or static export.

### Database

- Primary: PostgreSQL (managed).
- Cache: Redis (BullMQ queues for ai-agent-backend).
- Search: Azure AI Search (RAG layer for ai-agent-backend).
- Re-rank: Cohere.

### Docker

- ARM64 base images.
- Multi-stage builds: builder → runtime.
- Distroless or Alpine runtime preferred.
- Healthcheck endpoint mandatory.

### Lint + type checks

Before commit:
- `npm run lint` — ESLint must pass zero errors.
- `npx tsc --noEmit` — TypeScript must pass.
- `npm test` — Jest must pass (where tests exist).

Project rule reminders from CODING_CONVENTIONS.md:
- no `any` types
- no nested ifs / nested loops
- blank line before `return`
- no `console.log` (use `this.logger.log`)
- ~20-line methods
- no magic numbers
- no DB calls in loops (O(n))
- repository pattern
- no JSDoc / comments
- SOLID principles
- check CQRS before implementing

### Cost / token routing

LLM call rules (apply across ai-agent-backend):
- Classify task complexity: simple / medium / complex.
- Route: simple → Haiku 4.5 (90% Sonnet quality, 3x cheaper), medium → Sonnet 4.6, complex → Opus 4.7 (reasoning only).
- Cache system prompts via prompt caching.
- Budget tracking: track tokens + cost per session in DB.
- Log routing decision for later analysis.

### Multi-tenant isolation (project-critical)

Every new endpoint / query MUST filter by tenant.
Pre-ship check via smoke step 3 (tenant-A vs tenant-B isolation assert).

### Vault sync (durable rule)

After every commit affecting code → update Obsidian vault under `<vault-root>/microservices/`.
PostToolUse hook reminds automatically.

### Push policy (durable rule)

Commit but NEVER push without explicit user ask.
NEVER push to main.
User merges feature branches themselves — push branch + report + stop.

### Graphify refresh

After major code changes, refresh graphify:
```bash
python3 graphify-out/process_service.py microservices/<service> \
  "<vault-root>/_graphify" microservices
python3 graphify-out/rebuild_combined.py
```
Use `$(cat graphify-out/.graphify_python)` instead of `python3` in this repo.

---

## Section G — CONTEXT.md ubiquitous language

Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) "ubiquitous language" pattern.

### Why

Domain jargon table prevents 20-word explanations every turn. AI uses YOUR project's terms (not generic LLM jargon). Token savings + naming consistency across humans and AI.

### Where

`<service-root>/CONTEXT.md` — per-service domain vocabulary. Sits next to CLAUDE.md. Read AUTO by other skills (plan-cap step 1 if exists).

### Format

```markdown
# CONTEXT — <service> Ubiquitous Language

## Domain terms

| Term | Definition | NOT to be confused with |
|---|---|---|
| **<noun>** | <one-sentence meaning in YOUR project> | <generic term that sounds similar> |

## Verbs / actions

| Verb | What it means here |
|---|---|
| **<verb>** | <project-specific action> |

## Anti-vocabulary

DO NOT use these generic terms when YOUR project has a specific one:
- generic word → use OUR term: <specific>

## Cross-ref

- Architecture: <docs path>
- Code entry point: <file:line>
```

### Example (LLM/agent backend)

```markdown
# CONTEXT — <agent-backend>

| Term | Definition | NOT |
|---|---|---|
| **band** | a tenant (brand / chain) | "customer" |
| **agent** | a LangGraph node w/ system prompt + tool list | OpenAI Assistant |
| **format A/B/C** | message bubble template for chat output | UI component |
| **gate** | ReasoningChannelGate — strips raw JSON from stream | auth gate |
| **rescue** | empty-reply fallback path | retry logic |
| **thrash zone** | files that flap often during stream-pipeline bug hunts | hot files |

DO NOT use: "tenant" → use "band". "tool call" → use "tool invocation". "model" alone → say "LLM client" or "provider config".
```

### How to use

1. New repo? `touch CONTEXT.md` and seed top 5-10 terms in first session.
2. Every time AI uses generic term that has project-specific synonym → add to anti-vocab.
3. Bi-monthly review — prune dead terms, add new ones.
4. Reference in other skills — `plan-cap` step 1 reads CONTEXT.md if exists.

### Refuse-by-rule

If editing this service and CONTEXT.md is missing → suggest creating one as separate task (low priority for fresh repos · high priority for high-jargon services).

### Token savings

| Without CONTEXT.md | With CONTEXT.md |
|---|---|
| "the tenant — by which I mean a brand or chain…" (~15 tokens) | "band" (1 token) |
| 8 turns of disambiguation per session | 0 |
| Generic terms shipped to docs | Project-specific terms persist |

---

## Hand off

- After scaffolding/file placement → hand off to `tdd-stark` for the actual write
- After identifying failure mode → hand off to `debug-sherlock`
- Cross-service work touching 3+ sections → hand off to `avengers`

result: section loaded, conventions in scope.

---
name: research-strange
description: Doctor Strange — multi-source research mantra. Peers across web (Exa) + codebase (graphify + ripgrep) + library docs (Context7) + Obsidian vault to deliver decision-ready brief w/ citations. Five-step ordered mantra. Refuses without research question (vague topics blocked). Terminal output — brief w/ answer · evidence · confidence · gaps. Use when user wants to LOOK SOMETHING UP w/ sources, not act on it.
when_to_use: "Slash triggers — /research-strange, /research, /strange. Hero triggers — strange research, doctor strange, peer multiverse, what should we use, what's best. Keyword triggers — research, look up, what is, compare options, find out, investigate, dig into, multi-source, exa search, context7, cite sources, evidence-based, decision brief, market scan, library docs, what does X do, how does Y work, ค้นคว้า, หาข้อมูล, วิจัย. Does NOT fire on — debug (use debug-sherlock) · code search alone (use grep) · already planning a feature (use plan-cap step 2 search)."
allowed-tools: "Bash(curl *) Bash(grep *) Bash(rg *) Bash(find *) Bash(ls *) Bash(cat *) Bash(python3 *) Bash(gh *) Read Edit Write"
disable-model-invocation: false
---

# /research-strange — Multi-source research (Doctor Strange)

Doctor Strange peered through 14M timelines. You peer through 4 sources: web · code · docs · vault. Output = decision-ready brief, not a wiki page.

Goal: answer one research question w/ cited evidence across sources, flag contradictions, declare confidence.

## Recite verbatim as first response

> **Research mantra:**
> 1. **Question.** State research question in one sentence. Refuse if vague topic.
> 2. **Route.** Pick sources — web · code · docs · vault. Default = all 4 parallel.
> 3. **Gather.** Spawn parallel queries. Capture URL + timestamp + excerpt.
> 4. **Synthesize.** Cross-reference findings · flag contradictions · cite each claim.
> 5. **Brief.** Decision-ready output — answer · evidence · confidence · gaps.

Then begin.

---

## Step 1 — Question

Refuse without a sharp research question.

### Sharp question (accept)

- "Should your project use Drizzle ORM instead of TypeORM for hotel-service?"
- "What's the latest Cohere rerank threshold default in v3 API?"
- "Has anyone solved LangGraph stream-render leak in NestJS w/ SSE?"
- "Compare Pinecone vs Azure AI Search for multi-tenant RAG."

### Vague topic (refuse)

- "Tell me about ORMs"
- "Research RAG"
- "Look at Cohere"

If vague → STOP. Output:
```
**Research refuses vague topic.**
Sharpen: state question that has yes/no OR pick-one-of-N answer.
Examples:
- "X vs Y for use-case Z?"
- "Is X still recommended in 2026?"
- "What's the latest <specific> API for <specific case>?"
```

### Capture context

- Decision deadline (today / week / future)?
- Decision scope (exploration / commit-to-ship)?
- Budget for time (5min skim / 30min deep)?

## Step 2 — Route

Pick sources. Default = all 4 parallel for unknown territory.

| Source | Tool | Use when |
|---|---|---|
| **Web** | `ecc:exa-search` (neural) OR `ecc:deep-research` (firecrawl+exa) | current state · 2024-2026 facts · GitHub repos |
| **Code** | `rg` / `grep` / `graphify-out/graph.json` | "how do we do X today" · cross-service impact · existing pattern |
| **Library docs** | `ecc:documentation-lookup` (Context7) | API reference · framework features · version-specific behavior |
| **Vault** | Obsidian `radiant1/` + memory files | prior decisions · post-mortems · arch notes |

Routing rules:
- "should we X" → all 4
- "what does X library do" → docs + web
- "how do we X today" → code + vault
- "did we already decide X" → vault + code
- "what's current state of X tech" → web + docs

## Step 3 — Gather

Run queries in parallel. ONE agent per source for speed.

```bash
# Parallel structure
agent_web    = Agent(subagent_type: "search-specialist", prompt: "<question> via exa")
agent_code   = Agent(subagent_type: "Explore", prompt: "<question> via rg over microservices/")
agent_docs   = invoke ecc:documentation-lookup w/ framework + question
agent_vault  = grep across /Users/chaiwat/Documents/Obsidian\ Vault/radiant1/
```

For each finding, capture:
- **Claim** (one sentence)
- **Source** (URL · file:line · vault note path)
- **Date** (publish or last-modified)
- **Confidence** (high · medium · low based on source authority)

Output structure per source:
```
WEB findings (Exa):
- claim — source URL — date — confidence
- ...

CODE findings (grep/graphify):
- pattern — file:line — confidence
- ...

DOCS findings (Context7):
- API behavior — framework version — link
- ...

VAULT findings:
- prior decision — vault note path — date
- ...
```

## Step 4 — Synthesize

Cross-reference + flag contradictions.

### Convergence
```
Web + docs + code agree → high-confidence finding
```

### Divergence
```
Web says X, code says Y → flag contradiction, investigate cause:
- outdated code? → upgrade candidate
- web outdated? → check date
- different context? → scope-mismatch
```

### Gap analysis

Mark what's UNKNOWN — important for confidence calibration.

Output table:
```
| Finding | Evidence | Sources confirm | Contradictions | Gaps |
|---|---|---|---|---|
| <claim> | <URL + file:line> | web, docs, code | (none) | (none) |
| <claim> | <URL> | web only | code uses old pattern | no benchmark data |
```

## Step 5 — Brief

Decision-ready output. NOT a wiki.

Format:
```
**Research brief: <question>**

**Answer:** <one paragraph — directly answers question>

**Evidence (top 3-5):**
1. <claim> — <source>
2. <claim> — <source>
3. <claim> — <source>

**Confidence:** <high · medium · low> (N of 4 sources agree)

**Gaps:**
- <unknown 1>
- <unknown 2>

**Recommended next step:**
- ship / experiment / wait-for-evidence / discard
```

### Confidence calibration

| Sources agreeing | Confidence |
|---|---|
| 4/4 | high — proceed |
| 3/4 | medium-high — proceed w/ note |
| 2/4 | medium — explore further OR small experiment |
| 1/4 | low — refuse to recommend, more evidence needed |
| 0/4 conclusive | no signal — different question needed |

---

## Operating rules

- **Refuse vague topics.** Step 1 gate is hard.
- **Cite every claim.** No floating statements. URL · file · vault path required.
- **Date-stamp findings.** Web changes fast — claim from 2022 ≠ claim from 2026.
- **Parallel by default.** Web/code/docs/vault simultaneously, not sequential.
- **Synthesize don't dump.** Step 5 brief ≤ 1 page. User wants answer, not bibliography.
- **Flag contradictions explicitly.** Hidden conflict = bad ship later.
- **Confidence floor.** Below 2/4 sources = state "low confidence, need more evidence".

## Anti-patterns

- **Single-source research** — one Google query ≠ research
- **No citations** — opinions without source = pseudo-research
- **Bibliography dump** — list 50 links, no synthesis = output user can't act on
- **Stale findings** — citing pre-2024 source for "current state" question
- **Confirmation bias** — only searching to confirm prior belief
- **Skipping vault** — your project may have already decided this last quarter

## Common scenarios

| Question | Route | Brief output |
|---|---|---|
| "Drizzle vs TypeORM for new service" | web + docs + code | answer pick + 3 evidence + confidence + tradeoffs |
| "Has cohere v4 changed rerank API?" | docs (Context7) + web | API delta + breaking changes + migration steps |
| "Did we decide on multi-tenant strategy already?" | vault + code | prior decision + commits + current pattern |
| "What chunk size for Azure AI Search RAG?" | web + docs + vault (rag-tune baselines) | recommended range + your project baseline + tune candidate |
| "Best React state lib for alfred-app multi-brand" | web + code (Zustand current) | comparison + migration cost + recommend |

## Cross-ref to other skills

- After research → `plan-cap` (if action follows)
- Research for bug context → invoke as side-step from `debug-sherlock` step 2
- Research before tuning → invoke from `rag-tune` step 2 hypothesis
- Research as part of pre-build → already covered by `plan-cap` step 2; this skill is for standalone research questions

## Hand off

- Brief complete → terminal (user decides next: plan-cap if act / discard if no signal / note-kira if record)
- Question requires implementation choice → suggest `plan-cap` after brief
- Question requires bug context → suggest `debug-sherlock` as caller
- Question expanded into multi-question research → recommend `avengers` to parallel-dispatch

result: research brief delivered w/ cited evidence · confidence calibrated · gaps named · next-step suggested.

---
name: rag-tune
description: RAG pipeline tuning mantra for your project ai-agent-backend — Azure AI Search + Cohere rerank + CRAG + chunker sizing. Five-step ordered mantra. Baseline → hypothesis → probe on golden set → compare → ship-or-rollback. Refuses to tune without golden eval set + baseline metrics.
when_to_use: "Keyword triggers — rag, RAG, retrieval, rerank, cohere, azure search, ai search, vector search, chunker, chunk size, embeddings, top-k, threshold, CRAG, retrieval quality, recall, precision, hit rate, irrelevant chunks, missing chunks, RAG bug, retrieval drift, knowledge base tuning, kb tuning, tune retrieval, tune RAG, RAG eval"
allowed-tools: "Bash(curl *) Bash(npm *) Bash(npx *) Bash(jq *) Bash(grep *) Bash(rg *) Bash(node *) Bash(ls *) Bash(cat *) Bash(python3 *) Read Edit Write"
disable-model-invocation: false
---

# /rag-tune — RAG pipeline tuning mantra

Recite. Apply in order. Refuse to tune without golden set + baseline.

## Recite verbatim as first response

> **RAG tune mantra:**
> 1. **Baseline.** Capture current recall · precision · MRR · cost-per-query on golden set.
> 2. **Hypothesis.** ONE variable changed per cycle. State expected delta + sign.
> 3. **Probe.** Run eval on golden set under variant. Capture same metrics.
> 4. **Compare.** Delta vs baseline. Stat-sig check. Cost delta. Latency delta.
> 5. **Ship-or-rollback.** Pass-gate or revert. Update baseline if pass.

Then begin.

---

## Step 1 — Baseline

Cannot tune what you can't measure. Refuse without baseline.

### Required artifacts

- **Golden eval set** at `microservices/ai-agent-backend/test/replay-harness/golden/<scope>.jsonl`
- Each item: `{ query, expected_doc_ids[], expected_answer_fragment, tenant_id }`
- Minimum N=50 items, mix of intents (sale / hotel / reservation / forecast)
- Tenant-scoped (do NOT mix tenants in single eval run)

If golden set absent → STOP. Output:
```
**Refusing rag-tune without golden set.**
Need: 50+ query/expected pairs at <path>.
Build via: npm run replay-harness:extract <trace-id-range>
```

### Capture baseline metrics

Run eval on CURRENT config:
```bash
npm run test:replay-harness -- --golden=<path> --report=/tmp/rag-baseline-$(date +%Y%m%dT%H%M%S).json
```

Metrics to record:
| Metric | Definition | Target |
|---|---|---|
| Recall@k | Fraction expected docs in top-k retrieved | ≥0.85 |
| Precision@k | Fraction retrieved that ARE expected | ≥0.7 |
| MRR | Mean Reciprocal Rank of first correct doc | ≥0.7 |
| Answer F1 | Token overlap w/ expected fragment | ≥0.6 |
| Cost/query | Embedding + search + rerank + LLM tokens | track absolute |
| p95 latency | End-to-end RAG response time | < SLO |

Output baseline table. ALL future work compares to this.

## Step 2 — Hypothesis

ONE variable per cycle. Multi-knob tuning = anchoring on noise.

Variable allowlist (your project RAG knobs):

| Knob | Where | Typical range |
|---|---|---|
| `chunk_size_tokens` | chunker config | 200–800 |
| `chunk_overlap_tokens` | chunker config | 0–200 |
| `top_k_retrieval` | Azure AI Search call | 5–50 |
| `top_n_rerank` | Cohere rerank call | 3–15 |
| `rerank_threshold` | Cohere score cutoff | 0.3–0.7 |
| `crag_trigger` | CRAG confidence threshold | 0.4–0.7 |
| `embedding_model` | Azure OpenAI deployment | ada-002 / text-embedding-3-* |
| `query_rewrite` | pre-retrieval LLM rewrite | on/off |
| `hyde_enabled` | hypothetical doc embedding | on/off |
| `filter_by_metadata` | tenant + freshness filters | bool combo |

State hypothesis format:
```
HYPOTHESIS: lowering chunk_size_tokens from 600 → 400 will increase precision@5 by ≥5%
            because smaller chunks reduce off-topic content in top results
            EXPECTED DOWNSIDE: recall@5 may drop ≤2% (some context fragmented)
            COST IMPACT: chunk count doubles → 2x embedding storage
```

If hypothesis vague ("try different chunking") → STOP, sharpen first.

## Step 3 — Probe

Apply variant change to config. NOT deployed code. Use override config:

```bash
# Variant config file
echo '{ "chunk_size_tokens": 400, "chunk_overlap_tokens": 50 }' > /tmp/rag-variant.json

# Run eval w/ override
npm run test:replay-harness -- \
  --golden=<path> \
  --config-override=/tmp/rag-variant.json \
  --report=/tmp/rag-variant-$(date +%Y%m%dT%H%M%S).json
```

Same metrics. Same N. Same tenant. Same time-of-day if possible (Azure throttling varies).

Variants run sequentially, NOT concurrent (rate limits + measurement noise).

## Step 4 — Compare

Output delta table:

| Metric | Baseline | Variant | Delta | Sig |
|---|---|---|---|---|
| Recall@5 | 0.84 | 0.86 | +0.02 | yes (p<0.05, N=50) |
| Precision@5 | 0.68 | 0.74 | +0.06 | yes |
| MRR | 0.71 | 0.73 | +0.02 | borderline |
| Answer F1 | 0.62 | 0.65 | +0.03 | yes |
| Cost/query | $0.012 | $0.018 | +50% | n/a |
| p95 latency | 1.8s | 2.1s | +17% | n/a |

### Decision matrix

| Quality delta | Cost delta | Latency delta | Verdict |
|---|---|---|---|
| Up sig + | Flat / down | Flat / down | **SHIP** |
| Up sig + | Up | Flat | **SHIP if cost <2x baseline** |
| Up sig + | Up >2x | Up | **REWORK** — find cheaper variant |
| Flat | Any | Any | **REVERT** — hypothesis disproven |
| Down | Any | Any | **REVERT** — hypothesis disproven w/ regression |

Compare-against-baseline rule:
- Quality metrics must use same N and same golden set
- Cost compared as $/query (not absolute) for stability
- Latency p95 over ≥20 sample queries

## Step 5 — Ship-or-rollback

### Ship path (verdict = SHIP)

Hand off to `tdd-stark`:
- Write regression test asserting new metric floor (e.g. precision@5 ≥ 0.72)
- Update production config
- → smoke-spidey → ship-rocket → note-kira

Update baseline: rename `/tmp/rag-variant-*.json` → committed `test/replay-harness/baseline-<date>.json`. Future cycles compare to new baseline.

### Rollback path (REVERT / REWORK)

- Discard variant config
- Update breadcrumb ledger w/ disproven hypothesis (for debug-sherlock cross-reference)
- Loop back to step 2 with revised hypothesis

NEVER ship variant w/o regression test on new metric floor. Else next regression goes undetected.

---

## Anti-patterns

- **Multi-knob tuning** → can't attribute delta to single change
- **Re-using same golden across many cycles** → overfit. Refresh golden set quarterly w/ new prod queries.
- **Tuning on synthetic queries** → divergence from real tenant usage. Pull goldens from replay harness trace IDs.
- **Ignoring cost delta** → quality up 5% but cost up 10x = bad ship. Cost is a metric.
- **No tenant scope** → cross-tenant pollution gives false-good metrics.

---

## Common patterns (your project-specific)

| Scenario | Likely root cause | First knob to try |
|---|---|---|
| Customer says "wrong hotel info" | Recall miss | top_k 10→20 + rerank_threshold lower |
| Customer says "irrelevant context" | Precision low | chunk_size smaller + rerank top-n tighter |
| Slow response | Rerank too greedy | top_n_rerank reduce |
| Cost spike | Over-retrieval | top_k smaller + filter_by_metadata |
| Stale answers | Freshness filter | add metadata filter on `updated_at` |
| CRAG always triggers | Threshold too high | crag_trigger lower (0.6 → 0.5) |

---

## Operating rules

- **Refuse without golden set.** Step 1 hard gate.
- **One knob per cycle.** Step 2 enforce.
- **Same N, same scope.** Step 3-4 enforce.
- **Cost is a metric.** Step 4 decision matrix includes $.
- **Regression test on ship.** Step 5 mandatory.
- **Multi-tenant scope.** All eval runs tenant-scoped, no cross-pollution.

## Hand off

- Verdict SHIP → `tdd-stark` (write regression test) → smoke-spidey → ship-rocket
- Verdict REVERT → loop to step 2 w/ revised hypothesis
- Repeated REVERT (3 cycles) → escalate to architecture (consider model/index swap, not config tune)

result: variant evaluated against baseline, ship/revert verdict with delta evidence.

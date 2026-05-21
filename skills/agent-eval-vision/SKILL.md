---
name: agent-eval-vision
description: Multi-agent A/B eval mantra for your project ai-agent-backend — head-to-head comparison of sale / hotel / reservation / forecast agent variants on golden dataset. Five-step ordered mantra. Golden ready → baseline run → variant run → compare matrix (RAGAS + cost + latency) → verdict (ship / rework / reject). Refuses without golden dataset + both variants deployable. Hand off to ship (variant green) or debug-sherlock (regression).
when_to_use: "Keyword triggers — agent eval, eval agent, A/B agent, compare agent, sale agent, hotel agent, reservation agent, forecast agent, agent regression, agent comparison, golden set, RAGAS, eval-driven, eval driven, multi-agent eval, before merge agent, agent quality, sale v2, hotel v2, agent variant, langgraph eval"
allowed-tools: "Bash(npm *) Bash(npx *) Bash(jq *) Bash(grep *) Bash(rg *) Bash(curl *) Bash(node *) Bash(python3 *) Bash(ls *) Bash(cat *) Bash(mkdir *) Bash(date *) Read Edit Write"
disable-model-invocation: false
---

# /agent-eval — Multi-agent A/B eval mantra

For LangGraph agent changes in ai-agent-backend. Refuse to merge agent change without baseline-vs-variant eval.

## Recite verbatim as first response

> **Agent eval mantra:**
> 1. **Golden ready.** Tenant-scoped dataset w/ expected answer + tool-call trace.
> 2. **Baseline run.** Current agent on golden. Capture metrics.
> 3. **Variant run.** New agent on SAME golden. Capture same metrics.
> 4. **Compare matrix.** Quality (RAGAS) · cost · latency · tool-loop count. Decision matrix.
> 5. **Verdict.** SHIP · REWORK · REJECT. Hand off accordingly.

Then begin.

---

## Step 1 — Golden ready

Per-agent dataset required. Refuse without.

### Dataset structure

`microservices/ai-agent-backend/test/replay-harness/golden/<agent>.jsonl`:

```jsonl
{"id": "sale-001", "tenant_id": "<id>", "user_message": "What's the cheapest room tonight?", "expected": {"answer_contains": ["<room-name>", "<price>"], "tool_calls": ["get_inventory", "get_pricing"], "format": "format-a", "agent_used": "sale"}}
{"id": "sale-002", ...}
```

Per agent:
- N ≥ 30 (more for sale agent — primary surface)
- Span the catalog of failure modes (greet, book, modify, cancel, pricing query)
- Span tenant variety (≥ 3 tenants representative of band-mix)
- Include adversarial cases (multi-turn confusion, language mix, out-of-domain)

If absent → STOP. Output:
```
**Refusing agent-eval without golden set for <agent>.**
Need: ≥30 cases at test/replay-harness/golden/<agent>.jsonl
Build via: npm run replay-harness:extract --agent=<agent> --tenant=<id> --since=<date>
```

### Variant config

Both variants MUST be runnable in isolation:

```
# Baseline: current main branch agent
git checkout main

# Variant: branch under eval
git checkout <feature-branch>
```

OR via feature flag if both live in same branch:
```
AGENT_VARIANT=baseline npm run test:replay-harness ...
AGENT_VARIANT=variant npm run test:replay-harness ...
```

## Step 2 — Baseline run

Lock environment (same Azure OpenAI deployment, same Kimi model, same tenant config):

```bash
# Capture environment digest
git rev-parse HEAD > /tmp/eval-env-baseline.txt
node -e "console.log(JSON.stringify(process.env.AZURE_OPENAI_DEPLOYMENT))" >> /tmp/eval-env-baseline.txt

# Run baseline
mkdir -p /tmp/agent-eval-$(date +%Y%m%dT%H%M%S)/baseline
AGENT_VARIANT=baseline npm run test:replay-harness -- \
  --golden=test/replay-harness/golden/<agent>.jsonl \
  --report=/tmp/agent-eval-XXX/baseline/report.json
```

Per-case capture:
| Field | Why |
|---|---|
| `agent_used` | did router pick correct agent? |
| `tool_calls[]` | tool plan correctness |
| `tool_loop_count` | did force-answer.node fire? |
| `answer` | actual reply text |
| `format` | format-a/b/c/d/e/f detection |
| `cost_tokens` | input + output + cache hit ratio |
| `latency_ms` | end-to-end + per-node breakdown |
| `trace_id` | observability anchor |

## Step 3 — Variant run

SAME golden. SAME env vars. SAME tenant config. Same time window if possible (Azure throttling varies hourly).

```bash
mkdir -p /tmp/agent-eval-XXX/variant
AGENT_VARIANT=variant npm run test:replay-harness -- \
  --golden=test/replay-harness/golden/<agent>.jsonl \
  --report=/tmp/agent-eval-XXX/variant/report.json
```

Run baseline + variant **sequentially**, not concurrent (rate limits + noise).

## Step 4 — Compare matrix

Compute per-case + aggregate metrics. Output table:

### Quality matrix

| Metric | Baseline | Variant | Delta | Sig |
|---|---|---|---|---|
| Correct-agent rate | 94% | 96% | +2% | yes |
| Tool-call F1 | 0.82 | 0.85 | +0.03 | yes |
| Answer-contains hit rate | 0.78 | 0.81 | +0.03 | borderline |
| Format detection accuracy | 0.91 | 0.93 | +0.02 | borderline |
| Tool-loop exhaust count | 2 / 30 | 0 / 30 | -2 | yes (rare event, but eliminating matters) |
| Empty-reply-fallback fires | 1 / 30 | 0 / 30 | -1 | yes |

### Cost matrix

| Metric | Baseline | Variant | Delta |
|---|---|---|---|
| Median tokens/turn | 2400 | 2100 | -12% |
| Median cost/turn | $0.014 | $0.012 | -14% |
| Cache hit ratio | 38% | 52% | +14pp |

### Latency matrix

| Metric | Baseline | Variant | Delta |
|---|---|---|---|
| p50 latency | 1.4s | 1.3s | -7% |
| p95 latency | 3.2s | 2.9s | -9% |
| p99 latency | 6.1s | 4.8s | -21% |

### Failure-mode matrix (debug-sherlock catalog cross-ref)

| Mode | Baseline count | Variant count |
|---|---|---|
| streaming-gate-leaked-fragments | 0 | 0 |
| empty-reply-fallback-fired | 1 | 0 |
| stale-greeting-rescue-leak | 0 | 0 |
| chain-of-thought-leak | 0 | 0 |
| markdown-fence-not-stripped | 0 | 0 |
| format-a-shape-drift | 1 | 0 |
| action-buttons-key-drift | n/a | n/a |
| force-answer-budget-exhaust | 2 | 0 |

### Decision matrix

| Quality | Cost | Latency | Failure modes | Verdict |
|---|---|---|---|---|
| Up sig + | Flat / down | Flat / down | Same / better | **SHIP** |
| Up sig + | Up <30% | Flat | Same / better | **SHIP w/ cost note** |
| Up sig + | Up ≥30% | Flat | Same / better | **REWORK** — find cheaper |
| Flat | Down | Flat | Better | **SHIP** (cost win) |
| Flat | Flat | Flat | Same | **REJECT** — no signal |
| Down | Any | Any | Any | **REJECT** — regression |
| Any | Any | Up ≥20% | Any | **REWORK** — latency regression |
| Any | Any | Any | Worse | **REJECT** — failure mode added |

## Step 5 — Verdict

### SHIP

Hand off to `tdd-stark`:
- Write regression test asserting baseline-or-better on key metric (e.g. answer-contains ≥ 0.78)
- → smoke-spidey → ship-rocket

Update baseline: variant becomes new baseline. Commit golden + new baseline metrics under `test/replay-harness/baselines/<date>-<agent>.json`.

### REWORK

Variant has quality wins but unacceptable cost/latency tax. Loop to plan + tdd-stark to find cheaper variant. Re-run agent-eval.

### REJECT

Variant adds regression OR fails to move quality. Discard variant. Update breadcrumb ledger (debug-sherlock cross-ref) w/ what was tried.

---

## Anti-patterns

- **Eval on dev workstation only** → Azure-region latency differs from prod. Run from same region if possible.
- **Cherry-pick metrics** → if variant wins on 1 metric, loses on 3, summary IS the loss. No selective reporting.
- **Re-running eval until pass** → noise + Azure variance. ≤3 runs per variant; if results swing wildly, expand N.
- **No tenant scope** → mixed-tenant golden hides multi-tenant bugs.
- **Skipping failure-mode catalog check** → quality metrics can move up while NEW failure mode appears. Always check catalog count delta.

---

## Common eval scenarios (your project)

| Change | Eval focus |
|---|---|
| New sale agent prompt | answer-contains + format detection + cost |
| New tool added to hotel agent | tool-call F1 + tool-loop count |
| Reasoning gate config tweak | chain-of-thought-leak count |
| LLM provider switch (Azure→Kimi) | cost + latency + answer F1 |
| New router rule | correct-agent rate |
| Cohere rerank threshold | answer F1 (RAG quality changes ripple to agent) |

For RAG-side changes, prefer `rag-tune` skill first — agent-eval captures end-to-end, but rag-tune isolates retrieval signal.

---

## Operating rules

- **Refuse without golden.** Step 1 hard gate.
- **Sequential runs.** Step 2-3 enforce. Concurrent runs = noise.
- **Same env.** Same Azure deployment, same Kimi, same tenant. Step 2 captures env digest.
- **Failure-mode catalog mandatory.** Step 4 must include catalog count delta.
- **Regression test on ship.** Step 5 ties to tdd-stark handoff.
- **Cost is a metric.** No silent cost increase.

## Hand off

- SHIP → `tdd-stark` (write metric-floor regression test) → smoke-spidey → ship-rocket
- REWORK → `plan-cap` (revise approach) → re-enter agent-eval
- REJECT → `debug-sherlock` if root cause unclear, else discard

result: baseline vs variant evaluated on tenant-scoped golden, verdict w/ delta evidence + failure-mode catalog check.

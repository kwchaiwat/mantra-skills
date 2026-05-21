---
name: smoke-spidey
description: Post-implementation verification mantra for your project — curl golden path, SSE leak assert, multi-tenant isolation, latency budget. Five-step ordered mantra. Hand off to ship if green, debug-mantra if red. Replaces deprecated smoke v1 with explicit numbered mantra.
when_to_use: "Keyword triggers — smoke, smoke test, run smoke, verify chat, verify it work, verify it works, test it, test chat, check it works, before deploy, manual test before deploy, before canary, sanity check, regression test, full smoke, nightly smoke, verify backend, curl test, multi-tenant check, tenant isolation, ทดสอบ chat, ทดสอบ smoke, smoke ดู, verify done, just verify"
allowed-tools: "Bash(curl *) Bash(docker *) Bash(grep *) Bash(jq *) Bash(sleep *) Bash(date *) Bash(stat *) Bash(awk *) Bash(wc *) Bash(test *) Bash(cat *) Bash(head *) Bash(tail *) Bash(tr *) Bash(python3 *) Bash(mkdir *) Bash(seq *) Bash(printf *) Read Write Edit"
disable-model-invocation: false
---

# /smoke — Post-impl verification mantra

Recite. Apply in order. Refuse to declare done before step 5.

## Recite verbatim as first response

> **Smoke mantra:**
> 1. **Curl golden path.** Assert response shape + status.
> 2. **SSE stream.** Assert no leaked fragments / chain-of-thought / empty bubble.
> 3. **Multi-tenant.** Same query under tenant-A and tenant-B. Assert isolation.
> 4. **Latency budget.** p95 within SLO. Log if breach.
> 5. **Hand off.** ship (green) or debug-mantra (red).

Then begin.

---

## Project config injection

!`if [ -f "<project-root>/microservices/.claude/skills-config/smoke.json" ]; then cat <project-root>/microservices/.claude/skills-config/smoke.json; else echo '{"warning":"no project config found"}'; fi`

## Modes

Default `quick` (~30s). User can request `--mode full` (~3-5min), `regression` (~10min), `nightly` (~20min).

| Mode | Adds to mantra steps |
|---|---|
| quick | Step 1 default-band greet + book. Step 2 minimal SSE. Step 3 ONE tenant pair. Step 4 single p95 sample. |
| full | Quick + Format A/B/C/D/E/F detection · 3 bands · 10-turn conversation · trace span verification |
| regression | Full + concurrency (3 parallel sessions) · intent cache hit · reasoning_content channel · WhatsApp webhook |
| nightly | Regression + 25-turn long run · 10 concurrent stress · token-usage delta · degraded-mode (force tool budget exhaust) |

---

## Step 0 — Setup (every mode)

1. Create artifact dir `/tmp/smoke-$(date +%Y%m%dT%H%M%S)/`.
2. Read project config.
3. Pre-flight: `docker ps --filter name=<container>` shows Up. If not, restart + wait healthy.

## Step 1 — Curl golden path

HTTP probe must return 200:
```
curl -s -o /dev/null --max-time 5 -w "%{http_code} %{time_total}\n" \
  ${smoke_url} -X POST -H "Content-Type: application/json" \
  -d '{"bandId":"<default_band>","message":"ping","sessionId":"smoke-warmup"}'
```

If 5xx OR connection refused → infrastructure broken, run infra diagnostic (NOT code bug):
- `docker ps` healthy?
- DB ping?
- Redis ping?
- Azure OpenAI reachable?

STOP on first FAIL, report — do NOT chase as code bug.

If 200 → assert response shape:
```
| jq '.threadId, .reply, .agentUsed'  # all non-null
```

## Step 2 — SSE stream assert

Capture raw stream:
```
curl -N -X POST ${url} -H "Content-Type: application/json" \
  -d '<payload>' > /tmp/smoke-XXX/stream.txt 2>&1
```

Assert NONE of these patterns appear in stream:
- Raw `{` JSON-skeleton fragments before envelope closes
- Chain-of-thought prose: `/^The user (said|wants)|I need to/`
- Markdown ```json fence
- Static fallback: `"I'm having trouble pulling that information"`
- Empty assistant turn (zero-length content block)

Each pattern → which catalog mode (see debug-mantra step 2 routing).

If ANY pattern matches → STEP 5 hands off to debug-mantra with stream artifact.

## Step 3 — Multi-tenant isolation

Run SAME query under two different tenants:
```
# Tenant A
curl ... -d '{"bandId":"tenant-A","message":"<query>"}' > /tmp/smoke-XXX/tenant-a.json

# Tenant B
curl ... -d '{"bandId":"tenant-B","message":"<query>"}' > /tmp/smoke-XXX/tenant-b.json
```

Assert:
- Tenant-A response references ONLY tenant-A data (hotel names, reservations, etc.)
- Tenant-B response references ONLY tenant-B data
- Zero overlap between responses on tenant-scoped fields

If any leak → CRITICAL. STOP. Hand off debug-mantra IMMEDIATELY. Block ship.

This is project-critical (your project = multi-tenant multi-tenant SaaS).

## Step 4 — Latency budget

Measure p95 over N=10 sequential requests:
```
for i in $(seq 1 10); do
  curl -s -o /dev/null -w "%{time_total}\n" ${url} -X POST ...
done | sort -n | awk 'NR==10'
```

SLO defaults:
- Quick mode: p95 < 5s
- Full/regression: p95 < 3s

Breach → log in artifact, flag for next plan/refactor cycle but don't block ship unless > 2x SLO.

## Step 5 — Hand off

Output verdict:
```
SMOKE RESULT: <green | yellow | red>

Steps:
1. Curl golden path: <pass/fail>
2. SSE stream assert: <pass/fail>
3. Multi-tenant isolation: <pass/fail>
4. Latency p95: <Xms vs Yms SLO>

Artifacts: /tmp/smoke-XXX/

Hand off → <ship | debug-mantra>
```

---

## Operating rules

- **Infra failure NOT code bug.** Step 1 5xx → diagnose infra first.
- **Tenant leak = STOP.** Multi-tenant fail blocks ship absolutely.
- **Artifacts mandatory.** Every run produces `/tmp/smoke-<ts>/` with stream.txt + jq output.
- **No silent pass.** If any step skipped, mark verdict yellow + cite skipped step.
- **Pattern catalog tied to debug-mantra.** Step 2 fail mode names match debug-mantra step 2 routing table.

## Hand off

- All steps green → `ship-rocket`
- Step 2 SSE leak → `debug-sherlock` w/ stream artifact
- Step 3 tenant leak → `debug-sherlock` (CRITICAL)
- Step 4 budget breach → `ship-rocket` w/ note for next plan cycle

result: smoke green or referred to debug-mantra w/ artifact.

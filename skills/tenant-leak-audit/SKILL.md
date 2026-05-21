---
name: tenant-leak-audit
description: Service-wide multi-tenant leak audit mantra for your project — enumerate queries, classify with/without tenantId filter, test each suspect against tenant-A vs tenant-B, fix or justify, add lint rule. Five-step ordered mantra. Project-critical for multi-tenant SaaS. Refuses without target service path. Replaces ad-hoc grep w/ structured per-service audit.
when_to_use: "Keyword triggers — tenant audit, tenant leak audit, audit tenant, tenant isolation audit, multi-tenant audit, tenant scope audit, tenantId audit, missing tenantId, tenant bypass, cross-tenant leak, band leak, tenant security check, isolation check, scoping check, who can see, tenant scoping, ตรวจ tenant"
allowed-tools: "Bash(grep *) Bash(rg *) Bash(curl *) Bash(npm *) Bash(npx *) Bash(jq *) Bash(ls *) Bash(cat *) Bash(find *) Bash(wc *) Bash(awk *) Read Edit Write"
disable-model-invocation: false
---

# /tenant-leak-audit — Multi-tenant isolation audit mantra

Project-critical for your project (multi-tenant SaaS w/ tenant-per-band isolation). Tenant leak = customer-visible breach.

## Recite verbatim as first response

> **Tenant audit mantra:**
> 1. **Enumerate.** All queries / repository methods / raw SQL in target service.
> 2. **Classify.** Each query — has tenantId filter · justified bypass · MISSING.
> 3. **Test.** Each MISSING — run under tenant-A vs tenant-B, assert isolation.
> 4. **Fix or justify.** Add tenantId filter OR document why bypass is safe.
> 5. **Lint rule.** Add ESLint rule preventing recurrence.

Then begin.

---

## Required input

Target service path. Refuse without.

```
Usage examples:
  /tenant-leak-audit microservices/ai-agent-backend
  /tenant-leak-audit microservices/hotel-service/src/application/cqrs/queries
```

If absent → STOP. Output:
```
**Tenant audit needs target path.**
Provide: <relative-path-from-repo-root>
```

---

## Step 1 — Enumerate

Find all data-access call sites.

### TypeORM patterns (NestJS services)

```bash
TARGET="<path>"
rg -n --type ts \
  '(createQueryBuilder|find\(|findOne\(|findBy|findOneBy|update\(|delete\(|remove\(|save\()' \
  $TARGET \
  | grep -v '\.spec\.ts' \
  | grep -v 'node_modules' \
  > /tmp/tenant-audit-queries.txt

wc -l /tmp/tenant-audit-queries.txt
```

### Raw SQL

```bash
rg -n --type ts 'query\(`|query\(["\'']|EntityManager.*query' $TARGET \
  | grep -v '\.spec\.ts' \
  >> /tmp/tenant-audit-queries.txt
```

### Repository pattern callers

```bash
rg -n --type ts 'this\.<repo>\..*\(' $TARGET \
  | grep -v '\.spec\.ts' \
  >> /tmp/tenant-audit-queries.txt
```

### Aspire-app / alfred-app — Redux + RTK Query

```bash
# Redux Toolkit RTK Query endpoints
rg -n --type ts 'builder\.(query|mutation)\(' $TARGET \
  >> /tmp/tenant-audit-queries.txt

# Direct fetch / axios calls (check tenant header propagation)
rg -n --type ts '(axios|fetch)\s*\(' $TARGET \
  >> /tmp/tenant-audit-queries.txt
```

Output count + breakdown by file.

## Step 2 — Classify

For each enumerated query, classify into 3 buckets.

### Has tenantId filter ✓

Pattern matches:
- `.where('tenantId = :tenantId', { tenantId })` (TypeORM)
- `.find({ where: { tenantId } })` (TypeORM)
- `WHERE tenant_id = $1` (raw SQL)
- Query inherits tenant scope via base repository pattern w/ enforced filter

Verify by reading 3 lines above + below the match.

### Justified bypass ⚠️

Documented reason exists:
- `// TENANT-SAFE: aggregates across all tenants for admin dashboard`
- Comment cites Section A/B/C of project-patterns where bypass allowed
- Endpoint guarded by admin-only role check (verify guard upstream)

Each must have explicit comment. Implicit bypass = bug.

### MISSING ✗ (treat as leak)

Pattern matches a query/access but NO tenantId filter AND no comment justifying bypass.

Output classification table:
```
| File:Line | Code excerpt | Class | Reason |
|---|---|---|---|
| src/modules/X/X.service.ts:42 | createQueryBuilder('x').where(...) | ✓ has | tenantId in WHERE |
| src/modules/Y/Y.service.ts:88 | find({ id }) | ✗ MISSING | no tenantId, no comment |
| src/modules/Z/Z.service.ts:120 | repo.findAll() | ⚠️ bypass | comment cites admin-only |
```

## Step 3 — Test each MISSING

For each ✗ MISSING, prove isolation breach OR not.

### Test approach 1 — Runtime curl

Pick endpoint that invokes the suspect query. Run under two tenants:

```bash
# Tenant A
TENANT_A_TOKEN="<...>"
curl -s -H "Authorization: Bearer $TENANT_A_TOKEN" \
  ${endpoint} > /tmp/tenant-a.json

# Tenant B
TENANT_B_TOKEN="<...>"
curl -s -H "Authorization: Bearer $TENANT_B_TOKEN" \
  ${endpoint} > /tmp/tenant-b.json

# Assert no overlap on tenant-scoped fields
diff <(jq -r '.[].hotelId' /tmp/tenant-a.json | sort) \
     <(jq -r '.[].hotelId' /tmp/tenant-b.json | sort)
```

If overlap → LEAK confirmed.

### Test approach 2 — DB query inspection

Read what query SQL is generated:
```bash
# Enable TypeORM query logging
DEBUG=typeorm:* npm run start:dev
# Trigger endpoint, capture SQL
# Inspect WHERE clause for tenantId
```

### Test approach 3 — Unit test added

Write `.spec.ts` w/ two tenant fixtures, assert:
```typescript
const tenantA = await service.find({ tenantId: 'A' });
const tenantB = await service.find({ tenantId: 'B' });
expect(tenantA).not.toContainItemsFrom(tenantB);
```

For each MISSING, output:
- Confirmed leak: yes/no
- Test evidence: file path or curl artifact
- Severity: CRITICAL (cross-tenant data visible) / MAJOR (cross-tenant metadata) / MINOR (technically scoped but missing belt-and-braces)

## Step 4 — Fix or justify

### Fix path

Add tenantId filter at query site. Hand off to `tdd-stark`:
1. RED — write tenant-isolation test (approach 3 above)
2. GREEN — add `.where('tenantId = :tenantId', { tenantId })` or equivalent
3. REFACTOR — extract to base repository method if pattern repeats 3+ places
4. COVERAGE — assert ≥80% on changed lines
5. Hand off → smoke-spidey (run step 3 tenant isolation check)

### Justify path

If query MUST cross tenants (admin aggregate, monitoring, audit log):
1. Add comment above query: `// TENANT-SAFE: <reason>. Guarded by <RoleGuard>. See project-patterns Section A.`
2. Verify upstream guard enforces admin/internal-only access
3. Add unit test asserting non-admin caller is rejected before reaching this query

NEVER justify silently. Comment is mandatory for future-audit grep.

## Step 5 — Lint rule

Prevent recurrence. Add ESLint rule to fail CI on future MISSING.

Create `.eslintrc-tenant.json` rule (NestJS services):
```json
{
  "rules": {
    "no-tenant-leak/createqb-no-where-tenantid": "error",
    "no-tenant-leak/repo-find-no-tenantid": "error"
  }
}
```

If custom rule not yet built, use grep-based pre-commit hook:
```bash
# .husky/pre-commit
rg --type ts '(createQueryBuilder|find\(|findOne\(|repo\.\w+\()' \
   --files-with-matches \
   $(git diff --cached --name-only) \
  | xargs -I {} sh -c "rg -L 'tenantId|TENANT-SAFE' {} && echo 'TENANT LEAK SUSPECTED: {}' && exit 1"
```

For frontend (alfred-app / aspire-app): verify tenant header is set on every API call via interceptor.

---

## Output format

After step 5, output audit summary:

```
TENANT LEAK AUDIT — <service> — <date>

Enumerated:  <N> queries
Classified:  <X> ✓ scoped · <Y> ⚠️ justified · <Z> ✗ MISSING
Tested:      <Z> MISSING → <Q> confirmed leak · <R> safe-by-accident
Fixed:       <F> tenantId added · <J> justified w/ comment
Lint rule:   <added | already-present | manual-pre-commit>

CRITICAL leaks: <list w/ file:line>
MAJOR leaks:    <list>
MINOR leaks:    <list>
```

Hand off:
- 0 leaks → done
- CRITICAL leaks → hand off `tdd-stark` (priority fix) → smoke-spidey (tenant isolation step 3) → ship-rocket
- MAJOR leaks → schedule for next ship cycle, document in vault
- MINOR (safe-by-accident) → add belt-and-braces filter, not urgent

---

## Operating rules

- **No silent bypass.** Every cross-tenant query needs comment + role guard.
- **Belt-and-braces.** Even if upstream guard correct, query should filter — defense in depth.
- **No "tested once" assumption.** Step 3 tests every MISSING individually.
- **Refuse without service path.** Step 1 hard gate.
- **Lint rule mandatory.** Step 5 closes loop. Without lint, regression returns.

## When to run

| Trigger | Audit scope |
|---|---|
| New service introduced | Full service audit |
| New repository method added | New file audit |
| Customer-reported cross-tenant data sighting | Service containing that data audit |
| Quarterly hygiene | Rotating service audit |
| Pre-major-release | All tenant-touching services audit |
| After major refactor of repo/query layer | Service refactored audit |

---

## Hand off

- 0 leaks found → done, output audit summary to vault via note-kira
- CRITICAL leak → `tdd-stark` (fix) → `smoke-spidey` (verify) → `ship-rocket`
- Pattern repeats across files → propose base repository abstraction, hand off `plan-cap`
- Unclear if leak (test inconclusive) → hand off `debug-sherlock`

result: service-wide tenant isolation audit complete w/ classified queries, tested leaks, fixes or justifications, lint rule added.

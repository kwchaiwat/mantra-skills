---
name: tdd-stark
description: Test-first implementation mantra for your project — RED (failing test) → GREEN (minimal pass) → REFACTOR → COVERAGE check → lint + tsc + jest green. Refuse to write impl before failing test exists. Replaces implementation halves of deprecated build-feature and refactor skills.
when_to_use: "Keyword triggers — implement, write code, code this, start coding, build it, do it, จัดการเลย, ลุย, write feature, code feature, write test, add test, TDD, write impl, refactor code, simplify code, extract function, rename, ปรับ code, เขียน code"
allowed-tools: "Bash(npm *) Bash(npx *) Bash(jest *) Bash(node *) Bash(ls *) Bash(cat *) Read Edit Write"
disable-model-invocation: false
---

# /tdd-implement — Test-first implementation mantra

Recite. Refuse to write production code before failing test exists.

## Recite verbatim as first response

> **TDD mantra:**
> 1. **RED.** Write failing test first. Run. Confirm fail reason matches expectation.
> 2. **GREEN.** Minimal impl. Run. Confirm pass.
> 3. **REFACTOR.** Clean up impl + test. Re-run all tests.
> 4. **COVERAGE.** ≥80% on changed lines. lint + tsc + jest green.
> 5. **HAND OFF.** smoke (user-facing) or ship (internal).

Then begin.

---

## Step 1 — RED

Write the failing test BEFORE any production code.

For radiant1 services, test file location per `project-patterns`:
- NestJS backend (ai-agent-backend, hotel, reservation): adjacent `.spec.ts`
- alfred-app: no automated test framework (known gap) — write smoke check instead
- aspire-app: Cypress `cypress/e2e/<feature>.cy.ts`

Test structure (AAA):
```typescript
test('describes behavior under test', () => {
  // Arrange
  const input = ...;

  // Act
  const result = subjectUnderTest(input);

  // Assert
  expect(result).toBe(expected);
});
```

Run: `npm test -- <test-file>` (or `npx jest <path>`).

CONFIRM the test fails AND the failure reason matches expectation. A test that fails for the wrong reason is a false RED.

If test passes immediately → impl already exists OR test is wrong. STOP.

## Step 2 — GREEN

Minimal code to make test pass. Smallest possible diff.

Rules:
- NO refactoring during GREEN.
- NO adding features beyond what test asserts.
- NO speculative generality.

Run test. Confirm PASS. If other tests now broken → fix before moving on.

## Step 3 — REFACTOR

Clean up impl AND test. Both.

Apply your project coding rules (CODING_CONVENTIONS.md):
- no `any` types
- no nested ifs / nested loops
- blank line before `return`
- no `console.log` (use `this.logger.log`)
- ~20-line methods
- no magic numbers (extract constants)
- no DB in loops (O(n) violations)
- repository pattern
- no JSDoc / comments
- SOLID

Re-run ALL tests. ALL must pass. If any regression → revert refactor, try smaller.

## Step 4 — COVERAGE check

Run with coverage:
```
npm run test:cov -- <test-file>
```

Target: ≥80% on changed lines.

If below 80%:
- Add edge case tests (null/empty/large input)
- Add error path tests
- Add multi-tenant boundary tests (if relevant)

Then run final checks:
```
npm run lint
npx tsc --noEmit
npm test
```

All three MUST be green. If lint complains → fix code, NOT lint config.

## Step 5 — Hand off

Pre-handoff: write 1-line ledger entry for take-note-ob:
```
<file>:<line> — <what changed> — <why> — <test that proves it>
```

Then:
- User-facing change (chat/UI/API response) → hand off `smoke-spidey`
- Internal/refactor (no user-visible delta) → hand off `ship-rocket`

---

## Operating rules

- **Refuse without failing test.** If user says "just write it" → answer "TDD requires test first. Want me to draft test now?"
- **Refuse without passing baseline.** If existing test suite is red BEFORE your changes → fix baseline first, separately.
- **Refactor IS optional GREEN guard.** If GREEN code already clean enough, skip step 3. Don't refactor for ceremony.
- **Coverage is a floor, not goal.** 100% coverage of trivial code < 60% of critical paths. Use judgment.

## Mode flags

- `--refactor` — preserve behavior. Test count must NOT increase. Pre/post test counts must match. Use baseline-test-suite parity check.
- `--bugfix` — RED test must reproduce the bug. Then GREEN.

## Hand off

- User-facing → `smoke-spidey`
- Internal → `ship-rocket`
- Test parity violated on `--refactor` → revert, restart

result: implementation green, coverage ≥80%, lint/tsc/jest clean.

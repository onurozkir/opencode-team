---
name: team-qa
description: Test strategy and quality assurance methodology. Acceptance criterion -> test conversion, edge cases, e2e coverage. The playbook of the qa role.
---

# Quality Assurance

## What a test is

A test is **the verification of an acceptance criterion**. Not a test that shows the code runs, but a test that shows the behavior is correct.

The purpose of writing tests is code reliability, not coverage percentage.

## 1. Acceptance criterion -> test

Every acceptance criterion in the backlog must have a test. Build a mapping table:

| Criterion | Test | Type | Status |
|---|---|---|---|
| "No invoices" is shown on an empty list | `renders_empty_state_when_no_invoices` | unit | ✅ |

If there is no test that meets a criterion, that criterion is **untested**. Not "passed".

## 2. The test pyramid

| Type | What it tests | Ratio | Speed |
|---|---|---|---|
| Unit | Business logic, edge cases | 70% | ms |
| Integration | DB, API, file system | 20% | s |
| e2e | The user journey | 10% | s-min |

Do not write e2e for everything. e2e is fragile and slow; use it only for the main journeys that bring in money.

## 3. Mandatory edge cases

For every endpoint and every form:

| Case | Expected |
|---|---|
| Empty input | 400 + a meaningful error, not 500 |
| Oversized input | 413 or a controlled error |
| Corrupt data (invalid JSON, broken encoding) | 400, no crash |
| Unauthorized access | 403 + no resource leakage |
| Resource not found | 404, without leaking another user's data |
| The same request twice | No effect (idempotent, or no duplicate record) |
| Two concurrent requests | No data corruption (race condition) |
| Database connection dropped | Not 5xx, a proper error body |

## 4. Test quality

**Bad test** — does not verify the application:
```python
def test_invoice():
    assert invoice is not None
```

**Good test** — verifies the behavior:
```python
def test_parse_invoice_returns_none_for_missing_total():
    result = parse_invoice("Invoice No: 123\nDate: 2026-01-01")
    assert result["total"] is None
```

Rules:
- The test name should tell the behavior
- Each test verifies **one** behavior
- Do not write tests that look inside the application and verify it (if what is mocked does not represent reality, the test is useless)
- Tests should be independent of each other, order should not matter
- If it depends on time or randomness, fix it

## 5. Finding-fix loop

```
Finding -> Write the test (FIRST) -> See the test fail -> Fix the code -> Test passes -> Refactor
```

Do not fix a finding before writing the test. If the test is written afterwards, it does not verify the fix.

## 6. The trap of weakening a test

Deleting or loosening an assertion to make a test pass is **not a fix**. Do not do any of these:
- `assert x == 5` → `assert x is not None`
- Adding `pytest.skip` / `test.skip`
- Commenting out a line
- Increasing the timeout (it hides race conditions)

## 7. Reporting

| Criterion | Test | Result |
|---|---|---|

Do not show a criterion you could not test as "passed". Write "not testable + rationale".

## Output format

Markdown: Scope mapping table / Actual test run output / Bugs found and fixed / Not tested + rationale / Remaining risks.

In the user's language (Turkish or English).

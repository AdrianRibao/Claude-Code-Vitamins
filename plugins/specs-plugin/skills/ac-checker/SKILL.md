---
name: ac-checker
description: Verify acceptance criteria from TDDs are actually implemented in code. Check that tests exist, features are coded, coverage targets are met, and checkboxes are marked complete. Use when validating implementation completeness, reviewing branch readiness, or auditing TDD status.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - TodoWrite
  - Bash
  - Task
---

# Acceptance Criteria Checker Skill

> **Philosophy**: Every acceptance criterion must be verifiable through tests and implemented code before marking complete.

## When to Use

- Verifying acceptance criteria are actually implemented in code
- Checking if branch/PR is ready for review or merge
- Validating implementation completeness before deployment
- Auditing which criteria are done vs. pending
- Generating implementation status reports for stakeholders
- Ensuring test coverage targets are met

## How to Invoke

```
/ac-checker [tdd-path] [--update] [--coverage] [--branch main]
```

**Arguments**:

- `[tdd-path]` - Path to specific TDD file to check (required)
- `--update` - Update TDD by marking implemented criteria as complete
- `--coverage` - Run coverage analysis to verify targets
- `--branch` - Compare against specific branch (default: main)

**Examples**:

- `/ac-checker specs/tdds/incidents/01-incidents-backend.md`
- `/ac-checker specs/tdds/user-auth/01-user-auth-backend.md --coverage`
- `/ac-checker specs/tdds/dashboard/02-dashboard-ui.md --update`

## Resources

- [implementation-checks.md](implementation-checks.md) - Verification patterns and strategies
- [test-patterns.md](test-patterns.md) - How to find tests for criteria

## Behavioral Mindset

Implementation-first approach: verify code and tests exist before marking criteria complete. Search test files for matching test cases. Analyze code for feature implementation. Run coverage tools to validate targets. Generate actionable reports showing what's done vs. pending.

## Key Actions

### Phase 1: Parse TDD

1. **Read TDD**: Load TDD file and extract acceptance criteria section

    - Open the TDD file at specified path
    - Locate the "## Acceptance Criteria" section
    - Extract all criteria with their subsections

2. **Parse Criteria**: Extract each criterion with its checkbox state

    - Identify checkbox format: `- [ ]` (incomplete) or `- [x]` (complete)
    - Extract criterion text (everything after the checkbox)
    - Group by subsection (Core Functionality, Validation, Authorization, Testing, Code Quality)

3. **Identify Type**: Determine TDD type from path or content

    - Check file path for keywords: "backend", "ui", "api", "integration"
    - Look for type indicators in TDD metadata section
    - Default to "backend" if unclear

4. **Extract Coverage Targets**: Parse required coverage percentages

    - Find lines with format: "Test coverage ≥ XX% for..."
    - Extract three targets: domain logic (≥80%), business rules (≥90%), auth (≥95%)
    - Store for later validation if `--coverage` flag set

5. **Extract Permission Matrix Test Plan**: Parse the `## Permission Matrix Test Plan` section if present

    - Locate the `## Permission Matrix Test Plan` heading
    - Read the formula line: `N = [A] + [S] + [F] + [H] + [R] + 2·[P] + [E] = [TOTAL]` (accept the legacy `N = [A] + [S] + [H] + [R] + 2·[P] + [E]` form for TDDs written before the `F` term existed — flag it Medium: "formula predates referent coverage")
    - Parse every `PT-xx` row from the table: ID, Layer, Cell/Rule, Actor, Action/Field, Type, Expected signal, State assertion
    - Re-derive the expected `N` from the Authorization tables (Layer 1 cell count, scope-qualified cells, Hidden / R field-role pairs, PC-xx count, protected entry points) and from the Interface Contract / Data Model (referent pairs: each action's accepted FK inputs that reference scope-qualified resources), then cross-check against the declared `N`
    - If the Authorization section is non-empty AND the Permission Matrix Test Plan is missing, record a **Critical** finding and block the entire TDD acceptance — no further checks can compensate for an absent plan

### Phase 2: Discover Test Files

1. **Locate Test Directory**: Find test files relevant to TDD

    - Backend TDD: Look in `test/` or `__tests__/` directory
    - UI TDD: Look in `test/components/` or `test/ui/` directory
    - API TDD: Look in `test/api/` or `test/integration/` directory
    - Use Glob tool to find all test files matching pattern

2. **Map Feature to Tests**: Identify test files that should contain tests for this feature

    - Extract feature name from TDD path (e.g., "incidents" from `01-incidents-backend.md`)
    - Search for test files with matching names (e.g., `incidents_test.exs`, `incidents.test.js`)
    - Use Grep to search for module/class names mentioned in TDD
    - Check test describe blocks for feature references

### Phase 3: Verify Implementation

1. **Check Each Criterion**: For each acceptance criterion, verify three aspects:

    - **Tests Exist**: Search test files for matching test cases
        - Extract key terms from criterion (e.g., "create", "incident", "required fields")
        - Use Grep to search test files for these terms
        - Look for test descriptions like `test "user can create incident"`
        - Verify test assertions match expected behavior
    - **Code Exists**: Search source files for feature implementation
        - Identify functions/methods mentioned in criterion
        - Use Grep to find function definitions in source code
        - Check that business logic is present, not just stubs
        - Verify related modules/classes exist
    - **Checkbox Status**: Verify marked status matches actual implementation
        - Record current checkbox state from TDD (incomplete `- [ ]` or complete `- [x]`)
        - Compare against tests/code verification results
        - Flag mismatch if marked complete but tests/code missing
        - Flag if implemented but not marked complete

2. **Verify Permission Matrix Test Plan (always, when present)**: This check runs unconditionally — it does NOT require `--coverage`. The permission matrix is the highest-risk surface and is checked on every invocation.

    - **Plan completeness**:
        - Declared `N` is at minimum the derived `N` from the Authorization tables plus the referent pairs derived from the Interface Contract / Data Model. If declared < derived, mark Critical: "test count under-counted".
        - Every `❌` cell in the Layer 1 matrix has a matching `Deny` row in the plan. Missing rows are listed by `(actor, action)`.
        - Every scope-qualified `✅ Own / Team / Dept` cell has a paired `Cross-scope deny` (`1-S`) row.
        - Every `PC-xx` rule appears exactly twice (`Condition met` + `Condition not met`). Single-row PC-xx coverage is Critical.
        - Every `Hidden` (`2-H`) and `R` read-only (`2-R`) field-role pair has a row.
        - Every protected entry point has an `E` row asserting `401`.
        - Every (action, foreign-key input) pair whose referenced record is scope-qualified has a `Referent deny` (`1-F`) row. Derive the pairs from the Interface Contract / Data Model (accepted FK attributes and arguments, including nested/relationship writes and bulk id lists). Missing pairs are Critical — this is the IDOR class a green matrix cannot otherwise catch.
    - **Row quality (per PT-xx)**:
        - `Expected signal` cell is non-empty AND concrete (a status code, a tagged error, a framework exception). Reject "fails", "error", or empty.
        - `Type = Deny / Cross-scope deny / Referent deny / Read-only deny / Condition not met` rows have a non-empty `State assertion`. A deny test without a side-effect assertion is flagged High — it can pass while the action silently mutated state.
        - `Referent deny` rows assert nothing was written in **either** scope — a rejection that still persisted a partial write is the production bug pattern this row exists to catch.
        - `404` in `Expected signal` is only acceptable when `Type = Anti-discovery deny` AND the row contains a justification. Otherwise Critical: "generic 404 used as deny — hides authorization decision from logs".
        - `Type = Auth boundary` rows use `401` and not `403`. Mismatch is High.
    - **Test discovery**:
        - For each `PT-xx`, grep test files for the literal `PT-xx` token (in test names, `describe` blocks, tags, or comments). Missing tokens listed.
        - For each found test, confirm the test body asserts the declared expected signal (e.g. `403`, `:forbidden`, `NotAuthorizedError`) — not merely a generic `refute` or `assert_raise` of any error.
        - For deny tests, confirm an assertion exists on the state (e.g. `refute Repo.get(Record, id) |> changed?`, `assert_no_changes`, `expect(record.reload).to eq(...)`). A deny test that never reloads the target is flagged High.
        - When the plan contains `Referent deny` rows, confirm a **referent sweep test** exists: a test that introspects schema/resource definitions and enumerates (action, FK-input) pairs (e.g. iterates `Ash.Resource.Info.actions/1`, `reflect_on_all_associations`, `_meta.get_fields()`). Absence is High: "F rows present but no code-derived sweep — a newly added action can silently skip the ownership check".
    - Findings go into a dedicated "Permission Matrix" section of the report, ranked Critical > High > Medium, and any Critical permission-matrix finding marks the overall TDD as **NOT acceptable for merge** regardless of other passing checks.

3. **Verify Coverage Targets**: If `--coverage` flag set, validate test coverage

    - Run appropriate coverage command for project type:
        - Elixir: `mix test --cover`
        - Node.js: `npm test -- --coverage`
        - Python: `pytest --cov`
        - Ruby: `bundle exec rspec` (with SimpleCov)
    - Parse coverage output to extract percentage values
    - Compare against TDD targets (≥80% domain, ≥90% business rules, ≥95% auth)
    - Record which targets pass and which fail

### Phase 4: Generate Report

1. **Calculate Statistics**: Compute totals for all criteria status metrics

    - Total criteria count (all checkboxes in acceptance criteria section)
    - Implemented count (criteria with both tests and code present)
    - Marked complete count (criteria with `[x]` checkbox)
    - Pending count (criteria with neither tests nor code)
    - Mismatched count (marked complete but missing implementation, or vice versa)

2. **Create Implementation Report**: Generate detailed report with multiple sections

    - **Summary Section**: Overall statistics and status indicator
    - **Coverage Validation**: Table showing required vs actual percentages
    - **Criteria Status**: Table for each subsection with per-criterion status
    - **Missing Tests**: List criteria with code but no tests, include file locations
    - **Missing Code**: List criteria with tests but no implementation
    - **Not Implemented**: List criteria with neither tests nor code
    - **Checkbox Mismatches**: Separate lists for false positives and false negatives
    - **Recommendations**: Prioritized action items (Critical, High, Medium, Low)
    - **Next Steps**: Actionable checklist

3. **Save Report**: Write to `specs/tdds/reports/ac-implementation-[tdd-name]-[timestamp].md`

    - Create reports directory if it doesn't exist
    - Generate timestamp in format YYYY-MM-DD-HHMM
    - Extract TDD name from file path for report filename
    - Use Write tool to save complete report

### Phase 5: Update TDD (Optional with --update)

1. **Mark Complete**: Auto-update TDD to mark criteria as complete when verified

    - Read the original TDD file
    - For each criterion where tests AND code both exist:
        - Change `- [ ]` to `- [x]`
        - Add comment with verification timestamp if desired
    - Use Edit tool to update the TDD file in place

2. **Preserve Manual Marks**: Don't unmark criteria that were manually marked

    - If criterion is already marked `[x]`, leave it as-is even if verification incomplete
    - Only mark new completions, never unmark
    - This respects manual overrides for special cases
    - Report preserved marks separately in the report

## Verification Strategies

### Finding Tests for Criteria

Use multiple strategies to locate relevant tests:

**1. Keyword Search**: Extract key terms from criterion and search test files

Example:

- Criterion: "User can create incident with required fields"
- Search: "create incident", "required fields", "incident creation"

**2. Test Description Matching**: Look for test descriptions matching criterion

Example test descriptions:

- `it("allows user to create incident with required fields")`
- `test("incident creation requires title and description")`
- `describe("creating incidents")`

**3. Function Name Search**: Find tests for specific functions mentioned

Example:

- Criterion: "API returns 401 when authentication token is missing"
- Search: test files for "401", "authentication", "unauthorized"

**4. Module-Based Search**: Map TDD to implementation files, then find corresponding tests

Example:

- TDD: `specs/tdds/incidents/01-incidents-backend.md`
- Code: `lib/app/incidents.ex`
- Tests: `test/app/incidents_test.exs`

### Determining Implementation Status

For each criterion, use this decision tree:

```
Has tests?
  ├─ Yes -> Has implementation code?
  │   ├─ Yes -> ✅ IMPLEMENTED
  │   └─ No  -> ⚠️ TESTS ONLY (likely WIP)
  └─ No  -> Has implementation code?
      ├─ Yes -> ⚠️ CODE ONLY (needs tests)
      └─ No  -> ❌ NOT IMPLEMENTED
```

### Coverage Verification

When `--coverage` flag is used:

1. Run appropriate coverage command for project (Elixir: `mix test --cover`, Node.js: `npm test -- --coverage`, Python: `pytest --cov`, Ruby: `bundle exec rspec` with SimpleCov)

2. Parse coverage output to extract percentages

3. Compare against TDD targets (Domain logic: ≥80%, Critical business rules: ≥90%, Auth/authorization: ≥95%)

4. Report discrepancies in implementation report

## Report Format

The generated report includes:

### Summary Section

- Total Criteria count
- Implemented count (tests + code exist)
- Marked Complete count
- Pending count (no tests or code)
- Mismatched count (marked complete but missing implementation)
- Overall Status indicator

### Coverage Validation Section

Table showing target area, required percentage, actual percentage, and pass/fail status for Domain logic, Critical business rules, and Auth/authorization.

### Permission Matrix Section

A dedicated section that always runs when the TDD has an Authorization section. Contents:

- **Formula check**: derived `N` vs declared `N`, with the arithmetic spelled out.
- **Coverage table**: one row per `PT-xx`, with columns: `PT-ID`, `Type` (Allow / Deny / Cross-scope / Referent / Hidden / Read-only / PC met/unmet / Auth boundary / Anti-discovery), `Test found?`, `Signal asserted?`, `State asserted?`, `Status` (✅ / ⚠️ / ❌).
- **Missing-row table**: derived rows that should exist but don't (e.g. "Layer 1: Guest × Read is ❌ in the matrix but has no Deny row in the plan", "`annotate.report_id` references a scope-qualified resource but has no Referent deny row").
- **Quality findings**: list of plan rows with vague expected signals, missing state assertions, generic `404` denies, `Auth boundary` rows using the wrong status code, or `Referent deny` rows without a companion referent sweep test.
- **Acceptance verdict**: `ACCEPTABLE` or `BLOCKED — Critical permission matrix findings present`. Any Critical finding here blocks the overall acceptance regardless of other sections.

### Criteria Status Section

Organized by subsection (Core Functionality, Validation, Authorization, Testing, Code Quality) with table columns: Number, Criterion text, Tests (present/absent), Code (present/absent), Checkbox state, Overall status.

### Missing Tests Section

Lists criteria with implementation but no tests, showing criterion text, implementation location (file and function), current test status, and recommendation.

### Missing Code Section

Lists criteria with tests but no implementation, showing criterion text, test location, implementation status, and recommendation.

### Not Implemented Section

Lists criteria with neither tests nor code.

### Checkbox Mismatches Section

Two subsections: Marked Complete but Missing Implementation, and Implemented but Not Marked Complete.

### Recommendations Section

Prioritized list with Critical issues, High Priority items, Medium Priority items, and Low Priority items.

### Next Steps Section

Actionable checklist of tasks to complete implementation.

## Include in Verification

| Check Type        | Purpose                       | Example                                    |
| ----------------- | ----------------------------- | ------------------------------------------ |
| Test Search       | Find tests matching criterion | Search for "create incident" in test files |
| Code Search       | Verify implementation exists  | Find function `create_incident/2` in code  |
| Coverage Analysis | Validate percentage targets   | Run `mix test --cover` and parse output    |
| Checkbox Parsing  | Check completion status       | Extract checkbox states from TDD           |
| Status Comparison | Find mismatches               | Flag if marked complete but no tests       |

## Exclude from Verification

| Element             | Why Exclude             | Rationale                             |
| ------------------- | ----------------------- | ------------------------------------- |
| Open Questions      | Not criteria            | Separate section for unresolved items |
| Code comments       | Not verification target | Comments don't prove implementation   |
| Manual test steps   | Not automated           | Focus on automated test verification  |
| Future requirements | Not current scope       | Only check current iteration criteria |

## Outputs

- **Implementation Report**: Detailed status of each criterion with evidence
- **Updated TDD** (with --update): TDD with checkboxes auto-marked based on implementation
- **Coverage Report**: Test coverage analysis vs. targets

## Examples

### Check Backend TDD

```
/ac-checker specs/tdds/incidents/01-incidents-backend.md
```

Generates implementation report showing:

- 35/42 criteria have tests and code
- 5 criteria marked complete but missing tests
- Coverage: 85% domain, 88% business rules, 97% auth

### Check with Coverage Analysis

```
/ac-checker specs/tdds/user-auth/01-user-auth-backend.md --coverage
```

Runs test coverage and validates against targets:

- Runs: `mix test --cover`
- Parses coverage output
- Reports: All coverage targets met or shows gaps

### Check and Auto-Update TDD

```
/ac-checker specs/tdds/dashboard/02-dashboard-ui.md --update
```

Verifies implementation and updates TDD:

- Finds 8 criteria with tests and code
- Marks those 8 as complete in TDD
- Preserves manual marks
- Generates report with changes

## Quality Checklist

Before completing acceptance criteria check, verify:

- [ ] All acceptance criteria parsed from TDD
- [ ] Test files identified and searched
- [ ] Implementation code searched for each criterion
- [ ] Checkbox states compared with actual implementation
- [ ] Coverage targets validated (if --coverage flag)
- [ ] **Permission Matrix Test Plan parsed and validated (formula, row completeness, signal quality, state assertions, test discovery by `PT-xx` token)** — this check runs on every invocation, no flag required
- [ ] **Any Critical permission-matrix finding flips the overall verdict to BLOCKED**
- [ ] Report generated with detailed status
- [ ] Recommendations prioritized (critical to low)
- [ ] Next steps actionable

## Boundaries

**Will:**

- Search test files for matching test cases
- Search code for implementation of features
- Parse TDD acceptance criteria and checkbox states
- Run coverage tools and validate targets
- Generate detailed implementation status reports
- Auto-update TDD checkboxes with --update flag
- Flag mismatches between marked status and actual implementation
- **Parse the Permission Matrix Test Plan, re-derive the required test count from the Authorization tables, and verify each `PT-xx` row has a matching test that asserts the declared signal and the prevented state change**
- **Block TDD acceptance when permission-matrix Critical findings exist** (missing plan, missing deny rows, vague signals, generic 404 denies, missing state assertions)

**Will Not:**

- Judge quality or correctness of tests
- Evaluate code implementation quality
- Determine if business logic is correct
- Run tests to verify they pass
- Modify implementation code
- Change requirement definitions
- Make architectural decisions

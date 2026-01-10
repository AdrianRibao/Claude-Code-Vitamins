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

2. **Verify Coverage Targets**: If `--coverage` flag set, validate test coverage

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

**Will Not:**

- Judge quality or correctness of tests
- Evaluate code implementation quality
- Determine if business logic is correct
- Run tests to verify they pass
- Modify implementation code
- Change requirement definitions
- Make architectural decisions

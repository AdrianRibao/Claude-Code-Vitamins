# Acceptance Criteria Checker

Verify that acceptance criteria from TDDs are actually implemented in code with tests and proper coverage.

## Overview

The Acceptance Criteria Checker skill validates implementation completeness by checking:

- **Tests Exist**: Search test files for matching test cases for each criterion
- **Code Exists**: Verify implementation code is present, not just stubs
- **Coverage Targets**: Validate test coverage meets TDD requirements (≥80%, ≥90%, ≥95%)
- **Checkbox Status**: Compare marked completion status with actual implementation
- **Implementation Reports**: Generate detailed status showing what's done vs. pending

## Quick Start

### Check a Backend TDD

```bash
/ac-checker specs/tdds/incidents/01-incidents-backend.md
```

This will:

1. Parse acceptance criteria from the TDD
2. Search test files for matching tests
3. Search source files for implementation code
4. Compare checkbox states with actual implementation
5. Generate detailed implementation report
6. Save report to `specs/tdds/reports/ac-implementation-[name]-[timestamp].md`

### Check with Coverage Analysis

```bash
/ac-checker specs/tdds/user-auth/01-user-auth-backend.md --coverage
```

Runs test coverage command for the project:

- Elixir: `mix test --cover`
- Node.js: `npm test -- --coverage`
- Python: `pytest --cov`
- Ruby: `bundle exec rspec`

Validates actual coverage against TDD targets (≥80% domain, ≥90% business rules, ≥95% auth).

### Check and Auto-Update TDD

```bash
/ac-checker specs/tdds/dashboard/02-dashboard-ui.md --update
```

Verifies implementation and automatically updates TDD:

- Marks criteria as complete `[x]` when both tests AND code exist
- Preserves manual marks (never unmarks)
- Generates report showing what was updated

## What Gets Verified

### 1. Tests Exist

For each criterion, the skill searches test files using multiple strategies:

**Keyword Search**: Extracts key terms from criterion and searches test files

- Criterion: "User can create incident with required fields"
- Search: "create incident", "required fields", "incident creation"

**Test Description Matching**: Looks for test descriptions matching criterion

- `it("allows user to create incident with required fields")`
- `test("incident creation requires title and description")`
- `describe("creating incidents")`

**Function Name Search**: Finds tests for specific functions mentioned

- Criterion: "API returns 401 when authentication token is missing"
- Search: "401", "authentication", "unauthorized"

### 2. Code Exists

Verifies implementation in source files:

- Identifies functions/methods mentioned in criterion
- Uses Grep to find function definitions in source code
- Checks that business logic is present, not just stubs
- Verifies related modules/classes exist

### 3. Coverage Targets

When `--coverage` flag is used:

- Runs appropriate coverage command for project type
- Parses coverage output to extract percentages
- Compares against TDD targets:
    - Domain logic: ≥80%
    - Critical business rules: ≥90%
    - Auth/authorization: ≥95%
- Reports which targets pass/fail

### 4. Checkbox Status

Validates marked status matches implementation:

- Extracts checkbox states: `- [ ]` (incomplete) or `- [x]` (complete)
- Flags if marked complete but tests/code missing
- Flags if implemented but not marked complete

## Implementation Status Logic

For each criterion:

```
Has tests?
  ├─ Yes -> Has implementation code?
  │   ├─ Yes -> ✅ IMPLEMENTED
  │   └─ No  -> ⚠️ TESTS ONLY (likely WIP)
  └─ No  -> Has implementation code?
      ├─ Yes -> ⚠️ CODE ONLY (needs tests)
      └─ No  -> ❌ NOT IMPLEMENTED
```

## Report Format

The generated report includes:

### Summary Section

- Total Criteria: 42
- Implemented: 35 (tests + code exist)
- Marked Complete: 38
- Pending: 5 (no tests or code)
- Mismatched: 2 (marked complete but missing implementation)
- Overall Status: ⚠️ NEEDS ATTENTION

### Coverage Validation Section

| Target Area             | Required | Actual | Status  |
| ----------------------- | -------- | ------ | ------- |
| Domain logic            | ≥80%     | 85%    | ✅ PASS |
| Critical business rules | ≥90%     | 88%    | ❌ FAIL |
| Auth/authorization      | ≥95%     | 97%    | ✅ PASS |

### Criteria Status Section

Organized by subsection (Core Functionality, Validation, Authorization, Testing, Code Quality):

| #   | Criterion                         | Tests | Code | Checkbox | Status             |
| --- | --------------------------------- | ----- | ---- | -------- | ------------------ |
| 1   | User can create incident          | ✅    | ✅   | [x]      | ✅ IMPLEMENTED     |
| 2   | Required fields validated         | ✅    | ❌   | [ ]      | ⚠️ TESTS ONLY      |
| 3   | API returns 401 when unauthorized | ❌    | ✅   | [ ]      | ⚠️ CODE ONLY       |
| 4   | Incident title max 200 chars      | ❌    | ❌   | [ ]      | ❌ NOT IMPLEMENTED |

### Missing Tests Section

Lists criteria with implementation but no tests:

- **Criterion**: Required fields validated
- **Implementation**: `lib/app/incidents.ex:validate_required_fields/1`
- **Test Status**: No matching tests found
- **Recommendation**: Add tests to `test/app/incidents_test.exs`

### Missing Code Section

Lists criteria with tests but no implementation.

### Not Implemented Section

Lists criteria with neither tests nor code.

### Checkbox Mismatches Section

Two subsections:

- **Marked Complete but Missing Implementation**: Criteria marked `[x]` but tests/code not found
- **Implemented but Not Marked Complete**: Criteria with tests and code but still marked `[ ]`

### Recommendations Section

Prioritized action items:

- **Critical**: Missing tests for implemented features
- **High Priority**: Missing implementation for existing tests
- **Medium Priority**: Checkbox mismatches to correct
- **Low Priority**: Documentation updates

### Next Steps Section

Actionable checklist of tasks to complete implementation.

## Usage Patterns

### Pre-Merge Validation

Check if branch is ready for review:

```bash
/ac-checker specs/tdds/incidents/01-incidents-backend.md
```

Review report to see:

- Which criteria are fully implemented
- Which need more tests
- Which need more code
- Overall completion percentage

### Coverage Validation

Verify test coverage targets are met:

```bash
/ac-checker specs/tdds/user-auth/01-user-auth-backend.md --coverage
```

Runs coverage analysis and validates against TDD targets. Fails if coverage below required thresholds.

### TDD Checkpoint Updates

Mark completed criteria in TDD:

```bash
/ac-checker specs/tdds/dashboard/02-dashboard-ui.md --update
```

Automatically updates TDD checkboxes based on verification. Useful for keeping TDD in sync with implementation progress.

### Branch Comparison

Compare against specific branch:

```bash
/ac-checker specs/tdds/incidents/01-incidents-backend.md --branch develop
```

Checks implementation status relative to another branch (default: main).

## Examples

### Backend TDD Check

```bash
/ac-checker specs/tdds/incidents/01-incidents-backend.md
```

**Report shows**:

- 35/42 criteria have tests and code
- 5 criteria marked complete but missing tests
- 2 criteria implemented but not marked complete
- Coverage: 85% domain, 88% business rules, 97% auth

### UI TDD with Coverage

```bash
/ac-checker specs/tdds/dashboard/02-dashboard-ui.md --coverage
```

**Report shows**:

- 28/30 criteria implemented
- Coverage: 82% domain, 91% business rules, 96% auth
- All coverage targets met ✅
- 2 criteria need implementation

### API TDD with Auto-Update

```bash
/ac-checker specs/tdds/user-auth/03-user-auth-api.md --update
```

**Actions taken**:

- Found 8 criteria with both tests and code
- Marked those 8 as complete in TDD (changed `[ ]` to `[x]`)
- Preserved 3 manually marked criteria
- Generated report with changes

## Integration with Workflow

### With TDD Writer

1. Generate TDD: `/tdd incidents --type backend`
2. Implement features iteratively
3. Check implementation status: `/ac-checker specs/tdds/incidents/01-incidents-backend.md`
4. Address gaps shown in report
5. Re-check until complete
6. Update TDD: `/ac-checker [path] --update`

### Pre-PR Checklist

Before creating pull request:

```bash
# Check implementation completeness
/ac-checker [tdd-path]

# Verify coverage targets
/ac-checker [tdd-path] --coverage

# Update TDD checkboxes
/ac-checker [tdd-path] --update

# Commit updated TDD
git add specs/tdds/[feature]/
git commit -m "Update TDD with implementation status"
```

## Resources

- [implementation-checks.md](implementation-checks.md) - Verification patterns and strategies
- [test-patterns.md](test-patterns.md) - How to find tests for criteria

## Limitations

The checker **will not**:

- Judge quality or correctness of tests
- Evaluate code implementation quality
- Determine if business logic is correct
- Run tests to verify they pass
- Modify implementation code
- Change requirement definitions
- Make architectural decisions

It focuses solely on **verifying implementation exists** (tests + code) for each acceptance criterion.

## Boundaries

**Will**:

- Search test files for matching test cases
- Search code for implementation of features
- Parse TDD acceptance criteria and checkbox states
- Run coverage tools and validate targets
- Generate detailed implementation status reports
- Auto-update TDD checkboxes with --update flag
- Flag mismatches between marked status and actual implementation

**Will Not**:

- Modify or write test code
- Modify or write implementation code
- Change TDD requirements
- Judge test quality or coverage appropriateness
- Run tests to verify correctness
- Make technical or architectural decisions

## Troubleshooting

### "No acceptance criteria section found"

The TDD must have a section titled exactly:

```markdown
## Acceptance Criteria
```

### "No tests found for criterion"

The skill searches for matching test descriptions, keywords, and function names. If tests exist but weren't found:

- Test descriptions may not match criterion keywords
- Tests may be in unexpected file locations
- Test naming conventions may differ from expected patterns

Review the "Missing Tests" section of the report for details.

### "Coverage command failed"

The skill attempts to detect project type and run appropriate coverage command. If it fails:

- Verify test command works manually
- Check that coverage tool is configured
- Ensure test dependencies are installed

### "False positive: marked as missing but exists"

If implementation exists but wasn't detected:

- Code may use different naming than criterion specifies
- Implementation may be in unexpected file location
- Search patterns may need adjustment for codebase conventions

Review the report's search details to understand what was searched.

## Related Documentation

- [TDD Writer Skill](../tdd-writer/SKILL.md) - Generate TDDs
- [TDD Templates](../tdd-writer/templates/) - TDD structure reference
- [TDD Examples](../tdd-writer/examples/) - Well-formed TDD examples
- [TDD Style Guide](../tdd-writer/style-guide.md) - TDD writing conventions

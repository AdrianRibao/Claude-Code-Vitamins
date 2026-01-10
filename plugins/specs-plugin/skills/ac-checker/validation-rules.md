# Acceptance Criteria Validation Rules

Complete reference for validating acceptance criteria in TDDs.

## Table of Contents

1. [Required Subsections](#required-subsections)
2. [Format Requirements](#format-requirements)
3. [Testability Rules](#testability-rules)
4. [Testing Requirements](#testing-requirements)
5. [Type-Specific Requirements](#type-specific-requirements)
6. [Coverage Targets](#coverage-targets)

## Required Subsections

Every TDD acceptance criteria section MUST contain these subsections in order:

### 1. Core Functionality

Primary feature requirements and user-facing capabilities.

**Purpose**: Define what the feature does from a user perspective.

**Example**:

```markdown
### Core Functionality

- [ ] User can create a new incident with required fields
- [ ] User can view list of their incidents
- [ ] User can update incident status
- [ ] User can archive completed incidents
```

### 2. Validation

Input validation rules, constraint checks, and data integrity requirements.

**Purpose**: Ensure data quality and business rule enforcement.

**Example**:

```markdown
### Validation

- [ ] Title field is required and max 255 characters
- [ ] Description is required and max 5000 characters
- [ ] Status must be one of: draft, submitted, approved, rejected
- [ ] Invalid status transitions are rejected with clear error messages
```

### 3. Authorization

Permission checks, access control, and role-based requirements.

**Purpose**: Define who can perform which actions.

**Example**:

```markdown
### Authorization

- [ ] Unauthenticated requests return 401
- [ ] Users can only view their own incidents
- [ ] Managers can view all team incidents
- [ ] Admins have full access to all incidents
```

### 4. Testing

Test requirements with specific coverage targets and test types.

**Purpose**: Define testing expectations and quality standards.

**Example**:

```markdown
### Testing

- [ ] Unit tests written for all business logic and validations
- [ ] Unit tests cover all constraint rules and status transitions
- [ ] Policy tests verify all authorization rules and permissions
- [ ] Policy tests validate each actor role against policy matrix
- [ ] Integration tests verify complete CRUD workflows with authorization
- [ ] Test coverage ≥ 80% for domain logic
- [ ] Test coverage ≥ 90% for critical business rules
- [ ] Test coverage ≥ 95% for authentication and authorization logic
```

### 5. Code Quality

Code standards, documentation, and maintainability requirements.

**Purpose**: Ensure code meets project quality standards.

**Example**:

```markdown
### Code Quality

- [ ] All new modules have type annotations and documentation
- [ ] No compiler/linter warnings
- [ ] Code formatted per project standards
```

## Format Requirements

### Valid Checkbox Formats

✅ **Correct**:

```markdown
- [ ] Criterion description
- [x] Completed criterion
```

❌ **Incorrect**:

```markdown
- [] Missing space in checkbox
-[] Missing space after dash
* [ ] Wrong bullet character (asterisk)
+ [ ] Wrong bullet character (plus)
[ ] Missing dash
-[ ] Missing space after dash
```

### Criterion Structure

Each criterion should:

- Start with checkbox: `- [ ]` or `- [x]`
- Use clear, active voice
- Be a single, focused requirement
- End without punctuation (no period)

**Good**:

```markdown
- [ ] User receives email notification when incident is approved
- [ ] API returns 404 when incident ID does not exist
- [ ] Form validates email format before submission
```

**Bad**:

```markdown
- [ ] User should receive notification. (punctuation, passive voice)
- [ ] System works properly (vague)
- [ ] Handle errors and validate inputs (multiple requirements)
```

## Testability Rules

### Specific and Measurable

Criteria must be specific enough to verify through testing.

**Forbidden Vague Terms**:

- "properly"
- "correctly"
- "appropriately"
- "adequately"
- "sufficiently"
- "well"
- "good"
- "nice"

**Replace with specifics**:

| Vague                            | Specific                                                                       |
| -------------------------------- | ------------------------------------------------------------------------------ |
| "System handles errors properly" | "API returns 500 with error message when database connection fails"            |
| "UI looks good"                  | "UI displays incident form with title, description, and submit button"         |
| "Feature works correctly"        | "User can create incident and view it in incident list within 2 seconds"       |
| "Validation is sufficient"       | "Form rejects submissions with missing title field and displays error message" |

### Observable Outcomes

Criteria should define observable, verifiable outcomes.

**Good - Observable**:

```markdown
- [ ] User receives email within 5 minutes of incident approval
- [ ] Dashboard displays incident count badge with correct number
- [ ] Form disables submit button when required fields are empty
- [ ] API response time < 100ms for incident list endpoint
```

**Bad - Not Observable**:

```markdown
- [ ] System is performant
- [ ] Code is maintainable
- [ ] Architecture is scalable
- [ ] User experience is smooth
```

### Include Acceptance Conditions

Where applicable, specify conditions for acceptance.

**Pattern**: `[Action] when [condition] results in [outcome]`

**Examples**:

```markdown
- [ ] Creating incident with invalid email returns 422 error
- [ ] Archiving incident when user is not owner returns 403 forbidden
- [ ] Updating incident status from draft to submitted triggers email notification
- [ ] Listing incidents with search filter returns only matching results
```

## Testing Requirements

### Required Test Types

All TDDs must specify:

1. **Unit Tests**: Test individual functions and business logic
2. **Integration Tests**: Test complete workflows and interactions
3. **Coverage Targets**: Specific percentages for different code areas

### Coverage Targets Format

Must use this exact format with ≥ symbol and percentages:

```markdown
- [ ] Test coverage ≥ 80% for domain logic
- [ ] Test coverage ≥ 90% for critical business rules
- [ ] Test coverage ≥ 95% for authentication and authorization logic
```

**Also acceptable**:

```markdown
- [ ] Maintain ≥80% test coverage for domain logic
- [ ] Achieve ≥90% test coverage for critical business rules
```

❌ **Not acceptable**:

```markdown
- [ ] Test coverage should be high
- [ ] Most code is tested
- [ ] Good test coverage
```

### Test Specificity

Tests should be specific to the domain:

```markdown
- [ ] Unit tests cover incident creation with all field validations
- [ ] Unit tests verify status transition rules (draft→submitted→approved)
- [ ] Integration tests validate incident CRUD with authorization checks
- [ ] Edge case tests for concurrent incident updates
```

## Type-Specific Requirements

### Backend TDDs

**Must include**:

```markdown
### Testing

- [ ] Policy tests verify all authorization rules and permissions
- [ ] Policy tests validate each actor role against policy matrix
- [ ] Policy tests cover authentication failures and unauthorized access attempts
- [ ] Test coverage ≥ 95% for authentication and authorization logic
```

### UI TDDs

**Must include**:

```markdown
### Testing

- [ ] Accessibility tests verify WCAG 2.1 AA compliance
- [ ] Keyboard navigation tests for all interactive elements
- [ ] Screen reader tests for critical user flows
- [ ] UI tests cover all user-facing changes
```

### API TDDs

**Must include**:

```markdown
### Testing

- [ ] API contract tests verify request/response schemas
- [ ] Authentication tests for all protected endpoints
- [ ] Error handling tests for 4xx and 5xx responses
- [ ] Policy tests verify all authorization rules and permissions
```

### Integration TDDs

**Must include**:

```markdown
### Testing

- [ ] End-to-end tests verify complete user workflows
- [ ] Integration tests validate cross-domain interactions
- [ ] Event flow tests confirm message passing between systems
- [ ] Error recovery tests for external system failures
```

## Coverage Targets

### Standard Thresholds

| Code Category           | Minimum Coverage | Rationale              |
| ----------------------- | ---------------- | ---------------------- |
| Domain logic            | ≥80%             | General business logic |
| Critical business rules | ≥90%             | High-impact logic      |
| Auth/authorization      | ≥95%             | Security-critical code |

### When to Specify Higher Targets

Use higher coverage targets (≥95%) for:

- Payment processing
- Data encryption/decryption
- Authorization decisions
- Audit trail logic
- Compliance-related code

### Performance Testing

When applicable, include performance criteria:

```markdown
### Testing

- [ ] Performance tests validate query response times < 100ms
- [ ] Load tests verify system handles 1000 concurrent users
- [ ] Stress tests confirm graceful degradation under load
```

## Validation Severity Levels

### Critical (Must Fix Before Implementation)

- Missing required subsections
- Non-testable criteria (vague language)
- Missing coverage targets
- Missing type-specific requirements (policy tests, a11y tests)

### Important (Should Fix Soon)

- Incorrect checkbox format
- Criteria that combine multiple requirements
- Missing specific acceptance conditions
- Ambiguous language

### Minor (Nice to Have)

- Inconsistent wording style
- Missing punctuation in descriptions
- Criteria ordering within subsections

## Auto-Fix Capabilities

The checker can automatically fix:

✅ **Safe Auto-Fixes**:

- Checkbox format: `- []` → `- [ ]`
- Bullet character: `* [ ]` → `- [ ]`
- Missing subsection headers (adds placeholder)
- Coverage target format standardization

❌ **Requires Manual Review**:

- Vague language (needs rewriting)
- Missing requirements (needs domain knowledge)
- Testability issues (needs clarification)
- Incorrect requirement logic (needs validation)

# Acceptance Criteria Style Guide

Writing conventions and best practices for TDD acceptance criteria.

## Core Principles

1. **Testable**: Every criterion must be verifiable through testing
2. **Specific**: Use concrete terms, avoid vague language
3. **Measurable**: Include numbers, limits, or observable outcomes where applicable
4. **Focused**: One requirement per checkbox
5. **Consistent**: Follow standard format and structure

## Format Standards

### Checkbox Syntax

Always use this exact format:

```markdown
- [ ] Criterion description
```

**Spacing rules**:

- Dash, space, opening bracket, space, closing bracket, space, description
- No period at the end
- Use lowercase unless proper noun

### Active Voice

Use active voice with clear subject and action.

✅ **Good**:

```markdown
- [ ] User can create incident with required fields
- [ ] API returns 404 when resource not found
- [ ] Form validates email format before submission
```

❌ **Bad**:

```markdown
- [ ] Incident can be created (passive voice, unclear who)
- [ ] 404 is returned (passive voice)
- [ ] Email should be validated (passive, vague timing)
```

### One Requirement Per Criterion

Each checkbox should test exactly one thing.

✅ **Good**:

```markdown
- [ ] User can create incident
- [ ] User can update incident
- [ ] User can delete incident
```

❌ **Bad**:

```markdown
- [ ] User can create, update, and delete incidents
```

## Writing for Testability

### Use Concrete Terms

Replace vague qualifiers with specific conditions.

| Vague           | Concrete                    |
| --------------- | --------------------------- |
| "properly"      | Specify expected behavior   |
| "correctly"     | Define correct outcome      |
| "appropriately" | State specific requirements |
| "good"          | Measure quality criteria    |
| "sufficient"    | Set numeric thresholds      |

### Include Observable Outcomes

State what can be seen, measured, or verified.

✅ **Observable**:

```markdown
- [ ] Dashboard displays incident count badge
- [ ] Email sent within 5 minutes
- [ ] API response time < 100ms
- [ ] Error message "Title is required" displayed
```

❌ **Not Observable**:

```markdown
- [ ] Dashboard is informative
- [ ] Email is timely
- [ ] API is fast
- [ ] Error handling is good
```

### Specify Conditions

Use "when/then" pattern for conditional behavior.

**Pattern**: `[Action] when [condition]`

✅ **Good**:

```markdown
- [ ] API returns 401 when authentication token is missing
- [ ] Form disables submit button when required fields are empty
- [ ] User receives notification when incident is assigned to them
```

## Subsection Organization

### Core Functionality

Primary feature capabilities users interact with directly.

**Focus on**:

- User actions (create, read, update, delete)
- Main workflows
- Core business features

**Example**:

```markdown
### Core Functionality

- [ ] User can create incident with title and description
- [ ] User can view list of their incidents
- [ ] User can update incident status
- [ ] User can assign incident to team member
- [ ] User can add comments to incident
```

### Validation

Data quality, constraints, and business rule enforcement.

**Focus on**:

- Required fields
- Format validation
- Constraint checks
- Business rule validation

**Example**:

```markdown
### Validation

- [ ] Title field is required
- [ ] Title max length is 255 characters
- [ ] Email address follows RFC 5322 format
- [ ] Start date must be before end date
- [ ] Status transitions follow defined state machine
```

### Authorization

Access control, permissions, and role-based requirements.

**Focus on**:

- Authentication requirements
- Permission checks
- Role-based access
- Ownership rules

**Example**:

```markdown
### Authorization

- [ ] Unauthenticated requests return 401
- [ ] Users can only view their own incidents
- [ ] Managers can view all team incidents
- [ ] Admins have full access to all incidents
- [ ] Deleted incidents are not accessible to regular users
```

### Testing

Test requirements with coverage targets and test types.

**Focus on**:

- Unit test scope
- Integration test scope
- Coverage percentages
- Type-specific tests (policy, accessibility)

**Must include**:

- Coverage targets with specific percentages
- Backend/API: Policy tests
- UI: Accessibility tests

**Example**:

```markdown
### Testing

- [ ] Unit tests written for all business logic and validations
- [ ] Unit tests cover all status transition rules
- [ ] Policy tests verify all authorization rules and permissions
- [ ] Integration tests verify complete CRUD workflows
- [ ] Test coverage ≥ 80% for domain logic
- [ ] Test coverage ≥ 90% for critical business rules
- [ ] Test coverage ≥ 95% for authentication and authorization logic
```

### Code Quality

Code standards, documentation, and maintainability.

**Focus on**:

- Type annotations
- Documentation
- Linter/formatter compliance
- Code review standards

**Example**:

```markdown
### Code Quality

- [ ] All new modules have type annotations and documentation
- [ ] No compiler/linter warnings
- [ ] Code formatted per project standards
- [ ] All public functions have docstrings
```

## Coverage Target Format

### Required Format

Use ≥ symbol with specific percentages:

✅ **Correct**:

```markdown
- [ ] Test coverage ≥ 80% for domain logic
- [ ] Test coverage ≥ 90% for critical business rules
- [ ] Test coverage ≥ 95% for authentication and authorization logic
```

❌ **Incorrect**:

```markdown
- [ ] Test coverage should be high
- [ ] Test coverage at least 80%
- [ ] Good test coverage
- [ ] Test coverage >= 80%
```

### Standard Thresholds

Use these unless project specifies otherwise:

- Domain logic: ≥ 80%
- Critical business rules: ≥ 90%
- Auth/authorization: ≥ 95%

## Type-Specific Requirements

### Backend TDDs

**Always include**:

```markdown
- [ ] Policy tests verify all authorization rules and permissions
- [ ] Policy tests validate each actor role against policy matrix
- [ ] Policy tests cover authentication failures and unauthorized access attempts
- [ ] Test coverage ≥ 95% for authentication and authorization logic
```

### UI TDDs

**Always include**:

```markdown
- [ ] Accessibility tests verify WCAG 2.1 AA compliance
- [ ] Keyboard navigation tests for all interactive elements
- [ ] Screen reader tests for critical user flows
- [ ] UI tests cover all user-facing changes
```

### API TDDs

**Always include**:

```markdown
- [ ] API contract tests verify request/response schemas
- [ ] Authentication tests for all protected endpoints
- [ ] Error handling tests for 4xx and 5xx responses
- [ ] Policy tests verify all authorization rules and permissions
```

## Numbers and Limits

### Always Specify Limits

When mentioning constraints, include specific values.

✅ **Good**:

```markdown
- [ ] Title max length is 255 characters
- [ ] File upload size limit is 10MB
- [ ] API rate limit is 1000 requests per hour
- [ ] Password minimum length is 12 characters
```

❌ **Bad**:

```markdown
- [ ] Title has reasonable length
- [ ] File size is limited
- [ ] API has rate limiting
- [ ] Password is secure
```

### Performance Criteria

Include specific thresholds for performance requirements.

✅ **Good**:

```markdown
- [ ] API response time < 100ms for list endpoint
- [ ] Page loads in < 2 seconds on 3G connection
- [ ] Search returns results in < 500ms
- [ ] Background job completes in < 5 minutes
```

❌ **Bad**:

```markdown
- [ ] API is fast
- [ ] Page loads quickly
- [ ] Search is responsive
- [ ] Background job is efficient
```

## Error Handling

### Specify Error Codes and Messages

Define expected error responses clearly.

✅ **Good**:

```markdown
- [ ] API returns 401 with message "Authentication required" when token is missing
- [ ] API returns 403 with message "Insufficient permissions" when user lacks access
- [ ] API returns 422 with field-level errors when validation fails
- [ ] Form displays "Title is required" when title field is empty
```

❌ **Bad**:

```markdown
- [ ] API returns error when authentication fails
- [ ] API handles authorization errors
- [ ] Form shows validation errors
```

## Common Mistakes

### Avoid Combining Multiple Requirements

❌ **Wrong**:

```markdown
- [ ] User can create, update, and delete incidents and receive notifications
```

✅ **Right**:

```markdown
- [ ] User can create incident
- [ ] User can update incident
- [ ] User can delete incident
- [ ] User receives notification when incident is created
```

### Avoid Implementation Details

❌ **Wrong**:

```markdown
- [ ] Incident model has validates_presence_of :title
- [ ] Controller uses before_action :authenticate_user!
- [ ] Service object calls IncidentMailer.notify
```

✅ **Right**:

```markdown
- [ ] Title field is required
- [ ] Unauthenticated requests return 401
- [ ] User receives email notification when incident is created
```

### Avoid Subjective Language

❌ **Wrong**:

```markdown
- [ ] UI is intuitive
- [ ] Form is user-friendly
- [ ] Error messages are helpful
- [ ] Dashboard is easy to use
```

✅ **Right**:

```markdown
- [ ] UI displays incident form with labeled fields
- [ ] Form shows field-level validation errors
- [ ] Error message specifies which field failed validation
- [ ] Dashboard displays incident list with search and filter controls
```

## Examples

### Complete Acceptance Criteria Section

```markdown
## Acceptance Criteria

### Core Functionality

- [ ] User can create incident with title and description
- [ ] User can view list of their incidents
- [ ] User can update incident status
- [ ] User can assign incident to team member
- [ ] User can add comments to incident

### Validation

- [ ] Title field is required
- [ ] Title max length is 255 characters
- [ ] Description is required
- [ ] Description max length is 5000 characters
- [ ] Status must be one of: draft, submitted, approved, rejected
- [ ] Status transitions follow defined rules (draft→submitted→approved)

### Authorization

- [ ] Unauthenticated requests return 401
- [ ] Users can only view their own incidents
- [ ] Managers can view all team incidents
- [ ] Admins have full access to all incidents
- [ ] Users cannot update incidents after approval

### Testing

- [ ] Unit tests written for all business logic and validations
- [ ] Unit tests cover all status transition rules
- [ ] Policy tests verify all authorization rules and permissions
- [ ] Policy tests validate each actor role against policy matrix
- [ ] Integration tests verify complete CRUD workflows with authorization
- [ ] Edge case tests for concurrent incident updates
- [ ] Test coverage ≥ 80% for domain logic
- [ ] Test coverage ≥ 90% for critical business rules
- [ ] Test coverage ≥ 95% for authentication and authorization logic

### Code Quality

- [ ] All new modules have type annotations and documentation
- [ ] No compiler/linter warnings
- [ ] Code formatted per project standards
```

## Checklist Before Finalizing

Use this checklist to verify acceptance criteria quality:

- [ ] All required subsections present (Core, Validation, Authorization, Testing, Code Quality)
- [ ] All checkboxes use correct format `- [ ]`
- [ ] No vague language (properly, correctly, appropriately)
- [ ] All criteria are testable and specific
- [ ] Coverage targets explicitly stated with ≥ symbol
- [ ] Type-specific requirements included (policy/accessibility tests)
- [ ] Error codes and messages specified where applicable
- [ ] Performance criteria include specific thresholds
- [ ] Each criterion focuses on one requirement
- [ ] Active voice used throughout

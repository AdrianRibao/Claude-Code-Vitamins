# TDD Style Guide

## Core Principles

### WHAT, Not HOW

Every TDD section answers "what needs to be built" not "how to implement it". Implementation details belong in code.

### Tables Over Prose

Use structured tables for:

- Data model attributes
- Authorization matrices
- Interface contracts
- Business rules

### Signatures Only

Code blocks contain only:

- Function/method signatures
- Action names with parameters
- Return type annotations

**Never include**: function bodies, algorithm implementations, boilerplate.

### Maximum Code Block Length: 5 Lines

Any code block longer than 5 lines is a red flag. Reduce to signatures or move to implementation.

## Data Model Format

### Attributes Table

```markdown
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| id | UUID | Yes | Primary key |
| name | string | Yes | Display name, max 255 chars |
| status | enum | Yes | active, inactive, archived |
| created_at | datetime | Yes | UTC timestamp |
```

### Relationships Table

```markdown
| Relation | Target | Type | Description |
|----------|--------|------|-------------|
| owner | User | belongs_to | Record creator |
| items | Item | has_many | Child records |
| category | Category | belongs_to | Optional classification |
```

### Constraints

Use bullet points for database-level constraints:

- Unique constraint on `[field_a, field_b]`
- Check constraint: `end_date > start_date`
- Foreign key cascade delete on `parent_id`

## Interface Contract Format

### Actions Table

```markdown
| Action | Type | Arguments | Returns | Description |
|--------|------|-----------|---------|-------------|
| create | create | params, actor | Resource | Creates with validation |
| list | read | query, actor | [Resource] | Paginated with filters |
| update | update | id, params, actor | Resource | Updates allowed fields |
| archive | update | id, actor | Resource | Soft delete |
```

### Code Interface (Signatures Only)

```elixir
# Good: Signatures only
Domain.create_feature(params, actor: user)
Domain.list_features(query: [...], actor: user)
Domain.get_feature!(id, actor: user)
```

```elixir
# Bad: Implementation details
def create_feature(params, actor: user) do
  params
  |> validate_required([:name, :type])
  |> check_authorization(actor)
  |> Repo.insert()
end
```

## Authorization Matrix Format

### Policy Matrix

```markdown
| Actor | Create | Read | Update | Delete |
|-------|--------|------|--------|--------|
| Owner | ✅ Own | ✅ Own | ✅ Own | ✅ Own |
| Manager | ✅ Team | ✅ Team | ✅ Team | ❌ |
| Admin | ✅ All | ✅ All | ✅ All | ✅ All |
| Guest | ❌ | ✅ Public | ❌ | ❌ |
```

### Policy Rules

Use plain language for complex rules:

- Owner can only modify records in `draft` status
- Manager can approve records in their team's queue
- Admin override requires audit log entry

## Behavior Specification Format

### Given/When/Then Structure

```markdown
### Creating a Record

**Given**: User has `create` permission
**When**: User submits valid form data
**Then**:
- Record is created with `draft` status
- Audit log entry is recorded
- Success notification is displayed
```

### Edge Cases

Document edge cases with the same structure:

```markdown
### Creating Duplicate Record

**Given**: Record with same `name` exists for user
**When**: User attempts to create with duplicate name
**Then**:
- Creation is rejected
- Error message: "A record with this name already exists"
```

## Acceptance Criteria Format

### Testable Checkboxes

```markdown
## Acceptance Criteria

### Core Functionality
- [ ] User can create a new record with required fields
- [ ] User can view their own records in a list
- [ ] User can update records they own
- [ ] User cannot access records owned by others

### Validation
- [ ] Name field is required and max 255 characters
- [ ] Status defaults to "draft" on creation
- [ ] End date must be after start date

### Authorization
- [ ] Unauthenticated users are redirected to login
- [ ] Users without permission see 403 error
```

## Document Structure

### Header Section

```markdown
# TDD: [Feature Name]

| Document   | Link                              |
| ---------- | --------------------------------- |
| Parent PRD | [prd-name.md](../prds/prd-name.md) |
| Master TDD | [00-master.md](./00-master.md)    |

| Domain        | Priority | Status   | Last Updated |
| ------------- | -------- | -------- | ------------ |
| `Module.Name` | P1       | Planning | December 2025 |
```

### Sections Order

1. Overview (Purpose, Scope, Key Business Rules)
2. Data Model (Attributes, Relationships, Constraints)
3. Interface Contract (Actions, Code Interface)
4. Authorization (Policy Matrix, Policy Rules)
5. Behavior Specifications
6. Acceptance Criteria
7. Open Questions
8. Related Documents

## Testing Requirements Format

### Testing Subsection

All TDDs must include a **Testing** subsection under Acceptance Criteria with testable checkboxes.

### Coverage Targets

Specify minimum test coverage percentages for different code categories:

```markdown
### Testing

- [ ] Test coverage ≥ 80% for domain logic
- [ ] Test coverage ≥ 90% for critical business rules
- [ ] Test coverage ≥ 95% for authentication and authorization logic
```

### Test Types by TDD Type

**Backend TDD**:

- Unit tests for business logic and validations
- **Policy tests for authorization rules** (critical requirement)
- Integration tests for CRUD workflows
- Performance tests for query response times
- Load tests for concurrent operations

**UI TDD**:

- Unit tests for component logic and state
- Component tests for prop handling and events
- **Accessibility tests for WCAG compliance** (critical requirement)
- E2E tests for user workflows
- Visual regression tests for UI changes
- Cross-browser compatibility tests

**API TDD**:

- Unit tests for endpoint handlers
- **API contract tests for schema validation** (critical requirement)
- **Authentication/authorization tests** (critical requirement)
- Rate limiting tests
- Security tests for input sanitization
- Load tests for concurrent requests
- OpenAPI/Swagger spec validation

**Integration TDD**:

- Unit tests for data mapping and transformations
- Integration tests with mocked external services
- Contract tests for external API compatibility
- End-to-end tests with real test environment
- Chaos tests for failure scenarios
- Monitoring tests for metrics and alerts

### Policy Testing (Critical)

**Authentication and authorization tests are non-negotiable requirements**. Every TDD must explicitly include:

```markdown
- [ ] **Policy tests verify all authorization rules and permissions**
- [ ] **Policy tests validate each actor role against policy matrix**
- [ ] **Policy tests cover authentication failures and unauthorized access attempts**
- [ ] **Test coverage ≥ 95% for authentication and authorization logic**
```

### Performance Testing

Include measurable performance criteria:

```markdown
- [ ] Performance tests validate query response times < 100ms
- [ ] Performance tests validate page load < 3s, interaction < 100ms
- [ ] Performance tests validate API response times meet SLA (< 200ms)
```

### Test Specifications (Not Implementation)

Testing criteria specify **WHAT** to test, not **HOW** to test:

✅ **Good**: Testable requirements

```markdown
- [ ] Unit tests validate all constraint rules and status transitions
- [ ] Policy tests verify Owner can only update draft status records
- [ ] E2E tests cover complete user workflows (create, edit, delete)
```

❌ **Bad**: Test implementation code

```elixir
test "owner can update draft" do
  user = create_user()
  record = create_record(status: :draft, owner: user)
  assert {:ok, _} = update_record(record, %{name: "new"}, actor: user)
end
```

## Anti-Patterns to Avoid

### Code Bloat

❌ **Bad**: Full module with implementation

```elixir
defmodule MyApp.Feature do
  use Ash.Resource

  attributes do
    uuid_primary_key :id
    attribute :name, :string
    # ... 50 more lines
  end
end
```

✅ **Good**: Attributes table only

```markdown
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| id | UUID | Yes | Primary key |
| name | string | Yes | Display name |
```

### Implementation Details

❌ **Bad**: "Use GenServer for caching" ✅ **Good**: "Cache results for 5 minutes"

### Test Implementations

❌ **Bad**: Full test file with assertions ✅ **Good**: Acceptance criteria as checkboxes

### Vague Requirements

❌ **Bad**: "Handle errors appropriately" ✅ **Good**: "Display validation errors inline next to form fields"

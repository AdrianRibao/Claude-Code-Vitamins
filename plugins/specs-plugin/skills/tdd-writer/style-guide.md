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

❌ **Bad**: "Use GenServer for caching"
✅ **Good**: "Cache results for 5 minutes"

### Test Implementations

❌ **Bad**: Full test file with assertions
✅ **Good**: Acceptance criteria as checkboxes

### Vague Requirements

❌ **Bad**: "Handle errors appropriately"
✅ **Good**: "Display validation errors inline next to form fields"

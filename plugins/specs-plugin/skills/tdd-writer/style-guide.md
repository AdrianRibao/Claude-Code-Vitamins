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

## Authorization Model Format

TDDs specify authorization through three layers. Use what fits the feature's complexity — simple CRUD features may only need Layer 1, while features with sensitive data or complex workflows benefit from all three.

### Layer 1: Action Permissions

Who can perform which actions, and on what scope. Go beyond basic CRUD — include domain-specific actions like approve, publish, escalate, export, delegate.

**Scope qualifiers**:

| Scope | Meaning                       | Filter example              |
| ----- | ----------------------------- | --------------------------- |
| Own   | Records created by actor      | `user_id = actor.id`        |
| Team  | Records in actor's team       | `team_id IN actor.team_ids` |
| Dept  | Records in actor's department | `dept_id = actor.dept_id`   |
| Org   | All records in organization   | `org_id = actor.org_id`     |
| All   | No restriction                | -                           |

**Action permissions matrix**:

```markdown
| Actor | Create | Read | Update | Archive | Approve | Export |
|-------|--------|------|--------|---------|---------|--------|
| Owner | ✅ Own | ✅ Own | ✅ Own | ✅ Own | ❌ | ✅ Own |
| Manager | ✅ Team | ✅ Team | ✅ Team | ✅ Team | ✅ Team | ✅ Team |
| Admin | ✅ All | ✅ All | ✅ All | ✅ All | ✅ All | ✅ All |
| Guest | ❌ | ✅ Public | ❌ | ❌ | ❌ | ❌ |
```

Columns should reflect the actual actions in the Interface Contract, not just generic CRUD.

### Layer 2: Data Permissions

Which fields each role can see or modify. Use when different roles have different field visibility or edit rights.

**Visibility levels**:

- **RW** - Read and Write (can view and modify)
- **R** - Read-only (can view, cannot modify)
- **Hidden** - Not visible (field excluded from responses for this role)

```markdown
| Field | Owner | Manager | Admin | Guest |
|-------|-------|---------|-------|-------|
| name | RW | RW | RW | R |
| email | RW | R | RW | Hidden |
| status | R | RW | RW | R |
| salary | Hidden | R | RW | Hidden |
| ssn | Hidden | Hidden | R | Hidden |
| internal_notes | Hidden | RW | RW | Hidden |
```

**When to include Layer 2**:

- Features with sensitive/PII fields
- Different roles see different dashboards or detail views
- Field-level edit restrictions (e.g., only admins can change status)
- API responses vary by caller role

**When to skip**: All roles see and edit the same fields. Also skip for pure integration TDDs where field filtering is handled upstream — use the data boundary format instead (see integration template).

### Layer 3: Permission Conditions

When permissions apply or change based on context. Captures rules that can't be expressed in a static matrix.

**Condition types**:

| Type               | Example                                       |
| ------------------ | --------------------------------------------- |
| Status-based       | Owner can update only when `status = 'draft'` |
| Time-based         | Edits allowed within 24h of creation          |
| Relationship-based | Manager sees records in their team's scope    |
| Approval-based     | Publish requires `approved_by IS NOT NULL`    |
| Quantity-based     | Free tier limited to 10 records               |

**Permission conditions table**:

```markdown
| Rule ID | Condition | Effect | Actors |
|---------|-----------|--------|--------|
| PC-01 | `status = 'draft'` | Can update | Owner |
| PC-02 | `created_at > now() - 24h` | Can delete | Owner |
| PC-03 | `approved_by IS NOT NULL` | Can publish | Manager, Admin |
| PC-04 | `team_id IN actor.team_ids` | Can read | Manager |
```

Rule IDs (PC-01, PC-02...) should be referenced in acceptance criteria for traceability.

### Adaptation for Integration TDDs

Integration TDDs adapt Layer 2 from role-based columns to **data boundary columns** (Outbound/Inbound/Internal) since the key concern is what data crosses system boundaries, not per-role visibility. See the integration template for the format.

### When to Skip Authorization

If a feature has no authentication or authorization (public pages, anonymous endpoints, internal-only jobs), skip the authorization section entirely rather than generating empty tables.

### Policy Rules (Plain Language)

Use plain language to summarize complex interactions between layers:

- Owner can only modify records in `draft` status (PC-01)
- Manager can approve records in their team's queue (PC-04)
- Admin override requires audit log entry
- Salary field visible only to Managers (read) and Admins (read-write)

## Permission Matrix Test Plan

The Permission Matrix Test Plan is a **required, mechanically-derived artifact** of any TDD whose Authorization section is non-empty. It exists because the highest-severity bugs (privilege escalation, data exposure, IDOR) hide in the deny paths — the cells of the matrix where someone should NOT be able to do something. A test plan that only enumerates allow paths leaves those bugs unguarded.

The plan covers three dimensions: the **subject** (who acts — Layer 1 rows), the **object** (whose record is acted on — Layer 1 columns + scope), and the **referent** (which other records the payload points at — the `F` rows). IDOR lives in the third dimension: an authorized actor, inside their own scope, smuggles a foreign record in as an argument — the policy passes (the actor is legitimate), the FK passes (the record exists), and a cross-scope link is written. Only `Referent deny` rows catch this confused-deputy shape.

### Required Count (formula)

Total required rows `N = A + S + F + H + R + 2·P + E`:

| Symbol | Source                                                                                                                 | Counts                                                                                                                                                                                                                                                                                                                                                                                 |
| ------ | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `A`    | Layer 1 matrix                                                                                                         | Every cell — allow cells get an Allow row, deny cells get a Deny row                                                                                                                                                                                                                                                                                                                   |
| `S`    | Layer 1 scope-qualified `✅` cells                                                                                     | One cross-scope deny per scope-qualified allow (Owner-Own becomes "Owner attempts another Owner's record → 403")                                                                                                                                                                                                                                                                       |
| `F`    | Referent inputs — every (action, foreign-key input) pair where the input references another **scope-qualified** record | One Referent deny per pair — an authorized actor, inside their own scope, submits an id from another scope; expect explicit rejection and zero state written. Counts inputs, not actions (an action accepting 3 FKs contributes 3); covers create/update payloads, nested/relationship writes, bulk id lists, polymorphic ids. Global lookup tables (currencies, countries) are exempt |
| `H`    | Layer 2 `Hidden` (role, field) pairs                                                                                   | One row per pair asserting field absent from payload and DOM                                                                                                                                                                                                                                                                                                                           |
| `R`    | Layer 2 `R` read-only (role, field)                                                                                    | One row per pair asserting write attempt rejected (422 / changeset error / 403)                                                                                                                                                                                                                                                                                                        |
| `P`    | Layer 3 `PC-xx` rules                                                                                                  | Two rows per rule — one Allow when condition met, one Deny when condition not met                                                                                                                                                                                                                                                                                                      |
| `E`    | Protected entry points                                                                                                 | One row per route/action asserting `401` for unauthenticated request (distinct signal from `403`)                                                                                                                                                                                                                                                                                      |

The Acceptance Criteria > Testing subsection must assert `Policy test count ≥ N` and spell out the arithmetic so a reviewer can verify it without recomputing.

### Test Plan Table Format

```markdown
### Permission Matrix Test Plan

> N = A (Layer 1 cells) + S (cross-scope) + F (referent inputs) + H (Hidden pairs) + R (read-only pairs) + 2·P (PC rules) + E (auth boundaries)
> For this TDD: N = 12 + 3 + 2 + 2 + 2 + 2·3 + 4 = **31 required tests**.

| PT-ID  | Layer | Cell / Rule           | Actor   | Action / Field      | Type                | Expected signal                                  | State assertion                            |
| ------ | ----- | --------------------- | ------- | ------------------- | ------------------- | ------------------------------------------------ | ------------------------------------------ |
| PT-01  | 1     | Owner × create        | Owner   | create(own)         | Allow               | 201 Created, record returned                     | New row persisted with `owner_id = actor`  |
| PT-02  | 1     | Owner × approve       | Owner   | approve             | Deny                | 403 Forbidden                                    | No status change, no audit event           |
| PT-03  | 1-S   | Owner × update(other) | Owner   | update on other Owner's record | Cross-scope deny    | 403 Forbidden                                    | Target row unchanged                       |
| PT-04  | 1-F   | `create.project_id`   | Owner   | create naming another org's project | Referent deny       | 403 Forbidden (or 422, no disclosure)            | No row written in **either** scope; no link created |
| PT-05  | 2-H   | Hidden field          | Guest   | response.salary     | Hidden              | 200 OK, `salary` key absent from JSON and DOM    | -                                          |
| PT-06  | 2-R   | Read-only field       | Owner   | PATCH status        | Read-only deny      | 422 with field error `{status: ["read-only"]}`   | Row.status unchanged                       |
| PT-07  | 3     | PC-01 (status=draft)  | Owner   | update              | Condition met       | 200 OK                                           | Row updated                                |
| PT-08  | 3     | PC-01 (status=active) | Owner   | update              | Condition not met   | 403 Forbidden                                    | Row unchanged                              |
| PT-09  | E     | Unauthenticated GET   | none    | GET /features       | Auth boundary       | 401 Unauthorized (NOT 403, NOT 404)              | -                                          |
| PT-10  | 1-AD  | Cross-org GET         | Owner   | GET /features/:id   | Anti-discovery deny | 404 Not Found (justified: leaks existence)        | No record returned, no info in body         |
```

**Column rules**:

- **PT-ID**: zero-padded `PT-01`, `PT-02`, … globally unique within the TDD. The `-S` (cross-scope), `-F` (referent), `-H` (Hidden), `-R` (read-only), `-AD` (anti-discovery) suffix on the Layer column makes the row type scannable.
- **Layer**: `1`, `2`, `3`, or `E` (entry-point auth boundary). Suffix the type variant as above.
- **Cell / Rule**: the exact matrix cell or PC-xx referenced. Bare "Owner × update" is fine when unambiguous; otherwise quote the rule.
- **Actor**: the role under test, including `none` for unauthenticated.
- **Action / Field**: the verb and (where applicable) the field being read/written.
- **Type**: one of `Allow`, `Deny`, `Cross-scope deny`, `Referent deny`, `Hidden`, `Read-only deny`, `Condition met`, `Condition not met`, `Auth boundary`, `Anti-discovery deny`. No other values.
- **Expected signal**: the concrete return contract — status code AND error shape. `403 Forbidden` is acceptable; "fails" is not. `404` is acceptable ONLY with the `Anti-discovery deny` type AND a justification in the same cell.
- **State assertion**: for every Deny / Read-only / Cross-scope / Referent row, the row MUST also state what side effect was prevented — row unchanged, no audit event written, no notification fired; Referent rows assert nothing was written in **either** scope. An authorization test that doesn't check the side effect didn't happen is a coin-flip dressed as a guarantee.

### What Counts as a Deny Test (strict)

| Counts as a deny test                                                                                                                                            | Does NOT count                                                                                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Returns explicit `403 Forbidden` / `{:error, :forbidden}` / framework's `NotAuthorizedError` AND asserts no side effect                                          | Returns empty list / nil — could be empty for many non-auth reasons                                                               |
| Read-only writes rejected with `422` + field-level error AND asserts the value didn't persist                                                                    | Returns `200 OK` and trusts the client to ignore the rejection                                                                    |
| Hidden field asserted absent from the JSON payload AND from the rendered DOM (UI surfaces)                                                                       | Hidden field asserted absent from one but present in the other                                                                    |
| Uses a real authenticated actor in the **denied** role to prove the policy fires                                                                                 | Stubs/mocks the policy, disables auth middleware, or signs in as a superuser "for setup convenience"                              |
| `404` only when explicitly typed `Anti-discovery deny` with a written justification                                                                              | `404` used generically as a deny — hides the real authz decision from logs and breaks audit/forensic traceability                 |
| Referent deny: authorized actor inside their own scope submits another scope's id in the payload; asserts explicit rejection AND nothing written in either scope | Relying on "the actor is permitted and the FK resolves" — both hold in the IDOR case; only the referenced record's owner is wrong |

### Referent Sweep Test (required companion to `F` rows)

`F` rows are enumerated by hand — by the same author who may have forgotten the validation in the first place. A hand-written matrix cannot be trusted to model a risk its author didn't think of. Whenever the feature's actions accept references to other scope-qualified records (`F > 0`), the Acceptance Criteria > Testing subsection MUST also require a **referent sweep test**: a single test that introspects the application's schema/resource definitions, enumerates every (action, foreign-key input) pair, and asserts each one declares an ownership/scope validation. It must fail for a newly added action that omits the check, without anyone editing the test.

The sweep proves nothing is **forgotten**; the `F` rows prove the check actually **rejects**. Neither is sufficient alone.

| Stack            | Enumerate (action, FK-input) pairs from                                     |
| ---------------- | --------------------------------------------------------------------------- |
| Ash / Elixir     | `Ash.Resource.Info.actions/1` + each action's accepted attributes/arguments |
| Rails            | `reflect_on_all_associations` + strong-params permit lists                  |
| Django           | model `_meta.get_fields()` + serializer/form field sets                     |
| Prisma / TypeORM | schema metadata + DTO validation decorators                                 |

### Test Plan as Acceptance Gate

The Permission Matrix Test Plan is the single artifact a reviewer (or `ac-checker`) uses to decide whether the TDD is safe to accept. The TDD is NOT acceptable for implementation if:

- The plan is missing whenever the Authorization section is non-empty.
- Row count is less than the formula's `N`.
- Any `❌` cell in Layer 1 lacks a matching `Deny` row.
- Any `PC-xx` rule lacks both a met and unmet row.
- Any `Hidden` or `R` field-role pair is missing.
- Any (action, foreign-key input) pair whose referenced record is scope-qualified lacks a `Referent deny` row.
- `F > 0` but the Acceptance Criteria > Testing subsection does not require the referent sweep test.
- Any Deny row omits the rejection signal or the state assertion.
- `404` is used outside an `Anti-discovery deny` row.
- The Acceptance Criteria > Testing subsection does not assert the count.

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
4. Authorization (Action Permissions, Data Permissions, Permission Conditions, Policy Rules)
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

**Authentication and authorization tests are non-negotiable requirements**. Every TDD must include tests that cover all three authorization layers:

```markdown
### Layer 1 Tests (Action Permissions)
- [ ] **Policy tests verify every cell in the action permissions matrix**
- [ ] **Policy tests validate each actor role against all actions and scopes**
- [ ] **Policy tests cover cross-scope denial (actor A cannot access actor B's resources)**
- [ ] **Policy tests cover referent denial (authorized actor submits another scope's id in a payload — rejected, nothing written in either scope)**
- [ ] **A referent sweep test enumerates every (action, FK-input) pair from the schema/resource definitions and asserts each declares an ownership/scope validation**

### Layer 2 Tests (Data Permissions)
- [ ] **Policy tests confirm hidden fields are excluded from responses for unauthorized roles**
- [ ] **Policy tests confirm read-only fields reject write attempts**
- [ ] **Policy tests verify field visibility per role matches data permissions table**

### Layer 3 Tests (Permission Conditions)
- [ ] **Policy tests verify each permission condition rule (PC-xx) independently**
- [ ] **Policy tests cover status-based, time-based, and relationship-based conditions**

### General
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

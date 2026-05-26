# TDD: [Feature Name]

| Document   | Link                                  |
| ---------- | ------------------------------------- |
| Parent PRD | [prd-name.md](../prds/prd-name.md)    |
| Master TDD | [00-domain-master.md](./00-master.md) |

| Domain        | Priority | Status   | Last Updated   |
| ------------- | -------- | -------- | -------------- |
| `Module.Name` | P0-P3    | Planning | [Month] [Year] |

______________________________________________________________________

## Overview

### Purpose

[2-3 sentences: What problem does this solve? Why is it needed?]

### Scope

| Included     | Out of Scope                    |
| ------------ | ------------------------------- |
| ✅ Feature A | ❌ Feature X (future)           |
| ✅ Feature B | ❌ Feature Y (different domain) |

### Key Business Rules

| Rule ID | Description              |
| ------- | ------------------------ |
| BR-01   | [Business rule from PRD] |
| BR-02   | [Another business rule]  |

______________________________________________________________________

## Data Model

### Attributes

| Attribute  | Type     | Required | Description                 |
| ---------- | -------- | -------- | --------------------------- |
| id         | UUID     | Yes      | Primary key                 |
| name       | string   | Yes      | Display name, max 255 chars |
| status     | enum     | Yes      | draft, active, archived     |
| created_at | datetime | Yes      | UTC timestamp               |
| updated_at | datetime | Yes      | UTC timestamp               |

### Relationships

| Relation | Target | Type       | Description    |
| -------- | ------ | ---------- | -------------- |
| owner    | User   | belongs_to | Record creator |
| items    | Item   | has_many   | Child records  |

### Constraints

- Unique constraint on `[owner_id, name]`
- Status transitions: draft -> active -> archived (no reverse)

______________________________________________________________________

## Interface Contract

### Actions

| Action  | Type   | Arguments         | Returns    | Description             |
| ------- | ------ | ----------------- | ---------- | ----------------------- |
| create  | create | params, actor     | Resource   | Creates with validation |
| list    | read   | query, actor      | [Resource] | Paginated with filters  |
| get     | read   | id, actor         | Resource   | Single record by ID     |
| update  | update | id, params, actor | Resource   | Updates allowed fields  |
| archive | update | id, actor         | Resource   | Soft delete             |

### Code Interface (signatures only)

```
Domain.create_feature(params, actor: user)
Domain.list_features(query: [...], actor: user)
Domain.get_feature!(id, actor: user)
Domain.update_feature(id, params, actor: user)
Domain.archive_feature(id, actor: user)
```

______________________________________________________________________

## Authorization

### Layer 1: Action Permissions

| Actor   | Create  | Read    | Update  | Archive | [Domain Action] |
| ------- | ------- | ------- | ------- | ------- | --------------- |
| Owner   | ✅ Own  | ✅ Own  | ✅ Own  | ✅ Own  | [scope]         |
| Manager | ✅ Team | ✅ Team | ✅ Team | ✅ Team | [scope]         |
| Admin   | ✅ All  | ✅ All  | ✅ All  | ✅ All  | ✅ All          |

### Layer 2: Data Permissions

| Field          | Owner  | Manager | Admin |
| -------------- | ------ | ------- | ----- |
| [public field] | RW     | RW      | RW    |
| [restricted]   | R      | RW      | RW    |
| [sensitive]    | Hidden | R       | RW    |

> **RW** = Read-Write, **R** = Read-only, **Hidden** = Not visible. Include only when roles see different fields.

### Layer 3: Permission Conditions

| Rule ID | Condition                | Effect     | Actors  |
| ------- | ------------------------ | ---------- | ------- |
| PC-01   | `status = 'draft'`       | Can update | Owner   |
| PC-02   | `team_id IN actor.teams` | Can read   | Manager |

### Policy Rules

- Owner can only modify records in `draft` status (PC-01)
- Manager sees all records in their team's scope (PC-02)
- Admin actions are logged to audit trail

______________________________________________________________________

## Permission Matrix Test Plan

> **Formula**: `N = A + S + H + R + 2·P + E` `A` = Layer 1 cells · `S` = scope-qualified ✅ cells (cross-scope denies) · `H` = Hidden field-role pairs · `R` = Read-only field-role pairs · `P` = PC-xx rules (×2 each) · `E` = protected entry points (401 boundary)
>
> **For this TDD**: `N = [A] + [S] + [H] + [R] + 2·[P] + [E] = [TOTAL] required policy tests`. Enumerate every row before this TDD can be accepted. Deny rows MUST specify the exact rejection signal AND the state assertion (what side effect was prevented).

| PT-ID | Layer | Cell / Rule           | Actor | Action / Field                 | Type                | Expected signal                               | State assertion                       |
| ----- | ----- | --------------------- | ----- | ------------------------------ | ------------------- | --------------------------------------------- | ------------------------------------- |
| PT-01 | 1     | Owner × create        | Owner | create(own)                    | Allow               | 201 Created                                   | Row persisted with `owner_id = actor` |
| PT-02 | 1     | Owner × approve       | Owner | approve                        | Deny                | 403 Forbidden                                 | No status change, no audit event      |
| PT-03 | 1-S   | Owner × update(other) | Owner | update on other Owner's record | Cross-scope deny    | 403 Forbidden                                 | Target row unchanged                  |
| PT-04 | 2-H   | Hidden field          | Guest | response.[sensitive]           | Hidden              | 200 OK, key absent from JSON AND DOM          | -                                     |
| PT-05 | 2-R   | Read-only field       | Owner | PATCH [restricted]             | Read-only deny      | 422 with field error `{field: ["read-only"]}` | Field value unchanged on the row      |
| PT-06 | 3     | PC-01 met             | Owner | update (status=draft)          | Condition met       | 200 OK                                        | Row updated                           |
| PT-07 | 3     | PC-01 not met         | Owner | update (status=active)         | Condition not met   | 403 Forbidden                                 | Row unchanged                         |
| PT-08 | E     | Unauthenticated GET   | none  | GET /features                  | Auth boundary       | 401 Unauthorized (NOT 403, NOT 404)           | -                                     |
| PT-09 | 1-AD  | Cross-org GET         | Owner | GET /features/:id              | Anti-discovery deny | 404 Not Found (justified: leaks existence)    | No record body returned               |

> Rows above are starters. Add one row per ❌ cell, per scope-qualified ✅ cell, per Hidden / R field-role pair, per PC-xx (×2), and per protected entry point until the count matches `N`. See [style-guide.md](../../../skills/tdd-writer/style-guide.md#permission-matrix-test-plan) for column rules and what counts as a deny test.

______________________________________________________________________

## UI Specification

### Routes

| Route                | Page Component | Description            |
| -------------------- | -------------- | ---------------------- |
| `/features`          | FeatureList    | List all user features |
| `/features/new`      | FeatureForm    | Create new feature     |
| `/features/:id`      | FeatureDetail  | View feature details   |
| `/features/:id/edit` | FeatureForm    | Edit existing feature  |

### Screens

#### Feature List

**Purpose**: Display paginated list of user's features

**Components**:

| Component    | Description                                |
| ------------ | ------------------------------------------ |
| Header       | Page title + "New Feature" button          |
| FilterBar    | Status filter, search input                |
| FeatureTable | Sortable columns: name, status, created_at |
| Pagination   | Page navigation with page size selector    |
| EmptyState   | Shown when no features exist               |

#### Feature Form (Create/Edit)

**Fields**:

| Field       | Type     | Validation        | Notes                |
| ----------- | -------- | ----------------- | -------------------- |
| name        | text     | Required, max 255 | Auto-focus on load   |
| description | textarea | Optional          | Markdown supported   |
| status      | select   | Required          | Options from backend |

#### Feature Detail

**Sections**:

| Section       | Content                            |
| ------------- | ---------------------------------- |
| Header        | Name, status badge, action buttons |
| Metadata      | Created, updated, owner            |
| Description   | Rendered markdown                  |
| Related Items | List of associated items           |

### UI Authorization

- Buttons and menu items for unauthorized actions must be hidden (not just disabled)
- Navigation to unauthorized routes redirects to appropriate fallback
- Field visibility adapts per role — sensitive fields never rendered for unauthorized roles

______________________________________________________________________

## Behavior Specifications

### Creating a Record

**Given**: User has `create` permission **When**: User submits valid form data **Then**:

- Record is created with `draft` status
- `created_at` is set to current UTC time
- Audit log entry is recorded
- Success notification displayed, navigate to detail view

### Archiving a Record

**Given**: User owns the record and record is in `active` status **When**: User requests archive **Then**:

- Status changes to `archived`
- Record is excluded from default list queries
- Associated items remain accessible

______________________________________________________________________

## Acceptance Criteria

### Core Functionality

- [ ] User can create a new record with required fields
- [ ] User can view their own records in a list
- [ ] User can update records they own in draft status
- [ ] User can archive records they own

### Validation

- [ ] Name field is required and max 255 characters
- [ ] Status defaults to "draft" on creation
- [ ] Invalid status transitions are rejected

### Authorization

- [ ] Unauthenticated requests return 401
- [ ] Users cannot access records they don't own
- [ ] Admins can access all records
- [ ] Each permission condition (PC-xx) enforced correctly
- [ ] Field visibility matches data permission matrix per role
- [ ] Domain-specific actions restricted to authorized roles
- [ ] Unauthorized UI actions are hidden (buttons not rendered)
- [ ] Sensitive fields never rendered in DOM for unauthorized roles

### UI

- [ ] List displays user's features with pagination
- [ ] Form shows validation errors inline
- [ ] Loading state shown during submission
- [ ] Browser back/forward works correctly
- [ ] Layout adapts to mobile viewport

### Testing

- [ ] Unit tests written for all business logic and validations
- [ ] Unit tests cover all constraint rules and status transitions
- [ ] **Permission Matrix Test Plan implemented in full: total policy test count ≥ `N = A + S + H + R + 2·P + E = [TOTAL]` (substitute the values used in the Permission Matrix Test Plan section)**
- [ ] **Every `PT-xx` row in the Permission Matrix Test Plan has a corresponding executable test, named or tagged with its `PT-xx` ID for traceability**
- [ ] **Every Deny / Cross-scope / Read-only / Condition-not-met test asserts the exact rejection signal declared in its PT-xx row (e.g. `403 Forbidden`, `{:error, :forbidden}`, framework-specific `NotAuthorizedError`) — NEVER passes on empty result, no-op, or generic 404**
- [ ] **Every Deny / Cross-scope / Read-only / Condition-not-met test asserts the prevented side effect: row unchanged, no audit event, no notification, no downstream job enqueued**
- [ ] **Every Hidden field-role pair test asserts the field key is absent from the JSON payload AND absent from the rendered DOM (not merely hidden via CSS)**
- [ ] **Every Auth-boundary entry-point test asserts `401 Unauthorized` (distinct from `403 Forbidden`) for unauthenticated requests**
- [ ] **Any `404` deny is justified inline as `Anti-discovery deny` and present only on rows where leaking existence is itself the threat**
- [ ] **Policy tests use real authenticated actors in the denied role — never stub the policy, disable auth middleware, or sign in as a superuser "for setup convenience"**
- [ ] **Authorization tests verify hidden UI actions are not rendered for unauthorized roles**
- [ ] Integration tests verify complete CRUD workflows with authorization
- [ ] Integration tests validate relationship cascades and foreign keys
- [ ] E2E tests cover complete user workflows (create, edit, delete)
- [ ] **End-to-end tests render each page/form/component via the framework's real render pipeline (e.g. `Phoenix.LiveViewTest`, Testing Library, Capybara, Playwright) — not just unit-level component mounts**
- [ ] **Every change / live-validation event path has a test, including each validation-error branch**
- [ ] **Every submit success path asserts redirect/toast AND the backend effect: persisted row attributes, job enqueued, or API call payload — NOT just the redirect**
- [ ] **Every submit failure branch has a test: missing field, invalid format, validation rejection, anti-discovery, rate limit, server error**
- [ ] **Failure-branch tests assert the rendered error message or field-level error — NEVER only a static string like page title or header (which passes even when the whole validation pipeline is broken)**
- [ ] **UI bug fixes include a regression test that fails on the buggy code and passes on the fix**
- [ ] **Accessibility tests validate WCAG AA compliance**
- [ ] Test coverage ≥ 80% for domain logic
- [ ] Test coverage ≥ 90% for critical business rules
- [ ] **Test coverage ≥ 95% for authentication and authorization logic**
- [ ] Performance tests validate query response times < 100ms
- [ ] Performance tests validate page load < 3s, interaction < 100ms
- [ ] UI tests cover user-facing changes

### Code Quality

- [ ] All new modules have type annotations and documentation
- [ ] No compiler/linter warnings
- [ ] Code formatted per project standards
- [ ] **Every third-party library/SDK/API/CLI/service referenced (backend or UI) appears in `External Dependencies` with version, doc URL, fetch date, and section consulted**
- [ ] **Docs for every third-party surface were fetched fresh via context7 (preferred) or WebFetch — not recalled from memory, even for well-known vendors**

______________________________________________________________________

## External Dependencies

> **Non-negotiable**: Every third-party library, framework, SDK, API, CLI, or cloud service this feature depends on — backend OR UI — MUST be listed here with the version, the canonical doc URL, the date the docs were fetched, and the exact section/method/endpoint consulted. Docs MUST be fetched fresh during TDD authoring — via context7 (`resolve-library-id` + `query-docs`) for libraries/SDKs/frameworks, WebFetch for vendor service docs and release notes — never written from training memory. A reviewer must be able to re-fetch the same source to verify every method name, endpoint, payload field, header, scope, error code, and quota stated in this TDD.

| Name        | Kind      | Version | Doc Source                                  | Section Consulted        | Fetched (YYYY-MM-DD) |
| ----------- | --------- | ------- | ------------------------------------------- | ------------------------ | -------------------- |
| react       | Framework | v19.0.0 | context7: `/facebook/react`                 | useActionState, Suspense | 2026-05-26           |
| stripe-node | SDK       | v17.4.0 | https://docs.stripe.com/api/payment_intents | PaymentIntent.create     | 2026-05-26           |
| ...         | ...       | ...     | ...                                         | ...                      | ...                  |

> Omit this section only if the feature has zero third-party surfaces. Docs older than 90 days at implementation kickoff MUST be re-fetched and any drift reconciled before merge.

______________________________________________________________________

## Open Questions

- **OQ-01**: [Question about requirement] - Status: Open
- **OQ-02**: [Clarification needed] - Status: Resolved

______________________________________________________________________

## Related Documents

| Document     | Link                                   |
| ------------ | -------------------------------------- |
| Parent PRD   | [prd-name.md](../prds/prd-name.md)     |
| Master TDD   | [00-master.md](./00-master.md)         |
| Dependencies | [other-feature.md](./other-feature.md) |

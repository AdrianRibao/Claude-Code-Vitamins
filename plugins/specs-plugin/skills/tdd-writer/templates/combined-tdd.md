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
- [ ] **Policy tests verify every cell in the action permissions matrix**
- [ ] **Policy tests validate each actor role against all actions and scopes**
- [ ] **Policy tests cover authentication failures and unauthorized access attempts**
- [ ] **Policy tests verify each permission condition rule (PC-xx) independently**
- [ ] **Policy tests confirm data permissions: hidden fields excluded, read-only fields rejected on write**
- [ ] **Policy tests cover scope boundaries (own vs team vs all) with cross-scope denial**
- [ ] **Authorization tests verify hidden actions are not rendered for unauthorized roles**
- [ ] **Authorization tests confirm sensitive fields absent from DOM (not just hidden via CSS)**
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

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
- Status transitions: draft → active → archived (no reverse)

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

```elixir
Domain.create_feature(params, actor: user)
Domain.list_features(query: [...], actor: user)
Domain.get_feature!(id, actor: user)
Domain.update_feature(id, params, actor: user)
Domain.archive_feature(id, actor: user)
```

______________________________________________________________________

## Authorization

### Policy Matrix

| Actor   | Create  | Read    | Update  | Delete |
| ------- | ------- | ------- | ------- | ------ |
| Owner   | ✅ Own  | ✅ Own  | ✅ Own  | ✅ Own |
| Manager | ✅ Team | ✅ Team | ✅ Team | ❌     |
| Admin   | ✅ All  | ✅ All  | ✅ All  | ✅ All |

### Policy Rules

- Owner can only modify records in `draft` status
- Manager sees all records in their team's scope
- Admin actions are logged to audit trail

______________________________________________________________________

## Behavior Specifications

### Creating a Record

**Given**: User has `create` permission **When**: User submits valid form data **Then**:

- Record is created with `draft` status
- `created_at` is set to current UTC time
- Audit log entry is recorded

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

### Testing

- [ ] Unit tests written for all business logic and validations
- [ ] Unit tests cover all constraint rules and status transitions
- [ ] **Policy tests verify all authorization rules and permissions**
- [ ] **Policy tests validate each actor role against policy matrix**
- [ ] **Policy tests cover authentication failures and unauthorized access attempts**
- [ ] Integration tests verify complete CRUD workflows with authorization
- [ ] Integration tests validate relationship cascades and foreign keys
- [ ] Edge case tests for concurrent updates and data race conditions
- [ ] Test coverage ≥ 80% for domain logic
- [ ] Test coverage ≥ 90% for critical business rules
- [ ] **Test coverage ≥ 95% for authentication and authorization logic**
- [ ] Performance tests validate query response times < 100ms
- [ ] Load tests verify system handles expected concurrent operations

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
| UI TDD       | [feature-ui.md](./feature-ui.md)       |
| Dependencies | [other-feature.md](./other-feature.md) |

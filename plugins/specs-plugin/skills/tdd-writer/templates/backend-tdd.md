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

### Layer 1: Action Permissions

| Actor   | Create  | Read    | Update  | Archive | [Domain Action] |
| ------- | ------- | ------- | ------- | ------- | --------------- |
| Owner   | ✅ Own  | ✅ Own  | ✅ Own  | ✅ Own  | [scope]         |
| Manager | ✅ Team | ✅ Team | ✅ Team | ✅ Team | [scope]         |
| Admin   | ✅ All  | ✅ All  | ✅ All  | ✅ All  | ✅ All          |

> Replace `[Domain Action]` with feature-specific actions (approve, publish, escalate, export, etc.)

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

> **Formula**: `N = A + S + F + H + R + 2·P + E` `A` = Layer 1 cells · `S` = scope-qualified ✅ cells (cross-scope denies) · `F` = referent (action, FK-input) pairs on scope-qualified references (referent denies) · `H` = Hidden field-role pairs · `R` = Read-only field-role pairs · `P` = PC-xx rules (×2 each) · `E` = protected entry points (401 boundary)
>
> **For this TDD**: `N = [A] + [S] + [F] + [H] + [R] + 2·[P] + [E] = [TOTAL] required policy tests`. Every Deny row MUST specify the exact rejection signal AND the state assertion (what side effect was prevented). See [style-guide.md](../../../skills/tdd-writer/style-guide.md#permission-matrix-test-plan).

| PT-ID | Layer | Cell / Rule           | Actor | Action / Field                       | Type              | Expected signal                               | State assertion                                 |
| ----- | ----- | --------------------- | ----- | ------------------------------------ | ----------------- | --------------------------------------------- | ----------------------------------------------- |
| PT-01 | 1     | Owner × create        | Owner | create(own)                          | Allow             | `{:ok, record}`                               | Row persisted with `owner_id = actor`           |
| PT-02 | 1     | Owner × [domain-act]  | Owner | [domain action]                      | Deny              | `{:error, :forbidden}`                        | No state change, no audit event                 |
| PT-03 | 1-S   | Owner × update(other) | Owner | update on other Owner's record       | Cross-scope deny  | `{:error, :forbidden}`                        | Target row unchanged                            |
| PT-04 | 2-H   | Hidden field          | Owner | response.[sensitive]                 | Hidden            | Resource map omits the key                    | -                                               |
| PT-05 | 2-R   | Read-only field       | Owner | update with [restricted]             | Read-only deny    | Changeset error `{field: ["cannot change"]}`  | Field value unchanged                           |
| PT-06 | 3     | PC-01 met             | Owner | update (status=draft)                | Condition met     | `{:ok, record}`                               | Row updated                                     |
| PT-07 | 3     | PC-01 not met         | Owner | update (status=active)               | Condition not met | `{:error, :forbidden}`                        | Row unchanged                                   |
| PT-08 | E     | Unauthenticated call  | none  | any Domain action                    | Auth boundary     | `{:error, :unauthenticated}` (NOT :forbidden) | -                                               |
| PT-09 | 1-F   | `create.[fk_input]`   | Owner | create naming another scope's record | Referent deny     | `{:error, :forbidden}` or validation error    | No row written in either scope; no link created |

> Add one row per ❌ cell, per scope-qualified ✅, per referent (action, FK-input) pair, per Hidden / R field-role pair, per PC-xx (×2), and per protected entry point until total equals `N`. When `F > 0`, also require the referent sweep test (see style-guide).

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
- [ ] Each permission condition (PC-xx) enforced correctly
- [ ] Field visibility matches data permission matrix per role
- [ ] Domain-specific actions (approve, etc.) restricted to authorized roles

### Testing

- [ ] Unit tests written for all business logic and validations
- [ ] Unit tests cover all constraint rules and status transitions
- [ ] **Permission Matrix Test Plan implemented in full: total policy test count ≥ `N = A + S + F + H + R + 2·P + E = [TOTAL]` (substitute values from the Permission Matrix Test Plan section)**
- [ ] **Every `PT-xx` row has a corresponding executable test, named or tagged with its `PT-xx` ID for traceability**
- [ ] **Every Deny / Cross-scope / Referent / Read-only / Condition-not-met test asserts the exact rejection signal in its PT-xx row (e.g. `{:error, :forbidden}`, framework-specific `NotAuthorizedError`) — NEVER passes on empty result, no-op, or generic 404**
- [ ] **Every Deny / Cross-scope / Referent / Read-only / Condition-not-met test asserts the prevented side effect: row unchanged, no audit event, no notification, no downstream job enqueued**
- [ ] **Referent sweep test present when `F > 0`: introspects schema/resource definitions, enumerates every (action, FK-input) pair, and asserts each declares an ownership/scope validation — fails for a newly added action that omits the check**
- [ ] **Every Hidden field-role pair test asserts the field key is absent from the Resource map / serialized payload**
- [ ] **Every Auth-boundary entry-point test asserts `{:error, :unauthenticated}` (distinct from `:forbidden`)**
- [ ] **Any `404` deny is justified inline as `Anti-discovery deny` and present only when leaking existence is itself the threat**
- [ ] **Policy tests use real authenticated actors in the denied role — never stub the policy, disable auth, or sign in as a superuser "for setup convenience"**
- [ ] Integration tests verify complete CRUD workflows with authorization
- [ ] Integration tests validate relationship cascades and foreign keys
- [ ] Edge case tests for concurrent updates and data race conditions
- [ ] Test coverage ≥ 80% for domain logic
- [ ] Test coverage ≥ 90% for critical business rules
- [ ] **Test coverage ≥ 95% for authentication and authorization logic**
- [ ] Performance tests validate query response times < 100ms
- [ ] Load tests verify system handles expected concurrent operations
- [ ] UI tests cover user-facing changes (if applicable)

### Code Quality

- [ ] All new modules have type annotations and documentation
- [ ] No compiler/linter warnings
- [ ] Code formatted per project standards
- [ ] **Every third-party library/SDK/CLI surface used by this backend appears in `External Dependencies` with version, doc URL, fetch date, and section consulted**
- [ ] **Docs for every third-party surface were fetched fresh via context7 (preferred) or WebFetch — not recalled from memory**

______________________________________________________________________

## External Dependencies

> **Non-negotiable**: Every third-party library, SDK, CLI, or cloud service this backend depends on MUST be listed here with the version, the canonical doc URL, the date the docs were fetched, and the exact section/method consulted. Docs MUST be fetched fresh during TDD authoring — via context7 (`resolve-library-id` + `query-docs`) for libraries/SDKs, WebFetch for vendor service docs and release notes — never written from training memory. A reviewer must be able to re-fetch the same source to verify every method name, payload field, header, scope, error code, and quota stated in this TDD.

| Name      | Kind    | Version | Doc Source                    | Section Consulted         | Fetched (YYYY-MM-DD) |
| --------- | ------- | ------- | ----------------------------- | ------------------------- | -------------------- |
| oban      | Library | v2.18.0 | context7: `/sorentwo/oban`    | Worker, Pro plugins       | 2026-05-26           |
| ex_aws_s3 | SDK     | v2.5.3  | context7: `/ex-aws/ex_aws_s3` | put_object, presigned_url | 2026-05-26           |
| ...       | ...     | ...     | ...                           | ...                       | ...                  |

> Omit this section only if the feature has zero third-party surfaces. Docs older than 90 days at implementation kickoff MUST be re-fetched and any drift reconciled before merge.

______________________________________________________________________

## ✅ Decisions (Resolved)

<!-- Judgment calls made while writing this TDD (architecture, naming, limits, least-privilege authorization fills). Fold each into its section; log it here. -->

| Decision               | Choice   | Rationale                          |
| ---------------------- | -------- | ---------------------------------- |
| [Decision name] (D-01) | [Choice] | [Why, with the concrete trade-off] |

______________________________________________________________________

## Open Questions

<!-- Only questions a human must settle (business policy, compliance values, budget, external contracts, product-owner forks).
     Each must be self-contained (acronyms spelled out, 1-line context, concrete trade-offs, current implicit default, recommended option).
     If there are none, this section contains exactly:
     *No open questions — every design decision is recorded in ✅ Decisions (Resolved).* -->

- **OQ-01**: [Self-contained question] - Recommended: [option] - Status: Open

______________________________________________________________________

## Related Documents

| Document     | Link                                   |
| ------------ | -------------------------------------- |
| Parent PRD   | [prd-name.md](../prds/prd-name.md)     |
| Master TDD   | [00-master.md](./00-master.md)         |
| UI TDD       | [feature-ui.md](./feature-ui.md)       |
| Dependencies | [other-feature.md](./other-feature.md) |

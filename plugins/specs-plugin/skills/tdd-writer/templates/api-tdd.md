# TDD: [Feature Name] - API

| Document   | Link                                  |
| ---------- | ------------------------------------- |
| Parent PRD | [prd-name.md](../prds/prd-name.md)    |
| Master TDD | [00-domain-master.md](./00-master.md) |

| Domain        | Priority | Status   | Last Updated   |
| ------------- | -------- | -------- | -------------- |
| `API.Feature` | P0-P3    | Planning | [Month] [Year] |

______________________________________________________________________

## Overview

### Purpose

[2-3 sentences: What does this API enable? Who consumes it?]

### Scope

| Included            | Out of Scope               |
| ------------------- | -------------------------- |
| ✅ CRUD operations  | ❌ Bulk operations         |
| ✅ Webhook delivery | ❌ Real-time subscriptions |
| ✅ Pagination       | ❌ GraphQL interface       |

______________________________________________________________________

## Endpoints

### List Features

| Method | Path               | Description                   |
| ------ | ------------------ | ----------------------------- |
| GET    | `/api/v1/features` | List features with pagination |

**Query Parameters**:

| Parameter | Type    | Required | Description                         |
| --------- | ------- | -------- | ----------------------------------- |
| page      | integer | No       | Page number, default 1              |
| per_page  | integer | No       | Items per page, default 20, max 100 |
| status    | string  | No       | Filter by status                    |
| sort      | string  | No       | Sort field, default "created_at"    |
| order     | string  | No       | "asc" or "desc", default "desc"     |

**Response**: `200 OK`

```json
{
  "data": [{ "id": "...", "name": "...", "status": "..." }],
  "meta": { "page": 1, "per_page": 20, "total": 100 }
}
```

### Get Feature

| Method | Path                   | Description        |
| ------ | ---------------------- | ------------------ |
| GET    | `/api/v1/features/:id` | Get single feature |

**Response**: `200 OK`

```json
{
  "data": { "id": "...", "name": "...", "status": "...", "created_at": "..." }
}
```

### Create Feature

| Method | Path               | Description        |
| ------ | ------------------ | ------------------ |
| POST   | `/api/v1/features` | Create new feature |

**Request Body**:

| Field       | Type   | Required | Description                 |
| ----------- | ------ | -------- | --------------------------- |
| name        | string | Yes      | Feature name, max 255 chars |
| description | string | No       | Optional description        |

**Response**: `201 Created`

```json
{
  "data": { "id": "...", "name": "...", "status": "draft" }
}
```

### Update Feature

| Method | Path                   | Description    |
| ------ | ---------------------- | -------------- |
| PATCH  | `/api/v1/features/:id` | Update feature |

**Request Body**:

| Field       | Type   | Required | Description         |
| ----------- | ------ | -------- | ------------------- |
| name        | string | No       | Updated name        |
| description | string | No       | Updated description |
| status      | string | No       | New status          |

**Response**: `200 OK`

### Delete Feature

| Method | Path                   | Description    |
| ------ | ---------------------- | -------------- |
| DELETE | `/api/v1/features/:id` | Delete feature |

**Response**: `204 No Content`

______________________________________________________________________

## Authentication

### Method

Bearer token in Authorization header:

```
Authorization: Bearer <access_token>
```

### Token Requirements

| Requirement | Description                |
| ----------- | -------------------------- |
| Type        | JWT access token           |
| Expiry      | 1 hour                     |
| Refresh     | Via `/api/v1/auth/refresh` |

______________________________________________________________________

## Authorization

### Layer 1: Action Permissions (Endpoint Level)

| Endpoint             | Required Scope    | Allowed Roles | Scope    |
| -------------------- | ----------------- | ------------- | -------- |
| GET /features        | `features:read`   | Owner, Admin  | Own, All |
| POST /features       | `features:write`  | Owner         | Own      |
| PATCH /features/:id  | `features:write`  | Owner, Admin  | Own, All |
| DELETE /features/:id | `features:delete` | Admin         | All      |

### Layer 2: Data Permissions (Response Filtering)

| Field             | Owner  | Admin | Public |
| ----------------- | ------ | ----- | ------ |
| [public field]    | RW     | RW    | R      |
| [internal field]  | R      | RW    | Hidden |
| [sensitive field] | Hidden | R     | Hidden |

> API responses must filter fields based on caller role. Write attempts to read-only fields return 422.

### Layer 3: Permission Conditions

| Rule ID | Condition             | Effect          | Endpoints       |
| ------- | --------------------- | --------------- | --------------- |
| PC-01   | `status = 'draft'`    | Can update      | PATCH /features |
| PC-02   | `owner_id = actor.id` | Can access      | GET, PATCH      |
| PC-03   | Rate limit: 100/hour  | Throttle writes | POST /features  |

### Policy Rules

- Users can only access their own resources (PC-02)
- Only draft resources can be updated (PC-01)
- Write operations are rate limited per user (PC-03)

______________________________________________________________________

## Permission Matrix Test Plan

> **Formula**: `N = A + S + H + R + 2·P + E` `A` = Layer 1 endpoint × role cells · `S` = scope-qualified ✅ cells (cross-scope denies) · `H` = Hidden response-field role pairs · `R` = read-only request-field role pairs · `P` = PC-xx rules (×2 each) · `E` = protected endpoints (401 boundary)
>
> **For this TDD**: `N = [A] + [S] + [H] + [R] + 2·[P] + [E] = [TOTAL] required policy tests`. Every Deny row MUST specify the HTTP status AND error body shape AND the state assertion. See [style-guide.md](../../../skills/tdd-writer/style-guide.md#permission-matrix-test-plan).

| PT-ID | Layer | Cell / Rule           | Actor   | Action / Field            | Type                | Expected signal                                      | State assertion                       |
| ----- | ----- | --------------------- | ------- | ------------------------- | ------------------- | ---------------------------------------------------- | ------------------------------------- |
| PT-01 | 1     | Owner × POST          | Owner   | POST /features            | Allow               | 201 Created, body returns resource                   | Row persisted with `owner_id = actor` |
| PT-02 | 1     | Owner × DELETE        | Owner   | DELETE /features/:id      | Deny                | 403 `{code: FORBIDDEN}`                              | Row not deleted                       |
| PT-03 | 1-S   | Owner × GET(other)    | Owner   | GET /features/:id (other) | Cross-scope deny    | 403 `{code: FORBIDDEN}`                              | Row not returned                      |
| PT-04 | 2-H   | Hidden response field | Public  | response.[sensitive]      | Hidden              | 200 OK, key absent from JSON                         | -                                     |
| PT-05 | 2-R   | Read-only field       | Owner   | PATCH [internal field]    | Read-only deny      | 422 `{code: UNPROCESSABLE, details: [...]}`          | Field unchanged on persisted row      |
| PT-06 | 3     | PC-01 met             | Owner   | PATCH (status=draft)      | Condition met       | 200 OK                                               | Row updated                           |
| PT-07 | 3     | PC-01 not met         | Owner   | PATCH (status=active)     | Condition not met   | 403 `{code: FORBIDDEN}`                              | Row unchanged                         |
| PT-08 | E     | Missing token         | none    | GET /features             | Auth boundary       | 401 `{code: UNAUTHORIZED}` (NOT 403, NOT 404)        | -                                     |
| PT-09 | E     | Expired token         | expired | GET /features             | Auth boundary       | 401 `{code: UNAUTHORIZED}`                           | -                                     |
| PT-10 | 1-AD  | Cross-org GET         | Owner   | GET /features/:id         | Anti-discovery deny | 404 `{code: NOT_FOUND}` (justified: leaks existence) | No info leaked in body                |

> Add one row per ❌ endpoint × role cell, per scope-qualified ✅, per Hidden / R field-role pair, per PC-xx (×2), and per protected endpoint until total equals `N`.

______________________________________________________________________

## Error Responses

### Error Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human readable message",
    "details": [
      { "field": "name", "message": "is required" }
    ]
  }
}
```

### Error Codes

| HTTP Status | Code             | Description              |
| ----------- | ---------------- | ------------------------ |
| 400         | VALIDATION_ERROR | Invalid request body     |
| 401         | UNAUTHORIZED     | Missing or invalid token |
| 403         | FORBIDDEN        | Insufficient permissions |
| 404         | NOT_FOUND        | Resource does not exist  |
| 409         | CONFLICT         | Resource already exists  |
| 422         | UNPROCESSABLE    | Business rule violation  |
| 429         | RATE_LIMITED     | Too many requests        |
| 500         | INTERNAL_ERROR   | Server error             |

______________________________________________________________________

## Rate Limiting

| Scope                    | Limit         | Window |
| ------------------------ | ------------- | ------ |
| Per user                 | 1000 requests | 1 hour |
| Per IP (unauthenticated) | 100 requests  | 1 hour |
| Create operations        | 100 requests  | 1 hour |

**Headers**:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1609459200
```

______________________________________________________________________

## Webhooks

### Events

| Event           | Trigger             | Payload             |
| --------------- | ------------------- | ------------------- |
| feature.created | New feature created | Full feature object |
| feature.updated | Feature modified    | Full feature object |
| feature.deleted | Feature removed     | { id: "..." }       |

### Delivery

| Setting      | Value                      |
| ------------ | -------------------------- |
| Method       | POST                       |
| Content-Type | application/json           |
| Timeout      | 30 seconds                 |
| Retries      | 3 with exponential backoff |

### Signature Verification

```
X-Webhook-Signature: sha256=<hmac_signature>
```

______________________________________________________________________

## Behavior Specifications

### Pagination Edge Cases

**Given**: User requests page beyond available data **When**: GET /features?page=999 **Then**: Return empty data array with correct meta

### Concurrent Updates

**Given**: Two users update same feature simultaneously **When**: Both PATCH requests arrive **Then**: Last write wins, return updated resource

______________________________________________________________________

## Acceptance Criteria

### Authentication

- [ ] Requests without token return 401
- [ ] Expired tokens return 401
- [ ] Invalid tokens return 401

### Authorization

- [ ] Users can only access own resources (PC-02)
- [ ] Admin can access all resources
- [ ] Insufficient scope returns 403
- [ ] Each permission condition (PC-xx) enforced at endpoint level
- [ ] Response payloads filtered per data permissions (hidden fields excluded)
- [ ] Write attempts to read-only fields return 422

### Validation

- [ ] Missing required fields return 400
- [ ] Invalid field types return 400
- [ ] Errors include field-level details

### Rate Limiting

- [ ] Exceeded limit returns 429
- [ ] Rate limit headers present on all responses
- [ ] Limits reset after window expires

### Testing

- [ ] Unit tests for all endpoint handlers and request validation
- [ ] Unit tests validate error response formatting and codes
- [ ] **API contract tests verify request/response schemas match specification**
- [ ] **Permission Matrix Test Plan implemented in full: total policy test count ≥ `N = A + S + H + R + 2·P + E = [TOTAL]` (substitute values from the Permission Matrix Test Plan section)**
- [ ] **Every `PT-xx` row has a corresponding executable test, named or tagged with its `PT-xx` ID for traceability**
- [ ] **Every Deny / Cross-scope / Read-only / Condition-not-met test asserts the exact HTTP status AND error body shape declared in its PT-xx row — NEVER passes on empty array, 200 OK, or generic 404**
- [ ] **Every Deny / Cross-scope / Read-only / Condition-not-met test asserts the prevented side effect: no row created, no row mutated, no audit event, no webhook fired**
- [ ] **Every Hidden field-role pair test asserts the field key is absent from the JSON response payload**
- [ ] **Every Auth-boundary test asserts `401 UNAUTHORIZED` (distinct from `403 FORBIDDEN`) — `401` for missing/invalid/expired token, `403` only when authenticated but unauthorized**
- [ ] **Any `404 NOT_FOUND` deny is justified inline as `Anti-discovery deny` and present only when leaking existence is itself the threat**
- [ ] **Authorization tests use real authenticated callers in the denied role — never stub the policy, bypass middleware, or use admin credentials "for setup convenience"**
- [ ] Integration tests cover complete API workflows with real backend
- [ ] Integration tests validate webhook delivery and retry logic
- [ ] Rate limiting tests verify limits enforced correctly per scope
- [ ] Security tests validate input sanitization and injection prevention
- [ ] Load tests verify API handles expected concurrent requests
- [ ] Test coverage ≥ 85% for API endpoints
- [ ] Test coverage ≥ 95% for authentication and authorization middleware
- [ ] Performance tests validate response times meet SLA (< 200ms)
- [ ] OpenAPI/Swagger spec validation against implementation
- [ ] UI tests cover API consumer interfaces (if applicable)

### Code Quality

- [ ] All new modules have type annotations and documentation
- [ ] No compiler/linter warnings
- [ ] Code formatted per project standards
- [ ] **Every third-party library/SDK/CLI surface used by this API appears in `External Dependencies` with version, doc URL, fetch date, and section consulted**
- [ ] **Docs for every third-party surface were fetched fresh via context7 (preferred) or WebFetch — not recalled from memory**

______________________________________________________________________

## External Dependencies

> **Non-negotiable**: Every third-party library, SDK, CLI, or cloud service this API depends on MUST be listed here with the version, the canonical doc URL, the date the docs were fetched, and the exact section/method/endpoint consulted. Docs MUST be fetched fresh during TDD authoring — via context7 (`resolve-library-id` + `query-docs`) for libraries/SDKs, WebFetch for vendor service docs and release notes — never written from training memory. A reviewer must be able to re-fetch the same source to verify every method name, endpoint, payload field, header, scope, error code, and quota stated in this TDD.

| Name     | Kind      | Version  | Doc Source                     | Section Consulted             | Fetched (YYYY-MM-DD) |
| -------- | --------- | -------- | ------------------------------ | ----------------------------- | -------------------- |
| fastapi  | Framework | v0.118.0 | context7: `/tiangolo/fastapi`  | Dependencies, BackgroundTasks | 2026-05-26           |
| pydantic | Library   | v2.10.4  | context7: `/pydantic/pydantic` | Validators, model_serializer  | 2026-05-26           |
| ...      | ...       | ...      | ...                            | ...                           | ...                  |

> Omit this section only if the API has zero third-party surfaces. Docs older than 90 days at implementation kickoff MUST be re-fetched and any drift reconciled before merge.

______________________________________________________________________

## Related Documents

| Document     | Link                                       |
| ------------ | ------------------------------------------ |
| Parent PRD   | [prd-name.md](../prds/prd-name.md)         |
| Backend TDD  | [feature-backend.md](./feature-backend.md) |
| OpenAPI Spec | [openapi.yaml](./openapi.yaml)             |

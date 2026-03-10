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
- [ ] **Authentication tests validate token validation and expiry handling**
- [ ] **Authorization tests verify endpoint permissions and scope enforcement**
- [ ] **Authorization tests verify every permission condition (PC-xx) independently**
- [ ] **Authorization tests confirm data permissions: hidden fields excluded per role, read-only fields reject writes**
- [ ] **Authorization tests cover cross-scope denial (user A cannot access user B's resources)**
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

______________________________________________________________________

## Related Documents

| Document     | Link                                       |
| ------------ | ------------------------------------------ |
| Parent PRD   | [prd-name.md](../prds/prd-name.md)         |
| Backend TDD  | [feature-backend.md](./feature-backend.md) |
| OpenAPI Spec | [openapi.yaml](./openapi.yaml)             |

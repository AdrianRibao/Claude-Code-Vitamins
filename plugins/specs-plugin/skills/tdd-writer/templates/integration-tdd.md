# TDD: [Feature Name] - Integration

| Document   | Link                                  |
| ---------- | ------------------------------------- |
| Parent PRD | [prd-name.md](../prds/prd-name.md)    |
| Master TDD | [00-domain-master.md](./00-master.md) |

| Domain                | Priority | Status   | Last Updated   |
| --------------------- | -------- | -------- | -------------- |
| `Integration.Feature` | P0-P3    | Planning | [Month] [Year] |

______________________________________________________________________

## Overview

### Purpose

[2-3 sentences: What systems does this integrate? What problem does the integration solve?]

### Scope

| Included                    | Out of Scope              |
| --------------------------- | ------------------------- |
| ✅ External API integration | ❌ Batch synchronization  |
| ✅ Event-driven updates     | ❌ Historical data import |
| ✅ Error handling/retry     | ❌ Real-time streaming    |

______________________________________________________________________

## Systems Involved

| System           | Role      | Protocol    | Description           |
| ---------------- | --------- | ----------- | --------------------- |
| Our Backend      | Source    | HTTP/Events | Initiates integration |
| External Service | Target    | REST API    | Receives data         |
| Message Queue    | Transport | AMQP        | Async communication   |

______________________________________________________________________

## External Dependencies

> **Non-negotiable**: Every third-party library, SDK, API, CLI, or cloud service this TDD touches MUST be listed here with the version, the canonical doc URL, the date the docs were fetched, and the exact section/method/endpoint consulted. Docs MUST be fetched fresh during TDD authoring — via context7 (`resolve-library-id` + `query-docs`) for libraries/SDKs, WebFetch for vendor service docs and release notes — never written from training memory. A reviewer must be able to re-fetch the same source to verify every method name, endpoint, payload field, header, scope, error code, and quota stated in this TDD.

| Name              | Kind    | Version        | Doc Source                                           | Section Consulted               | Fetched (YYYY-MM-DD) |
| ----------------- | ------- | -------------- | ---------------------------------------------------- | ------------------------------- | -------------------- |
| stripe-node       | SDK     | v17.4.0        | https://docs.stripe.com/api/payment_intents (via WF) | PaymentIntent.create, confirm   | 2026-05-26           |
| Stripe Webhooks   | Service | API 2025-09-30 | https://docs.stripe.com/webhooks                     | Signature verification, retries | 2026-05-26           |
| aws-sdk-s3 (Node) | SDK     | v3.620.0       | context7: `/aws/aws-sdk-js-v3`                       | PutObjectCommand, presigned URL | 2026-05-26           |
| ...               | ...     | ...            | ...                                                  | ...                             | ...                  |

**Freshness window**: docs older than 90 days at implementation kickoff MUST be re-fetched before coding begins; bump the `Fetched` column and reconcile any drift in the TDD before merging.

______________________________________________________________________

## Data Flow

### Outbound Flow (Our System → External)

```
[Our Backend] → [Event Bus] → [Integration Worker] → [External API]
                                      ↓
                              [Retry Queue] ← [Failed]
```

### Inbound Flow (External → Our System)

```
[External Webhook] → [Webhook Handler] → [Validation] → [Our Backend]
                                              ↓
                                    [Dead Letter Queue] ← [Invalid]
```

______________________________________________________________________

## Events

### Published Events

| Event           | Trigger          | Payload              | Consumers        |
| --------------- | ---------------- | -------------------- | ---------------- |
| feature.created | Feature created  | { id, name, status } | External Service |
| feature.updated | Feature modified | { id, changes }      | External Service |
| feature.deleted | Feature removed  | { id }               | External Service |

### Consumed Events

| Event           | Source           | Payload                 | Handler      |
| --------------- | ---------------- | ----------------------- | ------------ |
| external.synced | External Service | { external_id, status } | SyncHandler  |
| external.failed | External Service | { external_id, error }  | ErrorHandler |

______________________________________________________________________

## Authorization

### Layer 1: Action Permissions (Integration Operations)

| Actor/System       | Publish Events | Consume Events | Trigger Sync | Replay DLQ |
| ------------------ | -------------- | -------------- | ------------ | ---------- |
| Our Backend        | ✅             | ✅             | ✅           | ❌         |
| Integration Worker | ❌             | ✅             | ✅           | ❌         |
| Admin              | ❌             | ❌             | ✅ Manual    | ✅         |
| External Service   | ❌             | ❌             | ❌           | ❌         |

### Layer 2: Data Permissions (What Data Crosses Boundaries)

| Field             | Outbound (sent) | Inbound (received) | Internal Only |
| ----------------- | --------------- | ------------------ | ------------- |
| [public field]    | ✅ Sent         | ✅ Accepted        | -             |
| [internal field]  | ❌ Excluded     | ❌ Ignored         | ✅            |
| [sensitive field] | ❌ Never sent   | ❌ Rejected        | ✅            |

> Sensitive data must never cross system boundaries. PII requires explicit data processing agreements.

### Layer 3: Permission Conditions

| Rule ID | Condition               | Effect                | Systems            |
| ------- | ----------------------- | --------------------- | ------------------ |
| PC-01   | API key valid           | Allow outbound calls  | Integration Worker |
| PC-02   | Webhook signature valid | Accept inbound events | Webhook Handler    |
| PC-03   | Rate limit not exceeded | Allow sync operations | Integration Worker |

______________________________________________________________________

## External API Contract

### Authentication

| Setting  | Value             |
| -------- | ----------------- |
| Method   | API Key in header |
| Header   | X-API-Key         |
| Rotation | Every 90 days     |

### Endpoints Used

| Endpoint              | Method | Purpose           |
| --------------------- | ------ | ----------------- |
| /resources            | POST   | Create resource   |
| /resources/:id        | PATCH  | Update resource   |
| /resources/:id        | DELETE | Remove resource   |
| /resources/:id/status | GET    | Check sync status |

### Request Format

```json
{
  "external_id": "our-feature-id",
  "data": {
    "name": "Feature Name",
    "metadata": { ... }
  }
}
```

### Response Format

```json
{
  "id": "external-resource-id",
  "status": "synced",
  "synced_at": "2025-01-01T00:00:00Z"
}
```

______________________________________________________________________

## Error Handling

### Retry Strategy

| Error Type       | Retry | Max Attempts | Backoff           |
| ---------------- | ----- | ------------ | ----------------- |
| Network timeout  | Yes   | 3            | Exponential       |
| 5xx errors       | Yes   | 3            | Exponential       |
| 429 rate limit   | Yes   | 5            | Rate limit header |
| 4xx client error | No    | 0            | N/A               |

### Dead Letter Queue

| Condition            | Action              |
| -------------------- | ------------------- |
| Max retries exceeded | Move to DLQ         |
| Invalid payload      | Move to DLQ         |
| Auth failure         | Alert, stop retries |

### Alert Conditions

| Condition             | Severity | Action             |
| --------------------- | -------- | ------------------ |
| 5+ failures in 5 min  | Warning  | Slack notification |
| 20+ failures in 5 min | Critical | PagerDuty alert    |
| Auth token expired    | Critical | PagerDuty alert    |

______________________________________________________________________

## Data Mapping

### Outbound Mapping (Our → External)

| Our Field  | External Field    | Transform       |
| ---------- | ----------------- | --------------- |
| id         | external_id       | Direct          |
| name       | title             | Direct          |
| status     | state             | Enum map        |
| created_at | created_timestamp | ISO 8601 → Unix |

### Inbound Mapping (External → Our)

| External Field | Our Field   | Transform          |
| -------------- | ----------- | ------------------ |
| id             | external_id | Store as reference |
| state          | sync_status | Enum map           |
| updated_at     | synced_at   | Unix → ISO 8601    |

### Enum Mappings

| Our Value | External Value |
| --------- | -------------- |
| draft     | pending        |
| active    | published      |
| archived  | deleted        |

______________________________________________________________________

## Behavior Specifications

### Successful Sync

**Given**: Feature is created in our system **When**: Create event is published **Then**:

- Integration worker picks up event
- External API is called with mapped data
- External ID is stored in our database
- Sync status is updated to "synced"

### Retry on Failure

**Given**: External API returns 503 **When**: Sync attempt fails **Then**:

- Event is requeued with backoff
- Retry counter is incremented
- After max retries, event goes to DLQ
- Alert is triggered if threshold exceeded

### Webhook Validation

**Given**: Webhook received from external service **When**: Signature verification fails **Then**:

- Request is rejected with 401
- Event logged for security audit
- No data changes occur

______________________________________________________________________

## Monitoring

### Metrics

| Metric            | Type       | Alert Threshold |
| ----------------- | ---------- | --------------- |
| sync_success_rate | Percentage | < 95%           |
| sync_latency_p99  | Duration   | > 5s            |
| queue_depth       | Gauge      | > 1000          |
| dlq_size          | Gauge      | > 100           |

### Dashboards

| Dashboard          | Purpose             |
| ------------------ | ------------------- |
| Integration Health | Overall sync status |
| Queue Metrics      | Message throughput  |
| Error Analysis     | Failure patterns    |

______________________________________________________________________

## Acceptance Criteria

### Sync Operations

- [ ] Created features are synced within 30 seconds
- [ ] Updated features are synced within 30 seconds
- [ ] Deleted features are removed from external system
- [ ] Sync status is accurately reflected in UI

### Error Handling

- [ ] Failed syncs are retried with exponential backoff
- [ ] Max retries moves event to dead letter queue
- [ ] Alerts fire when error rate exceeds threshold
- [ ] DLQ items can be manually replayed

### Monitoring

- [ ] All sync operations are logged
- [ ] Metrics are available in monitoring system
- [ ] Dashboards show real-time sync status
- [ ] Historical sync data retained for 30 days

### Security & Authorization

- [ ] API keys are stored in secrets manager
- [ ] Webhook signatures are verified (PC-02)
- [ ] Failed auth attempts are logged
- [ ] Credentials rotation is automated
- [ ] Sensitive fields never cross system boundaries (data permissions enforced)
- [ ] Each permission condition (PC-xx) verified at integration boundary
- [ ] Unauthorized replay attempts rejected

### Testing

- [ ] Unit tests for data mapping and transformation logic
- [ ] Unit tests validate enum conversions and edge cases
- [ ] Integration tests with mocked external service verify retry logic
- [ ] Integration tests validate webhook signature verification
- [ ] Integration tests verify dead letter queue processing
- [ ] Contract tests ensure compatibility with external API specification (recorded fixtures dated within freshness window, or live contract test in CI — not assumed payload shapes)
- [ ] **Every method/endpoint/payload field/header/scope/error code/quota stated in this TDD is traceable to an `External Dependencies` row**
- [ ] **`External Dependencies` table entries were fetched within the freshness window (≤ 90 days from implementation kickoff); stale rows re-fetched and drift reconciled before merge**
- [ ] End-to-end tests verify complete sync workflows with real test environment
- [ ] Chaos tests validate behavior under network failures and timeouts
- [ ] Load tests verify system handles expected event throughput
- [ ] **Security tests validate authentication and authorization flows**
- [ ] **Authorization tests verify sensitive data never sent in outbound payloads**
- [ ] **Authorization tests verify webhook signature validation rejects invalid requests**
- [ ] **Authorization tests verify each permission condition (PC-xx) at system boundaries**
- [ ] **Authorization tests verify unauthorized DLQ replay attempts are rejected**
- [ ] Test coverage ≥ 80% for integration workers and handlers
- [ ] Test coverage ≥ 90% for error handling and retry logic
- [ ] Monitoring tests verify metrics collection and alert triggering
- [ ] Performance tests validate sync latency meets SLA (< 5s p99)
- [ ] UI tests cover integration status displays (if applicable)

### Code Quality

- [ ] All new modules have type annotations and documentation
- [ ] No compiler/linter warnings
- [ ] Code formatted per project standards

______________________________________________________________________

## Related Documents

| Document          | Link                                       |
| ----------------- | ------------------------------------------ |
| Parent PRD        | [prd-name.md](../prds/prd-name.md)         |
| Backend TDD       | [feature-backend.md](./feature-backend.md) |
| External API Docs | [external-api-docs](https://...)           |
| Runbook           | [integration-runbook.md](./runbook.md)     |

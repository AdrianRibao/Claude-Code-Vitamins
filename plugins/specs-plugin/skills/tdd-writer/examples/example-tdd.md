# TDD: Invoice Resource

| Document   | Link                                     |
| ---------- | ---------------------------------------- |
| Parent PRD | [prd-invoicing.md](../prds/invoicing.md) |
| Master TDD | [00-invoicing-master.md](./00-master.md) |

| Domain      | Priority | Status   | Last Updated  |
| ----------- | -------- | -------- | ------------- |
| `Invoicing` | P0       | Planning | December 2025 |

______________________________________________________________________

## Overview

### Purpose

Enable freelancers to create, send, and track invoices. Supports PDF generation, email delivery, and payment status tracking. Core resource for the invoicing system.

### Scope

| Included                   | Out of Scope             |
| -------------------------- | ------------------------ |
| ✅ Invoice CRUD operations | ❌ Payment processing    |
| ✅ Line item management    | ❌ Recurring invoices    |
| ✅ PDF generation          | ❌ Multi-currency        |
| ✅ Email sending           | ❌ Credit notes (v2)     |
| ✅ Payment status tracking | ❌ Partial payments (v2) |

### Key Business Rules

| Rule ID | Description                                           |
| ------- | ----------------------------------------------------- |
| BR-01   | Invoice numbers are sequential with no gaps           |
| BR-02   | Sent invoices cannot be edited (immutable)            |
| BR-03   | VAT calculated automatically based on client location |
| BR-04   | Invoices marked overdue after 30 days past due date   |

______________________________________________________________________

## Data Model

### Invoice

| Attribute      | Type     | Required | Description                              |
| -------------- | -------- | -------- | ---------------------------------------- |
| id             | UUID     | Yes      | Primary key                              |
| user_id        | UUID     | Yes      | Owner (freelancer)                       |
| client_id      | UUID     | Yes      | Client reference                         |
| invoice_number | string   | Yes      | Sequential number, format: INV-YYYY-NNNN |
| status         | enum     | Yes      | draft, sent, paid, overdue, cancelled    |
| issue_date     | date     | Yes      | Date invoice was issued                  |
| due_date       | date     | Yes      | Payment due date                         |
| subtotal       | decimal  | Yes      | Sum of line items before VAT             |
| vat_rate       | decimal  | Yes      | VAT percentage (0-100)                   |
| vat_amount     | decimal  | Yes      | Calculated VAT amount                    |
| total          | decimal  | Yes      | subtotal + vat_amount                    |
| notes          | text     | No       | Optional notes for client                |
| paid_at        | datetime | No       | When payment was recorded                |
| sent_at        | datetime | No       | When invoice was emailed                 |
| created_at     | datetime | Yes      | UTC timestamp                            |
| updated_at     | datetime | Yes      | UTC timestamp                            |

### LineItem

| Attribute   | Type    | Required | Description                   |
| ----------- | ------- | -------- | ----------------------------- |
| id          | UUID    | Yes      | Primary key                   |
| invoice_id  | UUID    | Yes      | Parent invoice                |
| description | string  | Yes      | Service/product description   |
| quantity    | decimal | Yes      | Number of units               |
| unit_price  | decimal | Yes      | Price per unit                |
| amount      | decimal | Yes      | quantity × unit_price         |
| position    | integer | Yes      | Order in invoice (1, 2, 3...) |

### Relationships

| Relation   | Target   | Type       | Description       |
| ---------- | -------- | ---------- | ----------------- |
| user       | User     | belongs_to | Invoice owner     |
| client     | Client   | belongs_to | Invoice recipient |
| line_items | LineItem | has_many   | Invoice details   |

### Constraints

- Unique constraint on `[user_id, invoice_number]`
- `due_date >= issue_date`
- `status` can only transition: draft → sent → paid/overdue → cancelled
- `sent_at` required when status = sent
- `paid_at` required when status = paid

______________________________________________________________________

## Interface Contract

### Actions

| Action       | Type   | Arguments          | Returns   | Description               |
| ------------ | ------ | ------------------ | --------- | ------------------------- |
| create       | create | params, actor      | Invoice   | Create draft invoice      |
| update       | update | id, params, actor  | Invoice   | Update draft invoice only |
| send         | update | id, actor          | Invoice   | Send invoice via email    |
| mark_paid    | update | id, paid_at, actor | Invoice   | Record payment received   |
| cancel       | update | id, actor          | Invoice   | Cancel invoice            |
| list         | read   | filters, actor     | [Invoice] | List user's invoices      |
| get          | read   | id, actor          | Invoice   | Get invoice details       |
| generate_pdf | read   | id, actor          | binary    | Generate PDF document     |
| duplicate    | create | id, actor          | Invoice   | Copy invoice as new draft |

### Code Interface (signatures only)

```typescript
Invoices.create(params, actor: User): Invoice
Invoices.update(id, params, actor: User): Invoice
Invoices.send(id, actor: User): Invoice
Invoices.markPaid(id, paidAt: Date, actor: User): Invoice
Invoices.cancel(id, actor: User): Invoice
Invoices.list(filters: InvoiceFilters, actor: User): Invoice[]
Invoices.get(id, actor: User): Invoice
Invoices.generatePdf(id, actor: User): Buffer
Invoices.duplicate(id, actor: User): Invoice
```

______________________________________________________________________

## Authorization

### Policy Matrix

| Actor  | Create | Read   | Update   | Delete |
| ------ | ------ | ------ | -------- | ------ |
| Owner  | ✅ Own | ✅ Own | ✅ Draft | ❌     |
| Admin  | ❌     | ✅ All | ❌       | ❌     |
| Client | ❌     | ✅ Own | ❌       | ❌     |

### Policy Rules

- Users can only access their own invoices
- Only draft invoices can be updated (sent invoices are immutable)
- Invoices cannot be deleted, only cancelled
- Clients can view invoices sent to them via unique link

______________________________________________________________________

## Behavior Specifications

### Creating an Invoice

**Given**: User is authenticated **When**: User creates invoice with valid client and line items **Then**:

- Invoice created with status = "draft"
- Invoice number auto-generated (next sequential)
- Subtotal, VAT, and total calculated automatically
- Invoice appears in user's invoice list

### Sending an Invoice

**Given**: Invoice exists with status = "draft" **When**: User clicks "Send Invoice" **Then**:

- Status changes to "sent"
- `sent_at` set to current UTC time
- PDF generated and attached to email
- Email sent to client's email address
- Invoice becomes immutable (no edits allowed)

### Marking as Paid

**Given**: Invoice exists with status = "sent" or "overdue" **When**: User marks invoice as paid **Then**:

- Status changes to "paid"
- `paid_at` set to provided date
- Dashboard totals updated
- No further status changes allowed (except cancel)

### Automatic Overdue Detection

**Given**: Invoice has status = "sent" and due_date < today **When**: Daily job runs at midnight UTC **Then**:

- Status changes to "overdue"
- Optional: reminder email sent to client (if enabled)

### Edge Case: Duplicate Invoice

**Given**: Invoice exists (any status) **When**: User duplicates invoice **Then**:

- New invoice created with status = "draft"
- New invoice number generated
- issue_date set to today, due_date set to today + 30
- All line items copied
- Original invoice unchanged

______________________________________________________________________

## Acceptance Criteria

### Core Functionality

- [ ] User can create a new invoice with line items
- [ ] Invoice number auto-generates sequentially
- [ ] VAT calculates automatically based on rate
- [ ] User can preview invoice as PDF before sending
- [ ] User can send invoice via email to client

### Status Management

- [ ] Draft invoices can be edited
- [ ] Sent invoices cannot be edited
- [ ] User can mark sent/overdue invoices as paid
- [ ] Overdue status applied automatically after due date

### Validation

- [ ] Due date must be >= issue date
- [ ] At least one line item required
- [ ] Client must be selected
- [ ] All amounts must be positive

### Authorization

- [ ] Users can only see their own invoices
- [ ] Clients can view invoices via unique link
- [ ] Admins can view all invoices (read-only)

### Performance

- [ ] Invoice list loads in < 200ms
- [ ] PDF generation completes in < 3 seconds
- [ ] Email sent within 30 seconds of clicking "Send"

______________________________________________________________________

## Open Questions

| ID    | Question                            | Status         |
| ----- | ----------------------------------- | -------------- |
| OQ-01 | Should we support partial payments? | Deferred to v2 |
| OQ-02 | PDF template customization options? | Resolved       |

### OQ-01: Partial Payments (Deferred)

**Question**: Should invoices support partial payments with running balance?

**Why it matters**: Affects data model complexity and reporting.

**Resolution**: Deferred to v2. For v1, invoices are either paid or unpaid.

### OQ-02: PDF Templates (Resolved)

**Question**: How much can users customize the PDF template?

**Resolution**: V1 offers 3 preset templates with logo upload and primary color selection. Full template editor deferred to v2.

______________________________________________________________________

## Related Documents

| Document   | Link                                     |
| ---------- | ---------------------------------------- |
| Parent PRD | [prd-invoicing.md](../prds/invoicing.md) |
| Master TDD | [00-invoicing-master.md](./00-master.md) |
| Client TDD | [02-client-resource.md](./02-client.md)  |
| UI TDD     | [03-invoice-ui.md](./03-ui.md)           |

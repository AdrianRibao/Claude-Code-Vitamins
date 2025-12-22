# PRD: Invoicing for Freelancers

| Document   | Link                                 |
| ---------- | ------------------------------------ |
| Parent PRD | N/A (Master PRD)                     |
| Vision Doc | [vision.md](../../vision/company.md) |

| Product               | Priority | Status   | Last Updated  |
| --------------------- | -------- | -------- | ------------- |
| `Invoicing.Freelance` | P0       | Planning | December 2025 |

______________________________________________________________________

## Executive Summary

A simple invoicing system for freelancers and small businesses to create, send, and track invoices. Solves the problem of manual invoice creation using spreadsheets or generic documents that lack professional appearance and payment tracking. Target users are freelancers and micro-businesses in the EU.

______________________________________________________________________

## Problem Statement

### Background

Freelancers and small businesses must issue invoices that comply with local tax regulations (VAT requirements, sequential numbering, mandatory fields). Many struggle with creating professional invoices and tracking payment status.

### The Problem

Freelancers currently create invoices using Word documents, spreadsheets, or generic templates. This leads to:

**Impact**:

- 35% of freelancers report delayed payments due to unclear invoice details
- 50% spend over 2 hours per week on invoicing administration
- Non-compliant invoices can result in tax penalties and rejected expense claims

### Current Solutions

| Solution             | Limitations                                            |
| -------------------- | ------------------------------------------------------ |
| Word/Google Docs     | No automation, manual calculations, no tracking        |
| Spreadsheets         | Error-prone formulas, unprofessional appearance        |
| Enterprise software  | Too complex, expensive, designed for larger businesses |
| Generic invoice apps | Missing EU compliance features, limited customization  |

______________________________________________________________________

## Vision

Freelancers create professional, tax-compliant invoices in under 2 minutes and always know which clients owe them money.

______________________________________________________________________

## Goals and Non-Goals

### Goals

| ID   | Goal                         | Success Metric         | Target      | Timeframe |
| ---- | ---------------------------- | ---------------------- | ----------- | --------- |
| G-01 | Reduce invoice creation time | Time to create invoice | < 2 minutes | Launch    |
| G-02 | Ensure tax compliance        | Compliant invoices     | 100%        | Launch    |
| G-03 | Improve payment tracking     | Overdue visibility     | Real-time   | Launch    |
| G-04 | Reduce late payments         | Average payment time   | -30%        | 6 months  |

### Non-Goals

| ID    | Non-Goal                    | Rationale                                          |
| ----- | --------------------------- | -------------------------------------------------- |
| NG-01 | Payment processing          | Complex integrations, use existing payment methods |
| NG-02 | Accounting/bookkeeping      | Different problem domain, integrate with tools     |
| NG-03 | Inventory management        | Out of scope for freelancer focus                  |
| NG-04 | Multi-currency (beyond EUR) | Focus on EU market first                           |

______________________________________________________________________

## Target Users

### Primary Personas

| Persona            | Description                                                                 | Primary Needs                                              |
| ------------------ | --------------------------------------------------------------------------- | ---------------------------------------------------------- |
| Sofia (Freelancer) | 32-year-old graphic designer, 10-15 clients/month, invoices from her laptop | Quick invoice creation, professional templates, PDF export |
| Marco (Consultant) | 45-year-old IT consultant, 3-5 large clients, needs detailed time tracking  | Itemized invoices, payment reminders, client history       |

### User Segments

| Segment                | Size       | Priority | Notes                             |
| ---------------------- | ---------- | -------- | --------------------------------- |
| Solo freelancers       | 5M in EU   | P0       | Largest segment, simplicity focus |
| Micro-businesses (1-5) | 2M in EU   | P0       | Need multi-user, more features    |
| Creative professionals | 1.5M in EU | P1       | High invoice volume, visual needs |

______________________________________________________________________

## Document Hierarchy

This master PRD is the parent document for the following child PRDs:

| Document                                   | Type    | Status   | Description                  |
| ------------------------------------------ | ------- | -------- | ---------------------------- |
| [01-invoice-creation.md](./01-creation.md) | Feature | Planning | Invoice creation interface   |
| [02-client-management.md](./02-clients.md) | Feature | Planning | Client database and history  |
| [03-payment-tracking.md](./03-payments.md) | Feature | Planning | Payment status and reminders |

### Related TDDs

| TDD                                                   | Type    | Status   | Description                    |
| ----------------------------------------------------- | ------- | -------- | ------------------------------ |
| [invoice-backend.md](../tdds/invoicing/01-backend.md) | Backend | Planning | Invoice data model and actions |
| [invoice-ui.md](../tdds/invoicing/02-ui.md)           | UI      | Planning | Invoice creation interface     |

______________________________________________________________________

## User Workflows

### Create New Invoice

**Scenario: Freelancer creates invoice for completed project**

```
1. User clicks "New Invoice" from dashboard
2. System auto-fills next invoice number and current date
3. User selects client from dropdown (or creates new)
4. User adds line items with description, quantity, rate
5. System calculates subtotal, VAT, and total automatically
6. User clicks "Preview" to see PDF version
7. User clicks "Send" to email invoice to client
```

**Time**: < 2 minutes

### Track Payment Status

**Scenario: Freelancer checks which invoices are unpaid**

```
1. User opens "Invoices" from dashboard
2. System displays list with status badges (Paid, Pending, Overdue)
3. User filters by "Overdue" to see problem invoices
4. User clicks invoice to see details and payment history
5. User clicks "Send Reminder" to notify client
```

**Time**: < 1 minute

### Mark Invoice as Paid

**Scenario: Freelancer receives payment and updates status**

```
1. User receives bank notification of payment
2. User opens the corresponding invoice
3. User clicks "Mark as Paid"
4. System prompts for payment date and method
5. User confirms, invoice status changes to "Paid"
6. System updates dashboard totals
```

**Time**: < 30 seconds

### Edge Case: Client Disputes Invoice

**Scenario: Client requests changes to issued invoice**

```
1. Client contacts freelancer about invoice error
2. User opens invoice, clicks "Create Credit Note"
3. System generates credit note referencing original
4. User creates corrected invoice with new number
5. System maintains audit trail of all documents
```

**Time**: 2-3 minutes

**Notes**: Original invoice cannot be edited after sending; corrections require credit notes.

______________________________________________________________________

## Requirements

### Functional Requirements

| ID    | Requirement                                                   | Priority |
| ----- | ------------------------------------------------------------- | -------- |
| FR-01 | System generates sequential invoice numbers automatically     | P0       |
| FR-02 | System calculates VAT based on client location and type       | P0       |
| FR-03 | System generates PDF invoices with customizable template      | P0       |
| FR-04 | User can send invoices via email directly from system         | P0       |
| FR-05 | System tracks invoice status (Draft, Sent, Paid, Overdue)     | P0       |
| FR-06 | System sends automatic payment reminders for overdue invoices | P1       |
| FR-07 | User can duplicate invoices for recurring clients             | P1       |
| FR-08 | System generates quarterly revenue reports                    | P2       |

### Non-Functional Requirements

| ID     | Category      | Requirement        | Target                              |
| ------ | ------------- | ------------------ | ----------------------------------- |
| NFR-01 | Performance   | Invoice generation | < 3 seconds for PDF                 |
| NFR-02 | Availability  | System uptime      | 99.5%                               |
| NFR-03 | Security      | Data encryption    | AES-256 at rest, TLS 1.3 in transit |
| NFR-04 | Compliance    | GDPR               | Full compliance                     |
| NFR-05 | Compatibility | Browser support    | Chrome, Firefox, Safari, Edge       |
| NFR-06 | Localization  | Languages          | English, Spanish, German, French    |

### Business Rules

| ID    | Rule                                                | Rationale               |
| ----- | --------------------------------------------------- | ----------------------- |
| BR-01 | Invoice numbers must be sequential with no gaps     | EU tax compliance       |
| BR-02 | Sent invoices cannot be edited, only credited       | Audit trail requirement |
| BR-03 | VAT must be displayed separately from net amount    | Tax regulation          |
| BR-04 | Invoices marked overdue after 30 days past due date | Standard payment terms  |
| BR-05 | Credit notes must reference original invoice number | Accounting standards    |

______________________________________________________________________

## Success Metrics

| Metric                 | Baseline | Target    | Timeframe | How to Measure                  |
| ---------------------- | -------- | --------- | --------- | ------------------------------- |
| Invoice creation time  | 15 min   | < 2 min   | Launch    | In-app analytics                |
| Payment tracking usage | 0%       | 80%       | 3 months  | Users updating payment status   |
| Late payment rate      | 40%      | < 25%     | 6 months  | Invoices paid after due date    |
| User satisfaction      | N/A      | > 4.2/5.0 | 3 months  | In-app NPS survey               |
| Monthly active users   | 0        | 5,000     | 6 months  | Users creating 1+ invoice/month |

### Leading Indicators

| Indicator                | Target       | Tracking Frequency |
| ------------------------ | ------------ | ------------------ |
| Weekly invoice volume    | 10+ per user | Weekly             |
| Reminder email open rate | > 50%        | Weekly             |
| Template customization   | 60% of users | Monthly            |

______________________________________________________________________

## Scope

### In Scope

| Category     | Included                                                  |
| ------------ | --------------------------------------------------------- |
| Features     | Invoice creation, PDF generation, email sending, tracking |
| Platforms    | Web application (responsive)                              |
| Users        | Freelancers, micro-businesses                             |
| Integrations | Email (SMTP), PDF generation                              |

### Out of Scope

| Category   | Excluded                       | Rationale                     |
| ---------- | ------------------------------ | ----------------------------- |
| Payments   | Payment processing, bank links | Integrate with existing tools |
| Accounting | Double-entry bookkeeping       | Different product             |
| Inventory  | Stock management               | Not relevant for services     |
| Estimates  | Quotes and proposals           | v2 consideration              |

### Future Considerations (v2+)

| Feature            | Description                      | Tentative Timeline |
| ------------------ | -------------------------------- | ------------------ |
| Recurring invoices | Automatic monthly billing        | v2                 |
| Estimates/Quotes   | Convert quotes to invoices       | v2                 |
| Bank integration   | Payment matching with bank feeds | v2                 |
| Multi-currency     | USD, GBP support                 | v3                 |

______________________________________________________________________

## Constraints & Dependencies

### Constraints

| Type      | Constraint                 | Impact                       |
| --------- | -------------------------- | ---------------------------- |
| Timeline  | Launch by Q2 2025          | MVP scope only               |
| Budget    | €30K development           | Web-only, no mobile apps     |
| Technical | PDF generation library     | Evaluate open-source options |
| Legal     | VAT compliance per country | Need country-specific rules  |

### Dependencies

| Dependency             | Owner        | Status     | Risk   |
| ---------------------- | ------------ | ---------- | ------ |
| Email delivery service | SendGrid/SES | Available  | Low    |
| PDF generation library | TBD          | Evaluating | Medium |
| VAT rate database      | EU VIES      | Available  | Low    |

______________________________________________________________________

## Risks

| ID   | Risk                              | Probability | Impact | Mitigation                        |
| ---- | --------------------------------- | ----------- | ------ | --------------------------------- |
| R-01 | VAT rules complexity across EU    | High        | High   | Start with top 5 markets only     |
| R-02 | Users expect payment integration  | Medium      | Medium | Clear positioning, future roadmap |
| R-03 | PDF generation performance issues | Medium      | Medium | Async generation, caching         |
| R-04 | Email deliverability problems     | Low         | High   | Use reputable provider, SPF/DKIM  |

______________________________________________________________________

## Open Questions

| ID    | Question                             | Status   |
| ----- | ------------------------------------ | -------- |
| OQ-01 | Should we support invoice templates? | Resolved |
| OQ-02 | How to handle partial payments?      | Open     |

### OQ-01: Invoice Templates (Resolved)

**Question**: Should users be able to customize invoice templates?

**Resolution**: Yes, provide 3-5 professional templates with logo upload and color customization. Full template editing deferred to v2.

### OQ-02: Partial Payments (Open)

**Question**: How should the system handle partial payments?

**Why it matters**: Affects data model, UI complexity, and reporting.

**Possible answers**:

- [ ] Single payment only (simpler, mark paid/unpaid)
- [ ] Multiple payments with running balance
- [ ] Partial payments with automatic status updates

**Status**: Open - needs product decision

______________________________________________________________________

## Timeline

| Milestone      | Target Date   | Description               |
| -------------- | ------------- | ------------------------- |
| PRD Approval   | January 2025  | Requirements finalized    |
| TDD Completion | February 2025 | Technical design complete |
| Alpha Release  | March 2025    | Internal testing          |
| Beta Release   | April 2025    | 100 beta users            |
| GA Release     | May 2025      | Public launch             |

______________________________________________________________________

## Related Documents

| Document               | Link                                                              |
| ---------------------- | ----------------------------------------------------------------- |
| User Research Report   | [research/freelancer-interviews.md](../../research/interviews.md) |
| Competitive Analysis   | [research/competitor-analysis.md](../../research/competitors.md)  |
| VAT Requirements       | [legal/eu-vat-requirements.md](../../legal/vat.md)                |
| Technical Architecture | [tdds/invoicing/00-master.md](../tdds/invoicing/00-master.md)     |

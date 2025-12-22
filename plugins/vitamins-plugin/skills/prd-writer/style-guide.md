# PRD Style Guide

## Core Principles

### WHAT and WHY, Never HOW

Every PRD section answers "what problem are we solving" and "why it matters", never "how to implement it". Implementation details belong in TDDs.

### Clear Over Comprehensive

Prefer clarity and focus over exhaustive coverage. A short, clear PRD is better than a long, ambiguous one.

### Measurable Over Vague

Every goal and success metric must be measurable. If you can't measure it, you can't know if you succeeded.

### User-Centric Always

Frame everything from the user's perspective. Requirements exist to solve user problems.

## Problem Statement Format

### Good Problem Statements

```markdown
## Problem Statement

Household employees currently track their working hours manually on paper or spreadsheets.
This leads to:
- Inaccurate records (forgotten hours, rounding errors)
- Disputes between employers and employees
- Compliance risks for labor law violations
- Time wasted on manual calculations

**Impact**: 40% of household employers report time-tracking disputes monthly.
```

### Bad Problem Statements

```markdown
## Problem Statement

We need a time tracking system.
```

Why it's bad: Doesn't explain the problem, just proposes a solution.

## Goals and Non-Goals Format

### Goals Table

```markdown
## Goals

| ID | Goal | Success Metric | Target |
|----|------|----------------|--------|
| G-01 | Reduce time-tracking errors | Error rate | < 5% |
| G-02 | Eliminate manual calculations | Automation rate | 100% |
| G-03 | Improve employer-employee trust | Dispute rate | < 10% |
| G-04 | Ensure labor law compliance | Compliance score | 100% |
```

### Non-Goals Table

```markdown
## Non-Goals

| ID | Non-Goal | Rationale |
|----|----------|-----------|
| NG-01 | Replace payroll systems | Out of scope, integrate instead |
| NG-02 | Support contract workers | Focus on household employees only |
| NG-03 | Multi-language support | English/Spanish only for v1 |
| NG-04 | Mobile-first design | Desktop-first, mobile later |
```

Non-goals are as important as goals. They prevent scope creep.

## Target Users Format

### Persona Table

```markdown
## Target Users

| Persona | Description | Primary Needs |
|---------|-------------|---------------|
| Maria (Employee) | Household employee, works for 2 families, limited tech skills | Easy time logging, clear hours summary |
| Carlos (Employer) | Busy professional, employs 1 housekeeper, values convenience | Quick approvals, automated calculations |
| Ana (Administrator) | HR manager for family office, manages 5 employees | Bulk operations, reporting, compliance |
```

### User Characteristics

Document relevant characteristics that affect requirements:

```markdown
### Maria (Employee) - Detailed Profile

**Demographics**: 35-55 years old, predominantly female
**Tech comfort**: Low to moderate, uses WhatsApp daily
**Primary device**: Android smartphone, often budget models
**Key frustrations**: Forgotten hours, unclear pay calculations
**Success looks like**: "I can see exactly what I'll be paid this month"
```

## User Workflows Format

User workflows are step-by-step scenarios showing how users accomplish tasks. They're more concrete and actionable than abstract user stories.

### Standard Format

```markdown
## User Workflows

### Employee Logs Time

**Scenario: Employee clocks in at start of work day**

1. Employee opens app on phone
2. Taps "Start Work" button
3. System records timestamp and shows confirmation
4. Employee sees "Working since 9:00 AM" status

**Time**: 30 seconds

### Employer Approves Hours

**Scenario: Employer reviews and approves weekly hours**

1. Employer receives notification: "Maria submitted 32 hours"
2. Opens app, views time log summary
3. Reviews each day's clock-in/clock-out times
4. Taps "Approve All"
5. System confirms approval, notifies employee

**Time**: 2-3 minutes
```

### Edge Case Workflows

Document how the system handles non-happy-path scenarios:

```markdown
### Employee Disputes Hours

**Scenario: Employee disagrees with employer's correction**

1. Employee receives notification of hour adjustment
2. Views proposed change: "Changed clock-out from 7 PM to 5 PM"
3. Taps "Dispute"
4. Enters explanation: "I stayed late to finish cleaning"
5. Submits dispute
6. System notifies employer of dispute
7. Incident remains open until both agree

**Time**: 2-3 minutes
```

### Why Workflows Over User Stories

| Aspect     | User Stories                       | User Workflows                  |
| ---------- | ---------------------------------- | ------------------------------- |
| Format     | "As a... I want... So that..."     | Step-by-step scenario           |
| Clarity    | Abstract, needs interpretation     | Concrete, directly testable     |
| Time       | No duration estimate               | Includes time estimate          |
| UX         | Needs translation to UI            | Maps directly to UI flow        |
| Edge cases | Often separate acceptance criteria | Naturally included as scenarios |

## Requirements Format

### Functional Requirements Table

```markdown
## Functional Requirements

| ID | Requirement | Priority | User Story |
|----|-------------|----------|------------|
| FR-01 | System captures clock-in/clock-out times | P0 | US-01 |
| FR-02 | System calculates daily hours worked | P0 | US-02 |
| FR-03 | System calculates overtime (>40 hrs/week) | P1 | US-04 |
| FR-04 | Employer receives notification for approval | P1 | US-03 |
| FR-05 | System generates weekly summary report | P1 | US-02 |
```

### Non-Functional Requirements Table

```markdown
## Non-Functional Requirements

| ID | Category | Requirement | Target |
|----|----------|-------------|--------|
| NFR-01 | Performance | Time logging response | < 2 seconds |
| NFR-02 | Availability | System uptime | 99.5% |
| NFR-03 | Security | Data encryption | AES-256 at rest |
| NFR-04 | Accessibility | WCAG compliance | Level AA |
| NFR-05 | Scalability | Concurrent users | 10,000 |
```

## Success Metrics Format

### Metrics Table

```markdown
## Success Metrics

| Metric | Baseline | Target | Timeframe | How to Measure |
|--------|----------|--------|-----------|----------------|
| Time-tracking errors | 40% | < 5% | 3 months | User-reported disputes |
| User adoption | 0% | 80% | 6 months | Active users / total users |
| Time to log hours | 5 min | < 30 sec | 1 month | In-app analytics |
| Employer approval time | 2 days | < 4 hours | 3 months | Time from submission to approval |
| User satisfaction | N/A | > 4.0/5.0 | 3 months | In-app survey |
```

### Leading vs Lagging Indicators

Distinguish between leading (predictive) and lagging (outcome) indicators:

```markdown
### Leading Indicators (Weekly tracking)
- Daily active users
- Time entries per user
- Feature adoption rate

### Lagging Indicators (Monthly tracking)
- Error rate
- User satisfaction score
- Dispute resolution rate
```

## Scope Boundaries Format

### In/Out Scope Table

```markdown
## Scope

| In Scope | Out of Scope |
|----------|--------------|
| Time clock-in/clock-out | Payroll processing |
| Hours calculation | Tax calculations |
| Overtime calculation | Benefits administration |
| Employer approval workflow | Contract management |
| Weekly summary reports | Detailed analytics dashboard |
| Mobile and web access | Native mobile apps (v2) |
| English and Spanish | Other languages |
```

### Scope Rationale

For controversial scope decisions, explain why:

```markdown
### Why Payroll is Out of Scope

Payroll processing is explicitly out of scope because:
1. Requires complex integrations with banking systems
2. Regulatory requirements vary by jurisdiction
3. Existing solutions (Gusto, Paylocity) are mature
4. Our core value is time tracking accuracy, not payment processing

**Integration approach**: Export data in formats compatible with major payroll providers.
```

## Constraints Format

### Constraints Table

```markdown
## Constraints

| Type | Constraint | Impact |
|------|------------|--------|
| Timeline | Launch by Q2 2025 | Limits feature scope to MVP |
| Budget | $50K development budget | No native mobile apps |
| Technical | Must integrate with existing auth | Use OAuth, no custom auth |
| Legal | GDPR compliance required | Data retention limits |
| Resource | 2 developers available | Sequential feature development |
```

## Anti-Patterns to Avoid

### Implementation Details

**Bad:**

```markdown
## Requirements

FR-01: Use PostgreSQL database with JSONB columns for flexible time entry storage.
FR-02: Implement React frontend with Redux state management.
```

**Good:**

```markdown
## Requirements

FR-01: System stores time entries with date, start time, end time, and notes.
FR-02: User interface supports real-time updates across devices.
```

### Vague Requirements

**Bad:**

```markdown
FR-01: System should be fast.
FR-02: System should be secure.
FR-03: System should be user-friendly.
```

**Good:**

```markdown
FR-01: Time logging completes in < 2 seconds on 3G connection.
FR-02: All data encrypted at rest (AES-256) and in transit (TLS 1.3).
FR-03: New users can log time without training (< 30 seconds to first entry).
```

### Solution Masquerading as Problem

**Bad:**

```markdown
## Problem Statement

We need to build a mobile app for time tracking.
```

**Good:**

```markdown
## Problem Statement

Household employees struggle to accurately track working hours, leading to
payment disputes and compliance issues. Current solutions (paper, spreadsheets)
are error-prone and lack employer visibility.
```

### Unmeasurable Goals

**Bad:**

```markdown
## Goals

- Make time tracking easier
- Improve user experience
- Reduce errors
```

**Good:**

```markdown
## Goals

| Goal | Success Metric | Target |
|------|----------------|--------|
| Simplify time tracking | Time to log hours | < 30 seconds |
| Improve satisfaction | User rating | > 4.0/5.0 |
| Reduce errors | Dispute rate | < 5% |
```

## Document Structure

### Header Section

```markdown
# PRD: [Product/Feature Name]

| Document | Link |
|----------|------|
| Master PRD | [00-product-master.md](./00-master.md) |
| Related TDD | [TDD-feature.md](../tdds/feature.md) |

| Product | Priority | Status | Last Updated |
|---------|----------|--------|--------------|
| `Product.Feature` | P0 | Planning | December 2025 |
```

### Sections Order

1. Problem Statement & Background
2. Goals and Non-Goals
3. Target Users & Personas
4. User Stories
5. Requirements (Functional, Non-Functional)
6. Success Metrics
7. Scope (In/Out)
8. Constraints & Dependencies
9. Open Questions
10. Related Documents

## Writing Tips

### Be Specific

- "Users" → "Household employees aged 35-55 with limited tech skills"
- "Fast" → "< 2 seconds on 3G connection"
- "Secure" → "AES-256 encryption, SOC 2 Type II compliant"

### Use Active Voice

- "The system should be able to calculate..." → "System calculates..."
- "Users will be notified..." → "System notifies users..."

### Prioritize Ruthlessly

Use MoSCoW or P0/P1/P2:

| Priority | Meaning      | Rule                    |
| -------- | ------------ | ----------------------- |
| P0       | Must have    | Launch blocker          |
| P1       | Should have  | Important, not critical |
| P2       | Nice to have | If time permits         |
| P3       | Future       | Explicitly deferred     |

### Reference User Research

When available, cite research:

```markdown
**Note**: 73% of surveyed household employers reported monthly
time-tracking disputes (User Research Report, Oct 2024).
```

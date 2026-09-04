# PRD: [Product Name] - Master

| Document   | Link                      |
| ---------- | ------------------------- |
| Parent PRD | [parent.md](./parent.md)  |
| Vision Doc | [vision.md](../vision.md) |

| Product       | Priority | Status   | Last Updated   |
| ------------- | -------- | -------- | -------------- |
| `ProductName` | P0       | Planning | [Month] [Year] |

______________________________________________________________________

## Executive Summary

[2-3 sentences: What is this product? What problem does it solve? Who is it for?]

______________________________________________________________________

## Problem Statement

### Background

[Context about the current situation and why change is needed]

### The Problem

[Clear description of the problem being solved]

**Impact**:

- [Quantified impact point 1]
- [Quantified impact point 2]
- [Quantified impact point 3]

### Current Solutions

| Solution             | Limitations             |
| -------------------- | ----------------------- |
| [Current approach 1] | [Why it's insufficient] |
| [Current approach 2] | [Why it's insufficient] |

______________________________________________________________________

## Vision

[1-2 sentences: What does success look like? Where are we heading?]

______________________________________________________________________

## Goals and Non-Goals

### Goals

| ID   | Goal             | Success Metric | Target   | Timeframe |
| ---- | ---------------- | -------------- | -------- | --------- |
| G-01 | [Primary goal]   | [Metric]       | [Target] | [Time]    |
| G-02 | [Secondary goal] | [Metric]       | [Target] | [Time]    |
| G-03 | [Tertiary goal]  | [Metric]       | [Target] | [Time]    |

### Non-Goals

| ID    | Non-Goal               | Rationale |
| ----- | ---------------------- | --------- |
| NG-01 | [What we're NOT doing] | [Why not] |
| NG-02 | [What we're NOT doing] | [Why not] |

______________________________________________________________________

## Target Users

### Primary Personas

| Persona  | Description         | Primary Needs |
| -------- | ------------------- | ------------- |
| [Name 1] | [Brief description] | [Key needs]   |
| [Name 2] | [Brief description] | [Key needs]   |

### User Segments

| Segment     | Size        | Priority | Notes   |
| ----------- | ----------- | -------- | ------- |
| [Segment 1] | [Est. size] | P0       | [Notes] |
| [Segment 2] | [Est. size] | P1       | [Notes] |

______________________________________________________________________

## Document Hierarchy

This master PRD is the parent document for the following child PRDs:

| Document                             | Type    | Status      | Description         |
| ------------------------------------ | ------- | ----------- | ------------------- |
| [01-feature-a.md](./01-feature-a.md) | Feature | Planning    | [Brief description] |
| [02-feature-b.md](./02-feature-b.md) | Feature | Planning    | [Brief description] |
| [03-api.md](./03-api.md)             | API     | Not Started | [Brief description] |

### Related TDDs

| TDD                                                     | Type    | Status      | Description         |
| ------------------------------------------------------- | ------- | ----------- | ------------------- |
| [feature-a-backend.md](../tdds/feature-a/01-backend.md) | Backend | Not Started | [Brief description] |
| [feature-a-ui.md](../tdds/feature-a/02-ui.md)           | UI      | Not Started | [Brief description] |

______________________________________________________________________

## Cross-Cutting Requirements

### Functional Requirements

| ID    | Requirement               | Priority | Applies To          |
| ----- | ------------------------- | -------- | ------------------- |
| CR-01 | [Requirement description] | P0       | All features        |
| CR-02 | [Requirement description] | P1       | [Specific features] |

### Non-Functional Requirements

| ID     | Category      | Requirement   | Target   |
| ------ | ------------- | ------------- | -------- |
| NFR-01 | Performance   | [Requirement] | [Target] |
| NFR-02 | Security      | [Requirement] | [Target] |
| NFR-03 | Accessibility | [Requirement] | [Target] |
| NFR-04 | Availability  | [Requirement] | [Target] |

______________________________________________________________________

## Success Metrics

### Key Performance Indicators

| Metric  | Baseline  | Target | Timeframe | How to Measure |
| ------- | --------- | ------ | --------- | -------------- |
| [KPI 1] | [Current] | [Goal] | [When]    | [Method]       |
| [KPI 2] | [Current] | [Goal] | [When]    | [Method]       |

### Leading Indicators

| Indicator             | Target   | Tracking Frequency |
| --------------------- | -------- | ------------------ |
| [Leading indicator 1] | [Target] | Weekly             |
| [Leading indicator 2] | [Target] | Weekly             |

______________________________________________________________________

## Scope

### In Scope

| Category     | Included                    |
| ------------ | --------------------------- |
| Features     | [List of included features] |
| Platforms    | [Supported platforms]       |
| Users        | [Supported user types]      |
| Integrations | [Required integrations]     |

### Out of Scope

| Category   | Excluded          | Rationale |
| ---------- | ----------------- | --------- |
| [Category] | [What's excluded] | [Why]     |
| [Category] | [What's excluded] | [Why]     |

### Future Considerations (v2+)

| Feature            | Description   | Tentative Timeline |
| ------------------ | ------------- | ------------------ |
| [Future feature 1] | [Description] | v2                 |
| [Future feature 2] | [Description] | v3                 |

______________________________________________________________________

## Constraints & Dependencies

### Constraints

| Type      | Constraint   | Impact   |
| --------- | ------------ | -------- |
| Timeline  | [Constraint] | [Impact] |
| Budget    | [Constraint] | [Impact] |
| Technical | [Constraint] | [Impact] |
| Legal     | [Constraint] | [Impact] |

### Dependencies

| Dependency     | Owner         | Status   | Risk         |
| -------------- | ------------- | -------- | ------------ |
| [Dependency 1] | [Team/Person] | [Status] | [Risk level] |
| [Dependency 2] | [Team/Person] | [Status] | [Risk level] |

______________________________________________________________________

## Risks

| ID   | Risk               | Probability  | Impact       | Mitigation            |
| ---- | ------------------ | ------------ | ------------ | --------------------- |
| R-01 | [Risk description] | High/Med/Low | High/Med/Low | [Mitigation strategy] |
| R-02 | [Risk description] | High/Med/Low | High/Med/Low | [Mitigation strategy] |

______________________________________________________________________

## ✅ Decisions (Resolved)

<!-- Judgment calls made while writing this PRD. Fold each into its section; log it here with a one-line rationale. -->

One heading per decision. IDs are referenced from the sections above.

### D-01 — [Decision name]

**Choice.** [Choice]

**Why.** [Why, with the concrete trade-off]

______________________________________________________________________

## Open Questions

<!-- Only questions a human must settle (strategy, pricing, legal, budget, stakeholder priority, facts you cannot obtain).
     If there are none, replace the table and detail blocks with exactly:
     *No open questions — every design decision is recorded in ✅ Decisions (Resolved).* -->

| ID    | Question                      | Status |
| ----- | ----------------------------- | ------ |
| OQ-01 | [Question needing resolution] | Open   |

### OQ-01: [Question Title]

**Question**: [Full, self-contained question]

**Why it matters**: [Impact on project]

**Possible answers**:

- [ ] [Option 1] — [consequence] **(recommended)**
- [ ] [Option 2] — [consequence]
- [ ] [Option 3] — [consequence]

**Status**: Open - needs [stakeholder] input

______________________________________________________________________

## Timeline

| Milestone         | Target Date | Description                    |
| ----------------- | ----------- | ------------------------------ |
| PRD Approval      | [Date]      | All PRDs reviewed and approved |
| TDD Completion    | [Date]      | Technical design complete      |
| Development Start | [Date]      | Implementation begins          |
| Alpha Release     | [Date]      | Internal testing               |
| Beta Release      | [Date]      | Limited user testing           |
| GA Release        | [Date]      | General availability           |

______________________________________________________________________

## Related Documents

| Document               | Link   |
| ---------------------- | ------ |
| Vision Document        | [link] |
| User Research          | [link] |
| Competitive Analysis   | [link] |
| Technical Architecture | [link] |

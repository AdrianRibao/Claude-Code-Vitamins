# PRD: [Feature Name]

| Document    | Link                                   |
| ----------- | -------------------------------------- |
| Master PRD  | [00-product-master.md](./00-master.md) |
| Related TDD | [feature-tdd.md](../tdds/feature.md)   |

| Product           | Priority | Status   | Last Updated   |
| ----------------- | -------- | -------- | -------------- |
| `Product.Feature` | P0-P3    | Planning | [Month] [Year] |

______________________________________________________________________

## Overview

### Purpose

[2-3 sentences: What does this feature do? What problem does it solve?]

### Scope

| Included                 | Excluded                     |
| ------------------------ | ---------------------------- |
| [What's in this feature] | [What's NOT in this feature] |
| [Included functionality] | [Excluded functionality]     |

______________________________________________________________________

## Problem Statement

[Clear description of the specific problem this feature addresses]

**User Pain Points**:

- [Pain point 1 with evidence]
- [Pain point 2 with evidence]
- [Pain point 3 with evidence]

______________________________________________________________________

## Goals

| ID   | Goal             | Success Metric | Target   |
| ---- | ---------------- | -------------- | -------- |
| G-01 | [Primary goal]   | [Metric]       | [Target] |
| G-02 | [Secondary goal] | [Metric]       | [Target] |

### Non-Goals

| ID    | Non-Goal                     | Rationale |
| ----- | ---------------------------- | --------- |
| NG-01 | [What this feature won't do] | [Why not] |

______________________________________________________________________

## Target Users

| Persona  | Relevance | Primary Use Case            |
| -------- | --------- | --------------------------- |
| [User 1] | Primary   | [How they use this feature] |
| [User 2] | Secondary | [How they use this feature] |

______________________________________________________________________

## User Workflows

### [Workflow 1 Title]

**Scenario: [Brief description of the scenario]**

```
1. [Actor] [action]
2. System [response]
3. [Actor] [action]
4. System [response]
5. [Outcome achieved]
```

**Time**: [Estimated duration]

### [Workflow 2 Title]

**Scenario: [Brief description of the scenario]**

```
1. [Actor] [action]
2. System [response]
3. [Outcome achieved]
```

**Time**: [Estimated duration]

### Edge Case: [Scenario Name]

**Scenario: [What makes this an edge case]**

```
1. [Steps that lead to edge case]
2. System [how it handles the situation]
3. [Resolution or outcome]
```

**Time**: [Estimated duration]

______________________________________________________________________

## Requirements

### Functional Requirements

| ID    | Requirement               | Priority |
| ----- | ------------------------- | -------- |
| FR-01 | [What the system must do] | P0       |
| FR-02 | [What the system must do] | P0       |
| FR-03 | [What the system must do] | P1       |
| FR-04 | [What the system must do] | P2       |

### Non-Functional Requirements

| ID     | Category      | Requirement   | Target   |
| ------ | ------------- | ------------- | -------- |
| NFR-01 | Performance   | [Requirement] | [Target] |
| NFR-02 | Security      | [Requirement] | [Target] |
| NFR-03 | Accessibility | [Requirement] | [Target] |

### Business Rules

| ID    | Rule            | Rationale              |
| ----- | --------------- | ---------------------- |
| BR-01 | [Business rule] | [Why this rule exists] |
| BR-02 | [Business rule] | [Why this rule exists] |

______________________________________________________________________

## Success Metrics

| Metric     | Baseline  | Target | Timeframe | How to Measure |
| ---------- | --------- | ------ | --------- | -------------- |
| [Metric 1] | [Current] | [Goal] | [When]    | [Method]       |
| [Metric 2] | [Current] | [Goal] | [When]    | [Method]       |

______________________________________________________________________

## Constraints

| Type   | Constraint   | Impact                       |
| ------ | ------------ | ---------------------------- |
| [Type] | [Constraint] | [How it affects the feature] |

______________________________________________________________________

## Dependencies

| Dependency     | Type                       | Status   | Risk   |
| -------------- | -------------------------- | -------- | ------ |
| [Dependency 1] | Feature/External/Technical | [Status] | [Risk] |
| [Dependency 2] | Feature/External/Technical | [Status] | [Risk] |

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

**Why it matters**: [Impact on feature]

**Possible answers**:

- [ ] [Option 1] — [consequence] **(recommended)**
- [ ] [Option 2] — [consequence]

**Status**: Open - needs [stakeholder] input

______________________________________________________________________

## Related Documents

| Document        | Link                                                |
| --------------- | --------------------------------------------------- |
| Master PRD      | [00-master.md](./00-master.md)                      |
| Sibling Feature | [02-sibling.md](./02-sibling.md)                    |
| Backend TDD     | [feature-backend.md](../tdds/feature/01-backend.md) |
| UI TDD          | [feature-ui.md](../tdds/feature/02-ui.md)           |

---
name: prd-writer
description: Generate excellent Product Requirements Documents focused on problems, goals, users, and success metrics. Use when creating PRDs, defining product scope, or documenting feature requirements.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - AskUserQuestion
  - TodoWrite
  - Task
  - mcp__plugin_context7_context7__resolve-library-id
  - mcp__plugin_context7_context7__query-docs
---

# PRD Writer Skill

> **Philosophy**: PRDs define WHAT problem we're solving and WHY, not HOW to implement it. Technical details belong in TDDs.

## When to Use

- New product or feature requiring clear requirements definition
- Stakeholder alignment on scope, goals, and success criteria
- Complex features needing structured requirements discovery
- Before creating TDDs - PRDs are the parent specification
- PRD review requests to identify scope creep or missing requirements
- Consolidating PRD after Open Questions are answered

## Usage

```
/prd [feature-name] [--type master|feature|api|integration] [--no-questions] [--ask] [--review] [--consolidate @path]
```

| Flag                  | Purpose                                               |
| --------------------- | ----------------------------------------------------- |
| `--type`              | PRD type: master, feature, api, integration           |
| `--no-questions`      | Skip upfront questions (use with comprehensive brief) |
| `--ask`               | Surface decide-and-record items as Open Questions too |
| `--review`            | Analyze existing PRD for scope creep or gaps          |
| `--consolidate @path` | Apply OQ answers, tighten document                    |

## Output Location

PRDs are created at:

```
specs/prds/{product}/{nn}-{product}-{type}.md
```

**Examples:**

| Command                               | Output Path                                   |
| ------------------------------------- | --------------------------------------------- |
| `/prd user-auth --type master`        | `specs/prds/user-auth/00-user-auth-master.md` |
| `/prd user-auth-login --type feature` | `specs/prds/user-auth/01-user-auth-login.md`  |
| `/prd user-auth-api --type api`       | `specs/prds/user-auth/02-user-auth-api.md`    |

**Numbering convention:**

- `00-` Master PRD (product overview, document hierarchy)
- `01+` Feature PRDs (specific features, ordered by priority)

Ask user to confirm or override path during Phase 0 questions.

## Resources

- [style-guide.md](style-guide.md) - PRD writing conventions and rules
- [templates/](templates/) - PRD templates by type (master, feature)
- [examples/](examples/) - Reference PRD examples

## Behavioral Mindset

Requirements-first approach: every section answers "what" and "why", never "how". Use tables for scannable requirements. Write clear user workflows with step-by-step scenarios. Create measurable success metrics. Never include code or implementation details.

**Always ask clarifying questions before generating a PRD.** Understanding the problem deeply prevents scope creep and produces better specifications.

## Key Actions

### Phase 0: Requirements Discovery (Always)

1. **Read Context**: Review existing documents, related PRDs, market context
2. **Identify Problem**: Understand the core problem being solved
3. **Ask Questions**: Use `AskUserQuestion` to clarify before writing
    - What problem are we solving? For whom?
    - What are the primary goals and non-goals?
    - Who are the target users and their key workflows?
    - What does success look like? How will we measure it?
    - What constraints exist (timeline, resources, technical)?
    - What's explicitly out of scope?
4. **Confirm Understanding**: Summarize answers before proceeding

**Skip questioning only if:** User provides comprehensive brief AND explicitly says "no questions needed".

### Phase 1: PRD Core Generation

1. **Discover**: Analyze problem space, understand user needs, identify scope boundaries
2. **Map Document Hierarchy**: Identify parent PRDs, child feature PRDs, related TDDs
3. **Analyze Existing Patterns**: Find related PRDs in the codebase for consistency
4. **Structure**: Organize into logical sections following lean PRD template
5. **Specify**: Write problem statement, goals, user workflows, requirements, success metrics
6. **Link Documents**: Add "Related Documents" section with all relevant PRDs and TDDs
7. **Write PRD**: Save to file with placeholder `✅ Decisions (Resolved)` and Open Questions sections

### Phase 2: Decision Triage (decide by default)

1. **Invoke Sequential MCP**: Use `--ultrathink` to scan the complete PRD for ambiguities, missing decisions, edge cases, and scope gaps
2. **Classify each finding** (see [Question Policy](#question-policy)): decide-and-record, ask, or non-issue
3. **Decide-and-record**: Pick the answer you are confident in, fold it into the relevant section, and log it under `## ✅ Decisions (Resolved)` as one `### D-NN — title` heading per decision with a `**Choice.**` line and a `**Why.**` line — never a table (rationales with real trade-offs make it wider than a terminal) and never a single paragraph (a `wrap = "no"` formatter joins it into one line)
4. **Ask sparingly**: Only findings a human must settle become `OQ-NN` entries under `## Open Questions`. If none survive, write the no-open-questions line

### Phase 3: Finalization

1. **Update References**: Update master PRDs and related documents to reference the new PRD
2. **Validate**: Review for clarity, completeness, and measurability
3. **Quality Check**: Ensure no implementation details, all requirements testable, all links valid
4. **Present for Review**: Show complete PRD to user, await approval before TDD creation

## Include in PRDs

| Element           | Purpose                                     | Example                                                    |
| ----------------- | ------------------------------------------- | ---------------------------------------------------------- |
| Problem Statement | WHY this exists                             | "Users struggle to track their hours worked"               |
| Goals             | SUCCESS looks like                          | "Reduce time-tracking errors by 50%"                       |
| Non-Goals         | What we're NOT doing                        | "Not replacing payroll system"                             |
| Target Users      | WHO we're building for                      | "Household employees and employers"                        |
| User Workflows    | HOW users accomplish tasks                  | Step-by-step scenarios with time estimates                 |
| Requirements      | Functional needs                            | "System must calculate overtime automatically"             |
| Success Metrics   | HOW we measure                              | "90% user adoption within 3 months"                        |
| Scope Boundaries  | IN/OUT of scope                             | "In: Time tracking. Out: Payroll processing"               |
| Constraints       | LIMITATIONS we accept                       | "Must work offline on mobile"                              |
| Decisions         | Judgment calls you made                     | "D-01: Single employer per employee in v1"                 |
| Open Questions    | Only what a human must decide (often empty) | "OQ-01: Is the 14-day trial a firm commercial commitment?" |

## Exclude from PRDs

| Element                | Why Exclude                | Where It Belongs |
| ---------------------- | -------------------------- | ---------------- |
| Implementation code    | PRDs define WHAT, not HOW  | TDD, Codebase    |
| Database schemas       | Technical detail           | TDD              |
| API specifications     | Technical detail           | TDD (API type)   |
| UI wireframes          | Unless high-level concepts | Design docs      |
| Architecture decisions | Technical detail           | TDD, ADRs        |
| Deployment plans       | Operations detail          | DevOps docs      |

## MCP Integration

- **Sequential MCP**: Structured analysis of requirements and systematic PRD construction
- **Context7 MCP**: Industry patterns, competitive analysis, best practices
- **Serena MCP**: Project memory for cross-PRD consistency and domain understanding

## Decision Triage Workflow

PRD creation follows a **two-step process**: the core document is written first, then a deep-analysis pass hunts for gaps. The pass **decides by default** and only escalates what a human genuinely has to settle.

### Step 1: Create PRD Core

Generate all PRD sections EXCEPT Decisions and Open Questions:

1. Problem Statement & Background
2. Goals and Non-Goals
3. Target Users & Personas
4. User Workflows (step-by-step scenarios)
5. Requirements (Functional, Non-Functional)
6. Success Metrics & KPIs
7. Scope (In/Out)
8. Constraints & Dependencies
9. Related Documents

Write the PRD to file with placeholder sections:

```markdown
## ✅ Decisions (Resolved)

*Generating via deep analysis...*

## Open Questions

*Generating via deep analysis...*
```

### Question Policy

**Default: decide.** The deep-analysis pass exists to find gaps, not to produce a list of questions. Once a gap is found, close it yourself whenever you can do so with confidence — a reviewer's attention is the scarcest resource in this process, and a PRD that arrives with five questions the author could have answered is worse than one that arrives with five recorded decisions. Classify every finding into one of three tiers:

| Tier                  | When                                                                                                                                                                                                         | Action                                                                                                                                               |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Decide-and-record** | A defensible answer follows from the problem statement, the codebase, sibling PRDs, or common product practice                                                                                               | Choose it, fold it into the relevant section, add a `### D-NN — title` entry to `## ✅ Decisions (Resolved)` with `**Choice.**` and `**Why.**` lines |
| **Ask**               | Only a human can settle it: business strategy, pricing, legal/compliance, budget, priority between competing stakeholders, brand or UX preference, or facts you cannot obtain (contracts, customer promises) | Write an `OQ-NN` entry under `## Open Questions` with a recommended option                                                                           |
| **Non-issue**         | Already specified, or the answer would not change what gets built                                                                                                                                            | Say nothing; do not manufacture questions                                                                                                            |

**Ask-tier test** — an Open Question is legitimate only when *all three* hold:

- Answering it changes what gets built, not just how the document reads
- You could not reach a confident answer from the available context
- The reviewer has information or authority you do not

**Irreversible-decision guardrail**: when a decision touches money, legal exposure, data retention or privacy, or a public commitment, ask even if you have a preference — and state that preference as the recommended option.

If no finding survives the Ask-tier test, `## Open Questions` contains exactly this line and nothing else:

```markdown
*No open questions — every design decision is recorded in ✅ Decisions (Resolved).*
```

Never pad the section. Three decisions and zero questions is a better outcome than two manufactured questions.

`--ask` overrides the default and surfaces decide-and-record items as Open Questions as well (each still carrying your recommended answer), for the rare PRD where the user wants to review every judgment call before TDD creation.

### Step 2: Deep Analysis and Triage (ultrathink)

After the PRD core is written, invoke Sequential MCP with `--ultrathink` depth to analyze the complete PRD, resolve what can be resolved, and escalate only what cannot.

**Analysis Focus Areas**:

| Area                | What to look for                                               | Usual tier        |
| ------------------- | -------------------------------------------------------------- | ----------------- |
| Problem Clarity     | Is the problem well-defined? Are we solving the right problem? | Decide-and-record |
| User Understanding  | Do we understand all user segments? Edge cases?                | Decide-and-record |
| Scope Boundaries    | What should explicitly be out of scope? Future phases?         | Decide-and-record |
| Success Measurement | Are metrics measurable? Do we have baselines?                  | Decide-and-record |
| Stakeholder Input   | Business decisions, policy clarifications needed?              | Ask               |
| Dependencies        | External dependencies? Integration requirements?               | Decide-and-record |
| Risk Factors        | What could prevent success? Mitigation strategies?             | Decide-and-record |

**Sequential MCP Prompt Template**:

```
Analyze this PRD for ambiguities, missing decisions, edge cases and scope
gaps that would affect TDD creation.

For EACH finding, first try to resolve it yourself:

- If a defensible answer follows from the problem statement, the
  codebase, sibling PRDs, or common product practice, DECIDE. Output it
  as a decision: id (D-01, D-02, ...), the choice, a one-line rationale,
  and the PRD section to update.
- Only if a human must settle it (business strategy, pricing, legal or
  compliance, budget, stakeholder priority, brand preference, or facts
  you cannot obtain) output it as an open question: id (OQ-01, ...),
  the question, why it matters, possible answers with trade-offs, and
  the recommended option.
- Drop everything else.

Expect zero or very few open questions. Do not manufacture questions
to fill the section.
```

**Output Format**:

```markdown
## ✅ Decisions (Resolved)

One heading per decision. IDs are referenced from the sections above.

### D-01 — Employer model

**Choice.** Single employer per employee in v1.

**Why.** Every persona in Target Users has one employer; multi-employer adds a join and split reports.

### D-02 — Offline behavior

**Choice.** Read-only cache, writes queue.

**Why.** Constraint "must work offline" + Workflow 3 only needs viewing; queued writes match FR-07.

## Open Questions

| ID    | Question                                                | Status |
| ----- | ------------------------------------------------------- | ------ |
| OQ-01 | Is the 14-day free trial a firm commercial commitment?  | Open   |

### OQ-01: Trial length commitment

**Question**: Sales has quoted a 14-day free trial in two enterprise proposals. Is that a firm commitment the PRD must honor, or can the trial length be tuned during beta?

**Why it matters**: A fixed trial length becomes a Constraint and rules out the conversion experiments in Success Metrics.

**Possible answers**:

- [ ] Firm 14 days — honors quotes; conversion experiments limited to in-trial messaging **(recommended if the proposals were signed)**
- [ ] Flexible 7–30 days — enables experiments; Sales must re-confirm with both prospects

**Status**: Open — needs commercial decision
```

When nothing survives the Ask-tier test:

```markdown
## Open Questions

*No open questions — every design decision is recorded in ✅ Decisions (Resolved).*
```

### Automatic Execution

Both steps run automatically with a single `/prd` command:

```
/prd [feature-name] [--type master|feature|api|integration]

Execution:
  Step 1: Generate PRD core sections → Write to file
  Step 2: Invoke Sequential MCP + ultrathink → Triage → Record decisions, append only real Open Questions
  Result: Complete PRD with decisions recorded and zero-or-few questions for the reviewer
```

## Outputs

- **Master PRD**: Product overview, vision, document hierarchy, cross-cutting requirements
- **Feature PRD**: Specific feature requirements, user workflows, success metrics
- **API PRD**: API product requirements, consumer needs, integration patterns
- **Integration PRD**: External system integration requirements, data flows
- **PRD Review Report**: Analysis of existing PRD for scope creep with lean suggestions
- **Consolidated PRD**: Tightened document with OQ answers applied and resolved questions removed

## Examples

### Master PRD

```
/prd time-tracking --type master

# Generates master PRD with product vision, scope, document hierarchy
```

### Feature PRD

```
/prd time-tracking-mobile --type feature

# Generates feature PRD with user workflows, requirements, success metrics
```

### Review Existing PRD

```
/prd --review @specs/prds/time-tracking/00-master.md

# Analyzes PRD for scope creep, missing requirements, suggests improvements
```

### Consolidate After OQ Discussion

```
/prd --consolidate @specs/prds/time-tracking/01-mobile.md

# Applies OQ answers from conversation, tightens document, verifies thresholds
```

## Post-Creation Workflow

After creating a PRD:

1. **Present for review**: Show the complete PRD to the user
2. **Await approval**: Do NOT start TDD creation until user approves
3. **Review decisions, resolve questions**: Skim `✅ Decisions (Resolved)` and override any you disagree with; answer every remaining OQ item before TDD creation
4. **Update if needed**: Incorporate user feedback into the PRD

**CRITICAL**: Creating a PRD is a specification phase, NOT a TDD trigger. Always read and present the PRD for review before any TDD creation begins.

## Consolidation Workflow

After Open Questions are answered in discussion, use `--consolidate` to apply answers and tighten the document.

### Usage

```
/prd --consolidate @specs/prds/product-feature.md
```

### Consolidation Actions

1. **Read Context**: Parse PRD and recent conversation for OQ answers
2. **Apply Answers**: Integrate resolved decisions into relevant sections
    - Update Goals with clarified success criteria
    - Add missing User Workflows for resolved edge cases
    - Adjust Requirements based on scope decisions
    - Update Success Metrics with agreed baselines
3. **Collapse resolved questions into Decisions entries** (do NOT leave answered questions in the verbose `### OQ-NN` Question / Why it matters / Possible answers / Status format):
    - Move every **resolved** question into `## ✅ Decisions (Resolved)` (extend the existing section from creation time; create it if absent) as one `### OQ-NN — title` heading each with `**Choice.**` and `**Why.**` lines. Keeping the `OQ-NN` / `FQ-NN` id in the heading means cross-references elsewhere in the doc still resolve.
    - **Delete** the verbose detail blocks of resolved questions (the Decisions entry is now the record).
    - Keep only genuinely **open** questions under `## Open Questions`, retaining their detail blocks. If none remain, write `*No open questions — every design decision is recorded in ✅ Decisions (Resolved).*`
    - Leave partner/business question sets (e.g. an `AT-*` "Questions for [partner]" section) in their own section.
    - Update any footer/summary line to reference "✅ Decisions (Resolved)" instead of listing resolved OQ ids.
4. **Tighten Document**:
    - Remove redundant prose
    - Consolidate overlapping sections
    - Ensure tables are scannable
5. **Verify Thresholds**: Check document stays under line limits
6. **Present Changes**: Show summary of modifications

### Before/After Example

**Before consolidation:**

```markdown
## Open Questions

| ID | Question | Status |
|----|----------|--------|
| OQ-01 | Support multiple employers? | Open |
| OQ-02 | Maximum hours per week? | Open |
```

**After consolidation** (user answered: single employer v1, 60 hours max):

```markdown
## Scope

| In Scope | Out of Scope |
|----------|--------------|
| Single employer per employee | Multiple employer support (v2) |  <!-- Applied from OQ-01 -->

## Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-05 | System enforces 60-hour weekly maximum | P1 |  <!-- Applied from OQ-02 -->

## ✅ Decisions (Resolved)

### OQ-01 — Employer multiplicity

**Choice.** Single employer per employee (v1).

**Why.** Simpler data model + UI; multi-employer deferred to v2.

### OQ-02 — Weekly hours cap

**Choice.** 60-hour weekly maximum.

**Why.** Legal compliance + payroll correctness.

## Open Questions

*No open questions — every design decision is recorded in ✅ Decisions (Resolved).*
```

### When to Consolidate

- After answering 3+ Open Questions in discussion
- Before creating TDDs from the PRD
- When PRD context feels bloated
- Before sharing PRD with stakeholders

## Complexity Management

### When to Split PRDs

If a PRD exceeds these thresholds, suggest splitting into multiple documents:

| Indicator       | Threshold             | Action                           |
| --------------- | --------------------- | -------------------------------- |
| User workflows  | > 8 workflows         | Split by user segment or journey |
| Requirements    | > 30 requirements     | Split by feature area            |
| Target users    | > 4 distinct personas | Split by persona                 |
| Document length | > 400 lines           | Review for bloat                 |
| Document length | > 1000 lines          | **Hard limit** - must split      |

### Master PRD Pattern

For complex products, create a document hierarchy:

1. **Master PRD** (`00-product-master.md`): Vision, scope, document hierarchy, cross-cutting requirements
2. **Child PRDs** by feature:
    - `01-product-feature-a.md` - First feature PRD
    - `02-product-feature-b.md` - Second feature PRD
    - `03-product-api.md` - API requirements (if applicable)
    - `04-product-integration.md` - Integration requirements (if applicable)

Each child PRD links back to the master and to sibling PRDs.

## Quality Checklist

Before finalizing a PRD, verify:

- [ ] Problem statement is clear and compelling
- [ ] Goals are specific, measurable, achievable
- [ ] Non-goals explicitly define what we're NOT doing
- [ ] Target users are well-defined with clear needs
- [ ] User workflows are step-by-step scenarios with time estimates
- [ ] Requirements are testable and prioritized
- [ ] Success metrics have baselines and targets
- [ ] Scope boundaries are crystal clear
- [ ] No implementation details or code
- [ ] No technical architecture decisions
- [ ] Decision triage run via Sequential MCP + ultrathink (Phase 2 complete)
- [ ] Every decide-and-record finding is folded into its section and logged in `✅ Decisions (Resolved)` as a `### D-NN — title` entry with `**Choice.**` and `**Why.**` lines (no table)
- [ ] Every OQ passes the Ask-tier test (changes what gets built, not answerable with confidence, reviewer holds the authority) and carries a recommended option
- [ ] Open Questions is either genuine questions or exactly the no-open-questions line — never padded
- [ ] PRD presented for user review before TDD creation
- [ ] Large PRDs split with master document pattern

## Boundaries

**Will:**

- **Always ask clarifying questions before generating** (unless `--no-questions`)
- Generate lean PRDs focused on problems, goals, and requirements
- Use tables and structured formats over prose
- Create measurable success criteria
- Reference existing patterns in the codebase
- Identify and suggest removal of scope creep in reviews
- Consolidate PRDs by applying OQ answers and tightening documents

**Will Not:**

- Include implementation code or technical details
- Write database schemas or API specifications
- Make architecture decisions
- Create UI wireframes or detailed designs
- Generate TDDs (separate skill/phase)

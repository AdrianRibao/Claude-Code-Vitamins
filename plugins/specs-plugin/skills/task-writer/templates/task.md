---
id: NNN
title: [Short, specific title]
status: todo
priority: [P0 | P1 | P2 | P3]
size: [Small | Medium | Large]
updated: [YYYY-MM-DD]
---

# NNN — [Short, specific title]

[One paragraph: where this stands and the shape of the work. Update it as the task evolves. For a Large stub: "Needs a PRD before any estimate is trustworthy."]

## Why

\[Dated evidence of the gap. Every claim carries `file:line`, a command with its output, or an observation in a named environment. Tables when comparing layers or environments. Link the bug this grew out of, if any.\]

## Scope

1. [WHAT changes — file/function/setting; signatures or ≤5-line snippets only]
2. [WHAT changes]
3. [WHAT changes]

### Do NOT change

- [adjacent behavior that must stay identical]

### Out of scope

- [tempting nearby work this task does not cover]

## Constraints

[Only when a project rule bounds the approach. Quote it and cite where it lives. Otherwise omit.]

## Sequencing

\[Only when dependencies exist: depends on / blocks / why now or why not yet. A `blocked` task names its blocker here. Otherwise omit.\]

## Done when

| #    | Context   | Action   | Expected result |
| ---- | --------- | -------- | --------------- |
| AC-1 | [context] | [action] | [observable]    |
| AC-2 | [context] | [action] | [observable]    |

### Checklist

- [ ] AC-1: [one-line]
- [ ] AC-2: [one-line]

## Verification

- **Tests**: [unit for changed logic; E2E for a user-facing flow; one unchanged-behavior test per Do-NOT-change item — name files/corpora]
- **Confirm live**: [the command / URL / dashboard / host check that proves the outcome in the running system]

## Decisions

[Omit when nothing is decided and nothing is open.]

### D-01 — [question you resolved]

**Decided YYYY-MM-DD.** [answer]

**Why.** [rationale]

### OD-01 — [genuine fork] (open)

**Choices.** A) [choice: consequence]; B) [choice: consequence].

**Recommended.** [A|B] — [why]

## Related

[Only when there are links: PRD/TDD, bugs filed, runbooks, sibling tasks. Otherwise omit.]

## Next step

[The single concrete action that starts the work.]

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
- **Code review**: \[run the `code-review` skill on the final diff once tests are green, before the live check and before commit; on ship, the date, level, and each finding as fixed / filed / dismissed with a reason. `n/a — no code changed` for purely operational work\]
- **QA**: \[UI changes: `/qa` runs the QA plan below on local after the code review (staging after deploy, when the project has one). Otherwise `n/a — no UI change` — and delete the QA plan section\]
- **Confirm live**: [the command / URL / dashboard / host check that proves the outcome in the running system]

## QA plan

\[UI changes only. Conventions: the `qa-runner` style guide. Say WHAT to check, never an agent-browser command.\]

### Environments

| Env     | Base URL                  | Start / deploy            | Seed data          |
| ------- | ------------------------- | ------------------------- | ------------------ |
| local   | `http://localhost:[port]` | [command that starts it]  | [script / fixture] |
| staging | `[url]`                   | [how a change gets there] | [accounts / data]  |

\[Omit the staging row when the project has no staging.\]

### Logins

| Role   | local             | staging             |
| ------ | ----------------- | ------------------- |
| [role] | `[profile]-local` | `[profile]-staging` |

\[One agent-browser vault profile per role and environment — a profile stores a single login URL. The user saves profiles in the vault in their own terminal; never a credential or a command here. `anonymous` when no login is needed. Drop the staging column when the project has no staging.\]

### Checks

| #    | Covers | Check                   | Capability | Passes when            | Evidence   |
| ---- | ------ | ----------------------- | ---------- | ---------------------- | ---------- |
| QA-1 | AC-1   | [role] [does the thing] | `ui-flow`  | [observable, specific] | screenshot |

### Exploratory charter

[Changed screens, their neighbours, the risks to probe.]

\[`/qa` writes the `### QA record` here after each run.\]

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

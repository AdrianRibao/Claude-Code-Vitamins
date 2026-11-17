# Bug NNN — \[short description with `backticked` identifiers\] ([parenthetical scope])

- **Status:** open
- **Severity:** [Critical | Major | Minor | Trivial] — [one-line impact]
- **Reported:** [YYYY-MM-DD] ([reporter / confirmed by])
- **Area:** [subsystem] — \[`key/file.ext`\]
- **Repo:** \[`repo-name`\] ([layer]). [Note sibling repos and whether they need changes.]
- **Related:** [ticket id; branch; wiki page; PR/commit; sibling bug spec]

______________________________________________________________________

## Summary

[One paragraph: what is broken, who is affected, and a one-line scope statement that names the owning layer and pre-empts creep.]

## Symptom

[Concrete, reproducible observed behavior. Prefer real IDs/URLs and before/after blocks over prose.]

```
[observed (buggy) output]
```

Expected: [one line].

## Reproduction

- **Environment:** [URL / branch / build / account+role]
- **Steps:**
    1. [step]
    2. [step]
    3. [observe the bug]

[Seed/restore script if needed. If the bug cannot be reproduced, write "Not reproducible — proceeding by hypothesis:" and state the hypothesis.]

## Root cause

\[The causal mechanism in the code at `file:line`, with provenance (how/where verified, dated). NOT a restatement of the symptom — name the actual defect.\]

## Expected behavior

[The correct behavior. Use numbered rules when non-trivial so the fix and the acceptance criteria can reference them.]

- Rule 1 — [condition] -> [result]
- Rule 2 — [condition] -> [result]

## The fix

[WHAT changes: file/function and the behavior delta. Signatures or snippets only, ≤5 lines. The diff itself belongs in the PR.]

### Do NOT change

- [adjacent behavior that must stay identical]

### Out of scope

- [tempting nearby work this bug does not cover]

## Blast radius

[What the change could break, and what must remain unchanged. Each item maps to an unchanged-behavior test below.]

## Verification

### Regression test

\[The one test that captures this bug. State what it asserts and WHY it fails on the buggy code. With `--fix`, paste RED output then GREEN output.\]

### Unit & E2E tests

- [unit tests for the changed logic / validation]
- [E2E test for the user-facing flow, when the bug was user-visible]

### Unchanged-behavior tests

- [one per blast-radius item]

### Code review

\[Run the `code-review` skill on the final diff (fix + tests) once the suite is green, before the app confirmation and before commit. Record the date and level, then each finding as fixed (tests re-run) / filed (`/bugfix` or `/task`, linked) / dismissed (one-line reason).\]

### How to demo

1. [reproduce]
2. [confirm the fix in the running app — automated by the QA plan for UI bugs] — ✅ [expected] / ❌ [failure signal]

\[API or backend-only bug: write `QA: n/a — no UI change` here and delete the QA plan section below.\]

## QA plan

\[UI bugs only. Conventions: the `qa-runner` style guide. Say WHAT to check, never an agent-browser command. Run with `/qa @<this doc>` after the code review; staging with `--env staging` after deploy.\]

### Environments

| Env     | Base URL                  | Start / deploy            | Seed data         |
| ------- | ------------------------- | ------------------------- | ----------------- |
| local   | `http://localhost:[port]` | [command that starts it]  | [repro seed]      |
| staging | `[url]`                   | [how a change gets there] | [accounts / data] |

\[Omit the staging row when the project has no staging.\]

### Logins

| Role   | local             | staging             |
| ------ | ----------------- | ------------------- |
| [role] | `[profile]-local` | `[profile]-staging` |

\[One agent-browser vault profile per role and environment — a profile stores a single login URL. The user saves profiles in the vault in their own terminal; never a credential or a command here. `anonymous` when no login is needed. Drop the staging column when the project has no staging.\]

### Checks

| #    | Covers | Check                                 | Capability | Passes when                         | Evidence   |
| ---- | ------ | ------------------------------------- | ---------- | ----------------------------------- | ---------- |
| QA-1 | AC-1   | [role] replays the reproduction steps | `ui-flow`  | [the symptom is gone: what is seen] | screenshot |
| QA-2 | AC-N   | [blast-radius item visible in the UI] | `ui-flow`  | [unchanged behavior, observable]    | screenshot |

### Exploratory charter

[Changed screens, their neighbours, the risks to probe.]

\[`/qa` writes the `### QA record` here after each run.\]

## Acceptance criteria

| #    | Context   | Action   | Expected result |
| ---- | --------- | -------- | --------------- |
| AC-1 | [context] | [action] | [expected]      |
| AC-2 | [context] | [action] | [expected]      |

### Checklist

- [ ] AC-1: [one-line]
- [ ] AC-2: [one-line]
- [ ] Regression test fails on the buggy code and passes on the fix
- [ ] Code review run on the final diff; every finding fixed, filed, or dismissed

## Open decisions

[Only genuine blockers. Omit this section entirely if there are none.]

- **OD-01:** [question] — choices: [A] vs [B]; consequence: [...]

## ✅ Decisions (Resolved)

\[Only after an open decision is answered (`--consolidate`). Omit otherwise. One heading per decision.\]

### OD-01 — [question]

**Choice.** [answer]

**Why.** [rationale, with the concrete trade-off]

## Deploy & rollback

[Only when deploy is non-trivial. Otherwise omit.]

## When fixed

Once tests are green, the diff is code-reviewed, and the issue is confirmed gone in the running app (a passing local QA record for UI bugs): set **Status** to `fixed — PR #NNN, commit <hash> (one-line)` and move this file to `specs/bugs/fixed/`.

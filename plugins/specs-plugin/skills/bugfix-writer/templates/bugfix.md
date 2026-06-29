# Bug NNN — \[short description with `backticked` identifiers\] ([parenthetical scope])

- **Status:** open
- **Severity:** [Critical | Major | Minor | Trivial] — [one-line impact]
- **Reported:** [YYYY-MM-DD] ([reporter / confirmed by])
- **Area:** [subsystem] — \[`key/file.ext`\]
- **Repo:** \[`repo-name`\] ([layer]). [Note sibling repos and whether they need changes.]
- **Related:** [ticket id; wiki page; PR/commit; sibling bug spec]

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

### Unchanged-behavior tests

- [one per blast-radius item]

### How to demo

1. [reproduce]
2. [confirm the fix] — ✅ [expected] / ❌ [failure signal]

## Acceptance criteria

| #    | Context   | Action   | Expected result |
| ---- | --------- | -------- | --------------- |
| AC-1 | [context] | [action] | [expected]      |
| AC-2 | [context] | [action] | [expected]      |

### Checklist

- [ ] AC-1: [one-line]
- [ ] AC-2: [one-line]
- [ ] Regression test fails on the buggy code and passes on the fix

## Open decisions

[Only genuine blockers. Omit this section entirely if there are none.]

- **OD-01:** [question] — choices: [A] vs [B]; consequence: [...]

## Deploy & rollback

[Only when deploy is non-trivial. Otherwise omit.]

## When fixed

Set **Status** to `fixed — PR #NNN, commit <hash> (one-line)` and move this file to `specs/bugs/fixed/`.

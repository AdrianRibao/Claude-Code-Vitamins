# Bugfix Style Guide

Conventions for bug docs produced by the `bugfix-writer` skill. The skill's SKILL.md is the overview; this file is the detailed reference for *how* each section is written.

## Contents

- Document structure and section order
- Metadata block
- Severity scale
- Title and numbering
- Summary
- Symptom
- Reproduction
- Root cause (symptom vs cause)
- Expected behavior
- The fix (Do NOT change, Out of scope)
- Blast radius
- Verification and the red->green regression test
- Acceptance criteria (hybrid format)
- Assumptions vs Open decisions
- Anti-patterns

## Document Structure and Section Order

1. H1 title
2. Metadata block (bold-key bullets) + horizontal rule
3. `## Summary`
4. `## Symptom`
5. `## Reproduction`
6. `## Root cause`
7. `## Expected behavior`
8. `## The fix` (with `### Do NOT change` and `### Out of scope`)
9. `## Blast radius`
10. `## Verification`
11. `## Acceptance criteria`
12. `## Open decisions` (only if a genuine blocker exists)
13. `## Deploy & rollback` (only when deploy is non-trivial)
14. `## When fixed` (process footer)

Separate major blocks with a horizontal rule (`______…`) to match the existing corpus.

## Metadata Block

Use a **bold-key bullet list** directly under the H1 (not YAML), terminated by a horizontal rule. Six keys, always in this order:

```markdown
- **Status:** open
- **Severity:** Major — customer-facing dropdown labels are ambiguous
- **Reported:** 2026-06-23 (PO: Pau; confirmed by CEO)
- **Area:** `catalogue` — variant dropdown label builder (`variant-label.js`)
- **Repo:** `bloowatch-web` (frontend). Backend variant data is already correct.
- **Related:** Jira OP-6979; Confluence "P11"; sibling `specs/bugs/fixed/001-...md`

______________________________________________________________________
```

| Key          | Holds                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| **Status**   | Lifecycle: `open` -> `in progress` -> `fixed — PR #NNN, commit <hash> (one-line)` |
| **Severity** | One of the four levels below + a short impact clause                              |
| **Reported** | ISO date + who reported / confirmed                                               |
| **Area**     | Subsystem + the key file(s) in backticks                                          |
| **Repo**     | Which repo owns the fix; note sibling repos and whether they need changes         |
| **Related**  | Tickets, wiki pages, PRs/commits, sibling bug specs, runbooks                     |

## Severity Scale

Severity is always a metadata field (never folded into Status). Assign by impact, not by how hard the fix is.

| Severity     | Definition                                                                                                  | Examples                                              |
| ------------ | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| **Critical** | Data loss/corruption, security exposure, money handled wrong, or a core workflow fully down — no workaround | Auth bypass; payments double-charged; records deleted |
| **Major**    | A core workflow is broken or visibly wrong for many users; the workaround is painful                        | Customer-facing labels wrong; a key report miscounts  |
| **Minor**    | Non-core or cosmetic with an easy workaround                                                                | A tooltip typo; a rare edge case with a known dodge   |
| **Trivial**  | Negligible impact                                                                                           | Spacing; log noise                                    |

Critical and Major bugs touching data, migrations, money, or security trip the **irreversible-fix guardrail** — confirm the fix approach before proceeding.

## Title and Numbering

- H1: `# Bug NNN — <description with backticked identifiers> (<parenthetical scope>)`
- `NNN`: zero-padded, sequential, **per-repo**. Scan `specs/bugs/` and `specs/bugs/fixed/` for the highest number and add one.
- Slug in the filename is kebab-case; the H1 description is prose.

Example: `# Bug 001 — Variant dropdown labels drop the start hour and fall back to meaningless (1)(2)(3) numbering (eCommerce + POS)`

## Summary

One short paragraph: what is broken, who is affected, and a one-line **scope statement** that pre-empts creep. State the layer that owns the fix.

> When variants share a date range, the dropdown shows ambiguous `(1)(2)(3)` labels and some variants lose their start hour entirely, so customers cannot tell variants apart. **UI-only fix to the label string; no change to variant data or dedup logic.**

## Symptom

Concrete and reproducible. Prefer real IDs, URLs, and before/after blocks over prose. Show *observed* vs *expected*.

````markdown
Observed (buggy):

```
06-07-2026 → 10-07-2026 (1)
06-07-2026 → 10-07-2026 (2)
```

Expected: each entry disambiguated by start time / Option letter.
````

(Arrows are fine *inside fenced code blocks* that reproduce literal output; in prose and tables use `->`.)

## Reproduction

Deterministic steps so anyone can see the bug. Include, as needed:

- Environment (URL, branch, build, account/role)
- Seed script (e.g. a shell/ORM snippet that creates the failing data)
- Step-by-step navigation
- A restore/teardown note

If the bug cannot be reproduced, write `Not reproducible — proceeding by hypothesis:` and state the hypothesis explicitly. Never skip this section.

## Root Cause (symptom vs cause)

The single most important discipline. The root cause is the **causal mechanism** in the code, located at `file:line`, with provenance (how/where verified, dated).

✅ **Cause**: "`variant-label.js:42` builds the label from a positional index when two variants share a date range; it never reads `start_time`. (Verified on staging, school 1, 2026-06-23.)"

❌ **Symptom restated as cause**: "The labels show `(1)(2)(3)` instead of the time." (That is the symptom — it names no defect.)

A root cause that does not name a line of code (or a precise data/config defect) fails the gate.

## Expected Behavior

State the correct behavior. When non-trivial, use **numbered rules** so the fix and the acceptance criteria can reference them.

```markdown
- Rule 1 — shared date range AND start time -> `DD-MM-YYYY → DD-MM-YYYY (HH:mm Option X)`
- Rule 2 — shared date range, different times -> `(HH:mm)`, no Option label
- Rule 3 — only variant on its range -> `(HH:mm)`, no Option label
```

## The Fix (Do NOT change, Out of scope)

Specify WHAT changes — file/function, the behavior delta, and signatures or snippets **no longer than 5 lines**. Never paste the full implementation; the diff lives in the PR.

Always include two guardrail subsections:

- `### Do NOT change` — adjacent behavior that must stay byte-for-byte identical (e.g. "stock / places-left indicators", "dedup logic", "sort order").
- `### Out of scope` — tempting nearby work that this bug does not cover.

## Blast Radius

Enumerate what the change could break and what must remain unchanged. Each item here should map to an unchanged-behavior test in Verification. This is where you catch "the fix to A silently broke B".

## Verification and the red->green Regression Test

Three parts, always:

1. **Regression test** — the one test that *captures this bug*. State what it asserts and **why it fails on the current (buggy) code**. With `--fix`, paste the RED failure output, then the GREEN pass output.
2. **Unchanged-behavior tests** — one per blast-radius item, asserting the "Do NOT change" behavior still holds.
3. **Manual / how to demo** — numbered steps that reproduce then confirm, with ✅ expected and ❌ failure signals, mapped to acceptance criteria.

> A green regression test that was never shown to fail on the buggy code proves nothing. Demonstrating RED first is the load-bearing step.

## Acceptance Criteria (hybrid format)

Under one `## Acceptance criteria` heading, provide **both**:

A readable table for humans/QA:

```markdown
| #    | Context                              | Action                | Expected result                                  |
| ---- | ------------------------------------ | --------------------- | ------------------------------------------------ |
| AC-1 | 3 variants, same date + start time   | Open the dropdown     | Entries show `(HH:mm Option A/B/C)` in creation order |
| AC-2 | 3 variants, same date, diff times    | Open the dropdown     | Entries show `(HH:mm)`, no Option labels         |
```

And a mirrored checklist that `ac-checker` can parse and auto-mark:

```markdown
### Checklist

- [ ] AC-1: Parallel variants show `(HH:mm Option A/B/C)` in creation order
- [ ] AC-2: Same-range different-time variants show `(HH:mm)`, no Option label
- [ ] Regression test fails on the buggy code and passes on the fix
```

The final regression-test checkbox is **mandatory** in every bug doc.

## Assumptions vs Open Decisions

- **Assumption** (default-and-document): a minor uncertainty you resolved with a sensible default. Record it inline where it applies: *"Assumed Option letters re-number on delete (creation-order) — flag if stability is required."* Does not block the fix.
- **Open decision** (blocking): a genuine fork where a *correct* fix is impossible without an answer. Goes under `## Open decisions` with an `OD-NN` id, the choices, and their consequences. Omit the section entirely when there are none — do not manufacture questions.

## Anti-Patterns

| Anti-pattern                         | Why it fails                                                         |
| ------------------------------------ | -------------------------------------------------------------------- |
| Root cause restates the symptom      | Produces a band-aid; the real defect ships again under a new ticket  |
| Green test with no demonstrated RED  | The test may pass for reasons unrelated to the fix                   |
| "While I'm here" cleanups in the fix | Dilutes the regression test and widens blast radius                  |
| Manufactured Open Questions          | Stalls a ready fix on a teammate's review for no reason              |
| Marking `fixed` without evidence     | Status must follow a passing test, a merged PR, or explicit sign-off |

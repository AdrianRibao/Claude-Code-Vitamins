# Bug 001 — Variant dropdown labels drop the start hour and fall back to meaningless `(1)(2)(3)` numbering (eCommerce + POS)

- **Status:** fixed — PR #517, commit `9c1a4f0` (group-based labels with Option A/B/C; start hour always shown)
- **Severity:** Major — customer-facing; bookings abandoned or made blindly, driving refunds and support load
- **Reported:** 2026-06-23 (PO: Pau; confirmed by CEO)
- **Area:** `catalogue` — variant dropdown label builder (`src/widgets/booking/variant-label.js`)
- **Repo:** `bloowatch-web` (frontend). Backend variant data is already correct — P2 ships separate selectable entries; only the label string is wrong.
- **Related:** Jira OP-6979; Confluence "P11"; parent OP-6970; branch `fix/OP-6979-variant-dropdown-labels`

______________________________________________________________________

## Summary

When multiple variants share the same date range, the eCommerce and POS booking dropdowns show ambiguous labels — sequential `(1)`, `(2)`, `(3)` appended to identical date strings — so customers cannot tell variants apart. Separately, some variants on distinct dates render without their start hour at all. **UI-only fix to the dropdown label string; no change to variant data, dedup logic (P2 owns that), or stock/availability indicators.**

## Symptom

Reproduced on staging, product "Curso de vela" (school 1), 2026-06-23.

Observed (buggy):

```
25/05/2026 → 29/05/2026          ← no start hour
01/06/2026 → 05/06/2026 (1)
01/06/2026 → 05/06/2026 (2)
01/06/2026 → 05/06/2026 (3)
08/06/2026 → 12/06/2026          ← no start hour
```

Expected: every entry shows its start hour, and genuinely parallel variants are disambiguated by `Option A/B/C` rather than a positional index.

## Reproduction

- **Environment:** staging `widget` build, branch `main`, product id `4821`
- **Steps:**
    1. Seed a product with the variants in the demo below (one standalone, three parallel at the same date+time, two more standalone).
    2. Open the product in the customer-facing eCommerce widget.
    3. Open the variant dropdown — observe `(1)(2)(3)` and missing start hours.

Seed (Django shell):

```python
make_variants(product_id=4821, specs=[
    ("2026-05-25", "2026-05-29", "09:00"),
    ("2026-06-01", "2026-06-05", "11:30"),  # parallel x3
])
```

## Root cause

`variant-label.js:42` builds the label from the variant's **positional index** within the date-collision group (`(${i + 1})`) and only appends the start hour on the non-colliding branch — so colliding variants lose the hour and gain a meaningless number. The builder never groups by `(date_start, date_end, start_time)`. (Verified by code walkthrough + staging render, 2026-06-23.)

## Expected behavior

The label rule is evaluated per `(date_range, start_time)` group:

- Rule 1 — shared date range AND start time -> `DD-MM-YYYY → DD-MM-YYYY (HH:mm Option X)`, `X` in creation order (oldest = A)
- Rule 2 — shared date range, different start times -> `(HH:mm)`, no Option label
- Rule 3 — only variant on its range -> `(HH:mm)`, no Option label
- Edge — `start_time` null -> `DD-MM-YYYY → DD-MM-YYYY` (no parentheses); log for data review

## The fix

In `variant-label.js`, replace the positional-index fallback with a group key of `(date_start, date_end, start_time)`. Assign `Option X` only when a group has 2+ members; always render `(HH:mm)` when `start_time` is present.

```js
const key = `${v.date_start}|${v.date_end}|${v.start_time}`;
// group size > 1 → append ` Option ${letter(creationRank)}`; always show (HH:mm)
```

### Do NOT change

- Stock, availability, and "places left" / sold-out indicators (rendered after the label, unchanged)
- Dedup logic (P2 owns it — variants are already separate entries)
- Sort order of variants in the dropdown

### Out of scope

- Re-numbering stability when variants are added/deleted (creation-order is fine for MVP)
- Instructor name / meeting spot in the label — `Option A/B/C` is the chosen disambiguator

## Blast radius

- POS booking dropdown shares the component -> must get the same labels (covered by AC-5).
- Legacy Ember minisite has a parallel label path -> confirmed in scope; same rules applied.
- `start_time = null` legacy variants could crash the builder -> guarded (AC-6).

## Verification

### Regression test

`variant-label.test.js`: render the seeded product's dropdown and assert the label strings. **Fails on the buggy code** — it produces `(1)(2)(3)` and omits the hour on standalone variants.

```
RED:  expected "01-06-2026 → 05-06-2026 (11:30 Option A)" but got "01-06-2026 → 05-06-2026 (1)"
GREEN: 8 passing (eCommerce + POS label cases)
```

### Unchanged-behavior tests

- Stock / "places left" string still appended verbatim after the label.
- Dropdown sort order identical to pre-fix snapshot.

### Code review

`/code-review` on the final diff, 2026-06-24, default level — 2 findings:

- Fixed: `letter()` overflowed past `Z` for a 27th parallel variant -> wraps to `AA`; unit case added, suite re-run green.
- Dismissed: "group key ignores `date_end`" — it does not; the key is `(date_start, date_end, start_time)`.

### How to demo

1. Open the seeded product in the eCommerce widget; open the dropdown.
2. ✅ Each entry shows `(HH:mm)`; parallel variants show `Option A/B/C` in creation order.
3. ❌ Any `(1)/(2)/(3)`, any missing hour, or any Option label on a standalone variant.

## QA plan

### Environments

| Env     | Base URL                                  | Start / deploy                          | Seed data                                             |
| ------- | ----------------------------------------- | --------------------------------------- | ----------------------------------------------------- |
| local   | `http://localhost:4200/widget/4821`       | `npm run dev` (widget) + API on `:8000` | `make_variants(product_id=4821, …)` from Reproduction |
| staging | `https://staging.example.com/widget/4821` | PR -> CircleCI -> `staging_deploy_hold` | Same seed on product `4821`                           |

### Logins

| Role     | local                | staging                |
| -------- | -------------------- | ---------------------- |
| customer | `anonymous`          | `anonymous`            |
| staff    | `qa-pos-staff-local` | `qa-pos-staff-staging` |

### Checks

| #    | Covers | Check                                                       | Capability                 | Passes when                                                              | Evidence                |
| ---- | ------ | ----------------------------------------------------------- | -------------------------- | ------------------------------------------------------------------------ | ----------------------- |
| QA-1 | AC-1   | Customer replays the reproduction: open the dropdown        | `ui-flow`                  | The three parallel entries read `(11:30 Option A/B/C)` in creation order | screenshot              |
| QA-2 | AC-2   | Customer opens a same-range, different-time product         | `ui-flow`                  | `(09:00)`, `(11:30)`, `(14:00)`; no Option label                         | screenshot              |
| QA-3 | AC-5   | Staff opens the same product in POS                         | `ui-flow`                  | Labels identical to QA-1                                                 | screenshot              |
| QA-4 | AC-6   | Customer opens a product with a `start_time = null` variant | `ui-flow`, `console-clean` | Date range only, no parentheses; zero page errors                        | screenshot, console log |
| QA-5 | —      | Stock and "places left" still follow each label             | `ui-flow`                  | Same strings as the pre-fix snapshot                                     | screenshot              |

### Exploratory charter

Variant dropdown in the eCommerce widget and POS. Probe: 1, 2 and 4+ parallel variants, sold-out variants, long product names, mobile width.

### QA record

**local · 2026-06-24 · `http://localhost:4200/widget/4821` · working tree on `fix/OP-6979-variant-dropdown-labels` · agent-browser 0.43.0**

| #    | Result | Observed                                                    | Evidence                       |
| ---- | ------ | ----------------------------------------------------------- | ------------------------------ |
| QA-1 | pass   | `01-06-2026 → 05-06-2026 (11:30 Option A)`, `B`, `C`        | `qa-1.png`                     |
| QA-2 | pass   | `(09:00)`, `(11:30)`, `(14:00)`, no Option labels           | `qa-2.png`                     |
| QA-3 | pass   | POS labels match QA-1 exactly                               | `qa-3.png`                     |
| QA-4 | pass   | `25-05-2026 → 29-05-2026`; 0 page errors; variant id logged | `qa-4.png`, `qa-4-console.txt` |
| QA-5 | pass   | "3 places left" / "Sold out" unchanged after each label     | `qa-5.png`                     |

**Exploratory**: 1 finding — fixed: the dropdown lost keyboard focus after choosing a variant (pre-existing; `variant-select.js:57`, tests + review re-run, re-run clean).

**Evidence**: `specs/qa/runs/001-variant-dropdown-labels/2026-06-24-local/` (git-ignored)

**staging · 2026-06-25 · `https://staging.example.com/widget/4821` · build `9c1a4f0` (confirmed in the widget footer) · agent-browser 0.43.0**

| #    | Result | Observed                                          | Evidence   |
| ---- | ------ | ------------------------------------------------- | ---------- |
| QA-1 | pass   | `(11:30 Option A/B/C)` in creation order          | `qa-1.png` |
| QA-2 | pass   | `(09:00)`, `(11:30)`, `(14:00)`, no Option labels | `qa-2.png` |
| QA-3 | pass   | POS labels match QA-1                             | `qa-3.png` |
| QA-4 | pass   | Date range only; 0 page errors                    | `qa-4.png` |
| QA-5 | pass   | Stock strings unchanged                           | `qa-5.png` |

**Evidence**: `specs/qa/runs/001-variant-dropdown-labels/2026-06-25-staging/` (git-ignored)

## Acceptance criteria

| #    | Context                                            | Action                  | Expected result                                      |
| ---- | -------------------------------------------------- | ----------------------- | ---------------------------------------------------- |
| AC-1 | 3 variants, same date range + same start time      | Open eCommerce dropdown | `(11:30 Option A/B/C)` in creation order             |
| AC-2 | 3 variants, same date range, different start times | Open eCommerce dropdown | `(09:00)`, `(11:30)`, `(14:00)`; no Option labels    |
| AC-3 | 1 variant on its date range                        | Open eCommerce dropdown | `(HH:mm)` shown; no Option label                     |
| AC-5 | Same as AC-1                                       | Open POS dropdown       | Identical label format to eCommerce                  |
| AC-6 | A variant with `start_time = null`                 | Open dropdown           | Renders date range only, no crash; variant id logged |

### Checklist

- [x] AC-1: Parallel variants show `(HH:mm Option A/B/C)` in creation order
- [x] AC-2: Same-range different-time variants show `(HH:mm)`, no Option label
- [x] AC-3: Standalone variant shows `(HH:mm)`, no Option label
- [x] AC-5: POS dropdown matches eCommerce label format
- [x] AC-6: Null `start_time` renders safely and is logged
- [x] Regression test fails on the buggy code and passes on the fix
- [x] Code review run on the final diff; every finding fixed, filed, or dismissed

## When fixed

Status set to `fixed — PR #517, commit 9c1a4f0`; this file moved to `specs/bugs/fixed/`.

# Bug 003 — Category `online_product_count` ignores availability periods, over-counting products that are not currently sellable (catalogue API)

- **Status:** open
- **Severity:** Major — customer-facing; category cards advertise more bookable products than exist, eroding trust
- **Reported:** 2026-06-18 (PO: Pau; reproduced against production API)
- **Area:** `catalogue` — category serializer (`catalogue/api/serializers.py`), count query (`catalogue/queries.py`)
- **Repo:** `bloowatch-api` (backend). The frontend renders whatever the API returns — no change needed there.
- **Related:** sibling `specs/bugs/fixed/001-category-online-product-count-product-class.md`; runbook `specs/claudedocs/availability-periods.md`

______________________________________________________________________

## Summary

A category's `online_product_count` counts every product with `online=true`, even when the product has **no active availability period** and therefore cannot be booked. Category cards over-state how many products are bookable. **Backend-only fix to the count query; no change to the product model, the availability model, or the frontend.**

## Symptom

Reproduced against the production API, school 1, 2026-06-18.

```
GET /api/categories/?school=1
→ {"id": 12, "name": "Vela", "online_product_count": 9}
```

But only 7 of those products have an active availability period today; 2 are `online=true` with all availability windows in the past. PO: *"esta escuela tiene 9 productos asignados … pero solo 7 que son online y reservables."*

Expected: `online_product_count` = 7.

## Reproduction

- **Environment:** production API (read-only) + local checkout of `main`
- **Steps:**
    1. Seed a category with 3 products: 1 online with a current window, 1 online with only past windows, 1 offline.
    2. Call the count query / hit `GET /api/categories/`.
    3. Observe the count = 2 (online) instead of 1 (online AND currently available).

Seed (Django shell):

```python
make_products(category=cat, specs=[
    ("online+current", True, "2026-06-01", "2026-12-01"),
    ("online+past",    True, "2025-01-01", "2025-03-01"),
    ("offline",        False, None, None),
])
```

## Root cause

`catalogue/queries.py:58` builds the count with `Q(online=True)` only. It never joins to `AvailabilityPeriod` nor filters on a period overlapping `today`, so products whose windows have all elapsed are still counted. (Verified by ORM walkthrough + the production response above, 2026-06-18.)

| Product        | `online` | Active period today | Counted now | Should count |
| -------------- | -------- | ------------------- | ----------- | ------------ |
| online+current | ✅       | ✅                  | ✅          | ✅           |
| online+past    | ✅       | ❌                  | ✅ (bug)    | ❌           |
| offline        | ❌       | —                   | ❌          | ❌           |

## Expected behavior

- Rule 1 — count a product only when `online=True` AND it has an availability period overlapping `today`.
- Rule 2 — products with only past or only future periods are excluded.

## The fix

In `catalogue/queries.py`, add an availability filter to the existing count query. WHAT changes: the `Q` filter gains a current-period condition.

```python
today = date.today()
Q(online=True) & Q(availability_periods__start__lte=today,
                   availability_periods__end__gte=today)
# add .distinct() so the join does not double-count multi-window products
```

### Do NOT change

- The `Product` or `AvailabilityPeriod` models or migrations
- Any other consumer of `online=True` (listing, search) — only the count query changes
- The category serializer's response shape

### Out of scope

- Timezone handling of period boundaries (tracked separately; assume school-local date)
- Caching of category counts

## Blast radius

- The join can double-count products with multiple current windows -> `.distinct()` required (covered by the multi-window test).
- Other queries that reuse the same helper -> confirm none share this code path before editing.

## Verification

### Regression test

`test/catalogue/test_category_counts.py`: seed the three-product category above and assert `online_product_count == 1`. **Will fail on current code** (returns 2) because the past-window product is still counted.

Planned (spec-only — run with `--fix` to capture live output):

```
RED expectation:  assert 1 == online_product_count  →  AssertionError: 2 != 1
```

### Unchanged-behavior tests

- A product with two overlapping current windows is counted **once** (`.distinct()`).
- Search/listing endpoints return the same set as before the change.

### How to demo

1. Hit `GET /api/categories/?school=1` on the fixed build.
2. ✅ `online_product_count` = 7 for category "Vela". ❌ Still 9.

## Acceptance criteria

| #    | Context                                      | Action                  | Expected result                      |
| ---- | -------------------------------------------- | ----------------------- | ------------------------------------ |
| AC-1 | Product online with a current period         | Count category products | Included in `online_product_count`   |
| AC-2 | Product online with only past/future periods | Count category products | Excluded from `online_product_count` |
| AC-3 | Product online with 2 overlapping windows    | Count category products | Counted exactly once                 |

### Checklist

- [ ] AC-1: Online + currently-available product is counted
- [ ] AC-2: Online product with no current period is excluded
- [ ] AC-3: Multi-window product counted once (`.distinct()`)
- [ ] Regression test fails on the buggy code and passes on the fix

## Deploy & rollback

- Branch `fix/online-count-availability` -> PR -> CircleCI -> `staging_deploy_hold` -> promote.
- Rollback: revert the PR; the change is isolated to one query helper.

## When fixed

Set **Status** to `fixed — PR #NNN, commit <hash> (availability filter on count)` and move this file to `specs/bugs/fixed/`.

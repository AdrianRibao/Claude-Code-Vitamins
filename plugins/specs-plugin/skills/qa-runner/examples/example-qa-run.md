# Example: QA plan and record for a UI task

The QA sections of task `012 — Archive records from the list` (a Small UI task). The rest of the task file is omitted. It shows a check that failed and was fixed (QA-2), a pre-existing defect on the changed screen fixed in the same change, a pre-existing defect on an untouched screen filed as a bug, and an improvement filed as a task. The same sections appear in bug docs and, title-cased, in UI TDDs.

## QA plan

### Environments

| Env     | Base URL                      | Start / deploy                    | Seed data                                   |
| ------- | ----------------------------- | --------------------------------- | ------------------------------------------- |
| local   | `http://localhost:4000`       | `bin/dev`                         | `bin/seed-qa` — 3 records, 1 admin, 1 staff |
| staging | `https://staging.example.com` | PR -> CI -> `staging_deploy_hold` | `qa-*` accounts, data reset nightly         |

### Logins

| Role  | local            | staging            |
| ----- | ---------------- | ------------------ |
| admin | `qa-admin-local` | `qa-admin-staging` |
| staff | `qa-staff-local` | `qa-staff-staging` |

### Checks

| #    | Covers | Check                                | Capability          | Passes when                                                   | Evidence                |
| ---- | ------ | ------------------------------------ | ------------------- | ------------------------------------------------------------- | ----------------------- |
| QA-1 | AC-1   | Admin archives a record and confirms | `dialog`, `network` | Row shows "Archived"; exactly one `PATCH /records/:id` -> 200 | screenshot, request log |
| QA-2 | AC-1   | Admin archives a record and cancels  | `dialog`, `network` | Row unchanged; no `PATCH` sent                                | request log             |
| QA-3 | AC-2   | Staff opens the records list         | `ui-flow`           | No "Archive" button on any row                                | screenshot              |
| QA-4 | AC-1   | The QA-1 flow raises no errors       | `console-clean`     | Zero page errors and zero console errors                      | console log             |
| QA-5 | —      | Records list accessibility           | `a11y`              | No serious or critical violations at WCAG 2.1 AA              | audit report            |
| QA-6 | —      | Records list on a phone              | `responsive`        | Rows render as cards at 375×812, no horizontal scroll         | screenshot              |

### Exploratory charter

Records list and record detail, as admin and staff. Probe: empty list, archiving the last record on a page, long names, slow network on archive, mobile width.

### QA record

**local · 2026-11-17 · `http://localhost:4000` · `task/012-archive-records` at `HEAD 3f2a9c1`, code `9b1e04d2` · agent-browser 0.43.0**

| #    | Result           | Observed                                                                                                                                                                                                      | Evidence                         |
| ---- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| QA-1 | pass             | "Archived" shown; one `PATCH /records/42` -> 200                                                                                                                                                              | `qa-1.png`, `qa-1-requests.json` |
| QA-2 | pass (after fix) | First run sent the `PATCH` on cancel (handler ignored the dialog result); fixed in `archive_button.ex:31` with a regression test in `archive_button_test.exs`; tests + review re-run, QA-1..QA-4 re-run clean | `qa-2-requests.json`             |
| QA-3 | pass             | 3 rows, no "Archive" button; snapshot shows no hidden button either                                                                                                                                           | `qa-3.png`                       |
| QA-4 | pass             | 0 page errors, 0 console errors                                                                                                                                                                               | `qa-4-console.txt`               |
| QA-5 | pass             | 0 serious, 0 critical                                                                                                                                                                                         | `qa-5-a11y.json`                 |
| QA-6 | pass             | Card layout, no horizontal scroll                                                                                                                                                                             | `qa-6.png`                       |

**Exploratory**: 3 findings — fixed: pagination reset to page 1 after archiving the last row of page 2 (pre-existing, on the records list this task changes; `records_live.ex:88`, regression test added, tests + review re-run, list checks re-run clean); filed: bug 019 (record detail's "Back" link drops the list's sort order — pre-existing, on a screen this task does not touch); filed: task 013 (undo after archive — an improvement, not a defect).

**Evidence**: `specs/qa/runs/012-archive-records/2026-11-17-local/` (git-ignored)

**staging · 2026-11-18 · `https://staging.example.com` · build `a41c9e2` (confirmed in `/version`) · agent-browser 0.43.0**

| #    | Result | Observed                                        | Evidence                         |
| ---- | ------ | ----------------------------------------------- | -------------------------------- |
| QA-1 | pass   | "Archived" shown; one `PATCH /records/7` -> 200 | `qa-1.png`, `qa-1-requests.json` |
| QA-2 | pass   | Row unchanged; no `PATCH`                       | `qa-2-requests.json`             |
| QA-3 | pass   | No "Archive" button as staff                    | `qa-3.png`                       |
| QA-4 | pass   | 0 page errors, 0 console errors                 | `qa-4-console.txt`               |
| QA-5 | pass   | 0 serious, 0 critical                           | `qa-5-a11y.json`                 |
| QA-6 | pass   | Card layout at 375×812                          | `qa-6.png`                       |

**Evidence**: `specs/qa/runs/012-archive-records/2026-11-18-staging/` (git-ignored)

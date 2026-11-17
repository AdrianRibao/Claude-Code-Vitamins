### QA record

\[Written by `qa-runner` after each run, under the QA plan. One block per environment; a newer run replaces the older block for the same environment.\]

**[local|staging] · YYYY-MM-DD · `[base URL]` · \[local: branch at `HEAD <sha>`, code `<fingerprint>` / staging: deployed version and how it was confirmed\] · agent-browser [version]**

| #    | Result                                                                                                                                                                     | Observed                                                                   | Evidence |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | -------- |
| QA-1 | [pass / pass (after fix) / pass (baseline approved YYYY-MM-DD) / informational / skipped — reason / deferred by user — bug NNN / baseline created — needs approval / fail] | \[what was seen, not "as expected"; a fix names its `file:line` and test\] | `[file]` |

**Exploratory** (local only): \[N findings — fixed: … (`file:line`, test, pre-existing or not); filed: bug NNN (pre-existing, on a screen the change does not touch) / task NNN (improvement); deferred by user: bug NNN ("quote", date); dismissed: … (reason)\]

\[An `--only` re-run updates its rows here and keeps the rest of the block.\]

**Evidence**: `specs/qa/runs/[spec-file-stem]/[YYYY-MM-DD]-[env]/` (git-ignored)

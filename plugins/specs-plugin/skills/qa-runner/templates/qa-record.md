### QA record

\[Written by `qa-runner` after each run, under the QA plan. One block per environment; a newer run replaces the older block for the same environment.\]

**[local|staging] · YYYY-MM-DD · `[base URL]` · [commit, branch working tree, or deployed version] · agent-browser [version]**

| #    | Result                                                                                                                                        | Observed                                                          | Evidence |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- | -------- |
| QA-1 | [pass / pass (after fix) / pass (baseline approved YYYY-MM-DD) / informational / skipped — reason / baseline created — needs approval / fail] | \[what was seen, not "as expected"; a fix names its `file:line`\] | `[file]` |

**Exploratory** (local only): \[N findings — fixed: … (`file:line`, pre-existing or not); filed: task NNN (improvements only); dismissed: … (reason)\]

\[An `--only` re-run updates its rows here and keeps the rest of the block.\]

**Evidence**: `specs/qa/runs/[spec-file-stem]/[YYYY-MM-DD]-[env]/` (git-ignored)

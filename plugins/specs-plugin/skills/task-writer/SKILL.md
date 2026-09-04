---
name: task-writer
description: Capture backlog tasks — Small and Medium units of work that are neither bugs nor PRD-sized features — as evidence-backed task files on a one-screen board, and optionally drive a task to shipped. Use when adding work to the backlog, picking up a task, reformatting an existing backlog to the convention, or archiving shipped work with what it settled.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - AskUserQuestion
  - TodoWrite
  - Task
  - Bash
  - WebFetch
  - WebSearch
  - mcp__plugin_context7_context7__resolve-library-id
  - mcp__plugin_context7_context7__query-docs
---

# Task Writer Skill

> **Philosophy**: A task is a *unit of work with a known next step*. It is not a bug (that is a delta between observed and expected behavior, owned by `bugfix-writer`) and not a feature (that needs a PRD and a TDD). The task file is the truth: dated evidence, a bounded scope, testable done-criteria, and the decisions already made — so the next session starts where this one stopped instead of re-investigating. The board is a one-screen index of the open task files, nothing more.

## When to Use

- Work has been identified that is Small or Medium and has a concrete next step
- Picking up a task and driving it to shipped (`--do`)
- Bringing an existing backlog (loose notes, an old board) into this convention (`--reformat`, `--board`)
- Reviewing a task file for missing evidence, unbounded scope, or untestable done-criteria (`--review`)
- Archiving shipped work with a record of what it settled (`--ship`)

**Not for:**

- **Defects** -> `bugfix-writer` (`/bugfix`). A task file never holds a bug; the board never lists one.
- **Features that need design** -> `prd-writer` / `tdd-writer`. A Large task gets a stub on the board whose next step is "write the PRD", and nothing more.
- **Deferred design decisions** with no next step (a "considerations" file). Those are decisions, not tasks.

## Usage

```
/task [name] [--ticket ID] [--page URL] [--from @path] [--do NNN|@path] [--branch NAME] [--no-branch] [--ask] [--review @path] [--reformat @path|--all] [--board] [--consolidate @path] [--ship NNN|@path]
```

| Flag                      | Purpose                                                                                                                     |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `--ticket ID`             | Ingest the task from a tracker issue (Jira/Linear/GitHub) via MCP                                                           |
| `--page URL`              | Ingest the task from a wiki page (Confluence/Notion) via MCP or WebFetch                                                    |
| `--from @path`            | Ingest the task from a local file or pasted notes                                                                           |
| `--do NNN\|@path`         | Pick up a task: move it to Doing, branch, implement, verify the ship gates, archive, stage — never commit                   |
| `--branch NAME`           | Use/create this branch for `--do`; default auto-creates `task/<NNN>-<slug>` when on a protected branch                      |
| `--no-branch`             | Work on the current branch; do not create or switch branches                                                                |
| `--ask`                   | Surface decide-and-record items as open decisions for review (default: decide and record silently)                          |
| `--review @path`          | Gap analysis of an existing task file against the convention and the ship gates; reports, does not edit                     |
| `--reformat @path\|--all` | Rewrite an existing task file (or every file under `tasks/` and `archived/`) into the convention, preserving id and content |
| `--board`                 | Rebuild the board rows from the task files' frontmatter and run the consistency checks                                      |
| `--consolidate @path`     | Apply answered decisions, fold them into the file as dated decisions, tighten                                               |
| `--ship NNN\|@path`       | Archive a task finished outside `--do`: verifies the ship gates, sets `status: shipped`, moves it, indexes it — staged only |

## Output Location

```
specs/backlog/
  BACKLOG.md            # the board: one row per open task
  tasks/NNN-slug.md     # open tasks (todo / doing / blocked)
  archived/NNN-slug.md  # shipped tasks — never deleted
  archived/README.md    # archive index: what each shipped task settled
```

**Conventions** (these are what `--board` checks):

- `NNN` is zero-padded, sequential, and **permanent**. Scan `tasks/` and `archived/` for the highest id and add one. Never renumber; never reuse an archived id. Commit messages and other docs cite these numbers.
- Filenames are `NNN-kebab-slug.md`. The slug may be tidied; the number may not.
- Each file opens with YAML frontmatter (`id`, `title`, `status`, `priority`, `size`, `updated`, optional `note`) followed by `# NNN — <title>`.
- `status` is one of `todo` / `doing` / `blocked` / `shipped`. `shipped` appears only under `archived/`; a file under `tasks/` is never `shipped`.
- Every open task has exactly one board row; every board row points at a file under `tasks/`; the row's priority and size match the frontmatter.
- Every archived file has a row in `archived/README.md` and none on the board.

If the project already has a backlog with a `## How this works` section, **follow it** where it differs from the above and say so. If the project has a checker (`just backlog-check`, `scripts/backlog_check.py`, a pre-commit hook), run it after every change; otherwise perform the checks yourself.

Ask the user to confirm or override the location during Phase 0 only when no `specs/backlog/` exists yet.

## Resources

- [style-guide.md](style-guide.md) - Section-by-section writing conventions, frontmatter, sizing and priority scales, the board, the archive index
- [templates/task.md](templates/task.md) - The task file template
- [templates/BACKLOG.md](templates/BACKLOG.md) - The board template, including its `How this works` section
- [templates/archived-README.md](templates/archived-README.md) - The archive index template
- [examples/](examples/) - Reference task files (an operational task and a code task)

## Behavioral Mindset

Evidence first, then a bounded scope, then a testable definition of done. Investigate before writing: read the code the task touches, run the command whose output proves the gap, cite `file:line` and dates. A task file that says "the generator does not see history" is a note; one that says "`grep -c history ai/generation/signatures.py` returns 0 (2026-09-03)" is a task.

**Decide by default.** Most of what looks like a question while writing a task is a decision you can make from the code and the project's own rules. Make it, record it inline with the date, keep moving. Only a genuine fork that changes which code you touch, or a value only the owner holds (money, policy, an external commitment), becomes an open decision. See [Question Policy](#question-policy).

**Route, do not absorb.** A defect goes to `/bugfix`, even if it was found while writing a task; link it from the task's `Related` and move on. A Large item gets a stub whose only next step is the PRD.

## Key Actions

Copy this checklist and track progress. Steps tagged `(--do)` run only when driving the task.

```
Task Progress:
- [ ] 0. Ingested the source; classified: task / bug -> /bugfix / Large -> PRD stub
- [ ] 0. Located the backlog (or bootstrapped it); assigned the next permanent NNN
- [ ] 1. Gathered dated evidence from code, commands, or the running system
- [ ] 1. Sized and prioritized; asked only blocking questions
- [ ] 2. Wrote Scope (numbered), Do NOT change, Out of scope, Sequencing
- [ ] 2. Wrote Done when (AC table + mirrored checklist) and Verification
- [ ] 3. Triaged decisions: recorded the decided ones inline, listed only real forks
- [ ] 4. Added the board row; ran the consistency checks (project checker if present)
- [ ] 5. (--do) On a task/<NNN>-<slug> branch; status: doing; row under Doing
- [ ] 5. (--do) Implemented the Scope and nothing else; tests at the right levels green
- [ ] 5. (--do) Confirmed the outcome in the running system; evidence recorded, dated
- [ ] 6. (--do / --ship) Ship gates verified; Shipped + Follow-ups written; follow-ups filed as new tasks
- [ ] 6. (--do / --ship) status: shipped; git mv to archived/; index row says what it settled; board row removed
- [ ] 6. Everything staged together; did NOT commit — handed off to the user
```

### Phase 0: Intake, Classification & Numbering

1. **Ingest the source**:
    - `--ticket ID` -> use whatever tracker MCP tools are connected. Follow links to the real description.
    - `--page URL` -> use the connected wiki MCP or WebFetch.
    - `--from @path` -> read the file or pasted notes.
    - No source -> work from the prompt.
2. **Classify before writing anything**:
    - Observed behavior differs from intended behavior -> this is a bug. Stop, tell the user, and hand it to `/bugfix`. Do not create a task file.
    - Needs a data model, new screens, or a product decision before it can be estimated -> Large. Create a stub task (Why + Next step: "write the PRD") with `note: needs a PRD first` and stop there.
    - Otherwise -> a task. Continue.
3. **Locate the backlog**: find `specs/backlog/`. If it exists, read `BACKLOG.md` and honour its `How this works`. If it does not, confirm the location with the user and bootstrap it from the templates.
4. **Assign the id**: highest id across `tasks/` and `archived/` plus one. Never reuse.
5. **Ask only blocking questions** (see [Question Policy](#question-policy)).

### Phase 1: Evidence, Size & Priority

1. **Investigate the code**: read the modules the task touches; run the command or query whose output shows the gap; check the running system when that is where the truth is. Record every finding with a date and a `file:line` or command reference.
2. **Identify third-party surfaces**: if the task involves a library, SDK, API, CLI, or cloud service, fetch current docs (context7 -> WebFetch -> WebSearch) before asserting how it behaves. Never write a third-party step from memory.
3. **Size it** (build effort only, see [style-guide.md](style-guide.md)): Small = a focused change, a day or less, no new subsystem. Medium = several files or both stacks, or something operational that must be deployed and verified. Large = needs a PRD first (back to Phase 0).
4. **Prioritize it** (P0–P3): P0 the product is broken or the pilot is at risk; P1 next up; P2 worthwhile; P3 someday. Priority and size are independent.
5. **Check for an existing task** covering the same ground. Extend it rather than duplicating; a task file is a living document.

### Phase 2: Scope, Done Criteria & Verification

1. **Lead paragraph**: one paragraph that says where the task stands and what shape the work has ("The work is provider-side; the code and the deploy already handle a number change").
2. **Why**: the evidence from Phase 1, written so the next reader does not re-investigate.
3. **Scope**: numbered steps saying WHAT changes (file/function/setting), signatures or ≤5-line snippets only. Then:
    - `### Do NOT change` — adjacent behavior that must stay identical.
    - `### Out of scope` — tempting nearby work this task does not cover.
4. **Constraints** (when relevant): project rules that bound the approach (a `CLAUDE.md` rule, an invariant pinned by a test, a compliance limit).
5. **Sequencing** (when relevant): what this depends on, what it blocks, and why now or why not yet.
6. **Done when**: an acceptance-criteria table (`# | Context | Action | Expected result`) plus the mirrored `- [ ] AC-N:` checklist so `ac-checker` can parse it. Every criterion must be observable.
7. **Verification**: the tests to add or run (unit for changed logic, E2E for a user-facing flow, unchanged-behavior tests for each Do-NOT-change item) and how to confirm the outcome in the running system.
8. **Next step**: the single concrete action that starts the work. Mandatory, even on a stub.

### Phase 3: Decision Triage (decide by default)

1. **Invoke Sequential MCP** with `--ultrathink` to scan the task for ambiguities, edge cases, and forks the scope glosses over.
2. **Classify** each finding (see [Question Policy](#question-policy)): decide-and-record, ask, or non-issue.
3. **Record decisions inline** under `## Decisions` as `~~question~~ **Decided YYYY-MM-DD: answer.** rationale`. The strike-through keeps the question visible so nobody re-asks it.
4. **List only real forks** as `OD-NN` entries in the same section, each with choices, consequences, and a recommended option. If there is nothing to decide, omit the section.

### Phase 4: Board & Consistency

1. **Add the row** under `## Next` in `BACKLOG.md`: id link, title, priority, size, and a one-line hook that says why it matters or what unlocks it. Rows are ordered by priority, then by size ascending.
2. **Run the checks**: the project's checker if there is one, otherwise the seven checks listed in [Output Location](#output-location). Fix any finding before presenting.
3. **`mdformat`** the task file and the board.

### Phase 5: Execution (only with `--do`)

1. **Select the working branch** before changing any code:
    - `--branch NAME` -> create/switch to it.
    - On a protected branch (`main`/`master`/`develop`/`release/*`) -> create `task/<NNN>-<slug>` and switch; announce it.
    - Already on a feature branch -> use it as-is.
    - `--no-branch` -> stay put.
2. **Move to Doing**: set `status: doing`, bump `updated`, move the board row from Next to Doing.
3. **Implement the Scope and nothing else.** When the Scope turns out to be wrong, update the task file first, then the code. When the work reveals a defect, file it with `/bugfix` and link it; do not fix it inside the task.
4. **Tests at the right levels**: unit tests for changed logic, E2E for a user-facing flow, unchanged-behavior tests for each Do-NOT-change item. All green.
5. **Confirm in the running system**: run the command, drive the browser (`claude-in-chrome` for UI), hit the endpoint, or check the deployed host. Passing tests are necessary, not sufficient. Record the evidence, dated.
6. **Tick the checklist** in Done when as each criterion is met, with a one-line pointer to the evidence.

Without `--do`, Phases 5–6 are written as a plan; the implementer follows them later.

### Phase 6: Ship & Hand Off (`--do` and `--ship`)

1. **Verify the ship gates**: all five [Ship Gates](#the-five-ship-gates) must hold. **Do not archive, and do not advise committing, while any gate is unmet.**
2. **Write the shipping record** at the top of the file, after the lead paragraph:
    - `## Shipped` — what landed, the commits or PR, the verification evidence with dates, and the size as it turned out versus the estimate.
    - `## Decisions worth not re-litigating` — only when a decision deliberately departs from the obvious way; say why.
    - `## Follow-ups` — new work discovered, each **filed as its own task** (new id, row under Next) and linked. Leftovers are not follow-ups; a leftover means the task is not done.
3. **Archive**: set `status: shipped`, bump `updated`, `git mv` the file into `archived/` (same filename), delete the board row, add a row to `archived/README.md` whose last column says **what the task settled** — not what it did.
4. **Run the checks** again.
5. **Stage everything together** — code, tests, the moved task file, the board, the index, the new follow-up tasks — and **stop. The skill does not commit.** The user reviews the staged diff and makes the commit or opens the PR; the move rides in that commit.

### Reformatting an existing backlog (`--reformat`, `--board`)

1. **Read the whole existing corpus first**: every task file, the board, the archive index, and any `How this works` text. Extract the conventions already in force.
2. **Per file, preserve**: the id, the title's meaning, every dated fact, every decision (struck-through or not), every link, and the archive status. Nothing is deleted; content that fits no section goes under `## Notes` rather than being dropped.
3. **Per file, add what is missing** by investigating, not inventing: a lead paragraph, `Do NOT change` / `Out of scope`, `Done when` with a checklist, `Verification`, `Next step`. Where the evidence to write a section does not exist, write `[to be established]` and list it in the report.
4. **Route misfiled items**: a bug living in the backlog is reported to the user with the recommendation to recreate it via `/bugfix` and remove the task; the skill does not silently delete it.
5. **Rebuild the board** from frontmatter (`--board`), run the checks, `mdformat` everything, and present a report: files touched, sections added, items routed elsewhere, findings that need the user.

## The Five Ship Gates

A task is **not shipped**, and a `--do` change **must not be committed**, until all five hold (or are explicitly, justifiably waived in the file):

| Gate                               | Requirement                                                                                                                                                     |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Evidence-backed Why**            | The gap is shown with dated evidence (`file:line`, command output, or an observation in the running system) — not asserted                                      |
| **Bounded scope**                  | `Do NOT change` and `Out of scope` are explicit; the change is what Scope says and nothing else; defects found on the way were filed with `/bugfix`, not fixed  |
| **Done criteria met**              | Every `- [ ] AC-N` in Done when is checked, each with a pointer to its evidence                                                                                 |
| **Verified in the running system** | Tests at the right levels are green where code changed, AND the outcome is confirmed live (browser for UI, endpoint for APIs, host for operational work), dated |
| **Settled and handed off**         | The archive row says what was settled; follow-ups are filed as tasks; status is `shipped`; the board row is gone; everything is staged, nothing committed       |

> "Confirmed in the running system" is the gate people skip. A deploy that reported success while building a stale commit, a config change that never reached the host, a flag that was set on the wrong environment — each of these passes every test. Look at the real thing.

## Question Policy

**Default: decide.** Writing a task is an investigation, and investigations produce answers. If the code, the project's rules, the existing tasks, or fetched docs support a defensible answer, that is a decision — make it, record it under `## Decisions` with the date, keep moving. Classify every candidate question into one of three tiers:

| Tier                  | When                                                                                                                                                                    | Action                                                                                                         |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Decide-and-record** | A defensible answer follows from the code, the project's own rules, sibling tasks, or engineering practice (naming, an edge case, a default, an ordering)               | Decide; write `~~question~~ **Decided YYYY-MM-DD: answer.** rationale` under `## Decisions`                    |
| **Ask**               | A fork changes which code you touch and only the owner can pick (product behavior the specs leave open), or a value only they hold (money, policy, external commitment) | `AskUserQuestion` in Phase 0 when it blocks writing the task; otherwise an `OD-NN` entry with a recommendation |
| **Non-issue**         | Already specified, or the answer changes nothing                                                                                                                        | Say nothing; do not manufacture questions                                                                      |

**Irreversible-work guardrail**: when the task touches data migration, data loss, money, security, or an external party's configuration that is hard to undo (a provider account, a DNS record, a published number), the ask bar drops — confirm the approach before executing, even if the file looks complete.

`--ask` overrides the default and surfaces decide-and-record items as `OD-NN` entries too, each carrying your recommended answer.

## Include in Task Files

| Element            | Purpose                                                            | Example                                                                                   |
| ------------------ | ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| Frontmatter        | The structured truth the board copies from                         | `id: 008` … `size: Small` … `updated: 2026-09-03`                                         |
| Lead paragraph     | Where the task stands and the shape of the work                    | "The work is provider-side; the code already handles a number change"                     |
| Why                | Dated evidence of the gap                                          | "`grep -c history ai/generation/signatures.py` returned 0 (2026-09-03)"                   |
| Scope              | Numbered WHAT-changes + **Do NOT change** + **Out of scope**       | "1. Add `CLOSING` to `IntentType` … Do NOT touch the booking slot collection"             |
| Constraints        | Project rules that bound the approach                              | "`CLAUDE.md`: never keyword-match intents — must be signature-level"                      |
| Sequencing         | Depends on / blocks / why now                                      | "After 002 — decide Loki vs MLflow first"                                                 |
| Done when          | AC table + mirrored `- [ ]` checklist                              | "AC-1 … / `- [ ] AC-1: …`"                                                                |
| Verification       | Tests to add/run + how to confirm live                             | "Corpus rows for `gracias` across intents; confirm `reason: customer_request` in Grafana" |
| Decisions          | Decided items struck through and dated; open forks as `OD-NN`      | "~~Close mid-booking?~~ **Decided 2026-09-03: no.** Context decides, never the word"      |
| Related            | PRD/TDD, bugs filed, runbooks, sibling tasks                       | "Bug 014 (found while investigating); runbook `whatsapp-app-setup.md`"                    |
| Next step          | The single action that starts the work                             | "Write the PRD. Study the reference panel first."                                         |
| Shipped (archived) | What landed, commits, dated verification, actual vs estimated size | "Verified in production 2026-09-03: … Size was Medium, as estimated"                      |
| Follow-ups         | New work discovered, each filed as its own task                    | "Distributed tracing across both stacks -> task 015"                                      |

## Exclude from Task Files

| Element                       | Why Exclude                                             | Where It Belongs                   |
| ----------------------------- | ------------------------------------------------------- | ---------------------------------- |
| A defect and its fix          | Bugs have their own lifecycle and gates                 | `specs/bugs/` via `/bugfix`        |
| Feature requirements          | Needs personas, workflows, a data model                 | A PRD, then a TDD                  |
| Full implementation code      | The diff lives in the PR                                | The code / PR                      |
| Undated observations          | Rot silently; the next reader cannot trust them         | (add the date, or drop the claim)  |
| Leftovers labelled follow-ups | Hides unfinished work                                   | Finish the task, or split it first |
| Manufactured questions        | Stalls ready work                                       | (omit)                             |
| Status prose in frontmatter   | The board copies frontmatter; prose belongs in the lead | The lead paragraph                 |

## Lifecycle Management

- **States**: `todo` -> `doing` -> `shipped`, with `blocked` as a side state. On the board, state is the section (Doing or Next); a `blocked` task stays under Next with a hook beginning `Blocked —` and the blocker named in `Sequencing` or `Decisions`.
- **Branch** (`--do` only): `task/<NNN>-<slug>`, auto-created when on a protected branch, reused when already on a feature branch, overridable with `--branch` / `--no-branch`.
- **Living document**: a task file is updated as understanding improves — partial progress goes in the lead paragraph, settled questions get struck through, `updated` is bumped every time. It is not frozen at creation the way a bug doc is.
- **The move is staged, never committed by the skill**: on ship, `git mv` into `archived/`, edit the board and the index, `git add` all of it, and hand off.
- **`--ship`** is the explicit close for work done outside `--do`: it verifies the five gates, writes the shipping record from the evidence you give it, archives, and stages. It refuses when a gate is unmet.
- **Nothing is deleted.** A reversed decision gets its own task; the archived file stays.

## MCP Integration

- **Sequential MCP**: decision triage (Phase 3) and structured investigation of the gap.
- **Context7 MCP**: current docs for any third-party surface the task touches.
- **Tracker / wiki MCP** (Jira, Confluence, Linear, Notion, GitHub — whatever is connected): ingest for `--ticket` / `--page`. If none is available, ask the user to paste the notes or use `--from`.

## Outputs

- **Task file** (default): frontmatter, lead, Why, Scope with guardrails, Done when, Verification, Decisions, Next step — plus its board row, checks passing.
- **PRD stub** (Large intake): a task file with Why and `Next step: write the PRD`, `note: needs a PRD first`, and its board row.
- **Shipped task** (`--do` / `--ship`): the file with Shipped and Follow-ups, `status: shipped`, moved to `archived/`, indexed with what it settled, follow-ups filed, everything staged.
- **Review** (`--review`): findings against the convention and the five gates, no edits.
- **Reformatted backlog** (`--reformat`, `--board`): every file in the convention, ids and content preserved, board rebuilt, checks passing, a report of what was added and what needs the user.

## Examples

### Add a task from notes

```
/task generator-conversation-history --from @notes/history.md

# Investigates the generation signatures, records the grep evidence,
# sizes it Medium, writes specs/backlog/tasks/010-generator-conversation-history.md
# and its row under Next
```

### Pick up a task and ship it

```
/task --do 008

# Branches task/008-whatsapp-production-number, moves the row to Doing,
# performs the scope, confirms on the host, writes Shipped + Follow-ups,
# archives with "what it settled", stages everything — you commit
```

### Bring an existing backlog into the convention

```
/task --reformat --all --board

# Rewrites every task file preserving ids and content, flags the bug that
# should be a /bugfix, rebuilds BACKLOG.md, runs the checks, reports
```

### Route a defect

```
/task services-render-blank --from @notes/blank-columns.md

# "This is a defect (observed differs from intended). Not creating a task —
#  run: /bugfix services-render-blank --from @notes/blank-columns.md"
```

## Quality Checklist

Before finalizing a task file, verify:

- [ ] Classified correctly: not a bug (else `/bugfix`), not Large (else a PRD stub)
- [ ] Frontmatter complete (`id`, `title`, `status`, `priority`, `size`, `updated`); `id` matches the filename and the H1
- [ ] Lead paragraph states where the task stands and the shape of the work
- [ ] **Why carries dated evidence with `file:line` or command output — no undated claims**
- [ ] Scope is numbered WHAT-changes with no code block longer than 5 lines
- [ ] **`Do NOT change` and `Out of scope` are explicit**
- [ ] Sequencing names dependencies when any exist
- [ ] **Done when has an AC table AND a mirrored `- [ ] AC-N:` checklist (ac-checker compatible)**
- [ ] Verification names the tests to add/run and how to confirm in the running system
- [ ] Decisions: decided items struck through with a date and rationale; only genuine forks as `OD-NN`, each with a recommendation; section omitted when empty
- [ ] Next step is a single concrete action
- [ ] Any third-party step was verified against freshly fetched docs, not memory
- [ ] Board row present under the right section; priority and size match frontmatter; hook is one line
- [ ] Consistency checks pass (project checker if present)
- [ ] With `--do` / `--ship`: all five ship gates hold; Shipped + Follow-ups written; follow-ups filed as tasks; archive row says what was settled
- [ ] **No commit was made or advised; the move, the board, the index and the code are staged together**
- [ ] `mdformat` passes on every touched file

## Boundaries

**Will:**

- Ingest a task from a tracker, a wiki page, a file, or the prompt, and classify it before writing
- Investigate the code and the running system so the Why carries dated evidence
- Size and prioritize; bound the scope with `Do NOT change` / `Out of scope`
- Write testable done-criteria and a verification plan
- Decide by default and record decisions inline; ask only real forks
- Maintain the board and the archive index, and keep them consistent with the files
- In `--do`, branch, implement exactly the Scope, verify live, archive, and stage
- Reformat an existing backlog into the convention without losing content or ids

**Will Not:**

- Create a task for a defect (routes to `/bugfix`) or for a feature that needs a PRD (creates a stub only)
- Renumber or reuse ids, or delete an archived file
- Fix bugs discovered during `--do` inside the task (files them instead)
- Archive a task, or advise committing, while any of the five ship gates is unmet
- Make any git commit — it stages and hands off
- Edit code unless `--do` is passed

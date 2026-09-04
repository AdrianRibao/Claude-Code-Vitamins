# Backlog

The board. One line per open task; the detail lives in [`tasks/`](tasks/).

\[One or two sentences on what this backlog holds and what it deliberately does not — e.g. "Work queued after the pilot went live. Deferred design decisions live in POST-PILOT-CONSIDERATIONS.md; bugs live in `specs/bugs/`."\]

## Doing

| #   | Task                  | Priority | Size | Hook |
| --- | --------------------- | -------- | ---- | ---- |
| —   | *(nothing in flight)* |          |      |      |

## Next

| #                        | Task    | Priority | Size  | Hook                               |
| ------------------------ | ------- | -------- | ----- | ---------------------------------- |
| [001](tasks/001-slug.md) | [Title] | P1       | Small | [Why it matters / what it unlocks] |

Shipped work moves to [`archived/`](archived/README.md) and leaves this board.

## How this works

- **The board is the index; the task file is the truth.** Only id, title, priority, size and a one-line hook live here. Everything else — evidence, decisions, scope, follow-ups — belongs in the task file, so reading the board costs a screen rather than a thousand lines.
- **IDs are permanent.** `003` stays `003` forever, in `tasks/` and later in `archived/`. Commit messages and other docs cite these numbers; renumbering would silently break them. New tasks take the next free number and never reuse an archived one.
- **Filenames are `NNN-kebab-slug.md`.** The slug can be tidied; the number cannot.
- **Status lives in which section a row is in** — Doing or Next — not in a column. A task in flight moves up; a shipped one moves out. A blocked task stays under Next with a hook that starts `Blocked —`.
- **Shipped means archived**, not deleted: move the file to `archived/`, add a row to its index saying what the task *settled*, and delete the row here. The record survives; the noise does not.
- **Size** estimates build effort only. **Small** = a focused change, roughly a day or less, no new subsystem. **Medium** = several files or both stacks, or something operational that needs deploying and verifying. **Large** = needs a PRD first; the task is a stub until the PRD exists.
- **Priority** is about consequence, independent of size: **P0** the product is broken; **P1** next up; **P2** worthwhile; **P3** someday.
- **Bugs are not tasks.** A defect goes to `specs/bugs/` via `/bugfix` and is never listed here. A task may link the bug it grew out of.
- **Evidence is recorded inline in the task file**, dated, so the next session does not re-investigate.
- **Each task file opens with YAML frontmatter** — `id`, `title`, `status`, `priority`, `size`, `updated`. That is the structured truth; the board copies from it. `status` is one of `todo` / `doing` / `blocked` / `shipped`, and `shipped` belongs only in `archived/`.
- **Consistency is checked, not trusted**: every open task has a row and vice versa, ids are unique across `tasks/` and `archived/`, frontmatter is complete and matches the filename and H1, and the priority/size shown here match the frontmatter. `/task --board` runs these checks (and the project's own checker, if it has one) after every change.

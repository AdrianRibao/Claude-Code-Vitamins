## QA plan

\[UI changes only. For an API- or backend-only change, delete this section and write `QA: n/a — no UI change` in Verification. Conventions: [qa-runner style guide](../style-guide.md). Say WHAT to check, never an agent-browser command.\]

### Environments

| Env     | Base URL                  | Start / deploy            | Seed data               |
| ------- | ------------------------- | ------------------------- | ----------------------- |
| local   | `http://localhost:[port]` | [command that starts it]  | [script / fixture]      |
| staging | `[url]`                   | [how a change gets there] | [accounts / data reset] |

\[Omit the staging row when the project has no staging. Never production.\]

### Logins

| Role   | local             | staging             |
| ------ | ----------------- | ------------------- |
| [role] | `[profile]-local` | `[profile]-staging` |

\[One agent-browser vault profile per role and environment — a profile stores a single login URL. The user saves profiles in the vault in their own terminal; never a credential or a command here. `anonymous` when no login is needed. Drop the staging column when the project has no staging.\]

### Checks

| #    | Covers | Check                   | Capability | Passes when                                      | Evidence     |
| ---- | ------ | ----------------------- | ---------- | ------------------------------------------------ | ------------ |
| QA-1 | AC-1   | [role] [does the thing] | `ui-flow`  | [observable, specific]                           | screenshot   |
| QA-2 | —      | [screen] accessibility  | `a11y`     | No serious or critical violations at WCAG 2.1 AA | audit report |

\[Capabilities: `ui-flow`, `dialog`, `network`, `console-clean`, `a11y`, `responsive`, `visual`, `vitals`.\]

### Exploratory charter

[Changed screens, their neighbours, and the risks to probe — scopes the dogfood pass on local.]

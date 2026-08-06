# lean-ai-sdd-pipeline

This repo is the Spec-Driven Development pipeline itself — specs, architecture, ADRs, issue tracking, and Claude skills for one or more application projects that live as sibling repos. See `README.md` for the full model.

## On Every Session Start

This is an instruction to Claude, not something the user needs to ask for:

1. If a project is named, or obvious from context, invoke `system_workflow_resume` for that project yourself and open with the briefing it produces (what exists, what's in progress, open issues, recommended next action).
2. If no project is named and more than one exists under `projects/`, invoke `system_workflow_resume` with no argument to list all projects and their status first.

`system_workflow_resume` is a `system_*` skill — Claude runs it directly; it is not exposed as something the user types.

## Skills Available

### Project lifecycle (`project_*`) — user-facing

| Command | Step | Purpose |
|---|---|---|
| `/project_00_init <project-name> <repo-path>` | 1 | Validate and register a project's sibling application repo |
| `/project_01_brief <project-name>` | 2 | What the project is, who it's for, what success looks like |
| `/project_02_architecture <project-name>` | 3 | System layers (with repo path + responsibility + status), key decisions, data stores |
| `/project_adr <project-name> <decision-title>` | as needed | Record a project-wide architectural decision (starts `Proposed`) |
| `/project_90_status <project-name>` | anytime | Display-only status: health, current state, document/feature/issue/ADR summary, recommended next step |

### Feature lifecycle (`feature_*`) — user-facing

Run in order for each feature. Each step is gated on the previous document being `Status: Approved`, not just present.

| Command | Step | Purpose |
|---|---|---|
| `/feature_00_init <project-name> <feature-name>` | 1 | Create the feature's directory (requires project docs Approved) |
| `/feature_01_brief <project-name> <feature-name>` | 2 | What to build — writes stable `AC-N` acceptance criteria |
| `/feature_02_technical_design <project-name> <feature-name>` | 3 | How to build it — declares `API Specification Required: Yes/No` |
| `/feature_03_api_spec <project-name> <feature-name>` | 4, only if required | Exact contracts |
| `/feature_04_test_spec <project-name> <feature-name>` | 5 | Tests, each mapped to the `AC-N` it covers — defines "done" |
| `/feature_05_implementation_plan <project-name> <feature-name>` | 6 | Coding tasks with a real Status column and Tests Satisfied mapping |
| `/feature_adr <project-name> <feature-name> <decision-title>` | as needed | Record a feature-level decision (starts `Proposed`) |
| `/feature_90_status <project-name> <feature-name>` | anytime | Display-only status: health, current state, document/task/test summary, recommended next step |

### Issue lifecycle (`issue_*`) — user-facing

Issues are siblings of features, not nested beneath them.

| Command | Step | Purpose |
|---|---|---|
| `/issue_01_capture <project-name> "<description>"` | 1 | `01_issue.md` — classification happens automatically as part of this |
| `/issue_02_investigate <project-name> <issue-id>` | 2 | `02_investigation.md` — root cause + reconciliation classification |
| `/issue_03_resolve <project-name> <issue-id>` | 3 | `03_resolution.md` — apply the fix — never auto-committed or pushed |
| `/issue_04_verify <project-name> <issue-id>` | 4 | `04_verification.md` — confirm the fix, close the issue — automatically runs a consistency review |
| `/issue_90_status <project-name> [issue-id]` | anytime | Display-only status: single issue's stage/status, or a project-wide Open/Investigating/Resolved/Closed count |

Every issue always produces all four files (`01_issue.md`, `02_investigation.md`, `03_resolution.md`, `04_verification.md`), regardless of size. Classification assigns a category only; it does not change which files get created.

### Internal orchestration (`system_*`) — never user-facing

Do not tell the user to run these as commands. Invoke them yourself, at the point noted:

| Skill | Invoked by |
|---|---|
| `system_issue_classify` | `issue_01_capture`, automatically |
| `system_consistency_review` | `feature_05_implementation_plan` (once all tasks Complete), `issue_04_verify` (on issue closure), or Claude directly when asked to check consistency |
| `system_next_step` | Claude, when it needs to determine what comes next and a skill's own stated next step isn't enough |
| `system_workflow_resume` | Claude, at session start or when asked to re-orient |

## Operational Safety

Any time you (Claude) are about to modify files in the application repository (`repo_path`), whether via `issue_03_resolve` or while implementing a task from `05_implementation_plan.md`:

- Re-validate `repo_path` resolves to a real git repository before editing.
- Run `git -C {repo_path} status` first. If there's unrelated uncommitted work, tell the user and confirm before editing over it.
- Touch only the files required for the task — no opportunistic refactors.
- Never run `git commit` or `git push` in the application repository on your own initiative.
- Always summarize which files you changed.

## Notes

- No skills are copied into application repositories. Everything here operates against `projects/<project-name>/` in this repo, reaching into the application repo only via `repo_path` (validated in `project_00_init`) for source code inspection.
- The specification is the source of truth unless an approved change explicitly changes it. When code and documentation disagree, determine the intended behaviour first, update the highest authoritative artifact, then propagate downstream (`01_feature_brief → 02_technical_design → 03_api_spec → 04_test_spec → 05_implementation_plan`), and update the implementation last. Never rewrite approved documents merely to match an unapproved implementation change — if the documents are still correct, fix the code only.
- Never report a feature as built or verified just because a document exists — check task Status and the `Verified` field, per `system_next_step`.

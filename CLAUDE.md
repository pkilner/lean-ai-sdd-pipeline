# lean-ai-sdd-pipeline

This repo is the Spec-Driven Development pipeline itself — the Claude skills that drive specs, architecture, ADRs, and issue tracking for one or more application projects that live as sibling repos. See `README.md` for the full model.

## Working Agreement

@docs/working-agreement.md

The line above loads `docs/working-agreement.md`, which says how the owner and Claude work together on every project: how to communicate, whose decision is whose, what must be verified, and the rules for git, secrets, money, and access. Follow it without being asked. If its text is not already in your context, read that file before doing anything else.

## Where Artifacts Live

Every artifact the skills generate lives in the project's **application repository**, under `sdd/` — never in this repo. Skills call that directory `{project-root}`:

- `projects/<project-name>.yaml` in this repo is a local, gitignored pointer holding only `repo_path` (relative path from this repo's root to the application repository).
- `{project-root}` is `{repo_path}/sdd` — it holds `project.yaml`, `01_project_brief.md`, `02_project_architecture.md`, `adr/`, `features/`, and `issues/`.

Do not write project artifacts under `projects/` in this repo.

## On Every Session Start

This is an instruction to Claude, not something the user needs to ask for:

1. If a project is named, or obvious from context, invoke `system_workflow_resume` for that project yourself and open with the briefing it produces (what exists, what's in progress, open issues, recommended next action).
2. If no project is named and more than one is registered under `projects/`, invoke `system_workflow_resume` with no argument to list all projects and their status first.

`system_workflow_resume` is a `system_*` skill — Claude runs it directly; it is not exposed as something the user types.

## Skills Available

### Project lifecycle (`project_*`) — user-facing

| Command | Step | Purpose |
|---|---|---|
| `/project_00_init <project-name> <repo-path>` | 1 | Validate a project's sibling application repo, scaffold `sdd/` inside it, and register the local pointer |
| `/project_01_brief <project-name>` | 2 | What the project is, who it's for, what success looks like |
| `/project_02_architecture <project-name>` | 3 | System layers (with repo path + responsibility + status), key decisions, data stores |
| `/project_adr <project-name> <decision-title>` | as needed | Record a project-wide architectural decision (starts `Proposed`) |
| `/project_setup_guide <project-name> "<title>"` | as needed | Write a guide for something the owner sets up by hand, then check and record the result |
| `/project_infrastructure_map <project-name>` | after each guide | Create or update the diagrams of what exists, who can do what, and what is exposed |
| `/project_90_status <project-name>` | anytime | Status: concise feature spec/plan document states and overall feature status, saved to `sdd/STATUS.md` |

### Project scope and delivery

Maintain one project brief and one project architecture for the whole intended product. Release order does not define project scope. Separate agreed capabilities, unresolved product candidates, and explicit product exclusions; do not promote a candidate to a commitment or classify an agreed later capability as excluded.

A feature is a bounded, testable delivery increment. It may span several capabilities and system layers, including a proof-of-concept phase. Keep a lightweight delivery sequence in the project brief with feature IDs, outcomes, dependencies, and links to specifications once they exist. Planning rows are not evidence that feature documents exist or that work is approved or implemented.

### Feature lifecycle (`feature_*`) — user-facing

Run in order for each feature. Each step is gated on the previous document being `Status: Approved`, not just present.

| Command | Step | Purpose |
|---|---|---|
| `/feature_00_init <project-name> <feature-name>` | 1 | Create the feature's directory (requires project docs Approved) |
| `/feature_01_spec <project-name> <feature-name>` | 2 | What to build and how — stable `AC-N` acceptance criteria, design, and contracts |
| `/feature_02_plan <project-name> <feature-name>` | 3 | Tests mapped to the `AC-N` they cover, then coding tasks with a real Status column |
| `/feature_adr <project-name> <feature-name> <decision-title>` | as needed | Record a feature-level decision (starts `Proposed`) |
| `/feature_90_status <project-name> <feature-name>` | anytime | Display-only status: health, current state, document/task/test summary, recommended next step |

### Issue lifecycle (`issue_*`) — user-facing

Issues are siblings of features, not nested beneath them.

| Command | Step | Purpose |
|---|---|---|
| `/issue_01_capture <project-name> "<description>"` | 1 | Creates `issues/ISSUE-xxxx.md` — classification happens automatically as part of this |
| `/issue_02_investigate <project-name> <issue-id>` | 2 | Fills the Investigation section — root cause + reconciliation classification |
| `/issue_03_resolve <project-name> <issue-id>` | 3 | Fills the Resolution section — apply the fix — never auto-committed or pushed |
| `/issue_04_verify <project-name> <issue-id>` | 4 | Fills the Verification section — confirm the fix, close the issue — automatically runs a consistency review |
| `/issue_90_status <project-name> [issue-id]` | anytime | Display-only status: single issue's stage/status, or a project-wide Open/Investigating/Resolved/Closed count |

Every issue is one file, `issues/ISSUE-xxxx.md`, with four sections filled in as it progresses. Its header's `Related` field links it to features, acceptance criteria (`{feature-name} AC-N`), and ADRs. Classification assigns a category only.

### Internal orchestration (`system_*`) — never user-facing

Do not tell the user to run these as commands. Invoke them yourself, at the point noted:

| Skill | Invoked by |
|---|---|
| `system_issue_classify` | `issue_01_capture`, automatically |
| `system_consistency_review` | `feature_02_plan` (once all tasks Complete), `issue_04_verify` (on issue closure), or Claude directly when asked to check consistency |
| `system_next_step` | Claude, when it needs to determine what comes next and a skill's own stated next step isn't enough |
| `system_workflow_resume` | Claude, at session start or when asked to re-orient |

## Operational Safety

Any time you (Claude) are about to modify application code — files in the application repository (`repo_path`) outside `sdd/` — whether via `issue_03_resolve` or while implementing a task from `02_feature_plan.md`:

- Re-validate `repo_path` resolves to a real git repository before editing.
- Run `git -C {repo_path} status` first. If there's unrelated uncommitted work, tell the user and confirm before editing over it.
- Touch only the files required for the task — no opportunistic refactors.
- Never run `git commit` or `git push` in the application repository on your own initiative — this covers the `sdd/` artifacts too.
- Always summarize which files you changed.

## Notes

- No skills are copied into application repositories. Skills run from this repo and reach the application repo via `repo_path` (validated in `project_00_init`) — to read and write artifacts under `sdd/`, and to inspect source code.
- The specification is the source of truth unless an approved change explicitly changes it. When code and documentation disagree, determine the intended behaviour first, update the highest authoritative artifact, then propagate downstream (`01_feature_spec → 02_feature_plan`), and update the implementation last. Never rewrite approved documents merely to match an unapproved implementation change — if the documents are still correct, fix the code only.
- Keep `STATUS.md` current. Whenever a document is created, approved, accepted, or returned to Draft, or an issue changes status, run `project_90_status` before ending the turn so the snapshot matches. The skills that write documents say this too; it also applies when you change a status outside a skill.
- Never report a feature as built or verified just because a document exists — check task Status and the `Verified` field, per `system_next_step`.

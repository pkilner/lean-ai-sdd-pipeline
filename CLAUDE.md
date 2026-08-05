# lean-ai-sdd-pipeline

This repo is the Spec-Driven Development pipeline itself — specs, architecture, ADRs, issue tracking, and Claude skills for one or more application projects that live as sibling repos. See `README.md` for the full model.

## On Every Session Start

If the user names a project, or one is obvious from context:

1. Run `/system_workflow_resume <project-name>` for a full briefing (what exists, what's in progress, open issues, recommended next action).
2. If no project is named and more than one exists under `projects/`, run `/system_workflow_resume` with no argument to list all projects and their status first.

## Skills Available

### Project lifecycle (`project_*`)

| Command | Step | Purpose |
|---|---|---|
| `/project_init <project-name> <repo-path>` | 1 | Register a project and point it at its sibling application repo |
| `/project_brief <project-name>` | 2 | What the project is, who it's for, what success looks like |
| `/project_architecture <project-name>` | 3 | System layers, key technical decisions, data stores |
| `/project_adr <project-name> <decision-title>` | as needed | Record a project-wide architectural decision |

### Feature lifecycle (`feature_*`)

Run in order for each feature. Each ends with a human review touchpoint before the next begins.

| Command | Step | Purpose |
|---|---|---|
| `/feature_init <project-name> <feature-name>` | 1 | Create the feature's directory |
| `/feature_brief <project-name> <feature-name>` | 2 | What to build |
| `/feature_technical_design <project-name> <feature-name>` | 3 | How to build it |
| `/feature_api_spec <project-name> <feature-name>` | 4 (if needed) | Exact contracts |
| `/feature_test_spec <project-name> <feature-name>` | 5 | Tests — defines "done" |
| `/feature_implementation_plan <project-name> <feature-name>` | 6 | Coding tasks mapped to the tests they satisfy |
| `/feature_adr <project-name> <feature-name> <decision-title>` | as needed | Record a feature-level decision |

### Issue lifecycle (`issue_*`)

Issues are siblings of features, not nested beneath them.

| Command | Step | Purpose |
|---|---|---|
| `/issue_capture <project-name> "<description>"` | 1 | Record what was observed |
| `/system_issue_classify <project-name> <issue-id>` | 2 | Assign a category |
| `/issue_investigate <project-name> <issue-id>` | 3 | Root cause + reconciliation classification |
| `/issue_resolve <project-name> <issue-id>` | 4 | Apply the fix |
| `/issue_verify <project-name> <issue-id>` | 5 | Confirm the fix, close the issue |
| `/system_consistency_review <project-name> [feature-name]` | 6 | Confirm no residual drift |

### Governance (`system_*`, not normally invoked directly)

| Command | Purpose |
|---|---|
| `/system_workflow_resume <project-name>` | Full project briefing |
| `/system_next_step <project-name> [feature-name\|issue-id]` | What to run next, given current state |
| `/system_consistency_review <project-name> [feature-name]` | Cross-check docs against implementation |
| `/system_issue_classify <project-name> <issue-id>` | Categorize an issue |

## Notes

- No skills are copied into application repositories. Everything here operates against `projects/<project-name>/` in this repo, reaching into the application repo only via `repo_path` (set in `project.yaml`) for source code inspection.
- Change propagation is always top-down: `feature-brief → technical-design → api-spec → test-spec → implementation-plan → implementation`. When a change originates downstream (e.g. a bug found in code), update upward first, then confirm downward.

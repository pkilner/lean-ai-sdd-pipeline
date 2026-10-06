---
name: project_90_status
description: Report a project's status — health, current state, a table of every document with its location, Draft/Approved status, approval date, and next action, feature/issue/ADR summary, consistency result, and recommended next step — and save the same report as a snapshot to STATUS.md in the project root. Internally invokes system_workflow_resume, system_consistency_review, and system_next_step. Never modifies any other file.
---

The argument is the project name in kebab-case.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Steps

1. Internally invoke `system_workflow_resume {project-name}` to get the project's current state: `01_project_brief.md` / `02_project_architecture.md` status, the list of features with their states (in definition / Feature Specified / Feature In Progress / Feature Implemented / Feature Verified), and the list of issues with their statuses.

2. Internally invoke `system_consistency_review {project-name}` (project-wide — no feature name) to get a `PASS` / `PASS WITH WARNINGS` / `FAIL` verdict.

3. Internally invoke `system_next_step {project-name}` to get the single recommended next command.

4. Map the consistency verdict to a health indicator:
   - `PASS` → 🟢 Healthy
   - `PASS WITH WARNINGS` → 🟡 Attention Required
   - `FAIL` → 🔴 Blocked

5. Build the document inventory — one row for every document the workflow defines, whether or not it exists yet, so the reader can see where work stopped:
   - **Project documents:** `01_project_brief.md` and `02_project_architecture.md`, each `Approved`, `Draft`, or `Not created`.
   - **ADRs:** every file in `{project-root}/adr/`, each `Proposed` or `Accepted`.
   - **Per feature** under `{project-root}/features/`: all five documents in workflow order, each `Approved`, `Draft`, or `Not created`; `03_api_spec.md` is `Not required` when `02_technical_design.md` says so. For `05_implementation_plan.md`, add the task counts and the `Verified` field. Include the feature's own ADRs.
   - **Per issue** under `{project-root}/issues/`: all four documents, each `Present` or `Not created`.
   - **Planned features:** if `01_project_brief.md` lists planned features (for example a Delivery Sequence table) that have no directory yet, list each as `Not started`.

6. Fill in the **Approved** column for every row:
   - A document's approval date is its `> Approved:` header field. If the document is `Approved` but has no such field, show `not recorded` — never infer a date from file timestamps or git history.
   - An ADR's date is the date of its `Accepted` row in Status History.
   - Anything not yet approved or accepted shows `—`.

7. Fill in the **Next action** column for every row, using the same ordering logic as `system_next_step`. Only the first unfinished step in each workflow gets a command; later steps wait on it:
   - `Draft` → `Review and approve`
   - `Proposed` ADR → `Review and accept`
   - The first `Not created` document in a workflow → the command that creates it
   - A later `Not created` document → `Waiting on step {N}`
   - Approved implementation plan with tasks remaining → `Implement — {N} tasks remaining`
   - All tasks Complete but not Verified → `Consistency review`
   - Nothing needed → `—`

8. Present the report directly in your response, in the format below.

9. Write the same report to `{project-root}/STATUS.md`, replacing any previous snapshot. **Do not modify** `project.yaml`, any document, or any other project file.

---

## Report Format

Paths in the Location column are relative to `{project-root}` and written as Markdown links, so they open from `STATUS.md`. Omit a section's table when it has no rows and say "None yet." instead.

```markdown
# Project Status: {Project Name}

> Generated: {date and time of this run}
> Project: {project-name}

This file is a snapshot written by `project_90_status`. It is only as current as its Generated time. Each document's own Status field and each implementation plan's task table remain the source of truth.

**Pick up here:** {command from system_next_step} — {one sentence of why}

| | |
|---|---|
| Project Health | {🟢 Healthy / 🟡 Attention Required / 🔴 Blocked} |
| Current State | {Project Initialized / Project Defined} |
| Consistency Result | {PASS / PASS WITH WARNINGS / FAIL} |
| Features | {N} total |
| Issues | {N} total, {M} open |
| ADRs | {N} Proposed, {N} Accepted |

## Project Documents

| Document | Location | Status | Approved | Next action |
|---|---|---|---|---|
| Project Brief | [01_project_brief.md](01_project_brief.md) | {Approved / Draft / Not created} | {date / —} | {action / —} |
| Architecture | [02_project_architecture.md](02_project_architecture.md) | {Approved / Draft / Not created} | {date / —} | {action / —} |
| ADR {NNN}: {title} | [adr/adr_{NNN}_{slug}.md](adr/adr_{NNN}_{slug}.md) | {Proposed / Accepted} | {date accepted / —} | {action / —} |

## Features

### {feature-name} — {state}

| Step | Document | Location | Status | Approved | Next action |
|---|---|---|---|---|---|
| 1 | Feature Brief | [features/{feature-name}/01_feature_brief.md](features/{feature-name}/01_feature_brief.md) | {Approved / Draft / Not created} | {date / —} | {action / —} |
| 2 | Technical Design | features/{feature-name}/02_technical_design.md | {Approved / Draft / Not created} | {date / —} | {action / —} |
| 3 | API Spec | features/{feature-name}/03_api_spec.md | {Approved / Draft / Not created / Not required} | {date / —} | {action / —} |
| 4 | Test Spec | features/{feature-name}/04_test_spec.md | {Approved / Draft / Not created} | {date / —} | {action / —} |
| 5 | Implementation Plan | features/{feature-name}/05_implementation_plan.md | {Approved / Draft / Not created} — {X}/{Y} tasks Complete; Verified: {value} | {date / —} | {action / —} |

(Repeat per feature, in the order they were created. For a planned feature with no directory, give a heading `### {feature-name} — Not started` and one row whose Next action is the `/feature_00_init` command, or `Waiting on {earlier feature}` when it depends on one that is not finished.)

## Issues

### {issue-id} — {Status} — {Category}

| Step | Document | Location | Status | Next action |
|---|---|---|---|---|
| 1 | Issue | issues/{issue-id}/01_issue.md | {Present / Not created} | {action / —} |
| 2 | Investigation | issues/{issue-id}/02_investigation.md | {Present / Not created} | {action / —} |
| 3 | Resolution | issues/{issue-id}/03_resolution.md | {Present / Not created} | {action / —} |
| 4 | Verification | issues/{issue-id}/04_verification.md | {Present / Not created} — Result: {Pass / Fail} | {action / —} |

(Repeat per issue. Issue documents have no approval step, so there is no Approved column.)

## Consistency Findings

{Each finding from the consistency review with its severity, or "None".}
```

## Notes

- `STATUS.md` is a convenience copy for people. No skill reads it to determine state — `system_workflow_resume`, `system_consistency_review`, and `system_next_step` always read the documents themselves.
- Only link a Location when the file exists; show the path as plain text when it is `Not created`.
- Do not duplicate the state-determination logic already implemented in `system_workflow_resume`, `system_consistency_review`, or `system_next_step` — this skill only formats, presents, and saves what they report.

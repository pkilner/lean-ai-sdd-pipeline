---
name: system_next_step
description: Determine the next command to run for a given project, feature, or issue by inspecting which artifacts already exist. Use after an interruption or whenever it's unclear where a workflow left off.
---

The arguments passed to this skill are the project name in kebab-case, optionally followed by a feature name or issue ID.

## Steps

1. If only a project name is given:
   - If `projects/{project-name}/` does not exist, recommend `/project_init {project-name} {repo-path}`.
   - Else if `project-brief.md` is missing, recommend `/project_brief {project-name}`.
   - Else if `architecture.md` is missing, recommend `/project_architecture {project-name}`.
   - Else, report the project workflow is complete and list any features/issues in progress (see step 2/3 for each), or recommend `/feature_init {project-name} {feature-name}` to start a new feature.

2. If a feature name is given, walk the feature workflow in order and recommend the first missing step:
   1. `feature-brief.md` missing → `/feature_brief {project-name} {feature-name}`
   2. `technical-design.md` missing → `/feature_technical_design {project-name} {feature-name}`
   3. `api-spec.md` missing and technical-design.md indicates this feature has contracts → `/feature_api_spec {project-name} {feature-name}` (otherwise skip this step)
   4. `test-spec.md` missing → `/feature_test_spec {project-name} {feature-name}`
   5. `implementation-plan.md` missing → `/feature_implementation_plan {project-name} {feature-name}`
   6. All documents present → implementation is the next step; recommend checking `implementation-plan.md`'s task table for the next incomplete task, and running `/system_consistency_review {project-name} {feature-name}` periodically.

3. If an issue ID is given, walk the issue workflow in order and recommend the first missing step:
   1. `issue.md` missing → the issue ID doesn't exist; tell the user to check the ID or run `/issue_capture`
   2. `issue.md` Category is "Pending classification" → `/system_issue_classify {project-name} {issue-id}`
   3. `investigation.md` missing → `/issue_investigate {project-name} {issue-id}`
   4. `resolution.md` missing → `/issue_resolve {project-name} {issue-id}`
   5. `verification.md` missing → `/issue_verify {project-name} {issue-id}`
   6. All present and `issue.md` Status is `Closed` → issue is done.

4. Report the recommendation as a single clear next command, with one sentence of why.

---
name: system_next_step
description: (Internal — not for direct user invocation.) Determine the next command for a project, feature, or issue by inspecting which artifacts exist, their Draft/Approved status, and task Status values. Invoked by Claude — at session start via system_workflow_resume, or whenever it's unclear where a workflow left off.
user-invocable: false
---

**This is a system skill.** Each user-facing skill already states its own literal next step at the end — this skill exists for the cases where that isn't enough: resuming after an interruption, or when the user asks in plain language ("what should I do now?", "where were we?") rather than by naming a skill. Claude invokes it internally; it is not something a user types as a command.

The arguments passed to this skill are the project name in kebab-case, optionally followed by a feature name or issue ID.

## Steps

1. If only a project name is given:
   - If `projects/{project-name}/` does not exist → recommend `/project_init {project-name} {repo-path}`.
   - Else if `project-brief.md` is missing → recommend `/project_brief {project-name}`.
   - Else if `project-brief.md` exists but `Status: Draft` → report it needs review; recommend confirming approval before continuing.
   - Else if `architecture.md` is missing → recommend `/project_architecture {project-name}`.
   - Else if `architecture.md` exists but `Status: Draft` → report it needs review; recommend confirming approval before continuing.
   - Else → project is **Project Defined**. List any features/issues in progress (steps 2/3), or recommend `/feature_init {project-name} {feature-name}` to start a new feature.

2. If a feature name is given, walk the feature workflow in order and recommend the first missing or unapproved step:
   1. `feature-brief.md` missing → `/feature_brief {project-name} {feature-name}`
   2. `feature-brief.md` is `Draft` → needs review/approval
   3. `technical-design.md` missing → `/feature_technical_design {project-name} {feature-name}`
   4. `technical-design.md` is `Draft` → needs review/approval
   5. `technical-design.md`'s `API Specification Required` is `Yes` and `api-spec.md` is missing → `/feature_api_spec {project-name} {feature-name}`
   6. `api-spec.md` exists but is `Draft` → needs review/approval
   7. `test-spec.md` missing → `/feature_test_spec {project-name} {feature-name}`
   8. `test-spec.md` is `Draft` → needs review/approval
   9. `implementation-plan.md` missing → `/feature_implementation_plan {project-name} {feature-name}`
   10. `implementation-plan.md` is `Draft` → needs review/approval
   11. All approved, all tasks `Not Started` → **Feature Specified**; next step is to begin implementation.
   12. Some tasks `In Progress` or `Complete`, not all `Complete` → **Feature In Progress**; report which tasks remain.
   13. All tasks `Complete`, `Verified: Not yet` → **Feature Implemented**; a consistency review is pending or should be triggered.
   14. All tasks `Complete` and `Verified` is set → **Feature Verified**.

   Never report a feature as built merely because `implementation-plan.md` exists — that only reaches "Feature Specified"; task Status determines everything from there.

3. If an issue ID is given, walk the issue workflow in order:
   1. `issue.md` missing → the issue ID doesn't exist; tell the user to check the ID or run `/issue_capture`
   2. `Category`/`Complexity` still "Pending classification" → classification should have run automatically; if it didn't, something interrupted `issue_capture` — retry it
   3. No investigation recorded (file or inline) → `/issue_investigate {project-name} {issue-id}`
   4. No resolution recorded → `/issue_resolve {project-name} {issue-id}`
   5. No verification recorded → `/issue_verify {project-name} {issue-id}`
   6. `issue.md` Status is `Closed` → issue is done; its closing consistency review already ran automatically.

4. Report the recommendation as a single clear next command, with one sentence of why.

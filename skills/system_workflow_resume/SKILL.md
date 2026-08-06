---
name: system_workflow_resume
description: (Internal — not for direct user invocation.) Onboard to the current state of a project and deliver a briefing covering what exists, what is in progress, open issues, and what to do next — using Project Initialized / Project Defined / Feature Specified / Feature In Progress / Feature Implemented / Feature Verified state labels. Invoked automatically by Claude at the start of a session (see CLAUDE.md), or whenever the user asks to be re-oriented.
user-invocable: false
---

**This is a system skill.** Claude invokes it on its own at the start of a session per this repo's `CLAUDE.md`, or whenever the user asks something like "where were we" or "what's the state of this project" — it is not a command the user is expected to type themselves.

The argument is the project name in kebab-case. If omitted, list every project under `projects/` with a one-line status for each instead of a full briefing.

## Steps

1. Read `projects/{project-name}/project.yaml`, `01_project_brief.md`, and `02_project_architecture.md` (skip any that don't exist yet — report their absence instead of failing). Determine project state:
   - **Project Initialized** — `project.yaml` exists; `01_project_brief.md` and/or `02_project_architecture.md` missing or still `Draft`.
   - **Project Defined** — both `01_project_brief.md` and `02_project_architecture.md` exist with `Status: Approved`.

2. List every directory under `projects/{project-name}/features/`. For each, determine state by walking the same logic as `system_next_step`:
   - **(in definition)** — one or more of 01_feature_brief/02_technical_design/03_api_spec/04_test_spec/05_implementation_plan missing or still `Draft`. Report which document is the blocker.
   - **Feature Specified** — all required documents `Approved`, all implementation tasks `Not Started`.
   - **Feature In Progress** — some tasks `In Progress` or `Complete`, not all `Complete`.
   - **Feature Implemented** — all tasks `Complete`, but `Verified: Not yet`.
   - **Feature Verified** — all tasks `Complete` and `Verified` is set (consistency review passed).

   Never report a feature as Implemented or Verified on the strength of `05_implementation_plan.md` existing alone — that only means Specified.

3. List every directory under `projects/{project-name}/issues/`. For each, read `01_issue.md`'s Status and Category.

4. Deliver a concise briefing covering:
   - **What this project is** (one sentence, from `01_project_brief.md`)
   - **What has been built** — features at Implemented or Verified
   - **What is currently in progress** — features In Progress or still in definition, and open (non-`Closed`) issues
   - **Open decisions** — from `02_project_architecture.md`'s Open Decisions table
   - **Recommended next action** — invoke the logic in `system_next_step` for the most relevant in-progress feature or issue, or for the project itself if nothing is in progress

5. If `projects/{project-name}/` doesn't exist yet, say so and suggest running `/project_00_init {project-name} {repo-path}` to start it.

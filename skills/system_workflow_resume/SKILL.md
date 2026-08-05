---
name: system_workflow_resume
description: Onboard to the current state of a project and deliver a briefing covering what exists, what is in progress, open issues, and what to do next. Run at the start of a session, or whenever picking work back up after a break.
---

The argument passed to this skill is the project name in kebab-case. If omitted, list every project under `projects/` with a one-line status for each instead of a full briefing.

## Steps

1. Read `projects/{project-name}/project.yaml`, `project-brief.md`, and `architecture.md` (skip any that don't exist yet — report their absence instead of failing).

2. List every directory under `projects/{project-name}/features/`. For each, check which of `feature-brief.md`, `technical-design.md`, `api-spec.md`, `test-spec.md`, `implementation-plan.md` exist to determine how far along it is.

3. List every directory under `projects/{project-name}/issues/`. For each, read `issue.md`'s Status and Category.

4. Deliver a concise briefing covering:
   - **What this project is** (one sentence, from `project-brief.md`)
   - **What has been built** — features with a complete `implementation-plan.md`
   - **What is currently in progress** — features with partial documents, and open (non-`Closed`) issues
   - **Open decisions** — from `architecture.md`'s Open Decisions table
   - **Recommended next action** — invoke the logic in `system_next_step` for the most relevant in-progress feature or issue, or for the project itself if nothing is in progress

5. If `projects/{project-name}/` doesn't exist yet, say so and suggest running `/project_init {project-name} {repo-path}` to start it.

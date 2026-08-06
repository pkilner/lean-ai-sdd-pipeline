---
name: project_90_status
description: Display a status summary for a project — health, current state, document status, feature/issue/ADR summary, consistency result, and recommended next step. Internally invokes system_workflow_resume, system_consistency_review, and system_next_step. Display only — never writes or modifies any file.
---

The argument is the project name in kebab-case.

## Steps

1. Internally invoke `system_workflow_resume {project-name}` to get the project's current state: `01_project_brief.md` / `02_project_architecture.md` status, the list of features with their states (in definition / Feature Specified / Feature In Progress / Feature Implemented / Feature Verified), and the list of issues with their statuses.

2. Internally invoke `system_consistency_review {project-name}` (project-wide — no feature name) to get a `PASS` / `PASS WITH WARNINGS` / `FAIL` verdict.

3. Internally invoke `system_next_step {project-name}` to get the single recommended next command.

4. Map the consistency verdict to a health indicator:
   - `PASS` → 🟢 Healthy
   - `PASS WITH WARNINGS` → 🟡 Attention Required
   - `FAIL` → 🔴 Blocked

5. Count ADRs in `projects/{project-name}/adr/` by Status (`Proposed` vs `Accepted`).

6. Present the summary directly in your response, in the format below. **Do not write this to a file** — this skill displays status, it does not generate a report document. **Do not modify** `project.yaml`, any document, or any other project file.

---

## Output Format

```text
Project Status: {Project Name}

Project Health: {🟢 Healthy / 🟡 Attention Required / 🔴 Blocked}
Current State: {Project Initialized / Project Defined}

Project Brief:  {Draft / Approved / missing}
Architecture:   {Draft / Approved / missing}

Features ({N} total):
- {feature-name}: {state}
- ...

Issues ({N} total, {M} open):
- {issue-id}: {Status} — {Category}
- ...

ADRs: {N} Proposed, {N} Accepted

Consistency Result: {PASS / PASS WITH WARNINGS / FAIL}

Recommended Next Step: {command from system_next_step}
```

## Notes

- This skill never writes to disk. It is a read-only display over the combined output of the three system skills it invokes.
- Do not duplicate the state-determination logic already implemented in `system_workflow_resume`, `system_consistency_review`, or `system_next_step` — this skill only formats and presents what they report.

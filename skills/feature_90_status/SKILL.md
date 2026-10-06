---
name: feature_90_status
description: Display a status summary for a single feature — health, current state, document status, task summary, test summary, consistency result, and recommended next step. Internally invokes system_workflow_resume, system_consistency_review, and system_next_step. Display only — never writes or modifies any file.
---

The arguments are the project name in kebab-case, followed by the feature name in kebab-case.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Steps

1. Internally invoke `system_workflow_resume {project-name}` and read this feature's entry from the briefing it produces (its current state, and which document is blocking progress if it's still in definition).

2. Internally invoke `system_consistency_review {project-name} {feature-name}` to get the feature-scoped `PASS` / `PASS WITH WARNINGS` / `FAIL` verdict.

3. Internally invoke `system_next_step {project-name} {feature-name}` to get the recommended next command.

4. Map the consistency verdict to a health indicator:
   - `PASS` → 🟢 Healthy
   - `PASS WITH WARNINGS` → 🟡 Attention Required
   - `FAIL` → 🔴 Blocked

5. Read `02_feature_plan.md` (if it exists) to summarize task Status counts, the test count, and the Required/Optional/Not Applicable declaration for each test level.

6. Present the summary directly in your response, in the format below. **Do not write this to a file.** **Do not modify** any document or code file.

---

## Output Format

```text
Feature Status: {feature-name} ({project-name})

Feature Health: {🟢 Healthy / 🟡 Attention Required / 🔴 Blocked}
Current State: {in definition (blocked on X) / Feature Specified / Feature In Progress / Feature Implemented / Feature Verified}

Documents:
- 01_feature_spec.md:  {Draft / Approved on {date} / missing}
- 02_feature_plan.md:  {Draft / Approved on {date} / missing}

Tasks: {X}/{Y} Complete, {Z} In Progress, {W} Blocked, {V} Not Started

Tests: {N} total — Unit {Required/Optional/Not Applicable}, Integration {Required/Optional/Not Applicable}, End-to-End {Required/Optional/Not Applicable}

Consistency Result: {PASS / PASS WITH WARNINGS / FAIL}

Recommended Next Step: {command from system_next_step}
```

## Notes

- This skill never writes to disk. It is a read-only display over the combined output of the three system skills it invokes.
- Do not duplicate the state-determination logic already implemented in `system_workflow_resume`, `system_consistency_review`, or `system_next_step` — this skill only formats and presents what they report.

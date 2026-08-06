---
name: issue_03_resolve
description: Resolve an investigated issue — the fourth step in the issue workflow. Applies the recommended corrective action (spec update, code fix, or both) and records what changed in 03_resolution.md. Pass the project name and issue ID.
---

The arguments passed to this skill are the project name in kebab-case, followed by the issue ID.

## Steps

1. Read `projects/{project-name}/issues/{issue-id}/02_investigation.md` (required — do not proceed if missing).

2. **Before touching the application repository:**
   - Re-confirm `repo_path` from `project.yaml` resolves to a real git repository (per the checks in `project_00_init` — do not assume it still holds).
   - Run `git -C {repo_path} status` and note the current state. If there is unrelated uncommitted work, tell the user and confirm how to proceed rather than editing over it.
   - Only touch files required to implement the recommended corrective action — do not opportunistically refactor or "clean up" nearby code.

3. Apply the recommended corrective action from the investigation:
   - If it requires a spec update, update the relevant document(s) under `projects/{project-name}/features/{feature-name}/` (or `02_project_architecture.md` / `01_project_brief.md` for project-wide issues) and follow the standard top-down propagation order (01_feature_brief → 02_technical_design → 03_api_spec → 04_test_spec → 05_implementation_plan).
   - If it requires a code fix, implement it in the application repository.
   - If it requires both, do the spec update first, then the code fix, so the code is brought into line with an already-corrected spec.
   - **Never run `git commit` or `git push` in the application repository as part of this skill.** Leave changes staged/unstaged for the user to review and commit themselves.

4. Generate `projects/{project-name}/issues/{issue-id}/03_resolution.md` using the template below.

5. List every document and file actually changed, including files in the application repository — this list is both part of the output document and something you state plainly to the user.

6. Update `01_issue.md`'s Status to `Resolved` and add a Status History entry.

7. After writing, confirm to the user what was changed (spec files and/or application files) and recommend the next step.

---

## Output Template (`03_resolution.md`)

```markdown
# Resolution: {ISSUE-xxxx}

> Date: {today's date}
> Project: {project-name}

## Summary

One or two sentences describing what was done to resolve the issue.

## Changes Made

### Specification Changes

| Document | Change |
|---|---|
| (e.g.) features/{feature}/03_api_spec.md | Corrected field name from X to Y |

(Leave the table with "None" if no spec changes were needed.)

### Code Changes

| File (in application repo) | Change |
|---|---|
| (e.g.) src/api/routes.ts | Fixed validation to match 03_api_spec.md |

(Leave the table with "None" if no code changes were needed. Not committed or pushed — left for the user to review.)

## Verification Plan

What needs to be checked to confirm this resolution actually fixes the issue (tests to run, manual checks to perform). This is executed by `/issue_04_verify`, not here.
```

**Next step:** Run `/issue_04_verify {project-name} {issue-id}`.

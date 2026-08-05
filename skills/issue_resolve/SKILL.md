---
name: issue_resolve
description: Resolve an investigated issue — the fourth step in the issue workflow. Applies the recommended corrective action (spec update, code fix, or both) and records what changed. Pass the project name and issue ID.
---

The arguments passed to this skill are the project name in kebab-case, followed by the issue ID.

## Steps

1. Read `projects/{project-name}/issues/{issue-id}/investigation.md` (required — do not proceed if missing).

2. Apply the recommended corrective action from the investigation:
   - If it requires a spec update, update the relevant document(s) under `projects/{project-name}/features/{feature-name}/` (or `architecture.md` / `project-brief.md` for project-wide issues) and follow the standard top-down propagation order (feature-brief → technical-design → api-spec → test-spec → implementation-plan).
   - If it requires a code fix, implement it in the application repository (`repo_path`).
   - If it requires both, do the spec update first, then the code fix, so the code is brought into line with an already-corrected spec.

3. Generate `projects/{project-name}/issues/{issue-id}/resolution.md` using the template below, listing every document and file actually changed.

4. Update `issue.md`'s Status to `Resolved` and add a Status History entry.

5. After writing the file, confirm to the user what was changed and recommend the next step.

---

## Output Template

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
| (e.g.) features/{feature}/api-spec.md | Corrected field name from X to Y |

(Leave the table with "None" if no spec changes were needed.)

### Code Changes

| File (in application repo) | Change |
|---|---|
| (e.g.) src/api/routes.ts | Fixed validation to match api-spec.md |

(Leave the table with "None" if no code changes were needed.)

## Verification Plan

What needs to be checked to confirm this resolution actually fixes the issue (tests to run, manual checks to perform). This is executed by `/issue_verify`, not here.
```

**Next step:** Run `/issue_verify {project-name} {issue-id}`.

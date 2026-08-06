---
name: issue_90_status
description: Display a status summary for one issue when an issue ID is supplied, or a bucketed summary across all issues in a project when it is omitted. Internally invokes system_next_step (single-issue mode) and system_workflow_resume (list mode). Display only — never writes or modifies any file.
---

The first argument is the project name in kebab-case. The second, optional, argument is an issue ID.

## Steps

### If an issue ID is supplied

1. Read `01_issue.md` (`Category`, and the `## Impact / Severity` section as Severity) and whichever of `02_investigation.md`, `03_resolution.md`, `04_verification.md` exist.

2. Internally invoke `system_next_step {project-name} {issue-id}` to get the recommended next command.

3. Determine the current workflow stage from `Status` and which files exist: `Open` (no investigation yet), `Investigating` (investigation exists, not yet resolved), `Resolved` (awaiting verification), `Closed`.

4. Present:

```text
Issue Status: {issue-id} ({project-name})

Category: {Category, or "Pending Investigation"}
Severity: {content of the Impact / Severity section, or "Not recorded"}
Current Stage: {Open / Investigating / Resolved / Closed}

Investigation:  {Complete — see 02_investigation.md / Not started}
Resolution:     {Complete — see 03_resolution.md / Not started}
Verification:   {Pass / Fail / Not started}

Recommended Next Step: {command from system_next_step}
```

### If no issue ID is supplied

1. Internally invoke `system_workflow_resume {project-name}` to get the full issue list with Status and Category.

2. Bucket each issue by Status and which files exist:
   - **Open** — `Status: Open`, no `02_investigation.md` yet
   - **Investigating** — `Status: Open`, `02_investigation.md` exists but not yet resolved
   - **Resolved issues awaiting verification** — `Status: Resolved`
   - **Closed** — `Status: Closed`

3. Present:

```text
Issue Summary: {project-name}

Open ({N}): {issue-id, issue-id, ...}
Investigating ({N}): {issue-id, ...}
Resolved, awaiting verification ({N}): {issue-id, ...}
Closed ({N}): {issue-id, ...}
```

Do not write either output to a file. Do not modify any issue file.

## Notes

- This skill never writes to disk. It is a read-only display over the combined output of `system_next_step` / `system_workflow_resume` and direct reads of the issue's own files.
- Unlike `project_90_status` and `feature_90_status`, this skill does **not** display a health indicator. `system_consistency_review` verdicts are scoped to a project or a feature, not to an individual issue, so there is no PASS/PASS WITH WARNINGS/FAIL source to map a health indicator from at issue granularity — fabricating one would misrepresent what was actually checked.

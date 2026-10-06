---
name: issue_90_status
description: Display a status summary for one issue when an issue ID is supplied, or a bucketed summary across all issues in a project when it is omitted. Internally invokes system_next_step (single-issue mode) and system_workflow_resume (list mode). Display only — never writes or modifies any file.
---

The first argument is the project name in kebab-case. The second, optional, argument is an issue ID.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Steps

### If an issue ID is supplied

1. Read `{project-root}/issues/{issue-id}.md`: its `Status`, `Category`, `Related`, the Impact / Severity line, and which of sections 2–4 have been filled in.

2. Internally invoke `system_next_step {project-name} {issue-id}` to get the recommended next command.

3. Present:

```text
Issue Status: {issue-id} ({project-name})

Category: {Category, or "Pending Investigation"}
Severity: {Impact / Severity, or "Not recorded"}
Related: {features, acceptance criteria, ADRs — or "None recorded"}
Current Stage: {Open / Investigated / Resolved / Closed}

Investigation:  {Complete / Not started}
Resolution:     {Complete / Not started}
Verification:   {Pass / Fail / Not started}

Recommended Next Step: {command from system_next_step}
```

### If no issue ID is supplied

1. Internally invoke `system_workflow_resume {project-name}` to get the full issue list with Status, Category, and Related.

2. Bucket each issue by its `Status` field and present:

```text
Issue Summary: {project-name}

Open ({N}): {issue-id, issue-id, ...}
Investigated ({N}): {issue-id, ...}
Resolved, awaiting verification ({N}): {issue-id, ...}
Closed ({N}): {issue-id, ...}

By feature:
- {feature-name}: {issue-id (Status), ...}
- (project-wide): {issue-id (Status), ...}
```

Do not write either output to a file. Do not modify any issue file.

## Notes

- This skill never writes to disk.
- Unlike `project_90_status` and `feature_90_status`, this skill does **not** display a health indicator. `system_consistency_review` verdicts are scoped to a project or a feature, not to an individual issue, so there is no verdict to map one from.

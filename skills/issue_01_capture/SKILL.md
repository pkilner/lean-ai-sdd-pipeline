---
name: issue_01_capture
description: Capture a new issue — the first step in the issue workflow. Creates sdd/issues/ISSUE-xxxx/ in the project's application repository with an 01_issue.md describing what was observed, then automatically classifies it. Issues are siblings of features, not nested beneath them.
---

The arguments passed to this skill are the project name in kebab-case, followed by a short free-text description of what was observed (e.g. `my-app "sync fails silently when offline for more than 24h"`).

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Steps

1. Read `{project-root}/project.yaml` (required — do not proceed if the project has not been initialized).

2. Scan `{project-root}/issues/` for existing `ISSUE-xxxx` directories to determine the next number, zero-padded to 4 digits (e.g. if `ISSUE-0007` exists, next is `ISSUE-0008`; if none exist, start at `ISSUE-0001`).

3. Create `{project-root}/issues/ISSUE-{xxxx}/`.

4. Generate `01_issue.md` using the template below.

5. **Automatically invoke `system_issue_classify {project-name} {issue-id}`** before ending your turn — do not wait for the user to request it. This is an internal orchestration step, not something the user needs to trigger. Present the resulting Category to the user alongside the captured issue.

---

## Output Template

```markdown
# Issue {ISSUE-xxxx}: {Short Title}

> Status: Open
> Category: Pending classification
> Created: {today's date}
> Project: {project-name}
> Reported by: {user, if known}

## Description

What was observed, in the reporter's own words.

## Steps to Reproduce

(Only if applicable — omit for change requests or non-reproducible issues)

1. ...
2. ...

**Expected:** What should have happened.
**Actual:** What actually happened.

## Related Features / ADRs

List any features (`features/{feature-name}/`) or ADRs this issue appears to relate to. Leave blank if unknown — this will be filled in during investigation.

## Impact / Severity

If known: who/what is affected, and how badly.

## Status History

| Date | Status | Note |
|---|---|---|
| {today's date} | Open | Captured |
```

**Next step:** classification happens automatically as part of this skill. Once you see the Category, run `/issue_02_investigate {project-name} {issue-id}`.

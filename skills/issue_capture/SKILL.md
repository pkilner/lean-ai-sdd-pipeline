---
name: issue_capture
description: Capture a new issue — the first step in the issue workflow. Creates projects/{project-name}/issues/ISSUE-xxxx/ with an issue.md describing what was observed, then automatically classifies it. Issues are siblings of features, not nested beneath them.
---

The arguments passed to this skill are the project name in kebab-case, followed by a short free-text description of what was observed (e.g. `my-app "sync fails silently when offline for more than 24h"`).

## Steps

1. Read `projects/{project-name}/project.yaml` (required — do not proceed if the project has not been initialized).

2. Scan `projects/{project-name}/issues/` for existing `ISSUE-xxxx` directories to determine the next number, zero-padded to 4 digits (e.g. if `ISSUE-0007` exists, next is `ISSUE-0008`; if none exist, start at `ISSUE-0001`).

3. Create `projects/{project-name}/issues/ISSUE-{xxxx}/`.

4. Generate `issue.md` using the template below.

5. **Automatically invoke `system_issue_classify {project-name} {issue-id}`** before ending your turn — do not wait for the user to request it. This is an internal orchestration step, not something the user needs to trigger. Present the resulting Category and Complexity to the user alongside the captured issue.

---

## Output Template

```markdown
# Issue {ISSUE-xxxx}: {Short Title}

> Status: Open
> Category: Pending classification
> Complexity: Pending classification
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

List any features (`projects/{project-name}/features/{feature-name}/`) or ADRs this issue appears to relate to. Leave blank if unknown — this will be filled in during investigation.

## Impact / Severity

If known: who/what is affected, and how badly.

## Status History

| Date | Status | Note |
|---|---|---|
| {today's date} | Open | Captured |
```

**Next step:** classification happens automatically as part of this skill. Once you see the Category and Complexity, run `/issue_investigate {project-name} {issue-id}` — unless Complexity is `Simple` and the root cause is already obvious from the description, in which case you may resolve it directly (see `issue_investigate` for the lightweight path).

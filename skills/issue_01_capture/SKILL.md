---
name: issue_01_capture
description: Capture a new issue — the first step in the issue workflow. Creates sdd/issues/ISSUE-xxxx.md in the project's application repository describing what was observed, then automatically classifies it. One file holds the issue's whole lifecycle. Issues are siblings of features, not nested beneath them.
---

The arguments passed to this skill are the project name in kebab-case, followed by a short free-text description of what was observed (e.g. `my-app "sync fails silently when offline for more than 24h"`).

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Steps

1. Read `{project-root}/project.yaml` (required — do not proceed if the project has not been initialized).

2. Scan `{project-root}/issues/` for existing `ISSUE-xxxx.md` files to determine the next number, zero-padded to 4 digits (e.g. if `ISSUE-0007.md` exists, next is `ISSUE-0008`; if none exist, start at `ISSUE-0001`). Create `{project-root}/issues/` if it does not exist.

3. Generate `{project-root}/issues/ISSUE-{xxxx}.md` using the template below. Sections 2–4 stay as `_Not started._` — `issue_02_investigate`, `issue_03_resolve`, and `issue_04_verify` each fill in their own section of this same file.

4. Fill in **Related** with any features, acceptance criteria (`{feature-name} AC-N`), or ADRs the issue clearly points at. Leave it blank if unknown — investigation completes it.

5. **Automatically invoke `system_issue_classify {project-name} {issue-id}`** before ending your turn — do not wait for the user to request it. Present the resulting Category to the user alongside the captured issue.

**Keep the status current:** whenever this skill creates its document or changes a Status, finish by running `project_90_status {project-name}` so `STATUS.md` matches before ending your turn.

---

## Output Template

```markdown
# {ISSUE-xxxx}: {Short Title}

> Status: Open
> Category: Pending classification
> Created: {today's date}
> Project: {project-name}
> Reported by: {user, if known}
> Related: {features, acceptance criteria, ADRs — or blank}

## 1. Issue

**Description:** What was observed, in the reporter's own words.

**Steps to reproduce:** (omit for change requests or non-reproducible issues)

1. ...

**Expected:** What should have happened.
**Actual:** What actually happened.

**Impact / Severity:** Who or what is affected, and how badly, if known.

## 2. Investigation

_Not started._

## 3. Resolution

_Not started._

## 4. Verification

_Not started._

## Status History

| Date | Status | Note |
|---|---|---|
| {today's date} | Open | Captured |
```

**Next step:** classification happens automatically as part of this skill. Once you see the Category, run `/issue_02_investigate {project-name} {issue-id}`.

## Single project backlog

Maintain `<repo_path>/BACKLOG.md` as the project-wide work queue, with Backlog and Delivered sections. Reuse an existing canonical project queue when the project explicitly names another location; never create a competing website-only queue. Preserve item IDs, deduplicate requests, and link every open issue to its detailed issue file. Capture accepted deferrals as open work; move an item to Delivered only with recorded completion/acceptance. Archived history is evidence, not current status. During capture, scope changes and closeout, reconcile affected entries. Keep this detail out of the concise STATUS.md table.

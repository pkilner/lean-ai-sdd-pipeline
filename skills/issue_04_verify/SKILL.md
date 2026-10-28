---
name: issue_04_verify
description: Verify a resolved issue — the final step in the issue workflow. Confirms the resolution actually fixes the issue, records the Verification section of the issue's file, closes it out, and automatically runs a consistency review. Pass the project name and issue ID.
---

The arguments passed to this skill are the project name in kebab-case, followed by the issue ID.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Verify the failing boundary

When a defect crosses components, verify the connected path with its real output shape and relevant execution identity, not only each component separately. Local or owner-credential success is not live/restricted-identity proof. State what was actually exercised and any remaining verification blocker. No mandatory test-first sequence is implied.

## Steps

1. Read `{project-root}/issues/{issue-id}.md` (required — do not proceed if missing or if its Resolution section is still `_Not started._`).

2. Execute the verification plan from the Resolution section: run the relevant tests (see the feature's `02_feature_plan.md` if applicable) and perform any other checks it describes.

   Verify the reported failure and relevant regression risks, not just that code changed. Record actual outcomes; missing pre-fix red/green evidence alone is not a verification failure. Required checks and unresolved failures still block closure.

3. Replace the `_Not started._` placeholder under `## 4. Verification` with the section template below.

4. If verification passes, update the header `Status` to `Closed` and add a Status History row. If it fails, set `Status` back to `Open`, note why in Status History, and tell the user this needs to return to `/issue_02_investigate` or `/issue_03_resolve`.

5. **If the issue is now Closed, automatically invoke `system_consistency_review {project-name} {feature-name}`** for each feature in **Related** before ending your turn — do not wait for the user to request it. This confirms the fix didn't leave residual drift. If the issue was project-wide rather than feature-specific, run it without a feature name.

6. Confirm the outcome to the user, including the consistency review result.

**Keep the status current:** whenever this skill creates its document or changes a Status, finish by running `project_90_status {project-name}` so `STATUS.md` matches before ending your turn.

---

## Section Template (`## 4. Verification`)

```markdown
**Date:** {today's date}

**Steps performed:**

1. ...

**Result:** Pass / Fail

**Evidence:** Test output, command results, or observation confirming (or disconfirming) the fix.

**Outcome:** If Pass, the issue is closed. If Fail, what still fails and why this returns to investigation or resolution.
```

**Next step:** consistency review runs automatically once closed. If verification failed, return to `/issue_02_investigate {project-name} {issue-id}`.

## Single project backlog

Maintain `<repo_path>/BACKLOG.md` as the project-wide work queue, with Backlog and Delivered sections. Reuse an existing canonical project queue when the project explicitly names another location; never create a competing website-only queue. Preserve item IDs, deduplicate requests, and link every open issue to its detailed issue file. Capture accepted deferrals as open work; move an item to Delivered only with recorded completion/acceptance. Archived history is evidence, not current status. During capture, scope changes and closeout, reconcile affected entries. Keep this detail out of the concise STATUS.md table.

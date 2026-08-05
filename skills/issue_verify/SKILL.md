---
name: issue_verify
description: Verify a resolved issue — the fifth and final step in the issue workflow. Confirms the resolution actually fixes the issue, records verification.md, closes it out, and automatically runs a consistency review. Pass the project name and issue ID.
---

The arguments passed to this skill are the project name in kebab-case, followed by the issue ID.

## Steps

1. Read `projects/{project-name}/issues/{issue-id}/resolution.md` (required — do not proceed if missing).

2. Execute the Verification Plan from the resolution: run the relevant tests (see the feature's `test-spec.md` if applicable) and/or perform the manual checks it describes.

3. Generate `projects/{project-name}/issues/{issue-id}/verification.md` using the template below.

4. If verification passes, update `issue.md`'s Status to `Closed` and add a Status History entry. If verification fails, update Status back to `Open`, note why in Status History, and tell the user this needs to return to `/issue_investigate` or `/issue_resolve`.

5. **If the issue is now Closed, automatically invoke `system_consistency_review {project-name} {feature-name}`** for any feature(s) this issue touched (from `Related Features / ADRs`) before ending your turn — do not wait for the user to request it. This confirms the fix didn't leave residual drift. If the issue was project-wide rather than feature-specific, run it without a feature name to check the whole project.

6. After writing, confirm the outcome to the user, including the consistency review result.

---

## Output Template (`verification.md`)

```markdown
# Verification: {ISSUE-xxxx}

> Date: {today's date}
> Project: {project-name}

## Verification Steps Performed

1. ...
2. ...

## Result

Pass / Fail

## Evidence

Test output, command results, or manual observation confirming (or disconfirming) the fix.

## Outcome

If Pass: Issue is closed.
If Fail: Explain what still fails and why this returns to investigation or resolution.
```

**Next step:** consistency review runs automatically once closed. If verification failed, return to `/issue_investigate {project-name} {issue-id}`.

---
name: issue_investigate
description: Investigate a classified issue — the third step in the issue workflow. Determines root cause and classifies the code/documentation mismatch using the Reconciliation model, recorded in investigation.md. Pass the project name and issue ID.
---

The arguments passed to this skill are the project name in kebab-case, followed by the issue ID (e.g. `my-app ISSUE-0008`).

## Steps

1. Read `projects/{project-name}/issues/{issue-id}/issue.md` (required — do not proceed if missing, or if `Category` is still "Pending classification"; tell the user to run `/issue_capture` again, or `system_issue_classify` will already have run automatically as part of that).

2. Read any features or ADRs referenced in `Related Features / ADRs`, plus `projects/{project-name}/architecture.md` to locate the relevant layer and repository path.

3. Inspect the application repository (`repo_path` in `projects/{project-name}/project.yaml`) for the relevant implementation. This step is read-only — do not modify application code here.

4. If code and documentation disagree, do not assume either is correct — classify the mismatch using the Reconciliation model:
   - **Code defect** — implementation does not match agreed spec and documentation is correct
   - **Documentation drift** — implementation is correct but docs were never updated
   - **Unapproved implementation change** — someone changed behaviour without going through the spec workflow
   - **Ambiguous intent** — the spec itself doesn't clearly say what should happen
   - **Scope change** — the requirement itself has changed since the spec was written

5. Generate `projects/{project-name}/issues/{issue-id}/investigation.md` using the template below.

6. If `issue.md`'s `Category` is `Pending Investigation`, this investigation has now produced enough evidence to classify it properly — set `Category` to the correct value from the list in `system_issue_classify` before finishing.

7. Update `issue.md`'s Status History with an "Investigating" or "Investigated" entry.

8. After writing, present findings and the recommended corrective action to the user.

---

## Output Template (`investigation.md`)

```markdown
# Investigation: {ISSUE-xxxx}

> Date: {today's date}
> Project: {project-name}

## Root Cause

What is actually causing the observed behaviour. Be specific — reference files, functions, or spec sections.

## Affected Components / Layers

List the system layers and components involved (use layer names from `architecture.md`).

## Reconciliation Classification

**Classification:** Code defect / Documentation drift / Unapproved implementation change / Ambiguous intent / Scope change

**Reasoning:** Why this classification fits, with evidence from both the spec documents and the implementation.

## Related Features / ADRs

Confirmed list of features and ADRs this issue touches (update `issue.md` if this differs from what was recorded at capture time).

## Recommended Corrective Action

What should happen next, and where (which document(s) to update, which code to change, or both). Do not perform the fix here — this is a diagnostic document.

## Open Questions

Leave blank if none.
```

**Next step:** Run `/issue_resolve {project-name} {issue-id}`.

---
name: issue_02_investigate
description: Investigate a classified issue — the second user step in the issue workflow. Determines root cause and classifies the code/documentation mismatch using the Reconciliation model, recorded in the Investigation section of the issue's file. Pass the project name and issue ID.
---

The arguments passed to this skill are the project name in kebab-case, followed by the issue ID (e.g. `my-app ISSUE-0008`).

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Steps

1. Read `{project-root}/issues/{issue-id}.md` (required — do not proceed if missing, or if `Category` is still "Pending classification"; classification should have run automatically inside `/issue_01_capture`).

2. Read any features or ADRs listed in **Related**, plus `{project-root}/02_project_architecture.md` to locate the relevant layer and repository path.

3. Inspect the application repository (`repo_path` in the project pointer) for the relevant implementation. This step is read-only — do not modify application code here.

4. If code and documentation disagree, do not assume either is correct — classify the mismatch using the Reconciliation model:
   - **Code defect** — implementation does not match agreed spec and documentation is correct
   - **Documentation drift** — implementation is correct but docs were never updated
   - **Unapproved implementation change** — someone changed behaviour without going through the spec workflow
   - **Ambiguous intent** — the spec itself doesn't clearly say what should happen
   - **Scope change** — the requirement itself has changed since the spec was written

5. Replace the `_Not started._` placeholder under `## 2. Investigation` with the section template below.

6. Update the header: set `Status` to `Investigated`; complete `Related` with the features, acceptance criteria, and ADRs the issue actually touches; and if `Category` is `Pending Investigation`, set it to the correct value from the list in `system_issue_classify`. Add a Status History row.

7. Present findings and the recommended corrective action to the user.

**Keep the status current:** whenever this skill creates its document or changes a Status, finish by running `project_90_status {project-name}` so `STATUS.md` matches before ending your turn.

---

## Section Template (`## 2. Investigation`)

```markdown
**Date:** {today's date}

**Root cause:** What is actually causing the observed behaviour. Be specific — reference files, functions, or spec sections.

**Affected layers:** The system layers and components involved (layer names from `02_project_architecture.md`).

**Reconciliation classification:** Code defect / Documentation drift / Unapproved implementation change / Ambiguous intent / Scope change — and why it fits, with evidence from both the spec and the implementation.

**Recommended corrective action:** What should happen next, and where (which document to update, which code to change, or both). Do not perform the fix here.

**Open questions:** Leave blank if none.
```

**Next step:** Run `/issue_03_resolve {project-name} {issue-id}`.

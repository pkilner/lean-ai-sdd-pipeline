---
name: issue_03_resolve
description: Resolve an investigated issue — the third user step in the issue workflow. Applies the recommended corrective action (spec update, code fix, or both) and records what changed in the Resolution section of the issue's file. Pass the project name and issue ID.
---

The arguments passed to this skill are the project name in kebab-case, followed by the issue ID.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Steps

1. Read `{project-root}/issues/{issue-id}.md` (required — do not proceed if missing or if its Investigation section is still `_Not started._`).

2. **Before touching the application repository:**
   - Re-confirm `repo_path` from the project pointer resolves to a real git repository (per the checks in `project_00_init` — do not assume it still holds).
   - Run `git -C {repo_path} status` and note the current state. If there is unrelated uncommitted work, tell the user and confirm how to proceed rather than editing over it.
   - Only touch files required to implement the recommended corrective action — do not opportunistically refactor or "clean up" nearby code.

3. Apply the recommended corrective action from the investigation:
   - If it requires a spec update, update the relevant document(s) under `{project-root}/features/{feature-name}/` (or `02_project_architecture.md` / `01_project_brief.md` for project-wide issues), in the order `01_feature_spec → 02_feature_plan`.
   - If it requires a code fix, implement it in the application repository.
   - If it requires both, do the spec update first, then the code fix, so the code is brought into line with an already-corrected spec.
   - **Never run `git commit` or `git push` in the application repository as part of this skill.** Leave changes staged/unstaged for the user to review and commit themselves — this includes the spec updates, which live in the same repository under `sdd/`.

4. Replace the `_Not started._` placeholder under `## 3. Resolution` with the section template below, listing every document and file actually changed.

5. Update the header `Status` to `Resolved` and add a Status History row.

6. Tell the user plainly what was changed (spec files and/or application files) and recommend the next step.

**Keep the status current:** whenever this skill creates its document or changes a Status, finish by running `project_90_status {project-name}` so `STATUS.md` matches before ending your turn.

---

## Section Template (`## 3. Resolution`)

```markdown
**Date:** {today's date}

**Summary:** One or two sentences describing what was done.

**Specification changes**

| Document | Change |
|---|---|
| (e.g.) features/{feature}/01_feature_spec.md | Corrected field name from X to Y |

(Write "None" if no spec changes were needed.)

**Code changes** (not committed or pushed — left for the user to review)

| File (in application repo) | Change |
|---|---|
| (e.g.) src/api/routes.ts | Fixed validation to match the spec |

(Write "None" if no code changes were needed.)

**Verification plan:** What must be checked to confirm this actually fixes the issue. Executed by `/issue_04_verify`, not here.
```

**Next step:** Run `/issue_04_verify {project-name} {issue-id}`.

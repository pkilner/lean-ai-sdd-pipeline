---
name: project_00_init
description: Initialize a new project under projects/{project-name}/ — validates the sibling application repository, then creates project.yaml and the base directory scaffold. First step in the project workflow, run once per project.
---

The arguments passed to this skill are the project name in kebab-case, followed by the path to the sibling application repository (absolute or relative — it will be normalized).

## Steps

1. Validate the project name is kebab-case. If `projects/{project-name}/` already exists, stop and tell the user the project already exists — do not overwrite it.

2. Validate `repo_path` before creating anything. Do not continue if any check fails — report exactly which check failed and stop.

   - **Exists:** the path resolves to something on disk.
   - **Is a directory:** not a file.
   - **Is a git repository:** running `git -C {repo_path} rev-parse --is-inside-work-tree` succeeds.
   - **Is not this pipeline repository:** resolve both this repo's root and `repo_path` to absolute paths and confirm they differ. A project must not point at itself.
   - **Resolves correctly:** the resolved absolute path is reachable from this repo's root (no dangling symlink, no permission error).

3. Compute the normalized path as a relative path from this pipeline repo's root to the validated application repository (e.g. `../my-app`). Store this normalized form, not whatever raw string the user typed.

4. Create the directory scaffold:

   ```text
   projects/{project-name}/
   ├── adr/
   ├── features/
   └── issues/
   ```

5. Create `projects/{project-name}/project.yaml` using the template below.

6. Confirm creation to the user and recommend the next step.

---

## Output Template

`projects/{project-name}/project.yaml`:

```yaml
name: {project-name}
repo_path: {normalized-relative-repo-path}
description: TBD — run /project_01_brief {project-name}
created: {today's date}
status: active
```

- `repo_path` is always stored as a normalized relative path from this pipeline repo's root, validated per Step 2 above. Every other skill that needs to inspect or reference application source code (e.g. `system_consistency_review`, `issue_02_investigate`) reads this field — never re-derive or guess it elsewhere.
- `status` is one of `active`, `paused`, `archived`.

**Next step:** Run `/project_01_brief {project-name}`.

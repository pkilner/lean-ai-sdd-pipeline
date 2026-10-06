---
name: project_00_init
description: Initialize a new project — validates the sibling application repository, creates the sdd/ scaffold and project.yaml inside it, and registers a local pointer under projects/ in this pipeline repo. First step in the project workflow, run once per project (or once per machine, to register a project whose sdd/ already exists).
---

The arguments passed to this skill are the project name in kebab-case, followed by the path to the sibling application repository (absolute or relative — it will be normalized).

All of a project's artifacts — `project.yaml`, briefs, architecture, ADRs, features, issues — live in the application repository under `sdd/`. Every other skill calls that directory `{project-root}`. This pipeline repo holds only the skills plus one local, gitignored pointer per project (`projects/{project-name}.yaml`) recording where that application repository is.

## Steps

1. Validate the project name is kebab-case. If `projects/{project-name}.yaml` already exists, stop and tell the user the project is already registered — do not overwrite it.

2. Validate `repo_path` before creating anything. Do not continue if any check fails — report exactly which check failed and stop.

   - **Exists:** the path resolves to something on disk.
   - **Is a directory:** not a file.
   - **Is a git repository:** running `git -C {repo_path} rev-parse --is-inside-work-tree` succeeds.
   - **Is not this pipeline repository:** resolve both this repo's root and `repo_path` to absolute paths and confirm they differ. A project must not point at itself.
   - **Resolves correctly:** the resolved absolute path is reachable from this repo's root (no dangling symlink, no permission error).

3. Compute the normalized path as a relative path from this pipeline repo's root to the validated application repository (e.g. `../my-app`). Store this normalized form, not whatever raw string the user typed.

4. If `{repo_path}/sdd/project.yaml` already exists, this is an existing project being registered on this machine. Confirm its `name` matches `{project-name}` (if not, report the mismatch and stop), then skip Step 5 — do not create or modify anything under `sdd/`.

5. Create the directory scaffold in the application repository, and `{repo_path}/sdd/project.yaml` using the template below:

   ```text
   {repo_path}/sdd/
   ├── project.yaml
   ├── adr/
   ├── features/
   └── issues/
   ```

6. Create the pointer `projects/{project-name}.yaml` in this pipeline repo using the template below.

7. Confirm to the user what was created — the pointer, the project root, and that any new files in the application repository are uncommitted — and recommend the next step.

---

## Output Templates

`projects/{project-name}.yaml` (pointer, in this pipeline repo — gitignored):

```yaml
repo_path: {normalized-relative-repo-path}
```

`{repo_path}/sdd/project.yaml` (in the application repository):

```yaml
name: {project-name}
description: TBD — run /project_01_brief {project-name}
created: {today's date}
status: active
```

- `repo_path` is stored only in the pointer, as a normalized relative path from this pipeline repo's root, validated per Step 2 above. Every other skill reads the pointer to find `{project-root}` (`{repo_path}/sdd`) and to inspect application source code — never re-derive or guess it elsewhere.
- `status` is one of `active`, `paused`, `archived`.

**Next step:** Run `/project_01_brief {project-name}`.

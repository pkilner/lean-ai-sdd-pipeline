---
name: project_init
description: Initialize a new project under projects/{project-name}/ — creates project.yaml and the base directory scaffold. First step in the project workflow, run once per project.
---

The arguments passed to this skill are the project name in kebab-case, followed by the relative path from this pipeline repo's root to the sibling application repository (e.g. `my-app ../my-app`).

## Steps

1. Validate the project name is kebab-case. If `projects/{project-name}/` already exists, stop and tell the user the project already exists — do not overwrite it.

2. Create the directory scaffold:

   ```text
   projects/{project-name}/
   ├── adr/
   ├── features/
   └── issues/
   ```

3. Create `projects/{project-name}/project.yaml` using the template below.

4. Confirm creation to the user and recommend the next step.

---

## Output Template

`projects/{project-name}/project.yaml`:

```yaml
name: {project-name}
repo_path: {repo-path}
description: TBD — run /project_brief {project-name}
created: {today's date}
status: active
```

- `repo_path` is the relative path from this pipeline repo's root to the sibling application repository. It is read by every skill that needs to inspect or reference application source code (e.g. `system_consistency_review`, `issue_investigate`).
- `status` is one of `active`, `paused`, `archived`.

**Next step:** Run `/project_brief {project-name}`.

---
name: feature_00_init
description: Initialize a new feature under projects/{project-name}/features/{feature-name}/ — creates the directory scaffold. First step in the feature workflow, run once per feature.
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case (e.g. `my-app user-onboarding`).

## Steps

1. Read `projects/{project-name}/project.yaml` (required — do not proceed if the project has not been initialized; tell the user to run `/project_00_init` first).

2. **Review gate:** check `projects/{project-name}/01_project_brief.md` and `02_project_architecture.md`. Both must exist with `Status: Approved`. If either is missing or still `Draft`, stop here, tell the user which project-level document needs review/approval first, and do not create the feature.

3. If `projects/{project-name}/features/{feature-name}/` already exists, stop and tell the user the feature already exists — do not overwrite it.

4. Create the directory scaffold:

   ```text
   projects/{project-name}/features/{feature-name}/
   └── adr/
   ```

5. Confirm creation to the user and recommend the next step.

**Next step:** Run `/feature_01_brief {project-name} {feature-name}`.

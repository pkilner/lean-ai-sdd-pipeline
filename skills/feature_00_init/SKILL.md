---
name: feature_00_init
description: Initialize a new feature under sdd/features/{feature-name}/ in the project's application repository — creates the directory scaffold. First step in the feature workflow, run once per feature.
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case (e.g. `my-app user-onboarding`).

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Project scope and delivery

Maintain one project brief and one project architecture for the whole intended product. Release order does not define project scope. Separate agreed capabilities, unresolved product candidates, and explicit product exclusions; do not promote a candidate to a commitment or classify an agreed later capability as excluded.

A feature is a bounded, testable delivery increment. It may span several capabilities and system layers, including a proof-of-concept phase. Keep a lightweight delivery sequence in the project brief with feature IDs, outcomes, dependencies, and links to specifications once they exist. Planning rows are not evidence that feature documents exist or that work is approved or implemented.

## Steps

1. Read `{project-root}/project.yaml` (required — do not proceed if the project has not been initialized; tell the user to run `/project_00_init` first).

2. **Review gate:** check `{project-root}/01_project_brief.md` and `02_project_architecture.md`. Both must exist with `Status: Approved`. If either is missing or still `Draft`, stop here, tell the user which project-level document needs review/approval first, and do not create the feature.

3. If `{project-root}/features/{feature-name}/` already exists, stop and tell the user the feature already exists — do not overwrite it.

4. Create the directory scaffold:

   ```text
   {project-root}/features/{feature-name}/
   └── adr/
   ```

5. Confirm creation to the user and recommend the next step.

**Next step:** Run `/feature_01_brief {project-name} {feature-name}`.

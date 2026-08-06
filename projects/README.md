# projects/

This directory contains the specification and workflow artifacts for the application projects managed by this pipeline — one subdirectory per project, named after the project.

## Structure

```text
projects/
└── <project-name>/
    ├── project.yaml
    ├── 01_project_brief.md
    ├── 02_project_architecture.md
    ├── adr/
    ├── features/
    │   └── <feature-name>/
    │       ├── 01_feature_brief.md
    │       ├── 02_technical_design.md
    │       ├── 03_api_spec.md
    │       ├── 04_test_spec.md
    │       ├── 05_implementation_plan.md
    │       └── adr/
    └── issues/
        └── ISSUE-xxxx/
            ├── 01_issue.md
            ├── 02_investigation.md
            ├── 03_resolution.md
            └── 04_verification.md
```

Each `<project-name>/` corresponds to one sibling application repository, referenced by `repo_path` in that project's `project.yaml`. See the root `README.md` for the full workflow (project → feature → issue) and how these documents relate to each other.

## Creating a project

Developers shall not manually create project structures unless necessary. The `project_00_init` skill creates the required project scaffold — it validates the target application repository and writes `project.yaml`, so a project directory should only ever come into existence through that skill.

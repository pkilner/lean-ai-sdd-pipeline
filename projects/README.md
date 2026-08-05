# projects/

This directory contains the specification and workflow artifacts for the application projects managed by this pipeline — one subdirectory per project, named after the project.

## Structure

```text
projects/
└── <project-name>/
    ├── project.yaml
    ├── project-brief.md
    ├── architecture.md
    ├── adr/
    ├── features/
    │   └── <feature-name>/
    │       ├── feature-brief.md
    │       ├── technical-design.md
    │       ├── api-spec.md
    │       ├── test-spec.md
    │       ├── implementation-plan.md
    │       └── adr/
    └── issues/
        └── ISSUE-xxxx/
            ├── issue.md
            ├── investigation.md
            ├── resolution.md
            └── verification.md
```

Each `<project-name>/` corresponds to one sibling application repository, referenced by `repo_path` in that project's `project.yaml`. See the root `README.md` for the full workflow (project → feature → issue) and how these documents relate to each other.

## Creating a project

Developers shall not manually create project structures unless necessary. The `project_init` skill creates the required project scaffold — it validates the target application repository and writes `project.yaml`, so a project directory should only ever come into existence through that skill.

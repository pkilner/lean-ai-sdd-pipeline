# projects/

This directory holds one local pointer per application project registered on this machine. It contains no specifications — every artifact the pipeline generates lives in the project's own application repository, under `sdd/`.

## Structure

```text
projects/
└── <project-name>.yaml    ← pointer: repo_path only, gitignored
```

```yaml
repo_path: ../my-app
```

`repo_path` is the relative path from this pipeline repo's root to the sibling application repository. The project's artifacts are at `{repo_path}/sdd/`:

```text
<application-repository>/
└── sdd/
    ├── project.yaml
    ├── STATUS.md
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

Pointers are gitignored: they hold machine-specific paths, and keeping them out of history means nothing about a project is committed to this repo. See the root `README.md` for the full workflow (project → feature → issue) and how these documents relate to each other.

## Creating or registering a project

Developers shall not manually create project structures unless necessary. The `project_00_init` skill validates the target application repository, creates the `sdd/` scaffold and `project.yaml` inside it, and writes the pointer here. On a fresh clone of this repo, running `project_00_init` against an application repository that already has `sdd/project.yaml` registers the pointer without touching the existing artifacts.

# lean-ai-sdd-pipeline

A lean, practical **Spec-Driven Development pipeline** for Claude Code — Claude skills that take a project from initialization through feature specification to a fully-specified implementation plan, plus an issue-tracking workflow for defects, drift, and change requests.

It is intentionally **not** an enterprise ALM platform. The guiding principle:

> Provide the minimum amount of structure necessary to consistently produce high-quality software.

## Workspace layout

This pipeline repository and the application(s) it specs live as sibling directories:

```text
workspace/
├── lean-ai-sdd-pipeline/     ← this repo: specs, architecture, ADRs, issues, skills
└── <application-repository>/ ← source code, tests, deployment, configuration
```

The pipeline repository owns specifications, architecture, ADRs, workflow, and issue tracking. The application repository owns only source code, tests, deployment, and configuration. **No Claude skills are copied into the application repository** — run Claude Code from this pipeline repo (or a workspace root that contains both as siblings), and skills reach into the application repo via each project's `repo_path`.

## Repository layout

```text
lean-ai-sdd-pipeline/
├── skills/
│   ├── project_*
│   ├── feature_*
│   ├── issue_*
│   └── system_*
│
└── projects/
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

A project can wrap one application repository. The `projects/` directory supports more than one project if this pipeline repo is used to spec multiple applications.

## Skill namespaces

Every skill lives in exactly one of four namespaces.

### `project_*` — project lifecycle (user-facing)

| Skill | Step | Produces | Purpose |
|---|---|---|---|
| `project_init` | 1 | `project.yaml` + scaffold | Register a project and point it at its application repo |
| `project_brief` | 2 | `project-brief.md` | What the project is, who it's for, what success looks like |
| `project_architecture` | 3 | `architecture.md` | System layers, key technical decisions, data stores |
| `project_adr` | as needed | `adr/adr_NNN_*.md` | Record a project-wide architectural decision |

### `feature_*` — feature lifecycle (user-facing)

Run in order for each feature. Each step ends with a human review checkpoint — nothing proceeds automatically.

| Skill | Step | Produces | Purpose |
|---|---|---|---|
| `feature_init` | 1 | scaffold | Create the feature's directory |
| `feature_brief` | 2 | `feature-brief.md` | What to build — user stories, acceptance criteria, scope |
| `feature_technical_design` | 3 | `technical-design.md` | How to build it — design per system layer, data flow |
| `feature_api_spec` | 4 (if needed) | `api-spec.md` | Exact contracts — data models, DB schema, endpoints, events |
| `feature_test_spec` | 5 | `test-spec.md` | Unit, integration, and end-to-end tests — **defines "done"** |
| `feature_implementation_plan` | 6 | `implementation-plan.md` | Coding tasks in dependency order, each mapped to the tests it satisfies |
| `feature_adr` | as needed | `adr/adr_NNN_*.md` | Record a feature-level architectural decision |
| — | 7 | (code) | Implementation, in the application repository |

The test spec comes **before** the implementation plan. Tests define what "done" means; the implementation plan exists to satisfy them, not the other way around. Every implementation task references the test IDs it satisfies.

### `issue_*` — issue lifecycle (user-facing)

Issues are **siblings of features**, not nested beneath them — an issue can span multiple features.

| Skill | Step | Produces | Purpose |
|---|---|---|---|
| `issue_capture` | 1 | `issue.md` | Record what was observed |
| `system_issue_classify` | 2 | updates `issue.md` | Assign a category (see below) |
| `issue_investigate` | 3 | `investigation.md` | Root cause + reconciliation classification |
| `issue_resolve` | 4 | `resolution.md` | Apply the fix (spec, code, or both) |
| `issue_verify` | 5 | `verification.md` | Confirm the fix actually works, close the issue |
| `system_consistency_review` | 6 | report only | Confirm no residual drift was introduced |

Issue categories: Defect, Documentation drift, Design defect, Implementation drift, Test gap, Change request, Dependency/environment issue.

### `system_*` — governance and orchestration (not user-facing)

Invoked by Claude or by other skills, though nothing stops you from running one directly.

| Skill | Purpose |
|---|---|
| `system_issue_classify` | Assign an issue's category |
| `system_consistency_review` | Read-only cross-check of a feature's (or a whole project's) docs against the implementation |
| `system_next_step` | Inspect existing artifacts and report the next command to run |
| `system_workflow_resume` | Full briefing on a project's state — what exists, what's in progress, what's next |

## Reconciliation

When code and documentation disagree, never assume one is correct by default. Classify the mismatch first:

- Code defect
- Documentation drift
- Unapproved implementation change
- Ambiguous intent
- Scope change

Then recommend the corrective action. Documents are not automatically rewritten to match code — `issue_investigate` performs this classification as part of the issue workflow.

## Getting started

1. Point Claude Code at a workspace root that contains this repo and your application repo as siblings (or run it from inside this repo, if the application repo's absolute/relative path is reachable).
2. `/project_init <project-name> <repo-path-to-application-repo>`
3. `/project_brief <project-name>` → `/project_architecture <project-name>`
4. `/feature_init <project-name> <feature-name>` → walk the `feature_*` pipeline in order.
5. When something goes wrong: `/issue_capture <project-name> "<description>"` → walk the `issue_*` pipeline in order.
6. Anytime: `/system_workflow_resume <project-name>` to get re-oriented, or `/system_next_step <project-name> [feature-name|issue-id]` for a single next command.

## Design notes / why it's structured this way

- **One document, one review gate, per step.** Each `feature_*` and `project_*` skill refuses to proceed if its required upstream document is missing — this keeps work from drifting ahead of what's actually been agreed.
- **Change propagation is top-down.** `feature-brief → technical-design → api-spec → test-spec → implementation-plan → implementation`. If a change originates in the implementation (e.g. a bug discovery), update upward first, then confirm downward. `system_consistency_review` exists specifically to catch when this discipline slips.
- **Tests come before the implementation plan.** This flips the historical order in this pipeline on purpose: tests are the definition of "done", not an afterthought validated against whatever got built.
- **Layer names are not hardcoded.** `project_architecture` derives the project's real system layers (Frontend/Backend/Database, On-Device/Cloud, or whatever fits) once, and every other skill pulls from it — no skill assumes a specific stack.
- **Issues are first-class, not an afterthought bolted onto features.** They live as siblings of features because a single issue can span several of them, and because "something is wrong" is a fundamentally different workflow from "build something new."
- **No skills in the application repository.** Keeping all pipeline machinery in one repo means an application repo stays clean of tooling that isn't actually part of the shipped product, and one pipeline repo can spec more than one application if needed.

## License

See `LICENSE`.

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

The pipeline repository owns specifications, architecture, ADRs, workflow, and issue tracking. The application repository owns only source code, tests, deployment, and configuration. **No Claude skills are copied into the application repository** — run Claude Code from this pipeline repo (or a workspace root that contains both as siblings), and skills reach into the application repo via each project's validated `repo_path`.

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

## Review gates: Draft → Approved

Every generated document (`project-brief.md`, `architecture.md`, `feature-brief.md`, `technical-design.md`, `api-spec.md`, `test-spec.md`, `implementation-plan.md`) starts `Status: Draft`. The skill that owns the *next* step in the workflow checks this field before doing anything — if the prerequisite is still `Draft`, it stops, names the document that needs review, and does not proceed. It never approves a document on your behalf; approval only happens when you confirm the review checklist, at which point the status flips to `Approved` in that same document.

This is a single field, not an approval engine — but it means the pipeline cannot silently race ahead of what's actually been reviewed.

ADRs use their own vocabulary instead: `Proposed` when created, `Accepted` only once a human has actually reviewed and confirmed the decision.

## Skill namespaces

### `project_*` and `feature_*` and `issue_*` — user-facing

These are the skills you run directly.

**Project lifecycle** — run once per project:

| Skill | Step | Produces |
|---|---|---|
| `project_init` | 1 | `project.yaml` (validates `repo_path` — must exist, be a git repo, and not be this pipeline repo itself) |
| `project_brief` | 2 | `project-brief.md` |
| `project_architecture` | 3 | `architecture.md` (per-layer repository path, responsibility, status) |
| `project_adr` | as needed | `adr/adr_NNN_*.md` |

**Feature lifecycle** — run in order for each feature, each step gated on the previous being `Approved`:

| Skill | Step | Produces |
|---|---|---|
| `feature_init` | 1 | scaffold |
| `feature_brief` | 2 | `feature-brief.md` (stable `AC-N` acceptance criteria) |
| `feature_technical_design` | 3 | `technical-design.md` (declares whether an API spec is required) |
| `feature_api_spec` | 4, only if required | `api-spec.md` |
| `feature_test_spec` | 5 | `test-spec.md` — every test maps to the `AC-N` it validates; Unit/Integration/E2E each marked Required/Optional/Not Applicable |
| `feature_implementation_plan` | 6 | `implementation-plan.md` — every task tracks real Status (Not Started/In Progress/Blocked/Complete) and the test IDs it satisfies |
| `feature_adr` | as needed | `adr/adr_NNN_*.md` |
| — | 7 | implementation, in the application repository |

The test spec comes **before** the implementation plan — tests define "done", the plan exists to satisfy them. A feature is only **Implemented** once every task's Status is `Complete`; it's only **Verified** once a consistency review has actually passed. Neither is true just because `implementation-plan.md` exists.

**Issue lifecycle** — issues are siblings of features, not nested beneath them, since one issue can span several:

| Skill | Step | Produces |
|---|---|---|
| `issue_capture` | 1 | `issue.md` — automatically classified (category) as part of this step |
| `issue_investigate` | 2 | `investigation.md` — root cause + Reconciliation classification |
| `issue_resolve` | 3 | `resolution.md` — the fix (spec update, code fix, or both — never auto-committed or pushed) |
| `issue_verify` | 4 | `verification.md` — confirmation the fix works, closes the issue — automatically runs a consistency review as part of closing |

Every issue always produces all four files, regardless of size — `issue.md`, `investigation.md`, `resolution.md`, `verification.md`. Classification assigns a category only; it does not change which files get created.

### `system_*` — internal orchestration, not user-facing

These are invoked automatically by the skills above, or by Claude directly when it needs to check state — never by you typing a command. You will see their output (a classification, a consistency report, a briefing), but you don't invoke them:

- `system_issue_classify` — runs inside `issue_capture`
- `system_consistency_review` — runs inside `feature_implementation_plan` (once all tasks are Complete) and `issue_verify` (once an issue closes), or whenever you ask Claude to check whether something is actually consistent
- `system_next_step` / `system_workflow_resume` — Claude runs these itself at the start of a session, or whenever you ask "where were we" / "what's next" in plain language

If you want a consistency check or a status briefing, just ask — Claude runs the relevant system skill for you.

## Reconciliation

When code and documentation disagree, never assume one is correct by default. `issue_investigate` classifies the mismatch first:

- Code defect
- Documentation drift
- Unapproved implementation change
- Ambiguous intent
- Scope change

Then recommends the corrective action. Documents are not automatically rewritten to match code.

## Operational safety

Any skill that touches the application repository (`issue_resolve`, and implementation itself):

- Re-validates `repo_path` before editing anything
- Checks `git status` first and does not edit over unrelated uncommitted work without confirming with you
- Touches only the files required for the task at hand — no opportunistic refactors
- Never runs `git commit` or `git push` on your behalf
- Reports exactly which files it changed

## Getting started

1. Point Claude Code at a workspace root that contains this repo and your application repo as siblings (or run it from inside this repo, if the application repo's path is reachable).
2. `/project_init my-app ../my-app` → validates the repo, creates `projects/my-app/project.yaml`.
3. `/project_brief my-app` → review the draft → confirm → `/project_architecture my-app` → review → confirm.
4. `/feature_init my-app checkout-flow` → `/feature_brief my-app checkout-flow` (writes `AC-1`, `AC-2`, ...) → review → confirm.
5. `/feature_technical_design my-app checkout-flow` (declares `API Specification Required: Yes`) → review → confirm.
6. `/feature_api_spec my-app checkout-flow` → review → confirm.
7. `/feature_test_spec my-app checkout-flow` (writes `UT-1 … Covers AC-1`, etc.) → review → confirm.
8. `/feature_implementation_plan my-app checkout-flow` (writes tasks, each with a Status and Tests Satisfied) → review → confirm → start implementing.
9. As tasks are implemented, their Status moves to `Complete`. Once all are `Complete`, a consistency review runs automatically and, if it passes, the feature is **Verified**.
10. If something's wrong later: `/issue_capture my-app "checkout total is off by tax on international orders"` — classification happens automatically; then walk `issue_investigate` → `issue_resolve` → `issue_verify`.

## Design notes / why it's structured this way

- **One document, one review gate, per step.** No skill can silently build on top of something nobody has actually reviewed.
- **Change propagation is top-down.** `feature-brief → technical-design → api-spec → test-spec → implementation-plan → implementation`. If a change originates in the implementation (e.g. a bug discovery), update upward first, then confirm downward.
- **Tests come before the implementation plan**, and every test traces to an acceptance criterion. "Done" is defined before a single task is written, not inferred afterward.
- **Status is tracked, not assumed.** A plan existing doesn't mean anything is built; tasks being marked Complete doesn't mean anything is verified. These are different, explicitly tracked states.
- **Layer names and paths are not hardcoded.** `project_architecture` derives the project's real system layers and where their code actually lives, once, and every other skill pulls from it.
- **Orchestration is invisible where it can be.** Classification and consistency checks happen as part of the workflow, not as extra commands you have to remember to run.
- **No skills in the application repository.** Keeping all pipeline machinery in one repo means an application repo stays clean of tooling that isn't actually part of the shipped product.

## License

See `LICENSE`.

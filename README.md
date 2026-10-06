# lean-ai-sdd-pipeline

A lean, practical **Spec-Driven Development pipeline** for Claude Code — Claude skills that take a project from initialization through feature specification to a reviewed plan of tests and tasks, plus an issue-tracking workflow for defects, drift, and change requests.

It is intentionally **not** an enterprise ALM platform. The guiding principle:

> Provide the minimum amount of structure necessary to consistently produce high-quality software.

**How it works, in one paragraph:** Claude runs the skills below to take a project from initialization through feature specification (a spec, then a plan of tests and tasks) to implementation, and to track issues (defects, drift, change requests) through investigation, resolution, and verification. Claude does the drafting — every document, every classification, every consistency check. You do the reviewing — nothing proceeds past a review gate until you confirm a document, and code is never committed or pushed without you. For the full walkthrough, see [`docs/pipeline-overview.md`](docs/pipeline-overview.md).

```mermaid
flowchart TD
    classDef userSkill fill:#cfe2ff,stroke:#2b6cb0,color:#1a365d,stroke-width:1px
    classDef systemSkill fill:#ffe8cc,stroke:#c05621,color:#652b06,stroke-width:1px,stroke-dasharray: 3 2
    classDef doc fill:#e6f4ea,stroke:#2f855a,color:#22543d,stroke-width:1px
    classDef gate fill:#fff0f0,stroke:#c53030,color:#742a2a,stroke-width:1px
    classDef impl fill:#edf2f7,stroke:#4a5568,color:#1a202c,stroke-width:1px
    classDef structure fill:#f5f5f5,stroke:#718096,color:#2d3748,stroke-width:1px

    subgraph LEGEND[Legend]
        direction LR
        L1[User-facing skill]:::userSkill
        L2[Internal system skill]:::systemSkill
        L3[Generated document]:::doc
        L4{Review gate}:::gate
        L5[Implementation]:::impl
    end

    WS[Workspace]:::structure --> PR[Pipeline Repository]:::structure
    WS --> AR[Application Repository]:::impl
    PR --> PROJS[projects/ - local pointers]:::structure
    PROJS -->|validated repo_path| AR
    AR --> SDD[sdd/ - all project artifacts]:::structure

    subgraph PROJ[Project Workflow]
        direction TB
        PI[project_00_init]:::userSkill --> PY[project.yaml]:::doc
        PY --> PB[project_01_brief]:::userSkill --> PBD[01_project_brief.md - Draft]:::doc
        PBD --> G1{Approved?}:::gate
        G1 -->|no, revise| PB
        G1 -->|yes| PA[project_02_architecture]:::userSkill
        PA --> AD[02_project_architecture.md - Draft]:::doc --> G2{Approved?}:::gate
        G2 -->|no, revise| PA
        G2 -->|yes| ADR1[project_adr - optional]:::userSkill
        ADR1 --> ADOC1[ADR - Proposed]:::doc
    end
    SDD --> PI
    G2 -->|yes| FI

    subgraph FEAT[Feature Workflow]
        direction TB
        FI[feature_00_init]:::userSkill --> FS[feature_01_spec]:::userSkill --> FSD[01_feature_spec.md - Draft, AC-N and design]:::doc
        FSD --> G3{Approved?}:::gate
        G3 -->|no, revise| FS
        G3 -->|yes| FP[feature_02_plan]:::userSkill
        FP --> FPD[02_feature_plan.md - Draft, tests and tasks]:::doc --> G4{Approved?}:::gate
        G4 -->|no, revise| FP
        G4 -->|yes| IMPL[Implementation - application repo]:::impl
        ADR2[feature_adr - optional]:::userSkill --> ADOC2[ADR - Proposed]:::doc
    end

    IMPL -->|all tasks Complete| CR[system_consistency_review]:::systemSkill
    CR -->|PASS| VER[Feature Verified]:::doc

    subgraph ISSUE[Issue Workflow]
        direction TB
        IC[issue_01_capture]:::userSkill --> ISD[ISSUE-xxxx.md - one file per issue]:::doc
        ISD --> SIC[system_issue_classify]:::systemSkill
        SIC -->|Category set, or Pending Investigation| ISD
        SIC --> II[issue_02_investigate]:::userSkill --> IR[issue_03_resolve]:::userSkill --> IV[issue_04_verify]:::userSkill
        IV --> VF[Issue Closed]:::doc
    end

    VF --> CR

    %% Feedback loop: a consistency review finding can start a new issue
    CR -.->|drift found| IC

    %% Feedback loop: an accepted architecture decision propagates down to implementation
    ADR1 -.->|Accepted| AD
    AD -.->|layers or decisions updated| FS
    FS -.->|design updated| IMPL
```

## Workspace layout

This pipeline repository and the application(s) it specs live as sibling directories:

```text
workspace/
├── lean-ai-sdd-pipeline/     ← this repo: skills, plus a local pointer per project
└── <application-repository>/ ← source code, tests, deployment, configuration
    └── sdd/                  ← that project's specs, architecture, ADRs, issues
```

The pipeline repository owns the skills and the workflow. Each application repository owns its source code, tests, deployment, and configuration — and, under `sdd/`, every artifact this pipeline generates for it: `project.yaml`, briefs, architecture, ADRs, feature specs, and issues. Specs are versioned and access-controlled with the code they describe, so a private application's specs stay private even when this pipeline repo is shared. **No Claude skills are copied into the application repository** — run Claude Code from this pipeline repo (or a workspace root that contains both as siblings), and skills reach into the application repo via each project's validated `repo_path`.

## Repository layout

```text
lean-ai-sdd-pipeline/
├── .claude/skills → ../skills    ← how Claude Code discovers the skills
├── skills/
│   ├── project_*
│   ├── feature_*
│   ├── issue_*
│   └── system_*
│
└── projects/
    └── <project-name>.yaml       ← local pointer (repo_path only), gitignored

<application-repository>/
└── sdd/
    ├── project.yaml
    ├── STATUS.md                 ← snapshot written by project_90_status
    ├── 01_project_brief.md
    ├── 02_project_architecture.md
    ├── adr/
    ├── features/
    │   └── <feature-name>/
    │       ├── 01_feature_spec.md
    │       ├── 02_feature_plan.md
    │       └── adr/
    └── issues/
        └── ISSUE-xxxx.md
```

A project wraps one application repository. `projects/` holds one pointer per project registered on this machine, so this pipeline repo can drive more than one application. Pointers are gitignored — they hold machine-specific paths, and nothing about a project is committed here.

## Review gates: Draft → Approved

Every generated document (`01_project_brief.md`, `02_project_architecture.md`, `01_feature_spec.md`, `02_feature_plan.md`) starts `Status: Draft`. The skill that owns the *next* step in the workflow checks this field before doing anything — if the prerequisite is still `Draft`, it stops, names the document that needs review, and does not proceed. It never approves a document on your behalf; approval only happens when you confirm the review checklist, at which point the status flips to `Approved` in that same document and the approval date is recorded beneath it as `Approved: {date}`.

This is a single field, not an approval engine — but it means the pipeline cannot silently race ahead of what's actually been reviewed.

ADRs use their own vocabulary instead: `Proposed` when created, `Accepted` only once a human has actually reviewed and confirmed the decision.

## Skill namespaces

### `project_*` and `feature_*` and `issue_*` — user-facing

These are the skills you run directly.

**Project lifecycle** — run once per project:

| Skill | Step | Produces |
|---|---|---|
| `project_00_init` | 1 | `sdd/project.yaml` and scaffold in the application repo, plus the local pointer (validates `repo_path` — must exist, be a git repo, and not be this pipeline repo itself) |
| `project_01_brief` | 2 | `01_project_brief.md` |
| `project_02_architecture` | 3 | `02_project_architecture.md` (per-layer repository path, responsibility, status) |
| `project_adr` | as needed | `adr/adr_NNN_*.md` |
| `project_90_status` | anytime | health, current state, document/feature/issue/ADR summary, recommended next step — displayed, and saved as a snapshot to `sdd/STATUS.md` |

**Feature lifecycle** — run in order for each feature, each step gated on the previous being `Approved`:

| Skill | Step | Produces |
|---|---|---|
| `feature_00_init` | 1 | scaffold |
| `feature_01_spec` | 2 | `01_feature_spec.md` — what and how: stable `AC-N` acceptance criteria, design by layer, and contracts where the feature changes them |
| `feature_02_plan` | 3 | `02_feature_plan.md` — tests, each mapped to the `AC-N` it validates, then tasks with real Status (Not Started/In Progress/Blocked/Complete) mapped to the tests they satisfy |
| `feature_adr` | as needed | `adr/adr_NNN_*.md` |
| `feature_90_status` | anytime | display only — health, current state, document/task/test summary, recommended next step |
| — | 4 | implementation, in the application repository |

Within the plan, tests are written **before** tasks — tests define "done", the tasks exist to satisfy them. A feature is only **Implemented** once every task's Status is `Complete`; it's only **Verified** once a consistency review has actually passed. Neither is true just because `02_feature_plan.md` exists.

**Issue lifecycle** — issues are siblings of features, not nested beneath them, since one issue can span several:

| Skill | Step | Produces |
|---|---|---|
| `issue_01_capture` | 1 | `issues/ISSUE-xxxx.md` — automatically classified (category) as part of this step |
| `issue_02_investigate` | 2 | the file's Investigation section — root cause + Reconciliation classification |
| `issue_03_resolve` | 3 | the file's Resolution section — the fix (spec update, code fix, or both — never auto-committed or pushed) |
| `issue_04_verify` | 4 | the file's Verification section — confirmation the fix works, closes the issue — automatically runs a consistency review as part of closing |
| `issue_90_status` | anytime | display only — pass an issue ID for that issue's stage/status, or omit it for a project-wide Open/Investigating/Resolved/Closed count |

Every issue is one file, `ISSUE-xxxx.md`, with four sections filled in as it moves through the workflow. Its header records Status, Category, and the features, acceptance criteria, and ADRs it relates to — which is how an issue is linked to a feature. Classification assigns a category only.

### `system_*` — internal orchestration, not user-facing

These are invoked automatically by the skills above, or by Claude directly when it needs to check state — never by you typing a command. You will see their output (a classification, a consistency report, a briefing), but you don't invoke them:

- `system_issue_classify` — runs inside `issue_01_capture`
- `system_consistency_review` — runs inside `feature_02_plan` (once all tasks are Complete) and `issue_04_verify` (once an issue closes), or whenever you ask Claude to check whether something is actually consistent
- `system_next_step` / `system_workflow_resume` — Claude runs these itself at the start of a session, or whenever you ask "where were we" / "what's next" in plain language

If you want a consistency check or a status briefing, just ask — Claude runs the relevant system skill for you.

## Reconciliation

When code and documentation disagree, never assume one is correct by default. `issue_02_investigate` classifies the mismatch first:

- Code defect
- Documentation drift
- Unapproved implementation change
- Ambiguous intent
- Scope change

Then recommends the corrective action. Documents are not automatically rewritten to match code.

## Operational safety

Any skill that touches application code — files in the application repository outside `sdd/` (`issue_03_resolve`, and implementation itself):

- Re-validates `repo_path` before editing anything
- Checks `git status` first and does not edit over unrelated uncommitted work without confirming with you
- Touches only the files required for the task at hand — no opportunistic refactors
- Never runs `git commit` or `git push` on your behalf — that includes the `sdd/` artifacts, which are left uncommitted for you to review
- Reports exactly which files it changed

## Getting started

1. Point Claude Code at a workspace root that contains this repo and your application repo as siblings (or run it from inside this repo, if the application repo's path is reachable).
2. `/project_00_init my-app ../my-app` → validates the repo, creates `../my-app/sdd/project.yaml`, and registers the pointer `projects/my-app.yaml`.
3. `/project_01_brief my-app` → review the draft → confirm → `/project_02_architecture my-app` → review → confirm.
4. `/feature_00_init my-app checkout-flow` → `/feature_01_spec my-app checkout-flow` (writes `AC-1`, `AC-2`, ..., the design, and any contracts) → review → confirm.
5. `/feature_02_plan my-app checkout-flow` (writes tests such as `UT-1 … Covers AC-1`, then tasks, each with a Status and Tests Satisfied) → review → confirm → start implementing.
6. As tasks are implemented, their Status moves to `Complete`. Once all are `Complete`, a consistency review runs automatically and, if it passes, the feature is **Verified**.
7. If something's wrong later: `/issue_01_capture my-app "checkout total is off by tax on international orders"` — classification happens automatically; then walk `issue_02_investigate` → `issue_03_resolve` → `issue_04_verify`.

## Design notes / why it's structured this way

- **One document, one review gate, per step.** No skill can silently build on top of something nobody has actually reviewed.
- **The spec is the source of truth, not the code.** When code and documentation disagree, the fix doesn't default to updating docs to match whatever the implementation did. Determine the intended behaviour, update the highest authoritative artifact first, propagate downstream (`01_feature_spec → 02_feature_plan`), and update the implementation last. If the approved documents are still correct, only the code changes.
- **Tests come before tasks**, and every test traces to an acceptance criterion. "Done" is defined before a single task is written, not inferred afterward.
- **Status is tracked, not assumed.** A plan existing doesn't mean anything is built; tasks being marked Complete doesn't mean anything is verified. These are different, explicitly tracked states.
- **Layer names and paths are not hardcoded.** `project_02_architecture` derives the project's real system layers and where their code actually lives, once, and every other skill pulls from it.
- **Orchestration is invisible where it can be.** Classification and consistency checks happen as part of the workflow, not as extra commands you have to remember to run.
- **Specs live with the code they describe.** Every generated artifact is written to `sdd/` in the application repository, so specs share its history, review, and access control — and this pipeline repo can be shared without exposing any project.
- **No skills in the application repository.** Keeping all pipeline machinery in one repo means an application repo stays clean of tooling that isn't actually part of the shipped product.

## License

See `LICENSE`.

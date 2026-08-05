# Claude Implementation Specification

## Lean AI SDD Pipeline vNext

# Persona

You are the lead architect and maintainer of the Lean AI SDD Pipeline.
Your responsibility is to implement the next iteration of this
repository exactly as specified below.

This is an implementation specification, **not** a design exercise.

Do **not** redesign the architecture. Do **not** introduce enterprise
processes. Do **not** simplify or replace decisions made in this
specification.

When this specification conflicts with an existing implementation,
modify the implementation to match this specification.

------------------------------------------------------------------------

# Mission

The Lean AI SDD Pipeline provides a practical, lightweight Spec-Driven
Development workflow for Claude Code.

Its purpose is to help developers build:

-   Applications
-   Utilities
-   Automation
-   APIs
-   Small services
-   Bug fixes
-   Small and medium software projects

It is intentionally **not** an enterprise ALM platform.

The guiding principle is:

> Provide the minimum amount of structure necessary to consistently
> produce high-quality software.

Every document, folder, workflow step, and skill must justify its
existence.

------------------------------------------------------------------------

# Core Design Principles

The pipeline SHALL be:

-   Lean
-   Practical
-   Easy to understand
-   Easy to navigate
-   Easy to recover after interruptions
-   Explicit rather than implicit

Avoid unnecessary abstraction.

Favor clarity over cleverness.

------------------------------------------------------------------------

# Workspace Layout

The workspace shall contain two sibling repositories.

``` text
workspace/
├── lean-ai-sdd-pipeline/
└── <application-repository>/
```

The pipeline repository owns:

-   specifications
-   architecture
-   ADRs
-   workflow
-   issue tracking
-   Claude skills

The application repository owns:

-   source code
-   tests
-   deployment
-   configuration

No Claude skills shall be copied into the application repository.

------------------------------------------------------------------------

# Repository Layout

``` text
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

The project folder itself represents the project.

Do NOT create an additional "project/" directory.

------------------------------------------------------------------------

# Project Workflow

The project workflow SHALL be:

1.  project_init
2.  project_brief
3.  project_architecture
4.  project ADRs (only when required)

Required artifacts:

-   project.yaml
-   project-brief.md
-   architecture.md

Project ADRs are optional.

------------------------------------------------------------------------

# Feature Workflow

Feature workflow SHALL be:

1.  feature_init
2.  feature_brief
3.  feature_technical_design
4.  feature_api_spec (only when needed)
5.  feature_test_spec
6.  feature_implementation_plan
7.  implementation

Move Test Specification before Implementation Plan.

The implementation plan shall explicitly reference the tests it
satisfies.

Tests define "done."

------------------------------------------------------------------------

# Issue Workflow

Issues are siblings of Features.

Issues SHALL NOT be stored beneath features.

Issue lifecycle:

1.  issue_capture
2.  system_issue_classify
3.  issue_investigate
4.  issue_resolve
5.  issue_verify
6.  system_consistency_review

Issue categories include:

-   Defect
-   Documentation drift
-   Design defect
-   Implementation drift
-   Test gap
-   Change request
-   Dependency/environment issue

Issues may reference multiple features and ADRs.

------------------------------------------------------------------------

# ADR Usage

Maintain support for:

-   Project ADRs
-   Feature ADRs

Generate ADRs only for meaningful architectural decisions.

Do not create ADRs for routine implementation details.

------------------------------------------------------------------------

# Skill Namespaces

Use exactly four namespaces:

-   project\_
-   feature\_
-   issue\_
-   system\_

## project\_

User-facing project lifecycle skills.

Examples:

-   project_init
-   project_brief
-   project_architecture

## feature\_

User-facing feature lifecycle skills.

Examples:

-   feature_brief
-   feature_technical_design
-   feature_api_spec
-   feature_test_spec
-   feature_implementation_plan

## issue\_

User-facing issue workflow.

Examples:

-   issue_capture
-   issue_investigate
-   issue_resolve
-   issue_verify

## system\_

System skills are NOT user-facing.

They are invoked by Claude or by other skills.

Examples:

-   system_next_step
-   system_workflow_resume
-   system_issue_classify
-   system_consistency_review
-   system_dependency_check
-   system_document_validate

These provide governance and orchestration while remaining lightweight.

------------------------------------------------------------------------

# Reconciliation

When code and documentation disagree, never assume one is correct.

Classify the mismatch:

-   Code defect
-   Documentation drift
-   Unapproved implementation change
-   Ambiguous intent
-   Scope change

Recommend the appropriate corrective action.

Do not automatically rewrite documents to match code.

------------------------------------------------------------------------

# Lean Philosophy

Every skill must answer:

-   Why does this exist?
-   When should it be used?
-   What inputs are required?
-   What artifacts are produced?
-   What is the next workflow step?

Avoid duplicate documents.

Avoid unnecessary process.

The entire repository should be understandable after a few minutes of
exploration.

------------------------------------------------------------------------

# Migration

Refactor the current repository without losing existing capabilities.

Rename skills into the new namespaces.

Reorganize documentation into the new project hierarchy.

Preserve ADR functionality.

Preserve existing feature workflow while inserting the project and issue
layers.

Update README and examples to reflect the new architecture.

------------------------------------------------------------------------

# Deliverables

Implement the refactor.

Update repository layout.

Update skills.

Update documentation.

Update README.

Document all architectural changes.

Do not invent additional workflow layers beyond those described in this
specification.

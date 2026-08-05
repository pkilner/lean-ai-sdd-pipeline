# Migration: vNext Refactor

This document records the architectural changes made in implementing `Claude_Implementation_Spec_Lean_AI_SDD_Pipeline_vNext.md`, per that spec's requirement to document all architectural changes.

## Summary

The pipeline moved from "a set of skills you copy into a target app repo, which then writes `docs/` inside that repo" to "a standalone spec-and-issue-tracking repo, sibling to one or more application repos, that application repos never see." Skills were renamed into four fixed namespaces (`project_*`, `feature_*`, `issue_*`, `system_*`) and an issue-tracking workflow was added as a first-class, feature-independent lifecycle.

## Changes

### 1. Multi-project, sibling-repo layout

Previously, `docs/features/` and `docs/mindmaps/` lived inside the application repo, and pipeline skills were copied into that repo's `.claude/skills/`. Now specs, architecture, ADRs, and issues live in `projects/<project-name>/` inside this pipeline repo, and the application repo is referenced by relative path (`repo_path` in `project.yaml`). This repo can hold more than one `<project-name>` if it's used to spec multiple applications. No skills are ever copied into an application repo.

### 2. Skill renames

| Old | New |
|---|---|
| `sdd_feature_brief` | `feature_brief` |
| `sdd_tech_design` | `feature_technical_design` |
| `sdd_api_spec` | `feature_api_spec` |
| `sdd_impl_plan` | `feature_implementation_plan` |
| `sdd_test_spec` | `feature_test_spec` |
| `sdd_adr` | split into `project_adr` and `feature_adr` |
| `sdd_consistency_check` | `system_consistency_review` |
| `orient` | folded into `system_workflow_resume` |
| `update_mindmaps` | retired (see below) |
| — (new) | `project_init`, `project_brief`, `project_architecture` |
| — (new) | `feature_init` |
| — (new) | `issue_capture`, `system_issue_classify`, `issue_investigate`, `issue_resolve`, `issue_verify` |
| — (new) | `system_next_step` |

Numeric filename prefixes (`01_feature_brief.md`, etc.) were dropped in favor of the plain names in the spec's repository layout diagram (`feature-brief.md`, etc.) — build order is defined by the workflow, not by sortable filenames.

### 3. ADR skill split

`sdd_adr` handled both project- and feature-level ADRs via a positional "project" argument. Since the new repo layout has genuinely separate `adr/` directories at the project level (`projects/{project}/adr/`) and per-feature level (`projects/{project}/features/{feature}/adr/`), and each namespace's skills must stay user-facing and single-purpose, this became two thin, template-identical skills: `project_adr` and `feature_adr`.

### 4. Test spec moved before implementation plan

Per the spec's explicit instruction, `feature_test_spec` now runs before `feature_implementation_plan`. Tests are given stable IDs (`UT-N`, `IT-N`, `E2E-N`, `EC-N`, `PT-N`); `feature_implementation_plan` requires a "Tests Satisfied" column mapping every task to the test IDs it makes possible to write and pass, plus a Test Coverage Map confirming every test ID has a home. This makes the tests the definition of "done" rather than something derived from the plan after the fact.

### 5. Issue tracking added as a sibling of features

New five-step lifecycle: `issue_capture → system_issue_classify → issue_investigate → issue_resolve → issue_verify`, ending with `system_consistency_review` to catch residual drift. Issues live at `projects/{project}/issues/ISSUE-xxxx/`, not nested under any feature, since a single issue can span multiple features or be project-wide. `issue_investigate` applies the Reconciliation classification (Code defect / Documentation drift / Unapproved implementation change / Ambiguous intent / Scope change) from the spec.

### 6. `orient` and `update_mindmaps` retired in favor of `system_workflow_resume` + `system_next_step`

The old `docs/mindmaps/` trio (`project-overview.md`, `architecture.md`, `progress.md`) served two purposes: a cached summary of project state (`project-overview.md`, `architecture.md`) and a running progress log (`progress.md`), refreshed by `update_mindmaps`.

In the new layout:
- `project-overview.md` and `architecture.md`'s roles are now filled directly by `project-brief.md` and `architecture.md` (authored deliberately via `project_brief` / `project_architecture`, not auto-regenerated).
- `progress.md` has no replacement document. Progress is now always inferable live from which documents exist under each `features/{feature-name}/` and `issues/ISSUE-xxxx/` directory and their status fields — a cached progress log would be a duplicate of information the repo structure itself already encodes, which the pipeline's own Lean Philosophy ("avoid duplicate documents") argues against.
- `orient`'s briefing behavior is preserved by `system_workflow_resume`, scoped to the new multi-project structure and scanning directory contents (rather than reading a separately-maintained progress log) to determine what's in progress.
- The "what do I run next" logic that used to live at the bottom of each skill's Review Checklist is still there for the common case, but is now also available standalone as `system_next_step`, for resuming after an interruption without needing to remember where you left off.

`system_dependency_check` and `system_document_validate`, listed as illustrative examples of the `system_*` namespace in the spec, were not implemented — no workflow step in the spec requires them, and adding them would be process the spec's own Lean Philosophy argues against ("every skill must justify its existence").

## Verification

`system_consistency_review` was extended to work project-wide (all features) in addition to its original single-feature scope, and to check that every test ID in a feature's `test-spec.md` has a home in that feature's `implementation-plan.md` Test Coverage Map (new check, made possible by change #4).

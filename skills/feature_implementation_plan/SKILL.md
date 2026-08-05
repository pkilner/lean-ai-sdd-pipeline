---
name: feature_implementation_plan
description: Generate an Implementation Plan — the sixth step in the feature workflow, run after the Test Spec. Breaks the feature into individual coding tasks with dependencies, build order, layer assignment, a Status column for tracking real progress, and an explicit mapping to the tests each task satisfies. Tests define "done"; this plan exists to satisfy them.
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case.

## Steps

1. Read the following for context:
   - `projects/{project-name}/features/{feature-name}/feature-brief.md` (required)
   - `projects/{project-name}/features/{feature-name}/technical-design.md` (required)
   - `projects/{project-name}/features/{feature-name}/api-spec.md` (if it exists)
   - `projects/{project-name}/features/{feature-name}/test-spec.md` (required — do not proceed if missing)
   - `projects/{project-name}/architecture.md` (if it exists)

2. **Review gate:** check `test-spec.md`'s Status. If `Draft`, stop here and tell the user it needs review and approval first. If the user confirms approval in response, update its Status to `Approved`, then continue.

3. Generate `projects/{project-name}/features/{feature-name}/implementation-plan.md` using the template below. Every task starts with **Status: Not Started**.

4. Tasks must be broken down to the level of individual coding tasks — not milestones or epics. Each task should be something a developer can pick up and complete independently.

5. Every task table must include a **Tests Satisfied** column listing the test IDs from `test-spec.md` (e.g. `UT-3, UT-4`) that this task's implementation makes possible to write and pass. Tier 1 (unit) tests should be satisfied by the task that implements the logic they cover; Tiers 2–4 are satisfied collectively by a dedicated phase after implementation tasks are complete. Every test ID in `test-spec.md` must be referenced by at least one task or phase and appear in the Test Coverage Map — if a test has no home, that is a gap in either the plan or the test spec and must be resolved before proceeding.

6. Every implementation plan must include a dedicated documentation task for each phase (or a consolidated documentation phase at the end). Documentation tasks cover, specifically:
   - Public interfaces (what a caller outside the module needs to know)
   - Complex or non-obvious behaviour (algorithms, edge case handling, magic constants)
   - Architectural decisions worth a comment pointing at the relevant ADR
   - Configuration (env vars, feature flags, anything that changes behaviour without a code change)
   - Setup (a README or usage note for any module/service with a non-obvious setup requirement)

   This is not "add a doc comment to every public symbol" — routine, self-explanatory code needs no comment. Documentation tasks should call out specifically which interfaces, behaviours, or config need documenting, not just say "add docs".

7. After writing the file, present the Review Checklist to the user.

8. **Review gate:** if the user confirms the checklist is satisfied, update `Status: Draft` to `Status: Approved` in the document header before ending your turn. Implementation should not begin against a Draft plan.

9. **As implementation happens** (in the application repository, `repo_path` in `project.yaml`), whenever Claude completes work on one of these tasks, update that task's Status column before ending the turn (`Not Started` → `In Progress` → `Complete`, or `Blocked` with a one-line reason in Notes if stuck). Never mark a task `Complete` without the tests it's responsible for actually existing and passing.

10. **When every task's Status becomes Complete**, automatically invoke `system_consistency_review {project-name} {feature-name}` before telling the user the feature is done, and record the outcome in this document's `Verified` header field. Do not report a feature as fully built on the basis of the implementation plan alone — completion of all tasks means **Implemented**, not **Verified**; verification requires a passing consistency review.

---

## Output Template

```markdown
# Implementation Plan: {Feature Name}

> Status: Draft
> Created: {today's date}
> Project: {project-name}
> Feature ID: {feature-name}
> Depends on: test-spec.md
> Verified: Not yet

## Overview

Brief summary of the build approach and any important sequencing notes.

## Layer Key

Define one letter per system layer used in this project (pull these from `projects/{project-name}/architecture.md` — do not default to the example below unless it's actually accurate for this project).

Example for a client/server web app:
- **F** — Frontend
- **B** — Backend
- **S** — Shared (packages/types/schemas used by both)

Example for an offline-first mobile app:
- **D** — On-device (mobile app)
- **C** — Cloud (backend service)
- **S** — Shared

## Tasks

Tasks are numbered in build order. A task cannot begin until all tasks it depends on are complete.

### Phase 1: {Phase Name}

| # | Task | Layer | Depends On | Tests Satisfied | Status | Notes |
|---|---|---|---|---|---|---|
| 1 | | | — | UT-1, UT-2 | Not Started | |
| 2 | | | 1 | — | Not Started | |

### Phase 2: {Phase Name}

| # | Task | Layer | Depends On | Tests Satisfied | Status | Notes |
|---|---|---|---|---|---|---|
| N | | | | | Not Started | |

(Add as many phases as needed. Use phases to group logically related work, e.g. "Data Layer", "Core Logic", "API", "UI", "Sync".)

### Phase N: Tier 2–4 Tests

List all Tier 2, 3, and 4 tests here as tasks, after all implementation phases are complete. Reference `test-spec.md` for details. Skip levels marked Not Applicable in test-spec.md's Test Suite Metadata.

| # | Task | Layer | Depends On | Tests Satisfied | Status | Notes |
|---|---|---|---|---|---|---|
| N | Write and run Tier 2 integration/UI flow tests | | All relevant components complete | IT-1, E2E-1 | Not Started | |
| N | Run Tier 3 system/performance benchmarks | | Feature complete | PT-1 | Not Started | Real device/staging environment required |
| N | Execute Tier 4 field/manual acceptance script | | Tier 3 tests pass | — | Not Started | |

## Test Coverage Map

Confirm every test ID from `test-spec.md` is satisfied by at least one task above, and carry forward the acceptance criterion each test covers for end-to-end traceability from AC to task.

| Test ID | Covers (AC) | Satisfied By Task # |
|---|---|---|
| UT-1 | AC-1 | 1 |

## Task Detail

For any task that requires more explanation than fits in the table, expand it here.

### Task {N}: {Task Name}

What exactly needs to be built. Reference the relevant section of the technical design or API spec.

(Only include tasks that need expansion — skip straightforward ones.)

## Milestones

| Milestone | Tasks Required | Description |
|---|---|---|
| Core logic working | 1–N | Feature works in isolation, e.g. fully offline or with mocked dependencies |
| Integrated | N–M | All layers connected end to end |
| Feature complete | All | Passes all acceptance criteria |

## Open Questions

List any unresolved implementation questions. Leave blank if none.

---

## Review Checklist

Before starting implementation, confirm:

- [ ] Every task is at the level of an individual coding task (not a milestone or epic)
- [ ] Build order is correct — no task depends on something that comes after it
- [ ] Layer assignments are correct and match the layers used in technical-design.md
- [ ] All components from the technical design are represented in the task list
- [ ] All schema and API work from the API spec has corresponding tasks
- [ ] Every task has a Tests Satisfied column entry (specific test IDs or "—") and a Status of Not Started
- [ ] Every test ID in test-spec.md appears in the Test Coverage Map with its Covers (AC) carried forward
- [ ] Tier 2, 3, and 4 tests are in a dedicated final phase
- [ ] Documentation tasks are present and call out specific interfaces/behaviours/config to document
- [ ] Milestones are meaningful and achievable

**Next step:** When approved, begin implementation in the application repository. Update each task's Status as you go. When all tasks reach Complete, this skill automatically runs `system_consistency_review` before the feature is reported as done.
```

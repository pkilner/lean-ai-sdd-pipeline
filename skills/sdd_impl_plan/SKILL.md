---
name: sdd_impl_plan
description: Generate an Implementation Plan — the fourth step in the spec-driven pipeline. Breaks the feature into individual coding tasks with dependencies, build order, and layer assignment.
---

The argument passed to this skill is the feature name in kebab-case. Use it as the feature identifier throughout.

## Steps

1. Read the following for context:
   - `docs/features/{feature-name}/01_feature_brief.md` (required)
   - `docs/features/{feature-name}/02_tech_design.md` (required)
   - `docs/features/{feature-name}/03_api_spec.md` (required — do not proceed if missing)
   - `docs/mindmaps/architecture.md` (if it exists)

2. Generate `docs/features/{feature-name}/04_impl_plan.md` using the template below.

3. Tasks must be broken down to the level of individual coding tasks — not milestones or epics. Each task should be something a developer can pick up and complete independently.

4. Every task table must include a **Tier 1 Tests** column. For each implementation task, specify which Tier 1 (unit) tests must be written alongside it — these are part of the definition of done for that task. Use the following test tier model:
   - **Tier 1 — Unit tests:** Written alongside the implementation task, in the same commit. Fast, automated, no external environment (device/emulator/browser/network) required. List the specific test cases (e.g. "valid name accepted, empty name rejected").
   - **Tier 2 — Integration / UI tests:** Automated, but may require an emulator, simulator, browser, or a running local stack. Written once the component(s) they cover exist. Do not assign these to individual tasks — list them in a dedicated phase after all implementation tasks.
   - **Tier 3 — System / performance tests:** Require a realistic environment (real device, staging infra, load-testing setup). Always deferred to a dedicated phase after the feature is functionally complete.
   - **Tier 4 — Field / manual acceptance tests:** Human-in-the-loop validation against real-world conditions (e.g. outdoor GPS testing, live user pilots). Always deferred to the final phase.
   - If a task has no unit-testable logic (e.g. a pure layout/config task), write "—" in the Tier 1 Tests column.

5. Every implementation plan must include a dedicated documentation task for each phase (or a consolidated documentation phase at the end). Documentation tasks cover:
   - Doc comments on all public types, interfaces, and enums (describing purpose, not just restating the name)
   - Doc comments on all public functions and methods (describing parameters, return values, and any side effects)
   - Inline comments for non-obvious logic (e.g. algorithms, edge case handling, magic constants)
   - A README or usage note for any module or service that has a non-obvious setup requirement

   Documentation tasks should call out specifically which types, interfaces, and functions need documenting — not just say "add docs".

6. After writing the file, present the Review Checklist to the user.

---

## Output Template

```markdown
# Implementation Plan: {Feature Name}

> Status: Draft
> Created: {today's date}
> Feature ID: {feature-name}
> Depends on: 03_api_spec.md

## Overview

Brief summary of the build approach and any important sequencing notes.

## Layer Key

Define one letter per system layer used in this project (pull these from `docs/mindmaps/architecture.md` if it exists — do not default to the example below unless it's actually accurate for this project).

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

| # | Task | Layer | Depends On | Tier 1 Tests | Notes |
|---|---|---|---|---|---|
| 1 | | | — | Test case 1, test case 2 | |
| 2 | | | 1 | — | |

### Phase 2: {Phase Name}

| # | Task | Layer | Depends On | Tier 1 Tests | Notes |
|---|---|---|---|---|---|
| N | | | | | |

(Add as many phases as needed. Use phases to group logically related work, e.g. "Data Layer", "Core Logic", "API", "UI", "Sync".)

### Phase N: Tier 2–4 Tests

List all Tier 2, 3, and 4 tests here as tasks, after all implementation phases are complete. Reference the test spec (`05_test_spec.md`) for details.

| # | Task | Layer | Depends On | Tier 1 Tests | Notes |
|---|---|---|---|---|---|
| N | Write Tier 2 integration/UI flow tests | | All relevant components complete | — | |
| N | Run Tier 3 system/performance benchmarks | | Feature complete | — | Real device/staging environment required |
| N | Execute Tier 4 field/manual acceptance script | | Tier 3 tests pass | — | |

## Task Detail

For any task that requires more explanation than fits in the table, expand it here.

### Task {N}: {Task Name}

What exactly needs to be built. Reference the relevant section of the tech design or API spec.

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

Before running the next skill, confirm:

- [ ] Every task is at the level of an individual coding task (not a milestone or epic)
- [ ] Build order is correct — no task depends on something that comes after it
- [ ] Layer assignments are correct and match the layers used in 02_tech_design.md
- [ ] All components from the tech design are represented in the task list
- [ ] All schema and API work from the API spec has corresponding tasks
- [ ] Every task has a Tier 1 Tests column entry (specific test cases or "—")
- [ ] Tier 1 tests are assigned to the task they cover, not deferred to a later phase
- [ ] Tier 2, 3, and 4 tests are in a dedicated final phase
- [ ] Documentation tasks are present and call out specific types/functions to document
- [ ] Milestones are meaningful and achievable

**Next step:** When approved, run `/sdd_test_spec {feature-name}`
```

---
name: feature_test_spec
description: Generate a Test Spec — the fifth step in the feature workflow, run before the Implementation Plan. Defines unit, integration, and end-to-end tests, edge cases, performance scenarios, and pass/fail criteria. Tests define "done" — the implementation plan is built to satisfy these tests, not the other way around.
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case.

## Steps

1. Read the following for context:
   - `projects/{project-name}/features/{feature-name}/feature-brief.md` (required)
   - `projects/{project-name}/features/{feature-name}/technical-design.md` (required)
   - `projects/{project-name}/features/{feature-name}/api-spec.md` (if it exists — skipped for features with no new contracts)

2. Generate `projects/{project-name}/features/{feature-name}/test-spec.md` using the template below.

3. Give every test a stable ID (`UT-N` for unit, `IT-N` for integration, `E2E-N` for end-to-end, `EC-N` for edge case, `PT-N` for performance). `feature_implementation_plan` references these IDs directly — do not renumber tests once `implementation-plan.md` exists without also updating it.

4. Tests must cover all three levels: unit, integration, and end-to-end. Do not skip a level unless it genuinely does not apply — explain why if so.

5. Every test spec **must** include a "Test Suite Metadata" section (see template) that defines:
   - The named test suites and which tier each belongs to (see the tier model below)
   - The command(s) used to run each suite in the application repository — check `projects/{project-name}/project.yaml` for `repo_path`, then that repository's own docs, or ask the user if it doesn't document a test runner yet
   - Whether the runner produces machine-readable (JSON/XML) and/or human-readable output, and where reports land

6. After writing the file, present the Review Checklist to the user.

---

## Test Tier Model

- **Tier 1 — Unit tests:** Written alongside the implementation task, in the same commit. Fast, automated, no external environment (device/emulator/browser/network) required.
- **Tier 2 — Integration / UI tests:** Automated, but may require an emulator, simulator, browser, or a running local stack. Written once the component(s) they cover exist.
- **Tier 3 — System / performance tests:** Require a realistic environment (real device, staging infra, load-testing setup). Always run in a dedicated phase after the feature is functionally complete.
- **Tier 4 — Field / manual acceptance tests:** Human-in-the-loop validation against real-world conditions (e.g. outdoor GPS testing, live user pilots). Always run in the final phase.

## Output Template

```markdown
# Test Spec: {Feature Name}

> Status: Draft
> Created: {today's date}
> Project: {project-name}
> Feature ID: {feature-name}
> Depends on: technical-design.md, api-spec.md (if present)

## Overview

Brief summary of the testing approach and any important notes about test coverage.

## Test Suite Metadata

Every test spec must declare the following so a master runner (if this project has one) can include it
automatically. This section is required and must be filled in — do not leave it blank.

### Suites

| Suite Name | Tier | File(s) | Runner Command |
|---|---|---|---|
| (e.g.) UnitTests | 1 — Unit | `tests/unit/` | `npm test` / `pytest` / etc. |
| (e.g.) IntegrationTests | 2 — Integration | `tests/integration/` | project-specific |
| (e.g.) E2ETests | 2/3 — E2E | `tests/e2e/` | Requires a running environment; project-specific |

### Runner command(s)

```bash
# Command(s) that run all automatable suites for this feature:
{fill in — check the application repository's own tooling first}
```

### Report format

- Machine-readable output: {format, if any}
- Human-readable output: {format, if any}
- Exit code convention: {e.g. 0 = all pass, non-zero = failures}

## Unit Tests (Tier 1)

Test individual functions, methods, and components in isolation.

### {Component or Module Name}

| ID | Test | Input | Expected Output | Notes |
|---|---|---|---|---|
| UT-1 | | | | |

(Repeat for each component that has non-trivial logic. Skip purely structural/config code.)

## Integration Tests (Tier 2)

Test interactions between components or services.

### {Integration Scenario Name}

**ID:** IT-1

**Components involved:** List the components/services being tested together.

**Setup:** What state needs to exist before the test runs.

**Steps:**
1. ...
2. ...

**Expected result:** What should be true after the steps complete.

(Repeat for each integration scenario.)

## End-to-End Tests (Tier 2/3)

Test complete user journeys from the user's perspective.

### {Journey Name}

**ID:** E2E-1

**Preconditions:** What must be true before this journey begins.

**Steps:**
1. User does X
2. System does Y
3. User sees Z

**Expected result:** What the user experiences at the end of the journey.

**Variants to test:**
- Happy path
- (List meaningful variants — e.g. offline, error states, edge inputs — give each its own ID, e.g. E2E-1a)

(Repeat for each end-to-end journey.)

## Edge Case Tests

Test boundary conditions, error states, and unexpected inputs.

| ID | Edge Case | Test Approach | Expected Behaviour |
|---|---|---|---|
| EC-1 | | | |

(Source edge cases from technical-design.md and the acceptance criteria in feature-brief.md.)

## Performance Tests (Tier 3)

Only include if the feature has performance-sensitive paths (e.g. real-time processing, high-frequency data, large payloads, high concurrency).

| ID | Scenario | Load | Success Threshold |
|---|---|---|---|
| PT-1 | | | |

## Test Data Requirements

List any specific data that needs to exist or be generated to run these tests.

- ...

## Pass / Fail Criteria

A test run is considered passing when:

1. All unit tests pass
2. All integration tests pass
3. All end-to-end journeys complete successfully on both happy path and defined variants
4. All edge cases are handled without crashes or data loss
5. Performance thresholds are met (if applicable)
6. All acceptance criteria from feature-brief.md are verified by at least one test

## Open Questions

List any unresolved testing questions. Leave blank if none.

---

## Review Checklist

Before running the next skill, confirm:

- [ ] Unit tests cover all non-trivial logic
- [ ] Integration tests cover all key component interactions
- [ ] End-to-end tests cover all user journeys including offline and error paths (if applicable)
- [ ] Every acceptance criterion from the Feature Brief is covered by at least one test
- [ ] Every edge case from the Technical Design is covered
- [ ] Pass/fail criteria are specific and measurable
- [ ] Every test has a stable ID
- [ ] **Test Suite Metadata section is complete** — all suites named with tier and runner command
- [ ] **Runner command(s) specified** and verified against the application repository's actual tooling
- [ ] **Report format confirmed**
- [ ] No open questions remain

**Next step:** When approved, run `/feature_implementation_plan {project-name} {feature-name}`
```

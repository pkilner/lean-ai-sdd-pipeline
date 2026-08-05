---
name: feature_test_spec
description: Generate a Test Spec — the fifth step in the feature workflow, run before the Implementation Plan. Defines unit, integration, and end-to-end tests (each marked Required/Optional/Not Applicable), maps every test to the acceptance criteria it validates, and sets pass/fail criteria. Tests define "done" — the implementation plan is built to satisfy these tests, not the other way around.
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case.

## Steps

1. Read the following for context:
   - `projects/{project-name}/features/{feature-name}/feature-brief.md` (required — this is where AC-N identifiers come from)
   - `projects/{project-name}/features/{feature-name}/technical-design.md` (required)
   - `projects/{project-name}/features/{feature-name}/api-spec.md` (required only if technical-design.md's `API Specification Required` is `Yes`)

2. **Review gate:** confirm `technical-design.md` has `Status: Approved`, and `api-spec.md` too if it is required for this feature. If either is still `Draft`, stop here and tell the user which document needs review/approval first. If the user confirms approval in response, update that document's Status to `Approved`, then continue.

3. Generate `projects/{project-name}/features/{feature-name}/test-spec.md` using the template below.

4. Give every test a stable ID (`UT-N` for unit, `IT-N` for integration, `E2E-N` for end-to-end, `EC-N` for edge case, `PT-N` for performance). `feature_implementation_plan` references these IDs directly — do not renumber tests once `implementation-plan.md` exists without also updating it.

5. Every test must state which acceptance criterion (or criteria) it validates, using the `AC-N` identifiers from `feature-brief.md`, in a **Covers** column/field. A test with no functional AC to validate (e.g. a pure performance benchmark) may use `—`, but this should be the exception, not the default — most tests should trace back to an AC.

6. For each of the three test levels (Unit, Integration, End-to-End), explicitly mark it `Required`, `Optional`, or `Not Applicable` in the Test Suite Metadata section, with a one-line reason for anything not `Required`. A small utility feature may legitimately mark Integration and/or End-to-End as `Not Applicable` — do not force coverage that doesn't fit the feature's shape. Unit is `Required` unless the feature genuinely has no non-trivial logic.

7. Every test spec **must** include a "Test Suite Metadata" section (see template) that defines:
   - The named test suites and which tier each belongs to (see the tier model below)
   - The command(s) used to run each suite in the application repository — check `projects/{project-name}/project.yaml` for `repo_path`, then that repository's own docs, or ask the user if it doesn't document a test runner yet
   - Whether the runner produces machine-readable (JSON/XML) and/or human-readable output, and where reports land

8. After writing the file, present the Review Checklist to the user.

9. **Review gate:** if the user confirms the checklist is satisfied, update `Status: Draft` to `Status: Approved` in the document header before ending your turn. Until this document is Approved, `feature_implementation_plan` will refuse to proceed.

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
> Depends on: technical-design.md, api-spec.md (if required)

## Overview

Brief summary of the testing approach and any important notes about test coverage.

## Test Suite Metadata

Every test spec must declare the following so a master runner (if this project has one) can include it
automatically. This section is required and must be filled in — do not leave it blank.

### Levels

| Level | Required / Optional / Not Applicable | Reason (if not Required) |
|---|---|---|
| Unit | | |
| Integration | | |
| End-to-End | | |

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

Test individual functions, methods, and components in isolation. Omit this section only if Unit is marked Not Applicable above.

### {Component or Module Name}

| ID | Test | Input | Expected Output | Covers | Notes |
|---|---|---|---|---|---|
| UT-1 | | | | AC-1 | |

(Repeat for each component that has non-trivial logic. Skip purely structural/config code.)

## Integration Tests (Tier 2)

Test interactions between components or services. Omit this section only if Integration is marked Not Applicable above.

### {Integration Scenario Name}

**ID:** IT-1

**Covers:** AC-N

**Components involved:** List the components/services being tested together.

**Setup:** What state needs to exist before the test runs.

**Steps:**
1. ...
2. ...

**Expected result:** What should be true after the steps complete.

(Repeat for each integration scenario.)

## End-to-End Tests (Tier 2/3)

Test complete user journeys from the user's perspective. Omit this section only if End-to-End is marked Not Applicable above.

### {Journey Name}

**ID:** E2E-1

**Covers:** AC-N

**Preconditions:** What must be true before this journey begins.

**Steps:**
1. User does X
2. System does Y
3. User sees Z

**Expected result:** What the user experiences at the end of the journey.

**Variants to test:**
- Happy path
- (List meaningful variants — e.g. offline, error states, edge inputs — give each its own ID, e.g. E2E-1a, and its own Covers if it validates a different AC)

(Repeat for each end-to-end journey.)

## Edge Case Tests

Test boundary conditions, error states, and unexpected inputs.

| ID | Edge Case | Test Approach | Expected Behaviour | Covers |
|---|---|---|---|---|
| EC-1 | | | | AC-N |

(Source edge cases from technical-design.md and the acceptance criteria in feature-brief.md.)

## Performance Tests (Tier 3)

Only include if the feature has performance-sensitive paths (e.g. real-time processing, high-frequency data, large payloads, high concurrency).

| ID | Scenario | Load | Success Threshold | Covers |
|---|---|---|---|---|
| PT-1 | | | | — |

## Test Data Requirements

List any specific data that needs to exist or be generated to run these tests.

- ...

## Pass / Fail Criteria

A test run is considered passing when:

1. All Required-level tests pass (Optional levels, if run, should also pass but do not block)
2. All end-to-end journeys complete successfully on both happy path and defined variants (if End-to-End is Required)
3. All edge cases are handled without crashes or data loss
4. Performance thresholds are met (if applicable)
5. Every acceptance criterion from feature-brief.md is covered by at least one test with a matching Covers reference

## Open Questions

List any unresolved testing questions. Leave blank if none.

---

## Review Checklist

Before running the next skill, confirm:

- [ ] Each of Unit / Integration / End-to-End is explicitly marked Required, Optional, or Not Applicable, with a reason for anything not Required
- [ ] Unit tests cover all non-trivial logic (if Required)
- [ ] Integration tests cover all key component interactions (if Required)
- [ ] End-to-end tests cover all user journeys including offline and error paths, if applicable (if Required)
- [ ] Every acceptance criterion from the Feature Brief is covered by at least one test's Covers field
- [ ] Every test has a stable ID and a Covers reference (or `—` with good reason)
- [ ] Every edge case from the Technical Design is covered
- [ ] Pass/fail criteria are specific and measurable
- [ ] **Test Suite Metadata section is complete** — all suites named with tier and runner command
- [ ] **Runner command(s) specified** and verified against the application repository's actual tooling
- [ ] **Report format confirmed**
- [ ] No open questions remain

**Next step:** When approved, run `/feature_implementation_plan {project-name} {feature-name}`
```

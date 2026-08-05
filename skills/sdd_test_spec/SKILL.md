---
name: sdd_test_spec
description: Generate a Test Spec — the fifth and final step in the spec-driven pipeline. Defines unit, integration, and end-to-end tests, edge cases, performance scenarios, and pass/fail criteria.
---

The argument passed to this skill is the feature name in kebab-case. Use it as the feature identifier throughout.

## Steps

1. Read the following for context:
   - `docs/features/{feature-name}/01_feature_brief.md` (required)
   - `docs/features/{feature-name}/02_tech_design.md` (required)
   - `docs/features/{feature-name}/03_api_spec.md` (required)
   - `docs/features/{feature-name}/04_impl_plan.md` (required — do not proceed if missing)

2. Generate `docs/features/{feature-name}/05_test_spec.md` using the template below.

3. Tests must cover all three levels: unit, integration, and end-to-end. Do not skip a level unless it genuinely does not apply — explain why if so.

4. Every test spec **must** include a "Test Suite Metadata" section (see template) that defines:
   - The named test suites and which tier each belongs to (see the tier model in `sdd_impl_plan`)
   - The command(s) used to run each suite in this project — check `CLAUDE.md` or ask the user if this project doesn't document a test runner yet
   - Whether the runner produces machine-readable (JSON/XML) and/or human-readable output, and where reports land

5. After writing the file, present the Review Checklist to the user.

---

## Output Template

```markdown
# Test Spec: {Feature Name}

> Status: Draft
> Created: {today's date}
> Feature ID: {feature-name}
> Depends on: 04_impl_plan.md

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
{fill in — check CLAUDE.md or the project's existing test tooling first}
```

### Report format

- Machine-readable output: {format, if any}
- Human-readable output: {format, if any}
- Exit code convention: {e.g. 0 = all pass, non-zero = failures}

## Unit Tests

Test individual functions, methods, and components in isolation.

### {Component or Module Name}

| Test | Input | Expected Output | Notes |
|---|---|---|---|
| | | | |

(Repeat for each component that has non-trivial logic. Skip purely structural/config code.)

## Integration Tests

Test interactions between components or services.

### {Integration Scenario Name}

**Components involved:** List the components/services being tested together.

**Setup:** What state needs to exist before the test runs.

**Steps:**
1. ...
2. ...

**Expected result:** What should be true after the steps complete.

(Repeat for each integration scenario.)

## End-to-End Tests

Test complete user journeys from the user's perspective.

### {Journey Name}

**Preconditions:** What must be true before this journey begins.

**Steps:**
1. User does X
2. System does Y
3. User sees Z

**Expected result:** What the user experiences at the end of the journey.

**Variants to test:**
- Happy path
- (List meaningful variants — e.g. offline, error states, edge inputs)

(Repeat for each end-to-end journey.)

## Edge Case Tests

Test boundary conditions, error states, and unexpected inputs.

| Edge Case | Test Approach | Expected Behaviour |
|---|---|---|

(Source edge cases from 02_tech_design.md and the acceptance criteria in 01_feature_brief.md.)

## Performance Tests

Only include if the feature has performance-sensitive paths (e.g. real-time processing, high-frequency data, large payloads, high concurrency).

| Scenario | Load | Success Threshold |
|---|---|---|

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
6. All acceptance criteria from 01_feature_brief.md are verified by at least one test

## Open Questions

List any unresolved testing questions. Leave blank if none.

---

## Review Checklist

Before considering the pipeline complete, confirm:

- [ ] Unit tests cover all non-trivial logic
- [ ] Integration tests cover all key component interactions
- [ ] End-to-end tests cover all user journeys including offline and error paths (if applicable)
- [ ] Every acceptance criterion from the Feature Brief is covered by at least one test
- [ ] Every edge case from the Tech Design is covered
- [ ] Pass/fail criteria are specific and measurable
- [ ] **Test Suite Metadata section is complete** — all suites named with tier and runner command
- [ ] **Runner command(s) specified** and verified against this project's actual tooling
- [ ] **Report format confirmed**
- [ ] No open questions remain

**Pipeline complete.** This feature is fully specified. Ready for implementation.
```

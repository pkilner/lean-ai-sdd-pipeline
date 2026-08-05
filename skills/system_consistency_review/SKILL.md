---
name: system_consistency_review
description: Check consistency of pipeline documents and implementation. Read-only diagnostic — cross-references field names, formulas, and test coverage across a feature's documents and the application code, or across all features in a project. Does not auto-fix. Final step in the issue workflow and usable at any point in the feature workflow.
---

The arguments passed to this skill are the project name in kebab-case, optionally followed by a feature name in kebab-case. If no feature name is given, run the check across every feature under `projects/{project-name}/features/`.

## Steps

1. For each feature in scope, read all pipeline documents in `projects/{project-name}/features/{feature-name}/`:
   - `feature-brief.md` — extract acceptance criteria identifiers (AC-N)
   - `technical-design.md` — extract: state/field names, formula parameter names, edge case descriptions
   - `api-spec.md` — extract: data model field names, function/endpoint parameter names, validation rule field names (skip if this feature has no api-spec.md)
   - `test-spec.md` — extract: test IDs, test input field names, AC references
   - `implementation-plan.md` — extract: any field names mentioned in task descriptions, and the Test Coverage Map

2. Locate the implementation file(s) for this feature in the application repository (`repo_path` in `projects/{project-name}/project.yaml`). Use the layer assignments in `implementation-plan.md` together with the application repo's actual folder structure (check `projects/{project-name}/architecture.md` for where each layer's code lives) to find the relevant source files.

3. If implementation files exist, extract from them:
   - State/model field names (from the equivalent of a default-state object, struct, or schema definition)
   - Function/method parameter names for the logic described in `technical-design.md`
   - UI label strings or API responses that reference field names

4. Cross-reference and report the following categories of inconsistency:

   ### A. Field Name Mismatches
   Fields that appear in one document but not another, or with different names.

   | Field | In technical-design | In api-spec | In test-spec | In Implementation |
   |---|---|---|---|---|
   (Fill in for each field found — ✓ if present, ✗ if missing, ~ if partially/renamed)

   ### B. Formula Inconsistencies
   Formulas in `technical-design.md` that conflict with implementation code.
   List each formula with the spec version and the implementation version side by side.

   ### C. Stale Test Inputs
   Test inputs in `test-spec.md` that reference field names not in `api-spec.md`.
   List each stale reference with the field name used and the correct current name.

   ### D. Missing Test Coverage
   Acceptance criteria from `feature-brief.md` that are not referenced in `test-spec.md`. Also flag any test ID in `test-spec.md` that does not appear in `implementation-plan.md`'s Test Coverage Map.
   List each uncovered AC-N identifier and each orphaned test ID.

   ### E. Validation Rule Gaps
   Fields in `api-spec.md` with validation rules that have no corresponding edge case or test in `test-spec.md`.

   ### F. Implementation Drift
   Fields or functions in the implementation that are not documented in `api-spec.md`.
   Fields or functions in `api-spec.md` that are not present in the implementation.

5. Produce a summary with a severity rating for each finding:
   - **Critical** — field name mismatch between api-spec and implementation (will cause runtime errors)
   - **High** — test inputs reference wrong field names (tests would fail silently)
   - **Medium** — formula in technical-design contradicts implementation
   - **Low** — documentation gap (missing coverage, no runtime impact)

6. End with a **Consistency Score**: X/Y checks passing, where Y is the total number of checks performed. If run across multiple features, report one score per feature plus an overall total.

## Output Format

```markdown
# Consistency Review: {Project Name}{ / Feature Name if scoped}

> Run date: {today's date}
> Project: {project-name}
> Feature(s) checked: {feature-name or "all"}
> Documents checked: {list of docs found}
> Implementation checked: {path(s) or "not found"}

## Summary

{N} issues found across {M} checks.

| Severity | Count |
|---|---|
| Critical | N |
| High | N |
| Medium | N |
| Low | N |

## A. Field Name Mismatches

{table or "No mismatches found"}

## B. Formula Inconsistencies

{findings or "No inconsistencies found"}

## C. Stale Test Inputs

{findings or "No stale references found"}

## D. Missing Test Coverage

{findings or "All acceptance criteria covered"}

## E. Validation Rule Gaps

{findings or "All validation rules have test coverage"}

## F. Implementation Drift

{findings or "Implementation matches specification"}

---

## Consistency Score: {X}/{Y} checks passing

### Recommended Actions (in priority order)

1. {Critical fix}
2. {High priority fix}
...

If no issues: "All checks pass. Documents are consistent with implementation."
```

## Notes

- This skill is **read-only**. It reports gaps but does not modify any files.
- If the implementation does not exist yet, skip Section F and note that implementation has not started.
- When comparing field names, treat naming-convention variations as distinct (e.g. `samplingInterval` ≠ `sampling_interval`) unless the project has an established, documented convention for translating between them (e.g. camelCase in code vs snake_case in a database).
- If a document is missing (e.g. `implementation-plan.md` was not generated), note it but continue with the remaining documents.
- If any finding traces back to a genuine defect, unapproved change, or ambiguous intent rather than simple drift, recommend `/issue_capture {project-name} "..."` to track it formally rather than fixing it silently.
- After reporting, suggest running `/system_consistency_review {project-name} {feature-name}` again after fixes to confirm resolution.

## Change Propagation Reminder

If inconsistencies are found, remind the user of the correct propagation order:

```
feature-brief → technical-design → api-spec → test-spec → implementation-plan → implementation
```

Changes should always flow top-down. If a change originates in the implementation (e.g. a bug discovery), update upward first (technical-design → api-spec), then confirm downward (test-spec → implementation-plan → implementation).

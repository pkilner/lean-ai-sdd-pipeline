---
name: sdd_consistency_check
description: Check consistency of all pipeline documents and implementation for a feature. Reads all docs in docs/features/{feature-name}/ and the implementation code, then cross-references parameter names, field names, formulas, and test inputs. Reports gaps and inconsistencies. Read-only diagnostic — does not auto-fix.
---

The argument passed to this skill is the feature name in kebab-case.

## Steps

1. Read all pipeline documents in `docs/features/{feature-name}/`:
   - `01_feature_brief.md` — extract acceptance criteria identifiers (AC-N)
   - `02_tech_design.md` — extract: state/field names, formula parameter names, edge case descriptions
   - `03_api_spec.md` — extract: data model field names, function/endpoint parameter names, validation rule field names
   - `04_impl_plan.md` — extract: any field names mentioned in task descriptions
   - `05_test_spec.md` — extract: test input field names, AC references

2. Locate the implementation file(s) for this feature. Use the layer assignments in `04_impl_plan.md` together with this project's actual folder structure (check `docs/mindmaps/architecture.md` and/or `CLAUDE.md` for where each layer's code lives) to find the relevant source files.

3. If implementation files exist, extract from them:
   - State/model field names (from the equivalent of a default-state object, struct, or schema definition)
   - Function/method parameter names for the logic described in 02_tech_design.md
   - UI label strings or API responses that reference field names

4. Cross-reference and report the following categories of inconsistency:

   ### A. Field Name Mismatches
   Fields that appear in one document but not another, or with different names.

   | Field | In 02_tech_design | In 03_api_spec | In 05_test_spec | In Implementation |
   |---|---|---|---|---|
   (Fill in for each field found — ✓ if present, ✗ if missing, ~ if partially/renamed)

   ### B. Formula Inconsistencies
   Formulas in 02_tech_design.md that conflict with implementation code.
   List each formula with the spec version and the implementation version side by side.

   ### C. Stale Test Inputs
   Test inputs in 05_test_spec.md that reference field names not in 03_api_spec.md.
   List each stale reference with the field name used and the correct current name.

   ### D. Missing Test Coverage
   Acceptance criteria from 01_feature_brief.md that are not referenced in 05_test_spec.md.
   List each uncovered AC-N identifier.

   ### E. Validation Rule Gaps
   Fields in 03_api_spec.md with validation rules that have no corresponding edge case or test in 05_test_spec.md.

   ### F. Implementation Drift
   Fields or functions in the implementation that are not documented in 03_api_spec.md.
   Fields or functions in 03_api_spec.md that are not present in the implementation.

5. Produce a summary with a severity rating for each finding:
   - **Critical** — field name mismatch between api_spec and implementation (will cause runtime errors)
   - **High** — test inputs reference wrong field names (tests would fail silently)
   - **Medium** — formula in tech_design contradicts implementation
   - **Low** — documentation gap (missing coverage, no runtime impact)

6. End with a **Consistency Score**: X/Y checks passing, where Y is the total number of checks performed.

## Output Format

```markdown
# Consistency Check: {Feature Name}

> Run date: {today's date}
> Feature ID: {feature-name}
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
- If a document is missing (e.g. 04_impl_plan.md was not generated), note it but continue with the remaining documents.
- After reporting, suggest running `/sdd_consistency_check {feature-name}` again after fixes to confirm resolution.

## Change Propagation Reminder

If inconsistencies are found, remind the user of the correct propagation order:

```
01_feature_brief → 02_tech_design → 03_api_spec → 04_impl_plan → 05_test_spec → implementation
```

Changes should always flow top-down. If a change originates in the implementation (e.g. a bug discovery), update upward first (tech_design → api_spec), then confirm downward (impl_plan → test_spec → implementation).

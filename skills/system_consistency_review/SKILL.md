---
name: system_consistency_review
description: (Internal — not for direct user invocation.) Read-only diagnostic across project, feature, and issue artifacts — checks project.yaml/architecture/ADR consistency, AC-to-test-to-implementation coverage, documentation and implementation drift, and issue status coherence. Invoked automatically by feature_05_implementation_plan (on feature completion) and issue_04_verify (on issue closure), or by Claude when asked to check consistency. Does not auto-fix.
user-invocable: false
---

**This is a system skill.** It is never invoked directly by a user typing a command — it runs automatically when a feature's implementation plan reaches all-Complete, when an issue is closed, or when Claude decides a consistency check is warranted in response to a plain-language request (e.g. "is this feature actually done?"). Do not present this as something the user should run themselves; if a user wants to check consistency, invoke this skill on their behalf rather than telling them to run it.

The arguments passed to this skill are the project name in kebab-case, optionally followed by a feature name in kebab-case. If no feature name is given, run the Project checks plus the Feature checks for every feature under `projects/{project-name}/features/`.

## Steps

### A. Project Checks

1. Validate `projects/{project-name}/project.yaml` — required fields present, `repo_path` still resolves to a valid git repository not equal to the pipeline repo itself (same checks as `project_00_init`).
2. Compare `02_project_architecture.md`'s layer list and repository paths against what actually exists in the application repo — flag layers whose declared `Repository path` doesn't exist.
3. Check every row in `02_project_architecture.md`'s Locked Architectural Decisions table has a corresponding ADR in `projects/{project-name}/adr/` with `Status: Accepted`. Flag rows with no ADR, or whose ADR is still `Proposed`.

### B. Feature Checks (per feature in scope)

1. Read all pipeline documents in `projects/{project-name}/features/{feature-name}/`:
   - `01_feature_brief.md` — extract `AC-N` identifiers
   - `02_technical_design.md` — extract: state/field names, formula parameter names, edge case descriptions, `API Specification Required`
   - `03_api_spec.md` — extract: data model field names, function/endpoint parameter names, validation rule field names (skip if this feature has no 03_api_spec.md and 02_technical_design.md says it isn't required)
   - `04_test_spec.md` — extract: test IDs, `Covers` (AC-N) references, test input field names
   - `05_implementation_plan.md` — extract: field names mentioned in task descriptions, task Status values, and the Test Coverage Map

2. Locate the implementation file(s) for this feature in the application repository, using `02_project_architecture.md`'s per-layer `Repository path` fields together with the layer assignments in `05_implementation_plan.md`.

3. If implementation files exist, extract from them: state/model field names, function/method parameter names for logic described in `02_technical_design.md`, and UI label strings or API responses that reference field names.

4. Cross-reference and report the following:

   ### B1. Field Name Mismatches
   Fields that appear in one document but not another, or with different names.

   | Field | In 02_technical_design | In 03_api_spec | In 04_test_spec | In Implementation |
   |---|---|---|---|---|
   (✓ present, ✗ missing, ~ partially/renamed)

   ### B2. Formula Inconsistencies
   Formulas in `02_technical_design.md` that conflict with implementation code — spec version vs. implementation version, side by side.

   ### B3. Stale Test Inputs
   Test inputs in `04_test_spec.md` that reference field names not in `03_api_spec.md`.

   ### B4. Acceptance Criteria Coverage
   For every `AC-N` in `01_feature_brief.md`, confirm at least one test in `04_test_spec.md` has a matching `Covers` reference, and that test ID appears in `05_implementation_plan.md`'s Test Coverage Map. Flag: ACs with no covering test, tests with a `Covers` reference to an AC that doesn't exist (stale reference), and test IDs missing from the Test Coverage Map.

   ### B5. Validation Rule Gaps
   Fields in `03_api_spec.md` with validation rules that have no corresponding edge case or test in `04_test_spec.md`.

   ### B6. Implementation Drift
   Fields or functions in the implementation not documented in `03_api_spec.md`, and vice versa.

   ### B7. Status Integrity
   Any task marked `Complete` in `05_implementation_plan.md` whose Tests Satisfied are not actually present/passing in the application repo. A task should never be `Complete` without its tests existing.

### C. Issue Checks (issues referencing this feature, or all issues if project-wide)

1. For each issue under `projects/{project-name}/issues/` (filtered to ones listing this feature in `Related Features / ADRs`, when scoped to a feature):
   - Confirm `Status: Closed` issues have a `04_verification.md` with `Result: Pass`.
   - Confirm every artifact referenced in the issue (features, ADRs) still exists.
   - Flag any issue stuck in a non-terminal status with no Status History activity — note it for the user's attention (do not guess why).

### D. Severity and Verdict

Rate each finding:
- **Critical** — field name mismatch between 03_api_spec and implementation, or a Complete task with missing tests (will cause runtime errors or false confidence)
- **High** — test inputs reference wrong field names, or an AC has no covering test (tests would fail silently or done-ness is unverifiable)
- **Medium** — formula in 02_technical_design contradicts implementation, or a Locked decision lacks an Accepted ADR
- **Low** — documentation gap, no runtime impact

Produce a single verdict, not a numeric score:
- **FAIL** — at least one Critical finding
- **PASS WITH WARNINGS** — no Critical findings, but at least one High/Medium/Low finding
- **PASS** — no findings at all

## Output Format

```markdown
# Consistency Review: {Project Name}{ / Feature Name if scoped}

> Run date: {today's date}
> Project: {project-name}
> Feature(s) checked: {feature-name or "all"}
> Documents checked: {list of docs found}
> Implementation checked: {path(s) or "not found"}

## Verdict: PASS / PASS WITH WARNINGS / FAIL

## Summary

{N} findings ({X} Critical, {Y} High, {Z} Medium, {W} Low).

## A. Project Checks

{findings or "No issues found"}

## B. Feature Checks

### B1. Field Name Mismatches
### B2. Formula Inconsistencies
### B3. Stale Test Inputs
### B4. Acceptance Criteria Coverage
### B5. Validation Rule Gaps
### B6. Implementation Drift
### B7. Status Integrity

(each: table/findings, or "No issues found")

## C. Issue Checks

{findings or "No issues found"}

---

### Recommended Actions (in priority order)

1. {Critical fix}
2. {High priority fix}
...

If PASS: "All checks pass. Documents are consistent with implementation."
```

## Notes

- This skill is **read-only**. It reports gaps and recommendations but does not modify any files.
- If the implementation does not exist yet, skip B6/B7 and note that implementation has not started.
- When comparing field names, treat naming-convention variations as distinct (e.g. `samplingInterval` ≠ `sampling_interval`) unless the project has an established, documented convention for translating between them.
- If a document is missing (e.g. `05_implementation_plan.md` was not generated), note it but continue with the remaining documents.
- If any finding traces back to a genuine defect, unapproved change, or ambiguous intent rather than simple drift, recommend capturing it as a new issue (`issue_01_capture`) rather than fixing it silently here.

## Change Propagation Reminder

If inconsistencies are found between code and documentation, do not assume the documentation should move toward whatever the implementation currently does. Recommend this sequence instead:

1. **Determine the intended behaviour** — what should actually happen, independent of what either the docs or the code currently say.
2. **Identify the highest authoritative artifact that must change** to reflect that intent (e.g. `01_feature_brief.md` if an acceptance criterion itself was wrong, `02_technical_design.md` if the approach was wrong, `03_api_spec.md` if only a contract detail was wrong).
3. **Update that artifact first.**
4. **Propagate the change through downstream documents**, in order: `01_feature_brief → 02_technical_design → 03_api_spec → 04_test_spec → 05_implementation_plan`.
5. **Update the implementation last**, to match the now-corrected documents.

If the approved documents remain correct and only the implementation is wrong, update the code only — do not modify approved documents merely to match an unapproved implementation change. The specification remains the source of truth unless an approved change explicitly changes it. When it's unclear which side is wrong, recommend `issue_01_capture` to formally investigate rather than resolving the direction here.

---
name: system_consistency_review
description: (Internal — not for direct user invocation.) Read-only diagnostic across project, feature, and issue artifacts — checks project.yaml/architecture/ADR consistency, AC-to-test-to-implementation coverage, documentation and implementation drift, and issue status coherence. Invoked automatically by feature_implementation_plan (on feature completion) and issue_verify (on issue closure), or by Claude when asked to check consistency. Does not auto-fix.
user-invocable: false
---

**This is a system skill.** It is never invoked directly by a user typing a command — it runs automatically when a feature's implementation plan reaches all-Complete, when an issue is closed, or when Claude decides a consistency check is warranted in response to a plain-language request (e.g. "is this feature actually done?"). Do not present this as something the user should run themselves; if a user wants to check consistency, invoke this skill on their behalf rather than telling them to run it.

The arguments passed to this skill are the project name in kebab-case, optionally followed by a feature name in kebab-case. If no feature name is given, run the Project checks plus the Feature checks for every feature under `projects/{project-name}/features/`.

## Steps

### A. Project Checks

1. Validate `projects/{project-name}/project.yaml` — required fields present, `repo_path` still resolves to a valid git repository not equal to the pipeline repo itself (same checks as `project_init`).
2. Compare `architecture.md`'s layer list and repository paths against what actually exists in the application repo — flag layers whose declared `Repository path` doesn't exist.
3. Check every row in `architecture.md`'s Locked Architectural Decisions table has a corresponding ADR in `projects/{project-name}/adr/` with `Status: Accepted`. Flag rows with no ADR, or whose ADR is still `Proposed`.

### B. Feature Checks (per feature in scope)

1. Read all pipeline documents in `projects/{project-name}/features/{feature-name}/`:
   - `feature-brief.md` — extract `AC-N` identifiers
   - `technical-design.md` — extract: state/field names, formula parameter names, edge case descriptions, `API Specification Required`
   - `api-spec.md` — extract: data model field names, function/endpoint parameter names, validation rule field names (skip if this feature has no api-spec.md and technical-design.md says it isn't required)
   - `test-spec.md` — extract: test IDs, `Covers` (AC-N) references, test input field names
   - `implementation-plan.md` — extract: field names mentioned in task descriptions, task Status values, and the Test Coverage Map

2. Locate the implementation file(s) for this feature in the application repository, using `architecture.md`'s per-layer `Repository path` fields together with the layer assignments in `implementation-plan.md`.

3. If implementation files exist, extract from them: state/model field names, function/method parameter names for logic described in `technical-design.md`, and UI label strings or API responses that reference field names.

4. Cross-reference and report the following:

   ### B1. Field Name Mismatches
   Fields that appear in one document but not another, or with different names.

   | Field | In technical-design | In api-spec | In test-spec | In Implementation |
   |---|---|---|---|---|
   (✓ present, ✗ missing, ~ partially/renamed)

   ### B2. Formula Inconsistencies
   Formulas in `technical-design.md` that conflict with implementation code — spec version vs. implementation version, side by side.

   ### B3. Stale Test Inputs
   Test inputs in `test-spec.md` that reference field names not in `api-spec.md`.

   ### B4. Acceptance Criteria Coverage
   For every `AC-N` in `feature-brief.md`, confirm at least one test in `test-spec.md` has a matching `Covers` reference, and that test ID appears in `implementation-plan.md`'s Test Coverage Map. Flag: ACs with no covering test, tests with a `Covers` reference to an AC that doesn't exist (stale reference), and test IDs missing from the Test Coverage Map.

   ### B5. Validation Rule Gaps
   Fields in `api-spec.md` with validation rules that have no corresponding edge case or test in `test-spec.md`.

   ### B6. Implementation Drift
   Fields or functions in the implementation not documented in `api-spec.md`, and vice versa.

   ### B7. Status Integrity
   Any task marked `Complete` in `implementation-plan.md` whose Tests Satisfied are not actually present/passing in the application repo. A task should never be `Complete` without its tests existing.

### C. Issue Checks (issues referencing this feature, or all issues if project-wide)

1. For each issue under `projects/{project-name}/issues/` (filtered to ones listing this feature in `Related Features / ADRs`, when scoped to a feature):
   - Confirm `Status: Closed` issues have a `verification.md` (or inline `## Verification`) with `Result: Pass`.
   - Confirm every artifact referenced in the issue (features, ADRs) still exists.
   - Flag any issue stuck in a non-terminal status with no Status History activity — note it for the user's attention (do not guess why).

### D. Severity and Verdict

Rate each finding:
- **Critical** — field name mismatch between api-spec and implementation, or a Complete task with missing tests (will cause runtime errors or false confidence)
- **High** — test inputs reference wrong field names, or an AC has no covering test (tests would fail silently or done-ness is unverifiable)
- **Medium** — formula in technical-design contradicts implementation, or a Locked decision lacks an Accepted ADR
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
- If a document is missing (e.g. `implementation-plan.md` was not generated), note it but continue with the remaining documents.
- If any finding traces back to a genuine defect, unapproved change, or ambiguous intent rather than simple drift, recommend capturing it as a new issue (`issue_capture`) rather than fixing it silently here.

## Change Propagation Reminder

If inconsistencies are found, remind the user of the correct propagation order:

```
feature-brief → technical-design → api-spec → test-spec → implementation-plan → implementation
```

Changes should always flow top-down. If a change originates in the implementation (e.g. a bug discovery), update upward first (technical-design → api-spec), then confirm downward (test-spec → implementation-plan → implementation).

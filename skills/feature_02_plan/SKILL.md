---
name: feature_02_plan
description: Generate a Feature Plan — the third step in the feature workflow. One document listing the tests (each mapped to the AC-N it validates) and the coding tasks that satisfy them, with a Status column for tracking real progress. Tests define "done"; the tasks exist to satisfy them.
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Keep supporting work proportional

Name the user benefit, smallest deliverable and exclusions. Separate product and
supporting tasks; reuse existing tools/tests before adding new ones. Escalate new
subsystems, providers, material contracts or costs for scope review. Keep cost and
complexity notes in the existing closeout record, with unknowns explicit. Honor
an owner's batch implementation/testing policy; do not turn task dependencies
into compulsory per-task suite execution or invent test/code ratios.

## Integration and release readiness

In the plan, identify material external assumptions: actual execution identity, permissions, provider behavior, input/output boundaries and recovery dependencies. Resolve design-blocking unknowns before implementation approval. Prove required operational prerequisites with bounded read-only checks under the actual identity before release; owner credentials are not equivalent. Unknown required prerequisites block release, not unrelated local work. Checks that require new access or external actions retain their approval requirements.

For changed boundaries, plan representative producer-to-consumer validation using actual generated outputs, including relevant failure states. Isolated component tests do not prove the connection works. Reuse existing coverage and distinguish local, provider and live evidence. This does not require TDD.

Name one integration/closeout owner. Prepare a bounded batch listing exact artifacts, destinations, authorization, order, recovery and stop conditions. Batch focused fixes before required release gates; reuse only qualifying evidence. Documentation-only changes do not justify an application release unless they affect the deployed artifact. Record implementation, local validation, live verification and owner acceptance separately; never label unproven prerequisites as release-ready.

## Steps

1. Read the following for context:
   - `{project-root}/features/{feature-name}/01_feature_spec.md` (required — do not proceed if missing; this is where the AC-N identifiers and the design come from)
   - `{project-root}/02_project_architecture.md`

2. **Review gate:** check `01_feature_spec.md`'s Status. If `Draft`, stop here and tell the user it needs review and approval first. If the user confirms approval in response, update its Status to `Approved`, add `> Approved: {today's date}` on the line beneath it, then continue.

3. Generate `{project-root}/features/{feature-name}/02_feature_plan.md` using the template below. It always starts `Status: Draft` with no `> Approved:` line.

4. Define validation obligations before task breakdown. This is acceptance planning, not a mandatory test-first implementation sequence:
   - Reuse existing coverage where sufficient; justify new tests by a distinct acceptance criterion or regression risk. Implementation and tests may be developed in either order, but required checks must pass before completion. No mandatory red/green evidence or test/code ratio.
   - Visual and documentation obligations may use stable `RV-N` review IDs with explicit expected outcomes and recorded evidence; do not manufacture unit tests for prose or styling.
   - For security, billing, data integrity and release controls, explicitly cover relevant denial, failure and recovery behavior without imposing TDD.
   - Give every test a stable ID: `UT-N` unit, `IT-N` integration, `E2E-N` end-to-end, `EC-N` edge case, `PT-N` performance. Do not renumber once tasks reference them.
   - Every test states the acceptance criteria it validates in **Covers**, using the `AC-N` identifiers from the spec. A test with no AC to validate may use `—`, as the exception.
   - Every AC must be covered by at least one automated test or explicitly planned visual/document review.
   - Mark each of Unit, Integration, and End-to-End `Required`, `Optional`, or `Not Applicable`, with a one-line reason for anything not `Required`. Do not force coverage that doesn't fit the feature's shape.
   - Give the command that runs the tests in the application repository — locate it via `repo_path` in the project pointer and that repository's own tooling, or ask the user if none is documented yet.

5. Then write the tasks:
   - Break the work down to individual coding tasks, not milestones. Each starts `Not Started`.
   - Every task lists the test IDs it makes possible to write and pass in **Tests Satisfied**. Every test must be satisfied by at least one task; record that task number in the test's **Task** column. A test with no task, or an AC with no test, is a gap to resolve before proceeding.
   - Add a documentation task wherever a public interface, non-obvious behaviour, configuration, or setup needs explaining, naming what specifically to document.

6. After writing the file, present the Review Checklist to the user.

7. **Review gate:** if the user confirms the checklist is satisfied, update `Status: Draft` to `Status: Approved` in the document header, and add `> Approved: {today's date}` on the line beneath it, before ending your turn. Implementation should not begin against a Draft plan.

8. **As implementation happens** (in the application repository, outside `sdd/`), whenever Claude completes work on one of these tasks, update that task's Status before ending the turn (`Not Started` → `In Progress` → `Complete`, or `Blocked` with a one-line reason in Notes). Never mark a task `Complete` until its required tests pass and any planned review has recorded evidence.

9. At closeout, report delivered outcomes, validation results and limitations, and measured validation time where available. Note unnecessary overlap or supporting complexity for follow-up; do not infer effort from test counts.

10. **When every task's Status becomes Complete**, automatically invoke `system_consistency_review {project-name} {feature-name}` before telling the user the feature is done, and record the outcome in this document's `Verified` header field. All tasks Complete means **Implemented**, not **Verified**; verification requires a passing consistency review.

**Keep the status current:** whenever this skill creates its document, changes its Status, or the feature becomes Implemented or Verified, finish by running `project_90_status {project-name}` so `STATUS.md` matches before ending your turn.

---

## Output Template

````markdown
# Feature Plan: {Feature Name}

> Status: Draft
> Created: {today's date}
> Project: {project-name}
> Feature ID: {feature-name}
> Depends on: 01_feature_spec.md
> Verified: Not yet

## Test Approach

| Level | Required / Optional / Not Applicable | Reason (if not Required) |
|---|---|---|
| Unit | | |
| Integration | | |
| End-to-End | | |

**Run command:** the command(s) that run this feature's tests in the application repository, and what a pass looks like (for example, exit code 0).

**Test data:** any data that must exist or be generated.

## Tests

| ID | Level | Test | Expected Result | Covers | Task |
|---|---|---|---|---|---|
| UT-1 | Unit | | | AC-1 | 1 |
| IT-1 | Integration | | | AC-1, AC-2 | 3 |
| EC-1 | Edge case | | | AC-3 | 2 |

Add detail beneath the table for any test that needs setup or steps explained.

## Tasks

Layer key — one letter per layer this feature touches, from `02_project_architecture.md`.

Tasks are numbered in build order. A task cannot begin until the tasks it depends on are complete.

| # | Task | Layer | Depends On | Tests Satisfied | Status | Notes |
|---|---|---|---|---|---|---|
| 1 | | | — | UT-1 | Not Started | |
| 2 | | | 1 | EC-1 | Not Started | |

Add detail beneath the table for any task that needs more explanation than fits in a row, referencing the relevant part of the spec.

## Open Questions

List any unresolved questions. Leave blank if none.

---

## Review Checklist

Before starting implementation, confirm:

- [ ] Each of Unit / Integration / End-to-End is marked Required, Optional, or Not Applicable, with a reason for anything not Required
- [ ] Every acceptance criterion in the spec is covered by at least one test
- [ ] Every test has a stable ID, a Covers reference, and a task that satisfies it
- [ ] Every edge case in the spec has a test
- [ ] Every task is an individual coding task, in a workable build order, with a Status of Not Started
- [ ] All design and contract work in the spec has a corresponding task
- [ ] The run command is specified and matches the application repository's tooling

**Next step:** When approved, begin implementation in the application repository. Update each task's Status as you go. When all tasks reach Complete, this skill automatically runs `system_consistency_review` before the feature is reported as done.
````

## Single project backlog

Maintain `<repo_path>/BACKLOG.md` as the project-wide work queue, with Backlog and Delivered sections. Reuse an existing canonical project queue when the project explicitly names another location; never create a competing website-only queue. Preserve item IDs, deduplicate requests, and link every open issue to its detailed issue file. Capture accepted deferrals as open work; move an item to Delivered only with recorded completion/acceptance. Archived history is evidence, not current status. During capture, scope changes and closeout, reconcile affected entries. Keep this detail out of the concise STATUS.md table.

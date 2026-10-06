---
name: feature_01_spec
description: Generate a Feature Spec — the second step in the feature workflow. One document defining what is being built (with stable AC-N acceptance criteria) and how (design by layer, plus data, API, and validation contracts where the feature changes them).
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Steps

1. Read the following for context (required — do not proceed if `{project-root}/features/{feature-name}/` does not exist; tell the user to run `/feature_00_init` first):
   - `{project-root}/01_project_brief.md`
   - `{project-root}/02_project_architecture.md` — the project's actual system layers and settled decisions
   - Any existing `{project-root}/features/{feature-name}/01_feature_spec.md` (in case this is a revision)

2. Generate `{project-root}/features/{feature-name}/01_feature_spec.md` using the template below. It always starts `Status: Draft` with no `> Approved:` line, even when revising an approved spec.

3. Give every acceptance criterion a stable identifier: `AC-1`, `AC-2`, `AC-3`, etc. These identifiers are permanent once `02_feature_plan.md` or an issue references them — **never renumber an existing AC**, even across revisions. If revising a spec that already has downstream artifacts:
   - Editing the wording of an existing AC in place is fine.
   - A genuinely new criterion gets the next unused number (highest existing AC number + 1), even if a lower number was previously retired.
   - A criterion that no longer applies is marked `Retired` in place, not deleted and not renumbered — references to it stay resolvable.

4. Write the Design section to the size the feature needs, no larger:
   - Use this project's real layer names from `02_project_architecture.md`, and cover only the layers the feature touches.
   - Do not restate project-level decisions; link to the architecture and describe only what this feature adds or changes.
   - Include the contract subsections (Data Model and Schema, API Endpoints, Validation Rules) only when the feature introduces or changes a contract. Otherwise write "No contract changes." under Contracts.

5. After writing the file, present the Review Checklist to the user.

6. **Review gate:** if the user confirms the checklist is satisfied, update `Status: Draft` to `Status: Approved` in the document header, and add `> Approved: {today's date}` on the line beneath it, before ending your turn. Until this document is Approved, `feature_02_plan` will refuse to proceed.

**Keep the status current:** whenever this skill creates its document or changes a Status, finish by running `project_90_status {project-name}` so `STATUS.md` matches before ending your turn.

---

## Output Template

````markdown
# Feature Spec: {Feature Name}

> Status: Draft
> Created: {today's date}
> Project: {project-name}
> Feature ID: {feature-name}

## Summary

One sentence describing what this feature does and why it matters.

## Problem Statement

What user need or system gap does this feature address? Why does it need to exist?

## User Stories

- As a [user type], I want to [action] so that [outcome].

## Acceptance Criteria

Each criterion gets a permanent identifier. Do not renumber once assigned.

- **AC-1:** ...
- **AC-2:** ...

## Scope

### In Scope
- What this feature includes

### Out of Scope
- What this feature explicitly does not include

## System Layers Involved

| Layer | Involved? | Notes |
|---|---|---|
| {layer name} | Yes / No | Brief note on what happens in this layer |

## Design

### Approach

Two to four sentences describing the overall technical approach.

### {Layer Name}

Which components are involved, how data moves through this layer, what it persists, and any non-trivial logic. Be specific — this drives the tasks. (Repeat per layer the feature touches.)

### Contracts

Include each subsection only when this feature introduces or changes it. Otherwise write "No contract changes."

**Data Model and Schema** — each model or table with its fields, types, required flags, and indexes.

**API Endpoints** — for each: method and path, what it does, whether authentication is required, request shape, success response, and error responses.

**Validation Rules**

| Field | Rule |
|---|---|

### Edge Cases

| Edge Case | Expected Behaviour |
|---|---|

## Dependencies

Other features or system components this feature depends on.

## Open Questions

List any unresolved questions. Leave blank if none.

---

## Review Checklist

Before running the next skill, confirm:

- [ ] The problem statement accurately reflects the intent
- [ ] Every acceptance criterion has a stable AC-N identifier and is specific and testable
- [ ] Scope boundaries are clear
- [ ] Every layer this feature touches has a complete, feasible design
- [ ] Contracts are fully defined, or explicitly "No contract changes"
- [ ] Edge cases are covered
- [ ] Open questions are noted, and none blocks the plan

**Next step:** When approved, run `/feature_02_plan {project-name} {feature-name}`
````

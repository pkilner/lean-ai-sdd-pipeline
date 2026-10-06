---
name: feature_01_brief
description: Generate a Feature Brief — the second step in the feature workflow. Defines what is being built, why, for whom, and what done looks like, with stable AC-N acceptance criteria identifiers.
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Delivery increment boundaries

A feature may represent an entire proof-of-concept phase spanning multiple capabilities and layers. Reference its row in the project brief's delivery sequence. Define the outcome and acceptance criteria for this increment, the verified behavior inherited from earlier increments, and its dependencies. Do not repeat the whole project brief or treat this increment's omissions as project exclusions.

## Steps

1. Read the following for context (required — do not proceed if `{project-root}/features/{feature-name}/` does not exist; tell the user to run `/feature_00_init` first):
   - `{project-root}/01_project_brief.md` (if it exists)
   - `{project-root}/02_project_architecture.md` (if it exists)
   - Any existing `{project-root}/features/{feature-name}/01_feature_brief.md` (in case this is a revision)

2. Generate `{project-root}/features/{feature-name}/01_feature_brief.md` using the template below. It always starts `Status: Draft` with no `> Approved:` line, even when revising an approved brief.

3. Give every acceptance criterion a stable identifier: `AC-1`, `AC-2`, `AC-3`, etc. These identifiers are permanent once `02_technical_design.md`, `04_test_spec.md`, or any other downstream document references them — **never renumber an existing AC**, even across revisions. If revising a brief that already has downstream artifacts:
   - Editing the wording of an existing AC in place is fine.
   - A genuinely new criterion gets the next unused number (highest existing AC number + 1), even if a lower number was previously retired.
   - A criterion that no longer applies is marked `Retired` in place, not deleted and not renumbered — downstream references to it stay resolvable.

4. After writing the file, present the Review Checklist to the user.

5. **Review gate:** if the user confirms the checklist is satisfied, update `Status: Draft` to `Status: Approved` in the document header, and add `> Approved: {today's date}` on the line beneath it, before ending your turn. Until this document is Approved, `feature_02_technical_design` will refuse to proceed.

---

## Output Template

```markdown
# Feature Brief: {Feature Name}

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
- (add as many as needed)

## Acceptance Criteria

Each criterion gets a permanent identifier. Do not renumber once assigned.

- **AC-1:** ...
- **AC-2:** ...

## Scope

### In Scope
- What this feature includes

### Outside This Increment
- Capabilities delivered by other increments or not assigned yet; distinguish these from explicit project exclusions

### Inherited Behavior
- Earlier feature requirements and acceptance criteria that remain applicable; link their source rather than redefine them

## Applicability

Which parts of the product/system does this feature apply to — e.g. platforms, tiers, user segments, regions, or activity types? Note any variation in behaviour across those. Omit this section if the feature applies uniformly everywhere.

## System Layers Involved

List the layers relevant to this project (use the layer names from `{project-root}/02_project_architecture.md`).

| Layer | Involved? | Notes |
|---|---|---|
| {layer name} | Yes / No | Brief note on what happens in this layer |

## Dependencies

List any other features or system components this feature depends on.

## Open Questions

List any unresolved questions that need answers before or during development. Leave blank if none.

---

## Review Checklist

Before running the next skill, confirm:

- [ ] The problem statement accurately reflects the intent
- [ ] User stories cover the main use cases
- [ ] Every acceptance criterion has a stable AC-N identifier and is specific and testable
- [ ] Scope boundaries are clear
- [ ] Applicability is correct (or explicitly omitted)
- [ ] Open questions are noted

**Next step:** When approved, run `/feature_02_technical_design {project-name} {feature-name}`
```

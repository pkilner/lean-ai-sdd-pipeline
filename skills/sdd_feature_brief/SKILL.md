---
name: sdd_feature_brief
description: Generate a Feature Brief — the first document in the spec-driven development pipeline. Defines what is being built, why, for whom, and what done looks like.
---

The argument passed to this skill is the feature name in kebab-case (e.g. `user-onboarding`). Use it as the feature identifier throughout.

## Steps

1. Read the following for context, if they exist (skip any that don't — do not fail if a project hasn't created them yet):
   - `docs/mindmaps/project-overview.md`
   - `docs/mindmaps/architecture.md`
   - Any strategy or product docs under `docs/product/`
   - Any existing files in `docs/features/{feature-name}/` (check if this feature has been partially started)

2. If `docs/features/{feature-name}/` does not exist, create it.

3. Generate `docs/features/{feature-name}/01_feature_brief.md` using the template below.

4. After writing the file, present the Review Checklist to the user.

---

## Output Template

```markdown
# Feature Brief: {Feature Name}

> Status: Draft
> Created: {today's date}
> Feature ID: {feature-name}

## Summary

One sentence describing what this feature does and why it matters.

## Problem Statement

What user need or system gap does this feature address? Why does it need to exist?

## User Stories

- As a [user type], I want to [action] so that [outcome].
- (add as many as needed)

## Acceptance Criteria

A numbered list of specific, testable conditions that must be true for this feature to be considered complete.

1. ...
2. ...

## Scope

### In Scope
- What this feature includes

### Out of Scope
- What this feature explicitly does not include (prevents scope creep)

## Applicability

Which parts of the product/system does this feature apply to — e.g. platforms, tiers, user segments, regions, or activity types? Note any variation in behaviour across those. Omit this section if the feature applies uniformly everywhere.

## System Layers Involved

List the layers relevant to this project (e.g. Frontend, Backend, Database, Infra — use whatever layer names this project's architecture doc actually uses).

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
- [ ] Acceptance criteria are specific and testable
- [ ] Scope boundaries are clear
- [ ] Applicability is correct (or explicitly omitted)
- [ ] Open questions are noted

**Next step:** When approved, run `/sdd_tech_design {feature-name}`
```

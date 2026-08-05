---
name: feature_brief
description: Generate a Feature Brief — the second step in the feature workflow. Defines what is being built, why, for whom, and what done looks like.
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case.

## Steps

1. Read the following for context (required — do not proceed if `projects/{project-name}/features/{feature-name}/` does not exist; tell the user to run `/feature_init` first):
   - `projects/{project-name}/project-brief.md` (if it exists)
   - `projects/{project-name}/architecture.md` (if it exists)
   - Any existing `projects/{project-name}/features/{feature-name}/feature-brief.md` (in case this is a revision)

2. Generate `projects/{project-name}/features/{feature-name}/feature-brief.md` using the template below.

3. After writing the file, present the Review Checklist to the user.

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

List the layers relevant to this project (use the layer names from `projects/{project-name}/architecture.md`).

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

**Next step:** When approved, run `/feature_technical_design {project-name} {feature-name}`
```

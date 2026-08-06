---
name: project_01_brief
description: Generate a Project Brief — the second step in the project workflow. Defines what the project is, who it is for, and what success looks like, at the whole-project level (not a single feature).
---

The argument passed to this skill is the project name in kebab-case.

## Steps

1. Read `projects/{project-name}/project.yaml` (required — do not proceed if the project has not been initialized; tell the user to run `/project_00_init` first).

2. Read any existing `projects/{project-name}/01_project_brief.md` (in case this is a revision, not a first draft).

3. Generate `projects/{project-name}/01_project_brief.md` using the template below. It always starts `Status: Draft`, even when revising an already-Approved brief — a revision must be re-reviewed before it counts as approved again.

4. Update the `description` field in `project.yaml` to match the one-sentence Summary from the brief.

5. After writing the file, present the Review Checklist to the user.

6. **Review gate:** if the user confirms the checklist is satisfied (in this turn or a follow-up), update `Status: Draft` to `Status: Approved` in the document header before ending your turn. Do not change the status without explicit user confirmation. Until this document is Approved, `project_02_architecture` and `feature_00_init` will refuse to proceed.

---

## Output Template

```markdown
# Project Brief: {Project Name}

> Status: Draft
> Created: {today's date}
> Project: {project-name}

## Summary

One sentence describing what this project is and why it exists.

## Problem Statement

What gap, need, or opportunity does this project address?

## Target Users

Who is this for?

## Success Criteria

What does success look like for this project? Be specific and, where possible, measurable.

## Scope

### In Scope
- What this project includes

### Out of Scope
- What this project explicitly does not include

## Key Constraints

Technical, organizational, timeline, or resource constraints that shape decisions.

## Open Questions

List any unresolved questions. Leave blank if none.

---

## Review Checklist

Before running the next skill, confirm:

- [ ] The problem statement accurately reflects the intent
- [ ] Target users are clearly identified
- [ ] Success criteria are specific and, where possible, measurable
- [ ] Scope boundaries are clear
- [ ] Open questions are noted

**Next step:** When approved, run `/project_02_architecture {project-name}`
```

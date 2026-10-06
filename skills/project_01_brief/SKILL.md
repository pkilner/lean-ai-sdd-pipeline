---
name: project_01_brief
description: Generate a Project Brief — the second step in the project workflow. Defines what the project is, who it is for, and what success looks like, at the whole-project level (not a single feature).
---

The argument passed to this skill is the project name in kebab-case.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Project scope and delivery

Maintain one project brief and one project architecture for the whole intended product. Release order does not define project scope. Separate agreed capabilities, unresolved product candidates, and explicit product exclusions; do not promote a candidate to a commitment or classify an agreed later capability as excluded.

A feature is a bounded, testable delivery increment. It may span several capabilities and system layers, including a proof-of-concept phase. Keep a lightweight delivery sequence in the project brief with feature IDs, outcomes, dependencies, and links to specifications once they exist. Planning rows are not evidence that feature documents exist or that work is approved or implemented.

## Steps

1. Read `{project-root}/project.yaml` (required — do not proceed if the project has not been initialized; tell the user to run `/project_00_init` first).

2. Read any existing `{project-root}/01_project_brief.md` (in case this is a revision, not a first draft).

3. Generate `{project-root}/01_project_brief.md` using the template below. It always starts `Status: Draft`, even when revising an already-Approved brief — a revision must be re-reviewed before it counts as approved again. Remove any `> Approved:` line when the document returns to Draft.

4. Update the `description` field in `project.yaml` to match the one-sentence Summary from the brief.

5. After writing the file, present the Review Checklist to the user.

6. **Review gate:** if the user confirms the checklist is satisfied (in this turn or a follow-up), update `Status: Draft` to `Status: Approved` in the document header, and add `> Approved: {today's date}` on the line beneath it, before ending your turn. Do not change the status without explicit user confirmation. Until this document is Approved, `project_02_architecture` and `feature_00_init` will refuse to proceed.

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

### Included Capabilities
- Agreed capabilities across the whole product, independent of delivery timing

### Open Product Decisions
- Candidate capabilities that need a product decision; not delivery commitments

### Explicit Exclusions
- What the product deliberately does not include; not merely what ships later

## Delivery Sequence

A feature may be a capability or a delivery phase spanning several capabilities. Use stable feature IDs and record only the sequence agreed with the owner; do not invent dates. Link feature documents when they exist.

| Order | Feature ID | Outcome | Dependencies | Specification |
|---|---|---|---|---|

Delivery timing may remain unassigned without removing a capability from project scope.

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

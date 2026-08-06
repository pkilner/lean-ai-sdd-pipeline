---
name: feature_02_technical_design
description: Generate a Technical Design document — the third step in the feature workflow. Defines how the feature is built across the project's system layers, including data flow, algorithms, offline behaviour/sync where applicable, and whether an API spec is required.
---

The arguments passed to this skill are the project name in kebab-case, followed by the feature name in kebab-case.

## Steps

1. Read the following for context:
   - `projects/{project-name}/features/{feature-name}/01_feature_brief.md` (required — do not proceed if missing)
   - `projects/{project-name}/02_project_architecture.md` (if it exists — this defines the project's actual system layers, e.g. Frontend/Backend/Database, or On-Device/Cloud for offline-first mobile apps)

2. **Review gate:** check `01_feature_brief.md`'s Status. If it is `Draft`, stop here, tell the user it needs review and approval first, and do not generate this document. If the user confirms approval in response, update `01_feature_brief.md`'s Status to `Approved`, then continue.

3. Determine whether this feature introduces or changes any contract — new/changed data models, database schema, API endpoints, event schemas, or sync payloads. Set the `API Specification Required` field accordingly; do not leave it implicit or inferred later from prose.

4. Generate `projects/{project-name}/features/{feature-name}/02_technical_design.md` using the template below. Use the *actual* layer names for this project (from `02_project_architecture.md`) instead of the placeholders shown.

5. After writing the file, present the Review Checklist to the user.

6. **Review gate:** if the user confirms the checklist is satisfied, update `Status: Draft` to `Status: Approved` in the document header before ending your turn. Until this document is Approved, `feature_03_api_spec` and `feature_04_test_spec` will refuse to proceed.

---

## Output Template

```markdown
# Technical Design: {Feature Name}

> Status: Draft
> Created: {today's date}
> Project: {project-name}
> Feature ID: {feature-name}
> Depends on: 01_feature_brief.md
> API Specification Required: Yes / No

## Approach Summary

Two to four sentences describing the overall technical approach.

## Design by Layer

For each system layer this feature touches (use this project's real layer names — repeat this subsection per layer):

### {Layer Name}

**Components involved:** Which modules/services in this layer are involved.

**Data flow:** How data moves through this layer for this feature. Use a numbered sequence if helpful.

**Storage:** What gets persisted in this layer, and the shape of that data (fields, types, relationships). Full schema detail, if needed, comes in 03_api_spec.md.

**Key algorithms or logic:** Any non-trivial logic that needs to run in this layer. Be specific — this will drive implementation tasks.

## Offline Behaviour & Sync Strategy

*Include this section only if the project has offline-capable clients or multi-device sync. Delete it otherwise.*

### What works offline
Describe exactly what the user can do and what the system does when there is no connectivity.

### Sync Strategy
When connectivity returns, how does this feature's data sync? Describe:
- What triggers sync
- What gets uploaded/downloaded
- Conflict handling (if applicable)
- Retry behaviour

## Sequence Diagrams

Use Mermaid sequence diagrams to illustrate the key flows.

```mermaid
sequenceDiagram
    participant User
    participant System
    User->>System: (action)
```

Add one diagram per major flow (e.g. happy path, error path, sync path).

## Edge Cases

List edge cases that must be handled. For each, describe the expected system behaviour.

| Edge Case | Expected Behaviour |
|---|---|
| | |

## Open Questions

List any unresolved technical questions. Leave blank if none.

---

## Review Checklist

Before running the next skill, confirm:

- [ ] Every layer this feature touches has a complete, feasible design
- [ ] `API Specification Required` is explicitly set to Yes or No, not left implicit
- [ ] Offline behaviour is explicitly defined if applicable (nothing left as "TBD")
- [ ] Sync strategy is clear if applicable
- [ ] Edge cases are covered
- [ ] No open questions remain (or they are acceptable to carry forward)

**Next step:** When approved, run `/feature_03_api_spec {project-name} {feature-name}` if API Specification Required is Yes, otherwise go straight to `/feature_04_test_spec {project-name} {feature-name}`
```

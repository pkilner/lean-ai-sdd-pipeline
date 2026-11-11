---
name: feature_adr
description: Create a feature-level Architecture Decision Record (ADR). Can be triggered at any point in the feature workflow to capture a significant decision, the options considered, and the rationale. Pass the project name, feature name, and a short decision title as arguments.
---

## Documentation style

Before writing or updating documentation, apply `asd-ste100` through the
[shared SDD documentation rule](../../docs/working-agreement.md#sdd-documentation-style--owner-direction-2026-11-08).
Read the installed skill once per session, as that rule directs. Preserve this
workflow's templates, requirements, approvals and evidence.

The arguments passed to this skill are the project name in kebab-case, the feature name in kebab-case, and a short decision title (e.g. `my-app user-onboarding sync-conflict-strategy`).

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Steps

1. Read `{project-root}/features/{feature-name}/` (required — do not proceed if the feature has not been initialized).

2. Determine the output path: `{project-root}/features/{feature-name}/adr/`. Create it if it does not exist (it is normally created by `feature_00_init`).

3. List existing ADR files in that directory, parse the `NNN` from each filename, and take the largest number found. The new ADR's number is that largest number plus one (not the file count). If no ADR files exist, start at `001`.

4. Generate the ADR file as `adr_{NNN}_{decision-slug}.md` using the template below. Its Status is always `Proposed` — an ADR is not `Accepted` until a human has actually reviewed it.

5. After writing the file, confirm to the user where it was saved and that it is `Proposed`, not yet binding. If the user confirms acceptance in response, update the Status to `Accepted` and add a Status History row — do not do this automatically.

**Keep the status current:** whenever this skill creates its document or changes a Status, finish by running `project_90_status {project-name}` so `STATUS.md` matches before ending your turn.

---

## Output Template

```markdown
# ADR {NNN}: {Decision Title}

> Status: Proposed
> Date: {today's date}
> Project: {project-name}
> Feature: {feature-name}

## Context

Describe the situation that required a decision to be made. What problem were you solving? What constraints existed?

## Decision

State the decision clearly in one or two sentences.

## Options Considered

### Option 1: {Name}
- Description
- Pros
- Cons

### Option 2: {Name}
- Description
- Pros
- Cons

### Option N: {Name}
- Description
- Pros
- Cons

## Rationale

Why was this option chosen over the others? Reference the constraints, priorities, and tradeoffs that drove the decision.

## Consequences

What does this decision mean going forward?

- What becomes easier?
- What becomes harder?
- What future decisions does this constrain?

## Status History

| Date | Status | Note |
|---|---|---|
| {today's date} | Proposed | Initial proposal |
```

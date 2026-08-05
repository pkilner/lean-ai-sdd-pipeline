---
name: project_adr
description: Create a project-level Architecture Decision Record (ADR). Use for decisions that apply across the whole project rather than to a single feature. Pass the project name and a short decision title as arguments.
---

The arguments passed to this skill are the project name in kebab-case, followed by a short decision title (e.g. `my-app database-choice`).

## Steps

1. Read `projects/{project-name}/project.yaml` (required — do not proceed if the project has not been initialized).

2. Determine the output path: `projects/{project-name}/adr/`. Create it if it does not exist (it is normally created by `project_init`).

3. Count existing ADR files in that directory to determine the next number (e.g. if `adr_001_*.md` exists, next is `adr_002`).

4. Generate the ADR file as `adr_{NNN}_{decision-slug}.md` using the template below.

5. After writing the file, confirm to the user where it was saved. If the decision affects the project's system layers or locked decisions, remind the user to reflect it in `architecture.md` via `/project_architecture {project-name}`.

---

## Output Template

```markdown
# ADR {NNN}: {Decision Title}

> Status: Accepted
> Date: {today's date}
> Project: {project-name}

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
| {today's date} | Accepted | Initial decision |
```

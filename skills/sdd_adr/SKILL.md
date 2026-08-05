---
name: sdd_adr
description: Create an Architecture Decision Record (ADR). Can be triggered at any point in the pipeline to capture a significant decision, the options considered, and the rationale. Pass the feature name and a short decision title as arguments.
---

The argument passed to this skill is the feature name in kebab-case followed by a short decision title (e.g. `user-onboarding sync-conflict-strategy`). If no feature name is provided, the ADR is project-level.

## Steps

1. Parse the arguments:
   - First word = feature name (or `project` if this is a project-level decision)
   - Remaining words = decision title (convert to a readable title)

2. Determine the output path:
   - Feature-level: `docs/features/{feature-name}/adr/`
   - Project-level: `docs/adr/`
   - Create the directory if it does not exist.

3. Count existing ADR files in that directory to determine the next number (e.g. if `adr_001_*.md` exists, next is `adr_002`).

4. Generate the ADR file as `adr_{NNN}_{decision-slug}.md` using the template below.

5. After writing the file, confirm to the user where it was saved.

---

## Output Template

```markdown
# ADR {NNN}: {Decision Title}

> Status: Accepted
> Date: {today's date}
> Feature: {feature-name} (or "Project-level")

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

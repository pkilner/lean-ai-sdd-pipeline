---
name: project_architecture
description: Generate the project Architecture document — the third step in the project workflow. Defines the project's system layers, key technical decisions, and data stores. Every feature_* and system_* skill reads this to know the project's real layer names.
---

The argument passed to this skill is the project name in kebab-case.

## Steps

1. Read `projects/{project-name}/project.yaml` and `projects/{project-name}/project-brief.md` (required — do not proceed if either is missing).

2. If `project.yaml` has a `repo_path` pointing to an existing sibling repository, inspect its top-level structure to infer the project's actual system layers (e.g. `frontend/` + `backend/` → Frontend/Backend; a single package with `src/` → collapse to one layer; a mobile app + serverless backend → On-Device/Cloud). Do not guess beyond what the code and brief support — leave layers marked TBD where genuinely unclear.

3. Generate `projects/{project-name}/architecture.md` using the template below. Use this project's real layer names throughout — every other skill in this pipeline pulls layer names from this file, so accuracy here matters more than completeness.

4. After writing the file, present the Review Checklist to the user.

---

## Output Template

```markdown
# Architecture: {Project Name}

> Status: Draft
> Created: {today's date}
> Project: {project-name}

## System Layers

Common shapes, for reference only — replace with what actually fits this project:
- **Client/server web app:** Frontend, Backend, Database, Infra
- **Offline-first mobile app:** On-Device, Cloud, Sync
- **Data platform:** Ingestion, Storage, Processing, Serving

### {Layer Name}

- What lives here
- Status: not started / in progress / complete

(Repeat per layer)

## Key Technical Decisions

| Decision | Choice |
|---|---|

## Data Stores

| Store | Purpose |
|---|---|

## Locked Architectural Decisions

Decisions that are settled and should not be revisited without a new ADR.

| Decision | Choice |
|---|---|

## Open Decisions

| Decision | Options |
|---|---|

---

## Review Checklist

Before running the next skill, confirm:

- [ ] Layer names match how the application repository is actually organized
- [ ] Every layer has a status
- [ ] Locked decisions are genuinely settled, not aspirational
- [ ] Open decisions are tracked, not silently dropped

**Next step:** Run `/project_adr {project-name} {decision-title}` for any locked decision that deserves a full record, otherwise run `/feature_init {project-name} {feature-name}` to start the first feature.
```

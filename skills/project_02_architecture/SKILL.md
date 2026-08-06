---
name: project_02_architecture
description: Generate the project Architecture document — the third step in the project workflow. Defines the project's system layers (with repository path, responsibility, and status), key technical decisions, and data stores. Every feature_* and system_* skill reads this to know the project's real layer names and where their code lives.
---

The argument passed to this skill is the project name in kebab-case.

## Steps

1. Read `projects/{project-name}/project.yaml` and `projects/{project-name}/01_project_brief.md` (required — do not proceed if either is missing).

2. **Review gate:** check `01_project_brief.md`'s Status field. If it is `Draft`, stop here, tell the user the brief needs review and approval first, and do not generate this document. If the user confirms approval in response, update `01_project_brief.md`'s Status to `Approved`, then continue.

3. Using `repo_path` from `project.yaml`, inspect the application repository's top-level structure to infer the project's actual system layers and where each one lives (e.g. `frontend/` + `backend/` → Frontend/Backend; a single package with `src/` → collapse to one layer; a mobile app + serverless backend → On-Device/Cloud). Do not guess beyond what the code and brief support — leave layers marked TBD where genuinely unclear.

4. Generate `projects/{project-name}/02_project_architecture.md` using the template below. Use this project's real layer names throughout, and give every layer a real repository path — every other skill in this pipeline pulls layer names and paths from this file, so accuracy here matters more than completeness.

5. After writing the file, present the Review Checklist to the user.

6. **Review gate:** if the user confirms the checklist is satisfied, update `Status: Draft` to `Status: Approved` in the document header before ending your turn. Until this document is Approved, `feature_00_init` will refuse to proceed.

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

- **Repository path:** path within the application repo where this layer's code lives (e.g. `src/backend/`) — this is what `system_consistency_review` and `issue_02_investigate` use to locate implementation deterministically
- **Responsibility:** one or two sentences on what this layer is for
- **Status:** not started / in progress / complete

(Repeat per layer)

## Key Technical Decisions

| Decision | Choice |
|---|---|

## Data Stores

| Store | Purpose |
|---|---|

## Locked Architectural Decisions

Decisions that are settled and should not be revisited without a new ADR. Every row here should be backed by an `Accepted` ADR in `adr/` — `system_consistency_review` flags rows that aren't.

| Decision | Choice | ADR |
|---|---|---|

## Open Decisions

| Decision | Options |
|---|---|

---

## Review Checklist

Before running the next skill, confirm:

- [ ] Layer names match how the application repository is actually organized
- [ ] Every layer has a repository path, responsibility, and status
- [ ] Locked decisions are genuinely settled, each backed by an Accepted ADR
- [ ] Open decisions are tracked, not silently dropped

**Next step:** Run `/project_adr {project-name} {decision-title}` for any locked decision that deserves a full record, otherwise run `/feature_00_init {project-name} {feature-name}` to start the first feature.
```

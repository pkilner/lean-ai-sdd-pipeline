---
name: project_02_architecture
description: Generate the project Architecture document — the third step in the project workflow. Defines the project's system layers (with repository path, responsibility, and status), key technical decisions, and data stores. Every feature_* and system_* skill reads this to know the project's real layer names and where their code lives.
---

The argument passed to this skill is the project name in kebab-case.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

## Architecture coverage

Describe the whole agreed product, including capabilities assigned to later delivery increments. Read the project brief's delivery sequence, distinguish target architecture from implemented state, and keep unresolved product candidates conditional. Cover shared domain concepts, component relationships, data flows, security boundaries, and API conventions. Feature technical designs and API specifications refine this architecture rather than replace it.

For a layer not yet built, label its repository path as planned (or TBD if undecided) and its status as not started. Existing layers must use their actual paths. Do not create implementation directories merely to make the architecture appear complete.

## Steps

1. Read `{project-root}/project.yaml` and `{project-root}/01_project_brief.md` (required — do not proceed if either is missing).

2. **Review gate:** check `01_project_brief.md`'s Status field. If it is `Draft`, stop here, tell the user the brief needs review and approval first, and do not generate this document. If the user confirms approval in response, update `01_project_brief.md`'s Status to `Approved`, add `> Approved: {today's date}` on the line beneath it, then continue.

3. Using `repo_path` from the project pointer, inspect the application repository's top-level structure (ignoring `sdd/`, which holds these pipeline artifacts, not application code) to infer the project's actual system layers and where each one lives (e.g. `frontend/` + `backend/` → Frontend/Backend; a single package with `src/` → collapse to one layer; a mobile app + serverless backend → On-Device/Cloud). Do not guess beyond what the code and brief support — leave layers marked TBD where genuinely unclear.

4. Generate `{project-root}/02_project_architecture.md` using the template below. Use this project's real layer names throughout, and give every existing layer its real repository path and label proposed paths for not-started layers as planned — every other skill in this pipeline pulls layer names and paths from this file, so accuracy here matters more than completeness.

5. After writing the file, present the Review Checklist to the user.

6. **Review gate:** if the user confirms the checklist is satisfied, update `Status: Draft` to `Status: Approved` in the document header, and add `> Approved: {today's date}` on the line beneath it, before ending your turn. Until this document is Approved, `feature_00_init` will refuse to proceed.

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

## Whole-System Design

Describe component relationships, shared domain concepts and data flows, identity and security boundaries, and shared API/versioning conventions. Map agreed capabilities to components and delivery increments. Distinguish planned behavior from implemented behavior; leave detailed feature algorithms and contracts to feature documents.

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

- [ ] Existing layer names match the application repository; planned layers are explicitly distinguished
- [ ] Every layer has a repository path, responsibility, and status
- [ ] Locked decisions are genuinely settled, each backed by an Accepted ADR
- [ ] Open decisions are tracked, not silently dropped

**Next step:** Run `/project_adr {project-name} {decision-title}` for any locked decision that deserves a full record, otherwise run `/feature_00_init {project-name} {feature-name}` to start the first feature.
```

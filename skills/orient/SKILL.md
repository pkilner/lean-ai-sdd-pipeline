---
name: orient
description: Onboard to the current state of the project and deliver a briefing covering what exists, what is in progress, and what to do next.
---

Onboard yourself to the current state of this project and give a concise briefing.

## Steps

1. Read `CLAUDE.md` (or equivalent project-instructions file) to learn the project's name, purpose, and conventions.
2. Read all files in `docs/mindmaps/`, if the directory exists:
   - `project-overview.md`
   - `architecture.md`
   - `progress.md`
3. Read any additional files in `docs/` that seem relevant to current progress.
4. Deliver a concise briefing covering:
   - **What this project is** (one sentence)
   - **What has been built** so far
   - **What is currently in progress** (if anything)
   - **Open decisions** that still need to be made
   - **Recommended next action** based on the progress map

If `docs/mindmaps/` doesn't exist yet, say so and suggest running `/update_mindmaps` to bootstrap it from the current state of the repo.

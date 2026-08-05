---
name: update_mindmaps
description: Regenerate all mind maps in docs/mindmaps/ to reflect current project state. Run at the end of any significant work session.
---

Regenerate all mind maps to reflect current project state. If `docs/mindmaps/` does not exist yet, create it and the three files below from scratch.

## Steps

1. Read all files under `docs/` recursively.
2. Run `git log --oneline -20` to see recent commits.
3. Determine this project's primary source file extensions (check `CLAUDE.md` for the documented stack, or infer from what's most common in the repo — e.g. `.ts`/`.tsx` for TypeScript, `.py` for Python, `.swift` for Swift, `.go` for Go), then run something like:
   `find . -not -path '*/.git/*' -not -path '*/node_modules/*' -not -path '*/.build/*' \( -name '*.{ext1}' -o -name '*.{ext2}' \) | head -60`
   to get a sense of what code exists.
4. Update the following mind map files in `docs/mindmaps/`:

### `project-overview.md`
- Refresh if the product/project concept has evolved
- Update the "Last updated" date

### `architecture.md`
- Update the status of each major component/service/package (not started / in progress / complete)
- Add any new components or infrastructure decisions made
- Fill in any previously "TBD" decisions that have been resolved
- Update the "Last updated" date

### `progress.md`
- Move completed items into the Completed section
- Update the In Progress section with what is actively being worked on
- Revise Immediate Next Steps based on current state
- Fill in any Open Decisions that have been resolved
- Add a new row to the Sessions Log with today's date and a summary of what happened
- Update the "Last updated" date

5. Confirm to the user that mind maps have been updated and briefly summarise the changes made.

See `templates/mindmaps/` in this pipeline's repo for starter versions of these three files if bootstrapping a new project.

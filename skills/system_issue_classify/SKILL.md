---
name: system_issue_classify
description: Classify a captured issue into one of the standard categories. Second step in the issue workflow, invoked automatically after issue_capture or run standalone. Not user-facing in normal flow — governance/orchestration skill.
---

The arguments passed to this skill are the project name in kebab-case, followed by the issue ID.

## Steps

1. Read `projects/{project-name}/issues/{issue-id}/issue.md` (required — do not proceed if missing).

2. Based on the Description and (if present) Steps to Reproduce, assign exactly one category:
   - **Defect** — the implementation does something wrong relative to the agreed spec
   - **Documentation drift** — the spec no longer describes what the system actually does, but the system's behaviour is fine
   - **Design defect** — the spec itself describes a flawed approach
   - **Implementation drift** — the code has diverged from the spec without a corresponding spec update (distinct from a Defect: nothing is necessarily "broken", but code and spec disagree)
   - **Test gap** — existing tests do not cover the reported scenario
   - **Change request** — this is new/changed behaviour being requested, not a report of something wrong
   - **Dependency/environment issue** — caused by an external dependency, infrastructure, or environment configuration, not by this project's code or specs

   If genuinely unclear from the description alone, briefly inspect the relevant feature docs and/or application repository before deciding. If still ambiguous, classify as the closest fit and note the ambiguity in `issue.md`.

3. Update `issue.md`: set `Category` to the chosen classification, add a Status History entry.

4. Report the classification and reasoning to the user.

**Next step:** Run `/issue_investigate {project-name} {issue-id}`.

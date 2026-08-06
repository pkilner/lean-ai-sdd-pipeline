---
name: system_issue_classify
description: (Internal — not for direct user invocation.) Classify a captured issue's category. Second step in the issue workflow; invoked automatically by issue_capture. Invoked by Claude or other skills only.
user-invocable: false
---

**This is a system skill.** It is never invoked directly by a user typing a command — it is invoked automatically by `issue_capture`, or by Claude when an issue needs reclassifying (e.g. its description changed materially). Do not present this as something the user should run themselves.

The arguments passed to this skill are the project name in kebab-case, followed by the issue ID.

## Steps

1. Read `projects/{project-name}/issues/{issue-id}/issue.md` (required — do not proceed if missing).

2. Based on the Description and (if present) Steps to Reproduce, assign exactly one **Category**:
   - **Defect** — the implementation does something wrong relative to the agreed spec
   - **Documentation drift** — the spec no longer describes what the system actually does, but the system's behaviour is fine
   - **Design defect** — the spec itself describes a flawed approach
   - **Implementation drift** — the code has diverged from the spec without a corresponding spec update (distinct from a Defect: nothing is necessarily "broken", but code and spec disagree)
   - **Test gap** — existing tests do not cover the reported scenario
   - **Change request** — this is new/changed behaviour being requested, not a report of something wrong
   - **Dependency/environment issue** — caused by an external dependency, infrastructure, or environment configuration, not by this project's code or specs
   - **Pending Investigation** — there isn't enough evidence in the description alone to confidently choose one of the categories above, even after briefly checking the relevant feature docs and/or application repository

   If genuinely unclear from the description alone, briefly inspect the relevant feature docs and/or application repository before deciding. Do not force a best-fit guess when the evidence doesn't support one — set Category to `Pending Investigation` instead. `issue_investigate` determines and records the final category once it has actually investigated. Only use `Pending Investigation` when genuinely warranted; most issues have enough information to classify directly.

3. Update `issue.md`: set `Category` to the chosen value, add a Status History entry.

4. Report the classification and reasoning to whichever skill or user turn invoked this.

**Next step:** `/issue_investigate {project-name} {issue-id}`.

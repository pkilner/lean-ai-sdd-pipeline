---
name: system_issue_classify
description: (Internal — not for direct user invocation.) Classify a captured issue's category and complexity. Second step in the issue workflow; invoked automatically by issue_capture. Invoked by Claude or other skills only.
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

   If genuinely unclear from the description alone, briefly inspect the relevant feature docs and/or application repository before deciding. If still ambiguous, classify as the closest fit and note the ambiguity in `issue.md`.

3. Assign a **Complexity**:
   - **Simple** — the fix is small and self-contained: a one-line code change, a typo or single-field correction in a spec doc, no cross-feature or cross-layer impact, and the root cause is already obvious from the description. Simple issues use the lightweight inline path (`issue_investigate`/`issue_resolve`/`issue_verify` write directly into `issue.md` instead of creating separate files).
   - **Standard** — anything else: root cause isn't obvious yet, spans multiple files/layers/features, or carries real risk if done carelessly.

   When genuinely unsure, default to Standard — the cost of unnecessary rigor on a simple issue is much lower than the cost of under-investigating a real one.

4. Update `issue.md`: set `Category` and `Complexity` to the chosen values, add a Status History entry.

5. Report the classification and reasoning to whichever skill or user turn invoked this.

**Next step:** `/issue_investigate {project-name} {issue-id}`.

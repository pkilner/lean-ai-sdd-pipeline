---
name: project_90_status
description: Update sdd/STATUS.md as a concise table of feature spec status, feature plan status and overall feature status. No journal or audit report. Never modifies other project files.
---

## Documentation style

Before writing or updating documentation, apply `asd-ste100` through the
[shared SDD documentation rule](../../docs/working-agreement.md#sdd-documentation-style--owner-direction-2026-11-08).
Read the installed skill once per session, as that rule directs. Preserve this
workflow's templates, requirements, approvals and evidence.

The argument is the project name in kebab-case.

## Resolve and read

Resolve `projects/<project>.yaml`; `{project-root}` is `<repo_path>/sdd`, not the pipeline checkout. Read the application instructions and the feature specs/plans themselves. STATUS.md is a derived snapshot, never evidence of approval or completion.

## Update the status table

1. Inventory feature directories in order. For each, read the spec and plan Status, explicit approval/readiness, task statuses, Verified field and recorded closeout decision.
2. Write one row per feature with four columns only: Feature, Feature spec, Feature plan, Feature status. Link document status text to the actual document. Missing documents say Not created without a link.
3. Preserve actual document states: Draft, Ready for review, Ready for approval, Approved, or an explicitly recorded closed state. Do not infer readiness or approval from file existence, completed prose or this status refresh.
4. Derive feature state from evidence: Not started, In definition, Spec approved, Plan approved, In progress, Implemented (verification pending), or Closed — verified. Preserve Closed with accepted deferrals when explicitly accepted; never present deferred tasks or criteria as passed. If source states conflict, flag the row briefly and tell the user; do not silently rewrite the source.
5. Replace the table with current values. Do not append entries or historical updates. Report the update briefly.

## Output

```markdown
# Feature Status

| Feature | Feature spec | Feature plan | Feature status |
|---|---|---|---|
| 01 · Example | [Approved](features/01-example/01_feature_spec.md) | [Draft](features/01-example/02_feature_plan.md) | Spec approved |
```

## Boundaries

STATUS.md contains only the title and current feature table. Do not include run logs, evidence narratives, issue/ADR inventories, infrastructure versions, health reports, next-step essays or chronological notes. Evidence belongs in the corresponding feature/issue records; infrastructure facts belong in the infrastructure map. Git retains prior snapshots.

This skill only writes STATUS.md. A refresh does not require a new full-project audit. Required completion/issue-closure consistency reviews remain required by their own workflows; keep their evidence in the corresponding records, not STATUS.md. Use system_next_step or system_workflow_resume separately when orientation is requested or state is ambiguous, without pasting their reports into this file.

Review the result: every existing feature has one row, links resolve, states match their sources, no history was appended, and no approval or verification was invented.

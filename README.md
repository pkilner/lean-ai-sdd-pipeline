# lean-ai-sdd-pipeline

A lean, reusable **spec-driven development pipeline** for Claude Code — a set of skills that walk a feature from idea to fully-specified, test-covered implementation plan through five reviewable documents, plus two skills for keeping session-to-session project context fresh.

Extracted from real use across several feature cycles on a production project, then genericized to drop anything project-specific.

## What's in here

### The pipeline (`sdd_*`)

Run these in order for each feature. Each step writes one document and ends with a human review checkpoint before the next begins — nothing proceeds automatically.

| Skill | Step | Produces | Purpose |
|---|---|---|---|
| `sdd_feature_brief` | 1 | `01_feature_brief.md` | What to build — user stories, acceptance criteria, scope |
| `sdd_tech_design` | 2 | `02_tech_design.md` | How to build it — design per system layer, data flow, offline/sync if applicable |
| `sdd_api_spec` | 3 | `03_api_spec.md` | Exact contracts — data models, DB schema, endpoints, event schemas |
| `sdd_impl_plan` | 4 | `04_impl_plan.md` | Individual coding tasks in dependency order, with test tiers assigned |
| `sdd_test_spec` | 5 | `05_test_spec.md` | Unit, integration, and end-to-end tests; pass/fail criteria |
| `sdd_adr` | any | `adr_NNN_*.md` | Capture a significant architectural decision and its rationale, at any point |
| `sdd_consistency_check` | any | report only | Read-only diagnostic — cross-references field names, formulas, and test coverage across all five docs and the implementation; flags drift |

Documents are written to `docs/features/<feature-name>/` in the target project, numbered so the build order is obvious at a glance.

### Session continuity (`orient`, `update_mindmaps`)

| Skill | Purpose |
|---|---|
| `orient` | Read `CLAUDE.md` + `docs/mindmaps/` and deliver a briefing: what exists, what's in progress, what's next |
| `update_mindmaps` | Regenerate `docs/mindmaps/*.md` from current docs, code, and git history — run at the end of a work session |

These solve a different problem than the `sdd_*` pipeline: instead of specifying one feature, they keep a running, low-token-cost summary of the *whole project* so a new session (or a new person) can get oriented in one read instead of re-deriving state from git history and scattered docs.

## Installing into a project

1. Copy the contents of `skills/` into the target project's `.claude/skills/`:

   ```bash
   cp -r skills/* /path/to/target-project/.claude/skills/
   ```

2. Copy the mind map starters into the target project (skip if it already has `docs/mindmaps/`):

   ```bash
   mkdir -p /path/to/target-project/docs/mindmaps
   cp templates/mindmaps/*.md /path/to/target-project/docs/mindmaps/
   ```

3. Add the block from `CLAUDE.md.template` (in this repo) into the target project's `CLAUDE.md`, adjusting anything project-specific.

4. Fill in the three `docs/mindmaps/*.md` files for the project (or run `/update_mindmaps` once there's enough in the repo for Claude to infer them).

## Design notes / why it's structured this way

- **One document, one review gate, per step.** Each `sdd_*` skill refuses to proceed if its required upstream document is missing — this keeps a feature from drifting ahead of what's actually been agreed.
- **Change propagation is top-down.** `01 → 02 → 03 → 04 → 05 → implementation`. If a change originates in the implementation (e.g. a bug discovery), update upward first, then confirm downward. `sdd_consistency_check` exists specifically to catch when this discipline slips.
- **Layer names are not hardcoded.** The original version of this pipeline assumed an offline-first mobile app (On-Device/Cloud/Sync). This version pulls layer names from the target project's own `docs/mindmaps/architecture.md`, so it fits a plain web app, a data pipeline, or anything else without editing the skills themselves.
- **Test tiers are stack-agnostic.** Tier 1 (unit, no environment) → Tier 2 (integration/UI, needs an emulator/browser/local stack) → Tier 3 (system/performance, needs a real device or staging environment) → Tier 4 (manual/field acceptance). Every implementation task gets a Tier 1 assignment as part of its definition of done; Tiers 2–4 are always batched into a dedicated phase at the end.
- **Documentation is a task, not an afterthought.** Every implementation plan requires an explicit documentation task calling out which types/functions need doc comments — not a vague "add docs" line.

## License

See `LICENSE`.

---
name: project_infrastructure_map
description: Create or update the project's infrastructure map — a document of diagrams showing what exists and what is only planned, the boundaries between components, who can do what, what is reachable from outside, and where the build is heading. Run after each setup guide is completed. Pass the project name.
---

## Documentation style

Before writing or updating documentation, apply `asd-ste100` through the
[shared SDD documentation rule](../../docs/working-agreement.md#sdd-documentation-style--owner-direction-2026-11-08).
Read the installed skill once per session, as that rule directs. Preserve this
workflow's templates, requirements, approvals and evidence.

The argument passed to this skill is the project name in kebab-case.

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

**Guides folder:** `{guides-root}` is `{repo_path}/` followed by the `guides_path` field of `{project-root}/project.yaml`, or `docs/setup-guides` if the field is absent. The map is `{guides-root}/infrastructure-map.md`.

## Rules

- **Show as existing only what a check has confirmed.** The source is each guide's Recorded settings and the Verify output behind them. Something a guide plans but nobody has verified is drawn as planned.
- **The reader is the owner, who may be new to this technology.** Each diagram answers one question. Follow each with two or three plain sentences saying what to take from it.
- **Several small diagrams, not one large one.** A diagram that needs scrolling has failed.
- **If the map and reality disagree, the map is wrong.** Fix it, and say what changed.

## Steps

1. Read `{project-root}/02_project_architecture.md`, `{guides-root}/README.md`, every guide's Recorded settings and open follow-ups, and the existing map if there is one.

2. Write or update the map with the sections below. Keep sections that still apply; change boxes from planned to existing as guides are completed; correct every table.

3. Check the diagrams' syntax with a Mermaid parser if one is available. If none is, say that the diagrams were not parsed. Either way, say that you have not seen them rendered.

4. Tell the owner what changed on the map since the last version.

**Keep the status current:** when this skill creates the map, finish by running `project_90_status {project-name}` so `STATUS.md` matches before ending your turn.

## Sections

1. **Header.** One paragraph: what the map is, the date, which guides are done, and whether anything is running or costing money.

2. **How to read the diagrams.** The legend table below.

3. **The places.** Where everything lives, such as the owner's machine, the code host, and the cloud project, and what connects them. Show a missing connection as a dashed arrow labelled with the guide that will create it.

4. **Inside the environment today.** What has been created, grouped by kind.

5. **Who can do what.** Every identity, human or program, and what it may act on. Follow the diagram with a table of what each **can** and **cannot** do. State whether any long-lived keys exist.

6. **What each enabled service unlocks.** Which later resource each switched-on service or API is for.

7. **What is exposed.** A table: each thing, and who can reach it today. State what will become reachable from outside when the system is deployed, and what will stay private.

8. **Where this is heading.** The shape the current phase ends with, including the path a request takes and the path a deployment takes.

9. **Keeping this map current.** One short paragraph.

Leave out a section that has nothing to show yet, and add it when it does.

## Diagram Conventions

Use Mermaid flowcharts. Start every diagram with these classes:

```text
classDef exists fill:#e6f4ea,stroke:#2f855a,color:#1c4532
classDef planned fill:#f7fafc,stroke:#718096,color:#4a5568,stroke-dasharray: 5 4
classDef warning fill:#fffaf0,stroke:#c05621,color:#7b341e
classDef person fill:#ebf4ff,stroke:#2b6cb0,color:#1a365d
```

| Style | Meaning |
|---|---|
| `exists`, solid border | Exists today, confirmed by a check |
| `planned`, dashed border | Planned. The label names the guide that creates it |
| `warning` | Exists, but needs attention |
| `person` | A human |
| Solid arrow | A connection or permission that works today |
| Dashed arrow | A connection that does not exist yet |

To keep diagrams parsing:

- Put every label in double quotes, and use `<br/>` for a line break.
- Never use a reserved word as a node identifier. `default` and `end` are the usual culprits.
- Avoid the percent sign and currency symbols inside labels; write the words.
- Group with `subgraph id["Title"]`. A boundary, such as "reachable from the internet" or "private network", is a subgraph.
- Label an arrow with what flows or what is permitted: `a -- "push images" --> b`, `a -. "will run as" .-> b`.

**Next step:** none. Run this again after the next setup guide is completed.

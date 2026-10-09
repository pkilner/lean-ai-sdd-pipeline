---
name: project_setup_guide
description: Write a setup guide for something the owner must configure by hand, such as a cloud resource, an account setting, or a repository setting. Produces a numbered guide with fixed sections, adds it to the guides index, and, once the owner has carried it out, checks and records the result. Pass the project name and a short guide title.
---

The arguments passed to this skill are the project name in kebab-case, followed by a short title for the guide (e.g. `my-app "Cloud SQL"`).

**Path resolution:** `{project-root}` is the `sdd/` directory inside this project's application repository. Read the pointer `projects/{project-name}.yaml` in this pipeline repo for `repo_path`; `{project-root}` is `{repo_path}/sdd`. If the pointer does not exist, stop and tell the user to run `/project_00_init` first.

**Guides folder:** `{guides-root}` is `{repo_path}/` followed by the `guides_path` field of `{project-root}/project.yaml`. If the field is absent, use `docs/setup-guides` and add `guides_path: docs/setup-guides` to `project.yaml`.

## Rules

- **The owner does the work.** Never create or change the resource yourself. Run a guide's Verify commands only when the owner asks, and only because they read without changing anything.
- **Write it just before it is needed,** so it matches what has actually been built. Do not write a batch of guides far ahead.
- **Use real values.** Once a name or identifier is known from an earlier guide, write it into later guides so commands can be copied as they are.
- **No secrets in the guide.** A password, key, or token is kept in the owner's password manager or the platform's secret store, and the Recorded settings table says so instead of holding the value. Leave out anything a secret scanner would flag.
- **Be honest about what you cannot see.** You are writing from documentation, not from the owner's screen. Say so when handing the guide over, and name the steps most likely to differ.

## Steps

### Writing the guide

1. Read `{project-root}/02_project_architecture.md`, the feature plan task the guide belongs to, `{guides-root}/README.md` if it exists, and every earlier guide's Recorded settings.

2. Number the guide: the highest existing two-digit prefix in `{guides-root}` plus one, starting at `01`. The file is `{guides-root}/{NN}-{title-slug}.md`.

3. Take the steps from the provider's current official documentation, not from memory alone. Check that every documentation link resolves. Links to a signed-in console cannot be checked; say so.

4. Write the guide using the template below. Every section is required, in this order.

5. For the **Cost** section: if the step creates anything that costs money, give an estimate, state for each figure whether it was checked today and against what or is remembered, and add a step telling the owner to compare the provider's own displayed figure with the estimate before creating the resource, with a threshold at which to stop.

6. Add the guide to the index, `{guides-root}/README.md`, creating the index if it does not exist (see the index template). Its row says `Ready`.

7. Hand over: say what the guide creates, what it costs, any secret it involves and where that secret goes, and where you are least sure of the wording.

### After the owner has carried it out

8. Check the result. Either read the Verify output the owner pastes, or run the Verify commands if asked. Mask account identifiers in anything you show. Compare what exists with what the guide intended, and report every difference, including harmless ones.

9. Look once beyond the guide: note anything the provider created automatically that the guide did not ask for, and say whether it needs attention.

10. Fill in **Recorded settings** with what actually exists, and the date. Change the guide's index row to `Done and verified {date}`.

11. If something should be changed later, add a **Follow-up** section to the guide with its own steps and check, list it under **Open follow-ups** in the index, and record it in the feature plan. It stays open until its check passes.

12. Update the infrastructure map by invoking `project_infrastructure_map {project-name}`.

**Keep the status current:** whenever this skill creates a guide, records a result, or opens or closes a follow-up, finish by running `project_90_status {project-name}` so `STATUS.md` matches before ending your turn.

---

## Guide Template

````markdown
# Guide {N}: {Title}

## Purpose

What this guide creates, in one or two sentences. If it costs money, say so here.

## Where this fits

What the thing is, in plain language, and why the project needs it. Define each new term. Say what will and will not exist when the guide is done.

## Before you start

- The earlier guides this depends on.
- Anything the owner needs to have to hand, such as a password manager.

## Steps

1. **{What this step achieves.}** A link to the exact page, then each field and the value to enter.

2. ...

Links to the provider's own instructions for these steps.

## Settings and why

| Setting | Value | Why |
|---|---|---|

## Cost

What this guide costs, per month if recurring, and what would raise it. State how each figure was obtained. "Nothing" is a complete answer when it is true.

## Verify

Commands that read the result and change nothing. Each has a comment saying what to expect.

```bash
# Expect: ...
```

## Troubleshooting

| Problem | What to do |
|---|---|

## Remove

How to undo the guide, in order, and what stops being charged.

## Recorded settings

| Setting | Value |
|---|---|
| ... | |
| Date completed | |
````

## Index Template

`{guides-root}/README.md`:

```markdown
# Setup Guides

Step-by-step guides for the parts of {Project Name} that are set up by hand. You do the steps; each guide then gives a command that confirms the result.

For a picture of everything set up so far and how it connects, see the [infrastructure map](infrastructure-map.md).

## How to use a guide

1. Read **Purpose** and **Where this fits**.
2. Check **Before you start**.
3. Work through **Steps** in order.
4. Run the commands under **Verify**. They only read your setup.
5. Fill in **Recorded settings**. Never record a password, key, or token.

If a screen does not match a step, stop and ask. The guides are written from the provider's documentation without sight of your screen.

## Values used throughout

| Name | Value | Set in |
|---|---|---|

## The guides

| # | Guide | Creates | Status |
|---|---|---|---|
| 1 | [{Title}](01-{slug}.md) | {what it creates} | Ready |

## Open follow-ups

None.
```

**Next step:** the owner carries out the guide. When they report back, continue from step 8.

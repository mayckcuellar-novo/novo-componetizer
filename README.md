# figma-componetizer (Claude Code plugin)

Conform a Figma frame/screen/modal to the **Novo Design System** — bind DS text styles,
color variables, and swap raw layers for DS component instances, **without changing the
visual design**. Everything happens in Figma via the Figma MCP; no code is touched.

This repo is a **Claude Code plugin marketplace**, so installing is two commands — Claude
Code fetches, installs, and enables the skill for you.

## Install (in Claude Code)

```
/plugin marketplace add <your-github-username>/figma-componetizer
/plugin install figma-componetizer@novo-figma
```

That's it — the `figma-componetizer` skill is now available. Update later with:

```
/plugin marketplace update novo-figma
```

> Replace `<your-github-username>` with the GitHub owner this repo lives under. The
> marketplace is named `novo-figma` (see `.claude-plugin/marketplace.json`), which is why
> the install target is `figma-componetizer@novo-figma`.

## Prerequisites (the plugin can't provide these for you)

The skill orchestrates external tools — have these in place first:

1. **Figma MCP server, installed & authenticated.** The skill runs through `use_figma` /
   `search_design_system` / `get_screenshot`. Installing the Figma MCP also gives you the
   required **`figma-use`** skill automatically.
2. **Access to the Novo Design System library** in your Figma account/org. Every style,
   component, and variable is imported by its published library **key** — keys are global
   and stable across accounts, but only resolve for an account that can see this library.
3. **Edit access** to the target file, with the Novo DS library enabled in it.

## Use it

Give Claude a Figma node URL from any Novo file that uses the DS library, plus intent:

> componetize this: https://www.figma.com/design/<fileKey>/<name>?node-id=123-456

The skill classifies each layer (raw / foreign-DS / current-DS), then binds text styles,
color + radius variables, swaps buttons / inputs / selectors / radios / checkboxes / icons
/ dividers, and preserves layout, copy, and spacing.

## Keeping it current

`skills/figma-componetizer/references/ds-cheatsheet.md` holds the DS keys and gotchas. When
the Novo DS team ships or renames components those keys drift — update the cheat sheet
(find keys with the Figma MCP `search_design_system` tool), bump `version` in
`.claude-plugin/plugin.json`, commit, and users pick it up via `/plugin marketplace update`.

## Repo layout

```
.claude-plugin/marketplace.json        ← makes this repo an installable marketplace
figma-componetizer/                    ← the plugin
  .claude-plugin/plugin.json
  skills/figma-componetizer/           ← the skill itself
    SKILL.md
    references/ds-cheatsheet.md
```

# figma-componetizer (Claude Code plugin)

Conform a Figma frame/screen/modal to the **Novo Design System** — bind DS text styles,
color variables, and swap raw layers for DS component instances, **without changing the
visual design**. Everything happens in Figma via the Figma MCP; no code is touched.

This repo is a **Claude Code plugin marketplace**, so installing is two commands — Claude
Code fetches, installs, and enables the skill for you.

## Install (in Claude Code)

```
/plugin marketplace add mayckcuellar-novo/novo-componetizer
/plugin install figma-componetizer@novo-figma
```

That's it — the `figma-componetizer` skill is now available. Update later with:

```
/plugin marketplace update novo-figma
```

> The marketplace is named `novo-figma` (see `.claude-plugin/marketplace.json`), which is
> why the install target is `figma-componetizer@novo-figma` even though the repo is
> `novo-componetizer`.

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

## Changelog

Versions track `figma-componetizer/.claude-plugin/plugin.json`. Update to the latest with
`/plugin marketplace update novo-figma`.

### 1.1.0 — 2026-07-14

Recipes hardened against real conform passes (Payments + Account-info flows):

- **Structure** — rules for which absolute frames to FIX (floating AppHeader / absolute
  content over a skeleton / absolute centered titles) vs LEAVE (scrims, overlay
  drawers/modals, pinned close-X, DS-component-internal absolutes); dead-scrim removal.
- **Close-X** — place in-flow at the end of the `ModalHeader` title row when a header exists;
  float (corner-pinned) only when there's none. Centered-title de-absolute technique.
- **Nomenclature** — rename junk `Frame <n>` layers to role-based PascalCase; skip
  instance-internal nodes.
- **Fields** — `text-field` / `text-area` / `selector` / `phone-number-field` via the nested
  `Parent Input`; `Input` variant is `Empty`/`Filled` only; character counter needs
  `Bottom info=true`; leading icon via the country slot — hide the **Emoji frame** (not just
  the flag glyph) + the Vertical Divider to fix spacing.
- **Buttons/icons** — foreign/legacy `Button` remap; DS buttons ship with left/right icons
  visible (hide them); corrected `button-text-outlined` key; DS `close` icon key +
  Icon-frame-level swap for masked raw icons.

### 1.0.0 — 2026-07-10

Initial release — DS conformance skill packaged as a Claude Code plugin marketplace
(`SKILL.md` + `references/ds-cheatsheet.md`): classify layers by source, bind text styles /
color + radius variables, swap buttons / inputs / selectors / radios / checkboxes / icons /
dividers while preserving layout, copy, and spacing.

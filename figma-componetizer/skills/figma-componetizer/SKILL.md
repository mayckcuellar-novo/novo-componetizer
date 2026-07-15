---
name: figma-componetizer
description: >-
  Conform a Figma frame/screen/modal to the Novo Design System — purely in Figma, no code. Replaces
  raw/detached values AND legacy/other-DS styles with the CURRENT Novo DS: text styles, color
  variables, and component instances (buttons, text-field/dollar-amount-field/text-area, selector,
  search-bar, checkbox, icons), while preserving the original layout and spacing. Use when given a
  Figma node URL from ANY Novo file that uses the Novo Design System library plus intent like
  "conform to DS", "componentize this", "componetize", "match styles to DS", "apply the design
  system", "make this on-system", or simply "do this one" with a node URL. Pairs with the figma-use
  skill (REQUIRED before any use_figma call) and the novo-design-system skill (deeper token reference).
---

# Figma Componetizer

Take a Figma frame (screen or modal) that consumes the **Novo Design System** library and make every
layer on-system: bind raw text to DS **text styles**, raw colors to DS **color variables**, and swap
raw frames for DS **component instances** — **without changing the visual design, copy, or layout**.
All edits happen in Figma via `use_figma`. Nothing touches code.

This works in **any** file that has the Novo DS library enabled — everything is imported by library
**key**, never by file. Discover the target frame from the URL the user provides.

## Prerequisites

1. **Load the `figma-use` skill first.** `use_figma` is mandatory-gated by it; pass `skillNames: "figma-use"` on every `use_figma` call.
2. The `figma` MCP server must be authenticated. If tools aren't available, run the auth flow.
3. For the full token taxonomy/naming, the `novo-design-system` skill is the deeper reference. The exact keys this skill needs are in [references/ds-cheatsheet.md](references/ds-cheatsheet.md) — read it before editing.

## Freshness check (first componetize in a conversation only)

The DS keys/recipes here drift when the Novo DS changes, so on the **first** componetize request
in a conversation (skip on subsequent ones), do a quick, non-blocking version check:

1. `WebFetch` `https://raw.githubusercontent.com/mayckcuellar-novo/novo-componetizer/main/figma-componetizer/.claude-plugin/plugin.json` and read its `version`.
2. Compare it to **this skill's version: `1.3.1`** (kept in sync with `plugin.json` at every release).
3. If the remote version is **higher**, tell the user once, in one line, then proceed anyway:
   > ℹ️ A newer figma-componetizer is available (v1.2.0 → v`X.Y.Z`). Run `/plugin marketplace update novo-figma` in the Claude Code CLI to update.
4. If the fetch fails, times out, or versions match — say nothing and proceed with the bundled version.

Never block on this, and never try to self-update (Claude can't run `/plugin`); it's only a heads-up so
keys don't silently go stale.

## Core principle — classify by SOURCE, then act

For every node, decide one of three things. **"Already styled" does NOT mean skip** — first check which design system it comes from.

| Source of the node's style/variable/component | Action |
|---|---|
| **Raw** (literal hex, arbitrary font size, raw `Button` frame, unbound fill) | **Apply** the current DS token/component |
| **A different / legacy DS** (e.g. "Novo Design System 2023", "Central Library", "Website 2023", a rebrand-experiment library) | **Remap** to the current Novo DS equivalent |
| **Current Novo DS** (library `Novo Design System`) | **Leave it** — already conformant |

**Detecting a foreign style:**
- *Text style* — resolve `node.textStyleId` via `getStyleByIdAsync`. If its name doesn't match the current convention (`primary/header/*`, `primary/body/*`, `primary/body-s/*`, `primary/caption*`) or its key isn't a current-DS key, it's foreign → remap by reading the node's actual **fontSize + weight** and applying the matching current text style.
- *Color* — if a fill/stroke is bound to a variable/style from another library, remap by the **resolved hex** to the current `typography/*` / `fill/*` / `border/*` variable.
- *Component instance* — if `getMainComponentAsync()` resolves to a component from another library, **swap to the current DS component** of the same role, carrying over label/value/state.
- When a foreign style has **no clean current-DS equivalent**, flag it (see Guardrails) — never force a wrong mapping.

## Hard rules

- **VISUAL PARITY IS THE #1 RULE.** A component swap is valid ONLY if it is a true visual *mirror* of the
  original — same **size, color, justification/alignment, and structure**. If the DS component would change ANY
  of those, or there is no mirror component, **do NOT swap** — keep the element as-is and only apply styles/tokens
  (text style + color variables). Screenshot before and after every swap and compare; if it doesn't match, revert
  the swap and fall back to styles-only. Specific traps seen: using the `Small` button variant when the original is
  `Default` size; a swap that left-justifies content the original centered; black coin/icon buttons turning blue;
  swapping a whole row/component when only its label should be touched.
- **Never invent a token.** Every value resolves to a real DS style/variable/component. If nothing matches, flag it.
- **Never change copy, layout intent, or the visual design.** Bind/swap to the token that matches the *current* size/weight/color — don't redesign. Match by what's there.
- **Preserve placement & sizing** on every swap: same parent, same index, same width behavior (HUG vs FILL vs fixed width).
- **Purely Figma.** Never edit code or the novo-webapps repo.
- **Do input-component and icon swaps automatically when the match is certain; flag only when ambiguous.**

## Page-context protocol (critical)

`figma.currentPage` resets to the first page at the start of every `use_figma` call, and MCP
screenshots can change the active page mid-session. `getNodeByIdAsync` only resolves nodes on a
loaded page. So **at the start of every editing call**, ensure the right page is active:

```js
// locate the frame's page once (read-only), then reuse its id
let host = null;
for (const p of figma.root.children) {
  try { await p.loadAsync(); if (p.findOne(n => n.id === TARGET)) { host = p; break; } } catch (e) {}
}
await figma.setCurrentPageAsync(host);
```

Once you know the page id, start each subsequent call with
`await figma.setCurrentPageAsync(await figma.getNodeByIdAsync(PAGE_ID));`. If `getNodeByIdAsync`
returns null, the page isn't current — re-run the locate loop. Copies of the same screen on other
pages have **different** node ids; confirm you're on the intended frame.

## Workflow (run in phases, screenshot to verify after each)

1. **Locate & classify.** Find the frame, switch to its page, screenshot for reference. Traverse and
   bucket every node: instances (+ their source library), raw `Button` frames (fill/visible/label),
   text nodes (size/weight, textStyle source, fill-bound state), raw fills/strokes.
2. **Text styles.** Bind each raw OR foreign-DS text node by **size + weight** to the current DS
   style (h1/h3/h4/h5, body, body-s, caption, caption-s). Skip current-DS-styled and instance-internal
   (`id` contains `;`) and button-label text. Load `ABC Ginto Normal` Regular + Medium first.
3. **Text colors.** Bind text fills → `typography/default` (#242424) / `typography/grey` (#5e5d5d) /
   `typography/info` (blue #3d44e3). (On many screens these are already bound — only touch unbound/foreign.)
4. **Surfaces & borders.** white → `fill/default`, `#f4f4f4` → `fill/neutral`, `#f9f9f9` → `fill/light`,
   `#e2e1e1` strokes → `border/light`. Skip VECTOR (icon artwork) and instance-internal.
5. **Buttons.** (Buttons merged into ONE set on 2026-07-15 — see the cheatsheet section
   "BUTTONS MERGED INTO ONE SET".) Import the single **`button-text`** set
   (`c631c692daf669a7b9ea64a0d66adf65b3071f91`) and pick the look via the `Variant` property —
   do NOT use the old per-type keys (they're dead). Map raw `Button` frames by **label color & fill**:
   - filled brand → `Variant=filled`; transparent+blue border → `Variant=outlined`; grey/dark border →
     `Variant=outlined-neutral`; red border → `Variant=outlined-destructive`; **blue text, no border →
     `Variant=basic`**; **dark/grey text, no border → `Variant=basic-neutral`**; red text → `basic-destructive`;
     on a dark background → the `*-inverse` variants.
   - `Size` is `Regular` (default) / `Small`; `State` is `Resting`/`Hover`/`Pressed`/`Focus`/`Disabled`.
     `black-4` fill (rgba 0,0,0,0.04) = `State=Disabled`. Small pills (h≈28, 12px label) = `Size=Small`.
   - Preserve label (`Copy#…` text prop), icons (`[L]/[R] Icon Show` + `[L]/[R] Icon` instance-swap),
     parent/index, and width (HUG, FILL, or fixed). Read prop keys dynamically from `inst.componentProperties`.
   - **Decide by the link's ACTUAL displayed text color — resolve bound variables, don't trust a stale literal fill.**
     **Blue (`#3d44e3` / `typography/info`) text → `button-text`.** **Dark/grey (`#242424` / `typography/default`)
     non-underlined text → NO mirror** (`button-text-neutral` would wrongly ADD an underline) → keep the element raw
     and only apply text style + color variable. Many "see all / All X / See More" section links are DARK, not blue —
     do not turn them blue. A swap that changes a dark link to blue is a parity violation; verify color first.
   - **Also catch text *links* (not just frames named `Button`):** blue, interactive-colored text with a trailing
     chevron/arrow → `button-text`. BUT **never bulk-swap by a generic frame name like `Link`** — that name is reused
     for clickable rows, list items, and avatars. Target each see-all link **individually** (short label like
     "View all", "See More", "All X" + a chevron, sitting in a section header), and **exclude** anything whose
     frame contains multiple text fields, an amount/date/status, or an avatar. When unsure, swap one, screenshot,
     then proceed — do not loop a whole class of frames blind.
   - **No-match button → partial conformance.** If a borderless button has no clean DS variant — e.g. dark,
     **non-underlined** text where `button-text-neutral` would wrongly ADD an underline (a "Back" link) — do
     NOT swap the container. Instead apply the text style to its label and swap its icon to the DS icon, and
     flag the container choice for the user.
6. **Inputs & form components** (auto when certain): `text-field`, `dollar-amount-field`, `text-area`,
   `selector`, `search-bar`, `checkbox`. See the per-component patterns in the cheatsheet — key points:
   - Field components wrap a nested **`Parent Input`** instance that holds the real props (`[i] Info Show`,
     icon/prefix shows) — set those via the nested instance, the outer one exposes none.
   - Override nested text: `Title`→label, `Placeholder copy`→placeholder (or `Input copy` for a selector's
     selection variant). Hide unused slots: `(Optional)`, `🇺🇸`, `$`, `%`, `Support text`, `0/0`, and any
     stray trailing icons (e.g. search-bar's `x`/`crown`).
   - **Hide the reserved helper/support ROW** ("Frame 7" holding `Support text`) — it stays visible-but-empty
     and adds ~18px below the field.
   - If an external label already exists (sibling, not in the replaced group), hide the component's `Title` to
     avoid a duplicate. Replace the **whole label+box group**, not just the box. Set `layoutSizingHorizontal`
     to match the original (FILL for full-width, fixed for narrow fields).
7. **Icons — inventory and do ALL of them, not just the sidebar/buttons.** Scan the whole screen for icon glyphs
   (frames named "Icon", small single-color vector groups): section `›` chevrons, carousel `‹ ›` arrows, info `ⓘ`,
   `⋯` menus, trend/diagonal arrows, date-chip `calendar` glyphs, dropdown carets, etc. — they're easy to overlook
   inside cards/widgets. For each: swap to its DS icon component AND bind its vector fill to the matching icon-color
   token (`icon/default`/`grey`/`info`/`success`/`white`). Swap a raw icon → its DS icon component **only when the match is exact**. A raw icon is usually a
   wrapper frame (often auto-layout, named "Text"/"Icon") containing Icon › Mask group › Group › Vector —
   createInstance, insert at the inner icon's index, resize 24, remove the old graphic. **Do NOT swap icons that are
   raw vectors inside a "Mask group" / masked vector group, or whose position is set by a parent mask** — replacing
   them mispositions the instance (the mask/positioning is lost). Flag those and leave raw. **Then fix the color:**
   a freshly imported icon instance keeps the component's default (often dark) — bind its vector fill(s) to the
   icon-color token that matches the original (resting sidebar/nav icons → `color/icon/grey`; dark → `color/icon/default`).
   Sidebar nav icons have canonical matches — see the nav-icon table in the cheatsheet; swap them, don't flag.
   **Inline status/list icons count too** — checkmarks before list items, bullets, validation ticks, the info ⓘ —
   are real DS icons (e.g. `check`); classify and swap them, don't dismiss small icons as artwork.
   **DS components drop mock embellishments** — e.g. `password-field` has a show/hide eye but NO lock prefix.
   Don't recreate the dropped extras; flag the minor deviation.
7c. **Dividers.** Plain gray divider lines (thin frames, black @12% = `color/border/divider`) have **no DS
   component** (only the tokens `color/border/divider` + `dividers/border-width`). To "componetize" them: create a
   local `Divider` component from one line (`figma.createComponentFromNode`), bind its fill to `color/border/divider`,
   then replace the other divider frames with instances (FILL width). If the user only wants them on-token, just bind
   each frame's fill to `color/border/divider`.

7b. **Corner radius → DS variables.** Inventory raw corner radii and bind each to the `dim/radii/N` scale
   (4/8/12/16/20/24/32) via `setBoundVariable` on the four corner fields (uniform → all four; mixed top-corners →
   just the matching ones). Off-scale values (e.g. 2) and full-round `9999` (pills/avatars/coins) have no general
   alias — use the component-specific radius var (chip/coin/avatars/card border-radius) or leave + flag. Zero visual change.

8. **Spacing QA (always, after any size-changing swap).** DS components often differ in height from the raw
   frames they replace. Two recurring fixes:
   - A **fixed-height** card/content/form frame that held shorter raw boxes will squeeze its `paddingBottom`
     when taller components go in — set that frame's `layoutSizingVertical = 'HUG'` so padding stays symmetric.
   - Restore collapsed gaps via the parent's `itemSpacing`. Keep spacing on the **8-grid (8/16/24/32/40)**
     while staying faithful to the original; leave original valid micro-dims (2/4/6/10/12) alone.
9. **Verify.** Screenshot after each phase. For tall scrollable content, use `get_screenshot` with a larger
   `maxDimension` (isolated `node.screenshot()` on deep nodes can render blank — not a real error).
10. **Nomenclature & structure hygiene** (always, on every conform). See the cheatsheet sections
    "Layer nomenclature convention", "Structure: which absolutes to FIX vs LEAVE", and "Close-X placement".
    Key points:
    - **Rename junk auto-names** (`Frame <bignumber>`, `Frame <n>`) to structured role-based **PascalCase**
      (file convention, no spaces): `ModalHeader`, `ModalBody`, `<Purpose>Section`, `OptionRow`, `OptionText`,
      `<Purpose>Field`, `CloseIcon`, `<Name>Icon`, generic → `Container`. Detect with `/^Frame\s*\d{3,}$/i`;
      **skip nodes inside an INSTANCE** (component-internal, can't rename without detaching). Metadata-only,
      zero visual change. Also rename generic `Text`/`Container` holders to their role when clear.
    - **Absolutes**: FIX only real shell errors (floating `AppHeader`/`SideBarTabs` over placeholder slots,
      absolute `NewProtectedApp`, absolute titles). LEAVE legit floats: scrims (only if actually visible —
      remove a scrim fully occluded by an opaque full-bleed modal), overlay drawer/modal panels, pinned
      close-X, and absolutes INSIDE imported DS components (negative-margin/transform techniques).
    - **Close-X**: if the modal has a `ModalHeader` title row, move the X in-flow to the end of that row
      (title `layoutGrow=1`, header `paddingRight` symmetric); float (ABSOLUTE corner-pinned) ONLY when there's
      no header structure to hold it. Rename it `CloseIcon`.

## Recovery & re-entry (when the user has edited/reverted)

- **Re-classify from scratch after any manual user edit, undo, or restore.** Node IDs change and restored
  elements come back RAW (unstyled, unbound). Never reuse cached IDs or assume prior state — re-scan the frame.
- **There is no programmatic undo.** `use_figma` cannot revert prior operations, and removing a node destroys its
  content irrecoverably — only the user's Cmd/Ctrl+Z can restore it. So **prefer replacing a frame's inner content
  over removing the whole frame**, never bulk-remove, and if you damaged something, tell the user to undo (you can't).
- **Recover a wrong swap by cloning a known-good sibling.** If you turned an element into the wrong component and the
  original is gone, `clone()` a correct sibling of the same element (e.g. the dark version on another carousel slide)
  and drop it in — cleaner than trying to recolor/coerce the wrong component.
- **Tokenize-in-place is a valid finished state.** Conformance ≠ everything-is-a-component. Rows, avatars, dark
  links, cards with no DS mirror should end up raw-but-tokenized (DS text styles + color variables applied), and that
  is "done." Verify completeness with a final count (e.g. styled vs total text, unbound fills = 0).

## Incremental discipline

Work in small `use_figma` calls (≈ one phase or one component type each); validate with a screenshot before
moving on. Always `return` the created/mutated node ids. If a script errors, it's atomic (no changes) — read the
error, fix, retry. Don't batch all phases into one script.

## Guardrails — pause and ask the user when

- A component match is **ambiguous** (e.g. a borderless text button that could be brand vs neutral — decide by
  color). Note: **sidebar nav icons are NOT ambiguous** — they have canonical matches (see the nav-icon table /
  the `1459:40759` reference in the cheatsheet); swap them.
- A swap is **big/structural** with no clean DS counterpart (data tables / line-item cells, file-upload zones).
- A value/style has **no current-DS equivalent** (flag it; if you must proceed, note the deviation — e.g. a 12px
  *Light* tag mapped to `caption-regular` is slightly heavier because the DS has no light caption).
- **Never** change text content, accept that copy stays as-is, and surface (don't silently fix) anything that
  looks like a content change you didn't make.

## Discovering keys / new components

**Reference-driven mapping.** When a repeated pattern (sidebar / nav / a component family) is ambiguous, find a
canonical reference frame in a Novo file and read its instance + icon pairs to build the mapping — that's the
source of truth, better than guessing. (The sidebar nav-icon mapping came from the `Navigation` frame
`1459:40759`; re-read it to refresh/extend.)

The known keys are in [references/ds-cheatsheet.md](references/ds-cheatsheet.md). For anything not listed, use
`search_design_system` scoped to the Novo DS `libraryKey` (in the cheatsheet) with exact-ish names
(`text-field`, `selector`, `button-text-neutral`, `piggy-bank`, …). The field family is named with concrete
names (`text-field`, `dollar-amount-field`, `email-field`, `phone-number-field`, `percentage-field`, `text-area`,
`search-bar`) — a generic "input field" query misses them.

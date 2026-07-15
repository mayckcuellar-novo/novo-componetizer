# Novo DS — key cheat-sheet for componetizing

All assets belong to the **Novo Design System** library:
`libraryKey = lk-519357016546c689cf813774a90e5b7f20e361aeab6e21b22ccbc7433a846d9315231ab1ed9cb63084070c27d583052727e24cd9b98c49617b61df95d2444968`

Import by key: `figma.importStyleByKeyAsync(key)`, `figma.importComponentByKeyAsync(key)` (single
component / a specific variant), `figma.importComponentSetByKeyAsync(key)` (variant set — then pick a
child by `name`), `figma.variables.importVariableByKeyAsync(key)`. Keys can drift over time — if an
import fails or a value looks wrong, re-discover with `search_design_system` scoped to the libraryKey.

## Text styles (apply by size + weight)

| size / weight | style name | key |
|---|---|---|
| 32 medium | primary/header/h1 | `acd35c3589a646c01b42a7e9dcbf239693048c4c` |
| 24 medium | primary/header/h3 | `9ce7b1c24cf9be5f2a78f63e34835114ed8400e7` |
| 20 medium | primary/header/h4 | `4a564d05b8777b55f2b21948b159d4bae47fef66` |
| 18 medium | primary/header/h5 | `12d63c0cc62a7d657d0cc0cc10aa6dd78b147e1d` |
| 16 regular | primary/body/body-regular | `f52ed39919f0a634221c0953b12674f4710a7301` |
| 16 medium | primary/body/body-medium | `625eaeecd5092680e7eeb93897e1c372cdb2cb7a` |
| 14 regular | primary/body-s/body-s-regular | `7b5821902259faf2a42d3f804a5f29bf572630b9` |
| 14 medium | primary/body-s/body-s-medium | `27bbececac26eb8e2366456bfdbae1d28afcfac4` |
| 12 regular | primary/caption/caption-regulr | `94bf35835079e47eed5bbd7dbff4049cfc500c11` |
| 12 medium | primary/caption/caption-medium | `1e600fd03644591839f2359108b78bc9b40c8692` |
| 10 regular | primary/caption-s/caption-s-regular | `b25a934bca17337c7fad470149f3c48ad5d28222` |

Text styles set font/size/line-height/weight only — **not color**. Bind color separately (below).
12px *Light* has no DS style → use `caption-regular` and note the weight shift. Load
`ABC Ginto Normal` Regular + Medium before applying. Use `node.setTextStyleIdAsync(importedStyle.id)`.

## Color variables

Bind via `figma.variables.setBoundVariableForPaint(paint, "color", importedVar)` → reassign the
returned NEW paint to `node.fills`/`node.strokes` (clone the array).

| role | hex | variable | key |
|---|---|---|---|
| text default | #242424 | color/typography/default | `740783addb9a1977e60d2f3cdadd13457c3e493b` |
| text grey | #5e5d5d | color/typography/grey | `2998d915f718b86ed6bb41b3bc4b1b5b0c678a18` |
| text blue/link | #3d44e3 | color/typography/info | `fd93b57ad0c4e9550db2d4f484ca247aff047419` |
| text success (green amounts/Paid) | #006c43 | color/typography/success | `0af7f5af4e048c5dc87d02799ab717df8cba4c1f` |
| text error (Overdue/negative) | #bf0207 | color/typography/error | `6cb69f9367dd7447273ef3b623325237926fcdf4` |
| text warning | — | color/typography/warning | `4758276fc5c532c643fc37da9e697775dab326b5` |
| surface white | #ffffff | color/fill/default | `8e7991f37a7d39d9a2e401f0b5ba5279cc05e1f6` |
| surface grey | #f4f4f4 | color/fill/neutral | `a596560f5246e4e4a78f807ab9fce9c8bb4a984a` |
| surface light | #f9f9f9 | color/fill/light | `211d1e835421743a45d0eb00bb5f8d5119a880ef` |
| surface success-light (subtle green tint, e.g. revenue avatar bg #f0fcf5) | #f0fcf5 | color/fill/success/light | `9bfa2fc2a9a3d9339b5a406bbb6fb414f4ed4654` |
| border | #e2e1e1 | color/border/light | `10f1a70bcc32631a2a63634134037985e27efeed` |
| border (legacy other-DS #e7e8eb → map to border/light) | #e7e8eb | color/border/light | `10f1a70bcc32631a2a63634134037985e27efeed` |
| divider (thin gray line, black @12%) | rgba(0,0,0,.12) | color/border/divider | `b75fb2f3952e610cdece8e02737aaadd24d7fbac` |

**RULE — divider lines are a MANDATORY phase, and the DS HAS divider components.** Every frame pass must
sweep for thin separator lines (h ≤ 2, w ≥ 40, usually FRAMEs named "Divider" with fill black@12%):
- Named "Divider" lines → swap to **`Horizontal Divider`** SET `f625355b403d67b322328ba92c5b04a4e2db12a9`
  (variants `Tickness=1|2 × Color=Black|White` — note the DS's "Tickness" spelling). Pick Tickness by the
  line's height, preserve width (FILL if the old node filled).
- **`Vertical Divider`** SET = `fcd6d7a218403a0e4c34c40d55102a3abc4a61af` for vertical rules.
- Unnamed thin black@12% lines → bind fill to `color/border/divider` (var above) as a minimum.
- Divider fills are NOT caught by the standard color map (black@12% isn't in it) — without this phase they
  end up with no DS treatment at all. Do not use `createComponentFromNode` local dividers anymore; the DS
  component supersedes that old recipe. Skip dividers inside nav/sidebar.
- **Separators take FOUR shapes — check all of them:** (1) nodes named "Divider"; (2) unnamed thin frames
  whose grey paint is a **STROKE, not a fill** (1px "Container" frames with stroke #e2e1e1 are common —
  a fill-only scan misses them); (3) `LINE` nodes; (4) **border-bottom/top strokes** on containers
  (ModalHeader/AppHeader pattern). Shapes 1–3 → `Horizontal`/`Vertical Divider` instances; shape 4 is
  structural (swapping would reflow the layout) → bind the stroke to `color/border/light` or
  `color/border/divider` instead. Skip dashed strokes and chart/axis contexts (gridlines).
| icon grey (resting nav/sidebar icons) | grey | color/icon/grey | `30bbea3852d2a3c17dad386491951b3b9621626d` |
| icon dark | #242424 | color/icon/default | `45eab26e756c3e2f0abeef9e8b706217c702cee9` |

After swapping an icon, bind its VECTOR fill(s) to the matching icon-color token — a freshly imported icon
instance keeps the component default (often dark). Resting sidebar/nav icons should be `color/icon/grey`.
**Respect the ACTIVE/selected nav item:** its icon must match its active label color — `color/icon/info` (blue),
NOT grey. Don't blanket-bind every nav icon to grey; find the active item (blue text / active indicator) and color
its icon blue. (Match the icon token to the label's token: label `typography/info` → icon `icon/info`.)

## Buttons

Choose by fill + label color. Read the instance's prop keys dynamically (suffixes vary per set):
TEXT prop = `Copy#…`; `[L] Icon Show#…` / `[R] Icon Show#…` (boolean); `[L] Icon#…` / `[R] Icon#…`
(INSTANCE_SWAP — value = imported icon component `.id`).

**GOTCHA — always set `[R] Icon Show#…` = false when the original had no right icon.** `button-text-filled`
and `button-text-outlined` default to showing a right-arrow (→). If you only set the left icon + label, the
new instance renders a stray trailing arrow that wasn't in the source. Set BOTH show flags explicitly.
The `button-text-outlined` Default/Resting variant (`733f50c1…`) resolves to blue text + blue border +
transparent fill — an exact mirror for "Earn $40 / Settings / Download to CSV"-style pills (transparent fill,
1px blue stroke, blue label). Confirm the raw button's fill opacity is 0 (outlined) vs solid (→ filled).

**GOTCHA — full-width CTAs collapse to a hug pill.** A freshly created button instance HUGS its label
(~90px). If the original was a full-width CTA (e.g. Save/Continue at 552px spanning the column), the swap
shrinks it to a small centered pill. Capture the old button's width/`layoutSizingHorizontal` BEFORE removing
it, then on the new instance: if the parent is auto-layout set `layoutSizingHorizontal='FILL'`; otherwise
`resize(oldWidth, height)`. Note the button's own wrapper (`Button:margin`) may also hug — resizing the
instance grows it back. The DS filled-button radius is a full pill (r=height/2, e.g. 24 at h48) — this is
correct/on-system even if the source mock used a less-rounded rectangle.

| Figma look | component | key |
|---|---|---|
| filled brand | button-text-filled (SET — pick `Size=Default, State=Resting|Disabled`) | `c631c692daf669a7b9ea64a0d66adf65b3071f91` |
| outlined (blue border), Default/Resting variant | button-text-outlined | `733f50c1142659cab6b3d387aad186e7ad77422d` |
| outlined, Small/Resting variant | button-text-outlined | `78d81f2461084c6dd8eba49a43c39b23699dab14` |
| neutral text (dark, underlined), SET | button-text-neutral | `f639c834e200a7c0e06c6ff997453131f1906624` |
| brand text (blue, borderless), SET | button-text | `590b7fd8d6d91e38e78b9c0a80e6a37e87f5e4e7` |

- `black-4` fill (rgba 0,0,0,0.04) → use the **Disabled** variant.
- Borderless text buttons: **match the label color** — blue → `button-text`, dark/grey → `button-text-neutral`.
- Swap recipe: read old `parent`/index/sizing → `variant.createInstance()` → `parent.insertChild(idx, inst)`
  → set props → set `layoutSizingHorizontal` (HUG, FILL, or FIXED+`resize`) → `old.remove()`.

## Fields / inputs (the field family)

Single COMPONENTS that bundle label + field + prefix + helper. The outer instance exposes **no** props;
the real props live on a nested **`Parent Input`** instance — call `setProperties` on it for
`[i] Info Show#…` (info ⓘ icon by the label) and icon/prefix shows. Visible text is overridden by nested
layer name.

| component | key | notes |
|---|---|---|
| text-field | `7e78a5f0c1ccc8e0d6409ea07d9b453c1173f7b4` | single-line text |
| dollar-amount-field | `cf9a28d70eda8ba9ee115e52c4efe6c07180d2b4` | `$` prefix amount |
| text-area | `f8dd60c194f5ae85e4216c4a6da1f1b49a40f1f5` | multi-line note |
| password-field | `f76d9eaa736774ded54581b89e0b58a4c6636a14` | eye toggle on the right; add a 🔒 prefix via the country-slot recipe below |
| email-field | `7f8b17a0c19f08e5e10c51f4d8288e2977cc9d09` | |
| phone-number-field | `9af36d4e72c34dac1fe70f99d28ba0eb007b9e88` | |
| percentage-field | `c05d0184a02415c8f2beb3cd258933343fac58d2` | |
| search-bar (SET) | `9931d71918495aca8c335ba54582d572cc84d6b5` | variant `Size=Default, Input=Empty, State=Resting`; one `Search` text; hide trailing `x`/`crown` icons |

Override layers: `Title`→label, `Placeholder copy`→placeholder. Hide: `🇺🇸`, `$` (unless amount), `%`,
`0/0` (unless a counter is wanted). Hide the component's `Title` if an external label already exists.
Replace the whole label+box group; set width to match.

**RULE — mirror the ORIGINAL's slots; never hide unconditionally.** Before hiding a slot, check what the
raw input displayed:
- Original shows **"(Optional)"** next to the label → set `Requirement Show#…=true` (the DS
  `input-requirement` nested instance renders it) — do NOT hide the `(Optional)` layer.
- Original has a **helper line under the box** → set `Support Text Show#…=true` + set the `Support text`
  layer's characters (and keep its row visible) — deleting it loses design content.
- Original text is a **typed VALUE (dark #242424-ish), not a grey placeholder** → set `Input=Filled` on
  `Parent Input`, then write the value into the **`Input copy` TEXT layer** (that's the rendered layer in
  Filled state — "Placeholder copy" is not rendered there, and leaving `Input copy` untouched shows its
  literal default text "Input copy"). Same layer holds a selector's selected value
  (`State=Resting-selection`). Putting a value into `Placeholder copy` renders it grey — a visible change.
- Original shows a **trailing caret** (select) → `[R] Icon Show#…=true` + swap the `[R] Icon#…`
  INSTANCE_SWAP prop to `chevron-down`. Original shows a **× clear** affordance →
  `Clear Icon Show#…=true`. Both live on `Parent Input` — without them a select's right-side icons
  silently vanish in the swap.
- **NEVER force `[R] Icon Show#…=false`.** The password-field's eye toggle IS the `[R] Icon`
  (`hide`, shown by default). Only ever flip these right-slot toggles to TRUE to add something;
  toggling to false to "clean up" deletes a native affordance (the eye). Same for `Clear Icon Show`.
  Text-field's default is already false, so leaving it untouched is correct.

**GOTCHA — swapping a select's trailing caret + × collapses their spacing.** When a rich select is
left tokenized (coin/value kept) and only its trailing glyphs are swapped, replacing each raw glyph
with a DS icon at the *old frame's* width often reseats it wrong — a 24px DS `x` overflows its hugging
wrapper and overlaps the caret. Fix: size BOTH trailing icons the same (16px — the DS trailing-icon
convention), let each wrapper hug (`primaryAxisSizingMode='AUTO'`), and give their shared container a
real `itemSpacing` (~8). After any trailing-icon swap, re-measure the caret's right edge vs the ×'s
left edge and confirm a positive gap — don't trust that the swap preserved layout.

**RULE — standalone blue text-links MUST become `button-text` instances.** Any standalone blue
(typography/info) text that acts as an action/link ("See More", "Forgot your password?", "Get Started",
"Pay a bill", "See Payment History"…) is a button — swap it for the `button-text` set
(`590b7fd8d6d91e38e78b9c0a80e6a37e87f5e4e7`), picking Size by font size: 16px → `Size=Default`,
12px → `Size=Small`. A 14px link has no DS variant — normalize to Default (16) and note it.
Do NOT swap: inline links inside sentence text ("go here", "Read our FAQs"), blue data values
($ amounts), pill/badge text, or nav items. Tokenizing these links is NOT enough — they must be instances.

**RULE — obvious inputs MUST become DS field instances.** If an element is clearly an email/password/plain-text
input (label + 52px box + placeholder), swap it to the matching DS field. Visual-parity caution applies to
*which variant/kind* to pick, not to whether to componentize a self-evident input.

**Leading prefix icons (✉/🔒/etc.) ARE supported — via the country slot.** The `Parent Input` has no `[L] Icon`
prop, but the country-flag slot doubles as a leading-icon slot (canonical example: file
`wcXb0kBE05TVvqcvpSLDOB` "Login-screen" node `1:664`):
1. `Parent Input.setProperties({"Country Show#903:30": true})` — reveals the leading slot ("Frame 1" =
   Emoji flag + angle-down caret + Vertical Divider).
2. Hide the `Emoji` frame and the `Vertical Divider` instance.
3. `swapComponent` the nested `angle-down` instance to the desired icon and bind its color
   (usually `icon/grey`). Icon keys: **mail** = `4846e901f9abe84174e5e39e42fd07a2dba3ddde`,
   **lock** = `2b33c5cc8dc27c8c485fe4cbe00f5ec903baadd0`.
Never drop a mock's leading input icon — reproduce it with this recipe.

**GOTCHA — position-based icon lookups must be scoped to the FOREGROUND container.** When finding icons
by coordinates (e.g. "the copy icon near x1392"), run findAll on the drawer/modal node — NEVER on the whole
frame with a loose x/y range. Dashboard backgrounds extend past the 1440px viewport (off-canvas widgets at
x>1440!) and a wide range will bulk-swap unrelated background glyphs — a destructive parity break. If it
happens: identical sibling frames usually hold untouched originals at the same coordinates — clone them back
(same parent, same index, same x/y/size). Also: icon slots inside blue-outlined circle buttons (upload ⊕)
are BLUE (`icon/info`), not grey — match the sibling glyph color, not the default.

**GOTCHA — search swap leaves a stray magnifier.** In raw search rows the 🔍 glyph often lives OUTSIDE the
`Text Input` frame (a sibling `Text`>`Icon` wrapper). Swapping only the `Text Input` to `search-bar` leaves the
old magnifier floating on top of the new one. After every search-bar swap, look for a leftover 24px `Icon`
frame at the old magnifier position and delete it. Also: set the placeholder to the original copy (e.g.
"Search for a recipient" — DS default is just "Search"), and hide the trailing `x`/`crown` icons unless the
original had them.

## Selector (dropdown field)

`selector` SET key `7f713cfcfd40d522700f446233968dc9222a22bf`. Variants by `State`:
`Resting-no selection` (grey placeholder) | `Resting-selection` (dark value) | `Active-no selection` |
`Active-selection`. Only a `State` prop — override text by layer: `Title`→label,
`Placeholder copy`→placeholder (no-selection) or `Input copy`→value (selection). Hide `(Optional)`/flag.
**Hide the helper/support ROW** (parent of the `Support text` layer) — it reserves ~18px below the field.
Set `layoutSizingHorizontal='FILL'` (or fixed width for narrow fields). Bundles label+field+helper (~h96
before hiding the helper row → ~h78 after).

**GOTCHA — the `Active-*` variants BUNDLE an open menu** (`menu-single-select`, ~336px, 7 placeholder
rows of avatar+Label+Value). So an Active selector instance is ~396px tall, not ~78px. Two cases:
(1) the screen's open menu genuinely matches (avatar + label + value rows) → keep it and populate;
(2) the screen's menu is a different pattern (e.g. an "Add Recipient" action + plain name rows) →
PREFER populating the bundled menu rather than hiding it. The `menu-single-select` is a column of
`menu-list-item` instances (usually 7), each with toggles `Avatar show`, `Value show`, `Label 2 Show`,
`[L] Icon show`, `Checkbox show`, `Switch show`, `Divider show` and a `Label` text layer. To match a plain
recipient list: per active row turn Value/Label 2 off and set `Label`; HIDE the `Avatar`/`AA`/`crown`
sub-layers DIRECTLY (the `Avatar show` prop often doesn't apply — visibility override does); set extra rows
`visible=false`; put an action icon (e.g. `user-plus`) on the top row by swapping its left icon layer; then
DELETE the raw menu frames. Only fall back to hiding the whole `menu-single-select` if the row pattern truly
can't be matched. `menu-action` is a big pre-composed menu, not a per-row. `Active-no selection` = open
trigger, `Resting-*` = closed. NOTE: full-frame screenshots of a frame containing a freshly-edited nested
menu instance render STALE — screenshot the selector/menu node directly to verify.

## RULE — check the PARENT for sibling overlay controls (close X, scrim)

A modal/drawer's **close X is often a SIBLING of the given frame, not a child** — it lives in the parent
`Container` positioned over the modal's top-right (and a scrim overlay may be another sibling). If you scope
the pass to the node URL you were handed (a modal *body*), you will silently miss it. So at the START of a
modal/drawer pass: look at `root.parent.children`; conform any sibling `Icon`/`Text`-wrapped close glyph
and treat a full-bleed sibling rect as the scrim (leave/tokenize). Applies to the
plugin too — selecting the modal body misses the sibling X; select the parent container, or the pass must
walk up one level. Verify a modal by screenshotting the PARENT container, not just the given frame.

## RULE — NO absolute positioning anywhere; the whole frame is nested auto-layout

Zero `layoutPositioning='ABSOLUTE'` nodes in a conformed frame (outside the nav instance). At the end of a
pass, audit: `root.findAll(n => n.layoutPositioning==='ABSOLUTE')` must be empty. Common offenders and fixes:
- **"Floating content over a skeleton" mock pattern.** Many pages have an auto-layout skeleton
  (`Body > Main Content > NewProtectedApp` with a `Placeholder for AppHeader` + an empty `Section`) AND the
  real content + `AppHeader` as ABSOLUTE siblings floating on top. Fix: reparent the real `AppHeader` and the
  real content INTO the skeleton's auto-layout `NewProtectedApp` (insertChild 0 = header, 1 = content), set
  each `layoutPositioning='AUTO'` + `layoutSizingHorizontal='FILL'`, then DELETE the empty placeholder/Section.
  The result is ONE `NewProtectedApp` (auto-layout, HUG height) containing `[AppHeader, content]`. There must
  not be a duplicate absolute `NewProtectedApp`. (Verify which branch is live first — trace a known conformed
  node up to see whether the skeleton's Section or the absolute overlay actually holds the content.)
- **AppHeader as a floating overlay** → make it the FIRST in-flow child of the page's vertical auto-layout
  (`HORIZONTAL`, `SPACE_BETWEEN`, `counterAxisAlignItems=CENTER`, FILL width). Never leave it absolute.
- **A plain (`layoutMode=NONE`) content frame** whose children are manually positioned → set it to
  `VERTICAL` auto-layout, HUG height, FILL width so children flow (their old x/y offsets are ignored).
- **Centered header title (Back / title / X)** → one HORIZONTAL row: left item (Back) fixed/HUG left-aligned,
  title in-flow HUG in the MIDDLE, right item (X) fixed/HUG; put equal-FILL spacers OR make both side
  containers FILL so the HUG title lands centered. Never absolutely-position the title.

## RULE — never create a floating (absolute) node; fit it into the auto-layout

When placing a close X (or any element), **put it INTO the relevant auto-layout, not as an
`layoutPositioning='ABSOLUTE'` sibling.** For a modal close X the canonical structure is: make the header
title row (`Header4`) HORIZONTAL + `layoutSizingHorizontal='FILL'` + `counterAxisAlignItems='CENTER'`, give
the title `layoutGrow=1` (it fills and pushes the X right), then `appendChild` the X's 24px wrapper into that
row with `layoutPositioning='AUTO'`, `layoutGrow=0`, `layoutSizingHorizontal='FIXED'`. Restore symmetric
header padding (don't leave an asymmetric padR you added to clear a floating X). Floating frames are only
acceptable when nothing in the layout can host them.

## RULE — preserve margins/spacing; let height hug, don't sacrifice padding

If a container has a **FIXED height** and swapped content grew (fields with counters/support, DS instances
taller than raw), the extra height eats the bottom padding/margin. Wrong. Set the modal body and its content
container to **`primaryAxisSizingMode='AUTO'` (hug)** so the frame grows and the bottom margin (e.g. padB=24)
is preserved. **Height changing is fine; margins and gaps are not** — spacing tokens are load-bearing design.
After field/instance swaps in a fixed-height container, always set it to hug.

## Switch / toggle

The on/off **toggle** is the `Switch` SET `190758e4461804c8b6357f69196eed2e4aab80a7` — variants
`Marked=Off|On × State=Resting|Hover|Focus|Disabled`. Swap a raw toggle to `Marked=On, State=Resting`
(or Off). **Do NOT confuse it with the `switch` ICON** (`69c23bf8…`, the swap/transfer arrows glyph) —
searching "switch" surfaces the icon first; search "toggle" for the real control.

## Character counters on fields

A field showing an "N / M" counter (e.g. `7 / 30`): on the `Parent Input` set `Character Count Show#…=true`
and `Bottom info#…=true`, then set the `0/0` TEXT layer's characters to the count ("7 / 30"). GOTCHA — with
no support text beside it the counter collapses bottom-LEFT; set its bottom row's `primaryAxisAlignItems='MAX'`
(and the counter's `textAlignHorizontal='RIGHT'`) to keep it bottom-right as designs expect.

## Checkbox

`checkbox` SET key `9cf8d9b3d0f076bf17fbbd91f1bd5a43e6840c11`. Variants `Mark` =
`Unmarked` | `Marked` | `Partial`, × `State`. It's the 24×24 box only (label is a separate sibling) —
replace the raw checkbox square (often a 24×24 "Checkbox:margin" wrapper) with the
`Mark=Unmarked, State=Resting` (or `Marked`) variant at the same index.

## Radio

`radio` SET key `74341c8e67a833e85338e86d37551011fe990cb1`. Variants `Mark` =
`Unmarked` | `Marked`, × `State` (Resting/Hover/Pressed/Focus/Disabled). It's the 24×24 circle only
(label is a separate sibling — leave it as styled text). Replace the raw "Radio Button" circle frame
with `Mark=Marked, State=Resting` (selected) or `Mark=Unmarked, State=Resting` at the same index.
`radio-lockup` (`3ef58bd318b1cb0236777a9e69358cb1cbf1ce15`) = circle+label combined; use only when the
label isn't already a separate text node.

## Date fields — NO clean DS component

The DS field family has NO date-field. A raw date input (a `SingleDatePickerInput`/`GenerateFields`
group with a "yyyy-MM-dd" placeholder + calendar icon) has no visual mirror — **do not swap it to
text-field** (that drops the calendar and changes the affordance). Instead tokenize-in-place: bind its
text styles + colors, bind the box corner radius (usually r8 → `dim/radii/8`), and swap only the inner
calendar glyph to the DS `calendar` icon (`f492644a96ad30581d36e6e1c5864dff6bb22d89`, bind `icon/grey`).
The date picker's calendar trigger often surfaces as a tiny (~w20) "Button" inside the field — it's
part of the field, not a standalone button; don't swap it independently.

## Icons (swap only on an exact match)

| name | key |
|---|---|
| plus | `fc6d9d0f8a398714c7bda1daf24cf5fb964a4aae` |
| gift | `1d0b4ca41c11f0c9830f3e8e666eb8316c527ec5` |
| bank | `2c4b7047146ab4770cb4a2b2599daa0b14470252` |
| chevron-right | `7fcb838fc4adbc656e531933239741a877542654` |
| chevron-left | `6c6ab500bc83c17a6d49b52d96ac67e0f5a4710e` |
| x (close) | `78960eb942b3f73bf4d7bfd377d512bcf71f8ce4` |
| check | `311c5bc358375af826b7ab9eabc2aa19a8c8e1e0` |
| chevron-down | `aa1806a467effddc1e500b67874a8ced46cb9174` |
| more-horiz (⋯ menu) | `2150859f2f9dd16ad9a42ee6aa9a459db89cefed` |
| info-circle (ⓘ) | `620ed1045938591a504e75623668127eb3191d19` |
| arrow-bottom-right (↘) | `886bcb175d36d2c9e592e0b57121513dfea702f3` |
| calendar | `f492644a96ad30581d36e6e1c5864dff6bb22d89` |
| settings (gear/cog) | `ba3b77a59b8e20a9587734a799950720bc52e4b7` |
| download (⬇ tray) | `b8eed606ba2ebd2cf778992083cb978854c88738` |

**Cover ALL icons, not just sidebar + button icons.** Standalone/inline glyphs throughout a screen also need
swapping + coloring: section `›` chevrons (`chevron-right`), carousel `‹ ›` arrows (`chevron-left`/`-right`),
info `ⓘ` (`info-circle`), `⋯` menus (`more-horiz`), trend arrows (`arrow-bottom-right` etc.), date-chip glyphs
(`calendar`), dropdown carets (`chevron-down`). Don't stop at the nav.

**Time-duration chips ("30 days", "All Time", etc.):** the icon in a `TimeDurationDropdown` chip is a
**`calendar`** glyph, NOT a chevron — even though it reads like a dropdown. If a chip has only ONE icon
frame, it's the calendar (left of the label). Don't blanket-swap it to `chevron-down`. Filter-bar dropdowns
(`Select Type`/`Select category` etc.) DO use `chevron-down` carets — those live in `TransactionFilters`.

**Icon color variables** (bind the swapped instance's vector fill by the icon's role/color):
`color/icon/default` (#242424) `45eab26e756c3e2f0abeef9e8b706217c702cee9` ·
`color/icon/grey` `30bbea3852d2a3c17dad386491951b3b9621626d` ·
`color/icon/info` (blue) `8ac7e4e882224d9ca9cb08f0f5fe76d4a641e831` ·
`color/icon/interactive/resting` (blue) `e53222e360236db6a50a5494d2eb6d17b6ddd089` ·
`color/icon/success` (green) `f5d38ddfc578650662a771684bd26988c310cc75` ·
`color/icon/white` `dac759f36815f5b2304466e25c6aadb550235bea`.
| piggy-bank | `af0ca9f9d933b3687ef9e210535404d4c32d21b5` |
| inbox | `fa4d7fe5b5421b1f6fdf9ef31b6e5d34c21bccfb` |
| bar-chart | `8895864e54c0a904dbb3ddf7def594184623ff50` |

### Sidebar nav icons — these DO have canonical matches (don't flag them)

The left-nav icons map cleanly to DS icons. Canonical reference: the **Navigation** frame at
`Eu6Gs2I2OJHPFyNYhcqBis` node `1459:40759` (read its label→icon-instance pairs to refresh/extend).

| nav label | DS icon | key |
|---|---|---|
| Dashboard | layout | `11e9745514814ddf55bf773c2d7df41fd0d78f72` |
| Activity | bar-chart | `8895864e54c0a904dbb3ddf7def594184623ff50` |
| Send Money | send | `a99b0639ca8ed226215b57d4d2a839a00da37506` |
| Add Money | novo-dollar | `40f9f87543b922aa6262d77d87f880e110bb5595` |
| Move Money | switch | `69c23bf8b8df13008685d01b3a617f8a916a14b7` |
| Transfer Funds | split | `38e9c50646614c03964042905c391c8acf8a2519` |
| Send Invoice | contract-edit | `6db74b85e35a5395630570954631bf5882efa566` |
| Payments | dollar-stack | `a8fa6805a6640775099cb8051f58fb8e24094abd` |
| Account Info | bank | `2c4b7047146ab4770cb4a2b2599daa0b14470252` |
| Cards | credit-card | `d9328d25d75742a0b7afd215c2fd14a7f2784cb4` |
| Statements | paper-stack | `a86bb945a5bb62dbe7263f457103818fd8278be2` |
| Reserves | wallet | `65724397e72d2463c81a096b9717483d7346610d` |
| Invoices | contract | `db3b9e3203244dd620d30b89c88fd32545f2d521` |
| Apps | grid-plus | `26822cc16f3978e0dc0ef519412d1add6f07f303` |
| Support | message | `148211ffed19e6fa30b0243b8c7f501b0ee27053` |

New-DS icon names live in `iconRegistry.ts` (inbox, gift, search, plus, building, bank, chevron-*, x,
x-circle, info-circle, arrow-right, pencil…). Find any other key with `search_design_system`. Note
`info` → use `info-circle`. For a nav label not in the table above, check the `1459:40759` reference
before flagging — most have a match.

## Corner radius variables

Alias radii scale (FLOAT, scope CORNER_RADIUS): `dim/radii/4`=`abe717d45016d91ce6852b96bf1207aa6130d3cb`,
`/8`=`7f60d6421a2a3d5419ce8262d5964649d1280869`, `/12`=`d4256db002796ee602887cf6263f859f8240eb54`,
`/16`=`6bf84344794bac499f106449cd954276d9fc237a`, `/20`=`f86e25626abd9d727c10e3864cf507a1a142ce42`,
`/24`=`e27b74f89cd6728ad588a7d2da801a6d48076000`, `/32`=`4c176c4abf0b386904f4a6fdec0f9525814e9d97`.

Bind per corner via `node.setBoundVariable('topLeftRadius', radiusVar)` (also `topRightRadius`,
`bottomLeftRadius`, `bottomRightRadius`). For a uniform radius bind all four; for mixed corners (e.g. a list
header rounded only on top) bind just the corners whose value matches a token. Iterate the 4 corner fields and
bind each whose `node[field]` value is in the scale — handles uniform and mixed in one loop. Skip
instance-internal (`;`) and already-bound corners.

**No general token for:** off-scale values like `2` (leave/flag), and **full-round `9999`** (pills/avatars/coins).
For full-round, use the component-specific FLOAT vars instead: `chip/border-radius`=`dafb46d780714f8b81d4f4ed51c8b2c8ba000092`,
`coin/border-radius`=`d20c6e50efc2d0546696b55ace0fedc029c85b2b`, `avatars/border-radius`=`ce96b4eb57cea4925809f6acee7220d6f8f8ea30`,
`card/size/md/border-radius`=`dd940e59d2fba2a6d0e0c2fb971dbd089bb1437d` — or leave 9999 raw (valid full-round). Radius
binding is zero visual change.

## Discovery snippets

```js
// list styles used on a page (to find foreign vs current DS)
const ids = new Set();
for (const t of figma.currentPage.findAllWithCriteria({types:["TEXT"]})) if (t.textStyleId) ids.add(t.textStyleId);
const styles = []; for (const id of ids){ const s = await figma.getStyleByIdAsync(id); if (s) styles.push({name:s.name, key:s.key}); }
return styles;
```

```js
// resolve an instance's source component (foreign-DS check)
const m = await inst.getMainComponentAsync(); // m.name, m.key, m.parent (the SET)
```

## Structure: which absolutes to FIX vs LEAVE (critical — don't over-correct)

The "no absolute positioning" rule targets **broken page-shell scaffolding only**, NOT every
`layoutPositioning==="ABSOLUTE"` node. Auditing by raw absolute count is misleading — a healthy
drawer/modal frame legitimately has 9–18 absolutes.

**FIX (real structural errors — reparent into the auto-layout skeleton):**
- A floating `AppHeader` that is an ABSOLUTE *sibling of `Body`* while `Body/Main Content/NewProtectedApp`
  contains an empty `Placeholder for AppHeader` slot → move header into NewProtectedApp at the placeholder's
  index, `layoutPositioning=AUTO`, `layoutSizingHorizontal=FILL`, keep height FIXED; delete the placeholder.
- A floating `SideBarTabs` (nav) that is an ABSOLUTE sibling of `Body` while `Main Content` has an empty
  `Placeholder for Container` slot → move nav in at that index, AUTO, `layoutSizingHorizontal=FIXED` (keep
  224 width), `layoutSizingVertical=FILL`; delete the placeholder. (This is the one time reparenting the nav
  is correct — the placeholder footprint is identical, so it's a visual no-op. Never restyle/rebuild the nav.)
- An ABSOLUTE second `NewProtectedApp` content layer over a skeleton NewProtectedApp (early Payments pattern).
- Absolute Header3/title text inside a row → center via the row's auto-layout instead.
Always verify with a before/after screenshot — these reparents must be pixel-identical.

**LEAVE (legitimate — never touch):**
- **Scrim/dimmer**: empty `Container` ~1440×902 at 0,0. It MUST float over the dashboard.
- **Overlay panels**: the drawer (~512 wide) or modal (~640 wide) `Container`. Overlays MUST float.
- **Pinned close-X**: a 24×24 `Text`/`NovoLink` icon pinned top-right of a panel.
- **Absolutes INSIDE imported DS components/illustrations** — e.g. `RecommendationsLayout` "Link (negative
  margin)", `IntegrationsWidgetInner` "Container (transform)", `ReferAFriend` illustration pieces,
  `TransactionDetails` inner containers. These use absolute/negative-margin/transform as the component's own
  construction. Editing them breaks the component AND diverges from the DS — forbidden.

**Audit routine:** flag a frame as needing a fix ONLY if it has a floating `AppHeader`/`SideBarTabs` sibling,
an absolute `NewProtectedApp`, or a `Placeholder for …` slot. Top-level scrim+panel absolutes with no
placeholder = already correct, skip. (Verified across all Novo Mocks drawer/modal/dashboard frames: the only
real offenders were the page-shell frames + the PayPal modal 268:18598.)

## Nomenclature & dead-node cleanup (post-conform hygiene)

**Naming:** Novo Mocks names meaningful elements in **PascalCase, no spaces** (`AppDetailsModal`,
`RevenueAppDetails`, `ModalHeader`, `NewProtectedApp`, `SideBarTabs`, `NovoLink`). When conforming, rename
generic auto-names (`Text`, `Container`, `Frame N`) to their semantic role in that same format once the
function is clear — e.g. a top-right close affordance → `CloseIcon` (NOT "Text", "close", or "close icon").
Match the file's existing casing convention; don't invent a new one (no spaces, no kebab, no snake).

**Scrim is only "legit" if actually visible.** A dimmer/scrim `Container` (empty, black fill low opacity,
~viewport size) is a legitimate overlay ONLY when a *centered* dialog floats above a visible dimmed
background. If the modal on top is **full-bleed + opaque** (covers the whole viewport), the scrim is fully
occluded and serves no purpose — remove it. Red flags for a dead scrim: `childCount:0`, no `reactions`
(no click-to-dismiss), and an opaque sibling on top that covers it. Deleting it is zero visual change.

**Pinned close-X floats.** A close X that isn't part of the content flow should be an ABSOLUTE child of the
modal frame, `constraints {horizontal:"MAX", vertical:"MIN"}`, positioned in the corner — NOT held in place
by a dedicated layout column/gutter. Collapse any "3-column split" (empty gutter + content + X-column) into a
single content frame (content centered via `counterAxisAlignItems:"CENTER"`) plus the floating X. Floating is
correct here precisely because it *preserves* frame integrity (opposite of a floating AppHeader, which breaks
it). Verified on 268:18598 AppDetailsModal.

## Dropdowns / selects → DS selector (Resting-selection recipe)

A raw dropdown = label row (`Text` label + `*`/info) above a box (`Text` value + `chevron-down`). These are
FREQUENTLY MISSED because they're plain frames, not instances — always swap them to the DS **selector**
(set key `7f713cfcfd40d522700f446233968dc9222a22bf`).

Anatomy: the selector wraps a nested **`Parent Input`** instance (same input engine as text-field). Top level
exposes only `State` (variant: `Resting-selection` / `Resting-no selection` / `Active-selection` /
`Active-no selection`). Everything else is set on the `Parent Input` child via `setProperties`:
- `Requirement Show#914:0`, `[i] Info Show#903:21`, `Support Text Show#903:22`, `Character Count Show#903:25`,
  `Bottom info#8576:0`, `Clear Icon Show#903:31`, `[$] Prefix Show#903:27`, `[%] Suffix Show#903:26`,
  `Country Show#903:30`, `[R] Icon Show#903:23` (trailing chevron — keep true), `Title Show#1369:0`,
  `Input` (VARIANT `Filled`/`Placeholder`).

Text layers (find by name, `loadFontAsync` then set `.characters`): `Title` (label), `Input copy` (the
selected value, shown when Input=Filled), `Support text`, `(Optional)` (the requirement marker).

For a **closed dropdown showing a value** (most common in mocks): variant `State=Resting-selection`, on
Parent Input set `Bottom info=false` (drops the support/counter row → compact label+box), `Support Text Show`
+ `Character Count Show` + prefix/suffix/country/clear = false, `[R] Icon Show=true`. Set `Title`=label,
`Input copy`=value.

**Required "*" gotcha:** the DS requirement element is a plain TEXT reading `(Optional)` (DS marks *optional*
fields). Mocks usually show a required `*`. To preserve parity: `Requirement Show=true`, then override the
`(Optional)` text to `*` AND copy the original asterisk's `fills` onto it for color parity.

Placement: capture raw node's parent+index, `insertChild(index, selector)`, `layoutSizingHorizontal="FILL"`,
then remove the raw node. Height lands ~78 vs raw ~74 — auto-layout absorbs it. Verified on 293:3115
(Delivery details: Method + Pay/transfer to). This same dropdown pattern recurs across sibling Payments/flow
frames — sweep for raw label+value+chevron-down frames.

## Close-X placement — REFINED (header-in-flow vs float, decided by structure)

The deciding factor for a modal/drawer close X is **whether there's a ModalHeader title row to hold it**:

- **Modal HAS a `ModalHeader` with a title row** (the standard case) → the X belongs **in-flow at the end of
  the title row** (matches DS reference: `ModalHeader › Header5(HORIZONTAL) › [title (grows), X]`). Do NOT
  float it. Recipe (verified across 11 Payments modals): header is VERTICAL with one FILL-width title-row
  child; set title row `layoutMode=HORIZONTAL`, `counterAxisAlignItems=CENTER`, `itemSpacing=8`; set the
  title (first child) `layoutGrow=1` so it pushes the X right; reparent the floating X into the title row,
  `layoutPositioning=AUTO`; rename it `CloseIcon`. **Padding gotcha:** these headers ship asymmetric
  `paddingLeft:24 / paddingRight:40` while the floating X sat at 24px from the edge — set
  `paddingRight=24` so the in-flow X keeps its exact original position (and padding becomes symmetric).
- **Full-bleed / opaque modal with NO header structure** (e.g. PayPal AppDetailsModal 268:18598) → floating
  the X (ABSOLUTE, corner-pinned `constraints {horizontal:MAX, vertical:MIN}`) is correct, because there's no
  in-flow row to hold it and forcing a layout column would split the frame.

Rule of thumb: float ONLY when there's no structure to absorb it; whenever a header/title row exists, put the
X in-flow there. Either way, rename the container `CloseIcon` (PascalCase, per file convention).
Discovery: scan a frame for an ABSOLUTE ≤40px node containing an `x`/close instance whose modal ancestor has
a `ModalHeader` child — those are the ones to move in-flow.

## text-field & text-area component recipes (raw input → DS)

Keys: **text-field** `7e78a5f0c1ccc8e0d6409ea07d9b453c1173f7b4` (standalone component), **text-area**
`f8dd60c194f5ae85e4216c4a6da1f1b49a40f1f5`. Both wrap the same nested **`Parent Input`** as the selector, so
config is identical: set toggles on the Parent Input child (`Title Show#1369:0`, `Requirement Show#914:0`,
`[i] Info Show#903:21`, `Support Text Show#903:22`, `Character Count Show#903:25`, `Bottom info#8576:0`,
`Country Show#903:30`, `[R] Icon Show#903:23`, `[$]/[%]`, `Clear Icon Show#903:31`). Text layers by name:
`Title`, `Placeholder copy`, `Support text`, `(Optional)`, `0/0` (counter).

- **`Input` variant only has `Empty` / `Filled`** (NOT "Placeholder" — that value errors). Use `Empty` to show
  the placeholder; write the placeholder into `Placeholder copy`. Use `Filled` + `Input copy` for a value.
- **Character counter** needs BOTH `Character Count Show=true` AND `Bottom info=true` (Bottom info gates the
  whole bottom Frame 7 row). Set the `0/0` text (e.g. `0/ 250`); right-align via Frame 7
  `primaryAxisAlignItems="MAX"` when support text is hidden.
- **Leading icon inside the input** (e.g. email field with a leading info-circle): `Country Show=true`, then in
  the country slot (`Frame 4 › Frame 1`) hide the `🇺🇸` Emoji + `Vertical Divider`, and `swapComponent` the
  `angle-down` instance to the target icon. **GOTCHA:** there are TWO icon nodes named `info-circle` — the
  hidden Title-Section one (from `[i] Info Show`) and the visible country-slot one. `findAll` returns the
  Title-Section one FIRST; filter by `visible` + parent chain containing `Frame 1`/`Frame 4` (NOT
  `Title Section`) to grab the right one. Color it by binding its vector fill to `color/icon/info`
  (`8ac7e4e882224d9ca9cb08f0f5fe76d4a641e831`). The country-slot icon is locked to 24px (nested instance —
  `resize` won't stick); accept it or rescale.
- **text-area default height** (~96) is shorter than a tall mock box; set `layoutSizingVertical="FIXED"` +
  `resize` to match (e.g. ~142 incl. counter row).
- **Right-aligned label helper** (e.g. "Only visible to you" top-right of the label): the DS field has NO
  top-right helper slot (support text is bottom-left only). To keep mock parity, preserve the original label
  row and set the field's `Title Show=false`, swapping only the input body. Otherwise it moves to bottom-left.

## Layer nomenclature convention (rename junk auto-names on every conform)

Figma auto-names unlabeled frames `Frame <bignumber>` (e.g. `Frame 2087330580`) and `Frame <n>`. As part of
every conform/componentize pass, **rename renamable junk frames to structured, role-based PascalCase names**
(file convention: PascalCase, no spaces). Derive the name from the frame's role/content:

- Modal panel → `<Purpose>Modal` (e.g. `TransferFundsModal`) or `ModalContent`
- Modal title bar → `ModalHeader`; title+close row → `HeaderRow`; title cluster → `Title` / `TitleDropdown`
- Modal body → `ModalBody`; descriptive blurb → `Description`
- A labeled section → `<Purpose>Section` (e.g. `FromAnotherBankSection`)
- Repeated list/option rows → `OptionRow` / `<Item>Row`; text block within → `OptionText` / `TextBlock`
- Labeled field group → `<Purpose>Field`; label row → `FieldLabel`; input row → `InputRow`
- Close affordance → `CloseIcon`; other icon holders → `<Name>Icon`
- Generic box with no clear role → `Container` (NEVER leave it `Frame 12345`)

Duplicate names are fine in Figma (e.g. several `OptionText`). **Only rename renamable nodes** — skip anything
inside an INSTANCE (component-internal, e.g. a locked `Frame 8559`); renaming would require detaching and
break the instance. Detect junk with `/^Frame\s*\d{3,}$/i` and filter out nodes with an INSTANCE ancestor.
Renaming is metadata-only — zero visual change, no screenshot needed.

## Foreign/legacy `Button` remap + button-key correction

- **Correct button-text-outlined SET key: `8bca05e1b891e901fdded9f7dc7a3fcc97680be1`** (the previously-noted
  `733f50c1...` is STALE / not found — don't use it). Filled `c631c692daf669a7b9ea64a0d66adf65b3071f91` still
  valid. When in doubt, read the key off an existing on-system instance:
  `(await inst.getMainComponentAsync()).parent.key`.
- **Legacy `Button` component** (remote, set name just `Button`, variants `Size=Large/…, Type=Primary/Secondary,
  State=…`, text prop `Button Text#…`) is FOREIGN — remap: `Type=Primary`→`button-text-filled`,
  `Type=Secondary`→`button-text-outlined`, `Size=Large`→DS `Size=Default` (48px). Build a fresh DS instance
  (don't swapComponent — props don't carry), set the DS text prop (the one TEXT-type key), match width
  (FILL for full-width), remove the legacy one.
- **GOTCHA — DS button variants ship with left+right icons VISIBLE by default.** A freshly created
  `button-text-*` instance shows `←`/`→` arrows. Hide them: set both `[L] Icon Show#352:0` and
  `[R] Icon Show#352:11` to `false` (match the original, which usually has no icons). Always screenshot after
  a button build — the arrows are easy to miss.
- `Elements/Separator Line` (foreign) → DS **Horizontal Divider** (set `f625355b403d67b322328ba92c5b04a4e2db12a9`,
  variant `Tickness=1, Color=Black`). A header/brand **logo** component (`Primary`, `Logo=Blue-Black`) is not
  foreign junk — leave it.

## De-absolute a centered title in a title bar (back-link | title | close)

A common "header" pattern: a HORIZONTAL row with TWO equal growing halves (left half holds a back-link,
right half holds a right-aligned close-X) and the **title floats ABSOLUTE, centered on top**. The absolute is
a legit centering trick (auto-layout can't center a middle item when the side items differ in width) — but to
de-absolute it while KEEPING it centered: **insert the title in-flow BETWEEN the two growing halves** (at the
index right after the first half), set `layoutPositioning="AUTO"`, `layoutGrow=0`. Because both halves grow
equally, they split the remaining width evenly and the fixed-width title lands dead-center. Detect it as an
ABSOLUTE child of a HORIZONTAL row that has ≥2 siblings with `layoutGrow>0`. Zero visual change — verify with a
screenshot. (Applied to Add-money form title bars 339:22062/22226/21625.)

Note: title bars built as a balanced 3-item row `[half | title | half]` are ALREADY in-flow-centered — leave them.

## Close/X icon → DS `close`, and swapping masked raw icons at the Icon-frame level

- **DS close icon component key: `8f36ed657a636060467874480d13e70ab98719f2`** (name `close`, renders an X).
  There's also a `x` icon component used inside some field/search components. For a modal/screen dismiss ✕, use
  `close`.
- **Raw icons in these mocks are masked graphics**: `Text(24) › Icon(24, NONE) › Mask group › Group › Vector`.
  Do NOT swap the inner Vector (mask/positioning is lost). Instead swap at the **`Icon`-frame level**: create the
  DS icon instance, `appendChild` it to the `Icon` frame, `layoutPositioning="ABSOLUTE"`, `x=0,y=0`,
  `resize(24,24)`, then remove the old `Mask group` sibling(s). Position/size are preserved because the `Icon`
  frame's own placement in its parent is untouched. DS `close` ships dark (icon/default) — matches the typical
  ✕, no recolor needed.
- **Identify the dismiss ✕ specifically** (not back-chevrons, dropdown carets, or amount clear-×): it's the
  rightmost `Icon` frame in the title-bar/modal-header row. Back-chevron sits in the left half; clear-× lives in
  the amount field body. Swap only the close unless asked for all icons.

## FIX — leading-icon recipe: hide the Emoji FRAME, not the flag text

When adding a leading icon via the country slot (`Country Show=true` → `Frame 4 › Frame 1`), hiding only the
`🇺🇸` TEXT node leaves its parent **`Emoji` frame** (~16px wide) in the auto-layout, which adds a phantom gap
and throws off the leading-icon/value spacing. **Hide the `Emoji` frame itself** (`emojiFrame.visible=false`),
found by walking up from the `🇺🇸` text to the ancestor named `Emoji`. Also hide the `Vertical Divider`. Then
the swapped leading icon (mail/info/etc.) sits flush with correct spacing. (Corrected on 561:53, 524:188.)

## ⚠️ BUTTONS MERGED INTO ONE SET (2026-07-15) — supersedes all per-type button keys

The Novo DS collapsed the separate button component sets into a SINGLE set **`button-text`**
(set key **`c631c692daf669a7b9ea64a0d66adf65b3071f91`** — the old filled key, now the whole set;
90 variants). Pick the look via variant properties, not separate keys.

**DEAD keys (removed — do NOT use):** button-text-outlined `8bca05e1…` and `733f50c1…`,
button-text (link) `590b7fd8…`, button-text-neutral `9fdbef86…`, outlined-destructive `1dbeb880…`,
outlined-neutral `9fdbef86…`. Import `button-text` (c631c692…) and set variants instead.

**Variant props on `button-text`:**
- `Variant`: `filled` · `outlined` · `outlined-neutral` · `outlined-inverse` · `outlined-destructive`
  · `basic` · `basic-neutral` · `basic-inverse` · `basic-destructive`
- `Size`: `Regular` (was "Default") · `Small`
- `State`: `Resting` · `Hover` · `Pressed` · `Focus` · `Disabled`
- Also `Copy` (TEXT, the label), `[L] Icon Show#352:0` / `[R] Icon Show#352:11` (BOTH default **true** —
  hide unless the mock shows them), `[L] Icon#352:22` / `[R] Icon#352:33` (INSTANCE_SWAP).

**Old→new mapping:** filled→`Variant=filled`; blue-border outlined→`outlined`; grey/dark-border→
`outlined-neutral`; red-border→`outlined-destructive`; blue borderless text-link→`basic`;
dark/neutral borderless→`basic-neutral`; red text→`basic-destructive`; on dark bg→`*-inverse`.

**Swap recipe (new):**
```js
const set = await figma.importComponentSetByKeyAsync("c631c692daf669a7b9ea64a0d66adf65b3071f91");
const v = set.children.find(c => c.name === "Variant=filled, Size=Regular, State=Resting"); // or match by props
const b = v.createInstance();
b.setProperties({ "Copy#366:0": label, "[L] Icon Show#352:0": false, "[R] Icon Show#352:11": false });
// disabled state → pick State=Disabled variant; small pill → Size=Small
```
`black-4` fill (rgba 0,0,0,.04) = `State=Disabled`; small pill (h≈28/12px label) = `Size=Small`.

**Existing instances auto-migrated** on the library update (old Size=Default → Size=Regular, filled→
Variant=filled), so previously-conformed frames did NOT break — no rework needed. `button-icon`,
`button-coin*` remain separate sets (unaffected).

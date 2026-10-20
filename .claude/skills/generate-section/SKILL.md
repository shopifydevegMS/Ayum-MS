---
name: generate-section
description: The permanent operating standard for converting the AYUM Figma design into production Shopify sections in this Shopify theme — one section at a time. Use for any Figma-to-section implementation, follow-up bugfix, or design-driven change on the homepage, product page, or collection page. Covers Figma MCP inspection, pre-implementation reporting, the existing-first reuse ladder, the custom-class rule, preservation of merchant data, RTL/responsive/accessibility validation, and the final report format.
---

# AYUM — Figma → Shopify Sections Workflow

This skill is the **permanent source of truth** for converting the AYUM Figma design into production-ready Shopify sections in this repository. It is self-contained: it does not depend on any other Markdown file to be understood or followed.


**AYUM project identity (current):**
- Repository/project folder: `ayum` (GitHub: `shopifydevegMS/Ayum-MS`)
- Shopify theme: `Ayum-MS/main`
- Project name: **AYUM**
- Figma file key: `7cOVp6r92Z3heL1aTMP5N3` ("Ayum (Copy) autolayout", single page `0:1`)

| Page | Desktop (1440) | Mobile (375) |
| --- | --- | --- |
| Homepage | `2001:3` — https://www.figma.com/design/7cOVp6r92Z3heL1aTMP5N3/Ayum--Copy-autolayout?node-id=2001-3 | `2001:581` — https://www.figma.com/design/7cOVp6r92Z3heL1aTMP5N3/Ayum--Copy-autolayout?node-id=2001-581 |
| Product page (fruit template) | `2048:1536` — https://www.figma.com/design/7cOVp6r92Z3heL1aTMP5N3/Ayum--Copy-autolayout?node-id=2048-1536 | `2048:2071` — https://www.figma.com/design/7cOVp6r92Z3heL1aTMP5N3/Ayum--Copy-autolayout?node-id=2048-2071 |
| Collection page (**no Auto Layout — see §3.1**) | `1:787` — https://www.figma.com/design/7cOVp6r92Z3heL1aTMP5N3/Ayum--Copy-autolayout?node-id=1-787 | `1:1081` — https://www.figma.com/design/7cOVp6r92Z3heL1aTMP5N3/Ayum--Copy-autolayout?node-id=1-1081 |
| Collection filter sheet (mobile, open state) | — | `1:1287` — https://www.figma.com/design/7cOVp6r92Z3heL1aTMP5N3/Ayum--Copy-autolayout?node-id=1-1287 |

- These are context frames only; implementation still requires the exact desktop/mobile node for the one target section (each frame's top-level children are the numbered sections, e.g. `05 / Hero slider`). Mobile frames are **375** wide, not 390.
- Design content is Arabic (RTL) and prices are shown in KWD (د.ك).

Companion context:

- [CLAUDE.md](../../../CLAUDE.md) — short repository orientation, including the existing-first ladder (mandatory, do not modify from this skill).
- **No project plan file.** The user decided the AYUM Figma file is self-explanatory, so section inventory and node mapping come straight from Figma (see §3). Do not create a plan file unless the user asks.

---

## 0. Scope and hard limits

This skill governs: converting Figma nodes into Liquid sections/blocks/snippets, their `{% schema %}`, their CSS, any required JavaScript, template JSON wiring, localization, RTL, responsive behavior, accessibility, and the reporting around all of it.

**Absolute limits — these are never negotiable:**

- **Never implement the full website in one task.** One section per task. One page at a time.
- **Never inspect, redesign, or implement unrelated sections during a focused single-section task.** If you notice a problem elsewhere, note it in the report; do not fix it.
- Never start the next section after finishing one. Stop and report.
- Never modify Shopify implementation files during an audit/plan/report-only task.
- Never claim something was tested when it was only inspected statically.

---

## 1. Source-of-truth priority

**Preserve this ordering exactly. It overrides convenience, habit, and visual intuition.**

1. Section-specific Figma desktop/mobile nodes are the primary source of truth.
2. Use Figma MCP to read actual structure, spacing, sizes, colors, typography, variables/tokens, and assets whenever possible using:
   - `get_design_context`
   - `get_variable_defs`
   - `get_metadata`
   - `get_screenshot`
3. Read values from the Figma node data.
4. Do not measure or estimate values from an image when the real Figma node data is available.
5. Cropped section screenshots are visual confirmation only.
6. Full-page Figma links are general context only.
7. Full-page screenshots are general context only.
8. Do not re-analyze or implement unrelated sections during a single-section task.
9. If a Figma node and a screenshot disagree, prefer the Figma node values and clearly flag the discrepancy.
10. The latest explicit correction from the user overrides older instructions.
11. Desktop and mobile nodes are both authoritative.
12. Do not assume mobile is a scaled-down desktop.
13. Existing theme conventions come after direct section-specific Figma requirements.
14. Agent assumptions have the lowest priority.

Claude must inspect the **exact** desktop and mobile nodes supplied for each task. A whole-page Figma node must **not** be used as a substitute for exact section nodes when exact nodes are available.

Supplementary ordering for non-Figma conflicts (applies below rule 1 and above rule 14): current explicit user request → this skill → `CLAUDE.md` → verified **active** code path in this repository → verified Shopify OS 2.0 / Liquid platform behavior → general engineering practice → assumptions (state them).

When a conflict cannot be resolved safely: state the conflict, cite the evidence for each side, mark it **Needs verification**, and choose the least destructive path. Do not silently pick a convention.

---

## 2. Project facts (verified from this repository)

Re-verify anything marked *Needs verification* before relying on it.

### 2.1 Platform and theme

- **AYUM repository:** `ayum`. Theme verified from `config/settings_schema.json` → `theme_info`: **Kalles 5.3.0 by The4** (Shopify OS 2.0). Re-check it if the theme is ever updated.
- **No `package.json`, no bundler, no build step, no test suite, no lint config.** A saved Liquid/JSON/asset file **is** the deployable artifact.
- File counts and available sections/blocks/snippets/assets must be read from the current AYUM repository; do not reuse counts from the previous project.
- Deployment/validation tooling is the Shopify CLI only (`shopify theme dev | check | push | pull`), which **cannot be assumed runnable** here.

### 2.2 Rendering architecture

- Entry point `layout/theme.liquid`: resolves RTL, sets `<html>`/`<body>` flags, renders head snippets (`css-variables`, `js-variables`, `scripts`, `social-meta-tags`, `structured-data`), then renders section groups `header-group`, `footer-group`, `system-group`. `layout/password.liquid` covers the password page.
- Sections declare a `{% schema %}` and pull child blocks via `{% content_for "blocks" %}` (dynamic) or `{% content_for "block", type: "...", id: "..." %}` (pinned; the pinned block must be declared `"static": true` with the **same id** in the preset or the slot renders empty).
- **A block's `type` is its filename minus `.liquid`, including any leading `_`.** The underscore is load-bearing.
- Templates are `templates/*.json` (only `gift_card`, `password`, `search.other` remain `.liquid`). Many suffixed variants exist (17 `product.*`, 10 `collection.*`, many `page.*`) and are merchant-selectable — **do not break them**.
- The `hdt-` / `hdt_` prefix is The4's namespace across custom-element tag names, CSS classes, snippets, and schema types. **Preserve exact strings.**

### 2.3 CSS architecture

- Design tokens live in `snippets/css-variables.liquid` inside a `:root { … }` `{% style %}` block, computed from `settings.*`. Families: spacing `--spacing-*`, colors `--color-*` / `--hdt-btn-*`, typography `--f_family_1..5` / `--font-*` / `--text-*` / `--h0-size..h6-size` (+ `-mb` mobile), layout `--container-max-width*`, radii `--radius-*`.
- **`--color-*` tokens are space-separated RGB triplets** (e.g. `34 34 34`), consumed as `rgb(var(--color-x))` or `rgb(var(--color-x) / .5)`. `color: var(--color-foreground)` alone is invalid.
- Global CSS loads **once**, at the end of `css-variables.liquid`: `base.css` + `theme.css` (preload), then `product-card.css`, `collection-products.css`. Do not re-load these anywhere. `base.css` is a Tailwind-preflight + `hdt-` utility library — **do not hand-edit**.
- Per-section CSS is lazy-loaded as the **first line(s)** of the section file: `{{ 'NAME.css' | asset_url | stylesheet_tag }}`. The theme uses external `asset_url` stylesheets exclusively — there are **zero** `{% stylesheet %}` embedded blocks. (This is how the *theme's own* sections load CSS. AYUM changes don't follow it; they go in `dev1.css`, see §2.3a.)
- Instance-scoped CSS goes in an inline `{% style %}` block scoped by `#shopify-section-{{ section.id }}` (or `.hdt-section-spacing-{{ section.id }}` via `{% render 'spacing-vars', box: section %}`).
- AYUM's global custom override file is **`assets/dev1.css`** (created by the user in place of the previous project's `custom-general.css`). It is loaded **once**, globally, as the last stylesheet in `snippets/css-variables.liquid` (after `base.css`, `theme.css`, `product-card.css`, `collection-products.css`, `main.css`). On branch `dev1` it replaces `main.css` for every rule here; `main.css` stays loaded for other branches but gets no CSS on this branch, so its rules win over theme CSS at equal specificity. Do not load it anywhere else. Never create a second global override file. Use a global override file only for cross-page, custom-class-scoped overrides — not for section-local styling.
- Do not create custom-class-gated or per-section stylesheets for AYUM work; instance-specific CSS goes in `dev1.css` under the instance's `cca-` class (§2.3a). Do not reuse a previous-project-specific stylesheet or class name.

### 2.3a `cca-` prefix + `assets/dev1.css` (permanent — user rule)

All AYUM CSS changes are **additive and live in one file**, so the user can see every change in one place and remove them all by deleting `assets/dev1.css`.

- **Every new CSS class Claude adds starts with `cca-`**, e.g. `cca-pagination`, `cca-collection-strip`, `cca-filter-sheet__header` (BEM-style `__element` / `--modifier` after the prefix is fine). New CSS custom properties use `--cca-` (e.g. `--cca-card-gap`). Before using a name, grep the repo to confirm it's unused.
- **All new CSS goes in `assets/dev1.css`.** Don't create per-section stylesheets for AYUM work, and don't edit the theme's own stylesheets (`base.css`, `theme.css`, `product-card.css`, the per-section `*.css` files, etc.).
- **Every rule in `dev1.css` must be anchored on a `cca-` class**, e.g. `.cca-pagination { … }` or `.cca-collection .hdt-card-product { … }`. That way it only affects markup Claude marked, and never restyles native theme output on its own. A bare `.hdt-…` selector without a `cca-` ancestor is not allowed.
- **Restyling an existing theme element:** add a `cca-` class to its markup (or to a wrapper; or use the instance `custom_class` with a `cca-` value) and target it from `dev1.css`. Keep the native `hdt-` classes; only add new ones.
- **Group `dev1.css` by feature** with a comment header per section/component (e.g. `/* ==== cca-pagination — collection page ==== */`), so a single feature's CSS can also be removed on its own.
- **Exception: values that need Liquid** (per-instance settings, `section.id`, colors from a setting) can't go in a static `.css` file. Put those in an inline `{% style %}` block in the section, and emit **only `--cca-` custom properties** there, scoped to `#shopify-section-{{ section.id }}`. `dev1.css` then uses those variables. Spacing via `{% render 'spacing-vars', box: section %}` stays as the theme does it.
- `dev1.css` loads on every page (last in `snippets/css-variables.liquid`), so keep it lean: no duplicated rules, no unused rules left behind after a change.

### 2.4 JavaScript architecture

- Native ES modules via the **importmap** in `snippets/scripts.liquid` — aliases `@theme/global`, `@theme/events`, `@theme/utilities`, `@theme/floating`, `@theme/psw`, `@theme/zoom-psw`, `@theme/dayjs`, `@theme/libs`. Import from aliases, never raw asset paths. A new alias requires editing `scripts.liquid` **and** adding a matching `modulepreload`.
- **Editable JS source is `assets/events.js`, `assets/utilities.js`, `assets/custom.js` only.** All `*.min.js` are generated/vendor bundles with no in-repo source — never hand-edit them. `assets/theme.js` is intentionally empty. To change base-class behavior, subclass in `custom.js`.
- UI is Web Components: `customElements.define('hdt-…', class)`. `global.min.js` registers ~60 `hdt-*` elements; `facets.min.js` registers filtering elements.
- Cross-component communication uses the typed event bus in `assets/events.js` — reference `ThemeEvents` constants (`cart:update`, `variant:selected`, `dialog:open`, `filter:update`, …) and dispatch with the typed classes (`CartUpdateEvent`, `CartAddEvent`, `CartCountEvent`, `QuantitySelectorUpdateEvent`, `CartErrorEvent`). **Never hardcode a raw event string. `CartAddEvent.eventName === ThemeEvents.cartUpdate` — adds and updates share the `cart:update` channel.**
- Runtime config reaches JS as `window.themeHDN`, populated in **two stages**: `themeHDN.settings` in `snippets/js-variables.liquid` (head), `themeHDN.routes` + `themeHDN.extras` in `snippets/system-bottom.liquid` (footer). Code needing routes/extras must run after `system-bottom.liquid`.
- Reuse helpers from `assets/utilities.js` (`$4`/`$$4`/`$id4`, `debounce`, `throttle` — **rAF-based, not time-based**, `formatMoney`, `fetchCache`, `fetchConfig`, `matchMediaQuery`, `createImg`, `renderSrcset`, `requestIdleCallback`). Do not reimplement them.
- Guard editor-only code with `Shopify.designMode` (JS) / `request.design_mode` (Liquid); react to `shopify:section:load/unload/select/deselect/reorder` and `shopify:block:select/deselect`.

**JavaScript precedent rule:** inspect the AYUM repository for the closest existing isolated Web Component before adding JavaScript. Preserve the same guarded `customElements.define`, resilient/idempotent `connectedCallback`, per-instance state, and `disconnectedCallback` cleanup pattern. Do not copy previous-project-specific element names or asset filenames into AYUM.

### 2.5 Breakpoints — the real project values

**Verified from the repository:** the theme has exactly **two layout breakpoints — `768px` (md/tablet) and `1150px` (lg/desktop)** min-width, with `767` / `1149` max-width mirrors. (`750px` appears only in image `sizes` strings, never in layout.) Utility prefixes are mobile-first: unprefixed = mobile, `md:` = ≥768, `lg:` = ≥1150. Column counts come from `col_dk` / `col_tb` / `col_mb` schema selects → `hdt-grid-cols-{col_mb} md:hdt-grid-cols-{col_tb} lg:hdt-grid-cols-{col_dk}`.

Therefore the **project tiers are**:

| Tier | Range | Figma source |
| --- | --- | --- |
| Mobile | below 768px | Figma 375 frames |
| Tablet | 768px to below 1150px | **not designed** — interpolate |
| Desktop | 1150px and above | Figma 1440 frames |

**There is no 1440 breakpoint in this theme.** Do not introduce one, and do not treat "small desktop 1150–1439" as a separate designed tier. A 1440-wide Figma frame maps to the ≥1150 desktop tier; verify the layout is still sane at 1150–1439 (it is the same tier), and say so honestly in the report.

Validate the boundaries: **767 / 768 / 1149 / 1150**, plus a wide check at **1440** and narrow checks at **390**, **375**, and **320** for overflow. Adding a new layout breakpoint desyncs grid, typography tokens, and the `md:`/`lg:` utilities — if a design genuinely requires one, flag it as a documented exception before writing it.

Do not treat tablet as an enlarged mobile or a shrunk desktop automatically. Because 768–1149 is undesigned, infer it safely from: the project architecture, the Figma Auto Layout behavior (hug/fill/wrap), content constraints, the existing `col_tb` defaults, and reasonable interpolation. Prefer fluid sizing. **Never write a layout rule that only works at the exact screenshot width**, and avoid one-off magic widths not backed by Figma or a documented layout requirement.

### 2.6 RTL / LTR

- **This project supports both, and the AYUM design direction/language must be verified from the exact Figma nodes and active storefront configuration before implementation — do not assume RTL or LTR.**
- Storefront locales present: `ar`, `de`, `es`, `fr`, `zh-CN`, `en.default`. Schema locales: `de`, `es`, `fr`, `zh-CN`, `en.default` — **there is no `ar.schema.json`**, so Arabic Theme-Editor labels fall back to English.
- Direction is resolved **once** in `layout/theme.liquid` from `settings.right_to_left` (`enable` / `enable-with-list` against a hardcoded ISO list including `ar`) → sets `dir`, a `dir--rtl` / `dir--ltr` class, and passes `is_rtl` to `css-variables`, which emits `--value-logical-flip`, `--value-origin-start`, `--value-origin-end`.
- **Write logical properties first**: `margin-inline`, `padding-inline`, `inset-inline-start` / `inset-inline-end`, `border-inline-start` / `border-inline-end`, `text-align: start` / `end`. Add an explicit `.dir--rtl` override only where a physical value is unavoidable, and say why.
- Do not hardcode left/right where the layout should mirror. Do not force direction globally from a section. Do not reverse content that should stay LTR (some media, numeric sequences, slider geometry, certain icons) without checking the actual design.
- Legitimate exception pattern (proven in a previous project; no AYUM precedent yet): set `direction: ltr` on a **geometry-only** container (a grid track or an animation track) to get predictable physical placement, then restore `direction: rtl` on each text-bearing child. Document it when you do.
- In JS, read `document.documentElement.getAttribute('dir')` or `isRTL` from `@theme/global` — never re-detect direction.
- **Locale-driven direction rule (permanent — learned from bugfixes in a previous project on this same theme):** this theme resolves `dir` from the *storefront locale*, and `en.default` is the default locale — so the page is LTR for visitors on the English locale even when content is authored in Arabic. **AYUM content is authored in Arabic (confirmed by the user), so this rule applies to every AYUM section.** Therefore, for any section whose content is authored in Arabic: (1) never rely on the page `dir`, a single wrapper rule, or per-leaf selector lists for text direction; declare `direction: rtl` (with `unicode-bidi: isolate`) **once at each text-content boundary** — the card/column/content wrapper — so every descendant inherits it. **Direction and alignment are separate decisions: never bundle a `text-align` into the direction rule.** Alignment follows the *design* — centered elements (section headers, badges) stay `text-align: center` and must not receive `start`/`end`; only content the design edge-aligns gets `text-align: start`/`end` (which then resolves against the declared direction). Verified failure: pairing `text-align: start` with the direction declaration on centered header blocks shoved the whole header to the right. (2) Keep geometry tracks (grids, flex rows, slider viewport/container) separate: pin a track's direction explicitly only when a physical order must hold in every locale, and never override the `<hdt-slider>` container/track direction — its JS follows the page `dir`. (3) Validate the section in **both** the `ar` and `en.default` locales before claiming RTL correctness.
- **Validate both RTL and LTR.**

### 2.7 Typography and brand

- Verify AYUM typography from the exact Figma nodes and current theme settings. Currently `config/settings_data.json` embeds the Google font **Almarai** (300/400/700/800) and maps heading/body to `--f_family_1`. Do not carry the previous project's El Messiri assumption into AYUM.
- **Do not hardcode a `font-family`.** Prefer, in order: existing theme typography classes → project typography variables / CSS custom properties (`--font-heading-family`, `--font-body-family`, `--font-semibold`, `--f_family_1..5`) → inherited font-family.
- **Do not put Liquid inside a plain static `.css` asset** (e.g. `src: url({{ 'font.woff2' | asset_url }});` in a `.css` file will not be processed). Use an inline `{% style %}` block in the Liquid section, or a valid static asset URL strategy.

### 2.8 Figma design tokens (AYUM — verify per exact section node)

Read the real AYUM values per node each time with `get_variable_defs` / `get_design_context`. Do not reuse the previous project's color or typography token values as AYUM defaults.


### 2.8a Color schemes come first (permanent — user rule)

All section colors must come from the **theme color schemes** in Theme settings → Colors. Never hardcode colors.

- **How schemes work here:** schemes are defined by the `color_schemes` `color_scheme_group` in `config/settings_schema.json` (fields: `text`, `text2`, `background`, `background_gradient`, `background2`, `button`, `button_label`, `button_border`, `secondary_button_label`, `accent_color`, `line_border`, `overlay`, `pr_text`, `price`, `sale_price`, `input_primary`, `input_secondary`). `snippets/css-variables.liquid` turns each scheme into CSS variables (`--color-foreground`, `--color-foreground2`, `--color-background`, `--color-background2`, `--color-accent`, `--color-secondary`, `--color-tertiary`, `--hdt-btn-*`, …). A section applies one through the shared `colors_by_section` + `color_scheme` settings (`color-scheme="{{ section.settings.color_scheme }}"` on the wrapper, as in `sections/catalog_card.liquid`).
- **Currently the store has one scheme only: `scheme-1`.** Re-check `config/settings_data.json` → `current.color_schemes` before each task; the merchant may add more.
- **Order of choice:**
  1. Use an existing scheme's variables (e.g. `rgb(var(--color-foreground))`, `var(--hdt-btn-bg)`), via the section's `color_scheme` setting.
  2. If the Figma colors for a section don't match any existing scheme, **add a new color scheme** (allowed by the user). Report the scheme's hex values, taken from Figma node data, and where it will be used. Then add it **additively** as a new `scheme-N` key in `config/settings_data.json` → `current.color_schemes`, filling every field. Never change or delete existing schemes, and leave the rest of the file byte-for-byte unchanged. This file is also rewritten by Theme Editor syncs, so tell the user to commit/push it promptly. If the editor changed it in the meantime, ask the user to create the scheme in Theme settings → Colors instead.
  3. **Add a per-section color setting only when no scheme field can express the color** (e.g. a one-off decorative accent). Give it the Figma hex as its default, and state why a scheme couldn't do it.
- **Don't add color settings "just in case".** If a scheme covers it, no color setting.
- Never edit an existing scheme's values to fit one section, because that changes every section using it.

### 2.9 Naming conventions for AYUM

- Use an `ayum` / `ay` marker for new AYUM-specific sections, blocks, snippets, assets, custom elements, and scope classes **only when a project-specific marker is actually needed**.
- Prefer clear names such as `ayum-hero`, `ayum-featured-products`, `ayum-icon-grid`, `ayum-testimonials` for **files** (sections, blocks, snippets, JS assets) and custom elements.
- **CSS classes and CSS custom properties always use the `cca-` / `--cca-` prefix** (§2.3a), not `ayum-` or `hdt-ay`.
- **Pick a unique prefix and check it does not collide with an existing `hdt-` class or custom element before using it.**
- Do not carry previous-project-specific names (`meatclub-*`, `mc-*`, `hdt-mc*`, `truecare-*`, `hdt-tc*`, `_tc-*`) into AYUM unless they already exist in the AYUM repository for a verified reason.
- **Never** use vague names such as `custom-section`, `new-section`, `test-section`, `section-1`.


### 2.10 Product cards

- Dispatcher: `section.settings.product_des` / `block.settings.product_des` selects `snippets/card-product{N}.liquid` (plus `card-product-list`, `card-product-discount1..3`). `card-product1` is the default and is rendered directly by many surfaces.
- **Never assume a product-card style number from the previous project.** Re-verify all existing `card-product*.liquid` files and the exact AYUM dispatcher/selector before choosing a number; never reuse an occupied value or infer the next number from another card system.
- **Numbering is local to the specific dispatcher or selector.** Never assume the next number; never infer it from another card system; never reuse an occupied value; never hijack an existing style.
- Shared sub-snippets affect **every** card: `card-product-media`, `card-price` (→ `price.liquid`), `badges`, `color-list`, `hdt-pr-card-rating` (a **0-byte reviews hook** — if ratings render nothing, that is why).
- Choose the closest existing card by **real markup and behavior**, not appearance alone.

### 2.11 Money, cart, variants

- Three canonical price snippets by surface: `snippets/price.liquid` (PDP/quick-view, variant-reactive), `snippets/card-price.liquid` (cards), `snippets/line-item-price.liquid` (cart/order lines). Reuse one; **do not add a fourth**.
- Render money only from authoritative raw values: Liquid `| money` / `| money_with_currency` on `variant.price`, `line_item.final_price`, `cart.total_price`, …; JS `formatMoney(cents, format)` fed raw integer cents. **Never parse a number out of formatted price text. Never compute totals/tax/discounts client-side.** Shopify owns cart contents, prices, taxes, discounts, and checkout.
- Cart mutations go through the AJAX + Section-Rendering + event-bus flow (`cart/add.js|change|update` with a `sections:` param, then a typed `Cart*Event`). Reuse `<hdt-cart-drawer>` / `<cart-items-component>` / `<hdt-line-item>` / `<hdt-quantity-wrapp>`; do not hand-roll cart fetches or mutate cart DOM outside this flow.

### 2.12 Sliders

- Existing custom elements: `<hdt-slider>` (primary, ~50 usages), `<hdt-slideshow>`, `<hdt-slider-thumb>`, `<hdt-slider-counter>`, `<hdt-slider-sync>`, `<hdt-scrollsnap>`, `<hdt-marquee>` — all defined in generated `assets/global.min.js` / `libs.min.js`. **Do not add a second slider library.**

### 2.13 Integrations and data

- Theme metafields: `metafields.theme.*` (brand, countdown, category_image, product_stock, custom_badges, label, …), `metafields.home.menu`, `metafields.ecomrise.colors`. Reference by exact `namespace.key`, confirm usage by grepping theme code — **not** by trusting `.shopify/metafields.json` (git-ignored per-developer CLI cache). Guard every metafield read for the absent case.
- Integrations: ecomrise (wishlist/compare/swatches, dual-mode with `localStorage`), Appstle Subscriptions (`selling_plan`), Google Shopping feed, `@app` blocks in 8 sections. App markup/state is **not** theme-owned — preserve the extension points, do not restyle or rewire.
- **Reviews are unresolved** (`hdt-pr-card-rating.liquid` / `hdt-pr-single-rating.liquid` are 0-byte hooks; Ryviu is commented out). Anything depending on real ratings/reviews is **blocked** until the merchant names the app.
- **The4 license/activation** (`purchase_code` in `settings_schema.json` + the design-mode block in `custom.js`) must be left intact — never hardcode, regenerate, strip, relocate, or log it.

---

## 3. Figma MCP inspection rules

Before writing any code for a section, inspect the **exact** desktop node and the **exact** mobile node with Figma MCP. Use `get_metadata` for structure, `get_design_context` for real properties, `get_variable_defs` for tokens, and `get_screenshot` for visual confirmation only.

Inspect, where available:

- Auto Layout direction · Auto Layout gap · padding
- width and height behavior · fixed, hug, or fill sizing · min/max dimensions
- constraints · alignment
- component structure · component variants
- variables · design tokens
- typography styles · font size · font weight · line height · letter spacing · text alignment
- colors · opacity · fills · gradients
- borders · corner radius · shadows · effects
- image cropping · image fit · object position
- desktop/mobile differences · hidden layers
- decorative assets · exportable assets

**Claude must not infer exact values by visually measuring screenshots when node data is available.** Screenshots confirm overall appearance *after* the real node values have been read.

Practical notes for the AYUM Figma file (`7cOVp6r92Z3heL1aTMP5N3`):

- Page frames are listed in the project identity table at the top of this skill. They are **context only**.
- Section numbering in the frames skips `02`. Desktop has a `04 / Category nav` bar that mobile replaces with the burger menu and a sticky bottom nav. Hidden layers (e.g. the desktop `14 / Newsletter`, the mobile size selector) are not to be built unless the user asks.
- Desktop frames are too large to read inline with `get_metadata`; query the section's own node instead. For implementation, Claude must still inspect and use the exact desktop/mobile node IDs for the single target section, per the source-of-truth rules above.
- Do not carry any old previous-project frame IDs, draft/duplicate-node warnings, or node maps into AYUM. Build the AYUM node map from this Figma file and report the verified section nodes in the task report.
- If a large-node metadata call exceeds the token limit, query the section's own node instead of repeatedly requesting the whole page.
- **Figma asset URLs are short-lived and must never reach a production file.** Do not paste a `figma.com` or `api/mcp/asset` URL into Liquid, CSS, JS, or template JSON. If an icon or decorative glyph is needed, download it once and store/transcribe it locally using the project's established asset pattern. Never store a large raw SVG string inside `templates/*.json`.

### 3.1 Frames without Auto Layout (permanent — the AYUM collection page)

Collection frames (verified): desktop `1:787` — filter sidebar (280w, sticky) beside a 3-column grid; mobile `1:1081` — 2-column grid with a sticky filter/sort bar, quick-filter chips and active-filter pills; mobile `1:1287` — the filter bottom sheet opened by "كل الفلاتر". There is no backdrop node in `1:1287`. Hidden layers (price/size filters, collection description) are not built unless the user asks. Reuse the theme's native facets (`facets.min.js` elements, `snippets/active-filters.liquid`) for filtering/sorting; restyle them, don't rebuild them.

The collection-page frames are **not** built with Auto Layout, so the node data gives absolute x/y positions and fixed widths/heights. Those numbers describe **one viewport only**, not how the layout should behave. Treat them as follows:

- **Take from the node data:** typography (size, weight, line height, letter spacing), colors, radii, borders, shadows, icon sizes, and image aspect ratios. These stay exact.
- **Infer the layout intent from the arrangement:** which elements form a row, a column, or a grid; their alignment; and which ones repeat (e.g. product cards). Compute gaps from the distance between sibling edges. If spacing between siblings is inconsistent, use the dominant value and flag the difference in the report.
- **Rebuild layout with flex/grid + `gap`**, never `position: absolute` with copied x/y offsets (absolute positioning is only for true overlays such as badges or icons on an image).
- **Never copy fixed px widths/heights onto layout containers, cards, or images.** Use `width: 100%`, `max-width`, `fr` / `minmax()` grid tracks, `flex: 1 1 …`, and `aspect-ratio` for media. Heights come from content and aspect ratio, not from the frame.
- **Column counts come from the schema** (`col_dk` / `col_tb` / `col_mb` → `hdt-grid-cols-*` utilities), not from a fixed card width.
- **Fluid sizing between designs:** where desktop and mobile values differ (font size, section padding, gaps), interpolate with `clamp()` or switch at the 768 / 1150 breakpoints. Never add a breakpoint for a single frame width.
- **Fixed px is fine only for** small intrinsic things: icons, borders, radii, control heights (buttons, inputs, swatches), and hairlines.
- Check the result at every width in §2.5 (320 → 1440+). Nothing may overflow, overlap, or leave a large empty gap between the designed widths.

---

## 4. Project inspection before coding

A pre-implementation inspection is **required before every section task**. Inspect:

- the target template · the template JSON · the section currently used there
- the closest existing section · relevant blocks · relevant snippets
- relevant CSS files · relevant JavaScript · existing custom elements · existing assets
- existing schema settings · existing naming conventions · existing utility classes
- existing responsive conventions · existing RTL conventions
- the existing slider/carousel implementation
- existing Theme Editor behavior · existing merchant data
- any manually added project changes

Also confirm the **active code path**. This theme ships near-duplicate and legacy files (`custom-section.liquid` is active; `custom_section.liquid` is orphaned legacy; there are three predictive-search variants; 13+ product-card variants). Grep `templates/` and `sections/*-group.json` for the exact `"type"` / render call before editing anything.

Identify the **closest existing project implementation** before creating a new custom solution, and explain what can be reused and why.

### 4.1 Required pre-implementation report

Produce this **before editing**, unless the user explicitly requests immediate implementation without a report.

1. **Files inspected** — each file and why it matters.
2. **Relevant existing architecture** — verified render path, blocks, styles, scripts, schema, data sources, integration points.
3. **Closest reusable section / block / snippet** — and the reuse-ladder step chosen, with justification.
4. **Proposed files to modify.**
5. **Proposed schema changes.**
6. **Proposed CSS scope** (the exact scope class / `section.id` selector and the exact stylesheet file).
7. **Proposed JavaScript changes, if any** — including why HTML/CSS cannot do it.
8. **Data-preservation considerations** — which setting IDs, block types, and saved template instances are at risk.
9. **Risks** — regression, saved data, lifecycle, responsive, RTL, localization, a11y, performance, app compatibility.
10. **Out-of-scope areas** — explicitly listed.
11. **Unanswered questions that materially affect implementation.**

**Ask the user only about details that cannot be safely resolved by inspecting the codebase or Figma.** Do not ask questions answerable from project files, Figma MCP, existing templates, or existing Theme Editor schema. Otherwise proceed with a clearly stated, low-risk assumption.

---

## 5. Existing-first implementation strategy

Prefer the **earliest** option that safely satisfies the design:

1. Use the existing project section, block, snippet, setting, class, or component **as-is**.
2. Use the existing structure with **scoped CSS only**.
3. Use an **instance-specific custom class setting**.
4. Add a **new schema setting beside** existing settings.
5. Add a **new schema option beside** existing options.
6. **Extend** the closest existing block or snippet safely.
7. **Duplicate** the closest block/snippet only when extension is unsafe.
8. **Carefully modify an original section** only when the change is minimal, reusable, and cannot safely be isolated another way.
9. **Duplicate an existing section** only when the same section must support a materially different structure that cannot be enabled as another instance.
10. **Create a completely new section** only as a last resort.

**This project is not duplicate-first.**

### 5.0 No duplicate-to-edit (permanent — learned from a repeated failure in the previous project)

In the previous project Claude repeatedly **duplicated an existing section just to edit it**, instead of walking the ladder above. That is prohibited here.

- **Walk the ladder in order, one step at a time.** Start at step 1 and move to the next step only after proving, in the pre-implementation report, why the current step cannot satisfy the design. Skipping steps is a failure, even if the result looks correct.
- **Duplicating a section (step 9) is never the way to change an existing section.** To change how an existing section looks or behaves, use steps 1–8: scoped CSS, the instance `custom_class`, additive settings/options that default to blank/false, or a minimal edit to the original.
- Before duplicating **anything** (block, snippet, or section — steps 7 and 9) or creating a new section (step 10): state which earlier steps were tried, why each one is insufficient, and **ask the user for approval** before creating the file.
- "It's easier", "it's safer to keep the original untouched", or "the design looks different" are **not** valid reasons to duplicate.

- Do not create a custom section automatically because the Figma design looks visually different.
- Do not introduce a parallel component system when the existing architecture can support the design.
- Do not replace working native behavior with weaker custom behavior.
- Prove the existing system is insufficient before building new, and say so in the report.

### 5.1 Existing-first architecture (permanent rule)

**Never create a new Section, Snippet, Block file, Component, JavaScript file, CSS file, or any other project file when an existing implementation can be safely reused, extended, or duplicated.**

Always inspect the closest existing implementation before deciding the implementation strategy.

Priority:

1. Reuse the existing implementation.
2. Extend the existing implementation.
3. Duplicate the closest existing implementation only when extension is not safe.
4. Create a completely new implementation only as the final option.

**Before creating any new file, explicitly justify why:**

- the existing implementation cannot be reused
- extending it is unsafe
- duplicating it is required
- creating a completely new implementation is unavoidable

Avoid: over-engineering · unnecessary abstractions · duplicate code · duplicate snippets · duplicate sections · duplicate JavaScript · duplicate CSS · unnecessary helper files · unnecessary wrappers · unnecessary custom elements.

Favor the **smallest safe implementation** that follows the project's existing architecture.

An additive setting on a shared file that defaults to blank/false — and therefore changes nothing for every existing instance — is almost always safer and smaller than a duplicate file. Verify that claim by listing every existing instance of the file and confirming the new default is inert for each, then say so in the report.

This is a permanent project rule and applies to every future implementation.

---

An isolated new section is acceptable when the design requires structural markup/schema/CSS that would **regress a live shared instance**. Before taking that route, prove the shared AYUM implementation cannot be safely extended, and get user approval (§5.0). When you take that route, still reuse the theme's conventions inside the new file — `spacing-vars`, `image_tag`/`image_url`, `placeholder_svg_tag`, color schemes, `hdt-container`, blocks lifecycle, font/color tokens — and leave every shared file untouched.

---

## 6. Custom class setting rule (permanent)

**Every reusable section must have an instance-specific custom class setting** unless:

- the section already has an equivalent setting, or
- the section is internal and cannot have schema, or
- adding it would break a strict framework contract, or
- there is a clearly documented technical reason not to add it.

When no equivalent exists, add a text setting like:

```json
{
  "type": "text",
  "id": "custom_class",
  "label": "Custom CSS class"
}
```

Apply the value safely to the **outermost stable section wrapper**. Requirements:

- preserve all existing wrapper classes
- do not replace native section identifiers
- do not use the merchant-entered class as an HTML `id`
- escape/sanitize the value appropriately for a class attribute
- do not render malformed whitespace
- do not render an unnecessary empty class fragment (guard with `{% if section.settings.custom_class != blank %}`)
- do not rename an existing equivalent setting
- reuse the existing setting if the project already provides one

The purpose is to let each section **instance** be targeted without editing Liquid again.

**AYUM requirement:** inspect the current repository for an equivalent `custom_class` setting before adding one. Reuse an existing equivalent; do not copy previous-project-specific class names, stylesheet names, or section-instance values.

**In addition to the merchant custom class, implementation-specific CSS must still use a stable, unique project scope class (or `#shopify-section-{{ section.id }}`) where required.** Do not rely only on a merchant-entered class for internal implementation styling when that would make the section fragile — a merchant can clear or retype it.

Hard-won lesson (verified in this repo): **never put a layout-affecting box property on the height-observed sticky-header element.** A `border-bottom` on `<hdt-sticky-header>` fed the theme's `ResizeObserver` → `--header-height` feedback loop and inflated the header toward viewport height. Use `box-shadow: inset …` for hairlines on observed elements.

---

## 7. Preservation rules

When reusing, extending, or duplicating an existing section, block, or snippet, **preserve**:

- all existing schema settings · all existing block types · all existing setting IDs
- existing option values · presets · default values (unless intentionally adding a new default)
- existing section functionality · product and variant behavior · AJAX behavior
- cart drawer integration · slider behavior · Theme Editor reload behavior
- localization behavior · accessibility behavior
- merchant Customizer data · template JSON data
- selected products · selected collections · images · videos · links
- manually configured blocks · manually added code (unless the user explicitly asks to replace it)

And:

- **Never casually rename a setting ID.** Setting `id`s, block `type`s (filenames), event-name constants, and CSS token names are an **append-only contract** with `config/settings_data.json`, `templates/*.json`, `sections/*-group.json`, and cross-asset imports. Renaming silently drops the merchant's saved value or breaks imports.
- Never delete settings because the current design does not use them.
- Never simplify a duplicated base by removing controls.
- Add new capabilities **beside** existing capabilities.
- Never reset a section instance to preset/default data.
- Never rewrite template JSON in a way that discards merchant content.
- Never remove unrelated JSON section entries or reorder unrelated sections.

Editor JSON (`templates/*.json`, `sections/*-group.json`, `config/settings_data.json`) is **data, not source** — it carries an auto-generated banner and is rewritten by the Theme Editor. Change behavior in `.liquid` + `{% schema %}` defaults/presets. Touch template JSON only to wire or seed a section instance, and keep it strictly valid. **Never hand-edit `config/settings_data.json`** (only exception: adding a new color scheme, §2.8a).

When inserting a section into a template's `order`, preserve every other key, setting, block, and order entry **byte-for-byte**, and state in the report where the new entry landed and what shifted.

---

## 8. Shopify Theme Editor and schema

All merchant-managed content should remain editable where practical: headings · descriptions · labels · buttons · links · images · desktop images · mobile images · icons · videos · video posters · products · collections · repeated cards · colors · backgrounds · alignment · layout options · section spacing · content width · visibility options.

Schema rules:

- keep JSON valid (a stray comma breaks the whole page/template)
- preserve existing IDs, settings, blocks, presets
- keep labels clear; keep section and block names within Shopify limits; avoid unnecessarily long labels
- avoid duplicate IDs
- ensure empty settings do not create broken markup
- use sensible defaults
- avoid hardcoding merchant content
- use blocks for repeatable content where appropriate
- use `image_picker` instead of hardcoded final images
- use Shopify video / video-URL settings where appropriate
- use product and collection pickers where appropriate — the picker is authoritative for title and URL (do not add a parallel manual title/link setting that can drift)
- every section needs a **valid preset**, and the section must render sanely with the preset's defaults

**Localization:** merchant-entered content from the Theme Editor does **not** automatically require locale keys. Reusable interface strings added by the theme should use locale keys where appropriate (`{{ 'ns.key' | t }}` from `locales/*.json`; add to `en.default.json` first, then mirror into `ar.json` and peers with an identical JSON path — a key missing from a peer silently falls back to English; a key added only to a non-default locale never appears). Do not add locale keys for content the merchant directly edits unless the project architecture specifically requires it.

**Schema labels:** the shipped Kalles sections use `"t:"` keys resolved from `locales/*.schema.json`. For AYUM, inspect the current locale/schema files before choosing plain labels vs `"t:"` keys. Match whatever the file you are editing already does, and never mix in a `t:` key that does not exist.

---

## 9. CSS rules

Before editing CSS, inspect the project and identify the correct stylesheet for the target page or component. Claude must:

- confirm the stylesheet is actually loaded on the target template
- use the correct page-specific or component-specific file
- avoid global theme CSS when a scoped project file is available
- **never edit minified CSS**, and never hand-edit `assets/base.css` or `assets/theme.css`
- never place product-page CSS in a collection-only file
- never place collection-page CSS in a homepage-only file
- never use broad selectors that affect unrelated sections
- scope new CSS under a stable unique parent class (or `#shopify-section-{{ section.id }}`)
- preserve native utility classes
- use logical properties where possible
- avoid unnecessary `!important`; explain any that is required
- avoid high-specificity selector chains unless necessary (prefer winning on specificity honestly, e.g. `.cca-header .hdt-container`, over `!important`)
- avoid brittle `nth-child` selectors when semantic classes can be added safely
- avoid styles dependent on specific Theme Editor block order unless the design intentionally requires that order

**Where CSS goes in this project:**

| Need | Location |
| --- | --- |
| Instance-specific values that need Liquid (colors from settings, gated logic) | inline `{% style %}` in the section, scoped to `#shopify-section-{{ section.id }}`, emitting only `--cca-` variables (§2.3a) |
| Any new AYUM rule (section, block, card, override) | `assets/dev1.css`, anchored on a `cca-` class (§2.3a) |
| Rules gated to one instance of a **shared** section | `assets/dev1.css`, selectors prefixed with that instance's `cca-` custom class |
| Truly global rule | a token in `css-variables.liquid` — **never** a section file, never `base.css` |

Never hardcode a color, spacing, radius, or width where a token exists. **Colors follow §2.8a: color scheme variables first, a new color scheme second, a per-section color setting only as a last resort.** When a color setting is unavoidable, emit it as `{{ setting.rgb }}` and wrap usages in `rgb(...)`.

---

## 10. Section architecture

Every section should:

- have one stable outer wrapper
- preserve Shopify's section identity (`#shopify-section-{{ section.id }}`, `shopify_attributes` on blocks)
- support the custom class setting rule (§6)
- avoid broken output when optional settings are empty
- avoid unnecessary empty containers
- preserve semantic heading order
- use accessible buttons and links
- use responsive images (`image_tag` / `image_url` with `widths`/`sizes`, explicit `width`/`height`)
- use alt text correctly
- use lazy loading where appropriate
- avoid layout shift where practical
- remain functional in the Theme Editor
- remain functional when blocks are added, removed, or reordered
- avoid hard dependencies on preset block IDs
- support fallback behavior for missing mobile assets
- support merchant content without requiring code edits

Do not automatically add settings that add no merchant value. Where appropriate, reusable sections should support: section top/bottom spacing · desktop/mobile spacing · background color · background image · content width · alignment · desktop/mobile visibility · heading/content controls.

**Do not build a large generic settings framework onto every section — this theme already handles most of it globally.** Use the shared Kalles scaffold instead of inventing spacing settings: `section_layout` select (`container` / `wrapper_container` / `full_width` → `hdt-container*` classes), text `mg` / `mg_mb` / `pd` / `pd_mb` consumed via `{% render 'spacing-vars', box: section %}` inside a `{% style %}` block (CSVs are `top,right,bottom,left`), optional `colors_by_section` + `color_scheme`, `background_opacity`, `section_background`.

Blank states are part of the deliverable. Decide and implement explicitly: what renders when a picker is empty, in the **Theme Editor** (`request.design_mode` → onboarding placeholder) vs the **live storefront** (often: hide the section entirely rather than render a broken link/price/image).

---

## 11. Image and asset rules

- **Do not hardcode final Figma images into Liquid.** Prefer `image_picker` → product images → collection images → metaobjects/metafields → theme assets only when explicitly required.
- Where a design has separate desktop and mobile images: add both settings when appropriate, treat mobile as an **override**, and fall back to desktop when mobile is empty (`<picture>` with `<source media="(max-width: 767px)">`, or `image_mb | default: image` as the shipped `_slide-item` block does).
- Provide a placeholder path (`placeholder_svg_tag`) so a blank picker never renders a broken image.
- Inspect the Figma asset structure and export settings before adding assets. Do not export elements that can be created cleanly with CSS.
- Decorative shapes may use pseudo-elements when appropriate. Decorative elements must not block interaction — use `pointer-events: none`, and `aria-hidden="true"` where they carry no meaning.
- Do not use CSS backgrounds for meaningful content images that require alt text.
- Never commit a Figma MCP asset URL (see §3).

---

## 12. Slider and carousel rules

Before modifying or creating a slider, inspect: the project's existing slider library · its DOM contract · initialization · custom elements · data attributes · responsive settings · navigation · pagination · Theme Editor reinitialization · destroy/rebuild behavior.

Preserve: drag · swipe · keyboard behavior where supported · navigation · pagination · resize behavior · dynamic block changes · Theme Editor section reload.

- **Do not replace the native slider with a new library** unless there is a strong documented reason. There is no second slider library in this theme and there must not be one.
- Do not create fake static card widths that only match one viewport.
- Do not override internal slider variables without inspecting how the project uses them.
- If a **partial next card** ("peek") is required, achieve it through the existing slider system safely (precedent: a scoped `.hdt-slider__slide { flex: 0 0 43% }` under the section's own class on mobile, letting the native RTL-aware slider place the peek on the correct edge).
- **Do not convert a static Figma grid into a slider unless explicitly required. Do not convert a slider into a static grid unless explicitly required.**
- Do not globally restyle dots or arrows. Scope all slider appearance changes to the section.
- Reuse the native arrow/dot **elements** (semantics, `aria-*`, `disabled` at the ends, keyboard, RTL chevron flip) and change only their appearance.
- Slide counts come from `col_dk` / `col_tb` / `col_mb` at the 768/1150 breakpoints — do not invent per-viewport widths.
- Sliders re-init on `shopify:section:load`; keep initialization idempotent and never double-initialize.

---

## 13. Product card rules

If the task involves product cards, inspect: all relevant existing product-card snippets · product-card style dispatchers · schema options · Theme Editor selectors · homepage card systems · collection card systems · search card systems · related-product cards · quick view · wishlist · compare · ratings · badges · swatches · price · compare-at price · sold-out state · choose-options behavior · add-to-cart behavior · AJAX/cart-drawer integration.

- **Never assume the next product card number.** Numbering is local to the specific dispatcher or selector. Before adding a new style: inspect all existing options in AYUM, find the next free value **in that exact system**, do not reuse an occupied value, do not infer the number from another card system, and do not hijack an existing style.
- Register a new style only in the AYUM dispatcher(s) that actually need to offer it. Register the `when` branch(es) **and** the `product_des` select option, and load any card-specific CSS conditionally.
- Choose the closest existing card based on **real markup and behavior**, not just appearance.
- Preserve all native card functionality.
- If the native add-to-cart location differs from Figma, **reuse and reposition the native behavior** (`snippets/btn-ultra.liquid` and its quick-add / choose-options / sold-out paths) rather than replacing it with a weaker custom button. Extend it with an **optional** additive param (e.g. an `add_label` defaulting to the native key) so existing call sites are unaffected.
- **Do not globally clamp all product titles.** Any title clamp must be scoped to the custom card/section and use selectors that match the real DOM.
- Keep all card variants composing the shared sub-snippets so badges/price/rating stay uniform. A change to a shared sub-snippet affects **every** card — verify each variant still composes.

---

## 14. Video rules

Inspect existing project video implementations before creating custom JavaScript. Preserve: native video settings · video poster · play-button behavior · pause behavior · accessibility · mobile behavior · Theme Editor behavior.

- **Do not autoplay unless explicitly required.**
- Do not call `video.play()` on page load unless explicitly requested.
- Do not create a decorative play button that has no functionality.
- Avoid adding a custom media player when the project already has a working implementation (`sections/product-video.liquid` and the native media components exist).

---

## 15. JavaScript rules

**Do not add JavaScript when HTML/CSS or the existing component system can handle the task.** AYUM sections should ship with **zero** new JS whenever HTML/CSS or the existing component system is sufficient.

When JavaScript is genuinely required:

- use existing project patterns (a small isolated Web Component in its own `assets/*.js`, loaded `defer`)
- avoid global listeners when section-scoped listeners are possible; no `document`-level state
- avoid duplicate initialization — guard `customElements.define`
- support Theme Editor section load/unload events; make `connectedCallback` resilient (rAF retry until children are parsed) and idempotent
- clean up event listeners, observers, timers, and rAF handles in `disconnectedCallback`; prevent memory leaks
- use progressive enhancement — the section must render sanely with JS disabled
- preserve keyboard accessibility
- avoid blocking rendering — **never load a component script with a parser-blocking `script_tag` immediately before its element** (verified failure: the element upgraded mid-parse, `connectedCallback` found no children, and the listener was never bound). Use `<script src="…" defer>`.
- avoid unnecessary dependencies; never introduce a library for one section
- **never edit minified JavaScript**
- do not create global variable collisions — check the element name is not already registered (`grep customElements.define` across `assets/*.js` and the minified bundles)
- import shared helpers from `@theme/utilities` rather than reimplementing them

**Do not rewrite working JavaScript for a visual-only change.**

---

## 16. Accessibility

Preserve or improve: semantic HTML · heading hierarchy · link/button semantics · alt text · keyboard interaction · focus states · visible focus · `aria-label`s where needed · slider controls · video controls · form labels · reduced-motion behavior where relevant · readable text contrast.

- Do not use a `div` as a button when a `button` is appropriate.
- Do not remove screen-reader text (`sr-only`, `aria-*`) from native functionality.
- Do not hide meaningful content only visually without considering accessibility.
- Decorative SVG/icons: `aria-hidden="true"`, empty `alt`, `pointer-events: none`.
- Duplicated visual content (e.g. a marquee's second track) must be `aria-hidden` and must **not** carry `block.shopify_attributes` — only the real, editor-selectable copy does.
- Respect `prefers-reduced-motion` for any animation, and provide a genuinely static fallback (not a scrollable one).
- Preserve focus management in dialog/drawer components (derived from `DialogComponent` / `DrawerComponent`).

---

## 17. Performance

Prefer: existing assets · responsive images · Shopify image filters · appropriate image widths · lazy loading below the fold · minimal JavaScript · minimal DOM duplication · scoped CSS · reusable snippets where justified. Reuse `fetchCache`, `debounce`, `throttle`, `requestIdleCallback` from `utilities.js`.

Avoid: huge hardcoded images · unnecessary duplicate mobile and desktop markup · layout-shifting images without dimensions/aspect ratio · loading new libraries for one section · large inline scripts · repeated Liquid loops over the same data without need · eagerly loading section CSS globally.

Do not sacrifice correctness for premature micro-optimization.

---

## 18. Page-specific isolation

Every task prompt must identify out-of-scope areas.

**Homepage task:** do not touch product pages · do not touch collection pages · do not touch unrelated homepage sections.

**Product-page task:** do not touch the main product form · variant selection · price · quantity · gallery behavior · add-to-cart — unless explicitly in scope.

**Collection-page task:** do not touch filters · sorting · search · pagination · grid/list controls · the collection heading — unless explicitly in scope.

Shared global areas (announcement bar, header, footer, cart drawer, mobile toolbar) appear on **all** pages — a change there is a cross-page change. Treat it as its own task, scope it with a custom class, and verify it did not alter other pages or other instances of the same section.

---

## 19. Manual changes and merchant data

If the user says manual changes have already been made:

- inspect them first
- preserve them
- do not reset the file to an older base
- do not overwrite JSON settings
- do not remove configured content
- refactor only when necessary
- explain the reason before broad structural changes

**Assume project files may contain important manual modifications even when they differ from the original theme.** Do not replace a modified file with a clean vendor version. Theme JSON is also round-tripped by `shopify theme pull` — the template you read may differ from what an earlier report describes; trust the current file and note the drift.

---

## 20. Follow-up and bugfix workflow

For a follow-up fix:

- **diagnose the root cause first** — do not patch symptoms
- do not rewrite the whole section
- preserve the current implementation
- make the smallest safe fix
- touch the fewest files and lines
- keep CSS under the existing scope
- preserve settings, IDs, blocks, and merchant data
- verify desktop and mobile
- verify that one breakpoint fix does not regress another
- do not touch unrelated sections
- do not run broad refactors
- do not introduce new breakpoints, hardcoded values, or new event systems

The follow-up report must include: **root cause · exact fix · files changed · behavior preserved · responsive checks · Theme Editor checks · any remaining limitation.**

---

## 21. Validation

There is **no build, unit test, or repo-scripted lint** in this project. Validate with lightweight targeted checks and report exactly what ran.

Before claiming completion, verify:

- correct Figma desktop node was inspected
- correct Figma mobile node was inspected
- Figma values came from node data where available
- correct template was edited
- correct section instance was targeted
- correct CSS file was used
- CSS is scoped
- custom class setting exists, or an equivalent existing setting was reused
- existing settings were preserved
- existing blocks were preserved
- existing IDs were preserved
- merchant data was preserved
- no unrelated sections changed
- desktop matches Figma
- mobile matches Figma
- tablet behavior is safe
- behavior at the wide end of the desktop tier is safe
- RTL/LTR behavior is safe
- slider behavior still works
- product behavior still works
- video behavior still works
- Theme Editor reload works
- optional empty settings do not break markup
- schema JSON is valid
- Liquid syntax is valid (`{% style %}` / `{% schema %}` / `{% content_for %}` tags balanced)
- JavaScript has no obvious syntax errors
- no minified files were edited
- no unnecessary new library was introduced
- no `figma.com` / `api/mcp/asset` URL appears in any production file
- `assets/base.css`, `assets/theme.css`, `assets/*.min.js`, and `config/settings_data.json` are unchanged (except an approved additive color scheme, §2.8a)
- the diff is limited to the intended files

Practical checks that work here: JSON parse of every edited `{% schema %}` and template JSON; brace/tag balance; `git diff --stat` to confirm scope; grep for physical `left`/`right` in new CSS; grep for `var(--color-` not wrapped in `rgb(`; grep for new `@media` values outside 768/1150; grep `assets/dev1.css` to confirm every rule is anchored on a `cca-` class; grep the diff to confirm no CSS was added outside `dev1.css` (except `--cca-` variables in inline `{% style %}`).

**Do not run destructive commands. Do not run broad formatting across unrelated files. Never run `shopify theme check` yourself — when it would help, ask the user in the chat to run it and share the output.** Never report it as passed unless the user shared a passing result.

Runtime, visual, responsive, RTL, slider, cart, and editor-lifecycle behavior generally **cannot** be verified in this environment. Report those as **Requires storefront QA (Not run)** — never as passed.

---

## 22. Final implementation report format

Report, in this order:

1. Files inspected
2. Figma nodes inspected
3. Existing architecture reused
4. Files changed
5. Files intentionally untouched
6. Liquid/HTML changes
7. Schema settings added
8. Blocks/options added
9. CSS scope class
10. CSS changes
11. JavaScript changes
12. Theme Editor behavior
13. Responsive verification
14. RTL/LTR verification
15. Accessibility notes
16. Performance notes
17. Merchant-data preservation
18. Validation performed
19. Manual Theme Editor steps
20. Known limitations or QA notes

**The report must not claim something was tested if it was only inspected statically.** Clearly distinguish:

- **inspected** — read the code/node, reasoned about it
- **validated** — a check actually ran and its result is stated
- **manually testable** — the exact steps a human should perform
- **not testable in the current environment** — say so plainly

Mark every validation item **Passed / Failed / Not run / Not available / Requires manual verification**.

---

## 23. Section task prompt structure

Use this structure for every future section task. Fill in the exact nodes; do not substitute a page-level node.

```
Implement ONE section of the AYUM design into this Shopify theme.

Project context: AYUM Shopify project (`ayum`). Verify Shopify/theme version and active locale/direction from the repository before relying on them.
Target page:      [Homepage | Product | Collection]
Target template:  [templates/index.json | product.json | collection.json | <suffix>.json]
Section name:     [NAME]
Figma desktop node: [EXACT NODE ID]
Figma mobile node:  [EXACT NODE ID]
Screenshots:        [links/attachments — visual confirmation only]
Dynamic content source: [settings / blocks / product / collection / metafield / menu / app]
Grid or carousel behavior: [desktop / tablet / mobile]
Related product-card requirements: [card style, badges, price, add-to-cart placement]
Correct CSS file: assets/dev1.css (+ inline {% style %} with --cca- variables only if Liquid values are needed)
Existing section/base to inspect: [file path(s)]

RULES (all mandatory):
- Source-of-truth priority: SKILL §1. Read values from Figma node data, not screenshots.
- Existing-first ladder: SKILL §5. Justify the chosen step. Do not default to a new section.
- Preservation rules: SKILL §7. Never rename/delete a setting id, block type, event, or token.
- Custom class setting rule: SKILL §6.
- Responsive: SKILL §2.5 — breakpoints 768 / 1150 only. Verify 767/768/1149/1150 + 1440 + 390 + 375 + 320.
- RTL/LTR: SKILL §2.6 — logical properties; verify both directions.
- Out of scope: [explicit list — everything not named here]
- Validation checklist: SKILL §21.
- Final report: SKILL §22.
- Produce the pre-implementation report (SKILL §4.1) BEFORE editing.

Do not start the next section.
```

**Every section implementation prompt must end with: `Do not start the next section.`**

For a follow-up bugfix, use the same header (section, page, exact nodes, reported issue, where it appears) plus: reinspect the current active file → find the **root cause** → minimal scoped fix → regression-test the bug scenario + desktop/mobile + RTL + related states + editor reload. `Do not rebuild the section. Do not start the next section.`

---

## 24. Project plan

There is **no project plan file** for AYUM (user decision). The Figma file (§3) is the section inventory; each task's report (§22) is the record. Do not create a plan file unless the user asks. Never call a section verified without real desktop **and** mobile (**and** RTL) QA.

---

## 25. Permanent failure modes — explicitly prohibited

- implementing the full site in one task
- implementing unrelated sections
- creating a new section before inspecting existing options
- duplicating a section when scoped extension is enough
- duplicating a section in order to edit it (§5.0), or skipping ladder steps
- duplicating or creating a section/block/snippet without user approval
- deleting existing schema settings
- renaming setting IDs casually
- removing blocks
- resetting Customizer data
- resetting template JSON
- hardcoding merchant images
- hardcoding content that should be editable
- using screenshots to estimate values when Figma node data exists
- copying absolute x/y positions or fixed frame widths/heights from non-Auto-Layout frames into layout CSS (§3.1)
- treating mobile as scaled desktop
- ignoring tablet behavior
- using broad CSS selectors
- styling global sliders
- replacing native functionality with weaker custom code
- inventing product-card style numbers
- hijacking an existing style option
- choosing a product-card base without inspection
- breaking swipe or drag
- converting a grid into a slider without instruction
- autoplaying video without instruction
- adding unnecessary JavaScript
- adding unnecessary libraries
- editing minified files
- running broad refactors
- overwriting manual changes
- modifying unrelated templates
- claiming unperformed testing
- running `shopify theme check` yourself instead of asking the user in the chat

Project-specific additions, each learned from a real failure or constraint in this repository:

- committing a short-lived Figma asset URL into Liquid/CSS/JS/JSON
- hand-editing `config/settings_data.json` (the only exception is adding a new color scheme, per §2.8a) or `assets/base.css` / `assets/theme.css`
- hardcoding a color, or adding a section color setting where a color scheme can do the job (§2.8a)
- stripping, regenerating, relocating, or logging the The4 `purchase_code` / activation block
- putting a layout-affecting box property (border/padding/margin) on the height-observed `<hdt-sticky-header>` element
- loading a Web Component's script with a parser-blocking `script_tag` right before its element
- leaving a custom element with the default `display: inline` when it must clip (`overflow: hidden` does not clip an inline box, and `clientWidth` reads `0`)
- adding `overflow-x` to `html` / `body` to hide a section's overflow instead of containing it in the section
- introducing a third layout breakpoint (only 768 and 1150 exist)
- hardcoding `font-family` instead of using the theme font tokens (the theme font — currently Almarai — is already global)
- putting Liquid inside a plain static `.css` asset
- adding a new CSS class without the `cca-` prefix, putting AYUM CSS anywhere but `assets/dev1.css`, or writing a `dev1.css` rule not anchored on a `cca-` class (§2.3a)
- registering a custom element name that collides with a native `hdt-` component
- copying `block.shopify_attributes` onto a duplicated/decorative DOM copy
- inventing testimonial/review/marketing content that does not exist — seed only verified copy and leave the rest for the merchant

---

## 25b. Additional inherited standards

**Liquid partials.** Partials are shared with `{% render 'name', param: value %}`, passing every dependency explicitly — `render` is an **isolated scope**, and a missing required param renders **blank or a placeholder, not an error** (this looks exactly like data loss). Shared snippets document their contract in a leading `{%- comment -%}` header (`Renders / Accepts / Usage`, e.g. `snippets/price.liquid`) — match the documented params exactly and copy an existing call site. `{% include %}` is not used anywhere in the product/cart/money paths; do not introduce it. Start new section files with a `<!-- name.liquid -->` header comment, as existing files do.

**Dates.** Use the `@theme/dayjs` alias (`day.min.js`), not raw `Date`. Countdown timezone comes from `themeHDN.settings.shopTimezone` (emitted only when `countdown_timezone == 'shop'`).

**Mixed RTL text + numeric values (permanent — learned from the discount-badge bug).** For UI labels combining Arabic text with numeric/LTR content (discount percentages, ratings, quantities, measurements): never rely on the inherited page direction — it is locale-driven and LTR by default, which visually reorders the runs (e.g. `70%وفري`). Keep the semantic DOM/translation order (label first), give the element an explicit `direction: rtl; unicode-bidi: isolate` (or wrap the numeric part in `<bdi>`), and never use `row-reverse` or content reordering as a bidi fix. Geometry positioning stays a separate decision from text direction.

**Storefront money-format consistency (permanent).** AYUM has **no custom money formatter** — prices use Shopify's `| money` filters inside the three canonical price snippets (`card-price`, `price`, `line-item-price`), and `formatMoney` (`assets/utilities.js`) for client-rendered money. If a custom format is ever required, add it once in a shared snippet wired into all three, mirror it in `formatMoney`, and record the decision here. Never format prices independently per section/card, never hardcode currency strings, and never mutate numeric Shopify price values (presentation only). When touching prices, verify: current price, compare-at, ranges, dynamically updated variant prices, and the major card contexts.

**Native media-commerce sections (permanent — established in a previous project on this same Kalles version; re-verify each fact against AYUM's files before relying on it).** When Kalles already provides a specialized commerce component (e.g. the "Shoppable Video" section: `<hdt-video-player>` + product picker + `<hdt-slider>` + native dots), reuse its media lifecycle, native product connection, native add-to-cart path (`btn-ultra`), and native slider — and preserve its full schema. Add project-specific presentation and settings additively (instance-gated markup, blank-default settings, scoped CSS). Useful verified facts: the `<hdt-slider>` `config` attribute passes **any** Embla option through (`Object.assign(defaults, JSON.parse(config))` — including per-viewport `breakpoints` such as a mobile-only `align: "center"`); the ClassNames plugin puts `.is-selected` on snapped slides (native active-slide hook); `<hdt-video-player>` without the `autoplay` attribute binds its own click-to-play — but with `{ once: true }`, so it is play-only and dies after the first click; a play↔pause toggle needs one delegated listener (custom.js) calling the element's own `play()`/`pause()` gated on its `[playing]` attribute, which it reflects for every source (Shopify video / YouTube / Vimeo). **`.hdt-media-overlap-content` is a CSS GRID with two traps (verified in the previous project): its media children — `hdt-video-player` included — get `pointer-events: none` (re-enable them scoped when the design needs click-to-play), and its non-media children are force-placed into the inset CENTER content cell (`grid-area: content-r/content-c; width: 100%`) — an absolutely-positioned grid item resolves inset offsets against its grid AREA, so any edge-anchored overlay must re-span the full grid (`grid-area: full-r-start/full-c-start/full-r-end/full-c-end`) or its top/right will silently center.** Shopify Liquid `video.duration` is in MILLISECONDS — derive durations from it in Liquid instead of merchant-entered text or JS metadata listeners. When restyling a native media section, never trade a functional player for a static wrapper, and verify actual runtime playback (a headless-browser click test against `shopify theme dev` works) — CSS alone can silently kill interaction. Store pulls can revert uncommitted corrective files: after any `shopify theme pull`/"theme editor" commit, re-verify recent fixes still exist on disk before diagnosing deeper. Do not rebuild specialized media-commerce behavior from generic cards or third-party slider/player libraries.

**Currency display is presentation only** (`assets/hdt-currencies.js`, `localStorage['hdt_currency_iso']`). Shopify Markets/checkout owns the charged amount — never treat a converted display value as authoritative, and never hardcode a currency symbol from the design (currency figures in the Figma are placeholders).

**Security and privacy.** Never expose secrets, tokens, customer/personal data, private URLs, or environment values in code, output, or commits. Treat `.shopify/` as local, non-shared state — never reference its paths from theme code or surface its values.

**Git.** Do not run destructive git operations without explicit instruction. Commit or push only when explicitly asked. Prefer `shopify theme push --only <path>` to avoid overwriting merchant-edited JSON.

**Do not present a project-specific convention as a universal industry rule.** Keep universal engineering practice, Shopify/Kalles platform behavior, and AYUM project conventions distinguishable in your explanations.

**When not to implement:**

- the request is audit/plan/report only — inspect and report, do not edit
- the change requires renaming or removing a persisted identifier without an approved migration
- the change would edit generated/minified vendor output where editable source exists
- the change would strip or alter the license/activation code, or expose secrets
- the active code path cannot be confirmed and editing the wrong file risks a silent no-op or data loss
- the section depends on a blocked decision (e.g. the unresolved reviews app) — implement everything else and report the blocked part

---

## 26. Decision matrices

### Reuse vs extend vs duplicate vs rebuild

| Situation | Action |
| --- | --- |
| Existing section/snippet/component already does it | Use as-is; configure via existing settings |
| Needs a visual tweak only | Add a `cca-` class (or `cca-` custom class value) and style it in `assets/dev1.css` |
| New behavior useful to all instances | Add an additive setting/param, or extend the shared snippet/base class |
| Behavior must be isolated to one surface | Custom-class-gated markup/CSS or a blank-default setting on the original first; duplicate the closest file only after proving those insufficient **and getting user approval** (§5.0) |
| Change would regress a live shared instance or its preset | Prove an inert additive setting can't avoid the regression; only then, with user approval, build an isolated new section reusing the theme's conventions inside it |
| No existing architecture can satisfy it safely | Build new (last resort), following existing patterns |

### Text/content source

| Content | Source |
| --- | --- |
| Fixed UI copy the theme owns | translation key (`\| t` / `t:`) |
| Merchant-configurable per section | schema setting or repeatable block |
| Per-product/collection data | `metafields.<ns>.<key>` (verify it exists; guard the absent case) |
| Title/URL of a picked resource | the resource object (`collection.title`, `product.url`) — never a parallel manual setting |

### Direct change vs needs verification

| Signal | Action |
| --- | --- |
| Convention confirmed across several files, current | Implement following it |
| Appears once, or in a legacy/duplicate/experimental file | Confirm the active path first; mark **Needs verification** |
| Behavior owned by an app or by Shopify | Preserve the boundary; do not reimplement |

---

## 27. Final agent checklist

**Before implementing:**

- [ ] Restated the outcome and acceptance criteria, and what must not change.
- [ ] Inspected the **exact** desktop and mobile Figma nodes via MCP; read values from node data.
- [ ] Confirmed the active code path (right template / section instance / block / card variant / asset).
- [ ] Read complete relevant files, including schema, stylesheet, and any JS component.
- [ ] Identified the closest existing implementation and the reuse-ladder step, with justification.
- [ ] Checked saved-data impact (setting `id`s, block `type`s, template instances, merchant content).
- [ ] Listed out-of-scope areas explicitly.
- [ ] Produced the pre-implementation report (§4.1).

**Before claiming completion:**

- [ ] Edited only editable source — no `*.min.js`, `base.css`, `theme.css`, or `settings_data.json` hand-edits (except an additive color scheme, §2.8a).
- [ ] Preserved identifiers, presets, merchant data, native behavior, app boundaries, and the The4 license code.
- [ ] Custom class setting present (or an existing equivalent reused); all CSS scoped.
- [ ] All new CSS is in `assets/dev1.css`, every new class is `cca-`-prefixed, every rule is anchored on a `cca-` class (§2.3a).
- [ ] Colors come from color schemes (§2.8a); no hardcoded colors, no unneeded color settings.
- [ ] Used tokens, logical properties, `ThemeEvents`, shared snippets — no hardcoded colors/fonts/spacing, no invented events, no DOM-parsed money, no Figma URLs.
- [ ] Blank/empty states, mobile media fallback, and preset all handled.
- [ ] Ran the validation checks in §21 and recorded each as Passed / Failed / Not run / Not available / Requires manual verification.
- [ ] Wrote the final report in the §22 format, honestly distinguishing inspected vs validated vs not testable.
- [ ] **Did not start the next section.**
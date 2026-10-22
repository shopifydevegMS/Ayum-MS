# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Shopify Online Store 2.0 theme for the store "Ayum-MS", built on the commercial **Kalles 5.3.0** theme by The4 (docs: https://support.the4.co/collections/kalles-5). It is connected to Shopify via the GitHub integration — commits titled "Update from Shopify for theme Ayum-MS/main" are pushed back by Shopify when settings are changed in the theme editor. There is no build step, package.json, or test suite: files in the repo are served as-is.

## Commands

Uses the Shopify CLI (install separately):

- `shopify theme dev --store <store>.myshopify.com` — local preview with hot reload
- `shopify theme check`: Liquid/JSON linting (Theme Check). **Don't run it yourself.** When it would help, ask the user in the chat to run it.
- `shopify theme push` / `shopify theme pull` — sync with a store theme (prefer letting the GitHub integration sync `main`)

## Architecture

- **Layout**: [layout/theme.liquid](layout/theme.liquid) renders `header-group`, `footer-group` and `system-group` section groups, plus snippets `css-variables` (global CSS vars, loads `base.css`/`theme.css`), `js-variables` (exposes the `themeHDN` global config object), `scripts`, and `system-bottom` (loads `custom.js`).
- **RTL**: direction is decided in `theme.liquid` from `settings.right_to_left` — by default RTL turns on automatically for RTL locales such as `ar`. `is_rtl` is passed into `css-variables`, and `<html>` gets `dir` and a `dir--rtl` class. Store content is written in Arabic (the Figma design is RTL) and there is an Arabic locale ([locales/ar.json](locales/ar.json)), so check every layout change in both `ar` and `en.default`.
- **JavaScript**: native ES modules wired by an import map in [snippets/scripts.liquid](snippets/scripts.liquid) (`@theme/libs`, `@theme/global`, `@theme/utilities`, `@theme/events`, etc.). Most vendor files are minified (`*.min.js`) with no source in the repo, so don't edit them; put changes in readable files like `assets/custom.js`, `utilities.js`, `events.js`. Cross-component communication goes through `ThemeEvents` from `@theme/events` (cart update, dialog open/close, currency update, …).
- **Sections and theme blocks**: sections in `sections/` mostly render theme blocks from `blocks/` via `{% content_for 'blocks' %}` or static `{% content_for 'block', type: ..., id: ... %}`. Block files starting with `_` are private (only usable inside the sections/blocks that reference them); files without the prefix (`text`, `heading`, `button`, `group`, …) are general-purpose blocks.
- **CSS**: global styles are in `base.css` and `theme.css` (don't hand-edit either); all project CSS changes go in `assets/dev1.css` (see below), which loads last in `snippets/css-variables.liquid` so it wins over theme CSS; each section loads its own stylesheet (e.g. `{{ 'featured-collection-2.css' | asset_url | stylesheet_tag }}`). The theme's own classes and assets use the `hdt-` prefix.
- **Snippets**: shared partials, including many product-card variants (`card-product1`–`12`) chosen by theme settings.
- **Templates and config**: `templates/*.json` (with alternates such as `collection.left-sidebar.json`) plus `sections/*-group.json` and `config/settings_data.json` hold merchant content and settings. They are often overwritten by Shopify editor syncs, so put behavior in Liquid/schema, not in these files, and expect merge conflicts. The one required edit is filling the section instance with the Figma content (see "Fill the template with the Figma content" below; for homepage sections that means `templates/index.json`).
- **Locales**: storefront strings live in `locales/*.json` (`en.default.json` is the source); schema labels use `t:` keys from `*.schema.json`. When you add a translation key, add it to `en.default.json` and to `ar.json`.

## Building or changing sections: existing-first ladder (mandatory)

For every section task, use the **earliest** option that safely satisfies the design. Go in order; move down one step only after showing why the current step can't do it.

1. Use the existing project section, block, snippet, setting, class, or component **as-is**.
2. Use the existing structure with **scoped CSS only**.
3. Use an **instance-specific design option** (a new " - cca" option in the section's design select, or a new `design` select with `default` / `design_1`). Don't add "Custom CSS class" text settings to sections. See §6 of the skill.
4. Add a **new schema setting beside** existing settings.
5. Add a **new schema option beside** existing options.
6. **Extend** the closest existing block or snippet safely.
7. **Duplicate** the closest block/snippet only when extension is unsafe.
8. **Carefully modify an original section** only when the change is minimal, reusable, and cannot safely be isolated another way.
9. **Duplicate an existing section** only when the same section must support a materially different structure that cannot be enabled as another instance.
10. **Create a completely new section** only as a last resort.

Rules:

- **Never duplicate a section just to edit it.** This was a repeated mistake in a previous project. Changes to an existing section go through steps 1–8.
- Before steps 7, 9 or 10 (duplicating or creating a file), list the earlier steps tried and why each one doesn't work, then **ask the user for approval**.
- "The design looks different", "it's easier", or "it keeps the original untouched" are not reasons to duplicate.
- An additive setting that defaults to blank/false changes nothing for existing instances and is almost always safer than a duplicate. Confirm that by checking every existing instance of the section in `templates/*.json` and `sections/*-group.json`.
- Never rename or delete existing setting IDs, block types, or presets. Merchant data in the template JSON depends on them.

The full workflow (Figma inspection, reporting, CSS/RTL/JS rules, validation) is in the `generate-section` skill: [.claude/skills/generate-section/SKILL.md](.claude/skills/generate-section/SKILL.md).

## Fill the template with the Figma content (mandatory)

A section task isn't done when the code works. The section instance on the page must also look like Figma. After the Liquid/CSS/JS changes, edit that instance in the page's template JSON (`templates/index.json` for the homepage, `sections/header-group.json` / `footer-group.json` for header and footer sections, and so on):

- Set its settings and block content to match Figma: text copied exactly from the Figma node, codes, links, layout and size settings, and the matching `color_scheme`.
- If the section isn't on the page yet, add it in the right place in `order`.
- Hide demo blocks Figma doesn't show with `"disabled": true` instead of deleting them.
- Edit only that one instance. Keep every other section, block, setting and order entry exactly as it is, and keep the file's CRLF line endings.
- Don't invent content. If Figma has an image or text you don't have, leave the setting for the merchant and list it in the report. Never put a Figma asset URL in the JSON.
- **Range values must be on a step.** Before writing a `range` setting's value, read its `min`/`max`/`step` from the schema: Shopify refuses to upload the file unless `(value - min) / step` is a whole number (e.g. step 2 from 50 → even numbers only; Figma 153 becomes 154). Same when adding a range setting: `min`, `max` and `default` all on the step grid, at most 101 steps.
- Tell the user to commit the JSON soon, because a save in the Theme Editor overwrites it.

Full rules are in §7a of the `generate-section` skill.

## Collection page design has no Auto Layout

The collection-page Figma frames (`1:787` desktop, `1:1081` mobile, `1:1287` mobile filter sheet) use fixed positions and sizes. Take colors, typography, radii and aspect ratios from Figma, but build the layout with flex/grid, `gap`, `%`/`fr` widths, `aspect-ratio` and `clamp()`. Never copy x/y offsets or fixed frame widths and heights into the CSS. Full rules are in §3.1 of the `generate-section` skill.

## Carousels: cards run through the gutter (mandatory)

Every section with a carousel (product / collection / card `<hdt-slider>` track) gets the edge fix: the track's `.hdt-slider__viewport` is `overflow: visible` and the section's `.hdt-container` is `overflow-x: clip`, so cards are clipped at the container's outer edge instead of disappearing behind its padding while dragging. Add the section's `cca-` class to the shared "cca carousel edges" rule in `assets/dev1.css`. Never use one-sided negative margins, `100vw`, or removed container padding for this. Not for the hero/slideshow or product galleries. Full rules are in §12a of the `generate-section` skill.

## Colors come from the theme's color schemes

Never hardcode colors. Use the color schemes in Theme settings → Colors, through the section's `color_scheme` setting and the variables they produce (`rgb(var(--color-foreground))`, `var(--hdt-btn-bg)`, …). If no scheme matches the Figma colors, add a new scheme. A per-section color setting is a last resort for colors no scheme field can express. Full rules are in §2.8a of the `generate-section` skill.

## Our CSS: `cca-` prefix, all in `assets/dev1.css`

Every CSS change for this project is additive and lives in one file, so deleting `assets/dev1.css` removes it all.

On branch `dev1`, `assets/dev1.css` replaces `main.css` for all of these rules. `main.css` stays loaded (just before `dev1.css`) for other branches, but don't add CSS to it here.

- Every new CSS class starts with `cca-` (e.g. `cca-pagination`), and every new CSS variable with `--cca-`.
- All new CSS goes in `assets/dev1.css`. Don't create per-section stylesheets, and don't edit the theme's CSS files.
- Every rule in `dev1.css` is anchored on a `cca-` class (e.g. `.cca-collection .hdt-card-product`). To restyle a theme element, add a `cca-` class to its markup and keep its `hdt-` classes.
- Group `dev1.css` by feature, with a comment header per section.
- Only values that need Liquid (settings, `section.id`) go in an inline `{% style %}` in the section, and only as `--cca-` variables that `dev1.css` uses.
- **Out-rank the native rule, don't tie it.** `dev1.css` loads in `<head>`, but most sections load their own stylesheet later in `<body>` (`hdt-icon-box.css`, `testimonials.css`, …), and `theme.css` uses nested selectors. On equal specificity the theme wins, so the override silently does nothing. When a rule overrides a property the theme also sets, find the native selector first and use a more specific one: double the scope class (`.cca-home-faq.cca-home-faq .hdt-accordion-item`) or add one more class. Use `!important` only against an `#id` rule or an inline style, with a comment saying why. Before calling it done, check the computed value in the browser.
- **Don't pad or size an element that native JS animates.** For example, `<hdt-accordion>` animates `<details>` from its `<summary>` height, so padding on `<details>` breaks it. Put spacing on the inner parts (`summary`, the content) instead.

Full rules are in §2.3a of the `generate-section` skill.

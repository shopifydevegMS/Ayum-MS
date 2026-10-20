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
- **Templates and config**: `templates/*.json` (with alternates such as `collection.left-sidebar.json`) plus `sections/*-group.json` and `config/settings_data.json` hold merchant content and settings. They are often overwritten by Shopify editor syncs, so prefer changing Liquid/schema over hand-editing these, and expect merge conflicts if you do.
- **Locales**: storefront strings live in `locales/*.json` (`en.default.json` is the source); schema labels use `t:` keys from `*.schema.json`. When you add a translation key, add it to `en.default.json` and to `ar.json`.

## Building or changing sections: existing-first ladder (mandatory)

For every section task, use the **earliest** option that safely satisfies the design. Go in order; move down one step only after showing why the current step can't do it.

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

Rules:

- **Never duplicate a section just to edit it.** This was a repeated mistake in a previous project. Changes to an existing section go through steps 1–8.
- Before steps 7, 9 or 10 (duplicating or creating a file), list the earlier steps tried and why each one doesn't work, then **ask the user for approval**.
- "The design looks different", "it's easier", or "it keeps the original untouched" are not reasons to duplicate.
- An additive setting that defaults to blank/false changes nothing for existing instances and is almost always safer than a duplicate. Confirm that by checking every existing instance of the section in `templates/*.json` and `sections/*-group.json`.
- Never rename or delete existing setting IDs, block types, or presets. Merchant data in the template JSON depends on them.

The full workflow (Figma inspection, reporting, CSS/RTL/JS rules, validation) is in the `generate-section` skill: [.claude/skills/generate-section/SKILL.md](.claude/skills/generate-section/SKILL.md).

## Collection page design has no Auto Layout

The collection-page Figma frames (`1:787` desktop, `1:1081` mobile, `1:1287` mobile filter sheet) use fixed positions and sizes. Take colors, typography, radii and aspect ratios from Figma, but build the layout with flex/grid, `gap`, `%`/`fr` widths, `aspect-ratio` and `clamp()`. Never copy x/y offsets or fixed frame widths and heights into the CSS. Full rules are in §3.1 of the `generate-section` skill.

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

Full rules are in §2.3a of the `generate-section` skill.

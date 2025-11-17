# BaseKit Base Theme

This project now uses a shared base theme named `basekit` that owns all structure (Twig, SDC components, templates, and default styles). The site theme `gravelle1` is a sub‑theme that only provides brand tokens (colors/spacing/type) and compiled CSS, keeping behavior and markup centralized.

What you get
- Single source of truth for custom blocks/SDCs and templates in `basekit`.
- Sub‑themes (e.g., `gravelle1`) override only tokens and ship their own CSS.
- Easy updates: change component logic once in `basekit`; sites pick it up.

## Layout & Key Files

- Base theme: `web/themes/custom/basekit`
  - Tokens: `web/themes/custom/basekit/scss/base/var/_var_default.scss`
  - Sass entry: `web/themes/custom/basekit/scss/styles.scss`
  - Component styles: `web/themes/custom/basekit/components/<component>/<component>.scss`
  - Libraries: `web/themes/custom/basekit/basekit.libraries.yml`
  - Templates/SDCs: `web/themes/custom/basekit/components/**`, `web/themes/custom/basekit/templates/**`

- Sub‑theme: `web/themes/custom/gravelle1`
  - Declares base: `web/themes/custom/gravelle1/gravelle1.info.yml` (`base theme: basekit`)
  - Token config: `web/themes/custom/gravelle1/scss/_tokens.scss`
  - Global SCSS: `web/themes/custom/gravelle1/scss/styles.scss`
  - Component stubs: `web/themes/custom/gravelle1/components/<component>/<component>.scss`
  - Library overrides: `web/themes/custom/gravelle1/gravelle1.info.yml`
  - Gulp build: `web/themes/custom/gravelle1/gulpfile.js`

## How Inheritance Works

- Drupal resolves templates and libraries from the sub‑theme first; if not present, it falls back to `basekit`.
- SDC Twig in `basekit` attaches `basekit/<component>` libraries.
- The sub‑theme redirects those to its own compiled CSS via `libraries-override` so the markup comes from `basekit` but styles come from `gravelle1`.

Snippet (already present): `web/themes/custom/gravelle1/gravelle1.info.yml`

  libraries-override:
    basekit/base: false
    basekit/messages: false
    basekit/pushy: false
    basekit/fonts.typography: false
    basekit/hero_announcement: gravelle1/hero_announcement
    basekit/hero_headline: gravelle1/hero_headline
    basekit/media_text: gravelle1/media_text
    basekit/quote_feature: gravelle1/quote_feature
    basekit/grid_topics: gravelle1/grid_topics
    basekit/text_rich: gravelle1/text_rich
    basekit/media_slider: gravelle1/media_slider
    basekit/image_slider: gravelle1/image_slider

This ensures no double‑loading of `basekit` CSS and routes each component’s library to sub‑theme CSS.

## Token Overrides (Sass Modules)

Base tokens live in `basekit` and use `!default`. Configure them from the sub‑theme, then import BaseKit styles.

- Sub‑theme token module: `web/themes/custom/gravelle1/scss/_tokens.scss`

  @use '../basekit/scss/base/var/var_default';
  // To override, switch to:
  // @use '../basekit/scss/base/var/var_default' with (
  //   $brand-primary: #4b7bec,
  //   $brand-accent: #ff3366,
  //   $font-body: 'Inter', system-ui,
  //   $siteg: 1.25em
  // );

- Sub‑theme global stylesheet: `web/themes/custom/gravelle1/scss/styles.scss`

  @use './_tokens';
  @use 'layout';
  @use 'navigation';
  @use 'block';
  @use 'components/button';

Order matters: configure tokens before importing anything that uses them.

## Component SCSS: Thin Stubs

Each sub‑theme component SCSS is a thin stub that pulls in tokens and the BaseKit implementation. Example: `web/themes/custom/gravelle1/components/hero_headline/hero_headline.scss`

  @use '../../scss/_tokens';
  @use '../../../basekit/components/hero_headline/hero_headline';

This keeps all component logic in `basekit`. The sub‑theme only contributes token values.

## Overriding Component SCSS (Sub‑theme)

You have three options, from least to most invasive:

- Token overrides (preferred)
  - Adjust variables in `scss/_tokens.scss`. Because BaseKit styles use tokens with `!default`, brand changes cascade without touching component SCSS.

- Extend defaults (import and add rules)
  - Keep the base defaults and add site‑specific tweaks below your import.

    // web/themes/custom/<site>/components/<component>/<component>.scss
    @use '../../scss/_tokens';
    @use '../../../basekit/components/<component>/<component>';

    // Your overrides here (selectors match BaseKit markup)
    .block-type--<component> { /* ... */ }

- Replace entirely (do not import)
  - Implement all styles from scratch; do not import BaseKit’s component SCSS. This fully replaces the look while keeping the same markup.

    // web/themes/custom/<site>/components/<component>/<component>.scss
    @use '../../scss/_tokens';
    .block-type--<component> { /* full implementation */ }

Notes
- Ensure your sub‑theme maps each BaseKit component library to your own in `<site>.info.yml` under `libraries-override` so only your compiled CSS loads.
- Gulp includePaths must allow `@use '../../../basekit/...';` to resolve. The starter already sets:
  - includePaths: `['scss', '../basekit/scss', '../basekit/components']`
- After changes, rebuild CSS and clear caches:
  - cd `web/themes/custom/<site>` && `npm run build`
  - `lando drush cr`

## Overriding Component Twig (Sub‑theme)

Drupal’s theme resolution checks the sub‑theme first, then the base theme. To override component markup:

- Full component template
  - Copy from BaseKit to the same path in your sub‑theme and edit:
    - From: `web/themes/contrib/basekit/components/<component>/<component>.twig` (or `.html.twig`)
    - To:   `web/themes/custom/<site>/components/<component>/<component>.twig`

- Partial templates
  - Copy only the partial you need:
    - From: `web/themes/contrib/basekit/components/<component>/partials/<name>.html.twig`
    - To:   `web/themes/custom/<site>/components/<component>/partials/<name>.html.twig`

- Libraries
  - Keep library names the same and use `libraries-override` in your sub‑theme to point to your CSS so the BaseKit component attaches still work.

- Clear caches after Twig changes
  - `lando drush cr`

Tips
- Prefer small partial overrides to keep diffs minimal and ease BaseKit updates.
- If you add new BEM elements/modifiers in Twig, reflect them in your component SCSS so selectors stay in sync.

## Build Pipeline

Sub‑theme Gulp has been updated so imports from `basekit` resolve cleanly.

- `web/themes/custom/gravelle1/gulpfile.js`
  - includePaths: `['scss', '../basekit/scss', '../basekit/components']`

Build commands:
- cd `web/themes/custom/gravelle1` && `lando npm install` && `lando gulp`
- Clear caches after changes to Twig/PHP/config: `lando drush cr`

## Enabling the Base Theme

If you see “MissingThemeDependencyException: Base theme basekit has not been installed”, enable it once:
- `lando drush then basekit -y`
- `lando drush cr`

`gravelle1` remains the default site theme; it simply inherits from `basekit`.

## Adding a New Site/Theme

1) Option A — Scaffold from the starter (recommended)

- Run the scaffold script (from project root). BaseKit installs under `web/themes/contrib` by default:

  bash web/themes/contrib/basekit/subtheme-starter/scripts/new-subtheme.sh mysite "My Site"

- Build and enable:

  cd web/themes/custom/mysite && lando npm install && lando gulp
  lando drush then mysite -y && lando drush cset system.theme default mysite -y && lando drush cr

2) Option B — Manual sub‑theme

Create a new sub‑theme folder under `web/themes/custom/<site>` with:
- `<site>.info.yml` containing `base theme: basekit`
- `<site>.libraries.yml` with a `base` library pointing to your compiled CSS
- `scss/_tokens.scss` (configure tokens) and `scss/styles.scss` (import tokens, then BaseKit styles/your additions)
- `components/<component>/<component>.scss` thin stubs (as above)

2) Update `gulpfile.js` includePaths similarly so it can import from `../basekit/...`.

3) Build and clear caches.

## How This Plays with SDCs and Custom Blocks

- `basekit` Twig attaches component libraries (e.g., `basekit/hero_headline`).
- `gravelle1` uses `libraries-override` so those libraries load the sub‑theme’s compiled CSS.
- All block bundles’ visual logic lives in `basekit` SCSS; per‑site look comes from tokens.

## Troubleshooting

- Duplicate styles: ensure `libraries-override` in `gravelle1.info.yml` disables `basekit/base` (and related) and maps each component library.
- Sass cannot find BaseKit paths: check `@use '../basekit/scss/base/var/var_default'` and gulp `includePaths`.
- Base theme not installed: run `drush then basekit -y` and `drush cr`.

## Next Steps (Recipes)

To reuse and update configuration (block types, view modes, LB layouts, image styles, etc.) across sites, use Recipes. See `recipes/` in this repo and the aggregate recipe in `recipes/site/recipe.yml`. Apply locally with:

- `php core/scripts/drupal recipe recipes/site`
- `lando drush cim -y && lando drush cr`

This lets you compose a base recipe plus optional features (e.g., Article, individual custom block bundles) per site.

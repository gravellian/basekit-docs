# BaseKit Base Theme

This project now uses a shared base theme named `basekit` that owns all structure (Twig, SDC components, templates, and default styles). Site themes like `gravelle1` or `jsg1` are sub‑themes that primarily provide brand tokens (colors/spacing/type) and additional CSS, keeping behavior and markup centralized in BaseKit.

What you get
- Single source of truth for custom blocks/SDCs and templates in `basekit`.
- Sub‑themes (e.g., `gravelle1`) override only tokens and ship their own CSS.
- Easy updates: change component logic once in `basekit`; sites pick it up.

## Layout & Key Files

- Base theme: `basekit`
  - Source of truth: separate repo (e.g., `../basekit` in this workspace).
  - Installed in the site via Composer (typically under `web/themes/contrib/basekit`; do not edit the mirrored copy).
  - Tokens: `basekit/scss/base/var/_var_default.scss` (in the BaseKit repo).
  - Font defaults: `basekit/scss/layout/_page-format.scss` (`:root --font-*`).
  - Sass entry: `basekit/scss/styles.scss`.
  - Component styles: `basekit/components/<component>/<component>.scss`.
  - Libraries: `basekit/basekit.libraries.yml`.
  - Templates/SDCs: `basekit/components/**`, `basekit/templates/**`.

- Sub‑theme example: `web/themes/custom/gravelle1`
  - Declares base: `web/themes/custom/gravelle1/gravelle1.info.yml` (`base theme: basekit`)
  - Token config: `web/themes/custom/gravelle1/scss/_tokens.scss`
  - Global SCSS: `web/themes/custom/gravelle1/scss/styles.scss`
  - Component stubs: `web/themes/custom/gravelle1/components/<component>/<component>.scss`
  - Optional library overrides for individual components: `web/themes/custom/gravelle1/gravelle1.info.yml`
  - Gulp build: `web/themes/custom/gravelle1/gulpfile.js`

## How Inheritance Works

- Drupal resolves templates and libraries from the sub‑theme first; if not present, it falls back to `basekit`.
- SDC Twig in `basekit` attaches `basekit/<component>` libraries.
- By default, the sub‑theme loads `basekit/base` first, then its own `base` library to layer custom styling on top.
- For components where the sub‑theme wants fully custom CSS, it can override a specific BaseKit library (e.g. `basekit/hero_headline`) and map it to a sub‑theme library.

Snippet (current pattern): `web/themes/custom/gravelle1/gravelle1.info.yml`

```yaml
libraries:
  # Load BaseKit base CSS first, then layer gravelle1 overrides.
  - basekit/base
  - gravelle1/base

# When you port a component to the sub-theme and ship your own CSS,
# you can opt into overrides selectively (leave commented until ready):
# libraries-override:
#   basekit/messages: false
#   basekit/pushy: false
#   basekit/fonts.typography: false
#   # Map BaseKit component libraries to sub-theme outputs if you compile per-component CSS
#   # basekit/hero_headline: gravelle1/hero_headline
#   # basekit/hero_announcement: gravelle1/hero_announcement
```

## Token Overrides (Sass Modules)

Base tokens live in `basekit` and use `!default`. Configure them from the sub‑theme, then import BaseKit styles. (Font families are CSS variables, not Sass tokens.)

- Sub‑theme token module: `web/themes/custom/gravelle1/scss/_tokens.scss`

  ```scss
  @use 'base/var/var_default';
  // Or, to override BaseKit tokens from the sub‑theme:
  // @use 'base/var/var_default' with (
  //   $brand-primary: #4b7bec,
  //   $brand-accent: #ff3366,
  //   $brand-neutral: #8a7f76,
  //   $text-base: #eae6df,
  //   $siteg: 1.25em
  // );
  ```

- Sub‑theme global stylesheet: `web/themes/custom/gravelle1/scss/styles.scss`

  ```scss
  @use './_tokens';
  @use 'layout';
  @use 'navigation';
  @use 'block';
  @use 'components/button';
  ```

Order matters: configure tokens before importing anything that uses them.

## Overriding BaseKit Templates and Styling

### Overriding BaseKit Templates (Twig)

Drupal’s theme resolution checks the sub‑theme first, then the base theme. To override component markup:

- Full component template
  - Copy from BaseKit to the same path in your sub‑theme and edit:
    - From: `web/themes/contrib/basekit/components/<component>/<component>.twig` (or `.html.twig`)
    - To:   `web/themes/custom/<site>/components/<component>/<component>.twig`

- Partial templates
  - Copy only the partial you need:
    - From: `web/themes/contrib/basekit/components/<component>/partials/<name>.html.twig`
    - To:   `web/themes/custom/<site>/components/<component>/partials/<name>.html.twig`

- Clear caches after Twig changes:
  - `lando drush cr`

Prefer small partial overrides to keep diffs minimal and ease BaseKit updates.

### Overriding Global Styling (layout, typography)

- Configure brand tokens in the sub‑theme `_tokens.scss` as shown above.
- In any sub‑theme SCSS file that needs BaseKit mixins/variables:

  ```scss
  @use '../_tokens';
  @use 'base' as base;

  body {
    font-family: var(--font-body);
    color: base.$color-text;
  }
  ```

This keeps all global layout and typography logic in BaseKit while allowing per‑site branding via tokens and CSS variables.

### Overriding Component Styling (SCSS)

Each sub‑theme component SCSS is a thin stub that pulls in tokens and the BaseKit implementation.

Example: `web/themes/custom/gravelle1/components/hero_headline/hero_headline.scss`

```scss
@use '../../scss/_tokens';
@use 'base' as base;
@use 'basekit/components/hero_headline/hero_headline';

.block-type--hero_headline.block-view-mode--default {
  // site‑specific tweaks
  background-color: base.$surface-2;
}
```

You have three options, from least to most invasive:

- **Token overrides (preferred)**
  - Adjust variables in `scss/_tokens.scss`. Because BaseKit styles use tokens with `!default`, brand changes cascade without touching component SCSS.

- **Extend defaults (import and add rules)**
  - Keep the base defaults and add site‑specific tweaks below your import, as in the example above.
  - BaseKit’s component CSS still loads (via `basekit/base`), and your rules win by being at least as specific.

- **Replace entirely (SCSS + optional library override)**
  - Implement all styles from scratch; optionally skip importing BaseKit’s component SCSS.

    ```scss
    // web/themes/custom/<site>/components/<component>/<component>.scss
    @use '../../scss/_tokens';
    @use 'base' as base;

    .block-type--<component> { /* full implementation */ }
    ```

  - If you want your CSS to replace the BaseKit component CSS for that library, map it in `<site>.info.yml`:

    ```yaml
    libraries-override:
      basekit/<component>: <site>/<component>
    ```

Notes
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
  - includePaths: `['scss', '../../contrib/basekit/scss', '../../contrib/basekit/components']`

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

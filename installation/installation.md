# BaseKit Distribution Installation

This guide installs a fresh site using the BaseKit base theme + Recipes so you can reuse structure across projects and keep brand/UI overrides in a sub‑theme.

## What’s Included

- Base theme: `gravellian/basekit` (Twig, SDCs, SCSS, libraries)
- Recipes: `gravellian/basekit-recipe`
  - `recipes/basekit` — baseline modules + shared config (image/media/view modes)
  - Feature recipes (opt‑in): `recipes/article`, `recipes/blocks/*`
  - `recipes/site` — aggregate of base + features

## Prerequisites

- PHP 8.1+ and Composer
- Database + web server (or Lando/DDEV)
- SSH access to GitHub `gravellian` repos (added SSH key)

## 1) Create a Drupal Project

You can start from the official template or a company template.

```
composer create-project drupal/recommended-project mysite
cd mysite
```

Optional (Lando):

```
lando start
```

## 2) Require BaseKit Theme + Recipes

Add VCS repos and require packages in `composer.json` (merge these into existing sections; do not replace the file). Do not set installer-paths for recipes; core's recipe-unpack plugin handles unpacking into `recipes/`.

### 2a) composer.json edits (step-by-step)

1) repositories (append these entries; keep the Drupal packages repo):

```
"repositories": [
  { "type": "composer", "url": "https://packages.drupal.org/8" },
  { "type": "vcs", "url": "git@github.com:gravellian/basekit.git", "no-api": true },
  { "type": "vcs", "url": "git@github.com:gravellian/basekit-recipe.git", "no-api": true }
]
```

2) extra.installer-paths (edit only the custom themes line; keep all your existing paths):

```
"extra": {
  "installer-paths": {
    "web/themes/custom/{$name}": [
      "type:drupal-theme",
      "gravellian/basekit"
    ]
  }
}
```

Notes:
- If your project uses `type:drupal-custom-theme` already, keep that type instead of `type:drupal-theme` and still add `"gravellian/basekit"` to the array.
- Do not add any installer-path for `drupal-recipe`.

3) config.allow-plugins (ensure these are allowed):

```
"config": {
  "allow-plugins": {
    "composer/installers": true,
    "drupal/core-composer-scaffold": true,
    "drupal/core-project-message": true,
    "drupal/core-recipe-unpack": true
  }
}
```

composer.json additions:

```
"repositories": [
  { "type": "composer", "url": "https://packages.drupal.org/8" },
  { "type": "vcs", "url": "git@github.com:gravellian/basekit.git", "no-api": true },
  { "type": "vcs", "url": "git@github.com:gravellian/basekit-recipe.git", "no-api": true }
],
"extra": {
  "installer-paths": {
    "web/themes/custom/{$name}": [
      "type:drupal-theme",
      "gravellian/basekit"
    ]
  }
},
"config": {
  "allow-plugins": {
    "composer/installers": true,
    "drupal/core-composer-scaffold": true,
    "drupal/core-project-message": true,
    "drupal/core-recipe-unpack": true
  }
}
```

Install packages:

```
composer require gravellian/basekit:dev-main gravellian/basekit-recipe:dev-main -W
```

## 3) Create a Sub‑Theme (recommended)

Create a minimal sub‑theme so site branding stays separate from BaseKit.

### Option A — Scaffold from the starter (quickest)

From project root, generate a ready-to-edit sub-theme (BaseKit installs under `web/themes/contrib` by default):

```
bash web/themes/contrib/basekit/subtheme-starter/scripts/new-subtheme.sh mysite "My Site"
```

Build and enable it:

```
cd web/themes/custom/mysite && lando npm install && lando gulp
lando drush then mysite -y && lando drush cset system.theme default mysite -y && lando drush cr
```

### Option B — Manual sub‑theme

Create files under `web/themes/custom/<subtheme>/`:

- `<subtheme>.info.yml`:

```
name: <Subtheme>
type: theme
base theme: basekit
libraries:
  - <subtheme>/base
libraries-override:
  basekit/base: false
  basekit/messages: false
  basekit/pushy: false
  basekit/fonts.typography: false
  # Map BaseKit component libraries to sub‑theme outputs
  basekit/hero_headline: <subtheme>/hero_headline
  basekit/hero_announcement: <subtheme>/hero_announcement
  # …repeat for components you use
```

- `<subtheme>.libraries.yml` (point to compiled CSS):

```
<subtheme>/base:
  css:
    theme:
      css/styles.css: {}
<subtheme>/hero_headline:
  css:
    theme:
      components/hero_headline/hero_headline.css: {}
```

- `scss/_tokens.scss` and `scss/styles.scss`:

```
// _tokens.scss
@use '../basekit/scss/base/var/var_default';
// To override:
// @use '../basekit/scss/base/var/var_default' with (
//   $brand-primary: #4b7bec,
//   $font-body: 'Inter', system-ui
// );

// styles.scss
@use './_tokens';
@use 'layout';
@use 'navigation';
@use 'block';
```

- Component stubs (per SDC) importing BaseKit implementation:

```
// components/hero_headline/hero_headline.scss
@use '../../scss/_tokens';
@use '../../../basekit/components/hero_headline/hero_headline';
```

Build CSS (Lando):

```
cd web/themes/custom/<subtheme>
lando npm install
lando gulp
```

## 4) Enable Theme(s)

Enable BaseKit (once), then set your sub‑theme default (optional now or later):

```
lando drush then basekit -y
lando drush then <subtheme> -y
lando drush cset system.theme default <subtheme> -y
```

## 5) Apply Recipes (Config)

Apply the aggregate recipe (base + features) or start with base only.

```
# Aggregate
php core/scripts/drupal recipe gravellian/basekit-recipe:recipes/site

# Or base only
php core/scripts/drupal recipe gravellian/basekit-recipe:recipes/basekit

lando drush cim -y && lando drush updb -y && lando drush cr
```

You can comment/uncomment features in your own aggregator or apply individual feature recipes directly (e.g., `recipes/article`, `recipes/blocks/hero_headline`).

## 6) Verify

- Media types, responsive image styles, and image styles exist
- Custom block types you enabled via recipes are available
- BaseKit templates/SDCs render; sub‑theme CSS is loaded (inspect library names)

## Updating Later

- Update packages:

```
composer update gravellian/basekit gravellian/basekit-recipe -W
```

## (Optional) Install Docs via Composer

Ship the distribution’s documentation into your project’s `docs/` so teams always have the latest guides alongside the code.

1) Add the docs repo (if private):

```
composer config repositories.basekit-docs vcs git@github.com:gravellian/basekit-docs.git
```

2) Require the docs package and installers extender:

```
composer require oomphinc/composer-installers-extender:^2.0 gravellian/basekit-docs:dev-main -W
```

3) In `composer.json`, add installer types and a path for docs:

```
"extra": {
  "installer-types": ["gravellian-docs"],
  "installer-paths": {
    "docs/": ["type:gravellian-docs"]
  }
}
```

Update later with:

```
composer update gravellian/basekit-docs -W
```

- Re‑apply recipes and rebuild:

```
php core/scripts/drupal recipe gravellian/basekit-recipe:recipes/site
lando drush cim -y && lando drush updb -y && lando drush cr
cd web/themes/custom/<subtheme> && lando gulp
```

## Notes

- Recipes don’t install Composer deps; require BaseKit via Composer and enable it.
- Do not map `drupal-recipe` under `extra.installer-paths`; let `drupal/core-recipe-unpack` unpack packages into `recipes/` automatically.
- Prefer Layout Builder placements over theme‑region blocks. If you ship region blocks in recipes, add `config_actions` to map `theme:` to your sub‑theme.

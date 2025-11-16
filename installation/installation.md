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

## 0) Quick Start: Composer First, Then Lando (Drupal 11)

Starting with host Composer keeps tools simple and avoids depending on Composer inside Lando images. These steps assume an empty directory.

```
# 1. Create project directory and enter it
mkdir dev.justsomeguypainting.com2
cd dev.justsomeguypainting.com2

# 2. Create a Drupal 11 project (web as webroot)
composer create-project drupal/recommended-project:11.x .

# 3. Add BaseKit repositories to composer.json (see section 2a below),
#    then require Drush + BaseKit packages
composer require drush/drush
composer require gravellian/basekit:dev-main gravellian/basekit-recipe:dev-main -W

# 4. Initialize a Lando Drupal 11 app for this codebase
lando init \
  --source cwd \
  --recipe drupal11 \
  --webroot web \
  --name dev.justsomeguypainting.com2

# 5. Start the environment
lando start

# 6. Install Drupal (adjust DB URL if you changed defaults)
lando drush site:install --db-url=mysql://drupal11:drupal11@database/drupal11 -y

# 7. Show connection info (URLs, DB creds, etc.)
lando info
```

## 1) Create a Drupal Project (non‑Lando variant)

If you are not using Lando, you can still start from the official template or a company template:

```
composer create-project drupal/recommended-project mysite
cd mysite
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

2) extra.installer-paths (optional — for most projects you can keep the defaults that install BaseKit under `web/themes/contrib`; only change this if you intentionally want a different layout):

```
"extra": {
  "installer-paths": {
    "web/themes/custom/{$name}": [
      "type:drupal-custom-theme"
    ]
  }
}
```

Notes:
- For a standard `drupal/recommended-project`, you do not need to change installer paths for BaseKit; it will install to `web/themes/contrib/basekit` by default.
- If your project uses `type:drupal-custom-theme` already for custom themes, keep that type as-is.
- Do not add any installer-path for `drupal-recipe`; let core unpack recipes into `recipes/`.

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
# Build theme assets (recommended: on the host)
cd web/themes/custom/mysite
npm install
npm run build   # or: npx gulp

# Enable BaseKit + sub-theme inside Lando
cd ../../../..
lando drush then basekit -y
lando drush then mysite -y
lando drush cset system.theme default mysite -y
lando drush cr
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
@use '../../contrib/basekit/scss/base/var/var_default';
// To override:
// @use '../../contrib/basekit/scss/base/var/var_default' with (
//   $brand-primary: #4b7bec,
//   $font-body: 'Inter', system-ui
// );

// styles.scss
@use './_tokens';
@use 'layout';
@use 'navigation';
@use 'block';
```

If you keep these imports, make sure to define the corresponding partials:

```
// scss/_layout.scss
@use './_tokens';

// scss/_navigation.scss
@use './_tokens';

// scss/_block.scss
@use './_tokens';
```

- Component stubs (per SDC) importing BaseKit implementation:

```
// components/hero_headline/hero_headline.scss
@use '../../scss/_tokens';
@use '../../../basekit/components/hero_headline/hero_headline';
```

Build CSS (typically on the host):

```
cd web/themes/custom/<subtheme>
npm install
npm run build   # or: npx gulp
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
# Aggregate (Lando + recommended-project webroot)
lando php web/core/scripts/drupal recipe recipes/basekit-recipe/recipes/site

# Or base only
lando php web/core/scripts/drupal recipe recipes/basekit-recipe/recipes/basekit

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
lando php web/core/scripts/drupal recipe recipes/basekit-recipe/recipes/site
lando drush cim -y && lando drush updb -y && lando drush cr
cd web/themes/custom/<subtheme> && npm run build
```

## Notes

- Recipes don’t install Composer deps; require BaseKit via Composer and enable it.
- Do not map `drupal-recipe` under `extra.installer-paths`; let `drupal/core-recipe-unpack` unpack packages into `recipes/` automatically.
- Prefer Layout Builder placements over theme‑region blocks. If you ship region blocks in recipes, add `config_actions` to map `theme:` to your sub‑theme.
- If you see an error about a missing `PluginExists` validation constraint when applying recipes, ensure any module providing that constraint is required and enabled before running the `recipe` command.

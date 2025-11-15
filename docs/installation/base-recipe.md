# BaseKit Recipes

Use Recipes to install and update shared configuration (modules enabled, image styles, media types, content models, block bundles, Layout Builder, etc.) across multiple sites. The canonical recipes are published at `gravellian/basekit-recipe`.

## What’s Included

- Base recipe: `recipes/basekit`
  - Enables core/contrib modules (media, responsive images, etc.)
  - Imports shared config (image styles, media types, view modes)
- Feature recipes (enable per-site as needed):
  - `recipes/article` — Article content model + roles + views
  - `recipes/blocks/hero_headline` — custom block bundle
  - `recipes/blocks/hero_announcement` — custom block bundle
- Aggregator: `recipes/site` — composes base + features

## Prerequisites

- Drupal 11 project with Drush available
- Composer access to the private GitHub repo (SSH key for `gravellian`)

## Install via Composer

1) Add the VCS repository and require the package in your site’s `composer.json`:

```
"repositories": [
  { "type": "composer", "url": "https://packages.drupal.org/8" },
  { "type": "vcs", "url": "git@github.com:gravellian/basekit-recipe.git", "no-api": true }
],
"require": {
  "gravellian/basekit-recipe": "dev-main"
}
```

2) Install/update:

```
composer update gravellian/basekit-recipe -W
```

## Apply a Recipe

Preferred (by package path):

```
# Base only
php core/scripts/drupal recipe gravellian/basekit-recipe:recipes/basekit

# Aggregate (base + features)
php core/scripts/drupal recipe gravellian/basekit-recipe:recipes/site

# Individual feature
php core/scripts/drupal recipe gravellian/basekit-recipe:recipes/article
php core/scripts/drupal recipe gravellian/basekit-recipe:recipes/blocks/hero_headline
```

Finalize:

```
lando drush cim -y
lando drush updb -y
lando drush cr
```

## Updating Sites Later

1) Pull latest recipes:

```
composer update gravellian/basekit-recipe -W
```

2) Re‑apply the same recipe(s):

```
php core/scripts/drupal recipe gravellian/basekit-recipe:recipes/site
```

3) Import + rebuild:

```
lando drush cim -y && lando drush updb -y && lando drush cr
```

If theme tokens or CSS changed in your sub‑theme, rebuild:

```
cd web/themes/custom/<subtheme> && lando gulp
```

## Per‑Site Overrides

- Keep recipe‑owned YAML out of the site’s `config/sync/` to avoid drift.
- Store site‑specific differences in `config/sync/` (or a Config Split).
- Use Config Actions in the recipe for values that must vary per site (e.g., mapping theme on `block.block.*`). Example action snippet in `recipes/basekit/recipe.yml`:

```
config:
  import:
    - recipes/basekit/config
  actions:
    - config: block.block.gravelle1_main_menu
      op: set
      path: theme
      value: gravelle1
```

Tip: Prefer Layout Builder placements over theme‑region blocks to avoid `theme:` coupling.

## Alternative: Direct Import from Vendor

For deterministic, scriptable imports you can import directly from the vendor path:

```
lando drush cim -y --source=vendor/gravellian/basekit-recipe/recipes/basekit/config
```

Pair with Config Actions (or a follow‑up import) for per‑site tweaks; re‑apply the recipe periodically to keep actions in effect.

## Troubleshooting

- “Cannot find recipe path”: ensure the Composer repo and require are present and you used the `vendor:recipes/<name>` path.
- “Changes not taking effect”: re‑apply the recipe, then `drush cim`. Some removals require delete actions or manual cleanup.
- “Blocks didn’t appear”: if you rely on theme‑region blocks, ensure `config_actions` set the `theme` to your sub‑theme, or use Layout Builder.

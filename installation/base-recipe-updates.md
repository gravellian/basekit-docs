# Updating BaseKit Sites (Theme, Recipes, Docs)

This quick‑reference covers how to pull in updates to your BaseKit base theme, recipes, and docs, then apply them to a site running in Lando.

## Prerequisites
- Lando started for the project (`lando start`).
- Composer can access your VCS repos for:
  - `gravellian/basekit` (base theme)
  - `gravellian/basekit-recipe` (recipes)
  - `gravellian/basekit-docs` (optional docs package)
- Your site’s custom theme build tools installed (see project README).

## Typical Update Flow
1) Update Composer packages
- Update theme + recipes (and docs if installed):
  - `composer update gravellian/basekit gravellian/basekit-recipe -W`
  - If docs are installed: `composer update gravellian/basekit-docs -W`

2) Re‑apply recipes (aggregate)
- Use the packaged aggregate to ensure all BaseKit config (view modes, image styles, block types, etc.) is present:
  - `lando php web/core/scripts/drupal recipe recipes/basekit-recipe/recipes/site`

3) Import config, run DB updates, rebuild cache
- `lando drush cim -y`
- `lando drush updb -y`
- `lando drush cr`

4) Rebuild your site theme assets
- From your custom theme directory (example below uses `gravelle1`):
  - `cd web/themes/custom/gravelle1 && lando gulp`

5) Verify key config
- Media view modes (expect BaseKit `main_*` variants): Configuration → Display modes → View modes → Media.
- Image styles and responsive image styles are present.
- Custom block types provided by recipes appear and render.

6) Export and commit (if the site repo tracks config)
- Review pending changes: `lando drush cst` (or `lando drush cex --diff`)
- Export: `lando drush cex -y`
- Commit changes with clear messages, e.g.:
  - `config: import BaseKit recipe updates`
  - `theme: compile SCSS after BaseKit update`

## Applying Individual Recipe Sets (optional)
If you only need specific recipe changes (e.g., base + one feature), apply and import just those directories:
- Base kit config: `lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/basekit/config`
- Article feature: `lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/article/config`
- Blocks (examples):
  - `lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/blocks/hero_headline/config`
  - `lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/blocks/hero_announcement/config`
- Finish with: `lando drush cr`

## Troubleshooting
- Media `main_*` view modes missing after update
  - Re‑apply the aggregate recipe: `lando php web/core/scripts/drupal recipe recipes/basekit-recipe/recipes/site`
  - Import config: `lando drush cim -y && lando drush cr`
- Drush `cex` shows deletes you don’t expect
  - Those items aren’t in active config. Import from the relevant recipe directory (see above), then export again.
- Theme compile errors after update
  - Ensure your custom theme’s Sass `includePaths` point to the installed BaseKit theme and rerun `lando gulp`.

## One‑Liner (full refresh)
For a fast, full refresh after you’ve committed updates to BaseKit repos:

```
composer update gravellian/basekit gravellian/basekit-recipe gravellian/basekit-docs -W && \
lando php web/core/scripts/drupal recipe recipes/basekit-recipe/recipes/site && \
lando drush cim -y && lando drush updb -y && lando drush cr && \
(cd web/themes/custom/gravelle1 && lando gulp)
```

Adjust the theme path in the last line for your site’s custom theme.

# BaseKit Project (Composer template)

Install a complete BaseKit distro **using only** the `gravellian/basekit-project` Composer template. The project includes the BaseKit base theme, BaseKit recipes, docs, helper scripts, and a bundled sub-theme (`basekit_site`). Following these steps installs Drupal, enables the themes, and imports all recipe config so menus, branding, breadcrumbs, manage menu, sidebar/main menus, and account menu are placed automatically on `basekit_site`. Powered-by and search blocks ship disabled.

## Prerequisites
- PHP 8.1+, Composer, and Lando.
- SSH access to `gravellian` repos if private.
- Optional local workspaces: `../basekit`, `../basekit-recipe`, `../basekit-docs` for path repos.

## Install steps (BaseKit project only)
```bash
# 1) Create the project (use a path repo if developing locally)
mkdir dev.basekitsite.com1
cd dev.basekitsite.com1
# If you have the workspace repos locally, prefer the path mirror to avoid Packagist latency:
# composer create-project gravellian/basekit-project:dev-main . --repository='{"type":"path","url":"../basekit-project","options":{"symlink":false}}' --stability dev
composer create-project gravellian/basekit-project:dev-main .

# 2) Start Lando
lando start

# 3) Install Drupal (admin credentials are printed)
lando drush site:install --db-url=mysql://drupal11:drupal11@database/drupal11 -y

# 4) Enable all required contrib (includes background image formatter + superfish + paragraphs, etc.)
lando drush en -y block block_content field text image media media_library responsive_image views editor rest workflows node taxonomy layout_builder focal_point image_effects inline_svg crop paragraphs entity_reference_revisions layout_builder_styles layout_builder_modal easy_breadcrumb bg_image_formatter responsive_bg_image_formatter eva profile token superfish svg_image swiper_formatter admin_toolbar admin_toolbar_tools

# 5) Enable BaseKit themes and set the subtheme as default/admin
lando drush theme:install basekit basekit_site
lando drush cset system.theme default basekit_site -y
lando drush cset system.theme admin basekit_site -y

# 6) Import recipes in this order (to satisfy dependencies)
lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/paragraphs_base/config
lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/blocks_hero_headline/config
lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/blocks_hero_announcement/config
lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/blocks_common/config
lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/pages/config
lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/basekit/config
lando drush en -y prism
lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/article/config
lando drush cim -y --partial --source=/app/recipes/basekit-recipe/recipes/profile_author/config

# 7) Build theme assets
cd web/themes/custom/basekit_site
npm install
npm run build   # or: npx gulp
cd ../../../..

# 8) Clear caches
lando drush cr
```

What you get
- BaseKit theme at `web/themes/custom/basekit`.
- Bundled sub‑theme `web/themes/custom/basekit_site` set as default/admin.
- BaseKit recipes applied (blocks, menus, content types, media, etc.).
- Blocks for branding (logo only), breadcrumbs, header main menu, manage menu, sidebar/mobile main menu, footer menu, and account are placed on `basekit_site` via config; powered-by and search blocks ship disabled.

## Helper scripts (bundled)
- `scripts/recipes-apply.lando.sh`: inside Lando (`lando recipes-apply`), reapplies the aggregate recipe (`/app/recipes/basekit-recipe/recipes/site` by default) then runs `drush cim`, `drush updb`, `drush cr`.
- `scripts/update-basekit_host.sh`: on the host, updates BaseKit, recipes, and docs via Composer and, if Lando is available, reapplies recipes and rebuilds caches.

Make scripts executable after creation:
```bash
chmod +x scripts/*.sh
```

## Notes
- The project template already requires all contrib dependencies needed by the recipes (media, paragraphs, admin_toolbar, etc.).
- If `drupal recipe` inside Lando throws a `PluginExists` error, use the fallback `lando drush cim -y && lando drush updb -y && lando drush cr` against the unpacked recipe config (already under `recipes/`).
- When developing BaseKit locally, re-run `composer update gravellian/basekit gravellian/basekit-recipe -W` and `lando recipes-apply` to mirror changes. 


## Developer Notes
- To begin theming the `basekit_site` subtheme, run `lando npm install` then `lando gulp compile` (or `lando gulp watch`).
- The project includes composer-merge-plugin and composer-patches. Put overrides in `composer.local.json` (ignored) and patches in `patches/composer.patches.json`.
- Recommended local override when you keep workspace clones (`../basekit*`) alongside the site:

  ```
  {
    "repositories": {
      "basekit":        { "type": "path", "url": "../basekit",        "options": { "symlink": true } },
      "basekit-recipe": { "type": "path", "url": "../basekit-recipe", "options": { "symlink": true } },
      "basekit-docs":   { "type": "path", "url": "../basekit-docs",   "options": { "symlink": true } }
    },
    "config": {
      "allow-plugins": {
        "composer/installers": true,
        "drupal/core-composer-scaffold": true,
        "drupal/core-project-message": true,
        "drupal/core-recipe-unpack": true,
        "oomphinc/composer-installers-extender": true,
        "phpstan/extension-installer": true,
        "dealerdirect/phpcodesniffer-composer-installer": true,
        "php-http/discovery": true,
        "php-tuf/composer-integration": true
      }
    }
  }
  ```

  Tips:
  - Keep `composer.local.json` and `composer.local.lock` out of Git. Seed the lock with `cp composer.lock composer.local.lock` so partial updates work.
  - Mount the workspace folders into Lando (see `.lando.yml` example in the BaseKit development guide) so the symlinks resolve inside containers.
  - After editing BaseKit/recipe/docs, run `./scripts/site-task.sh getBase` (host) to copy the latest files, apply recipes, and rebuild caches/theme assets.

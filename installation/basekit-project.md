# BaseKit Project (Composer template)

Install a complete BaseKit distro **using only** the `gravellian/basekit-project` Composer template. The project includes the BaseKit base theme, BaseKit recipes, docs, helper scripts, and a bundled sub‑theme (`basekit_site`). When you follow these steps, menus, branding, and breadcrumbs are placed on `basekit_site`.

## Prerequisites
- PHP 8.1+, Composer, and Lando.
- SSH access to `gravellian` repos if private.
- Optional local workspaces: `../basekit`, `../basekit-recipe`, `../basekit-docs` for path repos.

## Install steps (BaseKit project only)
```bash
# 1) Create the project
mkdir dev.basekitsite.com1
cd dev.basekitsite.com1
composer create-project gravellian/basekit-project:dev-main .

# 2) (Optional) use local path repos for workspace development
# composer config repositories.basekit-path path ../basekit '{"options":{"symlink":false}}'
# composer config repositories.basekit-recipe-path path ../basekit-recipe '{"options":{"symlink":false}}'
# composer config repositories.basekit-docs-path path ../basekit-docs '{"options":{"symlink":false}}'
# composer update gravellian/basekit gravellian/basekit-recipe gravellian/basekit-docs -W

# 3) Start Lando
lando start

# 4) Install Drupal
lando drush site:install --db-url=mysql://drupal11:drupal11@database/drupal11 -y

# 5) Apply the BaseKit aggregate recipe (config + modules)
lando php web/core/scripts/drupal recipe recipes/basekit-recipe/recipes/site || \
  lando drush cim -y && lando drush updb -y

# 6) Ensure the bundled sub-theme is enabled and default (basekit_setup also sets this)
lando drush then basekit_site -y
lando drush cset system.theme default basekit_site -y
lando drush cset system.theme admin basekit_site -y

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
- `basekit_setup` module places Manage, branding, breadcrumb, and menus into regions on the default theme.

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

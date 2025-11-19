# BaseKit Project

`gravellian/basekit-project` is the Composer project template that glues the
BaseKit theme + BaseKit recipes + a site‑level toolkit together. New sites
should be created from this project so they all start with the same modules,
recipes, and sub‑theme wiring.

This document describes how to install a site from `basekit-project` and how
it relates to the BaseKit + Recipe repos used in this workspace
(`../basekit`, `../basekit-recipe`).

## 1. Project Layout

BaseKit lives in separate repos so you can re‑use it across many sites:

- Base theme repo: `../basekit`
  - Drupal theme: Twig, SDC components, SCSS, breakpoints.
- Recipe repo: `../basekit-recipe`
  - Drupal recipes: BaseKit, Article, Blocks, etc (config + module enables).
- Project template: `../basekit-project`
  - Composer project: requires `gravellian/basekit`, `gravellian/basekit-recipe`,
    and all contrib modules that BaseKit config depends on (editor, media, etc.).
  - Includes Lando config, Drush tooling, and a `web/` docroot (Drupal
    recommended‑project).

Sites like `dev.gravellecreative.com3` and `dev.justsomeguypainting.com2` are
instances created from this project and then themed via a sub‑theme
(`gravelle1`, `jsg1`).

## 2. Composer Dependencies

`basekit-project/composer.json` is responsible for requiring all code that the
recipes and the BaseKit theme assume exist. At a minimum, it should require:

```jsonc
"require": {
  "gravellian/basekit": "dev-main",
  "gravellian/basekit-recipe": "dev-main",

  // Editor + text formats
  "drupal/editor": "^1",
  "drupal/ckeditor5": "^1",

  // Media + images
  "drupal/media_library": "^1",
  "drupal/focal_point": "^2",
  "drupal/responsive_image": "^1",
  "drupal/image_effects": "^4",
  "drupal/svg_image": "^3",
  "drupal/inline_svg": "^1",
  "drupal/crop": "^2",
  "drupal/file_mdm": "^3",
  "drupal/file_mdm_exif": "^3",
  "drupal/file_mdm_font": "^3",

  // Admin UX
  "drupal/admin_toolbar": "^3",
  "drupal/admin_toolbar_tools": "^3",

  // Navigation / patterns
  "drupal/paragraphs": "^1",
  "drupal/superfish": "^1",
  "drupal/eva": "^3",
  "drupal/swiper_formatter": "^2",

  // Config + dev
  "drupal/config_split": "^2",
  "drupal/prism": "^2",
  "drupal/profile": "^1",
  "drupal/token": "^1"
}
```

In this workspace, `dev.gravellecreative.com3/composer.json` is treated as the
canonical reference for this list. The `basekit-project` Composer file should
closely mirror it so that new sites start with the same module set as the
gravellecreative dev site.

## 3. Local Path Repos for Workspace Development

For local development alongside `../basekit` and `../basekit-recipe`, the
project should use Composer **path** repositories so changes in those repos
can be mirrored in without pushing:

```jsonc
"repositories": {
  "packages.drupal": { "type": "composer", "url": "https://packages.drupal.org/8" },
  "basekit-path": {
    "type": "path",
    "url": "../basekit",
    "options": { "symlink": false }
  },
  "basekit-recipe-path": {
    "type": "path",
    "url": "../basekit-recipe",
    "options": { "symlink": false }
  }
}
```

Notes:
- Path repos are for **local dev only**. On production, installs should use
  the VCS/packagist versions of `gravellian/basekit*` instead.
- `symlink: false` mirrors files into the project (what we use in this
  workspace). Use `symlink: true` only for fast local iterations and only
  when your container can see the target paths.

## 4. Creating a New Site from `basekit-project`

From a workspace where `../basekit`, `../basekit-recipe`, and
`../basekit-project` exist:

```bash
# 1. Create a new site from the project template
composer create-project gravellian/basekit-project mysite
cd mysite

# 2. (Optional) For local dev alongside ../basekit*, add path repos
#    as shown above and mirror the latest BaseKit + recipes
/opt/homebrew/bin/composer update gravellian/basekit gravellian/basekit-recipe -W

# 3. Start Lando
lando start

# 4. Apply BaseKit + Article recipes
lando recipes-apply
# or, equivalently:
# lando php web/core/scripts/drupal recipe recipes/basekit-recipe/recipes/site
# lando drush cim -y && lando drush updb -y && lando drush cr

# 5. Scaffold a sub‑theme from the BaseKit starter
bash web/themes/contrib/basekit/subtheme-starter/scripts/new-subtheme.sh mysite "My Site"
cd web/themes/custom/mysite
lando npm install
lando gulp

# 6. Set the sub‑theme as default and start theming via tokens + stubs
cd ../../../..
lando drush then mysite -y
lando drush cset system.theme default mysite -y
lando drush cr
```

At this point you have a new site whose base theme, modules, recipes, and
sub‑theme wiring match the `dev.gravellecreative.com3` setup in this
workspace. All changes to BaseKit SCSS/Twig or recipes should be made in
`../basekit` and `../basekit-recipe`, then mirrored into sites via Composer
and reapplying the recipes.

## 5. Helper Scripts in the Project

`basekit-project` includes a small `scripts/` directory intended to be copied
into every site created from the template:

- `scripts/recipes-apply.lando.sh`
  - Runs inside Lando (via `lando recipes-apply`) to re‑apply a recipe,
    then runs `drush cim`, `drush updb`, and `drush cr`.
  - Default recipe path is `/app/recipes/basekit-recipe/recipes/site` but
    you can pass a different path as the first argument.

- `scripts/update-basekit_host.sh`
  - Runs on the host from the project root.
  - Calls `composer update gravellian/basekit gravellian/basekit-recipe gravellian/basekit-docs -W`
    to pull in the latest BaseKit code, recipes, and docs.
  - If Lando is available, it then calls `lando recipes-apply`, `lando drush updb -y`,
    and `lando drush cr` to refresh the site.

After creating a site from `basekit-project`, ensure these scripts are
executable:

```bash
chmod +x scripts/*.sh
```

You can then use:

- `./scripts/update-basekit_host.sh` on the host to update BaseKit across
  the project and re‑apply recipes.
- `lando recipes-apply` inside the Lando appserver to re‑run the recipe
  pipeline when you change configuration in `recipes/basekit-recipe`.

## 6. BaseKit Setup Module

The project template also includes a lightweight setup module under
`web/modules/custom/basekit_setup` that helps wire theme‑specific config
without hard‑coding any particular sub‑theme name.

- On install, `basekit_setup`:
  - Reads the current default theme from `system.theme`.
  - Checks whether that theme exposes a `manage` region.
  - Verifies the `Manage` menu (`system.menu.manage`) exists (from the recipe).
  - Creates a `system_menu_block:manage` block for the default theme in the
    `manage` region (if one does not already exist).

Usage in a new site:

1. Apply BaseKit recipes (so the manage menu and views exist).
2. Create and set your sub‑theme as the default theme.
3. Enable the module:

   ```bash
   lando drush en -y basekit_setup
   lando drush cr
   ```

After that, the `Manage` menu block will be placed automatically for whichever
theme is currently set as default, without baking any site‑specific theme name
into the shared recipes.

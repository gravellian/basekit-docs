# BaseKit Development (for Base theme contributors)

This guide is for developers working on the BaseKit base theme itself while building one or more sites. It explains two supported workflows and when to use them. Regular site installs should follow the normal install docs and do NOT need any of this.

## TL;DR

- Consumers (most sites): install BaseKit from GitHub via Composer and update with `composer update gravellian/basekit -W`. No symlinks.
- BaseKit contributors: use one of these while developing:
  - Path repo (symlink) to your local clone (fastest live loop)
  - VCS “source” install (real git checkout under web/themes)

## Option A — Path repository (symlink to your local clone)

Use when you have a local BaseKit clone elsewhere and want edits to reflect immediately in a site.

1) Clone BaseKit somewhere outside the site (example):

```
/Volumes/WebServer/Documents/dev/basekit
```

2) In the site’s `composer.json`, add a path repo BEFORE the VCS entry and keep theme installer paths as‑is:

```
"repositories": [
  { "type": "composer", "url": "https://packages.drupal.org/8" },
  { "type": "path", "url": "/Volumes/WebServer/Documents/dev/basekit", "options": { "symlink": true } },
  { "type": "vcs", "url": "git@github.com:gravellian/basekit.git", "no-api": true }
],
"extra": {
  "installer-paths": {
    "web/themes/contrib/{$name}": ["type:drupal-theme"]
  }
}
```

3) Update just BaseKit:

```
composer update gravellian/basekit -W
```

4) Verify it’s a symlink:

```
ls -l web/themes/contrib/basekit
# → should point at your local clone
```

5) Work in your clone, not the site:

```
cd /Volumes/WebServer/Documents/dev/basekit
git checkout -b feature/x
# edit → git commit → git push
```

6) In the site, clear caches and rebuild any sub‑theme CSS:

```
lando drush cr
cd web/themes/custom/<subtheme> && lando gulp
```

Notes
- Keep `web/themes/contrib/basekit/` (or custom/basekit/) ignored in the site’s Git.
- If you run Composer inside a container, the path must be visible inside the container; or run Composer on the host for this step.
- When done, remove the path repo entry and run `composer update gravellian/basekit -W` to switch back to GitHub.

## Option B — VCS “source” install (real git checkout under web/themes)

Use when you want a normal git repo right where Drupal loads themes (no external clone, no symlink).

1) Configure Composer to install BaseKit from source:

```
composer config repositories.basekit vcs git@github.com:gravellian/basekit.git
composer config preferred-install.gravellian/basekit source
composer update gravellian/basekit -W
```

2) Work directly in the installed checkout:

```
cd web/themes/contrib/basekit
git checkout -b feature/x
# edit → git commit → git push
```

3) Clear caches and rebuild sub‑theme CSS as needed:

```
lando drush cr
cd web/themes/custom/<subtheme> && lando gulp
```

Notes
- Keep the BaseKit path ignored by the site’s Git (it’s a nested repo).
- Don’t keep uncommitted changes when running Composer updates — they can be overwritten.

## Which should sites use?

- Sites consuming the distribution should NOT use path repos or symlinks. They should:
  - Add the VCS repo once: `composer config repositories.basekit vcs git@github.com:gravellian/basekit.git`
  - Require a tagged version (or dev-main for early adopters): `composer require gravellian/basekit:^1.0 -W`
  - Update later with `composer update gravellian/basekit -W`

## Keeping every site up to date

1) Merge BaseKit changes and tag releases in `gravellian/basekit` (e.g., `v1.1.0`).
2) In each site:

```
composer update gravellian/basekit -W
php core/scripts/drupal recipe gravellian/basekit-recipe:recipes/site  # if recipes changed
lando drush cim -y && lando drush updb -y && lando drush cr
cd web/themes/custom/<subtheme> && lando gulp  # if tokens changed
```

Optional: use Dependabot/Renovate to open PRs automatically when BaseKit or recipe tags are published.

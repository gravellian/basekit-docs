Composer Cheatsheet (BaseKit Sites with composer.local.json overrides)
=====================================================================

Purpose of composer.local.json
------------------------------
* Adds path repositories pointing to ../basekit, ../basekit-recipe, ../basekit-docs.
* When active, Composer uses those local folders instead of GitHub.
* This is local-only; production should never use it.

Local Development WITH overrides
--------------------------------
* Prefer the guard wrapper if your site has it:
  ./scripts/local-composer.sh install
  ./scripts/local-composer.sh update gravellian/basekit -W
* The guard ensures the BaseKit symlink stays intact.
* After editing ../basekit*, run the update command above, then rebuild caches.

Local Development WITHOUT overrides
-----------------------------------
* Temporarily rename the file (mv composer.local.json composer.local.json.bak).
* Run plain composer install or composer update so vendor/ uses the GitHub repos.
* This mode regenerates composer.lock (not composer.local.lock). Use it when you want to mimic production.

Updating composer.lock for production after BaseKit changes
----------------------------------------------------------
If you changed `../basekit`, you must update the site’s `composer.lock` so
production pulls the new commit. Use the helper script from the workspace
root (site-agnostic):

```bash
../scripts/update-basekit-lock.sh
```

That script temporarily disables `composer.local.json`, updates the lock from
the VCS repo, then restores local overrides and reinstalls.

Production / Remote Servers
---------------------------
* Never copy composer.local.json or composer.local.lock to production.
* Run composer install --no-dev --optimize-autoloader from the site root.
* To pull fresh BaseKit packages:
  composer update gravellian/basekit gravellian/basekit-recipe gravellian/basekit-docs --with-all-dependencies --no-dev --optimize-autoloader
* Finish with vendor/bin/drush updb -y && vendor/bin/drush cr.

If composer.local.json leaked to prod
-------------------------------------
* Delete composer.local.json (and composer.local.lock) and ensure COMPOSER env is unset.
* Re-run composer install so dependencies come from the official repos.

Quick Reference
---------------
* Local w/ overrides: ./scripts/local-composer.sh ...
* Local without overrides: rename/remove composer.local.json, run composer ...
* Production: never use composer.local.*, always run composer ... (official repos only).

Resetting to Standard Composer Mode
-----------------------------------
When the override blows up vendor or you want a clean slate:
1. mv composer.local.json composer.local.json.bak (or `.disabled`)
2. composer install
3. lando drush cr
4. ../scripts/site-update-basekit_host.sh dev.justsomeguypainting.com2 (optional)

# VS Code + WSL2 + Windows 11 (Lando) setup

These steps set up VS Code on Windows 11 with WSL2 so files format on save
and Drupal file types (including `.info`) are syntax highlighted. This assumes
Lando runs via Docker Desktop + WSL2.

## One-time installs

- Windows 11: Install Docker Desktop and enable WSL2 integration.
- WSL2: Install Ubuntu (or your preferred distro) and set it as default.
- Lando for Windows: https://lando.dev/download
- VS Code extensions:
  - Remote - WSL (ms-vscode-remote.remote-wsl)
  - PHP CS Fixer (junstyle.php-cs-fixer)
  - PHP Intelephense (bmewburn.vscode-intelephense-client)
  - Prettier - Code formatter (esbenp.prettier-vscode)
  - YAML (redhat.vscode-yaml)
  - Twig Language 2 (mblode.twig-language-2)
  - Twig Formatter (mblode.twig-formatter) optional, for format-on-save in `.twig`

## Open the repo in WSL

From a Windows terminal (or VS Code command palette):

```bash
wsl
cd /mnt/<drive>/path/to/dev.justsomeguypainting.com2
code .
```

Make sure VS Code shows the green "WSL: <distro>" status in the lower-left.

## Install dependencies inside WSL (via Lando)

```bash
lando start
lando composer install
```

This creates `vendor/bin/php-cs-fixer` inside the repo so VS Code can call it
from WSL.

## Workspace settings (recommended)

Use the same settings as macOS, but ensure they live in this repo so VS Code
WSL picks them up (`.vscode/settings.json`). Example:

```json
{
  "editor.formatOnSave": true,
  "[php]": {
    "editor.defaultFormatter": "junstyle.php-cs-fixer"
  },
  "[scss]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[yaml]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[twig]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "php-cs-fixer.executablePath": "${workspaceFolder:site}/vendor/bin/php-cs-fixer",
  "php-cs-fixer.rules": "@Drupal",
  "php-cs-fixer.formatHtml": false,
  "php-cs-fixer.onsave": true,
  "files.associations": {
    "*.module": "php",
    "*.install": "php",
    "*.theme": "php",
    "*.profile": "php",
    "*.inc": "php",
    "*.engine": "php",
    "*.info": "yaml",
    "*.info.yml": "yaml",
    "*.libraries.yml": "yaml",
    "*.routing.yml": "yaml",
    "*.links.menu.yml": "yaml",
    "*.links.task.yml": "yaml",
    "*.links.action.yml": "yaml",
    "*.services.yml": "yaml",
    "*.schema.yml": "yaml"
  }
}
```

Notes:

- Prettier uses `.prettierrc` in this repo for SCSS/JSON/YAML/Twig.
- If Twig is not formatting, install Twig Formatter and set it as the default
  formatter for `[twig]`.
- Keep files in WSL (not Windows file paths like `C:\`) to avoid slow I/O.
- Reload VS Code after changing workspace settings:
  `Developer: Reload Window`.
- Your local PHP version must meet the project requirement (Drupal 11 expects
  PHP 8.3+). If PHP is older, `php-cs-fixer` and `phpcs` will fail.
- If `${workspaceFolder:site}` does not resolve in a multi-root workspace,
  use an absolute path like
  `"/home/<user>/projects/<site>/vendor/bin/php-cs-fixer"` or move the setting
  into the root folder’s `.vscode/settings.json`.

## Common issues and fixes

### Multi-root workspace path mismatch

In multi-root workspaces, `${workspaceFolder}` resolves to the folder that
contains the file you are editing. If you are editing inside the theme folder,
VS Code will look for `vendor/bin/php-cs-fixer` *inside that theme*, which
does not exist.

Fix: name the site root and reference it explicitly:

```json
{
  "folders": [
    { "name": "site", "path": "." },
    { "path": "web/themes/custom/busops" }
  ],
  "settings": {
    "php-cs-fixer.executablePath": "${workspaceFolder:site}/vendor/bin/php-cs-fixer"
  }
}
```

If the PHP CS Fixer extension logs `ENOENT` with a literal
`${workspaceFolder:site}` path, switch to an absolute path in the workspace
file or open the repo as a single folder.

### php-cs-fixer missing in vendor/bin

If `vendor/bin/php-cs-fixer` is missing, install it as a dev dependency:

```bash
lando composer require --dev friendsofphp/php-cs-fixer
```

After this, `lando composer install` is enough for future setups.

### Production installs should exclude dev dependencies

On production or CI, install without dev packages:

```bash
composer install --no-dev --optimize-autoloader
```

If you build in Docker, make sure your Dockerfile uses `--no-dev`.

## Quick checks

- Open a `.module` file and confirm PHP highlighting.
- Open a `.info.yml` file and confirm YAML highlighting.
- Save a PHP file and confirm Drupal standard formatting is applied.

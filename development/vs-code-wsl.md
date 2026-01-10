# VS Code + WSL2 + Windows 11 (Lando) setup


These steps set up VS Code on Windows 11 with WSL2 so files format
on save and Drupal file types are syntax highlighted. This assumes
Lando runs via Docker Desktop + WSL2.


## VS Code settings locations (Windows)


- User settings (personal): `%APPDATA%\Code\User\settings.json`
- Folder settings (project-shared): `<repo>/.vscode/settings.json`
- Workspace settings (personal, multi-root): `<repo>/.vscode-workspace` (or other)


## One-time installs


Assumes Windows 11, WSL2 with (Debian or other) and Lando.


**VS Code extensions:**


- Remote - WSL (ms-vscode-remote.remote-wsl)
- PHP Intelephense (bmewburn.vscode-intelephense-client)
- Prettier - Code formatter (esbenp.prettier-vscode)
- YAML (redhat.vscode-yaml)
- Twig Language 2 (mblode.twig-language-2)
- Twig Formatter (mblode.twig-formatter) optional, for format-on-save in `.twig`
- Run on Save (emeraldwalk.runonsave) for PHP format-on-save with `phpcbf`


## Open the repo in WSL


From a Windows terminal (or VS Code command palette):


```bash
wsl
cd /mnt/<drive>/path/to/busops-wm-fsweb
code .
```


Make sure VS Code shows the green "WSL: <distro>" status in the lower-left.


## Install dependencies inside WSL (via Lando)


```bash
lando start
lando composer install
```


This creates `vendor/bin/phpcbf` and `vendor/bin/phpcs` inside the repo so you
can format to Drupal standards from WSL.


If `vendor/bin/phpcbf` or `vendor/bin/phpcs` are missing, install the Drupal
Coder tools as dev dependencies:


```bash
lando composer require --dev drupal/coder
```


## Workspace settings (recommended)


VS Code settings for this setup should live in the project root at settings.json when you open the folder directly. If you open a .code-workspace file instead, some settings (like files.associations and Peacock colors) won’t apply from settings.json and must be placed in the workspace file's "settings" or in user settings.


Example (PHP format-on-save with `phpcbf` using Lando):


```json
{
  "editor.formatOnSave": true,
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
    "editor.defaultFormatter": "serhatkaya.twig-formatter"
  },
  "emeraldwalk.runonsave": {
    "commands": [
      {
        "match": "\\.(php|module|inc|install|theme|profile)$",
        "cmd": "lando php vendor/bin/phpcbf --standard=.phpcs.xml ${file}",
        "isAsync": true
      }
    ]
  },
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


- Prettier uses `.prettierrc` in this repo for SCSS/JSON/YAML. Prettier does not
  format Twig by default.
- Use Twig Formatter and set it as the default formatter for `[twig]`, then
  reload VS Code.
- Keep files in WSL (not Windows file paths like `C:\`) to avoid slow I/O.
- Reload VS Code after changing workspace settings:
  `Developer: Reload Window`.
- Your local PHP version must meet the project requirement (Drupal 11 expects
  PHP 8.3+). If PHP is older, `phpcbf` and `phpcs` will fail.
- If path resolution is inconsistent, use an absolute path in the `cmd`.


## Common issues and fixes


### Override shared folder settings (per user)


Add these to your user settings at `%APPDATA%\\Code\\User\\settings.json` to
override shared defaults:


```json
{
  "editor.formatOnSave": false
}
```


Override the PHP auto-format command for yourself:


```json
{
  "emeraldwalk.runonsave": {
    "commands": [
      {
        "match": "\\.(php|module|inc|install|theme|profile)$",
        "cmd": "lando php vendor/bin/phpcbf --standard=.phpcs.xml ${file}",
        "isAsync": true
      }
    ]
  }
}
```


### Production installs should exclude dev dependencies


On production or CI, install without dev packages:


```bash
composer install --no-dev --optimize-autoloader
```


If you build in Docker, make sure your Dockerfile uses `--no-dev`.


## Quick checks


- Open a `.module` file and confirm PHP highlighting.
- Open a `.info.yml` file and confirm YAML highlighting.
- Run `vendor/bin/phpcbf --standard=.phpcs.xml path/to/file.php` and confirm
  Drupal standard formatting is applied.


## Simple formatting from the terminal


Run Drupal's standard formatter directly:


```bash
vendor/bin/phpcbf --standard=.phpcs.xml path/to/file.php
```


For Lando:


```bash
lando php vendor/bin/phpcbf --standard=.phpcs.xml path/to/file.php
```

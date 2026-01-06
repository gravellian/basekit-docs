# VS Code setup (Drupal formatting + file types)

These steps set up VS Code on macOS to auto-format on save and highlight
Drupal file types (including `.info` and other special extensions) as closely
as possible to Drupal coding standards.

## Extensions to install

Install these from the Extensions panel:

- PHP CS Fixer (junstyle.php-cs-fixer)
- PHP Intelephense
- Prettier - Code formatter
- YAML
- Twig Language 2
- Twig Formatter (optional, for format-on-save in `.twig` files)

## Workspace settings (recommended)

This repo already includes `.vscode/settings.json`. If you want to copy/paste
manually, use this as a baseline and adjust paths if needed:

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
  "php-cs-fixer.executablePath": "${workspaceFolder}/vendor/bin/php-cs-fixer",
  "php-cs-fixer.rules": "@Drupal",
  "php-cs-fixer.formatHtml": false,
  "php-cs-fixer.autoFixBySave": true,
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

- PHP formatting uses `vendor/bin/php-cs-fixer` and the local `.php-cs-fixer.php`.
  Run `lando composer install` to ensure the binary exists.
- Prettier reads `.prettierrc` in this repo for JS/SCSS/YAML/JSON.
- Twig formatting is best-effort. If Prettier is not formatting Twig for you,
  install Twig Formatter and set it as the default formatter for `[twig]`.

## Quick checks

- Open a `.module` file and confirm PHP syntax highlighting.
- Open a `.info.yml` file and confirm YAML highlighting.
- Save a PHP file and confirm Drupal coding standard formatting is applied.

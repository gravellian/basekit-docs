# VS Code setup (Drupal formatting + file types)

These steps set up VS Code on macOS to auto-format on save and highlight
Drupal file types (including `.info` and other special extensions) as closely
as possible to Drupal coding standards.

## Extensions to install

Install these from the Extensions panel:

- Run on Save (emeraldwalk.runonsave)
- PHP Intelephense
- Prettier - Code formatter
- YAML
- Twig Language 2
- Twig Formatter (optional, for format-on-save in `.twig` files)

## Workspace settings (recommended)

This repo uses a multi-root workspace, so settings live in
`dev.justsomeguypainting.com2.code-workspace` under `"settings"`. If you are
working in a single-folder workspace, you can put the same settings in
`.vscode/settings.json`. Use the workspace file when you need multiple folders
open at once (for `../basekit*` repos).

If you want to copy/paste manually, use this as a baseline and adjust paths if
needed:

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
    "editor.defaultFormatter": "esbenp.prettier-vscode"
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

- PHP formatting uses `vendor/bin/phpcbf` and the local `.phpcs.xml`.
  Run `lando composer install` to ensure the binary exists.
- If `vendor/bin/phpcbf` is missing, install the Drupal Coder tools:
  `lando composer require --dev drupal/coder`
- Prettier reads `.prettierrc` in this repo for JS/SCSS/YAML/JSON.
- Twig formatting is best-effort. If Prettier is not formatting Twig for you,
  install Twig Formatter and set it as the default formatter for `[twig]`.

## Quick checks

- Open a `.module` file and confirm PHP syntax highlighting.
- Open a `.info.yml` file and confirm YAML highlighting.
- Save a PHP file and confirm Drupal coding standard formatting is applied.

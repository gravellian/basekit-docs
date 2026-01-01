# Font usage

## Base theme typography overview (CSS variables)

Basekit uses CSS variables for font stacks so subthemes can override typography
without rebuilding basekit CSS:

- `--font-body`
- `--font-heading`
- `--font-heading-narrow`

## Update fonts in the base theme

1. Define the default CSS variables in basekit (e.g., in a global layout file):
   - `--font-body`
   - `--font-heading`
   - `--font-heading-narrow`
2. Ensure basekit typography uses `var(--font-*)` everywhere (mixins, layout,
   components).
3. Rebuild the theme assets.

## Override fonts in a subtheme

Define the CSS variables in your subtheme (loaded after basekit). Example:

```scss
:root {
  --font-body: 'Work Sans', 'Helvetica Neue', Arial, sans-serif;
  --font-heading: 'Bungee', 'Arial Black', sans-serif;
  --font-heading-narrow: 'Bungee Outline', 'Arial Black', sans-serif;
}
```

## Apply heading styles outside prose

Use the heading mixins directly (for UI headings not inside `.body-style`):

```scss
.block-title {
  @include body-h3;
}
```

## Component-level fonts

Components should use typography mixins so they inherit token overrides:

- `@include headingtext;` (uses `--font-heading`)
- `@include bodytext;` (uses `--font-body`)

## Build / sync checklist

When changing basekit typography or CSS variables:

1. Rebuild basekit CSS: `cd ../basekit && lando gulp`
2. Sync or update into the site:
   - `./scripts/sync-basekit-theme.sh <site-key>` or
   - `composer update gravellian/basekit -W`
3. Rebuild subtheme CSS: `cd web/themes/custom/<theme> && lando gulp`
4. Clear Drupal cache: `lando drush cr`

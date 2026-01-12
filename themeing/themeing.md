# How to theme a Basekit site with the basekit_site subtheme.

## Customize font families

BaseKit font families are CSS variables. Set them in the sub‑theme `:root` and
load the web fonts in the sub‑theme library so all SDCs inherit the new fonts.

1) Load fonts in the sub‑theme library:

```yml
# web/themes/custom/basekit_site/basekit_site.libraries.yml
base:
  css:
    theme:
      https://fonts.googleapis.com/css2?family=Changa:wght@300;400;500;600;700&family=Work+Sans:wght@300;400;500&display=swap:
        type: external
```

2) Set CSS variables in the sub‑theme layout:

```scss
/* web/themes/custom/basekit_site/scss/_layout.scss */
:root {
  --font-body: 'Work Sans', 'Helvetica Neue', Arial, sans-serif;
  --font-heading: 'Changa', 'Arial Black', sans-serif;
  --font-heading-narrow: 'Changa', 'Arial Black', sans-serif;
}
```

3) Rebuild assets and clear caches:

```sh
cd web/themes/custom/basekit_site && lando gulp
lando drush cr
```

Optional: If a specific component needs a different heading font, add a
component override in the sub‑theme SCSS and import it via `scss/styles.scss`
(see `scss/components/_index.scss`).

Global type styles live in one place when you start theming:

- Headings `h1`–`h6`, plus base body font usage:  
  `web/themes/custom/basekit_site/scss/_layout.scss`
- Most heading sizes/weights/spacing come from the token overrides:  
  `web/themes/custom/basekit_site/scss/_tokens.scss` (these override BaseKit’s
  Sass defaults via `!default` in the base theme).
- Body-style sizing tokens used by rich text and components:  
  `web/themes/custom/basekit_site/scss/_body-style.scss`
- Editor typography (CKEditor):  
  `web/themes/custom/basekit_site/scss/editor-styles.scss`

### Override headings + body-style (step-by-step)

1) Override BaseKit Sass defaults in `_tokens.scss` (sizes, weights, colors):

```scss
// web/themes/custom/basekit_site/scss/_tokens.scss
@forward 'base/var/var_default' with (
  $font-size-h1: 3.5em,
  $font-weight-h1: 800,
  $line-height-h1: 1.05em,
  $color-h1: #f7e9c2,
  $color-h2: #d89bff,
  $color-h3: #ff4d73,
  $font-size-body: 1.7em,
  $line-height-body: 1.35em,
  $color-text: #cdc3b5
);
```

2) Adjust the actual element rules in `_layout.scss` (if you want custom
letter-spacing, transforms, or per-level tweaks beyond tokens):

```scss
/* web/themes/custom/basekit_site/scss/_layout.scss */
h1 {
  letter-spacing: -0.03em;
  text-transform: uppercase;
}

h3 {
  letter-spacing: 0.04em;
}

.body-style {
  color: $color-text;
}
```

3) Update body-style sizing variables (used by rich text and components):

```scss
/* web/themes/custom/basekit_site/scss/_body-style.scss */
.body-style {
  --body-style-h2-size: 2.9em;
  --body-style-h2-line-height: 1.1em;
  --body-style-h3-size: 2.1em;
  --body-style-h3-line-height: 1.15em;
}
```

4) Keep the editor in sync if you want matching fonts/colors in CKEditor:

```scss
/* web/themes/custom/basekit_site/scss/editor-styles.scss */
.ck-content,
.cke_editable {
  color: $color-text;
}
```

### Heading utility classes (selective use)

Use the utility classes to apply heading styles only where needed (UI elements,
custom blocks, etc.), without changing global `h2–h6` rules. BaseKit does not
apply global `h2–h6` styles by default.

```scss
/* web/themes/custom/basekit_site/scss/_typography.scss */
.ui-h2 { @include body-h2; }
.ui-h3 { @include body-h3; }
.ui-h4 { @include body-h4; }
.ui-h5 { @include body-h5; }
.ui-h6 { @include body-h6; }
```

```html
<h3 class="ui-h3">Panel title</h3>
```

To customize the utilities, edit `_typography.scss` and add your tweaks after
the mixins (letter-spacing, casing, color, etc.).

### Body-style overrides (content scope)

`bodyStyle` drives typography inside `.body-style`. Use `_tokens.scss` for
defaults, then override colors and sizes in `_body-style.scss` to target only
rich text/content areas.

```scss
/* web/themes/custom/basekit_site/scss/_body-style.scss */
.body-style h2 { color: $color-h2; }
.body-style h3 { color: $color-h3; }
.body-style p,
.body-style li { color: $color-text; }
```

## Customize site colors

Set your brand palette first and let the semantic roles follow. Keep overrides in
`web/themes/custom/basekit_site/scss/_tokens.scss` and avoid hard-coding hex
values in components.

Recommended order for brand tokens:

```scss
@forward 'base/var/var_default' with (
  $brand-deep: #4f4a47,
  $brand-surface: #635d59,
  $brand-primary: #668fef,
  $brand-secondary: #c13121,
  $brand-accent: #fde021,
  $brand-neutral: #b59f86,
  $brand-light: #f1e9da
);
```

Use `$site-bgcolor` and `$page-bgcolor` for body/page surfaces, and map them to
your brand tokens in the same file (e.g., `$site-bgcolor: $brand-deep`,
`$page-bgcolor: $brand-surface`).

## Customize block styles

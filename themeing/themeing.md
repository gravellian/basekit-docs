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

## Customize site colors


## Customize block styles

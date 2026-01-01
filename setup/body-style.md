# Body Style (Prose Typography)

Basekit uses a single `.body-style` class for long‑form text (prose). The class
applies consistent font sizing, line‑height, spacing, and heading styling across
all rich‑text fields.

## Where body-style is defined (Basekit)

- Base mixin: `basekit/scss/base/var/_body-style.scss`
  - Applies typography to `.body-style` and common prose elements.
  - Uses CSS variables for body-style sizing/line-height/weight and heading sizes.
- Default CSS variables: `basekit/scss/layout/_page-format.scss` (`:root`)
  - `--body-style-font-size`
  - `--body-style-line-height`
  - `--body-style-font-weight`
  - `--body-style-h2-size` … `--body-style-h6-size`
  - `--body-style-h2-line-height` … `--body-style-h6-line-height`

## How to update body-style in the base theme

1. Edit default CSS variables in `basekit/scss/layout/_page-format.scss`.
2. (Optional) Update the structural rules in `basekit/scss/base/var/_body-style.scss`
   if you need different selectors or spacing rules.
3. Rebuild basekit CSS.

## How to customize body-style in a subtheme

Override only the CSS variables in the subtheme so you don’t duplicate the mixin.

File: `web/themes/custom/<theme>/scss/_body-style.scss`

```scss
@use 'base' as *;

.body-style {
  --body-style-font-size: #{$font-size-body};
  --body-style-line-height: #{$line-height-body};
  --body-style-font-weight: #{$font-weight-body};
  --body-style-h2-size: #{$font-size-h2};
  --body-style-h2-line-height: #{$line-height-h2};
  --body-style-h3-size: #{$font-size-h3};
  --body-style-h3-line-height: #{$line-height-h3};
  --body-style-h4-size: #{$font-size-h4};
  --body-style-h4-line-height: #{$line-height-h4};
  --body-style-h5-size: #{$font-size-h5};
  --body-style-h5-line-height: #{$line-height-h5};
  --body-style-h6-size: #{$font-size-h6};
  --body-style-h6-line-height: #{$line-height-h6};
}
```

Then include it in your subtheme stylesheet:
`web/themes/custom/<theme>/scss/styles.scss`

```scss
@use 'body-style';
```

## Using the .body-style class for prose

Apply `.body-style` to long‑text output (WYSIWYG/long‑form fields). This keeps
prose styling consistent across blocks and content types.

Drupal long‑text fields commonly add `.body-style` via field templates (for
example, a text field template can add `body-style` to the wrapper). Any field
that outputs formatted text can opt into `.body-style` to get the shared prose
styles.

## CKEditor (WYSIWYG) styling

The subtheme’s editor styles should reflect the same typography so the editor
matches front‑end prose.

File: `web/themes/custom/<theme>/scss/editor-styles.scss`

- Uses `var(--font-*)` for font families.
- Uses the same body-style sizing/line-height values so WYSIWYG matches
  front‑end output.

After changing body-style or font variables, rebuild the subtheme CSS and clear
caches.

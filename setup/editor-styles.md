# CKEditor presentation contract

BaseKit treats CKEditor as a structured content editor, not a page designer.
Authors choose headings, lists, links, media, and a small semantic style
vocabulary. Font files, exact sizes, colors, and decorative rendering remain
theme responsibilities.

## Shared boundary

BaseKit owns:

- the semantic class names and their default behavior;
- the `content` text-format toolbar and HTML allowlist supplied by the recipe;
- the readable editor canvas, defaulting to `60rem` with responsive gutters;
- parity between `.body-style` frontend prose and `.ck-content` editor prose;
- a regression fixture covering representative class combinations.

Each subtheme owns:

- the actual body, heading, and condensed-heading font families;
- semantic color and surface values;
- an intentional content-width override when its frontend column differs;
- optional, narrowly scoped site styles such as a branded button or illustrated
  underline, provided their classes and filter permissions are explicit.

Site adoption is a reviewed configuration change. Do not overwrite a site's
active editor or filter configuration from the recipe without reconciling its
database and config export first.

## Approved semantic vocabulary

The shared CKEditor Style menu may emit only these BaseKit presentation
classes:

| Role | Class |
| --- | --- |
| Body family | `font-body` |
| Heading family | `font-heading` |
| Condensed heading family | `font-heading-narrow` |
| Smallest, smaller, normal, larger, largest copy | `copy-smallest`, `copy-small`, `copy-normal`, `copy-large`, `copy-largest` |
| Primary, accent, secondary, neutral, light color | `text-primary`, `text-accent`, `text-secondary`, `text-neutral`, `text-light` |
| Semantic underline | `underline-accent` |
| Limited inline backgrounds | `surface-light`, `surface-primary`, `surface-accent` |

Size styles use `<p>` for whole paragraphs and `<span>` for selected text. Other
styles use `<span>` so they remain inline content annotations. The shared
contract does not expose pixel sizes, arbitrary colors, arbitrary font names,
spacing, positioning, borders, columns, or free-form backgrounds. Layout
Builder and theme components own layout and block-level presentation.

## Unified copy scale

All five size roles use the site's body baseline: 75%, 85%, 100%, 115%, 130%.
They are absolute roles within that scale, not multipliers of the surrounding
text. A Smaller span inside a Smaller paragraph stays 85%, not 72.25%.
Use Normal to explicitly restore the body baseline inside a smaller block;
removing the style instead restores that block's inherited default.

The Styles dropdown offers `Paragraph: Smallest/Smaller/Normal/Larger/Largest`
and `Text: Smallest/Smaller/Normal/Larger/Largest`. Place the cursor in a paragraph
for a paragraph style; select words for a text style. These are style toggles:
turn off the previous size when changing sizes. If old markup contains multiple
size classes, CSS uses a fixed precedence (largest last), never compounded sizes.
Use headings for structure, not merely to make copy larger. Reserve Smallest
for short supporting text, and check readability at mobile widths.

Blocks use the same roles on their `.body-style` wrapper. Applying a role to an
outer layout container is not the contract. Wrapper roles affect copy, not heading
sizes. Existing `copy-small` and `copy-large` classes remain valid.

`copy-scale.tokens()` emits the body and size tokens. `copy-scale.install()` emits
those tokens plus frontend block bindings. Subthemes should supply both mixins
from one `_tokens.scss` file, as in the starter. Available parameters are
`$body-size`, `$small`, `$large`, `$ui`, `$smallest`, and `$largest` (the new
parameters are appended to preserve existing calls). Editor CSS must load its own
tokens; it cannot depend on the frontend theme being active on an admin page.

`--font-size-body-{smallest,small,large,largest}` are shared by blocks, paragraphs,
and inline sizes; Normal uses `--font-size-body`. Keep the body baseline in rem
or another non-nesting unit. The old `--wysiwyg-copy-small/large` overrides are
superseded: migrate them to the shared body-size tokens during site adoption.
The old inline defaults (.82em / 1.25em) therefore change intentionally; review
existing content. Do not globally change a size token merely to tune one block.

## Theme tokens

Subthemes set the contract with CSS custom properties, normally in `:root` so
the same values compile into frontend and editor stylesheets:

```scss
:root {
  --wysiwyg-content-width: 60rem;
  --wysiwyg-gutter: clamp(1rem, 3vw, 2rem);
  --wysiwyg-font-body: var(--font-body);
  --wysiwyg-font-heading: var(--font-heading);
  --wysiwyg-font-heading-narrow: var(--font-heading-narrow);
  --font-size-body: max(18px, 1.7rem);
  --font-size-body-smallest: calc(var(--font-size-body) * .75);
  --font-size-body-small: calc(var(--font-size-body) * .85);
  --font-size-body-large: calc(var(--font-size-body) * 1.15);
  --font-size-body-largest: calc(var(--font-size-body) * 1.3);
  --wysiwyg-color-primary: var(--brand-primary);
  --wysiwyg-color-accent: var(--brand-accent);
  --wysiwyg-color-secondary: var(--brand-secondary);
  --wysiwyg-color-neutral: var(--brand-neutral);
  --wysiwyg-color-light: var(--brand-light);
  --wysiwyg-surface-light: rgba(255, 255, 255, .85);
  --wysiwyg-surface-primary: color-mix(in srgb, var(--brand-primary) 18%, transparent);
  --wysiwyg-surface-accent: color-mix(in srgb, var(--brand-accent) 18%, transparent);
}
```

Use values compatible with the site's supported browsers; the example
`color-mix()` values are illustrative. A subtheme may omit tokens whose role it
does not use, in which case BaseKit's conservative defaults apply.

## Wiring and font loading

Frontend long-text field templates use `.body-style`. The subtheme's
`ckeditor5-stylesheets` entry loads its compiled `css/editor-styles.css`; that
file must include the same typography tokens and BaseKit `bodyStyle` mixin as
the frontend build. Webfont CSS needed for the preview must also be reachable
from the editing document. Do not assume a frontend-only library is present in
CKEditor.

```scss
@use 'base' as *;

.ck-content,
.cke_editable {
  @include bodyStyle;
}
```

The base theme also constrains `.ck-editor__editable_inline` to
`--wysiwyg-content-width`. Override that token only when the rendered long-text
column is intentionally different.

## Verification

Run `npm test` and `npm run build` in BaseKit. Review
`tests/fixtures/wysiwyg-contract.html` on the frontend and reproduce the same
content in CKEditor at narrow and desktop widths. Compare font family, computed
font size, line height, color, underline, background, and content-column width.
Site-specific additions need their own representative fixture and must not
enable generic font, size, color-picker, or background-picker controls.

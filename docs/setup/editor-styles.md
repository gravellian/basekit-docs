# Editor Styles (WYSIWYG)

This theme centralizes body/prose styling in a single SCSS mixin so the same rules apply to CKEditor content and frontend body copy.

**Where it’s wired**

- Wrap long-text fields with `.body-style` in Twig:
  - `web/themes/custom/gravelle1/templates/field/field--text.html.twig`
  - `web/themes/custom/gravelle1/templates/field/field--text-with-summary.html.twig`
  - `web/themes/custom/gravelle1/templates/field/field--text-long.html.twig`
- Load editor stylesheet in CKEditor 5:
  - `web/themes/custom/gravelle1/gravelle1.info.yml` → `ckeditor5-stylesheets: css/editor-styles.css`

**Use the mixins**

- Apply prose styles anywhere:

```scss
.body-style {
  @include bodyStyle;
}
```

- Heading mixins (can be used inside components or globally):

```scss
h2 { @include body-h2; }
h3 { @include body-h3; }
h4 { @include body-h4; }
h5 { @include body-h5; }
h6 { @include body-h6; }
```

Headings and body-copy variables live in:
- `web/themes/custom/gravelle1/scss/base/var/_var_default.scss`

**CKEditor-specific styling**

- Editor content wrapper:

```scss
.ck.ck-content {
  @include bodyStyle; // reuse the same prose rules
}
```

- Media view mode sizing inside the editor uses region-first `main_*` prefixes to match filter + view modes:

```scss
.ck.ck-content {
  .media[class^='display--main_66'],
  figure[data-view-mode^='main_66'] { width: 65%; max-width: 65%; }

  .media[class^='display--main_50'],
  figure[data-view-mode^='main_50'] { width: 49%; max-width: 49%; }

  .media[class^='display--main_33'],
  figure[data-view-mode^='main_33'] { max-width: 32%; }

  .media[class^='display--main_25'],
  figure[data-view-mode^='main_25'] { max-width: 24%; }
}
```

**Frontend mixin details**

Core rules live in `web/themes/custom/gravelle1/scss/base/var/_body-style.scss` as `@mixin bodyStyle`. It handles:
- Spacing for block elements (p, lists, blockquote, figure)
- Heading styles via `@include body-h2..h6`
- Inline image alignment within text
- Figure/figcaption formatting and responsive sizing by view mode
- Table styling
- Horizontal rule styling
- Text alignment helper classes allowed by the “Article” format:

```scss
.text-align-left    { text-align: left; }
.text-align-center  { text-align: center; }
.text-align-right   { text-align: right; }
.text-align-justify { text-align: justify; }
```

Breakpoints use theme mixins like `@include desktop-small` (768px+). See `web/themes/custom/gravelle1/scss/base/var/_mixins.scss`.

**Responsive media**

- Twig adds classes that pair with the SCSS rules and view modes:
  - `web/themes/custom/gravelle1/templates/media/media--image.html.twig` → `display--<view_mode>` and `media-ext--<ext>`
- Preprocess sets `file_extension` for the Twig template:
  - `web/themes/custom/gravelle1/gravelle1.theme` → `gravelle1_preprocess_media__image()`
- Allowed view modes in the “Article” text format use the `main_*` prefix.

See `docs/setup/responsive-media.md` for the region‑first naming rationale and full setup.

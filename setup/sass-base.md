# Sass Base Module

The theme exposes a single Sass entrypoint that re‑exports all tokens, mixins, and utilities needed across the site. Use it once per file and you’re set.

Paths
- Base module: `web/themes/custom/gravelle1/scss/base/_index.scss`
- Var submodule: `web/themes/custom/gravelle1/scss/base/var/_index.scss`

Usage
- In global theme SCSS (under `scss/`):
  - `@use './base' as *;`
- In component SCSS (under `components/`):
  - `@use 'base' as *;` (gulp sets `includePaths: ['scss']` for components)

What it exports (one import gives you all of these)
- Variables from `_var_default.scss`
  - Layout: `$m-page-header`, `$d-pagew`, `$m-pagep`, `$d-pagep`, `$pagepad`
  - Colors: `$color-1..3`, `$gray`, `$white`, `$color-h*`, etc.
  - Typography: `$font-body`, `$font-heading`, `$font-size-body`, `$line-height-body`, `$font-weight-*`
  - Page: `$site-bgcolor`, `$page-bgcolor`
- Breakpoint mixins (from `_mixins.scss`)
  - `@include desktop-small { … }`      // 768–1023px
  - `@include desktop-medium { … }`     // 1024–1364px
  - `@include desktop-large { … }`      // 1365–1919px
  - `@include desktop-max { … }`        // 1920px+
- Typography mixins (from `_mixins.scss`)
  - `@include body-h2..h6` (heading styles used across body copy)
  - `@include headingtext`, `@include bodytext`, `@include uline`
- Prose mixin (from `_body-style.scss`)
  - `@include bodyStyle;` – shared rules for long‑text (frontend + CKEditor)
- Load effect mixins (from `_load-effects.scss`)
  - `@include fade-in-up()`, `@include zoom-pop()`, `@include blur-in()`
  - `@include reduced-motion-reset` – respects prefers‑reduced‑motion
- Icons (from `_icons.scss`)
  - Reserved for encoded SVG tokens (currently empty; safe to ignore)

File map
```
base/                        # aggregator
├─ _index.scss               # @forward 'var/index'
└─ var/
   ├─ _index.scss            # @forward default vars, mixins, body-style, icons, load-effects
   ├─ _var_default.scss      # theme variables (colors, sizes, spacing)
   ├─ _mixins.scss           # breakpoints + typographic mixins
   ├─ _body-style.scss       # prose/long‑text rules as a mixin
   ├─ _load-effects.scss     # animation/entrance effect mixins
   └─ _icons.scss            # placeholder for encoded SVGs
```

Examples
```scss
@use './base' as *;

// Variables
.button { background: $color-2; color: $white; }

// Breakpoints
.card {
  padding: 1em;
  @include desktop-small { padding: 1.25em; }
  @include desktop-medium { padding: 1.5em; }
}

// Heading style
.section-title { @include body-h2; }

// Prose block
.body-style { @include bodyStyle; }

// Load effects
.teaser { @include fade-in-up(); @include reduced-motion-reset; }
```

Conventions
- Use `@use 'base' as *;` once at the top of each SCSS file that needs tokens/mixins.
- Prefer the breakpoint mixins over hard‑coded media queries to keep breakpoints centralized.
- The variables in `_var_default.scss` use `!default` to allow future configuration; for now, change theme tokens by editing that file.

Related docs
- Editor Styles: `docs/setup/editor-styles.md`
- Responsive Media: `docs/setup/responsive-media.md`
- Image Styles inventory: `docs/setup/image-styles.md`
- Responsive Template: `docs/setup/responsive-template.md`

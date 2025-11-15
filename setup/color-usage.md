# Color usage and SCSS variables

This theme uses a small set of brand tokens, semantic roles, and tonal helpers so color stays consistent and easy to update. Avoid hard‑coding hex values inside components—reference tokens instead.

Where things live
- Variables: `web/themes/custom/gravelle1/scss/base/var/_var_default.scss`
- Mixins (includes heading helpers): `web/themes/custom/gravelle1/scss/base/var/_mixins.scss`
- Common usage examples: `web/themes/custom/gravelle1/scss/layout/_page-format.scss`, `web/themes/custom/gravelle1/scss/block/_block-defaults.scss`

Core brand tokens
- ` $brand-bg` — site/page background (was `$color-1`)
- ` $brand-primary` — primary brand (gold)
- ` $brand-accent` — accent (blue)
- ` $brand-neutral` — neutral/taupe for tertiary headings and UI

Back‑compat aliases (still supported)
- ` $color-1` → `$brand-bg`
- ` $color-2` → `$brand-primary`
- ` $color-3` → `$brand-accent`
- ` $color-4` → `$brand-neutral`

Semantic color roles
- Headings
  - ` $color-h1` → `$brand-primary`
  - ` $color-h2` → `$brand-accent`
  - ` $color-h3` → `$brand-neutral`
- Text
  - ` $color-text` (primary body text)
  - ` $text-muted`, ` $text-subtle` (reduced‑emphasis text)
- Links
  - ` $link-color`, ` $link-hover-color`, ` $link-click-color`, ` $link-border`
- Surfaces and lines
  - ` $surface-0` (page), ` $surface-1` (cards/blocks), ` $surface-2` (elevated)
  - ` $rule` (underlines/rules), ` $border` (UI borders)
- Focus & selection
  - ` $focus-ring`, ` $selection`

Tonal helpers (for controlled tints/shades)
- Helpers use the Sass module `sass:color`:
  - ` tint($color, $percent)` — mix with white by percent
  - ` shade($color, $percent)` — mix with black by percent
- Prebuilt ramps per brand:
  - ` $primary-light`, ` $primary-dark`
  - ` $accent-light`, ` $accent-dark`
  - ` $neutral-light`, ` $neutral-dark`

Usage guidelines
- Do
  - Use heading roles for titles: `color: $color-h1|$color-h2|$color-h3`.
  - Use ` $rule` for page/title underlines and ` $border` for component borders.
  - Use ` $surface-1`/` $surface-2` for card/panel backgrounds.
  - Use link tokens for anchors (do not hardcode hex on links).
  - Use ` $text-muted`/` $text-subtle` for reduced emphasis copy.
- Don’t
  - Don’t hardcode hex or ad‑hoc `rgba($brand, …)` inside components.
  - Don’t lighten/darken with random percentages—use ramps or the `tint`/`shade` helpers.

Examples

Headings (already wired)
```
.page-title { color: $color-h1; border-bottom: $stroke-2 solid $rule; }
h2 { color: $color-h2; }
h3 { color: $color-h3; }
```

Links
```
a:link, a:visited { color: $link-color; }
a:hover { color: $link-hover-color; }
a:active { color: $link-click-color; }
```

Card/Block surfaces
```
.card {
  background-color: $surface-1;
  border: $stroke-1 solid $border;
  color: $text-high;
}
```

Rules/underlines
```
.section-rule { border-bottom: $stroke-2 solid $rule; }
```

Focus & selection
```
:focus { outline: .2em solid $focus-ring; outline-offset: .1em; }
::selection { background: $selection; }
```

Creating a new variant (hover/active)
```
.btn-accent {
  background: $brand-accent;
  &:hover { background: $accent-light; }
  &:active { background: $accent-dark; }
}
```

Accessibility notes
- On the background `#635d59`, large headings with `$brand-primary`, `$brand-accent`, and `$brand-neutral` meet contrast for large text.
- Body copy should stay on ` $color-text`/` $text-muted` for readability.
- Use ` $rule`/` $border` for separation instead of low‑contrast text colors.

Overriding in subthemes
- Prefer overriding core tokens (`$brand-*`) rather than roles; the roles will follow.
- Keep the same variable names to minimize diff blast radius.

## Tonal helper recipes (quick reference)

Where they live: `web/themes/custom/gravelle1/scss/base/var/_var_default.scss` (uses `sass:color`).

Good defaults
- Hover: `tint(..., 12–18%)`
- Active/pressed: `shade(..., 12–18%)`
- Soft backgrounds: `tint(..., 6–12%)` then optionally wrap in `rgba(..., .15–.3)`
- Lines/borders: prefer `$rule`/`$border`; custom: `rgba(shade(..., 18%), .4–.6)`

Examples

Links with explicit tint/shade
```
a:link,
a:visited { color: $brand-accent; }
a:hover   { color: tint($brand-accent, 16%); }
a:active  { color: shade($brand-accent, 16%); }
```

Custom soft card using brand background
```
.card--soft {
  background: tint($brand-bg, 8%);
  border: $stroke-1 solid rgba($brand-neutral, .35); /* or $border */
}
```

Underline driven by neutral tone
```
.title-rule { border-bottom: $stroke-2 solid rgba(tint($brand-neutral, 8%), .5); }
```

Soft highlight behind inline text
```
.highlight { background-color: rgba(tint($brand-accent, 35%), .25); }
```

Badge with toned border
```
.badge--primary {
  background: $brand-primary;
  color: $text-high;
  border: $stroke-1 solid shade($brand-primary, 18%);
}
```

# Responsive Template

This site uses a stable, two‑phase responsive approach:

- Handheld (max-width: 767px): fluid layout and fluid type using `vw` so text and blocks scale smoothly across phones.
- Desktop (min-width: 768px): fixed page widths at each breakpoint with proportional font‑size steps to keep line length, rhythm, and spacing visually consistent on laptops and larger screens.

The goal is that content density and line length don’t jitter as screens change — only the canvas widens at known points.

**Where it’s implemented**

- Breakpoints: `web/themes/custom/gravelle1/gravelle1.breakpoints.yml`
- Layout + font scale: `web/themes/custom/gravelle1/scss/layout/_page.scss`
- Breakpoint mixins (for component use): `web/themes/custom/gravelle1/scss/base/var/_mixins.scss` (`desktop-small`, `desktop-medium`, `desktop-large`, `desktop-max`)

**Breakpoint behavior**

In `_page.scss`, `html { font-size: ... }` and `.sitew { width: ... }` are set per range. Desktop font sizes are chosen as percentages of the max (22px) to keep relative sizing stable.

```scss
/* handheld */
@media all and (max-width: 767px) {
  html { font-size: 2.6vw; }   // fluid type
  .sitew { width: 100vw; }
}

/* desktop - small - 768 - 1023 */
@media all and (max-width: 1023px) and (min-width: 768px) {
  html { font-size: 8.8px; }    // ~40% of 22px
  .sitew { width: 768px; }
}

/* desktop - medium - 1024 - 1364 */
@media all and (max-width: 1364px) and (min-width: 1024px) {
  html { font-size: 11.7333333333px; } // ~53.33%
  .sitew { width: 1024px; }
}

/* desktop - large - 1365 - 1919 */
@media all and (max-width: 1919px) and (min-width: 1365px) {
  html { font-size: 15.640625px; } // ~71.09%
  .sitew { width: 1365px; }
}

/* desktop - max - 1920+ */
@media all and (min-width: 1920px) {
  html { font-size: 22px; }     // 100% baseline
  .sitew { width: 1920px; }
}
```

This keeps paragraph line lengths and vertical rhythm stable across desktop ranges, while phones remain fluid.

**How this ties to other parts of the system**

- Responsive Media: View modes (`main_*`) map to percentage widths that correspond to these breakpoints and page widths. See `docs/setup/responsive-media.md` and `docs/setup/image-styles.md`.
- Editor Styles: CKEditor uses the same bodyStyle mixin and `display--main_*` sizing so the editor preview matches frontend. See `docs/setup/editor-styles.md`.

**Using breakpoint mixins in components**

For component styles, prefer the theme mixins (keeps magic numbers in one place):

```scss
@use '../base' as *; // gives access to desktop-* mixins

.my-component {
  /* base (handheld) styles */

  @include desktop-small { /* 768–1023 */ }
  @include desktop-medium { /* 1024–1364 */ }
  @include desktop-large { /* 1365–1919 */ }
  @include desktop-max { /* 1920+ */ }
}
```

**Adjusting the design**

If you change page width or font‑size at any breakpoint:
- Update `_page.scss` for `html { font-size }` and `.sitew { width }`.
- Review responsive image style mappings to ensure `main_*` widths still align with the visual grid.
- Rebuild theme CSS: `cd web/themes/custom/gravelle1 && lando gulp`.
- Clear caches: `lando drush cr`.

Validation checklist
- Frontend and CKEditor line lengths look consistent across desktop ranges.
- Media rendered via `display--main_*` matches intended percentages.
- No unexpected shifts between adjacent desktop breakpoints.

**Breakpoint Summary**

| Range            | Media query                                   | `.sitew` width | `html` font-size |
|------------------|-----------------------------------------------|----------------|------------------|
| Handheld         | `all and (max-width: 767px)`                  | `100vw`        | `2.6vw`          |
| Desktop small    | `all and (max-width: 1023px) and (min-width: 768px)`  | `768px`        | `8.8px`          |
| Desktop medium   | `all and (max-width: 1364px) and (min-width: 1024px)` | `1024px`       | `11.7333px`      |
| Desktop large    | `all and (max-width: 1919px) and (min-width: 1365px)` | `1365px`       | `15.640625px`    |
| Desktop max      | `all and (min-width: 1920px)`                  | `1920px`       | `22px`           |

Diagram

```
Handheld  ───────────── fluid (vw) ───────────▶ 768
Desktop   768 ── fixed ── 1024 ── fixed ── 1365 ── fixed ── 1920+ (max)
Fonts     8.8px ───────► 11.7333px ───────► 15.6406px ───────► 22px
Widths    768px  ─────► 1024px   ────────► 1365px   ────────► 1920px
```

# Mobile Menu (Pushy) — Implementation Guide

This site uses Pushy (off‑canvas) for the mobile navigation drawer. The drawer pushes the main content, keeps the hamburger visible, and renders blocks you place in the `Sidebar first` region.

Refer to docs/setup/utility-belt.md for build and cache commands used below.

## Overview

- Drawer library: Pushy v2.0.0 (no jQuery)
- Drawer content source: Drupal region `sidebar_first`
- Toggle: header button with class `.menu-btn`
- Overlay: `.site-overlay`
- Content panel: `#container` (content is translated when menu opens)

## Theme Files Involved

- Template
  - `web/themes/custom/gravelle1/templates/layout/page.html.twig`
- Library registration
  - `web/themes/custom/gravelle1/gravelle1.libraries.yml`
  - `web/themes/custom/gravelle1/gravelle1.info.yml`
- Pushy assets (checked into theme)
  - CSS: `web/themes/custom/gravelle1/js/pushy/css/pushy.css`
  - JS: `web/themes/custom/gravelle1/js/pushy/js/pushy.min.js`
  - Arrow: `web/themes/custom/gravelle1/js/pushy/img/arrow.svg`

## Markup

Ensure the following elements exist in your page layout (already present in the theme):

- Drawer (renders Sidebar first region)

```twig
<nav class="pushy pushy-right">
  <div class="pushy-content">
    {% if page.sidebar_first %}
      <div class="sidebar">
        {{ page.sidebar_first }}
      </div>
    {% endif %}
  </div>
  {# /pushy #}
</nav>
```

- Overlay

```html
<div class="site-overlay"></div>
```

- Content panel wrapper (required for push effect)

```html
<div id="container" class="sitew">...</div>
```

- Header menu button (hamburger)

```html
<button class="menu-btn">☰</button>
```

Notes

- The theme uses only `page.html.twig` for page layout (no SDC `site-page.twig`).
- Pushy v2 supports `data-menu-btn-selector` and `data-focus` on `<nav class="pushy …">` to customize the toggle selector or focus target. This theme sets both without changing design:

```twig
<nav class="pushy pushy-right" data-menu-btn-selector=".menu-btn" data-focus=".pushy a">
  ...
</nav>
```

## Library Wiring

The theme loads Pushy via the `gravelle1/pushy` library.

- `web/themes/custom/gravelle1/gravelle1.libraries.yml`

```yaml
pushy:
  css:
    theme:
      js/pushy/css/pushy.css: {}
  js:
    js/pushy/js/pushy.min.js: {}
```

- `web/themes/custom/gravelle1/gravelle1.info.yml`

```yaml
libraries:
  - gravelle1/base
  - gravelle1/messages
  - gravelle1/pushy
  - gravelle1/fonts.roboto
  - core/normalize
```

## Styling (Width + Push)

Default widths are implemented in `pushy.css`:

- Mobile: `80vw`
- Desktop (≥768px): `60vw`

You can change these in `web/themes/custom/gravelle1/js/pushy/css/pushy.css`:

```css
.pushy {
  width: 80vw;
}
@media (min-width: 768px) {
  .pushy {
    width: 60vw;
  }
  .pushy-left {
    transform: translate3d(-60vw, 0, 0);
  }
  .pushy-right {
    transform: translate3d(60vw, 0, 0);
  }
  .pushy-open-left #container {
    transform: translate3d(60vw, 0, 0);
  }
  .pushy-open-right #container {
    transform: translate3d(-60vw, 0, 0);
  }
}
```

Pushy automatically adds body classes (`pushy-open-right` or `pushy-open-left`) to animate both the drawer and `#container`.

## Drawer Menu Behavior
- Region‑scoped application: the theme preprocess switches any core menu block rendered in `sidebar_first` to use the Pushy menu template (`menu__pushy`) so it only affects the drawer.
- Demo‑style parents: expandable parents render as buttons and do not navigate; the parent link is added as the first child item inside the submenu (e.g., “All Articles”).
- Collapsed by default: submenus are collapsed unless the parent is in the active trail (arrow rotates when open).
- Styling isolation: header and `mgr-links` use default nav styles; the drawer uses `.pushy …` styling and resets so header styles don’t bleed in.

## Placing Menu Content

- Go to Structure → Block layout → gravelle1 → `Sidebar first` and place the blocks/menus you want in the drawer (e.g., Main navigation).
- The drawer renders whatever is in `Sidebar first`.

## Build & Cache

See docs/setup/utility-belt.md for full details. Typical flow:

```bash
# Build theme assets
cd web/themes/custom/gravelle1 && lando gulp

# Rebuild Drupal caches
lando drush cr
```

## Options

- Focus first link on open:
  - Add `data-focus="#your-link-id"` to `<nav class="pushy …">`.
- Use a different toggle selector:
  - Add `data-menu-btn-selector=".your-toggle"` to the pushy nav if you don’t use `.menu-btn`.
- Submenus
  - Add `.pushy-submenu` wrappers; Pushy auto-collapses/expands with built‑in JS.

## Troubleshooting

- Drawer appears above header on load
  - Ensure markup uses `<nav class="pushy …">` and not the older `#mobile-menu` off‑canvas.
  - Confirm `#container` exists and wraps your content.
  - Rebuild assets and clear caches; force‑refresh the browser.
- Hamburger doesn’t toggle
  - Confirm the button has class `.menu-btn` or set `data-menu-btn-selector` on the pushy nav.
- Drawer width not pushing correctly
  - Update widths in `pushy.css` for both the drawer and the panel transforms.
 - Header menu shows sublinks
   - Reduce the header block depth to 1 (Block config → Maximum number of menu levels) or leave it as‑is if that’s desired. Header/mgr-links styles are scoped and won’t affect the drawer.

## Version

- Pushy v2.0.0 (no jQuery). Upstream: https://chrisyee.ca/pushy/

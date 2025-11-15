# Custom Blocks (Template + SDC Standard)

This site’s custom blocks are implemented as Single‑Directory Components (SDCs) rendered inside a shared base template. The base template enforces a consistent, predictable wrapper and classes for all custom blocks; each bundle’s SDC provides the inner markup and styles.

Key ideas

- Base wrapper lives in `templates/block/block--block-content.html.twig` and applies to ALL custom block bundles.
- Every custom block renders with a single inner wrapper `.block-wrap` inside the outer `.block` container.
- SDC partials must NOT output `.block-wrap`; they render only their inner structure (image/title/content/text/etc.).
- View modes are exposed as classes and should drive styling differences where possible.
- If a bundle needs a special wrapper, create a more specific block template override (keep the required classes/elements).
- Lists/grids inside blocks use the unified `.block-items` pattern. See Custom Block Displays: [custom-block-display.md](custom-block-display.md).

Required structure and classes

- Outer container: `.block` with standard classes
  - `block`
  - `block-{provider}` (e.g., `block-block-content`)
  - `block-{plugin_id}` (from Drupal)
  - `block-type--{bundle}` when available (exposed via preprocess)
  - `block-view-mode--{view_mode}` when available (exposed via preprocess)
- Inner wrapper: `.block-wrap`
  - The base template adds `<div class="block-wrap">…</div>`
  - Adds `wlabel` when the block has a label so components can style labeled vs unlabeled states.
- Label handling:
  - The base template suppresses the default H2. Components output any visual title (commonly `<div class="title"><h3>…</h3></div>` when `props.has_label`).

Base template for custom blocks

- Location: `templates/block/block--block-content.html.twig`
- Responsibility: add required classes/wrappers and render `{{ content }}` inside `.block-wrap`.
- Never add bundle‑specific logic here. If a bundle/view mode needs extra structure or classes, override with a more specific suggestion (see below).

Per‑bundle block templates

- Path: `templates/block/block--block-content--type--<bundle>.html.twig`
- Always extend the base and only override `{% block content %}` to map fields and include the SDC:
  - `{% extends "block--block-content.html.twig" %}`
  - `{% block content %} {{ include('@gravelle1/components/<bundle>/<bundle>.html.twig', { props: { … } }) }} {% endblock %}`

SDC component layout (per bundle)

```
components/<bundle>/
├─ partials/
│ ├─ <view_mode>.html.twig
│ └─ default.html.twig
├─ <bundle>.component.yml
├─ <bundle>.twig or <bundle>.html.twig
├─ <bundle>.scss → compiled to <bundle>.css (+ .map)
```

Component rules

- Do not output `.block-wrap`; the base template provides it.
- Keep markup focused on content structure: `.image`, `.title`, `.content`, `.text`, etc.
- Attach your library in the SDC entry template (not in the block template).

View modes and CSS

- Preprocess exposes `block_view_mode`, added as `block-view-mode--{vm}` on the outer `.block`.
- Prefer styling based on bundle + view mode classes, e.g.:
  - `.block-type--media_text.block-view-mode--split_left .block-wrap { grid-template-columns: 1fr 1fr; }`
- Keep structural changes (field composition/formatters) in view modes; use component props for purely stylistic variations.

Working with block items (grids/lists)

- Field template `templates/field/field--block-content--field-block-items.html.twig` wraps items in a unified `.block-items` container and outputs items without extra wrappers.
- SDCs can render additional layout inside `.block-wrap`, but avoid adding a second container around `.block-items` unless necessary.
- For full details and templates, see `custom-block-display.md`.

Overrides and special cases

- If a specific bundle or view mode needs extra wrapper classes/elements, create a more specific block template suggestion and include the required structure:
  - `block--block-content--type--<bundle>--<view_mode>.html.twig`, or other Drupal suggestion variants.
  - Keep: the outer `.block` with standard classes, and one `.block-wrap` inside it.
- Do not add per‑bundle exceptions in the base template.

Preprocess variables

- `gravelle1_preprocess_block()` exposes:
  - `block_view_mode` for custom block content
  - `block_bundle` when available from the rendered entity

Naming and bundle guidance

- Keep block bundles stable and intentional (hero*\*, grid\*\*, media\_*text, quote\_\*, text_rich).
- Reuse field machine names where possible (e.g., `field_block_text`, `field_block_image`, `field_block_link`).
- View mode when changing structure/formatters (e.g., `split_left` vs `split_right`).
- Component props for behavior/style toggles (e.g., density, theme, clickable).

Example: media_text

- Block template: `templates/block/block--block-content--type--media-text.html.twig` extends the base, maps fields, and includes `@gravelle1/components/media_text/media_text.html.twig`.
- CSS targets the standard classes:
  - `.block-type--media_text.block-view-mode--split_left .block-wrap { grid-template-columns: 1fr 1fr; }`

Build and cache

- After changing Twig or SCSS:
  - `cd web/themes/custom/gravelle1 && lando gulp`
  - `lando drush cr`

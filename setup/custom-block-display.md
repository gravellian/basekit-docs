# Custom Block Displays: Unified “block-items” Markup

This site standardizes list‑style content inside custom blocks using a single, reusable pattern called “block-items”. It covers two sources of items:

- Paragraphs referenced by a reusable field (`field_block_items`)
- Views rendered via EVA (Entity View Attachment) on block_content entities

Both outputs share minimal, unified markup so SDCs can style them consistently via one grid selector.

## Goals

- One canonical wrapper: `.block-items`
- Minimal item markup: no extra field/item wrappers
- Reusable across any block bundle and any paragraph types
- Works the same for EVA displays attached to blocks

## Paragraphs: Reusable Reference Field

- Field: `field_block_items` on block_content bundles that need a set/list of items.
- Formatter: “Rendered entity” (view mode can be Default or a custom one); the theme ensures minimal markup when the parent is a custom block.

Templates and theme hooks:

- Field wrapper for block_content
  - `web/themes/custom/gravelle1/templates/field/field--block-content--field-block-items.html.twig`
  - Output: a single `<div class="block-items [block-items--bundle-{bundle}]">` wrapping all items; no per‑item wrappers.

- Minimal paragraph items inside this field
  - `web/themes/custom/gravelle1/templates/content/paragraph--bare.html.twig`
  - `web/themes/custom/gravelle1/gravelle1.theme:1` adds the `paragraph__bare` suggestion only when the parent is `block_content` and the field is `field_block_items`.
  - Result: each paragraph renders its inner fields only, ideal for grid layouts.

Optional hardening (if needed):

- Add a field‑name fallback so any entity’s `field_block_items` uses the wrapper:
  - `templates/field/field--field-block-items.html.twig` (same markup as above).

## EVA: Unified Output from Views

When a view is attached to a custom block via EVA (`display_plugin: entity_view` with `entity_type: block_content`), we normalize its markup:

- Preprocess
  - `web/themes/custom/gravelle1/gravelle1.theme:1` (`gravelle1_preprocess_views_view_unformatted`):
    - Detects EVA displays targeting block_content
    - Adds `is_block_items` flag and `block_items_classes` (includes `.block-items`)
    - Adds template suggestion `views_view_unformatted__eva_block_items`

- Template
  - `web/themes/custom/gravelle1/templates/views/views-view-unformatted--eva-block-items.html.twig`
  - Output: a single `.block-items` wrapper with each row’s content; no per‑row wrappers.

Example EVA config (image slider):

- `config/sync/views.view.block_images.yml:203` → `display_plugin: entity_view`, `entity_type: block_content`

## SDC Integration (per‑bundle components)

Each custom block bundle is implemented as an SDC. The SDC provides the outer chrome; the list/grid itself uses the standardized `.block-items` markup so components don’t need to reinvent list structure.

- grid_topics (Paragraphs)
  - Block template: `web/themes/custom/gravelle1/templates/block/block--block-content--type--grid-topics.html.twig`
  - Passes `{{ content.field_block_items|render }}` to component props; the field template already provides `.block-items` and minimal items.

- image_slider (EVA)
  - Block template: `web/themes/custom/gravelle1/templates/block/block--block-content--type--image-slider.html.twig`
  - Passes the EVA attachment `{{ content.block_images_entity_view_1|render }}` into the component; the EVA template provides `.block-items`.

Wrapper ownership rules to avoid double wrappers:

- Paragraphs: the field template owns the `.block-items` wrapper; components should not add another list wrapper.
- EVA: the views template owns the `.block-items` wrapper; components render the provided markup inside their chrome.

## SDC Data Flow (custom blocks)

Use SDCs to own markup and layout. Pass clean data into SDC props from the block
Twig template.

Flow (example: `grid_topics`)

1) Block template (data only)
- Template: `basekit/templates/block/block--block-content--type--grid-topics.html.twig`
- Reads raw values from the block entity (`field_block_title`, `field_block_text`,
  `field_block_link`) and passes the `field_block_items` paragraph list to the SDC
  as `props.items`.

2) SDC template (markup + wrappers)
- Template: `basekit/components/grid_topics/partials/default.html.twig`
- Loops over `props.items` and reads paragraph entity fields directly:
  - `item.entity.field_topic_title`
  - `item.entity.field_topic_text.processed`
  - `item.entity.field_topic_icon`
  - `item.entity.field_topic_link`
- Emits all wrapper markup and classes used by CSS.

3) Formatting and processing
- Text fields: use `.processed` where available to respect text formats.
- Links: read `title` + `url` from the field item.
- Media: render via Twig Tweak helpers when available (e.g., `drupal_entity()`).

Rules of thumb
- **Data is gathered in block Twig**; **markup is owned by SDC**.
- Avoid `{{ content.field_* }}` in SDCs to prevent field wrapper noise.
- Keep class names in SDC templates in sync with component SCSS.

## Styling: One Grid Selector

- SCSS defines a responsive grid that both paths share:
  - `web/themes/custom/gravelle1/scss/block/_block-items.scss`
  - Forwarded via `web/themes/custom/gravelle1/scss/block/_index.scss:4`
  - Use `.block-items` in component styles for layout; keep type‑specific styles inside each SDC as needed.

## Configuration Steps (Summary)

- For Paragraphs
  - Add `field_block_items` to the block bundle(s) that need a list of items.
  - Set formatter to “Rendered entity” (choose view mode as desired).
  - Create or reuse paragraph types (e.g., `topic_item`) — no special template required because `paragraph--bare` strips wrappers only within this field.

- For EVA
  - Create a view with an EVA display (`display_plugin: entity_view`) targeting `block_content` and the desired bundles.
  - Use “Unformatted list” rows; no per‑row wrapper.
  - Place the EVA display in the block’s view mode (manage display UI) or via config.

## Build & Cache

- Build theme CSS: `cd web/themes/custom/gravelle1 && lando gulp`
- Clear caches: `lando drush cr`

## Troubleshooting

- If the field override doesn’t apply, verify the field name is exactly `field_block_items` and check Twig debug comments for the matched template. Add the fallback `field--field-block-items.html.twig` if needed.
- Ensure block templates render the field/EVA via `|render` so the field/view templates can run.

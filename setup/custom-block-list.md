# Custom Block Types: List, Variants, Uses

This site implements custom blocks as Single‑Directory Components (SDC). Below is a quick reference of each block bundle, its available variants (view modes), and common usage guidance. See docs/setup/custom-blocks.md for implementation details.

## Active Bundles

- hero_announcement
  - Variants (view modes): default, variant_01, variant_02
  - Uses: prominent page announcements, home/section intros, time‑bound alerts. Supports optional image, link, and dense spacing via props.
- hero_headline
  - Variants: default, compact
  - Uses: simple hero headline with optional subtext; use compact for tighter pages or stacked contexts.
- media_text
  - Variants: default, split_left, split_right
  - Uses: 50/50 media and text sections. Choose split_left/right for image position; default for stacked/mobile‑first layouts.
- grid_topics
  - Variants: default
  - Uses: grid of topic/item cards for section overviews or landing pages.
  - Notes: Items come from Paragraphs field `field_block_items` (type: `topic_item`) containing per‑item fields: image, title, text, link.
- quote_feature
  - Variants: default
  - Uses: quotes/testimonials; can be extended with slider props if needed.
- text_rich
  - Variants: default
  - Uses: general rich‑text content blocks with optional CTA.
- media_slider
  - Variants: default, Landscape HD, Cinemascope
  - Uses: multiple image carousel using Swiper Formatter.
  - Notes: Field `field_block_media` (Media reference → Image, multiple). Formatter uses Swiper Entity with media view mode:
    - Landscape HD → `main_100_16x9`
    - Cinemascope → `main_100_21x9`

## Legacy/Transitional

- banner_headline
  - Variants: default, banner_announcement_01, banner_announcement_02
  - Uses: legacy predecessor to hero_* blocks. Prefer hero_headline/hero_announcement for new content.
- basic (core)
  - Variants: default
  - Uses: Drupal core “Basic” custom block; keep for utility or admin‑only content.

## Notes

- Variants are implemented as Drupal view modes and are mapped by the block’s Twig template to the corresponding SDC partial.
- Styling/behavioral differences that don’t change field composition should use SDC props rather than new view modes.
- Component source: `web/themes/custom/gravelle1/components/<bundle>/` with SCSS compiled to `components/<bundle>/<bundle>.css` and attached via `gravelle1.libraries.yml`.

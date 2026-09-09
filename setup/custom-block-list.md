# BaseKit Component Library

This is the concise inventory of BaseKit's target portable component contract.
The naming and separation rules in
[BaseKit Custom Block Architecture](custom-block-architecture.md) are
canonical.

## Unified component types

- `rich_text` — formatted editorial copy with an optional title and link.
- `hero` — prominent introductory content; named variants include `headline`,
  `announcement`, and `photo_banner`.
- `split_image` — text paired with a direct Drupal image field.
- `split_media` — text paired with a reusable Drupal Media entity.
- `slider_image` — slider backed by direct image-field items.
- `slider_media` — slider backed by reusable Media entities.
- `quote` — semantic quotation or testimonial content.
- `grid_items` — manually authored structured items; `topics` is the initial
  presentation.
- `content_listing` — View- or query-backed existing content; presentations
  include `grid`, `list`, and `contacts`.

Variants and styles must have descriptive machine names. Opaque sequential
names such as `display_001` and `variant_01` are not part of the contract.

## Migration aliases

| Existing bundle | Target |
| --- | --- |
| `basic`, `block_custom`, `text_rich` | `rich_text` |
| `hero_headline`, `hero_announcement`, `photo_banner` | `hero` variants |
| `media_text` | `split_image` |
| `image_text_split` | `split_media` |
| `image_slider` | `slider_image` |
| `media_slider` | `slider_media` |
| `quote_feature` | `quote` |
| `grid_topics` | `grid_items` |
| `contacts_grid` | `content_listing` |

These are migration targets, not permission to rename configuration in place.
Existing data remains supported until a tested field-level migration is
available. UAN's motion components remain site-specific during their
accessibility and behavior review.

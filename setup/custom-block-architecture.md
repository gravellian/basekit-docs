# BaseKit Custom Block Architecture

This document is the canonical contract for reusable custom blocks in BaseKit.
It supersedes the architectural guidance scattered across the older custom
block, display, and block-style notes. Those documents may remain as focused
how-to references, but new work and migrations should follow this contract.

## Goal

BaseKit provides a portable library of Drupal custom block types. A block has
the same content model, editing experience, display choices, markup contract,
and baseline behavior on every BaseKit site. Each site's `basekit_site`
subtheme then expresses that block in the site's own visual language.

Portability means a block can move between BaseKit sites without rebuilding
fields, templates, or behavior. It does not mean every site must look alike.

## Ownership

Each concern has one owner.

| Concern | Owner | Examples |
| --- | --- | --- |
| Content model and Drupal display configuration | `basekit-recipe` | Block bundles, fields, form displays, view displays, view modes, supporting media/paragraph configuration, Layout Builder styles |
| Reusable rendering and baseline presentation | `basekit` theme | Preprocess data, bundle templates, SDC props and markup, component libraries, default SCSS, JavaScript, stable CSS classes |
| Brand and site presentation | Site `basekit_site` subtheme | Color and type tokens, spacing, imagery, site layout, deliberate component skin overrides |
| Installation and updates | BaseKit project/tooling | Applying recipes, installing packages, rebuilding assets, validation and deployment |
| Editorial content | Individual site | Block instances, field values, placement, selected view mode and approved style classes |

The site theme must not redefine a portable block's fields or basic DOM
structure. The recipe must not contain brand styling. The base theme must not
contain client-specific content or identity.

## Rendering pipeline

The supported path is:

```text
recipe configuration
  -> block content entity + selected view mode
  -> BaseKit preprocess variables
  -> BaseKit bundle template
  -> BaseKit SDC props
  -> SDC view-mode partial
  -> BaseKit component library
  -> site token and presentation overrides
```

The bundle template is an adapter. It gathers Drupal render arrays or entity
values and passes a small, named props object to the Single Directory
Component (SDC). The SDC owns component markup. The subtheme styles the public
component contract and should not copy the Twig merely to change appearance.

## Public markup contract

Every portable custom block must expose these outer classes:

```html
<div class="block block-type--{bundle} block-view-mode--{view_mode}">
  <div class="block-wrap">
    <!-- component markup -->
  </div>
</div>
```

Rules:

- `block--block-content.html.twig` owns the single `.block-wrap` element.
- An SDC or view-mode partial must not add another `.block-wrap`.
- Repeated content uses `.block-items` around its item collection.
- Component-specific elements use stable, semantic classes; changing them is
  a public-contract change because site subthemes may target them.
- Bundle and view-mode classes are the preferred styling hooks. Do not target
  generated block UUID classes for reusable presentation.
- Drupal administrative attributes and render arrays must remain intact.

## Bundle names and separation rules

Bundle machine names describe the component's stable content contract, not a
particular site's appearance. Use lower-case, family-first identifiers with
underscores so related types sort together, for example `slider_image`,
`slider_media`, `split_image`, and `split_media`.

Create a new bundle when one or more of these are true:

- the fields or validation rules differ materially;
- the content has a different editorial or semantic meaning;
- the allowed asset types or authoring workflow differ;
- accessible markup or required behavior differs; or
- changing between the two would discard or reinterpret stored content.

Use a named variant when the same stored content supports both renderings and
only layout, emphasis, proportion, or treatment changes. Use a controlled
style class for small, independent choices such as spacing, width, surface,
text alignment, or edge-to-edge treatment. Use a Drupal display mode when the
rendering changes because of context, or when formatters, field order, aspect
ratio, or markup change materially.

Do not create opaque identifiers such as `display_001`, `variant_01`, or
`style_02`. Machine names are permanent configuration and integration
contracts. Give them stable semantic names such as `headline`, `photo_banner`,
`image_left`, `compact`, or `full_width`; labels may be more conversational.

Matching field sets strongly favor one bundle with variants, but field parity
alone is not sufficient. A quotation, for example, remains a separate
component because its semantic and accessibility contract differs from an
ordinary rich-text feature even if an older implementation happens to reuse
the same fields.

### Image and Media pairs

BaseKit may provide paired Image and Media bundles when both Drupal authoring
models serve real needs. The Image form uses a direct image field for a simple,
one-off upload. The Media form uses an entity reference for reuse, centralized
metadata, and support for multiple allowed media bundles. Paired bundles share
the same normalized SDC props and frontend presentation API wherever possible;
their storage and editorial contracts remain explicit in their names.

## View modes, variants, and styles

Use a **view mode** when the editor is choosing a consequential rendering
contract: field formatter, aspect ratio, field order, or materially different
markup. Each enabled custom-block view mode must have:

1. Recipe configuration for the entity view display.
2. An identically named SDC partial, or an explicitly documented decision to
   share the default partial.
3. A declared library and baseline styling where behavior requires it.
4. A fixture or real block that can be used for visual regression checking.

Use an SDC prop or modifier class when the difference is presentational and
does not require different Drupal display configuration. Use a Layout Builder
style class for contextual container treatment such as `box1`, `box2`,
`box1-items`, `box2-items`, or `wide`.

Use the recipe-provided `full_bleed` style (`block-full-bleed` in rendered
markup) when a block's background should reach both viewport edges while its
content remains on BaseKit's stepped `.sitew`/`.pagep` canvas. The style is a
background-painting primitive; it must not move the block content out of the
normal layout or add a second horizontal content gutter.

Names use lower-case semantic machine identifiers with underscores for Drupal
view modes and matching partial filenames. CSS classes use hyphens.

Unknown view modes may fall back to `default` so a page still renders, but the
fallback must be visible in automated validation; it must not hide an
incomplete component implementation.

## Styling contract

BaseKit component SCSS defines layout, responsive behavior, accessibility,
interaction, and safe defaults. Site SCSS supplies the brand.

A conforming site should:

- define the complete supported brand-token set in `_tokens.scss`;
- set typography through tokens and CSS custom properties;
- configure shared block treatments through BaseKit's Sass configuration APIs;
- keep component overrides in `scss/components/_<component>.scss`;
- scope overrides under `.block-type--<bundle>` and, when needed,
  `.block-view-mode--<view_mode>`;
- override color, type, spacing, surface, border, shadow, and decorative media
  without replacing shared structure or behavior;
- compile and commit production CSS from the committed SCSS sources.

Hard-coded values are acceptable for a genuinely unique brand decision, but
repeated values belong in tokens. `!important`, generated IDs, deep selectors,
and fixed geometry should be treated as warnings and justified during review.

## Hero headline presentation API

For `hero_headline` in its `default` view mode, the component owns the media
selector and SVG integration. Subthemes set these CSS custom properties on
`.block-type--hero_headline.block-view-mode--default`, using values from their
own `_tokens.scss`:

| Property | BaseKit fallback |
| --- | --- |
| `--hero-headline-icon-width` | `7em` |
| `--hero-headline-icon-height` | `auto` |
| `--hero-headline-icon-border` | `0 none` |
| `--hero-headline-icon-padding` | `0` |
| `--hero-headline-icon-radius` | `0` |
| `--hero-headline-icon-background` | `transparent` |
| `--hero-headline-icon-shadow` | `none` |
| `--hero-headline-icon-color` | BaseKit primary color |

These controls preserve the component's existing max-width/max-height limits.
They do not change the content model or apply automatically to other view modes.
The existing `background_banner` controls retain their separate
`--hero-headline-banner-*` namespace. A subtheme must not target `#icon-primary`
or add `!important` simply to skin the default icon.

Keep palette colors distinct from semantic roles. For example, `$on-color`
can be white while `$brand-light` is cream. Hero-heading and shadow roles may
reference those colors without scattering literals through component files.
Fixed decorative dimensions are permitted as named site tokens when they are
intentional; moving a value into a token does not make it intrinsically responsive.

BaseKit component CSS is compiled separately from subtheme Sass. Forwarding a
site Sass token alone cannot recolor an already-compiled component. Use its
documented CSS properties or a scoped site presentation override; extend the
shared API when another view mode needs a reusable control.

Release the BaseKit component change before publishing subtheme CSS that
depends on these properties, and update each site's Composer lock. For a
presentation-only update, do not import the site's full Drupal configuration.

## Configuration lifecycle

Portable configuration is authored and reviewed in `basekit-recipe`. A site
receives it through the BaseKit install/update workflow. Site configuration
exports may contain active copies because Drupal configuration management is a
full-site snapshot, but those copies are mirrors, not independent sources of
truth.

Therefore:

- change a shared block type in `basekit-recipe` first;
- change shared rendering in `basekit` first;
- update sites from those repositories rather than hand-editing equivalent
  YAML independently in several site repositories;
- record any intentional site divergence and test it during BaseKit updates;
- pin or otherwise record the exact recipe revision used by a site so the
  configuration snapshot is reproducible.

Recipe application must be additive and safe for an existing site. Deleting a
field, bundle, or view mode requires a migration and an explicit editorial-data
review.

## Definition of done for a portable block

A block is part of the BaseKit library only when all of the following are true:

- Its bundle, fields, form display, default display, and supported view modes
  are owned by `basekit-recipe`.
- A BaseKit bundle template maps Drupal data into a documented SDC prop schema.
- Every supported rendering path has component markup and its required library.
- The outer markup follows the public contract and remains accessible without
  site-specific CSS.
- At least two differently branded subthemes can style it without copying Twig
  or changing its content model.
- Its configuration and component have automated structural checks.
- Its default, empty/optional-field, responsive, and interactive states have
  been reviewed.
- Update and rollback notes exist for consequential schema changes.

## Audit checklist for a site

Before styling or migrating a BaseKit site, verify:

1. The site records the BaseKit theme and recipe revisions it uses.
2. Shared block bundles, fields, displays, and view modes match the recipe, or
   each divergence is documented.
3. Every enabled view mode resolves to the intended SDC partial.
4. The site theme contains no copied portable-block Twig or component markup.
5. The site supplies the full token contract and imports current BaseKit Sass
   APIs.
6. Component overrides are scoped and limited to presentation.
7. Production CSS is rebuilt from the current SCSS with the standard build.
8. Representative blocks are checked at mobile and desktop widths before
   deployment.

## Target portable block library

The unified BaseKit contract is organized around nine content models:

| Bundle | Contract |
| --- | --- |
| `rich_text` | Optional title, formatted body, and optional link; replaces `basic`, `block_custom`, and `text_rich` for portable content. |
| `hero` | Prominent title, text, action, and optional visual content, with named `headline`, `announcement`, and `photo_banner` variants. |
| `split_image` | Text and a direct Drupal image field, with named image-position and proportion variants. |
| `split_media` | The same presentation contract backed by a Drupal Media reference. |
| `slider_image` | A collection of direct image-field items. |
| `slider_media` | A collection of reusable Media entities. |
| `quote` | Quotation or testimonial content with semantic quotation markup and attribution support. |
| `grid_items` | A manually authored collection of structured items, initially including the `topics` presentation. |
| `content_listing` | A query- or View-backed listing of existing entities, with presentations such as `grid`, `list`, and `contacts`. |

The existing bundles remain supported during migration. Their target mappings
are `text_rich`/`basic`/`block_custom` to `rich_text`,
`hero_headline`/`hero_announcement`/`photo_banner` to `hero`, `media_text` to
`split_image`, `image_text_split` to `split_media`, `image_slider` to
`slider_image`, `media_slider` to `slider_media`, `quote_feature` to `quote`,
`grid_topics` to `grid_items`, and `contacts_grid` to `content_listing`.

Do not delete or rename an installed bundle merely to match this table. Each
mapping requires a field-level data migration, compatibility rendering during
the transition, and an editorial review. Site-specific motion components such
as UAN's `scrolling_text` and `scrolling_banner` remain outside the portable
contract until their behavior and reduced-motion accessibility are ready.

## Runtime body-copy scale

BaseKit provides the opt-in `scss/_copy-scale.scss` API, used by GC, JSG and
the starter templates. Existing sites do not change until they opt in. The legacy
Sass `$font-size-body` still feeds compiled defaults; changing it alone does
not update independently compiled BaseKit components.

| CSS token | Role |
| --- | --- |
| `--font-size-body` | Site's standard copy size |
| `--font-size-body-small` | Standard × 0.85 |
| `--font-size-body-large` | Standard × 1.15 |
| `--font-size-body-ui` | Standard × 0.65 |

Each subtheme defines `$body-size` and the three ratio tokens in `_tokens.scss`.
GC uses `max(18px, 1.85rem)`; JSG uses `max(18px, 1.9rem)`. Shared token names
and roles do not require identical brand typography. Root-relative sizes avoid
compounding through nested component wrappers.

`_typography.scss` includes the shared mixin rather than copying its selectors:

```scss
@use 'tokens' as site;
@use 'copy-scale';

@include copy-scale.install(
  $body-size: site.$body-size,
  $small: site.$body-small-ratio,
  $large: site.$body-large-ratio,
  $ui: site.$body-ui-ratio
);
```

The mixin publishes the CSS tokens, and `.body-style` connects
`--body-style-font-size` to `--font-size-body`. The `copy-small` and `copy-large`
classes (and `copy-ui`) belong on the `.body-style` wrapper. Other deliberate ratios may use
`calc(var(--font-size-body) * 0.9)` on the intended copy element. Grid Topics'
`.topic-text`/legacy `.text` and utility-belt breadcrumbs have explicit adapters. Other legacy
components and interface styles still require an audit; this is not a claim
that every text rule has been converted.

Keep heading sizes, brand colors, prose rhythm, and layout gutters separate
from this copy scale. Text fields fill their assigned layout regions; do not
add universal maximum field widths or automatic multi-column text flow.

Before migrating Urban Art Network's `dev.urbanartnetwork.org1`, compare its
block schema/rendering with this contract and implement this same copy API in
its subtheme. Preserve its brand choices. `dev.urbanartnetwork.org1-compare`
represents the live-site comparison; reconcile newer production content before
any migration deployment. Do not replace production content from a stale dev
database. Validate body and block copy plus small/large/UI roles at mobile and
desktop sizes before releasing the shared implementation.

## Architectural change process

When a site needs behavior the library does not provide:

1. Decide whether the need is reusable content structure, reusable rendering,
   or site-only presentation.
2. Implement it in the owner repository from the table above.
3. Add or update structural validation and a representative fixture.
4. Test it in two differently branded BaseKit sites.
5. Document any public class, prop, view-mode, or migration change.
6. Update consuming sites through the normal BaseKit update workflow.

This keeps GC, JSG, and future sites as consumers of one evolving component
system instead of parallel forks of it.

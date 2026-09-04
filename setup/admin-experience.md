# BaseKit administration and form-display plan

BaseKit standardizes on Gin for Drupal's administrative shell while each site
subtheme remains the source of truth for frontend rendering. Claro is the core
fallback. BaseKit does not subtheme Gin and does not load a site's complete
frontend stylesheet over administrative forms.

## Editing and visual fidelity

The theme surrounding a node form and the stylesheet inside CKEditor are
separate concerns. A node form may use Gin, while each CKEditor instance loads
the BaseKit editor stylesheet plus the active subtheme's editor overrides. The
editor stylesheet intentionally reproduces rendered body typography, semantic
classes, content width, and approved colors. It does not reproduce page chrome,
layout regions, or global frontend rules for buttons, tables, and form inputs.

The existing CKEditor presentation contract is sufficient for this phase. A
future preview surface should render through the frontend theme when authors
need fidelity beyond the editable body field.

## Ownership

- `basekit-project` requires Gin for new sites and installs it before recipes.
- `basekit-recipe` declares and installs Gin, selects it as the admin theme,
  and enables the admin theme for node create/edit routes.
- `basekit` and site subthemes continue to own frontend and editor rendering.
- A future small `basekit_admin` module may hold proven integration fixes; it
  must not become a replacement admin theme.
- Site recipes own form-display configuration for portable content types and
  block bundles. Site-specific fields remain site configuration.

## Form-display configuration

Form displays are ordinary Drupal configuration and can be included in the
recipe that owns the corresponding entity bundle. Begin by exporting a
carefully arranged form from the reference site, review the YAML, and copy the
portable configuration into that recipe's `config/` directory. Existing sites
must reconcile their active and exported configuration before adopting it.

### Example: Page node form

`recipes/pages/config/core.entity_form_display.node.page.default.yml` can make
the primary authoring flow predictable:

```yaml
content:
  title:
    type: string_textfield
    weight: -20
    region: content
  body:
    type: text_textarea_with_summary
    weight: -10
    region: content
  status:
    type: boolean_checkbox
    weight: 90
    region: content
  created:
    type: datetime_timestamp
    weight: 91
    region: content
  uid:
    type: entity_reference_autocomplete
    weight: 92
    region: content
hidden:
  promote: true
  sticky: true
```

The complete exported file also contains `uuid`, `langcode`, `dependencies`,
`id`, `targetEntityType`, `bundle`, `mode`, and third-party settings. Preserve
those generated values rather than hand-authoring a partial file. The important
policy is stable order: title and body first, publishing metadata later, and
unused controls hidden.

If BaseKit adopts Field Group later, the same display can place metadata in a
collapsed `Advanced` details group. That should be a deliberate contributed
module dependency, not assumed by the initial Gin rollout.

### Example: reusable image-and-text block form

The recipe that owns a custom block bundle can ship
`core.entity_form_display.block_content.image_text.default.yml`:

```yaml
content:
  info:
    type: string_textfield
    weight: -30
    region: content
  field_heading:
    type: string_textfield
    weight: -20
    region: content
  field_body:
    type: text_textarea
    weight: -10
    region: content
  field_media:
    type: media_library_widget
    weight: 0
    region: content
  field_link:
    type: link_default
    weight: 10
    region: content
hidden:
  langcode: true
```

Again, use Drupal's full exported configuration in the real recipe. Arrange
fields in the order authors think about the component, choose a purpose-built
widget such as Media Library, and hide fields that do not belong in the normal
authoring decision.

## Rollout phases

1. Require Gin 5, install it, and set `system.theme:admin` to `gin` for fresh
   BaseKit installations. Set `node.settings:use_admin_theme` to `true` so node
   create/edit forms actually use Gin rather than the frontend theme.
2. Adopt Gin on local BaseKit sites without changing frontend themes or editor
   configuration; validate node, block, media, user, Views, and Webform screens.
3. Record recurring friction before adding Gin settings or custom CSS.
4. Ship reviewed form displays one bundle at a time through their owning
   recipes, with active/config-sync reconciliation on existing sites.
5. Add a `basekit_admin` module only when repeated cross-site requirements
   cannot be expressed cleanly as configuration.

Do not couple Gin adoption to frontend redesign, CKEditor vocabulary changes,
or production deployment. Those remain independently reviewable changes.

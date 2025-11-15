# Image Styles

This project uses region‑first media view modes and responsive image styles for the "main" content region. Width variants include 100/66/50/33/25, with optional aspect ratios (1x1, 4x3, 16x9, 21x9).

The goal is consistency across editor previews and frontend output:
- View modes (e.g., `main_50`, `main_50_4x3`) match CSS selectors used by the theme.
- CKEditor renders the same widths using the same `display--main_*` classes.

Breakpoints are defined in the theme: `web/themes/custom/gravelle1/gravelle1.breakpoints.yml`
- handheld: `all and (max-width: 767px)`
- desktop-small: `all and (max-width: 1023px) and (min-width: 768px)`
- desktop-medium: `all and (max-width: 1364px) and (min-width: 1024px)`
- desktop-large: `all and (max-width: 1919px) and (min-width: 1365px)`
- desktop-max: `all and (min-width: 1920px)`

How to configure (in this site):

1) Create Media view modes for Image
- Structure → Display modes → View modes → Media: add view modes named `main_100`, `main_100_4x3`, `main_100_16x9`, `main_100_21x9`, `main_66`, `main_66_4x3`, `main_66_16x9`, `main_50`, `main_50_1x1`, `main_50_4x3`, `main_50_16x9`, `main_33`, `main_33_1x1`, `main_33_4x3`, `main_33_16x9`, `main_25`, `main_25_1x1`, `main_25_4x3`, `main_25_16x9` (adjust to your needs).

2) Configure the Image media type display per view mode
- Structure → Media types → Image → Manage display → select the new view mode from the tab.
- Set the image field formatter to “Responsive image” and choose a responsive image style that corresponds to the view mode name (e.g., `main_50_4x3`).
- For cropped variants, use Focal Point Scale and Crop in the underlying image styles.

3) Allow these view modes in the text format used by CKEditor
- `config/sync/filter.format.article.yml` lists allowed view modes for `<drupal-media>`. Ensure your new modes are included.

4) Theme integration (already wired)
- The Twig template `web/themes/custom/gravelle1/templates/media/media--image.html.twig` adds `display--<view_mode>` and `media-ext--<ext>` classes.
- SCSS targets these via `.media[class^='display--main_*']` and `figure[data-view-mode^='main_*']` in:
  - `web/themes/custom/gravelle1/scss/editor-styles.scss` (editor preview)
  - `web/themes/custom/gravelle1/scss/base/var/_body-style.scss` (frontend body copy)

Mapping to CSS selectors
- `main_100` → 100% width at desktop breakpoints
- `main_66` → ~65%
- `main_50` → ~49%
- `main_33` → ~32%
- `main_25` → ~24%

Tips
- Keep view mode names and responsive image styles in sync to reduce confusion.
- Use focal point for crops so editors can control subject framing.
- Export config when changes are made: `lando drush cex -y` (then commit YAML under `config/sync/`).

## Inventory (current config)

- main_100
  - main_100_max → desktop-max (1587)
  - main_100_large → desktop-large (1128)
  - main_100_med → desktop-medium (846)
  - main_100_small → desktop-small (635)
  - main_100_handheld → handheld (705)
- main_100_16x9
  - main_100_16x9_max → desktop-max (1587x893)
  - main_100_16x9_large → desktop-large (1128x635)
  - main_100_16x9_med → desktop-medium (846x475)
  - main_100_16x9_small → desktop-small (635x357)
  - main_100_16x9_handheld → handheld (705x396)
- main_100_21x9
  - main_100_21x9_max → desktop-max (1587x680)
  - main_100_21x9_large → desktop-large (1128x483)
  - main_100_21x9_med → desktop-medium (846x363)
  - main_100_16x9_small → desktop-small (635x357)
  - main_100_1x1_handheld → handheld (705x705)
- main_100_4x3
  - main_100_4x3_max → desktop-max (1587x1190)
  - main_100_4x3_large → desktop-large (1128x846)
  - main_100_4x3_med → desktop-medium (846x635)
  - main_100_4x3_small → desktop-small (635x476)
  - main_100_4x3_handheld → handheld (705x529)
- main_25
  - main_25_max → desktop-max (296)
  - main_25_large → desktop-large (210)
  - main_25_med → desktop-medium (157)
  - main_25_small → desktop-small (118)
  - main_50_handheld → handheld (340)
- main_25_1x1
  - main_33_1x1_max → desktop-max (511x511)
  - main_25_1x1_large → desktop-large (210x210)
  - main_25_1x1_med → desktop-medium (157x157)
  - main_25_1x1_small → desktop-small (118x118)
  - main_50_1x1_handheld → handheld (340x340)
- main_33
  - main_33_max → desktop-max (511)
  - main_33_large → desktop-large (364)
  - main_33_med → desktop-medium (273)
  - main_33_small → desktop-small (205)
  - main_100_handheld → handheld (705)
- main_33_1x1
  - main_33_1x1_max → desktop-max (511x511)
  - main_33_1x1_large → desktop-large (364x364)
  - main_33_1x1_med → desktop-medium (273x273)
  - main_33_1x1_small → desktop-small (205x205)
  - main_100_4x3_handheld → handheld (705x529)
- main_33_4x3
  - main_33_4x3_max → desktop-max (511x383)
  - main_33_4x3_large → desktop-large (364x273)
  - main_33_4x3_med → desktop-medium (273x205)
  - main_33_4x3_small → desktop-small (205x154)
  - main_100_4x3_handheld → handheld (705x529)
- main_50
  - main_50_max → desktop-max (1048)
  - main_50_large → desktop-large (747)
  - main_50_med → desktop-medium (416)
  - main_50_small → desktop-small (312)
  - main_100_handheld → handheld (705)
- main_50_16x9
  - main_50_16x9_max → desktop-max (780x439)
  - main_50_16x9_large → desktop-large (555x312)
  - main_50_16x9_med → desktop-medium (416x234)
  - main_50_16x9_small → desktop-small (312x175)
  - main_100_4x3_handheld → handheld (705x529)
- main_50_1x1
  - main_50_1x1_max → desktop-max (780x780)
  - main_50_1x1_large → desktop-large (555x555)
  - main_50_1x1_med → desktop-medium (416x416)
  - main_50_1x1_small → desktop-small (312x312)
  - main_100_1x1_handheld → handheld (705x705)
- main_50_4x3
  - main_50_4x3_max → desktop-max (780x780)
  - main_50_4x3_large → desktop-large (555x555)
  - main_50_4x3_med → desktop-medium (416x416)
  - main_50_4x3_small → desktop-small (312x312)
  - main_100_4x3_handheld → handheld (705x529)
- main_66
  - main_66_max → desktop-max (1048)
  - main_66_large → desktop-large (747)
  - main_66_med → desktop-medium (560)
  - main_66_small → desktop-small (421)
  - main_100_handheld → handheld (705)
- main_66_16x9
  - main_66_16x9_max → desktop-max (1048x590)
  - main_66_16x9_large → desktop-large (747x420)
  - main_66_16x9_med → desktop-medium (560x315)
  - main_66_16x9_small → desktop-small (421x237)
  - main_100_1x1_handheld → handheld (705x705)
- main_66_4x3
  - main_66_4x3_max → desktop-max (1048x786)
  - main_66_4x3_large → desktop-large (747x560)
  - main_66_4x3_med → desktop-medium (560x420)
  - main_66_4x3_small → desktop-small (421x316)
  - main_100_1x1_handheld → handheld (705x705)

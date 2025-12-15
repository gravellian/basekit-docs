# Block Styles and Overrides

BaseKit ships with reusable block wrappers (`.box1`, `.box2`, etc.) that are exposed through Layout Builder Styles. The Sass for these wrappers lives in `themes/contrib/basekit/scss/block/_block-styles.scss` and now supports both classic block wrappers and the corresponding `-items` variants that push the chrome down onto each `.block-items` child (paragraphs or view rows). Each wrapper still responds to the legacy hyphenated names (`.box-1`, `.box-2`, etc.) so existing sites continue to work, but new installs should standardize on the condensed class names (`.box1`, `.box2`, `.box1-items`, `.box2-items`).

This page explains how the shared configuration works and how to override or extend the styles inside your sub theme (`basekit_site`).

## 1. How the shared styles are structured

* `_block-styles.scss` imports a helper file, `_box-config.scss`, that contains a `$box-style-map` definition. Each entry (e.g. `box1`, `box2`) defines padding, backgrounds, heading colors, and reverse/linked variants.
* The map powers a couple of mixins (`box-element`, `box-modifiers`, `box-paragraph`) that are applied to both `.box-*` and `.box-*-items` selectors, so each class and its `-items` counterpart always stay in sync.
* Each block wrapper automatically zeroes padding/background via the `block-style-reset` mixin whenever you use the `-items` class, so only child items receive the box treatment.

## 2. Customizing styles in the base theme

If you need to tweak the defaults for every project (e.g., adjust padding on `box1`):

1. Edit `themes/contrib/basekit/scss/block/_box-config.scss`.
2. Update the relevant values inside `$box-style-map`.
3. Rebuild Sass (`npm run build` or `gulp`) from the base theme root.

The updates flow to both `.box-*` and `.box-*-items` automatically—no need to touch `_block-styles.scss`.

## 3. Overriding block styles in the sub theme

To change styles per project without altering the base theme, leverage the `$box-style-overrides` hook that `_block-styles.scss` now exposes. Inside your sub theme (e.g., `themes/custom/basekit_site/scss/block/_overrides.scss`), import `box-config` with a custom map:

```scss
@use 'sass:map';
@use '../../contrib/basekit/scss/block/box-config' with (
  $box-style-overrides: (
    'box1': (
      padding-mobile: 2em,
      padding-desktop: 3.5em,
      background: rgba(#ffffff, 0.85),
      heading-color: #1b1b1b,
    ),
    'box2': (
      reverse-background: #3b3b3b,
      reverse-button-background: #f5f5f5,
      reverse-button-color: #1b1b1b,
    ),
  )
);

@use '../../contrib/basekit/scss/block/block-styles';
```

Notes:

* Adjust the relative paths to match your sub theme’s directory structure.
* Only include the properties you want to override; everything else falls back to the base theme defaults thanks to `map.deep-merge`.
* If you need entirely new styles (e.g., `box-3`), add a new entry to `$box-style-overrides` (or `_box-config.scss` if the style should exist everywhere) and then expose the class through Layout Builder Styles configuration.

## 4. Applying styles in Layout Builder

1. Use the Layout Builder Styles UI to add style definitions that point to the classes you need (`box1`, `box1-items`, `box2`, `box2-items`, etc.). The Sass still recognizes the old hyphenated names, but new config should prefer the condensed IDs to keep things consistent.
2. When editing a layout, select the desired class. Choosing a `-items` variant will remove padding/background from the block wrapper and apply the box treatment to each paragraph/view item.
3. Any overrides you configured in your sub theme are reflected immediately once your Sass is rebuilt.

With this setup you keep the styling logic DRY, deliver consistent experiences across block wrappers and item grids, and maintain full control in each sub theme without touching the core BaseKit Sass.

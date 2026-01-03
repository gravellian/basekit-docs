# BaseKit tokens (var_default)

Source of truth: `basekit/scss/base/var/_var_default.scss`.
Override these in your subtheme via:

```scss
// web/themes/custom/basekit_site/scss/_tokens.scss
@forward 'base/var/var_default' with (
  $brand-primary: #4b7bec,
  $font-size-h1: 3.5em
);
```

## Brand + palette

```scss
$brand-bg
$brand-primary
$brand-accent
$brand-neutral
```

## Body + page colors/alignment

```scss
$site-bgcolor
$page-bgcolor
$body-align
$page-align
```


## Layout formatting

```scss
$m-page-header
$d-page-header
$siteg
$m-pagep
$d-pagep
$m-pagew
$d-pagew
$pagepad
$m-page-footer
$d-page-footer
$m-section-h2-height
$d-section-h2-height
```

## Text ramp + roles

```scss
$text-base
$text-bright
$text-high
$text-muted
$text-subtle
$text-muted-alpha
$text-subtle-alpha
$color-text
```

## Ramps + surfaces

```scss
$primary-light
$primary-dark
$accent-light
$accent-dark
$neutral-light
$neutral-dark
$surface-0
$surface-1
$surface-2
```

## Rules + focus

```scss
$rule
$border
$focus-ring
$selection
```

## Utility colors

```scss
$gray
$gray-dark
$white
$red
$teal
$orange
$violet
$green
$yellow
$black
```

## Headings (colors, sizes, weights)

```scss
$color-h1
$font-weight-h1
$font-size-h1
$line-height-h1

$color-h2
$font-weight-h2
$font-size-h2
$line-height-h2

$color-h3
$font-weight-h3
$font-size-h3
$line-height-h3

$color-h4
$font-weight-h4
$font-size-h4
$line-height-h4

$color-h5
$font-weight-h5
$font-size-h5
$line-height-h5

$color-h6
$font-weight-h6
$font-size-h6
$line-height-h6
```

## Body type

```scss
$font-weight-body
$font-weight-body-light
$font-weight-body-medium
$font-size-body
$line-height-body

$font-size-small
```

## Stroke + borders

```scss
$stroke-1
$stroke-2
$stroke-3
$border-light
$border-dark
```

## Sections + blocks

```scss
$section-bgcolor
$block-bgcolor
$block-pad-top
$block-pad-right
$block-pad-bottom
$block-pad-left
$block-border-radius
$block-pad
```

## Nav + link sizing

```scss
$nav-align
$link-pad-top
$link-pad-right
$link-pad-bottom
$link-pad-left
$link-pad
$m-link-size
$d-link-size
$link-height
$link-space
$d-link-space
$link-radius
```

## Menu link colors

```scss
$link-color
$link-hover-color
$link-click-color
$link-bgcolor
$link-hover-bgcolor
$link-click-bgcolor
$link-border
$link-hover-border
$link-click-border
```

## Active menu links

```scss
$link-isactive-color
$link-isactive-hover-color
$link-isactive-click-color
$link-isactive-bgcolor
$link-isactive-hover-bgcolor
$link-isactive-click-bgcolor
$link-isactive-border
$link-isactive-hover-border
$link-isactive-click-border
```

## Tables

```scss
$td-bg
```

## Forms

```scss
$form-pad
$form-border
$form-td-pad
$form-td-border
$form-td-bg
$form-td-bg2
$form-wrap-border
$form-wrap-pad
$form-wrap-bgcolor
$form-summary-size
$form-item-pad
$form-item-border
$form-item-after
$form-label-size
$form-input-border
$form-input-pad
$form-input-color
$form-input-size
$form-input-bgcolor
$form-select-pad
$form-select-bgcolor
$form-select-border
$form-select-size
```

## Buttons

```scss
$button-size
$button-height
$button-pad-top
$button-pad-right
$button-pad-bottom
$button-pad-left
$button-pad
$button-radius
$button-color
$button-bgcolor
$button-border
$button-hover-color
$button-hover-bgcolor
$button-hover-border
$button-click-color
$button-click-bgcolor
$button-click-border
$button-active-color
$button-active-bgcolor
$button-active-border
```

## Default images (data URIs)

```scss
$page-image-custom
$page-image-config
$page-image-article
$default-image-1x1
$default-profile-1x1
$notice
```

## Responsive Media setup

Responsive media styles rely on these Drupal modules:

- Media (core)
- Responsive Image
- Breakpoint
- CKEditor
- Text Editor
- Focal Point (if using focal point scale-and-crop)

Rather than naming responsive styles after content types or templates, I use region‑based names (for example, "main") that describe where the media appears in the layout. Regions like the primary content column are consistent across pages and content types, and their widths are already defined by our breakpoints. A region‑first convention decouples display semantics (where/how big) from content semantics (what/type), so the same view mode can be reused in Articles, Basic Pages, Paragraphs, and Blocks. It also survives design refactors: if the main column changes width at certain breakpoints, the styles behind a name like "main_50" can evolve without renaming. This makes the system predictable, scalable, and easy to map to breakpoints and SCSS. The scheme also extends naturally to other regions (e.g., "sidebar", "hero") as needed.

main_100_16x9 - applies to the "main" region, 100% width, 16x9 is the aspect ratio when the image is cropped

main_50_1x1 - applies to the "main" region, 50% width, 1x1 is the aspect ratio when the image is cropped

main_100 - applies to the "main" region, 100% width, no cropping

Images are cropped using the "Focal Point Scale and Crop" effect when an aspect ratio is specified.

See the list of responsive media styles in [image-styles.md](image-styles.md)

- **web/themes/custom/gravelle1/templates/media/media--image.html.twig**

  Adds display--<view_mode> and media-ext--<ext> class to media wrappers.

```twig
{% set classes = [
  'media',
  'media-type--' ~ content.field_media_image['#field_type']|default('unknown'),
  'display--' ~ content.field_media_image['#view_mode']|default('unknown'),
  'media-ext--' ~ file_extension|default('unknown'),
] %}

<div{{ attributes.addClass(classes) }}>
  {{ title_suffix.contextual_links }}
  {{ content }}
</div>
```

- **web/themes/custom/gravelle1/gravelle1.info.yml**

  Registers CKEditor 5 stylesheet assets/css/editor-styles.css.

- **web/themes/custom/gravelle1/scss/base/var/\_body-style.scss**

  CKEditor content widths for `.media[class^="display--main_*"]` and `figure[data-view-mode^="main_*"]`.

```scss
.ck.ck-content {
  & > h2:first-child {
    /* when the first child of the ckeditor is an h2, remove top margin*/
    // margin-top: 0 !important;
  }

  /* load all body-styles from include file */
  @include bodyStyle;

  /* these styles are for elements viewed while editing in ckeditor, needed to mimic the final output */
  .drupal-media-style-align-left, /* applies to media */
  .image-style-align-left {
    /* applies to to inline images */
    margin: 0 2em 1em 0;
    max-width: 65%;
  }

  .drupal-media-style-align-right, /* applies to media */
  .image-style-align-right {
    /* applies to to inline images */
    margin: 0 0 1em 2em;
    max-width: 65%;
  }

  .drupal-media-style-align-center, /* applies to media */
  .image-style-align-center {
    /* applies to to inline images */
    text-align: center;
    margin: 0 auto 1em auto;
    max-width: 95%;
  }

  li img,
  p img {
    /* display inline images within text */
    vertical-align: middle;
    border: none;
    margin: 0 0.5em;
  }

  .media[class^='display--main_66'],
  figure[data-view-mode^='main_66'] {
    width: 65%;
    max-width: 65%;
  }

  .media[class^='display--main_50'],
  figure[data-view-mode^='main_50'] {
    width: 49%;
    max-width: 49%;
  }

  .media[class^='display--main_33'],
  figure[data-view-mode^='main_33'] {
    max-width: 32%;
  }

  .media[class^='display--main_25'],
  figure[data-view-mode^='main_25'] {
    max-width: 24%;
  }
  /* /ck_content */
}
```

- **web/themes/custom/gravelle1/scss/base/var/\_body-style.scss**
  `@mixin bodyStyle` includes all display sizing; see `display--main_*` and `data-view-mode="main_*"`. Applies `@include bodyStyle;` to `.body-style` content.

```scss
@mixin bodyStyle {
  /* these styles are imported into _body-style.scss and editor-styles.scss */
  /* include all styles that can apply to the editor body and the editor    */
  /* below, to style only the editor, use _editor-styles.scss after         */
  /* @import "custom-overwrites/globals/_body-style-include"                */

  /* images with captions are wrapped in figure tags */

  figure {
    margin: 0;
    margin-bottom: 1em;
    width: 100%;
    @include desktop-small {
      figure[class^='display--main_66'] {
        max-width: 65%;
      }
      figure[class^='display--main_50'] {
        max-width: 49%;
      }
      figure[class^='display--main_33'] {
        max-width: 32%;
      }
      figure[class^='display--main_25'] {
        max-width: 24%;
      }
    }
    p {
      display: inline; /* remove spacing from paragraph tags in media items */
      margin-bottom: 0;
    }
    div {
      display: inline;
    }
    img {
      padding: 0;
      margin: 0; /* remove margin from img in figure tag */
      display: block;
      @include desktop-small {
        border-bottom: 0;
      }
      &.align-center img {
        margin: 0 auto; /* use margin to center img in figure tag */
      }
    }
  }

  figcaption {
    color: vars.$gray;
    outline-offset: -1px;
    background-color: unset;
    margin: 0 !important;
    padding: 2em 0 0 0 !important;
    font-size: 0.75em;
    font-style: italic;
  }

  /* alignment for figures */

  figure .media {
    /* when media is inside a figure, remove margins, don't align and fill width */
    float: none;
    width: 100%;
    max-width: 100%;
    margin: 0 !important;
    img {
      display: block;
      width: 100%;
    }
  }

  .align-right,
  .align-left,
  .align-center {
    float: none; /* do not float on mobile */
    margin-bottom: 1em;
    p {
      /* render p as inline */
      display: inline;
    }
  }

  .align-right {
    @include desktop-small {
      float: right;
      margin-left: 2em;
      max-width: 65%;
      img.inline-img {
        width: 100% !important; /* make aligned image fit figure */
      }
    }
  }

  .align-left {
    @include desktop-small {
      float: left;
      margin: 0 2em 0 0;
      max-width: 65%;
      img.inline-img {
        width: 100% !important; /* make aligned image fit figure */
      }
    }
  }

  .align-center {
    text-align: center;
    margin: 0 auto 1em auto;
    @include desktop-small {
      margin: 0 auto;
      max-width: 95%;
    }
  }

  @include desktop-small {
    figure:has([class*=' display--main_100']) {
      max-width: 100%;
    }

    figure:has([class*=' display--main_66']) {
      max-width: 65%;
    }

    figure:has([class*=' display--main_50']) {
      max-width: 49%;
    }

    figure:has([class*=' display--main_33']) {
      max-width: 32%;
    }

    figure:has([class*=' display--main_25']) {
      max-width: 24%;
    }
  }

  @include desktop-small {
    [class*=' display--main_66'],
    [data-view-mode*=' main_66'] {
      max-width: 65%;
    }
    [class*=' display--main_50'],
    [data-view-mode*=' main_50'] {
      max-width: 49%;
    }
    [class*=' display--main_33'],
    [data-view-mode*=' main_33'] {
      max-width: 32%;
    }
    [class*=' display--main_25'],
    [data-view-mode*=' main_25'] {
      max-width: 24%;
    }
  }

  /* force svg images and containers to fill display width */
  .media-ext--svg {
    width: 100%; /* on handheld svg media is 100% width */
    img {
      width: 100%;
    }
  }

  @include desktop-small {
    .media-ext--svg {
      &[class*='display--main_100'] {
        width: 100%;
      }
      &[class*='display--main_66'] {
        width: 65%;
      }
      &[class*='display--main_50'] {
        width: 49%;
      }
      &[class*='display--main_33'] {
        width: 32%;
      }
      &[class*='display--main_25'] {
        width: 24%;
      }
      img {
        width: 100%;
      }
    }
  }

  /* /bodyStyle */
}
```

Note: The Twig template expects `file_extension` to output a `.media-ext--<ext>` class. The theme provides this via a preprocess function:

- `web/themes/custom/gravelle1/gravelle1.theme` → `gravelle1_preprocess_media__image()`

## Add Media view modes

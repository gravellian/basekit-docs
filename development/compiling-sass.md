# Compiling Sass (Basekit + Subtheme)

This project has two gulpfiles: one in the base theme (`../basekit/`) and one
in the subtheme (`web/themes/custom/basekit_site/`). They behave the same.

## What the gulpfiles do

Each gulpfile defines these tasks:

- `sassTask`: compiles `scss/**/*.scss` into `css/` (global styles).
- `componentSassTask`: compiles `components/**/*.scss` into each component
  folder (component styles).
- `watch`: watches both folders and re-runs the matching task on change.

Both tasks run PostCSS (`autoprefixer` + `cssnano`). Sourcemaps are enabled only
when `NODE_ENV` is not `production`.

## Dev vs production

- Development build: sourcemaps ON (easier debugging).
- Production build: sourcemaps OFF (no map files or map references).

`NODE_ENV=production` is what controls this.

## Lando tooling (recommended)

From the site repo root:

Subtheme:
```bash
lando gulp        # production build (no sourcemaps)
lando gulp-dev    # development build (sourcemaps)
lando watch       # watch mode (development)
```

Base theme:
```bash
lando basekit-gulp       # production build (no sourcemaps)
lando basekit-gulp-dev   # development build (sourcemaps)
lando basekit-watch      # watch mode (development)
```

## Working directly in ../basekit

If you are inside `../basekit/` (without Lando tooling):

```bash
NODE_ENV=development npm run build
NODE_ENV=production npm run build
npm run watch
```

You can also use npx:

```bash
NODE_ENV=development npx gulp
NODE_ENV=production npx gulp
npx gulp watch
```

## Typical workflow

- Develop: use `watch` or a dev build with sourcemaps.
- Before commit/deploy: run a production build to remove sourcemap references.

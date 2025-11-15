# BaseKit Docs

Documentation for the BaseKit distribution.

- Installation and setup
- Base theme and sub-theme starter
- Recipes (base, features, aggregate) and update flow
- SDCs/custom blocks conventions

Install via Composer (site repo):

```
composer require oomphinc/composer-installers-extender:^2.0 gravellian/basekit-docs:dev-main -W
```

composer.json additions:

```
"extra": {
  "installer-types": ["gravellian-docs"],
  "installer-paths": {
    "docs/": ["type:gravellian-docs"]
  }
}
```


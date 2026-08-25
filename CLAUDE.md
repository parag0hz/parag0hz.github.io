# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Dongwon Lim's personal academic site (https://parag0hz.github.io), built on the
[al-folio](https://github.com/alshedivat/al-folio) Jekyll theme. Almost every file is upstream
theme code; the actual site content lives in a handful of places (see "Where the site content
lives"). `README.md`, `CUSTOMIZE.md`, `INSTALL.md`, and `FAQ.md` are upstream theme docs, not
docs for this site — `CUSTOMIZE.md` is still the best reference for how theme features work.

## Commands

Ruby/Bundler/Jekyll are **not installed on this machine**; Docker and Node are. Local preview:

```bash
docker compose up          # serves on http://localhost:8080 with livereload (:35729)
                           # entry_point.sh restarts Jekyll automatically when _config.yml changes
```

With a Ruby toolchain available instead:

```bash
bundle install
bundle exec jekyll serve                     # dev server
JEKYLL_ENV=production bundle exec jekyll build   # production build into _site/
purgecss -c purgecss.config.js               # strips unused CSS from _site (deploy only)
```

Formatting — this is the only CI gate that can fail on content changes:

```bash
npx prettier . --check     # what .github/workflows/prettier.yml runs
npx prettier . --write     # fix
```

There is no test suite. `.github/workflows/axe.yml` (accessibility) and the broken-link and
lighthouse workflows are `workflow_dispatch`-only.

Building requires `imagemagick` on PATH: `jekyll-imagemagick` regenerates responsive
480/800/1400px `.webp` variants of everything in `assets/img/` on every build.

## Deployment

`.github/workflows/deploy.yml` builds and publishes via `actions/deploy-pages` on push to `main`
(GitHub Pages source is set to "GitHub Actions"). Nothing needs to be committed to a `gh-pages`
branch — **`bin/deploy` is the legacy branch-push route and should not be used**; it force-pushes
`gh-pages` and rewrites the working tree.

## Architecture

Standard Jekyll, with these load-bearing pieces:

- `_config.yml` — single source of truth for site identity, feature flags (`enable_darkmode`,
  `enable_project_categories`, `search_enabled`, …), the emoji favicon (`icon:`), plugin config,
  and pinned CDN URLs + SRI hashes for every third-party JS/CSS library.
- `_plugins/*.rb` — custom Ruby Jekyll plugins run at build time. `download-3rd-party.rb` fetches
  the CDN libraries declared in `_config.yml` into `assets/libs/` (gitignored);
  `cache-bust.rb` adds the `bust_css_cache` filter used in `head.liquid`.
- `_pages/*.md` — every top-level page. Navbar membership and order come from each page's
  `nav:`/`nav_order:` front matter, not from a menu file. Currently only **about, publications,
  projects, cv** are `nav: true`; blog, books, people, repositories, teaching, submenus are all
  `nav: false` but still build and are reachable by URL.
- Collections (`_projects`, `_news`, `_books`, `_posts`) — declared in `_config.yml` under
  `collections:`. Rendered by `_layouts/*.liquid` + `_includes/*.liquid`.
- `_sass/` — compiled through `assets/css/main.scss`. Theme colors and light/dark CSS custom
  properties live in `_sass/_themes.scss` and `_sass/_variables.scss`; site-specific overrides
  were added to `_sass/_base.scss`.

## Where the site content lives

Editing the site almost always means touching one of these, not the theme internals:

- `_pages/about.md` — the homepage. The bio is Markdown, but the **Education / Research
  Experience** two-column block below the `---` is hand-written HTML with inline styles
  (institution logos from `assets/img/`, explicit font sizes). Match that structure when adding
  an entry rather than converting it to Markdown.
- `_projects/*.md` — one file per project card. Front matter drives everything:
  `category:` is a **year** (`2025`), and `_pages/projects.md` renders only the years listed in its
  `display_categories: [2025, 2024, 2023]`. Cards are ordered by `importance:` (ascending) within
  a year, and `img:` is the card thumbnail. A project whose `category` is not in
  `display_categories` silently disappears from the page.
- `_pages/cv.md` — embeds `assets/pdf/dongwon_cv.pdf` in a raw `<iframe>`. Note this page uses
  `layout: page`, so its `cv_pdf:` front matter and `_data/cv.yml` are both inert here; changing
  the CV means replacing the PDF.
- `_data/socials.yml` — social icons on the homepage.

## Site-specific customizations (don't undo these when syncing upstream)

- **Dark mode is removed twice over**: `enable_darkmode: false` in `_config.yml`, _and_ the toggle
  markup was deleted outright from `_includes/header.liquid`. Re-flipping the flag alone will not
  bring the toggle back.
- `_includes/social.liquid` has a non-upstream `{% when 'dacon_url' %}` case that renders an
  `<img>` (`assets/img/dacon.jpeg`) instead of a font icon, paired with `dacon_url:` in
  `_data/socials.yml`. Adding a similar image-based social link means editing both files.
- `_includes/head.liquid` uses `rel="icon" type="image/svg+xml"` for the emoji favicon (upstream
  uses `rel="shortcut icon"`), which is what makes the multi-codepoint 🧑🏻‍💻 emoji render.
- `_sass/_base.scss` forces `.card-img-top` to `aspect-ratio: 16 / 9; object-fit: cover` so
  project thumbnails of mixed sizes line up, and centers `.social .contact-icons`.

## Conventions

- Prettier formats `.md`, `.liquid`, `.yml`, `.js`, and `.scss`; run it before committing.
  `.prettierignore` exempts minified assets, `assets/css/main.scss`, `_scripts/*`, and the
  generated `_data/citations.yml`.
- `assets/libs/`, `_site/`, and `Gemfile.lock` are gitignored — `assets/libs/` is populated at
  build time by `_plugins/download-3rd-party.rb`.
- Upstream theme files carry al-folio's MIT license; when pulling upstream changes, re-check the
  customizations listed above.

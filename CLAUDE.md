# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Shi Hai's personal blog (史海的博客), a Jekyll static site deployed to GitHub Pages at
https://shihai1991.github.io. The theme is a fork of the **Hux Blog** boilerplate (Bootstrap 3 +
custom LESS). Content is primarily in Chinese.

## Commands

- **Local preview:** `npm start` → runs `bundle exec jekyll serve` (site at http://localhost:4000).
- **Live LESS/JS + preview:** `npm run dev` → runs `grunt watch` alongside `jekyll serve`.
- **Deploy:** `npm run push` → `git push origin master --tag`. GitHub Pages auto-builds on push to
  `master`; there is no separate build/release step. **Never commit a locally-built `_site/`.**
- There is no test suite and no linter.

Local preview needs `jekyll-paginate` (the only non-default plugin). There is **no Gemfile** — the
site relies on GitHub Pages' bundled gems, so install the gem manually for local use:
`gem install jekyll-paginate`.

## Architecture

### Jekyll structure
- `_layouts/` — four layouts: `default` (base HTML shell), `post` (articles), `page` (static pages),
  `keynote`. `default.html` assembles `_includes/` partials: `head`, `nav`, `footer`.
- `_includes/` — reusable partials. `intro-header.html` renders the per-page hero header;
  `featured-tags.html`, `friends.html`, `sns-links.html`, `short-about.html` populate the sidebar;
  `mathjax_support.html` is conditionally included per post.
- `_posts/` — ~150 articles, Markdown (GFM via kramdown, `rouge` syntax highlighter).
- Top-level `index.html` (paginated post list), `about.html`, `archive.html`, `404.html`, `feed.xml`.

### Styling build (IMPORTANT)
The live CSS is `css/shihai-blog.css`. Its **LESS source entry is `less/hux-blog.less`** (which
`@import`s `variables`, `mixins`, `sidebar`, `side-catalog`, `snackbar`, `highlight`). However,
`Gruntfile.js` is **stale**: its `uglify`/`less` tasks target `js/shihai-blog.js` and
`less/shihai-blog.less` (derived from `pkg.name`) — neither file exists. Consequences:
- `grunt` (default task) **will fail** as configured.
- `js/hux-blog.min.js` is committed pre-minified; there is no unminified `shihai-blog.js` source.
- To genuinely recompile LESS by hand: `lessc less/hux-blog.less css/shihai-blog.css` (then commit
  the CSS, since GitHub Pages does not run Grunt).
- `_config.yml` excludes `less/`, `Gruntfile.js`, `package.json`, READMEs from the Pages build, so
  LESS sources are never deployed — only the compiled `css/` is.

### PWA
- `sw.js` — service worker with a namespaced precache-then-runtime caching strategy
  (`CACHE_NAMESPACE = 'main-'`). The `DEPRECATED_CACHES` list is cleaned up on activate; bumping the
  precache version triggers cache invalidation.
- `pwa/manifest.json` + `pwa/icons/` — web app manifest; registered via `js/sw-registration.js`.

## Writing & editing posts

Posts live in `_posts/` as `YYYY-MM-DD-title.md`. Content is GFM Markdown, mostly Chinese.

Front matter conventions observed across the corpus:
- `layout: post` — always.
- `title`, `tags:` (list), `category:` (grouping key, e.g. `books`, `software engineering`, `python`,
  `AI`).
- `published: true|false` — **drafts use `false`** (39 posts are drafts).
- `catalog: true|false` — toggles the floating side table-of-contents (built from `<h>` headings by
  `side-catalog`). Default to `true` for long technical posts.
- `time:` — a **custom display string** (e.g. `'2026-08-05 12:20'`), NOT Jekyll's `page.date`. Formats
  are inconsistent across posts. Jekyll's canonical publish date comes from the filename prefix.
- `header-style: text` — switches the hero header to a text-only variant (no background image).
- `mathjax: true` — opt-in per-post MathJax (renders `mathjax_support.html`). Site default is off.
- `excerpt:` — feeds OG description / preview; use `<!--more-->` as the excerpt separator when needed.

### Dates and `future: true`
`_config.yml` sets `future: true`, so posts dated in the future **are published**. This is
intentional and in active use (e.g. dated 2027). Keep this in mind — a far-future filename date is
not a way to hide a draft; set `published: false` instead.

## Site configuration
All site-wide settings — title, SEO, SNS handles, sidebar, featured-tags thresholds, Disqus,
analytics IDs, PWA theme color — live in `_config.yml`. Edit there rather than hunting through
layouts/includes.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`awmanoj.github.io` — a personal blog built with Jekyll on the [Hyde](https://github.com/poole/hyde)/Poole theme, hosted by GitHub Pages. `README.md` is the *upstream theme's* readme (author: Mark Otto), not documentation of this site — don't treat it as a description of this repo.

## Build & deploy

There is no Gemfile, no CI config, and no build script. GitHub Pages builds the site itself from the `master` branch on push — that push *is* the deploy. Nothing else triggers a rebuild, which is why trivial "trigger" commits appear in the history.

Local preview (requires a system Jekyll with `jekyll-paginate`, since `_config.yml` sets `paginate: 15`):

```bash
jekyll serve            # http://localhost:4000, live rebuild
JEKYLL_ENV=production jekyll serve   # also injects Google Analytics
```

Analytics (`_includes/analytics.html`) is included only when `jekyll.environment == 'production'`, so local builds are analytics-free by default.

## Content

Posts live in `_posts/YYYY-MM-DD-slug.md`. `permalink: pretty` means the URL is `/YYYY/MM/DD/slug/`. Front matter convention used throughout:

```yaml
---
layout: post
title: Programmability in nginx config using map
author: Manoj Awasthi
categories: tech
---
```

`categories` is a free-form comma list (`tech`, `life`, `books`, `tech, algorithms`, …); nothing renders category pages today, so it's metadata only.

Standalone pages at the repo root (`about.md`) use `layout: page`. **That layout choice is load-bearing:** `_includes/sidebar.html` builds the nav by iterating `site.pages` and emitting a link for every page whose layout is `page`. Anything else is invisible in the nav; conversely, giving a new page `layout: page` silently adds it to the sidebar. Extra sidebar links (Books, GitHub projects, feed) are hardcoded in that include.

## Layout chain

`post.html` / `page.html` → `default.html` → `_includes/head.html` + `_includes/sidebar.html`. `post.html` also renders a "Related Posts" block from `site.related_posts` (with `lsi` off, Jekyll fills this with the most recent posts, not semantically related ones).

`atom.xml` (`layout: null`) emits the full feed; the public feed URL advertised in the sidebar is a FeedBurner proxy.

## Asset paths — use root-absolute, not `site.baseurl`

`baseurl` is `/`, so `{{ site.baseurl }}public/...` renders as `//public/...` — a protocol-relative URL that breaks on post pages. `head.html` has the broken version commented out with a link to the explanation; CSS is linked as plain `/public/css/...`. Follow that: reference assets as `/public/assets/img/foo.png`, never through `site.baseurl`.

## `public/`

Mixed bag, and the front matter decides the behavior:

- `css/` (`poole.css`, `syntax.css`, `hyde.css`) and `assets/img/` — theme CSS and all post images.
- `books.html`, `cmir.html` — have front matter, so they render through the Jekyll layouts.
- `tictactoe.html`, `sierpinski.html`, `amp.html` — no front matter, so Jekyll copies them verbatim. They are self-contained HTML/JS apps served at `/public/<name>.html` and are not part of the templating system.

## Leftovers

`_index.html` is an unused alternate homepage (full post bodies instead of a title list). The leading underscore means Jekyll ignores it; the live homepage is `index.html`.

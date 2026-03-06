# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal professional site for Eric Talevich, Ph.D. (etal.github.io) — currently a single landing page, a Jekyll static site using the Cayman theme, hosted on GitHub Pages.

Eric is VP Bioinformatics at DataXight and owner of Et al Bio, LLC (independent consulting). Background spans clinical genomics leadership (Caris, Karius, UCSF), cloud data platforms (DNAnexus), and biotech solution architecture (Form Bio). Creator of CNVkit; major contributor to Biopython. Ph.D. Bioinformatics, University of Georgia.

## Build & Development

```bash
bundle install          # Install dependencies
bundle exec jekyll serve  # Local dev server (http://localhost:4000)
bundle exec jekyll build  # Build static site to _site/
```

Requires Ruby and Bundler. Uses `github-pages` gem for GitHub Pages compatibility.

## Architecture

- **`_config.yml`** — Site metadata, theme selection (`jekyll-theme-cayman`), and social link URLs (GitHub, LinkedIn, BlueSky, Google Scholar)
- **`index.md`** — Main page content (Markdown with YAML front matter)
- **`_layouts/default.html`** — Custom layout overriding the Cayman theme; adds a header button strip for social/external links driven by `site.*_url` config values
- **`Gemfile`** — Pins `github-pages` ~> 232 and `jekyll-theme-cayman` ~> 0.2.0

The site is a single-page layout. To add header buttons, add a `*_url` key in `_config.yml` and a corresponding `{% if site.*_url %}` block in `_layouts/default.html`.

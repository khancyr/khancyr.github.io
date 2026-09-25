# khancyr.github.io

This repository hosts the source for Pierre Kancir’s personal blog site, built with **MkDocs** + **Material for MkDocs**.

## What’s included

- A **MkDocs** site configuration (`mkdocs.yml`)
- Content written in **Markdown** under `docs/`
- A blog section in `docs/blog`
- Static assets such as images, CSS and JS under `docs/images`, `docs/stylesheets`, and `docs/js`

## Requirements

- Python 3.10+ (or compatible)
- A virtual environment is recommended

## Setup (recommended)

```bash
python -m venv .venv
source .venv/bin/activate
pip install mkdocs-material "mkdocs-material[imaging]" mkdocs-rss-plugin
```

The social cards (`[imaging]`) also need the Cairo system library, e.g. `sudo apt install libcairo2`.

## Running locally

```bash
mkdocs serve
```

Then open: http://127.0.0.1:8000

## Building the site

```bash
mkdocs build
```

The generated site will be in `site/`.

## Writing a new blog post

1. Add a new Markdown file under `docs/blog/posts/`, e.g. `docs/blog/posts/my-new-post.md`
2. Start the file with frontmatter metadata:

```yaml
---
title: "My post title"
description: "One sentence shown in link previews and search results."
date:
  created: 2026-03-15
  updated: 2026-03-15
draft: false
categories:
  - Various
tags:
  - example
authors:
  - khancyr
---
```

3. Optional: `image: https://khancyr.github.io/blog/images/<file>` (absolute URL, file in `docs/blog/posts/images/`) replaces the generated social card in link previews and the RSS feed.
4. Write your content below the frontmatter.

## Deploying

This site is typically hosted via GitHub Pages (using the `gh-pages` branch or GitHub Actions). Update your deployment workflow if needed.

## License

See the `LICENSE` file for licensing details.

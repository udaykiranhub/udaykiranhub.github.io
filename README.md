# udaykiranhub.github.io

Source for <https://udaykiranhub.github.io>, a Web3 security research blog built with Jekyll and published by GitHub Pages.

## Add a post

Create one file, `_posts/YYYY-MM-DD-short-title.md`:

```markdown
---
title: "Title of the post"
category: writeup   # writeup -> Write-ups, log -> Research Log
tags: [defi, oracle]
---

Post content in Markdown.
```

Commit and push to `main`. GitHub rebuilds the site in a minute or two, and the post appears on the home page, on its section page, and at `/YYYY/MM/short-title/`.

- Revised a published post? Add `last_modified_at: YYYY-MM-DD` to its front matter.
- Hide a post without deleting it: `published: false`.
- Posts go through Liquid, so wrap code that contains `{{` or `{%` in `{% raw %}` … `{% endraw %}`.
- `_posts/2026-10-20-post-template.md` demonstrates tables, highlighted code, diffs and footnotes. Copy it as a starting point.

## Where things live

| Path | What it is |
|---|---|
| `_posts/` | Posts (content) |
| `index.md`, `research-log.md`, `writeups.md`, `about.md` | Pages (content) |
| `_config.yml` | Title, profile links, navigation, permalink format |
| `_layouts/`, `_includes/`, `assets/css/main.css` | Templates and styling |

## Preview locally (optional)

Needs Ruby and Bundler. Then:

```sh
bundle install
bundle exec jekyll serve
```

and open <http://localhost:4000>.

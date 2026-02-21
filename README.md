# Netanel Eliav - Blog Content Repo ✍️

This repository contains the public articles and assets that power the **Articles / Blog** section on my website.

It’s intentionally lightweight: a clean folder of Markdown posts + images, fetched directly by my site backend in production.

If you’re here to read, browse around.  
If you’re here to reuse something, please read the license section at the bottom.

---

## What this repo is

- My **public writing archive**
- Source of truth for **blog posts shown on my website**
- A simple structured store of:
  - Markdown articles
  - Series folders
  - Images / assets referenced by posts

---

## Folder structure

```
blog/
  articles/
    my-article.md
    another-article.md
    series-name/
      part-1.md
      part-2.md

  external/
    some-interview.md      # curated external/guest articles (is_external: true)

  assets/
    images/
      some-image.png
      another-image.jpg
    diagrams/
    ...
```

### `articles/`
- Every `.md` file here is a published article owned by Netanel.
- Subfolders inside `articles/` represent **series**.
- Files in a series are ordered by the `order` frontmatter field; filesystem order is used as a fallback.

### `external/`
- Markdown stubs for curated external or guest articles (interviews, guest posts, etc.).
- Must include `is_external: true` in frontmatter, plus `outlet` and `url` fields.
- These are fetched and displayed alongside owned articles but flagged differently in the UI.

### `assets/`
- Any images/diagrams referenced by articles live here.
- Articles should reference assets using raw GitHub URLs.

Example:

```md
![diagram](https://raw.githubusercontent.com/iNetanel/blog/main/assets/images/diagram.png)
```

---

## Article format (frontmatter)

Each article is Markdown with YAML frontmatter on top.

Example:

```md
---
title: "Why I Build Agentic Systems"
description: "A practical look at how I design production-grade AI agents."
date: "2025-01-12"
author: "Netanel Eliav"
tags: ["ai", "agents", "rag"]
featured: true
featured_image: "https://raw.githubusercontent.com/iNetanel/blog/main/assets/images/cover.png"
---

Your markdown content starts here...
```

### Supported fields

| Field            | Type      | Notes |
|-----------------|-----------|------|
| `title`         | string    | Required |
| `description`   | string    | Short SEO summary |
| `date`          | string    | Use ISO format `YYYY-MM-DD` |
| `author`        | string    | Defaults to "Netanel Eliav" if missing |
| `tags`          | string[]  | Optional |
| `featured`      | boolean   | Optional — surfaces article prominently |
| `featured_image`| string    | Full URL to image (recommended) |
| `series`        | string    | Auto-detected from subfolder name; can be overridden |
| `order`         | number    | Controls ordering within a series |
| `is_external`   | boolean   | Set `true` for external/guest articles (in `external/`) |
| `outlet`        | string    | Publication name — required when `is_external: true` |
| `url`           | string    | Original article URL — required when `is_external: true` |

---

## How production works

My website backend:
1. Reads this repo via GitHub raw content API
2. Scans both `/articles` (owned posts) and `/external` (curated external articles)
3. Builds an in-memory metadata cache (refreshed every 60 seconds)
4. Serves:
   - `/api/articles/list` — full metadata list
   - `/api/articles/{slug}` — rendered article detail (Markdown → sanitized HTML)
   - `/api/articles/series/{name}` — articles belonging to a series

In development, articles are read from local `./articles/` and `./external/` directories instead (no GitHub API calls, always-fresh cache).

No DB needed, no CMS needed.  
This repo *is* the CMS.

---

## Writing guidelines (for me)

- Keep filenames short and slug-friendly:
  - ✅ `why-rag-matters.md`
  - ✅ `agents-in-production.md`
  - ❌ `My Cool Article Final v2.md`

- Use **relative raw GitHub links** for images in `assets/`.
- Prefer ISO dates for correct sorting.

---

## Contributions / PRs

This is a personal writing repo, so I’m not actively accepting PRs for content.

If you spot a typo or error, feel free to open an issue - I’ll gladly fix it.

---

## License

Unless explicitly stated otherwise inside a post:

**All articles and content are © Netanel Eliav.  
No re-publication without permission.**

You may:
- Link to articles
- Quote short excerpts with attribution

You may not:
- Copy full posts
- Re-host content elsewhere
- Use this repo as your own blog source

If you want to license or reuse something commercially - contact me.

---

## Contact

Website: https://inetanel.com  
GitHub: https://github.com/iNetanel  
Twitter / X / LinkedIn: (you know where to find me 😄)

---

# Netanel Eliav — Blog Content Repo ✍️

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


### `articles/`
- Every `.md` file here is a published article.
- Subfolders inside `articles/` represent **series**.
- Files in a series are ordered alphabetically by filename unless explicitly set otherwise.

### `assets/`
- Any images/diagrams referenced by articles live here.
- Articles should reference assets using raw GitHub URLs.

Example:

```md
![diagram](https://raw.githubusercontent.com/iNetanel/blog/main/assets/images/diagram.png)


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

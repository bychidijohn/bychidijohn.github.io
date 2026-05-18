# Personal site

A minimal Jekyll site for hosting writeups and project pages, designed for senior ML engineering / applied science portfolios. Refined-minimal aesthetic, serif typography, dark-mode-aware, mobile-responsive.

## Quick start

### 1. Create the repo

For a user site at `https://YOUR-USERNAME.github.io`:
- Name the repo exactly `YOUR-USERNAME.github.io`
- Put all these files at the root

For a project site at `https://YOUR-USERNAME.github.io/sitename`:
- Name the repo whatever you want
- Set `baseurl: "/sitename"` in `_config.yml`

### 2. Customize `_config.yml`

Replace every `YOUR-...` placeholder. At minimum:
- `title` — your name
- `email` — your email
- `url` — your full GitHub Pages URL
- `github_username`, `linkedin_username`

### 3. Push to GitHub

```bash
git init
git add .
git commit -m "Initial site"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-USERNAME.github.io.git
git push -u origin main
```

In repo Settings → Pages, set the source to `main` branch, `/ (root)`. Build takes ~1 minute.

### 4. Preview locally (optional)

Requires Ruby 3.x:

```bash
bundle install
bundle exec jekyll serve
```

Open `http://localhost:4000`.

## Writing a new post

Create a file in `_posts/` named `YYYY-MM-DD-slug.md`:

```markdown
---
layout: post
title: "Your post title"
subtitle: "Optional subtitle for the post header"
date: 2026-02-01
reading_time: 8
tags: [causal-ml, bandits]
excerpt_short: "One-sentence summary shown on the home page."
---

Your content in Markdown. Code blocks, tables, footnotes, images all styled.
```

Posts auto-appear on the home page sorted newest-first.

## Adding a project

Edit `projects.html` — duplicate one of the `<li>` blocks and update the title, meta, and description.

## Customizing the look

Everything lives in `assets/css/style.css`. Key tokens at the top of the file:

- `--paper` / `--ink` — background and text colors
- `--accent` — the one accent color (currently terracotta). Change this one variable to retheme the site.
- `--display` / `--body` / `--mono` — font families
- `--measure` — content max-width (default 38rem, ~65ch — optimal for reading)

Dark mode is automatic via `prefers-color-scheme`.

## Structure

```
.
├── _config.yml          # site settings
├── _layouts/            # page templates
│   ├── default.html
│   └── post.html
├── _includes/           # header, footer
├── _posts/              # blog posts (Markdown)
├── assets/css/style.css # all styles
├── index.html           # home page
├── projects.html
└── about.html
```

## Things to do before going public

- [ ] Replace `YOUR-NAME`, `YOUR-GITHUB-USERNAME`, `YOUR-LINKEDIN-HANDLE` in `_config.yml`
- [ ] Update the intro copy in `index.html` to match your actual focus
- [ ] Update `about.html` with your real bio
- [ ] Update the four placeholder projects in `projects.html` with real links once they exist
- [ ] Replace the sample post with your first real writeup, or delete it
- [ ] Optional: add a favicon (`favicon.ico` at repo root)
- [ ] Optional: change `--accent` if terracotta isn't your vibe (try `#2c5f5d` teal, `#3d4a8a` indigo, or `#7a5c3f` walnut)

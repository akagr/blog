# akashagrawal.me

Personal site and blog of Akash Agrawal — [akashagrawal.me](https://akashagrawal.me).

Built with [Astro](https://astro.build) and deployed to GitHub Pages.

## Develop

```bash
npm install
npm run dev      # start local dev server at http://localhost:4321
npm run build    # build the production site to ./dist
npm run preview  # preview the production build locally
```

## Structure

- `src/pages/index.astro` — homepage (profile + featured projects + recent posts)
- `src/pages/blog/` — blog index and post pages
- `src/pages/about.astro` — about page
- `src/content/blog/` — blog posts (Markdown)
- `src/components/`, `src/layouts/` — shared UI
- `public/CNAME` — custom domain

## Writing a post

Add a Markdown file to `src/content/blog/`:

```markdown
---
title: My new post
description: A short summary shown in listings and meta tags.
pubDate: 2026-01-01
tags: ["example"]
---

Post body in Markdown.
```

## Deploy

Pushing to `master` triggers `.github/workflows/deploy.yml`, which builds with
`withastro/action` and publishes to GitHub Pages. Ensure the repository's
**Settings → Pages → Build and deployment → Source** is set to **GitHub Actions**.

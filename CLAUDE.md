# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development

Always start the dev server in background mode:

```
npx astro dev --background
```

Manage with `astro dev stop`, `astro dev status`, `astro dev logs`.

Build (includes Pagefind index generation):

```
npm run build
```

After every completed task, commit and push to `origin master` without waiting for a separate instruction.

## Architecture

**Layout chain**: `Layout.astro` (shell + canvas) → `BlogPost.astro` (article wrapper) → `[slug].astro` (static path generation).

- `Layout.astro` renders the full HTML shell. It contains the entire `<canvas id="bg-canvas">` animation in a single `<script is:inline>` block (sky, stars, nebulae, moon, clouds, fog, sea/waves). All canvas draw functions share module-level variables `skyR/G/B` which `drawSky()` updates every frame so `drawFog()` and `drawSea()` can tint themselves to match the live sky color.
- `Sidebar.astro` is transparent on desktop — the bg-canvas behind it shows through via a `::before` overlay (`rgba(2,3,18,0.52)`). On mobile (`max-width: 1024px`) the sidebar uses a solid `#04041c` background because the canvas is hidden.
- `global.css` carries all visual styles. Tailwind CSS v4 is configured as a Vite plugin (no `tailwind.config.js`). The `@plugin "@tailwindcss/typography"` line enables `.prose`; note that `.prose` sets `font-size` in `rem` (root-relative, not inherited), so overrides must target `.prose` directly.

**Content**: Posts live in `src/content/blog/*.md`. The collection schema (`src/content.config.ts`) requires `title`, `description`, `pubDate` (ISO 8601 with time and timezone for same-day sort determinism), and optional `tags`, `updatedDate`, `draft`, `image`. Drafts are excluded via `!data.draft` filter. Posts are sorted by `pubDate.valueOf()` (full timestamp). The slug is the filename minus extension.

**Search**: Pagefind runs after `astro build` (`npm run build`). The search UI at `/search` loads `pagefind-ui.js` as an IIFE via a dynamic `<script>` tag (not `import()`), then accesses `window.PagefindUI`.

**Deployment**: GitHub Actions workflow at `.github/workflows/deploy.yml` triggers on push to `master` and deploys to GitHub Pages (`runawayfox.github.io`). Pages source must be set to **GitHub Actions** in repository settings.

## Content authoring

Frontmatter template:

```yaml
---
title: "제목"
description: "설명"
pubDate: 2026-10-09T10:00:00+09:00
tags: ["태그1", "태그2"]
---
```

**CommonMark bold pitfall**: `**텍스트(English)**한국어` breaks bold because `)` followed by a Korean character cannot close `**` (CommonMark closing delimiter rule). Always place English parentheticals outside the bold span: `**텍스트**(English)`.

## Documentation

Full documentation: https://docs.astro.build

- [Routing](https://docs.astro.build/en/guides/routing/)
- [Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Content collections](https://docs.astro.build/en/guides/content-collections/)
- [Styling / Tailwind](https://docs.astro.build/en/guides/styling/)

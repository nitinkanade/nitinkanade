# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Nitin Kanade's personal portfolio + tech blog. Pure static HTML/CSS — no build step, no package manager, no JS framework, no tests. Deployed to GitHub Pages.

Live at `https://nitinkanade.github.io/nitinkanade/`. There is no custom domain configured (no `CNAME` file in the repo).

## Local preview

Open `index.html` directly in a browser, or serve the repo root with any static server (e.g. `python -m http.server`). There is no build, lint, or test command — there is nothing to compile.

## Deployment

`.github/workflows/static.yml` deploys to GitHub Pages on every push to `main`. It uploads the **entire repository** as the Pages artifact (`path: '.'`), so anything committed to root is published. There is no staging environment — `main` is production.

## Architecture notes that span multiple files

### Inlined, duplicated CSS

Each HTML page (`index.html`, `blog/index.html`, every `blog/posts/*.html`) carries its own copy of the CSS inside a `<style>` block. The CSS variables at the top (`--primary-color`, `--secondary-color`, etc.) are intentionally repeated. **When changing the visual theme, update every page** — there is no shared stylesheet. Blog post pages use a narrower `--container-width: 800px` for readability; the portfolio and blog index use `1100px`.

### Adding a blog post requires three coordinated edits

A new post is not just a new HTML file. To ship one:

1. Create `blog/posts/<slug>.html` (copy an existing post for the head/meta/style boilerplate).
2. Add a card to `blog/index.html` **and** append an entry to its `Blog` JSON-LD `blogPost` array (around line 35–55).
3. Add the post URL to `sitemap.xml` with `<lastmod>` and a `<priority>` around 0.7.

Missing any of these breaks SEO/indexing even though the page itself works.

### Relative paths shift by directory depth

Favicon and asset references depend on where the file lives:
- `index.html` (root) → `./images/favicon/...`
- `blog/index.html` → `../images/favicon/...`
- `blog/posts/*.html` → `../../images/favicon/...`

When moving or copying pages between directories, fix these paths.

### SEO is load-bearing

Every page has a hand-maintained set of: `<title>`, `description`, `keywords`, `author`, `robots`, `<link rel="canonical">`, full Open Graph + Twitter Card tags, and JSON-LD structured data (`Person`/`WebSite`/`WebPage` graph on the home page, `Blog`+`BreadcrumbList` for the blog index, `BlogPosting`+`BreadcrumbList` graph on each post). The site relies on these for Google rankings — preserve them when editing.

### Canonical URL convention

**All canonicals, OG URLs, JSON-LD `url`/`@id`, sitemap entries, and absolute internal references must use `https://nitinkanade.github.io/nitinkanade/` as the base.** This is the only domain currently serving the site (no `CNAME` file in the repo).

A previous version of `index.html` declared a canonical of `https://nitinkanade.com` while no custom domain was configured — this is what de-indexed the site (Google followed the canonical, found nothing, and dropped the page). If a custom domain is ever re-introduced, do all of these in one change:

1. Add a `CNAME` file at the repo root containing the bare domain (e.g. `nitinkanade.com`).
2. Update every `<link rel="canonical">`, `og:url`, `twitter:*` URL, and JSON-LD `url`/`@id` across all HTML files.
3. Update every `<loc>` in `sitemap.xml` and the `Sitemap:` line in `robots.txt`.
4. Verify the new property in Google Search Console and submit the sitemap.

Never let canonicals diverge between pages or point to a domain that doesn't resolve to this site.

### Image asset for OG/Twitter previews

Social previews use `images/favicon/web-app-manifest-512x512.png` (512×512). Don't reference `favicon.svg` for OG — it's 4.6 MB and many crawlers won't render SVG previews.

### Site verification files

`BingSiteAuth.xml`, `google9986fcfc5322e150.html`, and `robots.txt` at the repo root are search-engine verification/config files. Don't move or rename them — search consoles look for them at fixed paths.

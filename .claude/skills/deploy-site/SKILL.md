---
name: deploy-site
description: Generate and deploy static websites to go.merling.co.il. Use when the user wants to create, publish, or share a webpage, landing page, portfolio, blog post, or any static site. Generates HTML/CSS/JS and deploys via GitHub Pages.
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash]
argument-hint: "[description of the site to create]"
---

# Deploy Site Skill

## Overview

This skill generates static HTML/CSS/JS websites and deploys them to **go.merling.co.il** via GitHub Pages. Each site lives at a subpath (e.g., `go.merling.co.il/my-project`).

## Configuration

- **Domain**: go.merling.co.il
- **Hosting**: GitHub Pages
- **Repo**: `tzvimg/merling-site` on GitHub
- **Local path**: `C:/dev/merling-site`
- **URL pattern**: `https://go.merling.co.il/{subpath}`

## Workflow

### Step 1: Understand the Request

Parse the user's request to determine:
- **Site purpose**: landing page, blog post, portfolio, tool, demo, etc.
- **Subpath**: derive from the topic or ask the user (e.g., `ai-course`, `resume`, `demo-app`)
- **Language direction**: RTL for Hebrew, LTR for English
- **Content**: text, images, interactive elements needed

### Step 2: Generate the Site

Create files in `C:/dev/merling-site/{subpath}/`:

```
C:/dev/merling-site/{subpath}/
├── index.html      # Main page (always required)
├── style.css       # Styles (inline in HTML for simple sites)
├── script.js       # JavaScript (if interactive features needed)
└── assets/         # Images, fonts, etc. (if needed)
```

#### HTML Generation Guidelines

1. **Always include** proper `<!DOCTYPE html>`, charset, viewport meta
2. **Self-contained**: inline CSS/JS for single-page sites, separate files for complex ones
3. **Responsive**: mobile-first design, works on all screen sizes
4. **Modern design**: clean typography, good spacing, subtle animations
5. **RTL support**: add `dir="rtl"` and `lang="he"` for Hebrew content
6. **No external dependencies** unless explicitly requested - use vanilla HTML/CSS/JS
7. **Performance**: minimize file sizes, optimize images
8. **Accessible**: semantic HTML, proper headings, alt text, contrast

#### Design Defaults

- **Font**: system font stack (`-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`)
- **Colors**: dark theme by default (bg: #0a0a0a, text: #e0e0e0)
- **Accent**: gradient or solid color matching the content mood
- **Layout**: centered content, max-width for readability
- For Hebrew content, consider using `'Heebo'` or `'Assistant'` Google Fonts

### Step 3: Preview Locally (Optional)

If the user wants to preview before deploying:
```bash
# Open in default browser
start C:/dev/merling-site/{subpath}/index.html
```

### Step 4: Deploy

```bash
cd C:/dev/merling-site

# Stage the new/modified site
git add {subpath}/

# Commit with descriptive message
git commit -m "Deploy {subpath}: {brief description}"

# Push to GitHub (triggers GitHub Pages deployment)
git push origin master
```

After pushing, the site will be live at `https://go.merling.co.il/{subpath}` within 1-2 minutes.

### Step 5: Report

Tell the user:
- The URL where the site is live: `https://go.merling.co.il/{subpath}`
- Note that GitHub Pages deployment takes 1-2 minutes
- Offer to make changes or adjustments

## Deploying a Vite/React SPA to a Subpath

When the user asks to deploy an existing Vite app (e.g., `./viewer`) rather than generate a new static site:

### Key Gotcha: Absolute Paths Break Under Subpaths

Vite's `--base` flag only fixes **bundled asset** paths (JS/CSS imports in index.html). It does NOT fix:
- Runtime `fetch()` calls (e.g., `fetch('/api/data.json')`)
- Dynamic `<img src>` attributes from data (e.g., YAML/JSON content)
- `backgroundImage: url(...)` set in JS
- Any other absolute path constructed at runtime

### Solution: `assetUrl()` Utility

Create a utility that prepends `import.meta.env.BASE_URL`:

```ts
// src/utils/assetUrl.ts
const base = import.meta.env.BASE_URL || '/';
export function assetUrl(path: string): string {
  if (!path || !path.startsWith('/')) return path;
  if (base === '/') return path;
  return base.replace(/\/$/, '') + path;
}
```

Apply it to:
1. **Data fetching** - any `fetch()` with absolute paths
2. **Data normalization** - prefix `src`, `image`, `background` fields after loading data (YAML/JSON)
3. **Static references** - hardcoded paths in JSX (e.g., `<img src="/hero.png">`)

### Build & Deploy Steps

```bash
# 1. Build with subpath base
cd viewer
npx vite build --base /{subpath}/

# 2. Copy build output
cp -r dist/* C:/dev/merling-site/{subpath}/

# 3. Copy public assets (follow symlinks with -L)
cp -rL public/presentations C:/dev/merling-site/{subpath}/
cp -rL public/assets C:/dev/merling-site/{subpath}/

# 4. Deploy
cd C:/dev/merling-site
git add {subpath}/
git commit -m "Deploy {subpath}: {description}"
git push origin master
```

### No 404.html Needed

If the app uses query params (`?lesson=1`) instead of path-based routing, no SPA fallback is required. If it uses client-side routing (React Router with paths), add a `404.html` that redirects to `index.html`.

## Site Templates

### Landing Page
Best for: product launches, event pages, announcements
- Hero section with headline and CTA
- Feature highlights
- Footer with contact info

### Blog Post / Article
Best for: sharing written content, tutorials, essays
- Article layout with proper typography
- Reading-friendly width (max 700px)
- Optional table of contents

### Portfolio / Showcase
Best for: displaying work, projects, gallery
- Grid or masonry layout
- Image-heavy with descriptions
- Filter/category support

### Interactive Tool
Best for: calculators, converters, small web apps
- Functional UI with JavaScript
- Input/output sections
- Clear instructions

### Simple Share Page
Best for: quickly sharing a link, file, or info
- Minimal design
- Just the essential content
- Card-style layout

## Managing Existing Sites

### List deployed sites
```bash
ls -d C:/dev/merling-site/*/
```

### Update an existing site
Edit files in `C:/dev/merling-site/{subpath}/` and re-deploy.

### Remove a site
```bash
cd C:/dev/merling-site
git rm -r {subpath}/
git commit -m "Remove {subpath}"
git push origin master
```

## Important Notes

- Always confirm the subpath **spelling** with the user before deploying (renaming after deployment means rebuilding with a new base path)
- Check if a subpath already exists before creating (avoid overwriting)
- The root `index.html` and `CNAME` file should not be modified
- Keep sites self-contained - don't share assets between subpaths
- For sites with images, use the nano-banana-image skill to generate them if needed
- **HTTPS**: After first deploy to a new subpath, GitHub Pages may show "DNS Check in Progress" and HTTPS unavailable. This is normal - the SSL certificate is issued automatically once the DNS check passes (usually within 15 minutes). If stuck, remove and re-add the custom domain in Settings > Pages.

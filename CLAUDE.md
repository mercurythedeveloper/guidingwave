# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Single-page marketing site for **Guiding Wave**, deployed to `guidingwave.com` via GitHub Pages (see `CNAME`). Guiding Wave does product development and coaching ("Less process. More users."). No framework, no build tooling, no `package.json`.

## Architecture (read before editing)

`index.html` is now **flat, pre-rendered static HTML** — real DOM content, editable by hand. All page copy lives directly in the markup; there is no client-side unpacking step. This is the source of truth.

History: the file was originally a **Claude Design artifact bundle** — a loader shell plus the real page stored as a JSON-encoded string inside `<script type="__bundler/*">` tags, unpacked at runtime by a React `dc-runtime`. That rendered 100% client-side, so crawlers saw only a "Bundled Page" loader. On 2026-09-22 it was **pre-rendered into static HTML** at build time (directives expanded, fonts externalised, interactivity ported to vanilla JS) to fix indexing. The original bundle is recoverable from git history (commit `55b1f2f` and earlier) and was backed up during migration. The pre-render was produced by a one-off script (`build.py`, kept in the migration job's tmp dir, not committed) that decoded the bundle's `__bundler/template` + `manifest`.

Files:
- `index.html` — the whole page (~68 KB): `<head>` (SEO + JSON-LD + Nocturne `<style>`), static body content, one inline vanilla `<script>` at the end.
- `fonts/inter-*.woff2` — self-hosted Inter subsets, referenced by `@font-face` with `unicode-range` + `font-display:swap` (browser fetches only the subsets a page needs).
- `robots.txt`, `sitemap.xml`, `og-image.png` — SEO/static assets served as-is by GitHub Pages.

## Design system — "Nocturne"

The `<head>` `<style>` blocks embed a design system called **Nocturne**: CSS custom-property tokens (`--color-*`, `--accent-rgb`, `--header-bg`, `--muted`, `--color-divider`, etc.) plus component classes and `@keyframes` (`gw-rise`, `gw-pulse`). Retune the look by editing tokens, not hard-coded values. Theme is switchable: tokens are defined under `html[data-theme="dark"]` (default, base `#06080B`, accent `#4FC3FF`) and `html[data-theme="light"]` (cream `#E9E4DB`, accent `#A8400C`). Canvas animations read `--accent-rgb` live, so they recolor on theme switch.

## Interactivity (vanilla JS, no dependencies)

The single inline `<script>` at the end of `<body>` (ported from the original React component) drives:
- **Theme toggle** — `#gw-theme-toggle` button flips `html[data-theme]`, persists to `localStorage["gw-theme"]`. A tiny boot script in `<head>` applies the saved theme before paint (avoids FOUC).
- **FAQ accordion** — single-open; markup carries `data-faq` / `data-faq-q` / `data-faq-a` / `data-faq-sign`. All answers are in the DOM for SEO (closed ones use the `hidden` attribute); first item open by default.
- **Scroll reveal** — `[data-reveal]` sections fade/rise in via the `gw-rise` keyframe.
- **Canvas/SVG animations** — hero liquid-bounce (`#gw-hero-bounce`), approach ripple (`#gw-approach-canvas`), and the "journey" wave connector (`[data-journey*]`, `[data-step-dot]`). All respect `prefers-reduced-motion`.

## Page structure

Sticky `<header>`/`<nav>`, hero (`<h1>`), then `#services`, `#coaching`, `#approach`, plus "Three in one" (Venn SVG), "How we work", FAQ, testimonial/CTA, and `<footer>`. Sections carry `data-reveal`. **"Book a call"** CTAs (6) link to a Google Calendar appointment-scheduling URL; contact is `mailto:info@guidingwave.com`.

## Editing

Edit `index.html` directly — it is plain HTML/CSS/JS now. Note the site is styled with **inline `style=` attributes** on elements (an artifact of the original tooling), so most visual tweaks happen either on the element's inline style or via the Nocturne tokens in the `<head>` `<style>`. When you change page copy, also update the matching **JSON-LD** in `<head>` (esp. the `FAQPage` block, which must mirror the FAQ Q&A) and, if URLs/sections change, `sitemap.xml`.

Regenerating fonts or re-deriving from the original bundle is only needed if you deliberately revert the migration; for normal content/design work, edit the static file.

## Preview

Open `index.html` directly, or serve the directory (`python3 -m http.server`) and load `/`. Nothing to build. For a rendered screenshot / animation check, headless Chrome works: `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --screenshot=out.png --window-size=1280,2400 http://localhost:8000/`.

## Hosting & SEO

Deployed on **GitHub Pages** — static files only, no server-side processing. All SEO is baked into the committed markup: real `<title>` + `meta description`, `canonical` (`https://guidingwave.com/`), Open Graph + Twitter Card tags, `og-image.png`, and three `application/ld+json` blocks (`ProfessionalService`, `WebSite`, `FAQPage`). Content is fully present in the served HTML without JS. Keep it that way — do not reintroduce client-side content rendering, and keep the metadata/JSON-LD in sync with visible copy.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the website for **Mercy Children's Home**, a nonprofit caring for 32 orphaned children in Kasanda Bukuya, Uganda. It is a **pure static HTML site** — no framework, no build step, no package manager. All pages are self-contained `.html` files with all CSS and JavaScript inlined directly within them.

Deployed via **GitHub Pages** at: `https://kensports.github.io/mercychildrenshome/`

## Development Workflow

There is no build process. To develop:
- Edit `.html` files directly and open them in a browser, or use a local static server:
  ```
  python3 -m http.server 8080
  # or
  npx serve .
  ```
- Push to `main` to deploy (GitHub Pages auto-deploys from the main branch).
- There are no tests, no linter, and no CI pipeline.

## Architecture

### File Structure

Every page is fully standalone — nav, CSS, and JS are duplicated across files. There is no shared component system or templating engine.

| File | Purpose |
|------|---------|
| `index.html` | Main landing page (~2,000 lines): hero, about, mission, gallery, donation perks, volunteer stories, contact, admin panel |
| `about.html` | Founder Alex Ssengonzi's story, children overview |
| `mission.html` | Six pillars of care |
| `gallery.html` | Photo gallery |
| `donate.html` | All donation methods (GoFundMe, Mobile Money, PayPal, Wise, bank) |
| `impact.html` | Impact statistics |
| `videos.html` | YouTube/TikTok embeds |
| `sitemap.xml` | SEO sitemap |

### Supabase Backend

All dynamic content is managed through a **Supabase project** using its REST API and Storage directly from the browser (no server-side layer). The credentials are hardcoded in `index.html`:

```js
var SUPA    = 'https://qbyisxlqpuwowbtrying.supabase.co';
var SUPA_KEY = '...';   // anon public key
var BUCKET  = 'mercy-photos';
var BASE    = SUPA + '/storage/v1/object/public/' + BUCKET + '/';
```

The single database table is `mch_photos`, which is used as a key-value store. The `id` column is the key and `filename` stores either a storage filename or a JSON blob:

| Row `id` | `filename` value | Purpose |
|----------|-----------------|---------|
| `gallery_0` … `gallery_N` | image filename string | Hero/gallery photo slots |
| `story_0` … `story_2` | image filename string | Children's story photos |
| `stories_data` | JSON array of story objects | Story text & metadata |
| `impact_data` | JSON `{items:[...], updated:...}` | Impact statistics |
| `videos_data` | JSON array of video objects | YouTube/TikTok links & captions |
| `supporters_list` | JSON array of volunteer objects | Volunteer names, countries, photo URLs |
| `admin_pwd` | password string | Admin panel password |

### Image Naming Convention

Uploaded images follow this pattern: `{type}_{timestamp}.jpg`

Examples: `gallery_0_1776349884076.jpg`, `supporter_1777409225544.jpg`, `founder_1777409225544.jpg`

When replacing a key photo (e.g. `gallery_0`), the new file keeps the `gallery_0` prefix but gets a new timestamp suffix, and the `mch_photos` row is updated to point to the new filename.

### Admin Panel

A password-protected admin UI lives inside `index.html` (`#pwdOverlay`, `.admin-overlay`). The password is fetched from Supabase (`id=admin_pwd`) on submit. The admin panel manages:
- Key photo slots (hero image, gallery images, story photos)
- Impact statistics (4 editable metrics)
- Videos (add/remove YouTube/TikTok links)
- Volunteers/supporters (add, edit name/country, change photo, remove)

### Design System

CSS custom properties defined in each file's `:root` block:

| Variable | Value | Role |
|----------|-------|------|
| `--navy` | `#0d1f3c` | Primary brand color, nav, hero backgrounds |
| `--gold` | `#c9a84c` | Accent, CTAs, borders |
| `--cream` | `#faf8f5` | Page background |
| `--text2` | `#4a5568` | Body text |

Typography: **Fraunces** (serif, headings) and **Manrope** (sans-serif, body), both loaded from Google Fonts.

Animation: Scroll-triggered reveal via `.reveal` / `.reveal.visible` classes toggled by an `IntersectionObserver`.

### Internationalization

Google Translate is integrated via their JavaScript API. Language selection sets a `googtrans` cookie (`/en/{lang}`) and reloads the page. The language `<select>` is in the footer of each page and syncs from the cookie on load.

### Video Handling

The `videos.html` page supports YouTube full links, YouTube short links (`youtu.be`), TikTok full links, and TikTok short links (`vm.tiktok.com`). Short TikTok links are resolved via the TikTok oEmbed API before embedding.

### Key Patterns to Follow When Editing

- **Cross-page consistency**: Nav, footer, Google Fonts links, Font Awesome CDN link, and the CSS color variables must be kept consistent across all `.html` files since there is no shared template.
- **Image compression**: All image uploads go through a client-side `compressImage()` function (canvas-based) before upload to Supabase Storage — always preserve this step in any new upload flows.
- **Supabase upsert pattern**: Data writes use `Prefer: resolution=merge-duplicates` header so inserts act as upserts on the `id` column.
- **No external JS libraries**: All JavaScript is vanilla ES5-compatible. Do not introduce npm packages, bundlers, or ES6 module syntax.
- **Print stylesheet**: `@media print` rules in `index.html` hide the nav, floating buttons, and overlays — keep this in mind when adding new fixed/overlay elements.

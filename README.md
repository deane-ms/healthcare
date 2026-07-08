# Wellview Clinic

A single-page marketing site for **Wellview Clinic**, a fictional Singapore healthcare clinic. Built as one self-contained `index.html` — no build step, no dependencies, no backend of its own (the enquiry form delivers via [FormSubmit](https://formsubmit.co) instead).

🔗 **Live site:** https://deane-ms.github.io/healthcare/ (deployed automatically to GitHub Pages on every push to `main`)

![Wellview Clinic screenshot](assets/screenshot.png)

## Sections

- **Navbar** — sticky, with a mobile hamburger menu
- **Hero** — headline, CTAs, and a trust strip (support hours, certifications, patients served)
- **Services** — general consultation, pediatric care, diagnostics & lab, dental care
- **Meet Our Doctors** — team profiles with photo, specialty, and bio
- **Testimonials** — patient quotes and star ratings
- **Your First Visit** (`#first-visit`) — a 3-step visit journey plus a free, ungated "First Visit Checklist" lead magnet (real client-side download/print, no backend required)
- **Enquiry form** (`#contact`) — client-side validated appointment request form, delivered by email via FormSubmit
- **Footer** — quick links, contact details, social links

## Getting started

There's nothing to install or build. To preview the site locally, just open `index.html` in a browser:

```
Start-Process index.html
```

or double-click the file in Explorer.

> The page hotlinks all photos from `images.unsplash.com`, so an internet connection is needed to see images render.

## Deployment

Pushing to `main` triggers [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml), which publishes the repository root to **GitHub Pages**. No build step runs — the static files are uploaded as-is.

## Tech

- Plain HTML5, CSS3 (custom properties for theming), and vanilla JavaScript
- No frameworks, package manager, or bundler
- Type pairing: Fraunces (display headings) + Poppins (body/UI), loaded from Google Fonts
- Accessibility touches: skip link, `aria-*` attributes, focus-visible styles, `prefers-reduced-motion` support
- Scroll-triggered, staggered fade-in animations via `IntersectionObserver`
- SEO: meta description, Open Graph/Twitter tags, canonical link, `MedicalClinic` JSON-LD structured data, `robots.txt` and `sitemap.xml`
- Images use `loading="lazy"` + explicit `width`/`height` below the fold to avoid layout shift; the hero photo is `fetchpriority="high"`

## Enquiry form delivery (FormSubmit)

The `#contact` form posts to [FormSubmit](https://formsubmit.co)'s AJAX endpoint (`FORMSUBMIT_ENDPOINT` in the inline script) — no server of our own required. Two things to know if you change the target address:

- **First submission to a new address needs activation.** FormSubmit emails a one-time "Activate Form" link instead of delivering the enquiry; click it once and the address is live going forward.
- **Testing requires `http(s)://`, not `file://`.** FormSubmit rejects submissions from pages opened directly as a file — serve the folder locally (e.g. a quick static server) to test the real network call.
- **Swap the email for a private hash.** After activating, FormSubmit gives you a hash you can use instead of the raw email (`https://formsubmit.co/ajax/<hash>`) so the address isn't sitting in this repo's public source.
- FormSubmit returns HTTP 200 even when it fails — the actual result is the JSON body's `success` field (`"true"`/`"false"`), which the script checks explicitly.

## Regenerating the screenshot

`assets/screenshot.png` is a full-page capture of `index.html`, taken with the Playwright MCP server configured in [`.mcp.json`](.mcp.json) (`@playwright/mcp`). Fade-in sections are hidden until scrolled into view, so capture with `prefers-reduced-motion: reduce` emulated (or scroll through the page) to render every section before screenshotting.

## Notes for contributors

- Everything lives in `index.html`, organized with HTML comment banners (e.g. `<!-- ================= HERO ================= -->`) marking each section — use these to navigate instead of searching class names.
- The enquiry form validates client-side, then delivers via FormSubmit (see above) — there's still no server-side code in this repo, FormSubmit is the entire "backend."
- See [CLAUDE.md](CLAUDE.md) for more detailed architecture and conventions notes.

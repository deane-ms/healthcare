# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repository contains a single self-contained file, `index.html`, implementing a one-page marketing/landing site for a fictional Singapore healthcare clinic ("Wellview Clinic"). There is no build step, no package manager, no bundler, and no test suite — all HTML, CSS (inline `<style>` in the `<head>`), and JavaScript (inline `<script>` before `</body>`) live in that one file.

## Commands

There is no build, lint, or test tooling in this repo. To preview changes, open `index.html` directly in a browser (double-click it, or `Start-Process index.html` on Windows) — there is no dev server to start.

## Architecture

Everything is in `index.html`, organized top-to-bottom as: sticky navbar → hero → services → "Meet Our Doctors" → testimonials → enquiry form (`#contact`) → footer. Each section is marked with an HTML comment banner (e.g. `<!-- ================= HERO ================= -->`) — use these to jump to the right region instead of searching class names.

- **Styling**: all CSS lives in one `<style>` block, driven by CSS custom properties defined once in `:root` (colors, radius, shadow, max-width). Change the palette/spacing by editing those variables rather than individual rules. Section-specific styles are grouped under comment headers (`/* ---------- Hero ---------- */`, etc.) in the same order the sections appear in the HTML.
- **Scroll behavior**: nav links are plain in-page anchors (`href="#services"`) relying on `html { scroll-behavior: smooth }` — no JS-driven scrolling.
- **Scroll-triggered animation**: any element with the `.fade-in` class is faded/slid in via a single `IntersectionObserver` in the inline script; it falls back to revealing everything immediately if `IntersectionObserver` is unsupported.
- **Mobile nav**: the hamburger button toggles an `.open` class on `#nav-links` and updates `aria-expanded`; there's no separate mobile markup, just a CSS breakpoint at `859px`/`860px`.
- **Enquiry form (`#enquiry-form`)**: fully client-side. On submit it prevents the default action, validates name/email/phone with regexes, toggles `aria-invalid` + a per-field `.error-message` span, and on success logs a plain data object to the console and shows a `.form-status.success` banner. The block comment inside the submit handler marks exactly where a real `fetch('/api/appointments', ...)` POST would replace the `console.log` — there is no backend.
- **Images**: person photos (hero, doctor cards, testimonial avatars) are hotlinked directly from `images.unsplash.com/photo-<id>?...` URLs with explicit `w=` size params sized for their display context (800px for hero/doctor cards, 300px for testimonial thumbnails) — the page requires network access to render these; there are no local image assets.

## Conventions worth preserving

- Keep new sections consistent with the existing pattern: a `.section-heading` block (`eyebrow` + `h2` + intro `p`), a responsive `grid-template-columns: repeat(auto-fit, minmax(...))` card grid, and `.fade-in` on major blocks for the scroll-reveal effect.
- Form fields follow a fixed structure: `.field > label + input/textarea + span.error-message`, with `id="{name}-error"` wired via `aria-describedby` on the input — replicate this for any new form field so the existing `setFieldError()`/`validateForm()` JS logic can be extended consistently.

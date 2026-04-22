# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project purpose

This repo is a **single-page marketing-research microsite** for the product idea **MindfulNest** — a structured workspace for a property buyer and their real-estate agent (an alternative to the chaos of property-hunting in WhatsApp).

The site has three jobs on one page:
1. **Landing** — pitch the product and the problem in Russian, for real-estate agents.
2. **Report** — walk through the 8 key app screens (buyer side) with feature explanations.
3. **Questionnaire** — 14-question survey (Sean-Ellis PMF test + problem/solution validation + willingness to pay), filled out by practicing agents.

It is meant to be published on **GitHub Pages** (static hosting, no backend).

## Stack & conventions

- **Pure static HTML/CSS/JS in a single file** (`index.html`). No bundler, no framework, no package.json. Fonts are loaded from Google Fonts.
- Screenshots of the mobile app live in `scr/` as PNGs. Filenames contain spaces, so links in HTML must URL-encode them (`scr/2%20MindfulNest%20-%20Dashboard.png`).
- All user-facing copy is **in Russian**. Keep it that way unless the user switches language.
- Visual style is inspired by the reference report at `https://kishkisalexandra.github.io/ArchiveCSP/`: very dark navy background, violet (`#7C5CFF`) + teal (`#4DD0C1`) accents, `Space Grotesk` for headings, `Inter` for body, rounded cards with subtle gradients and glow effects.
- Survey submission has **no backend**. On submit the form:
  1. stores the response in `localStorage` under key `mindfulnest_responses`, and
  2. opens a `mailto:voytik_db@powerp.ai` prefilled with the JSON answers.
  There is also a "Download as JSON" button. If a real backend/Formspree/Google-Form is wired up later, replace the submit handler at the bottom of `index.html`.

## How to run / preview

```bash
# From the project root — any static server works
python3 -m http.server 8080
# then open http://localhost:8080
```

There is no build step. Editing `index.html` and refreshing the browser is the full dev loop.

## How to deploy to GitHub Pages

1. Push the repo to GitHub.
2. Repo → Settings → Pages → Source: `main` branch, `/ (root)`.
3. The site will be served from `https://<user>.github.io/<repo>/` — `index.html` is already the entry point.

## Architecture notes (single-file, but still worth knowing)

`index.html` is organised top-to-bottom as one scroll:

- **`<nav class="top">`** — sticky, translucent header with anchor links to each section.
- **`<header class="hero">`** — eyebrow pill, gradient headline, problem teaser, meta-stats, tab pill-nav, 4 stat cards.
- **`#problem`** — "split" comparison (WhatsApp today vs. MindfulNest).
- **`#solution`** — 5 alternating feature rows, each pairing one app screenshot with its explanation.
- **`#screens`** — thumbnail gallery of all 8 mockups; each tile links to the full-size PNG.
- **`#insights`** — 6-card grid of WOW differentiators.
- **`#api`** — API-connector / integrations pitch: hero card with hub-and-spoke wiring diagram (AmoCRM, Bitrix24, portals, WhatsApp, Google, Zapier), 6-card integrations grid, and a fake terminal code card showing a `POST /v1/cases` + webhook example. Classes: `.api-hero`, `.wiring`, `.pipes`, `.integrations`, `.integ`, `.code-card`.
- **`#survey`** — the form itself, grouped into 4 parts matching the questionnaire structure. The 1–10 scale for Q4 is generated in JS.
- **`<footer>`**.

CSS is one `<style>` block at the top. Custom properties on `:root` drive the palette — change `--violet`, `--teal`, `--bg` etc. to rethemem globally.

JS at the bottom handles: tab → scroll, IntersectionObserver to keep the active tab synced with scroll position, building the 1–10 scale, "download JSON", and the submit handler (localStorage + mailto).

## When editing

- **Do not split into multiple files** unless the user explicitly asks. Single-file is deliberate — it makes GitHub Pages trivial and keeps the site fast.
- **Do not introduce build tooling** (no webpack/vite/npm). Keep it to plain HTML/CSS/JS.
- If the user asks to add/remove survey questions, update *both* the markup in the `#survey` section *and* the `collectAnswers()` logic if field names change.
- The reference style (dark + violet/teal) is intentional — don't "modernise" it to a light theme without confirmation.
- Screenshots in `scr/` are the source of truth for what the app looks like; if a new screen is added, place it in `scr/` with a numbered filename and add it to both the feature rows (if relevant) and the `#screens` gallery.

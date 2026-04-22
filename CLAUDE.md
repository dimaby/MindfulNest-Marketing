# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project purpose

This repo is a **single-page marketing-research microsite** for the product idea **MindfulNest** — a structured workspace for a property buyer and their real-estate agent (an alternative to the chaos of property-hunting in WhatsApp).

The site has three jobs on one page:
1. **Landing** — pitch the product and the problem in Russian, for real-estate agents.
2. **Report** — walk through the 8 key app screens (buyer side) with feature explanations.
3. **Questionnaire** — 14-question survey (Sean-Ellis PMF test + problem/solution validation + willingness to pay), filled out by practicing agents.

It is meant to be published on **GitHub Pages** (static hosting, no backend).

There is also a Russian `README.md` aimed at humans browsing the repo — keep it in sync with this file when conventions change.

## Stack & conventions

- **Pure static HTML/CSS/JS in a single file** (`index.html`). No bundler, no framework, no package.json. Fonts are loaded from Google Fonts.
- Screenshots of the mobile app live in `assets/` as PNGs with numbered, hyphenated names: `01-login.png`, `02-dashboard.png`, `03-property-detail.png`, `04-add-case.png`, `05-onsite-capture.png`, `06-compare.png`, `07-chat.png`, `08-my-agents.png`. No spaces in filenames — no URL-encoding needed.
- All user-facing copy is **in Russian**. Keep it that way unless the user switches language.
- Visual style is inspired by the reference report at `https://kishkisalexandra.github.io/ArchiveCSP/`: very dark navy background, violet (`#7C5CFF`) + teal (`#4DD0C1`) accents, `Space Grotesk` for headings, `Inter` for body, rounded cards with subtle gradients and glow effects.
- Survey submission uses **Formspree** (endpoint `https://formspree.io/f/mvzdwjpn`, constant `FORMSPREE_ENDPOINT` in JS). The endpoint ID is public by design — Formspree throttles and captchas on the server side, no secret needed in the browser. On submit:
  1. Saves a safety copy to `localStorage` under key `mindfulnest_responses`.
  2. Builds a human-readable payload via `buildFormspreePayload()`: maps raw form slugs (`"yes-constant"` etc.) to full Russian sentences via the `MAP` object, and uses full question text with a numeric prefix as keys (so the Formspree dashboard and email are readable: `"01. Узнаёте ли Вы проблему..."`).
  3. Keys are **disambiguated with `a`/`b` suffixes** where one question maps to multiple fields — e.g. `06a. 3 самые полезные функции` / `06b. Другое (функция)`, `09a. Начнёте ли использовать сегодня` / `09b. Барьеры (комментарий)`, `10a/10b`, `11a/11b`. This stops Formspree (which sorts keys alphabetically) from collapsing or reordering duplicates like two `"09. ..."` rows.
  4. POSTs JSON with `Content-Type: application/json` and `Accept: application/json` so Formspree replies with JSON instead of redirecting.
  5. Special Formspree fields: `_subject` (email subject), `_replyto` (respondent's email — set only when the `contact` field passes a basic email regex; otherwise Formspree 422s on non-email text).
  6. On success → hides the form, shows `#thanks`. On failure → re-enables the button and shows `.submit-error` with fallback mailto to `me@dimaby.com`.
- **Required fields.** Key radio groups carry `required` — Q1, Q2, Q4 (1–10 scale), Q5, Q9, Q11, Q14. The browser blocks submit and scrolls to the first missing one, so critical answers can't silently arrive empty. Checkboxes, textareas and the contact fields are left optional on purpose.
- If you need to rename a form field, update **both** the option's `value` attribute in HTML and the corresponding key in the `MAP` object. If you rename/reorder a question, also update its key string in `buildFormspreePayload()`.
- Formspree free tier is 50 submissions/month. At that cap, move to Formspree paid, Cloudflare Worker + direct-to-Airtable, or Google Apps Script.
- **Historical note:** An earlier Airtable backend (table `Responses` in base `appTWpWhBl6vFYBD0`) was briefly wired in and then removed. Nothing in `index.html` references it anymore. The PAT that was committed in that period should be considered leaked and rotated.

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
- **`#solution`** — 5 feature rows, consistent layout (image left, description right). Descriptions are top-aligned.
- **`#screens`** — thumbnail gallery of all 8 mockups; each tile links to the full-size PNG. Responsive grid collapses 4 → 3 → 2 → 1 columns on narrower viewports.
- **`#insights`** — 6-card grid of WOW differentiators.
- **`#api`** — API-connector / integrations pitch: hero card with hub-and-spoke wiring diagram (AmoCRM, Bitrix24, portals, WhatsApp, Google, Zapier) and a 6-card integrations grid. No code example — the user explicitly removed it. Classes still in CSS: `.api-hero`, `.wiring`, `.pipes`, `.integrations`, `.integ`. (There's dead `.code-card` CSS left over from the removed terminal card; safe to delete if cleaning up.)
- **`#survey`** — the form itself, grouped into 4 parts matching the questionnaire structure. The 1–10 scale for Q4 is generated in JS.
- **`#thanks`** — hidden by default; shown after successful Formspree submit.
- **`<footer>`**.

CSS is one `<style>` block at the top. Custom properties on `:root` drive the palette — change `--violet`, `--teal`, `--bg` etc. to rethemem globally.

JS at the bottom handles: tab → scroll, IntersectionObserver to keep the active tab synced with scroll position, auto-checking the parent radio/checkbox when a user focuses an inline "Другое:" text field, building the 1–10 scale for Q4, collecting form data (`collectAnswers`), building the labelled Formspree payload (`buildFormspreePayload`), and the submit handler (localStorage safety copy → `fetch` to Formspree → success / error state).

### `.opt` / `.opt .extra` gotcha

Options in the survey are `<label class="opt">` with a radio/checkbox inside. Some options have an inline "Другое:" text field (`.opt.opt-inline` with an `<input type="text" class="extra">`). The `.opt input[...]` rules that style the custom checkbox/radio visuals (`appearance: none`, `width: 18px`, `display: inline-grid; place-items: center`) **must be scoped to `[type="checkbox"], [type="radio"]`** — never a bare `.opt input`, otherwise they leak into the text input and center/shrink its content. The text field is styled by `.opt .extra` which uses `flex: 0 0 100%` to force itself onto a new full-width row inside the option card.

## When editing

- **Do not split into multiple files** unless the user explicitly asks. Single-file is deliberate — it makes GitHub Pages trivial and keeps the site fast.
- **Do not introduce build tooling** (no webpack/vite/npm). Keep it to plain HTML/CSS/JS.
- **Do not add a terminal/code example to `#api`** — the user explicitly rejected it. Keep the section conceptual.
- If the user asks to add/remove survey questions, update the markup in `#survey`, the `MAP` object (if the question has predefined options), and the `buildFormspreePayload()` key list. Keep the numeric prefix convention (`NN.` or `NNa./NNb.` when one question produces multiple fields).
- The reference style (dark + violet/teal) is intentional — don't "modernise" it to a light theme without confirmation.
- Screenshots in `assets/` are the source of truth for what the app looks like; if a new screen is added, place it in `assets/` with a numbered filename (`09-...png` etc.) and add it to both the feature rows (if relevant) and the `#screens` gallery.
- Keep `README.md` (Russian, reader-facing) in sync with structural or workflow changes described here.

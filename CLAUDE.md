# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The homepage for the **NeurIPS 2026 Workshop on Resource-Aware Agentic AI**: a static, single-page site served by GitHub Pages. There is no build step, no package manager, no tests, and no CI. What is committed is what is served.

The remote is `git@github.com:Resource-Aware-Workshop/resource-aware-workshop.github.io.git`. An org `*.github.io` repo publishes from the default branch root, so **pushing to `main` deploys**. `.nojekyll` disables Jekyll processing so `assets/` is served verbatim.

Page content comes from the workshop proposal at `../neurips_workshop_proposal/proposal.tex` (summary, topic list, submission timeline, organizer list). Treat that file as the source of truth when updating copy, and do not invent details it does not state. Speakers, panelists, the schedule, and the OpenReview link are deliberately "to be announced" on the live page, even though the proposal names four confirmed speakers.

Where the NeurIPS 2026 acceptance email supersedes the proposal, the email wins: location is **Atlanta**, the workshop falls on **December 12-13, 2026** (exact day still pending from NeurIPS), and the **mandatory** accept/reject notification date is **September 29, 2026** rather than the proposal's September 28.

The page limit on the live site is **up to 8 pages excluding references and appendices**, a deliberate organizer decision that overrides the proposal's 4 pages. Do not "correct" it back to match `proposal.tex`.

## Working on it

Preview locally (relative asset paths require a server, `file://` breaks them):

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000`. Verification is visual. After edits, load the page and check the browser console for 404s on images, since missing assets fail silently.

## Architecture

**Everything renders from `index.html`.** All content is hardcoded HTML: no templating engine, no data-binding, no JS that injects content.

Layer breakdown:

- `assets/vendor/` — Bootstrap 5.3, AOS (scroll animations), Swiper, GLightbox, bootstrap-icons, php-email-form. Unmodified third-party drops. Never edit these.
- `assets/css/style.css` — the theme stylesheet, and the **only** CSS file `index.html` links. Section backgrounds, `#hero`, `#header`, `.speaker`, and `.section-with-bg` are defined here.
- Inline `<style>` in the `index.html` `<head>` — page-specific overrides layered on the theme: `.schedule-table`, `.tba-note`, and the 150px circular crops for `#organizers` and `#speakers` images.
- `assets/js/main.js` — stock BootstrapMade "TheEvent" theme JS: scrollspy, smooth-scroll with header offset, mobile nav toggle, back-to-top, AOS init, Swiper/GLightbox init. It also carries handlers for theme features this site does not use (gallery slider, buy-ticket modal); they no-op when the markup is absent.

The visual theme is BootstrapMade "TheEvent". Sass sources are not included (`assets/scss/Readme.txt` says they are pro-only), so **all styling changes are hand-edited CSS**.

### Section conventions

Sections live inside `<main id="main">` and follow a strict pattern. Breaking it breaks navigation or the visual rhythm:

1. Each `<section id="...">` must have a matching `<li><a class="nav-link scrollto" href="#...">` in the header nav, and vice versa. `main.js` scrollspy resolves a nav link's `hash` via `document.querySelector`, so a nav link with no matching section silently does nothing.
2. Alternate `class="section-with-bg"` on every other section. That class produces the alternating light/dark bands, so adding or removing a section means re-checking the alternation on the ones after it.
3. Wrap content in `<div class="container" data-aos="fade-up">` and use `<div class="section-header"><h2>Title</h2><p>subtitle</p></div>`. Dropping `data-aos` makes a section appear without the fade the rest of the page uses.

**People cards** (speakers, organizers) all reuse one markup shape; only the enclosing section id changes which CSS applies:

```html
<div class="col-lg col-md-4 col-sm-6 col-6">
  <div class="speaker" data-aos="fade-up" data-aos-delay="200" style="text-align: center;">
    <img src="assets/img/organizers/first_last.jpg" alt="Name" class="img-fluid">
    <br><a href="https://homepage/" style="font-size: 1.2em; color: #00356B;">Name</a></br>
    <p>Affiliation</p>
  </div>
</div>
```

Add or remove people by duplicating or deleting one `col-*` block. Photos go in `assets/img/{speakers,organizers}/` named `first_last.jpg`; the six organizer headshots were pulled from each person's own homepage. Images are cropped to circles by CSS, not by the file, so any roughly square photo works, but a portrait with the face off-center gets cropped badly (Yuxuan Lu's is the current example).

Keep the organizer grid as **one** `<div class="row">` of `col-lg-4 col-md-6 col-6` cards. Six divides evenly by both 3 and 2, so no row ends up with an orphan card. Splitting people across two `row` divs, which the original theme did, is what produced the ragged layout.

### Section backgrounds live in CSS, not HTML

`#hero` and `#about` pull their photos from `assets/css/style.css` (`../img/atlanta-night.jpg` and `../img/about-bg.jpg`), not from any `<img>` tag. Swapping the hero photo means editing that CSS rule, not the markup. `assets/img/` holds unused alternates (`hero-bg-0.jpg`, `background_1.jpg`, `background_2.jpg`, `venue-info-bg.jpg`, `subscribe-bg.jpg`) left over from the theme.

The hero photo is an Atlanta night skyline by ibuki Tsubo from Unsplash. The Unsplash License allows commercial use without attribution, but the footer credits the photographer anyway; keep that line if the photo stays.

### Sections that were removed

The theme this site descends from also had Accepted Papers, Student Registration Grant, Sponsors, Partners, Panelists, and FAQ sections. They were deleted, not hidden. When papers are decided, note that the Accepted Papers list was plain hardcoded `<li>` markup generated out of band from an OpenReview export, with nothing fetched at runtime.

### Branding assets

All stock TheEvent artwork has been deleted. The header is now a **text logo** (`#logo h1`, "Resource-Aware" plus a red-accented `<span>`), sized down from the theme's 36px default by a rule in the inline `<style>` because the full nav does not fit alongside it otherwise. The favicon is `assets/img/favicon.svg`, a hand-written "RA" monogram. There is no apple-touch-icon; add a PNG and its `<link>` if one is wanted.

### Contact address

The Contact nav link and the CFP contact line both point at the workshop group address, `resource-aware-workshop@googlegroups.com`. Do not substitute an individual organizer's address.

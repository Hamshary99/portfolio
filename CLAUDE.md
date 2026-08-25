# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal portfolio for Mohammed Bahey El-Deen (backend engineer, Cairo) built as a
**FIFA Ultimate Team "Team of the Week" player card**. The football metaphor is not
decoration — it is the organizing principle of the entire page, and every section maps
onto it. Preserve the mapping when editing:

| Football framing      | Actually contains                        |
|-----------------------|------------------------------------------|
| Scouting Report       | Professional summary + attribute radar   |
| Loan Spells           | Work history (contract roles)            |
| Man of the Match      | Projects                                 |
| Player Traits         | Skills / working style                   |
| Agent Details         | Contact information                      |
| Contract Status       | Availability                             |

## Build / run

**There is no build system, no package manager, no dependencies, and no tests.**
Do not add a `package.json`, bundler, or test runner unless explicitly asked.

The entire site is one self-contained file. To view it, open
`index.html` directly in a browser (`file://` works — there are
no fetches, no modules, and no CORS constraints), or serve the directory statically:

```bash
python -m http.server 8000
```

Prefer the server over `file://` when touching the pack intro: `sessionStorage` throws on an opaque origin, which silently skips the animation.

`.claude/launch.json` defines this server as `portfolio` for the preview tooling.

The only external runtime dependency is the Google Fonts stylesheet in `<head>`
(Teko / Manrope / JetBrains Mono). Offline, the page still renders with fallbacks.

## File layout

Two tracked files. Both matter:

- **`index.html`** — the whole site (~870 lines), in three zones:
  - `9–578` `<style>` — all CSS, sectioned by `/* ---------- HERO / CARD / SECTIONS ---------- */`
  - `580–778` `<body>` — hero + four `section.panel` blocks + `footer`
  - `780–867` `<script>` — card tilt handler, then an IIFE that draws the radar
- **`CV.pdf`** — source of truth for roles, dates and stack; linked from the footer.
- **`DESIGN_TOKENS.md`** — the **design source of truth**, written as a spec handoff for
  porting to another stack. Read it before changing anything visual.

## Architecture notes

**Everything is CSS — there are zero images.** The card, the gold foil texture, the
avatar, the crest, and the background grid are all gradients, `clip-path`, and text.
Keep it that way; it is why the page has no asset pipeline.

**The card shape is a `clip-path` polygon**, not a border-radius or an image:

```css
clip-path: polygon(6% 0%, 94% 0%, 100% 7%, 100% 86%, 50% 100%, 0% 86%, 0% 7%);
```

**The radar chart is hand-rolled SVG** built by an IIFE at `:805`. The `<svg id="radar">`
element ships empty in the markup and is populated at runtime — `angleFor(i)` / `pointAt(i, r)`
convert stat index + radius to coordinates on a hexagon. No charting library; do not add one.

**Colors are `:root` custom properties** (`--bg-deep`, `--gold`, `--cyan`, `--ink`, `--dim`, …)
defined at `:10`. Use the tokens rather than literal hex values. Note that the gold *card body*
palette is intentionally separate from the page palette.

### Gotcha: the six stat values are duplicated

The same six attributes live in **two places that must be kept in sync manually**:

- `:613–618` — the `.stat-grid` markup on the card
- `:808–813` — the `stats` array feeding the radar

They currently agree, but they are stored in **different orders** (the card reads
`SYS, SEC, API, SCL, SQL, DBG` row-wise; the radar uses `SYS, API, SQL, DBG, SCL, SEC` so
adjacent axes read sensibly). Nothing validates them against each other. If asked to change
a stat, change both.

## Design constraints from DESIGN_TOKENS.md

That file names three things as **do not change without asking** — they took several
iterations and an agent re-deriving them tends to regress the proportions:

1. The `clip-path` shield shape (a wavy/scalloped top was tried and rejected)
2. The gold → dark plate transition (the dark plate must flow continuously down through
   the shield's point, not stop short as a floating rounded box)
3. The tilt + sparkle + glow interactions

Copy CSS values from the HTML directly rather than re-implementing from the prose description.

## Pack opening intro

The page opens with a **FIFA 15-style pack opening** that gates the hero until dismissed.
FIFA 15 predates the FIFA 17+ "walkout," so there is deliberately **no player silhouette**:
the pack bursts, the light clears, and the card itself is the payoff.

Choreography (JS timings and CSS durations must be changed together):

| t (ms) | What happens                                                        |
|--------|---------------------------------------------------------------------|
| 0      | click -> `.pack` gets `.is-shaking` (0.5s)                           |
| 500    | `.is-bursting` + `.pack-flash.is-firing` + 46 shards; scene `.is-clearing` |
| 800    | `#card.is-dealing` — 1.2s flip from `rotateY(180deg)`                |
| 2060   | `finish()` — scene removed, `#card.is-dealt` triggers the sheen sweep |

### How it is wired

- **`body.pack-active` is the master switch.** A small inline script directly after the
  overlay markup adds it before the hero paints. CSS keys everything off it:
  `body:not(.pack-active) .pack-scene{ display:none; }`. So **with JS disabled, or under
  `prefers-reduced-motion`, or on a repeat visit, the class is never added and the page
  renders normally** — the intro can never become a hard gate.
- **The reveal uses the real card, not a copy.** A `.card-back` div was added inside
  `#card`; with `backface-visibility:hidden` on both faces and the existing
  `transform-style:preserve-3d`, the flip is a genuine 3D rotation of the actual card.
  Do not introduce a duplicate card for the animation.
- **The scene clears rather than fading out.** `.pack-bg` and `.pack-rays` fade to
  transparent at burst time so the card deals *underneath* a still-present overlay. The
  flash and shards sit above it. Without this the card would flip behind an opaque layer.
- **Specificity trap:** `body.pack-active #card{opacity:0}` outranks a bare
  `#card.is-dealing`, so the reveal rule is written `body.pack-active #card.is-dealing`.
  Keep the `body` prefix or the card stays invisible through the whole deal.
- **The tilt handler is gated.** `handleMove()` returns early while `pack-active` is set,
  so mouse movement cannot fight the deal animation by writing inline transforms.
- **Escape hatches:** a "Skip intro" button, the `Escape` key, and a 20s failsafe timer
  that calls `finish()` no matter what.
- **Replay:** guarded by `sessionStorage.packOpened`. To make it play on every load,
  delete that check in the inline script and in `finish()`.

### Verified in-browser

Sequence, burst, flip, mobile layout, no-JS CSS fallback, the tilt gate, and clean console
were all checked against a local server. **`prefers-reduced-motion` was verified by code
inspection only** — the browser tooling could not emulate the media query.

## Card ratings: where the numbers come from

Not arbitrary. The card follows FUT construction and is auditable against `CV.pdf`.

**OVR is the mean of the six face stats.** 76 + 77 + 72 + 75 + 73 + 82 = **455**, / 6 =
75.83 -> **OVR 76**. If you change any stat, either keep the sum at 455 or update the OVR,
the favicon (`76` is drawn into the inline SVG) and `og-card.png` to match.

| Stat | Value | Evidence in the CV |
|---|---|---|
| DBG | 82 | FX re-derivation bug traced across 5 microservices; race conditions caught in PR review |
| SEC | 77 | HMAC-SHA256 inter-service tokens, JWT, 3-role RBAC, Zod validation, rate limiting, OFAC/HIPAA/GDPR/CCPA/CMA |
| SYS | 76 | 4-service architecture, Postgres inbox idempotency, provider-aware rounding in a shared core package |
| SCL | 75 | Latency tuning, load balancing, query optimisation (**stated by the owner, not yet in the CV**) |
| SQL | 73 | Postgres inbox table, Decimal.js precision NAV maths, TypeORM, four engines |
| API | 72 | 26+ endpoints across projects, dedicated gateway service, REST/Express/NestJS |

An earlier revision rated SYS 81 / SEC 72 / DBG 83 with POT 89 and position CAM. Those were
changed because: SEC was the most under-rated stat against the evidence, SYS outran it, POT 89
implied a +13 growth curve that read as a boast while sitting next to a sentence reaching for
humility, and **CAM contradicted the hero copy** ("plays deep, holds the shape" is a CDM).

### Performance work: attributed, but still not in the CV

The owner confirmed the latency / load balancing / query optimisation / best-practices work
belongs to **Madkhol**, **Movies Reservation App** and **AI Assistant for Developers**
(explicitly *not* Reglint), and stated there are **no before/after numbers available**.

It is therefore written as work performed, with **no metrics and no outcome claims** - see the
Madkhol fixture, both project cards, the squad sheet "Performance" row, and the
"Tunes For Latency" trait. A regression check confirms zero numeric performance figures appear
in any user-facing copy.

**Do not add figures to these lines unless the owner supplies real ones.** If numbers arrive
later, the natural homes are the Madkhol fixture description and the two project cards.

**`CV.pdf` still does not mention any of it.** The page is now ahead of the CV, which is the
document recruiters and ATS filters actually read. Suggested bullets are in the session notes;
the PDF cannot be edited from here (no PDF library available in this environment).

## Deployment and sharing

The page is `index.html` at the repo root, so it serves at a host root with no config.
**The absolute URLs assume GitHub Pages at `https://hamshary99.github.io/portfolio/`.**
If you deploy anywhere else, three places must change together:

1. `og:url`
2. `og:image` / `twitter:image`
3. `"url"` and `"image"` in the JSON-LD block

`og-card.png` (1200x630, 111 KB) is generated, not hand-drawn - the source script lives in
the scratchpad, not the repo. It is a System.Drawing render using Bahnschrift Condensed as
a stand-in for Teko. To regenerate after a rating or name change, redraw it rather than
editing pixels. Deliberately **not** an SVG: Facebook, X and LinkedIn will not render one.

The JSON-LD `Person` block **omits email and telephone on purpose** - both are already
visible on the page, and adding them in machine-readable form makes bulk harvesting easier.

## Fonts

Only 7 weights are requested, and every one is used:

| Family | Weights | Used by |
|---|---|---|
| Teko | 600, 700 | display headings, ratings, card name |
| Manrope | 400, 700 | body copy; 700 is `<strong>` |
| JetBrains Mono | 400, 600, 700 | labels, stats, mono data |

Previously 12 were requested and 6 were dead, while JetBrains Mono 600 was *used but not
requested* and so was being synthesised by the browser. **If you add a `font-weight`, add
the matching weight to the Google Fonts URL**, and drop any weight you stop using.

Both preconnects matter: `fonts.googleapis.com` serves the CSS, `fonts.gstatic.com` serves
the actual `.woff2` files. Removing the `gstatic` one costs a full handshake mid-render.

## Print

There is a `@media print` block. The page is dark-on-dark, which prints as a black slab, so
the block flips to ink-on-paper, hides the intro/sparkles/CV button, and appends `href`
values after links via `::after` so a printed copy stays useful. Recruiters do print.

## CV and project links

`CV.pdf` (1 page, 307 KB, Word export with real text - not a scan) sits at the repo root and
is the **authoritative source for facts about roles, dates, and stack**. The page is a
stylised retelling of it; when the two disagree, the CV is right.

- Linked from the footer as "Full dossier" via `.cv-btn`, using
  `download="Mohammed-Bahey-El-Deen-CV.pdf"` so the file lands in the reader's downloads
  with a sensible name while staying `CV.pdf` in the repo.
- The three projects link to public GitHub repos (all verified reachable):
  `Hamshary99/Auto-Investment-Engine`, `Hamshary99/Movies-Reservation-App`,
  `Hamshary99/AI-Assistant-for-Developers`.

### Where the page and the CV diverge

Not bugs - deliberate or open decisions, but know them before editing copy:

- The CV titles the Madkhol role **"Backend Developer Intern"** and Reglint
  **"Software Developer"**. The page renders both identically as "On Loan", which does not
  convey that one was an internship.
- The CV lists **NestJS, MySQL, C/C++, AWS S3, GitHub Actions** under skills; none appear
  anywhere on the page. The page's "Player Traits" are soft traits only, so there is no
  hard tech-stack section.
- The CV carries the phone number and email as plain text. Publishing it in a public repo
  means both get scraped.

## Fixed issues (do not reintroduce)

All seven review findings plus the ray asymmetry are fixed and verified in-browser.
Each carries a trap worth knowing before editing:

1. **`.week-tag` overlap** — the old `margin:-2.4rem 0 2.4rem` assumed a one-line `<h1>`,
   but the `<h1>` has a hard `<br>`. Now `margin:0 0 2.4rem`; measured 8px clear.
   **Never reintroduce a negative top margin here.**
2. **Mobile card clipping** — the old `@media(max-width:520px){.card{width:260px;height:409px}}`
   shrank the box but not its fixed-px internals. Replaced with `zoom` on `.card-stage`
   (0.88 at <=520px, 0.76 at <=360px). `zoom` affects layout, unlike `transform`, so no
   compensating negative margins are needed, and a browser ignoring it renders the card
   unscaled rather than clipped. **Do not scale `.card` itself — the tilt handler and the
   deal animation both write to its `transform`.**
3. **Flag** — the `🇪🇬` emoji rendered as literal "EG" on Windows Chrome. Now an inline
   `<svg class="flag">`. Do not go back to a regional-indicator emoji.
4. **Sharing metadata** — description, `theme-color`, an inline-SVG favicon, and OG/Twitter
   tags are in `<head>`. `og:image` is still **commented out** and needs a real 1200x630
   **PNG** plus the deployed origin; SVG will not unfurl on Facebook/X/LinkedIn.
5. **Sub-9px type** — `.pot-tag`, `.chem-cap`, `.pack-sub` and the radar axis labels were
   raised to ~10.5-10.9px. This ate the card's entire vertical slack, so `.stats-plate`
   padding went `14px 26px 20px` -> `12px 26px 14px` and `.stat-grid` gap `18px` -> `15px`.
   **Re-check `face.scrollHeight === face.clientHeight` after any type-size change.**
6. **Radar accessibility** — the SVG now gets `role="img"` and an `aria-label` listing every
   value, built at draw time.
7. **`cursor:pointer` on `.card`** — removed; the tilt is hover-only and clicking does nothing.
8. **Lopsided light rays** — the pack rays used a 52deg conic period, and 52 does not divide
   360, so the pattern failed to tile. Now `repeating-conic-gradient` on a 45deg period.

### The stat duplication is gone

The card markup is now the **single source of truth**. `drawRadar()` reads the six values out
of `.stat-grid .stat` and only owns `AXIS_ORDER` (the presentational order of the axes).
Verified by changing DBG to 99 in the card markup alone and watching the radar polygon and
the aria-label both follow. Editing a stat in the HTML is now sufficient; there is no second
copy to keep in sync.

## Remaining known issues

- **Not deployed yet.** `index.html` is ready to serve, but GitHub Pages (or equivalent) has
  not been switched on, so every absolute URL above is currently a promise.
- **`prefers-reduced-motion` is still code-inspected only.** The browser tooling cannot
  emulate the media query, so that branch of the pack intro has never actually been run.
- **The `@media print` block has not been proof-printed** - it parses (14 rules in the CSSOM)
  and the selectors are sound, but no one has looked at a real print preview.
- **Only one layout breakpoint (520px)** outside the card. Tablet widths are unoptimised
  rather than broken.
- **Contact details are plain text**, on the page and inside `CV.pdf`. Publishing the PDF in
  a public repo means the phone number gets scraped. An open decision, not an oversight.
- **The football framing still has no plain-language fallback** for a reader who does not
  know FIFA Ultimate Team.

## Repository

- Remote `origin` → `https://github.com/Hamshary99/portfolio`, default branch `main`.
- The working copy was set up with `git init` + `remote add` + `fetch` + `checkout --track`
  rather than `git clone`, because the directory already contained `.claude/`.
- **`gh` is not authenticated** in this environment. `gh pr create` and pushes over HTTPS will
  fail until the user runs `gh auth login` themselves in an interactive terminal.
- `.claude/settings.local.json` is untracked and machine-local; it should not be committed.

# Portfolio Website - Claude Code Project Brief

## Repository
- **Repo**: `github.com/alxnhfr-bit/portfolio`
- **Live URL**: `https://alxnhfr-bit.github.io/portfolio/`
- **Hosting**: GitHub Pages (main branch, single `index.html`)
- **File**: Self-contained `index.html` (~50 KB) plus 16 `.webp` images in the repo root

## Owner
Alexander Neuhofer - Senior PM at Zalando, Berlin. Building AI product prototypes independently. Portfolio targets consumer PM roles in APAC.

## Architecture
Single `index.html`. No build step, no dependencies, no framework. Plain HTML/CSS/JS with Google Fonts via one `<link>`. All CSS lives in one `<style>` block in the head; all JS in one `<script>` at the end of the body. Images are external `.webp` files referenced by relative path (never base64). Deploy by pushing `index.html` to `main`.

### Images in the repo root
`avatar.webp` (256x256) and 15 screenshots, all 600px wide:
- `sundayatlas-{home,destinations,itinerary,map,inspo}.webp` (600x1304)
- `rise-{dashboard,training,ai-coach,wellbeing,supplements}.webp` (~600x1194)
- `brewlab-{landing,recipes,ratio,timer,shop}.webp` (~600x1132)

To convert or resize images use Python Pillow via **`/usr/bin/python3`**, which already has it; the Homebrew `python3` first on PATH does not, and refuses `pip install` under PEP 668. Note `sips` can read WebP but cannot write it.

### Video in the repo root
- `sundayatlas-flow.mp4` (1280x720, 36s, ~1.15MB) and `sundayatlas-flow-poster.webp` (17KB)

The uncompressed 19.4MB source (`Flow Video - Field Notes.mp4`) is gitignored and stays local. Re-encode with ffmpeg, not `avconvert`, whose presets are quality-targeted and barely shrink the file (best was 5.5MB, and its HEVC preset made it *larger*):

```
ffmpeg -i "source.mp4" -vf scale=1280:720 -c:v libx264 -preset slow -crf 30 \
  -pix_fmt yuv420p -an -movflags +faststart sundayatlas-flow.mp4
```

`-an` drops audio (the source has no audio track), `+faststart` lets it stream before fully downloading. CRF 30 at 1280x720 is visually indistinguishable from the 1920x1080 source at the size the drawer renders it, and gives retina headroom for the ~520px display width.

## Design System

### Style
Quiet, light editorial gallery. Large serif headline, five tinted project tiles with the app screens composed like product shots, and a slide-in drawer holding each full case study. No accent color; emphasis comes from the serif display face and generous whitespace.

### Fonts (one Google Fonts link)
- **Display**: Instrument Serif 400 (h1, project names in tiles and drawer)
- **UI/body**: Instrument Sans 400/500/600

### Colors (CSS custom properties)
```
--bg:         #fcfcfb   page background
--ink:        #17171a   primary text, dots, dark diagram cards
--text-2:     #55555a   secondary text, taglines, detail paragraphs
--text-3:     #6b6b70   tertiary, meta, labels, captions
--hair:       #e8e8e6   hairline rules and image borders
--btn-border: #e0e0de   pill button border
--dot-muted:  #b3b3b6   status dot for Prototype
--img-ph:     #f0f0ee   image placeholder background
```
Tile tints are `oklch()` with a hex fallback declared first:
SundayAtlas `#f6efe1` / `oklch(0.955 0.025 75)`; Signal `#eceff7` / `oklch(0.955 0.018 250)`; JobAgent `#f4edf5` / `oklch(0.955 0.02 320)`; Rise `#e6f4ec` / `oklch(0.955 0.025 160)`; BrewLab `#f7ece3` / `oklch(0.95 0.025 50)`. Drawer diagram panels: Signal `oklch(0.965 0.012 250)`, JobAgent `oklch(0.965 0.012 320)`.

Links are ink with `text-underline-offset: 5px` and `text-decoration-color: rgba(23,23,26,0.25)`, going to full ink on hover. Overlay is `rgba(23,23,26,0.28)` + `backdrop-filter: blur(3px)`.

### Type scale
- h1 `clamp(44px, 7.4vw, 116px)`, line-height 0.96, letter-spacing -0.022em, `text-wrap: balance`, max-width 1240px
- Tile name `clamp(28px, 2.6vw, 36px)` serif; tile number 13px; tile tagline 15px/1.45
- Drawer h2 `clamp(40px, 5vw, 56px)` serif; drawer tagline 19px/1.45
- Story: label 12px uppercase 0.06em; statement 19px/500; detail 15px/1.6
- Meta, status, nav, footer and "Case study" 13px; diagram node label 12px, node text 13px/1.4

### Layout and shape
- Container `max-width: 1560px`, side padding `--pad-x: clamp(16px, 5vw, 80px)` (the `.container` class). The design reference used `clamp(16px, 3vw, 40px)`; it was widened on request for more white space at the page edges, which also caps the grid at two columns (see Responsive Behaviour)
- Grid `repeat(auto-fit, minmax(min(100%, 480px), 1fr))`, gap 20px. SundayAtlas spans all columns
- Radii: tile 28px, tile screens 18px, drawer screens 16px, tile diagram cards 14px, drawer diagram cards 12px, drawer panels 20px, buttons 999px
- Shadows: tile screens `0 20px 40px -20px rgba(23,23,26,0.3)`; tile diagram cards `0 12px 28px -18px rgba(23,23,26,0.25)`; tile hover `0 36px 64px -44px rgba(23,23,26,0.4)`; drawer `-24px 0 80px rgba(23,23,26,0.12)`

### Tile screen compositions
Phones sit in a fixed-height stage with `overflow: hidden` and are deliberately cropped by its bottom edge. They overlap via negative margins and stack with z-index:
- SundayAtlas (flagship, stage `clamp(280px, 30vw, 420px)`): inspo (mt 64, mr -24, z1), destinations (mt 32, mr -24, z2), home (z3), itinerary (mt 32, ml -24, z2), map (mt 64, ml -24, z1). Screens `clamp(120px, 13vw, 176px)`
- Rise and BrewLab (stage `clamp(240px, 26vw, 340px)`): left (mt 40, mr -28, z1), centre (z2), right (mt 40, ml -28, z1). Screens `clamp(120px, 12vw, 160px)`
- Signal and JobAgent have no screenshots. They show a column of white diagram cards joined by 1px connectors, faded out with `mask-image: linear-gradient(to bottom, #000 70%, transparent 100%)`

**Images must keep `height: auto` in CSS.** They carry `width`/`height` attributes for CLS, and without `height: auto` that HTML `height` attribute (a presentational hint) beats `aspect-ratio` and the phones render full-height and hugely zoomed.

### Animation
- Tiles and drawer content fade up on reveal: opacity 0 to 1 and `translate: 0 14px` to 0, 0.6s `cubic-bezier(0.22,1,0.36,1)`, via IntersectionObserver (threshold 0.08, rootMargin `0 0 -6% 0`). Elements already in view on first load do not animate
- The reveal uses the CSS `translate` property, not `transform`, so the tile's `transform: translateY(-4px)` hover composes with it instead of fighting it
- Tile hover: lift 4px, 0.5s `cubic-bezier(0.22,1,0.36,1)`
- Drawer slide: `transform` 0.5s `cubic-bezier(0.32,0.72,0,1)`; overlay opacity 0.4s

### Forced light mode
`<meta name="color-scheme" content="light only">` plus a `@media (prefers-color-scheme: dark)` block that pins `background-color`/`color` so OS dark mode cannot invert the palette.

### Reduced motion
`@media (prefers-reduced-motion: reduce)` collapses all animation and transition durations and forces revealed elements visible. The JS also checks `matchMedia('(prefers-reduced-motion: reduce)')` and skips priming reveals entirely.

## Page Structure

```
Top bar (.topbar, not sticky)
  - avatar 28px + "Alexander Neuhofer"
  - LinkedIn, GitHub

Hero (header.hero)
  - h1: "I find friction in everyday experiences and turn it into focused, AI-powered products."
        (non-breaking hyphen &#8209; in AI-powered)
  - byline + "Connect on LinkedIn" / "GitHub"

main
  div.container.work
    - header row: "Selected work" / "Click a project for the case study"
    - grid of five tiles, each an <a href="#id" data-open="id">:
        01 SundayAtlas  Live . Flagship   (spans all columns, 5 screens)
        02 Signal       Live . Runs weekly (eval-loop diagram)
        03 JobAgent     Live . Runs daily  (daily-pipeline diagram, dark "3 . Score" card)
        04 Rise         Prototype          (3 screens)
        05 BrewLab      Prototype          (3 screens)

  div.drawer-overlay

  section.drawer  (the case studies)
    - sticky header "Case study" + Close pill
    - five <article class="case-study" id="sundayatlas|signal|jobagent|rise|brewlab">
        meta row, serif h2, tagline, optional live link
        SundayAtlas only: the flow video (figure.cs-flow) above the strip
        screens strip (SundayAtlas, Rise, BrewLab) or diagram panel (Signal, JobAgent)
        four story rows: Problem / Insight / Product|Build|Prototype / Takeaway
        SundayAtlas only: four feature lists in a 2-col grid
        "Next NN Name" pill linking to the next project (wraps 05 to 01)

Footer
  - "Alexander Neuhofer . 2026" / GitHub, LinkedIn
  - No email is shown; the design links GitHub instead
```

The case studies sit **inside `<main>` and before `<footer>`** on purpose. With JS off they are the bulk of the page's content, so they must not fall outside the main landmark or after the contentinfo landmark. `position: fixed` still resolves against the viewport from there because no ancestor creates a containing block.

### Progressive enhancement (important)
The case studies are **real content in the DOM**, not JS-generated. An inline script in the head adds a `js` class to `<html>`.
- **Without JS**: `.drawer` is a static block at the end of the page, all five case studies are visible, the overlay and Close button are hidden, tiles are ordinary anchors that jump to their case study, and "Next" is an ordinary link.
- **With JS**: the same markup becomes a fixed slide-in drawer; only `.case-study.is-active` is displayed.

Because of this, **never move the case-study content into JavaScript** and never hide it with CSS that is not scoped under `html.js`.

### Drawer behavior
Open on tile click (`preventDefault`, `history.replaceState` to `#id`), lock body scroll, mark the background `inert`, focus the Close button. Close via the Close button, overlay click or Escape: animate out, unmount the active article after 500ms, restore scroll, drop `inert`, clear the hash, return focus to the tile that opened it. `#hash` deep-links into a case study on load and on `hashchange`. "Next" swaps the active article, scrolls the panel to top and re-focuses Close.

The SundayAtlas flow video carries **`data-src` rather than `src`**, plus `preload="none"`. `setActive()` calls `hydrateVideo()` to attach the real `src` only when that case study is opened, so the landing page downloads none of the 1.15MB (only the 17KB poster), and the file is not fetched at all until someone presses play. `stopVideo()` pauses it when you switch to another case study or close the drawer. A `<noscript>` link to the mp4 sits inside the figure so it stays reachable without JS.

Timing and focus details that are easy to regress:
- The drawer uses a **forced reflow** (`void panel.offsetWidth`) before adding `.is-open`, not `requestAnimationFrame`. rAF does not fire in a hidden or throttled tab, which left deep-linked drawers stuck closed.
- Focus must be set **after** `.is-open` lands, because the panel is `visibility: hidden` until then and a hidden element cannot take focus.
- `close()` flips the `isOpen` flag **synchronously, before** restoring focus. The focus guard keys off that flag, so if it is still set the guard bounces focus straight back into the closing panel and the tile never gets it.
- `switchTo()` must re-focus Close: the "Next" link lives inside the article being unmounted, so focus would otherwise fall to `<body>`, outside the open dialog.
- On a deep link the browser scrolls the panel to the target article, so `panel.scrollTop` is reset again on `load` and via short timeouts.
- The drawer wiring is attached **before** the reveal setup, and the reveal work is wrapped in `try`/`catch` that strips `.reveal` on failure. `html.js .case-study { display: none }` is applied by the head script unconditionally, so a throw in the decorative layer must never be able to leave the case studies unreachable.

## Content Rules
- **No em dashes or en dashes** in any content, in any encoding (literal, `&#8212;`, `&mdash;`, `&#8211;`, `&ndash;`). Use commas, semicolons, colons or periods.
- **No mention of "Lovable" by name** in SundayAtlas content. Rise and BrewLab may mention it.
- If an email is ever shown, encode the `@` as `&#64;` so Cloudflare's email protection cannot mangle it on GitHub Pages. The current design shows no email.
- Case-study copy is fixed. Do not rewrite, shorten or reorder it without being asked.

## Known Issues / Pending Work

### Inherited from the design reference (faithful, not bugs)
These all follow from the reference's own `vw`-based `clamp()` values. Changing any of them means deliberately diverging from the approved design, so they were left as-is.
- **Flagship fan is side-cropped on narrow screens**: `clamp(120px, 13vw, 176px)` floors at 120px for viewports up to ~923px, so the five-phone cluster is a fixed 504px while the stage is only ~358px at 390px. The outer two phones show as slivers. Scaling the overlap with the phone width, or dropping to three phones below ~540px, would fix it at the cost of fidelity.
- **Diagram fade crosses card two below ~1024px**: the `mask-image` tail is 30% of the stage height, so at the 240px minimum height it starts mid-card. A fixed tail (`linear-gradient(to bottom, #000 calc(100% - 56px), transparent)`) would keep card two fully opaque at every height.
- **Compositions underfill the 768 to 1042px band**: phone widths track `vw` while tile width tracks the grid column count, so single-column tiles in that band have wide empty margins. `container-type: inline-size` on `.tile` plus `cqw` units would make the fan track the tile instead.

### Actual pending work
- **`og:image` is still missing**, so shared links render a text-only card. A 1200x630 social image is the remaining SEO task.
- **`<title>` and `og:title` still say "Product Builder"** even though the hero role line was dropped in the redesign. Kept deliberately because the existing head meta and OG tags had to be preserved.
- **Signal has no repo link.** If the repo is made public, add a live link to the Signal case-study head like the other projects have.

## Responsive Behaviour
Everything is fluid via `clamp()` and the auto-fit grid; there are no hand-written width breakpoints.
- **<= ~1089px**: tiles stack to one column
- **> ~1089px**: two columns. The widened `--pad-x` leaves a 1400px content box at the 1560px cap, which is under the 1480px three 480px tracks would need, so the grid never reaches three columns. Row one is SundayAtlas full width, then Signal / JobAgent, then Rise / BrewLab
- **Drawer**: `width: min(100%, 600px)`, so full width on small screens
- Verified with no horizontal overflow at 390, 768, 1024, 1440 and 1920px

## How to Edit
1. Clone the repo locally
2. Edit `index.html`
3. Preview with `python3 -m http.server 4178 --directory .` and open `http://127.0.0.1:4178/` (a plain `file://` open also works, but a server matches production)
4. Check 390 / 768 / 1024 / 1440 / 1920px, open a case study, press Escape, and load a `#deep-link` directly
5. Push to `main` (GitHub Pages auto-deploys)

## Related Repos
- **BrewLab prototype**: `github.com/alxnhfr-bit/brewlab` (standalone HTML with React via CDN)
- **SundayAtlas**: deployed on Vercel at `sundayatlas.vercel.app`
- **Design handoff** for this redesign: `design_handoff_portfolio_redesign/` (README spec, `reference/Portfolio v4.dc.html` prototype, reference screenshots)

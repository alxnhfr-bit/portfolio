# Portfolio Website - Claude Code Project Brief

## Repository
- **Repo**: `github.com/alxnhfr-bit/portfolio`
- **Live URL**: `https://alxnhfr-bit.github.io/portfolio/`
- **Hosting**: GitHub Pages (main branch, single `index.html`)
- **File**: Self-contained `index.html` (~70 KB) plus 17 `.webp` images and one `.mp4` in the repo root

## Owner
Alexander Neuhofer - Senior PM at Zalando, Berlin. Building AI product prototypes independently. Portfolio targets consumer PM roles in APAC.

## Architecture
Single `index.html`. No build step, no dependencies, no framework. Plain HTML/CSS/JS with Google Fonts via one `<link>`. All CSS lives in one `<style>` block in the head; all JS in one `<script>` at the end of the body. Images are external `.webp` files referenced by relative path (never base64). Deploy by pushing `index.html` to `main`.

### Images in the repo root
`avatar.webp` (256x256), `sundayatlas-flow-poster.webp` (1280x720), and 15 screenshots, all 600px wide:
- `sundayatlas-{landing,trips,itinerary,creators,extract}.webp` (600x1304)
- `rise-{dashboard,training,ai-coach,wellbeing,supplements}.webp` (600x1188 to 600x1200; CSS uses `600 / 1194` and `object-fit: cover` absorbs the few px of drift)
- `15grms-{brew,recipe,brewing,complete,journal}.webp` (600x1224)

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
Light, quiet gallery built on one translucent **Liquid Glass** material. An animated ambient light field sits behind the page; every tile, the drawer header and the pill buttons are the same glass refracting it through `backdrop-filter`. Work is grouped into three chapters (Shipped, Agents, Prototypes) rather than one flat grid. There is **no accent color and there are no tinted tiles**; all color comes from the ambient field seen through the glass.

### Fonts (one Google Fonts link)
- **Display**: Instrument Serif 400 (h1, project names in tiles and drawer)
- **UI/body**: Instrument Sans 400/500/600

### Colors (CSS custom properties)
```
--bg:        #fcfcfb   page background
--ink:       #17171a   primary text, dark band, dark diagram cards
--text-2:    #55555a   secondary text, taglines, detail paragraphs
--text-3:    #6b6b70   tertiary, meta, labels, captions
--on-dark:   #fcfcfb   primary text on the dark band
--on-dark-2: #c9c9cc   secondary text on the dark band
--on-dark-3: #9a9aa0   tile number on the dark band
--hair:      #e8e8e6   hairline rules and drawer image borders
--dot-muted: #b3b3b6   status dot for Prototype
--img-ph:    #f0f0ee   drawer image placeholder background
```
Drawer diagram panels keep a tint: Signal `oklch(0.965 0.012 250)`, JobAgent `oklch(0.965 0.012 320)`.

Links are ink with `text-underline-offset: 5px` and `text-decoration-color: rgba(23,23,26,0.25)`, going to full ink on hover. Overlay is `rgba(23,23,26,0.28)` + `backdrop-filter: blur(3px)`. Focus ring is `2px solid currentColor` with `outline-offset: 3px`, so it inherits ink on light and near-white on dark.

### The Liquid Glass material
This is the core of the design; everything else is layout around it. It responds to two inputs: the ambient field behind it, and the pointer.

**Ambient light field (page)** - `.ambient`, a fixed non-interactive layer, first child of `.page`:
```
position: fixed; inset: -14%; z-index: -1; pointer-events: none;
four radial-gradients in oklch at calc(N% +/- var(--lx)) calc(N% +/- var(--ly))
filter: blur(34px) saturate(120%);
animation: omAmbient 30s ease-in-out infinite alternate;
```
`.page` carries `position: relative; isolation: isolate` so the layer can sit at `z-index: -1` and still paint above the wrapper's own `#fcfcfb`. `isolation` does **not** create a containing block for `position: fixed`, so the drawer still resolves against the viewport from inside it.

**Ambient light field (dark band)** - the Agents band is opaque `#17171a`, so it gets its own `.band-ambient` (`position: absolute; inset: -20%; z-index: -1`, three gradients, `blur(36px)`, `omAmbient 34s`). The band carries `position: relative; isolation: isolate; overflow: hidden`.

**Glass surface (light)** - `.glass`, four background layers in this order: the pointer-tracked specular, the body wash, a top-left lens, a bottom shade. Then `backdrop-filter: blur(30px) saturate(205%) brightness(1.04)` and a six-layer inset shadow (bright top rim, hairline rim all round, light gathering at the left and right edges, soft bottom shade, outer cast). **Dark variant** `.glass-dark` is the same structure with low-alpha white instead of tint, `saturate(165%) brightness(1.06)`, and a black-based shadow stack.

**Hover** - small tiles lift (`translateY(-4px)`) and the rim brightens. The flagship band deliberately does **not** lift, only brightens; it is near full screen, so a lift reads as a glitch. This matches the reference.

**Pointer specular** - two delegated listeners on `document`, both `{ passive: true }`. Each glass element is marked `data-glass` and declares its own `--mx: 50%; --my: -8%; --spec: 0`.
- `pointermove` finds `e.target.closest('[data-glass]')`, computes the cursor as a percentage of the element's box, and writes `--mx`, `--my`, `--spec: 1`.
- `pointerout` ignores the event if `e.relatedTarget` is still inside the element; otherwise it eases back to rest over 520ms with `1 - (1-k)^3`, driven by `requestAnimationFrame`. **Custom properties cannot transition without `@property`**, which is why this lerp is written by hand.

**Scroll-driven light drift** - one rAF-throttled `scroll` listener writing on `documentElement`:
```
p = min(1, scrollY / max(1, scrollHeight - innerHeight))
--lx = sin(p * PI * 1.6) * 90px
--ly = p * 180 - 60px
```
Called once on mount. Skipped entirely under `prefers-reduced-motion: reduce`.

**The JS only ever writes custom properties.** It never touches layout.

**Legibility** - glass must never cost contrast:
- Every tile's title row sits on a legibility plate (`.tile--light .tile-text` / `.tile--dark .tile-text`), a `linear-gradient(to top, ...)` from `rgba(255,255,255,0.62)` or `rgba(10,10,12,0.62)` to transparent, with row padding `16px clamp(20px, 2.5vw, 32px) clamp(24px, 2.5vw, 32px)`.
- Body copy stays full-opacity ink. Never tint text to match the glass.
- An `@supports not ((-webkit-backdrop-filter: blur(1px)) or (backdrop-filter: blur(1px)))` block thickens the body wash so text still reads where `backdrop-filter` is unsupported. It changes nothing in browsers that support it.

**`-webkit-backdrop-filter` is required** and must accompany every `backdrop-filter` declaration (currently 7 of each). Safari drops the effect entirely without it.

### Type scale
- h1 `clamp(44px, 7.4vw, 116px)`, line-height 0.96, letter-spacing -0.022em, `text-wrap: balance`, max-width 1240px
- Chapter labels 13px 500 ink (`#fcfcfb` on the dark band), `padding: 0 0 22px`, no rule and no counts
- Tile name `clamp(28px, 2.6vw, 36px)` serif; tile number 13px; tile tagline 15px/1.45
- Drawer h2 `clamp(40px, 5vw, 56px)` serif; drawer tagline 19px/1.45
- Story: label 12px uppercase 0.06em; statement 19px/500; detail 15px/1.6
- Meta, status, footer and "Case study" 13px; diagram node label 12px, node text 13px/1.4

### Layout and shape
- Container `max-width: 1560px`, side padding `--pad-x: clamp(16px, 3vw, 40px)`
- Radii: flagship `clamp(28px, 3vw, 44px)`, tiles 28px, tile screens 18px, drawer screens 16px, tile diagram cards 14px, drawer diagram cards 12px, drawer panels 20px, buttons 999px
- Grid gap 20px. Agents grid `repeat(auto-fit, minmax(min(100%, 360px), 1fr))`; Prototypes grid `repeat(auto-fit, minmax(min(100%, 320px), 1fr))`
- Dark band `margin: clamp(56px, 7vw, 104px) 0; padding: clamp(48px, 6vw, 88px) 0`
- Tile screens: `1px solid rgba(23,23,26,0.08)`, `box-shadow: 0 20px 40px -20px rgba(23,23,26,0.3)`

### Chapter compositions
Phones sit in a fixed-height stage with `overflow: hidden` and are deliberately cropped by its bottom edge. They overlap via negative margins and stack with z-index:
- **Flagship** (SundayAtlas), stage `clamp(320px, 38vw, 540px)`, screens `clamp(132px, 14.5vw, 204px)` at `600/1304`: extract (mt 64, mr -24, z1), trips (mt 32, mr -24, z2), landing (z3), itinerary (mt 32, ml -24, z2), creators (mt 64, ml -24, z1)
- **Prototypes** (Rise, 15GRMS), stage `clamp(200px, 20vw, 260px)` with `padding-top: clamp(28px, 4vw, 48px)`, screens `clamp(120px, 12vw, 160px)`: left (mt 40, mr -28, z1), centre (z2), right (mt 40, ml -28, z1)
- **Agents** (Signal, JobAgent) have no screenshots. Each shows its pipeline as a stack of glass cards joined by 1px x 14px connectors, in a `clamp(220px, 24vw, 320px)` container faded out with `mask-image: linear-gradient(to bottom, #000 74%, transparent 100%)`

**Images must keep `height: auto` in CSS.** They carry `width`/`height` attributes for CLS, and without `height: auto` that HTML `height` attribute (a presentational hint) beats `aspect-ratio` and the phones render full-height and hugely zoomed.

### Animation
- `@keyframes omAmbient` drives both ambient layers, `transform`-only (translate3d + scale) so it stays on the compositor
- `@keyframes omDrift` drives the flagship's scroll cue, a 26px chevron at `rgba(23,23,26,0.42)`, `3.6s ease-in-out infinite`, `pointer-events: none`, decorative only
- Tiles and drawer content fade up: opacity 0 to 1 and `translate: 0 14px` to 0, 0.6s `cubic-bezier(0.22,1,0.36,1)`, via IntersectionObserver (threshold 0.08, rootMargin `0 0 -6% 0`). Elements already in view on first load are shown, not animated
- The reveal uses the CSS `translate` property, not `transform`, so the tile's `transform: translateY(-4px)` hover composes with it instead of fighting it
- Drawer slide: `transform` 0.5s `cubic-bezier(0.32,0.72,0,1)`; overlay opacity 0.4s

### Forced light mode
`<meta name="color-scheme" content="light only">` plus a `@media (prefers-color-scheme: dark)` block that pins `background-color`/`color` on `html`, `body`, `.page` and the drawer so OS dark mode cannot invert the palette.

### Reduced motion
`@media (prefers-reduced-motion: reduce)` collapses animation and transition durations, kills `[data-ambient]` and `[data-cue]` outright, forces revealed elements visible, and drops the hover lift. The JS also checks `matchMedia` and skips both the scroll drift and the reveal priming. **The pointer specular is allowed to stay** (it is a direct response to input, not ambient motion).

## Page Structure

```
.page  (position: relative; isolation: isolate)
  .ambient                      the page light field, z-index -1

  Top bar (.topbar, not sticky)
    - avatar 28px + "Alexander Neuhofer"
    - LinkedIn, GitHub

  Hero (header.hero)
    - h1: "I find friction in everyday experiences and turn it into focused, AI-powered products."
          (non-breaking hyphen &#8209; in AI-powered)
    - byline + "Connect on LinkedIn" / "GitHub"

  main
    div.work
      Chapter "Shipped"
        - label inside .container
        - a.flagship.glass  OUTSIDE the container, full-bleed but rounded,
          min-height min(86vh, 900px), column with space-between:
            scroll cue (flex:1, pinned bottom) / 5 screens / title row
            (the title row is re-wrapped in a 1560px container so it
             aligns with the rest of the page)

      Chapter "Agents"  (.band, full-width #17171a)
        - .band-ambient, its own light field
        - 2-up grid of dark glass tiles:
            02 Signal    Live . Runs weekly (eval-loop stack)
            03 JobAgent  Live . Runs daily  (daily-pipeline stack, dark "3 . Score" card)

      Chapter "Prototypes"
        - 2-up grid of light glass tiles:
            04 Rise    Prototype  (3 screens)
            05 15GRMS  Prototype  (3 screens in the tile, all 5 in the drawer)

    div.drawer-overlay

    section.drawer  (the case studies)
      - sticky glass header "Case study" + Close pill
      - five <article class="case-study" id="sundayatlas|signal|jobagent|rise|15grms">
          meta row, serif h2, tagline, optional live link
          SundayAtlas only: the flow video (figure.cs-flow) above the strip
          screens strip (SundayAtlas, Rise, 15GRMS) or tinted diagram panel (Signal, JobAgent)
          four story rows: Problem / Insight / Product|Build|Prototype / Takeaway
          SundayAtlas only: four feature lists in a 2-col grid
          "Next NN Name" glass pill linking to the next project (wraps 05 to 01)

  Footer
    - "Alexander Neuhofer . 2026" / GitHub, LinkedIn
    - No email is shown; the design links GitHub instead
```

The case studies sit **inside `<main>` and before `<footer>`** on purpose. With JS off they are the bulk of the page's content, so they must not fall outside the main landmark or after the contentinfo landmark. `position: fixed` still resolves against the viewport from there because no ancestor creates a containing block.

### Progressive enhancement (important)
The case studies are **real content in the DOM**, not JS-generated. An inline script in the head adds a `js` class to `<html>`.
- **Without JS**: `.drawer` is a static block at the end of the page, all five case studies are visible, the overlay and Close button are hidden, tiles are ordinary anchors that jump to their case study, and "Next" is an ordinary link.
- **With JS**: the same markup becomes a fixed slide-in drawer; only `.case-study.is-active` is displayed.

Because of this, **never move the case-study content into JavaScript** and never hide it with CSS that is not scoped under `html.js`. The same rule governs the reveal: `[data-reveal]` elements carry **no** opacity styling until JS adds `.reveal`, so nothing can be stranded invisible.

### Drawer behavior
Open on tile click (`preventDefault`, `history.replaceState` to `#id`), lock body scroll, mark the background `inert`, focus the Close button. Close via the Close button, overlay click or Escape: animate out, unmount the active article after 500ms, restore scroll, drop `inert`, clear the hash, return focus to the tile that opened it. `#hash` deep-links into a case study on load and on `hashchange`. "Next" swaps the active article, scrolls the panel to top and re-focuses Close.

The SundayAtlas flow video carries **`data-src` rather than `src`**, plus `preload="none"`. `setActive()` calls `hydrateVideo()` to attach the real `src` only when that case study is opened, so the landing page downloads none of the 1.15MB (only the 17KB poster), and the file is not fetched at all until someone presses play. `stopVideo()` pauses it when you switch to another case study or close the drawer. A `<noscript>` link to the mp4 sits inside the figure so it stays reachable without JS.

Timing and focus details that are easy to regress:
- The drawer uses a **forced reflow** (`void panel.offsetWidth`) before adding `.is-open`, not `requestAnimationFrame`. rAF does not fire in a hidden or throttled tab, which left deep-linked drawers stuck closed.
- Focus must be set **after** `.is-open` lands, because the panel is `visibility: hidden` until then and a hidden element cannot take focus.
- `close()` flips the `isOpen` flag **synchronously, before** restoring focus. The focus guard keys off that flag, so if it is still set the guard bounces focus straight back into the closing panel and the tile never gets it.
- `switchTo()` must re-focus Close: the "Next" link lives inside the article being unmounted, so focus would otherwise fall to `<body>`, outside the open dialog.
- On a deep link the browser scrolls the panel to the target article, so `panel.scrollTop` is reset again on `load` and via short timeouts.
- The drawer wiring is attached **before** the glass and reveal setup, and both decorative layers are wrapped in `try`/`catch`. `html.js .case-study { display: none }` applies unconditionally, so a throw in the decorative layer must never be able to leave the case studies unreachable.
- The reveal keeps a `data-revealed="1"` attribute so an element that has been revealed once stays revealed across re-scans, and a **600ms safety timer** unconditionally reveals everything if no observer callback ever fires.

### Divergences from the design reference (deliberate)
The reference prototype is `reference/Portfolio v5b Chapters.dc.html`. Two things in it were **not** reproduced:
- Its `_scan()` removes the `pointermove` / `pointerout` / `scroll` listeners every time it runs, and it runs on every update, so the glass goes dead after the first drawer open. README section 5 says to remove them on *teardown*. This build follows the README.
- Its reveal is driven by inline `el.style.transform`, which would fight the tile's hover `transform`. This build uses the CSS `translate` property instead, which composes.

The reference was also authored against an older repo snapshot: it names nine images that no longer exist (`sundayatlas-home/destinations/map/inspo`, `brewlab-*`), calls project 05 "BrewLab", and gives it the tagline "AI Coffee Brewing Assistant". **Content was taken from the live `index.html`, not from the reference.**

### Naming and the 15grms id
Project 05 was renamed from BrewLab to **15GRMS**. The anchor id, the `data-open`/`data-next` values, the entry in the JS `IDS` array and the image filenames all use lowercase `15grms`; the visible name is uppercase `15GRMS`.

**That id starts with a digit, so `document.querySelector('#15grms')` throws** ("not a valid selector") because a CSS identifier cannot begin with a digit unescaped. The site is safe because its JS resolves case studies with `document.getElementById(id)` and only ever builds the hash as a string. If you ever need a selector, scope it off the element (`document.getElementById('15grms').querySelector(...)`) or escape it as `#\\31 5grms`. The same applies to any CSS rule or `:target` selector.

## Content Rules
- **No em dashes or en dashes** in any content, in any encoding (literal, `&#8212;`, `&mdash;`, `&#8211;`, `&ndash;`). Use commas, semicolons, colons or periods.
- **No mention of "Lovable" by name** in SundayAtlas content. Rise and 15GRMS may mention it.
- If an email is ever shown, encode the `@` as `&#64;` so Cloudflare's email protection cannot mangle it on GitHub Pages. The current design shows no email.
- Case-study copy is fixed. Do not rewrite, shorten or reorder it without being asked.
- **Regenerate the whole file rather than patching it.** Partial writes have corrupted it before.

## Known Issues / Pending Work

### Inherited from the design reference (faithful, not bugs)
- **The flagship fan is cropped on narrow screens.** `clamp(132px, 14.5vw, 204px)` floors at 132px below ~910px, so the five-phone cluster is a fixed 564px while the stage is only 390px at phone width; the outer two phones are clipped by the stage's `overflow: hidden`. Verified that this causes no page overflow. Scaling the overlap with the phone width, or dropping to three phones below ~540px, would fix it at the cost of fidelity.
- **The flagship title row has no legibility plate**, unlike every other tile. The README asks for a plate on every title row, but the reference's flagship omits it, and PROMPT.md says to trust the reference for values. Its text sits on open glass at the bottom of a 900px band, which reads fine today but is the one spot where contrast depends on what the ambient field is doing behind it.
- **The pipeline fade can cross card two on short containers**, since the `mask-image` tail is 26% of a `clamp(220px, 24vw, 320px)` height.

### Actual pending work
- **`og:image` is still missing**, so shared links render a text-only card. A 1200x630 social image is the remaining SEO task.
- **`<title>` and `og:title` still say "Product Builder"** even though the hero role line was dropped. Kept deliberately because the existing head meta and OG tags had to be preserved.
- **Signal has no repo link.** If the repo is made public, add a live link to the Signal case-study head like the other projects have.
- **15GRMS story copy still describes the old build** (ratio calculator, curated bean shop) rather than the dial-in feedback loop the current screens show, and its home screen says "The Adler Original" while the recipe and journal screens say "The Hoffmann Method".
- **Not yet checked in Safari.** All 7 `backdrop-filter` declarations carry `-webkit-backdrop-filter`, but the glass has only been verified in the Chromium-based preview.

## Responsive Behaviour
Everything is fluid via `clamp()` and auto-fit grids; there are no hand-written width breakpoints.
- **Below ~1000px** both 2-up grids stack to one column
- **Drawer**: `width: min(100%, 600px)`, so full width on small screens
- Verified with **no horizontal overflow at 390, 768, 1024, 1440 and 1920px**. The only elements extending past the viewport by design are the two ambient layers (one `position: fixed`, one inside `overflow: hidden`) and the closed drawer parked at `translateX(100%)`

## How to Edit
1. Clone the repo locally
2. Edit `index.html` (regenerate it whole; see Content Rules)
3. Preview with `python3 -m http.server 4302 --directory .` and open `http://127.0.0.1:4302/`
4. Check 390 / 768 / 1024 / 1440 / 1920px, open a case study, press Escape, and load a `#deep-link` directly
5. Push to `main` (GitHub Pages auto-deploys)

**Preview gotcha:** when the browser pane is not focused (`document.hasFocus() === false`), CSS transitions and `requestAnimationFrame` stall, and screenshots lag a step behind. A drawer that reports `.is-open` but a computed `transform` of `translateX(600px)` is this artifact, not a bug; confirm by setting `transition: none` and re-reading the computed value.

## Related Repos
- **15GRMS**, formerly BrewLab: the old build is `github.com/alxnhfr-bit/brewlab` (standalone HTML with React via CDN). The redesign is not deployed yet, which is why the case study has no live link
- **SundayAtlas**: deployed on Vercel at `sundayatlas.vercel.app`
- **Design handoff** for the Liquid Glass redesign: `design_handoff_portfolio_redesign/` (README spec, PROMPT.md, `reference/Portfolio v5b Chapters.dc.html`)

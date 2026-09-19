# Portfolio Website - Claude Code Project Brief

## Repository
- **Repo**: `github.com/alxnhfr-bit/portfolio`
- **Live URL**: `https://alxnhfr-bit.github.io/portfolio/`
- **Hosting**: GitHub Pages (main branch, single `index.html`)
- **File**: Self-contained `index.html` (~65 KB) plus 17 `.webp` images and one `.mp4` in the repo root

## Owner
Alexander Neuhofer - Senior PM at Zalando. Building AI product prototypes independently. Portfolio targets consumer PM roles in APAC.

Do not add location, job history or metrics he has not supplied. An earlier draft of the hero carried a city that was never in the source material and it was removed. The page's whole job is credibility, and there are no invented numbers anywhere in it.

## Architecture
Single `index.html`. No build step, no dependencies, no framework. Plain HTML/CSS/JS with Google Fonts via one `<link>`. All CSS lives in one `<style>` block in the head; all JS in one `<script>` at the end of the body. Images are external `.webp` files referenced by relative path (never base64). Deploy by pushing `index.html` to `main`.

### Images in the repo root
`avatar.webp` (256x256), `sundayatlas-flow-poster.webp` (1280x720), and 15 screenshots, all 600px wide:
- `sundayatlas-{landing,trips,itinerary,creators,extract}.webp` (600x1304)
- `rise-{dashboard,training,ai-coach,wellbeing,supplements}.webp` (600x1188 to 600x1200)
- `15grms-{brew,recipe,brewing,complete,journal}.webp` (600x1224)

The landing page uses only five of these (`avatar`, `sundayatlas-trips/landing/creators`, `15grms-brew`). The rest are used by the case-study drawer, so none can be deleted.

To convert or resize images use Python Pillow via **`/usr/bin/python3`**, which already has it; the Homebrew `python3` first on PATH does not, and refuses `pip install` under PEP 668. Note `sips` can read WebP but cannot write it.

### Video in the repo root
- `sundayatlas-flow.mp4` (1280x720, 36s, ~1.15MB) and `sundayatlas-flow-poster.webp` (17KB)

The uncompressed 19.4MB source (`Flow Video - Field Notes.mp4`) is gitignored and stays local. Re-encode with ffmpeg, not `avconvert`, whose presets are quality-targeted and barely shrink the file:

```
ffmpeg -i "source.mp4" -vf scale=1280:720 -c:v libx264 -preset slow -crf 30 \
  -pix_fmt yuv420p -an -movflags +faststart sundayatlas-flow.mp4
```

## Design System: the "Glass" direction

Implemented 2026-09-18 from `design_handoff_portfolio_glass/`. That bundle's `README.md` is the source of truth for values; `Portfolio Glass.dc.html` is the reference prototype. The previous design (chapters, five equal tiles, a slider) is gone.

The shape of it: the hero states the role and scale of the work, each project gets a layout sized to its importance, and the App Store and live-product links sit on the landing page as hard proof rather than being buried.

### Colors
```
--ground:     #fbfbfa   page background, text on dark
--ink:        #17171a   primary text, solid buttons
--ink-2:      #2c2c31   project statements
--ink-3:      #46464c   hero eyebrow and lede, drawer secondary
--ink-4:      #55555a   labels, details, footer
--dark-panel: #131316   15GRMS chapter background
--on-dark-1:  #eaeaec   statement on dark
--on-dark-2:  #b8b8c0   label on dark
--on-dark-3:  #a8a8b0   detail on dark
--hair:       rgba(23,23,26,0.1)
```
Ambient gradients, light: `oklch(0.84 0.15 258 / 0.75)`, `oklch(0.88 0.14 72 / 0.72)`, `oklch(0.84 0.13 330 / 0.68)`, `oklch(0.87 0.12 168 / 0.6)`.
Ambient gradients, dark chapter: `oklch(0.62 0.17 260 / 0.6)`, `oklch(0.6 0.15 330 / 0.55)`, `oklch(0.68 0.14 75 / 0.5)`.

Body copy was checked at 4.5:1 or better against actual backgrounds. The glass is translucent over a coloured field, so **re-check contrast if the ambient gradients' lightness changes**.

### Typography
Instrument Serif 400 for display (h1, project titles, card titles); Instrument Sans 400/500/600 for everything else. One Google Fonts link, `display=swap`, both preconnects.

| Role | Size | Line height | Letter spacing |
| --- | --- | --- | --- |
| H1 | `clamp(44px, 6.6vw, 88px)` | 0.95 | -0.024em |
| H2 project | `clamp(34px, 4vw, 52px)` | 1 | -0.015em |
| H3 card | `clamp(26px, 2.4vw, 32px)` | normal | -0.01em |
| Hero lede | `clamp(16px, 1.4vw, 18.5px)` | 1.55 | |
| Statement | `clamp(17px, 1.6vw, 20px)` | 1.4 | |
| Card statement | 16.5px | 1.4 | |
| Detail | 15.5px | 1.62 | |
| Card detail | 14.5px | 1.6 | |
| Section label | 12.5px, 500 | normal | 0.08em uppercase |

### Spacing, radii, layout
One gutter variable: `--pad: clamp(16px, 3vw, 40px)`. Content column `max-width: 1560px`. Section gaps `clamp(28px, 3.5vw, 40px)`; panel padding `clamp(28px, 4vw, 48px)`; card padding `clamp(24px, 2.6vw, 30px)`.

Radii: `clamp(24px, 3vw, 40px)` large panels, `28px` cards and the portrait screenshot, `22px 22px 0 0` bleeding screenshots, `999px` pills and avatar.

### The glass material
Six recipes, as classes: `.glass-panel` (heavy, SundayAtlas), `.pill-id`, `.pill-nav`, `.btn-glass`, `.card-dark` (15GRMS text card), `.card` (project cards). Each is a layered gradient plus `backdrop-filter` plus an inset shadow stack. Exact values are in the handoff README under "Glass recipes"; the file matches them.

Three things are load-bearing:
1. **The material only works over the ambient field.** On a plain white parent it disappears.
2. **`.page` needs `isolation: isolate` and `overflow: hidden`.** `isolation` scopes what `backdrop-filter` samples, `overflow` bounds the negative-inset ambient layers. Neither creates a containing block for `position: fixed`, which is why the drawer still resolves against the viewport from inside.
3. **`backdrop-filter` is on exactly 8 elements, which is the intended ceiling.** Verified: 8 declarations, each paired with `-webkit-`. Adding more surfaces, or nesting glass inside glass beyond the one case in the dark chapter, costs frames on scroll. An `@supports not (...)` block thickens the gradients where `backdrop-filter` is unsupported.

### The ambient field
`.ambient` is `position: fixed; inset: -16%; z-index: 0; pointer-events: none`, four OKLCH radials, `filter: blur(40px) saturate(135%)`, `omAmbient 30s`. Content sits at `z-index: 1`. The dark chapter has its own `.dark-ambient` at `inset: -20%` with three radials and `omAmbient 34s`.

`@keyframes omAmbient` animates **transform only** (translate3d plus scale). **Never animate `background-position`, `filter`, or the gradient stops**, and never drive the gradient centre positions from scroll. See "Scroll performance" below: that mistake cost four rounds once already.

### The pointer specular
One `pointermove` listener on `.page`, `{ passive: true }`, throttled with a single in-flight `requestAnimationFrame`, feeding **one** element: the SundayAtlas panel's `.specular` layer. Percentages are of the page root's box, as the handoff specifies.

Deviation from the handoff, deliberate: `--mx` / `--my` are registered with `@property { inherits: false }` and written **on the specular layer itself**, not on the page root. The maths and the visual are identical; writing on the consumer keeps a pointermove from dirtying the inherited style of every element in the document. The registered initial values (`50%`, `-10%`) are the handoff's resting position, so the panel reads correctly before the first pointer event and on touch devices, which never fire one.

**Do not re-add per-card pointer listeners.** The previous design had a specular on every card and it was removed on request. This is the one element that gets it.

## Page Structure

```
.page  (position:relative; isolation:isolate; overflow:hidden; min-height:100vh)
  .ambient                        fixed light field, z-index 0

  .shell  (z-index 1, max-width 1560px, centred)
    .topbar    identity pill (glass) + nav pills: Work / GitHub / LinkedIn
               LinkedIn is solid ink, the other two are glass
    .hero      eyebrow / h1 / lede / two buttons
               "Download 15GRMS" (solid) and "Open SundayAtlas" (glass)

    main#work
      .sec   01 SundayAtlas   .panel.glass-panel, no bottom padding so the
                              screenshots bleed off the bottom edge.
                              .specular child. Header row: label, serif h2,
                              statement, "Case study" link.
                              .shots: three screenshots, centre one taller.
      .sec   02 15GRMS        .dark-panel on #131316 with .dark-ambient.
                              .dark-grid: .dark-text (card-dark) plus
                              .dark-shots, a pair of portrait screenshots
                              (brew, brewing). App Store button and a
                              "Case study" link.
      .sec   03 to 05         .cards: Signal, JobAgent, Rise as .proj-card.

      .drawer-overlay
      .drawer                 the five case studies (see below)

    footer
```

The case studies sit **inside `<main>` and before `<footer>`** on purpose. With JS off they are the bulk of the page's content, so they must not fall outside the main landmark.

### What the handoff left open, and what was chosen
The handoff explicitly said to decide these with the owner rather than guess. Both were decided on 2026-09-18:

- **Case studies**: the handoff has a "Case study" affordance with no destination designed. The owner chose to **keep the existing slide-in drawer**. Consequences, all deliberate:
  - SundayAtlas's "Case study" opens the drawer rather than linking to `sundayatlas.vercel.app` as the reference does. The hero's "Open SundayAtlas" already carries the live link.
  - The 15GRMS chapter gained a "Case study" link the reference does not have, or that case study would be unreachable.
  - The Rise card opens its case study rather than linking straight out, and its detail line reads "Case study" instead of "View the prototype". The prototype link lives inside that case study.
  - Without those three, three of the five case studies would only be reachable by typing a `#hash`.
- **The SundayAtlas screenshot row at narrow widths**: the owner chose a **scrollable snapping strip** over dropping to a single screenshot.

### Responsive: two fluid handovers, and one width breakpoint
The file has exactly **one** width media query, `@media (min-width: 820px)`, and it exists only to gate the shared panel height (see below). Everything else is fluid, including both narrow-width behaviours the handoff asked about, and that **should stay that way**:

- **`.shots`** is `grid-auto-flow: column` with `grid-auto-columns: minmax(150px, 1fr)` and `overflow-x: auto`. Three-up while each still fits, then a snapping strip. The 150px floor means a screenshot never drops below 150px, which is the legibility problem the handoff flagged. Measured: no scroll at 560 and above (about 153px each at 560), scrolls at 390.
- **`.dark-grid`** is flex-wrap, not the handoff's two-column grid. `.dark-text { flex: 1 1 260px }`, and `.dark-shots` is **`flex: 0 1 auto` with `width: clamp(260px, 43vw, 614px)`**.

  **Sizing the screenshot pair with a width clamp rather than a flex-basis is the whole trick, and it is easy to undo by accident.** `flex-wrap` breaks the line on *base* sizes, so a fixed `flex: 0 1 614px` basis drops the pair below the card the moment 614 plus the card stops fitting, which it did at 1113px. With `flex-basis: auto` the base size **is** the clamp, so the pair narrows and stays alongside the card instead. The owner asked on 2026-09-19 that the screenshots **stay next to the card, not below it**, so do not convert this back to a flex-basis.

  Measured 2026-09-19: beside the card at 820, stacked at 800, so the stack point is **about 810px**, and it only stacks there because the clamp has bottomed out at 260px and the card itself would otherwise be squeezed unreadable. Each screenshot is 300px (full size, matching the single screenshot this replaced) from about 1430px up, 268 at 1280, 213 at 1024, 187 at 900 and 123 at 390.

`.shots` carries 2px of block padding because `overflow-x: auto` computes `overflow-y` to `auto` as well, which would otherwise clip the images' top highlight. It used to pair that with `margin-top: -2px`; that is now `margin-top: auto` (see below), and the 2px simply joins the free space above the row.

### The two project panels are locked to one height
The owner asked on 2026-09-19 for the SundayAtlas panel to match the 15GRMS panel. They were not close: 596 vs 708 at 1440, and the gap *widened* to 296 at 768, because the dark panel was pinned by a fixed-width screenshot while SundayAtlas tracked `vw` through its `clamp()` heights.

Both now take `min-height: var(--panel-min)` and are `display: flex; flex-direction: column`. `--panel-min` is `min(52vw, 708px)`: the dark panel's screenshots are now fluid too, so its own height follows roughly 52vw (measured 708 at 1440, 666 at 1280, 532 at 1024, 468 at 900), and 708 is where the screenshots stop growing at 300px each.

Three consequences that are load-bearing:

- **`.shots` needs `margin-top: auto`.** Without it the spare height lands *below* the screenshots and they stop bleeding off the panel's bottom edge, which is the whole point of that composition.
- **`.dark-grid` needs `flex: 1`** so its `align-items: center` has height to centre the row in.
- **The screenshots grow into the extra height rather than leaving it above them.** Inside the same media query, `.shots .is-tall` takes `calc(var(--panel-min) - var(--panel-chrome))` and its neighbours 82% of that, replacing the base `clamp()` heights. Without this the panel simply got taller and left the space empty: 150px of it at 1440, which the owner flagged on 2026-09-19. The screenshots stay bled off the bottom edge and just show more of each phone, 56% of the frame at 1440 instead of 43%.

`--panel-chrome` is everything stacked above the screenshots: the panel's top padding and the header's bottom margin, both repeated verbatim from their real declarations, plus **126px**. That last number is the header at its tallest (123 at 1440, 108 at 1024, 103 at 900) plus the 2px of top padding `.shots` carries for its images' top highlight. It deliberately **errs high**, because the header shrinks at narrower widths: the screenshots then come out slightly short, leaving 41px of glass at 1440 and 53px at 900, instead of overflowing the panel and breaking the height match. Miss the 2px and the panel lands exactly 1px over 15GRMS, which is how this was found.

Measured **equal to the pixel at 1920, 1440, 1280, 1024 and 900**, with the screenshots at 498, 498, 456, 335 and 279. Two known deviations, both deliberate:

- **Between about 810 and 900 they drift by up to 50px** (426 vs 476 at 820). In that band the dark panel's height is driven by its *text card*, which grows as it narrows, so no single `vw` term tracks it. Raising `--panel-min` to catch it would pour 60 to 170px of dead glass into both panels at 1024 and 1280, which is a bad trade for a 100px-wide band.
- **Below about 810 the floor does not apply at all** (the media query). The screenshots have stacked under the card by then, that panel grows past 770 on its own, and holding SundayAtlas level with it would buy nothing but dead glass. An earlier attempt at exactly that put **216px** of empty glass above the SundayAtlas screenshots at 1000px, which is why the gate exists.

If `--panel-min` is ever changed, re-check that it still clears the SundayAtlas content at 1920, which is the tightest case (607 of content against a 708 floor).

Verified with **no horizontal overflow at 390, 640, 768, 800, 820, 900, 1024, 1200, 1280, 1440 and 1920px**.

## The case-study drawer

Not part of the Glass handoff. Retained on the owner's instruction so the five written case studies stay reachable, restyled to the new palette and glass pills.

### Progressive enhancement (important)
The case studies are **real content in the DOM**, not JS-generated. An inline script in the head adds a `js` class to `<html>`.
- **Without JS**: `.drawer` is a static block at the end of `<main>`, all five case studies are visible, the overlay and Close button are hidden, and every trigger is an ordinary anchor that jumps to its case study.
- **With JS**: the same markup becomes a fixed slide-in drawer; only `.case-study.is-active` is displayed.

Never move case-study content into JavaScript, and never hide it with CSS that is not scoped under `html.js`.

### Behaviour and the details that are easy to regress
Open on trigger click (`preventDefault`, `history.replaceState` to `#id`), lock body scroll, mark the background `inert`, focus Close. Close via Close, overlay click or Escape: animate out, unmount after 500ms, restore scroll, drop `inert`, clear the hash, return focus to the trigger. `#hash` deep-links on load and on `hashchange`.

- The drawer uses a **forced reflow** (`void panel.offsetWidth`) before adding `.is-open`, not `requestAnimationFrame`. rAF does not fire in a hidden or throttled tab, which left deep-linked drawers stuck closed.
- Focus must be set **after** `.is-open` lands: the panel is `visibility: hidden` until then and a hidden element cannot take focus.
- `close()` flips `isOpen` **synchronously, before** restoring focus, or the focus guard bounces focus back into the closing panel.
- `switchTo()` must re-focus Close: the Next link lives inside the article being unmounted.
- The drawer wiring is attached **before** the specular, and the specular is wrapped in try/catch, so a throw in the decorative layer can never leave the case studies unreachable.

The SundayAtlas flow video carries **`data-src` rather than `src`** plus `preload="none"`, hydrated only when that case study opens, so the landing page downloads none of the 1.15MB. A `<noscript>` link keeps it reachable without JS.

### The 15grms id
`document.querySelector('#15grms')` **throws**: a CSS identifier cannot begin with an unescaped digit. The JS resolves case studies with `getElementById` and only ever builds the hash as a string. If you need a selector, scope it off the element or escape it as `#\\31 5grms`.

## Scroll performance

Read this before optimising anything in the ambient or glass layers. It is the most expensive lesson in this repo's history.

The previous design shipped with visible scroll stutter. **The cause was scroll-driven light drift**: two custom properties were rewritten every scroll frame, and they sat inside `radial-gradient()` centre positions on layers carrying `filter: blur(34px)`. Gradient positions are not compositor-animatable, so each write discarded those layers' raster tiles and re-blurred roughly 8.5 Mpx at dpr 2.

Three rounds of fixes were shipped against a wrong diagnosis (`backdrop-filter` re-resolution) and **none made a perceptible difference**. The cause was found by A/B toggling the live page on the owner's own hardware with a console snippet that swapped one `<style>` element between labelled states.

Two rules that follow:
- **Never drive a gradient position, or anything else that lands in paint, from scroll.** The current design has no scroll-driven anything, which is the main reason it is quick. Keep it that way.
- **Measure on the target hardware before spending design fidelity.** The step queued up when the real cause was found was reducing the glass blur, which would have cost real fidelity and fixed nothing. The one analysis that named the true mechanism was overruled by three reviewers who were wrong.

Also assessed and rejected, still true of this design:
- **`will-change` or `translateZ(0)` on `.ambient`**: it already has an infinite transform animation and a filter, so it is composited. No benefit, and it pins tens of MB of backing store.
- **`contain`, `content-visibility`, or `transform` on `.page` or any ancestor of `.drawer`**: all create a containing block for `position: fixed` and would break the drawer.

## Content Rules
- **No em dashes or en dashes** in any content, in any encoding (literal, `&#8212;`, `&mdash;`, `&#8211;`, `&ndash;`). Use commas, semicolons, colons or periods.
- **No mention of "Lovable" by name** in SundayAtlas content. Rise and 15GRMS may mention it.
- If an email is ever shown, encode the `@` as `&#64;` so Cloudflare's email protection cannot mangle it on GitHub Pages. The current design shows no email.
- **Project copy is verbatim from the case studies.** The landing page surfaces each project's strongest existing line as its headline; it invents nothing. The hero headline and lede are the one piece of new copy, written for this redesign and approved by the owner.
- **Regenerate the whole file rather than patching it.** Partial writes have corrupted it before.

## Known Issues / Pending Work
- **`og:image` is still missing**, so shared links render a text-only card. A 1200x630 social image is the remaining SEO task.
- **`<title>` and `og:title` still say "Product Builder"**, kept because the existing head meta and OG tags had to be preserved. The new hero says something sharper; worth revisiting together.
- **Signal has no repo link.** If the repo is made public, add one.
- 15GRMS home screen says "The Adler Original" while the recipe and journal screens say "The Hoffmann Method".
- **Not yet checked in Safari.** All 8 `backdrop-filter` declarations carry `-webkit-`, but the glass has only been verified in the Chromium-based preview.
- The handoff's "About" and "Writing" nav items were explored and are **not** in this design. The nav is Work / GitHub / LinkedIn.

## How to Edit
1. Edit `index.html` (regenerate it whole; see Content Rules)
2. Preview with `python3 -m http.server 4302 --directory .` and open `http://127.0.0.1:4302/`
3. Check 390 / 560 / 800 / 820 / 900 / 1024 / 1440 / 1920px, open a case study, press Escape, and load a `#deep-link` directly. 800 and 820 straddle the 15GRMS stack point and the panel-height gate, so a change to either shows up there first
4. Push to `main` (GitHub Pages auto-deploys)

**Preview gotchas**, both confirmed the hard way:
- When the Browser pane is not displayed, the page composites no frames: screenshots come back blank or one step stale, `requestAnimationFrame` never fires, and CSS transitions do not progress. A drawer that reports `.is-open` but a computed `transform` of `translateX(600px)` is this artifact, not a bug. Confirm by setting `transition: none` and re-reading.
- **Do not fake widths with `document.body.style.width`.** `--pad` and every `clamp()` still resolve against the real viewport, so responsive thresholds measured that way are wrong. Use real `resize_window` calls.
- After editing, navigate with a cache-buster (`?v=2`) and assert something only the new file contains before trusting any result.

## Related Repos
- **15GRMS**, formerly BrewLab: live on the App Store since 17 September 2026, https://apps.apple.com/us/app/15grms/id6811369525 . Repo is `github.com/alxnhfr-bit/brewlab` (`brewlab` is the internal codename; the public name is 15GRMS). Its README is the source of truth for that case study's copy, **except for the two things the owner corrected directly on 2026-09-19**, which the README does not capture and which must not be reverted to it:
  - **The Problem is that recipes are scattered and their instructions are buried in video**, so trying a new one means looking it up, watching it brewed, and scrubbing back and forth to pull out the dose, ratio, temperature and pour timings. It is *not* the earlier framing of a clock and a scale competing for attention.
  - **The visual identity is deliberately not brown.** Brewing apps all reach for the same coffee tones, so this one is high contrast and typographic with three colour themes the user picks. That rationale lives in The Product row and in the craft feature list.
- **SundayAtlas**: deployed on Vercel at `sundayatlas.vercel.app`
- **Design handoffs**: `design_handoff_portfolio_glass/` is the current one. `design_handoff_portfolio_redesign/` is the superseded Liquid Glass chapters design

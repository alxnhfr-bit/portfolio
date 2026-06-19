# Portfolio Website - Claude Code Project Brief

## Repository
- **Repo**: `github.com/alxnhfr-bit/portfolio`
- **Live URL**: `https://alxnhfr-bit.github.io/portfolio/`
- **Hosting**: GitHub Pages (main branch, single `index.html`)
- **File**: Self-contained HTML (~2.5 MB) with 15 base64-embedded screenshots

## Owner
Alexander Neuhofer - Senior PM at Zalando, Berlin. Building AI product prototypes independently. Portfolio targets consumer PM roles in APAC.

## Architecture
Single `index.html` file. No build step, no dependencies, no framework. Pure HTML/CSS/JS with Google Fonts loaded via CDN. All 15 screenshots are base64-encoded JPEG inline in `<img>` tags. Deploy by pushing `index.html` to the repo.

## Design System

### Style
Apple product page aesthetic - full-viewport sections, alternating dark/light backgrounds, sticky frosted-glass nav, scroll-triggered animations.

### Fonts
- **Headings**: Inter Tight (weights 400-700)
- **Body**: DM Sans (weights 400-500)
- Loaded from Google Fonts CDN

### Colors (CSS custom properties)
```
--white: #fbfbfd
--black: #1d1d1f
--gray-bg: #f5f5f7
--dark-bg: #000
--blue: #2997ff
--text-light: #86868b
--text-dark-muted: #a1a1a6
```

### Layout
- Max content width: 760px (`.section-inner`)
- Nav max width: 1024px
- Section padding: 120px vertical, 24px horizontal
- Phone gallery: horizontally scrollable flex container

### iPhone Mockups
CSS-only frames with:
- Dynamic Island (pseudo-element)
- Titanium gradient finish (dark and light variants)
- Side buttons (power + volume via `::before` / `::after`)
- Screen glare overlay
- 5px padding around screen
- Border radius: 36px outer, 32px screen
- Width: 230px (desktop), 190px (tablet), 160px (mobile)

### Animations
- **Hero**: staggered fadeIn (0.2s - 1.2s delays)
- **Phone galleries**: `scaleReveal` animation triggered by IntersectionObserver (threshold 0.15), staggered 0.12s per phone
- **Story steps**: CSS transition on `.visible` class (opacity + translateY)
- **Scroll hint**: float animation, fades on scroll > 100px

### Dark Mode Override
Forced light mode via `<meta name="color-scheme" content="light only">` and `@media (prefers-color-scheme: dark)` overrides to prevent OS dark mode from breaking the design.

## Page Structure

```
Nav (fixed, frosted glass, toggles dark class over dark sections)
Hero
  - "Alexander Neuhofer."
  - "Product Builder"
  - Tagline
  - LinkedIn CTA
  - About line
  - Scroll hint chevron

Section 1: Rise (dark bg, id="rise")
  - 01 / Rise / AI Health & Fitness OS
  - 5 phones: Dashboard, Training, AI Coach, Wellbeing, Supplements
  - Story: Problem > Insight > Prototype > Takeaway
  - Tool credit: "Built with Lovable."
  - Link: https://rise-health-os.lovable.app/

Section 2: SundayAtlas (light bg, id="sundayatlas")
  - 02 / SundayAtlas / AI-Powered Travel Planning
  - 5 phones: Home, Destinations, AI Concierge, Itinerary, Trip Dashboard
  - Story: Problem > Insight > Prototype > Takeaway
  - Tool credit: "Built with React, TypeScript, and the Anthropic API."
  - Link: https://sundayatlas.vercel.app/

Section 3: BrewLab (dark bg, id="brewlab")
  - 03 / BrewLab / AI Coffee Brewing Assistant
  - 5 phones: Landing, Recipes, Ratio, Timer, Shop
  - Story: Problem > Insight > Prototype > Takeaway
  - Tool credit: "Built with Claude; landing animation crafted with Kling AI."
  - Link: https://alxnhfr-bit.github.io/brewlab/

Footer
  - alxnhfr@gmail.com (encoded as &#64; to prevent Cloudflare email obfuscation)
  - LinkedIn link
  - "Alexander Neuhofer - 2025"
```

## Content Rules
- **No em dashes** in any content. Use commas, semicolons, or periods instead.
- **No mention of "Lovable" by name** in SundayAtlas content (it was rebuilt with React/TypeScript/Claude Code).
- Rise and BrewLab can mention Lovable.
- Email `@` symbol must be HTML-encoded as `&#64;` to prevent Cloudflare email protection from mangling it on GitHub Pages.

## Known Issues / Pending Work
- **Phone height inconsistency**: SundayAtlas screenshots have a different aspect ratio than Rise/BrewLab, making those phone mockups taller. Fix: change `.iphone-screen img` from `height: auto` to `height: 480px; object-fit: cover; object-position: top;` (value adjustable).
- **Year in footer**: Currently says 2025, may need updating.

## Responsive Breakpoints
- **> 900px**: Full desktop (230px phones)
- **601-900px**: Tablet (190px phones, tighter gallery gaps)
- **<= 600px**: Mobile (160px phones, reduced padding, smaller nav gaps)

## How to Edit
Since all images are base64-embedded, the file is ~2.5 MB. Recommended workflow:
1. Clone the repo locally
2. Edit `index.html`
3. Test by opening in browser
4. Push to main branch (GitHub Pages auto-deploys)

To replace a screenshot: convert the new image to base64 (`base64 -w0 image.jpeg`), find the corresponding `<img src="data:image/jpeg;base64,...">` tag, and replace the base64 string. Images are in order: Rise (5), SundayAtlas (5), BrewLab (5).

## Related Repos
- **BrewLab prototype**: `github.com/alxnhfr-bit/brewlab` (standalone HTML with React via CDN)
- **SundayAtlas**: deployed on Vercel at `sundayatlas.vercel.app`

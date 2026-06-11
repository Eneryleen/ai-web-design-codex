# Breakpoints, Devices & Mobile-First

> How to choose breakpoints from content (not device models), apply mobile-first vs desktop-first deliberately, and reason about the real device landscape in CSS pixels. The senior outcome: layouts that hold across hundreds of viewport widths with the minimum number of breakpoints and zero device-specific hacks.

**Apply when:** designing any responsive layout, choosing breakpoints, or deciding mobile-first vs desktop-first. · **Related:** [Responsive & Fluid Design](./responsive-and-fluid-design.md) · [Fluid Typography & Spacing with clamp()](./fluid-type-and-spacing.md) · [Touch Ergonomics & Cross-Device Design](./touch-ergonomics-cross-device.md) · [Layout, Grids & Spacing Systems](../00-foundations/layout-grids-and-spacing.md)

## TL;DR — rules at a glance
- **Breakpoints come from content, not devices.** Resize the viewport from 320px upward; the exact px where the layout first looks wrong is your candidate breakpoint. Never pick a number because a phone model lands there.
- **Mobile-first = base CSS for small screens (no media query) + `min-width` queries to add complexity upward.** Progressive enhancement, not graceful degradation. Yields less total CSS than desktop-first.
- **Design in CSS (logical) pixels, never physical pixels.** iPhone 16 Pro = 402px CSS at DPR 3 — not its 1206px physical width. Media queries respond to CSS px only.
- **The viewport meta tag is mandatory:** `<meta name="viewport" content="width=device-width, initial-scale=1">`. Without it mobile assumes a ~980px canvas and no breakpoint fires.
- **3–5 major breakpoints cover almost everything:** ~360px (mobile base, no query), ~768px (tablet), ~1024–1280px (desktop). Add more only when content demands it.
- **320px is the absolute floor** — not 375px. Older iPhones, Android Go phones, and a user at 400% browser zoom on a 1280px monitor all produce a 320px CSS viewport.
- **Mobile-first is not universal.** SaaS dashboards, data-dense enterprise tools, and any product with >70% desktop traffic should be designed **desktop-first**. Match methodology to actual audience analytics.
- **Container queries (`@container`) for components, media queries for the page.** Media decides macro layout (is there a sidebar?); container decides micro layout (card in a narrow column vs wide). Baseline since 2023.
- **Fluid first, breakpoints second.** `clamp()`, `min()`, `max()`, `vw/dvh`, and intrinsic Grid (`auto-fit`/`minmax`) eliminate most breakpoints. Add a media query only when fluid sizing alone can't express the change.
- **Use `dvh`/`svh`, not `vh`, for full-screen mobile sections.** `100vh` includes the iOS Safari URL bar and overflows.
- **Never set `user-scalable=no` or `maximum-scale=1`.** Breaks accessibility zoom, fails WCAG 1.4.4. Fix the design instead.
- **Test orientation separately from width.** A landscape phone matches your tablet breakpoint but has phone ergonomics; use `@media (orientation: landscape)` for nav/video/modals.
- **Use `em` units in media queries, not `px`.** `48em` instead of `768px` — `em` in a media query context scales with the browser's root font size, so user-set text-zoom still fires the right breakpoint. (`rem` in media queries has historically been buggy in some browsers; prefer plain `em`.)
- **Use `(hover: hover) and (pointer: fine)` to gate hover/mouse UI patterns.** Touch devices report `(hover: none) and (pointer: coarse)`.
- **Centralize breakpoint values** (CSS custom properties / SCSS vars). Never scatter the same px across dozens of rules.

## Breakpoint strategy: content-driven vs device-driven

**Device-driven (wrong, legacy):** pick breakpoints at known device widths — 375px "iPhone", 768px "iPad", 1024px "laptop". Fragile because new models ship constantly and the phone landscape has hundreds of unique widths. Apple alone spans **320, 375, 390, 393, 402, 430, 440px** CSS across current/recent iPhones. A 375px breakpoint built for "the iPhone" is wrong the moment a 390px model ships, and a 430px upper bound is now wrong because iPhone 16 Pro Max is 440px.

**Content-driven (correct, modern):**
1. Start at 320px in DevTools responsive mode.
2. Slowly drag the viewport wider.
3. The first width where the layout looks awkward (line length too long, cramped columns, orphaned elements, broken nav) is the candidate breakpoint.
4. Add a breakpoint there. Repeat until 1440px+.
5. Do **not** round to a "nice" number for its own sake; if the design breaks at 734px, 734px is correct (though you may snap to a token for maintainability).

**Why device-targeting fails — the fragmentation math:** StatCounter data consistently shows no single CSS viewport width exceeding ~15% of mobile traffic, with long-tail fragmentation across hundreds of widths. You cannot enumerate devices; you can only respond to content.

```css
/* WRONG — device-driven, brittle */
@media (max-width: 375px) { /* "iPhone" */ }
@media (min-width: 768px) and (max-width: 1024px) { /* "iPad" */ }

/* RIGHT — content-driven, mobile-first, modern range syntax */
/* base = small screen, no query */
.cards { display: grid; gap: 1rem; }
@media (width >= 46em) { .cards { grid-template-columns: 1fr 1fr; } }   /* breaks here */
@media (width >= 75em) { .cards { grid-template-columns: repeat(3, 1fr); } }
```

## Mobile-first methodology & rationale

**Definition:** write base CSS for the smallest target with **no media query**, then layer `min-width` queries to progressively enhance for wider viewports.

```css
/* Base: mobile. Single column, stacked. */
.layout { display: grid; grid-template-columns: 1fr; gap: 1rem; }
.nav { display: none; }            /* hamburger handles small screens */
.menu-toggle { display: block; }

/* Enhance upward */
@media (width >= 48em) {           /* tablet: layout starts to breathe */
  .layout { grid-template-columns: 200px 1fr; }
  .nav { display: flex; }
  .menu-toggle { display: none; }
}
@media (width >= 80em) {           /* desktop: third column appears */
  .layout { grid-template-columns: 220px 1fr 300px; }
}
```

**Why mobile-first wins (when the audience is mobile-majority):**
- **Less total CSS.** Simple states need fewer property overrides. Desktop-first ships the rich layout to every device, then overrides it down — more rules, more specificity churn, larger payload on the devices least able to afford it.
- **Performance lands where it matters.** Mobile devices are slower and on worse networks; mobile-first ships them the lean base.
- **Forces prioritization.** A 360px column makes you decide what's essential — that clarity carries up to desktop.
- **Progressive enhancement, not graceful degradation.** Start from the resilient core and add capability, rather than start rich and hope it survives subtraction.

**`min-width` vs `max-width`:**
- `min-width` (mobile-first): start simple, add complexity upward. **Default choice.**
- `max-width` (desktop-first): start rich, subtract downward. Reserve for scoping a style *below* a ceiling (e.g., a full-bleed hero only on mobile).
- **Write styles that work everywhere first, then scope only the *deviations* behind `min-width` or `max-width`.** Minimizes override chains and specificity conflicts — especially with `:not()` and pseudo-class selectors.

**Use `em` units in media queries, not `px`:**
```css
/* WRONG — px ignores the user's browser font-size setting */
@media (min-width: 768px) { … }

/* RIGHT — 48em × browser-base-font-size fires at the right layout shift even after text-zoom */
@media (width >= 48em) { … }
```
`1em` in a media query always equals the browser's *root* font size (typically 16px, but user-adjustable). A user who sets 20px base gets the layout shift at 960px — exactly right, because their text needs that extra room.

### Input capability queries — beyond width
Senior responsive work distinguishes *viewport size* from *input capability*:
```css
/* Touch device: no reliable hover, coarse pointer. Gate any hover-dependent UI. */
@media (hover: hover) and (pointer: fine) {
  .tooltip-trigger:hover .tooltip { display: block; }
  .drag-handle { display: flex; }        /* irrelevant on touch */
}

/* Touch only: coarse pointer, enlarge interactive targets */
@media (hover: none) and (pointer: coarse) {
  .custom-select { min-height: 48px; }  /* finger-reachable */
}
```
A tablet in pointer-pen mode reports `(hover: none) and (pointer: fine)`. A hybrid laptop/tablet (Surface, iPad+keyboard) can transition — design for the least-capable state as the base.

### When mobile-first is the wrong default
Mobile-first is a tool, not dogma. Design **desktop-first** when analytics justify it:
- **SaaS admin dashboards, CRMs, data-dense enterprise tools** — power users live on desktop; the core UX is wide tables, multi-pane views, dense controls.
- **Financial / analytics / BI tools** — the value is in seeing a lot at once.
- **Any product where analytics show >70% desktop traffic.**

Forcing mobile-first here increases dev time for no UX gain and tends to produce stripped, unusable mobile views. Decide from real audience behavior, not ideology. See [CRM & Admin Dashboards](../03-site-types/crm-admin-dashboards.md).

## The real device landscape (in CSS pixels)

**Mobile CSS viewport widths, 2026 (logical px):**
- **360px** — most common Android mid-range globally (budget/mid-range Android dominates by volume).
- **375px** — iPhone SE (3rd gen) / iPhone 13 mini era; declining share.
- **390–393px** — iPhone 15/16 base, Pixel 9; high share among iOS users.
- **402px** — iPhone 16 Pro.
- **412px** — Galaxy S25, S25+; many mid-range Androids.
- **430px** — iPhone 15/16 Plus.
- **440px** — iPhone 16 Pro Max. First iPhone to exceed 430px CSS — previously a safe upper bound for "large phone", now 440px is in play.
- **~384px** — Galaxy S25 Ultra (despite its physical 1440px screen, CSS viewport is ~384px at DPR ~3.75).
- The **360–440px band covers the vast majority of mobile traffic.** Design the mobile base to be fluid across this whole band, not pinned to one width.

**Tablet:** iPad standard is 768–1024px CSS (portrait/landscape). Tablet is a small share of total global web traffic (commonly cited <5%, varies significantly by region and product category — verify with your own analytics). Don't over-invest in tablet-specific layouts; a good fluid design between mobile and desktop usually covers it.

**Desktop resolutions, 2026 (approximate market share, source: StatCounter):**
- **1920×1080** — largest single bucket, roughly 20–25% of desktop.
- **1366×768** — legacy; still meaningful globally (~6–9%).
- **1536×864** — scaled Windows default on 1920×1080 hardware; ~7–9%.
- **1440×900** — ~3–4%.
- **2560×1440** and wider ultrawides — growing; requires a capped container.
- Designing for 1920×1080 while ensuring 1366×768 works covers the majority of desktop. The *content container* should cap around 1200–1440px (below) regardless.

**Traffic split (approximate, varies by product category):** mobile >55–60%, desktop ~35–40%, tablet ~4–5%. Source: StatCounter GlobalStats. This *informs* — does not *dictate* — your methodology choice. Check your own analytics first.

### Device pixel ratio (DPR)
Physical pixels ÷ CSS pixels = DPR. **Media queries see CSS px; DPR only matters for image sharpness.**

| Device | Physical width | CSS viewport | DPR |
|---|---|---|---|
| iPhone 16 Pro Max | 1320px | 440px | 3 |
| iPhone 16 Pro | 1206px | 402px | 3 |
| iPhone 16 / 15 Pro | 1179px | 393px | 3 |
| iPhone 16 Plus / 15 Plus | 1290px | 430px | 3 |
| iPhone 13/14 | 1170px | 390px | 3 |
| iPhone SE (3rd gen) | 750px | 375px | 2 |
| Galaxy S25 Ultra | 1440px | ~384px | ~3.75 |
| Galaxy S25 / S25+ | 1080px | 412px | ~2.625 |
| Pixel 9 | 1080px | 412px | 2.625 |
| Pixel 9 Pro | 1344px | 448px | 3 |
| Retina MacBook (14"/16") | — | 1512 / 1728px | 2 |
| Standard 1080p monitor | 1920px | 1920px | 1 |

**Consequence:** anyone saying "1440px phone" (Galaxy S25 Ultra) or "1080px phone" is describing physical pixels — irrelevant to layout. The S25 Ultra's 1440px physical screen reports a **~384px CSS viewport.** Always design in CSS px.

**Serve resolution-appropriate images** so they aren't blurry on high-DPR screens. A 200×200 CSS image on a DPR 3 device needs a 600×600 physical source:
```html
<img src="card.jpg" srcset="card.jpg 1x, card@2x.jpg 2x, card@3x.jpg 3x"
     width="200" height="200" alt="…">
<!-- or responsive width-based -->
<img src="hero-800.jpg"
     srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1600.jpg 1600w"
     sizes="(width >= 48em) 50vw, 100vw" alt="…">
```
```css
.avatar { background-image: image-set("a.png" 1x, "a@2x.png" 2x, "a@3x.png" 3x); }
```

### Foldables (treat as progressive enhancement)
Foldables are a small but growing segment (Galaxy Z Fold, Z Flip, Surface Duo, Pixel Fold). Never build *for* them exclusively, but don't break *on* them.
- **Galaxy Z Fold 6 (unfolded):** ~882px CSS wide; folded ~373px.
- **Galaxy Z Flip 6 (open):** ~390px CSS wide.
- **Pixel Fold (open):** ~841px CSS wide.

**⚠ API status (2026):** The foldable CSS detection APIs have a turbulent history. The original `horizontal-viewport-segments` / `vertical-viewport-segments` media features and `env(viewport-segment-*)` were Samsung-authored drafts that the CSS WG did not adopt in that form. As of 2026, browser support is limited and inconsistent — check MDN for current status before shipping any fold-aware code.

The more broadly supported alternative is the **Device Posture API** (JavaScript) and the CSS `@media (device-posture: folded)` feature, which has experimental support in Chromium. Treat anything here as an enhancement layer, not a load-bearing feature.

What you *can* rely on today: foldables have **two distinct viewport widths** (folded vs unfolded). Your content-driven breakpoints will already handle both if the layout is fluid. The extra concern is the hinge zone — avoid placing interactive elements near the center-horizontal on landscape foldables:
```css
/* Use safe-area insets where supported; otherwise ensure center content has ≥32px padding */
.fold-safe { padding-inline: max(1rem, env(safe-area-inset-left)); }
```
- **Rule:** keep critical content away from the horizontal center on wide foldables. A foldable that fires no posture API must still get a working layout from your normal breakpoints.

## Orientation handling
Orientation is independent of width and must be tested separately. A phone in **landscape** (e.g., iPhone 15 Pro landscape ≈ 852px CSS wide) **matches your tablet breakpoint** but has phone ergonomics, less vertical space, and OS chrome.

```css
@media (orientation: landscape) and (max-height: 30em) {
  /* short landscape phone: collapse tall nav, shrink hero, prioritize the fold */
  .hero { min-height: 100svh; }
  .site-header { position: static; }
}
```
Use orientation queries for **navigation menus, video/iframe embeds, modals, and any tall hero** that assumes portrait height. For embeds, `aspect-ratio` queries help full-screen video:
```css
@media (aspect-ratio >= 16/9) { .video-stage { /* letterbox vs pillarbox handling */ } }
```

## Media queries vs container queries
- **Media queries → macro / page-level decisions:** is the sidebar shown? how many top-level nav items? overall page columns? Responds to the **viewport**.
- **Container queries → micro / component-level decisions:** a card renders compact in a narrow sidebar but expanded in the wide main column — *regardless* of viewport. Responds to the **container**.

```css
/* The component declares its container as a query context */
.card-region { container-type: inline-size; container-name: card-region; }

/* Card adapts to its container, not the page */
@container card-region (width >= 30rem) {
  .card { grid-template-columns: 120px 1fr; }  /* image beside text when there's room */
}
```
Container queries are **baseline-supported since 2023** (Chrome 105+, Safari 16+, Firefox 110+). Use both: media for the shell, container for reusable components that appear in differently-sized slots.

## Fluid-first: fewer breakpoints by design
Many "responsive problems" disappear without any media query:

```css
/* Intrinsic grid: as many columns as fit, each ≥280px, all flexible. Zero breakpoints. */
.grid { display: grid; gap: 1rem; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); }

/* Fluid type: smooth scaling between ~320px and ~960px, no font-size breakpoints */
h1 { font-size: clamp(1.75rem, 1.2rem + 3vw, 3rem); }
p  { font-size: clamp(1rem, 0.95rem + 0.3vw, 1.125rem); }

/* Fluid container: never exceed readable line length, fluid below */
.container { width: min(100% - 2rem, 1200px); margin-inline: auto; }
```
`clamp(min, preferred, max)`, `min()`, `max()`, `vw/dvh`, and Flexbox `wrap` express continuous change that breakpoints can only approximate in steps. **Utopia (utopia.fyi)** systematizes fluid type/space scales from one small and one large breakpoint, letting every width between interpolate smoothly — fewer hard breakpoints, no awkward jumps. Reach for a media query only when the change is *discontinuous* (an element appears/disappears, or layout restructures). See [Fluid Typography & Spacing with clamp()](./fluid-type-and-spacing.md).

## Viewport meta tag — required and non-negotiable
```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```
- **Without it:** mobile browsers assume a ~980px canvas and render the desktop layout zoomed out; **no breakpoint fires** and all responsive work is defeated.
- **Never** add `user-scalable=no` or `maximum-scale=1` — blocks pinch-zoom, fails WCAG 1.4.4 / 1.4.10, frustrates low-vision users. If zooming breaks the layout, the layout is broken — fix it.
- **Never** remove the tag "to test desktop on mobile." Use DevTools for that.

## Testing across devices
**DevTools emulation is necessary but not sufficient.** It does **not** replicate: iOS Safari URL-bar shrink/grow, safe-area insets on notched / Dynamic Island phones, real touch behavior, system font rendering, or realistic slow networks.

Tiered approach:
1. **DevTools responsive mode** — drag-resize to find content breakpoints; toggle DPR, touch, throttling, orientation.
2. **Multi-viewport mirror (Responsively App)** — see many sizes at once during build; mirrors clicks/scroll/nav across all panes.
3. **Real-device cloud (BrowserStack / LambdaTest)** — actual physical iOS/Android for final QA and visual regression.
4. **At least one real physical phone** on a real network before shipping — emulators lie about the things that matter most.

Use `env(safe-area-inset-*)` for notched/Dynamic-Island devices:
```css
.site-footer { padding-bottom: max(1rem, env(safe-area-inset-bottom)); }
```

## Concrete specs & numbers
- **Mobile band that matters:** 360–440px CSS. Floor = **320px** (Android Go, older iPhones, 400% browser zoom). Common: 360, 375, 390/393, 402, 412/430, 440.
- **Major breakpoint starter set:** ~360 (base, no query) · ~768 (tablet) · ~1024–1280 (desktop). 3–5 total is enough for most sites.
- **Media query units: use `em`.** `48em` ≈ 768px at default font size but correctly shifts at other font sizes. `px` in media queries ignores browser text-zoom settings.
- **Content container max-width:** **1200–1440px**, centered (`margin-inline: auto`) — keeps line length **65–75ch** (optimal for reading); 80–85ch is the practical max before readability degrades. Prevents unreadable lines on 2560×1440 / 3440×1440 ultrawides.
- **Touch targets:** **44×44 CSS px** (Apple HIG) · **48×48 dp** (Material / WCAG 2.5.5 AAA) · **≥24×24 CSS px** floor (WCAG 2.5.8 AA, WCAG 2.2). See [Touch Ergonomics](./touch-ergonomics-cross-device.md).
- **DPR:** iPhone 16 Pro/15 Pro/13/14 = 3 · iPhone 16 Pro Max = 3 · iPhone SE = 2 · Galaxy S25 Ultra ≈ 3.75 · Galaxy S25 ≈ 2.625 · Pixel 9 = 2.625 · Pixel 9 Pro = 3 · Retina MacBook = 2 · 1080p monitor = 1.
- **Foldable widths (2026):** Galaxy Z Fold 6 unfolded ~882px · Galaxy Z Flip 6 open ~390px · Pixel Fold open ~841px.
- **Framework defaults (reference, not gospel):**
  - **Bootstrap 5:** 576 / 768 / 992 / 1200 / 1400px.
  - **Tailwind:** `sm` 640 · `md` 768 · `lg` 1024 · `xl` 1280 · `2xl` 1536px.
  - **Material Design 3:** 600 / 840 / 1240 dp.
- **Container query baseline:** Chrome 105+ / Safari 16+ / Firefox 110+ (2023).
- **Modern range syntax:** `@media (width >= 768px)` preferred over `@media (min-width: 768px)` in new code (widely supported post-2023).

### Centralize breakpoints
Your project breakpoints should come from content analysis, not a framework template. The example below reflects this guide's starter set, *not* a fixed prescription:
```scss
// SCSS — single source of truth; generate media blocks from these vars
$bp-md: 48em;   // ~768px — tablet / layout starts expanding
$bp-lg: 64em;   // ~1024px — desktop
$bp-xl: 90em;   // ~1440px — ultrawide container cap

// Usage:
@media (width >= #{$bp-md}) { … }
```
```css
/* CSS custom properties CANNOT be read inside media query conditions —
   this is a known limitation; keep tokens in a preprocessor or design token layer.
   @media (width >= var(--bp-md)) {}  ← does NOT work */
```
Do NOT blindly copy Bootstrap's 576/768/992/1200/1400px or Tailwind's 640/768/1024/1280/1536px as your breakpoints — those are framework defaults tuned for their grid systems, not content-driven values for your project. Use them as a sanity reference only.

## Tools & libraries
- **Chrome DevTools Device Mode** — responsive resize, DPR/touch emulation, network throttling, orientation toggle. First stop for finding content breakpoints.
- **Responsively App (responsively.app)** — free/open-source dev browser; mirrors interactions across many viewports simultaneously; save device sets as Preview Suites. Best during active development.
- **BrowserStack Live** — 3,500+ real physical devices for final cross-device QA; Percy add-on for visual regression. Use for sign-off, not daily dev.
- **LambdaTest** — real-device cloud alternative to BrowserStack; competitive pricing.
- **Utopia (utopia.fyi)** — generates fluid type/space scales between two breakpoints; use to reduce hard breakpoints.
- **When NOT to use:** don't rely on DevTools emulation alone for shipping — it misses URL-bar behavior, safe-area insets, real touch, and true network conditions. Don't reach for a JS resize listener when CSS media/container queries do the job.

## Do / Don't
**Do**
- Set breakpoints where *content* breaks; verify by dragging the viewport from 320px up.
- Default to mobile-first `min-width` — unless analytics say the audience is desktop-majority.
- Write media queries in `em` units, not `px`, so text-zoom produces correct layout shifts.
- Design in CSS pixels; serve `srcset`/`image-set()` for DPR.
- Ship the viewport meta tag with `width=device-width, initial-scale=1` and nothing that blocks zoom.
- Prefer fluid sizing (`clamp`, intrinsic Grid) and add breakpoints only for discontinuous changes.
- Use `100dvh`/`100svh` for full-screen mobile sections; `env(safe-area-inset-*)` near edges.
- Gate hover/mouse-only UI behind `@media (hover: hover) and (pointer: fine)`.
- Test orientation explicitly and on at least one real device.
- Keep a single source of truth for breakpoint values (SCSS vars or design tokens, not CSS custom properties in media conditions).

**Don't**
- Don't target specific phone models (375px "for iPhone", 393px "for Pixel", etc.).
- Don't quote physical resolutions ("1440px Galaxy S25 Ultra") for layout decisions — that's physical, not CSS.
- Don't take a finished desktop layout and bolt on `max-width` queries to squash it ("mobile-last").
- Don't set `user-scalable=no` / `maximum-scale=1`.
- Don't `display:none` important content on mobile while leaving it in the DOM to download.
- Don't use `100vh` for mobile full-screen sections.
- Don't drive layout from JS `window.innerWidth` when CSS can do it.
- Don't apply mobile-first dogmatically to desktop-dominant dashboards.
- Don't copy framework breakpoints (Bootstrap / Tailwind) as your project's breakpoints — derive from content.

## Common pitfalls & how to avoid them
- **Mobile-last masquerading as responsive.** Fully-specified desktop CSS + `max-width` overrides = bloated payload, specificity wars, slow mobile. → Write the basics first; enhance with `min-width`.
- **Device-model breakpoints.** A 375px breakpoint built for "the iPhone" breaks the day a 390px model ships. → Content-driven breakpoints only.
- **Confusing viewport with screen resolution.** Galaxy S25 Ultra = 1440px physical but ~384px CSS. Pixel 9 = 1080px physical but 412px CSS. → Always design in CSS px; use DPR only for image selection.
- **The 1px gap bug.** `@media (max-width: 600px)` and `@media (min-width: 600px)` both *exclude* exactly 600px. → Use `max-width: 599.98px` or, better, range syntax: `@media (width < 600px)` / `@media (width >= 600px)`.
- **Disabling zoom.** `user-scalable=no` fails WCAG 1.4.4. → Remove it; fix the layout so zoom works.
- **Hidden-but-downloaded content.** `display:none` still ships bytes and exposes content to screen readers, hurting perf/SEO/a11y. → Conditionally render server-side or lazy-load, don't just hide.
- **JS-driven layout.** Width-detection + class toggling causes layout shift, fires after paint, and is fragile. → Media/container queries are synchronous and layout-safe.
- **Emulator-only testing.** Misses iOS URL-bar resize, safe-area insets, real touch, true networks. → Validate on real hardware before shipping.
- **Unbounded ultrawide.** No `max-width` container → 200-char line lengths on 34" monitors. → `width: min(100% - 2rem, 1440px); margin-inline: auto`.
- **`100vh` overflow on iOS.** URL bar height included → bottom content cut off. → `100svh`/`100dvh`.
- **Breakpointing problems intrinsic layout solves.** Reaching for media queries when `grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))` needs none. → Try intrinsic sizing first.
- **Mobile-first on desktop-dominant tools.** Stripped, unusable mobile dashboards + wasted effort. → Choose methodology from analytics.
- **`px` in media queries breaks text-zoom.** A user who increases browser font size from 16px to 20px needs the layout to shift at a wider px value — `px` won't adapt, `em` will. → Use `em` in all `@media` conditions.
- **Hover UI shipped to touch devices.** Tooltips, dropdown-on-hover, custom drag handles — all broken on touch. → Gate behind `@media (hover: hover) and (pointer: fine)`.
- **Copying framework breakpoints without content analysis.** Bootstrap's 576/768/992px and Tailwind's 640/768/1024px are tuned for their grid utilities, not your content. → Always verify against your actual content; adjust where needed.

## What real users complain about
- **"The bottom button is cut off / I can't tap submit"** — classic `100vh` + iOS URL bar overlap. Fix with `dvh`/`svh` and safe-area insets.
- **"I can't zoom in to read it"** — `user-scalable=no`; a top accessibility complaint from low-vision and older users.
- **"Site loads the full desktop version, tiny and unusable"** — missing/broken viewport meta tag → ~980px assumed canvas.
- **"Buttons are too small / I keep mis-tapping"** — touch targets below 44px; adjacent targets too close (<24px spacing).
- **"Text stretches across the whole giant monitor"** — no max-width container on ultrawide; unreadable line lengths.
- **"Looks fine in portrait, breaks when I rotate"** — width-only thinking; landscape phone hit the tablet breakpoint with no orientation handling.
- **"Notch/Dynamic Island covers the header"** — missing `env(safe-area-inset-*)` padding.
- **"Fast on Wi-Fi, painful on mobile data"** — desktop-first override bloat shipped to the slowest devices.

## Senior-level checklist
- [ ] Methodology chosen from analytics: mobile-first unless audience is desktop-majority (>70%) — decision documented.
- [ ] Viewport meta present: `width=device-width, initial-scale=1`; no `user-scalable=no` / `maximum-scale`.
- [ ] Breakpoints derived by dragging 320px → 1440px+; each one sits where content breaks, not a device width.
- [ ] 3–5 major breakpoints; intermediate/ultrawide added only where content requires.
- [ ] Layout verified at the 320px floor and across the 360–440px mobile band.
- [ ] Media query units are `em`, not `px` — text-zoom produces correct layout transitions.
- [ ] Fluid sizing (`clamp`/`min`/`max`/intrinsic Grid) used first; breakpoints reserved for discontinuous changes.
- [ ] Content container capped at 1200–1440px and centered; line length 65–75ch (max 85ch) on ultrawide.
- [ ] All breakpoint values centralized (SCSS vars or design tokens), not scattered; not in CSS custom properties used inside `@media` conditions.
- [ ] No 1px gap bug — boundaries use range syntax or `.98px` offsets.
- [ ] Container queries used for reusable components that live in variable-width slots.
- [ ] DPR handled: `srcset`/`sizes` or `image-set()` for @2x/@3x; no blurry assets on DPR 3–3.75.
- [ ] Full-screen sections use `dvh`/`svh`; edge content respects `env(safe-area-inset-*)`.
- [ ] Hover/mouse-only interactions (tooltips, hover dropdowns, drag handles) gated behind `(hover: hover) and (pointer: fine)`.
- [ ] Orientation handled where it matters (nav, video, modals, tall heroes).
- [ ] Foldable posture handled as graceful enhancement; nothing critical in the horizontal center on wide foldables.
- [ ] No important content `display:none`-and-still-downloaded on mobile.
- [ ] Tested in DevTools, a multi-viewport mirror, and on ≥1 real device on a real network.

## Sources
- WCAG 2.2 — https://www.w3.org/TR/WCAG22/
- MDN, viewport meta tag — https://developer.mozilla.org/en-US/docs/Web/HTML/Viewport_meta_tag
- MDN, container queries — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries
- MDN, media query range syntax — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries
- MDN, using media queries (em vs px) — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries#syntax_improvements_in_level_4
- MDN, hover / pointer media features — https://developer.mozilla.org/en-US/docs/Web/CSS/@media/hover
- MDN, Device Posture API (foldables, experimental) — https://developer.mozilla.org/en-US/docs/Web/API/Device_Posture_API
- StatCounter GlobalStats (screen/browser stats) — https://gs.statcounter.com/
- Utopia (fluid type & space) — https://utopia.fyi/
- Apple Human Interface Guidelines — https://developer.apple.com/design/human-interface-guidelines/
- Material Design 3 (layout & breakpoints) — https://m3.material.io/foundations/layout/applying-layout
- Tailwind CSS responsive design — https://tailwindcss.com/docs/responsive-design
- Bootstrap 5 breakpoints — https://getbootstrap.com/docs/5.3/layout/breakpoints/
- Responsively App — https://responsively.app/
- BrowserStack Live — https://www.browserstack.com/live
- LambdaTest — https://www.lambdatest.com/

# Responsive & Fluid Design

> The discipline of building layouts that adapt to *any* viewport and *any* container, driven by content needs and user preferences rather than fixed device widths. Master this to ship interfaces that read well from a 320px phone to a 4K monitor, respect user zoom/font-size settings, and use modern CSS (container queries, `clamp()`, intrinsic Grid, dynamic viewport units) instead of breakpoint sprawl.

**Apply when:** building any production web layout, page, or reusable component. · **Related:** [Breakpoints, Devices & Mobile-First](breakpoints-devices-mobile-first.md) · [Fluid Typography & Spacing with clamp()](fluid-type-and-spacing.md) · [Touch Ergonomics & Cross-Device Design](touch-ergonomics-cross-device.md) · [Layout, Grids & Spacing Systems](../00-foundations/layout-grids-and-spacing.md)

## TL;DR — rules at a glance
- HTML is fluid by default. CSS should *preserve* that fluidity, not fight it. Fixed-pixel layouts break it.
- Always ship `<meta name="viewport" content="width=device-width, initial-scale=1" />`. Without it, mobile renders at 980px and ignores all your responsive CSS.
- Mobile-first: write minimal base styles, then enhance with `min-width` queries. Less override, less specificity churn.
- **Content-out, not device-down.** Add a breakpoint when the layout visually breaks — not at iPhone/iPad widths.
- Set media-query breakpoints in **rem**, not px, so high-default-font users get the mobile layout (better for them).
- Use **container queries** for reusable components; reserve **media queries** for page-level layout and OS preferences (dark mode, reduced motion, print).
- Fluid type with one `clamp(min, preferred, max)` rule kills per-breakpoint `font-size` overrides. Min/max in **rem**, never px.
- Never use `vw` alone for `font-size` — it breaks browser zoom. Always wrap in `clamp()` with rem bounds.
- `max-width: 100%` on all media (`img`, `video`, `picture`, `svg`, `iframe`) prevents overflow.
- Body text line length: **60–75ch**; `max-width: 65ch` is the sweet spot.
- The unit test (Josh Comeau): *"Should this scale when the user changes their browser font size?"* YES → `rem`/`em`. NO → `px`.
- `fr` (Grid) and `flex` (Flexbox) distribute *available* space proportionally — intrinsically responsive, no math.
- Never `user-scalable=no`. Disabling pinch-zoom is a WCAG violation.
- Don't reorder focusable elements visually (`order`, `grid-area`) out of sync with DOM order — it breaks keyboard/SR flow.
- `100vh` is broken on mobile (overflows behind browser chrome). Use `100dvh` for full-screen layouts.
- Use logical properties (`padding-inline`, `margin-block`, `text-align: start`) in reusable components so they work in RTL and vertical writing modes.

## Why fixed-pixel layouts fail

A `width: 960px` container, `font-size: 14px`, `padding: 20px` design assumes one viewport. Reality in 2026:
- **Viewport diversity is unbounded.** Phones, foldables, tablets, laptops, ultrawides, embedded screens, split-window multitasking. There is no "standard" width to target.
- **Users zoom and resize text.** A px font-size overrides the user's browser default (e.g., a low-vision user who set 24px). WCAG 1.4.4 requires text resizable to 200% without assistive tech — fixed px on the root/body breaks this.
- **Content length varies.** A heading that fits on one line in English wraps to three in German. Fixed heights clip it.
- **It produces breakpoint sprawl.** Pixel layouts force a media query at every device width, then more as new devices ship.

HTML before CSS *already* reflows perfectly. The job of responsive CSS is to add structure (multi-column, grids) *without* removing that innate fluidity.

## The three pillars (Marcotte, 2010) — and what replaced them

RWD was coined by Ethan Marcotte. Original three pillars:
1. **Fluid grids** — proportional units (`%`, later `fr`), not fixed px columns.
2. **Flexible media** — `max-width: 100%` so images/video scale down to their container.
3. **Media queries** — branch styles by viewport.

Modern evolution (**Intrinsic Web Design**, term coined by Jen Simmons at An Event Apart ~2017–2018) extends this: layouts respond to the *content's intrinsic needs* using CSS Grid, Flexbox, `min()`/`max()`/`clamp()`, and writing modes — not just percentage grids. Today, container queries push responsiveness down to the component, decoupling it from the viewport entirely.

```css
/* Pillar 2 — non-negotiable baseline for all media.
   Note: apply display:block to <img>/<video>/<svg>/<iframe>, NOT <picture>.
   <picture> is a wrapper; setting block on it causes double-block in some renderers. */
img, video, svg, iframe { max-width: 100%; height: auto; display: block; }
picture { max-width: 100%; }   /* picture itself stays inline-ish; its img gets block above */
```

## Relative units: choosing correctly

| Unit | What it's relative to | Use for |
|------|----------------------|---------|
| `rem` | root `font-size` (default 16px) | **All** font sizes, global spacing, breakpoints |
| `em` | element's own/parent `font-size` | Component-local scaling: button padding, icon size inside text |
| `%` | parent's corresponding dimension | Fluid widths, legacy grids |
| `ch` | width of the `0` glyph in current font | Text-container `max-width` (approximate line length) |
| `vw`/`vh` | 1% of viewport width/height | Fluid scaling — **only inside `clamp()`** for type; `vh` is buggy on mobile (use `dvh`) |
| `dvh`/`svh`/`lvh` | dynamic/small/large viewport height | Full-screen layouts — use `dvh` to match the visible screen on mobile |
| `cqw`/`cqi` | 1% of container inline-size | Component-relative scaling (with container query) |
| `px` | absolute CSS pixel | Borders, shadows, hairlines, decorative elements that should NOT scale with text |

**The decision test (Josh Comeau):** *"If the user changes their browser's default font size, should this value change too?"* YES → `rem`/`em`. NO → `px`. A 1px border should stay 1px; a font size should grow.

```css
/* Correct unit mix */
.card {
  padding: 1.5em;              /* scales with the card's own font-size */
  border: 1px solid #ddd;      /* hairline — never scales */
  border-radius: 8px;          /* decorative — px is fine */
  max-width: 65ch;             /* readable line length */
  font-size: 1rem;             /* respects user root preference */
}
```

### rem vs em: the compounding trap
`em` compounds through nesting. Parent `font-size: 1.2em` × child `1.2em` = **1.44×** actual. This is the #1 source of "why is this text huge" bugs. Rule: use `rem` for type so sizing is always anchored to root; reach for `em` only when you *want* a value to track its local context (e.g., padding that scales with a button's own font-size).

### Don't do the 62.5% trick
`html { font-size: 62.5%; }` (to make `1rem` = 10px for easy math) **breaks third-party component libraries and browser extensions** that assume a 16px root. Not recommended. Just divide by 16 (or use a build-time function).

## Fluid grids: Grid & Flexbox are inherently responsive

Skip percentage math. `fr` and `flex` distribute *remaining* space after fixed tracks and gaps.

```css
/* The "RAM" pattern — responsive with ZERO media queries.
   Cards auto-fit, wrap, and fill; minimum 16rem, grow to fill. */
.grid {
  display: grid;
  gap: 1rem;
  grid-template-columns: repeat(auto-fit, minmax(min(16rem, 100%), 1fr));
}
```
The `min(16rem, 100%)` guard prevents overflow on viewports narrower than 16rem (a classic `minmax` bug on tiny screens).

```css
/* fr distributes proportionally: sidebar fixed, content takes the rest */
.layout { display: grid; grid-template-columns: 18rem 1fr; gap: 2rem; }
```

### grid-template-areas — the most underused Grid feature
Name regions, then reorder them with a *single* property change in one media query. No JS, no DOM reordering.

```css
.page {
  display: grid;
  gap: 1rem;
  grid-template-columns: 1fr;
  grid-template-areas: "header" "nav" "main" "aside" "footer";
}
@media (min-width: 48rem) {
  .page {
    grid-template-columns: 12rem 1fr;
    grid-template-areas:
      "header header"
      "nav    main"
      "aside  main"
      "footer footer";
  }
}
.page > header { grid-area: header; }
.page > nav    { grid-area: nav; }
.page > main   { grid-area: main; }
.page > aside  { grid-area: aside; }
.page > footer { grid-area: footer; }
```
**Caveat:** visual reordering via `grid-area`/`order` does NOT change DOM/tab order. Keep the DOM order logical so keyboard and screen-reader flow stay correct.

### subgrid — align nested children to the parent grid
`grid-template-columns: subgrid` lets a child grid inherit the parent's tracks, so e.g. card titles and footers line up across a row regardless of content length.
```css
.card-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem; }
.card { display: grid; grid-row: span 3; grid-template-rows: subgrid; }
/* title / body / footer now align across all cards in the row */
```
Browser support: Firefox 71+ (2019), Safari 16+ (2022), Chrome/Edge 117+ (2023). Baseline 2023. Use without a fallback for all evergreen targets.

## Container queries: component-level responsiveness

Media queries ask "how wide is the *viewport*?" The wrong question for a reusable component — a card in a sidebar and the same card in main content are different widths at the *same* viewport. Container queries ask "how wide is the *container*?"

```css
.card-wrap { container-type: inline-size; container-name: card; }

.card { display: grid; gap: 1rem; }
@container card (min-width: 30rem) {
  .card { grid-template-columns: 12rem 1fr; }  /* flip to horizontal */
}
```

**Rules:**
- Put `container-type: inline-size` on *layout wrappers* (card-grid, sidebar), NOT on hundreds of small nodes — there's a real reflow/perf cost.
- Prefer `inline-size` containment over `size` (which contains both axes and causes fragmentation in many layouts).
- **Name** containers (`container-name`) when a component is queried from multiple depths, to avoid collisions.
- Limit to **2–3 container breakpoints per component**. Over-nesting queried containers degrades performance.
- A container query *cannot* read the size of its own `container-type` element — query the parent.

### Container query units
- `1cqw` = 1% of container inline-size · `1cqh` = 1% of block-size · `1cqi` = inline · `1cqb` = block.
```css
/* fluid type relative to the CONTAINER, not the viewport */
.card h2 { font-size: clamp(1.25rem, 3cqw + 0.75rem, 3rem); }
```

### Division of labor
| Concern | Mechanism |
|---------|-----------|
| Page-level layout (shell, columns) | Media queries |
| OS/user preferences (dark mode, `prefers-reduced-motion`, print) | Media queries |
| Reusable component adaptation | Container queries |

## Fluid typography & spacing with clamp()

One rule scales smoothly between a min and max across all viewports — no per-breakpoint overrides.

```css
/* clamp(MIN, PREFERRED, MAX) — preferred mixes vw (scaling) + rem (zoom-safe) */
h1 { font-size: clamp(2rem, 4vw + 1rem, 3rem); }

/* Root fluid scaling anchored to user preference.
   Using calc() avoids mixing unit types directly in clamp() args,
   which is spec-edge and browser-resolved rather than formally defined. */
html { font-size: clamp(1rem, calc(1rem + 0.25vw), 1.125rem); }
/* Avoid: clamp(1em, 16px + 0.25vw, 1.125em) — mixing px with em in a math function
   relies on browser-specific resolution of mixed absolute+relative units. It works
   in practice today but is not spec-guaranteed; prefer the rem form above. */
```

**Hard rules:**
- Min/max bounds in **rem** (px ÷ 16). If you use px bounds, the `vw` term ignores user zoom.
- Never `vw`-only for `font-size` (`font-size: 4vw`): zooming doesn't change the viewport, so text never grows — WCAG 1.4.4 failure.
- **WCAG safety ratio:** if `max ≤ 2.5 × min`, the full `clamp()` range stays WCAG SC 1.4.4 compliant on modern browsers. (Derivation: at 200% zoom the viewport shrinks to half, so the `vw` term halves; rem bounds absorb the rest. Keep within ~2.5× to be safe.)

**Computing the preferred term** between two known points (x = viewport px, y = font px):
```
slope (vw) = 100 × (y2 − y1) / (x2 − x1)
offset (rem) = (x1·y2 − x2·y1) / (x1 − x2) / 16
font-size: clamp(<y1>rem, <slope>vw + <offset>rem, <y2>rem);
```

**Modular scale** with `pow()` (ratio 1.2 typical for body-level scales):
```css
:root {
  --step-1: calc(1rem * pow(1.2, 1));  /* 1.2rem */
  --step-2: calc(1rem * pow(1.2, 2));  /* 1.44rem */
  --step-3: calc(1rem * pow(1.2, 3));  /* 1.728rem */
}
```

**Caveats:** Pure fluid `clamp()` can desync from your spacing/baseline grid — line-heights can feel off at certain widths, and exact rendered size isn't predictable. For dense or rhythm-critical UI, consider stepped sizes at breakpoints. Also: scaling *padding/margin* with text isn't always right — low-vision users want bigger text, not bigger margins that fit fewer words per line. Scale type freely; scale spacing conservatively.

See [Fluid Typography & Spacing with clamp()](fluid-type-and-spacing.md) for the full treatment.

## min() / max() / clamp() beyond type
```css
/* Container that's fluid but never exceeds a readable cap, with safe gutters */
.container { width: min(70rem, 100% - 2rem); margin-inline: auto; }

/* Fluid gap with a floor and ceiling */
.stack { gap: clamp(1rem, 3vw, 2.5rem); }

/* Pick the larger so it never gets too small */
.hero { padding-block: max(4rem, 10vh); }
```

## @supports: progressive enhancement for responsive features

Use `@supports` to gate modern CSS behind a feature check, delivering a working baseline to older browsers without forking into a separate codebase.

```css
/* Fallback first: two-column float layout */
.layout { overflow: hidden; }
.sidebar { float: left; width: 200px; }
.content { margin-left: 220px; }

/* Enhancement: replace with Grid where supported */
@supports (display: grid) {
  .layout { display: grid; grid-template-columns: 200px 1fr; gap: 1.5rem; overflow: unset; }
  .sidebar, .content { float: none; width: auto; margin: 0; }
}

/* Gate container queries where supported */
@supports (container-type: inline-size) {
  .card-wrap { container-type: inline-size; }
}
```

**Rules:**
- Write the baseline (no `@supports`) first. Modern browsers override it; old ones use it.
- Don't `@supports` things that are universally supported (Grid, Flexbox, `clamp()` in 2026 — just ship them).
- Use `@supports not` to remove fallback cruft after the feature block.
- `@supports selector(:has(...))` gates selector features (`:has()` was the last to ship — Chrome 105, Safari 15.4, Firefox 121).

## Logical properties: writing-mode–aware layout

Physical properties (`margin-left`, `padding-top`, `border-right`) break for RTL scripts and vertical writing modes. Logical properties describe layout relative to the text flow direction.

| Physical | Logical equivalent | Meaning |
|----------|--------------------|---------|
| `margin-left` | `margin-inline-start` | Before text flow |
| `margin-right` | `margin-inline-end` | After text flow |
| `padding-top` | `padding-block-start` | Before block flow |
| `width` | `inline-size` | Dimension along text direction |
| `height` | `block-size` | Dimension perpendicular to text |
| `border-left` | `border-inline-start` | Leading edge of text |

**Rule:** use logical properties for all layout spacing, sizing, and borders unless you intentionally want a physical side (e.g., a decorative left-edge accent bar that should always be on the left). Use the shorthand forms: `margin-inline`, `padding-block`, `inset-inline`.

```css
/* Physical — breaks in RTL */
.card { padding-left: 1.5rem; padding-right: 1.5rem; text-align: left; }

/* Logical — works for LTR, RTL, and vertical writing modes */
.card { padding-inline: 1.5rem; text-align: start; }
```

All evergreen browsers support logical properties. IE does not — fallback only if required.

## aspect-ratio: reserve space, prevent layout shift
```css
.thumb  { aspect-ratio: 16 / 9; object-fit: cover; width: 100%; }
.avatar { aspect-ratio: 1; }       /* perfect square/circle holder */
.video  { aspect-ratio: 16 / 9; }  /* responsive embed, no padding hack */
```
Pairing `aspect-ratio` with `width`/`height` attributes on `<img>` is the modern way to eliminate CLS (see [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md)).

## Dynamic viewport units: dvh / svh / lvh (mobile browser chrome)

`100vh` on mobile has a long-standing bug: mobile browsers (iOS Safari, Chrome Android) exclude the collapsible address bar from `vh`, making a "full-height" hero taller than the visible screen and causing scroll bleed. The fix shipped in 2022 with three new units:

| Unit | Meaning |
|------|---------|
| `dvh` | **Dynamic** viewport height — tracks the *current* visible area (shrinks/grows as browser chrome hides/shows) |
| `svh` | **Small** viewport height — always the smallest visible area (chrome fully visible, safe floor) |
| `lvh` | **Large** viewport height — always the largest visible area (chrome hidden, most generous) |
| `dvw`/`svw`/`lvw` | Same trio for width |

**Rules:**
- Full-height hero sections: use `height: 100dvh` — it matches what users actually see.
- Fixed overlays (modals, drawers): use `height: 100dvh` so they don't extend behind the address bar.
- If you want conservative (never overflow): `min-height: 100svh`.
- Avoid `100vh` for anything that must fill the visual screen on mobile — use `dvh`.
- All evergreen browsers support `dvh`/`svh`/`lvh` as of 2023; no fallback needed for modern targets. For older support: `height: 100vh; height: 100dvh;` (second declaration wins where supported).

```css
/* Before: classic iOS Safari 100vh bug */
.hero { height: 100vh; }  /* overflows behind address bar on mobile */

/* After */
.hero { height: 100dvh; }  /* correct on all modern browsers */

/* Conservative: always at least the small viewport, grows if chrome hides */
.page-shell { min-height: 100svh; }
```

## Responsive media (don't just hide it)
Hiding with `display: none` on mobile **still downloads** the resource (especially background images). Serve the right asset:
```html
<img
  src="card-800.jpg"
  srcset="card-400.jpg 400w, card-800.jpg 800w, card-1600.jpg 1600w"
  sizes="(min-width: 48rem) 33vw, 100vw"
  width="800" height="450"
  alt="Descriptive text"
  loading="lazy" decoding="async" />
```
Use `<picture>` for art direction (different crops per viewport) or modern formats (`avif`/`webp`) with fallback. See [Imagery, Icons & Visual Assets](../00-foundations/imagery-icons-and-assets.md).

## Mobile-first, content-out workflow
1. Write base styles for the narrowest layout (single column, stacked).
2. Resize the browser until the layout *visually breaks* (line length too long, cards too wide, whitespace awkward). That's your breakpoint — note the rem value, not the device name.
3. Add a `min-width` query there. Repeat.
4. Validate at extremes: 320px, 768px, 1024px, 1440px, and a deliberately *odd* width (e.g., 900px, 1100px) to catch device-targeted assumptions.
5. Test at 200% browser zoom and with a 24px browser default font.

## Concrete specs & numbers
- Default browser root font: **16px = 1rem**. Don't override at root with px.
- **WCAG 1.4.4:** text resizable to **200%** without assistive tech.
- **clamp() WCAG-safe ratio:** `max ≤ 2.5 × min` always passes SC 1.4.4 on modern browsers.
- **Readable line length:** 60–75ch; `max-width: 65ch` is the sweet spot.
- **clamp() syntax:** `clamp([min], [preferred vw + rem], [max])`, e.g. `clamp(2rem, 4vw + 1rem, 3rem)`.
- **Fluid formula:** `v = 100 × (y2 − y1) / (x2 − x1)`; offset `r = (x1·y2 − x2·y1) / (x1 − x2)` (px). Divide offset by 16 for rem.
- **Fluid root:** `html { font-size: clamp(1rem, calc(1rem + 0.25vw), 1.125rem); }` — prefer this over the mixed-unit form.
- **Container query units:** `1cqw` = 1% inline-size, `1cqh` = 1% block-size, `cqi` = inline, `cqb` = block.
- **Viewport reference breakpoints** (use only as orientation, not targets): 480px (mobile landscape), 768px (tablet), 1024px (laptop), 1280px (desktop), 1536px (large). **Real best practice: break where content breaks.** Set them in rem (e.g., 30rem, 48rem, 64rem, 80rem, 96rem).
- **Modular scale ratio:** 1.2 common for body-level; 1.25–1.333 for more dramatic display scales.
- **Browser support (2026 baseline):** Container size queries — all evergreen engines stable (Chrome 105+, Safari 16+, Firefox 110+). Style queries on custom properties — Chrome/Edge/Safari stable; Firefox 131+ (Oct 2024). Non-custom-prop style queries (e.g. `@container style(font-style: italic)`) — Chrome 111+, Safari 18+, Firefox 135+. `clamp()` — all evergreen; IE cannot parse it (provide a static px fallback on the line before the `clamp()` rule only if IE11 is an explicit requirement — it should not be for new projects in 2026). `dvh`/`svh`/`lvh` — all evergreen since 2023; use freely.

## Tools & libraries
- **modern-fluid-typography.vercel.app** — visualize/calculate `clamp()` params from min/max size + min/max viewport. Use when hand-tuning a fluid type rule.
- **clampgenerator.com** — generate fluid typescales with a scale multiplier; exports CSS/Tailwind/SCSS/custom properties.
- **Chrome/Firefox DevTools Grid inspector** — overlay grid lines, track sizes, area names. Use it; most devs forget it exists.
- **fluid-tailwind (npm)** — Tailwind plugin for `clamp()`-based fluid utilities (community-rebuilt for v4). Note: **Tailwind v4** changed responsive/fluid utility generation — watch for v3-plugin breakage. See [CSS, Styling & Tailwind](../04-libraries-and-tools/css-styling-and-tailwind.md).
- **whatunit.com** — interactive unit-choice guide (ironically poor on mobile; cited as a "don't be this site" example).
- **Learning:** Kevin Powell (YouTube) for Grid/Flexbox/responsive; Josh Comeau (joshwcomeau.com) for units & accessibility; codingfantasy.com CSS Grid Attack for grid syntax practice.
- **When NOT to reach for a tool:** simple two-point fluid type — just write the `clamp()` by hand with the formula above. Don't add a Tailwind fluid plugin if you only need a handful of rules.

## Do / Don't

**Do**
- Ship the viewport meta tag in `<head>` on every page; omitting it makes mobile browsers render at ~980px and ignore responsive CSS.
- Write base (mobile) styles first; enhance with `min-width`.
- Set breakpoints in **rem** and where content breaks.
- Use `max-width: 100%` on all media.
- Use container queries for reusable components; media queries for page shell + OS prefs.
- Use `clamp()` with rem bounds for fluid type/spacing.
- Use `min-height` (not `height`) on text containers so they grow with content/zoom.
- Use `fr`, `minmax()`, `auto-fit`, and `grid-template-areas` to cut media queries.
- Keep DOM order logical even when visually reordering.
- Use `100dvh` for full-viewport-height layouts on mobile.
- Use logical properties (`padding-inline`, `margin-block`, `inline-size`) for all reusable components.
- Use `@supports` to gate progressive enhancements; write the baseline first.

**Don't**
- Don't set the root/body `font-size` in `px`.
- Don't use `vw` alone for `font-size`.
- Don't do the `62.5%` root hack.
- Don't put `container-type` on hundreds of small nodes.
- Don't `user-scalable=no` or `maximum-scale=1`.
- Don't reorder focusable elements out of DOM order purely for looks.
- Don't `display:none` heavy assets on mobile expecting bandwidth savings — use `srcset`/`<picture>`.
- Don't use fixed `height` on text — it clips on zoom/long content.
- Don't target device widths (320/375/414/768) as breakpoints.
- Don't use `100vh` for full-screen mobile layouts — use `100dvh`.
- Don't use physical properties (`margin-left`, `padding-right`) for reusable components — use logical properties.
- Don't mix px and relative units as direct arguments to `clamp()` without wrapping in `calc()` — it relies on browser-specific resolution of mixed units.

## Common pitfalls & how to avoid them
- **px root/body font-size overrides user preference.** A low-vision user who set 24px default gets 14px anyway → accessibility failure. Fix: rem-based type, anchored to root.
- **vw-only font-size ignores zoom.** Ctrl/Cmd+ doesn't change the viewport, so text never grows. Fix: `clamp(min-rem, …vw + …rem, max-rem)`.
- **62.5% trick breaks 3rd-party libs/extensions** that assume 16px root. Fix: don't; divide by 16.
- **`ch` is approximate.** It's the width of `0`; proportional fonts vary per glyph (a `0` may be 13px while `8` is 15px). Use `ch` for *approximate* line length, not exact character counts.
- **Device-width breakpoints rot.** New devices ship at new sizes constantly. Fix: content-driven, rem breakpoints.
- **px media queries break at high zoom/font.** "Mobile" layout shows at desktop widths for high-zoom users. Fix: rem breakpoints.
- **`container-type` everywhere → layout thrashing.** Fix: scope containment to layout wrappers only.
- **Visual reorder breaks keyboard/SR flow.** `order`/`grid-area` don't move the DOM. Fix: keep DOM order meaningful; reorder only non-focusable decoration freely.
- **`minmax(16rem, 1fr)` overflows below 16rem.** Fix: `minmax(min(16rem, 100%), 1fr)`.
- **`100vh` on mobile hides content behind the browser address bar.** iOS Safari and Chrome Android set `vh` relative to the maximum (address-bar-hidden) height, so `height: 100vh` heroes overflow the visible screen. Fix: `height: 100dvh`.
- **Not testing real conditions.** Responsive correctness depends on actual device conditions: test on throttled network (Chrome DevTools → 3G preset), at odd widths (e.g., 500px, 900px) to catch device-targeted assumptions, and on a physical small phone.

## What real users complain about
- **Design handoff vs fluid reality:** "How do I deal with a design team that wants exact pixel counts at each breakpoint?" — fluid CSS conflicts with pixel-precise mockups. Educate stakeholders that a value scales *between* anchor points; show the live range, not a static frame.
- **px confusion:** "Zoom works fine with px now — what's actually wrong with px?" The nuance is persistently missed: **browser zoom** scales px fine, but **OS/browser font-size preference** does not. px font sizes ignore the user's chosen default. Both mechanisms must work.
- **Grid syntax intimidation:** "The syntax is a pain… I've been avoiding it." Many devs retreat to Flexbox for everything because Grid has too many equivalent shorthands. Counter: learn the 4 essentials (`grid-template-columns`, `gap`, `auto-fit/minmax`, `grid-template-areas`) and ignore the rest until needed.
- **em vs rem flame wars:** "em and rem are unusably bad" vs "no they aren't." Root cause is em **compounding** in nested components (1.2em × 1.2em = 1.44×). Use rem for type to end the surprises.
- **Fluid type breaks rhythm:** "Line-heights feel off at certain widths and you can't predict the exact size." Real tradeoff of pure `clamp()`. Mitigate with conservative ranges and a tested baseline.
- **Scaling padding hurts a11y:** "Users need bigger text, not bigger margins that fit fewer words per line." Scale type liberally, spacing cautiously.
- **Tailwind responsive bloat:** long `class="mx-auto flex max-w-sm items-center gap-x-4 rounded-xl bg-white p-6 shadow-lg md:… lg:…"` strings hurt readability — a real portability-vs-readability tradeoff teams argue over.
- **`grid-template-areas` still "slept on"** years after shipping — devs don't realize one media query can re-lay-out a whole page with zero JS.

## Senior-level checklist
- [ ] `<meta name="viewport" content="width=device-width, initial-scale=1" />` present; no `user-scalable=no`/`maximum-scale`.
- [ ] Root/body `font-size` is NOT px; type uses rem (+ `clamp()` where fluid).
- [ ] All media have `max-width: 100%`; images have intrinsic `width`/`height` or `aspect-ratio` (no CLS).
- [ ] Breakpoints expressed in **rem** and placed where content breaks, not at device widths.
- [ ] Reusable components use **container queries** (`container-type: inline-size` on the wrapper, named if multi-depth); page shell + OS prefs use media queries.
- [ ] Fluid `clamp()` rules use rem min/max; `max ≤ 2.5 × min` for WCAG safety; no `vw`-only font sizes.
- [ ] Body text constrained to ~65ch; text containers use `min-height`, never fixed `height`.
- [ ] Layout uses `fr`/`auto-fit`/`minmax(min(…,100%),1fr)`/`grid-template-areas` to minimize media queries; `minmax` guarded against narrow-viewport overflow.
- [ ] Visual reordering (`order`/`grid-area`) never desyncs focusable elements from DOM/tab order.
- [ ] Verified at 320 / 768 / 1024 / 1440px **and** an odd width; at 200% zoom; with a 24px browser default font.
- [ ] Heavy assets aren't merely `display:none` on mobile — served via `srcset`/`sizes`/`<picture>`.
- [ ] `clamp()`/container-query usage checked against target browser matrix; px fallback only if legacy support is required.
- [ ] Full-viewport-height layouts use `dvh`, not `vh`; `svh` used where a conservative floor is needed.
- [ ] Reusable components use logical properties (`padding-inline`, `margin-block`, `inline-size`, `text-align: start`) so they work in RTL and vertical writing modes.
- [ ] Progressive enhancements (container queries, subgrid) gated with `@supports` if the project targets non-evergreen browsers; baseline written first.

## Sources
- web.dev — responsive design and fluid type guidance: https://web.dev
- Josh Comeau — units & accessibility (the font-size scaling test): https://www.joshwcomeau.com
- modern-fluid-typography (clamp calculator): https://modern-fluid-typography.vercel.app
- clamp generator (fluid typescales): https://clampgenerator.com
- WCAG 2.2 — SC 1.4.4 Resize Text (200%): https://www.w3.org/TR/WCAG22/
- MDN — container queries: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries
- MDN — dynamic viewport units (dvh/svh/lvh): https://developer.mozilla.org/en-US/docs/Web/CSS/length#relative_length_units_based_on_viewport
- MDN — CSS logical properties: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values
- Ethan Marcotte — "Responsive Web Design" (A List Apart, 2010): https://alistapart.com/article/responsive-web-design/
- Kevin Powell (CSS education): https://www.youtube.com/@KevinPowell
- CSS Grid Attack (Grid practice): https://codingfantasy.com/games/css-grid-attack/play

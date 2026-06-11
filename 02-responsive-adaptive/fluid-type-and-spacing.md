# Fluid Typography & Spacing with clamp()

> How to build type and spacing scales that interpolate smoothly across viewports with a single `clamp()` rule per token — eliminating per-breakpoint `font-size`/`margin` overrides — while staying fully WCAG-compliant for zoom and reflow. Master this to ship a coherent, breakpoint-light design system where headings, body, and spacing all scale in one proportional language from a 320px phone to a 1500px+ desktop.

**Apply when:** defining any type scale or spacing system; replacing per-breakpoint `font-size`/padding rules; building a token-driven design system. · **Related:** [Responsive & Fluid Design](responsive-and-fluid-design.md) · [Breakpoints, Devices & Mobile-First](breakpoints-devices-mobile-first.md) · [Web Typography Systems](../00-foundations/typography-systems.md) · [Design Systems & Design Tokens](../05-process-and-systems/design-systems-and-tokens.md)

## TL;DR — rules at a glance
- `clamp(min, preferred, max)`: `min`/`max` are **hard bounds**; `preferred` is a linear interpolation that mixes a viewport/container unit with `rem`, e.g. `clamp(2.25rem, 2vw + 1.5rem, 3.25rem)`.
- **Never use a bare `vw` for `font-size`.** `vw` does not respond to browser zoom → WCAG 1.4.4 failure. Always include a `rem` term in the preferred value so zoom moves the text.
- **Rule of 2.5×:** keep `max ≤ 2.5 × min` in every `clamp()`. This guarantees WCAG 1.4.4 (resize to 200%) passes at *every* viewport width, not just the endpoints.
- Prefer **`vi`** (inline-axis, writing-mode-aware) over `vw`, and **`cqi`** (container inline) for component-scoped text. Both behave better than raw `vw`.
- **Utopia approach:** define a type/space scale at a min viewport and at a max viewport using two modular ratios; let the browser interpolate. No type/space breakpoints needed.
- Compute the preferred value with **slope + intercept**, both in `rem`: `slope = (max − min) / (maxVW − minVW)`; `intercept = min − slope × minVW`; `preferred = [slope·100]vw + [intercept]rem`.
- **Do not assume `1rem === 16px`** in your math (OddBird, 2025). Users change their browser default. Keep `min`/`max` relative; never bake px-derived decimals as semantic intent.
- Apply fluid type **to headings/display first** (H1–H3). Fluid **body text is usually overkill** — use plain `rem` with 1–2 breakpoints.
- Spacing scale gets **one-up pairs** (`--space-s-m`, `--space-m-l`) with steeper slopes so component padding "breathes" more than type.
- **Vertical rhythm:** prefer a fluid line-height like `calc(1.3em + 0.3vw)` over a fixed unitless value — fixed `1.5` is too airy at 2rem+ headings.
- Test at **multiple zoom levels AND intermediate widths** (≈1050–2100px) — a clamp can pass at 320/1500 but fail in the middle.
- Watch **WCAG 1.4.10 Reflow**: clamp-based padding + fixed-width containers can cause horizontal scroll at 400% zoom (320px effective viewport).
- **Dynamic viewport units**: `vw`/`vi` equal the *large* viewport (mobile toolbar hidden). Use `svh`/`dvh` for full-viewport height elements; for fluid type preferred values, `vi` is fine.
- Keep the whole type + space system **small enough to memorize** — a scale you must constantly look up is broken (Utopia philosophy).

## How `clamp()` works

```css
font-size: clamp(MIN, PREFERRED, MAX);
```
- `MIN` — the value below which it never shrinks (small viewports). Use `rem`.
- `MAX` — the value above which it never grows (large viewports). Use `rem`.
- `PREFERRED` — evaluated continuously; the browser picks `max(MIN, min(PREFERRED, MAX))`. This is where the *fluid* part lives, and it **must** mix a viewport unit with `rem`.

The browser clamps to `MIN`/`MAX` outside the active range and follows `PREFERRED` between them. Because `PREFERRED` is a linear function of viewport width, the result is a straight-line interpolation between the two endpoints.

```css
/* 36px @ 600px viewport → 52px @ 1400px viewport */
h2 { font-size: clamp(2.25rem, 2vw + 1.5rem, 3.25rem); }
```

## Building a fluid value: slope + intercept

Given a min size, max size, and the viewport range you interpolate across, the preferred value is a line. Work in `rem` (assume the *math base* of 16px only to convert your design px to rem — it is not a runtime assumption).

```
slope     = (maxSize_rem − minSize_rem) / (maxVW_rem − minVW_rem)
preferredVW   = slope × 100            // the vw/vi coefficient
intercept_rem = minSize_rem − slope × minVW_rem
preferred = (slope × 100)vi + intercept_rem
```

Worked example — 18px → 20px across 320px → 1500px (`1rem = 16px` for conversion only):
```
minSize = 1.125rem (18px), maxSize = 1.25rem (20px)
minVW   = 20rem (320px),   maxVW   = 93.75rem (1500px)
slope     = (1.25 − 1.125) / (93.75 − 20)  = 0.125 / 73.75 = 0.001695
preferredVW   = 0.001695 × 100 = 0.1695   → 0.1695vi
intercept_rem = 1.125 − (0.001695 × 20)   = 1.125 − 0.0339 = 1.0911rem

preferred = 0.1695vi + 1.0911rem
```
```css
body { font-size: clamp(1.125rem, 1.0911rem + 0.1695vi, 1.25rem); }
```
Note: the Utopia generator uses a slightly different internal normalization and may produce `0.2174vi + 1.0815rem` — verify with the generator for your exact min/max viewports. Understand the formula to audit and hand-tune; use a generator for a full scale.

## The Utopia approach (two ratios, no breakpoints)

Utopia (Trys Mudford & James Gilyead) defines **two complete modular scales** — one anchored at the min viewport, one at the max — and interpolates every step fluidly between them. This removes the need for type/space breakpoints entirely.

Typical configuration:
- **Min viewport 320px**, base font **17px**, min ratio **1.2 (Minor Third)**.
- **Max viewport 1500px**, base font **20px**, max ratio **1.333 (Perfect Fourth)**.

The **two-ratio trick**: a *tighter* ratio at min and a *wider* ratio at max means the scale "opens up" on large screens — headings gain more hierarchy on desktop without overwhelming mobile. Each step gets its own `clamp()`:

```css
:root {
  --step--1: clamp(0.7813rem, 0.7747rem + 0.0326vi, 0.8rem);
  --step-0:  clamp(0.9375rem, 0.9158rem + 0.1087vi, 1rem);    /* base */
  --step-1:  clamp(1.125rem,  1.0815rem + 0.2174vi, 1.25rem);
  --step-2:  clamp(1.35rem,   1.2761rem + 0.3696vi, 1.5625rem);
  --step-3:  clamp(1.62rem,   1.5041rem + 0.5793vi, 1.9531rem);
  --step-4:  clamp(1.944rem,  1.7707rem + 0.8664vi, 2.4414rem);
  --step-5:  clamp(2.3328rem, 2.0734rem + 1.2971vi, 3.0518rem);
}
h1 { font-size: var(--step-5); }
p  { font-size: var(--step-0); }
```

Two distinct products come out of Utopia: a **fluid type scale** (font sizes) and a **fluid space scale** (spacing tokens). Both ship as CSS custom properties.

## Fluid spacing tokens

Build a spacing scale that **mirrors the type scale's ratio** (e.g. type ratio 1.25 → spacing steps multiply by ~1.25). Same proportional language = a system you can hold in working memory.

```css
/* All preferred values use vi (writing-mode-aware). Slopes derived from
   320px→1500px (20rem→93.75rem) range, same as the type scale. */
:root {
  --space-3xs: clamp(0.25rem,   0.2331rem + 0.0847vi, 0.3125rem);  /* 4→5px   */
  --space-2xs: clamp(0.5625rem, 0.5456rem + 0.0847vi, 0.625rem);   /* 9→10px  */
  --space-xs:  clamp(0.875rem,  0.8581rem + 0.0847vi, 0.9375rem);  /* 14→15px */
  --space-s:   clamp(1.125rem,  1.0911rem + 0.1695vi, 1.25rem);    /* 18→20px */
  --space-m:   clamp(1.6875rem, 1.6367rem + 0.2542vi, 1.875rem);   /* 27→30px */
  --space-l:   clamp(2.25rem,   2.1822rem + 0.3390vi, 2.5rem);     /* 36→40px */
  --space-xl:  clamp(3.375rem,  3.2733rem + 0.5085vi, 3.75rem);    /* 54→60px */

  /* one-up pairs: span two steps → steeper slope for component padding */
  --space-s-m: clamp(1.125rem,  0.9432rem + 0.9091vi, 1.875rem);   /* 18→30px */
  --space-m-l: clamp(1.6875rem, 1.4148rem + 1.3636vi, 2.5rem);     /* 27→40px */
}
```

**One-up pairs** (`--space-s-m`, `--space-m-l`) jump two steps with a steeper slope. Use them for component padding so internal spacing scales more aggressively than type — layouts gain room to breathe on desktop while staying compact on mobile.

```css
.c-card { padding: var(--space-s-m); }   /* ≈18px @320 → ≈33px @1500 */
.section { padding-block: var(--space-l-xl); }
```

For **vertical** space tokens you may interpolate against `vh` instead of `vi` (e.g. hero block spacing tied to viewport height) — but most spacing should track the inline axis.

## Accessibility: zoom, reflow, and the hard rules

Fluid type interacts directly with two AA success criteria. Getting the math wrong silently fails an audit.

### WCAG 1.4.4 Resize Text (AA) — the zoom rule
- Text must be **resizable to 200%** without loss of content or function.
- **Bare `vw`/`vi` does not respond to browser *text* zoom** (only page zoom moves it, and even that is partial). A heading using pure `vw` grew only **64px → 68px (6.25%) at 200% zoom** in Adrian Roselli's testing — a hard failure (he expected ~100% growth).
- **The `rem` term saves you.** Because `rem` tracks the user's root font setting, mixing `rem` into the preferred value means browser text-zoom still scales the text.
- **Rule of 2.5×:** if `maxFontSize ≤ 2.5 × minFontSize`, the clamp passes 1.4.4 at *all* widths in modern browsers. The reasoning: at 200% text zoom the `rem` term doubles, which doubles the effective minimum. If `2 × minSize ≥ maxSize`, the doubled minimum always meets or exceeds the unzoomed maximum — text is provably larger at every viewport. Violate this and there is a middle viewport band where the clamp is still held at `max` but the doubled `min` hasn't caught up. (e.g. min 16px → max 40px = 2.5× is safe; min 16px → max 48px = 3× fails in the middle band.)

### WCAG 1.4.10 Reflow (AA) — the 320px rule
- Content must reflow at **320 CSS px wide** (a 1280px screen at 400% zoom) with **no horizontal scroll** for vertical-reading content.
- Clamp-based padding + a fixed-width container can push total width past 320px → horizontal scrollbar → failure.
- `overflow: hidden` on a fluid-text container **clips content** at high zoom (WCAG F102 failure). Avoid it on text containers.
- **Sticky headers** that eat ~30% of a 400%-zoom viewport leave too little reading area — un-fix sticky elements at narrow widths via media query (technique C34).

### Other accessibility rules
- **Don't assume `1rem = 16px`** (OddBird, 2025): users change their browser default. Express `min`/`max` as relative intent, not px-derived decimals you treat as fixed pixels.
- **Optionally scale the root** so all downstream `rem` respect the user's preference *and* fluidity:
  ```css
  html { font-size: clamp(1em, 0.9em + 1vw, 1.5em); } /* OddBird 2025 */
  ```
  Note `em` here = the user's browser default, anchoring to their preference. If you set a root size, never set it in raw `px`.

## Container-relative fluid type (`cqi`)

When text lives in a **narrow sidebar, grid column, or reusable card**, `vw` sizes it to the *full viewport* — wrong. Use **`cqi`** (container query inline unit) so it scales to the component's actual width. Requires `container-type` on an ancestor.

```css
.component { container-type: inline-size; }

.banner {
  /* scales inside the component, not the viewport */
  font-size: clamp(1rem, 2cqi + 0.5rem, 1.5rem);
}
```
- `cqi` has **better zoom behavior than `vw`** in component contexts.
- Support: Chrome/Edge 105+, Firefox 110+, Safari 16+, Opera 91+, Samsung 20+.
- Decision rule: **`vi` for page/layout-scale type; `cqi` for anything inside a variable-width container.**

## Dynamic viewport units (`svw`/`dvw`/`lvw`)

Mobile browsers resize the viewport when their toolbar appears or disappears. Classic `vw`/`vi` equals the **large viewport** (toolbar hidden) — a full-bleed hero sized to `100vi` will be clipped when the toolbar is shown.

| Unit | Equals |
|------|--------|
| `svw` / `svi` | **Small** viewport — toolbar fully visible (most conservative) |
| `dvw` / `dvi` | **Dynamic** — updates as toolbar appears/disappears (can cause repaints) |
| `lvw` / `lvi` | **Large** — toolbar fully hidden (classic `vw` behavior) |

**Rules for fluid type/spacing:**
- For `clamp()` preferred values in **text**, stick to `vi`/`lvi` — a toolbar-height difference rarely matters for mid-scale text sizing.
- For **full-viewport layout elements** (hero heights, sticky-bar spacing, background fills), use `svh`/`dvh` to avoid clipping — not `vh`.
- `dvw` in a `clamp()` preferred value causes continuous repaints on scroll — avoid.
- Support for `svw`/`dvw`/`lvw`: all modern browsers since 2022 (Chrome 108+, Firefox 101+, Safari 15.4+).

## Vertical rhythm across breakpoints

Line-height, font-size, and line-length form a **triad** — change one and the other two need attention. A fixed unitless `1.5` is comfortable for 1rem body but becomes *too airy* on a 2rem heading.

```css
/* Fluid line-height: tightens at large sizes, opens at small */
h1, h2, h3 { line-height: calc(1.3em + 0.3vw); }
/* Body: keep line-height stable; fluidity is not needed here */
body { line-height: 1.5; }
```
"Molten Leading" (the CSS-Tricks technique by Nils Binder) applies the fluid pattern specifically to `line-height` to keep leading proportional as `font-size` changes. The pattern `calc(Nem + Xvw)` is the general form; tune `N` to your tightest acceptable leading at large sizes and `X` for how much it relaxes at small sizes.
- Keep body line length at **60–75ch** (`max-width: 65ch`) so fluid sizing never produces over-long measure.
- **Mirror the scales:** if type ratio is 1.25, use 1.25× spacing steps so the vertical rhythm (margins between text blocks) stays proportional to the type at every width.
- Size **icons at ~1.5em** to match base line-height and stay aligned to adjacent text as it scales.

## Concrete specs & numbers
- **Typical Utopia config:** min vw 320px, max vw 1500px; min base 17px, max base 20px; min ratio 1.2 (Minor Third), max ratio 1.333 (Perfect Fourth).
- **Smashing example:** 36px @600px → 52px @1400px ⇒ `clamp(2.25rem, 2vw + 1.5rem, 3.25rem)`.
- **WCAG 1.4.4:** text resizable to **200%** without loss.
- **WCAG 1.4.10:** reflow at **320 CSS px** (≈1280px @ 400% zoom), no horizontal scroll.
- **Safe ratio:** `max / min ≤ 2.5` guarantees 1.4.4 (e.g. 16px → 40px = 2.5×).
- **Pure-`vw` failure (Roselli):** heading grew 64px → 68px (6.25%) at 200% zoom = fail.
- **Named modular ratios:** Minor Second 1.067 · Major Second 1.125 · Minor Third 1.2 · Major Third 1.25 · Perfect Fourth 1.333 · Augmented Fourth 1.414 · Perfect Fifth 1.5 · Golden 1.618.
- **Andy Bell fluid scale example** (from CUBE CSS / Smol CSS work): small end `clamp(0.7rem, 0.66rem + 0.2vw, 0.8rem)` → large end `clamp(3.34rem, 2.45rem + 4.43vw, 5.61rem)`; step 700 `clamp(1.71rem, 1.45rem + 1.29vw, 2.37rem)`.
- **Body example (slope/intercept formula, 18px@320 → 20px@1500):** `clamp(1.125rem, 1.0911rem + 0.1695vi, 1.25rem)`. Utopia generator may produce slightly different coefficients due to internal normalization — cross-check with the generator for production code.
- **Space pair:** `--space-s` `clamp(1.125rem, 1.0911rem + 0.1695vi, 1.25rem)` vs `--space-s-m` `clamp(1.125rem, 0.9432rem + 0.9091vi, 1.875rem)` — the one-up pair has a 5× steeper vi coefficient, producing far more dramatic growth across the viewport.
- **`cqi`/container-query unit support:** Chrome/Edge 105+, Firefox 110+, Safari 16+, Opera 91+, Samsung 20+.
- **`clamp()` support:** >90% globally by 2022; universal 2024–2025.
- **Native `@function`** (CSS Values Level 5, encapsulates clamp math without Sass): flag-gated experimental in Chrome/Edge as of mid-2026; not production-ready. Track via caniuse.
- **Intermediate-width test band:** failures cluster around the max-breakpoint value when the viewport is ≈1050–2100px.

## Tools & libraries
- **utopia.fyi** — canonical fluid type + space generator (Mudford/Gilyead). Outputs CSS custom properties. Companion **Figma plugin v15** (June 2024: typographic variables, space pairs, dark mode, modes), **postcss-utopia**, and **utopia-core-scss** (`generateTypeScale`/`generateSpaceScale`, `allPairs: true`). *Use as the default for a full system.*
- **fluid-type-scale.com** (Aleksandr Hovhannisyan) — standalone calculator generating full clamp-based `font-size` custom properties; open source.
- **clampgen.com** — free clamp generator; supports **`vw` and `cqw`** modes; includes a modular-ratio type-scale generator. *Use for one-off values.*
- **clampgenerator.com** — exports clamp as plain CSS, SCSS variables, or Tailwind config snippets.
- **tailwind-utopia** (cwsdigital) — Tailwind plugin generating utility classes for interpolated Utopia type/space scales.
- **modern-fluid-typography.vercel.app** — open-source visualizer (referenced by Smashing) for fine-tuning clamp configs.
- **Tokens Studio** — manage fluid tokens Figma → CSS custom properties.
- **When NOT to use these:** for a single heading, hand-write the clamp — pulling in a plugin/build step is overkill. For body text, often skip fluid entirely (plain `rem` + a breakpoint).

## Do / Don't

**Do**
- Always include a `rem` term in the preferred value (`2vw + 1.5rem`, never `2vw`).
- Keep `max ≤ 2.5 × min` in every clamp.
- Prefer `vi` over `vw`; use `cqi` for component-scoped text.
- Use two ratios (tighter min, wider max) so hierarchy opens up on desktop.
- Build spacing one-up pairs for component padding; mirror the type ratio.
- Use a fluid line-height (`calc(1.3em + 0.3vw)`) on large headings.
- Generate scales with Utopia, then read/audit the output by hand.
- Test at 200% and 400% zoom *and* at intermediate widths.

**Don't**
- Don't use bare `vw`/`vi` for `font-size` (zoom fails → WCAG 1.4.4).
- Don't set `max > 2.5 × min`.
- Don't treat `1rem = 16px` as a runtime guarantee in your math/intent.
- Don't set the root `font-size` in `px`.
- Don't make body text fluid by default — usually unnecessary edge-case surface.
- Don't put `overflow: hidden` on fluid-text containers (clips at zoom, F102).
- Don't apply a Perfect Fifth/Golden ratio without a tighter mobile ratio — headings overflow on small screens.
- Don't use `vw` for text inside a narrow column — it sizes to the wrong box.

## Common pitfalls & how to avoid them
- **Pure `vw` font-size** — `clamp(1rem, 3vw, 3rem)` looks fine but the `vw` middle band ignores text zoom → 1.4.4 failure at intermediate widths. Fix: add a `rem` term, e.g. `clamp(1rem, 1.5vw + 0.5rem, 3rem)`.
- **`1rem = 16px` assumption** — px-derived decimals break for users who changed their default. Keep `min`/`max` relative.
- **`max > 2.5× min`** — creates a viewport band where 200% zoom is unreachable even with a `rem` term. Cap the ratio.
- **Same `vw` coefficient on tiny and large clamps** — bounds differ → change points land at different viewport widths → visual inconsistency across the scale. Derive each step's slope from its own endpoints.
- **Fluid body text** — the ROI is low: body text varies ≤2px across the entire viewport range in typical Utopia configs. Reserve fluidity for display/headings where the range is 10px+.
- **Clamp padding without Reflow testing** — fixed-width container + fluid spacing → horizontal scroll at 320px effective. Test at 400% zoom.
- **`overflow: hidden` on text containers** — clips content at high zoom (F102).
- **Sticky header at 400% zoom** — consumes the reading area; un-fix it at narrow widths (C34).
- **Large ratio without a mobile counterpart** — 1.5/1.618 makes desktop headings huge and mobile disproportionate. Use a tighter min ratio.
- **`vw` inside a sidebar/grid column** — sizes to viewport, not component. Switch to `cqi` with `container-type: inline-size`.
- **Figma-as-source-of-truth** — sub-pixel rounding differs Safari vs Chrome across zoom/DPR, causing icon-alignment and pagination drift. Treat computed CSS, not the Figma px, as truth.

## Known failure modes in the wild
- **Fluid body text scope creep:** applying fluid type to all text, not just display/headings, adds surface area (slope mismatches, zoom failures) for negligible design gain — keep fluidity to H1–H3 and display.
- **Invisible zoom failures (Roselli):** headings that *look* responsive but barely grow under text zoom (6.25% instead of ~100%) silently fail 1.4.4; teams ship them unaware until an accessibility audit.
- **Sub-pixel rounding across browsers:** Safari and Chrome compute sub-pixel positions differently at varying zoom/DPR — "pixel-perfect" Figma handoffs break icon alignment and text-adjacent UI under fluid sizing.
- **"Stuck" text for low-vision users:** `vw`-only sized text doesn't respond to browser text-zoom — defeating the user's accessibility preference.
- **Unmemorable scales:** a scale with 10+ tokens no one remembers gets abandoned or applied inconsistently mid-project; keep the system within working memory.

## Senior-level checklist
- [ ] Every `font-size` clamp's preferred value contains a `rem` term (no bare `vw`/`vi`/`cqi`).
- [ ] Every clamp satisfies `max ≤ 2.5 × min` (verify each step, not just base).
- [ ] Min/max bounds are `rem`/`em`, not px-derived decimals treated as fixed.
- [ ] Type scale uses two ratios (tighter min, wider max) where hierarchy benefits.
- [ ] Spacing scale exists, mirrors the type ratio, and includes one-up pairs for component padding.
- [ ] Component-scoped text uses `cqi` + `container-type`, not `vw`.
- [ ] Root `font-size` (if set) is `em`/clamp-based, never `px`.
- [ ] Line-height on large headings is fluid (`calc(... em + ... vw)`), body measure ≤ 65–75ch.
- [ ] Body text is plain `rem` (+ optional breakpoint) unless fluid is justified.
- [ ] Verified at **200% and 400% zoom** AND at intermediate widths (~1050–2100px).
- [ ] No horizontal scroll at 320px effective viewport (1.4.10); no `overflow: hidden` on text; sticky headers un-fix at narrow widths.
- [ ] Full-viewport layout elements use `svh`/`dvh`, not `vh`, to account for mobile browser toolbar.
- [ ] Whole type + space system is small enough to memorize.

## Sources
- Utopia — fluid type & space generator: https://utopia.fyi
- fluid-type-scale.com (Aleksandr Hovhannisyan): https://www.fluid-type-scale.com
- clampgen.com (clamp generator, vw/cqw): https://clampgen.com
- tailwind-utopia (cwsdigital): https://github.com/cwsdigital/tailwind-utopia
- utopia-core-scss (Trys Mudford): https://github.com/trys/utopia-core-scss
- Smashing Magazine — "Modern Fluid Typography Using CSS Clamp": https://www.smashingmagazine.com/2022/01/modern-fluid-typography-css-clamp/
- WCAG 2.2 — 1.4.4 Resize Text: https://www.w3.org/WAI/WCAG22/Understanding/resize-text.html
- WCAG 2.2 — 1.4.10 Reflow: https://www.w3.org/WAI/WCAG22/Understanding/reflow.html
- Adrian Roselli — "Responsive Type and Zoom" (pure-vw 6.25% zoom failure documentation): https://adrianroselli.com/2019/12/responsive-type-and-zoom.html
- CSS-Tricks — "Molten Leading" (fluid line-height technique, Nils Binder): https://css-tricks.com/molten-leading-css/
- MDN — Viewport units (svw/dvw/lvw reference): https://developer.mozilla.org/en-US/docs/Web/CSS/length#viewport-percentage_lengths

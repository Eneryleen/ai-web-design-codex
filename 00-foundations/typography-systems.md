# Web Typography Systems

> How to architect a production typographic system for the web: modular type scales, measure and leading as co-dependent variables, fluid sizing with `clamp()`, font loading and CLS control, OpenType features, and vertical rhythm. Read this to make defensible numeric decisions (ratios, line lengths, line-heights, weights) instead of eyeballing, and to ship type that is fast, accessible, and legible across devices.

**Apply when:** designing or implementing any text-bearing page or design system. · **Related:** [Fluid Type & Spacing with clamp()](../02-responsive-adaptive/fluid-type-and-spacing.md) · [Visual Hierarchy](visual-hierarchy.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md)

## TL;DR — rules at a glance

- **Pick one ratio per context.** Minor Third (1.2) for dense UIs, Major Third (1.25) for general design systems, Perfect Fourth (1.333) for editorial. A design system can run two ratios: tight (1.125–1.2) for UI, wide (1.333–1.5) for content.
- **Base = 16px.** Universal browser default and the practical WCAG floor for body. Only move to 18px after testing on real devices with real copy.
- **Measure = 45–75 characters (sweet spot 55–66).** Implement as `max-width: 65ch` (≈ 32–36rem). Shorter lines → tighter leading; longer lines → more leading.
- **Body line-height 1.5–1.6 desktop, 1.3–1.45 mobile. Headings 1.0–1.35.** Never ship body at the browser default of ~1.2 — too tight.
- **Measure and leading are co-dependent, not independent.** Longer lines mechanically require more line-height to return the eye to the next line start.
- **Fluid type uses `clamp(min-rem, vw + rem, max-rem)`.** The preferred value MUST mix `vw` and `rem`. Pure `vw` fails WCAG 1.4.4 (text won't scale on zoom). `min`/`max` MUST be `rem`, never `px`.
- **`font-display: swap` is the default** for text fonts (kills FOIT). Use `font-display: optional` + `preload` for zero-CLS critical fonts. Never `block` for body.
- **WOFF2 only.** Universal support since 2016; ~30% smaller than WOFF1 (Brotli). Skip WOFF/TTF fallbacks unless a contractual IE11 requirement applies.
- **Subset aggressively** — the single biggest lever. Latin-only subsets cut 90%+ (Roboto 168KB TTF → 12KB WOFF2; Inter 303KB → 22KB).
- **Self-host fonts.** Cache partitioning (Chrome 86+) erased the cross-site caching benefit of Google Fonts CDN.
- **Variable fonts win at 3+ weights or with animation;** below 3 weights, individual subsetted static WOFF2 files are usually smaller.
- **Two type families max per page.** One workhorse (body + UI) + one accent (display/brand). Three needs strong justification.
- **`text-wrap: balance` on h1–h3; `text-wrap: pretty` on body.** One line each, kills orphans.
- **Tabular data → `font-variant-numeric: tabular-nums`.** Prevents misaligned number columns.
- **Match fallback metrics with `size-adjust`/`ascent-override`** to eliminate font-swap CLS. Also consider `font-size-adjust: from-font` for automatic x-height matching (Chrome 127+, Firefox 92+, Safari 16.4+).
- **`font-optical-sizing: auto`** (browser default) activates the `opsz` variable axis automatically — correct glyph shapes at display vs body sizes with no extra code.

---

## Type scale: pick a ratio, generate steps

A type scale is a base size multiplied by a constant ratio at each step. Consistency of ratio is what makes a hierarchy read as intentional.

### Choosing the ratio

| Ratio | Name | Value | Use for |
|---|---|---|---|
| Minor Second | 1.067 | very subtle | dense data UIs only |
| Major Second | 1.125 | subtle | UI component scales |
| Minor Third | 1.2 | tight | dense UIs, app chrome |
| Major Third | 1.25 | balanced | general web, design systems (Material) |
| Perfect Fourth | 1.333 | strong | editorial, content-heavy sites |
| Augmented Fourth | 1.414 | dramatic | display-led layouts |
| Perfect Fifth | 1.5 | bold | agency/editorial hero type |

**Heuristic:** scale the type first (ratio tool), *then* tune weight for extra hierarchy. Do not paper over a broken size scale with weight.

### Worked scales (base 16px)

Perfect Fourth (1.333), 6 visible steps:

```
xs   = 9px
sm   = 12px
base = 16px
md   = 21px
lg   = 28px
xl   = 37px
2xl  = 50px
```

Major Third (1.25), tighter for multi-level UI:

```
sm   = 12.8px
base = 16px
md   = 20px
lg   = 25px
xl   = 31.25px
2xl  = 39px
```

As CSS custom properties (Major Third, computed from a single `--ratio`):

```css
:root {
  --ratio: 1.25;
  --text-base: 1rem;                                   /* 16px */
  --text-sm:  calc(var(--text-base) / var(--ratio));   /* 12.8px */
  --text-md:  calc(var(--text-base) * var(--ratio));   /* 20px */
  --text-lg:  calc(var(--text-md)   * var(--ratio));   /* 25px */
  --text-xl:  calc(var(--text-lg)   * var(--ratio));   /* 31.25px */
  --text-2xl: calc(var(--text-xl)   * var(--ratio));   /* 39px */
}
```

### Two ratios in one system

Use the tight ratio for UI tokens and the wide ratio for prose. They share the same 16px base, so they interlock at the body size but diverge at the extremes:

```css
:root {
  --ui-ratio: 1.2;       /* buttons, labels, nav, table cells */
  --prose-ratio: 1.333;  /* article h1–h4, lede, pull quotes */
}
```

### The three-font-size rule for headings

H1, H2, H3 must be **visibly distinct in size**. If two adjacent heading levels read as the same size, the hierarchy has failed. With a 1.2 ratio adjacent levels can be too close — compensate with weight (e.g., h2 600 vs h3 600 but a clear size gap, or add weight delta). Reference: [Visual Hierarchy](visual-hierarchy.md).

## Measure (line length)

- **Target 45–75 characters per line** for body; 55–66 is the sweet spot (Bringhurst's canonical figure is 66). Wider than 75 chars measurably degrades reading speed and eye-return accuracy on screen.
- **Canonical implementation:** `max-width: 65ch`. The `ch` unit ≈ width of `0` in the current font.
- **In `rem`:** roughly 32–36rem (65ch ≈ 520px at 16px base → ~32.5rem; `ch` unit is font-dependent so rem is the font-independent fallback).

```css
.prose { max-width: 65ch; }       /* self-limiting, font-aware */
.prose-rem { max-width: 38rem; }  /* font-independent fallback */
```

**Inverted-pyramid test:** if you widen the column and the leading still "feels right," your measure is too short *or* your leading is too low — they're coupled. Re-tune both, not one.

## Line-height (leading)

Leading is a function of measure, not a free constant. Longer lines need more leading so the eye reliably finds the next line's start. This is mechanical, not aesthetic.

| Context | Line-height |
|---|---|
| Body, desktop (45–75 char) | 1.5–1.6 |
| Body, mobile (narrow column) | 1.3–1.45 |
| Headings | 1.0–1.35 |
| UI components | 1.2–1.3 |
| Buttons / single-line labels | 1.0–1.2 |

```css
body        { line-height: 1.55; }
h1, h2, h3  { line-height: 1.1; }
.button     { line-height: 1; }
```

Use **unitless** line-height so children inherit the multiplier, not a fixed px value. The browser default (~1.2) is correct *only* for headings and single-line UI, never for body.

## Font weights

| Role | Weight |
|---|---|
| Body text | 400 (regular) |
| Slightly complex UI | 500 (medium) |
| Subheadings | 600 (semibold) |
| Headings | 700 (bold) |

- **Avoid 300 (light) for body** — low contrast, especially on non-retina displays.
- **Avoid 900 (black) for long headings** — strokes merge, legibility drops. Reserve black for short display lines.

## Fluid typography with `clamp()`

`clamp(MIN, PREFERRED, MAX)` replaces stepped breakpoints with continuous scaling.

### Hard rules

1. **`MIN` and `MAX` MUST be `rem`** (px doesn't honor the user's browser font setting). Convert px → rem by `/16`.
2. **`PREFERRED` MUST mix `vw` + `rem`.** Pure `vw` fails **WCAG 1.4.4 Resize Text (AA)** because viewport units don't respond to zoom. The `rem` term restores zoom scaling.

### The formula

Given `minSize`/`maxSize` (px) at `minViewport`/`maxViewport` (px):

```
vw-coefficient = 100 × (maxSize − minSize) / (maxViewport − minViewport)
rem-offset     = (minViewport × maxSize − maxViewport × minSize)
                 / ((minViewport − maxViewport) × 16)
```

Worked example — 36px @ 600px viewport → 52px @ 1400px:

```css
/* clamp(2.25rem, 2vw + 1.5rem, 3.25rem) */
h1 { font-size: clamp(2.25rem, 2vw + 1.5rem, 3.25rem); }
```

Another — 16px @ 320px → 18px @ 1280px (body):

```css
body { font-size: clamp(1rem, 0.9375rem + 0.2083vw, 1.125rem); }
```

### Utopia-style scale output

A generated scale ships as custom properties, each a `clamp()` with a `vw + rem` preferred value:

```css
:root {
  --step-0: clamp(1.13rem, 1.08rem + 0.22vw, 1.25rem);
  --step-1: clamp(1.35rem, 1.24rem + 0.55vw, 1.67rem);
  --step-2: clamp(1.62rem, 1.41rem + 1.05vw, 2.22rem);
}
```

### The preferred-value test

At `minViewport`, `PREFERRED` must be **≥ MIN**; at `maxViewport`, **≤ MAX**. If either fails, `clamp()` flat-lines (clamps) in that range and you lose the fluid behavior there. Verify both ends, not just the middle. See [Fluid Type & Spacing](../02-responsive-adaptive/fluid-type-and-spacing.md).

## Web font loading & performance

### Format

**WOFF2 only.** Universal modern browser support since 2016; IE11 is the only hold-out (global share <0.5% as of 2026). WOFF2 uses Brotli compression and is typically 30% smaller than WOFF1 for the same font data. Drop WOFF/TTF fallbacks unless a contractual IE11 requirement forces it.

### Subsetting — the biggest lever

| Font | Before | After (Latin subset, WOFF2) | Reduction |
|---|---|---|---|
| Roboto Regular | 168KB TTF | 12KB | ~93% |
| Inter | 303KB | 22KB (Latin-only) | ~93% |

Median web font request is ~35KB compressed; 90th percentile ~115KB; 99th percentile 500–700KB (problem territory). (HTTP Archive 2024 Web Almanac, Fonts chapter.) Subset to the character sets you actually render.

### `unicode-range` for lazy, per-script loading

```css
@font-face {
  font-family: "Inter";
  src: url("/fonts/inter-latin.woff2") format("woff2");
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+2000-206F;
}
@font-face {
  font-family: "Inter";
  src: url("/fonts/inter-cyrillic.woff2") format("woff2");
  unicode-range: U+0400-04FF;
}
```

The browser downloads only the files whose ranges contain glyphs used on the page — essential for multi-language sites.

### `font-display`

```css
@font-face {
  font-family: "Brand";
  src: url("/fonts/brand.woff2") format("woff2");
  font-display: swap;
}
```

| Value | Block period | Swap period | Use for |
|---|---|---|---|
| `block` | 3s | infinite | icon fonts only |
| `swap` | 100ms | infinite | **default for text** (kills FOIT) |
| `fallback` | 100ms | 3s | secondary text |
| `optional` | 100ms | 0 (no swap) | critical fonts with zero-CLS goal |
| `auto` | UA default | — | avoid; non-deterministic |

- **`swap`** is the correct default for body and brand text — fallback shows immediately, swaps when the font arrives.
- **`optional` + `preload`** gives a zero-CLS path: if the font loads in ~100ms (likely when preloaded) it shows; if not, fallback persists for the whole visit, never causing a shift. Google recommends A/B testing this — it can regress if misapplied.
- **Never `block` for body** — up to 3s of invisible text (FOIT).

### `preload` the critical files

```html
<link rel="preload" href="/fonts/brand-regular.woff2"
      as="font" type="font/woff2" crossorigin>
```

Preload only the 1–2 most critical WOFF2 files. `crossorigin` is required even same-origin (fonts fetch in CORS mode).

### Avoid request waterfalls

- **Inline `@font-face` + preload links in `<head>`** as critical CSS, before any external stylesheet — removes the HTML→CSS→font waterfall.
- **Never `@import` fonts in CSS** — serializes HTML → CSS → `@import` → font. Use `<link>` in `<head>` for parallel download.

### Eliminating font-swap CLS

Match the fallback's metrics to the web font so the swap doesn't reflow. Font-swap CLS is among the most common non-image contributors to layout shift; eliminating it is a reliable CLS win without measuring a specific percentage.

```css
@font-face {
  font-family: "Brand-fallback";
  src: local("Arial");
  size-adjust: 107%;       /* match x-height / glyph width */
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
}
/* Then: font-family: "Brand", "Brand-fallback", sans-serif; */
```

Example calibration: Anonymous Pro → Courier New needs approximately `size-adjust: 113%` (values are font-pair specific; compute with tooling, never hard-code a generic number).

**`font-size-adjust`** (now widely supported: Chrome 127+, Firefox 92+, Safari 16.4+) is the complementary property: it makes the fallback font's x-height match the web font's, without touching line metrics. Use alongside `size-adjust` in `@font-face`:

```css
body {
  font-family: "Brand", "Brand-fallback", sans-serif;
  font-size-adjust: from-font;  /* inherit the 'cap-height' ratio from the first available font */
}
```

`from-font` reads the metric from the active font, so the fallback auto-adjusts. A powerful zero-config option when the web font has correct metrics embedded.

Always include a real system fallback so `optional` has something correct to render:

```css
font-family: "Brand", -apple-system, BlinkMacSystemFont,
             "Segoe UI", Roboto, Oxygen, Ubuntu, sans-serif;
```

## Variable fonts

- **Break-even is 3 weights.** Below 3, individual subsetted static WOFF2 files are usually smaller; at 3+ weights or with weight animation, one variable file wins on total payload.
- **Pick one strategy.** Don't ship both a variable file and static fallbacks.
- Load one file; drive axes with CSS:

```css
@font-face {
  font-family: "InterVar";
  src: url("/fonts/inter-var.woff2") format("woff2-variations");
  font-weight: 100 900;     /* declare the supported range */
  font-display: swap;
}
h1   { font-weight: 800; }
body { font-weight: 400; }
.tight { font-variation-settings: "wght" 520, "opsz" 28; }

/* font-optical-sizing: auto — browser default in all modern engines.
   Activates the 'opsz' axis automatically to correct glyph shapes
   at display sizes vs body sizes. Disable only when manually controlling
   'opsz' via font-variation-settings. */
* { font-optical-sizing: auto; }
```

Adoption (2024 Web Almanac): ~32% of sites use variable fonts; the weight axis (`wght`) is the most commonly used axis, present on the majority of variable-font pages.

## OpenType features

Most are **off by default** even when the font supports them. Enable explicitly.

```css
:root {
  /* kerning + standard ligatures, everywhere */
  font-kerning: normal;                       /* 'kern' */
  font-variant-ligatures: common-ligatures;   /* 'liga' */
}

/* Body prose: oldstyle figures sit better in running text */
.prose { font-variant-numeric: oldstyle-nums; }   /* 'onum' */

/* Tables & data: tabular + lining figures align columns */
.data { font-variant-numeric: tabular-nums lining-nums; } /* 'tnum' 'lnum' */

/* Fractions */
.recipe { font-variant-numeric: diagonal-fractions; }     /* 'frac' */

/* Small caps for acronyms / labels */
.smallcaps { font-variant-caps: small-caps; }             /* 'smcp' */
```

Low-level escape hatch when a feature has no high-level property:

```css
.special { font-feature-settings: "ss01" 1, "cv05" 1; }
```

| Feature | Tag | Where |
|---|---|---|
| Kerning | `kern` | global |
| Standard ligatures | `liga` | global |
| Oldstyle numerals | `onum` | body prose |
| Lining numerals | `lnum` | tables, UI |
| Tabular numerals | `tnum` | any numeric column |
| Fractions | `frac` | recipes, math |
| Small caps | `smcp` | acronyms, eyebrows |
| Optical size | `opsz` | variable fonts with opsz axis |

**`tabular-nums` is mandatory for any aligned numeric data** — proportional figures misalign columns. Invisible to most devs, obvious to users.

## Font pairing

- **Two families max per page.** One workhorse (body + UI) + one accent (headings/brand).
- **Contrast through classification or extremes**, not size: serif vs sans-serif, or large weight/width deltas — not "same font, bigger."
- **Match x-heights.** A tall-x-height body paired with a low-x-height display font reads as accidental. Aim for intentional density.
- **Safest architecture:** a display font for headings + a neutral workhorse sans for body. Three families needs strong justification (performance and aesthetic cost).

## Vertical rhythm

- Use a **consistent base unit** — a 4px or 8px grid, or the computed body line-height — as the common denominator for margins, padding, and inter-element spacing.
- **Consistency beats mathematical perfection.** Relative units (`rem`, `em`, `%`) make pixel-perfect rhythm impossible across browsers; chasing it is wasted effort. A coherent spacing scale that everything snaps to is the real goal.

```css
:root { --space: 0.5rem; }   /* 8px base unit */
h2  { margin-block: calc(var(--space) * 4) var(--space); }
p   { margin-block: 0 calc(var(--space) * 2); }
```

See [Layout, Grids & Spacing](layout-grids-and-spacing.md) for the full spacing system.

## Readability & wrapping

```css
h1, h2, h3 { text-wrap: balance; }   /* even line lengths, no orphans */
p          { text-wrap: pretty; }    /* prevents single-word last lines */

.prose {
  hyphens: auto;              /* fixes ragged narrow columns; ~10.5% of pages use it */
  overflow-wrap: break-word;  /* fallback for long unbreakable strings */
}
```

`text-wrap: balance` is broadly supported but used on only ~2.5–2.7% of pages — cheap, high-impact polish. Apply `hyphens: auto` to long-form body, especially on mobile/narrow measures, paired with `overflow-wrap: break-word`.

## Accessibility constraints (non-negotiable)

- **Contrast:** 4.5:1 body (AA), 3:1 large text (≥18px regular or ≥14px bold), 7:1 AAA. Body below 16px rarely passes AA at the same color as it would at 16px.
- **Zoom test:** zoom to 200%. If any text clips, overflows, or becomes unreadable → **WCAG 1.4.4 failure** (the most common typography a11y bug). Pure-`vw` font sizes are the usual culprit.
- **`rem` for sizes** so the user's browser font preference is respected; never lock body text in `px`.

Full coverage: [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md).

## Concrete specs & numbers

- **Base:** 16px (1rem). 18px only after device testing.
- **Ratios:** 1.067 / 1.125 / 1.2 / 1.25 / 1.333 / 1.414 / 1.5.
- **Measure:** 45–75 char (sweet spot 55–66); `65ch` ≈ 32–36rem (font-dependent).
- **Line-height:** body 1.5–1.6 desktop, 1.3–1.45 mobile; headings 1.0–1.35; UI 1.2–1.3; buttons 1.0–1.2.
- **Weights:** body 400, UI 500, subhead 600, head 700; avoid 300 body / 900 long heads.
- **Contrast:** 4.5:1 / 3:1 large / 7:1 AAA.
- **`font-display` periods:** block 3s; swap/optional/fallback block 100ms; swap swap-period infinite; optional 0; fallback 3s.
- **WOFF2:** ~30% smaller than WOFF1 (Brotli); median ~35KB, p90 ~115KB, p99 500–700KB (2024 HTTP Archive).
- **Subset wins:** Roboto 168KB→12KB; Inter 303KB→22KB (~93%).
- **Variable fonts:** ~32% adoption (2024 Web Almanac); `wght` is the dominant axis; break-even at 3 weights.
- **Font CLS:** font-swap is a leading non-image CLS source; eliminate with metric overrides or `optional` + `preload`.
- **`text-wrap: balance` adoption:** ~2.5–2.7% of pages. `hyphens: auto`: ~10.5%.
- **Material Design 3 reference roles (size/line-height px):** Display L 57/64, M 45/52, S 36/44; Headline L 32/40, M 28/36, S 24/32; Title L 22/28, M 16/24, S 14/20; Body L 16/24, M 14/20, S 12/16; Label L 14/20, M 12/16, S 11/16.

## Tools & libraries

- **[Utopia](https://utopia.fyi)** — fluid type + space scale generator; outputs `clamp(vw + rem)` custom properties for the whole scale. Use as the default for fluid scales.
- **[Typescale](https://typescale.com)** — quick modular-scale preview to choose a ratio.
- **[Modern Font Stacks](https://modernfontstack.com)** — system font stacks per classification when you want zero font requests.
- **`size-adjust` calculators** — auto-compute fallback metric overrides (`size-adjust`, `ascent-override`, etc.) to kill swap CLS; some build tools generate these automatically.
- **`glyphhanger` / `fonttools`** — subset and convert to WOFF2 in the build.
- **When NOT:** don't reach for a font tool for a system-font UI — a good system stack ships zero bytes and avoids the entire loading problem.

## Do / Don't

**Do**
- Generate the scale from one ratio; tune weight after, not instead.
- Constrain prose to `65ch` and tie line-height to measure.
- Mix `vw + rem` in every fluid `clamp()`; keep `min`/`max` in `rem`.
- Subset to used scripts; ship WOFF2 only; self-host.
- `font-display: swap` for text; `optional` + `preload` for zero-CLS critical fonts.
- Add fallback `@font-face` metric overrides + `font-size-adjust: from-font` to eliminate swap CLS.
- Let `font-optical-sizing: auto` do its job for variable fonts with `opsz`; only override when controlling `opsz` manually.
- Enable `kern`, `liga`; use `tabular-nums` for all numeric columns.
- Apply `text-wrap: balance` to h1–h3, `pretty` to body.

**Don't**
- Use `px` for `clamp()` `min`/`max`, or pure `vw` for font-size (WCAG 1.4.4 fail).
- Ship body at line-height ~1.2 or weight 300.
- Use more than two families without strong justification.
- `@import` fonts or chain them behind an external stylesheet.
- Use `font-display: block` for body (FOIT up to 3s).
- Ship both variable and static versions of the same family.
- Rely on Google Fonts CDN for cross-site caching — it no longer exists (cache partitioning).

## Common pitfalls & how to avoid them

- **Pure-`vw` fluid type.** Text won't zoom → WCAG 1.4.4 fail. Always add a `rem` term.
- **`px` in `clamp()` bounds.** Ignores user font settings. Convert to `rem`.
- **Clamp flat-lines at one end.** Run the preferred-value test at both viewport extremes.
- **FOIT / FOUT layout shift.** Use `swap` + fallback metric overrides, or `optional` + `preload`.
- **`@import` font waterfall.** Move to `<link rel="preload">` / inline `@font-face` in `<head>`.
- **Misaligned table numbers.** `font-variant-numeric: tabular-nums`.
- **Headings too similar (1.2 ratio).** Add weight delta; honor the three-font-size rule.
- **Body too tight / lines too long.** Re-tune measure and leading together — they're coupled.
- **Shipping the full font.** Subset; a 168KB→12KB cut is typical.
- **Orphaned heading words.** `text-wrap: balance`.

## What real users complain about

- **"The text is tiny / I can't zoom."** Sub-16px body and `vw`-locked sizes that won't respond to zoom. Fix: 16px base in `rem`, mixed-unit `clamp()`.
- **"Lines are too wide, I lose my place."** No `max-width` on prose; full-bleed paragraphs. Fix: `65ch`.
- **"It flashes blank then the text pops in."** FOIT from `block`, or a visible reflow from an uncalibrated swap. Fix: `swap` + metric overrides, or `optional` + `preload`.
- **"Numbers in the table don't line up."** Proportional figures. Fix: `tabular-nums`.
- **"Everything looks the same weight/size."** Flat hierarchy — broken or absent scale. Fix: real ratio + weight deltas.
- **"Headings have one lonely word on the last line."** No `text-wrap: balance`.
- **"Light gray text on white, can't read it."** Weight 300 + low contrast. Fix: 400+, ≥4.5:1.

## Senior-level checklist

- [ ] One ratio per context chosen and documented (UI vs prose if split).
- [ ] Scale emitted as `rem` custom properties from a single base + ratio.
- [ ] Body 16px (`rem`); 18px only after device testing.
- [ ] Prose constrained to `45–75ch` (`max-width: 65ch`).
- [ ] Line-height set per context; body 1.5–1.6 desktop / 1.3–1.45 mobile; unitless.
- [ ] Every fluid size is `clamp(rem, vw + rem, rem)`; preferred-value test passes at both extremes.
- [ ] Fonts WOFF2-only, subset, self-hosted; `unicode-range` for multi-script.
- [ ] `font-display` strategy chosen (`swap` default; `optional` + `preload` for critical).
- [ ] Critical 1–2 WOFF2 preloaded; `@font-face` in `<head>`; no `@import`.
- [ ] Fallback `@font-face` metric overrides in place; swap CLS verified ≈ 0.
- [ ] Variable vs static decided by the 3-weight break-even; only one strategy shipped.
- [ ] `font-optical-sizing: auto` globally set; manual `opsz` only when overriding.
- [ ] `kern` + `liga` on; `tabular-nums` on all numeric columns; `onum`/`lnum` as appropriate.
- [ ] ≤ 2 families; x-heights compatible; contrast from classification/weight.
- [ ] Fallback CLS eliminated: metric overrides via `@font-face` descriptors + `font-size-adjust` considered.
- [ ] Spacing snaps to a 4/8px (or line-height) base unit.
- [ ] `text-wrap: balance` on h1–h3; `pretty` + `hyphens: auto` on body.
- [ ] Contrast ≥ 4.5:1 body / 3:1 large; 200% zoom passes with no clipping (WCAG 1.4.4).

## Sources

- Utopia — fluid type & space calculator: https://utopia.fyi
- Type Scale — modular scale tool: https://typescale.com
- Modern Font Stacks: https://modernfontstack.com
- Material Design 3 — Typography: https://m3.material.io/styles/typography
- MDN — `font-display`: https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display
- MDN — `font-variant-numeric`: https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-numeric
- MDN — `size-adjust`: https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/size-adjust
- MDN — `font-size-adjust`: https://developer.mozilla.org/en-US/docs/Web/CSS/font-size-adjust
- MDN — `font-optical-sizing`: https://developer.mozilla.org/en-US/docs/Web/CSS/font-optical-sizing
- web.dev — Best practices for fonts: https://web.dev/articles/font-best-practices
- WCAG 2.1 — 1.4.4 Resize Text: https://www.w3.org/WAI/WCAG21/Understanding/resize-text.html
- HTTP Archive — Web Almanac, Fonts chapter: https://almanac.httparchive.org/en/2024/fonts

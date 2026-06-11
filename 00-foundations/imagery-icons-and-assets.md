# Imagery, Icons & Visual Assets

> How to use photography, illustration, and icons so they load fast, render crisp, crop intelligently, stay accessible, and look intentional rather than generic. Master this and your sites hit Core Web Vitals targets, avoid layout shift, and read as authentic instead of stock-clichéd.

**Apply when:** any page that ships a raster image, illustration, SVG icon, or hero · **Related:** [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Responsive & Fluid Design](../02-responsive-adaptive/responsive-and-fluid-design.md) · [Avoiding Generic 'AI' Aesthetics](../07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md)

## TL;DR — rules at a glance

- Serve `AVIF > WebP > JPEG` via `<picture>`; the browser auto-picks the smallest format it supports. AVIF is 40-50% smaller than JPEG, WebP 25-35%.
- **Never** lazy-load the LCP/hero image. `loading="lazy"` above the fold delays LCP 200-800ms by pulling the image out of the preload scanner queue.
- Add `fetchpriority="high"` to the LCP image. Documented gains: LCP 2.6s → 1.9s (27%) on Google's own Oodle demo; real-world gains typically 200-500ms on mid-tier connections.
- Set explicit `width` and `height` on **every** `<img>`, or `aspect-ratio` on the container for dynamic/CMS content. Either prevents CLS; bare `height: auto` alone is not enough.
- LCP `<picture>` with AVIF needs a `<link rel="preload" imagesrcset="..." type="image/avif">` in `<head>` — the preload scanner cannot evaluate `<source type>` conditions.
- `loading="lazy"` + `decoding="async"` on every below-the-fold image; never on the first in-viewport image. Use default `decoding` (auto) on the LCP image.
- Inline SVG over icon fonts for all new projects — multi-color, themable, animatable, accessible, no FOIT/CORS/ad-blocker failures.
- Design icons on a 24×24 grid; run every SVG through SVGO before deploy (strips 30-50% metadata bloat).
- Use `currentColor` as fill so icons inherit `color` for free theming.
- Resize the file to its display size (×2 for Retina). Serving a 5000px image into a 600px slot wastes ~98% of pixels ((5000/600)² ≈ 69× pixel count overload).
- `object-fit: cover` + per-image `object-position` (via CSS custom props) for focal-aware cropping with zero JS.
- Avoid stock tropes (handshakes, headsets, forced laptop-pointing smiles) — banner blindness filters them. Authentic imagery consistently outperforms generic stock in conversion tests; treat this as a qualitative rule, not a single number.
- `sizes` cannot use `%`. Valid: `px`, `vw`, `em`, `calc()`. Always end with an **unconditional fallback** (no media condition): `... , 33vw`.
- Decorative images: `alt=""` (empty, no space). Never write "image of" / "photo of" in alt.

## Why this matters (the numbers that drive every decision)

- Images are the **LCP element on ~85% of desktop pages and ~76% of mobile pages** (2025 HTTP Archive / Web Almanac).
- Images are **~48% of total page weight** on median sites — the single biggest performance lever you own.
- A **1-second delay reduces conversions by ~7%**; **40% of visitors abandon at >3s** load. Image weight is usually the cause.

Implication: image strategy is performance strategy. Get format, sizing, priority, and lazy-loading right before you tune anything else.

## Modern formats: AVIF, WebP, JPEG

Decision order is fixed: **AVIF first, WebP fallback, JPEG/PNG as the universal `<img>` source.**

```html
<!-- For non-LCP images, add loading="lazy" decoding="async" to the <img>.
     For the LCP hero: fetchpriority="high", NO decoding="async" (let it default to auto). -->
<picture>
  <source type="image/avif" srcset="hero-800.avif 800w, hero-1200.avif 1200w, hero-1600.avif 1600w" sizes="100vw">
  <source type="image/webp" srcset="hero-800.webp 800w, hero-1200.webp 1200w, hero-1600.webp 1600w" sizes="100vw">
  <img src="hero-1200.jpg" alt="" width="1600" height="900"
       fetchpriority="high">
</picture>
```

- The browser walks the sources top-to-bottom and uses the **first `type` it supports** — no JS, no UA sniffing.
- Put `width`/`height`/`alt` on the `<img>` only; it is the element that actually renders. The `<source>` elements just supply candidate URLs.
- `srcset` + `sizes` belong on each `<source>` (and/or the `<img>`) for resolution switching — orthogonal to format switching.

**Format facts**

| Format | vs JPEG | Browser support (early 2026) | Use for |
|---|---|---|---|
| AVIF | 40-50% smaller; beats WebP by 20-50% on complex photos | 94.9% global | Photographic content, first choice |
| WebP | 25-35% smaller | 96.4% global | Universal fallback, broad-compat |
| JPEG | baseline | ~100% | Final `<img src>` fallback |
| PNG | lossless, large | ~100% | Only when transparency + lossless needed; prefer SVG for logos/icons |
| JPEG-XL (JXL) | lossless or 30-60% smaller than JPEG; superior on text/graphics | Chrome 110+, Firefox 128+, Safari 17+ (~85% global, 2026) | Not yet ready as a default; add as a leading `<source>` if your audience is fully modern |

Don't gate on the missing ~5% AVIF support — `<picture>` already handles it via the WebP/JPEG fallbacks.

**JPEG-XL (JXL) in 2026:** JXL is gaining traction but sits behind AVIF in global support. If you already have a multi-format `<picture>` pipeline, adding a JXL `<source>` ahead of AVIF costs little and delivers measurable gains on graphics-heavy content. For most projects, AVIF → WebP → JPEG remains the pragmatic default.

## Responsive images: `srcset` + `sizes`

Two distinct jobs:

- **`srcset` with `w` descriptors** = list of candidate widths. The browser picks based on viewport × DPR × the `sizes` hint.
- **`sizes`** = tells the browser **how wide the image will render** before layout, so it can choose a candidate early (preload scanner runs before CSS).

```html
<img src="photo-800.jpg"
     srcset="photo-400.jpg 400w, photo-640.jpg 640w, photo-800.jpg 800w,
             photo-1200.jpg 1200w, photo-1600.jpg 1600w, photo-2560.jpg 2560w"
     sizes="(max-width: 600px) 100vw, (max-width: 1024px) 50vw, 33vw"
     width="800" height="600" alt="...">
```

**Rules for `sizes`**

- Cannot use percentages — `25%` is **invalid**. Use `vw`, `px`, `em`, or `calc()`.
- Always end with an **unconditional fallback** (a length value with no media condition): `(max-width:600px) 100vw, 33vw`. The parser requires this final entry to have no `(media-query)` prefix.
- The value describes rendered width, not file width. `50vw` means "this image fills half the viewport at this breakpoint."
- Wrong `sizes` silently wastes bandwidth (oversized file) or ships a blurry image (undersized file). It is the most-botched attribute in responsive imagery.
- `sizes` is consumed by the preload scanner *before CSS runs* — it must match what layout actually produces, not what you wish it were.

**Standard breakpoint widths to generate:** `400, 640, 800, 1200, 1600, 2560`. **2560px is the practical ceiling** — covers Retina/2× without bloating weight.

Bandwidth reality: an 800px file (~128KB) vs a 480px file (~63KB) saves ~65KB per image on mobile. Multiply across a gallery and it dominates your mobile LCP.

**Reserving layout space — `width`/`height` vs `aspect-ratio`:**

```html
<!-- Option A: explicit attributes (known dimensions at build time) -->
<img src="photo.jpg" width="800" height="600" alt="...">

<!-- Option B: CSS aspect-ratio (useful when dimensions are dynamic/unknown) -->
<style>
  .card-img { aspect-ratio: 4 / 3; width: 100%; }
</style>
<img class="card-img" src="photo.jpg" alt="...">
```

`width`/`height` attributes are preferred when you know the dimensions (static images, build pipeline). `aspect-ratio` on the *container* is the right tool for dynamic content, CMS-sourced images where dimensions vary, or when the container is fluid and you can't know exact pixel counts. Both prevent CLS — pick the one that fits your data flow. Never rely on CSS `height: auto` alone without one of these two.

## LCP, lazy loading & fetch priority

This is where most CMS-built sites bleed performance.

```html
<!-- HERO / LCP: eager + high priority. NEVER lazy. -->
<!-- decoding="auto" (default) is safer on the LCP image — "async" can defer decode past paint -->
<img src="hero-1200.jpg" srcset="..." sizes="100vw"
     width="1600" height="900" alt=""
     fetchpriority="high">

<!-- BELOW THE FOLD: lazy everything -->
<img src="card-600.jpg" srcset="..." sizes="(max-width:600px) 100vw, 33vw"
     width="600" height="400" alt="..." loading="lazy" decoding="async">
```

**LCP AVIF preload:** The browser's preload scanner cannot evaluate `<picture>` `type` conditions for AVIF format selection. For AVIF heroes, add an explicit preload in `<head>` alongside your `<picture>` element:

```html
<link rel="preload" as="image"
      href="hero-1600.avif"
      imagesrcset="hero-800.avif 800w, hero-1200.avif 1200w, hero-1600.avif 1600w"
      imagesizes="100vw"
      type="image/avif">
```

Without this, AVIF `<source>` candidates may not be picked up by the preload scanner, leaving LCP to the slower JPEG `<img src>`.

- **Never** `loading="lazy"` on an above-fold image — it leaves the preload-scanner queue and LCP slips 200-800ms on mid-tier mobile. WordPress core measured a **7% LCP regression** from blindly lazy-loading everything.
- **`fetchpriority="high"`** on the LCP image only. Typical gain 200-500ms; up to 20-30% in lab tests (Google web.dev benchmarks).
- `decoding="async"` is safe for all **below-fold** images (off-main-thread decode). **Do not** use it on the LCP image — some browsers schedule async decode after paint, delaying LCP. Use the default (`auto`) for the hero.
- Chrome native lazy-load thresholds (stable since Chrome 90): **4G = 1250px** from viewport edge, **≤3G = 2500px**. These are hardcoded in Chrome source, not tunable.
- Native lazy loading is good enough — do not reintroduce a JS lazy-loader.
- **CSS background images are invisible to the preload scanner.** If a background is your LCP, add `<link rel="preload" as="image" href="bg.avif" type="image/avif">` in `<head>`. Better: use a real `<img>` for anything LCP-eligible.
- Do **not** hide URLs in `data-src` (old JS pattern). This breaks the browser preload scanner (images start loading later) and Googlebot may never crawl them → images missing from search index.

**Native lazy-load support (2026 baseline):** Chrome 77+, Edge 79+, Firefox 75+, Safari 15.4+ (images), Safari 16.4+ (iframes). Baseline across all modern browsers — no polyfill needed.

## Cropping, focal points & art direction

Two mechanisms — know which to use:

- **Resolution switching** (`srcset`/`sizes`): same image, different pixel sizes. For most content images.
- **Art direction** (`<picture>` with `media`): *different crops* per breakpoint. For heroes where the desktop composition fails on portrait mobile.

```html
<!-- Art direction: different crops per breakpoint + format fallback.
     Order matters: browser takes the FIRST matching <source>. -->
<picture>
  <!-- Mobile portrait crop: AVIF, then WebP, then JPEG fallback -->
  <source media="(max-width: 600px)" type="image/avif" srcset="hero-mobile-800x1200.avif">
  <source media="(max-width: 600px)" type="image/webp" srcset="hero-mobile-800x1200.webp">
  <source media="(max-width: 600px)" srcset="hero-mobile-800x1200.jpg">
  <!-- Desktop landscape crop: AVIF, then WebP -->
  <source type="image/avif" srcset="hero-desktop-1920x1080.avif 1920w, hero-desktop-2560x1440.avif 2560w"
          sizes="100vw">
  <source type="image/webp" srcset="hero-desktop-1920x1080.webp 1920w, hero-desktop-2560x1440.webp 2560w"
          sizes="100vw">
  <!-- Universal JPEG fallback (also the LCP element) -->
  <img src="hero-desktop-1920x1080.jpg" alt="" width="1920" height="1080"
       fetchpriority="high">
</picture>
```

**Focal-point cropping without JS** — `object-fit` + per-image `object-position` via custom properties:

```css
.media img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: var(--focus-x, 50%) var(--focus-y, 50%);
}
```
```html
<div class="media" style="--focus-x:30%; --focus-y:20%;">
  <img src="..." alt="..." width="1600" height="900">
</div>
```

Set `--focus-x/--focus-y` per image so faces/subjects stay in frame when the container ratio changes. Without `object-position`, `cover` center-crops and **cuts off faces**.

**Composition heuristics for heroes**

- Choose **single-focal-point** images. Two subjects on opposite edges are nearly impossible to crop responsively — one always gets cut on narrow viewports.
- Keep the focal point centered or offset ~1/3 from center. A subject in the lower-right of the source ends up at the bottom on mobile, right side on desktop — plan composition to survive both crops.
- **Match aspect ratios across all `<picture>` sources** at a given container. If sources have different ratios, the container resizes as a new source loads → CLS. Set `aspect-ratio` on the container or fixed `width`/`height` to lock it.

## Hero imagery sizing

- Desktop: **1920×1080 (16:9)**, or **2560×1440** for Retina-targeted hero slots.
- Mobile: a dedicated **800×1200 (portrait)** crop, or a **center 500px-wide crop** of the desktop image (crop, don't scale — avoids downloading off-screen content).
- **Never serve a 5000px desktop image to a 400px phone.** It's the #1 mobile LCP killer.
- Pair art-direction crops with format switching (AVIF/WebP) inside one `<picture>`.

## Icon systems: inline SVG vs sprites vs icon fonts

**Default to SVG. Do not start new projects on icon fonts.** Inline SVG beats icon fonts on every axis for modern browsers: multi-color, per-element animation, predictable positioning, no CORS failures, no FOIT, better accessibility. Inline SVG is baseline across all 2026-era browsers — no compatibility qualifier needed.

| Approach | Use when | Notes |
|---|---|---|
| **Inline SVG** | Above-fold / critical icons | Zero HTTP request, renders immediately, fully styleable per-element |
| **SVG sprite** (`<use href="sprite.svg#id">`) | Icon reused across many pages | One cached request; `<symbol>` definitions in a single file |
| **Icon font** | Legacy maintenance only | Avoid for new work |

**SVG sprite pattern**

```html
<!-- one cached file: sprite.svg with <symbol id="arrow" viewBox="0 0 24 24">…</symbol> -->
<svg class="icon" width="24" height="24" aria-hidden="true">
  <use href="/icons/sprite.svg#arrow"></use>
</svg>
```

**`currentColor` for theming** — paths inherit the element's CSS `color`:

```html
<svg viewBox="0 0 24 24" width="24" height="24" fill="none"
     stroke="currentColor" stroke-width="2" aria-hidden="true">
  <path d="M5 12h14M13 6l6 6-6 6"/>
</svg>
```
```css
.btn:hover .icon { color: var(--accent); } /* no second SVG needed */
```

**Why icon fonts fail** (all real failure modes): CORS errors on cross-origin font files, Firefox font-loading bugs, ad blockers stripping the request, user stylesheets overriding glyphs, FOIT (invisible until font loads), single-color only, unpredictable vertical alignment.

**Sizing reality:** the full Font Awesome library is **~250KB**. If you need 10-15 icons, that's almost all wasted — import only the SVGs you use (or tree-shake an icon component lib). A counterpoint to keep honest: 37KB of SVGs can compress to ~6KB as a `.woff`; fonts still win on raw transfer, but SVG wins on *every other dimension* (color, a11y, animation, robustness) — so use SVG.

## Icon & image accessibility

- **Standalone informative icon** (no adjacent text): put a `<title>` inside the SVG and `role="img"`:
  ```html
  <svg role="img" viewBox="0 0 24 24" width="24" height="24"><title>Download</title>…</svg>
  ```
- **Icon next to a visible text label:** mark the SVG `aria-hidden="true"` so it isn't double-announced.
- **Icon-only interactive control** (button/link): give the control `aria-label="Close"`; mark the inner SVG `aria-hidden="true"`.
- **Decorative photos:** `alt=""` — empty string, **no space**. `alt=" "` (a space) is *not* empty and may still be announced.
- **Never** start alt with "image of" / "photo of" — screen readers announce the image role already. Describe content directly.
- **Images of text** (WCAG): alt must duplicate the text **exactly**.
- **Keep alt ≤ ~125 characters** — many screen readers truncate beyond that.
- **Complex charts/diagrams:** don't cram everything into alt. Provide a short alt summary plus an **accessible data table** below the image (or a linked long description).
- **Don't ship AI-generated alt text unreviewed** — it can't see surrounding page context and will mislabel relevance. Review every line.

See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) for the full image/alt decision tree.

## Visual consistency

- **One illustration system per page.** Mixing libraries (e.g. unDraw + Open Peeps) clashes stroke weights, palettes, and perspective → visual fragmentation. Pick one style; recolor to brand.
- **Unify inconsistent client photos** (varied lighting/white balance) with a CSS duotone/tint using brand colors + `mix-blend-mode`:
  ```css
  .tile { position: relative; }
  .tile::after { content:""; position:absolute; inset:0;
    background: var(--brand-accent); mix-blend-mode: color; opacity:.85; }
  ```
- **Logos:** convert PNG logos to SVG (Inkscape, Vector Magic). Smaller on mobile, pixel-crisp on Retina, themable via `currentColor`.
- Keep icon **stroke width, corner radius, and grid (24×24)** consistent across the whole set. See [Color Theory & Color Systems](../00-foundations/color-theory-and-systems.md) for tint/duotone palettes and [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md) for codifying asset rules.

## Avoiding generic stock

Users' brains **filter** stereotypical stock imagery — Jakob Nielsen's *banner blindness*. The cure is authenticity, not more polish.

**Skip these tropes:** business handshakes, call-center headsets, forced smiles pointing at laptops, suspiciously tidy conference rooms, diverse-team-laughing-at-nothing.

**Do instead:** real team photos, actual product screenshots, real customers/workspaces, candid (not posed) shots. The qualitative finding from A/B test literature is consistent: authentic contextual images outperform generic stock on trust and conversion metrics. Treat it as a design principle, not a fixed percentage — the gap varies wildly by context and audience. See [Avoiding Generic 'AI' Aesthetics](../07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md).

## Concrete specs & numbers

- **AVIF** 40-50% smaller than JPEG; **WebP** 25-35% smaller; AVIF beats WebP by 20-50% on complex photos.
- **Support (early 2026):** WebP 96.4% global, AVIF 94.9% global (caniuse).
- **Responsive widths to generate:** 400, 640, 800, 1200, 1600, **2560 (max)**.
- **Hero:** desktop 1920×1080 (or 2560×1440 Retina); mobile 800×1200 portrait or 500px center crop.
- **Lazy-load thresholds:** 4G = 1250px, ≤3G = 2500px from viewport.
- **Lazy-load reliability:** Native `loading="lazy"` is sufficient for all modern browsers — no JS loader needed.
- **`fetchpriority="high"`:** +200-500ms typical LCP improvement; up to 20-30% LCP reduction in lab (Google web.dev Oodle demo: 2.6s → 1.9s, 27%).
- **Icon grid:** 24×24px. SVGO removes 30-50% metadata bloat.
- **Font Awesome full lib:** ~250KB; need ~10-15 icons → import only those.
- **alt text:** ≤125 characters; images of text duplicate text exactly.
- **Page-weight context:** images ~48% of median page weight; LCP element on ~85% desktop / ~76% mobile pages.
- **Business impact:** 1s delay ≈ −7% conversions; >3s ≈ 40% abandon. AVIF at quality 60-75 typically cuts 40-50% vs JPEG at equivalent perceived quality — actual savings depend heavily on image content and encoder settings.

## Tools & libraries

**Build / convert**
- **Sharp** (Node) — fastest server-side resize/convert; drive build pipelines to emit WebP/AVIF at 400/768/1080/1200 etc. Use directly for compression pipelines (Imagemin is effectively unmaintained since 2021).
- **ImageMagick** or **vips** — batch CLI resizing in scripts; vips is significantly faster for large batches.
- **SVGO** — mandatory SVG optimizer before deploy.
- Framework built-ins: **Next.js `<Image>`** (auto WebP, per-device sizing, lazy-by-default with LCP opt-out via `priority`), **Astro `<Image>`** (Sharp → `<picture>` AVIF/WebP at build), **Vite image plugins**, **@codestitchofficial/eleventy-plugin-sharp-images** (11ty, auto-builds `<picture>`).

**No-build / manual**
- **Squoosh.app** — in-browser compress/convert (AVIF/WebP/JPEG/PNG) with quality preview, no signup.
- **compressor.io** — lossy compression with visual diff preview.
- **Topaz Photo AI** — upscale poor client/event photos.
- **Progressive JPEG encoding:** enable `progressive: true` in Sharp/ImageMagick for large JPEG fallbacks (>40KB). Progressive JPEGs render a blurry preview before full decode, improving perceived performance on slow connections. Use only for the JPEG fallback — AVIF/WebP don't benefit from progressive encoding in the same way.

**CDN / on-the-fly**
- **Cloudinary**, **imgix** — URL-param transforms, AVIF/WebP auto-format, focal-point cropping. **Thumbor** — open-source smart cropping with focal detection.

**Icons & illustration**
- **Lucide, Phosphor, Heroicons, Tabler** — consistent 24px-grid open-source SVG sets (React/Vue/raw).
- **unDraw, Open Peeps, Blush, Icons8** — free illustration sets (pick ONE per page).
- **svg-spreact / SVGO** — build SVG sprites (`<symbol>` + `<use>`). **icomoon.io** — legacy icon-font builder (avoid for new work).

**Audit**
- **Lighthouse / PageSpeed Insights** — track LCP, CLS, and "serve images in next-gen formats" / "properly size images" / "efficiently encode" opportunities.

**When NOT to use:** don't reach for a CDN transform service for a tiny static site (build-time Sharp/Squoosh is simpler); don't add a JS lazy-loader (native `loading="lazy"` is sufficient); don't pull a whole icon font for a handful of icons.

## Do / Don't

**Do**
- Ship `<picture>` AVIF → WebP → JPEG for photographic content.
- Add `<link rel="preload" imagesrcset="..." type="image/avif">` in `<head>` for AVIF LCP images.
- Set explicit `width`/`height` on every `<img>` (or `aspect-ratio` on the container for dynamic content).
- `fetchpriority="high"` + eager load + default `decoding` on the LCP/hero; `loading="lazy"` + `decoding="async"` on everything below the fold.
- Generate exactly the sizes you display (×2 Retina), up to 2560px.
- Run all SVGs through SVGO; use `currentColor`; design on a 24×24 grid.
- Set `object-position` per image; choose single-focal-point heroes.
- Use authentic photography; unify mismatched photos with a brand tint.

**Don't**
- Don't lazy-load the hero, or apply `loading="lazy"` globally (CMS default trap).
- Don't use `decoding="async"` on the LCP image — it can defer decode past paint in some browsers.
- Don't serve 3000-5000px images to phones.
- Don't omit `width`/`height` (CLS) or rely on CSS `height: auto` alone without `aspect-ratio`.
- Don't use `%` in `sizes`, or forget the unconditional fallback value at the end.
- Don't start new projects on icon fonts.
- Don't write "image of…" or use `alt=" "` for decorative images.
- Don't mix illustration libraries on one page.
- Don't `data-src`-hide image URLs (breaks preload scanner + crawl/index).

## Common pitfalls & how to avoid them

- **Global `loading="lazy"` including the hero** → +200-800ms LCP, 7% regression. *Fix:* eager + `fetchpriority="high"` on LCP; lazy only below fold.
- **Desktop-res images to mobile** → 2-4× wasted data, mobile LCP tanks. *Fix:* `srcset`/`sizes` + correct `sizes`, or art-directed mobile crop.
- **Missing `width`/`height`** → CLS as content reflows. *Fix:* explicit attributes or `aspect-ratio`.
- **`data-src` JS lazy-loading** → breaks browser preload scanner (images start loading later) and Googlebot may never crawl them. *Fix:* native `loading="lazy"` with real `src`.
- **`decoding="async"` on LCP image** → some browsers schedule async decode after first paint, delaying LCP visually despite the image being downloaded. *Fix:* omit `decoding` on the LCP element (let browser default to `auto`).
- **Icon fonts** → CORS, ad blockers, user CSS, FOIT, single-color, bad alignment. *Fix:* inline SVG / SVG sprite.
- **250KB icon font for 10 icons** → dead weight. *Fix:* import only used SVGs / tree-shake.
- **Two opposite focal points in a hero** → one always cropped out. *Fix:* commission/select single-focal images.
- **`alt="image of X"` / `alt=" "`** → redundant / not actually hidden. *Fix:* direct description / `alt=""`.
- **Unreviewed AI alt text** → context-blind, often wrong. *Fix:* human review every alt.
- **Unpreloaded CSS background as LCP** → invisible to preload scanner. *Fix:* `rel="preload" as="image"` or use a real `<img>`.
- **`<picture>` sources with mismatched aspect ratios** → CLS on source swap. *Fix:* lock container ratio.
- **`object-fit: cover` with no `object-position`** → faces center-cropped off. *Fix:* set focal `object-position`.
- **Mixed illustration styles** → fragmented look. *Fix:* one library, recolor to brand.

## What real users complain about

- "The page jumps while it loads and I tap the wrong thing." — missing `width`/`height` → CLS from reflowing images.
- "It took forever on my phone." — oversized desktop images shipped to mobile; the hero was lazy-loaded.
- "Stock photos everywhere, feels fake / like every other corporate site." — banner-blindness tropes (handshakes, headsets); erodes trust.
- "Icons are blurry / wrong color in dark mode." — PNG icons not converted to SVG; no `currentColor` theming.
- "Icons vanished / showed empty boxes." — icon-font failure via ad blocker, CORS, or FOIT.
- "Screen reader read out a filename / 'image image'." — missing or redundant alt; decorative images not `alt=""`.
- Despite AVIF's documented gains (40-50% over JPEG) and near-universal browser support, the 2025 Web Almanac shows a large majority of sites still ship raw JPEG/PNG — format optimization remains the highest-leverage, lowest-hanging improvement on most codebases.

## Senior-level checklist

- [ ] Every photographic image uses `<picture>` with AVIF → WebP → JPEG fallback (JXL optional leading source if pipeline supports it).
- [ ] LCP/hero: eager load, `fetchpriority="high"`, `decoding="auto"` (default — not "async"), **not** lazy.
- [ ] LCP `<picture>` with AVIF has a matching `<link rel="preload" imagesrcset="..." imagesizes="..." type="image/avif">` in `<head>`.
- [ ] Every below-fold image has `loading="lazy"` and `decoding="async"`.
- [ ] Every `<img>` has explicit `width` + `height` **or** the container has `aspect-ratio`; CLS verified at 0 in Lighthouse.
- [ ] `srcset` covers 400-2560px; `sizes` matches real rendered width, uses no `%`, ends with an unconditional fallback value.
- [ ] No image file exceeds ~2× its largest display size.
- [ ] Mobile hero is an art-directed crop, not a downscaled desktop image.
- [ ] All SVGs run through SVGO; icons on a 24×24 grid; `currentColor` for theming.
- [ ] Critical icons inline; repeated icons via cached SVG sprite; **no icon fonts**.
- [ ] Icon a11y: `aria-hidden` beside labels, `<title>`/`aria-label` for standalone/icon-only.
- [ ] Alt text reviewed by a human; decorative → `alt=""`; ≤125 chars; no "image of"; images-of-text duplicated exactly.
- [ ] One illustration style per page; mismatched photos unified with brand tint.
- [ ] No stock-cliché tropes in any hero or feature image.
- [ ] Critical CSS background images preloaded (or replaced with `<img>`).
- [ ] Lighthouse/PSI shows no "next-gen formats", "properly size images", or "efficiently encode" opportunities.

## Sources

- HTTP Archive / Web Almanac (Media chapter, 2025): https://almanac.httparchive.org/
- caniuse — AVIF: https://caniuse.com/avif · WebP: https://caniuse.com/webp · JPEG-XL: https://caniuse.com/jpegxl
- web.dev — `fetchpriority` / Optimize LCP: https://web.dev/articles/fetch-priority · https://web.dev/articles/optimize-lcp
- web.dev — Browser-level (native) lazy loading: https://web.dev/articles/browser-level-image-lazy-loading
- web.dev — `<link rel="preload">` for responsive images: https://web.dev/articles/preload-responsive-images
- MDN — Responsive images (`srcset`/`sizes`, `<picture>`): https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images
- MDN — `decoding` attribute: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#decoding
- Jakob Nielsen / NN/g — Banner blindness: https://www.nngroup.com/articles/banner-blindness-original-eyetracking/
- SVGO: https://github.com/svg/svgo
- Sharp: https://sharp.pixelplumbing.com/ · Squoosh: https://squoosh.app/
- Lucide: https://lucide.dev · Phosphor: https://phosphoricons.com · Heroicons: https://heroicons.com · Tabler: https://tabler.io/icons

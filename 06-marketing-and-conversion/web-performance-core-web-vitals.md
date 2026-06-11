# Web Performance & Core Web Vitals

> How to design and build pages that load fast, respond instantly, and stay visually stable — measured the way Google and real users actually measure them. Covers Core Web Vitals (LCP/INP/CLS), the optimization levers for each, performance budgets with CI enforcement, perceived performance, animation performance, and the direct performance-to-conversion link. Outcome: a senior engineer who ships pages that pass CWV at the 75th percentile in the field, not just in a clean lab run.

**Apply when:** building or auditing any production page where speed, ranking, or conversion matters (almost always). · **Related:** [Conversion Rate Optimization](./conversion-rate-optimization.md) · [SEO & Discoverability](./seo-and-discoverability.md) · [Imagery, Icons & Assets](../00-foundations/imagery-icons-and-assets.md) · [Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md)

## TL;DR — rules at a glance
- **Targets (75th percentile of real visits):** LCP ≤2.5s · INP ≤200ms · CLS ≤0.1. All three must pass at p75 or the page fails.
- **The LCP image gets `fetchpriority="high"`, NEVER `loading="lazy"`.** This is the single highest-impact LCP fix. Lazy-loading your hero is the most common self-inflicted wound.
- **The LCP resource must be in the initial HTML** so the preload scanner finds it. JS-injected or CSS `background-image` heroes can't be preloaded → always slow.
- **Set explicit `width`/`height` (or `aspect-ratio`) on every img, video, iframe, ad slot.** This is the primary CLS fix — it reserves space before load.
- **Animate only `transform` and `opacity`.** They skip layout+paint, run on the GPU compositor thread. Animating `width/height/top/left/margin` triggers layout and drops frames.
- **Lab ≠ field.** A Lighthouse score of 90/100 does NOT mean you pass CWV. Lighthouse can't even measure INP (it uses TBT as a proxy). Use CrUX (PageSpeed Insights / Search Console / RUM) for real status.
- **Break up long tasks (>50ms)** with `scheduler.yield()`. Long main-thread tasks inflate INP input delay.
- **Budget JS at ≤400KB gzipped** for interactive pages. Enforce in CI (Lighthouse CI / bundlesize) — fail the build on regression.
- **Ship less JS:** tree-shake (`import { x } from 'lib-es'`), code-split, prefer SSR/SSG over client-side rendering for content.
- **Self-host subsetted WOFF2 fonts** with `font-display: swap`. Subsetting can take 200KB → <20KB.
- **Cache content-hashed assets immutably:** `Cache-Control: public, max-age=31536000, immutable`. HTML: `no-cache`.
- **Perceived speed counts:** skeleton screens feel faster than spinners (they communicate structure and reduce uncertainty); optimistic UI feels instant.
- **Performance dies by a thousand cuts.** +50KB animation lib, +100KB tracker, +500KB hero — each "small," cumulatively fatal. Audit third parties quarterly.

---

## 1. The three Core Web Vitals — definitions, thresholds, anatomy

All CWV are evaluated at the **75th percentile of real page visits**: 75% of sessions must hit "Good" for the page to pass. This is Google's single most important evaluation rule. A fast median with a slow tail still fails.

| Metric | Good | Needs Improvement | Poor | Measures |
|---|---|---|---|---|
| **LCP** (Largest Contentful Paint) | ≤2.5s | 2.6–4.0s | >4.0s | Loading: time to render the largest above-fold element |
| **INP** (Interaction to Next Paint) | ≤200ms | 201–500ms | >500ms | Responsiveness: worst-case interaction latency over page lifetime |
| **CLS** (Cumulative Layout Shift) | ≤0.1 | 0.1–0.25 | >0.25 | Visual stability: unexpected layout movement |

Thresholds are empirical, not arbitrary: LCP 2.5s reflects a target Google set as "ambitious but achievable" when CWV launched, based on the distribution of real origins in CrUX; INP 200ms reflects HCI research where delays under ~100ms feel instant and quality degrades sharply past ~300ms; CLS 0.1 is noticeable-but-not-disruptive, while ≥0.15 was consistently perceived as disruptive in user studies.

**INP replaced FID in March 2024.** FID measured only the *first* interaction's input delay. INP measures the *full* interaction duration (input delay + processing + presentation) across *all* interactions for the page's lifetime, then reports roughly the worst. FID data persists in CrUX for historical comparison, but only INP affects ranking signals from March 2024 onward.

### 1.1 LCP breaks into 4 sequential phases — optimize all four
| Phase | ~Budget | What it is | Lever |
|---|---|---|---|
| TTFB | ~40% | Server/network time to first byte | CDN, SSR cache, edge, fast origin |
| Resource Load Delay | <10% | Gap before the LCP resource starts downloading | Preload, in-HTML discoverability, `fetchpriority` |
| Resource Load Duration | ~40% | Time to download the LCP resource | Smaller/compressed image, modern format, CDN |
| Element Render Delay | <10% | Gap between resource done and pixels painted | Reduce render-blocking CSS/JS, avoid client render |

Optimizing only image size addresses one phase. If TTFB is 1.8s, a perfect image still fails.

### 1.2 INP interaction anatomy — 3 components
```
[ user clicks ] -> Input Delay -> Processing Duration -> Presentation Delay -> [ next paint ]
```
- **Input Delay:** other tasks blocking the main thread delay the event handler from even starting. Fix: fewer/shorter long tasks.
- **Processing Duration:** your event callbacks running. Fix: do less synchronous work; yield; defer non-urgent work.
- **Presentation Delay:** layout recalculation + paint after handlers finish. Fix: avoid layout thrashing, shrink DOM, `content-visibility`.

Core fix: **yield to the browser between phases** (`await scheduler.yield()`), and let the visual update paint before heavy work.

INP observes **all** interactions (clicks, taps, keypresses — not scroll/hover/zoom). For pages with >50 interactions, 1 outlier per 50 is dropped to reduce noise. The `web-vitals` library v3+ works around the browser's default PerformanceEventTiming 104ms reporting threshold to capture short interactions — always use the library rather than raw PerformanceEventTiming for complete INP data.

### 1.3 CLS — what causes shifts and how to kill them
A shift happens when a visible element changes start position between frames *without a user-initiated cause*. Inserting content above existing content is the classic destroyer.
- Always give images/videos/iframes/ads explicit `width`+`height` or `aspect-ratio`.
- Reserve space for ads, embeds, and late-arriving banners with min-height boxes.
- Never inject content above existing content after load (cookie bars, "you have items" banners) without reserved space.
- Animate position with `transform`, never `top/left/margin`.
- Use `font-display: optional` or size-matched fallbacks to avoid FOUT reflow.

```html
<!-- CLS-safe responsive image -->
<img src="/hero-1200.webp" width="1200" height="675"
     style="aspect-ratio: 16/9; width:100%; height:auto"
     fetchpriority="high" alt="...">
```

---

## 2. LCP optimization playbook

```html
<!-- In <head>: make the LCP image discoverable + prioritized BEFORE the parser reaches <img> -->
<link rel="preload" as="image" href="/hero-1200.webp" fetchpriority="high">
```
```html
<!-- The LCP <img> itself: high priority, NOT lazy -->
<!-- decoding="async" is generally fine; browsers with Decode Queue optimizations
     (Chrome 120+) no longer delay LCP paint for async decode -->
<img src="/hero-1200.webp" width="1200" height="675" fetchpriority="high"
     loading="eager" decoding="async" alt="Product hero">
```

Rules:
- **NEVER `loading="lazy"` on the LCP image** or anything above the fold. Lazy-load everything below the fold.
- **`fetchpriority="high"`** on the LCP element or its preload link is the highest-impact single-line fix.
- The LCP resource must be in the **initial HTML** for the preload scanner. CSS `background-image` and JS-injected images are invisible to the scanner — if you must use a CSS background hero, add `<link rel="preload" as="image">`.
- **Cut TTFB:** serve from CDN/edge, cache SSR/SSG output, avoid slow synchronous origin work. TTFB is ~40% of the LCP budget.
- **Prefer SSR/SSG** for content sites: client-side rendering forces JS execution before the LCP element can paint, adding unavoidable render-blocking delay. Astro islands / Next SSG / static HTML win here. (See [Frameworks for Design-Forward Sites](../04-libraries-and-tools/frameworks-for-design.md).)
- **Eliminate render-blocking resources:** inline critical CSS, defer non-critical CSS (`media="print" onload`), `defer`/`async` non-critical JS.
- **Preconnect** to required third-party origins early: `<link rel="preconnect" href="https://cdn.example.com" crossorigin>`.

---

## 3. INP optimization playbook

**Break up long tasks.** Any main-thread task >50ms is a "long task" and inflates input delay.
```js
// Yield between chunks so user input can interrupt and paint
async function processList(items) {
  for (const item of items) {
    doWork(item);
    if (navigator.scheduling?.isInputPending?.() || shouldYield()) {
      await scheduler.yield();           // modern; falls back below
    }
  }
}
// Fallback when scheduler.yield is unavailable:
const yieldToMain = () => new Promise(r => setTimeout(r, 0));
```

- **Render the visual response first, defer the heavy work.** Update the UI (e.g., toggle a class) inside the handler, then `await` a yield and do analytics/network/expensive computation after the paint.
- **Avoid layout thrashing:** never read layout (`offsetWidth`, `getBoundingClientRect`, `scrollTop`) right after writing styles in the same task — it forces synchronous reflow. Batch reads, then writes (read → write pattern; consider `requestAnimationFrame`).
- **Shrink the DOM and use `content-visibility: auto`** on off-screen sections to skip their layout+paint. Large DOMs scale rendering cost non-linearly, hurting presentation delay.
- **Debounce/throttle** high-frequency handlers; keep event callbacks lean.
- **Audit third-party JS** — analytics, ad pixels, chat widgets eat main-thread time and worsen INP, especially on high-traffic complex sites.

```css
/* Skip rendering work for below-the-fold sections until near viewport */
.below-fold-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 600px; /* prevents scrollbar jump / CLS */
}
```

---

## 4. Image & font optimization

**Images** (usually the LCP element and the largest byte cost):
- **Resize at the source** to max display width (typically ≤1200px for blog/content images) *before* upload. Don't expect plugins to fix an oversized original. A 1200px WebP at quality ~80 lands ~60–90KB.
- Serve **modern formats**: AVIF (best ratio) then WebP, with a fallback. Use `<picture>` for art direction / format negotiation.
- Use `srcset`/`sizes` for responsive delivery; let the browser pick the right resolution.
- Always set `width`/`height` (CLS). Lazy-load below-fold only.
```html
<picture>
  <source srcset="/img.avif" type="image/avif">
  <source srcset="/img.webp" type="image/webp">
  <img src="/img.jpg" width="1200" height="800" loading="lazy" decoding="async"
       sizes="(max-width:600px) 100vw, 1200px" alt="...">
</picture>
```
Always include `sizes` in `srcset` usage — omitting it makes the browser assume 100vw and download oversized images on desktop.

**Fonts:**
- **WOFF2 only** — WOFF2 is typically 30% smaller than TTF/OTF and has ~97% global browser support (2026). A WOFF1 fallback is unnecessary; drop it.
- **Subset** to the character ranges you actually use: full Latin ~200KB → properly subsetted <20KB.
- **Self-host** fonts to avoid DNS lookup + TLS handshake + connection setup to a third-party font CDN on every visit.
- **`font-display: swap`** for body text (prevents invisible text / FOIT). **`font-display: optional`** for non-critical fonts (skips loading on slow connections, avoids reflow).
- **Preload** the critical font used above the fold: `<link rel="preload" as="font" type="font/woff2" href="/fonts/inter-subset.woff2" crossorigin>`.
- Match fallback metrics (`size-adjust`, `ascent-override`) to minimize swap-induced CLS. (See [Web Typography Systems](../00-foundations/typography-systems.md).)

---

## 5. Critical CSS, lazy loading, code splitting, reducing JS

**Critical CSS:** inline the styles needed to render above-the-fold content in `<head>`; load the rest non-blocking.
```html
<style>/* critical above-the-fold rules only */</style>
<link rel="stylesheet" href="/full.css" media="print" onload="this.media='all'">
```

**Lazy loading:** `loading="lazy"` on below-fold images/iframes only. Use the facade pattern for heavy embeds (YouTube, Intercom, Drift, maps) — render a lightweight placeholder and load the real widget only on user interaction.

**Code splitting & reducing JS** (the biggest INP/TBT lever):
- **Tree-shake:** prefer named ES module imports. `import { debounce } from 'lodash-es'` ships one function; `import _ from 'lodash'` (CommonJS) forces the whole library in. Tree shaking + code splitting commonly cuts bundles 50–70%.
- **Route/component-level dynamic `import()`** so users download only what the current view needs.
- **Avoid over-transpilation:** shipping ES5 to modern browsers bloats output (one ES6 arrow line can expand to 11+ ES5 lines). Target modern baselines; use differential serving if needed.
- **Islands / partial hydration** (Astro, etc.): ship zero JS by default, hydrate only interactive components — near-perfect scores for content-heavy sites vs SPA approaches.
- **`<link rel="modulepreload">`** for critical code-split ES modules: unlike `preload`, `modulepreload` also parses and compiles the module, not just downloads it — measurably faster first interaction on module-heavy routes.

---

## 6. Caching, CDN & preloading

**Cache-Control:**
```
# Content-hashed static assets (file name changes when content changes)
Cache-Control: public, max-age=31536000, immutable

# HTML documents (must always revalidate, but may cache)
Cache-Control: no-cache

# API responses that tolerate slight staleness
Cache-Control: max-age=0, s-maxage=60, stale-while-revalidate=300
```
- `immutable` tells browsers to skip revalidation for the asset's lifetime — only safe with content hashing.
- `s-maxage` applies to **shared CDN caches** only, overriding `max-age` there while the browser still obeys `max-age`.
- **Stale-While-Revalidate:** serve from cache instantly, refresh in the background — great for tolerant API/data.

**CDN/edge:** put static assets and cacheable HTML on a CDN to slash TTFB (a major LCP component). Compress with Brotli (better than gzip for text, typically 15–20% smaller than gzip at comparable speed).

**Service Worker caching (repeat visits):** a Service Worker precache makes repeat visits instant — assets are served from the local cache before any network request. Use `workbox` for precaching content-hashed assets and runtime caching strategies. Combine with Cache-Control headers; the SW is the fast path on repeat, headers are the fallback for first visit.

**Resource hints (order of aggressiveness):**
- `preload` — fetch a critical resource now (LCP image, critical font). Use sparingly; over-preloading contends for bandwidth and can delay the LCP resource itself.
- `modulepreload` — for ES module scripts: downloads + parses + compiles. Prefer over `preload` for critical JS modules.
- `preconnect` — open the connection (DNS+TLS) to a required third-party origin early. Add `crossorigin` on font preconnects (CORS required): `<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>`. Do not preconnect to more than 2–3 origins; each held-open connection costs ~50KB CWND memory.
- `dns-prefetch` — resolve DNS only; cheap hint for likely-needed but not certain origins.
- `prefetch` — low-priority fetch of a resource likely needed for the *next* navigation.

---

## 7. Animation performance (60fps, GPU compositing)

**Budget: 60 FPS = 16.66ms per frame.** Miss it and frames drop (jank).

The browser pipeline is **Layout → Paint → Composite**. Cheap animations skip the first two:
- **`transform` and `opacity` are GPU-composited** — they skip layout and paint, run on the compositor thread (separate from the main JS thread), so they keep moving even if JS is busy.
- **`width/height/top/left/margin/padding` trigger layout → paint → composite** — expensive, drop frames. Never animate them.

| Want to animate | Do this (cheap) | Not this (jank) |
|---|---|---|
| Move | `transform: translate()` | `top` / `left` / `margin` |
| Resize | `transform: scale()` | `width` / `height` |
| Show/hide | `opacity` | `visibility` + reflow, `display` toggles |

```css
/* Promote to its own compositor layer ONLY while animating, then remove */
.card:hover { will-change: transform; }
.card { transition: transform .2s ease; }
.card:hover { transform: translateY(-4px); }
```
- **`will-change: transform` sparingly.** Each promoted element gets its own compositor layer consuming VRAM. Mass application → "layer explosion" → mobile stuttering. Add right before animating, remove after.
- Respect `prefers-reduced-motion: reduce` — disable non-essential motion.
- Prefer the **Web Animations API** / CSS transitions over JS per-frame style writes; if using a library, prefer transform/opacity-based ones. (See [Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md), [Micro-interactions & Motion](../07-aesthetics-and-trends/micro-interactions-and-motion.md).)

---

## 8. Perceived performance

Perceived speed often matters as much as measured speed.
- **Skeleton screens** make pages feel faster than spinners — they communicate structure and progress, reducing perceived wait-time uncertainty. Original research (Wroblewski et al.) reported ~20% faster perception; the precise uplift varies by context but the direction is consistent.
- **Optimistic UI:** apply the expected result immediately (e.g., mark "liked"), reconcile/rollback when the server responds — interactions feel instant.
- **Progress bars** that accelerate early and decelerate late feel faster than constant-speed bars.
- **Stream content** (SSR streaming, progressive rendering) so users see meaningful pixels ASAP rather than a blank screen.
- **Instant feedback** on every interaction (<100ms visual acknowledgement) keeps the action feeling user-caused. (See [Interaction Design & UI States](../01-ux-fundamentals/interaction-design-and-states.md), [Cognitive Load & Progressive Disclosure](../01-ux-fundamentals/cognitive-load-progressive-disclosure.md).)

---

## 9. Measurement: lab vs field

**Lab (synthetic, reproducible, for regression catching):**
- **Lighthouse** (DevTools + CLI): simulated throttled environment, lab data only. Run in **incognito with extensions disabled** (extensions skew scores). Run 3–4× and average — scores vary ±5–10 points on the same page.
- **Standard Lighthouse does NOT measure INP** — it reports **Total Blocking Time (TBT)** as a proxy. Improving TBT usually helps INP but isn't a guarantee.
- **WebPageTest** — deep waterfall, filmstrip, multi-location/device testing, connection profiling.

**Field (real users, ground truth for CWV pass/fail):**
- **CrUX (Chrome User Experience Report):** real Chrome users who opted into reporting. This is what Search Console grades. Access via **PageSpeed Insights** (lab+field combined) and **Search Console → Core Web Vitals report**.
- **RUM (Real User Monitoring):** your own field data via the `web-vitals` library or tools like Sentry Performance / Datadog RUM. Required for real INP debugging — the **Long Animation Frames (LoAF) API** data needed to attribute INP (which scripts caused the long task, which elements were affected) only comes from RUM, not Lighthouse. The `web-vitals` library v4+ exposes LoAF attribution via `onINP({ attribution })` — the `attribution.longAnimationFrameEntries` array shows exactly which scripts contributed to each slow interaction.

**Critical rule: lab ≠ field.** Lighthouse 90 (or even 100) does NOT guarantee passing CWV in Search Console. Always confirm with CrUX/RUM at p75.

```js
// Minimal RUM with the web-vitals library (v4+)
import { onLCP, onINP, onCLS } from 'web-vitals/attribution';
const send = (m) => navigator.sendBeacon('/rum', JSON.stringify(m));
// attribution variant captures LoAF data for INP debugging
onLCP(send); onINP(send, { reportAllChanges: false }); onCLS(send);
```

---

## 10. Performance budgets & CI enforcement

Performance degrades by a thousand cuts: +50KB animation lib, +100KB marketing tracker, +500KB unoptimized hero. Each looks small; cumulatively they sink scores. Budgets with **CI enforcement** are the only durable defense.

- **JS budget: ≤400KB gzipped** for interactive pages (≤200KB for content/marketing pages). The 400KB ceiling is widely cited from HTTP Archive analysis of correlation between JS weight and INP failure rates; tighter is always better.
- Enforce in CI with **Lighthouse CI** or **bundlesize** in GitHub Actions: fail the build when the main bundle exceeds the threshold or Lighthouse perf score drops below a floor (e.g., 90). Note: Lighthouse CI budget files compare **transfer size** (compressed), not parsed size — ensure your budget numbers match what you intend to measure.
- Track byte budgets per resource type (JS / CSS / images / fonts / total) and per route.
- **Audit third-party scripts quarterly** (analytics, ad pixels, chat, social embeds). HTTP Archive data consistently shows third-party JS as a top INP contributor — especially chat widgets and ad tags that execute on the main thread after load. Use facades for heavy embeds.

```yaml
# .github/workflows/lhci.yml (sketch)
- run: npm run build
- run: npx @lhci/cli autorun
  # lighthouserc.json assert: categories.performance >= 0.9,
  # resource-summary.script.transferSize <= 400000  ← transfer (compressed) bytes
```

---

## 11. The performance-to-conversion link

Performance is a marketing/revenue lever, not just an engineering metric. (Pair with [Conversion Rate Optimization](./conversion-rate-optimization.md).)
- **Load time vs conversion (observational):** Portent's 2019 study found e-commerce sites loading in 1s converted ~2.5–3× higher than those loading in 5s. These are correlational, not controlled — but directionally consistent across multiple industry analyses.
- **Mobile abandonment:** Google/SOASTA research found that as page load time goes from 1s to 3s, the probability of bounce increases 32%; from 1s to 5s, 90% (Google, 2017). Frequently cited as "53% abandon at >3s" but the actual data is a probability increase, not a single-stat threshold.
- **INP affects conversion:** Contentsquare's 2023 Digital Experience Benchmark found pages with good INP had materially better engagement and conversion than pages with poor INP. Treat as directional; no single clean percentage holds across all site types.
- **Case studies (Google web.dev):** **Rakuten 24** +53.4% revenue-per-visitor after CWV work; **Carpe** (LCP −52%, CLS −41%) → +10% traffic, +5% conversion, +15% revenue; **Relive** (LCP −50%+, CLS ≈0) → +3% conversion, −6% bounce, +9% pageviews/session. Source: https://web.dev/case-studies/
- **Mobile is the majority and the harder case:** ~62% of global web traffic is mobile (StatCounter 2024+), yet mobile conversion rates run roughly half of desktop — mobile performance is where the gap and opportunity live.

---

## Concrete specs & numbers
- **LCP:** Good ≤2.5s · NI 2.6–4.0s · Poor >4.0s — at p75.
- **INP:** Good ≤200ms · NI 201–500ms · Poor >500ms — at p75. Qualifying interactions: click, tap, keypress (not scroll/hover/zoom). >50 interactions → drop 1 outlier per 50.
- **CLS:** Good ≤0.1 · NI 0.1–0.25 · Poor >0.25 — at p75. ≥0.15 reads as disruptive.
- **Frame budget:** 16.66ms (60 FPS). Long task = >50ms on the main thread.
- **Cheap-to-animate:** `transform`, `opacity` (composited). Expensive: `width/height/top/left/margin/padding`.
- **WOFF2** ~30% smaller than TTF/OTF; ~97% browser support (2026) — no WOFF1 fallback needed. Subsetted font <20KB vs full Latin ~200KB.
- **Content image:** resize ≤1200px source; WebP q80 ≈ 60–90KB.
- **JS budget:** ≤400KB gzipped (interactive pages). Tree-shaking + code splitting: −50–70%.
- **Immutable cache:** `public, max-age=31536000, immutable` (31,536,000s = 1 year) for content-hashed assets.
- **State of the web (HTTP Archive Web Almanac 2024):** ~43% mobile / ~55% desktop origins pass all 3 CWV; LCP is the most-failed metric on mobile; INP pass rate is high overall but drops significantly on JS-heavy and top-1,000 sites. Check https://almanac.httparchive.org for the current year's numbers — these shift quarterly.
- **Third-party JS:** HTTP Archive data shows third-party JS consistently ranks among the top contributors to INP failures; treat any third-party script as a suspect in INP regressions until profiled.
- **Skeleton screens:** reported ~20–30% perceived-speed gain vs spinners in UX studies (Luke Wroblewski, 2013 original research; replicated directionally in subsequent studies). The precise number varies by context; the qualitative benefit is well-established.

## Tools & libraries
- **Lighthouse / Lighthouse CI** — lab regression testing in CI. Use for PR gating; do NOT treat as CWV pass/fail. No real INP (uses TBT proxy).
- **PageSpeed Insights** — lab + CrUX field in one view. Use to see real-world status per URL/origin.
- **Google Search Console → Core Web Vitals** — official field grading across your whole site; the source of truth for ranking impact.
- **WebPageTest** — deep diagnostics: waterfalls, filmstrips, multi-device/location, connection throttling.
- **web-vitals library (v4+)** — drop-in RUM for LCP/INP/CLS; use the `/attribution` export for LoAF-backed INP debugging. The library correctly handles the browser's 104ms PerformanceEventTiming threshold to capture all interactions. Use in production.
- **Long Animation Frames (LoAF) API** — browser API (Chrome 123+) that attributes slow frames to specific scripts. The `web-vitals` attribution module exposes this automatically via `attribution.longAnimationFrameEntries`.
- **Sentry Performance / Datadog RUM** — managed RUM with INP/LoAF attribution. Use when you need dashboards/alerting beyond raw beacons.
- **bundlesize** — per-file gzipped byte budgets in CI. Use alongside Lighthouse CI.
- When NOT to use: don't ship a heavy RUM/analytics stack to chase metrics if it itself adds enough JS to hurt INP — measure its own cost.

## Do / Don't
**Do**
- Make the LCP image discoverable in HTML and give it `fetchpriority="high"`.
- Set explicit dimensions / `aspect-ratio` on all media and ad slots.
- Self-host subsetted WOFF2 fonts with `font-display: swap`; preload the critical one.
- Animate only `transform`/`opacity`; respect `prefers-reduced-motion`.
- Break long tasks with `scheduler.yield()`; render visual feedback before heavy work.
- Tree-shake, code-split, prefer SSR/SSG/islands; ship modern JS.
- Cache content-hashed assets immutably; serve from a CDN; compress with Brotli.
- Enforce budgets in CI and confirm pass/fail with CrUX/RUM at p75.

**Don't**
- Lazy-load the LCP image or anything above the fold.
- Inject the hero via JS or CSS `background-image` without a preload link.
- Animate `width/height/top/left/margin/padding`; don't slap `will-change` on everything.
- Read layout properties right after writing styles (layout thrashing).
- Trust a Lighthouse score as proof of passing Core Web Vitals.
- Import whole libraries (`import _ from 'lodash'`) when named ES imports work.
- Add third-party scripts without measuring their main-thread and byte cost.
- Insert content above existing content after load without reserving space.

## Common pitfalls & how to avoid them
- **Lazy hero image** → LCP tanks. Use `eager` + `fetchpriority="high"` + preload.
- **CSS `background-image` / JS-injected hero** → invisible to preload scanner, always late. Use `<img>` or add `<link rel="preload" as="image">`.
- **"Lighthouse says 95, why is Search Console red?"** → lab ≠ field; you only fixed the median, not p75; or INP (unmeasured by Lighthouse) is the failure. Check CrUX/RUM.
- **Layout shift from late content** (ads, banners, web fonts, lazy images without dimensions) → reserve space; set dimensions; size-match fallback fonts.
- **Janky scroll/hover animations** → animating layout properties. Switch to `transform`/`opacity`.
- **`will-change` everywhere** → layer explosion, mobile VRAM exhaustion, stutter. Apply per-animation, remove after.
- **Huge JS from one library** → CommonJS default import defeats tree-shaking. Use ES module named imports and dynamic `import()`.
- **Third-party creep** → quarterly tracker added "just 100KB" each. Budget + facade pattern + CI gate.
- **Over-transpilation** → ES5 output bloat for users on modern browsers. Target modern baselines.

## What real users complain about
- "The button does nothing when I tap it" — high INP: the handler is queued behind a long task; nothing paints for hundreds of ms, so users double-tap (and trigger duplicate actions).
- "It jumped while I was reading / I clicked the wrong thing" — CLS from a late image, ad, or banner pushing content down mid-interaction.
- "Text was invisible, then flashed in" — FOIT from `font-display: block`/default; or visible reflow when the web font swaps with mismatched metrics.
- "It looked loaded but I couldn't do anything" — hydration gap in SPAs: pixels are there but JS hasn't wired up handlers; clicks are dropped.
- "Spinner forever" — no skeleton, no progress signal, so any wait feels broken; users abandon at ~3s on mobile.
- "Fine on my laptop, unusable on my phone" — desktop-only testing hides mobile CPU/network reality where most traffic and the failures live.

## Senior-level checklist
- [ ] LCP element identified; it's an `<img>` discoverable in initial HTML, `fetchpriority="high"`, not lazy, preloaded if a background/font.
- [ ] TTFB measured and acceptable (CDN/edge, cached SSR/SSG); render-blocking CSS/JS minimized; critical CSS inlined.
- [ ] All images/videos/iframes/ad slots have explicit `width`/`height` or `aspect-ratio`; below-fold media lazy-loaded.
- [ ] Fonts: self-hosted, subsetted, WOFF2, `font-display` set, critical font preloaded, fallback metrics matched.
- [ ] No long tasks >50ms on key interactions; handlers yield; visual feedback paints first; no layout thrashing.
- [ ] All animations use `transform`/`opacity`; `will-change` is scoped and removed; `prefers-reduced-motion` honored.
- [ ] JS tree-shaken, code-split, modern-targeted; main bundle ≤400KB gzipped; SSR/SSG/islands where appropriate.
- [ ] Caching: content-hashed assets `immutable`; HTML `no-cache`; CDN + Brotli; SWR where data tolerates it; Service Worker precache for repeat-visit instant loads.
- [ ] Third-party scripts inventoried; heavy embeds behind facades; quarterly audit scheduled.
- [ ] CI enforces Lighthouse perf floor + byte budgets; builds fail on regression.
- [ ] **Field-verified:** CrUX/RUM shows LCP ≤2.5s, INP ≤200ms, CLS ≤0.1 at p75 on real mobile + desktop traffic.

## Sources
- web.dev — Core Web Vitals, LCP, INP, CLS guides and thresholds: https://web.dev/articles/vitals · https://web.dev/articles/lcp · https://web.dev/articles/inp · https://web.dev/articles/cls
- web.dev — Optimize LCP / INP / CLS: https://web.dev/articles/optimize-lcp · https://web.dev/articles/optimize-inp · https://web.dev/articles/optimize-cls
- Chrome for Developers — INP replaces FID (March 2024): https://web.dev/blog/inp-cwv-march-2024
- Chrome for Developers — Long Animation Frames API: https://developer.chrome.com/docs/web-platform/long-animation-frames
- HTTP Archive Web Almanac (Performance / CWV chapter, updated annually): https://almanac.httparchive.org/
- Chrome UX Report (CrUX): https://developer.chrome.com/docs/crux
- web.dev case studies (performance → business metrics): https://web.dev/case-studies/
- PageSpeed Insights: https://pagespeed.web.dev/
- Lighthouse CI: https://github.com/GoogleChrome/lighthouse-ci
- web-vitals library v4: https://github.com/GoogleChrome/web-vitals
- WebPageTest: https://www.webpagetest.org/
- Google/SOASTA "The State of Online Retail Performance" (2017 bounce/load study): https://www.thinkwithgoogle.com/marketing-strategies/app-and-mobile/mobile-page-speed-new-industry-benchmarks/

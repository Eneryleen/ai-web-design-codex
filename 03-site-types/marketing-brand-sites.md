# Marketing & Brand Sites

> A marketing/brand site is a multi-page property whose job is to express identity, build trust, and convert — simultaneously. Unlike a single-purpose landing page, it carries the whole brand: homepage, product/feature pages, pricing, story, proof, and a coherent design system across all of them. This guide gives an AI builder the homepage strategy, narrative arc, multi-page IA, brand-as-system tokens, motion-as-brand discipline, and the Awwwards-polish-vs-usability balance needed to ship agency-grade work that still passes Core Web Vitals.

**Apply when:** building a company/product brand site (homepage + IA, not a single campaign page). · **Related:** [Landing Pages](./landing-pages.md) · [Scroll-Driven Experiences](./scroll-driven-experiences.md) · [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md)

## TL;DR — rules at a glance
- **First impression forms in ~50ms.** Brand and clarity must be instantaneous — communicate who you are + what you do in ≤3s. No "reveal over time."
- **The fold is asymmetric:** 57% of viewing time is above the fold, 74% within the first two screenfuls (at a 1080px viewport = 0–2160px), top 20% of the page captures 42%+ of attention. Front-load value. (NNg eyetracking, 130K+ fixations, 120 participants, 1920×1080 monitors.)
- **Avoid the Illusion of Completeness.** A full-screen hero with no visible cut-off makes users believe the page ends there — NNg qualitative studies consistently show the majority of users do not scroll past a full-screen hero with no peek. Let content peek above the fold.
- **The visual language of modern brand sites:** high contrast + generous whitespace + monochrome base + ONE accent color used sparingly. "Take the spacing that feels like enough, then double it."
- **Coherence is systemic.** High contrast needs whitespace or it feels aggressive; monochrome needs sharp type or it feels monotone. Tune the whole, not parts.
- **Customer is the hero, product is the mentor.** Narrative arc: problem (setup) → tension (conflict) → transformation (resolution). Brand ≠ "we are great"; brand = "here's who you become."
- **Brand is a system, not a page.** Design tokens (primitive → semantic) keep 5+ pages identical in type scale, spacing, color, motion. Centralize tokens before touching components.
- **Motion must earn its payload.** Animate the *product claim* (Linear = speed, Vercel = build velocity), never decorate. Decorative motion = perf cost, zero conversion return.
- **Don't segment primary nav by audience** ("For developers / For enterprise"). Multi-fit users get confused; everyone feels they must check all segments.
- **Hamburger on desktop is wrong.** Hidden nav robs users of scope/orientation. Expose the IA; always show the active state.
- **No hero carousels, no autoplay-without-control, no on-load popup.** Each is a documented conversion/usability killer (slide-2 CTR ≈ 0; WCAG forbids >5s motion w/o pause).
- **Performance IS brand.** LCP ≤2.5s, INP ≤200ms, CLS <0.1. A perf product on a slow site is a broken promise. A 1s delay ≈ −7% conversion.
- **Lead with font-weight, not font-size,** for hierarchy. Use a modular scale (Perfect Fourth 1.333) and a single coherent type system.
- **CTA = call-to-value, repeated every scroll-height.** "Start building" > "Learn more." Vague CTAs are the #1 hero mistake.
- **Dark mode is a token problem.** Define semantic tokens with light/dark values upfront in `@media (prefers-color-scheme: dark)`. Retrofitting dark mode into a hardcoded codebase is expensive and error-prone.
- **Brand voice extends to every string.** Button labels, error messages, empty states, and tooltips must sound like the same person as the homepage headline. Robotic error copy on a witty brand = fractured identity.
- **Social sharing is a brand surface.** Every URL shared is an impression: use page-specific OG images (1200×630), unique `<title>`/`<meta description>`, and a complete favicon set (32px ICO + 180px Apple + 192/512 PWA).

## What a brand site is (and is not)

| | Landing page | Brand / marketing site |
|---|---|---|
| Goal | ONE conversion | Express identity + convert across journeys |
| Scope | 1 page | Homepage + multi-page IA |
| Nav | Stripped | Full, exposed, persistent |
| Audience | One segment per page | Multiple, served without per-audience nav splits |
| Success metric | Conversion rate of that page | Brand recall, dwell, return rate, assisted conversions |
| Design system | Optional | Mandatory — consistency *is* the brand |

A brand site fails differently from a landing page: it fails by being *inconsistent*, *forgettable*, or *slow*, not just by under-converting one funnel. Optimize for identity + trust + craft *and* conversion, not conversion alone.

## Homepage strategy

The homepage is the single most valuable surface — Linear's growth is a widely-cited example of homepage-driven, word-of-mouth acquisition with minimal paid spend (Linear is private; ARR figures are community estimates, not disclosed). Treat it as the primary growth surface, not a brochure.

**NNg's 5 homepage principles (apply verbatim):**
1. **Easy access home from every page** — logo (top-left) links home + an explicit nav item. Logo-as-home is learned behavior; honor it.
2. **Communicate who you are + what you do in ≤3s.** The H1 is the contract. If a stranger can't restate your value after the hero, rewrite it.
3. **Reveal content through examples, not category links.** Show real product screens, real outcomes — not "Solutions →" stubs.
4. **Prompt clear next actions.** One primary CTA + at most a secondary. Don't make the user invent the next step.
5. **Keep it simple.** Every added element competes for the 57% of attention spent above the fold.

### Hero (above the fold)
- **Max 3 clickable items; ideally one primary CTA.** Wanting to do too much in the hero is the #1 mistake.
- **65%+ of above-fold attention sits in the top half.** Put H1, value prop, primary CTA in the **upper-center**, not lower-center.
- **Z-pattern scan:** top-left (logo) → top-right (nav/secondary CTA) → diagonal → bottom-right. End the above-fold block with the CTA in the bottom-right of the Z.
- **Show real product, not stock.** Real product visuals in the hero is one of three traits shared by brand sites converting visitor→trial >4% (alongside pricing visible without a sales call, and CTA repeated every scroll-height).
- **Let content peek.** Reserve the bottom ~10–15% of the viewport for a sliver of the next section so users know to scroll. Avoid the Illusion of Completeness.

```html
<!-- Hero: one job, exposed nav, peeking content, value-driven CTA -->
<header class="site-nav"> <!-- persistent, exposed, NOT a desktop hamburger -->
  <a href="/" class="logo" aria-label="Home">Acme</a>
  <nav aria-label="Primary"> <!-- by feature/use, NOT by audience type -->
    <a href="/product" aria-current="page">Product</a>
    <a href="/pricing">Pricing</a>
    <a href="/customers">Customers</a>
    <a href="/docs">Docs</a>
  </nav>
  <a class="btn-primary" href="/signup">Start building</a> <!-- call-to-value -->
</header>

<section class="hero"> <!-- ~85vh, not 100vh: leave a peek -->
  <h1>Ship features your competitors can't catch.</h1>
  <p class="sub">Deploy-on-push CI for product teams. No YAML, no waiting — live in 8 minutes.</p>
  <a class="btn-primary" href="/signup">Start building — no card required</a>
  <img src="/product.avif" alt="Acme build dashboard showing a 12s deploy" loading="eager" fetchpriority="high" width="1200" height="720">
</section>
```

### Hero copy framework (Zoom/Slack/Drift school)
- **Headline** = the specific transformation the customer gets that *only your brand* delivers. Speak to desires/fears; nothing vague.
  - Bad: `Empower your workflow.` / `The future of collaboration.`
  - Good: `Close the books in a day, not a week.` / `Ship features your competitors can't catch.`
- **Subheadline** = what you do + who you help + how + ONE handled objection. ("…for product teams. No YAML.")
- **CTA** = call-to-value, not call-to-action. `Start your free trial — no card required` > `Sign up`.

## Brand expression as a system

Consistency *is* the brand. The same type scale, spacing rhythm, color, radius, and motion easing must hold across homepage, product, pricing, and story pages. The mechanism is **design tokens** in two layers:

```css
:root {
  /* 1. PRIMITIVE tokens — raw values, no meaning */
  --blue-500: #3b82f6;
  --neutral-0: #ffffff;
  --neutral-950: #0a0a0a;
  --space-1: 0.25rem; --space-2: 0.5rem; --space-4: 1rem; --space-8: 2rem;
  --ease-brand: cubic-bezier(0.16, 1, 0.3, 1); /* signature easing */

  /* 2. SEMANTIC tokens — meaning, reference primitives */
  --color-brand-primary: var(--blue-500);
  --color-bg: var(--neutral-0);
  --color-text: var(--neutral-950);
  --color-text-muted: color-mix(in oklch, var(--neutral-950) 60%, transparent);
  --section-gap: clamp(4rem, 10vw, 9rem); /* generous, fluid vertical rhythm */
}
```

- **Why two layers:** swap `--blue-500` for a rebrand and every semantic reference updates; components never hardcode hex. This is how 5+ pages stay identical.
- **One accent, used sparingly.** Monochrome base (neutrals) + a single accent reserved for CTAs, active states, and key emphasis hits harder than five colors scattered. Stripe's multi-color gradient is a tightly-controlled proprietary identifier — a deliberate anomaly that works because every other variable is disciplined. Don't replicate the rainbow; replicate the discipline.
- **Distribution:** pipe tokens to CSS vars (+ Swift/Android/Sass if multi-platform) via Style Dictionary; sync from Figma via Token Studio. W3C Design Tokens Format Module (2024) standardizes the JSON.
- **Dark mode is a token problem, not a CSS problem.** Every semantic token must have a light and dark value. Do this upfront with `@media (prefers-color-scheme: dark)` on semantic tokens — retrofitting dark mode into a flat-hex codebase is days of work:

```css
:root {
  --color-bg: var(--neutral-0);
  --color-text: var(--neutral-950);
}
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: var(--neutral-950);
    --color-text: var(--neutral-0);
  }
}
/* Components reference --color-bg/--color-text. Zero changes needed. */
```

See [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md) and [Color Theory & Color Systems](../00-foundations/color-theory-and-systems.md).

### Typography as brand voice
- **Lead with weight, not size.** Experienced typographers establish hierarchy through font-weight contrast first; size second. A 600/400 weight pair reads as hierarchy even at one size.
- **Modular scale.** Perfect Fourth (1.333) is the default for marketing sites; Fibonacci steps (16–24–40–64–104px) work for dramatic display contrast.
- **Fluid type with `clamp()`,** but validate against WCAG 1.4.4 — text must remain readable and scale to 200% at 500% browser zoom; a hard `vw`-only clamp can break this. Always include a `rem` term.

```css
/* Perfect Fourth via clamp — has a rem term so zoom still scales it */
--text-base: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
--text-h1:   clamp(2.5rem, 1.8rem + 3.5vw, 4.5rem);
h1 { font-size: var(--text-h1); font-weight: 600; line-height: 1.05; letter-spacing: -0.02em; }
body { font-size: var(--text-base); line-height: 1.55; } /* 1.5–1.6, relative units, not px */
```

- **Line-height in relative units** (unitless/em/%), never px — longer measures need more leading. Body 1.5–1.6; tight display headings 1.0–1.1.
- **Variable fonts** (`woff2` with `@font-face` `font-weight: 100 900`) replace 4–6 static weight files with one request — cut font transfer size by ~60% for multi-weight brand type. Gate on `@supports (font-variation-settings: normal)` for old browsers.
- **Font loading pipeline:** `<link rel="preconnect" href="https://fonts.googleapis.com" crossorigin>` + `font-display: optional` (no FOUT, worst-case renders system font on first visit) for body text; `font-display: swap` + metric-matched `size-adjust` fallback only if branding requires the exact typeface on first paint. Never `font-display: block` — it holds rendering up to 3s.

```css
/* Metric-matched fallback prevents CLS when custom font swaps in */
@font-face {
  font-family: 'Brand Sans';
  src: url('/fonts/brand-sans-vf.woff2') format('woff2 supports variations'), url('/fonts/brand-sans-vf.woff2') format('woff2');
  font-weight: 100 900;
  font-display: optional;
}
```

- See [Web Typography Systems](../00-foundations/typography-systems.md) and [Fluid Typography & Spacing](../02-responsive-adaptive/fluid-type-and-spacing.md).

## Narrative & storytelling

Web design is behavioral engineering, not decoration. A beautiful site that doesn't guide action has failed its primary job. Story is the engine that moves people through the page.

- **Casting:** customer = hero, brand/product = mentor (the guide who hands the hero a tool). Never cast yourself as the hero.
- **Arc:** **setup** (the customer's problem) → **conflict** (the tension/cost of inaction) → **resolution** (the transformation your product enables).
- **Payoff:** narrative-first positioning correlates with higher engagement, more backlinks, and longer dwell time — the mechanism is that story gives readers something to retell, which drives share and return. No clean controlled study exists; treat as directional practitioner consensus, not a hard number.

### Scroll narrative (homepage as a journey)
Max **5 scenes**; total visitor journey **≤2–3 minutes**. Map each scene to a scroll band:

| Band | Stage | Job |
|---|---|---|
| 0–15% | **Hook** | 3s to hook — bold H1 + real product + a single unexpected motion beat |
| 15–35% | **Context** | Who is this brand, what problem exists (agitation) |
| 35–70% | **Journey** | Features as *progression* — each builds on the last |
| 70–85% | **Climax** | Emotional/insight peak — the "aha" or headline proof point |
| 85–100% | **Resolution** | CTA, pricing link, contact, next steps |

This is brand-site narrative, not a full pinned scroll experience — for heavy pinning/parallax mechanics see [Scroll-Driven Experiences](./scroll-driven-experiences.md).

## Multi-page IA & navigation

**IA ≠ navigation.** IA is the conceptual backbone (how content is organized); navigation is its UI expression. Get the IA right first (card sorting + tree testing) — then design nav.

**Canonical brand-site IA:**
```
/                    Home (identity + primary conversion path)
/product             What it does (or /features, /platform)
  /product/<feature> deep feature pages (reveal via examples)
/pricing             Visible without a sales call (a >4% trial-conversion trait)
/customers           Named proof / case studies (not a generic logo wall)
/about (or /story)   Brand narrative, team, mission
/blog, /docs         Content surfaces (SEO + trust)
/contact, /signup    Conversion endpoints
```

### Navigation rules
- **Expose the IA on desktop.** No hamburger ≥1024px — hidden nav removes scope and orientation cues. Mobile may collapse to a menu.
- **Strong information scent in labels:** specific, descriptive nouns. Avoid `Explore`, `Learn more`, `Click here` as nav items.
- **Always show the active state** — failing to indicate current location is the single most common nav mistake (NNg). Use `aria-current="page"` + a visible style.
- **Do NOT split primary nav by audience type.** "For developers / For enterprise" confuses multi-fit users and pressures everyone to check every segment. Organize by *what the product does*, surface audiences inside pages or via a secondary "Solutions" menu if truly needed.
- **Flat vs deep is a tradeoff, not a winner.** Flat = fewer clicks, more choices per level; deep = more clicks, fewer choices. Validate with tree testing; don't guess.
- **Kill orphan microsites** from old campaigns — they dilute brand authority and SEO. Fold them into the main IA or redirect.

See [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md).

## Motion as brand

Motion is a brand asset *only* when it carries meaning. The test: **does this animation demonstrate a product claim or aid comprehension?** If no → cut it.

- **Earns its payload:** Linear's snappy interactions *show* speed; Vercel's animated build log *shows* deploy velocity; Loom's embedded video *shows* the product. Product-demo video on a landing surface consistently outperforms static images in A/B tests when it reduces cognitive load — because it demonstrates, not decorates. (The "86%" figure widely cited from HubSpot blog posts is derived from aggregated self-reported data; treat as directional, not precise.)
- **Doesn't earn it:** parallax for parallax's sake, full-page WebGL with no narrative tie, entrance animations on every element.
- **Signature easing = brand identity.** Define a `--ease-brand` (or GSAP `CustomEase`) and use it everywhere for a distinctive, consistent feel.

### Implementation discipline (GSAP + Lenis)
```js
import Lenis from 'lenis';          // ~3kB gzipped; v1.0+ (Sept 2024)
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);

const lenis = new Lenis({ syncTouch: true }); // syncTouch, NOT deprecated smoothTouch
gsap.ticker.add((t) => lenis.raf(t * 1000));  // single rAF loop, synced
gsap.ticker.lagSmoothing(0);

// Respect the OS setting — non-negotiable
if (!window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
  gsap.to('.reveal', {
    opacity: 1, y: 0,
    scrollTrigger: { trigger: '.reveal', scrub: 1, anticipatePin: 1 }, // scrub:1 eases lag; anticipatePin:1 for big pins
  });
}
```
- **Always honor `prefers-reduced-motion`.** WCAG + NNg: any motion running >5s needs a pause/stop control; autoplay-without-control is forbidden. Moving elements also get pattern-blindness-ignored as "ads."
- **WebGL/Three.js/R3F:** powerful for 3D brand environments but heavy on mobile — gate behind device capability + reduced-motion, lazy-load, and budget against LCP. See [WebGL, 3D & Shaders](../04-libraries-and-tools/webgl-3d-shaders.md) and [Micro-interactions & Motion](../07-aesthetics-and-trends/micro-interactions-and-motion.md).

## Awwwards-tier polish vs usability

Award-grade craft and conversion are not opposites, but they trade off when polish hides function. Resolve it with a hierarchy: **usable → on-brand → spectacular**, in that order.

- **Polish that helps:** signature easing, real product motion, restrained accent gradient, immaculate spacing rhythm, custom but legible type.
- **Polish that hurts:** scroll-jacking that fights the user's intent, sub-2px low-contrast "elegant" text, 3D that tanks LCP, entrance animations that delay the H1.
- **Litmus:** if removing the effect makes the page *clearer* or *faster* with no loss of meaning, the effect was decoration. Keep effects that demonstrate or orient; cut the rest.
- **The accessibility floor is absolute** even at award tier: contrast minimums, keyboard nav, reduced-motion, focus states. Low-contrast text appears on 79.1% of homepages — the #1 accessibility failure 7 years running (WebAIM Million 2024). Don't join them for the sake of a "muted" aesthetic.

## Content strategy

- **Reveal through examples** (NNg #3): real screenshots, real numbers, real workflows beat abstract category links. Stripe's inline live API previews let developers evaluate integration complexity *on the page* — content as conversion tool.
- **Named, specific proof > generic logo walls.** Linear cites engineers by name at Vercel/Arc/Loom with team sizes; a changelog doubles as a marketing surface proving shipping velocity. Specificity is credibility.
- **Pricing visible without a sales call** — a defining trait of brand sites converting trial >4%. Hiding price reads as "expensive and slow."
- **Blog/docs/changelog are brand surfaces,** not afterthoughts — they carry SEO weight and demonstrate the product's life. See [Content Sites: Blogs & Docs](./content-blogs-and-docs.md) and [Copywriting & UX Writing](../06-marketing-and-conversion/copywriting-and-ux-writing.md).

### Brand voice in microcopy

The headline is not the only copy that carries brand voice. Every microcopy touchpoint must be written in the same register:

- **Button labels:** call-to-value everywhere, not just in the hero. `Create your first project` not `Submit`. `See what's included` not `View pricing`. Vague labels add friction at precisely the moment users are deciding.
- **Error states:** stay human, own the mistake. "Something went wrong on our end — we're on it" (brand owns it) beats "Error 500" (brand abandons the user). Error copy is a trust signal.
- **Empty states:** the first empty state a user sees is a retention moment. "You haven't created anything yet — here's how to start" with an inline action beats a blank table. Treat empty states as onboarding.
- **Tooltips and helper text:** write from user intent ("Why do I need this?"), not system description ("This field accepts...").
- **Voice consistency test:** read the headline, a button label, an error message, and a tooltip aloud in sequence. They should sound like the same person. If the hero is witty but the error is robotic, the brand is fractured.

### Social sharing and identity at every touchpoint

Brand consistency doesn't stop at the site boundary. Every URL that gets shared is a brand impression:

- **Open Graph tags (required on every page):**
```html
<meta property="og:title" content="Page-specific title, not the site name">
<meta property="og:description" content="Specific benefit, not a tagline. ≤155 chars.">
<meta property="og:image" content="https://example.com/og/home.png"> <!-- 1200×630px -->
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta name="twitter:card" content="summary_large_image">
```
- **OG images must be page-specific,** not a generic logo on white. Generate per-page OG images with Vercel OG (`@vercel/og`) or Satori — dynamic OG images are a first-class brand surface.
- **Favicon set:** 32×32 ICO + 180×180 Apple touch icon + 192/512 PWA icons + `site.webmanifest`. A blurry or missing favicon on a brand site is a detail that signals low craft.
- **`<title>` and `<meta name="description">`: every page needs both,** page-specific, ≤60 chars title / ≤155 chars description. Duplicate titles across pages are an SEO and brand signal failure.

## Performance / SEO balance

Performance is a brand statement, not a back-office concern — a perf-product site that scores near-perfect CWV proves its claim before a word is read. The conflict with rich brand visuals is real and must be engineered, not ignored.

- **Targets:** LCP ≤2.5s, INP ≤200ms, CLS <0.1. As of the 2024 CrUX dataset, LCP is the leading failure point — the majority of mobile pages fail it. Overall CWV pass rates vary by dataset; use your own field data (PageSpeed Insights CrUX tab) rather than global averages. Budget for LCP specifically.
- **Business stakes:** 100ms faster ≈ up to +8% conversion; 1s delay ≈ −7% conversion; 3s vs 1s load ≈ +32% bounce. Rakuten's CWV work: +33% conversion, +53% revenue/visitor.
- **Hero is your LCP element** — `fetchpriority="high"`, `loading="eager"`, AVIF/WebP, explicit `width`/`height` (prevents CLS), no JS-blocked render. Never lazy-load the hero image. Use `srcset` + `sizes` to serve appropriately-sized images per viewport — a 1200px image sent to a 390px phone is a 3–5× payload waste and a direct LCP hit:
```html
<img
  src="/hero-800.avif"
  srcset="/hero-400.avif 400w, /hero-800.avif 800w, /hero-1200.avif 1200w, /hero-2400.avif 2400w"
  sizes="(max-width: 640px) 100vw, (max-width: 1024px) 90vw, 1200px"
  alt="Acme dashboard — 12-second deploy"
  width="1200" height="720"
  fetchpriority="high" loading="eager"
>
```
- **Reserve space for everything async** (images, embeds, fonts) to keep CLS <0.1 — set dimensions and `font-display: swap` with a metric-matched fallback.
- **SSR/SSG (Next.js)** for first-paint speed on brand sites; hydrate motion progressively. In Next.js App Router (2023+), prefer the native **View Transitions API** (`next/navigation` + CSS `@view-transition`) for page transitions — Framer Motion's `AnimatePresence` is legacy for App Router and adds unnecessary JS weight. For Pages Router or non-Next setups, Framer Motion remains valid.
- **SEO ≠ at odds with brand** when IA is clean: descriptive nav labels, real headings (one H1), semantic structure, no orphan microsites, fast pages. CWV is a ranking signal — perf *is* SEO. See [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) and [SEO & Discoverability](../06-marketing-and-conversion/seo-and-discoverability.md).
- **Always re-run Lighthouse/PageSpeed after every banner, plugin, or motion addition** — brand sites bloat by accretion.

## Concrete specs & numbers
- **First impression:** ~50ms. **Communicate identity:** ≤3s. **Bounce risk:** rises sharply past 3s load (+32% bounces at 3s vs 1s).
- **Attention distribution:** 57% above the fold; 74% in first two screenfuls (@1080px viewport = 0–2160px); top 20% of page = 42%+ of viewing time. (NNg, 130K+ fixations, 120 participants, 1920×1080.)
- **Above-fold internal split:** 65%+ of above-fold attention in the top half of the viewport.
- **Hero clickables:** max 3, ideal 1.
- **Scroll narrative:** ≤5 scenes; ≤2–3 min journey.
- **Type scale:** Perfect Fourth 1.333 (or Fibonacci 16–24–40–64–104px). Body line-height 1.5–1.6 (relative units).
- **Core Web Vitals:** LCP ≤2.5s good / 2.5–4s NI / >4s poor · INP ≤200ms / 200–500 NI / >500 poor · CLS <0.1 / 0.1–0.25 NI / >0.25 poor.
- **Conversion sensitivity:** 100ms faster ≈ up to +8% conversion (Deloitte/WPO research); 1s delay ≈ −7%; Rakuten +33% conv / +53% rev-per-visitor after CWV work.
- **Contrast (WCAG):** AA 4.5:1 normal text, 3:1 large (≥24px or ≥18.67px bold) and UI components; AAA 7:1 normal, 4.5:1 large. Logotypes: no requirement. 79.1% of homepages fail contrast (~29.6 failing instances/page, WebAIM Million 2024).
- **Motion:** content moving >5s requires pause/stop control (WCAG 2.1). Lenis ≈ 3kB gzipped.
- **W3C Design Tokens Format Module** published 2024. No unverified token-adoption percentages.
- **OG image:** 1200×630px, page-specific. Favicon set: 32px ICO + 180px Apple touch + 192/512px PWA + `site.webmanifest`.
- **Font loading:** `font-display: optional` prevents FOUT; `font-display: block` holds render ≤3s (never use). Variable font `woff2` replaces 4–6 static weight files.
- **`<title>` length:** ≤60 chars. `<meta name="description">`: ≤155 chars. Both must be page-specific.

## Tools & libraries
- **GSAP + ScrollTrigger** — industry-standard scroll-synced motion; debounced scroll, refresh-synced updates, throttled resize, no scroll-jacking by default. Use `scrub:1`, `anticipatePin:1` for big pins, `CustomEase` for brand easing.
- **Lenis** (Darkroom Engineering) — smooth scroll, ~3kB. `import Lenis from 'lenis'` (v1.0+, replaces `@studio-freight/lenis`); `syncTouch:true`; React: `lenis/react`. Don't pair with native scroll-snap blindly.
- **Framer Motion** — React component/layout transitions; for Next.js Pages Router page transitions. In App Router, use native View Transitions API instead — Framer Motion's `AnimatePresence` is not App Router-compatible for page-level transitions and adds unnecessary weight.
- **Lottie / After Effects export** — icon micro-animations, loaders, illustrative motion without heavy JS. Use *instead of* WebGL when 3D isn't required.
- **WebGL / Three.js / R3F** — 3D brand environments; significant mobile cost — gate, lazy-load, budget vs LCP. NOT for content that must be crawlable or fast on low-end mobile.
- **CSS ScrollTimeline / View Transitions API** — native, progressive-enhancement alternative to GSAP for simpler scroll/transition cases (2024 support improving).
- **Style Dictionary** — JSON tokens → CSS vars + Swift + Android XML + Sass in one pipeline. **Utopia (utopia.fyi)** — fluid type/space `clamp()` generator. **Token Studio (Figma)** — design↔code token sync.
- **Contrast/a11y:** WebAIM Contrast Checker, TPGi Colour Contrast Analyser (CCA), Deque axe DevTools, Stark (Figma), WAVE.
- **Measurement:** Lighthouse / PageSpeed Insights (run after every change); Hotjar / Microsoft Clarity (scroll-depth heatmaps to validate fold per *your* audience); Optimal Workshop / Maze (card sorting + tree testing for IA).
- **OG image generation:** Vercel OG (`@vercel/og`) / Satori — JSX-to-PNG at the edge; generates page-specific OG images from template components.
- **Font tooling:** Google Fonts (free, variable-font capable); Fontshare / Bunny Fonts (GDPR-friendly CDNs); `next/font` (Next.js) auto-optimizes font loading with `font-display: optional` and metric-matched fallbacks built in.
- **Framework:** Next.js (SSR/SSG) for perf-first brand sites. See [Frameworks for Design-Forward Sites](../04-libraries-and-tools/frameworks-for-design.md) and [Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md).

## Do / Don't
**Do**
- Communicate identity + value in ≤3s; lead H1 with a specific transformation only you deliver.
- Build on a two-layer token system (primitive → semantic) so 5+ pages stay identical.
- Define dark-mode semantic tokens upfront (`@media (prefers-color-scheme: dark)`) — retrofitting is expensive.
- Use monochrome base + one accent sparingly + generous, fluid whitespace.
- Expose the full IA in desktop nav; always render the active state.
- Show real product visuals and named, specific proof.
- Make pricing visible without a sales call; repeat the primary CTA every scroll-height.
- Animate the product claim; define one signature easing; honor `prefers-reduced-motion`.
- Treat the hero image as your LCP element (eager, high priority, `srcset`/`sizes`, AVIF).
- Apply brand voice consistently in buttons, errors, empty states, and tooltips — not just headlines.
- Ship page-specific OG images, a complete favicon set, and unique `<title>`/`<meta description>` on every page.
- Re-run Lighthouse after every banner/plugin/motion addition.

**Don't**
- Don't ship a 100vh hero that hides all below-fold content (Illusion of Completeness).
- Don't use a hero carousel, on-load popup, or autoplay-without-control.
- Don't split primary nav by audience type, or use vague labels (`Explore`, `Learn more`).
- Don't use a desktop hamburger; don't omit the active-state indicator.
- Don't scatter 5 colors, lead hierarchy with size alone, or set line-height in px.
- Don't let polish drop contrast below AA or push LCP past 2.5s.
- Don't leave orphan campaign microsites diluting brand + SEO.
- Don't add motion that neither demonstrates nor orients.
- Don't use `font-display: block` — it blocks render for up to 3s.
- Don't hardcode hex in components — it breaks dark mode and rebrands.
- Don't use a generic logo-on-white image as the OG image for every page.

## Common pitfalls & how to avoid them
- **Full-screen hero swallows the page.** → Cap hero at ~85vh; let the next section peek; verify scroll-depth in Clarity/Hotjar.
- **Brand drifts page-to-page.** → Centralize spacing/type/color/easing in semantic tokens; ban hardcoded hex in components.
- **Award polish kills usability.** → Order priorities usable → on-brand → spectacular; cut any effect that doesn't clarify or orient.
- **"Muted/elegant" low-contrast text.** → Hit AA 4.5:1 (3:1 large); validate with WebAIM/axe — it's the #1 homepage failure.
- **Audience-split nav confuses everyone.** → Organize nav by what the product does; surface audiences inside pages.
- **Heavy WebGL/3D tanks mobile CWV.** → Gate by capability + reduced-motion, lazy-load, budget against LCP, never block the H1.
- **`clamp()` type fails zoom.** → Always include a `rem` term; test 200% / 500% zoom against WCAG 1.4.4.
- **Vague hero CTA.** → Replace `Learn more`/`Get started` with call-to-value: `Start building`, `Create your project now`.
- **Site bloats by accretion** (banners, pixels, embeds). → Re-measure CWV after every addition; enforce a perf budget.
- **Orphan microsites from old campaigns.** → Fold into main IA or 301-redirect.
- **Hardcoded hex in components breaks dark mode and rebrands.** → All color references go through semantic tokens; primitives are never used directly in components.
- **Generic OG image (logo on white) on every URL.** → Generate per-page OG images; a shared link to a docs page or blog post with a homepage thumbnail reads as low craft.
- **`font-display: block` stalls first render.** → Switch to `optional` (body) or `swap` + `size-adjust` fallback (display type where exact face matters). Preconnect to font CDN.
- **Brand voice breaks in error states and empty states.** → Audit every system-generated string (errors, empty tables, confirmation dialogs) against the same voice guidelines as the homepage copy.

## What real users complain about
(Synthesized from NNg studies, WebAIM data, and practitioner reports.)
- **"I thought that was the whole page."** Full-screen heroes with no peek make users miss everything below — NNg qualitative findings consistently show the majority of participants not scrolling past a full-screen hero with no visible continuation.
- **"Where am I?"** No active-state in nav = lost orientation — the most common nav complaint.
- **"I can't read the grey-on-white text."** Low contrast on 79.1% of homepages; a constant, top accessibility grievance.
- **"A popup hit me before I saw anything."** On-load modals (especially newsletter) before any product content = immediate frustration / bounce.
- **"I can't find the close button"** — tiny, same-color-as-background X is a top-cited annoyance.
- **"It hijacked my scroll"** — scroll-jacking that fights intent feels broken, not premium.
- **"The carousel moved before I read it"** — and slide 2+ goes unseen (≈89% of engagement lands on slide 1 in commonly cited UX studies; treat as directional).
- **"Which one am I — developer or enterprise?"** Audience-split nav stalls multi-fit users.
- **"It looks amazing but I don't know what they sell."** Style with no clear value prop in 3s = forgettable.
- **"Autoplay video I can't pause."** WCAG-forbidden and widely resented; also pattern-ignored as an ad.

## Senior-level checklist
- [ ] A stranger can state *who you are + what you do + next action* within 3s of the hero.
- [ ] Hero ≤3 clickables (ideally 1 primary CTA), call-to-value copy, real product visual, content peeks above the fold (≤~85vh).
- [ ] Value prop + H1 + primary CTA sit in the upper-center; above-fold ends with CTA in the Z bottom-right.
- [ ] Two-layer tokens (primitive → semantic) drive type, spacing, color, radius, easing; zero hardcoded hex in components.
- [ ] Monochrome base + one accent used only for CTAs/active/emphasis; whitespace generous and fluid (`clamp()` section gaps).
- [ ] Type: weight-led hierarchy, modular scale (1.333 or Fibonacci), body line-height 1.5–1.6 in relative units; `clamp()` has a `rem` term and passes 200%/500% zoom.
- [ ] Desktop nav exposes the IA (no hamburger ≥1024px), labels have strong scent, active state via `aria-current` + visible style.
- [ ] Nav is NOT split by audience type; no orphan microsites.
- [ ] Narrative casts customer as hero / product as mentor; ≤5 scenes; ≤2–3 min journey.
- [ ] Every animation demonstrates a product claim or aids orientation; signature easing defined; `prefers-reduced-motion` honored; any >5s motion has pause/stop.
- [ ] No hero carousel, no on-load popup, no autoplay-without-control; close affordances are obvious.
- [ ] Pricing visible without a sales call; primary CTA repeated every scroll-height; proof is named and specific.
- [ ] Hero = LCP element: AVIF/WebP + `srcset`/`sizes`, `fetchpriority="high"`, `loading="eager"`, explicit dimensions; never lazy-loaded.
- [ ] Dark-mode semantic tokens defined upfront in `@media (prefers-color-scheme: dark)` — zero hardcoded hex in components.
- [ ] Every page has a page-specific `<title>` (≤60 chars), `<meta description>` (≤155 chars), and 1200×630px OG image.
- [ ] Favicon set complete: 32px ICO, 180px Apple touch, 192/512px PWA, `site.webmanifest`.
- [ ] Font loading: variable `woff2` + `font-display: optional` (or `swap` with `size-adjust` fallback); `preconnect` to font CDN; never `font-display: block`.
- [ ] CWV verified: LCP ≤2.5s, INP ≤200ms, CLS <0.1 on mobile (Lighthouse re-run after final assets/banners).
- [ ] Contrast: all text ≥ AA (4.5:1 / 3:1 large), UI components ≥3:1, validated with axe/WebAIM.
- [ ] IA validated with card sort + tree test; scroll-depth (Clarity/Hotjar) confirms ≥50% of visitors reach the proof section; if not, move proof up.

## Sources
- Nielsen Norman Group (NNg) — homepage principles, fold/attention eyetracking study (130K+ fixations, 120 participants), navigation & motion guidance: https://www.nngroup.com/
- WebAIM Million 2024 (homepage accessibility / low-contrast prevalence, ~29.6 failing instances/page): https://webaim.org/projects/million/
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- Web Content Accessibility Guidelines (WCAG) 2.1/2.2 — contrast ratios, motion (2.3.3), resize text (1.4.4): https://www.w3.org/TR/WCAG21/
- Core Web Vitals / CrUX (Google): https://web.dev/vitals/ · PageSpeed Insights: https://pagespeed.web.dev/
- W3C Design Tokens Community Group — Design Tokens Format Module (2024): https://www.w3.org/community/design-tokens/
- GSAP / ScrollTrigger: https://gsap.com/docs/v3/ · Lenis (Darkroom Engineering): https://github.com/darkroomengineering/lenis
- Style Dictionary: https://amzn.github.io/style-dictionary/ · Utopia (fluid type/space generator): https://utopia.fyi/
- Vercel OG / Satori (dynamic OG image generation): https://vercel.com/docs/functions/og-image-generation
- Next.js View Transitions (App Router): https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating
- TPGi Colour Contrast Analyser: https://www.tpgi.com/color-contrast-checker/ · Deque axe DevTools: https://www.deque.com/axe/
- Reference brand sites (study, don't copy): https://linear.app · https://stripe.com · https://vercel.com · https://loom.com

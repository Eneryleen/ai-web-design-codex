# Master Checklists & Cheat-Sheets

> A fast-lookup reference an AI consults before designing and after building. Condensed pre-launch checklists (design, responsive, a11y, performance, SEO, content, analytics, security), a senior design-review rubric, quick-reference numbers for spacing/type/color, the squint test, and a decision tree for "what site type and stack do I need." This is the index of indexes — drill into the linked guides for depth.

**Apply when:** scoping a build, reviewing your own output, or doing final pre-launch QA · **Related:** [Design Process & Stages](../05-process-and-systems/design-process-and-stages.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Common Mistakes & Antipatterns](../08-pitfalls-and-antipatterns/common-mistakes-antipatterns.md)

## TL;DR — rules at a glance
- **Design in grayscale first, add color last.** Force hierarchy through spacing, size, and weight before color becomes a crutch.
- **Mobile-first, always.** Mobile accounts for ~60% of global web traffic (StatCounter 2025); Google uses the mobile version as the primary index. Content parity between mobile and desktop is mandatory.
- **8px grid for everything.** Spacing values from {4, 8, 12, 16, 24, 32, 40, 48, 64}. No `13px`, no `27px`.
- **Type scale, not arbitrary sizes.** 16px body minimum, ~1.25 ratio scale (16/20/25/31/39/49), line-height 1.4–1.6 body, line length 45–75 chars.
- **Contrast is law:** body text 4.5:1, large text 3:1, UI/icons/focus rings 3:1 (WCAG 2.2 AA).
- **Core Web Vitals gates:** LCP ≤2.5s, INP ≤200ms, CLS ≤0.1. Lab (Lighthouse) ≠ field (CrUX) — ship against both.
- **Never lazy-load the LCP image.** Per HTTP Archive, ~16–17% of pages do this and tank LCP. Add `fetchpriority="high"` instead.
- **Every interactive element needs 5 states:** default, hover, focus(-visible), active, disabled (+ error where relevant).
- **Touch targets:** 44×44px is the working standard (Apple HIG); WCAG 2.5.8 floor is 24×24 CSS px.
- **Accessibility overlays make things worse.** Build semantic HTML; never install accessiBe/UserWay as a "fix."
- **Run the squint test.** Blur the design 5–10px; the primary CTA/headline must stay dominant. If a decorative image wins, the hierarchy is broken.
- **Kill the scaffolding tells:** default favicon, "Vite App"/"React App"/"Create React App" title, lorem ipsum, debug comments, `noindex` from staging. These scream "unfinished."
- **Trust signals at decision points** (next to CTAs), not buried in the footer.

---

## How to use this document

There are two consultation points:

1. **Before building** — run the *decision tree* to fix site type + stack + rendering strategy, then load the *quick-reference numbers* into context so you don't emit arbitrary values.
2. **After building** — run the *pre-launch checklists* and the *design-review rubric* as a gate. Treat unchecked items as defects, not suggestions.

---

## Decision tree — site type & stack

Resolve top-down. Stop at the first match. Choose **rendering per route**, not per app.

```
Q1. Content mostly static, infrequently updated?
    → Blog / Portfolio / Docs / Marketing one-pager / AI-content-heavy site
    → Astro (preferred for content-heavy), 11ty, or Next.js SSG + Git-based CMS (Markdown/MDX, Decap, Keystatic)
    → Deploy: Vercel / Netlify / Cloudflare Pages (static + edge)

Q2. Marketing/landing with periodic content edits by non-devs?
    → Next.js ISR (or Astro + on-demand rebuild) + headless CMS (Sanity / Contentful / Strapi)

Q3. News / editorial with live, time-sensitive content?
    → Next.js or Nuxt hybrid (SSG for archive + SSR/ISR for fresh)

Q4. E-commerce?
    → Simplicity / small catalog → Shopify native (themes) or Webflow Ecommerce
    → Scale / custom storefront → Shopify Hydrogen (headless) or Medusa/commercetools + Next.js
    → Real-time inventory / complex pricing → SSR-heavy

Q5. SaaS app / dashboard / per-user authenticated data?
    → Next.js / Remix / SvelteKit (SSR shell) + CSR for interactive panels
    → Data-dense admin → server-driven tables, virtualization, optimistic UI

Q6. Non-technical client team + small budget (<$50k)?
    → WordPress (block themes) or Webflow — hand off a CMS they can run

Q7. Large enterprise with an existing CMS / governance?
    → Headless CMS + framework the in-house team already knows (don't impose a stack)

Q8. Installable / offline-capable PWA?
    → Any of the above + Web App Manifest + Service Worker (Workbox); ensure HTTPS, icons at 192/512px, and a valid `start_url`
```

**Rendering cheat-sheet:** SSG = fastest + cheapest, content known at build. ISR = SSG + scheduled/on-demand revalidation. SSR = per-request HTML, auth/personalization/freshness. CSR = client-only, app shells and highly interactive panels. Default to the *least dynamic* option a route can tolerate. See [Frameworks for Design](../04-libraries-and-tools/frameworks-for-design.md) and [SaaS Web Apps](../03-site-types/saas-web-apps.md).

**Deploy targets:** edge platforms (Vercel/Netlify/Cloudflare Pages) for static/ISR; traditional IaaS (AWS/GCP) when SSR needs full infra control, long-running jobs, or specific compliance.

**Note on Speculation Rules API:** Chrome-only as of 2026 (no Safari/Firefox support). Use as a progressive enhancement via `<script type="speculationrules">`; don't rely on it for core performance targets.

---

## Pre-launch checklist — Design

- [ ] Brand colors and fonts consistent across **all** pages (no rogue blues, no mixed font stacks).
- [ ] Favicon present and correct — **not** the Vite/Next/framework default.
- [ ] Page `<title>` meaningful and unique per page — not the scaffolding default ("My App", "Vite App").
- [ ] Hover, focus(-visible), active states defined on **every** interactive element.
- [ ] Disabled and loading states defined where applicable.
- [ ] Touch targets ≥44px on mobile.
- [ ] No lorem ipsum / placeholder copy anywhere.
- [ ] No debug comments, `console.log`, TODOs, or profanity in shipped source.
- [ ] Custom 404 page present and on-brand (with a way back home + search).
- [ ] All meaningful images have descriptive `alt`; decorative images have `alt=""`.
- [ ] Contrast verified: text 4.5:1, large text 3:1, UI/focus 3:1.
- [ ] Empty states, error states, and zero-data views designed (not blank screens).

## Pre-launch checklist — Responsive

- [ ] `<meta name="viewport" content="width=device-width, initial-scale=1">` present — without this, mobile browsers use a 980px virtual viewport and scaling breaks.
- [ ] Tested in Chrome, Firefox, Safari, Edge.
- [ ] Tested on **real** mobile devices, not just DevTools emulation (touch, real fonts, iOS Safari quirks).
- [ ] Mobile menu opens, closes, traps focus, and is reachable by keyboard.
- [ ] No overlapping/clipped buttons or text at any breakpoint.
- [ ] **Content parity** between mobile and desktop (critical for mobile-first indexing — don't hide content on mobile).
- [ ] No horizontal scroll at 320px (the usual culprit: a fixed-width element or unconstrained image).
- [ ] All images have explicit `width`/`height` or CSS `aspect-ratio` to prevent CLS.
- [ ] Verified at 320 / 375 / 768 / 1024 / 1440px; check 1920px+ for max-width clamping.

## Pre-launch checklist — Accessibility

- [ ] `<html lang="...">` set to the correct BCP 47 language tag (WCAG 3.1.1 Level A — screen readers won't choose the right voice without this).
- [ ] Tab through every interactive element — focus order is logical and matches visual order.
- [ ] No keyboard traps except intentional modal traps with **Escape to exit**.
- [ ] Form fields use correct input types (`email`, `tel`, `url`, `date`, `number`) — not generic `text`.
- [ ] Form fields have associated `<label>` elements (not just `placeholder`).
- [ ] Submit-via-Enter works on every form.
- [ ] Screen-reader pass (VoiceOver or NVDA) on at least one core flow.
- [ ] Focus indicators visible at ≥3:1 contrast — never `outline: none` without a replacement.
- [ ] Color is never the sole differentiator (errors, links, states have a non-color cue).
- [ ] Landmarks present: `<header> <nav> <main> <footer>`; one `<main>` per page.
- [ ] Skip-to-main-content link is the first focusable element.
- [ ] No ARIA misuse: every `role` has required owned elements; `aria-label` on interactive controls, not decorative spans; no redundant `role="button"` on `<button>`.
- [ ] axe-core or WAVE audit passes with **zero critical errors**.
- [ ] `prefers-reduced-motion` respected for non-essential animation.
- [ ] No accessibility overlay (accessiBe/UserWay/etc.) installed. See [WCAG guide](../01-ux-fundamentals/accessibility-wcag.md).

## Pre-launch checklist — Performance

- [ ] LCP element identified and **not** lazy-loaded.
- [ ] `fetchpriority="high"` on the LCP image (per HTTP Archive, only ~16–17% of pages set this — easy win).
- [ ] Webfont loading strategy: use `font-display: swap` for brand/display fonts; use `font-display: optional` for body text if CLS from FOUT is a risk (browser skips font on slow connections and uses fallback — eliminates layout shift). `preconnect`/`preload` for critical fonts either way.
- [ ] Critical CSS inlined or preloaded; non-critical CSS deferred.
- [ ] Every image has explicit `width` and `height`.
- [ ] Images in WebP/AVIF with fallbacks; responsive `srcset`/`sizes`.
- [ ] No render-blocking resources above the fold (historically only ~13–15% of pages pass this audit per HTTP Archive).
- [ ] JS `defer`/`async`; non-composited animations eliminated (use `transform`/`opacity` only).
- [ ] CDN for static assets; long-lived cache headers with content hashing.
- [ ] Lighthouse ≥90 on Performance, Accessibility, Best Practices, SEO.
- [ ] Validated against **field data** in PageSpeed Insights (CrUX), not just lab.
- [ ] For hero images: WebP ≤250KB or AVIF ≤150KB; never ship an uncompressed JPEG hero above the fold.
- [ ] Consider Speculation Rules API (`<script type="speculationrules">`) for instant in-site navigation (Chrome-only, progressive enhancement). See [Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md).

## Pre-launch checklist — SEO

- [ ] **Remove `noindex` / `Disallow: /` from production** — staging blocks that never got reverted are the #1 launch SEO disaster.
- [ ] `robots.txt` present and not blocking crawl of real content.
- [ ] XML sitemap at `/sitemap.xml`, submitted to Google Search Console **and** Bing Webmaster Tools.
- [ ] Canonical form chosen (www vs non-www, https, trailing slash) and **all variants 301 to it**.
- [ ] `rel="canonical"` set on every page.
- [ ] Exactly one `<h1>` per page with proper H1→H6 nesting (no skipped levels).
- [ ] No orphan pages — every page linked from at least one other.
- [ ] 301 redirects for every changed URL from the previous site version.
- [ ] Schema/JSON-LD validated via Google Rich Results Test.
- [ ] Open Graph + Twitter Card tags on key pages (title, description, image 1200×630 minimum — 1200×675 16:9 for X/Twitter large card; both render correctly at 1200×630).
- [ ] For multi-language sites: `hreflang` annotations on all alternate URLs (or in sitemap).
- [ ] Meta titles 50–60 chars, descriptions 150–160 chars, unique per page. See [SEO guide](../06-marketing-and-conversion/seo-and-discoverability.md).

## Pre-launch checklist — Content

- [ ] Spelling and grammar checked (whole site, not just homepage).
- [ ] Contact form submits **and** the email actually arrives.
- [ ] Contact details correct in header and footer.
- [ ] Privacy policy, terms, and cookie consent present and current.
- [ ] Legal pages match jurisdiction (GDPR / CCPA / local).
- [ ] Social links open the correct, live accounts.
- [ ] All PDFs/downloads work and are accessible (tagged PDFs where possible).
- [ ] Blog/news has real content — no "Hello World" or placeholder posts.

## Pre-launch checklist — Analytics & monitoring

- [ ] GA4 or privacy-friendly alternative (Plausible, Fathom) installed on **production**, not staging.
- [ ] Conversion events fire and are verified in GA4 DebugView / GTM Preview.
- [ ] **Consent Mode v2** configured if serving EU users — required by Google since March 2024; without it, GA4 cannot model conversions for EU traffic (ad campaigns go blind without it).
- [ ] Uptime monitoring (UptimeRobot, Better Stack) configured.
- [ ] Error monitoring (Sentry) configured with source maps.

## Pre-launch checklist — Security & technical

- [ ] SSL valid; all HTTP → HTTPS redirects work.
- [ ] Security headers set — verify at securityheaders.com (CSP, X-Frame-Options, X-Content-Type-Options, HSTS, **Permissions-Policy**, Referrer-Policy). Permissions-Policy replaces the deprecated Feature-Policy header; omitting it leaves a gap in the securityheaders.com audit.
- [ ] **MX records preserved** if migrating DNS (lose these and email goes dark for hours).
- [ ] DNS TTL lowered to 60–300s **before** a server migration, raised again after.
- [ ] Directory listing disabled.
- [ ] Default CMS passwords changed; inactive WP themes/plugins deleted.
- [ ] Production env vars correct (prod DB, debug off, no staging keys).
- [ ] Automated backups configured **and restoration tested** (an untested backup is not a backup).
- [ ] www↔non-www redirect configured.
- [ ] Honeypot field on every public form to absorb bot spam.

---

## Senior design-review rubric (8 dimensions)

Score each 1–5. Anything <4 is a blocker. Be adversarial — hunt for what's wrong, not for reasons to pass.

| # | Dimension | What "pass" looks like |
|---|-----------|------------------------|
| 1 | **Layout & Spacing** | 8px-grid adherence; consistent padding rhythm; generous white space; nothing crammed or touching edges. |
| 2 | **Typography** | ≤2–3 typefaces; clear modular scale; body ≥16px; line length 45–75 chars; line-height 1.4–1.6 body. |
| 3 | **Color & Contrast** | Purposeful palette; WCAG 4.5:1 text contrast; color never the only signal; one accent does the heavy lifting. |
| 4 | **Visual Hierarchy** | Squint test passes — primary element dominant when blurred; obvious entry point and reading order. |
| 5 | **Components** | Consistent, reusable; all states present (default / hover / active / focus / disabled / error / loading). |
| 6 | **Accessibility** | Focus visible; semantic HTML; alt text; fully keyboard operable; landmarks; reduced-motion respected. |
| 7 | **Responsiveness** | Verified at 320 / 768 / 1024 / 1440px; no overflow; content parity; touch ergonomics correct. |
| 8 | **Handoff Readiness** | Design tokens defined at minimum for spacing, color, type size, and border-radius; interaction/motion specs written (duration + easing); no magic numbers anywhere in styles. |

See [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md) and [Wireframing, Prototyping, Handoff & QA](../05-process-and-systems/wireframing-prototyping-handoff-qa.md).

---

## The squint test (protocol)

The fastest hierarchy check that exists. Run it on every key screen.

1. Apply a Gaussian blur of **5–10px radius** to the whole screen (design tool blur, or step back from the monitor and squint).
2. Ask: *what's still visually dominant?* It must be the **primary CTA or headline**.
3. If a decorative image, a stock photo, or a secondary element wins, reduce its weight (lower contrast, smaller size, less saturation) or raise the primary's.
4. Complement with the **5-second test**: show the un-blurred design to a stranger for exactly 5 seconds, then ask "what is this page about?" and "what would you do next?" Wrong answers = broken hierarchy or copy.
5. Validate with real behavior via heat maps (Hotjar, Microsoft Clarity) once live.

> Hierarchy is created *before* color. If the squint test only passes because of one bright button on a flat-gray page, good — that's the point. See [Visual Hierarchy](../00-foundations/visual-hierarchy.md).

---

## Quick-reference numbers

### Spacing — 8px grid
```
4  8  12  16  24  32  40  48  64  96  128   (px; all multiples of 8, plus the 4px half-step)
```
Material 3: 4dp base grid, 8dp increments. Pick from the scale; never hand-roll `13px`/`27px`. See [Layout, Grids & Spacing](../00-foundations/layout-grids-and-spacing.md).

### Type scale — ~1.25 (major third) from 16px
```
16 → 20 → 25 → 31 → 39 → 49 px
```
- Body: **16px minimum** on mobile (14px only for dense data UI / captions on desktop; never 14px as the default mobile body size — too small for normal reading distance).
- Line-height: **1.4–1.6** body, **1.1–1.25** headings.
- Line length: **45–75 characters** (~`20–35em` container width).
- Max **2–3 typefaces**. See [Typography Systems](../00-foundations/typography-systems.md).

```css
/* Fluid body type: 16px at 375px viewport → 18px at ~1200px viewport, clamped */
:root { font-size: clamp(1rem, 0.95rem + 0.4vw, 1.125rem); }
p { max-width: 65ch; line-height: 1.55; }
```

### Color & contrast (WCAG 2.2 AA)
- Body text: **4.5:1** against background.
- Large text (≥24px regular, or ≥18.66px/14pt **bold**): **3:1**.
- UI components, icon glyphs, focus indicators, chart elements: **3:1** vs adjacent colors.
- Focus appearance (2.4.13): ≥3:1 contrast change between focused/unfocused **and** ≥3:1 vs surroundings.
- Build palettes as scales (50→900); reserve one accent for primary actions. See [Color Theory & Systems](../00-foundations/color-theory-and-systems.md).

### Touch targets
- **44×44px** — Apple HIG, the working industry standard.
- **48×48dp** — Material.
- **24×24 CSS px** — WCAG 2.5.8 hard floor (or 24px clear spacing around smaller controls).
- Below 44px on mobile = mis-taps. See [Touch Ergonomics](../02-responsive-adaptive/touch-ergonomics-cross-device.md).

### Breakpoints (mobile-first, `min-width`)
```
base (≤639)  ·  640 (sm)  ·  768 (md)  ·  1024 (lg)  ·  1280 (xl)  ·  1536 (2xl)
```
Test minimum at **320 / 768 / 1024 / 1440**. Add breakpoints where the *content* breaks, not at fixed device sizes. See [Breakpoints & Mobile-First](../02-responsive-adaptive/breakpoints-devices-mobile-first.md).

### Core Web Vitals thresholds
| Metric | Good | Needs improvement | Poor |
|--------|------|-------------------|------|
| LCP | ≤2.5s | 2.5–4.0s | >4.0s |
| INP | ≤200ms | 200–500ms | >500ms |
| CLS | ≤0.1 | 0.1–0.25 | >0.25 |
| FCP | <1.8s | 1.8–3.0s | >3.0s |
| TTFB | <0.8s | 0.8–1.8s | >1.8s |

Targets: server response (TTFB) <800ms (CrUX "good") — Lighthouse flags it above 200ms as a lab hint, but that is not the field threshold; mobile page load <3s; full-bleed hero image WebP ≤250KB / AVIF ≤150KB.

### Motion & micro-interactions
- UI transitions: **150–300ms**; page/large transitions: up to ~500ms. Longer feels sluggish.
- Easing: ease-out for entrances, ease-in for exits; avoid linear for UI.
- Animate **only** `transform` and `opacity` (GPU-composited). Animating `width`/`top`/`left`/`box-shadow` causes layout/paint jank. See [Micro-interactions & Motion](../07-aesthetics-and-trends/micro-interactions-and-motion.md).

### SEO numbers
- Meta title **50–60** chars; meta description **150–160** chars.
- One `<h1>` per page; OG image **1200×630**.

---

## Tools & libraries

- **Audit/perf:** Lighthouse (lab), PageSpeed Insights + CrUX (field), WebPageTest, `web-vitals` JS lib (RUM).
- **Accessibility:** axe DevTools, WAVE, VoiceOver (macOS/iOS), NVDA (Windows), Accessibility Insights. **Avoid:** accessiBe/UserWay overlays — they degrade real AT experience (see overlayfactsheet.com).
- **Contrast:** browser DevTools contrast checker, Stark, Polypane.
- **Analytics:** GA4, or Plausible/Fathom for privacy-first; Hotjar / Microsoft Clarity for heatmaps & session replay.
- **Monitoring:** UptimeRobot / Better Stack (uptime), Sentry (errors).
- **Headers/security:** securityheaders.com, SSL Labs.
- **Styling/components:** Tailwind + a headless library (Radix, React Aria, shadcn/ui) for accessible-by-default primitives. See [CSS & Tailwind](../04-libraries-and-tools/css-styling-and-tailwind.md) and [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md).

**When NOT to reach for tools:** don't bolt on an accessibility overlay, a "performance optimizer" plugin, or a CSS framework to paper over a broken layout. Fix the HTML/CSS. Don't add a heavy animation library for a fade — CSS transitions suffice.

---

## Do / Don't

**Do**
- Start grayscale; introduce color only to reinforce existing hierarchy.
- Constrain every property to a system (spacing, type, color, shadow, radius, motion).
- Make core functionality work without JS, then progressively enhance.
- Place trust signals (reviews, security badges, guarantees) **at decision points** next to CTAs.
- Set explicit image dimensions and `fetchpriority="high"` on the LCP image.
- Validate against field data (CrUX), not just Lighthouse lab scores.

**Don't**
- Don't rely on color alone to convey meaning, state, or hierarchy.
- Don't `outline: none` without a visible replacement focus style.
- Don't lazy-load above-the-fold/hero images.
- Don't hide content on mobile that exists on desktop (breaks mobile-first indexing).
- Don't ship the framework default favicon, title, or scaffolding copy.
- Don't trust an untested backup, a single-browser test, or DevTools-only responsive checks.

---

## Common pitfalls & how to avoid them

- **Staging `noindex`/`Disallow: /` shipped to prod.** → Add a launch-day step that diffs `robots.txt` and meta robots against expected production values.
- **CLS from unsized images.** Per HTTP Archive Web Almanac, a large majority of pages still ship images without explicit dimensions (figures shift annually — check the current CWV chapter). → Enforce `width`+`height` or `aspect-ratio` lint rule.
- **Accidentally lazy-loaded LCP.** 16–17% of pages do this. → Never put `loading="lazy"` on the hero; reserve it for below-fold media.
- **Mobile JS bottleneck.** Mobile TBT is consistently 10–20× higher than desktop in HTTP Archive data (recent editions show mobile TBT medians in the 1,500–2,000ms range vs desktop ~90–100ms). → Ship less JS to mobile: code-split, defer, prefer SSR/SSG, audit third-party scripts.
- **Lab-only optimization.** Lighthouse simulates a mid-tier mobile device with CPU/network throttling (Lighthouse 10+ uses calibrated throttling, no longer a fixed Moto G4) — it approximates a real low-end user but is not your median visitor. → Confirm with CrUX/RUM before declaring victory.
- **Render-blocking resources.** Per HTTP Archive Web Almanac, only ~13–15% of pages pass the render-blocking audit (check the current Almanac for the latest figure). → Inline critical CSS, `defer`/`async` JS, self-host or preconnect fonts.
- **Lost email after DNS migration.** → Snapshot MX/TXT/SPF/DKIM records before any nameserver change; verify mail flow after cutover.
- **Suppressed focus styles.** → If you remove the default outline, ship a `:focus-visible` style at ≥3:1 contrast.
- **Over-engineered stack for a brochure site.** → Re-run the decision tree; a 5-page marketing site does not need SSR + a headless CMS.

See [Common Mistakes & Antipatterns](../08-pitfalls-and-antipatterns/common-mistakes-antipatterns.md) and [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md).

---

## What real users complain about

Synthesized from usability research and field feedback (see [Real User Complaints](../08-pitfalls-and-antipatterns/real-user-complaints.md)):

- **"It's too slow."** Slowness is the most common, least forgiven complaint. Multiple industry studies (Google/SOASTA 2018, Deloitte 2020) found mobile conversion rates drop measurably with each second of delay; exact percentages vary widely by industry and starting conversion rate. Use your own CrUX data to measure actual impact — don't cite generic "X% drop" figures without measuring.
- **"I can't tap the button."** Sub-44px targets, buttons too close together, sticky bars covering controls.
- **"The text is tiny / I had to zoom."** Body below 16px on mobile, low-contrast gray-on-gray.
- **"The page jumped while I was reading."** CLS from late-loading images, ads, and fonts.
- **"Where's the menu / how do I get back?"** Hidden or unlabeled hamburger, no obvious home link, broken back behavior on SPAs.
- **"The cookie banner ate the screen."** Full-screen consent walls, no easy "reject all."
- **"It looked broken on my phone."** Horizontal scroll, overlapping elements, content cut off at 320px.
- **"I didn't trust it."** No contact info, no reviews near the CTA, generic stock imagery, scaffolding tells. Trust is binary at the moment of purchase — missing signals cause abandonment with no second chance.

---

## Senior-level checklist (final gate)

Verifiable, binary, do-not-ship-without:

- [ ] Decision tree run; site type, stack, and per-route rendering strategy are explicit and justified.
- [ ] Design built grayscale-first; squint test passes on every key screen.
- [ ] All spacing on the 8px grid; type on a modular scale; ≤3 typefaces.
- [ ] Contrast verified everywhere (4.5:1 / 3:1); color is never the only signal.
- [ ] All five+ interactive states implemented; focus-visible at ≥3:1.
- [ ] Keyboard-only pass and one screen-reader pass on a core flow completed.
- [ ] axe-core/WAVE: zero critical errors.
- [ ] Responsive verified on real devices at 320/768/1024/1440; no horizontal scroll; content parity.
- [ ] LCP not lazy-loaded; `fetchpriority="high"`; all images sized; webfonts use `swap` or `optional` as appropriate (not default `auto`).
- [ ] CWV pass on **field data** (LCP ≤2.5s, INP ≤200ms, CLS ≤0.1); Lighthouse ≥90 across categories.
- [ ] SEO: prod indexable, sitemap + robots correct, canonicals + 301s set, one H1/page, OG tags present.
- [ ] Content: real copy, working contact form (email received), legal pages current.
- [ ] Analytics + uptime + error monitoring live on production.
- [ ] Security headers set; SSL valid; MX preserved; backups tested.
- [ ] No scaffolding tells: real favicon, real titles, custom 404, no lorem ipsum, no debug code.
- [ ] Design-review rubric scored ≥4 on all 8 dimensions.

---

## Sources
- HTTP Archive Web Almanac (annual; Performance & CWV chapters) — https://almanac.httparchive.org/
- web.dev — Core Web Vitals & thresholds — https://web.dev/articles/vitals
- Chrome UX Report (CrUX) — https://developer.chrome.com/docs/crux
- PageSpeed Insights — https://pagespeed.web.dev/
- WCAG 2.2 (W3C Recommendation) — https://www.w3.org/TR/WCAG22/
- Overlay Fact Sheet — https://overlayfactsheet.com/
- Refactoring UI (Wathan & Schoger) — https://www.refactoringui.com/
- Material Design 3 — https://m3.material.io/
- Apple Human Interface Guidelines — https://developer.apple.com/design/human-interface-guidelines
- Google Search Central (mobile-first indexing, SEO) — https://developers.google.com/search
- securityheaders.com — https://securityheaders.com/

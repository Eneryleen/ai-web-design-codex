# SEO & Discoverability for Designers

> How design and front-end decisions make or break organic discoverability: semantic structure, Core Web Vitals, structured data, social cards, rendering architecture, and the design choices that quietly kill rankings. Reading this lets you build sites that are technically indexable, AI-citable, and rich-result eligible by default — without an SEO retrofit later.

**Apply when:** building any public-facing page meant to be found via search or shared on social · **Related:** [Web Performance & Core Web Vitals](./web-performance-core-web-vitals.md) · [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Frameworks for Design-Forward Sites](../04-libraries-and-tools/frameworks-for-design.md)

## TL;DR — rules at a glance

- **One `<h1>` per page**, aligned with the title tag and primary keyword. Never skip levels (H1→H3). Never use headings for visual sizing — use CSS.
- **Use semantic landmarks** (`<main>`, `<nav>`, `<article>`, `<header>`, `<footer>`, `<aside>`) — not `<div>` soup. Crawlers, screen readers, and AI parsers read the structure.
- **SSR or SSG everything above the fold.** CSR-only SPAs get JS-render-queued (3–7 days lag for new sites). AI crawlers (ChatGPT, Perplexity, Claude) largely do not execute JS — no SSR = no citation.
- **Preload the LCP image with `fetchpriority="high"`; never `loading="lazy"` on the hero.** Lazy-load everything below the fold.
- **Inject JSON-LD structured data server-side, in `<head>`.** It must exist in raw HTML before JS runs.
- **Title 50–60 chars** (~600px). Over 70 chars → Google rewrites ~100% of the time.
- **Meta description 140–160 chars.** Doesn't rank; drives CTR. Better blank than duplicated.
- **OG image exactly 1200×630px, absolute URL.** Add `twitter:card` separately — X has no OG fallback for card type.
- **Set explicit `width`/`height` on every image** to reserve layout space and prevent CLS.
- **Internal linking is the only ranking lever fully under your control.** Important pages ≤3 clicks from home; no orphans.
- **CWV is a tiebreaker (~10–15% of signals), not a primary factor.** Escape the red zone; don't chase 79→99 Lighthouse for rank.
- **Slugs:** 3–5 hyphenated words, ~25–30 chars, no dates, no underscores, strip stop words.
- **`alt` describes content + context in ≤125 chars.** Decorative images get `alt=""`. No keyword stuffing.
- **`rel=canonical` on every parameterized/duplicate URL**, pointing to the clean version. Keep redirects single-hop 301.
- **Don't build new dynamic rendering** (bot-vs-user HTML) — Google deprecated it 2024–2025.
- **Never `Disallow: /` in `robots.txt`** (production deploy accident that wipes Google visibility). **Never block CSS/JS.** Use `<meta name="robots" content="noindex">` to suppress indexing — `robots.txt` only blocks crawling.
- **`<meta name="viewport" content="width=device-width, initial-scale=1">`** in every page `<head>` — its absence triggers Search Console mobile-usability errors.
- **Multi-language sites: ship `hreflang` on every variant** pointing to all other variants. Omitting it causes duplicate-content or wrong-locale ranking.
- **Add `max-image-preview:large` to the `<meta name="robots">` tag** on content pages — required for large image thumbnails in Google Discover.

---

## Semantic HTML & heading structure

Crawlers and AI parsers build a structural model of your page from its HTML. Semantic elements give that model explicit meaning; `<div>` soup gives it nothing.

**Landmark skeleton — use this, not nested divs:**

```html
<body>
  <header>            <!-- site logo, primary nav -->
    <nav aria-label="Primary">…</nav>
  </header>
  <main>              <!-- exactly one per page; the unique content -->
    <article>
      <h1>The single page headline</h1>
      <section><h2>Major section</h2>
        <h3>Subsection</h3>
      </section>
    </article>
    <aside>…</aside>  <!-- related/supplementary, not primary content -->
  </main>
  <footer>…</footer>
</body>
```

**Heading rules (hard):**

- **One `<h1>`** — the page/article headline only. It should match or closely align with the `<title>`.
- `<h2>` for major sections, `<h3>` for subsections. **Never skip a level** (H1→H3 with no H2 breaks the outline).
- **Never use a heading tag for visual styling.** Want big text that isn't a section? Use `<p class="lead">` + CSS, not `<h2>`.
- **Don't use headings for UI widget labels** ("Recent Posts", "Newsletter", sidebar titles). These pollute the semantic outline. Use `<p>`, `<span>`, or a visually-hidden heading scoped under an `aria-labelledby` region instead.
- The outline a crawler/screen-reader sees should read like a coherent table of contents on its own.

**Why it matters:** Google's heading parse and assistive-tech navigation both rely on a clean H1→H2→H3 tree. Lighthouse and the HTML5 Outliner flag breaks. See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) for the screen-reader side.

## Title tags & meta descriptions

**Title tag** — the single strongest on-page element you fully control (when Google keeps it):

- **50–60 characters / ~600px.** 51–55 chars is the sweet spot: Google rewrites it only ~40% of the time vs ~100% for titles over 70 chars.
- Front-load the primary keyword. Pattern: `Primary Keyword — Qualifier | Brand`.
- Unique per page. Duplicate titles signal thin/duplicate content.

```html
<title>Fluid Typography with clamp() — A Practical Guide | Codex</title>
<meta name="description" content="Build responsive type that scales smoothly between breakpoints using CSS clamp(). Formulas, pitfalls, and copy-paste snippets for production.">
```

**Meta description:**

- **Does not affect rankings.** Affects CTR only.
- **140–160 characters.** Google rewrites 60–70% of them regardless — write for the ~30% it keeps and for the snippet preview.
- **Blank beats duplicate.** A blank description lets Google pull a relevant snippet from body text; a duplicated one across pages reads as low quality.
- Include the keyword (it's bolded in SERPs when it matches the query) and a concrete value prop.

## Core Web Vitals as a ranking factor

CWV is a **confirmed tiebreaker, not a primary signal** — industry estimates put it at ~10–15% of ranking signals, applied mainly when content quality is otherwise equal. **Being in the red zone hurts; moving Lighthouse 79→99 does not move rank by itself.** The job is to exit "Poor," not to chase a perfect score.

| Metric | Good | Needs Improvement | Poor |
|---|---|---|---|
| **LCP** (Largest Contentful Paint) | ≤2.5s | 2.5–4s | >4s |
| **INP** (Interaction to Next Paint) | ≤200ms | 200–500ms | >500ms |
| **CLS** (Cumulative Layout Shift) | ≤0.1 | 0.1–0.25 | >0.25 |

- **Measured at the 75th percentile** of real-user page views (CrUX field data). Lab/Lighthouse numbers are diagnostic, not the ranking input.
- **INP replaced FID in March 2024.** Long tasks and heavy hydration are the usual culprits.
- **CLS:** The "Needs Improvement" threshold of 0.1–0.25 is where users begin to notice layout instability; 0.25+ is the "Poor" threshold that consistently degrades perceived quality.

**The three highest-leverage CWV fixes are design/markup decisions:**

1. **LCP** — preload the hero image, `fetchpriority="high"`, no lazy-load on it (see below).
2. **CLS** — explicit `width`/`height` on media; reserve space for banners/ads/embeds; `font-display: swap`.
3. **INP** — minimize JS on the critical path; defer non-essential scripts; avoid blocking the main thread on load.

Deep dive and budgets in [Web Performance & Core Web Vitals](./web-performance-core-web-vitals.md).

### LCP: preload, never lazy-load the hero

The single most common self-inflicted SEO wound is `loading="lazy"` on the LCP element. It can push LCP from ~2.1s to ~3.5s (Good→Needs Improvement).

```html
<!-- In <head>: preload the LCP image -->
<link rel="preload" as="image" href="/hero.avif" fetchpriority="high">

<!-- The hero image itself: eager + high priority, explicit dimensions -->
<img src="/hero.avif" width="1200" height="600" alt="…"
     fetchpriority="high" decoding="async">

<!-- Every below-the-fold image: lazy -->
<img src="/card.avif" width="400" height="300" alt="…" loading="lazy">
```

Rule: **preload LCP, lazy-load everything else.** Correctly applied, `loading="lazy"` improves LCP 15–30% on image-heavy pages — but never on the LCP element itself.

### CLS: reserve space before things load

```css
/* Prevent FOIT/FOUT layout shift from web fonts */
@font-face { font-family: "Inter"; src: url(/inter.woff2) format("woff2");
             font-display: swap; }
```

- **Always set `width`/`height`** (or `aspect-ratio`) on images, videos, iframes, ad slots.
- **Reserve space for late-injected UI** — cookie banners, promo bars, chat widgets. Injecting these after first paint without a reserved slot is a top CLS cause.
- Preload critical fonts: `<link rel="preload" as="font" type="font/woff2" crossorigin href="/inter.woff2">`.

## Mobile-friendliness & mobile-first indexing

**Google indexes the mobile version of your page — desktop-only testing is a trap.** If content, structured data, or links exist on desktop but not mobile, Google effectively doesn't see them.

- Parity: the **same content, headings, links, images, and structured data must be present on mobile** as on desktop. No "tap to load more" hiding indexable content behind interaction that Googlebot won't trigger.
- **No intrusive interstitials** above the fold on mobile — Google's policy applies a ranking demotion to pop-ups/overlays that block content on entry (announced 2017, still enforced). Implement consent banners as top-bar or bottom-bar elements that push content, not cover it.
- Touch targets and tap ergonomics — see [Touch Ergonomics & Cross-Device Design](../02-responsive-adaptive/touch-ergonomics-cross-device.md).
- Build mobile-first. See [Breakpoints, Devices & Mobile-First](../02-responsive-adaptive/breakpoints-devices-mobile-first.md).
- **Include `<meta name="viewport" content="width=device-width, initial-scale=1">` in every page `<head>`.** Its absence disables mobile scaling and triggers a mobile-usability warning in Search Console.

## Structured data (schema.org / JSON-LD)

Structured data makes pages eligible for **rich results** (stars, FAQs, breadcrumbs, recipe cards), which consistently generate higher CTR than standard blue-link results — exact lift varies by vertical and query type.

**Rules:**

- **JSON-LD is Google's recommended format.** Use it, not microdata/RDFa.
- **Inject it server-side, in `<head>`, in raw HTML before any JS runs.** Googlebot's renderer may not execute CSR-injected JSON-LD; AI crawlers won't at all.
- **Required fields only, and they must be accurate.** Google ignores types with incomplete required properties. Partial-but-accurate beats complete-but-fabricated.
- **Only ~30 of schema.org's 800+ types trigger visual rich results.** Prioritize: `Article`, `Product`, `FAQPage`, `HowTo`, `Organization`, `LocalBusiness`, `BreadcrumbList`, `Event`, `Recipe`, `VideoObject`.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Fluid Typography with clamp()",
  "author": { "@type": "Person", "name": "Jane Doe",
              "url": "https://example.com/authors/jane-doe" },
  "datePublished": "2026-06-11",
  "image": "https://example.com/og/clamp.png",
  "publisher": { "@type": "Organization", "name": "Codex",
                 "logo": { "@type": "ImageObject",
                           "url": "https://example.com/logo.png" } }
}
</script>
```

Validate with the **Rich Results Test** (Google's eligibility view) and **validator.schema.org** (stricter spec compliance).

## Open Graph & social cards

Social platforms read Open Graph (`og:`) tags to build link previews. **X/Twitter needs its own `twitter:card` tag — there is no OG fallback for the card type.** Without it, X renders the link as plain text even when OG tags exist.

```html
<!-- Open Graph: Facebook, LinkedIn, Slack, iMessage, Discord -->
<meta property="og:title" content="Fluid Typography with clamp()">
<meta property="og:description" content="Responsive type that scales smoothly. Formulas + snippets.">
<meta property="og:image" content="https://example.com/og/clamp.png"> <!-- absolute URL -->
<meta property="og:url" content="https://example.com/blog/fluid-type">
<meta property="og:type" content="article">

<!-- Twitter/X: required separately -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Fluid Typography with clamp()">
<meta name="twitter:description" content="Responsive type that scales smoothly.">
<meta name="twitter:image" content="https://example.com/og/clamp.png">
```

**Specs:**

- **OG image: exactly 1200×630px** (1200×675 / 16:9 also fine). Smaller may not trigger the large-card format on Facebook/LinkedIn.
- **Absolute URL required** — relative paths break previews everywhere.
- **`og:description` under ~110 chars** for clean display across platforms.
- Pages without OG images get a link-text-only preview on most platforms — losing essentially all visual CTR advantage from social shares.

Validate with the **Facebook Sharing Debugger** (also force-refreshes cached previews).

## URLs & information architecture

**Slug formula:** `[modifier-]primary-keyword`. Strip stop words (a, the, and, of). **3–5 meaningful words, ~25–30 chars.**

- **Keep total URL length under 60 characters.** Longer URLs get truncated in SERPs and are harder to share; top-ranked pages are disproportionately short. (No peer-reviewed lift number exists; the engineering reason is SERP display truncation, not a direct ranking signal.)
- **Hyphens, never underscores** — search engines treat `_` as a word-joiner (`blue_shoes` = one token), `-` as a separator.
- **No dates in slugs.** `/blog/2023/my-post` signals staleness and ages badly; `/blog/my-post` can be refreshed indefinitely.
- Lowercase, no parameters in the canonical path. Keep stable — every URL change needs a 301.

**IA & the 3-click rule:** every important page must be reachable within **3 clicks from the homepage.** Pages deeper than that are treated as lower priority. A shallow, logical hierarchy with breadcrumbs (`BreadcrumbList` schema) flows PageRank to what matters. Full treatment: [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md).

## Internal linking

**The only ranking factor fully within your control.** External backlinks you can't dictate; internal links you place every time you publish.

- Important pages ≤3 clicks from home; high-priority pages get the most internal links.
- **Descriptive anchor text.** "blue running shoes women" tells Google what the target is about; "click here" / "read more" tells it nothing.
- **No orphan pages** (zero internal links pointing in). Crawlers find them only via sitemap, they get no PageRank flow, and they rarely rank.
- Link target counts (rough): **pillar pages 8–15 links, standard cluster posts 3–8, high-priority pages 15+.**
- Build topic clusters: a pillar page links to cluster posts, and each cluster post links back to the pillar.

## Content & E-E-A-T basics

**E-E-A-T** = Experience, Expertise, Authoritativeness, **Trust** — and **Trust is Google's most important quality signal.** Design contributes concrete, visible trust signals:

- **Visible authorship:** real names, photos, bios, and dedicated author pages with credentials. This is a concrete on-page trust signal you design.
- **Generic bylines** ("Editorial Team", "Staff Writer") are a concrete *negative* signal on YMYL or authority-dependent content. Use a real person.
- **Author page minimum:** real name, photo, credentials/bio, and links to their other work or profiles.
- Surface dates (published + last-updated), sources, and contact/about pages. These are layout decisions with SEO weight.

## Image SEO & alt text

- **`alt` describes content + context in ≤125 chars.** Formula: *describe [subject] [action/context] in plain language.* Omit "image of" — it's redundant.
- **Decorative images get `alt=""`** (empty) — this tells screen readers to skip them. Don't omit the attribute entirely.
- **No keyword stuffing.** `alt="blue shoes, buy blue shoes, cheap blue shoes"` is spam to Google and noise to screen-reader users.
- **Modern formats:** WebP is 25–35% smaller than JPEG at equal quality; AVIF adds another 15–20% (slower to encode). Serve AVIF→WebP→JPEG via `<picture>` or a CDN.
- **Always set `width`/`height`** (CLS). Descriptive, hyphenated **file names** (`fluid-type-clamp.avif`, not `IMG_4821.jpg`) add minor signal.

```html
<picture>
  <!-- AVIF first: browser picks the first <source> it supports -->
  <source type="image/avif" srcset="/hero-800.avif 800w, /hero-1200.avif 1200w" sizes="100vw">
  <source type="image/webp" srcset="/hero-800.webp 800w, /hero-1200.webp 1200w" sizes="100vw">
  <img src="/hero-1200.jpg" width="1200" height="600"
       alt="Designer adjusting a clamp() formula in a code editor"
       fetchpriority="high">
</picture>
```

## JS rendering & SSR for SEO

**This is the architecture decision with the largest SEO consequence.** Choose the rendering strategy before writing components.

- **CSR-only SPA (React/Vue/Angular with no server render):** Googlebot crawls the empty shell, queues the page for JS rendering separately. For new/low-authority sites this can lag **3–7 days**. Content is invisible until then.
- **AI crawlers (ChatGPT, Perplexity, Claude) largely do not execute JavaScript.** No SSR/SSG = no AI citation, regardless of Google. This is increasingly as important as search ranking.
- **SSR or SSG is the only reliable architecture** for content that must be indexed promptly. Server-render (or statically generate) everything above the fold; hydrate for interactivity afterward.
- **Don't block JS/CSS in `robots.txt`** — Googlebot needs them to render. Blocking them produces broken renders and misjudged content.
- **Dynamic rendering** (serving different HTML to bots vs users) was **formally deprecated by Google in 2024–2025. Do not build new dynamic rendering.** Use SSR/SSG instead.

**Practical mapping:** marketing/blog/docs → SSG (Astro, Next.js `export`, Nuxt `generate`); app + content mix → SSR/ISR (Next.js, Nuxt, SvelteKit). See [Frameworks for Design-Forward Sites](../04-libraries-and-tools/frameworks-for-design.md).

## robots.txt, sitemaps & meta robots

### robots.txt

`robots.txt` controls which paths Googlebot may **crawl** — it does not control indexing. A page can be blocked in `robots.txt` and still appear in results (via backlinks) with no snippet. To prevent indexing, use `<meta name="robots" content="noindex">` or the `X-Robots-Tag` HTTP header instead.

```
User-agent: *
Disallow: /admin/
Disallow: /api/
Allow: /

# Never disallow CSS or JS — Googlebot needs them to render pages correctly
```

Critical rules:
- **Never block CSS or JS files.** Googlebot renders pages; blocking assets produces broken partial renders and leads to misjudged content quality.
- **Never `Disallow: /` on a production site.** It is a common deploy accident that makes the entire site disappear from Google within days.
- `robots.txt` is public — it reveals your site structure to attackers. Don't rely on it for security.

### `<meta name="robots">`

Fine-grained per-page indexing control. Inject server-side in `<head>`:

```html
<!-- Default (implicit — no tag needed if you want full indexing) -->
<meta name="robots" content="index, follow">

<!-- Block a page from appearing in results: -->
<meta name="robots" content="noindex, nofollow">

<!-- Allow indexing but control snippet/image display: -->
<meta name="robots" content="index, follow, max-snippet:-1, max-image-preview:large, max-video-preview:-1">
```

`max-image-preview:large` is required for Google to show large image thumbnails in Discover and image search. Add it to all indexable content pages.

### XML sitemap

Sitemaps help Google discover pages faster, especially on large or newly launched sites. Required fields: `<loc>` (absolute URL). Optional but useful: `<lastmod>` (ISO 8601 date, must be accurate — fake dates are ignored).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/blog/fluid-type</loc>
    <lastmod>2026-06-11</lastmod>
  </url>
</urlset>
```

- Keep the sitemap under 50,000 URLs / 50MB uncompressed; split into a sitemap index file beyond that.
- Submit via Google Search Console (Sitemaps report). Reference it in `robots.txt`: `Sitemap: https://example.com/sitemap.xml`.
- Exclude `noindex` pages from the sitemap — sending Google a URL in the sitemap and also marking it `noindex` sends a contradictory signal.

## Canonicalization & redirects

- **`rel=canonical` on every page with parameters, tracking UTMs, session IDs, or filter combinations**, pointing to the clean, parameter-free URL. On e-commerce filter pages this prevents thousands of duplicate-content variants from diluting link equity.
- Self-referencing canonical on normal pages is fine and recommended.
- **Single-hop redirects only.** Chains (A→B→C) leak equity and slow crawl. Always **301** for permanent moves.

```html
<link rel="canonical" href="https://example.com/shop/shoes">
<!-- on /shop/shoes?color=blue&utm_source=email -->
```

## hreflang & internationalization

If the site serves the same content in multiple languages or regional variants, `hreflang` tells Google which version to show which user. Without it, Google guesses — and often surfaces the wrong locale in results, or treats variants as duplicates.

```html
<!-- On every language variant of a page, list all variants including itself -->
<link rel="alternate" hreflang="en" href="https://example.com/shoes">
<link rel="alternate" hreflang="de" href="https://example.com/de/schuhe">
<link rel="alternate" hreflang="fr" href="https://example.com/fr/chaussures">
<link rel="alternate" hreflang="x-default" href="https://example.com/shoes">
```

Rules:
- **Every variant must reference all other variants** (including itself). A one-sided `hreflang` that isn't reciprocated is ignored.
- `x-default` is the fallback for users whose language isn't covered — typically the primary/English version.
- Use BCP 47 language codes: `en`, `en-US`, `en-GB`, `de`, `pt-BR`, etc.
- Alternative: ship `hreflang` in the XML sitemap instead of `<head>` — cleaner for large sites. Google accepts both; never use both simultaneously on the same page.
- `hreflang` is a **hint**, not a directive — Google may override it. Canonical must still be consistent.

## Avoiding SEO-killing design choices

These are common design defaults that silently destroy discoverability:

- Lazy-loading the hero/LCP image.
- Building public marketing pages as a CSR-only SPA.
- Heading tags used for sidebar/widget labels or visual sizing.
- Images with no `width`/`height` → CLS.
- Late-injected banners/widgets with no reserved space → CLS.
- Intrusive mobile pop-ups above the fold.
- Infinite scroll or "load more" hiding indexable content behind interaction Googlebot won't perform (provide paginated, linkable URLs too).
- Text baked into images (uncrawlable) — keep important copy as real HTML text.
- Orphan pages and deep (>3-click) burial.
- Using `robots.txt` to prevent indexing (it prevents crawling, not indexing — use `noindex` meta).
- Accidentally deploying `Disallow: /` to production.
- Omitting `max-image-preview:large` on content pages — Google won't show large image thumbnails in Discover.
- Missing `<meta name="viewport">` — disables mobile scaling, triggers Search Console mobile-usability warning.
- Serving multi-language content without `hreflang` — Google treats variants as duplicates or surfaces the wrong locale.

## Concrete specs & numbers

| Item | Spec |
|---|---|
| LCP | Good ≤2.5s · NI 2.5–4s · Poor >4s (75th pct) |
| INP | Good ≤200ms · NI 200–500ms · Poor >500ms |
| CLS | Good ≤0.1 · NI 0.1–0.25 · Poor >0.25 |
| Title tag | 50–60 chars (~600px); 51–55 sweet spot; >70 ≈100% rewritten |
| Meta description | 140–160 chars; 60–70% rewritten by Google |
| URL slug | 3–5 words, ~25–30 chars; keep under 60 chars total |
| OG image | 1200×630px (or 1200×675 / 16:9); absolute URL |
| OG description | <110 chars |
| alt text | ≤125 chars; `alt=""` for decorative |
| WebP savings | 25–35% vs JPEG |
| AVIF savings | +15–20% over WebP |
| Internal links | pillar 8–15 · cluster 3–8 · high-priority 15+ |
| Click depth | important pages ≤3 clicks from home |
| Redirects | single-hop 301 only |
| CWV ranking weight | ~10–15% of signals; tiebreaker |
| Viewport meta | `width=device-width, initial-scale=1` — required for mobile |
| meta robots (content pages) | `index, follow, max-image-preview:large, max-snippet:-1` |
| Sitemap max size | 50,000 URLs / 50MB uncompressed per file |
| hreflang | required on all variants for multi-language; must be reciprocal |

**Field-data context:** Only ~48% of mobile and ~56% of desktop pages pass all three CWV (Web Almanac 2025); LCP is the most-failed metric (~62% of mobile pages pass it). INP is the newest metric — passing rates vary significantly by JS framework weight.

## Tools & libraries

- **Google Search Console** — the source of truth: CrUX field CWV, rich-result eligibility, mobile usability, indexing status, URL Inspection. Use first and continuously.
- **PageSpeed Insights** — real-user CrUX + lab data; the authoritative CWV source. Trust this over GTmetrix or third-party lab tools.
- **Lighthouse** (Chrome DevTools) — SEO/perf/a11y/best-practices audit; flags missing landmarks and bad heading order. Lab numbers are diagnostic, not the ranking input.
- **Rich Results Test** (search.google.com/test/rich-results) — validate JSON-LD + preview rich results.
- **Schema Markup Validator** (validator.schema.org) — stricter spec check than Google's.
- **Screaming Frog** / **Sitebulb** — crawlers: broken links, redirect chains, orphans, duplicate meta, missing alt; Sitebulb adds visual architecture maps. (Check current pricing on vendor sites.)
- **Facebook Sharing Debugger** — validate OG and force-refresh cached previews.
- **Squoosh** (squoosh.app) — browser-based image compression/format conversion.
- **Image CDNs** (Cloudinary, imgix, Bunny, Cloudflare Images, Next.js Image) — auto-negotiate AVIF→WebP→JPEG, resize on the fly, edge-cache.
- **Frameworks:** Astro (zero-JS default, Lighthouse 100 with minimal effort — ideal for marketing/blog), Next.js (React SSR/SSG standard), Nuxt (Vue), SvelteKit.
- **WordPress:** Yoast SEO / RankMath manage titles, meta, sitemaps, OG, JSON-LD without code.
- **eslint-plugin-jsx-a11y** — enforces semantic HTML in React/JSX at build time.
- **Ahrefs / Semrush** — keyword research, backlinks, site health. **DeepCrawl / Lumar** — enterprise crawls (millions of pages).
- **When NOT to:** don't optimize against GTmetrix/third-party lab scores for ranking — only CrUX field data counts. Don't build dynamic rendering even though tooling for it exists.

## Do / Don't

**Do**

- Render content server-side (SSR/SSG) so it's in raw HTML for Google *and* AI crawlers.
- Ship one `<h1>`, a clean H2/H3 outline, and semantic landmarks.
- Preload the LCP image (`fetchpriority="high"`); lazy-load the rest.
- Set `width`/`height` (or `aspect-ratio`) on all media; `font-display: swap`.
- Inject accurate JSON-LD server-side in `<head>`; validate before shipping.
- Write unique 50–60 char titles and 140–160 char descriptions per page.
- Use real, credentialed, visible authors.
- Place descriptive internal links; keep everything ≤3 clicks from home.
- Include `<meta name="viewport">` and `<meta name="robots" content="index,follow,max-image-preview:large,max-snippet:-1">` on content pages.
- Add `hreflang` on every variant for multi-language sites; include `x-default`.
- Submit sitemap via Search Console; reference it in `robots.txt`.

**Don't**

- Build public marketing pages as a CSR-only SPA.
- Put `loading="lazy"` on the hero/LCP image.
- Use heading tags for styling or widget labels.
- Ship images without dimensions or with stuffed/missing alt.
- Duplicate titles/descriptions across pages.
- Use dates or underscores in slugs.
- Block JS/CSS in `robots.txt`; never deploy `Disallow: /` to production.
- Implement new dynamic rendering, or chain redirects.
- Use `robots.txt` to block indexing — use `<meta name="robots" content="noindex">` instead.
- Include `noindex` pages in your XML sitemap.

## Common pitfalls & how to avoid them

- **Lazy hero image** → LCP 2.1s→3.5s. *Fix:* `fetchpriority="high"`, no `loading="lazy"` on LCP.
- **CSR-only SPA for marketing** → 3–7 day index lag, zero AI citation. *Fix:* SSR/SSG.
- **Headings for widgets/styling** → polluted outline. *Fix:* `<p>`/`<span>` + CSS, or visually-hidden region headings.
- **Missing image dimensions / late-injected UI** → CLS. *Fix:* set `width`/`height`; reserve space for banners/widgets.
- **Dates in slugs** → artificial staleness. *Fix:* dateless slugs.
- **Titles >70 chars** → ~100% rewritten. *Fix:* 50–60 chars.
- **Duplicate descriptions** → worse than blank. *Fix:* unique or blank.
- **Underscores in slugs** → words merged. *Fix:* hyphens.
- **Keyword-stuffed alt** → spam + bad UX. *Fix:* one accurate ≤125-char description.
- **Missing `twitter:card`** → plain-text X links. *Fix:* add it explicitly.
- **JSON-LD only in CSR content** → may not execute. *Fix:* server-side in `<head>`.
- **Wrong/missing canonical on filter pages** → thousands of duplicates. *Fix:* canonical to clean URL.
- **Generic bylines on YMYL** → negative E-E-A-T. *Fix:* real authors.
- **No `font-display: swap`** → FOIT/FOUT → CLS. *Fix:* add it; preload fonts.
- **Orphan pages** → no PageRank, no ranking. *Fix:* internal-link every page.
- **Blocked JS/CSS in robots.txt** → broken renders. *Fix:* allow them.
- **`robots.txt` used to suppress indexing** → page still appears (via backlinks) with no snippet. *Fix:* `<meta name="robots" content="noindex">`.
- **`Disallow: /` deployed to production** → entire site vanishes from Google within days. *Fix:* audit deploy pipeline; stage/prod `robots.txt` must differ.
- **Multi-language site without hreflang** → wrong locale shown in results; variants treated as duplicates. *Fix:* reciprocal `hreflang` on every variant.
- **Missing `<meta name="viewport">`** → Search Console mobile-usability error; mobile users see desktop-scaled layout. *Fix:* add to every page template.
- **`noindex` pages in sitemap** → contradictory signals; Google ignores the sitemap hint but wastes crawl budget. *Fix:* exclude `noindex` URLs from sitemap.
- **Missing `max-image-preview:large`** → images won't show as large thumbnails in Discover. *Fix:* add to `<meta name="robots">` on content pages.

## Recurring failure patterns in the wild

Common root causes seen repeatedly across practitioner post-mortems and Search Console incidents:

- **CSR SPA launched without SSR:** pages sit unindexed for 3–7 days; newly published content is invisible to Google until the JS-render queue processes it.
- **Content visible in Google, absent from AI answers:** AI crawlers not executing JS; SSR/SSG was the fix.
- **Lighthouse 79→99, rankings unchanged:** CWV is a tiebreaker among near-equal pages — escaping "Poor" matters; chasing lab scores above the "Good" threshold does not.
- **Intrusive mobile interstitial → organic traffic drop:** Google's policy applies a ranking demotion to pop-ups that block content on mobile entry. Implement cookie/promo banners as banners that push content down, not overlays.
- **Google rewrites every title:** titles over 70 chars get rewritten ~100% of the time; keep them 50–60 chars.
- **E-commerce filter pages creating thousands of duplicate URLs:** missing `rel=canonical` on parameterized pages; fix is canonical pointing to the clean faceted-navigation URL.
- **CLS spike on production:** late-injected cookie banner with no reserved slot; fix is CSS `min-height` reservation before the banner loads.

## Senior-level checklist

- [ ] Rendering strategy chosen (SSR/SSG); above-the-fold content present in raw HTML (view-source confirms).
- [ ] Exactly one `<h1>`; H2/H3 outline has no skipped levels; semantic landmarks present.
- [ ] No heading tags used for styling or widget labels.
- [ ] LCP image preloaded with `fetchpriority="high"`; no `loading="lazy"` on it; all below-fold images lazy.
- [ ] All images/videos/iframes have `width`/`height` or `aspect-ratio`; `font-display: swap` set.
- [ ] CWV field data (CrUX/PSI) in "Good" at 75th pct for LCP/INP/CLS, or at minimum out of "Poor."
- [ ] Mobile parity verified: same content, links, and structured data as desktop; no intrusive interstitials.
- [ ] JSON-LD injected server-side in `<head>`, required fields accurate, passes Rich Results Test.
- [ ] Unique title 50–60 chars and description 140–160 chars per page.
- [ ] OG tags + separate `twitter:card`; OG image 1200×630, absolute URL; validated in Sharing Debugger.
- [ ] Slugs: hyphenated, ≤5 words, no dates/underscores, under 60 chars.
- [ ] `rel=canonical` on every parameterized/duplicate URL; redirects single-hop 301.
- [ ] Every important page ≤3 clicks from home; no orphan pages; descriptive anchor text.
- [ ] All images have accurate ≤125-char alt (or `alt=""` if decorative); modern formats (AVIF/WebP).
- [ ] Real, credentialed, visible authorship on content/YMYL pages.
- [ ] `<meta name="viewport" content="width=device-width, initial-scale=1">` present in every page `<head>`.
- [ ] `<meta name="robots">` correct: `noindex` on pages that should not appear in results; `max-image-preview:large` on content pages.
- [ ] `robots.txt` allows JS/CSS; no accidental `Disallow: /`; references sitemap URL.
- [ ] XML sitemap valid, submitted to Search Console, excludes `noindex` pages.
- [ ] Multi-language sites: `hreflang` on every variant, reciprocal, with `x-default`.
- [ ] Sitemap submitted; Search Console clean of indexing/mobile usability errors.

## Sources

- Google Search Central — Core Web Vitals & page experience: https://developers.google.com/search/docs/appearance/page-experience
- web.dev — Core Web Vitals (LCP, INP, CLS): https://web.dev/articles/vitals
- web.dev — INP: https://web.dev/articles/inp
- Google Search Central — structured data / JSON-LD: https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data
- Google Search Central — hreflang (i18n): https://developers.google.com/search/docs/specialty/international/localized-versions
- Google Search Central — robots.txt: https://developers.google.com/search/docs/crawling-indexing/robots/intro
- Google Search Central — sitemaps: https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview
- Google Search Central — robots meta tag / X-Robots-Tag: https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag
- Google Rich Results Test: https://search.google.com/test/rich-results
- Schema.org: https://schema.org/
- Schema Markup Validator: https://validator.schema.org/
- The Open Graph protocol: https://ogp.me/
- X/Twitter Cards docs: https://developer.x.com/en/docs/x-for-websites/cards/overview/about-cards
- Google PageSpeed Insights: https://pagespeed.web.dev/
- Web Almanac (HTTP Archive): https://almanac.httparchive.org/
- Astro: https://astro.build/
- Next.js: https://nextjs.org/
- Squoosh: https://squoosh.app/

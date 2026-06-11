# Single-Page Sites & One-Pagers

> One-page sites compress an entire story into a single scrollable document: hero → proof → action, navigated by anchors and scrollspy. This guide covers when one-pager scope fits, section ordering, anchor/scrollspy/smooth-scroll mechanics, the load-everything-at-once performance budget, SEO limits and mitigations, accessible focus management, and the long-scroll narrative — so an agent ships a focused, fast, ranking page instead of an unstructured scroll-tunnel.

**Apply when:** content scope is one focused goal (portfolio, event, single product, MVP, brand statement) and linear narrative beats a navigation hierarchy. · **Related:** [Landing Pages](./landing-pages.md) · [Scroll-Driven Experiences](./scroll-driven-experiences.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md)

## TL;DR — rules at a glance

- One page = one primary CTA goal. Never split intent ("call now" here, "download guide" there). Repeat the *same* CTA at hero, mid-page, and footer.
- Section order is promise → proof → action: Hero → Problem → Solution → Social Proof → Benefits/Offer → CTA → Footer.
- Add anchor `id`s before visual design — navigation shapes the build, not the reverse.
- One `<h1>` for the page, one descriptive `<h2>` per section. Never skip heading levels. ~70% of screen-reader users navigate long pages by headings.
- Everything loads at once, so the performance budget is the whole page: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1. Lazy-load below-fold images, never the LCP image.
- Free smooth scroll = `html { scroll-behavior: smooth }`. Free sticky nav = `position: sticky; top: 0`. Reach for JS only when CSS can't do it.
- `scrollIntoView({behavior:'smooth'})` and anchor jumps leave keyboard focus behind — manually `.focus()` the target with `tabindex="-1"`.
- Scrollspy active state: `IntersectionObserver`, not scroll-event listeners. If you must use scroll events, add `{passive:true}`.
- Limit sticky nav to ~5–7 essential links. Add a "Skip to main content" link. Disable sticky on short viewports.
- Respect `prefers-reduced-motion` — decorative parallax/fades become motion sick triggers and INP hits on mobile.
- Don't scroll-jack. Forced scroll sequences lock users out of contact/pricing and are a top abandonment cause.
- One-pager ≠ scrollytelling. Animation triggered by scroll without narrative intent adds weight, not story.
- SEO ceiling: a single page targets ~1 small keyword cluster. If you need multiple dedicated keyword targets, go multi-page or hybrid.
- Every one-pager ships with `<meta name="viewport">`, `<link rel="canonical">`, and OG meta — without them the mobile layout breaks and social shares are blank.
- Anchor clicks: use `history.pushState` (not `replaceState`) so browser Back returns to the previous section.

## When a one-pager fits (and when it doesn't)

**Fits when:**
- Scope is small and the story is linear: event page, single product, MVP/coming-soon, personal portfolio, brand/manifesto page, app download page, wedding/conference microsite.
- There is exactly one conversion goal (sign up, buy, RSVP, contact).
- The target keyword cluster is narrow — and a search-intent check shows Google already ranks one-page results for that query (if it ranks them, yours can rank too).
- Maintenance overhead must stay low (one page to keep current vs. a whole site).

**Doesn't fit when:**
- Content needs multiple dedicated keyword targets (e.g., separate ranking pages per product/service). A single page caps you at ~1 small cluster.
- Users repeatedly need deep reference content (docs, large catalogs, multi-step flows).
- There's a real catalog: forcing per-product detail into one scroll produces the classic complaint — "what should have been a page per product was one long scroll never meant for a computer."
- Commerce with browse → detail → back loops: if back navigation doesn't restore scroll position, users rage-abandon. One-pagers make this worse, not better.

**Hybrid (recommended for growth):** one-page homepage + supporting pages (blog, service pages) when organic traffic is the main growth channel. Build it on a stack that grows (Astro, 11ty) rather than a tool you'll have to escape later. **Carrd and similar no-code one-page platforms are deliberately single-page only** — if you anticipate adding a blog, service pages, or routing, start on Astro/11ty immediately.

## Section structure & narrative flow

A one-pager is a guided argument. Each section answers the question the previous one raised. Write the story as plain prose *before* any visual design; storyboard before high-fidelity mockups.

Canonical order — **promise → proof → action**:

| # | Section | Job | Must contain |
|---|---------|-----|--------------|
| 1 | **Hero** | Promise + above-fold CTA | `<h1>`, value prop, primary CTA #1 |
| 2 | **Problem** | Name the pain | the user's words, not yours |
| 3 | **Solution** | Show the mechanism | how it works, 1–3 steps |
| 4 | **Social proof** | De-risk | logos, testimonials, numbers |
| 5 | **Benefits / Offer** | Translate features → outcomes | value-driven copy, primary CTA #2 |
| 6 | **CTA section** | Final conversion ask | primary CTA #3 |
| 7 | **Footer** | Contact, legal, secondary nav | reachable without three miles of scroll |

Rules:
- **One offer beats five competing messages.** Narrow the message; guide attention section by section.
- **CTAs repeat by buyer readiness.** Hero CTA catches the already-convinced; mid-page CTA catches those who needed benefits/proof first; footer CTA catches the thorough reader. A single CTA loses everyone who needed more context before the one button.
- **Don't bury contact.** "Want to contact us? Better scroll down three miles!" is a real, repeated complaint. Put contact/pricing in the nav and footer; never gate them behind a mandatory scroll.

### Semantic HTML skeleton

```html
<a class="skip-link" href="#main">Skip to main content</a>
<header>
  <nav aria-label="Primary">
    <a href="#hero" class="logo">Brand</a>
    <ul>
      <li><a href="#solution">How it works</a></li>
      <li><a href="#proof">Customers</a></li>
      <li><a href="#pricing">Pricing</a></li>
      <li><a href="#contact" class="cta">Get started</a></li>
    </ul>
  </nav>
</header>
<main id="main">
  <section id="hero"      aria-labelledby="hero-h">     <h1 id="hero-h">…</h1> …</section>
  <section id="problem"   aria-labelledby="problem-h">  <h2 id="problem-h">…</h2> …</section>
  <section id="solution"  aria-labelledby="solution-h"> <h2 id="solution-h">…</h2> …</section>
  <section id="proof"     aria-labelledby="proof-h">    <h2 id="proof-h">…</h2> …</section>
  <section id="pricing"   aria-labelledby="pricing-h">  <h2 id="pricing-h">…</h2> …</section>
  <section id="contact"   aria-labelledby="contact-h">  <h2 id="contact-h">…</h2> …</section>
</main>
<footer>…</footer>
```

- One `<h1>`. One `<h2>` per `<section>`. Sub-points use `<h3>` — never jump `<h2>` → `<h4>`.
- `aria-labelledby` ties each landmark `<section>` to its heading so it's announced meaningfully.

## Anchor navigation + scrollspy

### Smooth scroll (zero JS)

```css
html { scroll-behavior: smooth; }
/* offset anchors so a sticky header doesn't cover the target heading */
/* scope to section[id] — [id] alone matches every id'd element on the page */
section[id], [data-anchor] { scroll-margin-top: 5rem; }
@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
}
```

`scroll-behavior: smooth` is supported in all current evergreen browsers. `scroll-margin-top` prevents the sticky nav from overlapping the landed-on heading — a near-universal anchor bug. Scope the selector to `section[id]` (or your anchor elements), not `[id]` globally, which would apply the offset to every id'd element on the page.

### Scrollspy (active link) — use IntersectionObserver

`IntersectionObserver` outperforms `window` scroll-event listeners; it fires only when a section crosses a threshold, off the main scroll path.

```js
const links = new Map(
  [...document.querySelectorAll('nav a[href^="#"]')].map(a => [a.hash.slice(1), a])
);
const obs = new IntersectionObserver((entries) => {
  for (const e of entries) {
    if (e.isIntersecting) {
      links.forEach(a => a.removeAttribute('aria-current'));
      links.get(e.target.id)?.setAttribute('aria-current', 'true');
    }
  }
}, { rootMargin: '-45% 0px -45% 0px', threshold: 0 });

document.querySelectorAll('main section[id]').forEach(s => obs.observe(s));
```

- `rootMargin: '-45% 0px -45% 0px'` makes a section "active" when it occupies the middle 10% band of the viewport — feels natural and avoids flicker between adjacent sections. **Edge case:** the very first and last sections may never hit this band at page top/bottom. Mitigate by also marking the first nav link active on load and the last on scroll-end, or widen the band (e.g., `-30% 0px -30% 0px`) if sections are short.
- Style the active state off `aria-current` (semantic + free a11y): `nav a[aria-current="true"] { … }`. Use `aria-current="true"` (not `aria-current="page"` — that signals the current page in multi-page nav, not section within a page).

### Focus management — the bug everyone ships

Native `#anchor` jumps and `scrollIntoView({behavior:'smooth'})` move the *viewport* but leave keyboard focus on the originating link. Sighted keyboard users get scrolled while their focus stays behind. Fix by focusing the target heading:

```js
document.querySelectorAll('nav a[href^="#"]').forEach(link => {
  link.addEventListener('click', (e) => {
    const id = link.hash.slice(1);
    const target = document.getElementById(id);
    if (!target) return;
    e.preventDefault();
    const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;
    target.scrollIntoView({ behavior: reduce ? 'auto' : 'smooth', block: 'start' });
    target.setAttribute('tabindex', '-1');   // make non-interactive element focusable
    target.focus({ preventScroll: true });   // don't fight the smooth scroll
    history.pushState(null, '', '#' + id);  // update URL + create history entry so Back works
  });
});
```

If anchor nav swaps in content without a full reload, move focus to the target heading and announce the change via an ARIA live region — screen-reader users expect a page-load-like notification.

## Sticky nav

```css
header {
  position: sticky;
  top: 0;
  z-index: 100;
}
/* disable sticky when the viewport is too short to spare the space */
@media (max-height: 300px) {
  header { position: static; }
}
```

- On the classic "sticky `ul` inside `nav`" pattern, `position: sticky` needs a parent **taller** than the sticky element to have room to stick. (`-webkit-sticky` was required for old iOS before Safari 13/2019; treat it as dead code for new builds.)
- **Firefox gotcha:** `body { overflow-x: hidden }` silently breaks `position: sticky`. If sticky "doesn't work," check for `overflow` on ancestors first.
- Limit to ~5–7 essential links; overloading the nav with every section defeats it. Don't hide desktop nav behind a hamburger for cosmetics — it raises task time for power users.
- **Mobile:** a bottom-of-screen sticky CTA bar converts well and stays thumb-reachable. Keep it from covering content — reserve its height with padding on `main`.
- Test for jitter on scroll; add `will-change: transform` only where a measured repaint problem exists, never blanket.

## Performance — everything loads at once

A one-pager has no second page to defer to: the entire DOM, every image, and every animation library are in the first document. The whole page is your LCP/INP/CLS budget.

- **LCP element (usually the hero image/headline):** `fetchpriority="high"`, never `loading="lazy"`. Preload it if it's a background or late-discovered image.
- **Below-fold images:** `loading="lazy"` **plus** explicit `width`/`height` (or `aspect-ratio`) — lazy without dimensions causes CLS as placeholders expand.
- **Reserve space for everything dynamic** (ads, embeds, lazy media) before it loads. Unreserved ad slots produce the "page constantly jumps around while ads load" abandonment.
- **Defer heavy scroll/animation libs** (GSAP, Lenis, Three.js). Load them after first paint, or only when their section nears the viewport (`IntersectionObserver` again).
- **Mobile motion budget:** complex parallax and long GSAP timelines hurt INP and "feel broken" on phones. Simplify or disable on small/touch devices.

```html
<!-- LCP hero -->
<img src="/hero-1600.webp" width="1600" height="900"
     fetchpriority="high" alt="…" />

<!-- below-fold -->
<img src="/proof-800.webp" width="800" height="600"
     loading="lazy" decoding="async" alt="…" />
```

Why it pays: per Google/DoubleClick research (2018), 53% of mobile site visits were abandoned when pages took longer than 3 s to load. Deloitte/Google (2020) found 0.1 s speed improvement correlated with ~1% e-commerce revenue uplift. Both figures are widely cited as directional benchmarks; treat them as orders-of-magnitude, not guarantees.

## SEO on one page — limits and mitigations

**The hard ceiling:** a single URL can rank for roughly one tight keyword cluster. You can't give five products five dedicated, separately-ranking pages. If you need that, the page type is wrong — go multi-page or hybrid.

**Mitigations that work within one page:**
- **Semantic structure is non-negotiable.** One `<h1>`, descriptive `<h2>` per section. Without an `<h1>`/`<h2>` hierarchy, search engines get no topical context from a wall of text.
- **Front-load the primary keyword** in `<title>`, `<h1>`, and the first section's prose.
- **Canonical URL:** add `<link rel="canonical" href="https://example.com/">` to collapse UTM/ref variants onto the canonical URL. One-pagers get linked with tracking parameters frequently; without canonical, link equity fragments.
- **Open Graph + Twitter Card meta:** one-pagers are often shared directly (events, products, portfolios). Without OG tags the share preview is blank or wrong:
  ```html
  <meta property="og:title" content="…" />
  <meta property="og:description" content="…" />
  <meta property="og:image" content="https://example.com/og-cover.jpg" />
  <meta property="og:url" content="https://example.com/" />
  <meta name="twitter:card" content="summary_large_image" />
  ```
  OG image: 1200×630 px, ≤300 KB. Missing OG is the single most-noticed omission when a one-pager gets posted on social/Slack.
- **Viewport meta:** `<meta name="viewport" content="width=device-width, initial-scale=1">` must be present. Absent it, mobile browsers zoom out, breaking layout and triggering CLS.
- **Check intent first:** if Google already surfaces one-page results for the target query, a one-pager is competitively viable. If results are all deep multi-page sites, reconsider.
- **Section deep-links:** anchor URLs (`/#pricing`) can earn jump-to-section sitelinks and let you share section-specific links.
- **Structured data:** add JSON-LD (`Organization`, `Product`, `Event`, `FAQPage`) — schema doesn't need multiple pages.
- **Hybrid for organic growth:** keep the one-page homepage, add a `/blog` or service pages for the long-tail. This is the standard move when SEO is the main channel.

## One-pager vs. scrollytelling — don't confuse them

A long-scroll one-pager is content stacked in sections. **Scrollytelling** is a narrative framework where scroll *is* the timeline controller. It qualifies only if all three hold:
1. **Narrative intent** — beginning, tension, reveal, resolution.
2. **Scroll–content synchronization** — content state is bound to scroll position.
3. **Experience logic** — *remove the scroll and the narrative collapses.* If it survives, it was decoration.

"Animation triggered by scroll" ≠ scrollytelling. Scroll-triggered fades without narrative intent add page weight, not story. **Parallax is a visual effect** (layered depth); scrollytelling is a framework that *may* use parallax but isn't defined by it. For the full pinning/scrubbing/parallax toolkit, see [Scroll-Driven Experiences](./scroll-driven-experiences.md) and the [Playbook: Build a Scroll-Driven One-Pager](../09-playbooks-and-checklists/playbook-scroll-experience.md). Note: engagement figures cited for scrollytelling campaigns (e.g., "NYT Snow Fall 12-min session") are case-specific and not reproducible benchmarks — do not cite them as norms.

**Never scroll-jack.** Hijacking the scrollbar into mandatory sequences ("tunnel-lock") blocks access to contact/pricing and is a top UX failure. The scrollbar is the user's; you may set narrative intent on it, not seize it.

## Concrete specs & numbers

Numbers in this section come from primary specs or widely-cited industry research. Stats without a traceable primary source have been removed.

- **Core Web Vitals (Google thresholds):** LCP ≤ 2.5s · INP ≤ 200ms (replaced FID March 2024) · CLS ≤ 0.1.
- **Mobile abandonment:** 53% of mobile visits abandoned past 3 s load — Google/DoubleClick "Find Out How You Stack Up" (2018); directional benchmark, methodology not public.
- **Speed → revenue:** Deloitte/Google "Milliseconds Make Millions" (2020) — 0.1 s improvement correlated with ~1% e-commerce revenue uplift. Treat as order-of-magnitude signal.
- **Touch targets:** 44×44 px (Apple HIG) · 48×48 dp (Material Design) · 24×24 CSS px minimum (WCAG 2.5.8 AA).
- **Focus indicators:** ≥ 3:1 contrast ratio against adjacent colors (WCAG 2.2 Focus Appearance). Replace default outlines, never remove them.
- **Heading navigation:** ~67–72% of screen-reader users navigate long pages by headings (WebAIM Screen Reader User Survey #9 and #10, 2021–2024).
- **Sticky nav:** `position: sticky; top: 0`; parent must be taller than the sticky element; `-webkit-sticky` not needed for Safari ≥ 13 (2019); disable under ~300 px viewport height.
- **Nav size:** 5–7 essential links maximum; more and users stop scanning.
- **OG image:** 1200×630 px, ≤ 300 KB; required for useful social previews.

## Tools & libraries

**Native first (reach for these before JS):**
- `scroll-behavior: smooth` (CSS) — zero-JS smooth anchor scrolling on `html`.
- `position: sticky` (CSS) — sticky nav with no JS.
- `IntersectionObserver` — scrollspy active state and lazy-init of heavy libs; replaces scroll-event listeners.
- `scrollIntoView({behavior:'smooth', block:'start'})` — JS smooth scroll *with manual focus management required*.

**Animation / smooth-scroll (only if narrative warrants):**
- **GSAP + ScrollTrigger** — industry standard for pinning, scrubbing, parallax, horizontal scroll, SVG morph, Lottie. Use for real scrollytelling, not fades.
- **Lenis** — lightweight modern smooth-scroll; pairs with GSAP; the current default (over Locomotive Scroll for new builds).
- **Locomotive Scroll v5** — established smooth-scroll + parallax; common in Next.js + GSAP builds.
- **Framer Motion** — React-native, good for component-level scroll-triggered animation.
- **Three.js / WebGL** — cinematic/immersive 3D one-pagers (see [WebGL, 3D & Shaders](../04-libraries-and-tools/webgl-3d-shaders.md)). Performance ceiling is high *with skilled teams* — costly otherwise.

**Build stacks:**
- **Astro** — top practitioner pick for one-pagers that may grow; zero-JS by default, Tailwind + Markdown.
- **Eleventy (11ty) / Hugo + Caddy** — fast static one-pagers; 11ty if you'll add templating/blog later.
- **Vite + Tailwind** — lightweight toolchain for vanilla HTML/CSS/JS.
- **Webflow** — native scroll interactions; "a skilled team on Webflow beats an average team on a bespoke stack."
- **Carrd** — purpose-built no-code one-page platform.
- **Bootstrap 5 / simplecss.org** — fast generic one-pagers when visual distinctiveness isn't the goal (accept template look).
- **Hosting:** GitHub Pages / Cloudflare Pages / Netlify — free static hosting, strong practitioner consensus.

**When NOT to use:** don't ship GSAP/Three.js for a portfolio that's really just sections of text and images — it's weight without story. Don't pick a no-code tool you'll have to migrate off once SEO/scale demands multi-page; start on Astro/11ty instead. See [Animation Libraries Stack](../04-libraries-and-tools/animation-libraries-stack.md) and [Frameworks for Design](../04-libraries-and-tools/frameworks-for-design.md).

## Do / Don't

**Do**
- Define one primary CTA goal; repeat it at hero, mid-page, and footer.
- Add anchor `id`s during structure, before visual design.
- One `<h1>`, one descriptive `<h2>` per section, no skipped levels.
- Use `IntersectionObserver` for scrollspy and lazy-init; `{passive:true}` if you must use scroll events.
- Manage focus on every anchor jump (`tabindex="-1"` + `.focus({preventScroll:true})`); use `history.pushState` so browser Back works.
- Scope `scroll-margin-top` to `section[id]`, not `[id]` globally.
- Lazy-load below-fold images with explicit `width`/`height`; reserve space for all dynamic content.
- Respect `prefers-reduced-motion`; provide a static path where motion conveyed information.
- Keep sticky nav to 5–7 links; add a "Skip to main content" link.
- Add `<link rel="canonical">`, OG/Twitter Card meta, and `<meta name="viewport">`.
- Check search intent before committing to one URL.

**Don't**
- Split CTA intent across sections ("call now" / "download guide").
- Apply `loading="lazy"` to the hero/LCP image.
- Ship lazy images without dimensions (CLS).
- Remove focus indicators (replace them, ≥3:1 contrast).
- Scroll-jack or force mandatory scroll sequences.
- Confuse scroll-triggered animation with scrollytelling.
- Overload sticky nav or hide desktop nav behind a hamburger for looks.
- Bury contact/pricing behind miles of scroll.
- Force a real catalog or multi-keyword content into one page.
- Use `history.replaceState` for anchor clicks (breaks browser Back).
- Target `[id]` globally for `scroll-margin-top` (affects every id'd element).
- Cite unverifiable engagement stats for scrollytelling (they are case-specific, not norms).

## Common pitfalls & how to avoid them

- **Wall of text, no structure** → search engines get no context. Fix: `<h1>` + per-section `<h2>`, semantic `<section>`/`<main>`.
- **Lazy-loading the LCP image** → directly delays the most important render. Fix: `fetchpriority="high"`, never `loading="lazy"` on the LCP element.
- **Lazy images without `width`/`height`** → CLS as placeholders expand. Fix: always set dimensions or `aspect-ratio`.
- **Removed focus indicators** → catastrophic for keyboard-only users. Fix: replace with a ≥3:1 visible style.
- **Smooth scroll without focus move** → keyboard focus stuck at the link while the page scrolls. Fix: focus the target heading.
- **Scroll-jacking** → users can't reach contact/pricing. Fix: never seize the scrollbar; narrative intent only.
- **Scroll animations mistaken for story** → weight, no narrative. Fix: apply the three-element scrollytelling test; cut anything that survives scroll removal.
- **Over-animating on mobile** → INP damage, "feels broken." Fix: simplify/disable heavy motion on touch/small screens.
- **No `prefers-reduced-motion`** → motion-sickness for vestibular users. Fix: gate decorative motion behind the media query.
- **One page for multi-keyword content** → capped at ~1 cluster. Fix: go multi-page/hybrid.
- **Overloaded sticky nav** → clutter. Fix: 5–7 essential links.
- **Single non-repeated CTA** → users who needed context scroll past it and never convert. Fix: repeat the CTA.
- **Hamburger on desktop for cosmetics** → higher task time. Fix: show nav inline on desktop.
- **No "Skip to main content"** → keyboard users tab through the whole nav on every jump. Fix: add a skip link.
- **Unreserved ad/embed slots** → jumpy page. Fix: reserve height before load.
- **Generic template aesthetics** → "looks like every other site built with the same tool." Fix: customize type/color/spacing; avoid the default purple/blue AI palette (see [Avoiding Generic 'AI' Aesthetics](../07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md)).
- **Missing OG meta / canonical** → blank or wrong social previews; fragmented link equity from UTM variants. Fix: add `og:title/description/image/url`, Twitter Card, and `<link rel="canonical">` before launch.
- **Missing `<meta name="viewport">`** → mobile browsers zoom out, breaking layout and triggering CLS. Fix: always include it; it is not optional.
- **`history.replaceState` on anchor clicks** → browser Back doesn't return to the previous section. Fix: use `history.pushState` instead.
- **`-webkit-sticky` cargo-culted into new builds** → dead code that signals an outdated snippet. `position: sticky` has worked without the prefix in all current browsers since Safari 13 (2019). Remove it from new code.

## What real users complain about

Synthesized from practitioner/forum discussion (r/web_design, r/webdev, r/webdesign):

- **"One long scroll never meant for a computer."** A multi-product catalog crammed into one page reads as broken on desktop. Don't force catalog scope into a one-pager.
- **"Want to contact us? Better scroll down three miles!"** Burying contact info is a repeated, specific gripe. Keep contact in nav + footer.
- **"It makes me motion sick, all that scrolling."** Unconstrained scroll animation is a genuine accessibility harm. Honor `prefers-reduced-motion`.
- **"Go back, have to re-scroll because the site doesn't save my place — rage, abandon cart."** Lost scroll position on browser-back is fatal for commerce. Preserve scroll restoration; this is a reason *not* to one-page a store.
- **"The page constantly jumps around while ads load."** CLS from unsized ad/embed slots is noticed and abandonment-inducing.
- **"That heavy AI feel — the inevitable purple/blue scheme, spot it a mile away."** Template homogeneity reads as low credibility. Customize.
- **"Too many animations, fonts and colours."** Over-designing is a top mistake even strong concepts make. Restraint > spectacle.

## Senior-level checklist

- [ ] Scope justified: one focused goal, one keyword cluster; intent-checked against current SERPs.
- [ ] Story written in plain prose before any mockup; sections follow promise → proof → action.
- [ ] Exactly one `<h1>`; one descriptive `<h2>` per `<section>`; no skipped heading levels.
- [ ] Anchor `id`s set during structure; `scroll-margin-top` scoped to `section[id]` (not `[id]` globally) offsets the sticky header.
- [ ] One primary CTA goal, repeated at hero, mid-page, and footer; copy is value-driven.
- [ ] Smooth scroll via CSS; JS smooth scroll moves focus (`tabindex="-1"` + `.focus({preventScroll:true})`); anchor clicks use `history.pushState` so Back works.
- [ ] Scrollspy uses `IntersectionObserver`; active state via `aria-current="true"` (not `aria-current="page"`); first/last section edge cases handled.
- [ ] "Skip to main content" link present and focusable first.
- [ ] LCP image: `fetchpriority="high"`, not lazy; below-fold images lazy with explicit dimensions.
- [ ] Space reserved for all dynamic content (ads/embeds/media); CLS ≤ 0.1 verified.
- [ ] LCP ≤ 2.5s and INP ≤ 200ms measured on mobile (field or lab).
- [ ] `prefers-reduced-motion` honored; heavy motion simplified/disabled on touch/small screens.
- [ ] Sticky nav ≤ 7 links; disabled under ~300px viewport height; no desktop hamburger for cosmetics.
- [ ] No scroll-jacking; contact/pricing reachable from nav and footer.
- [ ] Focus indicators visible at ≥3:1 contrast; touch targets ≥24×24 CSS px (≥44px comfortable).
- [ ] JSON-LD structured data added where applicable; primary keyword front-loaded in title/`<h1>`.
- [ ] `<link rel="canonical">`, OG meta (og:title/description/image/url), Twitter Card, and `<meta name="viewport">` present.
- [ ] OG image is 1200×630 px, ≤ 300 KB.
- [ ] If scroll narrative is claimed, it passes the three-element scrollytelling test.
- [ ] Tested on Firefox for the `overflow-x: hidden` sticky bug.

## Sources

- web.dev — Core Web Vitals (LCP, INP, CLS): https://web.dev/articles/vitals
- web.dev — INP replaced FID (March 2024): https://web.dev/blog/inp-cwv-march-2024
- MDN — `IntersectionObserver`: https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API
- MDN — `scroll-behavior`: https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-behavior
- MDN — `position: sticky`: https://developer.mozilla.org/en-US/docs/Web/CSS/position#sticky_positioning
- MDN — `scrollIntoView()`: https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView
- MDN — `prefers-reduced-motion`: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- MDN — `history.pushState()`: https://developer.mozilla.org/en-US/docs/Web/API/History/pushState
- WCAG 2.2 — Target Size (Minimum) 2.5.8: https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html
- WCAG 2.2 — Focus Appearance: https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html
- Google/DoubleClick — "Find Out How You Stack Up to New Industry Benchmarks for Mobile Page Speed" (2018): https://www.thinkwithgoogle.com/marketing-strategies/app-and-mobile/mobile-page-speed-new-industry-benchmarks/
- Deloitte/Google — "Milliseconds Make Millions" (2020): https://www2.deloitte.com/content/dam/Deloitte/ie/Documents/Consulting/Milliseconds_Make_Millions_report.pdf
- WebAIM Screen Reader User Survey #10 (2024): https://webaim.org/projects/screenreadersurvey10/
- Open Graph Protocol: https://ogp.me/
- Twitter Card meta: https://developer.twitter.com/en/docs/twitter-for-websites/cards/overview/abouts-cards
- NYT "Snow Fall" (2012): https://www.nytimes.com/projects/2012/snow-fall/
- GSAP ScrollTrigger: https://gsap.com/docs/v3/Plugins/ScrollTrigger/
- Lenis: https://github.com/darkroomengineering/lenis
- Astro: https://astro.build/

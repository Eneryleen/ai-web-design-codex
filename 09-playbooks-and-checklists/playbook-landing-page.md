# Playbook: Design & Build a Landing Page

> An end-to-end, decision-by-decision playbook for taking a landing page from brief to launch: define a single goal, build a message map, wireframe in question-flow order, design hero/proof/CTA, write copy at the right reading level, add safe motion, ship responsive + fast + measured, and A/B test. Following it produces a page that competes at the top quartile (10%+ conversion) rather than the 6.6% median.

**Apply when:** building a single-purpose conversion page for one offer + one traffic source. · **Related:** [Landing Pages](../03-site-types/landing-pages.md) · [Conversion Rate Optimization](../06-marketing-and-conversion/conversion-rate-optimization.md) · [Copywriting & UX Writing](../06-marketing-and-conversion/copywriting-and-ux-writing.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md)

## TL;DR — rules at a glance
- **One goal, one page, one primary CTA.** Multiple goals = confused visitors = no conversions. Strip global nav entirely (it's an exit ramp).
- **Message match is the #1 lever.** Ad headline → page headline → offer must tell the same story. Mismatch = back button before the page even renders.
- **Hero has one viewport and ~5 seconds** before users decide to stay or leave (Nielsen Norman Group, Microsoft attention research). Answer: what is it, who is it for, why care, what do I do next — all above the fold.
- **Write copy first, at low fidelity.** Lock final copy at wireframe stage (correct size/hierarchy) before any visual design. Nail copy, then dress it.
- **Reading level is a number:** Flesch 70–85 / Grade 5–7. Grade 5–7 copy converts 11.1% vs 5.3% at college level — a 2x lift from one change.
- **Section order follows the question flow:** What is it? → How does it help? → How does it work? → Why trust it? → What do I do now? (EPRC — Exposition → Process → Results → CTA — defined in §3.)
- **Social proof must be specific and quantifiable.** "Bookings up 40% in two months" converts; "Great service!" does not. Reviews can lift conversions up to 270%.
- **Speed is CRO.** LCP < 2.5s, INP < 200ms, CLS < 0.1. Pages at 1s convert ~3x better than at 5s; conversion drops ~4.42% per second in the first 5s.
- **Forms: ≤5 fields.** 81% abandon forms after starting; cutting 11→4 fields lifts conversions ~160%. Put privacy reassurance next to the field, not in the footer.
- **Animate only `transform` and `opacity`.** Always ship a `prefers-reduced-motion` fallback. Large translate/parallax can trigger vestibular symptoms; opacity is safe.
- **Mobile-first.** 82.9% of traffic is mobile while mobile CR (2.19%) trails desktop (4.03%) — a broken mobile page is catastrophic.
- **One page per traffic source/segment.** Email (19.3% CR) ≠ paid search (11.3% CR). Companies with 40+ pages see 500% more leads than those with 1–5 pages (HubSpot research).
- **Launch with analytics + heatmaps already wired.** GA4 events + session recordings (Clarity/Hotjar) configured *before* traffic, or you launch blind.
- **A/B test one variable at a time:** headline → CTA copy → hero visual. Only 17% of marketers test, yet testing averages +37% conversion gain.

## 1. Define the brief (goal · audience · offer)
Before any design, lock these in writing. If you can't fill these one-liners, stop and ask — do not design around ambiguity.

- **Single goal (one verb + one object):** e.g. "Start a free trial," "Book a demo," "Buy the course." Not two. The goal *is* the primary CTA.
- **Traffic source(s):** email / paid search / paid social / organic. Source dictates awareness level and copy depth (see §3). Build a separate page per source when budget allows.
- **Audience awareness (Schwartz, 5 stages):** Unaware → Problem-Aware → Solution-Aware → Product-Aware → Most-Aware. Cold paid-social traffic ≈ Problem/Solution-Aware (needs more setup). Branded search / retargeting ≈ Product/Most-Aware (lead with the offer, skip the education).
- **The offer:** the exact thing they get + the exact action. "14-day free trial, no card" is an offer; "Learn more" is not.
- **Primary metric + target:** conversion event + a number to beat (use median 6.6% / top-quartile 10% as anchors, then your own baseline).
- **Visitor archetypes to serve simultaneously:**
  - *Impulse users* decide in seconds — need the deal + a CTA above the fold immediately.
  - *Deliberate users* read everything — need proof mid-page and a CTA at the end.
  - A good page serves both: CTA above fold *and* repeated after proof *and* at the bottom.

**Value-proposition test:** the opposite statement must be equally plausible. "We have great service" fails — no competitor claims bad service. "Onboard your whole team in under 10 minutes" passes (a rival could plausibly take longer). Rewrite until the opposite is a real, losing position.

## 2. Build the message map
A message map is the page's argument on one page before any layout. Structure it as a hierarchy:

```
Core value proposition (1 sentence — passes the opposite test)
├─ Pillar 1: <benefit>   → proof: <specific stat / testimonial>
├─ Pillar 2: <benefit>   → proof: <specific stat / testimonial>
└─ Pillar 3: <benefit>   → proof: <specific stat / testimonial>
Primary objection 1 → rebuttal (placed inline where it arises)
Primary objection 2 → rebuttal
Single CTA + risk-reducer (free / no card / cancel anytime / money-back)
```

Rules:
- **3 pillars max.** More dilutes. Each pillar must map to a page section and carry its own concrete proof.
- **Lead with WHY it matters to the visitor, then HOW, then WHAT** (StoryBrand framework: visitor = hero, product = guide). Company-centric copy ("We are the leading provider…") loses to visitor-centric ("You can finally ship on Fridays").
- **List objections explicitly** ("too expensive," "too hard to set up," "will it work for my stack") and assign each a rebuttal + a placement. Objections get answered *inline at the moment of concern* — not exiled to a separate FAQ page.

## 3. Section order = the visitor's question flow
Each section exists to answer the next question the visitor has. Order is not aesthetic; it's psychological.

| # | Visitor question | Section | EPRC | Must contain |
|---|------------------|---------|------|--------------|
| 1 | What is it? Is it for me? | Hero | Exposition | Headline, subhead, product visual, primary CTA |
| 2 | How does it help me? | Benefits / value | Exposition | 3 benefit blocks tied to pillars |
| 3 | Why should I believe you? | Social proof | Results | Logos, quantified testimonials, ratings |
| 4 | How does it actually work? | How it works | Process | 3-step visual, product screenshots |
| 5 | What about my concern? | Objection handling / FAQ | Process | Inline rebuttals, pricing clarity |
| 6 | What do I get / cost? | Offer / pricing | Results | Plan(s), risk-reducer |
| 7 | What do I do now? | Final CTA | CTA | Restated value + single CTA + reassurance |

- **Match depth to awareness:** Most-Aware traffic can collapse 1–5 and go straight to the offer; Problem-Aware traffic needs all seven, longer.
- **Don't assume length.** Explicitly A/B test long-form vs short-form. High-consideration products (B2B SaaS, finance) often need long pages; impulse offers need short ones.
- **Scroll-heatmap diagnostic:** if scroll depth drops sharply (>50% falloff) after section 2, the order is wrong — a question got answered too late or never. The threshold is relative to your baseline, not an absolute number; compare pages against each other.

## 4. Wireframe — copy first, low fidelity
Apply the **copy-first wireframing rule**: write all final copy into a grayscale wireframe with correct size, spacing, and hierarchy *before* visual design. If the page persuades in plain Helvetica on white, design only amplifies it. If it doesn't, no gradient saves it.

- Set hierarchy with size/weight/spacing only — no color, no imagery yet.
- **Five-second test** on the wireframe: show it for 5s, hide it, ask what the user recalls. Recall order = actual hierarchy. If the CTA isn't recalled, fix prominence before proceeding. (Tools: Maze, UsabilityHub.)
- Mark every CTA, every proof element, and the single LCP candidate (usually the hero visual or headline).

## 5. Choose style & stack
**Style:** pick from a coherent system, not vibes — see [A Taxonomy of Web Design Styles](../07-aesthetics-and-trends/design-styles-taxonomy.md) and avoid the tells in [Avoiding Generic 'AI' Aesthetics](../07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md). The style must support the goal: high-contrast CTA, legible body, product visuals that show outcomes.

**Stack — choose by what the page must do:**

| Need | Best fit | Lighthouse (typical) | Notes |
|------|----------|----------------------|-------|
| Perf-critical marketing, mostly static | **Astro** (islands) | 95–100 out of box | 50–70% smaller JS than equiv React; AstroWind template ships hero/pricing/testimonial/FAQ |
| Agency/marketing team, CMS, no-code | **Webflow** | 90–95 | See webflow.com for current pricing; AWS CloudFront + Fastly CDN; native SEO/AEO controls |
| Design-led, Figma-like, animation-forward | **Framer** | 75–90 | See framer.com for current pricing; Vercel Edge; perf drops on animation-heavy pages |
| Custom server logic, edge A/B, deep API | **Next.js** | 80–90 (with effort) | Full-stack/SSR; you own the perf budget |

**When NOT to use no-code:** if the page needs custom server logic, edge-rendered A/B variants, or deep API integrations, Webflow/Framer hit hard walls — use Astro or Next.js. See [Frameworks for Design-Forward Sites](../04-libraries-and-tools/frameworks-for-design.md).

## 6. Design the hero
The hero must do four jobs inside one viewport, in ~5 seconds:

1. **Headline** — the value proposition; passes the opposite test; benefit-led, not feature-led.
2. **Subhead** — one sentence of clarification or who-it's-for.
3. **Product visual** — a screenshot, outcome visual, or real customer photo. **Never** a stock photo of smiling strangers; they communicate nothing and underperform measurably.
4. **Primary CTA** — button ~2x body text size, contrasting color vs background, action verb. Personalize when you can (personalized CTAs convert +202% vs generic).

Heuristics:
- **Peek the next section.** A full-viewport hero with no visible content below measurably reduces scroll engagement. Show ~80px of section 2 so users know to scroll.
- **No nav.** A logo-only top bar is the maximum acceptable navigation.
- **Above-the-fold is psychology, not SEO.** Google renders everything; humans snap-judge the first screen. Don't sacrifice the value prop to cram keywords.

```html
<section class="hero">
  <h1>Onboard your whole team in under 10 minutes</h1>
  <p class="sub">Self-serve setup for ops teams of 5–50. No IT ticket required.</p>
  <img src="/hero-1200.avif" srcset="/hero-600.avif 600w, /hero-1200.avif 1200w"
       sizes="(max-width:768px) 100vw, 600px" width="600" height="400"
       alt="Dashboard showing a team's onboarding progress" fetchpriority="high">
  <a class="cta" href="/signup">Start free — no card</a>
  <p class="reassure">14-day trial · cancel anytime</p>
</section>
```
> `width`/`height` on the image prevent CLS; `fetchpriority="high"` on the LCP image speeds LCP. Do not lazy-load the hero/LCP image.

## 7. Social proof
- **Specific + quantifiable beats generic.** "We cut support tickets 32% in 6 weeks — Acme, 40-person team" converts; "Amazing tool!" does not.
- **Hierarchy of proof (strongest first):** quantified case study > video testimonial > customer logos (recognizable) > star ratings/counts > text testimonial > press mentions.
- **Numbers:** testimonials ≈ +34% conversions (BigCommerce); video testimonials outperform text by ~80–86% (Wyzowl); customer reviews drive meaningful conversion lift — exact figures vary by product category and review volume (vendor-reported ceilings reach +270%; treat as directional upper bound, not median).
- **Place at section 3** (after benefits, before the deep "how it works") to answer "why believe you?" at the moment it arises, and again near pricing.
- Use real names, roles, companies, and photos. Fabricated or anonymized proof reads as fake and backfires.

## 8. CTA strategy
- **2–3 CTAs total**, all pointing to the *same* action: one above the fold, one after key proof/pricing, one at the bottom. **Never two competing primary CTAs side by side** ("Sign Up" + "Contact Sales" as equals = decision paralysis).
- If a secondary action is unavoidable (e.g. "Watch demo"), make it visually subordinate (ghost/text link), never a second primary button.
- Button copy = first-person outcome where possible: "Start my free trial" > "Submit".
- On long pages, repeat the CTA so the deliberate user always has one within a screen of where they decide.

**Forms:**
- **≤5 fields** for single-step forms; every additional field reduces completion. If you need more data, use a **multi-step form**: show 1–2 fields per screen with a progress indicator. Multi-step forms consistently outperform equivalent single-step forms for high-ask scenarios (B2B demos, complex sign-ups) because they apply the foot-in-the-door principle.
- **Never lazy-load the form** on the initial CTA above the fold — the field should be visible instantly or open immediately on click.
- **Input auto-formatting:** phone numbers, credit cards, dates — format as the user types (no post-submit error on format). Inline validation on blur, not on submit.
- **Error handling:** inline, adjacent to the failing field, in plain language ("Enter a valid email" not "Error: field validation failed"). Never clear valid fields on error.

## 9. Write the copy
- **Reading level is a hard target:** Flesch Reading Ease 70–85, Flesch-Kincaid Grade 5–7, sentences < 20 words. This alone moves CR from ~5.3% to ~11.1%.
- **Cut, then cut again** (Krug): draft → remove 50% → remove 50% of what's left. Every surviving word earns its place.
- **Visitor-centric, second person.** "You can finally…" not "We are the leading…". Lead with the visitor's outcome.
- **Specific verbs and numbers.** "Deploys in 90 seconds" beats "fast deploys."
- **Inline objection handling.** Privacy line next to the email field; refund/cancel terms next to the CTA; "works with X" next to the integration claim.
- Run a Flesch checker (Hemingway, Yoast) on final copy; if Grade > 8, rewrite. See [Copywriting & UX Writing](../06-marketing-and-conversion/copywriting-and-ux-writing.md).

## 10. Visuals & motion (safe by default)
- **Animate only `transform` and `opacity`** — GPU-accelerated, no layout thrashing. Never animate `width`/`height`/`top`/`left` on performance-sensitive elements.
- **Use Intersection Observer** for scroll-triggered reveals; lazy-load animation-heavy below-fold sections. **Never block LCP with animation init** — fire hero animations *after* the LCP element paints.
- **Page/section transitions:** 300–500ms max. **Parallax:** keep layer speed differential to 20–30%, and disable it under reduced motion.
- **Always ship a `prefers-reduced-motion` fallback.** Large translate/swipe/parallax can trigger vestibular-disorder symptoms; replace with opacity or no motion. Consider a visible in-page toggle for users who can't change OS settings.
- **Video:** user-initiated only — **never autoplay** (especially with audio). Video on a landing page can lift CR +80–86%, but autoplay drives abandonment.

```css
.reveal { opacity: 0; transform: translateY(16px); transition: opacity .4s, transform .4s; }
.reveal.in { opacity: 1; transform: none; }

@media (prefers-reduced-motion: reduce) {
  .reveal { transition: opacity .2s; transform: none; }   /* opacity only, no translate */
  .parallax { transform: none !important; }               /* kill parallax */
  video[autoplay] { /* author: don't autoplay at all */ }
}
```
For libraries and tradeoffs see [Web Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md) and [Micro-interactions & Motion Design](../07-aesthetics-and-trends/micro-interactions-and-motion.md).

## 11. Make it responsive
- **Mobile-first.** Design the single-column mobile layout first, then enhance up. 82.9% of visits are mobile.
- Touch targets: **44×44px min (Apple HIG), 48×48dp (Material), 24×24 CSS px min (WCAG 2.5.8)**. The primary CTA should be a full-width or near-full-width tap target on mobile.
- Use fluid type/spacing with `clamp()` instead of stacking breakpoints — see [Fluid Typography & Spacing with clamp()](../02-responsive-adaptive/fluid-type-and-spacing.md).
- Verify hero fits one mobile viewport without the CTA pushed below the fold. Test at 360px width (common Android) and 390px (iPhone).
- See [Breakpoints, Devices & Mobile-First](../02-responsive-adaptive/breakpoints-devices-mobile-first.md) and [Touch Ergonomics & Cross-Device Design](../02-responsive-adaptive/touch-ergonomics-cross-device.md).

```css
h1 { font-size: clamp(1.75rem, 4vw + 1rem, 3.25rem); }
.cta { min-height: 48px; padding-block: .9rem; }         /* ≥48dp tap target */
@media (max-width: 480px) { .cta { width: 100%; } }
```

## 12. Performance
- **Targets (Core Web Vitals):** LCP < 2.5s · INP < 200ms · CLS < 0.1. Pages at 1s convert ~3x better than 5s; 53% of mobile users abandon a page that takes > 3s.
- **Resize images to display size.** Serving a 5000px-wide image into a 400px slot is one of the biggest real-world perf mistakes. Use AVIF/WebP, `srcset`/`sizes`, explicit `width`/`height`.
- `fetchpriority="high"` on the LCP image; `loading="lazy"` on everything below the fold; never lazy-load the LCP image.
- Defer non-critical JS; don't let animation init block the main thread before LCP.
- Poor Core Web Vitals raise paid-search CPC via lower Quality Score (target QS 7–10). See [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md).

## 13. SEO & discoverability
- Even with paid traffic, ship the basics: one `<h1>` (the headline), descriptive `<title>` + meta description that **match the ad/message**, semantic landmarks, alt text.
- Don't trade the value prop for keyword stuffing above the fold — humans first, crawlers second.
- **Structured data:** add `Organization` and `Product` schema where truthful. Use `FAQPage` schema only if FAQ content is genuinely on the page — but note Google removed FAQ rich results for most sites in 2023; the structured data still benefits AI-powered search parsing (Perplexity, ChatGPT Search, Google AI Overviews).
- **Canonical + multi-page strategy:** when running separate pages per traffic source, set `<link rel="canonical">` to the primary URL to prevent duplicate-content dilution. If pages are paid-only with no organic intent, consider `noindex` to keep crawl budget on organic pages.
- **Open Graph + Twitter/X card tags:** required for social sharing previews. `og:title`, `og:description`, `og:image` (1200×630px). Even paid-traffic pages get shared organically.
- **AI search (2026 context):** Google AI Overviews, Perplexity, and ChatGPT Search now surface landing-page content in conversational answers. Clear entity definitions (what the product is, who it's for, key facts) structured in plain prose improve inclusion in AI-generated summaries. See [SEO & Discoverability for Designers](../06-marketing-and-conversion/seo-and-discoverability.md).

## 14. Wire analytics & heatmaps (before launch)
Launching without measurement = flying blind.
- **Consent management first.** Under GDPR (EU/UK), ePrivacy, CCPA (California), and growing US state laws, analytics and session-recording scripts require user consent before firing in affected jurisdictions. Wire a consent management platform (CMP — Cookiebot, Usercentrics, OneTrust, or the lightweight cookie-consent component from your stack) before GA4/Hotjar load. Fire tags conditionally via GTM consent mode, not unconditionally. Non-compliance is both a legal risk and a data-quality problem (unconsented data is unreliable).
- **GA4** with explicit conversion events (CTA click, form submit, scroll milestones). Use server-side tagging or GA4's Measurement Protocol as a consent-mode fallback to preserve conversion signal without third-party cookies (third-party cookies are deprecated in Chrome as of 2024 and absent in Safari/Firefox; rely on first-party data + server events).
- **Session recordings + heatmaps** (Microsoft Clarity is free; Hotjar) — configured before traffic arrives, gated behind consent.
- **One-question exit-intent survey:** "What stopped you from completing your signup today?" — highest-signal qualitative tool.
- **Heatmap diagnostics:** heavy clicks on a non-clickable element = hierarchy bug (make it clickable or de-emphasize). Scroll drop early = wrong section order or slow render.
- For form pages, track field-level abandonment to find the friction field.

## 15. A/B testing
- **Test one variable at a time, in priority order:** headline → CTA copy → hero visual → form length → offer framing.
- Each test needs a **hypothesis** and **statistical significance (typically 95% confidence)** before calling a winner. Small samples lie.
- Testing 10+ variations can deliver ~86% better results than single-variant tests; the average test yields ~+37% — yet only 17% of marketers test. This is the cheapest CR you'll buy.
- High-leverage explicit tests: long-form vs short-form, single vs multi-step form, with/without video, personalized vs generic CTA.
- See [Conversion Rate Optimization (CRO)](../06-marketing-and-conversion/conversion-rate-optimization.md).

## Concrete section-by-section template
```
[ Logo-only top bar — NO global nav ]

HERO (1 viewport, ~5s decision window)
  H1  : value prop (passes opposite test, benefit-led)
  Sub : who it's for / one clarifier
  Vis : product screenshot or outcome visual (NOT stock people)
  CTA : primary button (~2x body, contrast color)  +  risk-reducer line
  ↳ peek ~80px of next section

BENEFITS  (answers "how does it help me?")
  3 blocks, one per message-map pillar; benefit headline + 1–2 lines + icon

SOCIAL PROOF  (answers "why believe you?")
  recognizable logos  +  1–2 quantified testimonials (name/role/company/photo)
  + rating/count if real

HOW IT WORKS  (answers "how does it actually work?")
  3 steps, numbered, each with a product screenshot

OBJECTIONS / FAQ  (inline rebuttals at point of concern)
  pricing clarity, security/privacy, "works with <stack>", refund/cancel terms

OFFER / PRICING
  plan(s), what's included, risk-reducer (free / no card / money-back)
  CTA #2 (same action as hero)

FINAL CTA
  restated value prop in one line  +  single CTA  +  reassurance

FOOTER (minimal: legal, privacy, contact — NOT a navigation maze)
```

## Concrete specs & numbers
- **Conversion anchors:** median CR 6.6% (Unbounce Q4 2024, 464M visits / 41k pages); top quartile 10%+. Industry: Events & Entertainment 12.3%, Financial Services 8.4%, SaaS 3.8%.
- **Device:** desktop CR 4.03% vs mobile 2.19% (ContentSquare 2024); mobile = 82.9% of traffic.
- **Speed:** LCP < 2.5s, INP < 200ms, CLS < 0.1; 1s page ≈ 3x CR of 5s page (Portent); ~4.42% CR drop per second in first 5s; 53% abandon > 3s (Google/Deloitte mobile study).
- **Copy:** Flesch 70–85, Grade 5–7, sentences < 20 words; Grade 5–7 = 11.1% CR vs college = 5.3% (56% gap).
- **CTA:** button ≈ 2x body copy size, contrasting color, above fold + repeats; personalized CTA +202% vs generic.
- **Forms:** ≤5 fields; 11→4 fields = +160%; 5-or-fewer fields ≈ +120% vs longer; 81% start-then-abandon; 67% of abandoners never return.
- **Proof:** testimonials +34% (BigCommerce); video testimonials +80–86% over text; customer reviews strongly positive effect on conversion (exact lift varies heavily by product category and review count — "up to +270%" is a vendor-reported ceiling, not a median; treat as directional); video on page +80–86% (Wyzowl).
- **Traffic channels (CR, Unbounce 2024):** email 19.3% · Google paid search 11.3% · Facebook 9–13% (high variance by vertical). Instagram and social channels vary widely by vertical; treat single-channel benchmarks as directional, not targets. First-party audiences convert ~4x cold; retargeted visitors +43% more likely to convert.
- **A/B:** average +37% gain; 10+ variations ≈ +86% vs single-variant; only 17% of marketers test.
- **Motion:** transitions 300–500ms; parallax 20–30% layer differential.
- **Touch:** 44×44px (Apple HIG) · 48×48dp (Material) · 24×24 CSS px min (WCAG 2.5.8).
- **Library weights:** GSAP ~23 KB gz (modular); Framer Motion ~32 KB gz (non-modular); Motion (motion.dev) typically ships smaller per-feature due to tree-shaking.
- **Quality Score target (paid search):** 7–10.

## Tools & libraries
- **Astro** — perf-critical static marketing; 95–100 Lighthouse; AstroWind ships hero/pricing/testimonial/FAQ with Tailwind. Default pick for raw conversion + speed.
- **Webflow** (verify current pricing at webflow.com) — agencies/marketing teams needing CMS + no-code; AWS CloudFront + Fastly. Avoid if you need custom server logic or edge A/B.
- **Framer** (verify current pricing at framer.com) — design-led, animation-forward; Vercel Edge. Watch perf on heavy animation.
- **Next.js** — custom server logic, SSR, edge A/B variants, deep APIs; you own the perf budget (80–90 with effort).
- **GSAP** (~23 KB gz, modular; acquired by Webflow 2023, free for most site/app use under the standard license — check gsap.com/licensing for commercial SaaS/OEM edge cases) — complex timelines, SVG, 50+ elements, scroll scenes.
- **Motion** (motion.dev, MIT, modular) — GSAP alternative, faster for unknown/different value types; good default for new builds.
- **Framer Motion / Motion for React** — React micro-interactions + page transitions; built-in `reducedMotion="user"` via `MotionConfig`.
- **Measurement:** GA4 (events), Microsoft Clarity (free recordings/heatmaps) or Hotjar, Maze/UsabilityHub (5-second & preference tests).
- **NOT:** no-code builders when you need edge A/B, custom server logic, or deep integrations — they hit hard walls.

## Do / Don't
**Do**
- Lock one goal + one primary CTA; strip global nav.
- Match the ad message exactly on the headline.
- Write final copy at Grade 5–7 before designing.
- Order sections by the visitor's question flow (EPRC).
- Use specific, quantified social proof with real people.
- Resize images to display size; set explicit `width`/`height`; AVIF/WebP.
- Ship `prefers-reduced-motion` fallbacks; animate transform/opacity only.
- Build one page per traffic source/segment.
- Wire GA4 + heatmaps + exit survey before launch.
- A/B test one variable at a time to 95% confidence.

**Don't**
- Don't keep global navigation (it's an exit ramp).
- Don't run competing primary CTAs side by side.
- Don't use hero stock photos of smiling strangers.
- Don't autoplay video (especially with audio).
- Don't write company-centric copy ("We are the leading…").
- Don't serve 5000px images into 400px slots.
- Don't exile objection handling to a separate FAQ page.
- Don't pop a modal before the visitor sees any content.
- Don't lazy-load or animation-block the LCP element.
- Don't run one page for all traffic sources.

## Common pitfalls & how to avoid them
- **Generic value prop** ("great software for teams") — fails the opposite test. Fix: be specific enough that a competitor could plausibly claim the opposite.
- **CTA overload** ("Sign Up / Learn More / Watch Demo / Contact Sales") — decision paralysis. Fix: 2–3 CTAs, all the same action; subordinate any secondary link.
- **Stock hero photos** — communicate nothing. Fix: product screenshots, outcome visuals, real customer photos.
- **Global nav left on** — adds exit ramps. Fix: remove it; logo-only bar max.
- **Autoplay video** — drives abandonment. Fix: user-initiated playback, no audio default.
- **College-level copy** — long sentences, jargon, passive voice correlate with lower CR. Fix: Grade 5–7, active voice, short sentences.
- **No message match** — clicked a specific offer, landed on a generic page. Fix: mirror ad headline + offer at the top.
- **Objection handling buried in a separate FAQ** — Fix: inline rebuttals at the moment of concern (privacy by the field, refund by the CTA).
- **Immediate pop-up on load** — top UX rage-quit trigger. Fix: no modal before content; use exit-intent only.
- **Launch without analytics/heatmaps** — flying blind. Fix: GA4 events + recordings configured pre-traffic, gated behind consent.
- **Analytics firing without consent** — GDPR/CCPA violation and data quality risk. Fix: wire CMP first; fire GA4 conditionally; use server-side tagging as cookie-less fallback.
- **Oversized/unsized hero images** — slow LCP + CLS. Fix: resize to display size, set dimensions, `fetchpriority="high"`.
- **Mobile ignored** — 82.9% of traffic, already-lower CR. Fix: mobile-first, test at 360/390px.
- **One page for all sources** — mismatched awareness. Fix: source-specific pages.
- **A/B testing without a hypothesis or significance** — noise mistaken for signal. Fix: hypothesis + 95% confidence + adequate sample.
- **No canonical on multi-source pages** — duplicate content dilution. Fix: `<link rel="canonical">` on all variants pointing to the canonical URL.
- **No-code for custom needs** — Webflow/Framer wall on server logic / edge A/B / deep APIs. Fix: Astro or Next.js.

## What real users complain about
Synthesized from practitioner forums (e.g. r/web_design) and CRO research:
- **Pop-up before any content** — "I hadn't read a single word and got a modal" is a top rage-quit trigger.
- **Autoplaying video/audio** — instant back button, especially on mobile data.
- **Slow, heavy heroes** — giant unoptimized images make the page feel broken before it loads.
- **"We/our company" copy** — readers report tuning out company-centric pages; they want their own outcome.
- **Vague testimonials** — "Great service!" reads as fake; specifics earn trust.
- **Can't find what the product even is** — clever hero copy with no plain statement of "what is this?" frustrates.
- **Motion sickness from parallax/large swipe animations** — real vestibular symptoms; why reduced-motion support is non-negotiable.
- **Forms that demand too much** — abandonment spikes when fields exceed what feels necessary for the value offered.
- See [What Real Users Complain About (Synthesis)](../08-pitfalls-and-antipatterns/real-user-complaints.md) and [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md).

## Senior-level checklist
**Strategy**
- [ ] Single goal + single primary CTA written down.
- [ ] Traffic source(s) identified; awareness stage mapped to copy depth.
- [ ] Value prop passes the opposite test.
- [ ] Message map: ≤3 pillars, each with concrete proof; objections + placements listed.
- [ ] One page per source/segment (or a deliberate plan to start with one).

**Copy & content**
- [ ] Final copy written at low fidelity before visual design.
- [ ] Flesch 70–85 / Grade 5–7 verified; sentences < 20 words.
- [ ] Visitor-centric, second person, specific numbers/verbs.
- [ ] Ad headline ↔ page headline ↔ offer match.
- [ ] Objection rebuttals placed inline at point of concern.

**Structure & design**
- [ ] Sections ordered by question flow (EPRC).
- [ ] Hero answers what/who/why/next in one viewport; peeks next section.
- [ ] Hero visual = product/outcome/real customer, not stock people.
- [ ] Global nav removed (logo-only bar max).
- [ ] Social proof specific + quantified, with real attribution.
- [ ] 2–3 CTAs, same action, no competing primaries.
- [ ] Form ≤5 fields; privacy reassurance beside the field.
- [ ] 5-second test passed (CTA recalled).

**Motion & accessibility**
- [ ] Only `transform`/`opacity` animated.
- [ ] `prefers-reduced-motion` fallback for every animation; parallax disabled under it.
- [ ] No autoplay; video user-initiated.
- [ ] Scroll animations fire after LCP; below-fold animation lazy-loaded.
- [ ] Contrast meets WCAG; CTA passes contrast.

**Responsive & performance**
- [ ] Mobile-first; tested at 360px and 390px.
- [ ] Touch targets ≥ 44×44px / 48×48dp / 24 CSS px min.
- [ ] Images resized to display size; AVIF/WebP; `srcset`/`sizes`; explicit `width`/`height`.
- [ ] LCP < 2.5s, INP < 200ms, CLS < 0.1 on mobile.
- [ ] LCP image `fetchpriority="high"`, not lazy-loaded.

**SEO & discoverability**
- [ ] `<h1>`, `<title>`, meta description set and ad-matched; alt text present.
- [ ] `<link rel="canonical">` set; `noindex` applied if page is paid-only with no organic intent.
- [ ] Open Graph tags: `og:title`, `og:description`, `og:image` (1200×630px).
- [ ] Organization/Product structured data added where truthful.

**Launch & measure**
- [ ] Consent management platform (CMP) wired; analytics fire conditionally post-consent.
- [ ] GA4 conversion events firing; verified in GA4 DebugView.
- [ ] Server-side tagging or Measurement Protocol configured as cookie-less fallback.
- [ ] Heatmaps/session recordings live before traffic, gated behind consent.
- [ ] Exit-intent one-question survey wired.
- [ ] First A/B test queued (headline) with hypothesis + significance plan.
- [ ] Cross-browser + real-device QA done. See [Wireframing, Prototyping, Handoff & QA](../05-process-and-systems/wireframing-prototyping-handoff-qa.md).

## Sources
- Unbounce Conversion Benchmark Report (Q4 2024) — median 6.6% CR; channel/industry benchmarks; form abandonment data.
- ContentSquare 2024 Digital Experience Benchmark — desktop vs mobile CR, mobile traffic share.
- Portent (2019 research, frequently cited) — 1s page ≈ 3x CR of 5s page; 4.42% per-second CR drop.
- Google/Deloitte (2019) — "Milliseconds Make Millions" — 53% mobile abandonment > 3s.
- Nielsen Norman Group; Microsoft attention studies — ~5s hero decision window.
- BigCommerce — testimonials +34% CR.
- Wyzowl State of Video Marketing (annual) — video testimonials +80–86% over text; video on page lift.
- HubSpot research — 40+ landing pages = 500% more leads vs 1–5 pages.
- web.dev / Core Web Vitals (Google) — LCP/INP/CLS thresholds.
- GSAP (gsap.com) — licensing terms post-Webflow acquisition.
- Motion (motion.dev) library docs — bundle sizes, performance characteristics.
- WCAG 2.2 §2.5.8 Target Size (Minimum); Apple Human Interface Guidelines; Material Design — touch-target sizes.
- Google Search Central — FAQ rich results policy changes (2023); structured data guidelines.
- StoryBrand (Donald Miller, *Building a StoryBrand*, 2017) — visitor-as-hero framework.

# Landing Pages

> A landing page is a single-purpose, traffic-destination page engineered around ONE conversion goal. This guide gives an AI builder the anatomy, copy formulas, layout rhythm, trust mechanics, and numeric targets to design and ship landing pages that convert at or above industry medians — including B2B vs B2C divergence and lead-form vs click-through patterns.

**Apply when:** building any paid-traffic / campaign / product-launch destination page with a single goal. · **Related:** [Conversion Rate Optimization](../06-marketing-and-conversion/conversion-rate-optimization.md) · [Copywriting & UX Writing](../06-marketing-and-conversion/copywriting-and-ux-writing.md) · [Persuasion Psychology](../06-marketing-and-conversion/persuasion-psychology.md) · [Playbook: Landing Page](../09-playbooks-and-checklists/playbook-landing-page.md)

## TL;DR — rules at a glance
- **One page, one goal, one primary CTA** repeated at hero / mid-page / end. Kill competing CTAs.
- **Purchase Rate = Desire − (Labor + Confusion).** Raise desire; cut friction and cognitive load at the same time.
- **5-second test:** a stranger must know *what you sell, who it's for, what to do next* in ≤5s. If not, rewrite the H1.
- **Message match is non-negotiable:** the page H1, visual, and tone mirror the source ad. A weak match wastes every dollar of ad spend before the visitor even reads a word.
- **Benefits over features:** every feature answers "what's in it for me?" Sell the outcome, not the mechanism.
- **Hero must carry social proof.** Most heroes ship with zero proof above the fold — the single most common structural gap.
- **CTA = call-to-value, first person:** "Start my free trial" > "Sign up". First-person value-framed copy consistently outperforms generic labels in multivariate tests.
- **Risk reducer under every CTA:** "No credit card", "Cancel anytime", "Free 14 days" — removes the last-second objection before the click.
- **Handle objections BEFORE the button.** Top 3 in body copy; the rest in FAQ. An unresolved doubt at the CTA = no click.
- **Page length = f(offer complexity + risk).** Simple/low-risk → short. Expensive/complex/unknown brand → long. When unsure, A/B test.
- **Mobile-first hero for a ~400px-tall viewport.** ~70% of traffic is mobile; H1 + CTA + one visual must fit before scroll.
- **Strip the nav.** Every nav link, sidebar, and social icon is a leak. Keep ≤2–4 links + the CTA, or remove entirely.
- **Core Web Vitals gate conversion:** LCP <2.5s, INP <200ms, CLS <0.1. A 1s mobile delay ≈ −7% conversions (Google/Deloitte mobile speed research).
- **B2B CTA = relationship ("Book a demo"); B2C click-through CTA = transaction ("Shop now").** Never "Buy now" on B2B.
- **Build one page per traffic source.** Companies with 40+ landing pages generate ~12× more leads than those with ≤5 (HubSpot State of Marketing).

## The core formula and what it implies
**Purchase Rate = Desire − (Labor + Confusion)** (Julian.com). Three levers, optimize all three:
- **Desire ↑** — relevance (message match), benefit clarity, social proof, urgency that's real.
- **Labor ↓** — fewer form fields, fewer clicks, autofill, one decision per screen.
- **Confusion ↓** — one CTA, plain headline, clear hierarchy, no jargon, no dashboard dumps.

A page can have huge desire and still fail if the form has 9 fields (labor) or the headline says "Empower your workflow" (confusion). Treat each section as either adding desire or removing friction — if it does neither, cut it.

## Landing page anatomy (top to bottom)
The narrative arc used here (borrowed from Julian Shapiro's growth writing): **Exposition → Process → Results → Call to action**, with objection handling woven between value and CTA. Other frameworks (AIDA, PAS, StoryBrand) produce similar section sequences — the names don't matter; the logic does.

```
┌─ HERO ─────────────────────────────────────────────┐
│ H1 (benefit, ~10 words)                            │
│ Subhead (what it is + how it works, 1–2 sentences) │
│ [ Primary CTA ]  ← call-to-value, 1st person       │
│ risk-reducer microcopy (no card / cancel anytime)  │
│ trust signal (logo bar / ★ rating / user count)    │
│ context visual (product showing OUTCOME, not stock)│
└────────────────────────────────────────────────────┘
   SOCIAL PROOF strip (logos / star rating / count)
   PROBLEM / agitation  (the pain you remove)
   VALUE PROP 1  (benefit header + objection + visual)
   VALUE PROP 2  ...
   VALUE PROP 3  ...
   ── MID-PAGE CTA (repeat primary) ──
   DEEP SOCIAL PROOF (named testimonials / case study)
   FAQ  (each Q = a real buying objection)
   ── FINAL CTA (repeat primary) + risk reducer ──
   minimal footer (legal, privacy — no escape links)
```

### Hero — required elements
1. **H1** — benefit-driven, ideally ~10 words, hard cap ~20. Litmus: *"If a visitor reads only this line, do they know exactly what you sell?"*
   - Bad: `Empower your sales.` / `The future of collaboration.`
   - Good: `Inventory management for restaurants that hate spreadsheets.`
   - Good: `Get $20k back from the IRS in 20 minutes.`
2. **Subheadline** (Julian.com formula): Sentence 1 = *what is the product?* Sentence 2 = *how does the H1's claim actually work?* Keep to 1–2 sentences.
3. **Primary CTA button** — highest-contrast element on the page (see CTA section).
4. **Risk reducer** microcopy directly under the button.
5. **≥1 trust signal** above the fold — logo bar, star rating, or "Trusted by 150,000+ businesses". (Not "used by many" — specificity, not claims.)
6. **Context visual** — product screenshot showing the *outcome*. Visual quality ladder (best → worst): outcome screenshot → annotated UI shot → real customer photo → abstract illustration → stock photo. Avoid the **dashboard dump** (entire complex UI crammed into the hero = visitors see complexity, not benefit).

### Above-the-fold layout
Use a **Z-pattern** for the hero: eye travels top-left (logo/H1) → top-right → diagonal → CTA at bottom-right of the Z. For long-form sections below, expect **F-pattern** scanning — front-load value in headers and first lines.

```html
<section class="hero">
  <p class="eyebrow">For restaurant owners</p>
  <h1>Inventory management for restaurants that hate spreadsheets</h1>
  <p class="sub">Track stock, cut waste, and reorder in one tap. Connects to your POS — no manual counts.</p>
  <a class="cta" href="#signup">Start my free trial</a>
  <p class="risk">No credit card required · Free for 14 days</p>
  <ul class="logos" aria-label="Trusted by">…</ul>
  <img src="dashboard-outcome.webp" alt="Weekly waste down 23%" loading="eager" fetchpriority="high" width="640" height="420">
</section>
```

## Message match (ad scent)
The page must visually and verbally **continue the ad** the visitor just clicked. Headline, hero image, color, and offer should echo the ad creative so the visitor feels "yes, right place."
- Ad scent failure wastes every click before the page even loads. The visitor's split-second judgment is "did I land in the right place?" — a mismatch triggers immediate back-navigation.
- Dedicated pages per ad group consistently outperform a single generic destination because relevance scores improve (Google Ads Quality Score, Meta Relevance Score), lowering CPC and raising CVR simultaneously.
- **Build a dedicated page per ad group / traffic source.** One generic page = average match for everyone = mediocre conversion for all.
- **Never bait-and-switch:** if the ad promises a free tool, don't gate it behind a sales call. Promise = delivery.
- **Specific match checklist:** (1) H1 repeats or directly extends the ad headline; (2) hero image mirrors the ad visual or product shown; (3) offer/CTA matches exactly (same free trial duration, same discount, same tool name); (4) tone and reading level match the audience segment the ad targeted.

## Single conversion goal
- One **primary** action, repeated (not multiplied). The repeated CTA always points at the same goal.
- Secondary asks (newsletter, "learn more") get *lower visual weight* or get cut. Four equal-weight CTAs ("Buy / Demo / Learn More / Contact") = decision paralysis before the first scroll.
- **Remove the nav.** Sidebars, footer link farms, social icons, exit links = conversion leaks. Keep ≤2–4 links + one CTA, or strip nav entirely on dedicated paid pages.

## Features vs benefits
Every feature must pass *"what's in it for me?"* Translate mechanism → outcome → painkiller.

| Feature (mechanism) | Benefit (outcome) |
|---|---|
| Automated tax filing | Get $20k back from the IRS in 20 minutes |
| Real-time POS sync | Never count stock by hand again |
| AES-256 encryption | Your customer data can't be read, even by us |
| 99.99% uptime SLA | Your store stays open while competitors go dark |

**Per-feature section structure:** short value-prop header (avoid "Revolutionize your workflow") + a paragraph that *answers the specific objection this feature raises* + a visual of the feature in action. Each feature ties back to the hero's dominant value prop.

## Objection handling
If the visitor reaches the button with an unresolved concern, they don't click. Handle objections **proactively, before the CTA**.
- **Discover objections:** survey post-purchase customers ("What almost stopped you from buying?"); mine competitors' 1–3★ reviews; ask the sales team; analyze form drops / abandoned carts.
- **Place the top 3** in body copy alongside the relevant feature. Handle the rest in the FAQ.
- **FAQ = objection handler**, not a feature recap. Each Q is a real buying objection: "How long does setup take?", "Will this work for my industry?", "What happens after the trial?" Each answer reinforces the value prop.
- **Don't introduce NEW objections** in the FAQ that weren't already on the visitor's mind — you'd be planting doubt, not resolving it.

## Urgency and scarcity
Urgency lowers the cost of deciding *now* vs. later. It only works if it's true.

**Legitimate urgency:**
- Time-limited discounts with a real end date (e.g., sale ends Friday).
- Inventory scarcity shown in real time ("Only 3 seats left at this price" when true).
- Event-driven deadlines ("Enrollment closes March 31").
- Trial expiry ("Your free trial ends in 3 days" in a follow-up email sequence).

**Illegitimate urgency (destroys trust when noticed):**
- Countdown timers that reset on page refresh or on a new session.
- "Limited supply" on a digital product with zero marginal cost.
- "Only 2 left in stock" that is always 2.
- Fake "sale" prices where the original price was never charged.

**Rule:** only display urgency signals that would still be true if the visitor shared the page URL with a friend and they opened it 24 hours later. Once a visitor catches manufactured scarcity, they lose trust in every other claim on the page.

## Social proof
**Trust comes from specificity, not adjectives.**
- Named testimonials *with job title + company* > anonymous quotes. Include a **specific metric** ("cut food waste 23% in 6 weeks") — generic praise ("Great product!") is a weak signal.
- Real team/customer photos > stock. Logo bars of recognizable customers carry weight.
- **Placement ladder:** logos / star rating above fold → contextual testimonial *near the primary CTA* → case studies mid-page → security/compliance badges near the form or footer.
- Spiegel Research Center (Northwestern) found that products with 5+ reviews show purchase likelihood ~270% higher than zero-review products, and that the lift is strongest in the first 5–10 reviews — returns diminish after ~30.
- Social proof types ranked by persuasive weight for cold traffic: (1) named case study with a measurable outcome → (2) logo from a recognizable brand the visitor respects → (3) aggregate rating with a visible review count → (4) anonymous "Great product!" quote. Never use type 4 alone.
- Volume: **10–20 reviews** for competitive visibility, **25–30** signals category leadership.
- **Legal:** FTC rule (Oct 2024) — fake / AI-generated reviews carry penalties up to **$51,744 per violation**. Never fabricate testimonials, logos, or counts.

## CTA design and placement
- **Copy = call-to-value, first person.** "Start my free trial" > "Start free trial" > "Sign up". "Show me my heatmap" > "Get started". First-person value framing consistently outperforms generic labels across documented CRO tests; CTA buttons beat text links for discoverability and click-through.
- **Contrast over color preference.** The button must be the single highest-contrast element. The well-known "red beats green +21%" result (from Performable, later acquired by HubSpot) happened *because the page was mostly green* — red was the contrast outlier. The lesson is contrast, not color. Pick an accent used nowhere else on the page.
- **Squint test:** blur the page until shapes dissolve; the CTA must remain the most prominent thing. If it disappears, raise contrast.
- **Touch target:** min **44×44px** (Apple HIG), **48×48dp** (Material), **24×24 CSS px** minimum (WCAG 2.5.8). Text-on-button contrast ≥ **4.5:1** (WCAG 2.1 AA).
- **Risk reducer microcopy** under the button: "No credit card required", "Cancel anytime", "Free for 14 days" — directly addresses the last-second hesitation objection and is one of the lowest-effort, highest-reliability copy improvements.
- **Placement:** primary CTA in hero, again mid-page (after value props), again at the end after FAQ. Long pages need the button to recur within ~one viewport of any scroll position.

```css
.cta {
  display: inline-flex; align-items: center; justify-content: center;
  min-height: 48px; min-width: 48px;   /* exceeds 44px / 24px floors */
  padding: 0 1.5rem; border-radius: 8px;
  background: var(--accent);            /* used nowhere else */
  color: #fff;                          /* ≥ 4.5:1 vs --accent */
  font-weight: 600;
}
.cta:focus-visible { outline: 3px solid; outline-offset: 2px; } /* keyboard a11y */
```

## Section rhythm and visual hierarchy
Alternate **tension and relief**: problem (tension) → solution (relief) → proof (reassurance) → CTA (action). Don't stack three feature blocks with identical layout — vary alignment (text-left/text-right), break monotony with a testimonial or stat strip. Each section should advance the argument toward the single CTA; if a section neither raises desire nor lowers friction, delete it.

**Visual hierarchy rules:**
- **Type scale:** H1 ≥ 40px (mobile ≥ 28px), H2 ≥ 28px, body ≥ 16px. Never set body copy below 16px — it signals "fine print" and slows reading.
- **Whitespace = comprehension.** Each section needs ≥ 80px vertical padding on desktop, ≥ 48px on mobile. Dense sections feel like terms-of-service.
- **One dominant element per section.** If you have a headline, a subhead, a visual, a testimonial, and a CTA all in the hero, establish a clear size/weight/color hierarchy so the eye has a path. Competing-weight elements force the brain to prioritize and it gives up.
- **Color:** use ≤3 semantic colors — background, content, accent (CTA). Adding a fourth color for "emphasis" usually dilutes the CTA's signal.
- **Section separators:** prefer whitespace over lines or background color changes. If you do use alternating section backgrounds, keep the contrast subtle (e.g., white vs. 5% gray) — high contrast breaks reading flow.

## The page-length debate
**"The bigger the ask, the longer the page."** Length must match offer complexity and perceived risk — not a fixed ideal.
- **Short page wins** when: low price, simple offer, known/trusted brand, warm traffic.
- **Long page wins** when: high investment, complex/novel offer, unknown company, cold traffic needing education.
- The CRO literature contains individual case studies showing both shorter and longer pages outperforming their counterparts by 10–60%+. These results are context-specific — treat them as direction, not benchmarks to cite. Always run your own test with your traffic.
- **Decision framework:**

| Price | Complexity | Brand familiarity | Traffic | → Length |
|---|---|---|---|---|
| Low | Simple | Known | Warm | Short |
| High | Complex | Unknown | Cold | Long |
| Mixed | — | — | — | **Test both** |

Quality of persuasion > raw length. A long page of filler converts worse than a tight short one.

## Audience awareness & traffic temperature
Awareness level (Schwartz, roughly) drives architecture:
- **Cold** (TikTok / display / interrupt ads) — visitor doesn't know the problem or you. Lead with the *problem*, educate, longer page, more proof. Form *after* value delivery.
- **Warm** (retargeting, content) — knows the problem, comparing solutions. Lead with differentiation + proof.
- **Hot** (branded search, direct) — ready to act. Minimal friction, form/CTA above the fold, short page.

## B2B vs B2C
| | B2B | B2C |
|---|---|---|
| Headline | Customer **pain point** first, not features | Desire / outcome / aspiration |
| CTA | Relationship: "Book a demo", "Request a quote", "Talk to sales" | Transaction: "Shop now", "Add to cart", "Start free trial" |
| Primary pattern | **Lead-gen** (form) | **Click-through** (qualify → send to checkout) |
| Proof | Logos, case studies, ROI, compliance badges | Reviews, ratings, UGC, influencer, scarcity |
| Decision | Multi-stakeholder, long cycle; research spans multiple sessions and sources before form fill | Often single, impulsive |
| Page length | Usually longer (justify cost to a committee) | Often shorter for impulse buys |
| Never | "Buy now" | Over-gating a simple purchase behind a form |

B2B benchmark: 2–6% CVR is solid; top performers on dedicated paid pages push 10%+ (Unbounce 2024 Benchmark). B2B SaaS median is around 3.8% per Unbounce. Note that B2B purchase research increasingly happens on mobile — design the demo form for thumbs regardless of where the final conversion click occurs.

## Lead-form vs click-through patterns
**Lead-gen (form on page):**
- Warm/direct traffic → form **above the fold**. Cold/paid traffic → form **after value delivery**.
- **Forms ≤5 fields.** Each field you remove reduces abandonment. Top-of-funnel B2B is often just **name + email**; every additional field needs a business justification.
- **Progressive profiling:** ask only what you need now; enrich post-conversion. Each extra field is a conversion tax.

```html
<form>
  <label>Work email<input type="email" name="email" autocomplete="email" required></label>
  <label>First name<input name="given-name" autocomplete="given-name" required></label>
  <button class="cta" type="submit">Book my demo</button>
  <p class="risk">No card required · 15-minute call · Cancel anytime</p>
</form>
```

**Click-through (no form):** remove the form, qualify the visitor with benefits + proof on the landing page, then send them to checkout / app signup. Best for impulse / low-consideration B2C purchases. The landing page *sells*; the next page *transacts*.

**Click-through: modal vs. navigate.** Two patterns:
1. **Navigate** — CTA links to `/checkout` or `/signup`. Full-page transition; best for multi-step flows and product pages where the next page has substantial UI.
2. **Modal/overlay** — CTA opens an in-page overlay with the form/checkout. Zero perceived navigation; best for simple single-field captures (email for a waitlist, phone for a quote). Modal must close cleanly and not trap keyboard focus.

## A/B testing priorities
Not all elements have equal impact. Test in this order (highest expected lift first):

1. **H1 headline** — biggest single lever; swap vague benefit for specific outcome.
2. **Hero visual** — outcome screenshot vs abstract illustration vs stock photo.
3. **CTA copy** — first-person value vs generic ("Start my free trial" vs "Get started").
4. **Form length** — 2 fields vs 4 fields vs 6 fields.
5. **CTA color/contrast** — only after copy and offer are locked.
6. **Page length** — short vs long; set up as a redirect test so both versions are fully built.

**Testing hygiene:**
- Run one test at a time per page. Multi-variable MVT requires orders-of-magnitude more traffic.
- Use a sample-size calculator before launching. At a 5% base rate targeting 20% relative lift, you need ~3,800 visitors per variation for 80% power.
- Never call a test early based on an encouraging trend — early stopping inflates false-positive rates. Set the sample size before launch and hold to it.
- Run tests for ≥1 full business cycle (usually 2 weeks) to avoid day-of-week bias.

## Concrete specs & numbers
- **Conversion benchmarks:** cross-industry median **6.6%** (Unbounce 2024 Conversion Benchmark; 57M+ conversions, 41,000+ pages). Range ~3.8% (SaaS) to ~12.3% (leading industries). Dedicated landing pages typically convert at 5–15% vs 2–3% for generic site pages.
- **Attention:** users spend ~57–80% of viewing time above the fold (NNGroup scrolling-and-attention research). Design above the fold to capture intent; don't hide the value prop below scroll.
- **Core Web Vitals:** LCP **<2.5s** (Google data shows mobile bounce rates increase sharply after 3s; a 1s delay ≈ −7% conversions per Google/Deloitte). INP **<200ms** (replaced FID as a Core Web Vital in March 2024). CLS **<0.1**.
- **Touch target:** 44×44px (Apple HIG) / 48×48dp (Material) / 24×24 CSS px floor (WCAG 2.5.8).
- **Contrast:** button text ≥ 4.5:1 (WCAG 2.2 AA); CTA background vs page background should also be visually distinct — text contrast alone is insufficient if the button shape disappears.
- **Forms:** ≤5 fields; each additional field adds abandonment risk (direction consistent across multiple industry studies; exact % varies by context). B2B TOFU often name + email only.
- **Social proof:** 10–20 reviews for competitive visibility, 25–30 signals category leadership (Spiegel Research Center); purchase likelihood lifts sharply in the first 5–30 reviews.
- **Mobile:** ~70% of global web traffic (Statcounter 2024). Design hero for a ~400px-tall viewport first.
- **Headline:** ideally ~10 words, cap ~20. Subhead 1–2 sentences.
- **A/B testing:** use a sample-size calculator (e.g., Evan Miller's) targeting 80% statistical power at your expected minimum detectable effect. For a 5% base rate and 20% relative lift, you need ~3,800 visitors per variation — not 1,000. Low-traffic pages need longer run times, not fewer visitors.
- **FTC penalty:** up to $51,744 per fake/AI review violation (FTC final rule, Oct 2024).

## Performance: achieving LCP <2.5s
LCP is almost always caused by the hero image or the largest above-fold text block. Treat it as an engineering constraint, not an afterthought.

**Hero image checklist:**
1. Serve in **WebP** (or AVIF where support allows). Never ship a 600 KB JPEG hero.
2. Set explicit `width` and `height` attributes — prevents CLS from layout reflow.
3. Add `fetchpriority="high"` and `loading="eager"` to the LCP `<img>`. Do NOT lazy-load the hero image.
4. Preload the LCP image in `<head>`: `<link rel="preload" as="image" href="hero.webp" fetchpriority="high">`.
5. Target hero image ≤ **100 KB** on mobile, ≤ **200 KB** on desktop with a responsive `srcset`.

**Font checklist:**
- `font-display: swap` on all web fonts. A render-blocking font delays LCP.
- Preload the primary heading font: `<link rel="preload" as="font" crossorigin>`.

**General:**
- Inline critical CSS (above-fold styles) in `<style>` in `<head>`. Defer non-critical stylesheets.
- Avoid render-blocking third-party scripts in `<head>` (chat widgets, analytics) — defer or load async.
- Verify real-user LCP in PageSpeed Insights (field data), not just Lighthouse (lab). A 2.4s lab score can be a 3.8s field score on low-end Android.

## Tools & libraries
- **Builders:** Unbounce (Smart Traffic AI routing by segment; source of the 2024 Benchmark), Instapage (heatmaps + A/B), Leadpages, Replo (Shopify-native), Framer / Webflow (design-forward, SaaS-popular).
- **Behavior analytics:** Hotjar, Microsoft Clarity (free), Crazy Egg — find scroll-stop points, click maps, drop-offs.
- **A/B / MVT:** VWO, Optimizely; GA4 experiments for basic redirect/split tests. (Google Optimize is deprecated.)
- **Performance:** PageSpeed Insights / Lighthouse for LCP, INP, CLS.
- **Animation (only if it earns its weight):** GSAP for scroll-driven motion without tanking performance; Lottie/Rive for lightweight vector motion. See [Animation Libraries](../04-libraries-and-tools/animation-libraries-stack.md).
- **When NOT to use:** don't add a heavy page builder or animation lib if it pushes LCP past 2.5s or bloats the hero — performance beats polish on a conversion page.

## Do / Don't
**Do**
- Lead with a benefit headline that passes the 5-second test.
- Put ≥1 trust signal in the hero — missing proof above the fold is the single most common structural gap.
- Mirror the ad creative (message match) on every campaign page.
- Use first-person, value-driven CTA copy + a risk reducer.
- Handle the top 3 objections before the CTA; the rest in FAQ.
- Strip the nav and any link that competes with the goal.
- Design the hero for a ~400px mobile viewport first.
- Compress hero media; keep LCP under 2.5s.
- Build a separate page per ad group / source.

**Don't**
- Don't run multiple equal-weight CTAs.
- Don't ship vague headlines ("Empower / Supercharge / The future of X").
- Don't dump the entire UI into the hero.
- Don't use stock photos or abstract gradients that carry no information.
- Don't overload forms — every extra field costs conversions.
- Don't fake urgency (countdowns that reset on refresh) or fabricate reviews/logos.
- Don't introduce new doubts in the FAQ.
- Don't bait-and-switch the ad's promise.
- Don't design desktop-first and push the mobile CTA below the fold.

## Common pitfalls & how to avoid them
- **No social proof above the fold.** → Add a logo bar / star rating / user count to the hero — it's the most common structural gap.
- **Broken message match.** → Echo the ad's headline, image, and offer exactly; one page per ad group.
- **Generic CTA copy** ("Submit", "Click here", "Get started"). → Rewrite to first-person value: "Get my free audit".
- **No risk reducer.** → Add "No credit card required" / "Cancel anytime" under every CTA.
- **Stock / abstract imagery.** → Replace with product-outcome screenshots or real customer photos.
- **Dashboard dump.** → Show one annotated outcome, not the whole UI.
- **Nav left in place.** → Remove or trim to ≤2–4 links + CTA.
- **CTA overload.** → Pick one primary; downgrade or cut the rest.
- **Vague headline.** → Apply the litmus test; name the product and audience.
- **Form field overload.** → ≤5 fields; B2B TOFU = name + email; progressive profiling for the rest.
- **Fake urgency / fabricated proof.** → Use real scarcity only; comply with FTC.
- **FAQ that plants new doubts.** → Only answer objections already in the visitor's head.
- **Bait-and-switch.** → Deliver exactly what the ad promised.
- **Desktop-first build.** → Mobile-first; verify hero fits ~400px tall.
- **Uncompressed hero media → LCP >2.5s.** → Serve WebP/AVIF, set explicit width/height, `fetchpriority="high"` on the LCP image, preload it in `<head>`. Verify in PageSpeed Insights field data, not just Lighthouse lab.
- **Generic testimonials.** → Require name, title, company, and a metric.
- **One generic page for all campaigns.** → Scale to many targeted pages (40+ → ~12× leads vs ≤5).

## What real users complain about
Synthesized from CRO case studies, forums, and benchmark reports:
- **"I clicked the ad and the page was about something else."** Broken message match is the #1 felt failure — the visitor bounces in seconds.
- **"What does this even do?"** Vague hero copy ("Supercharge your synergy") forces the visitor to hunt; most leave instead.
- **"How much does it cost / do I need a card?"** Missing risk reducers and hidden pricing create last-second hesitation at the button.
- **"This form is asking for my life story."** 7–10 field B2B forms for a simple download feel like a tax; people abandon.
- **"The countdown reset when I refreshed."** Fake urgency, once noticed, nukes trust in the whole brand.
- **"The page took forever to load."** Heavy hero media → slow LCP → bounce before any copy is read.
- **"Every testimonial sounds fake."** Anonymous "Great product!" quotes read as planted; specificity and names are what convince.
- **"I couldn't tell which button to press."** Multiple equal CTAs cause paralysis.
- **"On my phone the button was off-screen / too small."** Desktop-first heroes and sub-44px targets frustrate the ~70% on mobile.

## Senior-level checklist
- [ ] H1 passes the 5-second test (what / who / next action clear in ≤5s).
- [ ] Subhead = "what it is" + "how it works", ≤2 sentences.
- [ ] Hero contains H1, subhead, one outcome visual, primary CTA, risk reducer, **and** ≥1 trust signal.
- [ ] Page headline/visual/tone match the source ad (one page per ad group).
- [ ] Exactly one primary goal; one CTA repeated at hero / mid / end; competing CTAs removed.
- [ ] CTA copy is first-person value; CTA is the highest-contrast element (squint test passes).
- [ ] CTA ≥44×44px touch, text contrast ≥4.5:1, visible focus ring.
- [ ] Every feature reframed as a benefit; each feature block handles its own objection.
- [ ] Top 3 objections handled before the CTA; FAQ covers the rest, introduces no new doubts.
- [ ] Social proof above the fold + a named, metric-bearing testimonial near the CTA.
- [ ] Nav stripped or ≤2–4 links + CTA; no leak links in the footer.
- [ ] Form ≤5 fields (B2B TOFU = name + email); progressive profiling planned.
- [ ] Pattern chosen deliberately: lead-form vs click-through; form placement matches traffic temperature.
- [ ] Page length matches offer complexity/risk; both lengths tested if uncertain.
- [ ] B2B CTA = relationship action (no "Buy now"); B2C click-through routes to checkout.
- [ ] Hero designed mobile-first for ~400px viewport; two-column collapses cleanly to one.
- [ ] LCP <2.5s, INP <200ms, CLS <0.1 (verified in Lighthouse/PSI field data + lab); hero image is WebP/AVIF, has explicit dimensions, `fetchpriority="high"`, and is preloaded.
- [ ] No fabricated reviews/logos/counts (FTC compliant); any urgency/scarcity signal is real and time-stable.
- [ ] Visual hierarchy: single dominant element per section; ≤3 semantic colors; section spacing ≥ 80px desktop / ≥ 48px mobile.
- [ ] Click-through pattern: CTA destination (navigate vs modal) chosen deliberately.
- [ ] Heatmap/session tooling installed to learn scroll-stop and drop-off after launch.
- [ ] A/B test plan: hypothesis written, primary metric defined, sample size calculated via power calculator before launch, minimum 2-week run.

## Sources
- Unbounce 2024 Conversion Benchmark Report — https://unbounce.com/conversion-benchmark-report/
- Nielsen Norman Group, scrolling and attention above the fold — https://www.nngroup.com/articles/scrolling-and-attention/
- WCAG 2.2 (contrast 1.4.3, target size minimum 2.5.8) — https://www.w3.org/TR/WCAG22/
- Apple Human Interface Guidelines (44pt minimum touch targets) — https://developer.apple.com/design/human-interface-guidelines/
- Material Design 3 (48dp touch targets) — https://m3.material.io/foundations/accessible-design/accessibility-basics
- web.dev Core Web Vitals (LCP/INP/CLS; INP replaced FID March 2024) — https://web.dev/articles/vitals
- Google/Deloitte mobile speed and revenue impact — https://www2.deloitte.com/content/dam/Deloitte/ie/Documents/Consulting/Milliseconds_Make_Millions_report.pdf
- HubSpot State of Marketing (40+ pages → 12× leads) — https://www.hubspot.com/marketing-statistics
- Spiegel Research Center, Northwestern — "How Online Reviews Influence Sales" — https://spiegel.medill.northwestern.edu/how-online-reviews-influence-sales/
- FTC final rule on fake/AI reviews (Aug 2024, effective Oct 2024) — https://www.ftc.gov/news-events/news/press-releases/2024/08/federal-trade-commission-announces-final-rule-banning-fake-reviews-testimonials
- Julian.com landing page formula & teardown (Purchase Rate = Desire − Labor+Confusion) — https://www.julian.com/guide/growth/landing-pages
- Evan Miller A/B test sample size calculator — https://www.evanmiller.org/ab-testing/sample-size.html

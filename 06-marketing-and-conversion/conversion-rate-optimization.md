# Conversion Rate Optimization (CRO)

> CRO is the discipline of increasing the share of visitors who complete a defined goal by removing friction, sharpening the value proposition, and validating changes with rigorous experiments. This guide gives you the heuristic frameworks (LIFT, Fogg, MECLABS), the funnel/statistics math, and the concrete element-level wins needed to design conversion-optimized pages and to know when *not* to A/B test.

**Apply when:** designing or auditing any page with a measurable goal — landing pages, checkout, signup, pricing, demo requests. · **Related:** [Landing Pages](../03-site-types/landing-pages.md) · [Copywriting & UX Writing](copywriting-and-ux-writing.md) · [Ethical Persuasion Psychology](persuasion-psychology.md) · [Web Performance & Core Web Vitals](web-performance-core-web-vitals.md)

## TL;DR — rules at a glance
- **Value proposition is the highest-leverage lever.** It sets the *ceiling* of your conversion rate. Fix it before touching color, layout, or microcopy.
- **Behavior = Motivation × Ability × Trigger (Fogg).** All three must fire at once. Reduce friction (Ability) first — it's the cheapest lever — then triggers, then motivation.
- **The best optimization is often deletion, not iteration.** Every removed form field and funnel step is a decision the user no longer has to make. 6 steps → 4 steps can be a +38% lift with zero creative changes.
- **Lead with outcome, not feature.** "Stop spending Fridays pulling reports. Get your week back." beats "Automated reporting for SaaS teams."
- **Page speed is a conversion factor.** ~7% conversion drop per 1s of mobile delay (Google/SOASTA 2017 mobile cohort); 1s load ≈ 3× the conversion rate of a 5s load. Confirmed directionally by Google/Deloitte 2019 (see §8).
- **Don't A/B test below ~300 conversions/month or ~10,000 sessions/month.** Use session recordings, 5-user tests, and surveys instead.
- **Fix usability before testing motivation.** Up to 70% of conversion problems are basic UX bugs you can *watch* in session replays.
- **CTA copy beats CTA color.** First-person framing ("Start my free trial") can swing CTR 10–90%. Color only matters for contrast.
- **Social proof removes anxiety.** Removing testimonials dropped conversions 35% in one practitioner case study (Michael Aagaard). Not a universal law — but treat proof as load-bearing, not decoration. Specific proof > generic quotes.
- **Message-match is upstream of the page.** If the ad says X and the page says Y, the page is dead before any on-page element matters.
- **One clear primary CTA per page.** Competing primary CTAs cannibalize each other; every secondary action must be visually subordinate.
- **Never peek and stop early.** Pre-calculate sample size; run minimum 2 weeks; stopping at first significance inflates false positives.
- **Guest checkout + trust badges near payment.** Forced account creation alone loses 28% of buyers.
- **Micro conversions are instrumentation**, not goals. They show *where* users drop so you fix the right stage — but always verify downstream macro impact.

---

## 1. CRO fundamentals: goals, micro vs macro, funnels

**Conversion goal** = the specific completed action you optimize for. Define it precisely and measurably before anything else.

- **Macro conversion** — the primary business outcome: purchase, paid signup, qualified demo request, lead form submit.
- **Micro conversion** — a step toward the macro goal: add-to-cart, email capture, video play, pricing-page view, account creation. These are the **instrumentation of your funnel** — they reveal *where* users drop so you fix the right stage.

> **Rule:** Optimize micro conversions only as diagnostics, never as ends. A higher add-to-cart rate does **not** always produce more purchases. Always measure downstream macro impact. If you genuinely double add-to-cart *and* the rest of the funnel holds, purchases multiply proportionally.

**Conversion rate** = conversions ÷ sessions (or ÷ unique visitors — pick one and stay consistent). Define the denominator explicitly; mixing sessions and visitors makes results incomparable.

### Funnel math (why deletion wins)
A funnel multiplies step completion rates. Small per-step gains compound:

```
6 steps × 85% each = 0.85^6 = 37.7% final conversion
4 steps × 85% each = 0.85^4 = 52.2% final conversion
→ +38% relative lift, zero creative changes — just two fewer steps
```

- **Funnel drop-off rule:** any step with **>15% drop-off** is a priority for *removal or radical simplification*, not for cosmetic optimization.
- **Diagnose before you design.** Pull funnel analytics first to find the worst-performing step. Optimizing the homepage while checkout is the real leak wastes effort.

---

## 2. The frameworks (use these to audit any page)

### LIFT Model (Chris Goward, 2009)
Six conversion forces. The **Value Proposition is the center**; the other five raise or lower it.

| Force | Question | Fix priority |
|---|---|---|
| **Value Proposition** (core) | Is the perceived benefit minus cost compelling? | Fix **first** — sets the ceiling |
| **Relevance** | Does the page match the traffic source and user intent? | 1 |
| **Clarity** | Is the value prop and CTA visually/verbally obvious? | 2 |
| **Anxiety** | What trust concerns block action? | 3 |
| **Distraction** | What competing CTAs/noise pull attention away? | 4 |
| **Urgency** | Is there a real deadline or scarcity? | 5 (last) |

Audit order: **Value Proposition → Relevance → Clarity → Anxiety → Distraction → Urgency.** Note Urgency is *last* — it amplifies a working page, it doesn't rescue a broken one.

### Fogg Behavior Model — B = M·A·T
**Behavior = Motivation × Ability × Trigger.** All three must converge simultaneously; it's a product, so any zero = no behavior.

- **Ability (friction) first** — cheapest lever. Remove fields, steps, decisions, load time.
- **Trigger second** — clear, well-placed CTAs and prompts at the moment of peak motivation.
- **Motivation last** — copy, social proof, ethical urgency. **Do not try to raise motivation on a broken UX.** Fix the friction, then persuade.

### MECLABS conversion sequence
Full formula: `C = 4m + 3v + 2(i − f) − 2a` (motivation m, value v, incentive i, friction f, anxiety a). The simplified takeaway: **motivation carries the highest weight (4×), value next (3×), friction and anxiety both negative.** This is why fixing **audience targeting / offer mismatch** yields far larger gains than design tweaks. If the wrong people arrive, no button color saves you. Reducing friction (−f) and anxiety (−a) both improve conversion — both are in your direct design control.

### Experiment prioritization — ICE / PIE / RICE
Rank tests *before* running them so you don't burn traffic on low-leverage experiments.
- **ICE** = Impact × Confidence × Ease
- **PIE** = Potential × Importance × Ease
- **RICE** = (Reach × Impact × Confidence) ÷ Effort

---

## 3. Value proposition clarity

The VP is the single highest-leverage variable. Get it right before anything else.

- **5-second test:** show the page to a first-time viewer for 5 seconds. If they can't answer *"What does this offer and why should I care?"*, the VP fails. Rewrite, don't redecorate.
- **Outcome-first beats feature-first.** Lead with the relief/result, not the mechanism.
  - Feature: `Automated reporting for SaaS teams`
  - Outcome: `Stop spending Fridays pulling reports. Get your week back.` *(real example: $0 → $4,800 MRR in 6 weeks)*
- **Reading level matters.** 5th–7th grade copy converted at **11.1%** vs **5.3%** for college-level — simplified copy *more than doubled* conversion. Target a 5th–8th grade reading level for headlines and CTAs.
- **Message-to-market match is upstream of the page.** Every paid ad must reuse the *exact* headline/offer language on the destination page. Mismatched ad-creative-to-page is the most common reason technically solid pages fail to convert.

---

## 4. CTA optimization

Copy and placement dominate. Color matters only for contrast.

### Copy (biggest lever)
- **First-person framing wins.** `Start my free 30-day trial` vs `Start your 30-day free trial` = **+90% CTR** in Michael Aagaard's original ContentVerve test (single experiment; directionally sound, not a universal guarantee — but widely replicated in practitioner work).
- **Specific over generic.** `Get Started` → `Book a Demo` moved CR 6.66% → 14.09% (**+111.55%**, PartnerStack). Test the *promise*, not just the verb.
- **Personalized CTAs outperform generic CTAs** when you can segment by visitor context — HubSpot's 2012 internal analysis cited ~202% better performance, but that figure is from a single dated study; treat it as directional evidence that specificity/personalization matters, not as a precise multiplier.

### Placement
- **Center-screen CTAs outperform side-aligned CTAs.** Blog-propagated figures like "682% more clicks" lack traceable primary sources — do not cite as fact. The directional rule (center/prominent placement wins) is sound; the precise number is not.
- **Inline/contextual CTAs outperform sidebar CTAs** — again, "121% higher CTR" is a secondary-source claim with no verifiable primary study. The rule stands: place CTAs where the user has just consumed the value argument, not in a sidebar removed from context.
- **Above-the-fold CTA improves CR ~10–15%**, but the primary CTA should also repeat after the value is established.
- **Social proof directly below the CTA increases CR.** Specific lift figures vary by page/audience; the mechanism (anxiety reduction at the moment of decision) is well-established. Pair them.
- **Place CTAs at eye-movement endpoints.** F-pattern (text-heavy, left-aligned scan) and Z-pattern (visual/minimal) — see [Visual Hierarchy](../00-foundations/visual-hierarchy.md).

### Count
- **One clear primary CTA per page.** Competing primary CTAs cannibalize each other; every additional primary action reduces overall conversion. Secondary actions must be visually subordinate (lower contrast, smaller, text-only).

### Color & size
- **No color universally converts best.** The winner is the one with the **highest contrast** against its surroundings. HubSpot A/B: red beat green 21% — *because the page background was green*.
- **60-30-10 rule:** 60% neutral, 30% secondary, 10% reserved for the CTA accent — creates hierarchy that points to the action.
- **Touch target:** minimum **44×44px** (Apple HIG), **48×48dp** (Material), **24×24 CSS px** (WCAG 2.5.8). Use the largest applicable.
- **WCAG contrast:** **4.5:1** for body text, **3:1** for large text and UI components — accessibility *and* conversion best practice.

```css
/* High-contrast, accessible primary CTA */
.cta-primary {
  min-height: 48px;
  min-width: 48px;
  padding: 14px 28px;          /* generous hit area beyond the label */
  font-size: 1.0625rem;
  font-weight: 600;
  color: #fff;
  background: #1d4ed8;          /* verify ≥4.5:1 vs label, ≥3:1 vs page */
  border-radius: 8px;
}
.cta-primary:focus-visible {
  outline: 3px solid #93c5fd;
  outline-offset: 2px;          /* keyboard users must see focus */
}
```

---

## 5. Social proof & anxiety reduction

Social proof removes anxiety — itself a conversion stopper.

- **Removing testimonials dropped conversion ~35%** in one practitioner case study (Michael Aagaard). Treat proof as load-bearing, not decoration.
- **Specificity wins.** Real names, photos, company, and **exact outcomes** ("cut onboarding from 6 weeks to 9 days") beat generic five-star blurbs.
- **Proof goes next to the action it supports:** testimonials near the CTA, trust seals near the payment form, customer logos/certifications on the pricing page. **Do not bury badges in the footer.**
- **Trust seals:** Baymard research shows ~18% of users explicitly look for security indicators before entering card data. Trust badges near payment fields raise CR for unknown brands; missing badges cause measurable abandonment (Baymard checkout usability research).
- **Video on landing pages:** Eyeview Digital (2012) reported +86% CR on a single landing page test. The precise figure is dated and site-specific — but product/customer video consistently outperforms static hero sections. Use real product or customer video, not stock footage.

---

## 6. Ethical urgency & scarcity

Real urgency amplifies; fake urgency destroys trust and invites regulatory risk.

- **Real scarcity/deadlines are not dark patterns.** Genuine low stock, a real cohort start date, a true offer expiry — fine to surface.
- **Fake urgency is a dark pattern.** Academic and practitioner research consistently links deceptive urgency/scarcity to buyer's remorse, trust erosion, and repeat-purchase suppression. CHI 2024 included studies on FOMO-driven deception; treat cited percentages from conference proceedings as context, not gospel, unless you have the specific paper. Regulatory exposure is real: FTC (US), CMA (UK), and EU consumer protection directives all target deceptive countdown timers and false "only 2 left" claims.
- **Test:** if you couldn't show the underlying data (real inventory count, real timer tied to a real event) to a regulator, don't display it. See [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md).

---

## 7. Forms, checkout & friction reduction

Friction reduction is the cheapest, most reliable CRO lever (Fogg: Ability first).

### Friction audit methodology
Before optimizing individual elements, audit the page systematically for friction:

1. **Analytics pass** — find the funnel step(s) with >15% drop-off. That's where to focus. Don't touch other steps first.
2. **Session replay triage** (Hotjar, Clarity, FullStory) — watch 20–30 sessions at the drop-off step. Tag: rage clicks, dead clicks, form abandonment points, scroll depth cut-offs, mobile layout breaks, JS errors in console.
3. **5-user moderated test** — a 45-minute session with 5 representative users on the drop-off flow reveals the majority of critical usability problems (Nielsen's law of diminishing returns: 5 users surface ~85% of problems).
4. **Survey / exit intent** — 1–2 question on-exit survey: "What stopped you from completing [action]?" Free-text answers surface systemic anxiety and confusion that analytics can't.
5. **Heuristic pass** using LIFT or Fogg: score each element on Value Prop / Clarity / Anxiety / Distraction / Urgency. Prioritize fixes by the combined friction score, not by gut feeling.

Only after completing steps 1–5 do you have a valid hypothesis for an A/B test — and often the friction audit alone surfaces fixes that don't need testing (known usability bugs, broken mobile layouts, missing trust signals).

### Form field discipline
- **Optimal checkout ≈ 8 fields.** 2024 average is 11.3 (down from 12.7 in 2019). **17% abandon** due to checkout complexity.
- **Use a single "Full Name" field.** 89% of sites split it needlessly.
- **Hide "Address Line 2" by default** (75% of sites wrongly show it) — reveal via a small "Add apartment/suite" link.
- **Make phone optional:** +4.2% completion, losing only 18% of phone capture — usually a net win.
- **Hide the coupon field by default.** A visible coupon field makes **35% pause to hunt for codes** and many abandon. Show it only after the user clicks "Have a code?".

### Account creation
- **Offer guest checkout.** Lifts completion up to **45%**. **28% abandon when forced to create an account.** Offer account creation *after* purchase as an opt-in (pre-fill from order data).
- **Progressive profiling:** collect only what advances the user one step. Defer account creation to confirmation; gather extra data post-purchase/post-signup.

### Checkout specifics
- **Express checkout** (Shop Pay et al.) converts **~1.72× better** — offer it prominently for returning/mobile users.
- **Surprise costs cause ~48% of abandonment.** Show shipping/taxes/fees as early as possible.
- **Remove unnecessary verification.** Removing email verification from e-commerce checkout: **+23% completion.** Cutting a SaaS signup from 4 → 2 fields **doubled trial starts.**

```html
<!-- Single name field; optional phone; deferred address line 2; deferred coupon -->
<label for="name">Full name</label>
<input id="name" name="name" autocomplete="name" required>

<label for="email">Email</label>
<input id="email" name="email" type="email" autocomplete="email"
       inputmode="email" required>

<label for="phone">Phone <span class="muted">(optional)</span></label>
<input id="phone" name="phone" type="tel" autocomplete="tel" inputmode="tel">

<button type="button" data-toggle="addr2">Add apartment, suite, etc.</button>
<button type="button" data-toggle="coupon">Have a discount code?</button>
```

For deeper input UX (validation timing, error messaging, autocomplete tokens), see [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md).

---

## 8. The page-speed-to-conversion link

Speed is a conversion factor, not just SEO. See [Web Performance & Core Web Vitals](web-performance-core-web-vitals.md).

- **Amazon internal data:** every **100ms of latency = ~1% revenue loss.**
- **Mobile:** every **1s of delay ≈ 7% conversion drop** (Google/SOASTA 2017, mobile cohort). Directionally confirmed by subsequent Google/Deloitte work.
- **Bounce:** 1s→3s load = **+32% bounce**; 1s→5s = **+90% bounce** (Google internal data, 2018).
- **1s load ≈ 3× the conversion rate of a 5s load.**
- **Google/Deloitte "Milliseconds Make Millions" (2019):** a 100ms improvement = **10.1% conversion lift in travel, 8.4% in e-commerce.**

**Core Web Vitals targets** (also conversion thresholds):

| Metric | Good | Poor |
|---|---|---|
| **LCP** (Largest Contentful Paint) | <2.5s | >4s |
| **INP** (Interaction to Next Paint) | <200ms | >500ms |
| **CLS** (Cumulative Layout Shift) | <0.1 | >0.25 |
| **TTFB** (Time to First Byte) | <200ms cached / <600ms dynamic | — |

> **INP replaced FID as a Core Web Vital in March 2024.** Do not optimize for FID anymore. **CLS remains a Core Web Vital** — layout shifts on load destroy trust and misfire taps.
> **Priority order:** a site loading in 8s on mobile loses ~7%/second — more than most headline or CTA tests can recover. Fix speed *before* running copy/design tests.

---

## 9. Rigorous A/B testing

A/B testing is for *validating motivation/copy changes after* usability is fixed and known best practices are implemented. It is not a substitute for fixing bugs.

### When you may A/B test (traffic gates)
| Monthly conversions | Approach |
|---|---|
| **<20–30** | Don't A/B test. Use session recordings, 5-user tests, surveys. |
| **30–100** | Only test *radical* changes (offer, pricing, whole-page). |
| **100–300** | A/B with statistical shortcuts; expect slow reads. |
| **300+** | Traditional A/B testing viable. |

- **Floor:** stores under ~300 orders/month or ~10,000 sessions/month **cannot reach valid significance** in a reasonable timeframe.
- **Each variant needs a minimum ~250 conversions** to avoid underpowering. For high reliability at large scale, target ~**30,000 visitors + ~3,000 conversions per variant**.

### Statistics discipline
- **Significance:** 95% confidence (**p = 0.05**). **Power:** **80%.**
- **MDE (minimum detectable effect):** 1–3% relative lift for large orgs; **2–5% relative** for most. Smaller MDE = much larger sample. Pick MDE *before* the test and compute sample size from it.
- **Duration:** minimum **2 weeks** (cover ≥2 full weekly cycles), maximum **6–8 weeks** (longer risks cookie churn, seasonality, external events).
- **Pre-calculate sample size and run to it.** Stopping when "results look good" is the #1 error — early significance is usually noise and **peeking inflates false-positive rates dramatically.**

### Test design rules
- **One variable at a time.** Changing headline + color + layout together makes wins unattributable. (Multivariate testing exists but needs far more traffic and a clear interaction hypothesis.)
- **Never declare a winner before the pre-set sample/duration is reached**, no matter how promising day-2 looks.
- **Don't optimize a micro conversion in isolation** — confirm the macro metric moved with it.

```
Sample-size sketch (per variant):
  baseline CR = 5%, target relative MDE = +10% (→ 5.5%), 95% conf, 80% power
  → ≈ 30,000 visitors per variant
  At 10,000 sessions/mo split 50/50 → ~6 weeks to read. Borderline — widen MDE
  or pursue qualitative methods instead.
```

---

## 10. Common CRO wins (high-confidence, ship before testing)

These are established best practices — implement them, *then* test refinements.

- **Remove the nav menu from dedicated landing pages.** Eliminating the nav removes the biggest single distraction; consistent practitioner finding across landing page experiments (WiderFunnel, HubSpot, and others report large lifts; individual results vary widely).
- **Add real product/customer video to the hero/landing page.** Video outperforms static heroes consistently; specific lift figures are page-dependent — treat any single published number as case-study context, not expectation.
- **Place social proof directly below the CTA.** The mechanism (reduces decision anxiety at action point) is well-established; apply it.
- **Guest checkout:** up to **+45% completion** (Baymard Institute checkout research).
- **Cut funnel steps / form fields:** see funnel math (§1) and forms (§7).
- **Simplify copy to 5th–7th grade reading level:** roughly doubled CR in the cited academic comparison (§3).
- **Express/one-tap checkout** for mobile and returning users: **~1.72× better** (Shopify Shop Pay internal data).
- **Build dedicated landing pages per audience/offer.** Each page that matches a specific source + intent removes message mismatch. HubSpot's 2011–2012 customer data showed companies with 40+ landing pages generated 500%+ more leads than those with <5 — this is a correlation (more pages = more targeted acquisition programs), not a design law, but the underlying principle is sound: specific pages for specific audiences outperform generic ones.

---

## 11. Mobile & channel reality

- **Mobile cart abandonment is ~86%** (vs ~74% desktop — SaleCycle/Statista aggregates; exact figures shift annually, treat as directional). Mobile CR typically **~2–2.5%** vs ~3% desktop — mobile converts lower not because users intend less but because UX friction and performance are worse.
- **Don't treat mobile as "smaller desktop."** Different interaction patterns, lower patience, worse performance. A site with 70% mobile traffic and no mobile-specific optimization is systematically underperforming. See [Touch Ergonomics](../02-responsive-adaptive/touch-ergonomics-cross-device.md).
- **Channel converts very differently.** Email traffic ~**19.3%** — nearly double paid search. Optimize and attribute per channel; don't average a "site conversion rate" and call it strategy.
- **B2B SaaS homepages serve cold traffic.** Track high-intent signals (demo request, pricing view, branded search) as *separate* conversion goals with their own optimization — don't treat the homepage as the conversion engine.
- **Optimize per traffic temperature, not just per device.** Cold traffic (display, social, generic search) requires maximum value-proposition clarity and social proof — it has no existing brand trust. Warm traffic (retargeting, email list, branded search) can skip the trust-building and go straight to offer/CTA. Hot traffic (cart abandonment email, checkout return) needs nothing but friction removal. Using the same page for all three systematically underperforms each.

---

## Concrete specs & numbers

- **Touch target:** 44×44px (Apple HIG) · 48×48dp (Material) · 24×24 CSS px (WCAG 2.5.8).
- **Contrast:** 4.5:1 body text, 3:1 large text / UI components (WCAG).
- **Core Web Vitals:** LCP <2.5s good / >4s poor · INP <200ms good / >500ms poor (replaced FID Mar 2024) · CLS <0.1 good / >0.25 poor · TTFB <200ms cached, <600ms dynamic.
- **Significance/power:** p=0.05 (95% confidence), 80% power. MDE 2–5% relative (most), 1–3% (large orgs).
- **A/B sample:** ≥250 conversions/variant minimum; ~30,000 visitors + ~3,000 conversions/variant for high reliability.
- **A/B duration:** 2 weeks min, 6–8 weeks max. A/B traffic floor ~300 conversions or ~10,000 sessions/month.
- **Checkout fields:** optimal ~8; 2024 avg 11.3 (was 12.7 in 2019).
- **Reading level:** 5th–7th grade → 11.1% CR vs college-level 5.3%.
- **Benchmarks (context, not targets):** median landing page CR 6.6% (Unbounce Q4 2024, 41k pages; top 10% >10%) · global e-commerce avg CR ~2.5–3% (IRP Commerce / Statista aggregates; varies by vertical) · Google Ads avg CR ~7% across industries (WordStream aggregate; industry range roughly 3–13%) · email traffic CR typically highest of any channel at ~15–20% (Monetate / Episerver; varies by list quality).
- **Funnel:** 6 steps @85% = 37.7%; 4 steps @85% = 52.2% (+38%).
- **Spend reality:** companies spend $1 on CRO for every $92 on acquisition (Econsultancy/Adobe survey data) — CRO is systematically underinvested relative to its ROI.

---

## Tools & libraries

- **Web analytics / funnels:** GA4, Plausible, or Matomo to find the worst-performing funnel step *before* optimizing. Define events for each micro/macro conversion.
- **Session replay & heatmaps:** Hotjar, Microsoft Clarity (free), FullStory — watch real sessions to catch hidden buttons, mobile overlap, Safari JS errors *before* testing. Up to 70% of conversion issues are basic UX bugs found this way.
- **A/B testing:** GA4 + a server-side or edge experimentation layer; VWO, Optimizely, or open-source GrowthBook. **Use only when above the traffic gate.**
- **Sample-size calculators:** Evan Miller's, Optimizely's — compute required N from baseline CR + MDE *before* launching.
- **Surveys / qualitative:** on-site micro-surveys (exit intent, post-purchase), 5-user moderated tests — the right tool *below* the A/B traffic gate.
- **When NOT to reach for tools:** don't install an A/B platform on a low-traffic site (you'll chase noise); don't add a session-replay script that itself harms LCP/INP — load it deferred and audit its performance cost.

---

## Do / Don't

**Do**
- Fix value proposition and message-match before any element-level work.
- Reduce friction (Ability) first: delete fields, steps, and decisions.
- Watch session replays to find UX bugs before testing anything.
- Use one clear primary CTA with first-person, outcome-led copy.
- Put proof and trust badges adjacent to the action they support.
- Pre-calculate sample size; run tests to a fixed duration (2–8 weeks).
- Optimize per channel and per device; track high-intent B2B signals separately.
- Show all costs early; offer guest checkout and express checkout.

**Don't**
- Don't A/B test below ~300 conversions / ~10,000 sessions per month.
- Don't peek and stop tests early at first significance.
- Don't change multiple variables in one A/B test.
- Don't optimize the homepage when checkout/pricing is the real leak.
- Don't treat CTA color as the primary lever — copy and placement win.
- Don't use fake urgency/scarcity (buyer's remorse, trust loss, legal risk).
- Don't force account creation before purchase.
- Don't show coupon fields, multiple name fields, or Address Line 2 by default.
- Don't run copy/design tests while the page loads in 8s on mobile.

---

## Common pitfalls & how to avoid them

- **Testing multiple elements at once** → can't attribute the win. Test one variable; use multivariate only with sufficient traffic and a real interaction hypothesis.
- **Peeking / early winners** → false positives. Pre-calc sample size, run to duration regardless of how good day-2 looks.
- **Optimizing the wrong page** → homepage tweaks while checkout leaks. Read funnel analytics first; attack the worst step.
- **Testing on insufficient traffic** → tests never reach significance. Below ~300 conversions/month, switch to qualitative methods.
- **Color-first CTA thinking** → marginal, inconsistent gains. Optimize contrast, then copy and placement.
- **"Mobile = smaller desktop"** → systematic underperformance. Optimize mobile interaction and speed separately.
- **Assuming the problem is the landing page** → often it's audience-message mismatch, misleading ad creative, or post-click checkout friction.
- **Fake urgency/scarcity** → documented buyer's remorse, eroded trust, regulatory risk (FTC/CMA/EU). Use only real deadlines/inventory.
- **Visible coupon field** → 35% pause to code-hunt and abandon. Hide behind "Have a code?".
- **Forced account creation** → 28% abandon. Offer guest checkout; make account opt-in post-purchase.
- **Ignoring page speed during testing** → losing ~7%/second on mobile, more than most tests recover.
- **Trust badges in the footer** → place them next to payment, proof next to CTA, certs on pricing.
- **Optimizing micro conversions as proxies** → higher add-to-cart ≠ more purchases. Verify macro impact.
- **Skipping session-replay triage** → up to 70% of issues are observable UX bugs (hidden buttons, Safari errors, mobile overlap) you should fix before testing.

---

## What real users complain about
Synthesized from practitioner reports and UX research patterns:

- **"Forced to make an account just to buy one thing."** Guest checkout absence is a top abandonment driver (28% leave for this alone).
- **"Surprise shipping/fees at the last step."** ~48% of abandonment traces to unexpected costs revealed late.
- **"The coupon box made me feel like I'm overpaying"** — users leave to hunt codes (35% pause) and don't return.
- **"It said only 2 left… every time I visit."** Obvious fake scarcity reads as manipulation and kills trust — and creates buyer's remorse and repeat-purchase suppression.
- **"I couldn't tell what the product actually does."** Failed 5-second VP test — vague, feature-jargon headlines.
- **"The form asked for my phone, address line 2, and a company name I don't have."** Irrelevant/over-long forms.
- **"The page took forever on my phone."** Slow mobile LCP/INP; users bounce before the hero renders.
- **"The button didn't seem to do anything"** — hidden/broken CTA, missing loading state, or Safari/mobile-specific JS error visible only in session replay.
- **"Tap targets were too small / overlapping on mobile."** Sub-44px targets and overlap from untested responsive layouts.

---

## Senior-level checklist
- [ ] Primary conversion goal defined with an explicit denominator (sessions vs visitors).
- [ ] Micro conversions instrumented per funnel step; worst step identified from analytics.
- [ ] Value proposition passes the 5-second test; headline is outcome-led, 5th–8th grade reading level.
- [ ] Message-match verified: ad/source copy matches the destination headline and offer.
- [ ] Session replays reviewed; UX bugs (hidden buttons, mobile overlap, JS errors) fixed before any test.
- [ ] One clear primary CTA; first-person, specific copy; center-aligned where layout allows.
- [ ] CTA touch target ≥44×44px (≥48dp mobile); WCAG contrast 4.5:1 text / 3:1 UI; visible focus state.
- [ ] Specific social proof near the CTA; trust badges near payment; logos/certs on pricing.
- [ ] Nav removed on dedicated landing pages; distractions minimized.
- [ ] Forms trimmed to essentials: single Full Name, optional phone, Address Line 2 hidden, coupon hidden.
- [ ] Guest checkout offered; account creation deferred to opt-in post-purchase; express checkout available.
- [ ] All costs shown early; no surprise fees at the final step.
- [ ] Core Web Vitals in "good": LCP <2.5s, INP <200ms, CLS <0.1, TTFB <600ms — verified on mobile.
- [ ] Any urgency/scarcity is backed by real data (inventory/deadline); no dark patterns.
- [ ] A/B test only above the traffic gate (~300 conversions / ~10,000 sessions/month).
- [ ] Sample size pre-calculated from baseline CR + MDE; one variable per test; run 2–8 weeks; no peeking.
- [ ] Macro impact confirmed (not just a moved micro metric); per-channel and per-device results reviewed.

---

## Sources
- Unbounce Conversion Benchmark Report (Q4 2024 dataset, ~41,000 landing pages) — median landing page CR 6.6%.
- Google / Deloitte, "Milliseconds Make Millions" (2019) — 100ms improvement → 10.1% travel / 8.4% e-commerce conversion lift.
- Google / SOASTA (2017) — mobile load time to conversion; ~7%/s figure origin.
- Google (2018) — load-time to bounce rate figures (1s→3s = +32%; 1s→5s = +90%).
- Apple Human Interface Guidelines — 44×44pt minimum tap target.
- Google Material Design — 48×48dp minimum touch target.
- W3C WCAG 2.2 — 2.5.8 Target Size (24×24 CSS px), 1.4.3/1.4.11 contrast (4.5:1 / 3:1).
- web.dev Core Web Vitals — LCP, INP (replaced FID March 2024), CLS, TTFB thresholds.
- Michael Aagaard / ContentVerve — first-person CTA framing experiment ("my" vs "your"); testimonial removal case study.
- HubSpot (2011–2012) — landing page volume vs lead volume correlation; CTA color/contrast study; note: most HubSpot blog stats are from internal customer cohort analyses, not peer-reviewed studies — treat as directional.
- Baymard Institute — checkout usability and cart-abandonment research (field counts, guest checkout, cost transparency, trust seals).
- MECLABS Institute — conversion heuristic formula `C = 4m + 3v + 2(i − f) − 2a`.
- Shopify / Shop Pay — express checkout conversion lift (~1.72×, internal data).
- CHI 2024 proceedings — deceptive urgency/scarcity research; note: CHI publishes hundreds of papers per year — specific claims should be traced to a paper title/authors before citing as authority.
- FTC, CMA, EU Consumer Rights Directive — regulatory framework for deceptive scarcity/urgency claims.

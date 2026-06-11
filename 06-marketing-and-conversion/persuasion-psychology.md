# Ethical Persuasion Psychology

> How to design conversion-positive interfaces using established behavioral science (Cialdini's 7 principles, the Fogg Behavior Model, cognitive biases, nudges) while staying on the ethical side of the line that separates persuasion from dark patterns. The senior-level outcome: you can deploy social proof, scarcity, anchoring, and defaults that lift conversion *and* survive user scrutiny, screenshots, and regulators.

**Apply when:** designing any page meant to drive a decision — landing pages, pricing, checkout, signup, onboarding, lead magnets. · **Related:** [Conversion Rate Optimization](./conversion-rate-optimization.md) · [Copywriting & UX Writing](./copywriting-and-ux-writing.md) · [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md) · [The Laws of UX](../01-ux-fundamentals/laws-of-ux.md)

## TL;DR — rules at a glance

- **Persuasion vs. manipulation, the one test:** if the user understands the offer, freely accepts, and stands to gain → persuasion. If misled, pressured, or tricked → manipulation. Would you explain this design to the user face-to-face? If no, don't ship it.
- **Cialdini's 7:** Reciprocity, Commitment/Consistency, Social Proof, Authority, Liking, Scarcity, Unity. Sequence: **give first (reciprocity) → micro-yeses (commitment) → scarcity only if true.**
- **Fogg B=MAP:** Behavior happens only when **M**otivation, **A**bility, and a **P**rompt converge above threshold *at the same moment*. Diagnose failures **Prompt → Ability → Motivation**, in that order. Teams wrongly blame motivation first.
- **Loss aversion ≈ 2.0×:** losses hurt ~1.5–2.5× more than equal gains please (Kahneman/Tversky). Frame high-stakes offers as "avoid losing X," not just "gain X." Don't use loss framing on low-stakes asks (newsletter) — it reads as manipulation.
- **Anchor first:** the first number sets the reference. Show crossed-out original price → then discounted price (anchoring + loss aversion combo). Anchor must be *authentic* or it's recognized as fake.
- **Defaults are the strongest nudge.** Make opt-out exactly as easy as opt-in (ideally 1 click). Asymmetric friction = dark pattern.
- **Social proof works, freshness rules:** 74–83% of consumers distrust reviews older than 3 months (BrightLocal Local Consumer Review Survey). Show dates, photos, verified-buyer badges. A stale 4-star wall is *worse* than no reviews.
- **Scarcity must be true.** "Only 2 left" when genuinely 2 left = ethical. Same copy regardless of stock = dark pattern that destroys trust faster than it builds urgency.
- **1–2 dominant persuasion signals per page section.** Stacking scarcity timer + popup proof + authority badges + urgency copy creates anxiety, not conversion.
- **Reduce friction before adding motivation.** 39% abandon checkout over surprise costs; 19% over forced account creation. Fixing friction beats any persuasion tactic.
- **Reciprocity gift must be specific, useful, immediate.** A vague "ebook" breeds cynicism, not obligation.
- **Commitment via micro-steps:** 2-step checkout where step 1 is non-threatening cuts abandonment dramatically (foot-in-the-door). Always show total step count.
- **Unity > similarity:** "I built this because I faced exactly your problem" (shared identity) beats demographic targeting.
- **Peak-End Rule:** users judge an experience by its peak moment and its end. Invest in the success/confirmation screen and error recovery, not just the happy path.
- **Decoy effect:** a 3rd pricing tier that is asymmetrically dominated makes one other tier look clearly superior. Use honestly — all tiers must be real products.
- **Reactance:** push past user tolerance and they comply less, not more. If CTAs are being dismissed unread, diagnose reactance before adding motivation.
- **Regulatory floor (2026):** pre-ticked consent = GDPR violation. Hard-to-cancel subscription = DSA/CMA target. Fake urgency = FTC exposure. These are not soft ethical preferences — they carry financial penalties.

---

## Cialdini's 7 principles — UX implementations

Each principle activates a heuristic that runs largely below conscious deliberation. Used truthfully they speed good decisions; used falsely they erode trust permanently.

### 1. Reciprocity — give value before asking
People feel obligated to return favors. In UX, give *first*: a genuinely useful free tool, template, audit, or guide before requesting email/payment.
- **Rule:** the gift must be specific, useful, and delivered *immediately*. A vague "free ebook" creates cynicism, not obligation.
- **Pattern:** free interactive tool → results shown → "Save your results" (account ask at the value peak). See the resume-builder pattern under Loss Aversion below.
- **Sequencing:** reciprocity goes *first* in any persuasion sequence — you earn the right to ask.

### 2. Commitment & Consistency — micro-yeses
Once people take a small step, they act consistently with it. **Foot-in-the-door:** multi-step forms outperform single-step.
- A 2-step checkout where step 1 is non-threatening (email + zip) dramatically reduces abandonment vs. one giant form.
- Onboarding: ask for the smallest commitment first ("pick one goal"), then build. Each completed micro-step raises follow-through.
- Pair with the **Zeigarnik Effect** (incomplete tasks nag): progress bars and "2 of 4 complete" checklists pull users back.

### 3. Social Proof — others like me did this
The single highest-leverage on-page persuasion lever for most sites. Detailed patterns in [Effective social-proof patterns](#effective-social-proof-patterns) below.
- **Hierarchy (strongest → weakest):** personal network recommendation > verified-buyer review *with photo* > star rating > aggregate count ("50,000+ customers").
- Reviews increase conversion; displaying 5+ reviews lifted conversion ~270% vs. 0 reviews on lower-priced products (Spiegel Research Center / Northwestern, 2017). Individual site impacts vary widely — validate with your own A/B tests.

### 4. Authority — credible expertise
People defer to credible experts. Show *verifiable* authority.
- **Signal hierarchy (most → least effective):** verified expert credentials > institutional logos > media mentions > awards > aggregate user counts.
- Make logos clickable to the actual mention; unlinked "As seen in" rows read as decoration. Don't claim authority you can't substantiate.

### 5. Liking — we comply with those we like and resemble
Warmth, attractiveness, similarity, and genuine compliments increase compliance.
- Human faces, real team photos, conversational copy, and shared values build liking.
- Avoid hollow flattery; specific recognition ("You shipped 12 projects this month") beats generic praise.

### 6. Scarcity — limited = valuable
Scarcity raises perceived value and urgency — **only when truthful.**
- **Truthfulness rule:** "Only 2 rooms left" when genuinely scarce = ethical. Identical message regardless of actual availability = dark pattern.
- Types: quantity ("3 left"), time ("sale ends Sunday"), access ("invite-only beta"). Each must reflect reality and be verifiable on the backend.
- Deploy scarcity *last* in the sequence and only if contextually relevant — never as the opening move.

### 7. Unity — shared identity (Cialdini's 7th, *Pre-Suasion*, 2016)
The strongest of the seven. Shared *identity* (same profession, team, values, lived experience) beats mere similarity.
- "**We both faced this problem**" > "here is a solution." "I built this because I faced exactly what you're facing" activates shared identity more than demographics.
- **Implementation:** a founder story that mirrors the user's problem; a real community with real shared-identity content.
- **Pitfall:** "join our family" rhetoric with no actual community is recognized instantly as marketing speak.

### Sequencing the 7 principles on a single page
Don't scatter all seven at once. A well-sequenced page uses them in order of cognitive trust-building:

1. **Reciprocity first** — give value before asking anything. Tool/guide/audit above the fold or on first engagement.
2. **Authority + Liking** — establish credibility and rapport in the hero/about section. Credentials, team faces, founder story.
3. **Unity** — shared-identity story ("We built this because we faced your problem"). Activates before the pitch.
4. **Social Proof** — verified reviews, case studies, logos, user counts. After credibility is established, not as a substitute for it.
5. **Commitment micro-step** — a low-friction CTA ("Start free," "See your score") that creates the first yes.
6. **Scarcity** — only if truthfully applicable. Final nudge on a high-intent user, not as an opening move.
7. **Consistency prompts** — post-signup onboarding. Now that users committed, use progress/Zeigarnik to complete setup.

Stacking all seven simultaneously → anxiety + reactance. Sequencing them → trust ladder.

---

## The Fogg Behavior Model — B = MAP

**Behavior = Motivation × Ability × Prompt.** A target behavior happens only when all three exceed the action threshold *simultaneously*. A missing prompt means no behavior regardless of motivation. A present prompt cannot produce behavior if ability is below threshold regardless of motivation. Motivation matters only after prompt and ability are confirmed adequate.

```
        high │  ........ action line ........
            M│  ●  fires here          ● fires here
   motivation│   (hard task,            (easy task,
            │    high motiv.)            low motiv.)
        low └───────────────────────────────────
              hard ←──── Ability ────→ easy
              (Prompt must be present at the moment, or nothing fires)
```

### Diagnostic sequence — Prompt → Ability → Motivation
Diagnose conversion failure in this order. Most teams skip straight to motivation and miss friction.
1. **Is there a prompt at all?** Right CTA, right place, right time? No prompt → no behavior, regardless of motivation.
2. **Is ability high enough?** Is the action 1 click or 6 clicks? Reduce steps, fields, cognitive load *before* anything else. 39% of checkout abandons are friction/cost, not motivation.
3. **Only now: is motivation sufficient?** If prompt and ability are solid and it still fails, *then* address motivation (value prop, proof, framing).

### Contextual prompts (timing is everything)
Same prompt, different result depending on timing relative to ability and engagement:
- ❌ "Complete your profile" notification sent cold → ignored. User's ability/engagement is low.
- ✅ The same nudge appearing *right after* the user uploads their first photo → succeeds. Ability is already high; the user is mentally engaged in the relevant flow.
- **Rule:** trigger prompts at moments of high ability and high engagement, not on a schedule.

### Fogg's design maxims (the ethical anchor)
From BJ Fogg, *Tiny Habits* (2019) and behaviormodel.org:
- **"Help people do what they already want to do."**
- **"Help people feel successful."**
Ethical persuasion *strengthens* existing user intention; it does not manufacture pressure. If you're inventing the desire, you've crossed into manipulation.

### The prompt-overuse trap
Prompts (push notifications, tooltips, modals, emails) are the most abused B=MAP element: easy to add, politically near-impossible to remove. **Every added prompt dilutes every previous prompt.** Audit and *cut* prompts before adding new ones.

---

## Cognitive biases used in design

Use these to make good decisions easier, not to obscure costs.

### Anchoring
The first number seen sets the reference for all that follow.
- **Pricing:** show the most expensive/anchor option first; show crossed-out original price, then discounted price — **anchoring + loss aversion** in one move.
- **Donations:** pre-populated suggested amounts consistently increase donation rates versus open-ended fields; test your own amounts — optimal anchors vary by cause and audience.
- **Evidence:** Tversky & Kahneman 1974 — median estimates differed 4×+ (512 vs. 2,250) based solely on the anchor number. Ariely, Loewenstein & Prelec 2003 ("Coherent Arbitrariness," QJE) — willingness-to-pay correlated significantly with SSN last digits across product categories; magnitude varied by condition and product.
- **Pitfall:** an implausible crossed-out price is instantly read as fake. The anchor must be authentic.

### Loss Aversion
Losses feel ~1.5–2.5× (commonly cited **2.0×**) more painful than equal gains feel good. Neuro research: losses activate pain regions more intensely than gains activate reward centers.
- **Framing rule:** "Save $50" outperforms "Get $50 off" in high-stakes decisions. "Don't lose your progress" beats "Continue for free" for onboarding re-engagement.
- **The resume-builder pattern (ethical):** let users *build* the resume (creating endowment), then ask for an account only at download — losing the work they already made is the motivator. Compare with forced-account-first, which loses 19%.
- **Pitfall:** loss framing in low-stakes contexts ("You're about to lose your spot!" for a newsletter) reads as manipulative. Reserve it for genuinely high-value, time-sensitive situations.

### Framing
The same fact, framed differently, changes the decision. "95% fat-free" beats "5% fat." Frame around what the user keeps or avoids losing, not abstract gains. Tie to loss aversion in high-stakes flows.

### Defaults
The pre-selected option is what most users keep — defaults are the **strongest** nudge.
- **Ethical default rule:** communicate the *rationale* for the default, show benefits **and** drawbacks, and make opt-out as easy as opt-in (1 click ideally).
- **Evidence:** defaults routinely double or triple uptake vs. explicit opt-in; the classic result is organ-donor registration rates (Johnson & Goldstein 2003, *Science*) differing by 40–60 percentage points across countries based solely on default. In UX, even minor opt-out friction (an extra click) measurably raises default-keep rates. The friction *is* the manipulation — remove it.
- **Pitfall:** pre-checked add-ons users don't notice → chargebacks and "they sneaked this in" reviews. Net negative.

### Endowment effect (the "Canva effect")
People over-value what they already possess.
- **Pricing copy:** frame the premium tier as **"Everything in Free, plus…"** so users feel they keep what they have (endowment) *and* gain more — not that they lose the free tier. This also neutralizes confirmation bias against upgrading.
- Free trials and "build first, sign up later" flows create endowment that loss aversion then activates.

### Decoy effect (asymmetric dominance)
When a third, inferior option is added, it makes one of the other two options look significantly better by comparison.
- **Pricing application:** a 3-tier pricing page where the middle tier is "dominated" (worse on key dimensions) by the top tier steers users toward the top tier. The middle tier exists as a decoy, not as a real choice.
- **Ethical constraint:** all tiers must be genuinely available and honestly described. A tier designed purely to make another look good is fine; a tier that does not exist or contains hidden downgrades is fraud.
- **Implementation:** place the recommended tier adjacent to the decoy; ensure the differentiating features are scannable side-by-side.

### Reactance — the persuasion backfire
When users perceive that their freedom of choice is being threatened, they resist — often doing the opposite of the intended action. Reactance is the direct enemy of aggressive persuasion tactics.
- **Triggers:** aggressive scarcity ("You must decide now"), confirmshaming, forced paths, repeated popups, guilt copy ("Don't miss out!").
- **Rule:** every persuasion layer you add past the user's tolerance threshold generates negative reactance that cancels earlier positive work. A single pushy modal can undermine the whole page.
- **Diagnostic:** if users are dismissing your CTAs without reading them, you may have triggered reactance — not a motivation problem. Session replays and heatmaps reveal this.
- **Mitigation:** give users explicit control ("Remind me later," "Not right now" in neutral language), reduce pressure cues, and give choice rather than eliminating it.

### Cognitive load and persuasion vulnerability
High cognitive load degrades the capacity to evaluate claims critically. Complex pages that overwhelm working memory (too many features, deep jargon, visual noise) can increase conversion in the short term but exploit reduced critical evaluation — a dark pattern.
- **Ethical rule:** use cognitive simplicity (chunking, plain language, progressive disclosure) to make decisions *easier to evaluate*, not harder to resist.
- **Practical constraint:** Miller's Law (7 ± 2) is not just a UX nicety — violating it while pitching a complex product actively impairs informed consent.

### Sunk cost effect in onboarding
Users who have already invested time/data into a flow overweight that investment in their continuation decision — they complete flows not because they want the outcome but because they don't want to "waste" what they've done.
- **Ethical use:** multi-step onboarding that shows progress ("Step 3 of 5 — 60% done") leverages this honestly when users genuinely want the outcome.
- **Dark pattern boundary:** hiding step count so users don't know how much remains, or artificially lengthening flows to increase sunk cost, is manipulation. Always show total step count and let users exit cleanly.

### Supporting laws (see [Laws of UX](../01-ux-fundamentals/laws-of-ux.md))
- **Miller's Law** — working memory ≈ 7 ± 2; chunk nav, pricing tiers, feature lists into groups of ≤5–7.
- **Hick's Law** — decision time grows with the number of choices; fewer clear options = faster decisions. *Caveat:* if reducing 5→2 doesn't help, the problem is value-prop clarity, not choice count.
- **Serial Position Effect** — first and last items are remembered best; put the most important pricing tier / most compelling feature at the **start and end** of lists.
- **Peak-End Rule** — judged by peak intensity + the ending; make the success/confirmation screen and error recovery outstanding.
- **Zeigarnik Effect** — incomplete tasks nag; progress bars and onboarding checklists exploit this ethically to drive completion.
- **Doherty Threshold** — UI must respond <400ms or engagement/flow drops; perceived speed is a trust signal that reduces abandonment.
- **Fitts's Law** — primary CTA must be large and close to the decision point.
- **Jakob's Law** — users expect your site to work like the others they know; deviate from convention only when the benefit is large and demonstrated.

---

## Nudges vs. dark patterns — the ethics line

**Nudges preserve freedom of choice. Dark patterns remove it.** A nudge makes the good choice easier while leaving the other choices fully available and easy. A dark pattern deceives, misdirects, shames, or obstructs the alternative.

> **Dark pattern (NN/g):** a design pattern that prompts users to take an action benefiting the company by **deceiving, misdirecting, shaming, or obstructing** alternative choices.

### The Ethics Test (5 questions — all must pass)
1. Is the user **fully informed**?
2. Is it **easy to opt out** (as easy as opting in)?
3. Is the design **respectful of user intent**?
4. Is this **nudging or shoving**?
5. Would you be **comfortable explaining this design to users face-to-face**?

### The asymmetric-friction rule
Ethical design applies **equal friction** to desirable and undesirable actions. The classic dark-pattern signal: subscribe is 1 click but unsubscribe is 6 steps (the Audible example). If your cancel/unsubscribe/decline path is harder than the accept path, you have a dark pattern — regardless of intent.

### One-line classifier
> "If a user understands the offer, freely accepts it, and stands to gain — it's persuasion. If they're misled, pressured, or tricked — it's manipulation."

For the full catalog and remediation, see [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md).

### Regulatory exposure — what dark patterns cost in 2026
Regulators have moved from guidelines to enforcement. This is not theoretical.

| Regulation | Scope | Key dark-pattern rules | Penalties |
|---|---|---|---|
| **EU Digital Services Act (DSA)** | Platforms with EU users; fully enforced Aug 2024 | Bans "deceptive interfaces" designed to manipulate user decisions; very large platforms (>45M EU users) face audits | Up to 6% global annual turnover; repeat: up to 10% |
| **EU Digital Markets Act (DMA)** | "Gatekeeper" platforms | Must not prevent users from unsubscribing as easily as subscribing; ban on consent dark patterns | Up to 10% global turnover; 20% for repeat infringement |
| **GDPR / ePrivacy** | EU/EEA users | Consent must be freely given, specific, informed, unambiguous — pre-ticked consent boxes are invalid; asymmetric friction on cookie opt-out = enforcement target | Up to €20M or 4% global turnover |
| **FTC Endorsement Guides (2023 update)** | US, any advertiser | Paid/incentivized reviews must be disclosed; fake reviews prohibited; "clearly and conspicuously" standard | Civil penalties up to $51,744 per violation |
| **UK CMA (2024+)** | UK, post-Brexit | Subscription traps (hard-to-cancel), drip pricing, fake urgency → active enforcement priority | Unlimited fines under Enterprise Act 2002 |

**Practical rules:**
- Pre-ticked checkboxes for marketing consent → illegal under GDPR everywhere you have EU users.
- Hiding the unsubscribe path behind multiple steps → CMA/DSA enforcement target.
- "Only 2 left" or countdown timers that are not backed by real data → FTC and CMA false-advertising exposure.
- Fake reviews or undisclosed incentivized reviews → FTC violation in the US; DSA violation in EU.
- Cookie banners where "Accept All" is prominent and "Reject All" is buried or absent → GDPR enforcement (French CNIL, Irish DPC have issued large fines).

---

## Effective social-proof patterns

Social proof can lift conversions substantially. Displaying 5+ reviews lifted conversion ~270% vs. 0 reviews for lower-priced products (Spiegel Research Center / Northwestern University, 2017). 75% of consumers worry about fake reviews (BrightLocal); one suspicious review casts doubt on the whole section. Only authenticity signals preserve the lift.

### What to show, in order of trust
1. **Personal-network recommendation** (referrals, "your colleague uses this").
2. **Verified-buyer review with photo/video** — 67% say photos+ratings influenced a purchase.
3. **Star rating** — 57% only use businesses rated ≥4 stars; 3.5 is increasingly seen as inadequate.
4. **Aggregate count** ("50,000+ customers") — weakest; use as support, not the headline.

### Authenticity signals (non-negotiable)
- **Dates on every review.** 74% only trust reviews from the last 3 months; 83% consider reviews older than 3 months irrelevant. A dated wall of 2021 reviews signals neglect.
- **Verified-buyer badges** and **reviewer photos/videos**.
- **Show negative reviews too.** Surfacing negatives *before* positives can raise purchase intent ~8% — buyers want accountability over fake perfection.

### Anti-patterns
- Reviews with no dates, no photos, no verified badge → distrusted by the 75% worried about fakes.
- Fake "X people are viewing this right now" or fake countdown timers → recognized, screenshotted, trust destroyed.
- A stale review wall is worse than no reviews.

### Minimal trustworthy review card
```html
<article class="review" itemscope itemtype="https://schema.org/Review">
  <header>
    <img src="/u/anita.jpg" alt="" width="40" height="40" loading="lazy">
    <span itemprop="author">Anita R.</span>
    <span class="badge" aria-label="Verified buyer">✓ Verified buyer</span>
    <span itemprop="reviewRating" itemscope itemtype="https://schema.org/Rating">
      <span aria-label="4 of 5 stars"
            itemprop="ratingValue">4</span><span hidden itemprop="bestRating">5</span>★★★★☆
    </span>
    <time itemprop="datePublished" datetime="2026-05-02">May 2, 2026</time>
  </header>
  <p itemprop="reviewBody">Cut our onboarding from 3 days to 40 minutes…</p>
  <img class="ugc" src="/reviews/anita-photo.jpg" alt="Customer's dashboard" loading="lazy">
</article>
```
Use Schema.org `Review`/`AggregateRating` only for genuinely collected reviews (Google penalizes fake/self-serving review markup).

---

## Pricing-page persuasion (worked example)

Combines several principles ethically in one component:
- **Anchoring + Serial Position:** order tiers so the highest-value/anchor tier is prominent at the start or end; place the recommended tier where the eye lands.
- **Endowment:** "Everything in Free, plus…" on the paid tier.
- **Loss aversion:** annual toggle shows "Save $X/yr" (loss framing), not just "−20%".
- **Hick's/Miller's:** ≤3–4 tiers, each with ≤5–7 differentiating features.
- **Social proof:** one short verified testimonial near the primary CTA.
- **Authenticity:** real crossed-out price only; no invented "was" price.
- **No dark patterns:** no pre-checked add-ons, no fake "1 left," cancel as easy as subscribe.

Basecamp is the canonical reference for restraint: simplified pricing (one flat tier for years), easy cancellation, no manipulative upsell. 37signals is private; revenue figures are not public — cite the design decisions, not a revenue number you can't verify.

---

## Concrete specs & numbers

- **Loss aversion ratio:** 1.5–2.5×, most cited **2.0×** (Kahneman/Tversky).
- **Cart abandonment:** 70.22% avg (Baymard, 50-study aggregate); 71.72% in 2025 (Uptain).
- **Top abandonment reasons:** 39% surprise extra costs · 19% forced account creation · 18% checkout too long/complex.
- **Checkout fields:** avg **11.3**; target **6–8** (Baymard). Recoverable lost orders (US/EU) estimated in the hundreds of billions annually (Baymard — check current figure at baymard.com, as this updates yearly); up to **35.26%** conversion lift from better checkout UX (Baymard).
- **Reviews:** 95% read reviews before buying · 93% say reviews impact decisions · 79% trust online reviews like personal recommendations · 67% more purchases when reviews visible.
- **Review freshness:** 74% trust only last-3-month reviews · 83% consider >3-month reviews irrelevant.
- **Ratings threshold:** 57% only use businesses rated ≥4 stars.
- **Negative-first:** showing negatives before positives → +8% purchase intent.
- **Fake-review concern:** 75% of consumers.
- **Anchoring evidence:** Tversky & Kahneman 1974 (512 vs. 2,250 medians from arbitrary wheel-spin anchor); Ariely, Loewenstein & Prelec 2003 — WTP correlated with SSN digits across product categories (specific dollar amounts varied by condition; see original QJE paper).
- **Defaults evidence:** Johnson & Goldstein 2003 (*Science*) — organ-donor opt-in rates 40–60 percentage points higher in opt-out vs. opt-in countries; UX friction effects directionally consistent but smaller in magnitude.
- **Dark-pattern prevalence:** ~10% of 11,000 ecommerce sites (Mathur et al., WWW '19, Princeton/U Chicago); 95% of 240 Google Play apps contained ≥1 dark pattern, avg 7 patterns each (Bösch et al., U Zurich 2019).
- **Doherty Threshold:** UI response **<400ms**.
- **Touch targets:** ≥**48×48dp** (Material Design 3) · ≥**44×44pt** (Apple HIG) · ≥**24×24 CSS px** (WCAG 2.5.8).
- **Working memory:** **7 ± 2** (Miller); chunk to ≤5.
- **Benchmarks:** B2B avg conversion **2.23%**, top 10% reach **11.7%**. Design-led companies outperformed the S&P 500 by **211%** over a 5-year period (McKinsey Design Index, 2018); the specific number is from McKinsey, not Adobe.

---

## Tools & libraries

- **A/B & experimentation:** VWO, Optimizely, GrowthBook (OSS), PostHog (OSS, also analytics + feature flags). Use to *validate* every persuasion change — never assume a tactic lifts conversion; many backfire.
- **Reviews / social proof:** Yotpo, Trustpilot, Judge.me (Shopify), Google reviews. Require verified-buyer status, photos, and visible dates.
- **Analytics / funnel:** PostHog, Amplitude, GA4 — diagnose the Fogg sequence (where prompts fire, where ability/friction kills the funnel).
- **Session replay:** Hotjar, FullStory, PostHog replays — observe real friction before adding motivation.
- **Schema:** Schema.org `Review`/`AggregateRating` for *genuine* reviews only.
- **When NOT to use:** avoid third-party "urgency/scarcity popup" widgets (fake "X viewing now," fake timers) and confirmshaming popup tools — they're dark-pattern factories that trigger backlash and legal exposure.

---

## Do / Don't

**Do**
- Diagnose conversion in Fogg order: Prompt → Ability → Motivation.
- Reduce friction (fields, steps, surprise costs) before adding any persuasion.
- Give real value first (reciprocity); make the gift specific and immediate.
- Use truthful scarcity tied to actual backend state.
- Show review dates, verified badges, and photos; surface some negatives.
- Frame defaults with rationale + drawbacks; make opt-out 1 click.
- Use "Everything in Free, plus…" framing (endowment) on paid tiers.
- Use a decoy pricing tier honestly — all tiers real, side-by-side differentiators scannable.
- Sequence principles: Reciprocity → Authority/Liking → Unity → Social Proof → Commitment → Scarcity.
- Give users a neutral exit from any prompt ("Not right now") to prevent reactance.
- Show total step count in multi-step flows; let users exit cleanly.
- Verify regulatory compliance: no pre-ticked consent, cancel ≤ subscribe difficulty, disclosed incentivized reviews.
- Limit to 1–2 dominant persuasion signals per section.
- A/B test every claim; let data, not theory, decide.

**Don't**
- Stack scarcity timer + popup proof + authority badges + urgency copy on one screen.
- Invent crossed-out "original" prices or fake "X left / X viewing."
- Use confirmshaming ("No thanks, I don't like saving money").
- Force account creation before checkout.
- Pre-check paid add-ons users won't notice.
- Make cancel/unsubscribe harder than subscribe (asymmetric friction).
- Apply loss framing to low-stakes asks (newsletter "lose your spot").
- Show a wall of stale (>3-month) reviews.
- Trigger multiple prompts in sequence before the user has acted on the first (reactance).
- Use high cognitive load (jargon walls, overwhelming feature lists) as a persuasion substitute — it impairs evaluation, not just friction.
- Hide step count in multi-step flows to manufacture sunk cost.
- Pre-tick marketing consent checkboxes (GDPR violation in any EU-user context).

---

## Common pitfalls & how to avoid them

- **Blaming motivation first.** The real problem is usually friction (ability) or a missing/mistimed prompt. → Run the Fogg sequence; fix prompt and ability before touching motivation.
- **Prompt overuse.** Every new modal/notification dilutes the rest. → Audit and cut prompts before adding; trigger on engagement, not schedule.
- **Dishonest scarcity.** Fake timers/viewer counts are increasingly recognized and destroy trust faster than they build urgency. → Only show scarcity backed by real data.
- **Stale reviews.** A 2021 4-star wall signals neglect to 74–83% of buyers. → Sort by recency, show dates, prune dead reviews.
- **Confirmshaming.** Gets screenshotted and shared negatively. → Use neutral decline copy ("No thanks").
- **Implausible anchors.** Obviously inflated "was" prices read as fake. → Use authentic original pricing.
- **Forced account creation.** 19% abandon. → Offer guest checkout / "build first, sign up at download."
- **Sneaky defaults.** Unnoticed add-ons → chargebacks + negative reviews. → Disclose clearly; never pre-check paid items.
- **Tiny touch targets.** CTAs below spec cause mis-taps and abandonment. → ≥48×48dp / 44×44pt / 24px WCAG.
- **Social proof without authenticity signals.** Undated, photoless, unverified reviews distrusted by 75%. → Add dates, badges, photos.
- **Hollow Unity rhetoric.** "Join our family" with no community → recognized as marketing speak. → Provide real shared-identity content or drop the framing.
- **Loss aversion in low-stakes flows.** Feels manipulative. → Reserve for high-value, time-sensitive decisions.
- **Signal overload.** Too many persuasion cues at once cause anxiety. → 1–2 dominant signals per section.
- **Misapplied Hick's Law.** Cutting 5→2 options without a clearly right choice doesn't help. → Fix value-prop clarity, not just count.
- **Ignoring Peak-End.** A functional-but-flat confirmation/error screen wastes the moment users remember most. → Make peaks and endings outstanding.
- **Triggering reactance.** Stacking aggressive tactics past user tolerance causes the opposite of the intended action. → If users dismiss CTAs without reading them, diagnose reactance via session replays before adding more motivation.
- **Using cognitive complexity to sell.** Feature-wall pages that overwhelm working memory can lift short-term conversion but impair critical evaluation — a dark pattern. → Use chunking and plain language to make evaluation easier.
- **Hidden step count in multi-step flows.** Users feel trapped; when they realize how long the form is, they abandon with resentment. → Always show "Step X of Y."
- **Regulatory exposure from dark patterns.** Pre-ticked consent, hard-to-cancel subscriptions, fake urgency copy → active enforcement under DSA (6% global revenue), FTC (per-violation civil penalties), CMA. → Audit against the regulatory table above before shipping.

---

## What real users complain about

- **"Where's the cancel button?"** — multi-step or hidden cancellation (asymmetric friction) is the most-cited and most-screenshotted complaint; it generates lasting brand backlash.
- **"Surprise fees at the last step."** — shipping/taxes/handling appearing only at the final checkout screen; the #1 abandonment driver (39%).
- **"I had to create an account just to buy."** — forced registration; 19% leave.
- **"These reviews are all from years ago."** — stale review walls read as neglect/fakery.
- **"That countdown reset when I refreshed."** — fake urgency timers are detected and shared as evidence of dishonesty.
- **"'No thanks, I hate saving money' — really?"** — confirmshaming microcopy reliably triggers ridicule.
- **"They pre-checked an add-on I never wanted."** — sneaky defaults lead to chargebacks and 1-star reviews.
- **"Too many popups/banners shouting at once."** — signal overload feels like pressure, not help.

See the broader synthesis in [Real User Complaints](../08-pitfalls-and-antipatterns/real-user-complaints.md).

---

## Senior-level checklist

- [ ] Every persuasion element passes the **5-question Ethics Test** (could explain it face-to-face).
- [ ] Conversion failure diagnosed **Prompt → Ability → Motivation**, with friction reductions shipped before motivation tactics.
- [ ] Checkout: guest option available; surprise costs surfaced early; fields trimmed toward 6–8 (from 11.3 avg).
- [ ] Scarcity claims are backed by **real backend state**; no fake timers/viewer counts.
- [ ] Reviews show **dates, verified badges, photos**; sorted by recency; some negatives surfaced.
- [ ] Anchors use **authentic** original prices; crossed-out + discounted shown together.
- [ ] Defaults disclose rationale + drawbacks; **opt-out is as easy as opt-in (1 click)**.
- [ ] Paid tiers framed **"Everything in Free, plus…"** (endowment), not "lose the free tier."
- [ ] No **asymmetric friction**: cancel/unsubscribe/decline as easy as the accept path.
- [ ] **≤1–2 dominant persuasion signals** per page section; no stacking; sequenced Reciprocity → Authority → Proof → Commitment → Scarcity.
- [ ] Loss framing only on **high-stakes, time-sensitive** decisions.
- [ ] Touch targets ≥**48×48dp / 44×44pt / 24px WCAG**; UI responds **<400ms**.
- [ ] Pricing/feature lists chunked to **≤5–7** items; most important tier/feature at **start and end**.
- [ ] Pricing page uses **decoy tier** honestly — all tiers real, differentiating features scannable side-by-side.
- [ ] Success/confirmation and error-recovery screens designed as **peaks** (Peak-End Rule).
- [ ] Multi-step onboarding shows **total step count**; users can exit cleanly (no hidden sunk-cost trap).
- [ ] No reactance triggers: no confirmshaming, no forced paths, no dismiss-hostile modals; include "Not now" in neutral language.
- [ ] **Regulatory compliance verified:** no pre-ticked marketing consent (GDPR), cancel path ≤ subscribe path (DSA/CMA), review incentives disclosed (FTC), fake urgency copy removed.
- [ ] Every persuasion claim is **A/B tested**, not assumed.
- [ ] No confirmshaming, no sneaky defaults, no fake social proof anywhere in the flow.

---

## Sources

- Robert Cialdini, *Influence* (1984, rev. 2021) and *Pre-Suasion* (2016, introduces Unity principle).
- BJ Fogg, *Tiny Habits* (2019); Behavior Model — https://behaviormodel.org
- Tversky & Kahneman (1974), "Judgment under Uncertainty: Heuristics and Biases," *Science* 185(4157). Kahneman & Tversky (1979), "Prospect Theory," *Econometrica*.
- Ariely, Loewenstein & Prelec (2003), "Coherent Arbitrariness," *Quarterly Journal of Economics* 118(1).
- Johnson & Goldstein (2003), "Do Defaults Save Lives?", *Science* 302(5649) — organ-donor defaults.
- Spiegel Research Center / Northwestern University (2017), "How Online Reviews Influence Sales."
- BrightLocal Local Consumer Review Survey — https://www.brightlocal.com/research/local-consumer-review-survey/
- Baymard Institute — checkout & cart-abandonment research — https://baymard.com/lists/cart-abandonment-rate
- Mathur et al. (2019), "Dark Patterns at Scale," *WWW '19*, Princeton/U Chicago.
- Bösch et al. (2019), "Tales from the Dark Side: Privacy Dark Strategies and Privacy Dark Patterns," *PoPETs* / U Zurich.
- McKinsey Design Index (2018), "The Business Value of Design."
- EU Digital Services Act — https://digital-strategy.ec.europa.eu/en/policies/digital-services-act-package
- EU Digital Markets Act — https://commission.europa.eu/strategy-and-policy/priorities-2019-2024/europe-fit-digital-age/digital-markets-act-ensuring-fair-and-contestable-digital-markets_en
- FTC Endorsement Guides (revised 2023) — https://www.ftc.gov/legal-library/browse/rules/guides-concerning-use-endorsements-testimonials-advertising
- Laws of UX (Miller, Hick, Fitts, Doherty, Jakob, Serial Position, Peak-End, Zeigarnik) — https://lawsofux.com
- Nielsen Norman Group — dark patterns / deceptive design — https://www.nngroup.com
- Material Design 3 (touch targets) — https://m3.material.io
- Apple Human Interface Guidelines — https://developer.apple.com/design/human-interface-guidelines
- WCAG 2.2, Success Criterion 2.5.8 Target Size (Minimum) — https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html

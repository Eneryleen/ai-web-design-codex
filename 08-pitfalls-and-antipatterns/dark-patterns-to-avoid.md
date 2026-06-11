# Dark Patterns to Avoid

> Catalogs the deceptive design patterns regulators now treat as illegal — roach motel, forced continuity, confirmshaming, sneak-into-basket, hidden costs, fake urgency, privacy zuckering, and hard-to-cancel flows — with the GDPR/FTC/DSA rules that govern them. Outcome: you can build flows that convert through clarity and honest defaults instead of manipulation, avoiding fines, churn, and the trust collapse that follows enforcement.

**Apply when:** designing pricing, checkout, signup, consent, cancellation, or any conversion-critical flow · **Related:** [Ethical Persuasion Psychology](../06-marketing-and-conversion/persuasion-psychology.md) · [Conversion Rate Optimization (CRO)](../06-marketing-and-conversion/conversion-rate-optimization.md) · [E-commerce UX](../03-site-types/ecommerce-ux.md) · [Common Design Mistakes & Antipatterns](./common-mistakes-antipatterns.md)

## TL;DR — rules at a glance
- A dark pattern = **Deception** (user doesn't grasp what they're doing) + **Asymmetry** (the path the user wants is harder than the path you want) + **Cognitive Exploitation** (loss aversion, urgency, sunk cost). If a flow has any of the three, fix it.
- **SYMMETRY RULE (legal, not stylistic):** cancel/opt-out must be as visible, accessible, and low-effort as sign-up/opt-in. EDPB: *"If consent is easier than refusal, consent is invalid."*
- **Consent obtained via dark patterns is legally void** under GDPR, CPRA, and the EU DSA. You don't get the data; you get the fine.
- **Show total cost early.** 48% of US cart abandonment is caused by hidden costs revealed at checkout. Put shipping/tax/fees on the product page or first price display.
- **Pre-checked boxes for non-essential data/marketing = GDPR violation.** Default every non-essential checkbox to OFF (opt-in).
- **Click-to-Cancel parity (FTC):** if signup is 1 click online, cancel must be 1 click online through the same channel. No phone-only cancellation for online subscriptions.
- **Decline copy must be neutral** ("No thanks" / "Skip"), never guilt-inducing ("No, I like missing out"). Confirmshaming is increasingly illegal.
- **Fake urgency/scarcity = deceptive trade practice.** No resetting timers, no fabricated "12 people viewing." Of 393 sites with countdown timers, 36% were fake (Princeton).
- **Never force account creation before purchase.** Guest checkout is the default; "Save your info?" *after* purchase is the ethical alternative — and lifts conversion.
- **Both buttons visible.** A bold colored "Accept" next to tiny grey "No thanks" is visual-interference manipulation, even if the link technically works.
- **Label ads at the content's own visual weight.** Native ads mimicking editorial without disclosure = disguised advertising.
- Fines are existential: GDPR up to 4% global revenue / €20M; DSA up to 6% global turnover; FTC v. Amazon Prime consent decree $25M (2023) with required UX redesign; EU DMA/DSA enforcement escalating.
- Ethical UX wins long-term: transparent pricing + guest checkout cut abandonment up to 30%; free-shipping disclosure makes buyers 88% more likely to complete.

## What a dark pattern actually is

A dark pattern (a.k.a. "deceptive pattern") is a UI crafted to trick users into doing things they didn't mean to. The term was coined in 2010 by UK UX specialist Harry Brignull; the authoritative taxonomy lives at **deceptive.design** (formerly darkpatterns.org).

**The 3-trait test** — if a design has any one of these, it is suspect:

| Trait | Question to ask | Example |
|---|---|---|
| **Deception** | Does the user understand what they're agreeing to? | "Continue" button that also signs you up for a paid trial |
| **Asymmetry** | Is the company-preferred action far easier than the user-preferred one? | One-click "Accept All", five-step "Reject All" |
| **Cognitive Exploitation** | Does it weaponize a known bias (loss aversion, urgency, sunk cost)? | "Only 1 left!" + a countdown timer that resets on reload |

**Why "it converts" is not a defense.** Short-term lift from manipulation erodes trust, raises churn, and triggers enforcement. StubHub's drip pricing raised revenue ~20% and now faces regulatory scrutiny; FTC's Amazon Prime consent decree required $25M in refunds plus mandatory UX redesign. The metric you optimized becomes the evidence against you.

## The pattern catalog

Each entry: *what it is → why it's harmful/illegal → the ethical alternative that still converts.*

### Roach Motel (easy in, hard out)
- **What:** Trivial to sign up; deliberately difficult to leave. Multi-page cancellation, hidden cancel buttons, mandatory phone calls.
- **Harm:** Violates FTC Click-to-Cancel and EU symmetry-of-choice. Drives furious chargebacks and 1-star reviews.
- **Real cases:** Amazon Prime (FTC suit + consent decree Dec 2023 → **$25M** in refunds + mandatory UX redesign; multi-page cancellation with nag screens labeled "iliad flow" internally). Financial Times (a "Confirm change" button that *upgrades to yearly* instead of canceling; the real "Confirm cancel" sits small at the bottom). Microsoft 365 (showed a cheaper $69.99 plan only mid-cancellation).
- **Ethical alternative:** Cancel in the same number of steps and same channel as signup. One screen, one clear "Cancel subscription" button. Offer a pause/downgrade option *after* you accept the cancel intent, not as a wall in front of it.

### Forced Continuity (silent auto-renewal)
- **What:** Free/cheap trial silently converts to recurring paid charges; renewals happen with no plain-language warning.
- **Harm:** FTC treats undisclosed auto-renewal as a deceptive trade practice. EU consumer enforcement agencies document subscription-trap complaints as among the highest-volume consumer harm categories. Silent billing also triggers chargeback escalation — processors may terminate your merchant account before any regulator acts.
- **Real case:** Interaction Design Foundation — yearly-only auto-renew, a "Get ready for another great year!" email instead of an explicit renewal notice, access removed immediately on cancel (instead of running out the paid period), payment method re-added on any new purchase.
- **TRIAL REMINDER RULE:** Email **7 days before** the trial/term ends with the renewal price *in the subject line*: `Your subscription renews for $99 on Jun 18` — not "Get ready for another great year." Let users keep paid access until the period they paid for actually ends.

### Confirmshaming
- **What:** Guilt-loaded decline copy to shame users out of the safe choice.
- **Harm:** Manipulative; explicitly cited in EDPB guidance and increasingly enforced.
- **Don't:** `[ Subscribe ]   No thanks, I prefer paying full price` / `No, I like missing out`.
- **Do (COPY NEUTRALITY RULE):** decline = `No thanks` or `Skip`, in the same visual weight as accept. Neutral copy still converts; it just doesn't manufacture regret.

### Sneak into Basket
- **What:** Items/fees the user never selected appear in the cart — insurance, donations, expedited shipping, tips.
- **Harm:** Basket sneaking on ticketing platforms is among the top user complaints to consumer bodies (UK CMA, EU regulators). Pre-selected add-ons are illegal under EU Consumer Rights Directive Art. 22: consumers must actively select optional extras — pre-ticked defaults are prohibited and must be refunded on request. A 20% default tip on Indiegogo was flagged as such.
- **Real cases:** RyanAir (pre-checked travel insurance, opt-out seat/priority add-ons). IRCTC Indian Railways (auto-added travel insurance via pre-selected checkbox, charging millions unknowingly).
- **Ethical alternative:** Every add-on is opt-IN, unchecked by default, with the price shown next to it.

### Hidden Costs / Drip Pricing
- **What:** Fees (shipping, tax, service, "facility," "resort") appear only at the final checkout step.
- **Harm:** The **single most common** dark pattern AND the leading cart-abandonment cause: **48% of US shoppers abandon specifically due to unexpected costs at checkout** (Baymard). Exploits sunk-cost fallacy — you've invested effort, so you tolerate the surprise.
- **Real cases:** Ticketmaster (fees add 25–50% revealed only at the end). Xfinity ($70/mo advertised → $80/mo at checkout, $10 off only via autopay + bank account). Resort hotels (documented 40% final increase over advertised rate).
- **TOTAL COST RULE:** Show all mandatory fees on the product page or at the *first* price display. If a fee is unavoidable, it is part of the price — full stop.

### Fake Urgency / Scarcity
- **What:** Countdown timers, "Only 1 left!", "12 people viewing" — fabricated or recycled.
- **Harm:** Deceptive trade practice. Princeton found **36% of countdown timers on 393 sites were fake.**
- **Real case:** Booking.com — "Only 1 room left at this price!" and "15 people are currently looking" shown seconds apart, often aggregated from long periods or fabricated.
- **URGENCY HONESTY RULE:** Show real stock numbers and real countdowns only when genuinely true. Never reset a timer. Never invent viewer counts. *Honest* scarcity ("3 left in your size — restock in ~2 weeks") is fine and trustworthy.

### Trick Questions
- **What:** Confusing wording, double negatives, mismatched checkbox polarity so "yes" and "no" controls behave inconsistently.
- **Harm:** Defeats informed consent; invalidates it under GDPR Art. 7 (consent must be unambiguous). UK ICO and EU DPAs routinely cite ambiguous checkbox language as grounds for enforcement.
- **Real case:** UK CMA's 2022 hotel-sector review found multiple booking platforms using inconsistent opt-in/opt-out wording across the same form, contributing to enforcement notices.
- **Don't:** `☐ Uncheck this box if you do not wish to not receive emails`.
- **Do:** One direction of meaning, plain language: `☐ Email me product updates` (unchecked). All checkboxes in a form must use the same polarity — never mix opt-in with opt-out on the same page.

### Privacy Zuckering
- **What:** Tricking users into sharing more data than intended — prominent "Accept All," buried "Reject," friction on data-rights requests, auto-enrollment in data uses.
- **Harm / real cases:**
  - **Honda** — California CPPA fined **$632,500 (2025)**: single bright "Accept All" vs. multi-step "Reject All," excessive friction on data-rights requests, extra verification for authorized agents. Forced UX redesign + 5-year transparency reporting.
  - **WhatsApp/Meta** — **€225M** GDPR fine for non-transparent data sharing.
  - **Google + Facebook (France, 2022)** — both fined for cookie banners where rejecting was harder than accepting.
  - **LinkedIn** — auto-enrolled users into AI training on their content (2024); changed default to opt-in only after public backlash + regulator attention.
- **DEFAULT PRIVACY RULE:** All data-sharing, marketing, and tracking defaults to **OFF**. Reject and Accept buttons are equally prominent, side by side, one click each.

### Nagging
- **What:** Repeated interruptions pressuring an action the user already declined — re-prompts for notifications, account upgrade, app install, newsletter pop-up, cookie banner.
- **Harm:** DSA Art. 25 explicitly bans "repeating a request that has already been refused" as a manipulative interface pattern. The Amazon Prime consent decree cited repeated "benefit reminder" screens injected into the cancellation path as a named violation. iOS (since iOS 12) and Android (since API 33) enforce one-prompt-then-done for system notification permissions — re-prompting after the OS "Never" is blocked at the platform level. Web-push re-prompting also blocked unless the user explicitly re-opens a preferences surface.
- **NOTIFICATION PERSISTENCE RULE:** If the user dismisses a permission/notification prompt, don't re-show it that session. Cooldown **≥30 days** before re-prompting. Never re-enable something the user disabled (the "Kayak" anti-pattern: Kayak was documented re-enabling price-alert emails after users unchecked them on any new booking).
- **Anti-pattern to name in design reviews:** "Save before you go" exit-intent pop-ups that re-trigger on every page if the user has ever dismissed one.

### Obstruction
- **What:** Adding artificial steps to discourage an action — endless cancellation funnels, requiring a phone call or written letter for an online-signup cancellation, intentionally tedious data-export (e.g., 30-day wait for GDPR data access when the data is already available).
- **Harm:** Asymmetry trait by definition; FTC Click-to-Cancel Final Rule and EU GDPR Art. 12 (data-subject rights must be fulfilled "without undue delay" and with "no excessive friction"). GDPR Art. 12(5) explicitly prohibits charging a fee or obstructing data-rights requests. Regulators have fined companies for making opt-outs require ID verification that acceptance did not require — the Honda CPPA fine is the canonical example.
- **Real cases:** Meta (Irish DPC found its data-portability flow required steps that sign-in did not, contributing to overall GDPR enforcement). Honda (CPPA 2025 — required name, address, phone to opt out of data sale; one click to opt in).
- **Ethical alternative:** Every action the user has a right to take must be at most as many steps as the equivalent company-preferred action. GDPR DSAR response ≤30 days, extendable to 90 with notice — not 30 days just to generate a download link.

### Disguised Ads
- **What:** Ads styled as content, navigation, or download buttons. Native ads mimicking editorial without disclosure. Influencer posts without #ad or #sponsored.
- **Harm:** FTC native-advertising guidance (2015) + updated Endorsement Guides (2023) require clear, conspicuous labeling at the point of first contact. "Sponsored" buried in a footer or in 8px grey type fails the conspicuous test. The 2023 Guides explicitly extend to AI-generated endorsements, virtual influencers, and social-media platforms.
- **Real cases:** FTC actions against Lord & Taylor (2016) and Warner Bros. (2016) for undisclosed paid influencer placements; 2023 Guides tightened disclosure for all paid relationships including micro-influencers. UK ASA routinely upholds complaints against "Ad" labels that are smaller or lower-contrast than the surrounding content.
- **AD LABELING RULE:** "Sponsored" / "Ad" at the same font size and contrast as the content label, placed *before* the content begins — not after. A fake "Download" button next to a real one is obstruction + disguise (also a separate CNET/CNET-Media documented pattern).

### Forced Account Creation
- **What:** Requiring signup before allowing purchase.
- **Harm:** A documented dark pattern; suppresses conversion. **Guest checkout availability reduces abandonment by up to 30% and lifts conversion 10%+.**
- **NO FORCED ACCOUNT RULE:** Guest checkout is the default offer. After purchase, ask "Save your info for next time?" — same data, user's choice.

### Bait and Switch
- **What:** User intends one action; a different (usually more profitable for the vendor) outcome is delivered instead. Classic: an ad for a discounted product leads to a page where that product is "out of stock" but a pricier alternative is prominent. Modern: a "Close" or "X" button performs a different action than closing.
- **Harm:** FTC considers bait-and-switch advertising a deceptive trade practice (16 CFR Part 238). EU Unfair Commercial Practices Directive Art. 6 prohibits it as a misleading action. "Close" buttons that redirect rather than close are also an obstruction variant.
- **Real cases:** Windows update dialogs (2016) where the "X" close button was briefly mapped to "Confirm upgrade." Download sites where "Download" buttons are ads for unrelated software installers.
- **Ethical alternative:** Every interactive element does what its label says. If a product is out of stock, say so — recommend the alternative as a genuine suggestion, not a forced redirect.

### Cognitive Exploitation of Children / Vulnerable Users
- **What:** Confusing virtual-currency conversions, pressure mechanics aimed at minors.
- **Harm:** Roblox — children describe Robux conversions as "scary"; a documented case of a child spending $4,000 without consent. California's **CAADCA** specifically bans dark patterns targeting under-18s.
- **Ethical alternative:** Show real-money equivalents at the point of purchase; hard spend caps; explicit parental gates.

## Regulatory landscape (know the teeth)

| Regime | What it bans / requires | Max penalty |
|---|---|---|
| **GDPR (EU)** | Valid consent must be freely given, specific, informed, unambiguous; pre-ticked boxes invalid; symmetry of choice | **4% global annual revenue or €20M**, whichever higher |
| **EDPB Guidelines 03/2022** | Defines dark patterns in social-media consent flows (scoped to social platforms but cited as general benchmark across sectors); *"If consent is easier than refusal, consent is invalid."* | (Interpretive — enforced via GDPR) |
| **EU Digital Services Act (DSA)** | Prohibits manipulative interfaces on all digital services; VLOPs under enforcement since Aug 2023; all other platforms since Feb 2024 | **6% of global turnover** (VLOPs); 1% for others |
| **FTC Act §5 (US)** | "Bringing Dark Patterns to Light" (2022) classifies manipulative design as deceptive/unfair trade practice; Click-to-Cancel Final Rule (Oct 2024, eff. 2025): online cancel must match signup channel and step count | Civil penalties + refunds; FTC v. Amazon Prime consent decree $25M (2023) |
| **CCPA / CPRA (California)** | Consent via dark patterns is invalid; symmetry of choice required | Honda fined **$632,500 (2025)** |
| **UK ICO** | GDPR-equivalent; cookie-banner symmetry | **£17.5M or 4% worldwide turnover** |
| **CAADCA (California)** | Bans dark patterns targeting under-18 users | Per California AG |
| **EU Consumer Rights Directive (2011/83/EU) Art. 22** | Prohibits pre-ticked boxes for optional extras; any charge not explicitly selected by consumer must be refunded | Per national consumer enforcement |
| **India Dept. of Consumer Affairs** | Guidelines define **13 prohibited** dark-pattern practices (Nov 2023) | Per Indian consumer law |
| **EU Digital Fairness Act (proposed Oct 2024)** | Would consolidate dark-pattern prohibitions across consumer law; explicitly bans hidden auto-renewal, sneak-into-basket, fake urgency, nagging, and consent-management dark patterns beyond social media | (Proposed — not yet in force as of 2026) |

**Core legal principle to encode in every consent flow:** consent gathered through asymmetry or deception is *not consent*. You gain no legal basis for the data — only liability. Build the symmetric flow and the consent is actually valid.

## Code patterns: symmetric & honest UI

These are minimal correct implementations — copy the structure, not just the spirit.

**Symmetric consent banner.** Reject and Accept are the *same* button class, same size, same prominence, both one click. Settings is a tertiary text link.

```html
<div class="consent" role="dialog" aria-label="Cookie preferences">
  <p>We use cookies for analytics. You can reject non-essential cookies.</p>
  <div class="consent__actions">
    <!-- Same class = same visual weight. This is the legal requirement. -->
    <button class="btn btn--equal" data-consent="reject">Reject all</button>
    <button class="btn btn--equal" data-consent="accept">Accept all</button>
  </div>
  <a class="consent__link" href="/cookie-settings">Manage preferences</a>
</div>
```

```css
/* Both choices are identical in size/contrast. NO ghost-button on reject. */
.btn--equal {
  font: inherit; padding: 12px 20px; min-height: 44px; /* touch target */
  border: 1px solid currentColor; border-radius: 8px;
  background: var(--surface); color: var(--text);
}
/* Anti-pattern to AVOID (visual interference): */
/* .btn--reject { background: transparent; color: #bbb; font-size: 11px; } */
```

**Default-OFF data toggles.** Render server-side state; never ship `checked` on non-essential toggles.

```html
<label><input type="checkbox" name="analytics">        Allow analytics</label>     <!-- unchecked -->
<label><input type="checkbox" name="marketing">        Email me offers</label>      <!-- unchecked -->
<label><input type="checkbox" name="essential" checked disabled> Essential (required)</label>
```

**Honest total price** — fees resolved before the user commits, not at the final step.

```js
// Show the all-in price at the FIRST price display, not at checkout step 5.
const total = base + shipping + tax + serviceFee;        // all mandatory fees
priceEl.textContent = fmt(total);
breakdownEl.innerHTML = `Item ${fmt(base)} · Shipping ${fmt(shipping)} ·
                         Tax ${fmt(tax)} · Service ${fmt(serviceFee)}`;
// If a mandatory fee is unknowable until checkout, it must NOT exist as mandatory.
```

**Honest scarcity** — bind the message to a real source of truth; never a random number.

```js
// OK: real inventory. Bad: const viewers = 5 + Math.floor(Math.random()*20);
if (stock.real <= 5) badge.textContent = `${stock.real} left in stock`;
// Timer: persists across reload; expiry is a real server timestamp, never reset.
const remaining = sale.endsAtMs - Date.now();   // if <= 0, the sale is actually over
```

**Cancellation parity** — self-serve, same channel, mirror of signup.

```html
<!-- Signup was 1 click → cancel is 1 click, same place, no phone wall. -->
<form action="/subscription/cancel" method="post">
  <button type="submit">Cancel subscription</button>
</form>
<!-- Retention offer (pause/downgrade) may appear AFTER intent is accepted, not before. -->
```

## Concrete specs & numbers
- **Trial reminder:** send **7 days** before renewal; price in the subject line.
- **Notification re-prompt cooldown:** **≥30 days**; never within the same session.
- **Consent buttons:** Reject and Accept equal in size, color contrast, and step count — both **1 click**.
- **Checkout reality (Baymard 2024):** **70.19%** overall cart abandonment (80.25% on Cyber Monday); **48%** abandon on hidden costs; **22%** on checkout complexity; average flow = **5.1 steps, 11.3 form elements**.
- **Upside of honesty:** fixing documented checkout issues can lift conversion up to **35.26%** (avg site has **39** improvement areas); free-shipping disclosure → buyers **88%** more likely to complete, abandonment down up to **18%**; guest checkout → abandonment down up to **30%**, conversion up **10%+**.
- **Prevalence (FTC/ICPEN/GPEN sweep 2024):** **75.7%** of 642 company sites/apps used ≥1 dark pattern; **66.8%** used ≥2. **95%** of popular Android apps; **97%** of popular EU consumer apps (2025 sweep). **56%** of 10,000 EU sites used pre-ticked cookie boxes; **72%** hid reject behind multiple menus.
- **Drip pricing:** inflates final paid amount ~**20%** vs. advertised; resort fees documented at **40%** over advertised rate.
- **Detection limits:** current tooling identifies only **45.5%** of known dark-pattern types (8 tools vs. a 68-type taxonomy, arXiv:2412.09147) — you cannot rely on an automated scanner to clear you; manual review is mandatory.

## Tools & libraries
- **deceptive.design** (Harry Brignull) — authoritative taxonomy of 15+ types + a "Hall of Shame." Use as your reference checklist. (darkpatterns.org redirects here.)
- **FTC "Bringing Dark Patterns to Light" (2022)** — the US regulatory framework; free PDF. Read before designing US-facing consent/subscription flows.
- **EDPB Guidelines 03/2022** — EU definitions and prohibited consent patterns. The binding reference for EU consent UI.
- **Baymard Institute** — 200,000+ hours of checkout/UX research, 650+ guidelines, benchmarks across 326 e-commerce sites. Use for checkout/pricing decisions.
- **CookieEnforcer** (UW-Madison + Google research) — ML extension that auto-rejects optional cookies; useful as a *test* of whether your banner is honestly rejectable.
- **CPPA enforcement guidance** — California's symmetry-of-choice rules for consent.
- **When NOT to lean on tools:** automated dark-pattern detectors cover <half the taxonomy. They're a smoke test, not a compliance sign-off. Note: the Interaction Design Foundation publishes good UX courses but its own platform is community-flagged for dark patterns — judge content, not the vendor's practices.

## Do / Don't

**Do**
- Show total price (incl. all mandatory fees) at the first price display.
- Offer guest checkout by default; ask to save info *after* purchase.
- Default every non-essential checkbox to unchecked (opt-in).
- Make Reject as prominent and as few clicks as Accept.
- Use neutral decline copy ("No thanks", "Skip").
- Email trial reminders 7 days out with price in the subject.
- Let users keep paid access until the period they paid for ends.
- Show real stock/timers only when genuinely true.
- Label ads at the content's own visual weight.

**Don't**
- Reveal fees only at the final checkout step.
- Pre-check insurance, donation, tip, marketing, or "expedited" boxes.
- Use guilt copy on decline buttons.
- Bury cancel behind phone calls or multi-page funnels.
- Reset countdown timers or fabricate viewer/stock counts.
- Re-prompt for a permission the user just dismissed.
- Style decline as tiny grey text next to a bold colored Accept.
- Re-enable settings or re-add payment methods the user removed.
- Force account creation before purchase.
- Map "X" / "Close" / "Back" to any action other than what the label implies.
- Link to a different product when the advertised one is "unavailable" without explicit labeling.

## Common pitfalls & how to avoid them
- **"Visual interference" sneaks in via design tokens.** A correct decline *link* rendered as 11px grey while Accept is a filled primary button is still a dark pattern. **Fix:** give both choices buttons from the same hierarchy tier (see [Visual Hierarchy](../00-foundations/visual-hierarchy.md)); contrast and size must be comparable.
- **Marketing asks for "just one pre-checked box."** It's a GDPR violation and invalidates *all* consent in that flow. **Fix:** opt-in only; track that the lift from honesty (88% on free-shipping disclosure) usually beats the trick.
- **Engineering ships "cancel = email support."** That's roach motel under Click-to-Cancel. **Fix:** self-serve cancel in the same channel and step count as signup.
- **A/B tests reward the manipulative variant.** Short-term CTR rises while churn and chargebacks (lagging metrics) rise later. **Fix:** measure retention, refund rate, and support tickets — not just immediate conversion. See [CRO](../06-marketing-and-conversion/conversion-rate-optimization.md).
- **Copywriters reach for urgency.** Fake timers are a deceptive trade practice. **Fix:** use honest scarcity or none; see [Ethical Persuasion Psychology](../06-marketing-and-conversion/persuasion-psychology.md).
- **Cookie banner from a CMP defaults to dark.** Many consent-management platforms ship asymmetric defaults. **Fix:** configure equal Accept/Reject, no pre-ticked non-essential categories.
- **"Save 20% — auto-renews" disclosed in 9px footnote.** Insufficient. **Fix:** renewal terms at the same weight as the price, near the CTA.
- **Donation/tip default set "to be nice."** A 20% default tip is illegal under EU consumer law (CRD Art. 22). **Fix:** default 0 / unselected.
- **Engineering maps "X" to a non-close action.** Seen in modal dialogs, push-notification prompts, and update nags. Even if unintentional, it's bait-and-switch. **Fix:** every close/dismiss control must close/dismiss — never confirm, upgrade, or redirect.
- **Checkout "back" button resets cart or loses promo.** Creates artificial loss aversion that traps users. **Fix:** cart state + applied discounts must survive navigation — no exceptions.

## What real users complain about
Synthesized from r/userexperience, r/assholedesign, regulator complaint records, and the dossier:
- **"I had to cancel three times."** FT-style flows where the obvious confirm button *changes your plan* instead of canceling; users report accidentally upgrading while trying to leave.
- **"They emailed 'great year ahead' — never said I was being charged."** IDF-style renewal emails disguised as goodwill; users only learn from their bank statement.
- **"The price doubled at checkout."** Xfinity/Ticketmaster fee reveals; users feel baited and abandon (the 48% number is these people).
- **"Reject took five menus; Accept was one button."** Cookie-banner fatigue; users click Accept out of exhaustion and resent it.
- **"It charged me for insurance I never picked."** RyanAir/IRCTC pre-checked add-ons; "ingeniously evil" is a recurring phrase.
- **"My kid spent $4,000 and the currency is deliberately confusing."** Roblox/Robux; parents can't tell real-money cost from the UI.
- **"It re-asked for notifications every single time."** Nagging; users uninstall.
- **"Honda made me give my name, address, and phone just to opt out of data sale."** Excessive friction on a right that should be one click — the exact conduct the CPPA fined.

## Senior-level checklist
- [ ] Every non-essential checkbox in the product defaults to **unchecked**? (Audit signup, checkout, settings.)
- [ ] All mandatory fees shown at the **first** price display, not final step?
- [ ] **Guest checkout** offered by default; account creation only optional post-purchase?
- [ ] Cancel path = same channel, same/fewer steps as signup (Click-to-Cancel)?
- [ ] Consent UI: Reject and Accept equal in **size, contrast, and click count**?
- [ ] No pre-ticked boxes for marketing, tracking, insurance, donation, or tips?
- [ ] Decline copy is **neutral** (no confirmshaming) everywhere?
- [ ] Countdown timers and stock/viewer counts are **genuinely real**; timers never reset?
- [ ] Auto-renewal disclosed at price-level weight; reminder email 7 days out with price in subject?
- [ ] Paid access runs to the end of the paid period after cancellation?
- [ ] Dismissed permission prompts respect a **≥30-day** cooldown; never re-enable disabled settings?
- [ ] Ads labeled at the content's own visual weight; no fake buttons?
- [ ] Data-rights / opt-out requests are **one-step**, no excessive verification?
- [ ] Success metrics include **retention, refunds, and support tickets** — not just immediate conversion?
- [ ] Manual review done (automated scanners catch <half the taxonomy)?
- [ ] Every interactive element (including close/X buttons, CTAs) does exactly what its label says (no bait-and-switch)?
- [ ] Pre-ticked add-ons, insurance, or donation boxes absent? (EU CRD Art. 22 refund obligation applies even if it "slips through.")

## Sources
- deceptive.design — https://www.deceptive.design/
- FTC, "Bringing Dark Patterns to Light" (2022) — https://www.ftc.gov/reports/bringing-dark-patterns-light
- FTC Click-to-Cancel Final Rule (Oct 2024) — https://www.ftc.gov/legal-library/browse/rules/negative-option-rule
- FTC Endorsement Guides (2023 update) — https://www.ftc.gov/legal-library/browse/rules/guides-concerning-use-endorsements-testimonials-advertising
- FTC v. Amazon.com (Prime consent decree, Dec 2023) — https://www.ftc.gov/legal-library/browse/cases-proceedings/2023180
- EDPB Guidelines 03/2022 on Deceptive Design Patterns — https://www.edpb.europa.eu/
- EU Digital Services Act — https://digital-strategy.ec.europa.eu/en/policies/digital-services-act-package
- EU Consumer Rights Directive 2011/83/EU Art. 22 (pre-ticked boxes) — https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32011L0083
- EU Digital Fairness Act proposal (Oct 2024) — https://ec.europa.eu/info/law/better-regulation/have-your-say/initiatives/14174-Digital-Fairness-Act_en
- Baymard Institute (checkout/cart-abandonment research) — https://baymard.com/
- Dark Pattern Analysis Framework (68-type taxonomy; tool coverage 45.5%) — https://arxiv.org/abs/2412.09147
- California CPPA v. Honda (2025) — https://cppa.ca.gov/enforcement/

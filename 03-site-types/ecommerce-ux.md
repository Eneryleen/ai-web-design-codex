# E-commerce UX

> Patterns for the conversion-critical surfaces of an online store: product listing, product detail, search, cart, and checkout, plus mobile commerce, reviews, price display, and abandonment recovery — grounded in Baymard Institute large-scale research. The senior-level outcome is a store that meets the Amazon/Walmart benchmarks users already expect (Jakob's Law), removes friction at every funnel step, and converts at rates close to the desktop ceiling on every device.

**Apply when:** building or auditing any transactional store (catalog → PDP → cart → checkout) · **Related:** [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md) · [Conversion Rate Optimization](../06-marketing-and-conversion/conversion-rate-optimization.md) · [Touch Ergonomics & Cross-Device Design](../02-responsive-adaptive/touch-ergonomics-cross-device.md) · [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md)

## TL;DR — rules at a glance
- **Checkout is the single biggest lever.** Industry avg cart abandonment is 70.19%; better checkout design alone can recover up to **35.26%** of lost sales (~$260B/yr US+EU). 65% of checkouts score "mediocre or worse," only 2% "good," 0% "perfect" — the bar is low.
- **Guest checkout is mandatory and visually primary.** Forced account creation causes **26%** of abandonments. Offer guest checkout above the fold; rank it visually above sign-in.
- **Surface all costs as early as possible.** "Extra costs too high" (shipping/tax/fees) is the #1 abandonment reason at **48%**. Show shipping cost / threshold / tax estimate on the PDP and cart, never only at the last step.
- **7–8 form fields, not 11.3.** Each field past 8 costs 4–6% completion; cutting 12→7–8 lifts conversion **25–35%**. Combine first+last name, auto-fill city/state from ZIP, type-don't-pick for state and card expiry.
- **Express checkout above the fold on cart AND checkout.** Apple/Google/Shop Pay early placement ~2x conversion vs end-of-flow (Stripe internal data). Apple Pay: +22.3% conversion lift (vendor-reported; directional). Digital wallets = 49–56% of global e-commerce transaction value (FIS Worldpay 2024).
- **Mobile is the gap to close.** Mobile = 68–72% of traffic but converts 1.53–2.85% vs desktop 3.36–3.9%. Tap targets ≥44×44px, primary CTAs in the bottom thumb zone, sticky add-to-cart / pay button.
- **Hide the coupon field behind a reveal.** A visible empty promo box sends 5–10% of users off-site hunting for codes.
- **Variants as swatches/buttons, never dropdowns.** Cross out / disable out-of-stock options inline. Swap product photos to match the selected color. 57% of sites still use cramped variant dropdowns.
- **Filters: count next to every option, AND across types / OR within a type, checkboxes (never radios), removable chips, no dead-end zero-results.**
- **Precision builds trust.** Exact delivery dates beat ranges; accurate live inventory beats "usually ships." Vague generalizations read as untrustworthy.
- **Reviews need reviewer context** (verified-purchase badge, fit/size/use-case). Let users sort to read negative reviews first — they want the worst case.
- **Free-shipping threshold messaging** cuts abandonment 15–23%; 93% of shoppers navigate to qualify, 58% add items to hit it.
- **1–2 recognizable security badges near the payment field**, not 20 in the footer (+15–30% for unknown brands). Wallpapering with badges weakens trust.
- **Progress indicator, max 3 steps** (contact → shipping/payment → review). Abandonment emails are the top recovery channel after checkout fixes (41.18% open, 50% convert when opened).

---

## 1. Product listing (PLP)

67% of leading sites have mediocre-to-poor navigation UX; 36% have product-list usability broken enough to actively harm discovery. The PLP's job is fast scanning and confident filtering toward a single PDP.

### Product cards — minimum viable info
Per card, no more, no less:
1. **Photo** large enough to distinguish items at a glance (clear product silhouette, consistent crop/background across the grid).
2. **Concise name containing the key differentiating attribute** (e.g., "Merino Wool Crew — Charcoal", not just "Crew Neck").
3. **Price** (and original/strikethrough price if discounted — see §8).
4. **Variant indicator** — color swatches or "+3 colors" so users see options without clicking.
5. Optional: rating + count, badge (New/Sale/Bestseller), wishlist toggle.

```
┌─────────────────┐
│                 │  ← image ratio fixed (e.g. 3:4), object-fit: cover
│     PHOTO       │     reserve aspect-ratio box to prevent CLS
│                 │
├─────────────────┤
│ Merino Crew     │  name w/ differentiator, 1–2 lines, ellipsize line 2
│ ●●●○ (128)      │  rating optional
│ $89  $̶1̶1̶9̶       │  price + strikethrough compare-at
│ ⬤ ⬤ ⬤ +2        │  color swatches, hover/tap to preview
└─────────────────┘
```

- **Quick View** is now *acceptable and helpful* for visually-driven categories (apparel, decor) — Baymard revised earlier skepticism circa 2022. Still avoid it for complex products that need PDP-level interaction (config, fit tools, long spec tables).
- **Lazy-load images below the fold** (native `loading="lazy"` or IntersectionObserver). On a 48-item grid with 300 KB images, eager loading 48 images adds ~14 MB of upfront transfer. Always reserve the image box with `aspect-ratio` to prevent CLS (see also §2 gallery preload note).
- Hover state: surface a second image (alternate angle / on-model) and the variant swatches. Never hide the price or CTA behind hover — mobile has no hover.

### Filtering (faceted navigation)
- **Sidebar filters on desktop** (left rail). Baymard now advises *caution* with horizontal filter toolbars — they cause layout problems and reduce the number of visible filter options.
- **Show a result count next to every option:** `Blue (34)`, `Under $50 (212)`. This is the single best defense against dead-end zero-result combinations.
- **Faceted logic:** **AND across filter types** (Brand AND Color), **OR within a type** (Blue OR Red). Implement multi-select inside a category with **checkboxes — never radio buttons** (radios force single-select and break the OR-within-type model).
- **Color filters must use swatches, not text labels alone.** "Navy" vs "Blue" vs "Indigo" is ambiguous as text; a swatch is unambiguous.
- **Active filters as removable chips** above the grid, plus **Clear All** and per-category reset.
- **Desktop: real-time updates** — results refresh as selections change. **Mobile: full-screen or bottom-sheet drawer with an explicit "Show 142 Results" apply button** — never live-refresh under a mid-interaction drawer (disorienting page jumps).
- **Never render a zero-results page with no recovery path.** If a combination yields nothing, keep the user on the grid, show "0 results — try removing a filter," and offer one-tap removal of the most restrictive facet.

### Sorting
- Default sort = "Relevance" (or "Featured"/"Best Selling" for curated merchandising). Offer: Price ↑/↓, Newest, Top Rated, Best Selling.
- Sort is a single-select control (dropdown/segmented), distinct and separate from filters. Don't bury sort inside the filter drawer on mobile — give it its own affordance.

### Breadcrumbs
- **Breadcrumbs are mandatory on PLPs and PDPs.** They serve three functions: back-navigation without full page reload, hierarchical orientation (women › shoes › running), and SEO breadcrumb rich results via `BreadcrumbList` schema.
- Use `aria-label="Breadcrumb"` on the `<nav>` and `aria-current="page"` on the final item. Breadcrumbs replace "back" button use for 38% of users navigating multi-level taxonomies (NN/g category finding). Place above the page H1.

### Pagination vs load-more vs infinite scroll
- **Pagination is now acceptable** with proper implementation (Baymard 2024 reversal) — "Load more" is no longer mandatory. Pagination preserves deep-linking, back-button position, and footer access.
- **Infinite scroll works well on mobile for discovery** but breaks footer reachability and back-button scroll restoration — use "Load more" + lazy pages there, or scroll-restore the position on back-navigation.
- Whatever you choose, **persist scroll position on back from PDP** — returning to the top of a 200-item grid is a top complaint.

---

## 2. Product detail page (PDP)

### Visual hierarchy — the conversion path
Guide the eye down a single dominant path; subordinate or remove anything that competes:

```
hero media → product title → price → variant selector → add-to-cart → supporting detail
```

- **Hero gallery** dominates above the fold. **Title** is the largest text. **Price** sits immediately under or beside the title. **Variant selector** then **add-to-cart** form a tight, unmissable cluster. Shipping/returns/trust cues are *supporting*, placed after the CTA cluster, not competing with it.
- Anything that pulls attention from this path (carousels of unrelated promos, oversized "you may also like", animated banners) gets demoted below the fold.

### Gallery
- **Multiple angles + scale context + lifestyle/on-model** shots. Single-angle imagery and photos that don't convey real scale are documented conversion killers.
- Thumbnails (desktop left/below) with a large main image; **click to open a fullscreen lightbox** (desktop and mobile) with swipe navigation. In-page zoom lens on desktop hover is secondary. Pinch-zoom on mobile must work inside the lightbox, not on the page itself. 2–3× zoom minimum.
- **Match model/context to the product category.** "Going-out" makeup styling on activewear models reads as wrong instantly (real critique pattern) — keep photography context-consistent.
- Reserve the image box with `aspect-ratio` to prevent layout shift; `<link rel="preload" as="image">` the LCP hero image. Lazy-load all non-hero gallery thumbnails.

### Variants
- **Swatches / buttons, not dropdowns.** 57% of sites still use cramped dropdowns. Buttons are larger tap targets, show all options at once, and support inline stock state.
- **Out-of-stock variants visible but disabled / crossed-out inline** — never force the user to add to cart to *discover* unavailability. Show "Notify me" on the disabled option where supported.
- **Swap the gallery to color-specific photos when a color is selected.** Photo↔variant mismatch erodes trust and converts worse.
- Show the selected variant's own price/stock/SKU. Reflect the choice in the URL (`?variant=charcoal-m`) for shareable, deep-linkable state.

```html
<!-- Color variant as swatches with inline OOS state -->
<fieldset>
  <legend>Color: <span data-selected>Charcoal</span></legend>
  <label><input type="radio" name="color" value="charcoal" checked>
    <span class="swatch" style="--c:#333" aria-label="Charcoal"></span></label>
  <label><input type="radio" name="color" value="navy">
    <span class="swatch" style="--c:#1b2a4a" aria-label="Navy"></span></label>
  <label data-oos><input type="radio" name="color" value="olive" disabled>
    <span class="swatch swatch--oos" aria-label="Olive — out of stock"></span></label>
</fieldset>
```

### Add-to-cart, Buy Now, quantity, and feedback
- **Add-to-Cart is the highest-contrast element on the page.** Persistent, large, unambiguous label ("Add to Cart" / "Add to Bag").
- **"Buy Now" / express CTA is optional but valuable** for impulse and repurchase scenarios. Place it immediately below or alongside Add-to-Cart, never *above* it (Buy Now skips the cart — unexpected for many users and hides cross-sell opportunity). Label it "Buy Now" or "Checkout" not "Express."
- **Quantity: use a stepper with a typable field** (− / value / +), not a pure dropdown. 61% of sites don't use the optimal button+field combination for quantity.
- **Strong add-to-cart confirmation feedback** — a visible toast/mini-cart slide-in confirming "Added · 1 item · $89." Weak feedback makes users re-click and add duplicates (NN/g finding).
- **Sticky add-to-cart bar** on long desktop PDPs and on mobile. Model: the pattern where the bar transitions **"Add to Cart" → "Choose Your Size"** when no required variant is selected — it both reminds and instructs without scrolling back up.

### Price & cost transparency on the PDP
- Show **price, any compare-at strikethrough, and the unit/financing breakdown** (see §8).
- **Surface shipping cost or the free-shipping threshold on the PDP**, plus a delivery estimate. Pushing cost disclosure to checkout is the top abandonment cause (48%).
- **BNPL hint** (e.g., "or 4× $22.25 with [provider]") near the price — BNPL is associated with AOV lifts of 20–50% and revenue lifts of ~10–20% (vendor-reported; Klarna/Afterpay merchant data, not independent RCT) — but keep it a single concise line, not a banner.

### Copy
- **Front-load and scan-optimize.** Users skim, reading most at the start. Lead with the differentiating spec, then bullets, then long prose.
- **No fluffy/AI marketing voice** ("you become the best version of yourself"). People skip it and lose trust. Concrete attributes (material, weight, dimensions, care, compatibility) convert.

### Trust on the PDP
Trust is built through: real photography (no stock/CGI deception), an **immediately visible return policy** (link or inline summary — Baymard flags buried/missing return policy as a top PDP trust failure), accurate live inventory, current verified reviews, recognizable brand/security signals, and policy text that reads like a human wrote it. The single fastest trust fix is making the return policy visible without a click. "We don't accept returns" stated clearly is more trusted than silence or a buried link.

---

## 3. Search

- **On-site search users convert at multiples of browsers** — treat search as a first-class surface, not a filter.
- **Autocomplete/suggest** with product thumbnails, category scopes, and recent/popular queries. Tolerate typos, synonyms, and plurals ("tshirt" = "t-shirt" = "tee"). Highlight the matched portion of the suggestion in bold.
- **Autocomplete "no match" handling:** when the partial query produces no suggest candidates, show "Search for [query]" as the first item (submitting the raw query). Never show an empty dropdown — it reads as broken.
- **Search results page = a PLP** — apply the same filters, sorting, counts, and card rules from §1.
- **Never dead-end on "0 results."** Show corrected-spelling suggestions, broaden automatically ("No results for *blue runing shoe* — showing *blue running shoes*"), and surface popular categories. A bare zero-results screen is a conversion killer.
- Make the search field obvious (visible input on desktop, one-tap reveal on mobile) and keep the user's query in the box on the results page.
- **Expose a `q` query parameter in the URL** of the results page — enables sharing, analytics, and personalised email recovery ("You searched for X — here's what's back in stock").

---

## 4. Cart

The cart is a review-and-reassure step, not a sales pitch. Reduce anxiety; remove reasons to leave.

- **Show full line items with thumbnail, name, selected variant, qty stepper, line price, and remove.** Editable in place.
- **Order summary always visible** (subtotal, shipping/estimate, tax/estimate, total). **Do not collapse the summary into an accordion** — users want to verify items without tapping.
- **Coupon field on the *cart* page is acceptable; inside checkout it is problematic.** Best pattern everywhere: **toggle-reveal** ("Have a code?") so the empty box doesn't trigger off-site code-hunting (5–10% leak).
- **Free-shipping progress meter:** "You're $14 from free shipping" + progress bar. 93% navigate to qualify, 58% add items to hit it; messaging cuts abandonment 15–23%.
- **Express checkout buttons above the fold on the cart** (Apple/Google/Shop Pay/PayPal) alongside the standard "Checkout" button.
- **Persistent, one-tap cart access** from a header/bottom-nav icon with an item-count badge. Cart contents should survive sessions (logged-in) and reasonable timeouts (guest).
- Show **stock urgency only when true** ("Only 3 left") — fabricated scarcity is a dark pattern and erodes the trust you just built (see [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md)).
- **Cross-sell/upsell placement:** "Frequently bought together" and "Complete the look" are acceptable in the cart *below the fold*, after the primary checkout CTA. Never between the item list and the checkout button (increases abandonment). On the PDP, below-the-fold "You may also like" is safe; above-the-fold cross-sell competes with the conversion path and hurts CVR.

---

## 5. Checkout

65% of checkouts are mediocre-or-worse; this is where the 35.26% recoverable conversion lives.

### Guest checkout (non-negotiable)
- **Offer guest checkout above the fold, visually prioritized over sign-in.** Forced account creation = 26% of all abandonments.
- Pattern: three options — **Guest** (primary), Sign in, Create account — with guest as the default/most prominent. Offer account creation *after* purchase ("Save your details? Set a password") using data already collected.

### Field economy
- **Target 7–8 fields total** (industry avg 11.3). Each field past 8 costs 4–6% completion; 12→7–8 lifts conversion 25–35%.
- **Combine first + last name** into one field.
- **Full address autocomplete first, ZIP-to-city/state fallback.** The best pattern: an address autocomplete (Google Places API, Loqate, or SmartyStreets) that fills street, city, state, and ZIP from a single type-ahead field. 55% of sites lack even ZIP-to-city/state lookup. ZIP-to-city/state alone is acceptable; manual full entry is not.
- **State: a text input, not a dropdown** — typing two letters beats the iOS picker.
- **Card expiry: a typed 4-digit field, not dropdowns.**
- Use correct `type`/`inputmode`/`autocomplete` so mobile keyboards and autofill do the work:

```html
<input name="name"  autocomplete="name"            type="text">
<input name="email" autocomplete="email"  inputmode="email" type="email">
<input name="zip"   autocomplete="postal-code" inputmode="numeric">
<input name="addr"  autocomplete="address-line1">
<input name="cc"    autocomplete="cc-number" inputmode="numeric"
       pattern="[0-9 ]*">
<input name="exp"   autocomplete="cc-exp"  inputmode="numeric"
       placeholder="MM/YY" maxlength="5">
```

### Flow & progress
- **Max 3 steps:** contact info → shipping/payment → review. Show a **persistent "Step 2 of 3" indicator** — it defuses progress anxiety and reduces abandonment.
- Single-page (accordion) checkout is also fine if the field count and visible step cues are right; the rule is *few fields + clear progress*, not a specific page count.
- **Keep the order summary visible throughout checkout** (don't hide it behind a tap).

### Payment
- **Express checkout above the fold here too** (Apple Pay +22.3% conversion / +22.5% revenue — vendor-reported, treat as directional; early placement ~2x vs end-of-flow per Stripe internal data). Digital wallets = 49–56% of global e-commerce transaction value (FIS Worldpay 2024).
- **Offer BNPL alongside card/wallet** (AOV +20–50%, revenue +10–20% — vendor-reported from Klarna/Afterpay merchant data) — but as one clean option, not clutter on the payment step.
- **1–2 recognizable security badges near the payment fields** (not only the footer): McAfee 79% / Verisign 76% / PayPal 72% recognition; +15–30% for lesser-known brands; 18% of users explicitly look for a security signal before entering card data. **Do not wallpaper with 20+ badges — it weakens credibility.**

### Errors & validation
- **Inline, field-level validation** on blur, not a single error dump on submit. Keep already-entered values on error.
- **Specific, recoverable messages:** "Card declined — try another card or check the number," not "Error." Point to the exact field; move focus to the first error.
- **Never lose the user's cart or form data** on a payment failure or back-navigation.
- Format/validate card, ZIP, and phone live; show accepted-card icons; mask but allow reveal of sensitive entries where appropriate.

---

## 6. Mobile commerce

Mobile is 68–72% of traffic but converts 1.53–2.85% vs desktop 3.36–3.9% — the largest single addressable gap. The failure mode is **porting a desktop layout to a phone**.

- **Tap targets ≥ 44×44px** (Apple HIG), **48×48dp** (Material 3), **24×24 CSS px min** (WCAG 2.5.8). Adult fingertip covers ~44–57px. Most mobile checkout forms ship sub-44px targets → mis-taps and errors.
- **Thumb zone:** 49–75% of phone use is one-handed thumb tapping. Put primary CTAs and core nav in the **bottom two-thirds** of the screen.
- **Bottom tab navigation beats hamburger.** Hiding nav behind a hamburger reduces product discovery 20–30%. Cart must be reachable in **one tap** from a persistent bottom-nav icon.
- **Sticky bottom action button** — "Add to Cart" on PDP, "Pay Now" / "Place Order" in checkout — pinned above the safe-area inset.
- **The keyboard covers 40–50% of the screen when open.** Keep the active field and the submit button visible; scroll the focused input into view; don't trap the CTA under the keyboard.
- **Filters as a full-screen / bottom-sheet drawer** with an explicit "Show X Results" apply button — never a squeezed desktop sidebar, never live-refresh under the drawer.
- **No hover-dependent interactions, no fixed-width desktop layouts, no top-of-screen-only nav.** Replace hover-reveals (swatches, quick view) with tap.

```css
/* Sticky mobile add-to-cart / pay bar above the safe area */
.sticky-cta {
  position: sticky; bottom: 0;
  padding: 12px 16px calc(12px + env(safe-area-inset-bottom));
  background: #fff; box-shadow: 0 -4px 16px rgb(0 0 0 / .08);
}
.sticky-cta button { min-height: 48px; width: 100%; font-size: 16px; }
/* 16px input font-size prevents iOS auto-zoom on focus */
input, select, textarea { font-size: 16px; }
```

---

## 7. Reviews & social proof

- **Zero reviews on a new store is a top conversion killer.** Absence of reviews is one of the most-cited barriers to first-purchase trust (Baymard; NN/g). Seed honest social proof early — even 5–10 detailed verified reviews outperform none. Don't launch a PDP with zero reviews and no plan to collect them.
- **Reviewer context is what builds trust:** verified-purchase badge, plus relevant structured attributes — size/fit, body type, height, skin type, use case, "true to size" sliders. Generic "5 stars, great!" carries negligible weight compared to a 3-star review that answers "does it fit large?"
- **Let users sort/filter to read negative reviews first.** Users actively seek the worst case to gauge risk; a "Lowest rated" sort and a rating-distribution histogram (filterable by star) signal confidence, not weakness.
- **Show the rating distribution histogram**, photo/video reviews, and "X of Y found helpful" voting.
- **Platform note:** Google reviews carry the highest consumer trust and surface in Google AI Overviews (as of 2025). Trustpilot has an ongoing merchant-community reputation problem around paid removal requests; don't rely on it as your sole platform. Collect natively on Google and use a review platform (Yotpo/Okendo/Judge.me) for on-site display.
- Social proof beyond reviews: real-purchase counts, UGC galleries, press/credential logos, and named-customer signals — all only if genuine.

---

## 8. Price display

- **Front-and-center, high-contrast, unambiguous.** Currency symbol + price near the title on PDP and on every card.
- **Discounts:** show current price prominently, original as a strikethrough `compare-at`, and optionally the % or amount saved. Don't fake "original" prices — illegal in many jurisdictions and a trust killer.
- **Total cost transparency early** (§2/§5): shipping cost or free-shipping threshold and a tax estimate on PDP/cart. Hidden extras = 48% abandonment.
- **Precision over ranges:** exact delivery date ("Arrives Tue, Jun 17") beats "3–7 business days"; live "12 in stock" beats "usually available." Vague ranges read as evasive.
- **BNPL / financing line** under price ("4× $22.25" / "from $19/mo") where offered — concise, one line.
- **Unit pricing** ("$0.32 / 100ml") for consumables/groceries; **per-period** for subscriptions with the full first-charge total disclosed before purchase.

```html
<p class="price">
  <span class="now" aria-label="Current price">$89.00</span>
  <del class="was">$119.00</del>
  <span class="save">Save 25%</span>
</p>
<p class="bnpl">or 4 interest-free payments of $22.25</p>
```

---

## 9. Reducing cart abandonment

Average abandonment is 70.19% (Baymard 50-study avg); mobile 78–85%, desktop 65–70%, tablet 80.74%. Attack the named causes in priority order:

| Cause (% of abandoners) | Fix |
|---|---|
| Extra costs too high — **48%** | Show shipping/tax/fees on PDP & cart; free-shipping threshold meter |
| Forced account creation — **26%** | Guest checkout above the fold, prioritized over sign-in |
| Don't trust site with card — **25%** | Security badges near payment, real photography, clear policies, recognizable wallets |
| Delivery too slow — **23%** | Precise delivery dates; expedited options; set expectations on PDP |
| Checkout too long/complex — **22%** | 7–8 fields, 3-step progress, autofill, express checkout |

- **Free-shipping notification** cuts abandonment 15–23% and raises AOV.
- **Recovery emails** are the top channel after on-site fixes: **41.18% open rate, ~50% conversion when opened** (note: openers are self-selected warm intent — effective conversion of all abandoners is far lower). Send a 3-touch sequence (≈1h, 24h, 48h) with the exact cart contents and a one-click return-to-cart.
- **Post-purchase / confirmation-page upsells** accept at 3–8%; **one-click** post-purchase upsells dramatically outperform redirect-based offers. Keep them off the critical checkout path.

---

## Concrete specs & numbers
- **Cart abandonment:** 70.19% avg · mobile 78–85% · desktop 65–70% · tablet 80.74%.
- **Conversion rate:** mobile 1.53–2.85% · desktop 3.36–3.9%. Mobile traffic share 68–72%.
- **Recoverable via checkout UX:** up to 35.26% conversion, ~$260B/yr US+EU.
- **Checkout fields:** target 7–8 (industry avg 11.3); each field >8 = −4–6% completion; 12→7–8 = +25–35%.
- **Express checkout:** ~2x conversion when above the fold vs end-of-flow (Stripe internal). Apple Pay +22.3% conversion / +22.5% revenue (vendor-reported; directional). Digital wallets = 49–56% of global e-commerce transaction value (FIS Worldpay 2024).
- **BNPL:** AOV +20–50%, revenue +10–20% (vendor-reported, Klarna/Afterpay merchant data).
- **Free shipping:** 93% navigate to qualify; 58% add items to hit threshold; messaging −15–23% abandonment.
- **Security badges:** +15–30% for unknown brands; recognition McAfee 79% / Verisign 76% / PayPal 72%; 18% look for a security indicator before paying.
- **Tap targets:** ≥44×44px (Apple HIG) · 48×48dp (Material 3) · ≥24×24 CSS px (WCAG 2.5.8). Fingertip ~44–57px.
- **Mobile keyboard** covers 40–50% of screen. Input `font-size:16px` to block iOS zoom.
- **Recovery emails:** 41.18% open · ~50% convert when opened. Confirmation upsell accept 3–8%.
- **Checkout step indicator:** max 3 steps.
- **Baymard prevalence (room to improve):** 57% missing mobile color swatches · 50% missing quick view · 55% no address auto-lookup · 61% non-optimal quantity controls · 50% omit some fulfillment options in selectors · 95% of leading sites don't highlight current nav location.
- **Quality baselines:** 67% of sites have mediocre-poor navigation · 65% mediocre-or-worse checkout (2% good, 0% perfect) · 36% have product-list UX that harms discovery.

## Tools & libraries
- **Platform/checkout:** Shopify (+ Shop Pay) or a headless stack (Medusa, Commerce.js, Saleor, BigCommerce) front-ended by Next/Astro. Use platform express-checkout SDKs rather than building wallet flows.
- **Payments:** Stripe / Stripe Elements (Payment Element bundles cards + wallets + BNPL), Adyen, Braintree, PayPal. Apple Pay / Google Pay via the **Payment Request API** / wallet buttons.
- **Search:** Algolia, Typesense, or Meilisearch for typo-tolerant instant search + faceting — don't hand-roll fuzzy search.
- **Reviews:** a platform that captures verified-purchase + structured fit/size attributes (Yotpo, Okendo, Judge.me, or Google review collection). Avoid leaning solely on Trustpilot.
- **Structured data (Schema.org):** implement `Product`, `Offer`, `AggregateRating`, and `BreadcrumbList` JSON-LD on every PDP and PLP. This enables Google Product rich results (price, rating, availability in SERP), breadcrumb rich results, and feeds into AI Overview citations. Test with Google's Rich Results Test. Omitting this leaves SERP real estate and click-through rate on the table.
- **Address autocomplete:** Google Places Autocomplete API, Loqate, or SmartyStreets — covers international postal formats. Platform checkout providers (Shopify, Stripe) handle this out of the box; add it explicitly for headless builds.
- **When NOT to:** don't build a custom payment field set — use the provider's hosted/Element fields for PCI scope reduction. Don't build bespoke faceted search at scale — use a search engine. Don't reinvent address autocomplete — use the platform/Google/Loqate lookup.

## Do / Don't
**Do**
- Offer guest checkout above the fold, visually above sign-in.
- Disclose shipping/tax/fees on the PDP and cart.
- Use swatch/button variants with inline OOS state and color-matched photos.
- Use full address autocomplete (Google Places / Loqate); fall back to ZIP→city/state; type-don't-pick for state and card expiry.
- Put express checkout above the fold on cart and checkout.
- Use ≥44×44px tap targets and a sticky bottom CTA on mobile.
- Show result counts on every filter option and removable filter chips.
- Give strong, single add-to-cart confirmation feedback.
- Show return policy visibly on PDP (no click required).
- Add breadcrumb navigation on PLPs and PDPs with `BreadcrumbList` JSON-LD.
- Implement `Product`/`Offer`/`AggregateRating` Schema.org JSON-LD on every PDP.

**Don't**
- Don't force account creation or hide guest checkout.
- Don't reveal extra costs only at the final step.
- Don't show an empty coupon box (toggle-reveal instead).
- Don't use dropdowns for variants, state, or card expiry.
- Don't render dead-end zero-result filter/search pages.
- Don't collapse the order summary into an accordion.
- Don't port desktop nav/filters/hover to mobile.
- Don't wallpaper checkout with 20+ trust badges or fake scarcity.
- Don't place cross-sell between the item list and the checkout button.
- Don't omit the return policy from the PDP — hiding it implies unfavorable terms.
- Don't launch a PDP with zero reviews and no collection plan.

## Common pitfalls & how to avoid them
- **Costs revealed only at the last step** — the #1 abandonment cause (48%). Lift them to PDP/cart and add a free-shipping meter.
- **Forced account creation** (26%). Make guest checkout the prominent default; offer account creation post-purchase.
- **Visible empty coupon field inside checkout** — sends users off-site (5–10%). Toggle-reveal pattern.
- **Desktop ported to mobile** — top nav, hamburger, sidebar filters, hover, fixed widths all fail. Design mobile-first with bottom nav and drawers.
- **Photo doesn't update on color select** / mismatched model context — confusion and distrust. Bind gallery to variant.
- **OOS discovered only after add-to-cart** — show disabled/crossed-out variants on PLP and PDP.
- **Color filters as text labels only** — "navy" vs "blue" ambiguity. Use swatches.
- **No sticky add-to-cart on long PDP/mobile** — users scroll back up to buy. Pin it.
- **Zero-results with no recovery** — keep user on grid, suggest filter removal, offer one-tap reset.
- **Collapsed order summary** — users can't verify items. Keep it visible.
- **Thin/single-angle photography, no scale** — add angles, lifestyle, and scale references.
- **Fluffy AI marketing copy** — users skip it. Front-load concrete attributes.
- **Weak add-to-cart feedback** — users add duplicates (NN/g). Show a clear confirmation toast/mini-cart.
- **No reviews on a new store** — seed honest verified reviews; show reviewer context.
- **20+ trust badges** — weakens credibility. Use 1–2 recognizable badges near payment.

## What real users complain about
- "I got to the last step and *then* saw $14 shipping." (extra costs — the top quantified complaint)
- "Why do I have to create an account just to buy one thing?" (forced sign-up)
- "There was an empty promo-code box, so I left to find a code and never came back."
- "I selected 'olive' and the photo stayed blue — is the photo wrong or the product?" (variant/photo mismatch)
- "It let me add it to the cart, then said the size was sold out at checkout." (late OOS discovery)
- "The mobile filters opened a tiny desktop sidebar I couldn't tap accurately." (ported filters, sub-44px targets)
- "I tapped 'Add to Cart' twice because nothing happened — ended up with two." (weak confirmation)
- "I wanted the 1- and 2-star reviews first to see what's actually wrong." (negative-review-first intent)
- "A no-name store wanted my card with zero trust signals on the page." (missing payment-adjacent badges)
- "Came back from a product and the listing dumped me at the top of 200 items." (lost scroll position)
- Practitioners on r/ecommerce: zero social proof is the most-cited conversion killer for new stores; Trustpilot is distrusted vs Google reviews.

## Senior-level checklist
- [ ] Cards carry photo + differentiating name + price + variant indicator; aspect-ratio reserved (no CLS); images below the fold are lazy-loaded.
- [ ] Filters: sidebar (desktop) / apply-button drawer (mobile), counts on every option, checkboxes, AND-across / OR-within, removable chips, Clear All, no dead-ends.
- [ ] Breadcrumbs on PLP and PDP: visible, accessible (`aria-label="Breadcrumb"`, `aria-current="page"`), `BreadcrumbList` JSON-LD present.
- [ ] Color filters and variants use swatches; OOS shown inline (disabled/crossed-out) before add-to-cart.
- [ ] PDP hierarchy: media → title → price → variants → add-to-cart → support; sticky add-to-cart on mobile/long PDP.
- [ ] Gallery: multi-angle + scale + context, color-matched on variant select, lightbox zoom with swipe, LCP hero preloaded, thumbnails lazy-loaded.
- [ ] Return policy visible on PDP without a click.
- [ ] Shipping cost / free-ship threshold + delivery estimate shown on PDP and cart; precise dates over ranges.
- [ ] Search: typo-tolerant autocomplete with thumbnails; results page = full PLP; no zero-result dead-end.
- [ ] Cart: editable line items, always-visible summary, toggle-reveal coupon, free-ship meter, express checkout above fold, one-tap access.
- [ ] Guest checkout above the fold, prioritized over sign-in; account creation offered post-purchase.
- [ ] Checkout: 7–8 fields, combined name, full address autocomplete (or ZIP→city/state minimum), typed state + expiry, correct `autocomplete`/`inputmode`.
- [ ] Express checkout (Apple/Google/Shop Pay) above the fold on cart and checkout; BNPL offered cleanly.
- [ ] 1–2 recognizable security badges near payment; inline field-level errors that preserve input and point to the field.
- [ ] Progress indicator ≤3 steps; order summary visible throughout.
- [ ] Mobile: ≥44×44px targets, bottom-nav cart, sticky pay button above safe-area, `font-size:16px` inputs, keyboard-aware scroll.
- [ ] Reviews: verified-purchase badges, reviewer context (size/fit/use), rating histogram, negative-review-first sorting; at least one review platform collecting Google-verified reviews; no PDPs launched with zero reviews.
- [ ] Schema.org JSON-LD: `Product`, `Offer`, `AggregateRating` on PDPs; `BreadcrumbList` on PLPs/PDPs. Validated with Rich Results Test.
- [ ] Price: prominent, honest compare-at, % saved, BNPL line, unit/period pricing where relevant.
- [ ] Abandonment recovery email sequence wired (cart contents + one-click return); confirmation-page / one-click upsell live.
- [ ] No dark patterns (fake scarcity/countdowns, hidden costs, sneak-into-basket, confirmshaming).

## Sources
- Baymard Institute — Cart & Checkout, Product Lists & Filtering, Product Page, Mobile, and Search UX research (200,000+ cumulative research hours; 110+ checkout guidelines; 160+ product-finding guidelines; 400+ sites benchmarked): https://baymard.com/research
- Baymard Institute — Cart Abandonment Rate Statistics: https://baymard.com/lists/cart-abandonment-rate
- Nielsen Norman Group (NN/g) — e-commerce UX, trust, and add-to-cart feedback research: https://www.nngroup.com/topic/ecommerce/
- Jakob's Law / Laws of UX: https://lawsofux.com/jakobs-law/
- Apple Human Interface Guidelines — touch targets / Apple Pay: https://developer.apple.com/design/human-interface-guidelines
- Material Design 3 — touch target sizing: https://m3.material.io/foundations/accessible-design/accessibility-basics
- WCAG 2.2 — Target Size (Minimum) 2.5.8: https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html
- Stripe — payments / express checkout documentation: https://stripe.com/docs/payments
- FIS Worldpay Global Payments Report 2024 — digital wallet transaction share: https://worldpay.com/en-gb/worldpay-global-payments-report
- Google — Rich Results Test + Product structured data documentation: https://developers.google.com/search/docs/appearance/structured-data/product
- **Vendor-sourced stats caveat:** Apple Pay conversion lift (+22.3%), BNPL AOV/revenue lifts, and express checkout placement multipliers are derived from payment-provider marketing materials (Apple, Klarna, Afterpay, Stripe). Treat as directional. No independent peer-reviewed replication is known. Baymard and NN/g figures are from independent research.

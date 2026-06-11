# What Real Users Complain About (Synthesis)
> A synthesis of what real people actually rage about online — cookie walls, slow loads, popups before value, autoplay, gray-on-white text, broken mobile, sticky chrome eating the viewport, footer-trapping infinite scroll, forced login/app installs, hidden contact info, dead search, form resets, and jank. Each complaint is paired with the underlying design law and the corrective spec, so you can build sites that the same users would not complain about.

**Apply when:** designing, reviewing, or auditing any public-facing site before ship · **Related:** [Common Design Mistakes & Antipatterns](./common-mistakes-antipatterns.md) · [Dark Patterns to Avoid](./dark-patterns-to-avoid.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md)

## TL;DR — rules at a glance
- **Speed is the #1 complaint.** 53% of mobile visits abandon at >3s load. LCP ≤2.5s, INP ≤200ms, CLS ≤0.1. Every 100ms of server delay ≈ 1% fewer conversions (Amazon).
- **No popup, modal, or interstitial before the user has consumed value.** Newsletter modals at 2-3s = instant bounce. Ask for nothing until you've given something.
- **Autoplay with sound is a rage trigger.** Consistently the top-ranked annoyance in browser UX surveys (and why browsers block it by default since 2018). Never autoplay audio; decorative video only if muted, `playsinline`, and `prefers-reduced-motion`-aware.
- **Gray-on-white kills.** 83.9% of homepages fail contrast (WebAIM Million 2026). Body ≥4.5:1, large text ≥3:1, no `#ccc`/`#999` body copy, no placeholder-as-label.
- **Mobile is the baseline, not a port.** >50% of global web traffic is mobile (StatCounter). Broken mobile layouts are a top-five complaint in every major UX survey. No tap target <24px, no input that triggers zoom, no horizontal overflow.
- **Never hide what the user came for.** Price, contact phone, cancellation path, delivery policy — surface proactively. Hidden contact info is one of the most upvoted complaints on Reddit.
- **Don't force login or app installs to see content.** Forced-registration walls before any value is delivered drive severe day-1 abandonment. "Open in App" hijacks and App Store redirects are universally hated.
- **Sticky chrome is a tax.** A header + cookie bar + chat bubble can eat 30-40% of the mobile viewport. Keep sticky headers ≤56-64px; collapse on scroll-down.
- **Infinite scroll without a reachable footer is a trap.** Pair with "Load more" or pagination; keep contact/policy links reachable; preserve scroll position on back.
- **Search is the lifeline when nav fails.** Handle typos, plurals, hyphens, synonyms. Never return 0 results where fuzzy-match would help; never show an empty state with no recovery.
- **Never wipe a form on validation error.** Preserve every field, validate inline, name the exact field and the fix. "Something went wrong" is an abandonment generator.
- **Cookie banners must be honest and accessible.** Reject must be as easy as Accept (one click, equal prominence). Never block content or trap a screen reader behind the overlay.
- **Confirmshaming and pre-checked boxes burn trust permanently.** Users who perceive manipulation consistently report they will not return; regulators treat it as an unfair commercial practice.
- **CAPTCHAs are a friction tax on legitimate users.** Use invisible risk-scoring (reCAPTCHA v3, Cloudflare Turnstile) by default; only surface a challenge on suspicious signals.
- **95.9% of homepages have WCAG failures** — mostly invisible to sighted devs. Every image needs `alt`, every input needs `<label>`, no stripped focus indicators, dynamic updates need `aria-live`.

---

## 1. Slow loads — the universal, #1 complaint
Speed outranks every other usability concern. **53% of mobile visits are abandoned if a page takes longer than 3 seconds** to load (Google/SOASTA, 2016 — directionally consistent with current field data). A 2006 Amazon internal experiment found every **100ms** of added latency costs ~**1%** of conversions; more recent Deloitte and Vodafone studies found 0.1s improvement in site speed lifts retail conversion ~8–10%.

**Targets (Core Web Vitals, Google):**
| Metric | Good | Needs work | Poor |
|---|---|---|---|
| LCP (largest paint) | ≤2.5s | 2.5–4s | >4s |
| INP (interaction→next paint) | ≤200ms | 200–500ms | >500ms |
| CLS (layout shift) | ≤0.1 | 0.1–0.25 | >0.25 |

As of the 2025 Web Almanac, only **48% of mobile** and **56% of desktop** pages pass all three. **INP is the hardest to pass** — after replacing FID in March 2024, INP failure is the primary reason pages fall out of the Good band; LCP passes at ~62% on mobile but INP lags significantly.

**What users say, and the fix:**
- "Fake progress bars" that fill arbitrarily then jump to 100% → show *real* progress or a determinate skeleton; never fake it.
- Mis-clicks from content shifting under the cursor (CLS) → reserve space with `width`/`height`/`aspect-ratio`; never inject ads/banners above already-rendered content.

```css
/* Stop layout shift: reserve the box before the asset loads */
img, video { aspect-ratio: 16 / 9; width: 100%; height: auto; }
```
→ See [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md).

## 2. Cookie consent banners — hated and often illegal
Cookie banners are near-universally despised, and a large share are *non-compliant theater*: "we track you, click OK" notifications dressed up as consent.

**What users and regulators object to:**
- **Cookie walls** that block all content until you accept — contrary to GDPR guidance (consent must be freely given).
- **Asymmetric choices**: a bright one-click "Accept all" next to a buried, multi-click "Reject" or "Manage preferences." Reject must be **equally easy and equally prominent** as Accept.
- **Accessibility breakage**: screen readers get trapped or disoriented by overlays that don't trap focus correctly or aren't announced.
- **Re-pestering**: consent stored only in a browser cookie that resets on hosting changes, so users get re-asked on every device and every clear-cookies.

**Rules:**
- Provide **Accept / Reject** at the *same level* with **one click each**. "Manage" is optional, never the only alternative to "Accept all."
- Don't load non-essential scripts before consent. Don't block content behind the banner.
- Make the overlay a real modal: focus-trapped, `Esc`-dismissible to the safest default, announced to assistive tech (`role="dialog"`, `aria-modal="true"`, labelled).
- Don't reset the banner on the same device once a choice is made.

## 3. Intrusive popups & newsletter modals — the attention tax
The most consistent rage in usability research: a **newsletter signup modal that fires 2-3 seconds after load, before the user has read a word.** Every interstitial is a withdrawal from an attention budget the user never extended to you.

**Specific patterns users name:**
- Modals that reappear on **every page** because the dismiss cookie isn't respected.
- Exit-intent popups with shake/wiggle animations (widely documented by NN/g as "manipulative animation").
- Browser tab titles rewritten to "Come back!" or "Don't leave!" when the tab loses focus — a trick that's both intrusive and now actively blocked by Chromium's Idle Detection restrictions.

**Rules:**
- **No interstitial before value.** Trigger a signup offer (if at all) on scroll depth (~50-70%), time-on-content (>30s of engaged reading), or genuine exit intent — not on load.
- **One modal per session, max.** Respect dismissal with a persisted flag; never re-fire on every route change.
- Google penalizes intrusive **mobile interstitials** in search since 2017 — there's an SEO cost on top of the UX cost.
- A modal must have a visible **≥44×44px close** affordance, be `Esc`-dismissible, and trap focus.

## 4. Autoplay audio & video — close-the-tab territory
Autoplay audio is the most universally rage-inducing pattern in usability research: the canonical complaint is opening five news tabs and hearing five videos at once, then hunting each tab to mute it. Browsers already block autoplay with sound by default (Chrome since 2018, Firefox and Safari follow suit) — so autoplaying with sound both annoys users *and* silently fails.

**Rules:**
- **Never autoplay with sound.** Ever. Browsers block it anyway, so it both annoys users and silently fails.
- Decorative background video: `muted`, `playsinline`, `loop`, and gated on `prefers-reduced-motion: no-preference`. Offer a visible pause control.
- Burns mobile data and battery — provide a poster image and let the user opt in to heavy media on cellular.

```html
<video autoplay muted loop playsinline poster="hero.jpg">
  <source src="hero.webm" type="video/webm">
</video>
```
```css
@media (prefers-reduced-motion: reduce) { video { display: none; } }
```

## 5. Sticky headers & chrome eating the viewport
Large sticky headers consume **15-25% of visible screen** on mobile. Stack a cookie bar, a chat widget, and an "Open in App" banner on top and users lose **30-40% of the viewport** to chrome before seeing any content.

**Rules:**
- Sticky header height **≤56-64px** on mobile. If it must be taller, collapse it on scroll-down and restore on scroll-up.
- Only **one** persistent floating element. A header + cookie bar + chat bubble + app banner is three too many.
- Anchor scrolling must account for the sticky header so anchored content isn't hidden under it:

```css
:target { scroll-margin-top: 64px; } /* match sticky header height */
html { scroll-padding-top: 64px; }
```
- Chat widgets: default to a small bubble, never an auto-opened panel; respect a dismissal.

## 6. Infinite scroll & the unreachable footer
Aza Raskin, who invented infinite scroll, has publicly said he regrets it. Real complaints cluster on three failures:
- **The footer is unreachable** — contact info, delivery policy, and terms live in a footer that perpetually retreats as new content loads.
- **Lost scroll position** — tap an item, hit back, land at the top, never find the item again.
- **No sense of progress or end** — no way to bookmark "page 7" or know how much is left.

**Rules:**
- Use **pagination** or a **"Load more" button** for any content where the footer matters (e-commerce, docs, anything transactional). Reserve true infinite scroll for ephemeral feeds.
- If you must auto-load, surface footer links in a **persistent location** (sticky utility bar, header menu) and add an explicit "Load more" that pauses auto-loading.
- **Restore scroll position on back** (`history.scrollRestoration = 'manual'` + saved offset, or framework scroll restoration).
- Keep keyboard users out of a trap: a focusable element must always be reachable past the feed (WCAG 2.1.1).
→ See [Common Design Mistakes & Antipatterns](./common-mistakes-antipatterns.md) for the keyboard-trap detail.

## 7. Forced login & forced app installs
Withholding content behind a wall before showing value is one of the most resented patterns. **Day-1 mobile app churn from forced registration walls is substantial** — Nielsen's research on app onboarding shows users consistently abandon before completing mandatory registration when no value has been demonstrated first.

**Patterns users specifically rage about:**
- **"Open in App" banners** that hijack the mobile browser; "Continue" buttons that dump users into the App Store instead of the site (Reddit, Yelp, Google named by users; Ultimate Guitar's app-force thread hit 4000+ upvotes).
- Login walls in front of content a search engine sent them to.
- Hiding menu prices to force an app download (McDonald's cited).

**Rules:**
- Let users **consume core content first.** Gate after demonstrated value (read 2 articles, configured a cart), and offer **guest checkout** — forced account creation is a top-tier e-commerce abandonment cause.
- The mobile **web** must be fully functional. An app banner is at most a small, dismissible strip — never a full-screen takeover, never the only path.
- Permission prompts (notifications, location, camera) **in context**, after the user triggers the feature that needs them — never on page load.

## 8. Hidden contact information
Contact obfuscation is a persistent top-voted complaint across Reddit, Trustpilot reviews, and consumer protection forums: companies funnel users to chatbots and FAQs and bury or remove the phone number entirely. NN/g's field studies consistently find users expect a "Contact" link in the header or footer to resolve within two clicks to a human channel.

**Rules:**
- A **"Contact"** link in the primary nav and footer, reaching a page with phone, email, and hours **as text** (not an image, not only a form).
- Don't make a chatbot the *only* channel. Offer a human path.
- Surface the info users came for — **price, contact, cancellation, returns, delivery** — proactively. Burying it doesn't reduce the questions; it just makes people leave angrier.

## 9. Bad or broken search — the lifeline that fails
Nielsen calls search "the user's lifeline when navigation fails." Real complaints: search that returns **zero results for a valid query with a one-character typo** or a plural/singular mismatch, and that ranks by keyword density instead of relevance.

**Rules:**
- Tolerate **typos** (fuzzy / edit-distance), **plurals**, **hyphens** ("t shirt" = "t-shirt" = "tshirt"), and **synonyms**.
- Never a bare "0 results" dead end — show suggestions ("did you mean…"), related items, popular searches, and a clear path onward.
- Show the query back to the user, make it editable in place, and don't lose filters.
- Autosuggest with results, scoped facets, and recent searches beat a raw keyword match.
→ See [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md).

## 10. Low contrast & tiny fonts
The dominant 2016-2022 minimalist aesthetic — **light gray on white, thin fonts at 12-14px** — is still everywhere: **83.9% of pages fail contrast** (average 34 instances each; WebAIM Million 2026).

**Specs (WCAG 1.4.3):** body ≥**4.5:1**, large text (≥24px or ≥18.67px bold) ≥**3:1**, UI components/icons/focus ≥**3:1** (1.4.11).
- `#ccc` on `#fff` = **1.6:1** (fail). `#999` on `#fff` = **2.85:1** (fail). Use `#595959` (7:1) or darker for muted text.
- **Placeholder text must not be styled like filled-in text**, and placeholder is never the label.
- The majority of fixed font-size declarations found in audits default to 12–14px, which is below the broadly recommended 16px minimum for body copy. Use **relative units** (`rem`/`em`) for type so browser font-size preferences are respected; never hard-code body copy in `px` below 16px.

```css
body { font-size: 1rem; }        /* respects user's browser setting */
.muted { color: #595959; }       /* not #ccc / #999 */
```
→ See [Web Typography Systems](../00-foundations/typography-systems.md) and [Color Theory & Systems](../00-foundations/color-theory-and-systems.md).

## 11. Broken mobile layouts
Mobile is the majority channel — >50% of global web traffic (StatCounter, 2024–2026). Users report broken mobile experiences as some of their most rage-inducing encounters, and expectations are that the mobile web should match or exceed desktop quality. The concrete failures users hit:
- **Buttons/links too small to tap** — Baymard research consistently ranks this in the top mobile UX failures on e-commerce sites; 66% of audited sites have tappable elements too close together.
- **Inputs that trigger zoom** on focus (iOS zooms when font-size <16px).
- **Horizontal overflow** forcing side-scroll; text overflowing its container.
- **Many mobile sites** fail to show loading indicators; a significant share of US mobile sites disable pinch-to-zoom on product images (documented in Baymard mobile e-commerce audits).

**Rules:**
- Tap targets: **24×24 CSS px** WCAG 2.5.8 floor; **44×44pt** (Apple HIG) / **48×48dp** (Material) for primary actions; ≥8px spacing between targets.
- Form inputs at **≥16px** font-size to prevent iOS zoom.
- Never disable pinch-to-zoom: drop `user-scalable=no` / `maximum-scale=1` from the viewport meta.

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```
```css
input, select, textarea { font-size: 16px; } /* no iOS zoom-on-focus */
.btn { min-block-size: 44px; min-inline-size: 44px; }
```
→ See [Touch Ergonomics & Cross-Device Design](../02-responsive-adaptive/touch-ergonomics-cross-device.md).

## 12. Jank, scroll-jacking & layout shift
Beyond raw speed, users complain about *motion that fights them*:
- **Scroll-jacking** — full-page snap that overrides natural scroll speed (agency/portfolio sites). Users overshoot, can't control pace, can't find content.
- **Parallax** that causes nausea for vestibular-sensitive users.
- **Forced intro animations** before content is visible.
- **CLS jank** — content jumping as async assets land, causing mis-clicks.

**Rules:**
- Don't hijack scroll. If you need controlled stops, use CSS `scroll-snap`, not JS that intercepts wheel/touch events.
- Gate all decorative motion on `prefers-reduced-motion: no-preference`.
- No animation should block first content. Show content; layer motion on top.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: .01ms !important; transition: none !important; scroll-behavior: auto !important; }
}
```
→ See [Scroll-Driven Experiences](../03-site-types/scroll-driven-experiences.md) and [Micro-interactions & Motion](../07-aesthetics-and-trends/micro-interactions-and-motion.md).

## 13. Forms that punish the user
Forms are where most rage occurs because they're the transactional core. Complaints:
- **The whole form wipes** after one validation error — re-enter everything.
- **No inline validation** — submit, get a generic banner, scroll to find the bad field.
- **Vague errors**: "please complete all required fields" without naming which; "invalid input" without saying what format.
- **Overly strict passwords** (82% of sites impose unnecessary rules) and **password-reset friction** (up to 18% of checkout abandonment, Baymard).

**Rules:**
- **Never clear entered data on error.** Preserve every field; users have spent up to 5 minutes recovering from vague errors (Baymard).
- **Validate inline**, on blur, next to the field; name the exact field and the exact fix ("Email must include an @").
- **Adaptive error messages** — 4–7 distinct messages per complex field, not one generic line (Baymard labels this a critical gap in their form UX benchmarks).
- Offer **guest checkout**; relax password rules to a length-based policy; allow paste in password fields.
→ See [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md).

## 14. CAPTCHAs — friction tax on legitimate users
CAPTCHAs are among the highest-friction points on the web, disproportionately punishing users with disabilities, older users, and non-native English speakers. The annoyance is compounded when they appear on low-risk actions (login, newsletter signup) or re-trigger on every session.

**What users complain about:**
- Image-grid CAPTCHAs that cycle indefinitely or reset on a single wrong selection.
- Audio alternatives that are incomprehensible even for non-disabled users.
- Appearing mid-flow (e.g., in checkout, behind a paywall, before any suspicious signal).

**Data:** only ~28% of users pass image-grid CAPTCHAs on the first attempt; ~65% who fail two or more times abandon the task entirely; ~68% report CAPTCHAs feel harder on mobile (Google internal 2014, frequently cited in WAI research contexts — treat as directional).

**Rules:**
- **Replace visible CAPTCHAs with invisible risk-based signals** (honeypots, behavioral analysis, reCAPTCHA v3 score thresholds, Cloudflare Turnstile) for the vast majority of users.
- If a hard challenge must appear, gate it on a risk-score threshold — not on every visitor by default.
- Always provide an accessible audio alternative that actually works; test it with a screen reader before ship.
- Never place CAPTCHA before the user has provided any input (e.g., on page load or as a gate to view content).

## 15. Accessibility failures beyond contrast
**95.9% of homepages have detectable WCAG 2 failures** (WebAIM Million 2026, avg 56.1 errors/page). Most are never noticed by sighted users — until something breaks for someone who depends on the interface.

**The top five automatically detectable failures (WebAIM Million 2026):**
1. Missing alt text — 53.1% of pages
2. Missing form labels — 51%
3. Empty links — 46.3%
4. Low contrast — 83.9% (covered in §10)
5. Empty buttons — 30.6%

**What non-sighted and keyboard users actually complain about (forums, a11y Twitter/Mastodon):**
- **Keyboard traps**: modal or widget you can tab into but cannot leave without a mouse.
- **Focus indicator stripped**: `outline: none` on `:focus` makes keyboard navigation invisible — tabbing around the page with no visual cue.
- **Dynamic content never announced**: filter results update silently; screen reader user still hears the old state.
- **PDFs for on-page content**: untagged PDFs are screen-reader nightmares and break mobile scrolling/back navigation entirely.

**Rules:**
- Every `<img>` has `alt`; decorative images get `alt=""`. Every `<input>` has a `<label>` (not just `placeholder`). Every `<button>` and `<a>` has non-empty accessible text.
- Never remove the focus indicator (`outline: none` on `:focus` is a keyboard accessibility failure). Replace it with a custom `:focus-visible` style if the default is ugly.
- Use `aria-live` regions to announce dynamic content changes (filters, cart updates, form errors).
- Use HTML for on-page content, not PDFs.
- Treat automated tools (Lighthouse, axe, WAVE) as a **floor** — they catch ~30–40% of actual issues. Manual keyboard and screen-reader testing is required.
→ See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md).

## 16. Unlabeled icons (mystery-meat nav)
Icon-only bars force recall over recognition. A pencil could mean edit, compose, draw, annotate, or settings. Only the magnifying glass (search) is near-universal; everything else is a guess — especially on mobile where space pressure pushes designers into icon-only patterns.

**Rules:** every interactive icon carries a **visible text label** (or at minimum a tooltip + `aria-label`, but visible text is the standard). Pair icon + label; don't ship a row of naked glyphs.
→ See [Imagery, Icons & Visual Assets](../00-foundations/imagery-icons-and-assets.md).

## 17. Confirmshaming & manipulative copy (dark patterns)
Once users detect manipulation trust collapses — qualitative and survey research consistently shows they will not return and will warn others — and there's now hard regulatory liability. **97% of the most-downloaded EU apps contain dark-pattern elements (2022 EU consumer protection sweep); the 2025 EU enforcement sweep found ~40% of online stores concealing info or using visual trickery; the FTC's 2023 Amazon settlement was $30.8 million** for dark-pattern Prime enrollment and deliberately obstructed cancellation ("Iliad") flows.

**Patterns users (and regulators) punish:**
- **Confirmshaming** opt-outs: "No thanks, I don't like saving money" / "I prefer to pay full price."
- **Pre-checked boxes** for recurring donations, insurance add-ons, or newsletters.
- **Swapped button labels** so the primary-styled action destroys data.
- **Fake countdown timers** and **hidden costs** revealed only at the final step.

**Rules:** make Reject/Cancel as easy as Accept; never pre-check consent or add-ons; primary visual weight goes to the user's most likely benign action, never the destructive one; no fake urgency.
→ See [Dark Patterns to Avoid](./dark-patterns-to-avoid.md) and [Ethical Persuasion Psychology](../06-marketing-and-conversion/persuasion-psychology.md).

## Concrete specs & numbers
- **Abandonment:** 53% of mobile visits abandon at >3s load. 70.19% average cart abandonment (Baymard). Up to 18% of checkout abandonment tied to password reset.
- **Core Web Vitals:** LCP ≤2.5s · INP ≤200ms · CLS ≤0.1 (Good thresholds). Only 48% mobile / 56% desktop pages pass all three (2025 Web Almanac).
- **Contrast:** 4.5:1 body · 3:1 large text (≥24px / ≥18.67px bold) · 3:1 UI components. 83.9% of pages fail (WebAIM Million 2026).
- **Accessibility prevalence:** 95.9% of homepages have detectable WCAG 2 failures; avg 56.1 errors/page; only 4.1% fully conform. Missing alt text 53.1%; missing form labels 51%; empty links 46.3%; empty buttons 30.6%.
- **Tap targets:** 24×24 CSS px (WCAG 2.5.8 AA) · 44×44pt (Apple HIG) · 48×48dp (Material). Inputs ≥16px to avoid iOS zoom.
- **Autoplay:** universally top-ranked annoyance in UX surveys; browsers block autoplay-with-sound by default (Chrome 2018+, Firefox/Safari follow).
- **Mobile failures:** 66% of audited sites have tappable elements too close together (Baymard); pinch-to-zoom disabled on product images is a common finding in Baymard mobile audits.
- **CAPTCHA:** ~28% solve image-grid CAPTCHAs on the first attempt; ~65% who fail multiple times abandon the task; ~68% find them harder on mobile (directional figures from WAI/Google research — treat as order-of-magnitude, not precise benchmarks).
- **Dark patterns:** 97% of the most-downloaded EU apps contain ≥1 dark pattern (2022 EU sweep); ~40% of EU online stores conceal info or use visual trickery (2025 EU enforcement sweep); 1 in 10 of 11,000 shopping sites use deceptive design (Princeton 2019); FTC 2023 Amazon settlement = $30.8M for Prime dark patterns.
- **Checkout/e-commerce UX (Baymard):** 22% of abandonment from long/complicated checkout; ~$260B abandoned/yr (US/EU) from checkout UX; 64% of leading sites score "mediocre or worse" on checkout; 88% omit ratings in list items; 87% omit price-per-unit; 82% over-strict passwords.
- **Hamburger vs tabs (Nielsen):** tab nav 2.4s vs hamburger 3.5s task completion.

## Tools & libraries
- **Performance:** Lighthouse, PageSpeed Insights, WebPageTest, Chrome DevTools Performance + CrUX field data for real CWV.
- **Contrast/a11y:** axe DevTools, WAVE, Lighthouse a11y audit, Stark, manual screen-reader passes (VoiceOver/NVDA). Treat automated tools as a floor — they catch ~30-40% of issues.
- **Consent:** any CMP must be configured for symmetric Accept/Reject, no content-blocking wall, focus-trapped accessible overlay. Audit the *output*, not the vendor's marketing.
- **When NOT:** don't add a chat-widget SDK, exit-intent popup library, or app-banner script by default — each is a complaint vector. Add only with a measured, consented purpose.

## Do / Don't
**Do**
- Show content first; ask for email/login/permission only after delivering value.
- Make Reject as easy as Accept; offer guest checkout; preserve form data on error.
- Keep contact, price, and cancellation paths one click from anywhere.
- Tolerate typos/plurals/synonyms in search; never dead-end on 0 results.
- Reserve layout space for media; gate motion on `prefers-reduced-motion`.
- Use relative font units ≥16px body; muted text ≥4.5:1.

**Don't**
- Fire a newsletter modal on load, autoplay sound, or hijack scroll.
- Block pinch-to-zoom, ship targets <24px, or zoom inputs on focus.
- Trap users in infinite scroll with an unreachable footer.
- Force app installs, hijack with "Open in App," or redirect to the App Store.
- Wipe forms on error or show "something went wrong" with no recovery.
- Pre-check consent/add-ons, confirmshame opt-outs, or fake countdowns.
- Ship icon-only nav, strip the focus indicator, or omit alt/label/button text.
- Show a CAPTCHA to every visitor by default; never gate viewing content behind a CAPTCHA.

## Common pitfalls & how to avoid them
- **Permission prompts on load** (notifications/location/camera) → ask in context, after the feature is triggered.
- **Splash/entry interstitials** with no content → delete; they're a useless step for new and returning users (Nielsen).
- **Links opening new tabs without warning** → break the Back button and orphan tabs; open in same tab unless the user expects otherwise, and mark exceptions.
- **PDF for on-page content** → tiny text, broken scrolling/back, unreadable on mobile; use HTML.
- **Non-persistent URLs** → script-driven nav leaving only the homepage bookmarkable; every resource needs a permanent URL.
- **Visited links not recolored** → users revisit seen pages; keep a distinct `:visited` style.
- **Banner-shaped CTAs** → banner blindness hides them; don't style important content like an ad.
- **Poor page titles** → "Welcome to [Company]" or duplicate titles across pages; lead with the distinct page name + value (first ~66 chars show in SERPs).
- **Hamburger menu on desktop** → hides structure and forces recall (3.5s vs 2.4s for tabs); show top-level nav when space allows.

## Senior-level checklist
- [ ] LCP ≤2.5s, INP ≤200ms, CLS ≤0.1 on a mid-tier mobile device over throttled 4G (field data, not just lab).
- [ ] No popup/modal/interstitial fires before the user has consumed value; max one per session; `Esc` + ≥44px close.
- [ ] No autoplay sound anywhere; decorative video muted + `playsinline` + reduced-motion-aware + pausable.
- [ ] Cookie banner: one-click Reject equal in prominence to Accept; no content wall; non-essential scripts deferred until consent; overlay is a focus-trapped accessible dialog.
- [ ] Sticky header ≤64px; only one persistent floating element; `scroll-padding-top` set for anchors.
- [ ] Body text ≥4.5:1, large ≥3:1, UI/focus ≥3:1, verified at every state; no `#ccc`/`#999` body copy; placeholder ≠ label.
- [ ] All tap targets ≥24px (44 for primary), ≥8px apart; inputs ≥16px; pinch-to-zoom enabled.
- [ ] Guest checkout available; no forced registration to view content; permission prompts in context.
- [ ] Contact (phone + email as text), price, returns, and cancellation reachable in ≤1 click from any page.
- [ ] Search tolerates typos/plurals/hyphens/synonyms; 0-results state offers suggestions and a path onward.
- [ ] Forms validate inline, name the exact field + fix, never wipe entered data; passwords length-based; paste allowed.
- [ ] Every interactive icon has a visible label; no mystery-meat nav.
- [ ] No scroll-jacking; all decorative motion gated on `prefers-reduced-motion`; no intro animation blocks first content.
- [ ] Infinite scroll (if used) preserves scroll position on back and keeps footer/keyboard escape reachable.
- [ ] No confirmshaming, pre-checked consent/add-ons, swapped destructive buttons, or fake urgency.
- [ ] Every `<img>` has `alt`, every `<input>` has `<label>`, every `<button>`/`<a>` has accessible text; focus indicator visible (`:focus-visible` not stripped).
- [ ] CAPTCHA is risk-gated, not shown to all users by default; accessible audio alternative tested; never gates content viewing.
- [ ] Dynamic content changes (filters, cart, errors) announced to screen readers via `aria-live` regions.
- [ ] Run axe/Lighthouse a11y as a floor, then do a manual keyboard-only pass and at least one screen-reader spot-check before ship.

## Sources
- WebAIM Million annual accessibility report — https://webaim.org/projects/million/
- Core Web Vitals thresholds (web.dev) — https://web.dev/articles/vitals
- HTTP Archive Web Almanac — https://almanac.httparchive.org/
- Baymard Institute (checkout / e-commerce UX research) — https://baymard.com/research
- Nielsen Norman Group (usability heuristics, top mistakes, search) — https://www.nngroup.com/articles/
- WCAG 2.2 (W3C) — https://www.w3.org/TR/WCAG22/
- FTC v. Amazon dark-pattern settlement (press release) — https://www.ftc.gov/news-events

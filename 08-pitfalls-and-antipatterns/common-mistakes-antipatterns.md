# Common Design Mistakes & Antipatterns
> A field guide to the recurring, measurable mistakes that wreck usability, accessibility, and conversion: low-contrast text, tiny tap targets, killed focus states, carousels, scroll hijacking, autoplay, intrusive modals, footer-trapping infinite scroll, mystery-meat nav, spacing/typeface chaos, layout-shift jank, broken tables, placeholder-as-label, and unexplained disabled states. Each entry gives the spec, the threshold, the failure mode, and the correct pattern so you can avoid shipping them at a senior level.

**Apply when:** designing or reviewing any web UI before ship · **Related:** [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Dark Patterns to Avoid](./dark-patterns-to-avoid.md) · [What Real Users Complain About](./real-user-complaints.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md)

## TL;DR — rules at a glance
- **Contrast:** body text ≥ **4.5:1**, large text (≥24px or ≥18.67px bold) ≥ **3:1**; UI borders/icons/focus rings ≥ **3:1** (WCAG 1.4.3, 1.4.11). 79.8% of the top 1M homepages fail this (WebAIM Million 2025) — don't be one.
- **Tap targets:** **24×24 CSS px** WCAG 2.2 AA floor; **44×44** (Apple HIG / WCAG AAA) or **48×48dp** (Material) for primary actions.
- **Never** `outline: none` without a replacement focus ring at ≥3:1. Keyboard users (~7.5M in the US) navigate by it.
- **Carousels:** auto-rotating ones get ~1% click-through and 84% of those clicks hit slide 1. Prefer a static hero (43.03% interaction vs 1.96% in the canonical Notre Dame A/B test).
- **Scroll hijacking:** don't override native scroll with JS; use CSS `scroll-snap` if you need controlled stops.
- **Autoplay with sound:** browsers block it (Chrome since 2018); WCAG 1.4.2 bans uncontrolled audio >3s. Silent decorative video is fine only with `prefers-reduced-motion` guard and a visible pause control.
- **Modals/popups:** never on page load for first-time visitors (Google demotes mobile interstitials since 2017). Escape + visible 44px close, one per page, exit-intent or scroll-depth trigger.
- **Infinite scroll:** always pair with a "Load more" button and keep the footer reachable, or you create a WCAG 2.1.1 / 2.4.3 keyboard trap.
- **Every input** needs a persistent `<label>`. Placeholder is never the sole label.
- **Disabled buttons** must explain the unmet prerequisite (or use an enabled button that errors on click).
- **Icons** always carry a visible text label (mystery-meat nav forces recall over recognition).
- **CLS ≤ 0.1.** Set `width`/`height` on `<img>`/`<video>`, reserve space with `aspect-ratio`, manage font swap.
- **Spacing** from a 4px/8px grid only. **Typefaces** ≤ 2–3. **Color is never the sole signal** (WCAG 1.4.1).
- **Tables** never overflow the viewport — wrap in `overflow-x: auto` or stack at ≤600px.

---

## 1. Low-contrast text — the #1 failure
Low text contrast is the single most common automated accessibility failure: **79.8% of the top 1M homepages fail WCAG AA contrast** (WebAIM Million 2025 — essentially flat from 80.7% in 2024; improvement has stalled).

**Thresholds (WCAG 1.4.3 AA / 1.4.6 AAA):**
| Text | AA | AAA |
|---|---|---|
| Normal (<24px, <18.67px bold) | 4.5:1 | 7:1 |
| Large (≥24px, or ≥18.67px bold) | 3:1 | 4.5:1 |

**Common critical fails:**
- `#ccc` gray on `#fff` = **1.6:1** (fail). The classic "light gray placeholder-looking body text."
- Pure green `#00FF00` on white = **1.4:1** (critical fail). Bright saturated colors are deceptively low-contrast.
- White text on light brand-color buttons; mid-gray secondary text on near-white cards.

**Rules:**
- Check contrast at **every state**: default, hover, focus, active, disabled, error — not just default.
- Don't trust "looks fine on my monitor." Use a tool (see below).
- Disabled elements are the *only* WCAG contrast exemption — but keep them legible anyway (see §14).

```css
/* Wrong: 1.6:1 */
.muted { color: #ccc; background: #fff; }
/* Right: #595959 on #fff = 7:1 (passes AAA) */
.muted { color: #595959; background: #fff; }
```

## 2. Tiny tap targets
Targets too small to hit reliably cause mis-taps, rage, and abandonment — worst on mobile and for motor-impaired users.

**Specs:**
- **WCAG 2.5.8 (2.2) AA:** minimum **24×24 CSS px**, with spacing so a 24px circle doesn't overlap a neighbor.
- **WCAG 2.5.5 AAA / Apple HIG:** **44×44 pt/px**.
- **Material Design:** **48×48 dp**.

**Rules:**
- Use 44×44 as your working default for any primary action; 24×24 is a floor, not a goal.
- The *target* can exceed the *visual* size — pad the hit area with padding or a pseudo-element.

```css
/* Visual icon is 24px, but tap target is 44px */
.icon-btn { display: inline-grid; place-items: center; inline-size: 44px; block-size: 44px; }
```

## 3. Missing / removed focus states
`outline: none` or `outline: 0` without a replacement destroys keyboard navigation. This affects ~7.5M keyboard-reliant users in the US alone and fails **WCAG 2.4.7 (Focus Visible) AA**.

**Rules:**
- Never strip the outline without replacing it. The fix is a *better* ring, not no ring.
- Focus indicator must hit **3:1 contrast** against adjacent colors (WCAG 1.4.11). WCAG 2.4.13 AAA tightens this further: the focus indicator area must be ≥ the perimeter of the focused component × 2 CSS px — in practice, a 2px solid ring around the whole element satisfies this.
- Use `:focus-visible` so mouse clicks don't trigger a visible ring but keyboard and AT focus does.

```css
/* Wrong */
button:focus { outline: none; }
/* Right */
button:focus-visible {
  outline: 2px solid #1a56db;   /* ≥3:1 vs background */
  outline-offset: 2px;
}
```

## 4. Carousels / auto-rotating sliders
Carousels are the canonical "looks busy, does nothing" pattern.

**The data:**
- Notre Dame homepage A/B test (2013, widely cited): the hero carousel's first-slide click rate was ≈**1% of all visitors**; the equivalent static hero slot saw **43.03%** interaction vs carousel **1.96%** (interaction = any click in zone, not just the CTA).
- **84%** of the few who click choose slide 1; engagement past slide 1 is negligible on auto-rotating carousels.
- Banner blindness studies consistently show rotating content is pattern-matched and ignored.

**Why they fail:** banner blindness, content hidden behind rotation, motion accessibility violations, and they compete with your primary CTA.

**If you must use one:**
- **No auto-advance.** Manual prev/next only.
- Provide a visible, labeled **pause** control (WCAG 2.2.2 — anything that auto-moves >5s needs pause/stop/hide).
- Labeled controls, keyboard operable, slide-position announced via ARIA.
- Respect `prefers-reduced-motion`.

**Better:** a single strong hero with one CTA; or a static grid of stacked promo blocks.

## 5. Scroll hijacking
JavaScript that intercepts `wheel`/`touchmove` to override native scrolling breaks the user's platform scrolling contract. It's disorienting everywhere and especially harmful on mobile, where large inline scroll areas trap touch events and scrollbars are hidden by default (iOS/Android/macOS hide inactive scrollbars).

**Rules:**
- Don't `preventDefault()` on scroll to drive page motion.
- For controlled, sectioned scroll experiences use **CSS `scroll-snap`** — it's interruptible and respects momentum.
- For scroll-tied animation, drive it from scroll *position* (CSS scroll-driven animations / IntersectionObserver), never by hijacking the scroll event itself.

```css
.snap-container { scroll-snap-type: y proximity; overflow-y: auto; }
.snap-section { scroll-snap-align: start; min-block-size: 100svh; }
```
See [Scroll-Driven Experiences](../03-site-types/scroll-driven-experiences.md) for doing this well.

## 6. Autoplaying media
Autoplaying audio is universally despised and browser-blocked. Chrome has enforced an audio-autoplay block policy since 2018 (Autoplay Policy, Chrome 66+); Firefox and Safari followed. WCAG **1.4.2** requires user control for any audio that plays >3s.

**Rules:**
- **Audio + autoplay = never** without explicit user action.
- Silent, looping, decorative video is acceptable **only if** it respects `prefers-reduced-motion` and doesn't block content.
- Always provide a visible pause/mute control.

```css
@media (prefers-reduced-motion: reduce) {
  .bg-video { display: none; }
  .bg-video-poster { display: block; } /* static <img> shown instead */
}
```

## 7. Intrusive modals & popups
Popups that appear before the user reads anything spike bounce rates and, since January 2017, Google's mobile-search ranking algorithm demotes pages with **intrusive interstitials** (full-screen overlays, large banners covering the main content, stand-alone popups requiring dismissal before accessing content).

**Timing guidance:**
- Trigger on exit-intent or after ≥50% scroll depth — not on load.
- Delay ≥5s if a timed trigger is unavoidable; earlier triggers consistently show worse engagement in practitioner A/B literature.
- Avoid stacking popups: one per page max. The "4 layers to read a sentence" pattern (notification prompt + email modal + cookie banner + sidebar ad) is the most-cited user frustration in qualitative UX research.

**Rules:**
- **Never** trigger on page load for first-time visitors.
- **One popup per page**, maximum.
- Keyboard-accessible close: visible ≥44×44 target **and** Escape key support.
- Trap focus inside the modal while open; return focus to the trigger on close.
- Prefer exit-intent or scroll-depth triggers over timers.

## 8. Infinite scroll that traps the footer
Continuously loading content as the user nears the bottom pushes the footer perpetually out of reach. For keyboard users this is a hard trap: **fails WCAG 2.1.1 (Keyboard) and 2.4.3 (Focus Order)** — footer links (delivery policy, contact, legal) become unreachable. A recurring practitioner complaint: "e-commerce sites with delivery policy links in the footer you can never reach because new stuff always loads."

**Rules:**
- Provide an accessible **"Load more" button** as the primary or alternative mechanism — it gives keyboard users a stop point and makes the footer reachable.
- Announce newly loaded content with an ARIA live region.
- Preserve scroll position and focus when new items append.
- Consider classic pagination for content with a footer that matters (commerce, docs).

## 9. Mystery-meat navigation
Term coined 1998 (Vincent Flanders): navigation where you can't tell what a control does without hovering/clicking — most commonly **icon-only nav**. It forces *recall* over *recognition*.

**The data:**
- NNGroup study: hamburger menu on **desktop** — task time **3.5s** vs **2.4s** with a visible tab bar (~46% slower). The efficiency gap compounds on information-dense sites with deep navigation.

**Rules (NN Group):** every navigation icon needs a **visible text label**.
- Icon + label, always, for primary nav.
- Hamburger is acceptable on mobile, harmful on desktop for non-trivial navigation — expose top-level items as a tab/horizontal nav on wide screens.
- Don't rely on hover to reveal what a link is.

```html
<!-- Wrong: mystery meat --><a href="/cart"><svg .../></a>
<!-- Right --><a href="/cart"><svg aria-hidden="true" .../><span>Cart</span></a>
```
See [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md).

## 10. Too many fonts / colors
More than **2–3 typefaces** collapses hierarchy into noise; pairing *visually similar* fonts reads as a mistake. Unsystematic color does the same.

**Rules:**
- **2–3 typefaces max.** Build hierarchy with **weight + size + spacing**, not new fonts or font color alone.
- Color must have purpose; **never the sole signal** (WCAG 1.4.1) — red/green text alone excludes color-blind users. Pair with icon, underline, or text.
- Limit your palette: a few semantic roles (brand, neutral scale, success/warning/danger) — not arbitrary one-off hexes.

See [Web Typography Systems](../00-foundations/typography-systems.md) and [Color Theory & Systems](../00-foundations/color-theory-and-systems.md).

## 11. Inconsistent spacing
Arbitrary pixel values destroy visual rhythm. Real-world offenders: containers at `163×403px`, a `border-width: 1.5849902px`. These come from dragging in a design tool instead of using a scale.

**Rules:**
- Every spacing value derives from a **base-4 or base-8 grid**: 4, 8, 12, 16, 24, 32, 48, 64…
- Tokenize spacing; reference tokens, never raw magic numbers.
- Round borders/radii to whole pixels.

```css
:root { --space-1:4px; --space-2:8px; --space-3:12px; --space-4:16px; --space-6:24px; --space-8:32px; }
.card { padding: var(--space-4); gap: var(--space-3); }
```
See [Layout, Grids & Spacing](../00-foundations/layout-grids-and-spacing.md) and [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md).

## 12. Layout-shift jank (CLS)
Content that jumps as the page loads makes users mis-click and feel the page is broken. **CLS good ≤ 0.1, needs work 0.1–0.25, poor >0.25** (CLS = impact fraction × distance fraction).

**Causes & fixes:**
- **Images without dimensions** → set `width`/`height` attributes (or `aspect-ratio`); the browser reserves space.
- **Web font swap (FOUT/FOIT)** → `font-display: optional` is safest for CLS (skips swap if the font is late); `font-display: swap` if fast text matters more; combine with `size-adjust`/`preload`. Never omit `font-display`.
- **Injected DOM** (ads, banners, cookie bars) → reserve fixed space; never insert above existing content.

```html
<!-- Set width+height so the browser reserves layout space before load -->
<!-- fetchpriority="high" on the LCP image — don't lazy-load it -->
<img src="hero.jpg" width="1200" height="675" alt="…"
     fetchpriority="high" decoding="async">
<!-- Non-critical below-fold images: lazy-load -->
<img src="card.jpg" width="400" height="300" alt="…"
     loading="lazy" decoding="async">
```
```css
.media { aspect-ratio: 16 / 9; } /* reserves space even before load */
@font-face { font-family: Inter; src: url(inter.woff2) format("woff2"); font-display: optional; }
```
See [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md).

## 13. Non-responsive tables
Fixed-width tables overflow small viewports, forcing horizontal scroll to read a single row.

**Fixes (pick one):**
- **Wrap** in a scroll container — simplest, keeps tabular structure, and the scroll is contained, not page-wide.
- **Stack** at ≤600px: turn each row into a card with `data-label` pseudo-content.

```css
/* Option A: contained horizontal scroll (preferred — preserves semantics) */
.table-wrap { overflow-x: auto; }  /* -webkit-overflow-scrolling: touch is deprecated since iOS 13 */

/* Option B: stack on narrow screens */
@media (max-width: 600px) {
  table, thead, tbody, tr, td { display: block; }
  /* sr-only pattern — safer than position:absolute left:-9999px alone */
  thead {
    position: absolute; width: 1px; height: 1px;
    padding: 0; margin: -1px; overflow: hidden;
    clip: rect(0,0,0,0); white-space: nowrap; border: 0;
  }
  td { padding-left: 50%; position: relative; }
  td::before { content: attr(data-label); position: absolute; left: 12px; font-weight: 600; }
}
```
```html
<!-- scope attributes are required for screen reader row/column association -->
<th scope="col">Price</th>
…
<td data-label="Price">$24.00</td>
```

## 14. Placeholder-as-label
Using placeholder text instead of a real label is a top form mistake. Placeholder **vanishes on the first keystroke**, isn't reliably announced as the field's accessible name, and placeholder gray usually **fails 4.5:1**.

**Rules:**
- **Every `<input>`/`<textarea>` has a persistent visible `<label>`.** Placeholder is supplementary (a format hint), never the sole label.
- If you want a floating-label look, keep the label in the DOM and animate it — don't delete it.

```html
<!-- Wrong --><input placeholder="Email">
<!-- Right -->
<label for="email">Email</label>
<input id="email" type="email" placeholder="name@example.com">
```
See [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md).

## 15. Disabled buttons without explanation
A grayed-out submit button with no reason leaves users guessing the unmet prerequisite. WCAG exempts disabled controls from contrast, but that's not license to make them mysterious.

**Rules:**
- Tell the user **what to do to enable it** — inline helper text, a tooltip, or list the missing requirement.
- Better in many cases: keep the button **enabled** and show a **specific error** on click (so screen-reader and keyboard users get feedback and can discover the requirement).
- If disabled, keep it visibly legible and clearly *disabled* (not just faintly different).

```html
<button type="submit" aria-describedby="why-disabled" disabled>Pay now</button>
<p id="why-disabled">Add a payment method to continue.</p>
```
See [Interaction Design & UI States](../01-ux-fundamentals/interaction-design-and-states.md).

## 16. Additional critical antipatterns

**Forced registration / login walls before content.**
Baymard Institute (large-scale checkout UX research) consistently identifies forced account creation as a top abandonment reason — roughly 1 in 4 checkout abandonment events traces to it. Always offer guest checkout; let users register *after* purchase.

**CAPTCHA friction.**
CAPTCHAs add measurable conversion friction. Prefer passive signals (reCAPTCHA v3, honeypot fields, rate-limiting) over challenge CAPTCHAs. If you must show one, ensure it is keyboard-accessible and provides an audio alternative (WCAG requirement).

**Inline scroll traps for filter options.**
Filter panels locked in a tiny fixed-height `overflow-y: auto` container cause users to miss options that exist but aren't visible. Baymard's e-commerce UX research documents this as a "false simplicity" failure — users conclude the option doesn't exist and either abandon or over-filter. If the filter list exceeds ~8 items, expose it in a panel with a "Show more" toggle rather than truncating with scroll.

**`scroll-behavior: smooth` without `prefers-reduced-motion` guard.**
CSS `scroll-behavior: smooth` and equivalent JS `scrollIntoView({behavior:'smooth'})` trigger vestibular motion for users who have set reduced-motion preferences. Always gate:
```css
@media (prefers-reduced-motion: no-preference) {
  :root { scroll-behavior: smooth; }
}
```

**Sticky header obscuring focused element.**
A `position: sticky` header that doesn't account for scroll-padding causes the keyboard-focused element to scroll into view but sit hidden behind the header. Fix: `scroll-padding-top: <header-height>` on `<html>`.

**Tiny body text:** body copy below 16px on mobile is cognitively taxing; keep ≥16px. Desktop body: 16–20px.

## Concrete specs & numbers
- **Text contrast (1.4.3 AA):** 4.5:1 normal · 3:1 large (≥24px / ≥18.67px bold). **AAA (1.4.6):** 7:1 / 4.5:1.
- **Non-text contrast (1.4.11):** 3:1 for UI components, icons, focus rings, form borders, chart elements.
- **Focus visible (2.4.7) AA / appearance (2.4.13) AAA:** indicator ≥3:1; AAA requires focus indicator area ≥ perimeter of the unfocused component × 2 CSS px.
- **Target size:** 24×24 CSS px (2.5.8 AA) · 44×44 (2.5.5 AAA / Apple HIG) · 48×48dp (Material).
- **CLS:** ≤0.1 good · 0.1–0.25 needs work · >0.25 poor. Always set `<img>`/`<video>` width+height.
- **Carousel:** ~1% click-through · 84% pick slide 1 · static hero 43.03% vs carousel 1.96% interaction (Notre Dame, 2013).
- **Autoplay:** Chrome audio autoplay blocked since 2018 (Chrome 66 Autoplay Policy). WCAG 1.4.2: audio >3s needs user control.
- **Popups:** trigger at exit-intent or ≥50% scroll depth, not on load. Google mobile interstitial demotion active since 2017.
- **Hamburger desktop:** 3.5s vs 2.4s (NNGroup). Expose top-level items as horizontal nav on ≥tablet.
- **PageSpeed score:** 90–100 Good / 50–89 Needs work / 0–49 Poor.
- **Typefaces:** ≤2–3 per page. **Body text:** ≥16px mobile, 16–20px desktop.
- **Spacing grid:** base-4 or base-8 only.
- **WebAIM Million 2025:** 79.8% of 1M homepages fail contrast (essentially flat from 80.7% in 2024 — progress is stalling).
- **Auto-motion control:** anything moving/auto-updating >5s needs pause/stop/hide (WCAG 2.2.2).
- **Legal:** EU **European Accessibility Act** effective **June 28, 2025** mandates WCAG 2.1 AA for web services; fines up to **€100,000/violation** (Germany).

## Tools & libraries
- **Contrast:** WebAIM Contrast Checker, browser DevTools contrast picker, axe DevTools, Lighthouse, WAVE.
- **Keyboard/focus:** tab through manually first; then axe / Accessibility Insights for automated focus-order checks.
- **CLS / perf:** Lighthouse, PageSpeed Insights, Chrome Web Vitals overlay, `web-vitals` JS lib.
- **Scroll experiences:** native CSS `scroll-snap` and CSS scroll-driven animations — prefer these over JS scroll-hijack libraries.
- **When NOT to reach for a library:** carousels, scroll-jack plugins, and custom modal libs that don't trap focus / support Escape — most accessibility regressions enter through these. If a component lib doesn't pass keyboard + contrast checks out of the box, don't adopt it.

## Do / Don't
**Do**
- Set explicit `width`/`height` (or `aspect-ratio`) on every image and video.
- Give every input a persistent visible `<label>`.
- Replace removed outlines with a 3:1 `:focus-visible` ring.
- Pair every icon with a text label.
- Derive all spacing from a 4/8px scale; limit to ≤2–3 typefaces.
- Keep the footer reachable; pair infinite scroll with "Load more."
- Test the whole UI keyboard-only and at every interactive state.

**Don't**
- Don't `outline: none` without a replacement.
- Don't auto-rotate carousels or autoplay audio.
- Don't fire modals/popups on page load for first-time visitors.
- Don't hijack native scroll with JS.
- Don't use placeholder as the only label.
- Don't disable a button without saying why.
- Don't use color as the only way to convey meaning.
- Don't let tables overflow the viewport.
- Don't use arbitrary pixel spacing (`163px`, `1.58px` borders).

## Common pitfalls & how to avoid them
- **"It passes contrast in the design file" but fails in build** — recheck in the real DOM at every state (hover/focus/disabled/error); brand buttons and overlays often regress.
- **Custom focus ring invisible on dark mode** — set ring color per theme; verify 3:1 against *both* backgrounds.
- **Modal closes but focus is lost** — trap focus while open, restore to the trigger on close, support Escape.
- **Infinite scroll "works with a mouse"** — but a keyboard user can never reach the footer; ship "Load more."
- **`font-display: swap` causes a visible jump** — switch to `optional`, or add `size-adjust`/local fallback metrics to minimize the swap shift.
- **Responsive table scrolls the whole page** instead of just the table — the overflow container is missing or on the wrong element; put `overflow-x:auto` on a tight wrapper.
- **Disabled button "looks enabled"** — ensure the disabled state is visually distinct *and* explained.
- **Hamburger on desktop** — expose top-level nav as a tab bar ≥ tablet; reserve the hamburger for mobile.
- **Sticky header covers focused element** — add `scroll-padding-top: <header-height>` to `<html>` so scroll-into-view for keyboard focus clears the header.
- **`scroll-behavior: smooth` ignores reduced-motion** — gate it: `@media (prefers-reduced-motion: no-preference) { :root { scroll-behavior: smooth; } }`.

## What real users complain about
- "Notification prompt, email popup, sidebar ads, banner ad — you have to fight through 4 layers to read a single sentence." (interstitial pileup)
- "E-commerce sites with delivery-policy links in the footer you can never reach because new stuff always loads." (infinite-scroll footer trap)
- Autoplaying video with sound that starts before the page finishes loading — top driver of immediate back-button presses.
- Gray-on-white body text that's "impossible to read" in sunlight or on cheap screens.
- Disabled "Submit" with no hint about what's missing — users abandon rather than hunt for the unmet field.
- Tiny mobile tap targets where the wrong link fires.
- CAPTCHAs that are "harder on mobile" and never accept the first answer.
See [What Real Users Complain About](./real-user-complaints.md) for the full synthesis.

## Senior-level checklist
- [ ] All body text ≥4.5:1, large text ≥3:1; UI borders/icons/focus ≥3:1 — verified at default/hover/focus/active/disabled/error.
- [ ] Every interactive target ≥24×24 CSS px (44×44 for primary actions).
- [ ] No `outline:none` without a visible `:focus-visible` replacement at ≥3:1.
- [ ] Keyboard-only pass completes: no traps, logical focus order, footer reachable.
- [ ] No auto-rotating carousels; any motion >5s has pause/stop/hide; `prefers-reduced-motion` respected.
- [ ] No JS scroll hijacking; controlled scroll uses `scroll-snap`.
- [ ] No audio autoplay; decorative video is silent + motion-aware + has a pause control.
- [ ] No popup on page load for first-time visitors; modals trap focus, support Escape, have a 44px visible close.
- [ ] Infinite scroll has a "Load more" alternative and an ARIA live region.
- [ ] Every input has a persistent visible `<label>`; placeholders are hints only.
- [ ] Disabled buttons explain the prerequisite (or are enabled and error on click).
- [ ] Every nav icon has a visible text label; desktop nav isn't hamburger-only.
- [ ] ≤2–3 typefaces; color is never the sole signal (icon/underline/text backup present).
- [ ] All spacing on a 4/8px scale; no arbitrary pixel or sub-pixel borders.
- [ ] CLS ≤0.1: images/videos sized, fonts have `font-display`, injected UI reserves space.
- [ ] Tables never overflow the viewport (contained scroll or stacked at ≤600px).
- [ ] Body text ≥16px on mobile.
- [ ] Sticky header: `scroll-padding-top` set to header height so focused elements aren't hidden behind it.
- [ ] `scroll-behavior: smooth` (CSS or JS) gated behind `prefers-reduced-motion: no-preference`.
- [ ] Filter/option lists with >8 items use a panel with "Show more" — not a tiny inline scroll container.
- [ ] No forced account creation before checkout; guest path exists.

## Sources
- WCAG 2.2 (W3C): https://www.w3.org/TR/WCAG22/
- WebAIM Million 2025 annual accessibility report: https://webaim.org/projects/million/
- Apple Human Interface Guidelines: https://developer.apple.com/design/human-interface-guidelines/
- Material Design — Accessibility / target size: https://m3.material.io/foundations/designing/structure
- web.dev — Cumulative Layout Shift (CLS): https://web.dev/articles/cls
- web.dev — Optimize CLS: https://web.dev/articles/optimize-cls
- Nielsen Norman Group — Icon usability: https://www.nngroup.com/articles/icon-usability/
- Baymard Institute — checkout & e-commerce UX research: https://baymard.com/research
- European Accessibility Act (EU 2019/882): https://eur-lex.europa.eu/eli/dir/2019/882/oj
- Chrome Autoplay Policy (2018): https://developer.chrome.com/blog/autoplay/
- Google Search — Intrusive interstitials (2017): https://developers.google.com/search/blog/2016/08/helping-users-easily-access-content-on
- NNGroup — Hamburger menus: https://www.nngroup.com/articles/hamburger-menus/

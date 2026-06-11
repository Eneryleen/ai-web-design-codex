# Accessibility (WCAG 2.2) for Designers & Devs

> How to design and build to WCAG 2.2 Level AA — the legal baseline in the US (Section 508/ADA), EU (EN 301 549, European Accessibility Act), and Canada (AODA). Covers POUR, semantic HTML, keyboard/focus, ARIA discipline, contrast, text alternatives, motion, target sizes, forms, and screen-reader testing. Senior outcome: ship interfaces that are operable by keyboard and screen reader, pass automated *and* manual audits, and don't generate ADA lawsuit exposure.

**Apply when:** every project, from design phase onward (retrofitting costs 10×) · **Related:** [Forms & Input UX](forms-and-input-ux.md) · [Interaction Design & UI States](interaction-design-and-states.md) · [Color Theory & Color Systems](../00-foundations/color-theory-and-systems.md) · [Micro-interactions & Motion](../07-aesthetics-and-trends/micro-interactions-and-motion.md)

## TL;DR — rules at a glance
- **POUR**: Perceivable, Operable, Understandable, Robust. All 4 required for conformance. **AA is the target** (A+AA = 86 success criteria in 2.2).
- **`lang` + `<title>` on every page** — `<html lang="en">` (SC 3.1.1 AA); unique `<title>` per page/route (SC 2.4.2 AA). Two of the most frequently missed AA requirements.
- **Semantic HTML first** — it gives you ~80–90% of accessibility free: roles, keyboard, focus order, headings. `<button>` for actions, `<a href>` for navigation, `<input>`/`<select>` for fields. Never `<div onclick>`.
- **First rule of ARIA: don't.** Pages with ARIA averaged **41% more** detectable errors than pages without (WebAIM Million). ARIA adds semantics, never behavior.
- **Contrast**: 4.5:1 normal text, 3:1 large text (≥24px or ≥18.66px bold) and non-text UI (icons, borders, focus rings).
- **Visible focus always.** 78% of top-1M home pages had focus-indicator issues. Use `:focus-visible`; never bare `outline:none`.
- **Target size 2.5.8 (AA, new in 2.2)**: 24×24 CSS px minimum, or 24px spacing buffer. Inline text links are exempt.
- **Color is never the only signal** — pair with text/icon/pattern (error = red **and** icon **and** message).
- **Every interactive element has an accessible name.** Icon-only button → `aria-label` on the button, `aria-hidden` on the SVG.
- **Forms**: real `<label>` for every field, `<fieldset>`/`<legend>` for groups, native `required`, `aria-invalid` + `aria-describedby` for errors.
- **One `<h1>` per page**, never skip heading levels (no h2→h4). Headings = structure, not styling.
- **Skip link** first focusable element; **focus trap + restore** for modals; move focus on SPA route change.
- **Motion**: author static-first, opt in via `@media (prefers-reduced-motion: no-preference)`; no flashing >3×/sec.
- **Automated tools catch ~30–40%.** Manual keyboard + screen-reader testing is mandatory. Overlays (accessiBe, UserWay) do **not** create compliance — accessiBe reached a consent order with a **$1M civil penalty** with the FTC (Jan 2025).

## POUR — the four principles

| Principle | Means | Primary techniques |
|---|---|---|
| **Perceivable** | Users can perceive the content (sight/sound) | text alternatives, captions, contrast, reflow, resizable text |
| **Operable** | Users can operate the UI (keyboard/pointer/AT) | keyboard access, no traps, visible focus, target size, no flashing, enough time |
| **Understandable** | Content & operation are predictable | clear labels, consistent navigation, error identification/recovery, `lang` on `<html>` (SC 3.1.1), readable text |
| **Robust** | Works across browsers, devices, assistive tech | valid markup, correct roles/names/states, compatible ARIA |

WCAG 2.2 reached W3C Recommendation **Oct 5, 2023**, backwards-compatible with 2.1/2.0. It added 9 success criteria and **removed 4.1.1 Parsing** (browsers now self-correct markup, so it's auto-satisfied). Note: ISO/IEC 40500 covers WCAG 2.0 (2012); WCAG 2.2 has not been published as an ISO standard.

## Semantic HTML — the foundation

Native elements ship with keyboard handling, roles, focusability, and AT support. Recreating them with `<div>` means re-implementing all of it (and you will miss cases).

```html
<!-- DON'T: zero keyboard support, no role, not focusable, no Enter/Space -->
<div class="btn" onclick="save()">Save</div>

<!-- DO: focusable, Enter/Space fire it, announced as "Save, button" -->
<button type="button" onclick="save()">Save</button>

<!-- Navigation is a link, not a button -->
<a href="/pricing">Pricing</a>

<!-- Landmarks orient screen-reader users; let them jump by region -->
<header>…</header>
<nav aria-label="Primary">…</nav>
<main id="main">…</main>   <!-- exactly one <main> -->
<aside>…</aside>
<footer>…</footer>
```

**Two other foundational requirements:**
- **`lang` on `<html>` (SC 3.1.1, AA)** — screen readers use this to select pronunciation engine. `<html lang="en">`. Without it, a French or Japanese SR mispronounces every word. For multilingual segments: `<span lang="fr">Bonjour</span>`.
- **Unique, descriptive `<title>` (SC 2.4.2, AA)** — the first thing a screen reader announces on page load and tab switch. Format: *Page name — Site name*. Every page needs a distinct title; never leave the CMS default "Home | Home."

**Headings define the document outline** (67% of SR users navigate by heading per WebAIM SR Survey). One `<h1>`; never skip levels. Style with CSS — don't pick an `<h4>` because it "looks right."

```html
<h1>Account settings</h1>
  <h2>Profile</h2>
  <h2>Billing</h2>
    <h3>Payment method</h3>   <!-- not <h4> -->
```

## Keyboard navigation

Everything operable by mouse must be operable by keyboard (SC 2.1.1). Tab/Shift+Tab move focus; Enter/Space activate; Esc closes; arrows move within composite widgets (menus, tabs, radio groups).

- **Tab order must be logical** — follows DOM order. Don't reorder visually with CSS in ways that desync from DOM (flex `order`, grid placement) for interactive content.
- **No keyboard traps** (SC 2.1.2) except intentional modal focus traps that Esc releases.
- **`tabindex="0"`** makes a non-native element focusable (only when you must, e.g. a custom widget). **Never `tabindex="1"`+** — positive values hijack natural order globally. `tabindex="-1"` = focusable by script only, not by Tab.
- Test by **tabbing every page top to bottom**: is order logical, is everything reachable, can you escape every component, is focus always visible?

```js
// Roving tabindex for a custom toolbar/menu: one stop in tab order, arrows move within
items.forEach((el, i) => el.tabIndex = i === activeIndex ? 0 : -1);
```

## Visible focus

A focus indicator must be visible whenever something is focused (SC 2.4.7) — **78% of top-1M home pages fail this.**

```css
/* Show a ring for keyboard users, not on mouse click. Modern browsers all support :focus-visible */
:focus-visible {
  outline: 3px solid #1a73e8;   /* ≥3:1 vs adjacent colors (SC 1.4.11) */
  outline-offset: 2px;          /* gap so the ring isn't swallowed by the element */
}

/* If you remove the default outline, you MUST replace it */
button:focus { outline: none; }            /* ❌ never alone */
button:focus-visible {                      /* ✅ */
  outline: 2px solid currentColor;
  outline-offset: 2px;
}
```

- **WCAG 2.2 SC 2.4.11 Focus Not Obscured (Minimum), AA** — a focused element must not be **entirely** hidden by author content (sticky headers/footers, cookie bars). Add `scroll-margin` / sticky-aware scrolling so focus stays visible. 2.4.12 (AAA) requires *no part* hidden.
- **SC 2.4.13 Focus Appearance (AAA)** — focus indicator ≥2px perimeter and 3:1 contrast against both focused/unfocused states.

```css
/* Keep keyboard focus clear of a sticky header */
:target, :focus { scroll-margin-top: 6rem; }
```

## ARIA — and when NOT to use it

**The 5 rules of ARIA:**
1. Use a **native HTML element** if one exists. (A real `<button>` beats `role="button"`.)
2. **Don't override native semantics** unless you truly must (e.g. don't put `role="heading"` on a `<button>`).
3. All interactive ARIA widgets must be **keyboard operable** — ARIA gives semantics, you supply behavior.
4. **Never `aria-hidden="true"` on a focusable element** — focus lands on something the screen reader won't announce.
5. Every interactive element needs an **accessible name**.

> ARIA is a promise to implement behavior the browser would have given you for free. Pages using ARIA averaged **41% more errors** because the promise is usually broken. Reach for it only for widgets HTML can't express (tabs, comboboxes, live regions, custom menus) and verify against **a11ysupport.io** before trusting an attribute.

### Accessible names

Resolution order (first non-empty wins):
**`aria-labelledby` → `aria-label` → native (`<label>`/`alt`/button text) → `title` (last resort, unreliable).**

```html
<!-- Icon-only button: name on the button, hide the glyph -->
<button type="button" aria-label="Close">
  <svg aria-hidden="true" focusable="false"><!-- × --></svg>
</button>

<!-- Logo link describes its FUNCTION, not appearance -->
<a href="/"><img src="logo.svg" alt="Acme — Home"></a>
```

- **Don't add the role to the name**: `aria-label="Close"`, not `"Close button"` — the SR already says "button."
- **Never set both** `aria-labelledby` and `aria-label` on one element — `aria-labelledby` always wins; the `aria-label` is dead config.
- **`aria-labelledby` pointing to a missing/renamed ID produces NO name** — a frequent regression after refactors. Verify IDs exist.
- **`aria-label` is not machine-translated** by Google Translate; for i18n sites prefer **visually-hidden real text** so translation and SRs both get it:

```css
.sr-only {            /* visible to SR, invisible on screen, translatable */
  position: absolute; width: 1px; height: 1px;
  padding: 0; margin: -1px; overflow: hidden;
  clip: rect(0 0 0 0); clip-path: inset(50%); white-space: nowrap; border: 0;
}
```

### Live regions
```html
<div aria-live="polite" id="status"></div>     <!-- non-urgent: validation, search hints -->
<div aria-live="assertive" role="alert"></div>  <!-- critical, interrupts: blocking error -->
```
Use `polite` by default; `assertive` only for things that must interrupt. The region must exist in the DOM **before** you inject text, or it may not announce.

## Color contrast

| Content | AA | AAA |
|---|---|---|
| Normal text (<24px / <18.66px bold) | **4.5:1** | 7:1 |
| Large text (≥24px / ≥18.66px bold) | **3:1** | 4.5:1 |
| Non-text UI (icons, input borders, focus rings, chart colors) | **3:1** (SC 1.4.11) | — |

Worked examples on white `#fff`: `#767676` = 4.5:1 (just passes AA) · `#333` = 12.6:1 (AAA) · black = 21:1 · `#999` = 2.8:1 (**fails**) · `#B3B3B3` = 2.1:1 (**fails**). Placeholder-gray text usually fails — don't rely on it for real content.

**Color alone never conveys meaning** (SC 1.4.1). Required-field asterisks, error states, links in body text, chart series, status dots — all need a second cue (text, icon, underline, pattern, shape).

```css
/* Underline body links so they're distinguishable without color */
.prose a { text-decoration: underline; }
```

**Forced colors / Windows High Contrast Mode**: When users enable Windows High Contrast or `forced-colors: active`, the OS overrides your palette with system colors. Custom components that draw borders via `box-shadow` or `outline` styled with explicit colors may become invisible. Use the `forced-colors` media query to restore critical UI cues:
```css
@media (forced-colors: active) {
  .custom-checkbox { border: 2px solid ButtonText; }
  .focus-ring:focus-visible { outline: 3px solid Highlight; }
}
```
Don't test only in the browser — toggle the actual Windows High Contrast theme and verify all interactive states are still visible.

## Text alternatives

```html
<img src="chart.png" alt="Revenue up 24% from Q1 to Q2, $1.2M to $1.5M">  <!-- informative: describe the info -->
<img src="divider.svg" alt="">     <!-- decorative: EMPTY alt = SR skips it -->
<img src="cart.svg" alt="Items in cart"> <!-- functional: describe the FUNCTION -->
```
- **`alt=""` (empty, present)** vs **omitting `alt`**: omitting makes the SR read the filename. Empty alt = skip entirely. Always include `alt` on `<img>`.
- Functional images: describe **purpose**, not pixels. A magnifier icon that submits search → `alt="Search"`, not `"magnifying glass"`.
- Complex images (charts/diagrams): short `alt` + a longer text equivalent nearby (caption, adjacent prose, or `aria-describedby`).
- `<svg>` used as an image: `role="img"` + `<title>` (or `aria-label`). Decorative inline SVG: `aria-hidden="true"`.
- Media: captions for video (1.2.2), audio described or transcript provided; autoplay audio must stop after 3s or offer pause/stop/volume (SC 1.4.2).

## Motion & `prefers-reduced-motion`

Vestibular disorders (and related conditions like migraine and epilepsy) are common; parallax, large transforms, and auto-motion trigger dizziness, nausea, and disorientation for many users. **Flashing must not exceed 3×/sec** (SC 2.3.1).

**Defensive pattern (preferred): static-first, opt into motion.** Users who prefer reduced motion never even parse the animation CSS.
```css
.card { opacity: 1; transform: none; }   /* final/static state by default */

@media (prefers-reduced-motion: no-preference) {
  .card { animation: fade-up 0.4s ease; }
}
```

**Nuclear reset for legacy codebases** (blanket-kill existing animations):
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

**JS / GSAP / scroll libraries** — honor the same preference and listen for changes:
```js
const mq = matchMedia('(prefers-reduced-motion: reduce)');
function apply() { mq.matches ? timeline.pause(0) : timeline.play(); }
mq.addEventListener('change', apply); apply();
```
Test without changing OS settings: Chrome DevTools → Rendering → *Emulate CSS media feature prefers-reduced-motion*.

**Related media queries (senior awareness)**:
- `@media (prefers-contrast: more)` — user requests higher contrast; increase border weights, contrast ratios.
- `@media (forced-colors: active)` — Windows High Contrast; see Color Contrast section.
- `@media (prefers-color-scheme: dark)` — not a WCAG requirement but AA-aligned: ensure contrast ratios hold in dark mode; dark-mode tokens aren't just color inversions.

## Target sizes (SC 2.5.8, new in WCAG 2.2, AA)

**24×24 CSS px minimum** for pointer targets — OR a 24px-diameter spacing buffer so adjacent targets don't overlap. **Inline text links are exempt** (constrained by line height); standalone buttons and icon controls are **not**.

Platform enhanced targets (SC 2.5.5 AAA = 44×44): Apple HIG **44pt**, Material **48dp**, Fluent **44px**. Default browser checkboxes/radios are ~13–16px and **fail 2.5.8** — pad or resize them.

```css
.icon-btn {           /* hit area meets 24px even if the glyph is smaller */
  min-width: 24px; min-height: 24px;
  display: inline-grid; place-items: center;
  padding: 6px;       /* expands the clickable region */
}
/* Native checkbox is ~16px; enlarge the control + its hit area */
input[type="checkbox"] { width: 24px; height: 24px; }
```
Usability research consistently shows small targets significantly raise error rates for users with motor impairments (tremors, spasticity, limited dexterity) — 24px is the floor, not the goal; aim for 44px on touch interfaces.

## Accessible forms

- **A real `<label for>` (or wrapping label) for every field.** Placeholders are not labels — they vanish on input and fail contrast.
- **Group related controls** with `<fieldset>` + `<legend>` (the only semantic grouping for radios/checkboxes).
- **Native `required`** (not `aria-required`, which is only for custom widgets) — it also blocks submission.
- **Errors**: `aria-invalid="true"` on the field, `aria-describedby` to the message, and announce dynamically via a live region.
- **WCAG 2.2 form-relevant additions**: 3.3.7 **Redundant Entry** (A) — auto-fill/allow reuse of info already given in the same flow; 3.3.8 **Accessible Authentication** (AA) — no cognitive-function test (CAPTCHA/puzzle/transcription) without an alternative; 3.2.6 **Consistent Help** (A) — help in the same relative location across pages.

```html
<form novalidate>
  <fieldset>
    <legend>Shipping speed</legend>
    <label><input type="radio" name="speed" value="std" required> Standard</label>
    <label><input type="radio" name="speed" value="exp"> Express</label>
  </fieldset>

  <label for="email">Email <span aria-hidden="true">*</span></label>
  <input id="email" name="email" type="email" required
         autocomplete="email"
         aria-invalid="true" aria-describedby="email-err">
  <p id="email-err" class="error">
    <svg aria-hidden="true" focusable="false">…</svg>   <!-- icon: color isn't the only cue -->
    Enter a valid email address.
  </p>

  <div aria-live="polite" class="sr-only" id="form-status"></div>
</form>
```
Use correct `type` (`email`, `tel`, `url`, `number`) and `autocomplete` tokens — better keyboards on mobile and required by SC 1.3.5 Identify Input Purpose. Key tokens: `name`, `given-name`, `family-name`, `email`, `tel`, `street-address`, `postal-code`, `country`, `bday`, `new-password`, `current-password`. Wrong or missing tokens make password managers and assistive tech fail; `autocomplete="off"` on personal-data fields is an accessibility blocker.

## Skip links & focus management

**Skip link** = first focusable element on every page; hidden at rest, visible on focus.
```html
<a href="#main" class="skip-link">Skip to main content</a>
…
<main id="main" tabindex="-1">…</main>
```
```css
/* Use translate instead of left:-9999px — avoids horizontal scrollbar on RTL/narrow viewports */
.skip-link {
  position: absolute; top: 8px; left: 8px;
  transform: translateY(-200%); /* off-screen */
  z-index: 1000; padding: 8px 12px;
  background: #000; color: #fff;
  transition: transform 0.1s;
}
.skip-link:focus { transform: translateY(0); }
```

**Modal dialogs**: on open, move focus inside; trap Tab/Shift+Tab within; Esc closes; on close, **return focus to the trigger**. Mark background `inert`.
```html
<dialog id="dlg"></dialog>           <!-- native <dialog>.showModal() handles focus trap -->
<main inert>…</main>                  <!-- explicit inert on siblings for custom modals (native dialog sets inert in modern Chrome/Firefox but not universally) -->
```

**SPA route changes**: SRs don't announce client-side navigations. After route change, **programmatically focus the new `<h1>`** (or a skip target) and/or announce the new page title via a `polite` live region.
```js
afterRouteChange(() => {
  const h1 = document.querySelector('main h1');
  h1.setAttribute('tabindex', '-1'); h1.focus();
  liveRegion.textContent = `${document.title} loaded`;
});
```

## WCAG 2.2 new criteria — quick reference

| SC | Level | Requirement |
|---|---|---|
| 2.4.11 Focus Not Obscured (Min) | AA | Focused component not *entirely* hidden by author content (sticky bars) |
| 2.4.12 Focus Not Obscured (Enh) | AAA | No part of focused component hidden |
| 2.4.13 Focus Appearance | AAA | ≥2px perimeter, 3:1 contrast focused vs unfocused |
| 2.5.7 Dragging Movements | AA | Drag actions also doable via single pointer (tap/click), unless essential |
| 2.5.8 Target Size (Min) | AA | 24×24 CSS px, or 24px spacing |
| 3.2.6 Consistent Help | A | Help in same relative order across pages |
| 3.3.7 Redundant Entry | A | Don't re-ask info already given in the same process |
| 3.3.8 Accessible Authentication (Min) | AA | No cognitive-function test without an alternative |
| 3.3.9 Accessible Authentication (Enh) | AAA | No cognitive-function test at all |
| — 4.1.1 Parsing | — | **Removed** in 2.2 (auto-satisfied) |

## Concrete specs & numbers
- **Contrast**: 4.5:1 normal / 3:1 large (≥24px or ≥18.66px bold) / 3:1 non-text UI. AAA: 7:1 and 4.5:1.
- **Target size**: 24×24 CSS px (2.5.8 AA) or 24px buffer; 44×44 (2.5.5 AAA). Apple 44pt, Material 48dp, Fluent 44px. Default checkbox ~13–16px → fails.
- **Reflow (1.4.10)**: no horizontal scroll at **320 CSS px** wide (= 400% zoom on 1280px). Build mobile-first; avoid fixed widths.
- **Text spacing (1.4.12)** must survive user overrides: line-height ≥ 1.5×, paragraph spacing ≥ 2×, letter-spacing ≥ 0.12×, word-spacing ≥ 0.16× font size. Don't clip content when users force these.
- **Line length**: ≤ ~80 characters for readability.
- **Flashing (2.3.1)**: ≤ 3 flashes/second.
- **Audio autoplay (1.4.2)**: auto-stop ≤ 3s, or provide pause/stop/volume control.
- **Timeouts (2.2.1)**: warn + allow extend; data-loss-free interpretation ≥ 20h for authenticated sessions.
- **Conformance**: A + AA = **86 success criteria** in WCAG 2.2. AA is the legal floor everywhere that matters.

## Screen reader testing

Automated tools catch ~30–40% of WCAG failures. Manual SR testing is non-negotiable before shipping.

**Minimum coverage**: NVDA + Chrome (Windows), VoiceOver + Safari (iOS). Add JAWS + Chrome for enterprise products.

**Core NVDA/JAWS keyboard commands**

| Action | NVDA | JAWS |
|---|---|---|
| Start reading | Insert+↓ | Insert+↓ |
| Next heading | H | H |
| Next landmark | D | R |
| Next link | K | Tab or U |
| Next button | B | B |
| Next form field | F | F |
| List all headings | Insert+F7 | Insert+F6 |
| List all links | Insert+F7 | Insert+F7 |
| Read current element | Insert+Tab (current) | Insert+Tab |
| Click/activate | Enter or Space | Enter or Space |

**VoiceOver + Safari (macOS)**: VO = Control+Option. VO+U opens the Rotor (navigate by heading/link/landmark). VO+Right/Left reads item by item.

**VoiceOver + Safari (iOS)**: Swipe right = next element, double-tap = activate, three-finger swipe = scroll. Use the Rotor (two-finger twist) to change navigation mode.

**What to verify on every key flow:**
1. **Page announced on load** — title spoken, correct language.
2. **Landmarks reachable** — can jump to `<main>`, `<nav>` without reading every element.
3. **Heading structure makes sense** — list headings and verify they form a logical outline.
4. **Every control has a name and role** — buttons say "Close, button", not "button" or a filename.
5. **Form fields** — label announced when focus lands; errors announced on submit; `aria-invalid` changes cause SR to report "invalid entry."
6. **Dynamic content** — live regions announce status messages; modals announce their title on open; route changes announce new page title.
7. **Tab order** — tab through the entire page; every interactive element reachable; focus never disappears silently.
8. **Images** — informative images describe content; decorative images are skipped entirely; functional images describe their action.

**Common SR bugs to hunt for:**
- Button reads filename (`img-close-16.png`) — SVG not aria-hidden, button has no label.
- Field reads "edit, blank" — label missing or not associated.
- Modal content read before focus moves inside — focus management missing.
- Route change: SR reads stale page title or nothing — live region/focus not updated.
- Dynamic error text not announced — live region not pre-rendered or using wrong `aria-live` value.

## Tools & libraries
- **axe-core** (Deque) — engine behind DevTools, `jest-axe`, Playwright, Cypress. CI gate. ~40% coverage. **Lighthouse** uses the same engine.
- **WAVE** (WebAIM) — visual overlay of errors/alerts/structure; shows contrast ratios in the Details panel but can misread text rendered over images/gradients.
- **Pa11y CI** — headless CLI for pipelines (axe or HTMLCS).
- **Colour Contrast Analyser (TPGi)** / **Stark** (Figma/Sketch/XD) / **WebAIM Contrast Checker** — sample/verify contrast at design time and on real pixels.
- **Screen readers — test in this priority**: **NVDA + Chrome** (free, Win; most common combo) and **JAWS + Chrome** (enterprise default) first; **VoiceOver + Safari** on macOS/iOS (essential — VoiceOver behaves differently from NVDA/JAWS and iOS is a major AT platform); **TalkBack + Chrome** for Android.
- **ANDI** (SSA bookmarklet) / **Chrome DevTools Accessibility tree** / **a11ysupport.io** (ARIA support matrix) — verify computed names, roles, states, and whether a pattern is actually supported.
- **When NOT to use**: never ship an **accessibility overlay** (accessiBe, UserWay) as your compliance strategy — the EU explicitly advises against them and accessiBe reached a consent order with a **$1M civil penalty** with the FTC (Jan 2025). Automated tools alone are insufficient (~30–40% coverage) — they don't replace keyboard + SR testing.

## Do / Don't
**Do**
- Set `lang` on `<html>` and a unique `<title>` on every page — these are often missed and are AA requirements.
- Build accessibility in from the design phase; treat it as ongoing, not a one-time audit.
- Reach for native HTML; add ARIA only for widgets HTML can't express, verified on a11ysupport.io.
- Give every interactive element a clear accessible name; verify in the DevTools accessibility tree.
- Use `:focus-visible`, keep focus visible and unobscured, restore focus after modals/route changes.
- Pair every color signal with text/icon/pattern; check contrast for text *and* non-text UI.
- Author static-first and opt into motion; honor `prefers-reduced-motion` in CSS and JS.
- Test by keyboard top-to-bottom and with NVDA/JAWS on real pages before shipping.

**Don't**
- Don't omit `lang` from `<html>` or leave `<title>` as a generic default.
- Don't use `<div>`/`<span>` for buttons, links, or inputs.
- Don't `outline:none` without a replacement; don't rely on `:focus` for custom rings.
- Don't add a role to `aria-label` ("Close button"); don't set both `aria-label` and `aria-labelledby`.
- Don't `aria-hidden` a focusable element; don't point `aria-labelledby` at a non-existent ID.
- Don't use placeholder as a label; don't convey state by color alone.
- Don't use positive `tabindex`; don't skip heading levels or ship multiple `<h1>`s.
- Don't autoplay sound/parallax without controls; don't rely on overlays or automated scans for compliance.

## Common pitfalls & how to avoid them
- **`aria-labelledby` regression after refactor** — renamed IDs silently zero out the accessible name. Lint/test computed names in CI.
- **Bare `outline:none`** for "design" — removes the only focus cue. Replace with a 3:1, ≥2px `:focus-visible` ring.
- **`aria-label` and i18n** — not translated by Google Translate; use `.sr-only` real text instead.
- **Custom checkbox/radio/`<div>` widgets** — drop keyboard support and roles. Style native inputs or use a vetted headless lib; never hand-roll without ARIA APG patterns.
- **Sticky header eats focus** (2.4.11) — add `scroll-margin-top`/sticky-aware scroll so the focused element isn't covered.
- **Placeholder-only fields** — disappear on type, fail contrast, lost for SR/zoom users. Always a persistent `<label>`.
- **Tiny icon buttons** — 16px hit areas fail 2.5.8; expand to 24px with padding.
- **Heading-as-style** — picking `h3` because it looks right breaks the outline. Style with CSS, keep order semantic.
- **Live region added with its text at once** — may not announce. Render the empty region first, inject text after.
- **Decorative image without `alt`** — SR reads the filename; use `alt=""`.
- **Missing `lang` on `<html>`** — SR picks wrong pronunciation engine; affects every word on the page.
- **Generic or duplicate `<title>`** — SR users rely on page titles to orient; "Home | Home" or a React app with a static "App" title fails SC 2.4.2.

## What real users complain about
- **"Tab and I'm lost"** — no visible focus ring (78% of home pages), so keyboard users can't tell where they are.
- **"The screen reader just says 'button, button, button'"** — icon-only controls with no accessible name.
- **"Overlay made it worse"** — accessiBe/UserWay widgets break native AT behavior and shortcuts; users actively avoid sites running them (and they don't confer compliance — FTC consent order, $1M civil penalty).
- **"Parallax site made me nauseous"** — vestibular triggers from scroll-jacking/parallax that ignore `prefers-reduced-motion`.
- **"Couldn't find my error"** — error shown only as red border, no message, no focus moved to it.
- **"CAPTCHA locked me out"** — image/audio puzzles with no accessible alternative (now a 2.2 AA failure, 3.3.8).
- **"Modal trapped me"** — focus not trapped (Tab escapes behind the dialog) or not returned to the trigger on close.
- **"Site re-asked everything"** — multi-step flows that don't preserve previously entered data (3.3.7).

## Senior-level checklist
- [ ] Keyboard-only pass on every page: logical order, all controls reachable, focus always visible & unobscured, no unintended traps, Esc closes overlays.
- [ ] `<html lang="...">` set correctly; `<title>` unique and descriptive on every page/route.
- [ ] One `<h1>`; heading levels not skipped; landmarks present (`main`/`nav`/`header`/`footer`); `<nav>`s labeled if multiple.
- [ ] Skip link is first focusable, visible on focus, targets `<main>`.
- [ ] Every interactive element has a verified accessible name (checked in the accessibility tree, not assumed).
- [ ] ARIA: only where native HTML can't do it; no role on labels, no `aria-hidden` on focusable, no dual label sources, IDs resolve.
- [ ] Contrast: text 4.5:1/3:1, non-text UI & focus ring ≥3:1; no meaning by color alone.
- [ ] All `<img>` have correct `alt` (informative/decorative `""`/functional); media captioned/transcribed; no uncontrolled autoplay audio.
- [ ] Targets ≥24×24px (or 24px buffer); native checkbox/radio resized.
- [ ] Forms: persistent labels, `fieldset`/`legend` groups, native `required`, `aria-invalid` + `aria-describedby`, live-region announcements, correct `type`/`autocomplete`; accessible auth alternative.
- [ ] Reflow at 320px / 400% zoom with no horizontal scroll; survives 1.4.12 text-spacing overrides.
- [ ] `prefers-reduced-motion` honored in CSS and JS; no flashing >3×/sec.
- [ ] Modals trap + restore focus and use `inert`/`<dialog>`; SPA route changes move focus and announce.
- [ ] Automated scan clean (axe/Lighthouse) **and** a manual NVDA/JAWS + VoiceOver pass on key flows.
- [ ] No accessibility overlay used as the compliance mechanism.

## Sources
- WCAG 2.2 (W3C Recommendation): https://www.w3.org/TR/WCAG22/
- Understanding WCAG 2.2: https://www.w3.org/WAI/WCAG22/Understanding/
- ARIA Authoring Practices Guide (APG): https://www.w3.org/WAI/ARIA/apg/
- Using ARIA (incl. "rules of ARIA"): https://www.w3.org/TR/using-aria/
- WebAIM Million annual report: https://webaim.org/projects/million/
- WebAIM Screen Reader User Survey: https://webaim.org/projects/screenreadersurvey/
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- MDN — prefers-reduced-motion: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- MDN — :focus-visible: https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible
- Deque axe-core: https://github.com/dequelabs/axe-core
- a11ysupport.io (ARIA support matrix): https://a11ysupport.io/
- EN 301 549 (EU harmonized standard): https://www.etsi.org/standards
- FTC action against accessiBe (Jan 2025): https://www.ftc.gov/news-events/news/press-releases

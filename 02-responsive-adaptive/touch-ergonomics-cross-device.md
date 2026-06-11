# Touch Ergonomics & Cross-Device Design

> How to design and build interfaces that work for fingers and thumbs, not just mouse pointers — sizing touch targets in physical space, placing actions in thumb-reachable zones, detecting *input type* (not screen size), respecting notches/safe areas, and keeping scroll smooth on cheap hardware. Master this to ship one codebase that feels native on a $90 Android phone, an iPad, a touchscreen laptop, and a desktop with a mouse.

**Apply when:** designing or building any page or component that real people touch — i.e., almost everything in 2026+. · **Related:** [Responsive & Fluid Design](responsive-and-fluid-design.md) · [Breakpoints, Devices & Mobile-First](breakpoints-devices-mobile-first.md) · [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md)

## TL;DR — rules at a glance
- **Touch-first baseline, enhance for mouse.** Design for `pointer: coarse` / `hover: none`, then layer hover + precision on top. Never gate functionality behind hover.
- **Detect input, never screen size.** A 1440px touchscreen TV is coarse; a 13" Surface with stylus is fine. Use `@media (pointer:…)` / `(hover:…)`, not `min-width`.
- **Minimum touch target: 24×24 CSS px (WCAG 2.5.8 AA, hard floor), 44×44 (WCAG 2.5.5 AAA / Apple HIG), 48×48dp (Material).** Aim for 48 on primary actions.
- **Physical size beats pixels.** Goal is ~9mm (~1cm) of real screen. Pixel values are proxies that drift across PPI. Sub-7mm targets spike error rates.
- **Min 8dp gap between adjacent targets.** Small visual chips are fine if their *hit area* is padded out to 44–48px.
- **Thumb zone is the layout constraint.** ~49% hold phones one-handed; ~75% of interactions are thumb-driven. Put primary actions lower-center; never in top corners.
- **Safe areas are mandatory.** Every fixed/sticky element on mobile must use `env(safe-area-inset-*)`, and the viewport meta must include `viewport-fit=cover` or the insets stay `0`.
- **`touch-action: manipulation`** on interactive elements kills the ~350ms tap delay and double-tap zoom — without disabling pinch-zoom.
- **Passive listeners or jank.** `addEventListener('touchstart', fn, { passive: true })`. Never block the scroll thread.
- **Hover is never the only affordance.** Anything revealed on `:hover` must also appear on `:focus-within` and be reachable by touch.
- **Semantic HTML is the cheapest touch a11y.** `<button>`/`<a>`/`<input>` give focus, key activation, roles, and tap highlight for free. `<div onClick>` recreates all of that — usually wrong.
- **Gestures are invisible.** Any unique gesture needs a visible non-gesture fallback. Don't fight the OS edge-swipe-back.
- **Sticky header ≤ 50px on mobile.** Footer CTAs: `padding-bottom: calc(1rem + env(safe-area-inset-bottom))`.
- **Test at 20× CPU throttle** (Chrome DevTools) to feel a budget Android device.
- **Use `dvh`/`svh` instead of `vh` for full-screen layouts.** `100vh` is permanently broken on iOS Safari (excludes browser chrome). `100dvh` = dynamic (shrinks with keyboard), `100svh` = small (always excludes chrome), `100lvh` = large (ignores chrome). Baseline 2023+ (Chrome 108, Safari 15.4, Firefox 101).
- **Respect `prefers-reduced-motion`.** Motion-heavy transitions and gesture cues must degrade gracefully — many mobile users trigger vestibular symptoms from parallax and rapid translate animations.
- **Use Pointer Events, not Touch Events, for custom gesture logic.** `pointerdown`/`pointermove`/`pointerup` unifies mouse, touch, and pen. Touch Events are legacy; Pointer Events are the 2026 baseline.

---

## 1. Touch targets: size in physical space, not pixels

A pixel is not a unit of human anatomy. The same 44px is ~11.7mm on an old 96dpi screen and ~7mm on a dense phone. The real goal is **fingertip-sized contact area**, so treat pixel specs as *minimum proxies* and verify physical size on dense displays.

**The standards (memorize these):**

| Standard | Minimum | Notes |
|---|---|---|
| WCAG 2.5.8 (AA, WCAG 2.2) | **24×24 CSS px** | Hard legal/audit floor. Exception if adequate spacing compensates. |
| WCAG 2.5.5 (AAA) | **44×44 CSS px** | Enhanced best practice. |
| Apple HIG (iOS/iPadOS) | **44×44 pt** | visionOS raises to **60pt (~80px)** — lower spatial pointing precision. |
| Material Design 3 | **48×48dp** (~9mm) | Visual element may be 24dp with **invisible 12dp padding** all sides. |
| Microsoft Fluent (web) | **44×44px** | Matches Apple HIG; for Windows/UWP the native unit is effective pixels (epx) with the same 44 floor. |

**Why 44 still isn't generous:** Parhi et al. (2006, MobileHCI) measured average index fingertip width at **1.6–2cm** (≈45–57px @96dpi); thumb pad roughly **2.2cm (~75px)**. Both *exceed* 44px. So 44px targets still produce thumb errors; reserve them for secondary controls and use 48px+ for anything primary or thumb-operated.

**Fitts's Law in one sentence:** acquisition time rises logarithmically as target size shrinks and distance grows. Below ~7mm physical, error rates climb sharply; WCAG 2.5.8 rationale documents significantly higher error rates for users with motor impairments at sub-minimum sizes. Bigger + closer + edge-anchored = faster, fewer misses.

**The pattern that satisfies all of this** — small visual, large hit area:

```css
/* Small icon button that still hits 48px touch target */
.icon-btn {
  display: inline-grid;
  place-items: center;
  inline-size: 1.5rem;      /* 24px visible glyph */
  block-size: 1.5rem;
  padding: 0.75rem;         /* expands hit area to 48px without growing the icon */
  /* prefer this over a tiny clickable <svg> */
}

/* Or extend the hit area invisibly without affecting layout */
.compact-link { position: relative; }
.compact-link::after {
  content: "";
  position: absolute;
  inset: -10px;             /* 20px taller/wider invisible hit zone */
}
```

**Spacing:** Material Design 3 recommends **≥ 8dp** between adjacent targets. When targets sit edge-to-edge (toolbars), padded hit areas matter more than visible gaps — the gap between *hit areas* must be non-zero even if visuals overlap.

## 2. Input-agnostic design: detect the pointer, not the breakpoint

The single most common touch bug is inferring "touch" from screen width. It's wrong in both directions: 768px ≠ touch, and a 2000px Surface Studio *is* touch. Use CSS interaction media features — they describe the actual hardware.

| Query | Means | Typical device |
|---|---|---|
| `(pointer: coarse)` | primary pointer is imprecise | phone, touchscreen |
| `(pointer: fine)` | primary pointer is precise | mouse, trackpad, stylus |
| `(hover: hover)` | primary pointer can hover | desktop, laptop |
| `(hover: none)` | cannot hover | touchscreen |
| `(any-pointer: coarse)` | *any* input is coarse | hybrid laptop w/ touchscreen |
| `(any-hover: hover)` | *any* input can hover | iPad + trackpad |

**Combinations worth knowing:** `coarse + hover:none` = phones/touchscreens. `fine + hover:hover` = desktop/laptop. **`coarse + hover:hover` = smart TVs / game consoles** (D-pad cursor that hovers but is imprecise).

```css
/* Baseline = touch. Enlarge controls for coarse pointers. */
@media (any-pointer: coarse) {
  /* any-pointer, not pointer: a touchscreen laptop with a mouse still gets big targets */
  button, [role="button"] { min-block-size: 48px; }
  input[type="checkbox"], input[type="radio"] { inline-size: 26px; block-size: 26px; }
}

/* Hover/precision affordances only where they actually work */
@media (hover: hover) and (pointer: fine) {
  .card:hover .card__actions { opacity: 1; }
}
```

**Rule:** use `any-pointer`/`any-hover` to *add capability for coarse input* (enlarge), and `hover:hover`+`pointer:fine` to *add precision affordances* (reveal-on-hover). Never use `min-width` to decide either.

## 3. The hover-less problem

On a touch device, `:hover` fires on tap and **sticks** until the user taps elsewhere — so unguarded hover styles look permanently broken. Worse, content revealed only on hover is *invisible* to touch users.

```css
/* WRONG — tooltip vanishes for touch users, sticks on tap */
.has-tip:hover .tooltip { display: block; }

/* RIGHT — gate hover, and provide keyboard + focus parity */
@media (hover: hover) {
  .has-tip:hover .tooltip { display: block; }
}
.has-tip:focus-within .tooltip { display: block; } /* keyboard + tap-to-focus */
```

**Hard rules:**
- Every hover reveal pairs with `:focus-visible` / `:focus-within`.
- No information or action lives *only* in hover state. Provide a persistent control, a tap-to-toggle, or an always-visible variant on coarse pointers.
- Combine `:hover` and `:focus-visible` selectors so keyboard users get the same treatment.
- **Caveat — `:focus-visible` on touch:** Mobile browsers (iOS Safari, Android Chrome) suppress the focus ring after a pointer tap — `:focus-visible` may not render on touch activation even though the element is focused. This means `:focus-visible` alone is not sufficient as a touch-disclosure mechanism; use `:focus-within` on a parent or an explicit `aria-expanded` toggle pattern instead.

## 4. Thumb zones & ergonomic placement

~75% of mobile interactions are thumb-driven and ~49% of users operate one-handed at some point during a session (Hoober, "How Do Users Really Hold Mobile Devices?", UXmatters 2013, n=~1,333 observed interactions). Map the screen to reach difficulty:

- **Easy (natural thumb arc):** lower ~60% of height, and the side near the holding hand. On a ~6" phone the comfortable zone is the **lower ~40% of width × lower ~60% of height**.
- **Stretch:** center and upper-middle.
- **Hard:** **top-left and top-right corners** — the worst point for one-handed use, and top-left is the classic (bad) home of the back button and modal-close `×`.

**Placement rules:**
- Primary CTAs and primary nav → **bottom**, ideally lower-center. This is why bottom tab bars and bottom sheets win on mobile.
- Bottom navigation: **max 5 tabs.** More → overflow into a bottom sheet or secondary nav.
- Move modal/card dismiss controls to a **bottom button** or full-width footer on large phones; a top-right `×` on a 6.7"+ phone is the hardest reach for right-handed thumbs.
- Destructive actions go *away* from the easy zone (or require confirmation) so they aren't fat-fingered.

## 5. Gestures: powerful, undiscoverable, OS-contested

Gestures have **zero affordance** — nobody sees a swipe. So:

- Any gesture that provides *unique* functionality **must** have a visible non-gesture alternative (a visible swipe hint arrow, or an accessible delete button alongside swipe-to-delete).
- **Don't fight system edge swipes.** Right-to-left / left-edge **swipe-back is an OS gesture on iOS and Android** — the OS intercepts left-edge handlers. There's no learned "swipe-forward," so accidental back has no gesture undo. Keep ~20px clear of screen edges for custom horizontal gestures, or don't bind them.
- **Respect `prefers-reduced-motion`.** Swipe-driven animations and parallax transitions should have a motion-safe variant. Users with vestibular disorders frequently trigger this on mobile.
- Let the browser own what it can. Use `touch-action` to split responsibilities:

```css
/* Horizontal carousel: browser handles vertical scroll, JS handles horizontal swipe */
.carousel { touch-action: pan-y; }

/* Interactive element: kill 350ms delay + double-tap-zoom, KEEP pinch-zoom */
button, a, [role="button"], .tappable { touch-action: manipulation; }

/* DON'T do this on a scroll region — disables scrolling AND pinch-zoom (a11y break) */
/* .scroller { touch-action: none; } */
```

`touch-action: manipulation` tells the browser to handle panning and pinch-zoom natively while disabling double-tap-to-zoom (removing the legacy ~350ms click delay introduced to distinguish single from double taps). It does **not** disable pinch-zoom. In 2026 the delay is already suppressed by `width=device-width` in the viewport meta on all major browsers, but `manipulation` is still best practice as a belt-and-suspenders guarantee. Setting `touch-action: none` on a large scrollable region breaks scrolling and pinch-zoom for low-vision users — never do it.

## 6. Safe areas, notches & the soft keyboard

Modern phones have notches, Dynamic Island, rounded corners, and a home indicator. Fixed/sticky UI placed at the literal edge renders *under* them.

**Two-step activation (both required):**

```html
<!-- 1. Opt in, or env() insets all return 0 -->
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
```

```css
/* 2. Pad fixed/sticky edges with the device insets. Names are case-SENSITIVE. */
.app-footer {
  position: fixed; inset-block-end: 0; inset-inline: 0;
  padding-bottom: calc(1rem + env(safe-area-inset-bottom)); /* clears home indicator */
}
.app-header {
  position: sticky; inset-block-start: 0;
  padding-top: env(safe-area-inset-top);
  max-block-size: 50px; /* keep ≤50px content area on mobile */
}
.full-bleed { padding-inline: env(safe-area-inset-left) env(safe-area-inset-right); }
```

- Insets: `safe-area-inset-top | -right | -bottom | -left`. These are the only `env()` safe-area variables in the spec — there is no `safe-area-max-inset-*` family. For pre-layout estimates, use a fallback: `env(safe-area-inset-bottom, 20px)`.
- **Case matters:** `safe-area-inset-top` works; `SAFE-AREA-INSET-TOP` is ignored.
- **`dvh`/`svh`/`lvh` instead of `vh` for full-height layouts.** `100vh` on iOS Safari has always equaled the viewport height *excluding* the collapsible browser chrome — meaning it overflows when the chrome is visible. Use `100svh` (always shrinks to exclude chrome) or `100dvh` (updates dynamically as chrome shows/hides). Baseline: Chrome 108+, Safari 15.4+, Firefox 101+.

**Soft keyboard:** when the virtual keyboard opens, `window.innerHeight` does *not* reliably shrink. Use the **Visual Viewport API** for layout that must track the keyboard (e.g., a sticky composer bar):

```js
visualViewport.addEventListener('resize', () => {
  const kb = window.innerHeight - visualViewport.height; // ~keyboard height
  document.documentElement.style.setProperty('--kb', `${Math.max(kb, 0)}px`);
});
```

## 7. Sticky elements on mobile

- **Sticky header ≤ 50px content height.** Taller headers eat 15–25% of a portrait phone viewport and measurably raise content-close rates. Combine with safe-area-top padding.
- Avoid stacking multiple stickies (header + tab bar + banner) — they compound to occupy half the screen.
- Sticky footers/CTAs must clear the home indicator with `env(safe-area-inset-bottom)` and must not cover the *last* row of content (add matching scroll padding to the content container).
- Consider hiding the sticky header on scroll-down / revealing on scroll-up to reclaim space — but drive it with the [Scroll-Driven Animations API](#8-performance-on-low-end-devices), not a JS scroll handler. (Anchor: see §8 below.)

## 8. Performance on low-end devices

Most of the world's web traffic is on budget Android hardware. Touch UX *is* performance UX — jank during scroll feels like a broken touch interface.

**Use Pointer Events for custom gesture logic — not Touch Events:**

```js
// Pointer Events = unified mouse + touch + pen, 2026 baseline in all browsers.
// Touch Events (touchstart/touchmove) are legacy; avoid for new code.
el.addEventListener('pointerdown', onStart);
el.addEventListener('pointermove', onMove);
el.addEventListener('pointerup',   onEnd);
el.setPointerCapture(e.pointerId); // keep tracking after pointer leaves element
```

For scroll performance, **passive listeners are still required** even with Pointer Events when listening to `touchmove` or `wheel`:

```js
// Tells the browser the handler won't preventDefault, so it can scroll immediately.
el.addEventListener('touchmove', onMove, { passive: true });
window.addEventListener('wheel',    onWheel, { passive: true });
// Chrome/Firefox default-passive 'touchstart'/'touchmove' on document since ~2017,
// but explicit { passive: true } is still required on element-level listeners.
```

Non-passive `touchmove` on `document`/`body` forces the browser to *wait for JS* before scrolling — severe jank every frame.

**Rules:**
- **Scope listeners tightly** to the specific element; never attach touch/pointer handlers to `document.body` (blocks the GPU scroll thread globally).
- **Never do layout-changing work inside `pointermove`/`scroll`.** Batch into `requestAnimationFrame`.
- **Drop JS parallax on mobile.** It's a leading cause of low-end scroll jank. Replace with CSS-only effects or remove.
- **Prefer the Scroll-Driven Animations API** (`animation-timeline: scroll()` / `view()`, Chrome 116+, Firefox 128+, Safari 18+) — runs **off the main thread**, immune to JS jank, unlike classic scroll handlers that block it.
- **Respect `prefers-reduced-motion`.** Wrap all scroll-driven animations: `@media (prefers-reduced-motion: no-preference)`.
- **Test honestly:** Chrome DevTools → Performance → **CPU 20× slowdown** approximates a budget Android device. Throttle network too (Slow 4G).

```css
/* Off-main-thread scroll progress bar — no JS, no jank */
@keyframes grow { to { transform: scaleX(1); } }

/* Always wrap scroll-driven animations in prefers-reduced-motion */
@media (prefers-reduced-motion: no-preference) {
  .progress {
    transform-origin: left; transform: scaleX(0);
    animation: grow linear; animation-timeline: scroll(root block);
  }
}
```

## 9. Semantic HTML & touch-correct controls

Native elements are the fastest path to correct touch behavior. `<button>`, `<a>`, `<input>` come with keyboard focus, Enter/Space activation, ARIA role, correct cursor, and tap-highlight. A `<div onClick>` has **none** of that — you must manually add `tabindex="0"`, `role="button"`, `onKeyDown` for Enter/Space, and a focus ring, and most implementations miss at least one.

- **Tables with row actions:** put a real `<button>` in a cell. **Do not make the whole `<tr>` clickable** (or `role="button"` on it) — it breaks keyboard nav, text selection, and nested interactive cells.
- **Forms:** set `inputmode` + `autocomplete` on every field. `inputmode="numeric"` shows a number keypad *without* `type="number"`'s spinner/decimal quirks; `inputmode="decimal"` for prices. `autocomplete` surfaces device-store autofill, cutting mobile form friction massively. (See [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md).)

```html
<input inputmode="decimal" autocomplete="off" name="price" />
<input type="email" inputmode="email" autocomplete="email" name="email" />
<input type="tel"   inputmode="tel"   autocomplete="tel"   name="phone" />
```

## Concrete specs & numbers

- **Touch target floors:** 24×24 CSS px (WCAG 2.5.8 AA) · 44×44 (WCAG 2.5.5 AAA / Apple HIG) · 48×48dp (Material) · 44×44px (Fluent/web) · 60pt/~80px (visionOS).
- **Material visual/hit split:** 24dp visible + 12dp invisible padding each side = 48dp hit.
- **Spacing between targets:** ≥ 8dp (Material Design 3).
- **Finger sizes (Parhi et al., MobileHCI 2006):** index fingertip ~1.6–2cm (45–57px @96dpi); thumb pad ~2.2cm (~75px).
- **Physical-size goal:** ~9mm (~1cm). Below ~7mm = elevated errors (documented in WCAG 2.5.8 rationale).
- **Thumb zone (~6" phone):** easy = lower ~40% width × lower ~60% height; top corners hardest.
- **Sticky header:** ≤ 50px content height on mobile.
- **Bottom nav:** ≤ 5 tabs.
- **Tap delay:** ~350ms double-tap-to-zoom delay; suppressed by `width=device-width` viewport meta and additionally by `touch-action: manipulation`. In 2026 this is a non-issue on modern browsers with a correct viewport meta, but `manipulation` remains best practice.
- **Passive listeners:** `{ passive: true }` — universal in all modern browsers.
- **Scroll-Driven Animations API:** Chrome 116+, Firefox 128+, Safari 18+ (off-main-thread).
- **`dvh`/`svh`/`lvh` viewport units:** Chrome 108+, Safari 15.4+, Firefox 101+. Use instead of `vh` for full-height mobile layouts.
- **Low-end simulation:** DevTools CPU 20× slowdown.
- **`touch-action: manipulation`** — disables double-tap-to-zoom, preserves pan and pinch-zoom. Not a literal shorthand alias; behavioral spec only.

## Tools & libraries

- **CSS interaction media features** (`pointer`, `hover`, `any-pointer`, `any-hover`) — native, zero deps. The correct way to branch on input type. Use instead of any JS touch-detection library.
- **`touch-action` property** — native gesture routing. Use `manipulation` on tappables, `pan-y`/`pan-x` on custom swipe containers. Never `none` on scroll regions.
- **`env()` + `safe-area-inset-*`** — native notch/home-indicator handling. Requires `viewport-fit=cover`. No library substitutes for this.
- **`dvh`/`svh`/`lvh` viewport units** — replace `vh` for full-height layouts. `100svh` = always excludes mobile chrome; `100dvh` = updates dynamically. Chrome 108+, Safari 15.4+, Firefox 101+.
- **Pointer Events API** (`pointerdown/pointermove/pointerup/setPointerCapture`) — unified mouse+touch+pen event model. The 2026 baseline for custom gesture logic; Touch Events (`touchstart`/`touchmove`) are legacy.
- **Scroll-Driven Animations API** (`animation-timeline: scroll()/view()`) — Chrome 116+, Firefox 128+, Safari 18+. Use over JS scroll handlers for progress bars, reveals, pinning. Pair with `prefers-reduced-motion` guard.
- **`addEventListener(..., { passive: true })`** — required for `touchmove` and `wheel` listeners to avoid blocking the scroll thread; no library needed.
- **`window.visualViewport`** — accurate dimensions when the soft keyboard is open; `window.innerHeight` is unreliable then.
- **`inputmode` + `autocomplete` attributes** — native virtual-keyboard control and autofill; never reach for a JS keypad widget.
- **When NOT to use a JS gesture library** (Hammer.js et al.): for back/forward/scroll — the OS and browser already own these. Only reach for gesture libs for genuinely custom interactions, and always add a visible fallback.

## Do / Don't

**Do**
- Design for `pointer: coarse` / `hover: none` first; enhance upward.
- Enlarge controls under `@media (any-pointer: coarse)` (checkbox/radio ≥26px, buttons ≥48px).
- Pad small icons' hit area to 44–48px while keeping the glyph small.
- Put primary CTAs/nav in the lower-center thumb zone.
- Add `viewport-fit=cover` *and* `env(safe-area-inset-*)` to every fixed/sticky edge.
- Use `svh`/`dvh` instead of `vh` for full-height mobile layouts.
- Set `touch-action: manipulation` on interactive elements.
- Use Pointer Events (`pointerdown/pointermove/pointerup`) for custom gesture logic.
- Use `{ passive: true }` on `touchmove`/`wheel` listeners and `requestAnimationFrame` for layout work.
- Pair every hover reveal with `:focus-within`.
- Wrap all scroll-driven animations with `@media (prefers-reduced-motion: no-preference)`.
- Use native `<button>`/`<a>`/`<input>` and `inputmode`/`autocomplete`.

**Don't**
- Don't infer touch from `min-width` breakpoints.
- Don't apply `:hover` styles without an `@media (hover: hover)` guard.
- Don't put primary actions or dismiss `×` in top corners on large phones.
- Don't ship sticky headers > 50px on mobile.
- Don't attach touch/pointer listeners to `document`/`body`, or omit `{ passive: true }` on `touchmove`/`wheel`.
- Don't make whole `<tr>` rows clickable for navigation.
- Don't use JS `mousemove` to "detect" non-touch (fires on stylus, races at load).
- Don't set `touch-action: none` on scrollable regions.
- Don't ship JS parallax to mobile.
- Don't use `vh` for full-height mobile layouts (use `svh`/`dvh`).
- Don't write new gesture logic against Touch Events — use Pointer Events.

## Common pitfalls & how to avoid them

- **Screen-width touch detection.** 768px ≠ touch; a 2000px Surface Studio *is* touch. → Use `@media (pointer/hover)`.
- **Unguarded `:hover`.** Triggers on tap and sticks, looking broken. → Wrap in `@media (hover: hover)`.
- **Missing `viewport-fit=cover`.** Then all `env(safe-area-inset-*)` return `0` and notch/home-indicator overlaps go unfixed. → Add it to the viewport meta.
- **Case-wrong env() names.** `SAFE-AREA-INSET-TOP` silently fails. → lowercase only.
- **Non-passive touch listeners on document/body.** Forces browser to await JS before scrolling — per-frame jank. → `{ passive: true }`, scoped to the element.
- **`<div onClick>` controls.** Lose focus, key activation, role, cursor, tap highlight. → Native `<button>`, or add `tabindex`+`role`+`onKeyDown` and verify all.
- **Clickable `<tr>`.** Breaks keyboard nav, selection, nested controls. → Action button in a cell.
- **Top-right modal/card close on 6.7"+ phones.** Worst reach for right-handed thumbs. → Bottom button / full-width footer.
- **Untested sticky header height.** > 50px can swallow 15–25% of a portrait viewport. → Measure; cap at 50px.
- **JS hover-detection (`mousemove`).** Fires on stylus, fails with JS off, races at load. → CSS media features.
- **Ignoring the ~350ms tap delay** in non-iOS/old Android WebViews. → `touch-action: manipulation` + `width=device-width`.
- **Using `innerHeight` with keyboard open.** Layout breaks. → Visual Viewport API.
- **Gesture-only navigation with no affordance.** Accidental swipe-back has no undo gesture. → Visible alternative; don't bind left-edge swipes.
- **JS-scroll-driven animations.** Block main thread, jank on low-end. → Scroll-Driven Animations API / CSS scroll timelines.
- **`touch-action: none` on a scroller.** Kills scroll *and* pinch-zoom (a11y break). → Use `pan-x`/`pan-y` instead.
- **`100vh` on iOS Safari.** Always equals the large viewport (chrome excluded), causing content to be clipped when browser chrome is visible. → Use `100svh` or `100dvh`.
- **Touch Events for new gesture code.** `touchstart`/`touchmove` are legacy; they don't unify stylus or mouse. → Use Pointer Events (`pointerdown`/`pointermove`).
- **Scroll-driven animation without `prefers-reduced-motion` guard.** Causes vestibular symptoms for affected users. → Wrap in `@media (prefers-reduced-motion: no-preference)`.

## What real users complain about

- *"The button looks tappable but nothing happens unless I hit the exact icon."* — visual element is the hit area; no padding. Fix with a 44–48px padded zone.
- *"The whole row is a link, so I can't select the text or tap the button inside it."* — clickable `<tr>` swallowing nested controls and text selection.
- *"On my new iPhone the Submit button is half-hidden behind the bar at the bottom."* — fixed footer with no `env(safe-area-inset-bottom)` padding.
- *"Tooltips/menus stay stuck open after I tap them on my phone."* — sticky `:hover` on touch, no `@media (hover: hover)` guard.
- *"I can never reach the X to close the popup one-handed on a big phone."* — dismiss control in a top corner.
- *"Scrolling stutters and feels heavy on this site"* (cheap Android) — non-passive listeners and/or JS parallax/scroll handlers blocking the main thread.
- *"I tapped a link and it zoomed in instead."* / *"taps feel laggy"* — double-tap-zoom and the legacy click delay; no `touch-action: manipulation`.
- *"The keyboard covers the input I'm typing into."* — layout computed from `innerHeight` instead of the Visual Viewport.
- *"I keep accidentally going back when I swipe."* — custom left-edge gesture conflicting with the OS swipe-back.
- *"The full-screen hero looks cropped on my iPhone — the bottom is cut off."* — `100vh` used for a full-height section; browser chrome overlaps it. Fix with `100svh`.

## Senior-level checklist

- [ ] All interactive targets ≥ 24×24 CSS px (audit floor); primary/thumb-operated controls ≥ 48px. Adjacent targets ≥ 8dp apart.
- [ ] Small visual controls have padded hit areas reaching 44–48px (verified on a dense display, not just in DevTools).
- [ ] No input inference from screen width — only `@media (pointer/hover/any-pointer/any-hover)`.
- [ ] Coarse-pointer branch enlarges form controls (checkbox/radio ≥26px, buttons ≥48px) via `any-pointer: coarse`.
- [ ] Every `:hover` reveal is guarded by `@media (hover: hover)` *and* mirrored on `:focus-within`/`:focus-visible`.
- [ ] No functionality exists *only* in hover state.
- [ ] Primary CTAs/nav in lower-center thumb zone; dismiss controls reachable one-handed on 6.7"+ phones.
- [ ] Bottom nav ≤ 5 tabs.
- [ ] Viewport meta includes `viewport-fit=cover`; every fixed/sticky edge uses correct lowercase `env(safe-area-inset-*)`.
- [ ] Sticky header content ≤ 50px on mobile; no stacked stickies eating the viewport.
- [ ] `touch-action: manipulation` on tappables; `pan-x`/`pan-y` (never `none`) on custom scroll/swipe containers.
- [ ] All `touchmove`/`wheel` listeners are `{ passive: true }`, element-scoped, and rAF-batched; no JS parallax on mobile.
- [ ] Custom gesture logic uses Pointer Events (`pointerdown`/`pointermove`/`pointerup`), not Touch Events.
- [ ] Scroll animations use the Scroll-Driven Animations API / CSS scroll timelines, not JS scroll handlers; wrapped in `@media (prefers-reduced-motion: no-preference)`.
- [ ] Full-height layouts use `svh`/`dvh`, not `vh`.
- [ ] Native semantic elements used; any custom control has full `tabindex`/`role`/key-activation/focus-ring.
- [ ] Forms set `inputmode` + `autocomplete` per field; layout uses Visual Viewport API when the keyboard is open.
- [ ] Custom gestures have visible non-gesture alternatives; no conflict with OS edge-swipe-back.
- [ ] Verified at DevTools CPU 20× slowdown + Slow 4G — scroll stays smooth.

## Sources

- WCAG 2.2 — Success Criterion 2.5.8 Target Size (Minimum), 2.5.5 Target Size (Enhanced): https://www.w3.org/TR/WCAG22/
- Apple Human Interface Guidelines — Layout / target sizes: https://developer.apple.com/design/human-interface-guidelines/layout
- Material Design 3 — Accessibility / touch target sizing: https://m3.material.io/foundations/designing/structure
- MDN — `pointer`/`hover`/`any-pointer`/`any-hover` media features: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/pointer
- MDN — `touch-action`: https://developer.mozilla.org/en-US/docs/Web/CSS/touch-action
- MDN — `env()` and safe-area insets: https://developer.mozilla.org/en-US/docs/Web/CSS/env
- MDN — Visual Viewport API: https://developer.mozilla.org/en-US/docs/Web/API/Visual_Viewport_API
- MDN — Pointer Events: https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events
- MDN — `dvh`/`svh`/`lvh` viewport units: https://developer.mozilla.org/en-US/docs/Web/CSS/length#viewport-percentage_lengths
- web.dev — Passive event listeners / scroll performance: https://web.dev/articles/use-passive-event-listeners
- Chrome for Developers — Scroll-driven animations: https://developer.chrome.com/docs/css-ui/scroll-driven-animations
- Hoober, S. — "How Do Users Really Hold Mobile Devices?" UXmatters, 2013: https://www.uxmatters.com/mt/archives/2013/02/how-do-users-really-hold-mobile-devices.php
- Parhi P., Karlson A., Bederson B. — "Target Size Study for One-Handed Thumb Use on Small Touchscreen Devices", MobileHCI 2006.
- Smashing Magazine — *The Thumb Zone: Designing For Mobile Users* (thumb reach zones, not swipe specs): https://www.smashingmagazine.com/2016/09/the-thumb-zone-designing-for-mobile-users/

# Micro-interactions & Motion Design

> Motion is a usability tool, not decoration. This guide gives you the decision rules, easing curves, duration tokens, choreography patterns, accessibility gates, and performance constraints to add animation that confirms actions, preserves spatial continuity, and directs attention — without inducing jank, latency, or vestibular harm.

**Apply when:** building any interactive UI — buttons, modals, route changes, scroll reveals, loaders. · **Related:** [Interaction Design & UI States](../01-ux-fundamentals/interaction-design-and-states.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Web Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md)

## TL;DR — rules at a glance

- **Every animation must serve one of four functions:** feedback, continuity, hierarchy, or delight. If the answer is "it looks cool," delete it.
- **Frequency governs intensity.** Button press ≤100ms; dropdown 150–200ms; modal 200–300ms; one-time onboarding can be 500ms + spring. The more often seen, the shorter and subtler.
- **Ease-out for entrances** (decelerate to rest), **ease-in for exits** (accelerate off-screen), **ease-in-out** for in-place transitions. Never use linear for UI motion.
- **Asymmetric timing:** entrance longer than exit (e.g. modal 300ms in, 200–250ms out).
- **Animate only `transform`, `opacity`, `filter`, `clip-path`.** These stay on the GPU compositor thread. Animating `top/left/width/height/margin` forces layout every frame → guaranteed jank.
- **No-motion-first.** Default to no animation; opt in via `@media (prefers-reduced-motion: no-preference)`. Protects vestibular-disorder users and unsupported browsers.
- **Stagger, don't fire simultaneously.** Guide the eye in one direction (top→bottom for vertical lists). Reverse on exit (last-in, first-out).
- **Web targets ~150–200ms for transitions; native mobile 300–400ms.** <100ms is imperceptible; >1000ms frustrates.
- **Never scroll-reveal body copy.** Reserve scroll animation for secondary/decorative content. Reading text that fades in per-paragraph is the #1 motion complaint.
- **Set reduced-motion override duration to `0.01ms`, not `0`** — true zero skips `transitionend`, breaking JS state machines.
- **Use `will-change` only after measuring a real problem** in DevTools. Pre-emptive layer promotion blows GPU memory and crashes low-end mobile.
- **Animate to reduce perceived latency, not to fill time.** Skeletons and optimistic UI feel fast; animations that just fill down-time frustrate.

## Purpose of motion: the four-function test

Before adding any animation, classify it. If it fits none, remove it.

| Function | What it does | Examples | Constraint |
|---|---|---|---|
| **Feedback** | Confirms the system received/processed an action | Button press, toggle flip, checkbox tick, form-error shake | Fastest. ≤100–150ms, fires within the input window |
| **Continuity** | Spatial wayfinding — "where I came from, where I'm going" | Shared-element morph, drawer slide, route transition, list reorder | Medium. 200–300ms; the path must read as continuous |
| **Hierarchy** | Draws attention to what matters now | Staged entrance, pulse on a new notification, focus ring | Subordinate to content; one focal point at a time |
| **Delight** | Earned, infrequent personality | Onboarding mascot, success confetti, empty-state flourish | Rare. Expressive OK (500ms+) but never on repeated UI |

Some UI structures are *driven* by motion and become confusing without it — e.g. the spatial models of Facebook Paper and Google Inbox. Motion communicates structure: it informs users where they are, where they came from, and where they will go next. NNGroup's framing: leverage animation "as clues about what is currently happening" rather than surface-level delight that quickly sours.

## Easing curves

Linear motion looks unnatural because nothing in the physical world starts and stops at constant velocity (NNGroup). Match the curve to the element's lifecycle.

| Phase | Curve | Why | Material 3 token |
|---|---|---|---|
| **Entrance** (element appears) | **ease-out** — full velocity in, decelerate to rest | Arrives quickly, settles gently; reads as "arriving" | `cubic-bezier(0, 0, 0.2, 1)` |
| **Exit** (element leaves) | **ease-in** — slow start, accelerate off-screen | Pulls away; reads as "leaving" | `cubic-bezier(0.4, 0, 1, 1)` |
| **In-situ** (moves/resizes in place) | **ease-in-out / standard** | Symmetric, no start/end emphasis | `cubic-bezier(0.2, 0, 0, 1)` |
| **Snap-back / returns** (dismiss drawer) | **sharp** (fast ease-in-out) | Returns immediately; no settle needed | `cubic-bezier(0.4, 0, 0.6, 1)` |
| **Expressive / playful** | **emphasized / spring** (slight overshoot) | Overshoot + settle feels physical (follow-through) | `cubic-bezier(0.2, 0, 0, 1.2)` |

```css
/* Reusable easing tokens */
:root {
  --ease-out:    cubic-bezier(0, 0, 0.2, 1);    /* entrances */
  --ease-in:     cubic-bezier(0.4, 0, 1, 1);     /* exits */
  --ease-std:    cubic-bezier(0.2, 0, 0, 1);     /* in-situ */
  --ease-sharp:  cubic-bezier(0.4, 0, 0.6, 1);   /* snap-back */
  --ease-emph:   cubic-bezier(0.2, 0, 0, 1.2);   /* overshoot */
}
```

### Spring physics (JS animation only)

Spring curves can't be expressed as cubic-bezier — they're physics simulations with three parameters: **stiffness** (how forcefully the spring pulls), **damping** (how quickly oscillation dies), and **mass** (inertia of the element). Good defaults: `stiffness: 300, damping: 30, mass: 1` (responsive, one overshoot, settles fast). For soft/bouncy feel: lower damping to 15–20. For stiff/immediate: raise stiffness to 500+, damping to 40+.

```js
// Motion (React) — spring transition
<motion.div
  animate={{ x: 100 }}
  transition={{ type: 'spring', stiffness: 300, damping: 30 }}
/>

// GSAP — elastic ease approximates spring
gsap.to(el, { x: 100, ease: 'elastic.out(1, 0.4)', duration: 0.6 });
```

Use spring for: modal entrance, notification pop-in, drag release, expressive buttons. Do not use spring for: exit animations (accelerate off cleanly with ease-in), repeated micro-interactions (the bounce costs time).

## Duration guidelines

Too fast (<100ms) is imperceptible; too slow (>500ms for UI, >1000ms for anything) reads as a delay. Tie duration to **action type** and **distance traveled**.

**By action type:**

- Simple feedback (toggle, checkbox): **~100ms**
- Standard UI transitions (dropdown, tooltip, tab): **100–300ms**
- Substantial screen changes (modal, side panel): **200–300ms**
- Large complex moves: **up to 500ms max** before it feels like waiting

**Material Design 3 duration tokens:** `instant = 100ms`, `fast = 160ms`, `base = 240ms`, `slow = 360ms`.

**Carbon (IBM) distance-based duration** — scale time to how far the element travels across the viewport:

| Travel distance | Duration token |
|---|---|
| ≤ 25% of view width | fast |
| 26–50% | medium |
| 51–100% | slow |

**Platform multiplier:** web UI animation is roughly 2× faster than native mobile. Target **150–200ms on web**; the same motion on a native app can run **300–400ms** because mobile users expect more spatial transition.

**Asymmetry:** entrance > exit. A modal that takes 300ms to appear should disappear in 200–250ms — exiting elements need less attention, matching how the brain processes appearance vs. disappearance.

## Choreography & stagger

Do not animate everything at once — simultaneous movement of all elements creates an overwhelming experience (Material Design Motion Choreography). Stagger to guide the eye in one direction.

**Rules:**
- **Stagger direction matches reading direction:** top→bottom for vertical lists, left→right for horizontal.
- **Reverse stagger on exit:** last-in, first-out.
- **One focal point at a time** (Disney's Staging) — the eye can only track one thing.
- **Per-item delay: 40–80ms** between successive items. Each item's own animation duration: **200–300ms**. Cap total stagger cascade at ~400ms for long lists regardless of item count (clamp `--i` at 5 or 6).

```css
/* Stagger with a CSS custom property index — no JS animation lib needed */
.card {
  opacity: 0;
  transform: translateY(16px);
  transition: opacity .25s var(--ease-out), transform .25s var(--ease-out);
  transition-delay: calc(min(var(--i), 6) * 60ms); /* set --i per child; cap at item 6 */
}
.card.is-visible { opacity: 1; transform: none; }
```

```js
// IntersectionObserver drives the .is-visible class; CSS does the stagger
const io = new IntersectionObserver((entries) => {
  entries.forEach((e) => {
    if (e.isIntersecting) { e.target.classList.add('is-visible'); io.unobserve(e.target); }
  });
}, { threshold: 0.1 });
document.querySelectorAll('.card').forEach((el, i) => {
  el.style.setProperty('--i', i);
  io.observe(el);
});
```

## Hover & press feedback

The tap cycle (finger-down to finger-up) is ~460ms; the **prime feedback window is 100–130ms from input**. Acknowledge within that window or the UI feels dead.

- **Press / active:** ≤100ms, instant. Use Disney's **Squash & Stretch** for tactile weight — a slight scale-down on `:active` gives the button physical mass. Keep volume consistent (scale, don't distort).
- **Hover:** add a **150–200ms delay** before triggering hover-only effects to avoid firing on mouse-pass-over (cursor traversal — the pointer crosses an element en route to a different target). Hover is desktop-only — never gate essential info behind it (touch has no hover).
- **Anticipation (Disney):** the hover state should telegraph what the click will do (e.g. a button that lifts on hover signals it's pressable).

```css
.btn {
  transition: transform .1s var(--ease-out), background-color .15s var(--ease-out);
}
.btn:active { transform: scale(0.97); }          /* squash, ≤100ms */
.btn:hover  { background-color: var(--accent-600); }
```

## Page & route transitions

- **Web standard: 150–200ms** — users are task-focused; long transitions block their goal. Mobile can extend to 300–400ms.
- Use shared-element continuity (`view-transition-name`) so a clicked card morphs into the destination header — this preserves spatial context far better than a crossfade.

```css
/* MPA cross-document transitions — add to BOTH pages */
@view-transition { navigation: auto; }
.hero-image { view-transition-name: hero; } /* same name on both pages → auto morph */
```

```js
// SPA — wrap the DOM update; browser animates the diff
document.startViewTransition(() => updateDOM());
```

**View Transition API support (as of mid-2026):** SPA (`document.startViewTransition`) — Chrome/Edge 111+, Firefox 135+, Safari 18+. MPA cross-document (`@view-transition`) — Chrome/Edge 126+, Safari 18.2+, Firefox 135+. Always feature-detect (`'startViewTransition' in document`); the API degrades gracefully (the DOM update still happens without animation).

## FLIP — animating layout without layout cost

FLIP (First–Last–Invert–Play) is the standard technique for animating elements between two layout positions (reorder, expand, shared-element) without animating `width/height/top/left`:

1. **First** — record `getBoundingClientRect()` before the change.
2. **Last** — apply the new layout; record `getBoundingClientRect()` after.
3. **Invert** — apply a `transform` that visually restores the element to its *First* position.
4. **Play** — remove the transform via transition/animation; element glides to its *Last* position using only compositor-thread `transform`.

```js
const first = el.getBoundingClientRect();
applyNewLayout();                        // DOM/class change that causes reflow
const last  = el.getBoundingClientRect();
const dx = first.left - last.left;
const dy = first.top  - last.top;
el.style.transform = `translate(${dx}px, ${dy}px)`;
requestAnimationFrame(() => {
  el.style.transition = 'transform .3s cubic-bezier(0,0,0.2,1)';
  el.style.transform  = '';
});
```

Motion (React) handles FLIP automatically via `layout` prop. GSAP Flip plugin does the same imperatively.

## Motion principles mapped to UI

**Material motion:** durations + standard/emphasized/sharp easing tokens (above), distance-based timing, and choreography (stagger, one focal point). Use it as the productive baseline for app UI.

**Disney's 12 principles, the five that map to UI:**

- **Anticipation** — hover telegraphs the click outcome.
- **Follow-through / Overshoot** — a modal overshoots its final position then settles; feels natural (use `emphasized` curve).
- **Squash & Stretch** — press states give tactile weight; *volume must stay consistent* (scale, don't smear).
- **Timing** — too fast = invisible, too slow = waiting.
- **Staging** — animate one thing at a time to direct the eye.

## Scroll-driven animations (CSS-native, 2024+)

`animation-timeline: scroll()` lets you link an animation's progress directly to scroll position — no JavaScript, no `IntersectionObserver`. Landed in Chrome/Edge 115+, Safari 18+, Firefox 110+ (behind flag until Firefox 132 — released Oct 2024).

```css
@keyframes fade-in {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: none; }
}

.section {
  animation: fade-in linear;
  animation-timeline: view();        /* triggers as element enters viewport */
  animation-range: entry 0% entry 40%;
  animation-fill-mode: both;
}
```

Use `view()` for element-enters-viewport triggers; use `scroll()` for progress-tied scrub effects (progress bars, parallax). Feature-detect and keep the element fully visible in non-supporting browsers — this is progressive enhancement, not a hard dependency. **Gate behind `prefers-reduced-motion: no-preference`** like any other animation.

## prefers-reduced-motion (accessibility, not optional)

Parallax, zoom, and large-distance motion cause dizziness, nausea, and hours-long vertigo for users with vestibular disorders. **WCAG 2.3.3 (AAA):** "Motion animation triggered by interaction can be disabled, unless the animation is essential to functionality." **WCAG 2.3.1 (AA — Three Flashes):** never flash content more than 3 times per second.

**No-motion-first pattern** — default off, opt in. This also covers browsers that don't support the query:

```css
.spinner { animation: none; }
@media (prefers-reduced-motion: no-preference) {
  .spinner { animation: spin 2s linear infinite; }
}
```

**Block the animation stylesheet entirely for reduced-motion users:**

```html
<link rel="stylesheet" href="animations.css"
      media="(prefers-reduced-motion: no-preference)">
```

**Global safety net** — when you can't audit every animation, neutralize them, but use `0.01ms` (not `0`) so `transitionend`/`animationend` still fire and JS state machines don't hang:

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

**JS animations (GSAP) — wrap in `matchMedia`** so all timeline/scroll code is automatically scoped:

```js
const mm = gsap.matchMedia();
mm.add('(prefers-reduced-motion: no-preference)', () => {
  /* all ScrollTrigger / timeline code here — auto-reverted when preference changes */
});
```

Smooth-scroll libraries (Lenis) must also be gated behind `no-preference` — forced scroll velocity changes are themselves a vestibular trigger.

## Performance: compositor-only or jank

The browser pipeline is Layout → Paint → Composite. Animate properties that touch only the **last** step.

| Property | Pipeline cost | Verdict |
|---|---|---|
| `transform`, `opacity` | Composite only (GPU thread) | ✅ Animate freely |
| `filter`, `clip-path` | Composite (GPU) | ✅ Generally safe |
| `top/left/right/bottom`, `width/height`, `margin` | **Layout** every frame | ❌ Never animate |
| `background-color`, `border-radius`, `box-shadow`, `color` | **Paint** every frame | ⚠️ Avoid in animation |

Animating `transform`/`opacity` keeps the browser on the compositor thread and avoids layout/paint work — the frame budget improvement over positional properties is dramatic and measurable in DevTools. Animating `top/left` or `width/height` forces a full layout pass on every frame, causing dropped frames even on fast hardware.

**`will-change` — sparingly.** Promote a layer only *after* observing a real problem in DevTools, never pre-emptively. Excessive layer promotion exhausts the GPU memory budget and crashes browsers on low-end mobile. Remove it once the animation completes.

**CSS variable trap:** changing one inherited CSS variable that cascades across many elements triggers "wildfire" style recalc every frame. Prevent this with `@property { inherits: false }` scoping the recalc to a single element:

```css
@property --glow {
  syntax: '<length>';
  inherits: false;     /* scope recalc to this element only */
  initial-value: 0px;
}
```

**Verify before shipping** with Chrome DevTools: **Paint Flashing** overlay (green = repaint — should not flash during a transform animation), **Layers** panel (compositor layers + GPU memory), **Animations** panel (record/replay/slow), **Performance** panel (frame timeline; aim for 60fps / 16.7ms budget).

## Not overdoing it

Animate to **reduce perceived latency**, not to fill time. Skeleton screens, progress indicators, and optimistic UI make a system *feel* faster without changing actual speed. Animations that simply fill down-time frustrate users in usability testing (NNGroup).

- **One-time, not every-time:** if a first-time visitor must watch the same hero/intro animation on every visit, it trains them to hate the site. Animate once; store a flag in `sessionStorage`.
- **Body copy is sacred:** never scroll-reveal the text users came to read. Reserve scroll animation for images, icons, and decorative/supporting elements.
- **Cumulative cost:** each animation may only slow down users minimally, but the effect of many such animations becomes increasingly noticeable and measurably slows task completion (NNGroup). Audit the *total* motion budget of a page, not just each piece.

## Concrete specs & numbers

- **Press feedback:** ≤100ms; prime window 100–130ms from input; full tap cycle ~460ms.
- **Hover trigger delay:** 150–200ms to avoid pass-over misfires.
- **Toggle/checkbox:** ~100ms.
- **Dropdown/tooltip/tab:** 100–300ms.
- **Modal/panel:** 200–300ms in; 200–250ms out (asymmetric).
- **Large/complex move:** ≤500ms hard ceiling.
- **Route transition:** web 150–200ms; mobile 300–400ms.
- **Perception bounds:** <100ms imperceptible; >1000ms frustrating.
- **Material 3 durations:** instant 100 / fast 160 / base 240 / slow 360ms.
- **Carbon distance→duration:** ≤25% fast, 26–50% medium, 51–100% slow.
- **Stagger:** 40–80ms per-item delay; each item animates 200–300ms; cap total cascade at ~400ms (clamp index at 5–6); `threshold: 0.1` in IntersectionObserver.
- **Reduced-motion override:** `0.01ms` (never `0`).
- **Frame budget:** 16.7ms/frame for 60fps; `transform`/`opacity` compositor-only; layout/paint properties kill frame rate.
- **Flicker:** ≤3 flashes/second (WCAG 2.3.1 AA). Disable interaction-triggered animation on request (WCAG 2.3.3 AAA).

## Tools & libraries

| Tool | Size (gzip) | Use for | Avoid when |
|---|---|---|---|
| **Motion (ex-Framer Motion)** | ~32KB | React component transitions, layout animations (FLIP via `layout` prop), `AnimatePresence` exit anims, gestures, spring physics | Non-React projects; size-critical pages |
| **GSAP + ScrollTrigger** | 23KB core / ~48KB +ST | Complex sequenced timelines, scroll storytelling (pin/scrub), SVG morph, canvas/Three.js. Framework-agnostic | **License caveat:** owned by Webflow (acquired Jan 2025); prohibits use in Webflow-competing tools, terminable at discretion — review license before use |
| **View Transition API** | 0KB (native) | Page/route morphing, shared-element continuity | Older browsers — feature-detect |
| **WAAPI** (`element.animate`) | 0KB (native) | Imperative compositor-thread animation without a library | Complex orchestration/scrubbing |
| **Motion One** | ~3KB | Friendly WAAPI wrapper; lightweight vanilla animation | Heavy timelines |
| **CSS scroll-driven animations** | 0KB (native) | Scroll-progress ties, viewport-entry reveals — Chrome/Edge 115+, Safari 18+, Firefox 132+ | Browsers that don't support; always progressive-enhance |
| **IntersectionObserver** | 0KB (native) | Scroll-reveal triggers (`threshold: 0.1`); pair with CSS stagger | Continuous scrub (use ScrollTrigger or scroll-driven) |
| **Rive** | runtime | Interactive state-machine animation responding to input; smaller than Lottie | Static playback only |
| **Lottie/LottieFiles** | runtime | Designer-made AE exports: empty states, loaders, branded vectors (read-only) | Interactive/responsive needs (use Rive) |
| **AOS** | ~8KB | Zero-config data-attribute reveals on simple marketing pages | Complex sequencing |
| **Lenis** | small | Awwwards-style smooth scroll — **must** gate behind `no-preference` | Anywhere accessibility is a priority and unscoped |

```js
// WAAPI — native, compositor-thread, no dependency
element.animate(
  [{ opacity: 0, transform: 'translateY(16px)' }, { opacity: 1, transform: 'none' }],
  { duration: 300, easing: 'cubic-bezier(0,0,0.2,1)', fill: 'forwards' }
);
```

Motion has far higher weekly npm downloads than GSAP but is React-only; GSAP is framework-agnostic. Choose by stack and license tolerance. See [Web Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md) for full decision matrix.

## Do / Don't

**Do**
- Classify every animation as feedback / continuity / hierarchy / delight before building it.
- Use ease-out for entrances, ease-in for exits, ease-in-out in place.
- Keep repeated-use UI fast (100–200ms) and one-time anims expressive (500ms + spring).
- Restrict animated properties to `transform`, `opacity`, `filter`, `clip-path`.
- Default to no motion; opt in with `prefers-reduced-motion: no-preference`.
- Stagger lists in reading direction; reverse on exit; cap cascade at ~400ms total.
- Use FLIP for layout transitions; use scroll-driven animations for viewport-entry reveals.
- Use skeletons/optimistic UI to mask latency.
- Verify with DevTools Paint Flashing + Layers before shipping.

**Don't**
- Animate `top/left/width/height/margin/box-shadow/background-color`.
- Scroll-reveal body copy or hijack horizontal scroll for text.
- Add `will-change` pre-emptively to every element.
- Use linear easing for UI.
- Replay the same intro animation on every page load.
- Fire all elements simultaneously.
- Use `0` (use `0.01ms`) in reduced-motion overrides.
- Gate essential information behind hover.
- Use spring curves for exits — use ease-in to accelerate off-screen cleanly.

## Common pitfalls & how to avoid them

- **Scroll-reveal on body text** — the #1 complained-about pattern: users describe it as cute once, tedious the second time, and actively annoying thereafter (NNGroup usability testing). → Reveal only secondary/decorative content; never the copy.
- **Animating layout-triggering properties** — `top/left/width/margin` recalculates geometry every frame → dropped frames. → Use `transform` (translate/scale) instead; use FLIP for layout changes.
- **Pre-emptive `will-change`** — exhausts GPU memory, crashes low-end mobile. → Add only after measuring; remove after the animation.
- **`0`-duration reduced-motion override** — skips `transitionend`, hanging JS state machines waiting on completion. → Use `0.01ms`.
- **Inherited CSS-variable animation** — cascading recalc across thousands of nodes (8ms+/frame). → `@property { inherits: false }`.
- **Simultaneous mass animation** — overwhelming; eye can't track. → Stagger, one focal point.
- **Stagger that never ends** — 80ms delay × 20 items = 1.6s before last item animates. → Clamp index at 5–6 regardless of list length.
- **Same-duration everything** — ignores distance and frequency. → Apply distance-based + frequency-intensity scaling.
- **Down-time fillers** — animations that just pass time frustrate. → Animate to reduce *perceived* latency only.
- **Unscoped smooth scroll** — forced velocity triggers vestibular symptoms. → Gate Lenis/smooth-scroll behind `no-preference`.
- **Spring on exit** — spring oscillates before settling; exits need to leave fast. → Use ease-in for all exit animations; spring only for entrances/expressive moments.

## What real users complain about

- "I hate that it has to load every single section." — scroll-reveal sites in NNGroup usability testing.
- "It's cute the first time, tedious the second time, and annoying every other time." — representative feedback from usability studies on scroll-reveal body content.
- "If I have to fight the website to read it, I'm gone." — horizontal-scroll-for-text functions as an instant back button for most users.
- **Cumulative drag:** individual animations feel minor, but the effect of many animations becomes increasingly noticeable and measurably slows task completion (NNGroup).
- **Repeated intros:** forcing the same hero animation on every visit trains users to resent the site.
- **Motion sickness:** parallax/zoom/large-distance motion causes real dizziness, nausea, and lasting vertigo for vestibular-disorder users — a hard accessibility failure, not a preference.

## Senior-level checklist

- [ ] Every animation maps to feedback / continuity / hierarchy / delight (purpose test passed).
- [ ] Entrances ease-out, exits ease-in, in-place ease-in-out; nothing linear.
- [ ] Durations within type/distance bands; entrance > exit (asymmetric).
- [ ] Repeated-use UI ≤200ms; expressive motion reserved for one-time moments.
- [ ] Only `transform`/`opacity`/`filter`/`clip-path` animated — verified in DevTools Paint Flashing (no repaint flashes).
- [ ] Layers panel checked; no excessive layer promotion; `will-change` only where measured.
- [ ] `prefers-reduced-motion: no-preference` opt-in (no-motion-first); global `0.01ms` safety net present.
- [ ] JS/scroll animation wrapped in `gsap.matchMedia` or equivalent; smooth scroll gated.
- [ ] Lists staggered in reading direction; reversed on exit; cascade capped at ~400ms; one focal point at a time.
- [ ] No scroll-reveal on body copy; no horizontal-scroll text.
- [ ] One-time intros flagged in `sessionStorage`; not replayed.
- [ ] Route/page transitions 150–200ms (web); shared-element continuity via `view-transition-name` where helpful, feature-detected.
- [ ] No flicker >3×/sec (WCAG 2.3.1 AA); interaction-triggered animation disable honored (WCAG 2.3.3 AAA).
- [ ] Spring physics used only for entrances/expressive moments; exits use ease-in.
- [ ] CSS scroll-driven animations used (where supported) in preference to JS scroll listeners for viewport-entry reveals.
- [ ] FLIP used for layout-position transitions (not `top/left` animation).
- [ ] 60fps confirmed in Performance panel on a mid/low-end device.

## Sources

- Nielsen Norman Group — Animation usability, executing UX animations, and scroll-animation findings (nngroup.com).
- Material Design 3 — Motion: easing & duration tokens, Motion Choreography (m3.material.io).
- Carbon Design System (IBM) — Motion duration guidelines (carbondesignsystem.com).
- MDN / web.dev — `prefers-reduced-motion`, animation performance (compositor vs. layout), View Transition API, Web Animations API, CSS scroll-driven animations (developer.mozilla.org, web.dev).
- W3C WCAG 2.2 — Success Criteria 2.3.1 (Three Flashes, AA) and 2.3.3 (Animation from Interactions, AAA) (w3.org/WAI/WCAG22).
- GSAP docs — `matchMedia`, ScrollTrigger, licensing note re: Webflow acquisition (gsap.com).
- Motion (Framer Motion) docs — spring API, layout animations, AnimatePresence (motion.dev).
- CSS scroll-driven animations spec and browser implementation notes (drafts.csswg.org, Chrome Developers blog).

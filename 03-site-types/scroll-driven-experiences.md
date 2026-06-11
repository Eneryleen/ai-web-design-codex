# Scroll-Driven Experiences (Magic Scroll, Pinning, Parallax)
> How to design and build award-tier scroll-driven one-pagers: scroll storytelling, pinning/sticky timelines, parallax depth, scroll-linked animation, and frame sequences — using GSAP ScrollTrigger, Lenis, and native CSS scroll-driven animations. The senior outcome: motion that serves narrative and guides attention, runs jank-free on the compositor thread, and degrades gracefully for reduced-motion users and mobile.

**Apply when:** campaign microsites, product showcases, agency/portfolio hero experiences — NOT task-driven flows · **Related:** [Single-Page Sites & One-Pagers](./single-page-one-pagers.md) · [Web Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md) · [Micro-interactions & Motion Design](../07-aesthetics-and-trends/micro-interactions-and-motion.md) · [Playbook: Build a Scroll-Driven One-Pager](../09-playbooks-and-checklists/playbook-scroll-experience.md)

## TL;DR — rules at a glance
- Scroll animation is a **storytelling medium, not decoration**. If you cannot articulate WHY an effect exists (pacing or attention), delete it.
- Animate **only `transform` and `opacity`**. Everything else (`width`, `height`, `top`, `left`, `margin`, `box-shadow`, `background-color`) forces layout/paint and causes jank.
- **Use scroll-driven UX on the right pages only**: marketing microsites, product showcases, portfolios. Never on dashboards, forms, checkouts, blogs, or e-commerce flows.
- **Pin a container, animate its children.** Never animate the pinned element itself — it breaks ScrollTrigger's measurement.
- **Parallax ratios:** background 20–40% of scroll velocity, foreground 60–80%, fixed UI overlays 100%.
- **`prefers-reduced-motion` is a medical necessity** (vestibular disorders affect tens of millions; VEDA cites ~35M Americans alone). Wrap all scroll motion in `@media (prefers-reduced-motion: no-preference)`.
- **Native CSS first where supported**: `animation-timeline: scroll()` / `view()` run off the main thread. Declare `animation-timeline` AFTER the `animation` shorthand (later declaration wins).
- **Lenis + GSAP**: drive `lenis.raf()` from the GSAP ticker, call `lenis.on('scroll', ScrollTrigger.update)`, and set `gsap.ticker.lagSmoothing(0)`. Don't double-tick.
- **`ScrollTrigger.refresh()`** after fonts/images/layout settle, or trigger positions will be wrong.
- **Create ScrollTriggers top-to-bottom in DOM order** — pin-spacer insertion miscalculates downstream triggers otherwise.
- **`overflow: clip`, not `overflow: hidden`** on scroll-animation containers (hidden breaks CSS scroll-seeking).
- **Never `content-visibility: auto`** on ancestors/siblings of a pinned element — it silently breaks pin position calc.
- **Reveal-on-scroll budget**: < 400ms duration, < 200ms stagger per item. Fast scrollers must never wait to read.
- **100ms response budget** (Google RAIL model) for scroll-linked animation. WCAG 2.2 SC 2.3.3 (AAA, "Animation from Interactions") separately requires that users can *disable* scroll-triggered motion — it does not mandate a timing budget.
- **Apple-style frame sequences = WebP image sequence**, never `<video>` seeking.
- **Don't overdo it.** Restraint reads as premium; constant motion reads as a tech demo.

## Why and where (the strategic gate)

Scroll-driven motion is expensive to build, easy to abuse, and actively harmful on the wrong page. Gate every project against intent.

| Page type | Scroll-driven? | Smooth scroll (Lenis)? |
|---|---|---|
| Campaign microsite / brand story | Yes | Yes |
| Product showcase / feature reveal | Yes | Yes |
| Agency / portfolio hero | Yes | Yes |
| Landing page (conversion) | Sparingly — above-fold reveal only | Optional, test it |
| Blog / docs / content | No (maybe a reading progress bar) | No |
| E-commerce browse/checkout | No | No |
| Dashboard / SaaS app / forms | No | Never |

The rule: **experience-driven pages can hijack scroll; task-driven pages must not.** Smooth-scroll libraries are technically scroll-hijacking — they break the user's muscle memory and the browser's native scroll position. On interactive apps this increases click-target errors and frustrates users with assistive tech.

## Motion hierarchy & depth (the core illusion)

Perceived depth comes from **differential speed**, not 3D rendering.

- **Background** shifts slightly (20–40% of scroll speed).
- **Midground** moves at medium speed (~50%).
- **Foreground / CTAs** move fastest (60–80%).
- **Fixed UI overlays** stay at 100% (locked to viewport).

This produces a believable 3D parallax without the GPU cost of real 3D. Keep layer count low (3–4 planes); each additional moving layer is more compositing work and more visual noise.

## Pinning & scroll-linked timelines (GSAP ScrollTrigger)

The signature "magic scroll" effect is a **pinned section with a scrubbed timeline**: pin a container in place, then drive an animation timeline by scroll position so it plays forward/back as the user scrolls.

```js
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger); // required; GSAP >= 3.3.0

const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".panel",
    start: "top top",      // default becomes 'top top' when pin: true
    end: "+=2000",         // scroll distance the pin lasts
    pin: true,             // pins .panel; spacer is auto-inserted
    scrub: 0.5,            // playhead takes 0.5s to catch up to scroll
    anticipatePin: 1,      // hide flash of unpinned content on large sections
    // markers: true,      // DEV ONLY — remove for production
  },
});

// Animate CHILDREN of the pinned element, never .panel itself.
tl.from(".panel .headline", { yPercent: 40, opacity: 0 })
  .from(".panel .image",   { scale: 1.2 }, "<");
```

Non-negotiables:
- **Never animate the pinned element itself.** Animate only its children. Animating the pinned node breaks all start/end measurement.
- **`scrub`**: `true` = 1:1 sync; `0.3`–`1` = smooth catch-up (more organic, but high values feel sluggish). `scrub: 0.5` means the playhead takes 0.5s to reach the scroll position.
- **Create triggers in DOM order, top to bottom.** Pin spacers are inserted into the layout; out-of-order creation makes every downstream trigger miscalculate.
- **No `transform` or `will-change` on ancestors** of a pinned element — it breaks `position: fixed` pinning. Restructure HTML before reaching for `pinReparent: true` (DOM reparenting is expensive).
- **Nested pins**: set `pinnedContainer` on the inner trigger; never mix nested pins blindly.
- **`anticipatePin: 1`** on large sections compensates for threaded-scroll repaint lag that flashes unpinned content.
- **`ScrollTrigger.refresh()`** after images load, fonts resolve, or layout changes — triggers do not auto-recalculate.

### Batching lists/grids
For 10+ identical elements (cards, gallery items), use one batch instead of N triggers:
```js
ScrollTrigger.batch(".card", {
  onEnter: (els) => gsap.to(els, { opacity: 1, y: 0, stagger: 0.1, overwrite: true }),
  start: "top 85%",
});
```

## Smooth scroll: Lenis + GSAP integration

Lenis (~3 kB gzipped) adds inertial smooth scrolling. The correct wiring keeps Lenis and ScrollTrigger in lockstep:

```js
import Lenis from "lenis";

const lenis = new Lenis({ lerp: 0.1, duration: 1.5, syncTouch: true });
// autoRaf defaults: do NOT also call lenis.raf() manually if autoRaf is on — double-ticking = double speed.

lenis.on("scroll", ScrollTrigger.update);          // keep triggers in sync
gsap.ticker.add((time) => lenis.raf(time * 1000)); // drive Lenis from GSAP's clock
gsap.ticker.lagSmoothing(0);                        // CRITICAL: prevents 1–2 frame de-sync jitter
```

- **`gsap.ticker.lagSmoothing(0)` is mandatory** in this setup. Without it GSAP introduces artificial delay that de-syncs smooth scroll from ScrollTrigger markers.
- Lenis v1.0 (Sept 2024) removed `smoothTouch`; use `syncTouch: true` for touch smoothing. Recommended feel: `lerp: 0.1`, `duration: 1.5`.
- The package is `lenis` (the old `@studio-freight/lenis` is deprecated). React users: `lenis/react` exposes `<ReactLenis>`.
- **Alternative:** GSAP **ScrollSmoother** (Club plugin) does smooth scroll + ScrollTrigger natively with no proxy wiring. Prefer it if you already pay for GSAP Club.
- **Never** apply Lenis/Locomotive to interactive apps, checkouts, or forms.

## Native CSS scroll-driven animations

Where supported, native CSS animations driven by scroll run **off the main thread** — the performance win JS cannot match. Use them for progress bars, reveals, and parallax that don't need a JS timeline.

Two timelines:
- **`scroll()`** — progress through a scroll container (e.g., a reading progress bar).
- **`view()`** — progress of an element through the viewport (e.g., reveal as it enters/exits).

```css
/* Reading progress bar tied to document scroll */
@media (prefers-reduced-motion: no-preference) {
  @supports (animation-timeline: scroll()) {
    .progress {
      transform-origin: left;
      animation: grow linear;
      animation-timeline: scroll(root block); /* declare AFTER the shorthand */
      /* exclude footer from the bar: */
      animation-range: 0% calc(100% - 500px);
    }
    @keyframes grow { from { transform: scaleX(0); } to { transform: scaleX(1); } }
  }
}

/* Reveal an element as it enters the viewport */
@media (prefers-reduced-motion: no-preference) {
  @supports (animation-timeline: view()) {
    .reveal {
      animation: fade-up linear both;
      animation-timeline: view();
      animation-range: entry 0% entry 100%; /* scope to entry phase */
    }
    @keyframes fade-up {
      from { opacity: 0; transform: translateY(2rem); }
      to   { opacity: 1; transform: translateY(0); }
    }
  }
}
```

Rules:
- **Declare `animation-timeline` AFTER the `animation` shorthand.** The shorthand resets `animation-timeline` to `auto`; later declaration wins the cascade.
- **Set `animation-fill-mode: both`** (or include it in the `animation` shorthand). Without it, scroll-driven animations snap back to their `from` state when the element is outside the animation range.
- Use **named ranges** (`entry`, `exit`, `cover`, `contain`) to scope animation to a visibility phase. Default range is `normal` (`0% 100%`).
- **`overflow: clip`, not `hidden`** on the container — `hidden` breaks scroll-seeking.
- Always wrap in `@media (prefers-reduced-motion: no-preference)` AND `@supports (animation-timeline: scroll())` for accessible, cross-browser deployment. Base styles must read fine with neither.
- Support reality (mid-2026): Chrome/Edge 115+, Safari 18+ (Sept 2024) ship `animation-timeline: scroll()` and `view()`. Firefox is behind a flag. The [scroll-timeline polyfill](https://github.com/flackr/scroll-timeline) covers the gap but runs on the main thread (loses the off-thread perf win) — use sparingly, or fall back to GSAP ScrollTrigger.

## Reveal-on-scroll (the most-abused effect)

Reveal-on-scroll is fine for hero/section entrances and bad for every paragraph. The failure mode: a fast scroller reaches content that hasn't faded in yet and has to wait — or scroll down and back up — to read it.

- **Budget**: animation < 400ms, stagger < 200ms per sequential item.
- Reveal **sections and key elements**, not body copy line-by-line.
- Use **`IntersectionObserver`** (JS) or `animation-timeline: view()` (CSS) for declarative entry. Native CSS `view()` timelines (Chrome 115+, Safari 18+) now handle most reveal patterns without IntersectionObserver — prefer them where supported.
- Once revealed, content stays revealed. Don't re-hide on scroll-up unless the design genuinely requires it.

## Horizontal scroll sections

Convert vertical scroll into horizontal movement inside a pinned section:
```js
const track = document.querySelector(".h-track");
gsap.to(track, {
  x: () => -(track.scrollWidth - window.innerWidth),
  ease: "none",
  scrollTrigger: {
    trigger: ".h-wrap",
    pin: true,
    scrub: 1,
    end: () => "+=" + (track.scrollWidth - window.innerWidth),
    invalidateOnRefresh: true, // re-evaluates the end() callback on resize so the distance stays accurate
  },
});
```
- **Provide a visual affordance** (peeking next card, arrows, progress dots, a hint label). Horizontal scroll with no cue is a discoverability failure — users assume the section ended.
- On mobile/touch, prefer a **native horizontal scroll** with `scroll-snap` over hijacked pin-scroll; it matches platform expectations:
  ```css
  .h-track {
    display: flex;
    overflow-x: auto;
    scroll-snap-type: x mandatory;
    -webkit-overflow-scrolling: touch;
  }
  .h-track > * { scroll-snap-align: start; flex: 0 0 100vw; }
  ```
  Gate the GSAP pin behind `ScrollTrigger.isTouch === 0` (pointer-only) and fall back to the native scroll above on touch devices.

## Frame-by-frame sequences (Apple-style)

For scroll-scrubbed product hero sequences:
- **Pre-extract frames as WebP images.** Do **not** seek a `<video>` element — browser video APIs are built for linear playback, not high-frequency bidirectional scrubbing. Typical sequence: 60–150 frames for a 3–5 s scrub (export at 24–30 fps, then drop alternate frames until file-size budget is met). Target < 200 kB per frame at display size; compress with Squoosh/cwebp `-q 80`.
- **Progressive load**: preload a set of low-quality keyframes (every 10th frame, ~10× smaller) for instant visual response; swap in full-quality frames per-index as they finish loading in the background.
- Draw frames to a `<canvas>` and index by `Math.round(progress * (frameCount - 1))`.
- **Preload critical frames**: use `<link rel="preload" as="image">` for the first 5–10 frames so the hero doesn't flash blank on arrival.
- For genuinely complex 3D you can't fake with sprites, escalate to Three.js — see [WebGL, 3D & Shaders](../04-libraries-and-tools/webgl-3d-shaders.md).

## Performance discipline

- **Compositor-only properties.** `transform` (translate/scale/rotate/skew) and `opacity` are the only safe animatable properties. Anything else triggers layout or paint → jank.
- **`will-change` sparingly.** It promotes the element to its own compositor layer (~a few MB VRAM each). Apply via JS only when an animation is about to run; **remove it after**. Over-promotion degrades performance.
- **Keep animation distance proportional to scroll distance.** ~300px of scroll should feel like a ~0.3–0.5s animation. Scrub distances much longer than content height create scroll fatigue.
- **Throttle/let the library handle rAF.** ScrollTrigger already coalesces to the rAF loop; don't add your own scroll listeners doing layout reads.
- **`content-visibility: auto`** is great for perf elsewhere but **forbidden** on ancestors/siblings of pinned elements (GSAP issue #465).
- **iOS Safari**: address-bar show/hide changes viewport height mid-scroll and makes pins jump. Use `ScrollTrigger.normalizeScroll(true)` — it intercepts `touchmove` events so GSAP controls scroll position directly, preventing the height-change jump and normalizing scroll reporting inconsistencies across iOS Safari versions.
- Respect the **100ms response budget** (Google RAIL model) for scroll-linked animation and Core Web Vitals — see [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md).

## Accessibility & reduced motion

`prefers-reduced-motion` is the single most important rule here. Vestibular disorders (VEDA cites ~35M Americans; estimates of hundreds of millions globally for episodic vertigo) can cause dizziness, nausea, and migraine from parallax and large scroll motion.

```css
/* Base styles are fully readable with NO animation. Then opt-in to motion: */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

- **Progressive enhancement**: write base styles first (readable without JS/animation), then layer scroll effects inside `@media (prefers-reduced-motion: no-preference)`.
- In JS, gate animations behind the same query and **disable Lenis** for reduced-motion users. Prefer **`gsap.matchMedia()`** over a one-shot `window.matchMedia` check — it re-runs when the user changes OS settings mid-session and integrates cleanly with `gsap.context()` scoping:
  ```js
  const mm = gsap.matchMedia();
  mm.add("(prefers-reduced-motion: no-preference)", () => {
    const lenis = new Lenis({ lerp: 0.1, duration: 1.5 });
    lenis.on("scroll", ScrollTrigger.update);
    gsap.ticker.add((t) => lenis.raf(t * 1000));
    gsap.ticker.lagSmoothing(0);
    // ... register ScrollTrigger timelines here
    return () => lenis.destroy(); // cleanup on mq change
  });
  ```
- React: use Framer Motion's `useReducedMotion()` — it reads the system preference reactively and stays synced if the user toggles OS settings mid-session.
- **Reduced-motion must not break content.** Disabling animation should reveal the final state, never leave elements stuck at `opacity: 0` or off-screen.
- Browser support for the query: Chrome 74+, Edge 79+, Firefox 63+, Safari 10.1+ (effectively universal).
- Keyboard users must still reach all content; scroll-jacking must not trap focus or skip interactive elements. See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md).

## Mobile & fallback strategy

- **Pinned/horizontal scroll is heavier on mobile.** Test on real devices; favor simpler vertical reveals on small screens.
- Replace hijacked horizontal pins with **native `scroll-snap`** on touch.
- **`ScrollTrigger.isTouch`**: `0` = pointer-only, `1` = touch-only, `2` = both — branch behavior accordingly.
- Smooth scroll on touch: `syncTouch: true` in Lenis (was `smoothTouch`, removed in v1.0). Many teams ship without smooth-touch at all — native momentum scroll is often better on mobile.
- Account for the **iOS address-bar viewport jump** (`normalizeScroll`) or use `dvh`/`svh` units for full-height sections.

## SPA / framework cleanup

Failing to tear down triggers leaks memory and accumulates jank across route changes.
```jsx
// React
useLayoutEffect(() => {
  const ctx = gsap.context(() => {
    gsap.to(".box", { x: 200, scrollTrigger: { trigger: ".box", scrub: true } });
  }, rootRef);
  return () => ctx.revert(); // kills all triggers/animations in this scope
}, []);
```
- Scope everything with **`gsap.context()`** and `revert()` on unmount. If `gsap.context()` is unavailable (e.g., plain JS without a component model), use `ScrollTrigger.getAll().forEach(t => t.kill())` as the escape hatch.
- Call **`ScrollTrigger.refresh()`** after route transition layout settles (and after Barba.js page transitions if used).

## Concrete specs & numbers

- **Compositor-safe properties:** `transform`, `opacity` only.
- **Parallax ratios:** background 20–40%, midground ~50%, foreground 60–80%, fixed UI 100% of scroll speed.
- **Scrub:** `true` = 1:1; `0.3–1` = smooth; `scrub: 0.5` = 0.5s catch-up.
- **Reveal budget:** < 400ms duration, < 200ms stagger per item.
- **Animation-to-scroll ratio:** ~300px scroll ≈ 0.3–0.5s animation feel.
- **Response budget:** 100ms max (Google RAIL model). WCAG 2.2 SC 2.3.3 (AAA) = users can disable scroll animation, not a timing budget.
- **ScrollTrigger defaults:** `start: 'top bottom'`; becomes `'top top'` when `pin: true`. Resize throttle: 200ms. `fastScrollEnd` velocity: 2500px/s.
- **`will-change`:** ~a few MB VRAM per promoted layer; apply just-in-time (add class/attribute in `requestAnimationFrame` before animation starts), remove after.
- **Lenis:** ~3 kB gzipped; `lerp: 0.1`, `duration: 1.5`, `syncTouch: true`.
- **GSAP minimum for ScrollTrigger:** 3.3.0.
- **Native CSS scroll-driven animations:** Chrome/Edge 115+, Safari 18+ (Sept 2024) shipped; Firefox: behind flag as of mid-2026. `animation-range` default `normal` = `0% 100%`.
- **`prefers-reduced-motion` support:** Chrome 74+, Edge 79+, Firefox 63+, Safari 10.1+ (effectively universal).
- **WCAG 2.2 SC 2.3.3** (AAA, "Animation from Interactions"): users must be able to disable scroll-triggered motion. Not a timing budget — the 100ms budget is Google RAIL.

## Tools & libraries

- **GSAP + ScrollTrigger** — industry standard for timeline-based scroll animation; handles pinning, scrubbing, batching. `gsap.registerPlugin(ScrollTrigger)`. Use for any non-trivial pinned timeline.
- **Lenis** (`darkroomengineering/lenis`) — ~3 kB smooth scroll for marketing/portfolio sites. `lenis/react` ships `<ReactLenis>`. NOT for apps/checkouts.
- **GSAP ScrollSmoother** — Club plugin; smooth scroll + ScrollTrigger in one, no proxy wiring. Use instead of Lenis if you have Club.
- **Native CSS scroll-driven animations** — `scroll()`/`view()`; use first for progress bars, reveals, simple parallax where supported (off-main-thread perf).
- **IntersectionObserver** — native JS trigger-on-enter; valid on all browsers for reveal patterns. Prefer `animation-timeline: view()` (Chrome 115+, Safari 18+) for declarative entry cases — it runs off the main thread.
- **Framer Motion (Motion for React)** — declarative animation + `useReducedMotion()` hook.
- **Barba.js** — page transitions, often paired with GSAP + Lenis on award sites.
- **Spline / Three.js** — 3D hero scenes and complex scroll sequences when sprite/WebP frames aren't enough.
- **Locomotive Scroll** — competing smooth scroll; heavier than Lenis. **ScrollMagic** — legacy, mostly replaced by ScrollTrigger; avoid for new work.
- See [Web Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md) for the full stack picture.

## Do / Don't

**Do**
- Justify every effect by narrative or attention; cut the rest.
- Animate `transform`/`opacity` only.
- Pin containers, animate their children.
- Create triggers top-to-bottom in DOM order.
- Wrap motion in `prefers-reduced-motion: no-preference` + `@supports`; in JS use `gsap.matchMedia()` not a one-shot `matchMedia().matches`.
- Call `ScrollTrigger.refresh()` after assets/fonts load.
- Use `overflow: clip` on scroll containers.
- Provide an affordance for horizontal scroll.
- Set `gsap.ticker.lagSmoothing(0)` with Lenis.
- Use WebP image sequences for frame scrubbing.
- Clean up triggers on unmount (`ctx.revert()`).

**Don't**
- Animate `width`/`height`/`top`/`left`/`margin`/`box-shadow`/`background-color`.
- Animate the pinned element itself.
- Put `content-visibility: auto`, `transform`, or `will-change` on ancestors of pinned elements.
- Use `overflow: hidden` on scroll-driven containers.
- Apply smooth scroll to apps, dashboards, forms, or checkouts.
- Reveal-fade every paragraph of body text.
- Animate big number counters (0 → 150,243) on entry — zero info value, wastes attention.
- Ship `markers: true` to production.
- Double-tick Lenis (`autoRaf` + manual `raf()`).
- Leave reduced-motion users with stuck `opacity: 0` content.

## Common pitfalls & how to avoid them

- **Animating the pinned element** → breaks all position calc. Animate only children.
- **`content-visibility: auto` near a pin** (GSAP #465) → sections jump past boundaries. Remove it from ancestors/siblings.
- **Missing `lagSmoothing(0)`** with Lenis → 1–2 frame jitter between scroll position and markers. Add it.
- **Out-of-DOM-order triggers** → cascading miscalculation from pin-spacer insertion. Create top-to-bottom.
- **Double-ticking Lenis** → double-speed scroll. Run `lenis.raf()` from the ticker OR `autoRaf`, never both.
- **`overflow: hidden`** → breaks CSS scroll-seeking. Use `overflow: clip`.
- **`transform`/`will-change` on pin ancestors** → breaks `position: fixed`. Restructure HTML; `pinReparent: true` only as last resort.
- **iOS pin jumping** (address bar resize) → use `ScrollTrigger.normalizeScroll(true)` / `dvh` units.
- **No `ScrollTrigger.refresh()` after load** → triggers set before fonts/lazy images have wrong positions.
- **Over-revealing text** → fast scrollers wait to read. Reveal sections, not paragraphs; keep budgets tight.
- **`<video>` seeking for frame sequences** → janky bidirectional scrub. Use WebP image sequence + canvas.
- **No SPA cleanup** → memory leaks and worsening jank per visit. `ctx.revert()` on unmount.
- **Smooth scroll on interactive pages** → broken muscle memory, higher click-target error rates.

## What real users complain about

- **"I scroll to the bottom then come back up to read."** Over-applied reveal-on-scroll makes content unreadable for fast scrollers.
- **"Where did the rest go?"** Horizontal scroll sections with no visual cue read as the section ending.
- **Motion sickness / nausea.** Heavy parallax and large scroll motion trigger vestibular symptoms; missing reduced-motion support is a real harm, not a nit.
- **"The page fights me."** Smooth-scroll hijacking applied where users expect native scroll (apps, long content) breaks expectations and feels laggy.
- **"It got slower the more I clicked around."** Leaked ScrollTriggers in SPAs accumulate jank across route changes.
- **"Pretty but pointless."** Animated counters and motion-for-motion's-sake waste seconds of attention with no information value — the page feels like a tech demo, not a product.

## Senior-level checklist

- [ ] Every scroll effect is justified by narrative pacing or attention; unjustified ones removed.
- [ ] Page type is experience-driven (microsite/showcase/portfolio), not task-driven.
- [ ] Only `transform`/`opacity` animated; verified no layout-thrashing properties in keyframes.
- [ ] Pinned elements animate children only; no `transform`/`will-change`/`content-visibility: auto` on their ancestors.
- [ ] Triggers created top-to-bottom in DOM order; nested pins use `pinnedContainer`.
- [ ] `ScrollTrigger.refresh()` runs after fonts + images + layout settle.
- [ ] Lenis (if used) wired with `lenis.on('scroll', ScrollTrigger.update)`, ticker-driven `raf`, `gsap.ticker.lagSmoothing(0)`, no double-tick; absent on apps/forms/checkouts.
- [ ] Native CSS scroll animations declare `animation-timeline` after the shorthand, include `animation-fill-mode: both`, scoped with named ranges, in `@supports` + reduced-motion guards.
- [ ] `overflow: clip` (not `hidden`) on scroll-animation containers.
- [ ] Reveal animations < 400ms, stagger < 200ms; body copy not faded line-by-line.
- [ ] Horizontal scroll has a discoverability affordance; mobile uses native `scroll-snap`.
- [ ] Frame sequences use WebP images + canvas, with low→high quality progressive load.
- [ ] `prefers-reduced-motion: reduce` disables motion AND Lenis, leaving final content fully visible. In GSAP: handled via `gsap.matchMedia()` so it re-evaluates if user changes OS setting mid-session.
- [ ] Base layout is readable with JS and animation disabled (progressive enhancement).
- [ ] iOS pin jump handled (`normalizeScroll` / `dvh`); tested on real touch devices.
- [ ] `will-change` applied just-in-time and removed after; no over-promotion.
- [ ] `markers: true` removed from production.
- [ ] SPA: triggers scoped with `gsap.context()` and reverted on unmount.
- [ ] Core Web Vitals and the 100ms response budget verified under throttling.

## Sources
- GSAP ScrollTrigger documentation — https://gsap.com/docs/v3/Plugins/ScrollTrigger/
- Lenis (darkroomengineering) — https://github.com/darkroomengineering/lenis
- CSS scroll-driven animations (MDN) — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations
- scroll-timeline polyfill — https://github.com/flackr/scroll-timeline
- prefers-reduced-motion (MDN) — https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- WCAG 2.2 — https://www.w3.org/TR/WCAG22/

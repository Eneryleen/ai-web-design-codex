# Playbook: Build a Scroll-Driven One-Pager

> An end-to-end, step-by-step playbook for building a magic-scroll, award-tier one-page experience: write the narrative, storyboard the beats, pick the right stack, implement pinning/parallax/reveal, hit Core Web Vitals, gate everything behind `prefers-reduced-motion`, simplify on mobile, QA, and ship. Outcome: a senior-level scrollytelling site that animates only `transform`/`opacity`, stays legible at every scroll position, and degrades gracefully.

**Apply when:** building a marketing one-pager, portfolio, or product story where scroll *is* the interaction. · **Related:** [Scroll-Driven Experiences](../03-site-types/scroll-driven-experiences.md) · [Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [Single-Page Sites & One-Pagers](../03-site-types/single-page-one-pagers.md)

## TL;DR — rules at a glance

- **Narrative first.** Write the 3–4 act story in plain prose before opening a design tool. If you can't summarize the arc in 2 sentences, it's not a scrollytelling project yet.
- **Storyboard before code.** Low-fi frames per scene expose ~80% of narrative inconsistencies before they get expensive.
- **Pick the lightest stack that works.** Content site → native CSS `scroll-timeline` (~0 JS). One-pager → Lenis + GSAP ScrollTrigger (~150 KB). Award-tier WebGL → Astro + Lenis + GSAP + R3F + r3f-scroll-rig.
- **Animate `transform` and `opacity` only.** They run on the compositor thread; everything else triggers layout/paint. Add `force3D: true` for GPU promotion.
- **Pinned scenes stay short.** 1.5–2× viewport height max; `end: '+=2000'` is the practical ceiling before users feel trapped.
- **Use the canonical Lenis↔GSAP sync** with `gsap.ticker.lagSmoothing(0)`. Never run Lenis and GSAP ScrollSmoother together.
- **Package is `lenis`,** not `@studio-freight/lenis`. v1.0 dropped `smoothTouch`; use `syncTouch: true`.
- **CSS scroll-timeline uses `overflow: clip`,** never `overflow: hidden` (hidden breaks scroll-seeking). Put `animation-timeline` *after* the `animation` shorthand. Supported in all major browsers as of 2025 (Chrome 116+, Firefox 110+, Safari 18+).
- **`ScrollTrigger.config({ ignoreMobileResize: true })`** on init or mobile address-bar resize triggers constant refresh and jank.
- **Initialize after fonts.** Always run `initScrollAnimations()` inside `document.fonts.ready.then(...)`. Web fonts swap layout after DOMContentLoaded, shifting all trigger positions.
- **Gate all motion** behind `@media (prefers-reduced-motion: no-preference)` + `@supports`. Replace motion with opacity/color — don't just kill feedback.
- **Mobile is first-class.** Disable scroll-jacking and WebGL scrolling on mobile; serve a static/stepped fallback.
- **Clean up GSAP.** `useGSAP()` with a scoped ref in React; manually kill ScrollTriggers on Astro View Transitions. Otherwise: phantom triggers.
- **Never lazy-load the LCP hero.** Use `fetchpriority="high"`. Always set explicit `width`/`height` on images.
- **Provide a pause control** for any auto-playing motion — WCAG 2.2.2 requires it independently of `prefers-reduced-motion`.

---

## 1. Plan the narrative (before any design)

Scrollytelling is film, not a slideshow. The scroll position is the playhead; the user controls time.

1. **Write the story in prose.** One paragraph. Identify the **acts**: tension → reveal → payoff (3–4 beats max).
2. **Define the through-line.** A single sentence: "As you scroll, X transforms into Y, proving Z."
3. **Gut check:** if the arc can't be summarized in **2 sentences**, you don't have a scrollytelling project — you have a list section that should just be a normal page.
4. **Map each act to a content goal.** Every beat must deliver one idea the user *reads*, not just watches. Animation is the delivery mechanism, never the message.

> **Legibility law:** something meaningful must be visible and readable at *every* scroll position. Never sacrifice legibility to the animation. If a frame has motion but nothing to read, cut it.

## 2. Storyboard the scroll beats

Sketch each scene as a low-fi frame (paper, Figma, or ASCII) *before* writing CSS/JS.

For each beat capture:

| Field | Example |
|---|---|
| Scene # / act | 2 of 4 — "the reveal" |
| Trigger | Section top hits viewport top |
| Mechanic | Pin + horizontal scrub |
| Scroll distance | `+=1800px` (~1.5 viewports) |
| What stays readable | Headline pinned top-left throughout |
| Entry / exit state | Cards off-screen right → settled grid |
| Reduced-motion fallback | Cards fade in, no horizontal travel |
| Mobile fallback | Vertical stack, native scroll, no pin |

**Scroll budget:** sum all pinned distances. If total pinned scroll exceeds **~6000px** (3 × 2000px max scenes) on a standard 900px viewport, cut beats. A page that requires scrolling **more than ~3× its visible content height** to clear animations will lose users on mobile before they reach the CTA. Keep each pin ≤ **~2000px (≈ 2× viewport height)**; prefer 1500px.

## 3. Choose the stack (decision tree)

```
Is it pure content / blog / docs?
  └─ YES → native CSS scroll-timeline + view()    (0 JS, off-main-thread)
Is it a marketing one-pager, moderate animation?
  └─ YES → Lenis + GSAP ScrollTrigger             (~150 KB total)
Award-tier portfolio with WebGL / 3D?
  └─ YES → Astro + Lenis + GSAP + R3F + r3f-scroll-rig
Want minimal JS but some choreography?
  └─ YES → Astro, GSAP scoped to island components only
```

- **Never** run Lenis *and* GSAP ScrollSmoother at once — two smoothers fight for the scroll.
- Prefer CSS scroll-timeline when you can: it runs **off the main thread** with zero JS overhead.
- Reach for WebGL only when the narrative demands depth/material that DOM can't express. WebGL is a budget sink (draw calls, VRAM, decode).

## 4. Wire up smooth scroll (Lenis + GSAP)

Package is **`lenis`** (the `@studio-freight/lenis` name is deprecated). v1.0 (Sept 2024) dropped `smoothTouch`; use `syncTouch: true`.

**Canonical sync — memorize this exactly:**

```js
import Lenis from 'lenis';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

const lenis = new Lenis({ syncTouch: true }); // touch smoothing; replaces smoothTouch

// 1. Tell ScrollTrigger to update on every Lenis scroll event
lenis.on('scroll', ScrollTrigger.update);

// 2. Drive Lenis from GSAP's ticker (seconds → milliseconds)
gsap.ticker.add((time) => {
  lenis.raf(time * 1000);
});

// 3. MANDATORY: kill lag smoothing or GSAP delays the scroll animation
gsap.ticker.lagSmoothing(0);
```

**Also required:**

```css
/* scroll-behavior: smooth on <html> breaks ScrollTrigger refresh timing.
   Let Lenis own smoothing instead. */
html { scroll-behavior: auto !important; }
```

- `autoRaf: true` gives a zero-config Lenis, but if you're syncing with GSAP's ticker (above), do **not** also enable `autoRaf` — you'd run two rAF loops.
- React: `import { ReactLenis } from 'lenis/react'` and wrap the app; pass `root` to drive window scroll.

**Mobile address-bar jank fix — add this once at initialization:**

```js
// Mobile browsers fire a window resize on every address-bar show/hide during scroll.
// Without this, ScrollTrigger.refresh() runs constantly → jank.
ScrollTrigger.config({ ignoreMobileResize: true });
```

**Cross-device scroll normalization (optional, desktop/laptop trackpads):**

```js
// Normalizes scroll speed across devices. Do NOT combine with Lenis — pick one.
// Use only when NOT using Lenis.
ScrollTrigger.normalizeScroll(true);
```

## 5. Implement pinning, parallax, reveal (GSAP ScrollTrigger)

### Pin a scene

```js
ScrollTrigger.create({
  trigger: '.scene',
  start: 'top top',
  end: '+=2000',          // scroll distance the user travels through the pinned scene
  pin: true,
  scrub: 1,               // 1 = 1s catch-up smoothing; true = locked to scroll
  // markers: true,       // dev only — remove before shipping
});
```

- **Scene sizing:** minimum meaningful pin ≈ **500px**; typical rich 3-beat scene ≈ **1500px**; user-tolerance ceiling ≈ **2000px** — beyond that, users feel trapped regardless of how good the animation is. "Sweet spot" and "ceiling" are the same range; do not exceed 2000px thinking it is safe if the animation is interesting.
- `scrub: 1` adds 1s of inertia between scroll and animation; `scrub: true` ties the playhead 1:1.

### Reveal on enter (no FOUC)

```css
.reveal { opacity: 0; }   /* set hidden in CSS so it never flashes visible */
```

```js
gsap.utils.toArray('.reveal').forEach((el) => {
  gsap.fromTo(el,
    { opacity: 0, y: 40 },
    {
      opacity: 1, y: 0, force3D: true,
      scrollTrigger: { trigger: el, start: 'top 80%' },
    }
  );
});
```

- **Why `fromTo`, not `from`:** `gsap.from({ autoAlpha: 0 })` renders the element *visible* first, then flashes hidden when JS loads (FOUC). Set `opacity: 0` in CSS + `fromTo` from 0→1.
- **Why `forEach`:** one tween targeting many elements animates them *all* from a single trigger position. Loop to give each its own trigger.

### Parallax (transform only)

```js
gsap.to('.bg-layer', {
  yPercent: -20, ease: 'none', force3D: true,
  scrollTrigger: { trigger: '.section', start: 'top bottom', end: 'bottom top', scrub: true },
});
```

Use `yPercent`/`xPercent` (transforms), never `top`/`left` (layout).

### Responsive variants

```js
const mm = gsap.matchMedia();
mm.add('(min-width: 768px)', () => {
  // desktop pin + scrub timeline here; auto-reverted at breakpoint exit
});
mm.add('(max-width: 767px)', () => {
  // mobile: simple fades, no pin
});
```

## 6. Native CSS scroll-timeline (the JS-free path)

Best for content sites: off-main-thread, ~0 bundle. Chrome 116+, Firefox 110+, Safari 18+ (Sept 2024) — all major engines support it as of 2025. **Polyfill (`flackr/scroll-timeline`) is only needed for older Safari 17 or Firefox < 110** — check your analytics before shipping it (~5 KB extra).

```css
@supports ((animation-timeline: scroll()) and (animation-range: 0% 100%)) {
  @media (prefers-reduced-motion: no-preference) {
    .progress {
      transform-origin: left;
      animation: grow linear;            /* shorthand FIRST */
      animation-timeline: scroll(root);  /* timeline AFTER shorthand to avoid cascade override */
    }
    @keyframes grow { from { transform: scaleX(0); } to { transform: scaleX(1); } }

    /* Reveal as element enters viewport */
    .card {
      animation: fade-in linear both;
      animation-timeline: view();
      animation-range: entry 0% cover 40%;
    }
    @keyframes fade-in { from { opacity: 0; transform: translateY(40px); } to { opacity: 1; transform: none; } }
  }
}

/* Containers that clip overflow MUST use clip, not hidden —
   hidden breaks scroll-seeking on the timeline. */
.timeline-host { overflow: clip; }
```

## 7. WebGL tier (only when justified)

Stack: `@react-three/fiber` + `@react-three/drei` + `@14islands/r3f-scroll-rig`. Single `GlobalCanvas` outside the router; single `SmoothScrollbar`.

- **Render on demand:** `frameloop="demand"` so the GPU idles when nothing changes.
- **All Three.js mutation lives in `useFrame`,** never React state (state re-renders the React tree per frame = death).
- **Compress geometry — Draco:** `gltf-pipeline -i source.glb -o output.glb --draco.compressionLevel=10` (max=10). Cuts glTF geometry **90–95%** (up to 10×); decode runs in a Web Worker, off the main thread.
- **Compress textures — KTX2 + Basis Universal:** a 200 KB PNG decompresses to **20 MB+ of VRAM**; KTX2 stays compressed on GPU (~10× less memory).
- **Draw calls:** target **under 100** for stable 60fps. Merge geometry / instance.
- **Mobile:** r3f-scroll-rig docs explicitly recommend **disabling SmoothScrollbar and all scrolling WebGL on mobile** — usually laggy. Serve a poster image instead.
- r3f-scroll-rig reads `getBoundingClientRect()` **once on mount** then tracks via IntersectionObserver + ResizeObserver — don't add your own per-frame layout reads.

## 8. Performance — hit Core Web Vitals

- **Animate `transform` + `opacity` only.** These are the only properties that run entirely on the compositor without layout or paint. Animating `left/top/width/height` triggers reflow.
- **`will-change` discipline:** the browser reserves GPU memory for *every* element carrying it, even idle. Apply immediately before animating, remove after. Never blanket-apply.
- **Throttle to one update per frame.** Scroll events fire **100+/sec** on high-refresh displays; the frame budget is **16.67ms (60fps)**. Always route through rAF (Lenis/GSAP ticker already does this).
- **LCP < 2.5s:** never lazy-load the hero/LCP image; `fetchpriority="high"`, preload it.
- **CLS < 0.1:** reserve exact placeholder dimensions for anything async (images, embeds, fonts). Always set explicit `width`/`height` on `<img>`.
- **INP < 200ms** (replaced FID, March 2024; ~43% of sites fail it): keep scroll handlers thin; move heavy work off the main thread.
- **Refresh after layout changes:** call `ScrollTrigger.refresh()` in any content-loaded/AJAX callback; otherwise triggers sit at stale positions.
- **Font loading is a trigger-position hazard.** Web fonts swap in after DOM ready and shift layout, invalidating all ScrollTrigger positions. Always initialize inside `document.fonts.ready`:

```js
document.fonts.ready.then(() => {
  initScrollAnimations();
  ScrollTrigger.refresh();
});
```

- **CSS containment for animated sections.** On scroll-animated sections that don't affect siblings, add `contain: layout style paint` to reduce the browser's recalc scope per scroll tick.

## 9. Accessibility & reduced motion

- **Gate everything.** Wrap *all* scroll animation (CSS and JS) in `@media (prefers-reduced-motion: no-preference)`.
- **Don't just disable — substitute.** Replace motion with opacity/color transitions so functional feedback (progress, state change) survives.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    /* 0.001ms not 0ms: some browsers skip animation-end callbacks at exactly 0,
       which breaks JS code that waits for animationend/transitionend. */
    animation-duration: 0.001ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.001ms !important;
    scroll-behavior: auto !important;
  }
}
```

```jsx
// React: reacts live if the user flips the OS setting mid-session
// 'motion/react' = Motion v11+ (formerly framer-motion); also works with 'framer-motion'
import { useReducedMotion } from 'motion/react'; // or 'framer-motion'
const reduce = useReducedMotion();
```

```js
// GSAP-native approach (preferred when already using GSAP — no extra dep)
gsap.matchMedia().add('(prefers-reduced-motion: no-preference)', () => {
  // all scroll animations here; auto-reverted when condition is false
});
```

- **WCAG 2.2.2 (Pause/Stop/Hide)** requires a pause/stop control for auto-playing motion **regardless** of `prefers-reduced-motion`. Don't conflate the two requirements.
- **WCAG 2.3.3 (Animation from Interactions, AAA):** scroll-driven motion should be disable-able. `prefers-reduced-motion` satisfies the spirit.
- Keep focus order and keyboard navigation intact — pinned/scrubbed sections must still be reachable and readable without scrolling animation.

## 10. Mobile fallback

- **Disable scroll-jacking** on touch. At best, *dramatically* simplify: drop pins, scrubs, and parallax.
- Serve a **stepped or static** version: native scroll, fades on enter (Scrollama or IntersectionObserver), no horizontal scrub.
- WebGL: swap the canvas for a poster image (see §7).
- Use `gsap.matchMedia()` to build the mobile set, not CSS hacks over the desktop timeline.

## 11. Cleanup in SPAs (React & Astro)

```jsx
// React — useGSAP scopes + auto-cleans all GSAP created inside it
import { useGSAP } from '@gsap/react';
const container = useRef(null);
useGSAP(() => {
  gsap.to('.box', { x: 200, scrollTrigger: '.box' });
}, { scope: container }); // selectors scoped to container; reverted on unmount
```

```js
// Vanilla JS / Astro — gsap.context() is the non-React equivalent of useGSAP().
// All GSAP created inside is auto-reverted when ctx.revert() is called.
const ctx = gsap.context(() => {
  gsap.to('.box', { x: 200, scrollTrigger: '.box' });
}, containerElement); // scoped selector root

// Cleanup:
ctx.revert();
```

```js
// Astro with View Transitions — kill before swap, re-init after
document.addEventListener('astro:before-swap', () => {
  ScrollTrigger.getAll().forEach((t) => t.kill());
  // also revert SplitText instances before swap
  ctx?.revert();
});
document.addEventListener('astro:page-load', () => {
  initScrollAnimations(); // re-create on each navigation
});
```

> Astro gotcha: ScrollTrigger `markers` and triggers may misbehave with the View Transitions API until a full reload — manual kill/reinit on navigation events is required.

## Concrete specs & numbers

- **Frame budget:** 16.67ms/frame (60fps). Handler work past this = visible jank.
- **Bundles:** Lenis ~3 KB gzip · GSAP core + ScrollTrigger + SplitText ~150 KB total.
- **Pin distance:** min meaningful ~500px · typical rich scene ~1500px · user-tolerance ceiling ~2000px (`end: '+=2000'`). Do not exceed 2000px.
- **CSS scroll-timeline support (2025+):** Chrome 116+ · Firefox 110+ · Safari 18+ — all major engines. Polyfill only needed for Safari ≤ 17 / Firefox < 110.
- **Draco:** 90–95% geometry reduction (up to 10×); decode in Web Worker. Max `compressionLevel=10`.
- **Textures:** 200 KB PNG → 20 MB+ VRAM; KTX2 + Basis ~10× less GPU memory.
- **Draw calls:** target < 100 for stable 60fps.
- **Core Web Vitals:** LCP < 2.5s · INP < 200ms (~43% of sites fail) · CLS < 0.1.
- **Scroll event rate:** 100+/sec on high-refresh; throttle to ~16ms (one per frame).
- **Rule of thumb:** Draco + KTX2 typically moves mobile WebGL scenes from slideshow (< 30fps) to usable (45+ fps) by cutting geometry transfer and VRAM pressure. Measure your own baseline; do not skip profiling.

## Tools & libraries

- **`lenis`** — smooth scroll, ~3 KB. Use `syncTouch: true`. React: `lenis/react` → `ReactLenis`. *Not* `@studio-freight/lenis`.
- **GSAP ScrollTrigger** — `npm i gsap`; `gsap.registerPlugin(ScrollTrigger)`. Key opts: `pin`, `scrub`, `trigger`, `start`, `end`, `toggleActions`, `refreshPriority`, `invalidateOnRefresh`, `immediateRender`.
- **`@gsap/react`** — `useGSAP()` hook, scoped auto-cleanup. Use instead of manual `useEffect` cleanup.
- **`gsap.context(fn, root)`** — vanilla-JS/Astro equivalent of `useGSAP`. Scopes selectors to `root`; call `.revert()` to kill all contained GSAP. Use in Astro `astro:before-swap` cleanup.
- **`ScrollTrigger.config({ ignoreMobileResize: true })`** — prevents constant `refresh()` on mobile address-bar resize events. Add once at init.
- **`ScrollTrigger.normalizeScroll(true)`** — cross-device scroll normalization. Do NOT use alongside Lenis (conflict).
- **CSS scroll-timeline** — `animation-timeline: scroll()/view()`, `animation-range`, `scroll-timeline-name`, `view-timeline-name`, `timeline-scope`. Zero JS. Use when content-first.
- **`@react-three/fiber` + `drei`** — WebGL in React. `frameloop="demand"`. Mutate in `useFrame`.
- **`@14islands/r3f-scroll-rig`** — sync R3F meshes to DOM scroll. Single GlobalCanvas + SmoothScrollbar. Disable on mobile.
- **Scrollama.js** — IntersectionObserver step triggers. Use for step-based narrative *without* smooth scrub; lighter than ScrollTrigger.
- **GSAP SplitText** — per-char/word/line reveals. Revert before Astro View Transitions, re-init after.
- **Motion / Framer Motion** — `useReducedMotion()` reactive hook (stays in sync if user flips OS setting).
- **`IntersectionObserver` direct** — use a small custom helper instead of Sal.js (last release 2019, unmaintained). 10 lines of JS gets you threshold-based reveals with zero dependencies.
- **gltf-pipeline** — Draco compression CLI. `gltf-pipeline -i src.glb -o out.glb --draco.compressionLevel=10`.
- **Barba.js** — page-transition manager for multi-page sites; pairs with GSAP Flip + SplitText. Out of scope for true one-pagers.
- **Don't use:** GSAP ScrollSmoother *together with* Lenis; `scroll-behavior: smooth` with ScrollTrigger; full GSAP for a 3-fade landing page.

## Do / Don't

**Do**
- Write the narrative and storyboard before any code.
- Animate only `transform`/`opacity`; add `force3D: true`.
- Use the canonical Lenis↔GSAP sync + `lagSmoothing(0)`.
- Keep pins ≤ ~2000px; show scroll progress on pinned scenes.
- Use function-based `start`/`end` + `invalidateOnRefresh: true`.
- One ScrollTrigger per element via `forEach`.
- Gate motion behind `prefers-reduced-motion` *and* `@supports`.
- Provide a pause control for auto-play (WCAG 2.2.2).
- `fetchpriority="high"` on the hero; explicit `width`/`height` on images.
- `useGSAP({ scope })` / `gsap.context()` / `astro:before-swap` cleanup.
- `ScrollTrigger.config({ ignoreMobileResize: true })` at init.
- Initialize inside `document.fonts.ready.then(...)`, then call `ScrollTrigger.refresh()`.

**Don't**
- Don't ship a scrollytelling site without a 2-sentence arc.
- Don't run Lenis + ScrollSmoother together.
- Don't animate `left/top/width/height`.
- Don't use `gsap.from({ autoAlpha: 0 })` (FOUC) — use `fromTo` + CSS `opacity: 0`.
- Don't use `overflow: hidden` with CSS scroll-timeline — use `overflow: clip`.
- Don't lazy-load the LCP hero.
- Don't blanket-apply `will-change`.
- Don't leave `markers: true` in production.
- Don't scroll-jack on mobile.
- Don't only disable motion for reduced-motion users — substitute feedback.

## Common pitfalls & how to avoid them

- **ScrollTrigger on a tween nested in a timeline** → parent playhead fights the scroll playhead. Fix: one ScrollTrigger on the parent timeline, or keep tweens independent.
- **One tween, many targets** → all animate from a single trigger. Fix: `forEach` → one tween+trigger each.
- **Pinned triggers created out of scroll order** → later triggers ignore pin-added spacing. Fix: create in scroll order, or assign `refreshPriority` — higher value = calculated earlier. Pattern: outermost/first-in-page trigger gets the highest number (e.g. `refreshPriority: 3`), nested/later ones lower (e.g. `refreshPriority: 1`). Default is `0`.
- **Hard-coded `end: '+=' + el.offsetHeight`** caches on creation, breaks on resize. Fix: function value `end: () => '+=' + el.offsetHeight` + `invalidateOnRefresh: true`.
- **`scroll-behavior: smooth` on `<html>`** delays refresh scroll-to-zero, misaligning positions. Fix: `html { scroll-behavior: auto !important; }`; let Lenis smooth.
- **`overflow: hidden` in CSS scroll-timeline** clips the scroll-seeking mechanism. Fix: `overflow: clip`.
- **FOUC from `from({ autoAlpha: 0 })`.** Fix: CSS `opacity: 0` + `gsap.fromTo(0→1)`.
- **Two sequential `gsap.to()` on the same property** → second jumps back to the cached start. Fix: `immediateRender: false` on the second, or `.fromTo()`, or one timeline + one trigger.
- **Async content shifts layout post-init** → triggers at wrong positions. Fix: `ScrollTrigger.refresh()` in the loaded callback; set explicit image dimensions.
- **Web fonts shift layout after DOMContentLoaded** → all trigger positions stale. Fix: wrap `initScrollAnimations()` + `ScrollTrigger.refresh()` in `document.fonts.ready.then(...)`.
- **Mobile address-bar show/hide fires `window resize`** → triggers constant `ScrollTrigger.refresh()` → jank. Fix: `ScrollTrigger.config({ ignoreMobileResize: true })` once at initialization.
- **No cleanup in React/Astro SPAs** → duplicate/phantom triggers. Fix: `useGSAP({ scope })` / kill on `astro:before-swap`.
- **Animating layout props** triggers reflow. Fix: transforms + opacity only.
- **`will-change` everywhere** wastes GPU memory. Fix: add right before animating, remove after.
- **No pause/stop control** breaks WCAG 2.2.2 — independent of reduced-motion.
- **Scroll-jacking with no affordance** → jumpy for normal scrollers, skipped sections for fast ones. Fix: progress indicator + pin < ~2000px.

## What real users complain about

- **"I scrolled forever and nothing happened."** Over-long pins / oversized scroll budget. Cap pins; keep each beat meaningful.
- **"It felt laggy / juddery."** Non-compositor properties animated, or scroll handler over budget. Transforms+opacity only; throttle to rAF.
- **"I couldn't read the text — it moved too fast."** Motion outran legibility. Keep something readable at every position; slow scrub on text-bearing beats.
- **"It hijacked my scroll and I felt trapped."** Scroll-jacking without a progress indicator or exit. Show progress; keep pins short; never disable native scroll fully.
- **"Brutal on my phone — battery and stutter."** WebGL/smooth-scroll left on for mobile. Disable scroll-jacking and WebGL on touch; serve static fallback.
- **"Animations played even though I asked the OS to reduce motion."** Missing `prefers-reduced-motion` gate. Wrap all motion; substitute, don't just delete.
- **"The hero took forever to show."** Lazy-loaded LCP image. Prioritize it.

## Senior-level checklist

- [ ] Narrative arc summarized in ≤ 2 sentences; 3–4 acts defined.
- [ ] Storyboard frame per scene with entry/exit, reduced-motion + mobile fallback noted.
- [ ] Stack chosen via the decision tree (lightest that works).
- [ ] Lenis is `lenis` package, `syncTouch: true`; canonical GSAP sync + `lagSmoothing(0)`.
- [ ] `html { scroll-behavior: auto !important; }` set.
- [ ] Only `transform`/`opacity` animated; `force3D: true` where needed.
- [ ] Each animated element has its own ScrollTrigger (`forEach`).
- [ ] `start`/`end` are function-based with `invalidateOnRefresh: true`.
- [ ] Pins ≤ ~2000px; progress indicator on pinned scenes.
- [ ] No FOUC: CSS `opacity: 0` + `fromTo`.
- [ ] CSS scroll-timeline guarded by `@supports`; containers use `overflow: clip`; `animation-timeline` after shorthand.
- [ ] WebGL: Draco + KTX2, `frameloop="demand"`, < 100 draw calls, disabled on mobile.
- [ ] LCP hero not lazy-loaded (`fetchpriority="high"`); all images have width/height.
- [ ] CWV verified: LCP < 2.5s, INP < 200ms, CLS < 0.1.
- [ ] All motion gated by `prefers-reduced-motion`; feedback substituted, not deleted.
- [ ] Pause/stop control present for any auto-play (WCAG 2.2.2).
- [ ] Mobile fallback: scroll-jacking/WebGL off, native scroll, stepped reveals.
- [ ] GSAP cleanup verified (no phantom triggers after navigation).
- [ ] `markers: true` removed; `ScrollTrigger.refresh()` wired to async content.
- [ ] `document.fonts.ready` gate: scroll animations initialized only after fonts load.
- [ ] `ScrollTrigger.config({ ignoreMobileResize: true })` set at init.
- [ ] `gsap.context()` (vanilla/Astro) or `useGSAP({ scope })` (React) used — no manual `useEffect` kill loops.
- [ ] Keyboard + focus order intact through pinned sections.

## Sources

- GSAP ScrollTrigger docs — https://gsap.com/docs/v3/Plugins/ScrollTrigger/
- GSAP common ScrollTrigger mistakes — https://gsap.com/resources/st-mistakes/
- GSAP `gsap.context()` — https://gsap.com/docs/v3/GSAP/gsap.context()/
- useGSAP() hook — https://gsap.com/resources/React/
- Lenis — https://github.com/darkroomengineering/lenis
- CSS scroll-driven animations (MDN) — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations
- CSS scroll-driven animations browser support (caniuse) — https://caniuse.com/css-scroll-driven-animations
- scroll-timeline polyfill — https://github.com/flackr/scroll-timeline
- r3f-scroll-rig — https://github.com/14islands/r3f-scroll-rig
- React Three Fiber — https://r3f.docs.pmnd.rs/
- gltf-pipeline (Draco) — https://github.com/CesiumGS/gltf-pipeline
- Core Web Vitals (INP) — https://web.dev/articles/inp
- prefers-reduced-motion (MDN) — https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- WCAG 2.2.2 Pause, Stop, Hide — https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html
- Scrollama — https://github.com/russellgoldenberg/scrollama

# Web Animation Libraries & Stack
> Decision framework for choosing and combining web animation tech — CSS, WAAPI, View Transitions, GSAP, Motion, Lenis, Anime.js, Lottie, AutoAnimate. Optimizes for the senior outcome: 60 fps, minimal bundle, accessible motion, and the right tool per job instead of reflexively reaching for a library.

**Apply when:** adding any motion to a site — micro-interactions, scroll effects, page transitions, layout animations, or motion graphics. · **Related:** [Micro-interactions & Motion Design](../07-aesthetics-and-trends/micro-interactions-and-motion.md) · [Scroll-Driven Experiences](../03-site-types/scroll-driven-experiences.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md)

## TL;DR — rules at a glance
- **Order of escalation: CSS → WAAPI → library.** Don't add a dependency until complexity genuinely requires it.
- **Animate only `transform`, `opacity` for guaranteed compositor performance.** `filter` and `clip-path` are usually composited but can rasterize on heavy/complex values — profile on mobile. Animating `width`/`height`/`top`/`left`/`margin` triggers layout every frame — the #1 performance killer.
- **`transform: translate()` instead of `top`/`left`; `transform: scale()` instead of `width`/`height`.**
- **Easing: ease-out for entering, ease-in for exiting.** Linear motion looks robotic. Enter slightly longer than exit (~300 ms in / 200 ms out).
- **Always honor `prefers-reduced-motion`.** Vestibular disorders affect ~35% of US adults over 40 (NIH/NIDCD); parallax/scroll-hijack can cause nausea in this population.
- **Stack picks fast:** React/Vue app → **Motion (MIT)**; complex marketing/portfolio, SVG morph, timeline → **GSAP**; After Effects asset → **Lottie**; list reorders → **AutoAnimate**; MPA page transitions → **CSS View Transitions**.
- **GSAP license trap:** free since Webflow acquired it (Aug 2023), but you may **not** use it in tools that compete with Webflow (no-code/web-builders). Motion is the MIT-safe alternative.
- **Don't put Motion on non-interactive server-rendered content** — it forces `'use client'`. The full `motion` component is ~18 KB gzip; the old Framer Motion package (non-modular) ~32 KB. Either is dead weight vs. CSS keyframes at 0 KB.
- **CSS custom properties trigger paint** even in compositor values. Use `@property` with `inherits: false` for animatable typed vars.
- **Batch reads then writes.** Alternating DOM reads/writes in a loop (DOM thrashing) is the cardinal performance sin.
- **`will-change: transform` sparingly** — only just before animating, remove after. Overuse = GPU memory pressure on mobile.
- **Prefer CSS scroll-driven animations** (`animation-timeline: scroll()/view()`) over JS scroll listeners — compositor vs. main thread.
- **Lottie is heavy:** one animation can add 1,400+ DOM nodes. Use `.lottie` (dotLottie) for up to 80% smaller files; canvas renderer above ~1,500 elements.

---

## The decision tree (read top to bottom; stop at first match)
```
1. Can CSS transitions / @keyframes do it?            → use CSS (0 KB)
2. Scroll-linked progress/reveal/parallax?            → CSS scroll-driven animation (compositor)
3. Page/route transition?
     SPA same-document  → View Transitions API (Baseline 2025)
     MPA cross-document → CSS View Transitions (Chrome/Edge/Safari) + instant fallback
4. List add/remove/reorder, zero-config?              → AutoAnimate
5. Needs React lifecycle / exit anim / shared element → Motion
6. Precise timeline, SVG morph, scrub, pin, stagger   → GSAP (+ScrollTrigger/Flip/SplitText)
7. Framework-agnostic mid-complexity, no React        → Anime.js v4
8. After Effects motion graphic                       → Lottie (.lottie / dotLottie)
9. Smooth scroll feel                                  → Lenis (+ ScrollTrigger if scrubbing)
```

## CSS first: transitions, keyframes, easing
Most UI motion (hover, focus, toggle, fade, slide) needs **no JS**.
```css
/* Enter: ease-out, slightly longer. Exit: ease-in, shorter. */
.card { transition: transform .3s cubic-bezier(.16,1,.3,1), opacity .3s ease-out; }
.card[data-leaving] { transition-duration: .2s; transition-timing-function: ease-in; }

@keyframes slide-up { from { opacity:0; transform: translateY(12px) } to { opacity:1; transform:none } }
.reveal { animation: slide-up .4s ease-out both; }
```
- **Compositor-safe set:** `transform`, `opacity` — reliably composited. `filter` and `clip-path` are usually composited but can fall back to rasterization (heavy blur/complex paths); profile on low-end devices before relying on them at 60 fps.
- **`both`** fill mode keeps the start/end state applied (no flash).
- Custom easing via `cubic-bezier()` or `linear()` (multi-stop springs). `ease-out` ≈ `cubic-bezier(0,0,.2,1)`; expressive decel ≈ `cubic-bezier(.16,1,.3,1)`.

### `@starting-style` + `transition-behavior` for enter transitions
CSS can now animate elements from `display: none` — no JS needed:
```css
.dialog { opacity: 0; transform: translateY(8px); transition: opacity .2s ease-out, transform .2s ease-out,
  display .2s allow-discrete, overlay .2s allow-discrete; }
.dialog[open] { opacity: 1; transform: none; }
@starting-style { .dialog[open] { opacity: 0; transform: translateY(8px); } }
```
`@starting-style` sets the start-of-enter keyframe; `transition-behavior: allow-discrete` (shorthand: `allow-discrete` in the transition value) lets discrete properties (`display`, `overlay`) participate. **Baseline 2024 — Chrome 117+, Firefox 129+, Safari 17.5+; use `@supports` guard.**

### `@property` for animatable custom properties
`var(--x)` in a transform still triggers paint and style recalc on every element that inherits it. Register a typed property so it can animate without inheriting/paint storms:
```css
@property --angle { syntax: "<angle>"; inherits: false; initial-value: 0deg; }
.spinner { background: conic-gradient(from var(--angle), …); transition: --angle .6s linear; }
```

## Web Animations API (WAAPI)
Native, **zero bundle**, JS-controlled keyframes; runs on the compositor when animating compositor properties. Use when you need JS control (dynamic values, promises, playback control) but don't want a library.
```js
const anim = el.animate(
  [{ transform: "translateY(12px)", opacity: 0 }, { transform: "none", opacity: 1 }],
  { duration: 300, easing: "cubic-bezier(0,0,.2,1)", fill: "both" }
);
await anim.finished;            // promise-based sequencing
// Scroll/view timelines: { timeline: new ViewTimeline({ subject: el }) }
```

## CSS scroll-driven animations (native, compositor)
Replaces JS scroll listeners for progress bars, reveals, parallax — runs **off the main thread**, so no jank under JS load.
```css
@keyframes grow { from { transform: scaleX(0) } to { transform: scaleX(1) } }
.progress { transform-origin: left; animation: grow linear; animation-timeline: scroll(root block); }

@keyframes fade-in { from { opacity:0; transform: translateY(20px) } to { opacity:1; transform:none } }
.section { animation: fade-in linear both; animation-timeline: view(); animation-range: entry 0% cover 40%; }
```
- Support: Chrome 115+, Edge 115+, Firefox 115+ (`scroll()`/`view()` + ranges), Safari 16.0+ (basic scroll-linked; 17.2+ for `view()` and named timelines). **Progressive enhancement** — without support, elements simply render in final state. Guard exotic ranges with `@supports (animation-timeline: scroll())`.

## CSS View Transitions API
Two-line site-wide page/state transitions. SPA wraps the DOM update; MPA opts in via CSS.
```js
// SPA / same-document
document.startViewTransition(() => { /* mutate DOM, e.g. swap route content */ });
```
```css
/* MPA / cross-document — both pages opt in */
@view-transition { navigation: auto; }
/* granular: tag the elements that should morph between pages/states */
.hero-img { view-transition-name: hero; }
::view-transition-old(root), ::view-transition-new(root) { animation-duration: .5s; }
```
- **Same-document:** Baseline Newly Available (Oct 2025) — Chrome 111+, Edge 111+, Safari 18+, Firefox 144+; >85% support.
- **Cross-document (MPA):** Chrome 126+, Edge 126+, Safari 18.2+; **Firefox not supported as of mid-2026**. Fallback = instant navigation (no animation) — test it.
- Only **one** rendered element per `view-transition-name` at a time, or the transition errors/skips.

## GSAP (GreenSock)
Framework-agnostic imperative timeline engine. Best for **complex sequencing, scrubbing, pinning, SVG morphing, text splitting** — award-site (Awwwards/FWA) territory. GSAP core ~23.5 KB gzip.
```js
import { gsap } from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);          // register every plugin explicitly

gsap.timeline({ scrollTrigger: { trigger: ".panel", start: "top center",
  end: "+=500", scrub: true, pin: true, invalidateOnRefresh: true } })
  .to(".title", { x: 200, ease: "power2.out" })
  .to(".img",   { scale: 1.2 }, "<");        // "<" = start with previous tween
```
**Plugins (all free since the Aug 2023 Webflow acquisition):** ScrollTrigger (scroll-driven), Flip (FLIP = First/Last/Invert/Play — GPU-efficient layout animations without animating `width`/`height`), Draggable, MorphSVG, SplitText, ScrollSmoother.

**React integration — always clean up:**
```js
useLayoutEffect(() => {
  const ctx = gsap.context(() => { gsap.to(".box", { x: 100 }); }, rootRef);
  return () => ctx.revert();                  // kills tweens + ScrollTriggers on unmount/HMR
}, []);
```
- **Import discipline:** `import { gsap } from "gsap"` + per-plugin imports; never assume tree-shaking does it for you.
- `pin: true` changes layout flow — call `ScrollTrigger.refresh()` after dynamic content/layout shifts; use `invalidateOnRefresh: true` for correct re-measurement.
- **Reduced motion:** `gsap.matchMedia()` auto-reverts when the query stops matching; `matchMediaRefresh()` to re-evaluate on a UI toggle.
```js
const mm = gsap.matchMedia();
mm.add("(prefers-reduced-motion: no-preference)", () => { /* animations only here */ });
```
- **License caveat:** free since Aug 2023, **closed-source**, Webflow may change terms at discretion; **forbidden in Webflow-competing builders/no-code tools.** For MIT-safe projects use Motion.

## Motion (formerly Framer Motion)
MIT, declarative, React/Vue-first. **16M downloads/month** (npm, 2025). Mini `animate()` 2.6 KB gzip; full React `motion` component ~18 KB; legacy Framer Motion package (non-modular) ~32 KB. Motion's own benchmarks claim 2.5× faster than GSAP on unknown-start-value tweens and 6× faster on cross-unit conversion — treat as directional, not authoritative.
```jsx
import { motion, AnimatePresence, useReducedMotion } from "motion/react";

const reduce = useReducedMotion();            // honor OS setting
<AnimatePresence>
  {open && (
    <motion.div key="panel"                   // stable key REQUIRED for exit
      initial={{ opacity: 0, y: 8 }} animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: 8 }}
      transition={reduce ? { duration: 0 } : { duration: .25, ease: [0,0,.2,1] }} />
  )}
</AnimatePresence>
```
- **`layout` prop** → automatic FLIP layout animations. **`layoutId`** → shared-element transitions across `AnimatePresence` (only one element with that id rendered at once).
- Hooks: `useMotionValue`, `useScroll`, `useTransform`, `useReducedMotion`.
- **Vue:** use `motion/vue` (`<Motion>` component, same API surface as React). Framework-agnostic `animate()` works anywhere.
- **Next.js App Router:** requires `'use client'`. Don't sprinkle it on static/server content — reserve for genuinely interactive client components; use CSS for the rest.

## Lenis (smooth scroll)
~3 KB, zero runtime deps, by darkroom.engineering. Wraps native scroll via a RAF loop. **Preserves `position: sticky`, anchor links, and accessibility** (unlike pre-v2 Locomotive Scroll, which breaks sticky). Direct Locomotive replacement.
```js
const lenis = new Lenis();
// Drive Lenis from GSAP's ticker — NOT a separate RAF — so ScrollTrigger stays in sync
lenis.on("scroll", ScrollTrigger.update);
gsap.ticker.add((t) => lenis.raf(t * 1000));
gsap.ticker.lagSmoothing(0);
```
- **Not a direct drop-in for Locomotive Scroll** — API differs; shared concept of RAF-driven scroll, but Lenis v2+ rewrote the public API. Verify scroll event listeners when migrating.
- **Accessibility:** smooth scroll is scroll-hijacking. Gate behind `prefers-reduced-motion` (`new Lenis()` only when `no-preference`) and test keyboard-only navigation (Tab, Space/PageDown, anchor focus).

## Anime.js (v4)
MIT, lightweight, framework-agnostic. v4 (2024) — full TypeScript rewrite, ESM-first, redesigned animation engine. 66k★ GitHub. Handles CSS, SVG, JS object props, timelines, staggering, spring physics. **Best for mid-complexity projects without a React/Vue dependency** where GSAP is overkill and CSS is too limited. Bundle: ~17 KB gzip for the full package; import only the modules you use (timeline, stagger, etc.) for smaller builds.

## Lottie / lottie-web
Renders After Effects animations exported as JSON via Bodymovin (Airbnb-originated). Great for **vector motion graphics** (loaders, onboarding, empty states); a liability if unoptimized.
```js
import { DotLottie } from "@lottiefiles/dotlottie-web";       // prefer dotLottie
new DotLottie({ canvas, src: "/anim.lottie", autoplay: false, loop: true });
```
- **Use `.lottie` (dotLottie):** up to **80% smaller** (e.g. 1.3 MB JSON → 58 KB). Optimize JSON in LottieFiles Optimizer (20–90% reduction) before shipping.
- **Renderer:** SVG (default) for icons/illustrations; switch to **canvas above ~1,500 elements**. One naive Lottie can add 1,400+ DOM nodes (SVG renderer) — removing/canvas-rendering can cut a page from 2,000+ to <600 nodes.
- **Loading:** lazy-load below the fold with `IntersectionObserver`; show a static first-frame image if the asset is >500 KB or >0.5 s to load.
- **AE export hygiene:** avoid masks (use alpha-matte workarounds or video), avoid embedded JPEG/PNG (embed SVG or load separately), flatten layers sharing color+animation. Firefox can hit ~109% CPU on a 12 s animation (Chrome/Safari stay <60%) — test cross-browser.
- **When to use WebM video instead:** for full-screen decorative motion graphics with no interactivity (no play/pause scrub), transparent WebM (VP9 + alpha) is often smaller than dotLottie at high element counts and uses the GPU video decoder instead of CPU-driven SVG. Measure both before choosing.

## AutoAnimate
~2.3 KB gzip, zero-config. Animates DOM **mutations** (add/remove/reorder) via MutationObserver + FLIP + RAF. Handles **10,000+** simultaneous elements; **respects `prefers-reduced-motion` automatically**.
```jsx
import { useAutoAnimate } from "@formkit/auto-animate/react";
const [parent] = useAutoAnimate();
return <ul ref={parent}>{items.map(i => <li key={i.id}>{i.label}</li>)}</ul>;
```
- **Not for** gesture/physics/scroll-linked animation — only list/layout mutations.

## Concrete specs & numbers
**Bundle sizes (gzip):** Motion mini `animate()` 2.6 KB · Motion full React `motion` component ~18 KB · Anime.js v4 ~17 KB · GSAP core 23.5 KB · Framer Motion legacy (non-modular) ~32 KB · Lenis ~3 KB · AutoAnimate ~2.3 KB · WAAPI / CSS 0 KB.

**Duration targets:** desktop/web 150–200 ms · mobile UI 200–300 ms · page transitions 500–700 ms. Enter ~300 ms / exit ~200 ms. (Tablet UI follows mobile range; wearable web is not a practical target.)

**Perception thresholds:** ≤100 ms = perceived instant (NN/g) · ~100 ms = perceptual processor cycle median (Model Human Processor, Card/Moran/Newell 1983; range 50–200 ms) · 1 s = upper limit of a connected feedback loop.

**Lottie file-size targets:** icons <30 KB · UI animations <100 KB · illustrations <300 KB · full-screen <500 KB. dotLottie up to 80% smaller than equivalent JSON Lottie. For looping motion graphics, modern WebM video is often smaller and more CPU-efficient than Lottie SVG at high element counts; benchmark before choosing.

**Lottie node budgets (SVG renderer):** icons 20–50 nodes · illustrations 100–500 · switch to canvas at 1,500+.

**Browser support:** View Transitions same-doc — Baseline Oct 2025 (Chrome/Edge 111+, Safari 18+, Firefox 144+), >85% global. Cross-doc — Chrome/Edge 126+, Safari 18.2+, no Firefox (mid-2026). CSS scroll-driven — Chrome/Edge 115+, Firefox 115+ (scroll+view+ranges), Safari 16.0+ (basic), 17.2+ (view timelines). `@starting-style` / `transition-behavior: allow-discrete` — Baseline 2024 (Chrome 117+, Firefox 129+, Safari 17.5+).

## Tools & libraries
| Need | Use | Don't use |
|---|---|---|
| Hover/focus/toggle, simple reveal | CSS transitions/keyframes | A JS library |
| Enter transition from `display:none` | `@starting-style` + `transition-behavior: allow-discrete` | JS show/hide toggle |
| Scroll progress / reveal / parallax | CSS scroll-driven animation | JS scroll listeners (main-thread jank) |
| SPA route/state transition | View Transitions API | Hand-rolled crossfade |
| MPA page transition | CSS `@view-transition` + fallback | JS page-transition libs (extra weight) |
| List add/remove/reorder | AutoAnimate | GSAP/Motion (overkill) |
| React exit anims, shared element, layout | Motion (`AnimatePresence`/`layoutId`/`layout`) | GSAP inside React render tree |
| Timeline, scrub, pin, SVG morph, split text | GSAP + plugins | Motion (weaker timeline control) |
| Non-React mid-complexity | Anime.js v4 | GSAP if you don't need its plugins |
| After Effects motion graphic | Lottie (dotLottie) | Re-building it in code |
| Smooth scroll feel | Lenis (+ ScrollTrigger proxy) | Locomotive pre-v2 (breaks sticky) |
| Physics/gesture-led React UI | React Spring (or Motion springs) | — |
| Tailwind, zero-JS utility motion | TailwindCSS Motion | — |
| Render animation → video | Remotion | An in-browser lib (wrong tool) |

## Do / Don't
**Do**
- Escalate CSS (incl. `@starting-style` for enter transitions) → WAAPI → library; justify every dependency by complexity.
- Animate `transform`/`opacity` for guaranteed compositor; use `filter`/`clip-path` with awareness of rasterization fallback.
- Use ease-out in, ease-in out; enter longer than exit.
- Wire `prefers-reduced-motion` into every motion path (CSS `@media`, `useReducedMotion`, `gsap.matchMedia`, AutoAnimate is automatic).
- Clean up GSAP/ScrollTrigger via `gsap.context().revert()` in `useLayoutEffect`.
- Drive Lenis from `gsap.ticker` so ScrollTrigger stays in sync.
- Ship Lottie as dotLottie, lazy-load below fold, switch to canvas >1,500 elements.
- Provide stable `key`s to `AnimatePresence` children.

**Don't**
- Animate `width`/`height`/`top`/`left`/`margin`/`padding` with a JS library.
- Add Motion to non-interactive server content (~18–32 KB for 0 KB CSS work, plus forces `'use client'`).
- Use `var(--x)` for animatable values without `@property … inherits: false`.
- Leave `will-change` on permanently.
- Hijack scroll (Lenis/ScrollSmoother) without a reduced-motion gate + keyboard test.
- Use GSAP in a product that competes with Webflow.
- Ship a naive multi-MB Lottie JSON.

## Common pitfalls & how to avoid them
- **GSAP license trap:** no-code/web-builder products can't legally use GSAP post-Aug 2023 acquisition; Webflow may revoke at discretion. → Use Motion (MIT) for those products.
- **Motion bloat on static content:** ~18–32 KB (depending on package/import) to do what `@keyframes` do at 0 KB; forces `'use client'`. → CSS for static, Motion only for interactive.
- **Wrong GSAP import:** `import gsap from 'gsap'` plus unregistered plugins. → Named imports + `gsap.registerPlugin(...)` per plugin.
- **ScrollTrigger stacking in SPA:** instances survive route changes, animations pile up. → Kill via `gsap.context().revert()` in `useLayoutEffect` cleanup.
- **Lottie JSON bloat:** naive 35 s loader ≈ 1.2 MB / 4–5 s load. → Optimize/convert to dotLottie; canvas renderer; lazy-load.
- **Custom props fall off compositor:** `var(--color)` in a transform still paints. → `@property` typed vars with `inherits: false`.
- **Smooth scroll breaks `sticky`:** Locomotive pre-v2. → Lenis preserves sticky; verify before choosing.
- **View Transitions MPA in Firefox:** unsupported mid-2026. → Test instant-navigation fallback.
- **`AnimatePresence` no exit anim:** missing/unstable `key`. → Stable unique key per child.
- **SplitText kills semantics:** per-character spans break screen readers. → `aria-label` on parent, `aria-hidden="true"` on every child span.
- **DOM thrashing:** read-write-read-write in a loop. → Batch all reads, then all writes (or let FLIP libs handle it).

## What real users complain about
- **"Scroll feels hijacked / makes me nauseous."** Smooth-scroll + parallax without a reduced-motion gate; momentum scroll fights native scrollbar and trackpad expectations.
- **"The page janks/stutters when I scroll."** JS scroll listeners on the main thread, or animating layout properties — drops below 60 fps under load.
- **"Loader takes forever."** Multi-MB unoptimized Lottie JSON blocking first interaction.
- **"My keyboard/anchor links stopped working."** Smooth-scroll libs that didn't preserve anchor focus or `position: sticky`.
- **"Exit animation just snaps."** Missing `key` on `AnimatePresence` children, or DOM removed before the transition runs.
- **"Animations stack/glitch after navigating."** Uncleaned GSAP/ScrollTrigger instances in SPA routes.
- **"Fonts/icons flash then jump."** Reveal animations without `fill: both`/`forwards`, or layout shift from animating sized properties.

## Senior-level checklist
- [ ] Picked the lowest-cost tier that solves it (CSS → WAAPI → library), and can justify any dependency.
- [ ] Every animated property is `transform`/`opacity` (no layout props); `filter`/`clip-path` profiled on mobile if used.
- [ ] `prefers-reduced-motion` handled on every motion path; verified by toggling the OS setting.
- [ ] Durations within targets (web 150–200 ms, mobile 200–300 ms, page transitions 500–700 ms); ease-out in, ease-in out.
- [ ] Scroll effects use CSS scroll-driven where possible; JS scroll work is throttled/compositor-aware.
- [ ] Smooth scroll (if any) preserves sticky + anchors; passes keyboard-only navigation.
- [ ] GSAP/ScrollTrigger cleaned up on unmount; Lenis driven from `gsap.ticker`.
- [ ] Lottie shipped as dotLottie, under size budget, lazy-loaded, renderer chosen by node count; cross-browser CPU checked.
- [ ] Motion confined to interactive client components; `AnimatePresence` children keyed.
- [ ] No `var()` in animatable values without `@property`; `will-change` added/removed surgically.
- [ ] GSAP license compatible with the product (not a Webflow competitor).
- [ ] `@starting-style` used for enter transitions where supported; `@supports` guard for older targets.
- [ ] View Transitions fallback (instant nav) verified, especially Firefox MPA.
- [ ] Measured: 60 fps in DevTools Performance, no long tasks, no layout-thrash warnings.

## Sources
- MDN — View Transitions API: https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API
- MDN — CSS scroll-driven animations: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations
- MDN — Web Animations API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API
- MDN — `@property`: https://developer.mozilla.org/en-US/docs/Web/CSS/@property
- MDN — `prefers-reduced-motion`: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- GSAP docs: https://gsap.com/docs/v3/
- Motion docs: https://motion.dev/docs
- Lenis: https://github.com/darkroom-engineering/lenis
- Anime.js: https://animejs.com/
- AutoAnimate: https://auto-animate.formkit.com/
- LottieFiles / dotLottie: https://lottiefiles.com/ · https://dotlottie.io/
- web.dev — animations & performance: https://web.dev/articles/animations-guide
- Material Design — motion: https://m3.material.io/styles/motion/overview

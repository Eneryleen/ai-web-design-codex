# Native Interactive UI Primitives: Popover, Anchor Positioning, Dialog & inert

> How to build menus, tooltips, dropdowns, modals, command palettes, and toasts on the 2026 platform with zero or minimal JS — using the Popover API, CSS Anchor Positioning, native `<dialog>`, `details`/`summary`, invoker commands (`command`/`commandfor`), and `inert`. The senior-level outcome: top-layer overlays that never clip, never z-index-fight, light-dismiss correctly, animate in and out, and degrade gracefully — while knowing exactly when these replace Floating UI / Radix and when they still do not.

**Apply when:** Building any overlay (menu, dropdown, tooltip, modal, toast, picker, command palette), deciding whether to pull in a headless library, or fixing clipping/stacking/focus-trap bugs. · **Related:** [Component Libraries & Headless UI](./component-libraries-headless.md) · [CSS, Styling & Tailwind](./css-styling-and-tailwind.md) · [Interaction Design & UI States](../01-ux-fundamentals/interaction-design-and-states.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md)

## TL;DR — rules at a glance
- **Two separate problems.** Anchor Positioning answers *where* a floating element goes; the Popover API answers *whether/when* it shows (toggle, light-dismiss, top-layer, keyboard, basic a11y). A complete menu/tooltip needs **both**. This split is the single most important mental model here.
- **Popovers are always non-modal.** They do **not** make the page `inert` — background stays clickable and tabbable. The only true modal primitive is `dialog.showModal()`. Reserve modals for sign-in, confirmations, anything that must block the page.
- **The top layer is the killer feature.** A shown popover or modal dialog escapes every `z-index`, every `overflow: hidden` ancestor, and every transformed container. The perennial clipping/stacking bugs of hand-rolled overlays vanish with zero JS.
- **The platform gives guardrails, not finished components.** Popover wires focus-return, Esc, light-dismiss, `aria-expanded` on the trigger, and a generic group — but assigns **no** `menu`/`listbox`/`tooltip` role. Per Scott O'Hara: popover is *"an ingredient, not the main course."*
- **Pick popover state by intent:** `auto` (default) = menus/share sheets/pickers/command palettes (light-dismiss + Esc + one-at-a-time); `manual` = toasts and persistent teaching UI (no light-dismiss, can coexist, needs explicit close); `hint` = tooltips (light-dismissible, hover/focus driven).
- **`<dialog open>` ≠ modal.** The `open` attribute shows the dialog but does **not** trap focus and does **not** inert the background. Modal behavior exists only via `showModal()` (or the `show-modal` invoker command).
- **`type="button"` on every** `popovertarget` / `commandfor` / `command` button. Inside a `<form>`, buttons default to `type="submit"` — a silent form-submission footgun.
- **Toasts are not announced.** The Popover API never tells a screen reader a popover appeared. Keep a persistent `role="status"` / `aria-live="polite"` region on page load and inject text into it.
- **Animate the top layer with both tools:** `@starting-style` defines the enter FROM-state; `transition-behavior: allow-discrete` keeps `display`/`overlay` from snapping so the EXIT is visible. You need both, and `@starting-style` only works with **transitions**, not `@keyframes`.
- **Collision-aware flyouts need 3-4 fallbacks:** base + block flip + inline shifts for edge triggers. `flip-block`/`flip-inline`/`flip-start` cover simple flips; named `@position-try` blocks cover custom shifts.
- **Exclusive accordions are free:** give every `<details>` the same `name`. No JS. But ask whether forcing one-open-at-a-time helps the user first.
- **Never style an opaque `::backdrop` on `popover="manual"`** — UA forces `pointer-events: none` on popover backdrops, so the page *looks* blocked while staying clickable. Need a scrim? Use `dialog.showModal()`.
- **`inert` is the manual version of modal inerting.** `showModal()` inert-s the background automatically; for a blocking *non-modal* popover, an off-canvas drawer, or a custom overlay you must toggle `inert` on the rest of the page yourself — it removes a subtree from tab order **and** the a11y tree.
- **`closedby` is not Baseline.** Light-dismiss for `<dialog>` (`closedby="any"`) ships in Chrome/Edge/Firefox but **not Safari** as of mid-2026 — progressive-enhance, always keep an explicit close control.
- **Progressive enhancement is the framing for all of it.** Base behavior works everywhere; wrap advanced styling in `@supports`; animations and flips degrade to instant, non-broken states.

## The mental model: positioning vs. visibility

These are orthogonal. Conflating them is the #1 conceptual error.

| Concern | Question | Owned by | Without it you get |
|---|---|---|---|
| **Visibility / behavior** | Should it show? Toggle, dismiss, focus, Esc, top-layer? | **Popover API** (`popover`, `popovertarget`) | a `position:absolute` div you must show/hide and dismiss by hand |
| **Positioning** | Where does it go relative to the trigger? Collision flip? | **CSS Anchor Positioning** (`anchor-name`, `anchor()`, `position-area`, `position-try`) | a centered/off-screen box, or JS scroll/resize listeners |

A real tooltip/menu = **Popover API for the *whether* + Anchor Positioning for the *where*.** You can use either alone (a popover centered with `margin: auto`, or anchor positioning on a non-popover element), but the combination is what replaces Floating UI for the common cases.

```html
<button popovertarget="menu">Actions ▾</button>
<menu popover id="menu">…</menu>
```
```css
button { anchor-name: --menu-btn; }
#menu {
  position-anchor: --menu-btn;     /* tie to the trigger        */
  position-area: bottom span-right; /* where: below, align right */
  position-try-fallbacks: flip-block flip-inline; /* collision   */
  margin: 0; inset: auto;          /* reset UA inset (see below) */
}
```

## The Popover API — visibility, dismissal, top layer

### Declarative wiring
- `popover` on the element + `popovertarget="id"` on a `<button>`. The button toggles it. **No JS.**
- `popovertargetaction="show|hide|toggle"` chooses the action (default `toggle`). Use `hide` for an in-popover close button.
- The trigger automatically gets `aria-expanded` reflecting state and an implicit relationship to the popover — you do **not** add `aria-controls`/`aria-expanded` by hand.

### The three states — pick by *intent*

| Value | Use for | Light-dismiss (click outside) | Esc | Multiple open at once | Needs explicit close |
|---|---|---|---|---|---|
| `auto` (default) | menus, dropdowns, share sheets, pickers, **command palettes** | ✅ | ✅ | ❌ (opening one closes others in the same scope) | no |
| `manual` | **toasts**, persistent teaching/onboarding UI | ❌ | ❌ | ✅ | **yes** — add a `hide` button |
| `hint` | **tooltips** | ✅ | ✅ | closes other `hint`s but **not** `auto` popovers | no |

`hint` is the newest; it lets a tooltip appear *without* closing an open menu — critical for hover help inside an open dropdown.

### What you get for free vs. what you must add
**Free:** top-layer rendering, light-dismiss (auto/hint), Esc to close (auto/hint), focus return to the trigger on close, `aria-expanded` on the trigger, one-at-a-time stacking (auto).
**You must add:** the component **role** (`role="menu"`, `role="listbox"`, `role="tooltip"`…), arrow-key navigation between items, live-region announcements for toasts, and `autofocus` if focus should move into the popover (it does **not** auto-move focus unless a descendant has `autofocus`).

### `:popover-open` and `::backdrop`
- Style the shown state with `[popover]:popover-open { … }`.
- Popovers get a `::backdrop` pseudo-element, but the **UA forces `pointer-events: none !important`** on it (this is *how* light-dismiss works). So a dark `::backdrop` on a `manual` popover obscures the page while leaving it clickable — a trap. A visible scrim that actually blocks belongs on a **modal dialog**.

## CSS Anchor Positioning — the *where*

Replaces JS positioning engines for the common menu/tooltip/dropdown cases.

```css
.trigger  { anchor-name: --t; }            /* declare the anchor   */
.floater  {
  position: fixed;
  position-anchor: --t;                     /* bind to that anchor  */
  position-area: bottom center;             /* high-level placement */
  /* or low-level: top: anchor(--t bottom); left: anchor(--t left); */
}
```

- **`anchor-name`** names an element as an anchor; **`position-anchor`** binds a positioned element to it; **`anchor(<side>)`** reads the anchor's edge for use in `top`/`left`/etc.; **`position-area`** is the ergonomic grid syntax (`bottom center`, `top span-right`, `inline-end`).
- The browser keeps the floater in sync on scroll/resize **with no JS listeners** — this is the whole point.
- **Reset UA insets.** Popovers and dialogs ship UA `inset`/`margin` styles that fight `anchor()` placement and cause mispositioning. Add `inset: auto; margin: 0;` on anchored popovers/dialogs.

### Collision-aware fallbacks (`position-try`)
Production flyouts need **3-4** fallbacks, not one:

```css
.menu {
  position-anchor: --t;
  position-area: bottom span-right;
  position-try-fallbacks:
    flip-block            /* 1. trigger near bottom edge → open upward   */
    flip-inline           /* 2. trigger near right edge  → align left     */
    flip-block flip-inline /* 3. corner case → flip both                  */
    --shift-up;           /* 4. named custom shift for the worst case      */
}
@position-try --shift-up {
  position-area: top span-left;
  margin-bottom: 8px;
}
```
- `flip-block` / `flip-inline` / `flip-start` cover the common mirror flips. Named `@position-try {}` blocks let you change `position-area`, margins, and sizing for bespoke fallbacks.
- **Safari nuance:** core `anchor()`/`position-anchor` shipped in **Safari 18.2**, but `@position-try` auto-flip needs **Safari 18.4+**. On 18.2-18.3 it *places* correctly but does **not** flip — so always provide a sane base position that is acceptable un-flipped.

### Anchored container queries — the flipped-arrow fix (Chrome 143)
Historically you couldn't style the floater based on *which* fallback fired (so tooltip arrows pointed the wrong way after a flip). Now you can, in pure CSS:

```css
.tooltip { container-type: anchored; position-try-fallbacks: flip-block; }
.tooltip::before { content: '▲'; bottom: 100%; top: auto; }   /* default: below trigger, arrow up */
@container anchored(fallback: flip-block) {
  .tooltip::before { content: '▼'; top: 100%; bottom: auto; }  /* flipped: above trigger, arrow down */
}
```
- **Descendants-only, like all container queries:** an `@container anchored(...)` block can only style **descendants** of the `container-type: anchored` element, never that element itself. The arrow above works because `::before` is a descendant pseudo-element; to restyle the floater's own box (e.g. flip its `transform-origin`) you must move those styles onto an inner wrapper.
- **Implicit anchor with popovers:** a popover targeted by `popovertarget` already has an implicit anchor relationship to its trigger, so you can skip `anchor-name`/`position-anchor` and reference `anchor()` directly.
- **Support:** anchored container queries are **Chromium-only (Chrome 143+)** as of mid-2026; treat the un-queried (default-direction) arrow as the graceful fallback everywhere else.

## Native `<dialog>` — the only real modal

```html
<button type="button" commandfor="signin" command="show-modal">Sign in</button>
<dialog id="signin">
  <form method="dialog">
    <h2>Sign in</h2>
    <input name="email" autofocus>
    <button type="button" commandfor="signin" command="close">Cancel</button>
    <button>Continue</button> <!-- method=dialog submit closes & returns value -->
  </form>
</dialog>
```

- **`showModal()` (or `command="show-modal"`) = modal:** focus is trapped, background is inert, Esc closes, a real (pointer-blocking) `::backdrop` renders, it's in the top layer. **`open` attribute / `show()` = non-modal:** visible, **no** focus trap, **no** inert background. Mixing these up is the most common dialog bug.
- **Focus:** prefer `autofocus` on the first meaningful field. Do **not** add `tabindex` to the `<dialog>` itself (spec warns against it), and do **not** hand-roll a focus trap on a modal dialog — the browser already blocks tabbing to background content. Over-trapping can prevent reaching browser chrome (URL bar, tabs), which you must never block.
- **`<form method="dialog">`** closes the dialog on submit and sets `dialog.returnValue` — no JS needed for the common confirm flow.
- **`closedby` attribute** controls light-dismiss for dialogs: `closedby="any"` (light-dismiss like an auto popover), `closedby="closerequest"` (Esc/platform close action only — the implicit default for `showModal()`), `closedby="none"` (no built-in dismiss; you must provide one). **Not Baseline as of mid-2026 — Chrome/Edge/Firefox ship it, Safari does not** (WebKit bug 284592; expected via Interop 2026). Treat `closedby="any"` as progressive enhancement only, and always wire a visible close button + `cancel`/`request-close` so Safari users aren't trapped.
- **`::backdrop`** on a modal dialog is the place for a real scrim: `#d::backdrop { background: rgb(0 0 0 / .5); backdrop-filter: blur(3px); }`.

### Popover vs. dialog — decision table
| Need | Use |
|---|---|
| Menu, dropdown, share sheet, picker, command palette | **`popover` (auto)** |
| Tooltip | **`popover="hint"`** + anchor positioning |
| Toast / snackbar | **`popover="manual"`** + own `aria-live` region |
| Must block the page (sign-in, destructive confirm, paywall) | **`<dialog>` + `showModal()`** |
| Background must stay interactive | popover (any) |

## `inert` — manual background-blocking

`showModal()` inert-s the background **for you**. `inert` is the manual primitive for every case where you need that blocking without a modal dialog — and the platform gives you nothing automatic there.

```html
<main inert>…page content…</main>      <!-- becomes non-interactive + invisible to AT -->
<aside class="drawer">…off-canvas nav…</aside>
```

- **What it does:** an `inert` element and its subtree are removed from the **tab order** and the **accessibility tree**, can't be clicked, focused, text-selected, or found via in-page search. It is `aria-hidden` + `tabindex=-1` for the whole subtree, enforced by the UA.
- **When you need it (the platform won't do it for you):**
  - A **non-modal** `<dialog show()>` or `popover` that should nonetheless block the page — popovers are *always* non-modal and never inert the background, so toggle `inert` on the rest yourself.
  - **Off-canvas drawers / mobile nav** while open behind a scrim, and the **closed** drawer's offscreen contents (keep them out of the tab order until shown).
  - Custom overlays not built on `<dialog>`.
- **Gotchas:** there is **no default visual styling** — pair `inert` with your own scrim so sighted users see the block. Inert-ing an ancestor of the focused element silently drops focus to `<body>`; move focus into the active region first. Never inert an ancestor of the very overlay you're showing.
- **Don't double up:** a modal `<dialog>` already inert-s everything else — adding manual `inert` on top is redundant and can fight focus return. Use `inert` *instead of* `showModal()`, not alongside it.
- **Baseline since April 2023** (Firefox 112 was last); reflected by the `HTMLElement.inert` IDL property. Note Baseline tracks engine support, not AT exposure — verify with a real screen reader.

## Invoker Commands — `command` / `commandfor`

Declarative, JS-free wiring for dialogs and popovers (Baseline late 2025 / early 2026).

```html
<button type="button" commandfor="d" command="show-modal">Open</button>
<button type="button" commandfor="t" command="toggle-popover">Menu</button>
```
- **Built-in commands:** `show-modal`, `close`, `request-close` (dialog); `show-popover`, `hide-popover`, `toggle-popover` (popover). `request-close` fires a cancelable `cancel` event so you can confirm "discard changes?" before closing.
- **Custom commands** use a `--` prefix (reserved namespace) and dispatch a `CommandEvent` you handle in JS — a clean declarative hook for app-specific actions.
- **Always `type="button"`** — these often sit inside forms.

## `details` / `summary` — disclosure & accordions

```html
<details name="faq"><summary>What is anchor positioning?</summary><p>…</p></details>
<details name="faq"><summary>What is the top layer?</summary><p>…</p></details>
```
- **Same `name` on multiple `<details>` = exclusive accordion** (browser auto-closes the previously open one). Baseline; Firefox added `name` in v130. **Heed the Chrome team's caution:** one-open-at-a-time can frustrate users who want to compare panels — choose deliberately.
- `<details>`/`<summary>` is keyboard- and screen-reader-accessible out of the box, and its `display` can now be `flex`/`grid` (Interop 2025 work; Safari 17 normalized disclosure markers).
- **Animate open/close** via `::details-content` (Baseline newly available since Sept 2025):

```css
::details-content {
  height: 0; overflow: clip;
  transition: height .4s ease, content-visibility .4s allow-discrete;
}
@supports (interpolate-size: allow-keywords) {
  :root { interpolate-size: allow-keywords; }
  [open]::details-content { height: auto; }
}
@supports not (interpolate-size: allow-keywords) {
  [open]::details-content { height: 150px; overflow-y: auto; } /* fallback: fixed height */
}
```
**Caveat:** animating to `height: auto` requires `interpolate-size: allow-keywords` (or `calc-size(auto, size)`), which is **Chromium-only** as of mid-2026 — Firefox/Safari degrade to an instant open/close, which is fine. Note the syntax change: single-arg `calc-size(auto)` was removed; current form is `calc-size(auto, size)`.

## Enter/exit animation on the top layer

Animating elements that toggle `display` (popovers, dialogs) and live in the top layer needs three pieces that work together. **`@starting-style` and `allow-discrete` are complementary, not interchangeable.**

```css
[popover] {
  opacity: 0; transform: scaleY(.9); transform-origin: top;
  transition:
    opacity .25s, transform .25s,
    display .25s allow-discrete,   /* keep mounted during exit  */
    overlay  .25s allow-discrete;  /* keep in top layer on exit */
}
[popover]:popover-open { opacity: 1; transform: scaleY(1); }
@starting-style {
  [popover]:popover-open { opacity: 0; transform: scaleY(.9); } /* enter FROM-state */
}
```
- **`@starting-style`** = the *from* state for the entering element (it has no prior style to transition from). Works only with **transitions**, never `@keyframes`.
- **`transition-behavior: allow-discrete`** lets discrete properties (`display`, `overlay`, `content-visibility`) transition instead of snapping, so the **exit** is actually visible.
- **Include `overlay`** in the transition or **stacked popovers jump behind each other** on exit (the element leaves the top layer instantly). Safari historically lacked the `overlay` property — a known source of exit-animation glitches.
- Same pattern for `<dialog>` using `[open]` and `[open]::backdrop` selectors (animate the backdrop's opacity too).

## Concrete specs & numbers
- **Popover API** — Baseline **Newly available Jan 2025**, **Widely available April 2025** (Chrome 116, Edge 116, Firefox 125, Safari 17 / iOS 18.3 — the iOS light-dismiss fix in 18.3 is what gated the April promotion).
- **CSS Anchor Positioning** — Baseline 2026 / "Newly available": Chrome 125+, Edge 125+, Firefox 147+, Safari 26. Core `anchor()`/`position-anchor` since **Safari 18.2**; **`@position-try` flip needs Safari 18.4+** (18.2-18.3 places but does not auto-flip).
- **Anchored container queries** (Anchor Position Level 2) — **Chrome 143**; enables `container-type: anchored` + `@container anchored(fallback: …)` to fix flipped arrows in pure CSS.
- **Entry/exit animation** — animate `display`/`content-visibility` (Chrome 116); `transition-behavior: allow-discrete`, `@starting-style`, `overlay` property (Chrome 117). Cross-engine `transition-behavior`/`allow-discrete`: Chrome 121+, Safari 17.5+, Firefox 129+. `allow-discrete` Baseline since **Aug 2024**; `@starting-style` Baseline as of **early 2026**.
- **Invoker Commands** (`command`/`commandfor`) — Baseline across Chrome/Edge, Firefox, Safari **late 2025 / early 2026**. Custom commands use the `--` prefix.
- **`<dialog closedby>`** — **NOT Baseline (mid-2026)**: Chrome/Edge/Firefox only; Safari/iOS missing (WebKit bug 284592, targeted by Interop 2026). Progressive-enhance, never depend on it for dismissal.
- **`inert`** — Baseline **Newly available April 2023** (Firefox 112 last); `HTMLElement.inert` IDL property. Baseline reflects engine support, not assistive-tech exposure.
- **`<details name>` exclusive accordions** — Baseline (Firefox added `name` v130). **`::details-content`** — Baseline newly available since **Sept 2025**.
- **`height: auto` animation** — needs `interpolate-size: allow-keywords` **or** `calc-size(auto, size)`; `interpolate-size` is **Chromium-only** as of mid-2026 (others degrade to instant). Single-arg `calc-size(auto)` removed.
- **`details`/`summary`** — Interop 2025 focus area; most gaps closed (Safari 17 markers, `display: flex/grid` allowed).
- **Scale of the problem being solved natively:** Floating UI core sees ~**23M weekly npm downloads**. Dropping a Floating UI tooltip + Radix Dialog + GSAP ScrollTrigger saves roughly **44 kB** (not counting GSAP core).

## Tools & libraries
Native primitives reduce — but don't eliminate — the need for these. Reach for a library when the native a11y/positioning gaps below bite.

- **Floating UI** — low-level positioning/collision engine (successor to Popper.js, same authors). Pure geometry, unstyled, framework-agnostic; powers Radix, Mantine, and others internally. **Still the recommendation for `select`/combobox overlays** and any complex multi-axis collision logic.
- **Radix Primitives / Radix UI** — headless, WAI-ARIA-compliant React components (Popover, Tooltip, Dialog, Select) on Floating UI; bring your own styles. **Still right for React apps where complete combobox/select a11y is non-negotiable.** See [Component Libraries & Headless UI](./component-libraries-headless.md).
- **Headless UI (Tailwind Labs)** — headless accessible React/Vue primitives; exists largely because focus traps + ARIA state are tedious to hand-roll.
- **React Aria (Adobe)** — hooks-based accessible primitives incl. `Popover`; deepest a11y/i18n.
- **OddBird `css-anchor-positioning` polyfill** — broader anchor-positioning support, but **does not implement the full spec** — verify against your fallbacks.
- **A11y Dialog, Micromodal** — lightweight, dependency-light accessible modal wrappers that leverage native `<dialog>`.
- **templUI (Go + templ + Tailwind)** — a 2026 library that rewrote its modal layer onto native `<dialog>` — a signal of the direction of travel.
- **Tippy.js** — legacy tooltip library; commonly the thing being *replaced* in 2026 migrations to native + Radix/Floating UI.

### When native replaces the library — and when it doesn't
| Pattern | Native is enough | Keep a library |
|---|---|---|
| Tooltip (hover/focus help) | ✅ `hint` popover + anchor + container query for the arrow | rich interactive tooltips, hundreds with shared logic |
| Menu / dropdown (clickable items) | ✅ `auto` popover + `role="menu"` + your arrow-key JS | full WAI-ARIA menu with type-ahead → Radix/React Aria |
| Modal dialog | ✅ `<dialog>` + `showModal()` | rarely needed; libs add convenience only |
| Toast | ✅ `manual` popover + own `aria-live` | toast queue/stacking manager → a small lib |
| **Combobox / autocomplete / `select`** | ❌ no native a11y for filtered listbox + active-descendant | **Yes — Radix/React Aria/Floating UI** |
| **Complex collision** (multi-fallback with size constraints, nested flyouts, sub-menus) | ⚠️ partial via `position-try` | **Floating UI** for fine control |

## Do / Don't
**Do**
- Use **both** Popover API (visibility) and Anchor Positioning (placement) for menus/tooltips.
- Use `<dialog>.showModal()` (or `show-modal`) whenever you need focus trapping or an inert background.
- Reach for `inert` (plus your own scrim) when you need to block the page *without* a modal dialog — blocking non-modal popovers, open drawers, the closed-drawer offscreen subtree, custom overlays.
- Set `type="button"` on every `popovertarget`/`command`/`commandfor` button.
- Add `inset: auto; margin: 0;` to anchored popovers/dialogs to stop UA styles fighting `anchor()`.
- Keep a persistent `role="status"`/`aria-live="polite"` region for toasts from page load.
- Provide 3-4 `position-try-fallbacks`, with a sane un-flipped base position (Safari 18.2-18.3).
- Wrap advanced styling/animation in `@supports`; verify the base behavior works without it.
- Add the component role yourself (`role="menu"`/`listbox`/`tooltip`) and `autofocus` if focus must enter the popover.
- Animate top-layer exits by including `display` and `overlay` with `allow-discrete`.

**Don't**
- Don't use `<dialog open>` expecting modal behavior — it traps nothing and inert-s nothing.
- Don't style a dark/opaque `::backdrop` on a `popover="manual"` (page looks blocked, content stays clickable).
- Don't assume a toast popover is announced — it never is without a live region.
- Don't add `tabindex` to the `<dialog>` element, and don't hand-roll a focus trap on a modal dialog.
- Don't rely on `@starting-style` with `@keyframes` — it only works with transitions.
- Don't ship a single-position flyout to production — edge triggers will overflow the viewport.
- Don't reach for Floating UI/Radix for a basic tooltip or modal in 2026 — measure the native path first.
- Don't inert an ancestor of the currently-focused element (focus silently falls to `<body>`), and don't stack manual `inert` on top of a modal `<dialog>` (it already inert-s everything else).
- Don't depend on `closedby="any"` for dismissal — it's absent in Safari as of mid-2026; ship a real close button.

## Common pitfalls & how to avoid them
- **`<dialog open>` instead of `showModal()`** → no focus trap, no inert background. Fix: use the method or `command="show-modal"`.
- **Missing `type="button"` in a form** → the popover/command button submits the form. Fix: always set `type="button"`.
- **Opaque `::backdrop` on a `manual` popover** → UA `pointer-events: none` means the page is visually blocked but clickable/tabbable behind it. Fix: use `dialog.showModal()` for a real scrim.
- **Silent toasts** → no screen-reader announcement. Fix: persistent `aria-live` region; inject text into it (use `assertive` only for urgent errors). If you add the region dynamically, wait ~2s before injecting so the a11y API registers it.
- **Misplacement after open/close on iOS Safari** → anchor-positioned + popover dropdown jumps on close (community-traced to WebKit recalculating layout before the anchor reference restabilizes + Safari's missing `overlay` property; see whatwg/html #11694). Mitigation: test on real WebKit, keep `overlay` in transitions, accept that some Safari builds glitch the exit.
- **Flyout flows off-screen** because only one position was set, or because Safari 18.2-18.3 didn't flip. Fix: multiple `position-try-fallbacks` **and** an acceptable un-flipped base.
- **Tooltip arrow points the wrong way after a flip** → fix in pure CSS with anchored container queries (Chrome 143); elsewhere fall back to a centered/no-arrow style.
- **Exit animation snaps instantly** → you forgot `allow-discrete` on `display`/`overlay`, or used `@keyframes` instead of a transition with `@starting-style`.
- **UA inset fights `anchor()`** → element lands in the wrong place on popovers/dialogs. Fix: `inset: auto; margin: 0;`.
- **Over-trapping focus** in a custom modal → users can't reach browser chrome. Fix: trust `showModal()`'s built-in trap; don't add your own.
- **Relying on `closedby="any"` for dismissal** → Safari users (mid-2026, no support) get a dialog with no click-outside close. Fix: treat it as enhancement; always provide an explicit close button + Esc/`request-close`.
- **Manual `inert` left stale** → page stays blocked after the overlay closes, or focus vanishes to `<body>` when you inert an ancestor of the focused element. Fix: toggle `inert` in lockstep with show/hide, and move focus into the overlay before inert-ing the rest.

## What real users complain about
- **iOS/Safari overlay glitches** are the most-reported real-world issue: anchored popover dropdowns that jump or mis-place on close (WebKit layout-recalc timing + missing `overlay`). Always validate on real iOS Safari, not just Chromium.
- **"The menu opened but my screen reader said nothing"** — toasts and custom popovers with no live region; the most common a11y regression when teams drop a library for native.
- **Accidental form submits** from `popovertarget`/`command` buttons without `type="button"` — surfaces as "clicking the dropdown reloaded the page."
- **Menus clipped inside scroll containers** in older hand-rolled code — exactly the bug the top layer eliminates; users notice the difference immediately.
- **Exclusive accordions that close the panel users were reading** — the `name` feature applied without thinking about whether one-at-a-time helps.
- **Animations that "pop in but vanish instantly"** — missing `allow-discrete`/`overlay`, visible to anyone watching closely.

## Senior-level checklist
- [ ] Chose popover vs. dialog by **intent** (block the page → dialog `showModal()`; else popover).
- [ ] Correct popover **state**: `auto` (menus), `manual` (toasts, + close button), `hint` (tooltips).
- [ ] **Both** Popover API and Anchor Positioning wired for menus/tooltips; `inset: auto; margin: 0;` reset applied.
- [ ] **3-4 `position-try-fallbacks`** + an acceptable un-flipped base (Safari 18.2-18.3 safety).
- [ ] `type="button"` on every `popovertarget`/`command`/`commandfor` button.
- [ ] Component **role** added by hand (`menu`/`listbox`/`tooltip`); arrow-key nav implemented where expected.
- [ ] `autofocus` placed where initial focus should land in modals/focus-capturing popovers.
- [ ] Modals use `showModal()` (not `open`); no manual focus trap; no `tabindex` on `<dialog>`; visible close control; `closedby` treated as enhancement (absent in Safari, mid-2026).
- [ ] Non-modal blocking (drawers, blocking popovers, custom overlays) handled with `inert` + own scrim; no `inert` on a focused-element ancestor; no `inert` stacked on a modal `<dialog>`.
- [ ] Persistent `aria-live` region present for toasts from page load; `polite` default, `assertive` only for urgent errors.
- [ ] `::backdrop` scrim only on modal dialogs / auto popovers, never on `manual` popovers.
- [ ] Enter/exit animation uses `@starting-style` + `transition` + `allow-discrete` on `display` **and** `overlay`.
- [ ] Advanced styling/animation behind `@supports`; base behavior verified in an engine without it.
- [ ] Exclusive `<details name>` chosen deliberately; disclosure animation degrades to instant outside Chromium.
- [ ] Tested on **real iOS Safari** (overlay/anchor glitches), not just Chromium.
- [ ] Library kept only where native falls short: **combobox/`select`** and **complex collision** (Radix / React Aria / Floating UI).

## Sources
- MDN — Popover API: https://developer.mozilla.org/en-US/docs/Web/API/Popover_API
- MDN — Using the Popover API: https://developer.mozilla.org/en-US/docs/Web/API/Popover_API/Using
- MDN — CSS anchor positioning: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning
- MDN — Using CSS anchor positioning: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning/Using
- MDN — `<dialog>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog
- MDN — `<details>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details
- MDN — `@starting-style`: https://developer.mozilla.org/en-US/docs/Web/CSS/@starting-style
- MDN — `transition-behavior`: https://developer.mozilla.org/en-US/docs/Web/CSS/transition-behavior
- MDN — `overlay`: https://developer.mozilla.org/en-US/docs/Web/CSS/overlay
- MDN — `inert`: https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/inert
- MDN — `::details-content`: https://developer.mozilla.org/en-US/docs/Web/CSS/::details-content
- web.dev — Anchor positioning: https://developer.chrome.com/blog/anchor-positioning-api
- web.dev — Four new CSS features for smooth entry/exit animations: https://developer.chrome.com/blog/entry-exit-animations
- web.dev — Invoker Commands: https://developer.chrome.com/blog/command-and-commandfor
- Scott O'Hara — "The popover attribute": https://www.scottohara.me/blog/2023/01/29/popover-semantics.html
- Una Kravets — Carousels & anchor positioning (anchored container queries): https://developer.chrome.com/blog/anchored-container-queries
- WHATWG HTML issue #11694 (Safari popover/anchor + overlay): https://github.com/whatwg/html/issues/11694
- OddBird — CSS anchor positioning polyfill: https://github.com/oddbird/css-anchor-positioning
- Floating UI: https://floating-ui.com/
- Radix Primitives: https://www.radix-ui.com/primitives
- React Aria: https://react-spectrum.adobe.com/react-aria/
- Baseline / Web Platform Dashboard: https://web.dev/baseline

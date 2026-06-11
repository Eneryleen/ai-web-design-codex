# Interaction Design & UI States

> How interactive elements communicate what they do, respond to input, and behave across their full lifecycle. Covers affordances/signifiers, feedback timing, the nine UI states every component needs, microinteractions, optimistic UI, perceived performance, focus management, and keyboard interaction. The senior-level outcome: components that feel responsive, are discoverable, and never leave a user (or a screen reader) guessing.

**Apply when:** designing or building any interactive component, screen, or flow. · **Related:** [Forms & Input UX](forms-and-input-ux.md) · [Accessibility (WCAG 2.2)](accessibility-wcag.md) · [Micro-interactions & Motion](../07-aesthetics-and-trends/micro-interactions-and-motion.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md)

## TL;DR — rules at a glance
- Design **all nine states** for every interactive component: Default, Hover, Focus, Active/Pressed, Disabled, Loading, Error, Empty, Success. Default-only and Default+Hover are the #1 junior gap.
- **Signifiers > affordances.** Every pixel "affords" clicking; only signifiers (cursor, label, border, contrast) make interactivity discoverable. Make interactive things look interactive.
- **Feedback timing windows:** ≤100ms feels instant · ≤1s keeps flow without a spinner · ≥1s needs a spinner/skeleton · ≥10s needs a determinate progress bar with an estimate.
- **Hover ≠ Focus.** Hover is mouse-only and purely visual. Focus is input-agnostic, drives keyboard control, and is non-negotiable for accessibility.
- **Never `outline: none` without an equal-or-better replacement.** Most destructive one-line accessibility regression in CSS.
- **Validate on `blur`, not on every keystroke.** Showing "invalid email" after the first character is a universally hated antipattern.
- **Error copy:** say what went wrong + how to fix it; never blame the user. Don't rely on red alone.
- **Empty states are onboarding.** Headline + why-empty body + a primary CTA matching the action that fills the space.
- **Optimistic UI only for low-stakes, reversible, idempotent actions** (likes, cart, comments). Never for payments, bookings, or irreversible ops. Always store prior state + a rollback.
- **`aria-disabled="true"`** keeps the element focusable and announced; the **HTML `disabled` attribute** removes it from tab order entirely. Pick deliberately; never both at once.
- **Trap focus in modals**, close on `Escape`, and **return focus** to the trigger element on close. Consider `inert` on the background instead of manual key intercept.
- **`aria-live="polite"`** for status/success/validation messages that appear without a focus move — screen readers won't announce them otherwise.
- **Skip link** = first focusable DOM element; visually hidden until `:focus` (via `clip`/`transform`, never `display:none`).
- **Animate `transform`/`opacity` only** (GPU-accelerated). Default easing is `ease-out`, ~200–300ms for micro UI. Respect `prefers-reduced-motion`.
- **Never lazy-load above-the-fold/LCP images.** Skeletons should match real layout precisely to avoid layout shift.

---

## 1. Affordances & signifiers

**Affordance** = the relationship between user and object — what actions are *possible* (a button can be pressed). **Signifier** = the perceivable cue that communicates the affordance (the label, the raised look, the cursor change). In screen UI, *signifiers do the entire job* because every pixel technically affords clicking — there's no physical handle telling a user "grab here." (Don Norman, *The Design of Everyday Things*.)

The **Norman Door** applied to UI: if a user can't tell what an element does, or whether it's interactive at all, that's a **discoverability/feedback failure** — never a user failure. This principle pre-dates touchscreens and scales forward: on mobile, cursor and hover are gone, so labels, shape, and contrast do all the signifier work.

Core signifiers to deploy consistently:
- **Cursor**: `cursor: pointer` for clickable, `not-allowed` for disabled, `text` for editable, `grab`/`grabbing` for draggable, `wait`/`progress` for busy.
- **Visual elevation/contrast**: borders, fills, shadows that separate an interactive element from static text.
- **Labels & icons**: a clear verb ("Save", "Delete") beats an ambiguous icon. If icon-only, add `aria-label` and ideally a tooltip.
- **Underlines for links** in body text. Color alone fails colorblind users and WCAG (don't rely on color as the sole differentiator).
- **Consistency**: the same signifier must mean the same thing everywhere. If primary actions are filled blue, a filled-blue element that *isn't* a primary action is a lie.

```css
button, [role="button"], a { cursor: pointer; }
[aria-disabled="true"], :disabled { cursor: not-allowed; }
```

## 2. Feedback — closing the loop

Every user action must produce a perceivable system response. The thresholds (Nielsen/NNG; originally from Card, Moran & Newell, *The Psychology of Human-Computer Interaction*, 1983):

| Window | Perception | Required response |
|--------|-----------|-------------------|
| ≤100ms | Instantaneous; direct manipulation | Immediate visual change (press state) |
| ~1s | Flow uninterrupted; user notices delay but stays in context | State change; optional inline spinner |
| 1–10s | Attention wanders | Spinner or skeleton; keep UI responsive |
| ≥10s | User will switch tasks | **Determinate** progress bar + time estimate |

**Move feedback earlier than the result.** Show the loading/pressed state *on the input event*, before the server even acknowledges. Users perceive >100ms of dead time as lag.

## 3. The nine interactive states

Designing only the happy path is the most common failure. Spec **every** state before handoff. AI design tools default to happy-path + mock data and will *not* surface error/empty/loading/role states unless explicitly told — so **start from the empty/error state, not the filled one.**

| State | Trigger | What changes | Key rule |
|-------|---------|--------------|----------|
| **Default** | Resting | Baseline tokens | Must already read as interactive |
| **Hover** | Pointer over (mouse only) | Subtle bg/border/elevation | Transition duration 150–200ms; visual only — never the sole way to reveal critical info |
| **Focus** | Tab / programmatic | Visible focus ring | Input-agnostic; WCAG-mandatory; ≥2px, 3:1 contrast |
| **Active/Pressed** | mousedown / key press | Inset/darker/scale-down | Respond within 100–150ms to feel instant |
| **Disabled** | Unavailable | ~38% opacity / muted | Explain *why* nearby; choose `aria-disabled` vs `disabled` deliberately |
| **Loading** | Async in flight | Spinner/skeleton/disabled-with-loader | Lock against double-submit; preserve layout |
| **Error** | Action/validation failed | Message + icon + recovery | State cause + fix; don't blame; not red-only |
| **Empty** | No data yet | Headline + why-empty body + primary CTA | Onboarding moment; CTA must match the action that fills the space |
| **Success** | Action completed | Inline confirmation or toast | Non-blocking; auto-dismiss toasts at 4–5s (or persistent if action is irreversible); no full-page interstitial for routine ops |

**Hover vs Focus — the distinction seniors never blur:** Hover serves mouse users and is decorative. Focus serves keyboard, switch, and screen-reader users and is *functional* (it determines what Enter/Space act on). They can co-exist visually but must be styled and reasoned about separately. Use `:focus-visible` so mouse clicks don't paint a ring while keyboard navigation does.

```css
.btn:hover  { background: var(--btn-hover); }            /* mouse, cosmetic */
.btn:focus-visible {                                     /* keyboard, functional */
  outline: 2px solid var(--focus);
  outline-offset: 2px;                                   /* clear of the border */
}
.btn:active { transform: translateY(1px); }              /* pressed */
```

### 3a. Button state spec (handoff format)

A spec sheet beats a prototype for dev handoff. For each variant (primary/secondary/ghost/destructive) track, per state: **bg, border, text color, icon color, shadow, cursor, loader behavior.**

| State | bg | border | text | shadow | cursor | loader |
|-------|----|--------|------|--------|--------|--------|
| Default | `--brand` | none | `--on-brand` | sm | pointer | — |
| Hover | `--brand-600` | none | `--on-brand` | md | pointer | — |
| Focus | `--brand` | — | `--on-brand` | + ring 2px/3:1 | pointer | — |
| Pressed | `--brand-700` | none | `--on-brand` | none | pointer | — |
| Disabled | `--brand`@38% | none | `--on-brand`@38% | none | not-allowed | — |
| Loading | `--brand` | none | transparent | sm | progress | centered spinner, label hidden |

## 4. Disabled state — the focusability trap

Two mechanisms, **different semantics — never use both:**

- **HTML `disabled`** — removes the element from tab order, ignores all events, screen readers skip it. Use when the element is truly inert and there's nothing to explain.
- **`aria-disabled="true"`** — element stays **focusable and announced** ("dimmed"/"unavailable") but you must block activation in JS. Use when a keyboard user needs to reach it to *understand why* it's disabled (e.g., a submit button gated on a form).

```html
<!-- Reachable + explained -->
<button aria-disabled="true" aria-describedby="why">Publish</button>
<p id="why">Add a title before publishing.</p>
```
```js
btn.addEventListener('click', e => {
  if (btn.getAttribute('aria-disabled') === 'true') { e.preventDefault(); return; }
  /* …submit… */
});
```

**Rules:** Use disabled buttons *sparingly*. "Greyed out for no reason" is a top NNG complaint. Always provide an inline reason or tooltip. Prefer letting the user click and then showing a validation error over a permanently dead button when the fix is non-obvious.

**Loading state — use `aria-disabled` not `disabled`:** A loading button should remain focusable so keyboard users can hear "loading" announced. Set `aria-disabled="true"` + `aria-label="Saving…"` (or `aria-live` in a nearby region), block JS submission, and restore on settle. HTML `disabled` during a load silently removes the button from tab order mid-keyboard session.

## 5. Microinteractions

Dan Saffer's anatomy: **Trigger → Rules → Feedback → Loops/Modes.**
- **Trigger** — what starts it (user action or system event).
- **Rules** — what's allowed to happen.
- **Feedback** — what the user perceives (the visible/audible/haptic part).
- **Loops & Modes** — what happens over time and in edge cases (repeat, timeout, empty data).

The detail layer is what separates polished products from rough ones. **Saffer's longevity principle:** what delights on first use can grate on the 100th. Subtlety scales; spectacle doesn't. Validate every microinteraction by imagining it fired hundreds of times a day.

Good candidates: toggle transitions, like/heart bounce, form-field focus glow, pull-to-refresh, copy-to-clipboard confirmation, button press depression. Each needs designed **error and empty branches**, not just the success animation.

## 6. Transitions & motion

- **Default easing = `ease-out`** (fast start, gentle settle) — it communicates responsiveness and physical weight. `ease-in` only for exits (object leaving). `ease-in-out` for elements moving within view. `linear` only for continuous spinners/progress.
- **Durations:** micro/hover UI **200–300ms**; enter **200–500ms** `ease-out`; exit **150–300ms** `ease-in`; elastic/bounce **800–1200ms** custom `cubic-bezier`. Anything >500ms on a routine UI element feels sluggish.
- **Animate only `transform` and `opacity`** — GPU-composited, no layout/paint. Avoid animating `top/left/width/height/margin` (jank, battery drain, INP regressions).

```css
.card { transition: transform .25s ease-out, opacity .25s ease-out; }
```

- **`prefers-reduced-motion`** — wrap *non-essential* motion in `(prefers-reduced-motion: no-preference)` so only opted-in users animate; supply a reduced fallback for everyone else. (Inverting to a `reduce` query also works but the `no-preference` gate is the cleaner pattern.)

```css
@media (prefers-reduced-motion: no-preference) {
  .modal { animation: slide-up .3s ease-out; }
}
```

> **Caution on the catch-all reset** — the widely-copied `* { transition-duration: .01ms !important }` pattern nukes *all* transitions including functional ones (focus-ring reveal, error-state color change). Prefer scoped `no-preference` gates or explicit `reduce` overrides on specific animations rather than a blanket reset.

- **Modern primitives (2024+):** prefer CSS `@starting-style` + `transition` for enter animations and the **View Transitions API** for page/DOM-state transitions over JS animation libraries for simple cases — less code, no main-thread cost.

## 7. Loading — skeletons vs spinners

| Wait | Use | Why |
|------|-----|-----|
| <1s | **Nothing** | A spinner that flashes for 300ms reads as jitter/annoyance |
| 1–10s | **Skeleton** (content-shaped) or **spinner** | Skeletons reduce perceived wait and bounce; spinner fine for small/atomic loads |
| >10s | **Determinate progress bar** + estimate | Indefinite spinner for 30s is a top frustration — users can't judge whether to wait |

**Skeletons:**
- Must **match the real content's shape and layout precisely** — mismatched skeletons cause a jarring layout shift on load, *erasing* the perceived-performance gain.
- **Shimmer/wave** animation is perceived as shorter than **pulse/opacity-fade**.
- Skeletons reduce perceived load time even when actual load time is unchanged — the mechanism is setting layout expectations and eliminating "blank page" anxiety.

**The filled-duration illusion:** an interval packed with frequent visual changes feels *longer*, not shorter. Over-animating a loading state can paradoxically make the wait more painful. Keep loading motion calm.

## 8. Optimistic UI

Update the UI *immediately* as if the action succeeded, before the server confirms. Appropriate **only** for actions that are **low-stakes, reversible, and idempotent** — likes, follows, comment posts, cart quantity, reordering. **Never** for payments, bookings, irreversible deletes, or anything the user would be harmed by if it silently failed.

**The rollback is the heart of optimistic UI.** Checklist:
1. **Capture** previous state before mutating.
2. **Apply** optimistic change to local state.
3. Ensure **API idempotency** so retries are safe.
4. On failure: **roll back** to captured state + show a subtle, non-modal error with a **retry**.
5. On success: reconcile with server truth.

```js
async function toggleLike(post) {
  const prev = { ...post };                 // 1. capture
  post.liked = !post.liked;                 // 2. apply
  post.likes += post.liked ? 1 : -1;
  render(post);
  try { await api.setLike(post.id, post.liked); }   // idempotent endpoint
  catch { Object.assign(post, prev); render(post); toast('Could not update. Retry?', retry); } // 4. rollback
}
```

**React Query pattern** for concurrent/racing mutations: `cancelQueries` (stop in-flight refetches) → snapshot via `getQueryData` → `setQueryData` to inject the optimistic value → on error restore the snapshot → `invalidateQueries` on settle to converge on server state without flicker. **Sequence chained mutations** — if mutation B depends on A's result, guard the ordering or B will operate on stale optimistic state.

## 9. Perceived performance & Core Web Vitals

Perceived performance often matters more than raw speed for satisfaction: a faster-*feeling* app beats a technically faster one with poor feedback. Levers:
- **Immediate feedback** on input (press/loading state before server ack).
- **Skeletons** that match layout.
- **Optimistic UI** for eligible actions.
- **Progressive rendering** — show shell + above-the-fold first.

Vitals that interaction design directly moves:
- **LCP ≤ 2.5s** at the 75th percentile. **Never lazy-load the LCP/hero image** — measured median 75th-pct LCP **3,546ms with lazy-loading vs 2,922ms without**. Add `fetchpriority="high"` to the LCP image (improved Google Flights LCP **2.6s → 1.9s**; only ~15% of eligible pages used it as of 2024).
- **INP** (Interaction to Next Paint) replaced FID as a Core Web Vital in 2024 — it measures time from user input to the next painted frame. Keep main-thread work short, debounce expensive handlers, and animate compositor-only properties to keep INP low.

```html
<img src="/hero.avif" fetchpriority="high" alt="…">   <!-- LCP image: eager + high priority -->
<img src="/below.avif" loading="lazy" alt="…">         <!-- safe only below the fold -->
```

## 10. Focus management

- **`:focus-visible`**, not `:focus`, so mouse users don't see rings on click but keyboard users do.
- **Logical tab order** = DOM order. **Never** use positive `tabindex` (`1`,`2`…) — it overrides natural order and breaks as content changes. Use `tabindex="0"` (in order) or `tabindex="-1"` (focusable only programmatically).
- **Roving tabindex** for composite widgets (menus, toolbars, radio groups): one stop in tab order, arrow keys move focus within.
- **Move focus on context change** — after opening a dialog, revealing an error summary, or routing in an SPA, send focus to the new content (or its container with `tabindex="-1"`) so keyboard/SR users land in the right place.

### Modal focus trapping (required)
```js
function trapFocus(modal, trigger) {
  const f = modal.querySelectorAll('a[href],button,input,select,textarea,[tabindex]:not([tabindex="-1"])');
  const first = f[0], last = f[f.length - 1];
  first.focus();                                  // move focus in
  modal.addEventListener('keydown', e => {
    if (e.key === 'Escape') return close();        // Esc closes
    if (e.key !== 'Tab') return;
    if (e.shiftKey && document.activeElement === first) { e.preventDefault(); last.focus(); }
    else if (!e.shiftKey && document.activeElement === last) { e.preventDefault(); first.focus(); }
  });
  function close() { modal.hidden = true; trigger.focus(); }  // RETURN focus to trigger
}
```
Rules: Tab/Shift+Tab cycle **within** the modal; **Escape** closes; on close **return focus to the triggering element** (not `document.body`). Add `aria-modal="true"` + `role="dialog"` + a labelled title.

**Modern alternative — `inert`:** Set `inert` on the rest of the page when a modal is open. Supported in all evergreen browsers since 2023. The `inert` attribute removes elements from tab order *and* from accessibility tree — no JS key-intercept needed for the background. Still manually trap focus on the modal itself.

```html
<div id="app" inert>…page content…</div>
<dialog role="dialog" aria-modal="true" aria-labelledby="dlg-title">…</dialog>
```

### `aria-live` for dynamic state changes
When state changes without a focus move (loading completes, error appears, success toast fires), screen readers won't announce the change unless you use a live region. Rules:
- `aria-live="polite"` — announces after the user is idle. Use for status messages, success toasts, field-level validation that appears on blur.
- `aria-live="assertive"` — interrupts immediately. Use only for critical errors (payment failed, session expired). Overuse causes constant interruption.
- Keep live regions in the DOM from page load (injecting them dynamically is unreliable in some SR+browser combos).

```html
<div role="status" aria-live="polite" aria-atomic="true" class="sr-only" id="live-region"></div>
```
```js
document.getElementById('live-region').textContent = 'Changes saved.';
```

### Skip link
First focusable element in the DOM. Visually hidden until focused — hide with `clip`/`transform`, **never `display:none`** (which removes it from keyboard access too).
```html
<a class="skip-link" href="#main">Skip to main content</a>
```
```css
/* Clip approach — no off-screen overflow, RTL-safe */
.skip-link {
  position: absolute;
  clip-path: inset(50%);
  width: 1px; height: 1px;
  overflow: hidden;
  white-space: nowrap;
}
.skip-link:focus {
  clip-path: none;
  width: auto; height: auto;
  top: 1rem; left: 1rem;
}
```

## 11. Keyboard interaction

Match native semantics — use real `<button>`/`<a>`; only add ARIA + key handlers when you can't.

| Key | Expected behavior |
|-----|-------------------|
| `Tab` / `Shift+Tab` | Move forward/back through focusable elements in DOM order |
| `Enter` | Activate buttons & links; submit forms from a text input |
| `Space` | Activate buttons; toggle checkboxes; page-down in scroll |
| `Escape` | Close modal/menu/popover; cancel; revert inline edit |
| `Arrow keys` | Move within composites: menus, tabs, radios, sliders, listboxes, grids |
| `Home` / `End` | Jump to first/last item in a composite |

A custom clickable `<div>` must add `role="button"`, `tabindex="0"`, **and** handle Enter+Space — three things native `<button>` gives free. Prefer native.

## Concrete specs & numbers
- **Focus indicator (WCAG 2.2 SC 2.4.11, Focus Appearance, AA):** ≥ area of a 2px-thick outline around the component, **3:1 contrast** between focused and unfocused states. **Enhanced (SC 2.4.12, AAA):** 4.5:1, larger area, element fully visible when focused.
- **SC 2.4.7 Focus Visible (WCAG 2.1, AA):** visible focus is required. WCAG 2.2 introduced the stronger SC 2.4.11 alongside it; SC 2.4.7 was *not* removed but 2.4.11 is the meaningful conformance target in 2026.
- **SC 1.4.11 Non-Text Contrast (AA):** **3:1** for UI component boundaries and state indicators (hover/focus/active).
- **Hover transition duration:** 150–200ms (not a CSS `transition-delay` — that would feel laggy; this is the `transition-duration` of the color/bg change).
- **Focus paint:** ~100–150ms after Tab. **Pressed feedback:** within 100–150ms.
- **Loading thresholds:** <1s none · 1–10s spinner/skeleton · >10s determinate progress bar.
- **Animation durations:** micro/hover 200–300ms `ease-out` · enter 200–500ms `ease-out` · exit 150–300ms `ease-in` · elastic 800–1200ms.
- **Material Design 3 state-layer opacity** (overlay of content color): hover **8%** · focus **10%** · pressed **10–12%** · dragged **16%** · disabled **38%** on content, **12%** on container.
- **Disabled text:** ~38% opacity (Material Design convention). WCAG SC 1.4.3 explicitly **exempts disabled elements** from contrast requirements — the goal is "visually distinct from enabled" not "4.5:1 readable." Don't cite 4.5:1 for disabled states.
- **LCP ≤ 2.5s** @ p75. Lazy-loaded LCP: **3,546ms vs 2,922ms** p75 (web.dev/optimize-lcp). **`fetchpriority="high"`** on LCP: 2.6s→1.9s (Google Flights, web.dev).
- **Skeletons:** shimmer perceived shorter than pulse; match real layout or layout shift cancels the benefit.

## Tools & libraries
- **Focus trap:** `focus-trap` / `focus-trap-react`; or use accessible primitives (Radix UI, React Aria, Headless UI) that ship trapping, roving tabindex, and Escape handling — prefer these over hand-rolling. See [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md).
- **Optimistic state:** TanStack Query / SWR (`onMutate` + rollback); React 19 `useOptimistic`.
- **Animation:** prefer **CSS `@starting-style` + View Transitions API** for simple cases. Reach for Motion (Framer Motion)/GSAP for orchestration, springs, or scroll-driven sequences — not for a hover color change. See [Animation Libraries & Stack](../04-libraries-and-tools/animation-libraries-stack.md).
- **When NOT to:** don't pull in an animation library for a 250ms hover transition; don't hand-roll modal focus management when a headless lib does it correctly; don't build a spinner where a skeleton matching the layout is cheaper and feels faster.

## Do / Don't
**Do**
- Design all nine states before handoff; start from empty/error, not the happy path.
- Use `:focus-visible` with a ≥2px, 3:1-contrast ring and `outline-offset`.
- Validate on `blur`; clear the error the moment the input becomes valid.
- Trap focus in modals, close on Escape, return focus to the trigger.
- Show feedback on the input event, before the server responds.
- Match skeletons to real layout; use determinate bars past 10s.
- Capture prior state + rollback for every optimistic mutation.
- Use `aria-live="polite"` for any state change that appears without a focus move.

**Don't**
- `outline: none` with no replacement. Ever.
- Use positive `tabindex`, or combine HTML `disabled` with `aria-disabled`.
- Validate on every keystroke, or write blame-y error copy.
- Lazy-load above-the-fold / LCP images.
- Animate `top/left/width/height`; keep to `transform`/`opacity`.
- Use an indefinite spinner for a 30s job, or a skeleton for a sub-1s load.
- Use optimistic UI for payments, bookings, or irreversible actions.

## Common pitfalls & how to avoid them
- **Happy-path-only states** → checklist every component against the nine-state matrix; reviewers reject PRs missing error/empty/loading/success.
- **Removed focus outline** → enforce a project-wide `:focus-visible` token; lint for `outline: none`.
- **Lazy-loaded LCP image** → audit hero/above-the-fold images; `fetchpriority="high"`, never `loading="lazy"` there.
- **Over-animation** → cap routine UI transitions at ~300ms; one microinteraction per interaction; test the "100th-use" feel; honor `prefers-reduced-motion`.
- **Spinner past 10s** → switch to determinate progress with an estimate.
- **Keystroke validation** → move to `onBlur`; debounce only for async availability checks (username taken).
- **Disabled with no reason** → inline helper text or tooltip; or allow the click and show a precise error instead.
- **Optimistic UI without rollback** → never ship a mutation without its rollback path; permanent stale UI is the failure mode.
- **HTML `disabled` where `aria-disabled` was needed** → keyboard users can't reach it to learn why; choose deliberately per §4.
- **Skeleton mismatch / sub-1s skeleton** → match layout exactly; render instantly for fast loads.
- **No modal focus trap / focus not returned** → keyboard users Tab behind the overlay or get dumped at page top. Use `inert` on the background or a headless library; don't hand-roll.
- **Missing `aria-live` region** → screen readers silently ignore dynamic state changes (loading complete, inline error, success toast) when focus doesn't move. Always pair state changes with a polite live announcement.
- **Positive `tabindex`** → confusing, brittle tab order; use 0 / -1 + roving pattern.
- **Chained optimistic mutations** → sequence/guard so B doesn't act on A's unconfirmed state.

## What real users complain about
- *"Why can't I click this? What am I missing?"* — disabled buttons with no explanation (NNG).
- *"I hate error messages written by developers"* — validation firing on the first keystroke and copy that blames the user instead of helping recover.
- **30-second indefinite spinners** with no progress — users can't judge whether to wait, and abandon.
- **Junior design reviews** consistently miss error/empty/loading/disabled states — showing only default + hover is the most common critique at senior designer level.
- **AI-generated UIs** ship only the happy path with mock data; error/empty/role states never appear unless explicitly prompted.
- **Tables** that ignore edge cases: text overflow/truncation per column type, sort/filter state styling, empty state, loading state, mobile collapse, bulk-select state.

## Senior-level checklist
- [ ] All nine states designed and specced (bg/border/text/icon/shadow/cursor/loader) per variant: Default, Hover, Focus, Active, Disabled, Loading, Error, Empty, Success.
- [ ] `:focus-visible` ring ≥2px, ≥3:1 contrast, with `outline-offset`; no bare `outline: none`.
- [ ] Hover and Focus styled and reasoned about separately; hover never the sole reveal of critical info.
- [ ] Feedback fires within timing windows (≤100ms press, spinner ≥1s, progress bar ≥10s).
- [ ] Disabled choice (`disabled` vs `aria-disabled`) deliberate; reason for disablement is visible.
- [ ] Validation on `blur`; errors state cause + fix, don't blame, not color-only.
- [ ] Empty states have headline + why + primary CTA matching the fill action.
- [ ] Optimistic UI limited to reversible/idempotent actions; rollback + retry implemented.
- [ ] Modals trap focus (or use `inert` on background), close on Escape, return focus to trigger; `role="dialog"` + `aria-modal`.
- [ ] `aria-live="polite"` region present for status/success/validation messages that appear without a focus move.
- [ ] Skip link is first focusable element, hidden via clip/transform, visible on focus.
- [ ] No positive `tabindex`; composites use roving tabindex; keyboard map complete.
- [ ] Animations use `transform`/`opacity`, default `ease-out`, ≤300ms routine; `prefers-reduced-motion` respected.
- [ ] LCP image eager + `fetchpriority="high"`; nothing above the fold lazy-loaded.
- [ ] Skeletons match real layout; no sub-1s skeleton flashes.

## Sources
- Nielsen Norman Group — Response Times: The 3 Important Limits — https://www.nngroup.com/articles/response-times-3-important-limits/
- Nielsen Norman Group — Skeleton Screens / progress indicators — https://www.nngroup.com/articles/progress-indicators/
- Card, Moran & Newell — *The Psychology of Human-Computer Interaction* (1983) — original 100ms/1s/10s response-time model
- Don Norman — *The Design of Everyday Things* (affordances vs signifiers)
- Dan Saffer — *Microinteractions* (trigger → rules → feedback → loops/modes)
- W3C WCAG 2.2 (SC 2.4.7, 2.4.11, 2.4.12, 1.4.11) — https://www.w3.org/TR/WCAG22/
- WAI-ARIA Authoring Practices Guide (dialog/focus/keyboard patterns) — https://www.w3.org/WAI/ARIA/apg/
- web.dev — Optimize LCP / `fetchpriority` / don't lazy-load LCP — https://web.dev/articles/optimize-lcp
- web.dev — Interaction to Next Paint (INP) — https://web.dev/articles/inp
- Material Design 3 — State layers — https://m3.material.io/foundations/interaction/states/overview
- MDN — `prefers-reduced-motion` — https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- MDN — View Transitions API — https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API
- TanStack Query — Optimistic Updates — https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates

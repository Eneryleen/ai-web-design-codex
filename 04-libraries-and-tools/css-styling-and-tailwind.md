# CSS, Styling & Tailwind

> How to choose and operate a styling architecture in 2025-2026: Tailwind (utility-first), CSS Modules, zero-runtime CSS-in-TS (vanilla-extract/StyleX/Panda), and modern native CSS (nesting, `@layer`, `:has()`, container queries, custom properties). The senior-level outcome: a maintainable, fast, token-driven styling system that survives team scale and is RSC-compatible.

**Apply when:** Choosing or wiring up any styling approach, defining design tokens, fixing class-soup/specificity/bundle problems. · **Related:** [Design Systems & Design Tokens](../05-process-and-systems/design-systems-and-tokens.md) · [Color Theory & Systems](../00-foundations/color-theory-and-systems.md) · [Component Libraries & Headless UI](./component-libraries-headless.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md)

## TL;DR — rules at a glance
- **Default to Tailwind v4** for new projects, prototypes, shadcn/ui apps, and RSC/Next.js app-router. It is fast, token-native, and tree-shakes to 5–20 kB gzipped.
- **Never use runtime CSS-in-JS (styled-components, Emotion) in React Server Components.** They rely on `useContext` for style injection, which RSC forbids → build errors or FOUC. styled-components has been in maintenance mode since Jan 2024.
- **The fix for class soup is component extraction, not `@apply`.** `@apply` defeats Tailwind's atomic deduplication and re-introduces the BEM naming problem. Reserve it for non-component contexts (base HTML, third-party widgets).
- **In Tailwind v4, define all tokens in CSS via `@theme`.** `tailwind.config.js` no longer exists in v4; one `@theme` block generates both CSS variables and utility classes.
- **Use `@layer reset, base, components, utilities`** declared at the top of your stylesheet to control cascade order. Later layers win over earlier layers — but **unlayered styles always beat any layered style** regardless of specificity; watch for third-party CSS that arrives unlayered.
- **Separate primitive from semantic tokens.** `--blue-500` is primitive; `--color-brand-primary` is semantic. Components reference only semantic tokens.
- **Theme with CSS custom properties, not Sass variables.** CSS variables are runtime-dynamic (dark mode = a `.dark` override, zero JS, zero recompile); Sass variables are compile-time only.
- **Container queries replace most JS-driven responsive logic.** Set `container-type: inline-size` on the parent; query the parent's width, not the viewport.
- **Always use `clsx`/`cn` for conditional Tailwind classes**, and `prettier-plugin-tailwindcss` to canonically sort class strings.
- **Modern vanilla CSS (nesting + `@layer` + container queries + `:has()` + custom properties) covers ~90% of historical SCSS** with no build step.
- **For large multi-theme design systems needing TypeScript type safety, choose vanilla-extract** (zero runtime, RSC-compatible).
- **Tailwind gives zero accessibility guarantees** — semantic HTML, `aria-*`, focus states, and contrast remain entirely your responsibility.
- **Never hardcode hex/spacing in components.** Drive everything from tokens; arbitrary values like `bg-[#3a7bd5]` are a smell, not a feature.

## The core mental model: locality of behavior vs separation of concerns

Utility-first CSS (Tailwind) is a deliberate philosophical inversion. Classic CSS prized **separation of concerns** — semantic class names (`.card__title`) in HTML, styles abstracted into a separate stylesheet. Tailwind prizes **locality of behavior** — styles co-located with the markup that uses them, so a component is fully legible in one file.

This is not "inline styles with extra steps." Utilities are a **constrained design-token API**: `p-4` is not `padding: 16px` arbitrary — it is one stop on a shared scale. The constraint is the value. The cost is verbosity in markup; the payoff is no naming, no dead CSS, no cross-file cascade surprises, and automatic deduplication.

The cascade itself is a **feature, not a bug.** The historical pain was specificity wars and `!important` escalation. The native answer is cascade layers (`@layer`), not abandoning the cascade.

## Styling approaches — decision matrix

| Approach | Runtime cost | RSC-compatible | Type safety | Best for |
|---|---|---|---|---|
| **Tailwind v4** | None (static) | Yes | Via tooling | New projects, RSC apps, shadcn/ui, prototypes |
| **CSS Modules** | None | Yes | Partial (typed-css-modules) | CSS-strong teams, complex animations, non-component rendering |
| **vanilla-extract** | None (build-time) | Yes | Full TS | Large design systems, many themes, type-safe tokens |
| **StyleX (Meta)** | None (atomic) | Yes | Full TS | Very large apps; atomic dedup at scale |
| **Panda CSS** | None (build-time) | Yes | Full TS | Teams wanting CSS-in-JS ergonomics + static output |
| **styled-components** | High (12.7 kB + render) | **No** | Partial | Legacy only; maintenance mode since Jan 2024 |
| **Emotion** | High (7.9 kB + render) | **No** | Partial | Legacy; still under MUI v5 |
| **Modern vanilla CSS** | None | Yes | None | Marketing/art-directed sites, no build step |

**Heuristic:** Tailwind for most apps. CSS Modules when the team thinks in CSS or needs heavy keyframe work. vanilla-extract for a typed design system. Vanilla CSS for boutique/art-directed sites where you'd fight any framework's defaults.

## Tailwind CSS — utility-first done right

### Why it wins (and where it doesn't)
**Pros:** no naming, no dead CSS (unused utilities never ship), atomic deduplication (a utility's CSS is emitted once regardless of usage count), token-constrained by default, excellent with component frameworks, near-instant builds in v4.

**Cons:** verbose markup ("class soup" without discipline), a learning curve for the utility vocabulary, zero accessibility help, and a poor fit for highly bespoke art-directed layouts where you'd reach for arbitrary values constantly.

### Tailwind v4 (released January 22, 2025) — the CSS-first model
v4 deletes `tailwind.config.js`. Configuration moves into CSS via `@theme`. Tokens defined there auto-generate **both** CSS custom properties **and** utility classes from a single source of truth.

```css
/* app.css */
@import "tailwindcss";

@theme {
  --color-brand-primary: oklch(0.62 0.19 255);   /* → bg-brand-primary, text-brand-primary, ... */
  --color-brand-muted:   oklch(0.92 0.03 255);
  --font-display: "Inter", sans-serif;            /* → font-display */
  --spacing-18: 4.5rem;                           /* → p-18, m-18, gap-18 */
  --breakpoint-3xl: 120rem;                        /* → 3xl: variant */
}
```

A token like `--color-brand-primary` simultaneously exposes the CSS variable `var(--color-brand-primary)` and the utilities `bg-brand-primary`, `text-brand-primary`, `border-brand-primary`. No JS config, no duplication.

**v4 also ships:** the Oxide engine (Rust), a P3 wide-gamut default palette, and native cascade-layer output. v4 internally uses `@property` for registered custom properties (enabling typed, animatable tokens). **Minimum browsers:** Safari 16.4+, Chrome 111+, Firefox 128+ (relies on native `@property`, `color-mix()`, cascade layers). If you must support older browsers, stay on v3.

### The `@apply` debate — settled
Tailwind's own team discourages `@apply` outside escape-hatch use. Reasons:
1. It **duplicates CSS output** — the whole point of atomic utilities is that each declaration ships once; `@apply` re-inlines them per component class.
2. It **re-creates the naming problem** (`.btn`, `.btn--primary`) Tailwind was built to remove.
3. It inflates the bundle and decouples the class from the design-token vocabulary.

**The correct abstraction unit is a component**, not a CSS class. When a pattern repeats, extract a React/Vue/Svelte component:

```jsx
// Right: component extraction
function Button({ variant = "primary", className, ...props }) {
  return <button className={cn(buttonStyles[variant], className)} {...props} />;
}

// Wrong: @apply as a primary pattern (defeats dedup, re-introduces BEM)
// .btn-primary { @apply px-4 py-2 bg-brand-primary text-white rounded-md; }
```

Reserve `@apply` for places you genuinely cannot place a component — styling raw `prose` HTML, base element defaults, or third-party DOM you don't control.

### Class soup — the heuristics
- If a `class`/`className` attribute exceeds **one line**, that's the signal to **extract a component**, not to reach for `@apply`.
- Use `class-variance-authority` (CVA) for variant logic (this is how shadcn/ui does it).
- Use `clsx`/`cn` for conditional classes — never raw ternary string interpolation:

```jsx
// Right
<div className={cn("rounded-lg p-4", isActive && "ring-2 ring-brand-primary", className)} />
// Wrong — unmergeable, unreadable in review
<div className={`rounded-lg p-4 ${isActive ? "ring-2 ring-brand-primary" : ""} ${className}`} />
```

`cn` is the standard wrapper of `clsx` + `tailwind-merge` (the latter dedupes conflicting utilities so a later `p-6` overrides an earlier `p-4`).

### Class-ordering convention
Enforce one order so diffs and reviews are sane. Recommended sequence: **layout (flex/grid/gap) → sizing → spacing → typography → colors → borders → effects → states (`hover:`/`focus:`/`dark:`)**. Don't enforce this by hand — install `prettier-plugin-tailwindcss` (auto-sorts to canonical order) and `eslint-plugin-tailwindcss` (flags unknown utilities, wrong order, missing classes).

### Tokens, not arbitrary values
Arbitrary values (`bg-[#3a7bd5]`, `mt-[37px]`) bypass the design system. Each one is a place where the system can drift. Define everything in `@theme` (v4) and treat any arbitrary value in review as a question: "why isn't this a token?"

## CSS Modules — locally scoped, zero runtime
Available since 2015, supported out-of-the-box by every major bundler. You write normal CSS in `Component.module.css`; the build tool hashes class names to globally-unique strings and hands you a JS object.

```css
/* Card.module.css */
.card { padding: var(--space-4); border-radius: var(--radius-lg); }
.card:hover { box-shadow: var(--shadow-md); }
```
```jsx
import styles from "./Card.module.css";
export const Card = (props) => <div className={styles.card} {...props} />;
```

**Strengths:** zero runtime, real CSS (full access to keyframes, pseudo-elements, complex selectors), RSC-compatible, no special vocabulary. **Best for** teams with strong CSS backgrounds, animation-heavy work, or non-component rendering. **Discipline cost:** dead CSS is **not** auto-purged — delete the `.module.css` when you delete the component. Use `typed-css-modules` for type safety on the imported object.

## Zero-runtime CSS-in-TS — vanilla-extract, StyleX, Panda
These give CSS-in-JS authoring ergonomics (co-location, TypeScript types, theme objects) but compile to **static CSS at build time** — no runtime style injection, RSC-compatible.

- **vanilla-extract** — `.css.ts` files, full TypeScript type safety, theme contracts. Ideal for large design systems with many themes. Types catch token typos at compile time.
- **StyleX (Meta)** — atomic CSS-in-JS from the Chakra UI team, open-sourced in 2023. Used internally at Meta; atomic deduplication dramatically reduces bundle size at large scale. No specific public bundle reduction figure has been released.
- **Panda CSS** — created by the Chakra UI team as a standalone zero-runtime alternative; familiar CSS-in-JS DX, static output.

```ts
// vanilla-extract: theme.css.ts
import { createTheme } from "@vanilla-extract/css";
export const [themeClass, vars] = createTheme({
  color: { brandPrimary: "oklch(0.62 0.19 255)" },
  space: { sm: "0.5rem", md: "1rem" },
});
// Button.css.ts
import { style } from "@vanilla-extract/css";
import { vars } from "./theme.css";
export const button = style({ padding: vars.space.md, background: vars.color.brandPrimary });
```

## Runtime CSS-in-JS and its decline
styled-components and Emotion inject styles **at runtime** using React Context + `useContext`. Three problems killed this for new work:

1. **RSC-incompatible.** Server Components cannot use `useContext`/`useState`. styled-components/Emotion therefore error or cause FOUC in streaming SSR within Next.js 13+ app-router. This is architectural, not a bug to be patched.
2. **Render cost.** Style generation happens on every render with dynamic props. Emotion maintainer Sam Magura benchmarked a **48% render-time reduction** (54.3 ms → 27.7 ms on an M1 Max) by migrating from Emotion to Sass Modules in a real production component tree; the gap widens on lower-powered hardware.
3. **Pure overhead bundle.** styled-components is 12.7 kB minzipped, Emotion 7.9 kB — runtime weight that buys zero styling capability over static approaches.

styled-components entered **maintenance mode in January 2024** (critical fixes only, no new features).

**Rule:** do not start new runtime-CSS-in-JS in RSC/Next.js app-router projects. Migrate where feasible; if stuck on MUI v5 (Emotion), isolate it to Client Components.

## Modern vanilla CSS — ~90% of SCSS, no build step

### Cascade layers (`@layer`)
Declare order **once** at the top; later layers always win over earlier layers regardless of selector specificity. **Critical gotcha: unlayered styles beat all layered styles** — third-party CSS that ships without `@layer` will override your carefully layered rules. Solution: import third-party CSS into a low-priority named layer (`@layer third-party { @import "lib.css"; }`).

```css
@layer reset, base, components, utilities;   /* declare order first */
@layer base      { a { color: var(--color-link); } }
@layer utilities { .text-muted { color: var(--color-muted); } }  /* wins over base */
```
Baseline-supported since **Chrome 99 / Firefox 97 / Safari 15.4 (2022)**.

### Native nesting
```css
.card {
  padding: var(--space-4);
  & .title { font-weight: 600; }       /* be explicit with & */
  &:hover  { box-shadow: var(--shadow-md); }
}
```
**Pitfall:** omitting `&` before a child selector triggers `:is()` specificity, which differs from SCSS nesting and can surprise you. Always write `&` for predictable behavior. Nesting has **90%+ global support (2026)**. Unlike SCSS's `&__element` concatenation, native nesting keeps full class names greppable and codemoddable.

### Custom properties (CSS variables)
Runtime-dynamic, JS-readable via `getComputedStyle`, inheritable. The foundation of theming:

```css
:root      { --color-bg: oklch(0.99 0 0); --color-fg: oklch(0.2 0 0); }
.dark      { --color-bg: oklch(0.18 0 0); --color-fg: oklch(0.95 0 0); }
body       { background: var(--color-bg); color: var(--color-fg); }
```
Dark mode is a class swap — **zero JS, zero recompile.** Sass variables can't do this (compile-time only).

**`light-dark()` (2024+):** a native shorthand for two-value color switching tied to `color-scheme`: `color: light-dark(oklch(0.2 0 0), oklch(0.95 0 0))`. Baseline since Chrome 123 / Firefox 120 / Safari 17.5.

### `@property` — registered custom properties
`@property` gives a custom property a **type, initial value, and inheritance flag**, enabling smooth CSS transitions on custom properties and preventing undefined cascade behavior:

```css
@property --hue {
  syntax: "<number>";
  inherits: false;
  initial-value: 255;
}
```
Baseline since Chrome 85 / Firefox 128 / Safari 16.4. Tailwind v4 relies on `@property` internally for token animation support.

### `:has()` — the parent/relational selector
```css
.card:has(img)         { padding-top: 0; }           /* style parent based on child */
label:has(+ input:invalid) { color: var(--color-danger); }
.gallery:has(> :nth-child(4)) { grid-template-columns: repeat(3, 1fr); }
```
**92%+ global support (2026).** Eliminates many JS class-toggling hacks.

### Container queries — portable components
Respond to the **parent's** width, not the viewport. A card placed in a sidebar vs a wide grid adapts itself.

```css
.card-wrap { container-type: inline-size; container-name: card; }
@container card (min-width: 400px) {
  .card { grid-template-columns: 8rem 1fr; }
}
```
**`container-type` values:** `inline-size` (width-only containment, most common) vs `size` (width + height containment, rarely needed). **Container query length units:** `cqw`/`cqh` (1% of container width/height) let you size children relative to their container — useful for fluid typography within a component.

**#1 mistake:** forgetting `container-type: inline-size` on the parent — without it, `@container` matches nothing. Named queries also need `container-name`. This removes one of the last reasons to drive layout from JS.

### `@scope` — explicit style encapsulation
`@scope` (Chrome 118+, 2023; Firefox/Safari in progress) limits rule applicability to a DOM subtree without Shadow DOM:

```css
@scope (.card) to (.card-footer) {
  p { color: var(--color-muted); }   /* only <p> inside .card, not inside .card-footer */
}
```
Useful for component-level encapsulation where CSS Modules feel like overkill and Shadow DOM is unavailable. **Not yet baseline (2026); use as progressive enhancement.**

### `@starting-style` — enter-transition animations
`@starting-style` (Chrome 117+, Firefox 129+) defines the style from which a transition begins when an element is first inserted into the DOM or changes from `display: none`:

```css
.dialog[open] { opacity: 1; transform: translateY(0); transition: opacity 0.2s, transform 0.2s; }
@starting-style { .dialog[open] { opacity: 0; transform: translateY(-8px); } }
```
Eliminates JS-driven "add class on next frame" animation hacks. **Not yet universally baseline; check caniuse before using without a fallback.**

### `color-mix()` — token-based color derivation
```css
.btn-hover { background: color-mix(in oklch, var(--color-brand-primary) 80%, black); }
```
Baseline since Chrome 111 / Firefox 113 / Safari 16.2. Use it to derive hover/active states from a base token instead of defining separate tokens for every shade.

## Design tokens — the pipeline
Tokens are the smallest indivisible unit of a design system. The professional pipeline:

**Figma Variables → JSON (DTCG format) → Style Dictionary → CSS custom properties → framework config (e.g., Tailwind `@theme`)**

- **Two-tier tokens:** *primitive* (`--blue-500`, raw scale values) feed *semantic* (`--color-brand-primary`, `--color-danger`, intent-named). **Components reference only semantic tokens** so a rebrand changes one mapping, not 400 files.
- **DTCG format:** the W3C Design Token Community Group specifies a standard JSON schema for design tokens. Figma Variables now exports in this format; Style Dictionary v4 supports it natively — use DTCG-format JSON as the canonical exchange format.
- **Style Dictionary (Amazon)** transforms one JSON/YAML source to CSS, SCSS, Less, iOS, Android, React Native, and many other platforms — single source, many targets.
- **Automation:** the Figma Variables REST API enables a CI loop — designer edits a token in Figma → CI pulls DTCG JSON → Style Dictionary regenerates CSS → a PR is opened automatically.
- **Tokens Studio** (Figma plugin) and **Supernova** (enterprise, multi-brand) sit on top of this pipeline for Git-synced token workflows.

**Rule:** never hardcode hex or spacing in component files. If you're typing a hex value into a component, the token layer has a gap.

## Concrete specs & numbers
- **Tailwind v4 build speed:** incremental rebuilds dramatically faster than v3 due to the Oxide (Rust) engine; the v4 announcement describes "~5× faster full builds" and "~100× faster incremental." No specific microsecond figure has been officially published; do not cite 192 µs.
- **Production Tailwind bundle:** 5–20 kB gzipped after JIT/purge — *or* megabytes if content paths are misconfigured (v4 auto-detects sources; v3 needs a correct `content` array). 5 kB represents a minimal app; 20 kB is realistic for a full-featured app.
- **Runtime CSS-in-JS render cost:** Emotion maintainer Sam Magura measured a 48% render-time reduction (54.3 ms → 27.7 ms on M1 Max) when moving a real production component tree from Emotion to Sass Modules. Source: "Why We're Breaking Up with CSS-in-JS" (dev.to/srmagura, 2022).
- **Bundle weight (pure runtime overhead):** styled-components 12.7 kB minzipped · Emotion 7.9 kB minzipped.
- **Native CSS support (2026):** nesting 90%+ · `:has()` 92%+ · `@layer` baseline since Chrome 99 / Firefox 97 / Safari 15.4 (2022) · `color-mix()` baseline Chrome 111 / Firefox 113 / Safari 16.2 · `light-dark()` baseline Chrome 123 / Firefox 120 / Safari 17.5 · `@property` baseline Chrome 85 / Firefox 128 / Safari 16.4.
- **Tailwind v4 min browsers:** Safari 16.4+ · Chrome 111+ · Firefox 128+.
- **Tailwind v4 palette:** P3 wide-gamut default (richer than sRGB on modern displays).

## Tools & libraries
- **Tailwind v4** — default utility framework; CSS-first `@theme`; Oxide engine. **Use** for most new apps, RSC, shadcn/ui. **Not** for highly bespoke art-directed layouts.
- **Tailwind v3** — `tailwind.config.js`, JIT; **use** when you must support pre-2023 browsers.
- **CSS Modules** — locally scoped real CSS, zero runtime; **use** for CSS-strong teams / animation-heavy work; **watch** dead-CSS cleanup.
- **vanilla-extract** — zero-runtime CSS-in-TS, full type safety; **use** for large multi-theme design systems.
- **StyleX** — atomic zero-runtime CSS-in-JS from Meta; **use** at very large scale.
- **Panda CSS** — zero-runtime, CSS-in-JS DX from Chakra UI team; **use** when teams want that authoring feel with static output.
- **styled-components / Emotion** — runtime CSS-in-JS; **do NOT use** in new RSC projects (incompatible); legacy/MUI only.
- **Style Dictionary (Amazon)** — token transformation, 50+ output platforms; core of pro pipelines.
- **Tokens Studio / Supernova** — Figma-side token management and Git sync (Supernova = enterprise multi-brand).
- **prettier-plugin-tailwindcss** — canonical class sorting; **essential** at team scale.
- **eslint-plugin-tailwindcss** — lints unknown utilities / order / missing classes.
- **clsx** — conditional class joining; `tailwind-merge` dedupes conflicting Tailwind utilities (later wins); `cn` = standard wrapper of both.
- **class-variance-authority (CVA)** — typed variant API; powers shadcn/ui.
- **shadcn/ui** — Radix UI + Tailwind copy-in components; most popular React kit 2024-2025. See [Component Libraries & Headless UI](./component-libraries-headless.md).
- **Radix UI** — headless accessible primitives; pairs with Tailwind.
- **UnoCSS** — on-demand utility engine, alternative to Tailwind; faster for some setups.

## Do / Don't
**Do**
- Extract a component the moment a class string passes one line.
- Define all tokens in `@theme` (v4) or `tailwind.config.js` (v3); reference semantic tokens only.
- Declare `@layer reset, base, components, utilities` at the top and put rules in the right layer.
- Wrap unlayered third-party CSS in a named `@layer` to prevent it from beating your rules.
- Theme via CSS custom properties (`:root` + `.dark` override); use `light-dark()` for two-value color switching.
- Use `cn`/`clsx` for conditional classes and `prettier-plugin-tailwindcss` for ordering.
- Use container queries for component-portable responsiveness; specify `cqw`/`cqh` units for fluid sizing.
- Pick vanilla-extract/StyleX when you need typed, zero-runtime tokens at scale.
- Use DTCG-format JSON as the canonical token exchange format.

**Don't**
- Use `@apply` as a primary pattern (kills atomic dedup, revives BEM naming).
- Ship runtime CSS-in-JS (styled-components/Emotion) into RSC/Next.js app-router.
- Sprinkle arbitrary values (`bg-[#3a7bd5]`) instead of tokens.
- Mix Tailwind and styled-components in one app (two incompatible philosophies, double the bundle).
- Forget `container-type: inline-size` on the container parent.
- Rely on Tailwind for accessibility — it provides none.
- Leave orphan `.module.css` files when deleting components.
- Cite specific benchmark numbers (µs/ms) not traceable to a published source.

## Common pitfalls & how to avoid them
- **Class soup** — endless utility strings on every element. *Fix:* extract components aggressively; CVA for variants; `cn` for conditionals.
- **Misconfigured content paths (v3)** → PurgeCSS doesn't run → megabytes shipped. *Fix:* ensure `content` covers every template; v4 auto-detects.
- **`@apply` as the main pattern** → duplicated CSS, bigger bundle, BEM-by-the-back-door. *Fix:* components first, `@apply` only as escape hatch.
- **Arbitrary values instead of tokens** → silent design drift. *Fix:* `@theme` tokens; treat arbitrary values as a review question.
- **Dead CSS in CSS Modules** → `.module.css` left after component deletion. *Fix:* delete together; consider lint/CI checks.
- **Runtime CSS-in-JS + RSC** → `useContext` forbidden → build errors / streaming FOUC. *Fix:* zero-runtime approach, or isolate to Client Components.
- **SCSS `&__element` concatenation** → class name ungreppable, breaks codemods. *Fix:* full class names; native nesting keeps them searchable.
- **Tailwind on art-directed sites** → constant arbitrary values, fighting defaults. *Fix:* use vanilla CSS / CSS Modules where layouts are non-standard.
- **Forgetting `&` in native nesting** → unexpected `:is()` specificity. *Fix:* always prefix child selectors with `&`.
- **No `container-type` set** → container queries match nothing. *Fix:* `container-type: inline-size` (and `container-name` for named queries).
- **Unlayered third-party CSS overriding layered rules** → specificity surprise. *Fix:* `@layer third-party { @import "lib.css"; }` to subordinate it.
- **Accessibility assumed** → Tailwind enforces no `alt`, `aria-*`, semantics, or contrast. *Fix:* own a11y deliberately. See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md).

## What practitioners report
- **"Class soup is unreadable."** Real at small scale without discipline; mitigated by component extraction + `prettier-plugin-tailwindcss`. The complaint usually marks a missing extraction habit, not a Tailwind flaw.
- **Tailwind at large developer scale breaks without enforced ordering, shared component primitives, and a token config.** The failure mode is *process*, not the tool; canonical sorting + a component layer + a token source are non-negotiable at scale.
- **SCSS `&__element` concatenation is a maintainability tax** — you cannot grep for the rendered class name, making search and codemods unreliable.
- **Runtime CSS-in-JS perf surprises appear under load and on lower-powered hardware.** The 48% regression measured by the Emotion maintainer (see Sources) was invisible on high-end hardware but significant in production traffic.
- **RSC migration breaks runtime CSS-in-JS projects structurally** — styled-components/Emotion hit hard walls moving to Next.js app-router; there is no clean in-place patch.
- **Tailwind fights bespoke art-directed layouts** — on marketing sites with non-standard layouts, practitioners frequently reach for arbitrary values, at which point plain CSS would have been simpler.

## Senior-level checklist
- [ ] Styling approach chosen against the decision matrix and justified (RSC compatibility, runtime cost, team CSS fluency, theming needs).
- [ ] No runtime CSS-in-JS in RSC/app-router code paths.
- [ ] Tokens defined once (v4 `@theme` / Style Dictionary), split primitive vs semantic; components reference semantic only; DTCG JSON format used as exchange format.
- [ ] Zero hardcoded hex/spacing and no stray arbitrary values in components.
- [ ] `@layer reset, base, components, utilities` declared; third-party CSS wrapped in a named layer; cascade order intentional; no `!important` hacks.
- [ ] `prettier-plugin-tailwindcss` + `eslint-plugin-tailwindcss` wired into the repo.
- [ ] Conditional classes via `cn`/`clsx`; variants via CVA; no raw ternary string interpolation.
- [ ] Repeated patterns are components, not `@apply` blocks; `@apply` only in justified escape hatches.
- [ ] Dark mode via CSS custom-property overrides (`:root` + `.dark`), zero recompile; `light-dark()` used where appropriate.
- [ ] Container queries used for portable components; every container has `container-type`; `cqw`/`cqh` units used for fluid sizing within components.
- [ ] Production CSS bundle verified in the 5–20 kB gzipped range (content/source paths correct).
- [ ] Accessibility owned explicitly: semantic HTML, focus-visible, `aria-*`, contrast ratios checked.
- [ ] Browser support matches the framework floor (Tailwind v4: Safari 16.4+/Chrome 111+/Firefox 128+) or v3 chosen deliberately.
- [ ] CSS Modules (if used) have no orphaned `.module.css` files.
- [ ] `@property` used for custom properties that are animated or need type enforcement.

## Sources
- Tailwind CSS — https://tailwindcss.com/
- Tailwind CSS v4.0 announcement — https://tailwindcss.com/blog/tailwindcss-v4
- vanilla-extract — https://vanilla-extract.style/
- StyleX (Meta) — https://stylexjs.com/
- Panda CSS — https://panda-css.com/
- Style Dictionary (Amazon) — https://styledictionary.com/
- Sam Magura, "Why We're Breaking Up with CSS-in-JS" — https://dev.to/srmagura/why-were-breaking-up-wiht-css-in-js-4g9b
- MDN, Cascade layers (`@layer`) — https://developer.mozilla.org/en-US/docs/Web/CSS/@layer
- MDN, `:has()` — https://developer.mozilla.org/en-US/docs/Web/CSS/:has
- MDN, CSS container queries — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries
- MDN, `@property` — https://developer.mozilla.org/en-US/docs/Web/CSS/@property
- MDN, `color-mix()` — https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix
- MDN, `light-dark()` — https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/light-dark
- MDN, `@scope` — https://developer.mozilla.org/en-US/docs/Web/CSS/@scope
- MDN, `@starting-style` — https://developer.mozilla.org/en-US/docs/Web/CSS/@starting-style
- W3C Design Token Community Group (DTCG) — https://design-tokens.github.io/community-group/format/
- shadcn/ui — https://ui.shadcn.com/
- Radix UI — https://www.radix-ui.com/
- Can I Use (support data) — https://caniuse.com/

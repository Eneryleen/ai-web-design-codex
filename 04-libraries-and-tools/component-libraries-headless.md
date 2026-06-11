# Component Libraries & Headless UI

> How to choose, customize, and own UI components in 2025-2026: the headless model (behavior/accessibility without styles — Radix, Base UI, React Aria, Headless UI, Ark UI, Ariakit), the copy-in model (shadcn/ui), and batteries-included kits (MUI, Mantine, Chakra). The senior-level outcome: accessible, on-brand components you control, with no vendor lock-in, no generic-library look, and no library-mixing footguns.

**Apply when:** Picking a component foundation, building a design system, customizing shadcn/ui, fixing a11y or theming problems, or evaluating bundle/accessibility tradeoffs. · **Related:** [CSS, Styling & Tailwind](./css-styling-and-tailwind.md) · [Design Systems & Design Tokens](../05-process-and-systems/design-systems-and-tokens.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Avoiding Generic 'AI' Aesthetics](../07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md)

## TL;DR — rules at a glance
- **Headless = behavior + accessibility, zero styles.** The library handles focus, ARIA, keyboard nav, RTL; you write every line of CSS. This is the dominant model for 2024-2026 design systems.
- **shadcn/ui is NOT a library — it's a CLI that copies source into your repo** (`npx shadcn add button`). You own the code: no version bumps, no breaking-change management, no lock-in — but bugs and upstream security fixes are now your job.
- **Default stack for most apps:** shadcn/ui + Tailwind v4 + Radix (or Base UI) primitives. This is also what LLMs generate by default — a real ecosystem force multiplier.
- **For accessibility-critical / government / i18n-heavy apps: React Aria.** Deepest WAI-ARIA conformance, 30+ locales, RTL, multiple calendar systems, accessible drag-and-drop. No other library matches it.
- **For React + Vue + Solid from one codebase: Ark UI** (XState machines internally). The only major cross-framework headless option.
- **Base UI 1.0 (released early 2025) is a maintained Radix alternative from the MUI team.** Render-prop pattern instead of `asChild`; positioned as a Radix replacement with active maintenance and a single-package install.
- **Never mix headless libraries in one component tree.** React Aria dropdown inside a Radix modal → focus-trap and layering conflicts. Pick one, stay consistent.
- **The shadcn "beige problem":** leave the CSS variables at defaults and every shadcn site looks identical. Customize `--primary`, `--radius`, and fonts before production.
- **Theme with the 3-layer token model in OKLCH:** primitives → semantic (`--primary`, `--background`) → component overrides. Define under `:root` and `.dark` in `globals.css` *before* writing components.
- **Never `outline: none` without a replacement.** Use `:focus-visible` with a ≥3px outline at ≥3:1 contrast (WCAG 2.2).
- **MUI: never barrel-import** (`@mui/material`). Use path imports (`@mui/material/Button`) — up to 6× slower dev startup otherwise.
- **Batteries-included (MUI/Mantine/Chakra) trades control for speed.** Mantine when you want everything in one box; MUI for enterprise Material; Chakra v3 only on greenfield (v2→v3 is a full rewrite).
- **An `onClick` on a `<div>` is not a button.** Use native `<button>`/`<a>`, or the library's component — never reinvent interactive elements.

## The core mental model: behavior vs presentation

A UI component has two separable concerns:

1. **Behavior** — focus management, ARIA roles/states, keyboard interaction (arrow keys in a menu, Escape to close, type-ahead in a listbox), focus trapping, dismiss-on-outside-click, positioning/collision, RTL, i18n. This is *hard*, easy to get subtly wrong, and identical across every brand.
2. **Presentation** — colors, spacing, typography, radius, motion, the actual look. This is your brand and must be 100% yours.

**Headless libraries ship (1) and explicitly omit (2).** That is the entire pitch. You stop reimplementing accessible comboboxes for the hundredth time and stop fighting a styled library's CSS to match your brand.

```
                 STYLES (you write)
                       ▲
   styled kits  ───────┼──────  headless
   (MUI/Mantine/Chakra)│        (Radix/Base UI/React Aria/Ark/Headless UI)
                       ▼
            BEHAVIOR + A11Y (library writes)
```

**shadcn/ui sits orthogonal to this axis:** it is a *distribution model*, not a library. It takes Radix (or Base UI) headless primitives, pre-wraps them in Tailwind-styled components, and copies that source into your repo. You get a styled starting point that you then own and edit.

## Distribution models — the dimension people miss

| Model | Examples | You get | Updates | Lock-in |
|---|---|---|---|---|
| **npm dependency** | Radix, React Aria, MUI, Mantine, Chakra | A versioned package in `node_modules` | `npm update`; breaking changes are migrations | Yes |
| **Copy-in / open code** | shadcn/ui | Source files *in your repo* | Manual: re-run CLI or hand-merge upstream diffs | No |

The copy-in model inverts the tradeoff. Pros: full control, no breaking-change churn, no vendor risk, you can fix any bug immediately. Cons: you are now the maintainer — upstream a11y fixes, bug fixes, and security patches do **not** reach you automatically. You must watch the upstream repo and port changes by hand.

## Headless library comparison

| Library | Maker | Frameworks | Components | Weekly npm | Stars | Gzip | Standout |
|---|---|---|---|---|---|---|---|
| **Radix Primitives** | WorkOS | React | 28-30 | ~9.1M | 18.8k+ | ~30 KB | Composable anatomy API, `asChild`, ubiquitous (shadcn base) |
| **React Aria** | Adobe | React | 40+ | 4.47M | 15.1k+ | — | Deepest WAI-ARIA, 30+ locales, RTL, calendars, a11y DnD |
| **Headless UI** | Tailwind Labs | React, Vue | ~12 | 5.49M | 28.6k+ | — | Simplest API, tightest Tailwind fit; smallest scope |
| **Base UI** | MUI team | React | 25+ | 3.7M | 9.5k+ | — | Radix successor; render props; shadcn-compatible |
| **Ark UI** | Chakra Systems | React, Vue, Solid | 35+ | 634.7k | 5.2k+ | — | Only cross-framework option; XState internals |
| **Ariakit** | community | React | 25+ | 697.9k | 8.6k+ | — | Smallest bundles relative to feature set |

### Radix Primitives — the de facto standard
- Composable anatomy: `Dialog.Root` / `Dialog.Trigger` / `Dialog.Content` / `Dialog.Overlay` — you assemble the parts.
- `asChild` merges the primitive's behavior onto *your* element instead of rendering an extra wrapper:

```tsx
// Without asChild: renders <button> wrapping your <a> → invalid/extra DOM
// With asChild: behavior merges onto your <a>, no wrapper
<Dialog.Trigger asChild>
  <a href="#" className="btn">Open</a>
</Dialog.Trigger>
```

- **Per-component installs:** `@radix-ui/react-dialog`, `@radix-ui/react-dropdown-menu`, etc. — each is a separate package. Granular tree-shaking, but a typical app using 10-15 primitives accumulates 10-15 `node_modules` entries. In monorepos, deduplicate with `pnpm` workspace resolutions to avoid version-mismatch bugs.
- **Caveats:** update cadence slowed after the WorkOS acquisition; Combobox/multi-select have long-standing gaps; the form component is "an afterthought" and clashes with existing controls — don't use Radix Form, wire your own form layer (React Hook Form + Zod is the standard pairing).

### Base UI — actively maintained Radix alternative
- From the MUI team; launched **1.0 stable in 2025**, positioned as the actively-maintained alternative to Radix.
- Single tree-shakable package `@base-ui/react` (no per-component installs).
- Uses **render props** instead of `asChild` for explicit DOM control; faster rendering.
- **Base UI is a direct npm dependency**, not a copy-in model. Use it as a drop-in where you want active maintenance guarantees; the shadcn/ui CLI can use it as the primitive layer if configured at init (verify against current CLI docs — this was experimental as of 2025).

### React Aria — the accessibility ceiling
- Adobe's React Spectrum foundation; 40+ components; **the correct and only choice for accessibility-critical, government, or heavily-internationalized apps.**
- Built-in: 30+ language translations, RTL, multiple calendar systems, accessible drag-and-drop, keyboard multi-selection, cross-device touch normalization.
- Ships built-in class hooks (`.react-aria-Button`) so you can style without prop drilling.
- **Costs:** renders a large stack of context providers in the React tree — a known DX friction when mixing. Use it for the *entire* component surface or not at all. Teams mixing React Aria into an existing tree reliably hit context-isolation and mobile press-event passthrough bugs that only surface at integration time.

### Headless UI — simplest, smallest
- Tailwind Labs; ~12 components (Combobox, Dialog, Disclosure, Listbox, Menu, Popover, RadioGroup, Select, Switch, Tabs, Textarea, Transition), most beginner-friendly API, tightest Tailwind integration (the `combobox` `virtual` prop is excellent for long lists).
- **Limitation:** tiny scope. You *will* need a second library for anything beyond menus/dialogs/comboboxes/switches — which then risks the mixing footgun below.

### Ark UI — cross-framework
- Chakra Systems; **React + Vue + Solid from one codebase** — the only major option for multi-framework design systems.
- XState state machines internally → predictable behavior in complex widgets (combobox, date picker).
- Style via `data-scope` / `data-part` attributes; single package `@ark-ui/react`.

### Ariakit — bundle-minimal
- 25+ components, smallest bundles relative to features; state-driven styling via `data-active-item` / `data-focus-visible`.
- **When to prefer over Base UI:** when bundle size is a hard constraint *and* you need composable abstractions beyond what Headless UI covers (e.g. combobox, toolbar, composite widgets). Base UI is the better default for most new projects; Ariakit wins when you are trimming every KB.
- Community-maintained (not backed by a vendor); vet maintenance health before adopting on long-lived products.

## shadcn/ui — the copy-in model in depth

**It is not installed as a dependency.** The CLI copies component source into your repo (commonly `components/ui/`), built on Radix (or Base UI) + Tailwind.

```bash
npx shadcn@latest init        # sets up tokens, cn() util, components.json
npx shadcn@latest add button dialog dropdown-menu
# → writes components/ui/button.tsx etc. — now YOUR files
```

- ~50+ copy-paste components; 60+ community themes at `ui.shadcn.com/theme`.
- **Shadcn Create** (introduced with the v2 CLI overhaul, late 2024) scaffolds a pre-themed project from design presets — use it on greenfield to escape the default look immediately.
- **The `cn()` helper** (clsx + tailwind-merge) is how variants compose; keep it.

### The token architecture (do this BEFORE writing any page)
Three layers, OKLCH for perceptually-uniform steps, defined under `:root` and `.dark`, exposed to Tailwind v4 via `@theme inline`:

```css
/* globals.css */
:root {
  /* 2 — semantic tokens (intent), referenced by components */
  --background: oklch(1 0 0);
  --foreground: oklch(0.15 0 0);
  --primary: oklch(0.55 0.22 264);          /* your brand, NOT default */
  --primary-foreground: oklch(0.98 0 0);
  --radius: 0.5rem;
}
.dark {
  --background: oklch(0.16 0 0);
  --foreground: oklch(0.96 0 0);
  --primary: oklch(0.62 0.20 264);
  --primary-foreground: oklch(0.16 0 0);
}
@theme inline {                              /* 3 — expose to Tailwind utilities */
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --radius-lg: var(--radius);
}
```
Layer 1 (primitives like `--color-blue-500: oklch(0.6 0.16 250)`) feeds the semantic layer. Retrofitting tokens after building pages is painful — commit upfront.

### Killing the "beige" generic look
Every shadcn project that ships defaults looks the same. Differentiate by changing, at minimum:
- **`--primary`** to your real brand hue in OKLCH — not just a nearby gray-blue. A chroma value below 0.10 still reads as beige; aim for ≥0.15 chroma for a recognizable brand color.
- **`--radius`** — `0.125rem` reads enterprise/sharp; `0.75rem` reads friendly/rounded; `1.5rem+` reads pill/consumer. This is the single fastest aesthetic differentiator.
- **Fonts** — swap Inter (the default) for a deliberate type pairing. A geometric sans + a serif already separates you from 90% of shadcn sites.
- **Density & shadows** — tighten padding (`py-1` buttons vs default `py-2`) for compact enterprise feel; add multi-stop shadows for depth. Defaults are deliberately neutral.
- **At least 3 of these 4 must change** before the site stops reading as "default shadcn."

Tools: `tweakcn.com` (interactive editor, Tailwind v4) and `shadcnstudio.com` (generator + previewer) — pull a full preset *before* writing pages. See [Avoiding Generic 'AI' Aesthetics](../07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md).

### Tailwind v3 → v4 caveat
New shadcn projects auto-start on Tailwind v4 (CSS-first `@theme`, OKLCH, no `tailwind.config.js` palettes). Existing v3-era shadcn components need migration on upgrade — existing projects stay on v3 until you migrate them. Don't half-migrate.

## Batteries-included kits — when control matters less than speed

| Kit | Components | Weekly npm | Stars | Styling | Bundle | Use when |
|---|---|---|---|---|---|---|
| **MUI** | 100+ | 5.8M | 93k+ | CSS-in-JS (Emotion) | 300 KB+ full | Enterprise + Material Design |
| **Mantine 9** | 120+ + 100+ hooks | ~1.2M | 28k+ | CSS Modules (no runtime) | ~80-120 KB | "Everything in one box" |
| **Chakra UI v3** | 60+ | ~1.5M | 38k+ | Panda CSS (was Emotion) | ~70-100 KB | Greenfield only (v2→v3 = rewrite) |

### MUI — Material, with footguns
- Most components and the largest ecosystem, but **Material Design is losing aesthetic favor** (a core team member publicly conceded MD "doesn't look good to many people nowadays"). MUI v6 (Sept 2024) introduced MD3 (Material You) support, but the overall aesthetic remains visibly Material — a hard sell for bespoke brands.
- **Never barrel-import.** `@mui/icons-material` named imports can be 6× slower in dev; path imports cut dev startup up to 60% and shave 100 KB+:
```tsx
import Button from '@mui/material/Button';        // ✅ path import
// import { Button } from '@mui/material';        // ❌ barrel — slow dev, huge bundle
```
- `sx` prop costs ~0.2ms/component (≈200ms for 1,000 elements) — fine for pages, a problem in long lists; prefer static styles at scale.
- MUI X DataGrid alone is 90 KB+ gzipped; advanced DataGrid/date-pickers are **paid**. Lighter stacks can cut ~54% bundle vs MUI → measurably faster FCP.
- MUI's own headless layer is **Base UI** (covered above).

### Mantine 9 — the one-box option
- 120+ components, 100+ hooks, **CSS Modules → zero runtime overhead.**
- Rich sub-packages: `@mantine/form`, `@mantine/dates`, `@mantine/notifications`, `@mantine/tiptap`, `@mantine/spotlight`, schedule (v9). Best fit when you want forms, dates, notifications, rich-text all from one vendor.

### Chakra UI v3 — greenfield only
- Late-2024 **complete rewrite:** Emotion → Panda CSS, adopted Ark UI for component logic, React 18+. v2→v3 is a full API break — migration is expensive enough that teams stay on v2 indefinitely. Do **not** start a new project on v2.

## Accessibility — what the library gives vs. what you owe

Headless libraries hand you correct ARIA, roles, keyboard nav, and focus management. **They cannot fix your visual choices.** You still owe:

- **Contrast (WCAG 2.2):** 4.5:1 normal text (<18pt), 3:1 large text (≥18pt or ≥14pt bold), 3:1 for focus indicators and UI component boundaries.
- **Focus indicators:** use `:focus-visible`, never raw `:focus` (avoids ring on mouse click while keeping it for keyboard). Never `outline: none` without an alternative — minimum 3px solid at 3:1 contrast against adjacent colors.
```css
.btn:focus-visible { outline: 3px solid var(--ring); outline-offset: 2px; }
```
- **Touch targets:** 24×24 CSS px minimum (WCAG 2.5.8, AA). Aim higher: 44×44px (Apple HIG), 48×48dp (Material).
- **Skip link:** a visually-hidden "Skip to main content" anchor as the first focusable element on every page — mandatory for keyboard users.
- **Focus trap modals/drawers:** focus the first focusable element on open, restore focus to the trigger on close. Radix and React Aria do this automatically; if you hand-roll, you own it.
- **Never fake interactives:** `onClick` on a `<div>`/`<span>` without `role`, `tabIndex={0}`, and keydown handlers is invisible to keyboard and screen-reader users. Use the real element.

See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) for the full standard.

## Decision guide — pick one and commit

```
Need React + Vue + Solid from one codebase? ───────────────▶ Ark UI
A11y-critical / government / 30+ locales / RTL / calendars? ▶ React Aria (whole surface)
Want everything (forms/dates/notifications) in one box? ────▶ Mantine 9
Enterprise mandate for Material Design? ───────────────────▶ MUI (path imports!)
Shipping fast, brand control, own your code? ──────────────▶ shadcn/ui + Tailwind v4
Building a bespoke design system, strong CSS team? ────────▶ Radix or Base UI primitives
Bundle size is the hard constraint? ──────────────────────▶ Ariakit
```

**Rule:** brand demands pixel-perfect control + strong CSS team → headless primitive. Speed-to-market wins → shadcn/ui. Need-it-all → Mantine. Enterprise Material → MUI. And **whatever you pick, do not mix it with another headless library in the same tree.**

## Concrete specs & numbers
- **Bundles (gzip, approximate):** Radix per-component packages vary (each ~3-10 KB, total ~30 KB for a typical set) · shadcn/ui ~35-50 KB per typical install · Chakra v3 ~70-100 KB · Mantine ~80-120 KB · MUI full 300 KB+ · MUI X DataGrid alone 90 KB+.
- **MUI dev perf:** path imports vs barrel → up to 60% faster dev-server startup, −100 KB+ bundle. `sx` ≈ 0.2ms/component (~200ms for 1,000 elements). Lighter stacks measure ~54% smaller bundle than MUI full.
- **WCAG 2.2 (current standard, published Oct 5 2023):** contrast 4.5:1 normal text, 3:1 large text, 3:1 focus indicators; min touch target 24×24 CSS px (AA).
- **WAI-ARIA:** current W3C Recommendation is **1.2**; 1.3 is a working draft. Target 1.2 for production compliance.
- **shadcn:** 50+ components, 60+ community themes; CLI invocations are not formally published — treat any specific weekly figure as unverified.
- **Color:** define tokens in OKLCH (perceptually-uniform lightness steps) for consistent dark-mode and palette generation.

## Tools & libraries
- **shadcn/ui CLI** — `npx shadcn add <component>`; copies styled Radix/Base-UI components into your repo. Use for most apps; don't treat as an npm dep.
- **Shadcn Create** — scaffold a pre-themed project from presets (greenfield only).
- **tweakcn.com / shadcnstudio.com** — shadcn theme editors with Tailwind v4 + OKLCH; pull a preset before building.
- **Radix Primitives** — headless React base; per-component packages.
- **Base UI (`@base-ui/react`)** — maintained Radix successor, single package, render props.
- **React Aria Components** — max accessibility/i18n; whole-surface use only.
- **Headless UI / Ark UI / Ariakit** — Tailwind-tight / cross-framework / bundle-minimal respectively.
- **eslint-plugin-jsx-a11y** — AST lint for JSX a11y issues in CI.
- **jest-axe** — unit-level automated a11y assertions on rendered components.
- **Axe DevTools** — browser extension scanning live pages for WCAG violations.
- **When NOT to use:** don't add `vaul` (shadcn's drawer) for new production work — effectively unmaintained as of 2025; build a custom drawer or wait for Base UI's swipeable drawer. Don't use Radix Form — wire your own form layer.

## Do / Don't
**Do**
- Treat shadcn/ui as source you own; watch upstream and port a11y/security fixes by hand.
- Define the 3-layer OKLCH token model under `:root` + `.dark` before building pages.
- Customize `--primary`, `--radius`, fonts, density to escape the default look.
- Use native interactive elements or the library's components; `:focus-visible` for rings.
- Pick exactly one component foundation per app and stay consistent.
- Path-import MUI; lazy-load heavy widgets (DataGrid, rich-text, date pickers).

**Don't**
- Compare shadcn/ui to MUI/Chakra as if they're the same paradigm — they aren't.
- Ship shadcn defaults to production (the "beige" problem).
- Mix React Aria and Radix (or any two headless libs) in one component tree.
- `outline: none` without a visible, contrast-compliant replacement.
- Barrel-import MUI / `@mui/icons-material`.
- Start a new project on Chakra v2, or rely on Vaul / Radix Form for production.

## Common pitfalls & how to avoid them
- **Treating shadcn as an npm library.** It's source in your repo; updates are manual. Decide your upstream-tracking process up front.
- **Default CSS variables left untouched** → identical-looking sites. Customize tokens before production.
- **Mixing headless libraries.** React Aria dropdown inside a Radix modal → focus-trap/layering conflicts. One library per tree.
- **MUI barrel imports.** Up to 6× slower dev, bloated bundle. Always path-import.
- **Radix Form / Radix Combobox.** Form is an afterthought; Combobox/multi-select have unfixed gaps post-WorkOS. Bring your own form layer; vet the combobox.
- **Faking buttons with `<div onClick>`.** Inaccessible to keyboard + screen readers. Use `<button>`/`<a>` or the library component.
- **Chakra v2 on new projects.** v3 is a full rewrite; you'll be stranded.
- **Vaul in production.** Deprecated/unmaintained; replace or wait for Base UI's drawer.
- **React Aria provider stack mid-mix.** Heavy context tree; commit to whole-surface adoption.

## What real users complain about
- *"shadcn isn't a library, why does it have no version?"* — confusion from the copy-in model; people expect `npm update` and there is none.
- **Every shadcn site looks the same** — the "beige" default theme is instantly recognizable; teams that skip token customization ship visibly generic UI.
- **Radix has stalled** since the WorkOS acquisition — Combobox and multi-select cited as long-unfixed; this drove migration interest toward Base UI.
- **Radix Form "doesn't play well"** with existing control components; widely treated as an afterthought.
- **React Aria's iOS/Android press-event bugs** (`onPressEnd` passthrough on ComboBox items) burned teams mid-adoption before 2025 fixes in the React Spectrum repo — check release notes if on an older pin.
- **MUI feels dated and heavy** — Material aesthetics out of favor, bundle pain; one core member publicly conceded MD "doesn't look good to many people nowadays." MUI v6 shipped MD3 support but the overall aesthetic is still visibly Material.
- **Chakra v2→v3 migration is a wall** — full API break (Emotion→Panda); many teams simply stay on v2.
- **Headless UI is too small** — covers so few components that mixing in another library becomes "necessary," which then triggers the mixing conflicts.
- **Vaul abandonment** leaves shadcn drawer users maintaining a dependency the community calls "practically deprecated."

## Senior-level checklist
- [ ] One component foundation chosen for the whole app; no second headless library in any shared tree.
- [ ] Choice justified against constraints: a11y depth, frameworks, brand control, bundle, speed.
- [ ] (shadcn) 3-layer OKLCH tokens defined under `:root` + `.dark`, exposed via `@theme inline`, *before* page work.
- [ ] (shadcn) `--primary`, `--radius`, fonts, density, shadows customized — does not look like default.
- [ ] (shadcn) Upstream-tracking process agreed for porting a11y/security fixes.
- [ ] (MUI) Path imports enforced (lint rule); heavy widgets lazy-loaded; paid X components budgeted.
- [ ] All interactive elements native or library components; no `<div onClick>` controls.
- [ ] `:focus-visible` rings everywhere; no `outline: none` without a ≥3px / ≥3:1 replacement.
- [ ] Contrast meets WCAG 2.2 (4.5:1 / 3:1 / 3:1); touch targets ≥24×24 CSS px.
- [ ] Skip link present as first focusable element; modals/drawers trap and restore focus.
- [ ] `eslint-plugin-jsx-a11y` in CI; `jest-axe` on key components; Axe DevTools spot-check passes.
- [ ] No production dependency on Vaul or Radix Form unless explicitly vetted.
- [ ] Tailwind version consistent (don't half-migrate v3→v4).

## Sources
- shadcn/ui — https://ui.shadcn.com
- Radix Primitives — https://www.radix-ui.com/primitives
- Base UI — https://base-ui.com
- React Aria Components — https://react-spectrum.adobe.com/react-aria/
- Headless UI — https://headlessui.com
- Ark UI — https://ark-ui.com
- Ariakit — https://ariakit.org
- Mantine — https://mantine.dev
- Chakra UI — https://chakra-ui.com
- MUI — https://mui.com
- Tailwind CSS — https://tailwindcss.com
- tweakcn — https://tweakcn.com
- WCAG 2.2 — https://www.w3.org/TR/WCAG22/
- WAI-ARIA 1.2 (current Recommendation) — https://www.w3.org/TR/wai-aria-1.2/
- WAI-ARIA 1.3 (working draft) — https://www.w3.org/TR/wai-aria-1.3/

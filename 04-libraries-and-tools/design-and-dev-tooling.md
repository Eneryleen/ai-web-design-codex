# Design & Dev Tooling

> The non-runtime tooling that makes great design shippable: Figma (auto-layout, variables, dev mode) and the W3C token pipeline, prototyping, Vite, font subsetting/self-hosting, image pipelines (sharp), Storybook, ESLint/Prettier, Lighthouse/PageSpeed, browser devtools, and color/contrast checkers. The senior-level outcome: a build-and-QA loop where design tokens flow Figma→code, fonts and images are optimized by default, accessibility/perf are measured continuously, and handoff produces real layout behavior — not screenshots.

**Apply when:** Setting up a project's build/QA toolchain, wiring design-token sync, optimizing fonts/images, doing design QA on a build, or reading a Figma handoff. · **Related:** [Design Systems & Design Tokens](../05-process-and-systems/design-systems-and-tokens.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Wireframing, Prototyping, Handoff & QA](../05-process-and-systems/wireframing-prototyping-handoff-qa.md)

## TL;DR — rules at a glance
- **Figma Auto Layout maps 1:1 to Flexbox**: direction→`flex-direction`, spacing→`gap`, padding→`padding`, Hug→content-sized, Fill→`flex:1`. Figma added CSS Grid layout mode at Config 2024 (May 2024) and renamed the old "auto layout direction"→"auto layout flow." A handoff without Auto Layout is nearly useless for dev.
- **Establish variable architecture (primitives → semantic → modes) before designing a single screen.** Not optional for any system meant to scale. Figma Variable types: Number, Color, String, Boolean, Alias. (Composite/Array types were proposed but not shipped as of mid-2026.)
- **Figma Dev Mode is paid org-seat only.** An individual Professional plan does NOT unlock Dev Mode on files you don't own — the designer must add you as a paid seat in their org. Free Viewers still get Alt-hover distances, right-click → Copy as → CSS, and the Properties panel.
- **Never paste Figma's generated CSS verbatim into production.** It is context-free absolute px, ignores your tokens, and inverts how design systems work. Use it for reference distances/colors only.
- **The DTCG format spec reached a stable v1.0 milestone** (Community Group, not a W3C Recommendation) providing cross-tool token portability without bespoke transform scripts. Canonical pipeline: Tokens Studio → DTCG JSON in Git → Style Dictionary → platform outputs.
- **Vite by default for SPA/component dev.** Cold start ~200ms vs Webpack ~2.5s; esbuild pre-bundles `node_modules` 10–20× faster than Babel; Rolldown (Rust) is in active development as Vite's future production bundler but was still experimental in stable releases as of mid-2026. HMR is near-instant module-level. Note: Next.js uses Turbopack, not Vite; Remix uses Vite.
- **Self-host fonts; subset to the charset you actually render.** Median site ships 238 KB of fonts but renders ~101 glyphs from a 248+-glyph file. Latin/ASCII subset → 60–76% smaller. Google Fonts CDN no longer gives cross-site cache (Chrome 86, 2020) and violates GDPR in the EU.
- **WOFF2 only** (~30% smaller than older formats, ~97% support). Use `@fontsource-variable/[name]` for variable fonts via npm.
- **sharp for image pipelines, not Squoosh CLI.** Squoosh CLI is unmaintained since 2023 and broken on Node 18+. Squoosh's web app is still fine for one-off manual comparison. Ship WebP (~95% support) broadly; AVIF (~92%) when CPU cost is acceptable.
- **Lighthouse = lab only.** It cannot measure INP (uses TBT as a proxy). Always check PageSpeed Insights for real-user CrUX field data over its 28-day window before declaring a page "fast."
- **Contrast: 4.5:1 normal text, 3:1 large text (≥18pt / ≥14pt bold) and UI/graphics** (WCAG 2.2, unchanged). AAA = 7:1 / 4.5:1. Contrast is the #1 a11y failure: 83.6% of sites fail it.
- **ESLint (Flat Config, v9+ default) for correctness, Prettier for formatting, `eslint-config-prettier` to turn off conflicting ESLint style rules.** `eslint-plugin-prettier/recommended` bundles both — put it last in the config array. Enforce via Husky + lint-staged. Do not confuse the two packages: `eslint-config-prettier` = turns off rules; `eslint-plugin-prettier` = runs Prettier as a rule.
- **Storybook CSF is the source of truth** for docs + visual tests simultaneously. Don't migrate stories off CSF — story metadata (title, args, argTypes, play functions) travels with the file and is consumed by Storybook Docs, Chromatic, and test runners. Non-CSF formats lose this portability.

---

## Figma: Auto Layout = CSS layout

Auto Layout is not a "nice-to-have." It is the only way Figma encodes real layout intent, and it translates directly to Flexbox. A designer who positions elements absolutely produces measurements that do not correspond to how the layout reflows — useless for responsive dev.

| Figma Auto Layout | CSS |
|---|---|
| Flow: Horizontal / Vertical | `flex-direction: row / column` |
| Spacing between items | `gap` |
| Padding (4-side or per-side) | `padding` |
| "Hug contents" | container sized to content (default flex item) |
| "Fill container" | `flex: 1` (grow to fill) |
| "Fixed" width/height | explicit `width` / `height` |
| Alignment grid (9-point) | `justify-content` + `align-items` |
| Distribute "Space between" | `justify-content: space-between` |
| Wrap | `flex-wrap: wrap` |
| (Config 2024) CSS Grid mode | `display: grid` + `grid-template-*` |

**Senior reading of a handoff:** Hug + Hug = an inline-ish content box; Fill + Hug = a row that stretches horizontally but is content-tall (most nav bars, cards). If a frame has no Auto Layout, treat px positions as approximate and ask for it to be added.

## Figma: Variables = design tokens

Variables are Figma's token system. Build a three-tier architecture before any screen design:

1. **Primitives** — raw values: `color/blue/500 = #3B82F6`, `space/4 = 16`. No semantic meaning.
2. **Semantic tokens** — alias primitives to intent: `color/bg/brand → color/blue/500`, `color/text/default → color/gray/900`. Components reference only these.
3. **Modes** — light/dark/high-contrast/brand are *modes* of the same semantic collection. Switching a mode reassigns every alias atomically.

Variable types: **Number** (spacing, radius, type scale), **Color**, **String** (content, font-family names), **Boolean** (feature flags / show-hide), **Alias** (reference to another variable — the backbone of semantic tokens). Composite/Array types were proposed but not shipped as of mid-2026.

```
// Tier 1 primitive        // Tier 2 semantic (alias)      // Tier 3 mode override
color/gray/900 = #111827   color/text/default → gray/900   [dark]  text/default → gray/050
color/gray/050 = #F9FAFB   color/bg/surface   → gray/050   [dark]  bg/surface   → gray/900
```

This is the same primitive→semantic split used in code (see [CSS, Styling & Tailwind](./css-styling-and-tailwind.md) and [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md)). Keeping the tiers identical across Figma and code is what makes sync lossless.

## Figma: Dev Mode — what you actually get, and the pricing trap

Dev Mode is a **paid org-seat feature.** Reality of access tiers:

| Role | Has | Lacks |
|---|---|---|
| Paid Dev/Full seat in the org | Full Dev Mode workspace, code panel, "Ready for Dev" status, change-compare, plugins, MCP | — |
| Individual Professional plan (on someone else's org file) | Nothing extra | **No Dev Mode** — must be added as paid seat in *that* org |
| Free Viewer | Alt-hover distance measurement, right-click → Copy as → CSS, Properties panel, inspect values | Dedicated Dev Mode workspace, "Ready for Dev", compare changes |

**The genuinely useful Dev Mode features** (per practitioner consensus) are the **"Ready for Dev" status** and **change-compare diff** — not the CSS copy-paste. Tailwind users additionally get class suggestions, which is the most-cited concrete value; non-Tailwind / strict-token teams get almost nothing from auto-generated CSS.

**MCP integration:** Figma's MCP server lets AI coding assistants (Cursor, Copilot, Claude) read selected frame structure + variables directly. This is far better than screenshotting — but still: feed it your token names, don't accept raw px.

## The token sync pipeline (Figma ⇄ code)

The **Design Tokens Community Group (DTCG)** spec reached a stable v1.0 milestone (a Community Group deliverable, not a W3C Recommendation), standardizing token JSON so Figma, Sketch, and Penpot can exchange tokens without bespoke transforms.

**Canonical bidirectional pipeline:**
```
Figma Variables
  └─(Tokens Studio plugin)→ DTCG JSON  ──push/pull──  Git repo (GitHub/GitLab)
                                                          │
                                              (Style Dictionary build)
                                                          ▼
                          CSS vars · Tailwind theme · iOS · Android · JSON
```
- **Tokens Studio** — Figma plugin; bidirectional sync to Git as DTCG JSON; multi-mode (light/dark).
- **Style Dictionary** — transforms one DTCG source into every platform's format. The single transform engine.
- **Penpot** — open-source, self-hostable Figma alternative with *native* W3C token support (no plugin).
- **Tokens Studio free tier** — sufficient for single-file/smaller projects without multi-mode Git sync requirements.

Rule: **tokens live in Git as the source of truth.** Figma and code are both consumers of the same DTCG JSON. Never hand-copy hex values between the two.

## Prototyping tools

Prototypes exist on a fidelity spectrum. Choose the tool that matches the question being answered — don't over-build.

| Fidelity | Tool | When to use |
|---|---|---|
| Low (flow/structure) | Figma prototype mode, pen+paper | Does the flow make sense? No assets needed. |
| Mid (interaction feel) | Figma Smart Animate + overlays, Principle | Does the transition feel right? Micro-interactions. |
| High (real code behavior) | Framer, ProtoPie | Complex conditional logic, sensor input, multi-step flows. |
| Production-equivalent | Storybook play functions, custom Next/Vite page | "Does it work in the actual stack?" — not a prototype, a spike. |

**Figma prototype mode** covers 80% of needs: set `Overflow: Horizontal/Vertical scroll` on frames, use **Smart Animate** for transitions (it interpolates matching layer names between frames — works exactly like CSS `transition` on named elements), and overlay components for modal/drawer patterns. Sufficient for most client/stakeholder reviews.

**Framer** is the right upgrade when: (1) conditional logic branches are too complex for Figma's prototype graph; (2) you need real web-rendered output for stakeholder sign-off; (3) the component uses a state machine the designer needs to validate visually. Framer exports React — treat exported code as a reference, not production-ready.

**ProtoPie** excels at hardware/sensor interactions (scroll velocity, device tilt, camera) and multi-screen conditional logic — common in mobile-native prototyping. Not needed for typical web work.

**Rule:** Prototypes are throwaway validation artifacts. Budget 2–4 hours max on a mid-fi Figma prototype, 1–2 days on a high-fi Framer prototype. If it takes longer, build the real feature instead.

## Vite as the dev/build core

| Metric | Vite | Webpack |
|---|---|---|
| Dev server cold start | ~200ms | ~2.5s (≈15×) |
| Dependency pre-bundle | esbuild (10–20× faster than Babel) | Babel |
| HMR update | near-instant (module-level) | 500–1600ms |
| Production bundler | Rolldown (Rust, replaces Rollup — experimental in stable as of mid-2026) | Webpack |

**Scope:** Vite applies to SPA/component dev, Remix, and Astro. **Next.js uses Turbopack** (Rust, default dev server in Next 15+) — Vite advice does not apply there. The rest of this section targets Vite projects.

**Operate it correctly:**
- **Never leave DevTools "Disable cache" on** during Vite dev — it murders startup and reload speed by refetching pre-bundled deps every reload.
- **Avoid barrel files.** `index.js` re-exporting many modules forces Vite to load the whole barrel on startup. Import the leaf directly:
  ```js
  import { formatDate } from './utils/formatDate.js'   // good
  import { formatDate } from './utils'                  // bad — pulls the whole barrel
  ```
- **Profile slow starts** with `vite --profile`, then open the dump in [speedscope](https://www.speedscope.app/) to find the heavy plugin/transform.

## Font tooling: subsetting + self-hosting

Fonts are the highest-ROI, most-neglected optimization. Hard numbers (top-1M, 2024): 81.5% of sites use ≥1 web font; average page loads **238 KB** of fonts; 63.6% load fonts before FCP; 25% load >75 KB before FCP. Median site ships 3 fonts / 95 KB / **101 visible glyphs**, while its smallest font already contains **248 glyphs** (2–3× more than rendered).

**Subset to the charset you render.** Real cases: Montserrat full→ASCII subset 64.6 KB → **15 KB (−76%)**; Tom Hazledine 196.9 → 71.3 KB (−63%); Kia.com 3 MB → 30 KB (Latin); Paul Conroy 400 → 140 KB (LCP −20%, FCP −10%).

**Tools:**
- **Fontsource** — `npm i @fontsource/inter` (static) or `@fontsource-variable/inter` (variable). Self-hosted, no CDN round-trip. 15–30% better metrics than Google Fonts CDN on typical 4G.
- **Google Webfonts Helper** (`gwfh.mranftl.com`) — download subset WOFF2 + generate `@font-face`. Reported 180 KB → 18 KB.
- **glyphhanger** (Zach Leatherman) — `--LATIN` / `--US_ASCII`, TTF→WOFF2.
- **fonttools / pyftsubset** — precise unicode-range control:
  ```bash
  pyftsubset font.ttf --unicodes="U+0000-00FF,U+2018-2019,U+201C-201D" \
    --flavor=woff2 --output-file=font-latin.woff2
  ```

**Self-host, full stop.** Cross-site Google Fonts caching was removed in **Chrome 86 (2020)** — "users already have it cached" is false. Google Fonts CDN also **violates GDPR in the EU** (German court, Jan 2022) by leaking IPs. Self-hosting is strictly faster *and* compliant.

**Use `unicode-range` to let the browser skip download entirely when glyphs aren't on the page:**
```css
@font-face {
  font-family: "Inter";
  src: url("/fonts/inter-latin.woff2") format("woff2");
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
                 U+02DA, U+02DC, U+2018-2019, U+201C-201D, U+2022,
                 U+2026, U+2212, U+2215;  /* Latin + common punctuation */
  font-display: swap;            /* kills FOIT, but CLS if metrics differ */
  size-adjust: 107%;            /* match fallback metrics to web font */
  ascent-override: 90%;
  descent-override: 22%;
}
```
- `unicode-range` is not just documentation — browsers skip the network request entirely if none of the declared codepoints appear in the document. Always include it.
- `swap` causes a flash + CLS unless fallback metrics match — use `size-adjust` / `ascent-override` / `descent-override`.
- For above-the-fold fonts where FOUT is unacceptable, prefer `font-display: optional`.
- **Preload (`<link rel="preload" as="font" crossorigin>`)** the one or two fonts on the critical path. Don't preload all fonts — each preload is a forced network request at highest priority regardless of whether the page uses that glyph range.
- Preloading *and* using `swap` can backfire: the preload prioritizes the file but `swap` still flashes. Shopify's font preload via Early Hints improved LCP by **500ms** for merchants — but only paired with correct display strategy.

## Image pipelines: sharp, not Squoosh CLI

**Squoosh CLI is dead** — unmaintained since early 2023, broken on Node 18+. Any build referencing `squoosh-cli` will silently fail/error on modern Node. The **Squoosh web app** is still useful for manual side-by-side quality comparison.

Use **sharp** (Node) for automated processing:
```js
import sharp from "sharp";
await sharp("hero.jpg")
  .resize(1600, null, { withoutEnlargement: true })
  .webp({ quality: 80 })            // ~95% browser support — default choice; tune 75–85 per asset
  .toFile("hero.webp");
await sharp("hero.jpg")
  .resize(1600).avif({ quality: 50 }) // ~92% support, smaller, CPU-heavy on mobile; tune 40–60
  .toFile("hero.avif");
```
- **WebP** is the broad default (~95%). **AVIF** (~92%) compresses better but is CPU-intensive to decode on low-end mobile — serve via `<picture>` with WebP fallback, don't force it.
- Resize to the largest rendered size (account for DPR), then let `srcset`/`sizes` pick. See [Imagery, Icons & Assets](../00-foundations/imagery-icons-and-assets.md).

## Storybook: component workshop = test + doc source of truth

Storybook 8 integrates with Vitest for component testing and ships a significantly faster build pipeline than Storybook 7 (primarily via the switch to Vite-native builds and lazy story loading).

- **CSF (Component Story Format) is non-negotiable.** A story file *is* the source of truth for both visual tests and docs — metadata travels with the file across environments. Don't convert stories to other formats.
- Essential addons: **addon-a11y** (axe-core in the panel — catches contrast/ARIA per component), **addon-interactions** (play functions = interaction tests), **addon-docs** (auto docs from CSF + JSDoc).
- A minimal CSF story:
  ```tsx
  import type { Meta, StoryObj } from "@storybook/react";
  import { Button } from "./Button";
  const meta: Meta<typeof Button> = { component: Button, args: { children: "Save" } };
  export default meta;
  export const Primary: StoryObj<typeof Button> = { args: { variant: "primary" } };
  export const Disabled: StoryObj<typeof Button> = { args: { disabled: true } };
  ```
- Use Storybook as the **a11y + visual-regression gate** in CI, not just a gallery.

## Linting & formatting: ESLint + Prettier (the right way)

Two tools, two jobs: **ESLint = correctness**, **Prettier = formatting**. They overlap on stylistic rules and *conflict* unless bridged.

- **Flat Config (`eslint.config.js`) is the ESLint v9+ default** (released April 2024); legacy `.eslintrc` is deprecated but still supported via `@eslint/eslintrc` compatibility shim.
- **`eslint-config-prettier`** turns off all ESLint rules that Prettier owns (formatting-only). This is the conflict bridge — without it, both tools fight on every save.
- **`eslint-plugin-prettier`** runs Prettier as an ESLint rule, reporting diffs as lint errors. `eslint-plugin-prettier/recommended` includes `eslint-config-prettier` automatically.
- Put `prettierRecommended` **last** in the config array so it wins over any other formatting rules.

```js
// eslint.config.js (flat config)
import js from "@eslint/js";
import prettierRecommended from "eslint-plugin-prettier/recommended";

export default [
  js.configs.recommended,
  // ...your rules, framework plugins...
  prettierRecommended,   // MUST be last: disables conflicting rules + runs Prettier as a rule
];
```

**Enforce at commit** with Husky + lint-staged so unformatted/error code can't land:
```jsonc
// package.json
"lint-staged": {
  "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"],
  "*.{css,md,json}": ["prettier --write"]
}
```

## Performance auditing: Lighthouse vs PageSpeed Insights vs field data

**Lighthouse measures only lab (synthetic) data** in a throttled emulated environment. **PageSpeed Insights** wraps Lighthouse *and* adds real-user **CrUX field data** over a 28-day rolling window. They answer different questions.

- **Lighthouse cannot measure INP** (it isn't an interaction harness) — it reports **TBT as a proxy.** A page can score TBT 0ms in Lighthouse and still have terrible real-user INP. **Always check PSI field data** before declaring a page fast.
- **Mobile Lighthouse throttles to a ~2014-era mid-range phone on slow 3G/4G.** Scores 20–30 points below desktop are expected, not a crisis.
- **Score variance ±5 points is normal** across identical runs; 10–15 point gaps between local Lighthouse and PSI are normal. **Act only on changes >10 points and directional trends.**
- **Third-party scripts dominate.** Analytics/chat/pixel tags can cost 20–40 points regardless of your own work. Quantify each before adding it.

**Site-wide tooling:**
- **Unlighthouse** — CLI that audits *every* page of a site, not one URL. Run before launch.
- **DebugBear** — hosted continuous Lighthouse monitoring with historical trending (regression alerts).

See [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) for the metric playbook.

## Browser DevTools for design QA

**Chrome DevTools:**
- **Rendering tab** — emulate `prefers-reduced-motion`, `prefers-color-scheme`, `prefers-reduced-transparency`, forced-colors; paint-flashing; FPS meter. Essential for verifying media-query design states.
- **Elements → CSS layers panel** + **specificity hover tooltip** — debug `@layer` order and which rule wins.
- **HD color spaces** — pick/verify `oklch()`/`display-p3`; the color picker shows out-of-sRGB-gamut warnings.
- **Animations panel** + cubic-bezier editor + **scroll-driven animations debugger** — tune timing visually.
- **Coverage tab** — find unused CSS/JS shipped to the page.

**Firefox DevTools** — use for what Chrome does worse:
- **Grid Inspector** is superior (overlays line numbers, area names, gaps).
- **ARIA / Accessibility inspector** exposes the computed a11y tree cleanly.
- Stricter W3C spec adherence → good for catching edge-case spec violations Chrome tolerates.

**Extensions:** **CSS Peeper** (inspect any site's styles/assets), **axe DevTools** (automated a11y incl. contrast), **Stark** (Figma/XD inline contrast + colorblind simulation at design time).

## Color & contrast checking

WCAG 2.2 thresholds (unchanged from 2.1). Relative-luminance ratio = `(L1 + 0.05) / (L2 + 0.05)`.

| Context | AA | AAA |
|---|---|---|
| Normal text | 4.5:1 | 7:1 |
| Large text (≥18pt / ≥24px, or ≥14pt / ≥18.66px bold) | 3:1 | 4.5:1 |
| UI components & graphical objects (icons, borders, focus rings) | 3:1 | — |

- **WebAIM Contrast Checker** — hex input, alpha/opacity support, lightness slider to nudge to passing.
- **Stark** (design-time, in Figma) and **axe DevTools** (build-time, in browser) — catch failures before and after handoff.
- Contrast is the **#1 accessibility violation** — 83.6% of sites fail it (WebAIM Million 2024). ADA digital accessibility lawsuits have grown year-over-year since 2018 (Usablenet reports ~4,600+ in 2023). Treat it as a launch blocker, not a nicety. See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) and [Color Theory & Systems](../00-foundations/color-theory-and-systems.md).

## Concrete specs & numbers
- **Contrast:** 4.5:1 normal text · 3:1 large text (≥18pt / ≥14pt bold) · 3:1 UI/graphics · AAA 7:1 / 4.5:1. Formula `(L1+0.05)/(L2+0.05)`.
- **Core Web Vitals targets:** LCP <2.5s · CLS <0.1 · INP <200ms good / 200–500ms needs-improvement / >500ms poor (INP replaced FID March 2024).
- **Vite:** dev cold start ~200ms (Webpack ~2.5s, ≈15×) · esbuild pre-bundle 10–20× vs Babel · HMR near-instant (Webpack 500–1600ms). Rolldown production bundler: experimental as of mid-2026.
- **Fonts (top-1M, 2024):** 81.5% use a web font · avg 238 KB/page · 63.6% load before FCP · 25% >75 KB before FCP. Median 3 fonts / 95 KB / 101 glyphs (smallest font 248 glyphs). p95: 9 fonts / 542 KB.
- **Subsetting wins:** Montserrat 64.6 KB → 15 KB (−76%); Tom Hazledine −63%; Kia 3 MB → 30 KB; gwfh 180 KB → 18 KB.
- **WOFF2:** ~30% smaller than older formats, ~97% global support.
- **Image formats:** AVIF ~92% support (CPU-heavy mobile), WebP ~95% (broad default).
- **Lighthouse:** ±5 pt run-to-run variance normal; 10–15 pt local-vs-PSI gap normal; mobile 20–30 pt below desktop expected; third-party scripts cost 20–40 pt.
- **Business stakes (field):** Vodafone LCP −31% → +8% sales; Agrofy LCP −70% → −76% load abandonment; Shopify Early Hints font preload → LCP −500ms.
- **Storybook 8:** Vitest integration, Vite-native builds, lazy story loading. Faster than Storybook 7 in practice; no single canonical speedup number applies universally.
- **Cache history:** cross-site Google Fonts caching removed Chrome 86 (2020); Google Fonts CDN ruled GDPR-violating in Germany (Jan 2022).

## Tools & libraries
- **Figma** — design + Auto Layout + Variables + Dev Mode (paid org seat) + prototype mode (Smart Animate) + MCP server for AI assistants. Use for all UI design; NOT a token source of truth (Git is).
- **Framer** — high-fidelity web-rendered prototypes + conditional logic. Use when Figma prototype graph is too limiting; exports React (reference only).
- **ProtoPie** — multi-screen conditional/sensor prototyping. Use for mobile-native or hardware interaction prototypes; overkill for typical web.
- **Tokens Studio** (Figma plugin) — bidirectional DTCG sync to Git. Use for any system with a code design system; pair with Style Dictionary.
- **Style Dictionary** — one DTCG source → all platform outputs. The token transform engine.
- **Penpot** — open-source, self-hostable, native W3C tokens. Use when avoiding Figma lock-in / needing self-host.
- **Vite** — dev server + Rolldown build. Default for SPA/component dev. NOT for cases needing a mature legacy Webpack-only plugin you can't replace.
- **Fontsource** — self-hosted fonts via npm. Use over Google Fonts CDN always.
- **glyphhanger / pyftsubset / gwfh** — font subsetting. Use whenever you ship a web font.
- **sharp** — Node image processing. Use in every build pipeline. Squoosh **web app** for manual quality comparison; never `squoosh-cli`.
- **Storybook 8** — component workshop + a11y/interaction/visual tests. Use for any reusable component library.
- **ESLint (Flat Config) + Prettier + eslint-config-prettier + Husky + lint-staged** — correctness + formatting gate. Use on every project.
- **Lighthouse** (lab) + **PageSpeed Insights** (field/CrUX) — always pair. **Unlighthouse** for whole-site; **DebugBear** for continuous trending.
- **WebAIM Contrast Checker / axe DevTools / Stark** — contrast at build and design time.
- **Chrome DevTools** (Rendering tab, layers, oklch picker, animation/scroll debuggers, Coverage) + **Firefox DevTools** (Grid Inspector, ARIA tree) + **CSS Peeper**.

## Do / Don't

**Do**
- Build the primitive→semantic→mode variable architecture before designing screens.
- Keep tokens in Git as DTCG JSON; treat Figma and code as consumers.
- Self-host + subset every web font; ship WOFF2; match fallback metrics.
- Pipe all images through sharp; serve WebP broadly, AVIF via `<picture>` fallback.
- Pair Lighthouse (lab) with PSI (field) and act on >10-point/directional changes.
- Treat contrast failures as launch blockers; verify at design time (Stark) and build time (axe).
- Put `eslint-plugin-prettier/recommended` last; enforce with lint-staged.
- Use Dev Mode's "Ready for Dev" + change-compare; read CSS values for reference only.

**Don't**
- Don't paste Figma's generated CSS into production — context-free absolute px ignoring your tokens.
- Don't accept a handoff built with absolute positioning instead of Auto Layout.
- Don't use Google Fonts CDN (no cross-site cache since Chrome 86; GDPR risk in EU).
- Don't reference `squoosh-cli` in a build — dead, broken on Node 18+.
- Don't ship a font's full glyph set when you render Latin/ASCII.
- Don't leave DevTools "Disable cache" on during Vite dev; don't import via barrel files in Vite projects (forces the full re-export tree to load on startup, hurting HMR).
- Don't trust a Lighthouse 0ms TBT as proof of good INP — check CrUX.
- Don't run ESLint + Prettier without `eslint-config-prettier` (rule collisions).
- Don't migrate Storybook stories off CSF.

## Common pitfalls & how to avoid them
- **Dev Mode pricing trap.** Individual Professional plan ≠ Dev Mode on org files you don't own. → Get added as a paid seat in the designer's org.
- **Applying Vite rules to a Next.js project.** Next 15+ uses Turbopack for dev (not Vite) — Vite-specific advice (barrel files, "Disable cache" quirk, vite.config profiling) does not apply. → Check your framework before applying Vite optimizations.
- **Copy-pasting Figma CSS.** Output is absolute, context-free, token-blind. → Reference distances/colors only; write CSS against your tokens.
- **No Auto Layout in handoff.** Absolute positions don't reflect reflow; measurements are meaningless. → Require Auto Layout; otherwise measurements are approximate.
- **Squoosh CLI in pipeline.** Silently fails on Node 18+. → Replace with sharp.
- **"Users have the font cached."** False since Chrome 86 (2020). Plus EU GDPR exposure. → Self-host.
- **`font-display: swap` CLS.** Mismatched fallback metrics shift layout. → `size-adjust`/`ascent-override`/`descent-override`, or `font-display: optional` above the fold.
- **Preload + swap.** Preload prioritizes the file but swap still flashes. → Match metrics or use `optional`; don't preload everything.
- **"Disable cache" left on in Vite.** Refetches pre-bundled deps every reload. → Turn it off during dev.
- **Barrel imports.** Vite loads the whole `index.js` re-export tree on start. → Import leaf modules directly.
- **Mobile Lighthouse panic.** 20–30 pt below desktop is expected throttling. → Compare like-for-like; trust trends.
- **Third-party script tax.** Pixels/chat/analytics cost 20–40 pt. → Audit each addition's cost.
- **No INP measurement.** Lighthouse can't measure it. → Use PSI CrUX field data.
- **ESLint/Prettier wars.** Rules collide. → `eslint-config-prettier`, prettier recommended last.

## What practitioners complain about

(Consensus from r/webdev, r/design, and designer/developer community threads — upvote counts are approximate and change over time.)

- **Figma Dev Mode pricing is "a bizarre system."** The loudest, most-upvoted complaint: individual Professional holders cannot use Dev Mode on org files they don't own — they must be added as paid seats in the designer's org.
- **"Copying and pasting CSS from Figma makes no sense. It's out of context. It's the opposite of how design systems work."** Senior devs are split: Tailwind users find Dev Mode's class suggestions valuable; teams on strict SCSS/token systems "found no value."
- **The actually-loved Dev Mode features are "Ready for Dev" + change-compare**, not CSS copy-paste — devs consistently cite the status flag and diff as the genuine wins.
- **Lighthouse is treated as directional, not gospel.** Recurring practitioner sentiment: "Optimization tasks are somewhere deep in the backlog"; "marketing team tosses a few dozen tracking scripts into it, tanking the metrics"; "A score of 40 is good enough for Amazon." Strong consensus that third-party scripts dominate the score and a perfect number isn't the goal.

## Senior-level checklist
- [ ] Prototype tool selected based on question fidelity (Figma prototype / Framer / ProtoPie / Storybook spike); prototype budgeted ≤2–4h (Figma) or ≤1–2d (Framer).
- [ ] Figma variable architecture exists in three tiers (primitives → semantic → modes) before screen design.
- [ ] Tokens live in Git as DTCG JSON; Tokens Studio + Style Dictionary sync verified bidirectional.
- [ ] Handoff frames use Auto Layout (or Grid); no critical layout is absolutely positioned.
- [ ] Dev seats arranged so devs have real Dev Mode access (or workflow accepts Viewer-only inspect).
- [ ] No Figma-generated CSS pasted verbatim; production CSS references tokens.
- [ ] Vite project: no barrel imports, "Disable cache" off, profiled if cold start >1s.
- [ ] All web fonts self-hosted, WOFF2, subset to rendered charset; `unicode-range` descriptor set; fallback metrics matched (`size-adjust`/`ascent-override`/`descent-override`) — no CLS.
- [ ] Critical-path fonts have `<link rel="preload" as="font" crossorigin>` (LCP font only; not all fonts).
- [ ] Image pipeline uses sharp; WebP default + AVIF `<picture>` fallback; no `squoosh-cli`.
- [ ] Storybook with addon-a11y + interactions; stories in CSF; runs in CI.
- [ ] ESLint Flat Config + Prettier + `eslint-config-prettier` (last); Husky + lint-staged active.
- [ ] Lighthouse (lab) AND PSI (field/CrUX) checked; INP read from field data, not TBT.
- [ ] Third-party script cost quantified per script.
- [ ] Contrast verified at design (Stark) and build (axe) time: 4.5:1 / 3:1 thresholds met as launch blocker.
- [ ] Design-state QA done in Rendering tab (reduced-motion, dark, forced-colors).

## Sources
- W3C Design Tokens Community Group format (DTCG): https://www.designtokens.org/
- Figma Dev Mode docs: https://help.figma.com/hc/en-us/articles/15023124644247
- Figma Variables docs: https://help.figma.com/hc/en-us/articles/15339657135383
- Vite: https://vite.dev/
- Rolldown (Vite's Rust bundler): https://rolldown.rs/
- Next.js Turbopack: https://nextjs.org/docs/architecture/turbopack
- Framer: https://www.framer.com/
- ProtoPie: https://www.protopie.io/
- Style Dictionary: https://styledictionary.com/
- Tokens Studio: https://tokens.studio/
- Penpot: https://penpot.app/
- Fontsource: https://fontsource.org/
- Google Webfonts Helper: https://gwfh.mranftl.com/
- fonttools (pyftsubset): https://fonttools.readthedocs.io/
- glyphhanger: https://github.com/zachleat/glyphhanger
- sharp: https://sharp.pixelplumbing.com/
- Squoosh (web app): https://squoosh.app/
- Storybook: https://storybook.js.org/
- ESLint Flat Config: https://eslint.org/docs/latest/use/configure/configuration-files
- Prettier (eslint-config-prettier): https://github.com/prettier/eslint-config-prettier
- Lighthouse: https://developer.chrome.com/docs/lighthouse/overview/
- PageSpeed Insights: https://pagespeed.web.dev/
- Unlighthouse: https://unlighthouse.dev/
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- WebAIM Million 2024: https://webaim.org/projects/million/
- WCAG 2.2 contrast (1.4.3): https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html
- INP (web.dev): https://web.dev/articles/inp
- axe DevTools: https://www.deque.com/axe/devtools/

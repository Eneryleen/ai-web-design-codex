# A Taxonomy of Web Design Styles

> A reference catalog of the visual styles a senior web designer chooses between: what each one is, when it fits, how to execute it in real CSS/HTML, what it costs in usability and accessibility, and which industries it serves. Use it to pick a style deliberately — because it reinforces the content and audience — not because it is trending.

**Apply when:** picking or justifying a visual direction before wireframing/build · **Related:** [Typography Systems](../00-foundations/typography-systems.md) · [Color Theory & Systems](../00-foundations/color-theory-and-systems.md) · [Trends 2025-2026](trends-2025-2026.md) · [Avoiding Generic 'AI' Aesthetics](avoiding-generic-ai-aesthetics.md)

## TL;DR — rules at a glance
- **Style is a means, not a goal.** Pick the one that reinforces your content and matches audience expectations; reject novelty-for-novelty.
- **Every style has a native industry.** Glassmorphism → SaaS dashboards; claymorphism → kids/gaming; dark luxe → developer tools; Y2K → fashion/music; Swiss → finance/docs/publishing.
- **Form follows function.** Every visual element should exist because it serves communication or affordance — not for decoration. Remove elements that do not serve the user; do not remove elements merely because they look messy by a different style's rules.
- **Processing fluency = trust.** Simpler, faster-to-parse layouts read as more credible. Users consistently self-report preferring clean, uncluttered layouts across UX surveys — but conversion impact is content- and audience-dependent; do not treat "cleaner = always better" as universal.
- **Whitespace is structure, not leftover space.** Start the layout with whitespace; let content live inside it (Swiss method).
- **Jakob's Law has a price tag.** Departing from established affordances (brutalism, OS-window nav) costs measurable usability. Quantify the cost before you commit.
- **Accessibility is a hard gate.** Any style that can't hit WCAG 2.1 AA — 4.5:1 text, 3:1 UI/large-text contrast — needs a fallback/adaptation path. This kills naive neumorphism and unmodified glassmorphism for primary content.
- **2026 technical foundation:** variable fonts + CSS Grid + `clamp()` for responsive type/layout; semantic two-layer color tokens for theming; three-state theme toggle (light/dark/system); CSS `animation-timeline: scroll()` (Baseline 2024) for scroll-driven effects without JS.
- **Never invert light mode to make dark mode.** Use semantic tokens (base + role). Pure `#000` *backgrounds* break inner-shadow styles (neumorphism, claymorphism); `#000` is fine as a shadow *color* (neobrutalism).
- **Glass and clay don't fully work in pure CSS.** Glass needs `@supports` fallback; clay needs SVG or a generator (claymorphism.com).
- **Bento cells must be self-contained stories** unified by consistent spacing, border weight, and color.
- **Neobrutalism is calculated chaos, not "ugly."** High contrast, semantic HTML, thick borders, offset hard shadows, system-weight fonts.

---

## How to choose a style (decision frame)

Run these four filters before committing. A style must pass all four.

1. **Audience expectation (Jakob's Law).** Where does this audience spend its time, and what affordances do they expect? Developers tolerate dense dark UI; a healthcare patient base does not tolerate experimental nav.
2. **Content fit.** Does the style amplify the dominant content type? Type-led suits editorial/publishing; bento suits feature roundups; minimalism suits a single hero product.
3. **Trust sensitivity.** Finance, healthcare, legal, B2B enterprise are trust-sensitive → bias to Swiss/minimal/flat. Entertainment, fashion, creator tools tolerate brutalism/Y2K/maximalism.
4. **Accessibility ceiling.** Can the style hit WCAG 2.1 AA for primary content without contortion? If not, it's decoration-only or needs an adaptation path.

**Quick industry map:**

| Industry | Default-fit styles | Avoid |
|---|---|---|
| Finance / fintech | Swiss, minimal, flat | brutalism, Y2K, neumorphism, claymorphism |
| Healthcare | minimal, flat, soft editorial | brutalism, Y2K, maximalism, heavy skeuomorphism |
| Developer tools | dark luxe, Swiss/minimal | claymorphism, Y2K, skeuomorphism |
| SaaS / B2B | minimal, glass (dashboards), bento, dark luxe | maximalism, Y2K |
| E-commerce (mass) | minimal, flat/material, editorial | brutalism, neumorphism |
| Fashion / music / events | Y2K, maximalism, editorial, aurora | flat-corporate, plain Swiss |
| Publishing / blogs / docs | Swiss, editorial, typographic-led | glass, neumorphism, claymorphism |
| Agencies / studios | editorial, typographic-led, brutalism, aurora | plain flat-corporate, claymorphism |
| Kids / casual gaming | claymorphism, maximalism | dark luxe, Swiss |
| Portfolios / agencies | editorial, brutalism, aurora, maximalism | plain flat-corporate |

---

## Minimalism

**What:** Reduction to essentials. Generous whitespace, one focal point per view, restrained palette (1 accent), few type sizes, no decorative elements.

**When to use:** Single hero product, premium/luxury positioning, trust-sensitive content, anything where focus and credibility matter more than personality.

**How to execute:**
- Start from whitespace. One primary action per screen; everything else is secondary or removed.
- Palette: 1 neutral background, 1 ink color, 1 accent. Type: 2–3 sizes max.
- Use scale and spacing (not borders/shadows) to create hierarchy. See [Visual Hierarchy](../00-foundations/visual-hierarchy.md).

**Pros:** Fast to process (high fluency → trust), ages well, cheap to maintain, accessible by default.
**Cons:** Low differentiation if generic; can read as "empty" or "unfinished" without strong content/photography; easy to do badly (sparse ≠ minimal).
**Fit:** Apple, premium consumer, luxury, high-end SaaS. Canonical example: **Apple** (near-pure whitespace, product as hero, 12-col grid).

---

## Swiss / International Typographic Style

**What:** The grid-driven, objective design system: strict alignment, mathematical type scale, asymmetric balance, grotesque sans-serif, content over ornament. The ancestor of "minimalism" but with a rigorous grid backbone.

**When to use:** Information-dense but credibility-critical content — financial products, documentation, publishing, dashboards that must feel orderly.

**How to execute:**
- 12-column CSS Grid; everything snaps to columns and a baseline rhythm.
- Type scale on a ratio: **1.25× (major third)** or **1.333× (perfect fourth)**. Body 16–18px, line-height 1.5–1.6, **measure 60–75 characters** per line.
- Grotesque sans: **Inter** (variable, screen-optimized Helvetica heir) or **Neue Haas Grotesk / Helvetica Neue** where licensing budget exists.

```css
:root {
  --ratio: 1.25;
  --step-0: 1rem;                              /* body 16px */
  --step-1: calc(var(--step-0) * var(--ratio));
  --step-2: calc(var(--step-1) * var(--ratio));
}
.container { display: grid; grid-template-columns: repeat(12, 1fr); gap: 24px; }
.lead { max-width: 68ch; line-height: 1.55; }  /* enforce measure */
```

**Pros:** Maximum clarity and order, scales to complex content, timeless, highly accessible.
**Cons:** Can feel cold/corporate without warmth from imagery or voice; demands discipline (one broken alignment is visible).
**Fit:** **Stripe** (Swiss clarity on financial complexity), **Medium**, **Notion** (block grid), **Kinfolk** (editorial-Swiss). See [Layout, Grids & Spacing](../00-foundations/layout-grids-and-spacing.md).

---

## Editorial / Typographic-Led

**What:** Type is the primary design element, not supporting copy. Oversized display type (120px+) as hero, strict grid, expressive pairings, variable-weight transitions on scroll.

**When to use:** Publishing, agencies, portfolios, brand stories, anything where words carry the message and you want craft to show.

**How to execute:**
- Display hero at fluid scale; let type dominate the first viewport.
- Pair a display face with a workhorse text face; use variable-font weight axes for scroll-driven emphasis.
- Strict grid alignment keeps the bigness from becoming chaos.

```css
.hero-title {
  font-size: clamp(3rem, 10vw, 8rem);   /* ~48px → 128px */
  font-variation-settings: "wght" 700;
  line-height: 0.95; letter-spacing: -0.02em;
}

/* Native CSS scroll-driven weight transition (Baseline 2024: Chrome 115+, Firefox 110+) */
@supports (animation-timeline: scroll()) {
  @keyframes wght-shift { to { font-variation-settings: "wght" 900; } }
  .hero-title {
    animation: wght-shift linear;
    animation-timeline: scroll();
    animation-range: entry 0% cover 40%;
  }
}
```
**Scroll animation:** CSS `animation-timeline: scroll()` is Baseline 2024 and should be the first choice for scroll-driven weight/position transitions in 2026. GSAP ScrollTrigger / Lenis remain the fallback for complex multi-element choreography or browsers below Baseline 2024. Always wrap in `@supports` and honor `prefers-reduced-motion`. See [Scroll-Driven Experiences](../03-site-types/scroll-driven-experiences.md) and [Typography Systems](../00-foundations/typography-systems.md).

**Pros:** Strong personality and craft signal; cheap assets (type, not imagery); excellent on text-first content.
**Cons:** Demands real typographic skill; big type can wreck mobile measure/wrapping; risk of style over readability.
**Fit:** Awwwards "Typography-Heavy" collection, **Kinfolk**, agency/portfolio sites.

---

## Brutalism / Neo-Brutalism

**What:** Raw, high-contrast, anti-polish. **Neobrutalism** is the productized web variant: thick black borders, hard offset box-shadows, flat pastel/primary fills, system-weight fonts, exposed structure. **It is calculated chaos, not "ugly design."**

**When to use:** Creator tools, indie products, dev tooling, brands whose ethos is bold/independent and whose audience reads non-standard UI as intentional.

**How to execute:**
- Borders **2–4px solid `#000`**. Hard shadow `box-shadow: 4px 4px 0 #000` (offset ≈ border weight → "lifted card").
- Fills: pastel or saturated primary. Keep semantic HTML — accessibility comes from structure, not from softening the look.
- On interaction, shift the element toward the shadow to simulate a press.

```css
.nb-card {
  border: 3px solid #000;
  box-shadow: 6px 6px 0 #000;
  background: #ffd84d;
  transition: transform .08s, box-shadow .08s;
}
.nb-card:active { transform: translate(3px,3px); box-shadow: 3px 3px 0 #000; }
```

**Pros:** Instantly distinctive, cheap (no imagery), high contrast → naturally accessible text.
**Cons:** Some users read it as a prototype/unfinished build; "bold for boldness' sake" off-brand creates confusion; departs from affordances (Jakob's Law cost).
**Fit:** **Gumroad** (canonical), **gitingest.com** (sparked the viral "what style is this?" thread), 37signals/HEY (Swiss-brutalist hybrid with personality). **Not for** luxury, emotional lifestyle, healthcare, finance. Use **neobrutalism.dev** (React/Tailwind components).

---

## Glassmorphism

**What:** Frosted-glass surfaces: translucency + background blur + a thin light border, layered over a colorful/gradient background to suggest depth.

**When to use:** SaaS dashboards, app marketing, overlay panels/cards over a vivid backdrop — where surfaces float above content.

**How to execute:**

```css
.glass {
  background: rgba(255,255,255,0.18);          /* 0.1–0.25 */
  backdrop-filter: blur(24px);                 /* ≥24px; Apple Liquid Glass shipped ~8px and was widely criticised */
  border: 1px solid rgba(255,255,255,0.20);
  border-radius: 16px;
}
@supports not (backdrop-filter: blur(1px)) {   /* ~4% of 2026 traffic; mainly older/budget browsers */
  .glass { background: rgba(255,255,255,0.82); }/* solid fallback, no blur */
}
```
- **Harden foreground text** (solid, high-contrast color; never rely on the glass to provide contrast). Never put primary CTAs or long-form text on unmodified glass.
- **Apple Liquid Glass (macOS/iOS 26):** Practitioners flagged the system default at ~8px blur as too weak for legibility. Use ≥24px. Provide a "Reduce Transparency" / `prefers-reduced-transparency` path — Apple sets the precedent and users expect it.
- **2026 status:** `backdrop-filter` is Baseline 2022 (Chrome 76+, Firefox 103+, Safari 9+). Remaining non-support is negligible (<4%); `@supports` fallback is still good practice for performance budgets on low-end Android.

**Pros:** Premium, modern, conveys depth/hierarchy via layering.
**Cons:** Text frequently drops below **3:1**; `backdrop-filter` is GPU-heavy and **tanks budget Android below 30fps**; overuse flattens hierarchy.
**Fit:** SaaS dashboards, app landings (Framer/Webflow templates). Provide a "reduce transparency" path like Apple's accessibility toggle. See [Accessibility](../01-ux-fundamentals/accessibility-wcag.md).

---

## Neumorphism (soft UI)

**What:** Elements appear extruded from or pressed into a single monochrome surface using paired light/dark shadows. **An antipattern for interactive controls in production.** Decorative/illustrative use in non-interactive contexts is acceptable with explicit contrast mitigation.

**When to use:** Rarely. Decorative/illustrative components, non-critical toggles in a controlled palette — only with explicit high-contrast alternatives.

**Why it fails:** The two-shadow system (light `#fff` ~30%, dark same-hue ~20%) on a flat background yields **near-zero contrast** between control and surface → fails WCAG; buttons/inputs become invisible to low-vision users. Viral on Dribbble in 2020, abandoned by major tech in production by 2021.

```css
/* Illustrative only — do NOT ship as the sole affordance for interactive controls */
.neu {
  background: #e0e5ec; border-radius: 16px;
  box-shadow: 8px 8px 16px #a3b1c6, -8px -8px 16px #ffffff;
}
```
**Pros:** Soft, tactile, calm aesthetic.
**Cons:** Fails contrast; pure `#000` or dark-mode backgrounds destroy the inner shadows (the shadow system requires a midtone base, e.g. `#e0e5ec`); no clear affordance state; no viable dark-mode version.
**Dark mode:** Neumorphism has no recoverable dark-mode form — the shadow math breaks on dark surfaces. If you must support dark mode, this style cannot be used for interactive controls.
**Fit:** Essentially none at WCAG AA for interactive controls. If insisted upon, pair every control with a real border/contrast indicator and provide a fully separate high-contrast theme.

---

## Skeuomorphism

**What:** Interfaces mimicking real-world materials and objects — leather, paper, realistic gradients, drop shadows, physical metaphors. Includes OS-metaphor UIs (desktop windows, folder icons) and any surface that tries to look like a physical thing.

**When to use:** Nostalgia-driven brand moments, tutorials/onboarding where a real-world metaphor genuinely aids comprehension (e.g., an audio mixer plugin), playful/creative tools, "OS window" concept sites.

**2026 signal — Apple Liquid Glass:** Apple's macOS/iOS 26 (WWDC25) revived layered physical-material surfaces as a system paradigm. This gives skeuomorphism renewed legitimacy in consumer-product UI — but it is not a green light for decorative material texture everywhere. Apple's execution constrains it to interaction surfaces (dock, control panels), not to typography or reading surfaces. If you reference this look, see the Glassmorphism section for the blur legibility requirements.

**How to execute:** Use real textures/illustration sparingly to teach an affordance, not to decorate every surface. Keep interactive affordances obvious and standard underneath the skin. Never rely on texture alone to communicate state (hover, active, disabled) — always pair with a color or shape change.

**Pros:** Intuitive metaphors lower the learning curve for novel interactions; strong emotional/nostalgic pull.
**Cons:** Cognitive overload from richly detailed surfaces; slower task completion; heavy assets; material detail can date to a specific era within 2–3 years.
**Fit:** **PostHog** OS-window homepage (split reception: "everything on screen, no 20MB hero" vs "frustrating to navigate"; panel hidden on mobile = responsive failure). Treat experimental OS-style nav as high-risk (Jakob's Law). Consumer apps leveraging Apple Liquid Glass vocabulary are a valid 2026 use case if executed with the blur/contrast discipline described under Glassmorphism.

---

## Flat / Material

**What:** **Flat** removes depth cues entirely (solid fills, no gradients/shadows). **Material Design** is flat-plus-physics: layered surfaces, elevation via consistent shadows, motion that obeys rules.

**When to use:** Product UIs, Android-first apps, large design systems needing consistency at scale, mass-market e-commerce.

**How to execute:**
- Flat: solid fills, clear typographic hierarchy, color/scale for emphasis (not shadows).
- Material: use an **elevation scale** (consistent shadow tokens by z-level), state layers for hover/press, and a tonal/role-based color system.

**Pros:** Fast, scalable, accessible, framework-supported, predictable.
**Cons:** Can look generic/templated; flat-only can hurt affordance clarity (which thing is clickable?); "default AI look" risk — see [Avoiding Generic 'AI' Aesthetics](avoiding-generic-ai-aesthetics.md).
**Fit:** Google products, Android, broad SaaS and e-commerce, internal tools, [CRM & Admin Dashboards](../03-site-types/crm-admin-dashboards.md).

---

## Maximalism

**What:** Deliberate abundance — saturated color, layered texture, mixed type, dense composition. The anti-minimalism, controlled by an underlying system.

**When to use:** Fashion, music, events, creative agencies, campaign pages where personality and memorability beat conversion efficiency.

**How to execute:**
- Mix **2–3 typeface families** at varying weights (classic pairing: display serif × wide grotesque).
- Anchor chaos with one governing grid and a constrained-but-vivid palette so it reads intentional, not random.
- Keep body text on calm, high-contrast surfaces — never on the busiest texture.

**Pros:** Memorable, expressive, strong brand differentiation.
**Cons:** Easy to oversaturate; obscure display fonts wreck readability; cognitive overload kills conversion on transactional pages.
**Fit:** Campaign/brand microsites, [Marketing & Brand Sites](../03-site-types/marketing-brand-sites.md). **Not for** transactional flows, finance, healthcare.

---

## Bento Grids

**What:** Modular, varied-size cells (named after Japanese bento boxes) — each cell a self-contained story, unified by consistent spacing, border weight, and color. Popularized by Apple keynotes.

**When to use:** Feature roundups, dashboards, product overviews, "everything we do" sections where heterogeneous content needs equal-weight modular presentation.

**How to execute:**
- CSS Grid with fixed 2–4 column tracks (or `auto-fill`); cells use `aspect-ratio` to hold proportions. Common cells: **1×1** (small stat), **2×1** (feature card), **2×2** (hero feature).
- **Each cell must stand alone** while staying visually connected through one spacing/border/color system.

```css
.bento {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}
.bento > * { border-radius: 20px; aspect-ratio: 1 / 1; }
.feat-2x1 { grid-column: span 2; aspect-ratio: 2 / 1; }
.hero-2x2 { grid-column: span 2; grid-row: span 2; aspect-ratio: 1 / 1; }
```

**Pros:** Scannable, flexible, responsive (cells reflow/resize), trendy-but-functional.
**Cons:** Feels template-like if not customized; works best when it matches an already-minimal brand (forcing it onto a loud brand backfires).
**Fit:** **macOS Control Center**, **Procreate** site, Apple-style product pages, SaaS feature sections.

---

## Retro / Y2K

**What:** Late-90s/early-00s revival: bright primary/secondary palettes, metallic/chrome accents, multi-chrome gradients, pixel/monospace type, glossy bubbles, sparkle.

**When to use:** Fashion, music, events, youth/creator campaigns, nostalgia-driven launches.

**How to execute:**
- Color recipe: bright primary/secondary + metallic/white accents; multi-chrome gradient across 3–4 adjacent hues (teal → electric blue → violet → magenta).
- **Chrome/metallic text** (strong 2025 trend): `background: linear-gradient(90deg, #c0c0c0, #ffffff, #8a8a8a); -webkit-background-clip: text; -webkit-text-fill-color: transparent;` — combine with high font weight (variable font 800–900) and tight letter-spacing. Pair with `filter: drop-shadow(0 0 8px rgba(255,255,255,0.6))` for glow.
- Always pair the loud surfaces with **high-contrast neutral cards** that carry the actual text.

**Pros:** Energetic, on-trend for youth markets, highly memorable for campaigns.
**Cons:** Reads as "a joke" in trust contexts; neon gradients + mono fonts signal "not serious"; short shelf-life.
**Fit:** Campaign/event/fashion/music pages. **Fails immediately** for SaaS, healthcare, legal, B2B enterprise.

---

## Dark Luxe

**What:** Premium dark UI: deep near-black base (not pure `#000` for surfaces), one electric accent at high saturation, near-white high-contrast type, restrained micro-animation, editorial information density. The UI should feel like a focused instrument, not a marketing page.

**When to use:** Developer tools, premium SaaS, technical products whose audience values speed and sophistication over marketing fluff.

**How to execute:**
- Base near-black (not pure `#000` for surfaces — use it carefully); single high-energy accent; sharp white type.
- Production palettes: **Linear `#0f0f0f` + electric violet**; **Vercel `#000000` + high-contrast white**; **Raycast warm charcoal `#1c1c1e` + orange**.
- Use a **semantic two-layer token system** (base colors + semantic roles); never invert light mode.

```css
:root[data-theme="dark"] {
  --bg-base: #0f0f0f;
  --surface-1: #181818;        /* elevated; not pure black */
  --text-strong: #fafafa;
  --accent: #7c5cff;           /* single electric accent */
}
```

**Pros:** Sophisticated, focused, low eye-strain in dev contexts, strong brand signal.
**Cons:** Pure-black pitfalls (washed midtones, invisible inner shadows); harder to hit contrast on dim accents; not universally preferred — offer a light theme.
**Fit:** **Linear**, **Vercel**, **Raycast**. See [Color Theory & Systems](../00-foundations/color-theory-and-systems.md) for dark-mode token design.

---

## Claymorphism

**What:** Soft, inflated, 3D "clay" shapes with rounded forms and dual inner shadows + an outer depth shadow. Friendly and tactile.

**When to use:** Kids' products, casual gaming, playful onboarding/illustration, light consumer apps.

**How to execute:**
- Dual inner shadows: **lighter inner top-left, darker inner bottom-right**, plus an **outer depth shadow**.
- Border-radius **well beyond 50%** of element dimension; the organic bulge comes from **SVG path manipulation** (drag side-midpoint handles outward) — **pure CSS is unreliable** for the inflated look. Use **claymorphism.com** to generate shadows.

```css
.clay {
  border-radius: 36px;
  background: #c8b6ff;
  box-shadow:
    inset 6px 6px 12px rgba(255,255,255,0.55),
    inset -6px -6px 12px rgba(90,60,160,0.35),
    0 18px 40px rgba(90,60,160,0.30);          /* outer depth */
}
```

**Pros:** Friendly, approachable, distinctive in consumer/playful contexts.
**Cons:** Bright-pastel + heavy inflation reads as a children's app; pure CSS won't render true clay shapes; saturation must be calibrated to audience age.
**Dark mode:** Reduce background lightness but keep inner-shadow opacity ratios; test that the light inner shadow remains perceptibly lighter than the background — on `#1a1a2e` base, the `rgba(255,255,255,0.55)` value from the light example may need to rise to 0.65+ or switch to an absolute hex.
**Fit:** Kids/gaming/casual apps. **Not for** enterprise, finance, healthcare.

---

## Aurora / Fluid Gradients

**What:** Soft, iridescent, northern-lights gradients layered over a dark/neutral base, contained by clean UI cards.

**When to use:** SaaS/app landings, AI products, hero backdrops needing energy without imagery.

**How to execute:**
- Start with a **dark/neutral base**; layer **2–3 iridescent colors** as `radial-gradient`/`conic-gradient`; add a soft highlight layer.
- **Never place long-form text directly on the gradient** — use UI cards (16–24px radius, subtle inner shadow) to contain content.

```css
.aurora {
  background:
    radial-gradient(60% 60% at 20% 20%, #6ee7ff55, transparent 60%),
    radial-gradient(55% 55% at 80% 30%, #c084fc55, transparent 60%),
    radial-gradient(60% 60% at 50% 90%, #34d39955, transparent 60%),
    #0b0b12;                                   /* dark base */
}
.aurora .card { border-radius: 20px; box-shadow: inset 0 1px 0 rgba(255,255,255,.15); }
```
Animate slowly with GSAP/CSS for life; keep motion gentle and `prefers-reduced-motion`-aware. See [Micro-interactions & Motion](micro-interactions-and-motion.md).

**Pros:** Energetic, modern, cheap (no imagery), pairs well with glass and dark luxe.
**Cons:** Text-contrast trap on the gradient itself; animation cost; by 2025–2026, the teal-purple-dark-base combination has become the *default* AI startup look — use it only if you're willing to differentiate via custom hue selection, unique gradient topology, or a contrasting section treatment.
**Cliché threshold (2026):** "dark base + teal + purple radial blobs + glass card" is the most-copied SaaS landing pattern. To avoid generic: (a) shift your hue range to something off-default (amber → coral, or a monochromatic tight range); (b) use irregular conic-gradient shapes rather than two symmetrical radials; (c) contrast with at least one fully flat / solid-color section.
**Fit:** AI/SaaS landings, app marketing (Framer templates). If your brief explicitly requires differentiation from competitors, treat aurora as a starting point requiring customization, not a finished solution.

---

## Concrete specs & numbers
- **WCAG 2.1 AA contrast:** 4.5:1 normal text · 3:1 large text (≥18pt / 14pt bold) and UI components.
- **CSS scroll-driven animation:** `animation-timeline: scroll()` + `animation-range` — Baseline 2024 (Chrome 115+, Firefox 110+); wrap in `@supports (animation-timeline: scroll())` and honor `prefers-reduced-motion`; GSAP ScrollTrigger as fallback for complex choreography.
- **Swiss type scale:** body 16–18px, line-height 1.5–1.6, measure 60–75ch; heading ratio 1.25× (major third) or 1.333× (perfect fourth).
- **Fluid type with `clamp()`:** `clamp(1.25rem, 2.5vw + 0.5rem, 2rem)` → ~20px @320px to ~32px @1440px, no breakpoints. See [Fluid Type & Spacing](../02-responsive-adaptive/fluid-type-and-spacing.md).
- **Variable font perf:** one variable file ~60–80KB replaces 4–8 static weights (~200–400KB); use `font-display: swap` to avoid FOIT.
- **Glassmorphism CSS:** `background: rgba(255,255,255,0.1–0.25)`; `backdrop-filter: blur(≥24px)` (practitioners say ≥24px for legibility; Apple Liquid Glass shipped ~8px and was criticised); `border: 1px solid rgba(255,255,255,0.2)`. `backdrop-filter` is Baseline 2022 (~96%+ global support in 2026); `@supports` fallback still warranted for performance (solid bg 0.7–0.85 opacity, no blur).
- **Neobrutalism:** borders 2–4px solid `#000`; `box-shadow: 4px 4px 0 #000` (offset = border weight); pastel/primary fills.
- **Neumorphism shadows:** light `#fff` ~30% + dark same-hue ~20% on flat monochrome → near-zero contrast (fails AA).
- **Bento (Apple-style):** CSS Grid, `auto-fill` or fixed 2–4 tracks; cells use `aspect-ratio`; sizes 1×1 / 2×1 / 2×2.
- **Claymorphism:** dual inner shadows (light TL, dark BR) + outer depth shadow; border-radius > 50% of element; organic bulge via SVG, not CSS.
- **Dark luxe palettes:** Linear `#0f0f0f` + violet · Vercel `#000000` + white · Raycast `#1c1c1e` + orange.
- **Y2K gradient:** 3–4 adjacent hues, teal → electric blue → violet → magenta + metallic/white accents.
- **Aurora:** dark/neutral base + 2–3 iridescent radial/conic layers + soft highlight; content cards 16–24px radius + subtle inner shadow.
- **Maximalism type:** mix 2–3 families/weights; common pairing display serif × wide grotesque.
- **Audience signal:** Users consistently prefer clean, uncluttered layouts in self-report surveys; do not cite "84.6%" — this figure circulates without a primary source and should not be used as a reference number.
- **Theme toggle:** three-state (light/dark/system) — Linear, GitHub, Vercel.

## Tools & libraries
- **Tailwind CSS** — utility-first; pairs with strict grids; used by Linear, Vercel starter, most neobrutalist projects.
- **shadcn/ui** — independent component collection by shadcn (not Vercel), built on Radix + Tailwind; defaults are Swiss/minimal. One of the most-starred UI repos on GitHub (80k+ stars by 2025). See [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md).
- **neobrutalism.dev** — React/Tailwind neobrutalist components (used by gitingest.com).
- **claymorphism.com** — shadow generator; use it because pure CSS can't reliably render clay shapes.
- **Framer / Webflow** — built-in templates for glass, aurora, bento, dark-luxe SaaS landings.
- **CSS `animation-timeline: scroll()`** — Baseline 2024 native scroll-driven animation; use first for single-element scroll effects before adding JS. **GSAP (ScrollTrigger)** — for complex multi-element choreography or pre-Baseline-2024 targets. **Lenis** — lightweight smooth scroll. See [Animation Libraries Stack](../04-libraries-and-tools/animation-libraries-stack.md).
- **Inter (variable, v4 2024)** — de-facto Swiss/minimal web font; available on Google Fonts and rsms.me. Note: Inter is a system font on some platforms, so consider local() first in your @font-face stack to skip the network load. **Neue Haas Grotesk / Helvetica Neue** — canonical Swiss where licensing budget exists. **Geist** (Vercel, 2023) — modern Inter alternative, variable, optimized for dark UIs.
- **CSS Grid + custom properties + `backdrop-filter`** — foundational stack; gate `backdrop-filter` behind `@supports`.
- **Figma** — dominant prototyping tool; **Awwwards / Dribbble / Muzli / Behance** — trend discovery (neumorphism peaked on Dribbble 2020, declined 2021).
- **When NOT to reach for tools:** don't pull a heavy animation lib for a Swiss/minimal doc site; don't ship `backdrop-filter` to a budget-Android audience without a measured fallback.

## Do / Don't
**Do**
- Choose a style because it fits audience + content + trust level + accessibility ceiling.
- Harden foreground contrast independently of any translucent/soft surface.
- Provide `@supports` fallbacks for `backdrop-filter` and a "reduce transparency" path.
- Use semantic two-layer tokens for theming; ship a three-state toggle.
- Anchor loud styles (maximalism, Y2K, brutalism) with one governing grid and calm text surfaces.
- Test contrast on the actual rendered surface, including over gradients/images.

**Don't**
- Adopt a style to chase novelty or look "AI-modern."
- Ship neumorphism as the sole affordance for interactive controls.
- Put primary CTAs or long-form text on unmodified glass or directly on a gradient.
- Invert light-mode colors to fake dark mode.
- Force bento/Y2K/brutalism onto a brand whose identity contradicts them.
- Use pure `#000` surfaces for styles that rely on inner shadows.

## Common pitfalls & how to avoid them
- **Neumorphism contrast failure** → near-invisible controls; don't ship it as the affordance — add real borders/contrast or skip it.
- **Glassmorphism legibility + perf** → text under 3:1, sub-30fps on budget Android; harden text, use blur ≥24px (not 8–16px as Apple Liquid Glass shipped), gate behind `@supports`, provide reduce-transparency path, never place on primary CTAs.
- **Neobrutalism misapplication** → reads as prototype/off-brand; only use where bold/independent is on-brand; keep semantic HTML.
- **Maximalism overload** → too many colors/textures/fonts; cap families at 3, govern with a grid, keep body text calm.
- **Naive dark-mode inversion** → washed midtones, dead inner shadows; use semantic tokens, avoid pure black surfaces.
- **Bento rigidity** → template-like; customize, and only use where an already-minimal brand supports it.
- **Claymorphism childishness** → reads as kids' app; calibrate saturation/illustration to audience age; not for enterprise/finance/healthcare.
- **Skeuomorphism clutter** → detailed surfaces slow task completion; use metaphor to teach, not to decorate.
- **Y2K/retro misfit** → signals "not serious"; campaign/fashion/music only, never trust-sensitive verticals. Chrome/metallic text is now viable technique but still restricted to these contexts.
- **Aurora cliché** → dark base + teal + purple radial blobs is the default AI startup look by 2026; differentiate via custom hue range, irregular gradient topology, and at least one flat/solid contrast section.
- **Experimental nav (OS-window)** → immediate drop-off; accepted patterns are accepted for good reasons (Jakob's Law) — avoid hidden panels/draggable windows for customers, and never hide core nav on mobile.

## What real users complain about
- **Apple Liquid Glass** (r/UXDesign, 755 upvotes): "Blur looks roughly 8px, needs to be at least 24px" — frosted system UI flagged as inaccessible; Apple's mitigation is the "Reduce Transparency" toggle.
- **Glassmorphism in the wild:** text routinely below 3:1 on frosted surfaces; on budget Android the GPU-heavy blur drops frame rate below 30fps.
- **Neumorphism:** low-vision users can't find buttons/fields — controls blend into the surface; the style went viral on Dribbble (2020) then was dropped in production by 2021.
- **PostHog OS-window homepage** (r/web_design, 287 upvotes): split — "everything's on screen, no 20MB hero / auto-playing video" vs "frustrating to navigate as a customer"; the OS panel isn't shown on mobile (a responsive failure).
- **Neobrutalism:** some users read it as a prototype or an incomplete build; "bold for boldness' sake" without brand alignment causes confusion.
- **gitingest.com** (r/web_design, 2,766 upvotes): the real-world neobrutalism example that prompted "what is this style?" — i.e., the look is distinctive enough to confuse first-time visitors.
- See [What Real Users Complain About (Synthesis)](../08-pitfalls-and-antipatterns/real-user-complaints.md).

## Senior-level checklist
- [ ] Style choice passes all four filters: audience expectation, content fit, trust sensitivity, accessibility ceiling.
- [ ] Primary text meets 4.5:1 (3:1 large/UI) on its **actual rendered surface**, including over glass/gradient/image.
- [ ] `backdrop-filter` has an `@supports` fallback; a reduce-transparency path exists where used.
- [ ] Theming uses semantic two-layer tokens; light/dark/system toggle present; no light→dark color inversion.
- [ ] No primary CTA or long-form copy on unmodified glass or directly on a gradient.
- [ ] Neumorphism (if present) is decorative only, with real affordance indicators alongside.
- [ ] Type system: variable font(s), `font-display: swap`, fluid `clamp()` scale, measure 60–75ch. Consider `local()` for fonts with system-font equivalents (Inter).
- [ ] Scroll effects use native `animation-timeline: scroll()` (with `@supports`) before reaching for GSAP; `prefers-reduced-motion` honored.
- [ ] Loud styles (maximalism/Y2K/brutalism) anchored by one grid; body text on calm high-contrast surfaces.
- [ ] Bento cells are each self-contained and unified by one spacing/border/color system.
- [ ] Claymorphism shapes use SVG/generator (not pure CSS); saturation calibrated to audience age.
- [ ] No core navigation hidden on mobile; experimental nav justified against quantified Jakob's-Law cost.
- [ ] `prefers-reduced-motion` honored for aurora/scroll animations.
- [ ] Reviewed against [Avoiding Generic 'AI' Aesthetics](avoiding-generic-ai-aesthetics.md) — the style reads intentional, not defaulted.

## Sources
- r/UXDesign — Apple Liquid Glass legibility discussion (755 upvotes; "blur ~8px, needs ≥24px").
- r/web_design — gitingest.com / neobrutalism thread (2,766 upvotes).
- r/web_design — PostHog OS-window homepage thread (287 upvotes).
- https://caniuse.com/ — `backdrop-filter` Baseline 2022; CSS `animation-timeline` Baseline 2024.
- https://www.w3.org/TR/WCAG21/ — WCAG 2.1 AA contrast requirements.
- https://web.dev/baseline — Baseline feature support definitions.
- https://neobrutalism.dev/ — neobrutalist React/Tailwind component library.
- https://www.claymorphism.com/ — claymorphism shadow generator.
- https://ui.shadcn.com/ — shadcn/ui components (independent, ~80k+ GitHub stars).
- Apple WWDC25 — macOS/iOS 26 Liquid Glass design system announcement (June 2025).
- Production references (publicly inspectable): apple.com, stripe.com, linear.app, vercel.com, gumroad.com, raycast.com, notion.so, medium.com.

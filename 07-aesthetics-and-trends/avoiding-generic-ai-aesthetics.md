# Avoiding Generic 'AI' Aesthetics

> AI tools default to the statistical median of 2019–2024 web design: centered hero, three feature cards, indigo→purple gradient, Inter. This guide shows how to recognize that "slop" fingerprint and override it at the system level — custom type, committed color, grid tension, intentional motion, real content — so output reads as *designed, not generated*. The senior-level outcome: every visual decision traces to a deliberate reason, and the page is distinguishable from a template.

**Apply when:** building any public-facing or brand site, or when a generated UI looks "fine but forgettable." · **Related:** [Typography Systems](../00-foundations/typography-systems.md) · [Color Theory & Systems](../00-foundations/color-theory-and-systems.md) · [Design Styles Taxonomy](./design-styles-taxonomy.md) · [CSS, Styling & Tailwind](../04-libraries-and-tools/css-styling-and-tailwind.md)

## TL;DR — rules at a glance
- **The slop fingerprint:** centered hero + 3 equal feature cards + `from-indigo-500 to-purple-600` gradient + Inter + uniform 16px radius + 0.1-opacity shadows. If you see ≥3 of these, you have AI slop.
- **The bar:** if you cannot articulate *why* a color/font/spacing value was chosen, it's a default, not a decision. Replace it.
- **Override at the system level first.** Define a token sheet + `DESIGN.md` *before* generating any component. Better prompts alone don't fix defaults.
- **Type is the fastest differentiator.** Swap Inter for a typeface with personality (Space Grotesk, Bricolage Grotesque, Fraunces). One change, recognizable page.
- **Commit to color.** One dominant color + 1–2 sharp accents beats a timid, evenly-distributed palette. Pull accents *from the real product UI*, not a downloaded mesh gradient.
- **One orchestrated page-load animation** beats scattered micro-interactions. Motion = state/attention/brand, never decoration.
- **Real content > placeholder aesthetic.** Actual screenshots, real photos, specific founder-voice copy. "Build the future" is a tell.
- **Strategic imperfection signals craft:** grain, loose illustration, slightly off-grid type. Intentional and systematic, not sloppy.
- **Obsessive detail closes the gap:** correct letter-spacing at display size (-0.02em to -0.04em), designed hover/focus states, real OG images, designed empty states. AI skips these; you don't.
- **Internal spacing ≤ external spacing** (Gestalt proximity). Violate this and hierarchy collapses.
- **Exact values, never descriptions** when prompting: `#E63946`, not "warm red." Descriptions reinterpret randomly.
- **Design for user action,** not for the mockup or the portfolio. Pretty + no conversion = failure.
- **Don't over-correct into noise:** stacked gradients + parallax + animated backgrounds is also slop, just louder.

---

## 1. Recognize the slop: the AI-default fingerprint

AI generators output the *average* of their training data. Tailwind UI's default button was `bg-indigo-500` from ~2019; LLMs trained on billions of tokens of that code now reproduce indigo→purple by default. This is a statistical artifact, not a design choice — which is exactly why it reads as cheap.

**The slop checklist (the more that match, the more generic):**
- [ ] Centered hero, centered everything below it (no compositional tension)
- [ ] Three feature cards in an equal icon-grid directly below the hero
- [ ] `linear-gradient(indigo-500 → purple-600)` on white background
- [ ] Inter (or system-ui) as the only typeface, all at weight 400/600
- [ ] Uniform `border-radius: 16px` on every card, button, input
- [ ] `box-shadow` at `opacity: 0.1` everywhere — depth without commitment
- [ ] Vague headline: "Build the future," "Your all-in-one platform," "Scale without limits"
- [ ] Stock/AI imagery: "diverse group looking at a laptop," generic 3D blobs
- [ ] A mesh gradient whose colors have *no relationship* to the actual product UI

> Reddit, r/web_design (arrogant_sparrow, 2025): "What I hate is that fucking color palette with the white background and purple accent. I see it everywhere. It makes beautiful sites look cheap or made by AI simply because of how common it is."

The mesh-gradient slop is the new Bootstrap navbar. r/Frontend (Academic-Yam3478): "We just traded the 'bootstrap navbar' for the 'tailwind mesh gradient.' … if AI trains on current sites, and current sites use AI to generate gradients… we get stuck in a visual echo chamber. The only way out is to force human inputs (photos, real-world colors) back into the process."

## 2. Override at the system level (before any component)

Prompting "make it less generic" fails because the model still has no constraints to obey. Fix it upstream with a token sheet and a brief the AI tool must read.

### 2.1 A `DESIGN.md` the AI must consume first
Keep it short, named, and concrete. This is the single highest-leverage artifact.

```markdown
# DESIGN.md
Aesthetic references: Linear (precision, mono accent), Notion (warm muted, semantic color).
Type: Display = "Clash Display" 600; Body = "Newsreader" 400; Mono = "JetBrains Mono".
Color tokens: --bg #0B0B0F; --text #ECECF1; --accent #FF5C00 (use sparingly, CTAs + active states only).
Radius: buttons 6px, cards 10px, inputs 4px (NOT uniform).
Motion: one staggered page-load reveal (0.1/0.2/0.3s); no scroll parallax; respect prefers-reduced-motion.
Banned: indigo/purple gradients, Inter, stock illustration, vague superlative headlines, uniform shadows.
```

### 2.2 Semantic CSS tokens (define before writing components)
Name by *role*, never by appearance. `--purple` rots the moment the brand color changes; `--color-action-primary` survives.

```css
:root {
  /* roles, not colors */
  --color-bg:            #0B0B0F;
  --color-surface:       #16161C;
  --color-text-primary:  #ECECF1;
  --color-text-muted:    #9A9AA8;
  --color-action-primary:#FF5C00;
  --color-feedback-success:#2FBF71;
  --color-feedback-error: #E5484D;

  /* 8px-base spacing, perceptual (not linear) scale */
  --space-1: 4px;  --space-2: 8px;  --space-3: 12px; --space-4: 16px;
  --space-6: 24px; --space-8: 32px; --space-12: 48px; --space-16: 64px; --space-24: 96px;

  --radius-input: 4px; --radius-button: 6px; --radius-card: 10px;
}
```

See [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md) for the full token architecture.

### 2.3 Prompting AI tools: how to enforce constraints

`DESIGN.md` only works if the AI tool reads it *before* generating. Structural rules:

1. **Front-load, never append.** Constraints placed after the component request are outweighed by the statistical prior. Lead with: *"Apply DESIGN.md strictly. Banned: Inter, indigo, purple gradients, uniform 16px radius, centered-only layout. Token file:"* — then the request.
2. **Positive + negative together.** Positive-only ("use Space Grotesk") leaves room for defaults to fill the gaps. Add explicit bans for the top 5 slop signals.
3. **Exact values, not adjectives.** `color: #FF5C00` not "warm orange accent." `border-radius: 6px` not "slightly rounded." Descriptors reinterpret on every run.
4. **Reference a real site for tone, not for copy.** "Compositional tension similar to Linear.app's hero — product screenshot right, short left-aligned headline, no centered symmetry." This gives the model a concrete model without asking it to clone.
5. **After generation: one audit pass.** Prompt: "Check this output against DESIGN.md. List every violation." The model will catch ~80% of its own slop if asked adversarially.
6. **Iterate on the token sheet, not the prompt.** If you need to add "no Inter" to every prompt, add it to `DESIGN.md` once and reference the file. Prompt length doesn't scale; a constrained token file does.

## 3. Typography: the fastest differentiator

Replacing Inter is the single change that most transforms a page from "generated" to "recognizable." Linear, Stripe, and Vercel each commissioned or custom-modified type for this reason (Vercel's "Geist," Stripe's bespoke headline serif).

**Drop-in replacements that keep readability:**
- **Geometric/grotesque sans:** Space Grotesk, Bricolage Grotesque, Cabinet Grotesk, Satoshi
- **Editorial serif (authority/warmth):** Fraunces, Newsreader, Crimson Pro, Playfair Display
- **Display (hero personality):** Clash Display, Obviously
- **Mono with character:** JetBrains Mono, IBM Plex Mono

**Make the type system *contrast*, not just *swap*:**
- Use **weight extremes** — 100/200 against 800/900 — not the timid 400-vs-600 default.
- Make the **hero ≥ 3× body size** (e.g., 72px display vs. 18px body). Between adjacent heading levels (h1→h2→h3), use a 1.25–1.5× step ratio — not 3×, which compresses to unreadable h3.
- Pair a display/serif headline with a clean sans body (Stripe's model) for editorial tension.

```css
h1 { font-family: "Clash Display"; font-weight: 600; font-size: clamp(2.5rem, 6vw, 4.5rem); line-height: 1.05; letter-spacing: -0.02em; }
p  { font-family: "Newsreader"; font-weight: 400; font-size: 1.125rem; line-height: 1.55; }
```

> Avoid shipping *kinetic typography* in production. Animated text fights screen readers and crawlers and causes layout shift that wrecks CLS. Admire it on Awwwards; don't paste it into a real product. See [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md). Full type guidance in [Typography Systems](../00-foundations/typography-systems.md).

## 4. Color: commit, and pull from the real UI

Timid, evenly-distributed palettes read as templates. **One dominant color + 1–2 sharp accents** reads as a decision.

- **Derive accents from the actual product UI**, not a stock gradient. r/Frontend (AimenLeene_): "Pulling colors from the actual UI is such an easy win — suddenly the hero looks like part of the product instead of a generic SaaS template. It reads as 'intentional design' instead of 'I downloaded this gradient at 2 a.m.'" If you can't find 2–3 strong colors in your UI, that's the real problem (euro-data-nerd).
- **Color should be semantic.** Notion: yellow highlights, blue links, red warnings. Figma: color distinguishes editing/collaboration states. Color carries meaning, not decoration.
- **Shade by hue rotation,** not by dumping black/white in. To lighten, rotate toward the nearest light hue; to darken, toward the nearest dark hue. Never rotate more than 20–30°. This keeps shades vivid instead of muddy/gray.
- **Gradients are not universally banned — the problem is the *indigo→purple on white* instance, not the technique.** When to use vs. avoid:
  - **Avoid:** mesh gradients unrelated to UI hues; `from-indigo-500 to-purple-600` as the primary brand expression; gradients as "make it look expensive" decoration.
  - **Use well:** a radial vignette darkening image edges (improves text contrast), a subtle 5–10% saturation gradient tying hero to page bg, a noise/grain gradient as texture. Keep gradient opacity ≤ 15% for backgrounds unless it *is* the feature.
  - Key test: remove the gradient. If the page still communicates the brand, the gradient was decoration; if something meaningful disappears, it's structural.

Deep dive: [Color Theory & Systems](../00-foundations/color-theory-and-systems.md).

## 5. Grid tension & composition

Centered-everything is the compositional equivalent of the purple gradient: zero editorial intent.

- **Break symmetry deliberately.** Asymmetric hero (text left, product right; or a 60/40 split), off-center focal points, content that bleeds to one edge.
- **Enforce the proximity rule (Gestalt):** space *within* a group < space *between* groups. A card's internal padding must be smaller than the gap to the next card. Violating this is the #1 hierarchy killer.
- **Use the spacing scale with intent** — vary padding by role (hero ≠ card ≠ footer). Identical 24px padding everywhere signals a template.
- **Vary radius by component** — inputs 4px, buttons 6px, cards 10px. Uniform 16px everywhere is a tell.

### 5.1 Alternatives to the hero + 3-card default

When the brief says "feature section," these avoid the equal-icon-grid trap:

| Pattern | When to use |
|---|---|
| Bento grid (mixed-size tiles, 2–3 columns) | Feature parity across many items; visual weight expresses priority |
| Zigzag (alternating image-left / text-right rows) | 3–5 features that each need explanation; slows scanning, increases depth |
| Numbered process list (large numeral + prose) | Sequential flows, onboarding steps, "how it works" |
| Single-feature spotlight per scroll section | Premium/single-product; each feature deserves its own moment |
| Stats + callout band | Social proof or scale claims; use real numbers, not "10× faster" |

None of these is inherently superior — the choice must trace to the content and brand character.

```css
/* internal ≤ external: hierarchy holds */
.feature      { padding: var(--space-6); }      /* 24px inside the group */
.feature-list { display: grid; gap: var(--space-12); } /* 48px between groups */
```

More: [Layout, Grids & Spacing](../00-foundations/layout-grids-and-spacing.md) · [Visual Hierarchy](../00-foundations/visual-hierarchy.md) · [Visual Design Principles & Gestalt](../00-foundations/visual-design-principles.md).

## 6. Motion: one orchestrated moment, not confetti

Scattered micro-interactions feel busy and generic. One well-choreographed page-load reveal feels intentional. Motion must do a job: communicate state, direct attention, or reinforce brand.

- **Staggered entrance** with `0.1s / 0.2s / 0.3s` delays gives a deliberate, sequenced feel.
- **Match motion to brand:** Linear's animations are mathematical and exact; Duolingo's are bouncy and expressive. Pick the curve that matches the voice.
- **Always respect `prefers-reduced-motion`.** Non-negotiable for accessibility.

```css
@media (prefers-reduced-motion: no-preference) {
  .reveal { opacity: 0; transform: translateY(12px); animation: rise .5s ease-out forwards; }
  .reveal:nth-child(1){animation-delay:.1s} .reveal:nth-child(2){animation-delay:.2s} .reveal:nth-child(3){animation-delay:.3s}
}
@keyframes rise { to { opacity:1; transform:none; } }
```

For React, use the Motion library (formerly Framer Motion); for choreographed scroll/timelines, GSAP. Prefer CSS-only for plain HTML. See [Micro-interactions & Motion](./micro-interactions-and-motion.md) and [Animation Libraries Stack](../04-libraries-and-tools/animation-libraries-stack.md).

## 7. Real content beats placeholder aesthetic

The fastest way to look generated is to look like a template waiting for content.

- **Real product imagery:** Loom uses actual screen recordings; Amplitude shows real dashboard data; Notion shows real workspace screenshots. Use the product, not stock.
- **No "diverse group at a laptop"** and no AI-generated 3D blobs — the visual twin of the purple gradient.
- **Founder-voice copy.** Basecamp (Jason Fried) sounds like a person, not a committee. Superhuman's every word signals speed. Stripe ships "Financial infrastructure for the internet" — specific and opinionated, not "Empowering teams to build better products."
- **Specificity wins:** name the user, the outcome, the number. Vague superlatives are interchangeable and forgettable.

See [Copywriting & UX Writing](../06-marketing-and-conversion/copywriting-and-ux-writing.md) and [Imagery, Icons & Assets](../00-foundations/imagery-icons-and-assets.md).

## 8. Strategic imperfection & art direction

In a sea of AI-smooth surfaces, controlled roughness reads as a human hand.

- **Texture:** subtle grain overlay (SVG `feTurbulence` or CSS `noise.png` at 3–8% opacity), dot/grid backgrounds at 2–4px, slight mis-registration. Premium-but-not-uptight — the pattern appears in any brand that needs to feel tactile rather than digital-clean (grain + tactile photography + slightly off-grid type).
- **Loose illustration:** wobbly outlines, naive shapes, hand-drawn marks. Imperfection *is* the strategy — it immediately separates a page from AI-smoothed defaults.
- **Expressive type systems** where letterforms perform: variable-weight fonts animated to visualize brand values (sound, energy, data), or display faces with exaggerated optical quirks. Aspirational reference, not copy target.
- **Imperfection is intentional and consistent** — applied as a system, not random noise. Inconsistency reads as unfinished; controlled roughness reads as craft.

### 8.1 Obsessive detail — the "designed, not generated" tell at close range

AI-generated UIs fail at micro-scale. These are the specifics that separate assembled from designed:

- **Icon sizing is exact and consistent:** all icons in a group share one size from the spacing scale (16, 20, or 24px); mixed 18/19/22px is a tell.
- **Hover states exist on every interactive element** — not just `opacity: 0.8` but a considered color shift, underline, or shadow change that encodes the brand motion vocabulary.
- **Focus rings are styled, not suppressed.** `outline: none` without a custom `:focus-visible` replacement is an accessibility failure and a craft failure.
- **Letter-spacing at display sizes:** headline type at ≥ 48px almost always needs `letter-spacing: -0.02em` to `-0.04em`; the default tracks too loose and looks typographically naive.
- **OG image and favicon match the page.** A white favicon or Twitter card with no design = zero attention to distribution surfaces.
- **Empty/error/loading states are designed**, not left as spinners on white. These are the states real users see most often when things go wrong.
- **No orphaned words in hero headlines.** Max 2 lines; tweak font-size or `max-width` so the last line has ≥ 3 words.

## 9. The "designed, not generated" bar (self-audit)

Before shipping, every primary decision must pass: *can I state the reason in one sentence?*

| Decision | Slop default | "Designed" alternative + reason |
|---|---|---|
| Font | Inter 400/600 | "Fraunces 600 headline — editorial warmth matching a wellness brand" |
| Accent | indigo→purple | "#FF5C00 pulled from the product's active-state orange" |
| Layout | centered + 3 cards | "asymmetric 60/40 hero; features in a 2-up zigzag to slow scanning" |
| Radius | 16px everywhere | "4/6/10px by component to encode element role" |
| Motion | hover scale on all | "single staggered load reveal; brand reads as precise" |
| Hero image | stock illustration | "real dashboard screenshot — hero = product, not marketing" |

If any cell still reads like the middle column, it's not done.

## Concrete specs & numbers
- **Spacing:** 8px base; perceptual scale 4·8·12·16·24·32·48·64·96px. 4pt baseline grid for typography; line-heights divisible by 4.
- **Body line-height:** 1.4–1.6 (140–160%). At 18px → 25–29px. Wide paragraphs ~1.8; narrow columns ~1.5.
- **Type size ratios:** hero display ≥ 3× body (e.g., 64px / 18px = 3.6×). Adjacent heading levels: 1.25–1.5× step (e.g., h1 64px → h2 48px → h3 32px). Desktop h1:body ≈ 3–4:1; never compress h3/h4 below 1.1× body or they lose hierarchy.
- **Mobile body:** floor at 16px; desktop body of 18–20px → 16px mobile. Do not scale headings proportionally — use `clamp()` with a tighter range.
- **Weight contrast:** 100/200 vs 800/900. Do not use 400/600 as the primary contrast pair.
- **Hue-rotation shading:** rotate toward nearest light/dark hue; max 20–30°.
- **Hero text legibility over images:** a bottom-to-transparent gradient overlay (`background: linear-gradient(to top, rgba(0,0,0,.65), transparent)`) is more reliable than `text-shadow`. If using text-shadow, keep blur ≤ 8px with 0 offset — a 50px blur creates a hazy corona around each glyph and degrades on complex backgrounds.
- **Letter-spacing:** display type ≥ 48px → `letter-spacing: -0.02em` to `-0.04em`. Body ≤ 18px → 0 or slight positive for light weights.
- **Focus rings:** never `outline: none` without a custom `:focus-visible` replacement. Use `outline: 2px solid var(--color-action-primary); outline-offset: 2px`.
- **Staggered reveals:** `animation-delay: 0.1s, 0.2s, 0.3s`.
- **Grain texture:** SVG `feTurbulence` or PNG noise at 3–8% opacity; 2–4px dot size for dot grids.
- **UI scale:** 8pt linear for components, 4pt half-step for icons/small text.

## Tools & libraries
- **Tailwind v3/v4 + shadcn/ui** — the AI-ready stack; produces the *most* generic output **unless** you override its default tokens with a custom palette and per-component radii. Configure `theme.colors` and ban indigo. See [CSS, Styling & Tailwind](../04-libraries-and-tools/css-styling-and-tailwind.md).
- **v0 by Vercel** — strong frontend generation; still defaults to slop without a constraints brief. Feed it `DESIGN.md`.
- **Motion (ex-Framer Motion)** — React animation. **GSAP** — complex scroll/timeline choreography. CSS-only for plain HTML.
- **CSS custom properties + token system** — the primary weapon against defaults; define roles before components.
- **Figma** — design-system source of truth (component library + CSS-variable sheet minimum).
- **Reference libraries:** Awwwards (bleeding-edge taste), Dribbble, Mobbin, land-book.com, Muzli. **Use for *why*, not copy-paste** — cloning Stripe's gradient without its compositional logic just makes new slop.
- **Variable fonts** — one file, many weights/widths; good for responsive type scaling. (Avoid for kinetic text in production — see §3.)
- **When NOT to use:** don't reach for GSAP/parallax/WebGL on a content or conversion page where they add load and break edge cases. Restraint is a feature.

## Do / Don't
**Do**
- Write `DESIGN.md` + semantic tokens before generating a single component; front-load constraints in AI prompts (§2.3).
- Replace Inter with a typeface that carries the brand's voice.
- Commit to one dominant color + sharp accents pulled from the real UI.
- Ship one orchestrated page-load animation; respect `prefers-reduced-motion`.
- Use real screenshots, real photos, and specific founder-voice copy.
- Vary radius, padding, and shadow by component role.
- Keep internal spacing < external spacing.
- Set letter-spacing on display type; style all hover/focus states; design the OG image.
- Run an adversarial audit pass on AI-generated output against `DESIGN.md`.

**Don't**
- Don't ship indigo→purple gradients on white, or Inter-only typography.
- Don't center everything or drop three equal cards under the hero by default.
- Don't use stock/AI "team at a laptop" imagery or mesh gradients unrelated to UI hues.
- Don't describe colors ("warm red") to AI tools — give exact hex.
- Don't over-correct into stacked gradients, parallax, and animated backgrounds.
- Don't prioritize the mockup over real content, real devices, and conversion.
- Don't suppress `:focus-visible` without a replacement — it breaks accessibility and signals unfinished craft.

## Common pitfalls & how to avoid them
- **Inter by reflex** → it's the LLM statistical default; pick a typeface with intent (§3).
- **Purple gradient fingerprint** → ban it in `DESIGN.md`; derive accents from the UI (§4).
- **Uniform 16px radius / 24px padding / 0.1 shadows** → vary by role; commit to depth or drop it (§5).
- **Hero + 3 cards trifecta** → break the grid; use asymmetry/zigzag (§5).
- **Vague headlines** → name user + outcome + number; founder voice (§7).
- **Color-as-description in prompts** → exact hex/tokens; the model reinterprets words every run.
- **Generic mesh gradient unrelated to product** → match UI hues or remove; or use subtle noise/radial vignette instead (§4).
- **Over-design** → noise, not signal; subtract until each effect earns its place.
- **No letter-spacing at display size** → set `letter-spacing: -0.02em` to `-0.04em` on all type ≥ 48px.
- **Default/missing hover and focus states** → every interactive element needs a considered state; `:focus-visible` must be replaced if `outline: none` is set.
- **Gradient ban applied too broadly** → the indigo/purple *instance* is banned, not gradients as a technique; structural overlays, noise gradients, and radial vignettes are valid (§4).
- **AI prompt describing instead of specifying** → vague constraint prompts ("make it look premium") leave statistical defaults in place; use §2.3 pattern.
- **Desktop-first** → most traffic for hotels/e-commerce/local services is mobile; design mobile-first ([Breakpoints & Mobile-First](../02-responsive-adaptive/breakpoints-devices-mobile-first.md)).
- **Designing for the portfolio** → gorgeous in mockups, broken at odd viewports / with real copy. Test real content on real devices.
- **Copying patterns blind** → learn *why* a pattern works before reusing it, or you reproduce slop with new colors.

## What real users complain about
*(synthesized from r/web_design, r/Frontend, 2025)*
- **The palette itself signals "AI."** The white-bg + purple-accent combo is so common it makes good work look cheap (arrogant_sparrow). Distinctiveness is now a quality signal, not a luxury.
- **Devs can assemble blocks but not compose.** "Most web devs just know how to add tailwind premade page blocks. If you asked them to write some interesting HTML and CSS — they'd just be stuck." (sheriffderek) — the gap between *assembled* and *designed*.
- **Beautiful sites feel brittle.** "When you build for visual impact first, you often end up with overly complex animations that break on different devices… fragile CSS that falls apart when content changes, poor semantic HTML under all the visual polish." (jefdiesel) — aesthetics over structure backfires.
- **The visual echo chamber.** AI trains on AI-generated gradients; the only exit is forcing human inputs — real photos, real-world colors — back in (Academic-Yam3478).
- **Hero must look like the product.** A background unrelated to the UI "feels like a marketing layer, not the product." (euro-data-nerd / AimenLeene_)
- **Economic gravity toward generic.** "Everyone wants to build something for a billion… quirky designs serve a niche audience." (InternationalWait538) — distinctiveness costs conviction; budget for it deliberately.

See the broader synthesis in [Real User Complaints](../08-pitfalls-and-antipatterns/real-user-complaints.md) and [Common Mistakes & Antipatterns](../08-pitfalls-and-antipatterns/common-mistakes-antipatterns.md).

## Senior-level checklist
- [ ] `DESIGN.md` exists with named references, type, hex tokens, motion rules, and a banned list.
- [ ] Semantic CSS tokens defined (`--color-action-primary`, etc.) before components.
- [ ] No Inter / system-ui as the brand face; chosen type carries voice with weight + size contrast.
- [ ] No indigo→purple gradient; accents derived from the real product UI.
- [ ] Radius, padding, and shadow vary by component role — no uniform 16px/24px/0.1.
- [ ] Layout has intentional asymmetry/tension; not centered-everything + 3 cards.
- [ ] Internal spacing < external spacing throughout (proximity holds).
- [ ] One orchestrated page-load animation; `prefers-reduced-motion` honored; no kinetic text in prod.
- [ ] Imagery is real product/photo; copy is specific and founder-voiced; zero vague superlatives.
- [ ] Tested with real content at narrow + odd viewports; mobile-first.
- [ ] Every primary decision passes the one-sentence "why" test (§9 table).
- [ ] The page is distinguishable from a shadcn/Tailwind default at a glance.
- [ ] Obsessive detail pass complete: display letter-spacing set; hover/focus states on all interactive elements; `:focus-visible` custom ring; OG image designed; no orphaned headline words.
- [ ] AI prompting used §2.3 discipline: constraints front-loaded, exact hex values, adversarial audit pass run after generation.

## Sources
- r/web_design and r/Frontend community threads (2025) — quoted inline above; treat as practitioner sentiment, not measured data.
- Refactoring UI (Adam Wathan & Steve Schoger) — spacing scale, hue-rotation shading, weight/size contrast principles.
- Apple Human Interface Guidelines; Material Design; WCAG 2.2 — for touch-target and motion-accessibility norms referenced in linked guides.

*Statistics circulating in trend write-ups (e.g., "38% of visitors leave over poor design," "94% human-content win rate," "40–62% AI-code flaw rate") appear in the dossier but are not independently verified here; cite only with the original source attached.*

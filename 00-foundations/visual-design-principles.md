# Visual Design Principles & Gestalt

> The grammar of visual composition: how balance, contrast, emphasis, scale, proximity, alignment, repetition, and white space — plus the Gestalt grouping laws the brain runs automatically — combine into layouts that read as clear, intentional, and beautiful. Master this and your screens feel designed rather than assembled; ignore it and even pixel-perfect components add up to noise the user cannot parse.

**Apply when:** composing any layout — deciding what groups with what, what dominates, and where the eye lands. · **Related:** [Visual Hierarchy & Scanning Patterns](./visual-hierarchy.md) · [Layout, Grids & Spacing Systems](./layout-grids-and-spacing.md) · [Color Theory & Color Systems](./color-theory-and-systems.md) · [The Laws of UX](../01-ux-fundamentals/laws-of-ux.md)

## TL;DR — rules at a glance

- **One focal point per view.** The largest / highest-contrast / most-isolated element wins by default. If two things fight to be the focal point, neither is.
- **Gestalt is involuntary.** Users *will* group elements by proximity, similarity, enclosure, and motion whether you intend it or not. Design the groupings you want, or fight the ones you got.
- **Proximity is the strongest grouping cue** — it overrides color and shape. Two identical buttons placed far apart read as unrelated; two different shapes placed close read as a group.
- **Functionally identical → look identical. Functionally different → look different.** Violating similarity is the deepest source of user confusion.
- **The internal ≤ external rule:** space *inside* a component must be less than space *between* components. Padding inside a card < margin between cards. Get this wrong and grouping collapses.
- **Balance via visual weight, not symmetry.** Weight = size × darkness × saturation × texture × distance-from-center. A small saturated red balances a large pale gray.
- **Contrast creates hierarchy** across size, weight, shape, texture — not just color. But **never reduce text contrast to de-emphasize** — it breaks legibility (and WCAG) before it reduces prominence.
- **Cards = common region.** Enclosure groups heterogeneous content into one scannable unit even without proximity. Most reliable grouping tool in dense UI.
- **Design grayscale first, color last.** Forces hierarchy from spacing/size/weight so color isn't a crutch. (Refactoring UI)
- **Start with too much white space, remove until satisfied.** Your default instinct is always too tight, never too loose.
- **Align everything to a grid, 100% of the time.** 1–2px misalignment reads as sloppy even to people who can't name why.
- **Prägnanz:** the brain resolves to the simplest interpretation. Make the simplest clear form; complexity must be deliberate and meaningful, never accidental.
- **Repetition = system.** Consistent spacing, color, and type weights signal "same family." Inconsistency reads as error or noise, not variety.
- **Not everything can be important.** Emphasizing everything emphasizes nothing — hierarchy *requires* deliberate de-emphasis of the secondary.

---

## 1. The mental model: composition is grouping + ranking

Two questions answer most layout decisions:

1. **What belongs with what?** → solved by the **Gestalt grouping laws** (proximity, similarity, common region, common fate, closure, continuity, figure-ground).
2. **What matters most?** → solved by the **emphasis principles** (scale, contrast, focal point, balance) and detailed in [Visual Hierarchy](./visual-hierarchy.md).

White space, alignment, and repetition are the *connective tissue* that make both legible. Everything below is one of these three jobs.

---

## 2. Gestalt principles — the grouping laws the brain runs for you

Gestalt ("unified whole") describes how humans perceive structure from parts automatically and pre-attentively (before conscious reading). You don't get to opt out; you only get to direct it.

### 2.1 Proximity — *strongest cue*

Elements placed close together are perceived as related. Proximity **overrides similarity and color**: two identical blue buttons far apart will not read as a set.

- Signal distinct groups with **white-space gaps**, not borders or lines, wherever possible.
- A form label must be visibly closer to its input than to the next field above it.
- The internal ≤ external rule is just proximity applied to nesting.

```css
/* Label binds to its OWN input because the gap below is larger */
.field { margin-bottom: 24px; }        /* between fields (external) */
.field label { margin-bottom: 6px; }   /* label→input (internal, smaller) */
```

### 2.2 Similarity

Elements sharing color, shape, size, or weight are perceived as one set. **The hard rule:** functionally identical elements must look identical; functionally different elements must look different. A "secondary" button that looks like a link, or two CTAs styled differently for no reason, breaks the user's model.

### 2.3 Common region — *the card*

Elements inside the same bounded area (card, panel, border, background fill) read as a group **regardless of proximity**. This is the single most reliable grouping tool in dense UI because it works even when you can't add enough space.

- The card pattern is the primary application. A subtle background fill (`#f8f9fa` on white) is often enough — no border needed.
- **Card borders/backgrounds must clear 3:1 contrast from the page** (WCAG 1.4.11) or the boundary disappears and grouping fails.

### 2.4 Common fate

Elements moving in the same direction (same speed, same vector) are perceived as belonging together. Applies to dropdowns, modals sliding in, stepper transitions, accordion reveals, skeleton loaders, and scroll-parallax layers.

- Related elements must animate **together** — same timing, same direction, same easing. If a panel's items stagger in independently they read as separate objects, not one group.
- Parallax layers moving at different speeds intentionally signal depth/separation (ground vs. figure) — use it only when you want the composition to split, not bind.
- Skeleton loaders: all placeholder rows shimmer in sync → read as one loading region; async shimmer → feels broken.

(See [Micro-interactions & Motion](../07-aesthetics-and-trends/micro-interactions-and-motion.md).)

### 2.5 Closure

The brain completes incomplete shapes into whole figures. Uses:

- **Partial visibility at scroll edges** (a half-cut card peeking off-screen) signals "more content this way" — a key affordance for horizontal carousels.
- Enables **simpler icons** — you don't need to draw the whole shape for it to read.

### 2.6 Continuity

Elements arranged along a line or smooth curve feel related; the eye follows curves effortlessly. Aligning items to a shared edge or baseline creates an invisible line that binds them. **Disrupt continuity deliberately** (a break in the grid, a shift in rhythm) to signal a new section begins.

### 2.7 Figure-ground

Users auto-separate foreground (figure) from background (ground). The figure must differ significantly from the ground.

- **Background images must never compete with foreground text/CTAs.** Use a scrim/overlay, blur, or duotone to push the image back.
- Check perceived brightness, not hue, to confirm text survives a colored/photo background:
  `brightness = √(0.299·r² + 0.587·g² + 0.114·b²) / 255` where r/g/b ∈ [0,255] — result 0→1; value > 0.5 → use dark text, ≤ 0.5 → light text. (This is the sRGB perceived luminance approximation, distinct from the WCAG relative luminance formula which gamma-expands each channel first.)

### 2.8 Prägnanz (law of good form / symmetry)

Ambiguous compositions resolve to the **simplest** possible interpretation. Design the simplest clear form. Symmetry, regularity, and order reduce cognitive load; accidental complexity reads as error. If a layout looks busy, the fix is usually *removal*, not rearrangement.

---

## 3. Composition principles — ranking and balance

### 3.1 Scale

Relative size signals importance and rank. The largest element is the focal point by default.

- **Cap at 3 distinct sizes per view** (small / medium / large). More than 3 = noise, not hierarchy. The "3" is a practitioner ceiling, not an empirical law — use it as a forcing function to make size meaningful. The type system may define 4 levels page-wide (see § Specs), but any single viewport region should activate at most 3 simultaneously.
- Size is *relational* — the ratio between sizes matters more than absolute px. Use a modular scale (§ Specs).
- De-emphasize labels/metadata by making them genuinely smaller + lighter, not "slightly smaller."

### 3.2 Emphasis / focal point — the Von Restorff effect

Every composition needs **one dominant element**. Achieve it via the biggest size, highest contrast, brightest/most-saturated color, most isolation (white space around it), or most complex texture.

- **Von Restorff (isolation) effect:** the element that differs *most* from its surroundings is remembered best. Your **primary CTA should be the single most visually different element on the page** — different color, weight, and shape from everything around it.
- Isolation is underrated: surrounding an element with empty space makes it a focal point with zero added "loudness."

### 3.3 Balance & visual weight

Balance = distribution of visual weight so the composition feels stable. **Visual weight** is driven by:

| Factor | Heavier when… |
|---|---|
| Size | larger |
| Color value | darker |
| Saturation | more saturated |
| Temperature | warmer (red heaviest, yellow lightest) |
| Texture | more complex / detailed |
| Position | higher on the page; farther from center |

A **small bright saturated red** can balance a **large pale desaturated gray** — equal weight, unequal size. This is the engine of asymmetric layouts.

- **Symmetrical:** mirror-image around a central axis (left-right most common in web; top-bottom applies to stacked hero layouts). Formal, stable, trustworthy → corporate/institutional. Risk: static/boring when overused; the eye has no incentive to travel.
- **Asymmetrical:** unequal elements of *equivalent weight* on each side — achieved by trading size for saturation, or position for texture. The dominant mode in contemporary web design — dynamic yet stable. Default to this for editorial/marketing/product layouts.
- **Radial:** elements around a center point. Rare in web layout; used for icons, central-metric dashboards, radial nav.

### 3.4 Contrast

Juxtapose dissimilar elements to signal distinctness and build hierarchy. Contrast applies to **size, weight, shape, texture, and color** — reach for non-color contrast first so the design survives grayscale and colorblindness.

> **NNGroup warning:** never reduce *text contrast* to de-emphasize content. It destroys legibility (and accessibility) before it meaningfully reduces prominence. De-emphasize with **lighter font-weight** or a **pre-established muted palette shade** instead — never by dimming toward the background.

---

## 4. White space — macro and micro

White space (negative space; needn't be white) is an active design element, not leftover.

- **Macro white space:** large gaps between major sections / around the composition. Creates breathing room and section separation, and reads as **premium/minimal/luxury**. More macro space → higher perceived value.
- **Micro white space:** small gaps between lines, list items, icon↔label, within components. Drives legibility and lowers cognitive load. Cramped micro spacing makes content feel cheap and hard even when everything else is right.
- **Heuristic:** start with *too much* white space and remove until satisfied. Your untrained default is always too tight. (Refactoring UI)
- White space is also a **grouping tool** (proximity) and an **emphasis tool** (isolation) — it does triple duty.

---

## 5. Alignment

Every element should align to a grid **or** to another element. Misalignment creates visual noise even when users can't articulate why — the brain reads broken continuity as carelessness.

- **Align to a grid 100% of the time.** Even 1–2px drift reads as sloppy.
- **Prefer one strong edge.** Pick a left (or right) edge and commit; scattered center-points of differently sized elements look chaotic.
- **Body text:** left-align. The consistent left edge gives the eye a reliable return point (reading rhythm). **Never center-align body text longer than ~2–3 lines** — ragged left edges force the eye to re-hunt each line's start.
- **Center-align** is for short display text only: hero headings, short CTAs, single-line captions, empty states.

```css
.prose { text-align: left; max-width: 65ch; }   /* readable */
.hero h1 { text-align: center; }                 /* short, display only */
```

---

## 6. Repetition / consistency

Repeated visual elements unify a design and signal "one system." Reuse the *same* spacing steps, color shades, type weights, corner radii, and shadow levels everywhere. Inconsistency is perceived as error or noise — not as creative variety.

- Variation should come from **a constrained set of tokens**, not from improvised one-off values. (See [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md).)
- Repetition is similarity applied across a whole page/site: it's what makes a design feel "of a piece."

---

## 7. Proportion & scale relationships

Sizes and divisions should have **intentional ratios**, not arbitrary pixel counts.

- **Rule of thirds:** place key elements along thirds rather than dead-center for dynamic, balanced layouts. This is a compositional heuristic from photography — it has no empirical validation for screen UI, but remains a useful forcing function against centering everything reflexively.
- **Golden ratio (1:1.618):** commonly cited for content/sidebar splits and type scales. No rigorous UI research confirms a usability benefit over other well-proportioned ratios. Use it if a ratio around 1.6 fits your grid cleanly (e.g., 8+5 = 13-col, or 5:8 column split on a 13-col); don't distort a 12-col grid to chase the exact figure.
- **Practical column splits on a 12-col grid:** 6+6 (equal, symmetric), 8+4 (content-heavy), 4+4+4 (three-up), 9+3 (main + aside). Decide from content priority, not aesthetic ratios alone.
- Whenever you set a size not derived from a grid step, ask "why this number?" — if no good answer, default to the nearest scale value.

---

## 8. How it all combines — a worked layout

A pricing section, built from the principles:

1. **Common region:** each plan is a card — borders ≥3:1 vs. page background.
2. **Similarity:** all three cards share identical structure, type, and spacing (they're functionally equal).
3. **Emphasis / Von Restorff:** the recommended plan breaks similarity *deliberately* — accent border + filled CTA + slight scale-up. Now it's the focal point.
4. **Proximity / internal ≤ external:** tight padding inside each card; larger gutters between cards.
5. **Hierarchy (scale):** price (large) > plan name (medium) > feature list (small) — 3 sizes.
6. **Alignment:** feature lists left-aligned on a shared edge; prices baseline-aligned across cards.
7. **White space:** generous macro space above/below the section sets it apart; micro space between feature rows ≥ 0.5em.
8. **Hick's Law:** 3 plans, not 7 — fewer choices convert better. (See [Laws of UX](../01-ux-fundamentals/laws-of-ux.md).)

---

## Concrete specs & numbers

**Contrast (WCAG)**
- AA: **4.5:1** normal text (<18pt / <14pt bold); **3:1** large text (≥18pt/24px, or ≥14pt/18.66px bold).
- AAA: **7:1** normal; **4.5:1** large.
- **1.4.11 Non-text Contrast (AA): 3:1** for UI components — form borders, focus rings, meaningful icons, *card boundaries*.
- Low contrast is the #1 web a11y failure — **81% of home pages** (WebAIM Million 2024, webaim.org/projects/million). See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md).

**Type & reading**
- Body base **16px** (web standard); mobile **16–18px**.
- Line height **1.4–1.6×** unitless (16px → ~22–25px). Use `line-height: 1.5`, not `24px`.
- Line length **45–75 characters** (target ~66); WCAG cap **80**. Practical: `max-width: 60–65ch`.
- Paragraph `margin-bottom` **>** leading gap between lines. At 16px base with `line-height: 1.5`, lines are 24px apart and the leading gap between lines (24−16 = 8px) is 8px; paragraph spacing of 16–24px is clearly larger and prevents proximity collapse. Scale this proportionally — at 20px base, aim for 20–32px paragraph spacing.
- List-item spacing ≥ **0.5em**.
- Modular scale ratios: **1.250** Major Third (versatile UI), **1.333** Perfect Fourth (editorial hierarchy), **1.618** Golden (creative). Below **1.125** the steps are imperceptible.
- Example 1.25 scale: 16 → 20 → 25 → 31 → 39px.
- **Cap 3–4 type levels** per page; **≤2 typeface families** — vary by weight/size/color within them.

**Spacing & grids**
- **8-point grid:** 8/16/24/32/40/48/64 (Apple + Material). **4-point grid** (4/8/12/16/20/40) for dense UI.
- Constrained scale (Refactoring UI): **4, 8, 16, 24, 32, 48, 64** — keep to **6–9 steps** total.
- Column grids: **4-col** mobile (~375px) → **8-col** tablet (~768px) → **12-col** desktop (~1440px). Gutters/margins in multiples of 8 (16–32px typical).

**Color & depth**
- Build **≥10 shades** per role: 8–10 greys, 5–10 primary, 5–10 accent (Refactoring UI sweet spot: **9 shades**, divides cleanly).
- Use very dark grey (**#1a1a1a**) for text, not pure **#000**.
- Limit accent/hierarchy colors to **2–3**; reserve the highest saturation for the **primary CTA**.
- Gradients: keep hues within **30°** on the wheel to avoid muddy mid-tones.
- Shadow elevation: **~5 levels** is enough; combine a tight dark ambient shadow + a larger soft diffuse one for realistic depth.

**Cognitive limits feeding composition**
- **Miller's Law:** 7±2 items in working memory (Miller 1956); revised by Cowan (2001) to ~4 chunks in modern research. Practical rule: chunk nav/menus/lists into groups of **4–7**; treat 7 as a hard ceiling.
- **Hick's Law:** decision time grows logarithmically with the number of choices (Hick 1952). Reduce visible options; use progressive disclosure for infrequently-used actions. This is a distinct law from Pareto — do not conflate them.
- **Pareto (80/20):** ~80% of user actions involve ~20% of features. Rank those high-frequency actions first in visual hierarchy; relegate the tail to secondary menus or progressive disclosure.
- **Doherty Threshold:** keep system response <**400ms** to preserve flow (affects perceived interaction quality).
- **Fitts' Law:** primary targets large and within thumb reach; destructive actions small and far from primary. Touch target minimums: **44×44 CSS px** (WCAG 2.5.5 AAA / Apple HIG pt), **48×48dp** (Material Design M3), **24×24 CSS px** minimum floor (WCAG 2.5.8 AA, added 2.2).

---

## Tools & libraries

- **Stark / Contrast (Figma), WebAIM Contrast Checker, axe DevTools** — verify 4.5:1 / 3:1 / 1.4.11 before shipping.
- **Type-scale.com, Modularscale.com** — generate modular type scales from a ratio.
- **CSS `:root` custom properties** — encode the spacing/type/shadow scales as tokens so repetition is enforced, not hoped for.
- **Tailwind's default scale** already implements an 8/4-point spacing system and a modular type scale — good defaults to lean on. See [CSS, Styling & Tailwind](../04-libraries-and-tools/css-styling-and-tailwind.md).
- **When NOT to reach for a tool:** don't add a UI library to "get consistency" if your real problem is an undefined scale — define tokens first.

---

## Do / Don't

**Do**
- Solve hierarchy in grayscale, then add color as the last layer.
- Make padding-inside < margin-between for every grouped component.
- Give every view exactly one focal point; make the CTA the most-different element.
- Use cards/common-region to group heterogeneous content; clear 3:1 boundary contrast.
- De-emphasize with lighter weight or a muted palette shade.
- Align to the grid for every element; pick one strong edge.
- Default to more white space; remove only if it reads as disconnected.

**Don't**
- Don't dim text toward the background to "de-emphasize" it (a11y + legibility break).
- Don't use 8–10 type sizes or arbitrary 13/17/23px spacing — commit to a scale.
- Don't center-align multi-line body text.
- Don't style functionally-identical controls differently (or different ones the same).
- Don't let a background image compete with foreground text — scrim/blur it.
- Don't place related controls far apart and expect color/shape to group them.
- Don't try to emphasize everything.

---

## Common pitfalls & how to avoid them

| Pitfall | Why it fails | Fix |
|---|---|---|
| Low-contrast text to "quiet" content | Inaccessible + hard to read before it's less prominent | Reduce font-weight or use a muted palette shade |
| Too many font sizes (8–10) | Noise, not hierarchy | Cap 3–4 levels system-wide; use a modular scale |
| Arbitrary spacing (13/17/23px) | Broken rhythm reads as amateur even when each element is fine | 4px or 8px grid only |
| White-space phobia (cramming) | Filling empty space with elements raises cognitive load and kills focus | Treat empty space as intentional; remove elements, not space |
| `internal > external` spacing | Items bind to the wrong neighbor; groups dissolve | Padding inside < margin between, always |
| Cards with invisible boundaries | Common-region grouping silently fails | Border/fill ≥3:1 vs. page (WCAG 1.4.11) |
| Two competing focal points | Eye doesn't know where to land; both lose | Pick one; de-emphasize the other |
| Misalignment (1–2px drift) | Reads as sloppy/untrustworthy | Snap to grid; share edges/baselines |
| Color as the *only* signal | Fails colorblind users + dies in grayscale | Pair color with size/weight/shape/position |
| Accidental symmetry-breaking | Reads as a bug, not a choice | Break similarity only with clear intent (the focal element) |

---

## What real users complain about

Synthesized from accessibility audits and UX feedback patterns:

- **"I can't read the gray-on-white text."** The most common complaint, and it maps directly to the #1 a11y failure (81% of home pages, WebAIM Million 2024). Light placeholder/secondary text is the usual culprit.
- **"Everything looks the same — I can't tell what's clickable."** Similarity violated in reverse: links, buttons, and static text aren't visually distinct enough.
- **"It feels cluttered / overwhelming."** Cramped micro spacing + too many simultaneous focal points + no clear region grouping. Users read it as stress.
- **"Which button do I press?"** Two equally-weighted CTAs, or a secondary action styled as prominently as the primary. No Von Restorff standout.
- **"It looks cheap / untrustworthy."** Usually inconsistent spacing, misalignment, and too little white space — the Aesthetic-Usability effect working against you: low polish reads as low credibility.
- **"I didn't see there was more below/beside."** No closure cue (no peeking card edge) on a carousel or scroll region, so users miss content entirely.

See [What Real Users Complain About](../08-pitfalls-and-antipatterns/real-user-complaints.md).

---

## Senior-level checklist

- [ ] **Squint test:** blur the screen — exactly one focal point dominates; the CTA is the standout blob, not the logo.
- [ ] **Grayscale test:** the design's hierarchy and grouping survive with all color removed.
- [ ] Every grouped component satisfies **internal padding < external margin**.
- [ ] Every element snaps to the grid; shared edges/baselines verified (no 1–2px drift).
- [ ] **≤3 sizes** per view, **≤4 type levels** per page, **≤2 type families**, **2–3 accent colors** total.
- [ ] All spacing values come from one **4/8-point scale** (no orphan 13/17/23px).
- [ ] Text contrast ≥ **4.5:1** (normal) / **3:1** (large); UI/card boundaries ≥ **3:1** (1.4.11).
- [ ] No de-emphasis achieved by lowering text contrast — only weight/palette shade.
- [ ] Functionally identical controls look identical; functionally different ones look different.
- [ ] Body text left-aligned, **45–75ch** measure; centering reserved for short display text.
- [ ] Cards/regions used for heterogeneous grouping have **visible boundaries**.
- [ ] Background images don't compete with foreground text (scrim/blur/duotone applied).
- [ ] Animations group related elements via **common fate** (move together, same vector/speed).
- [ ] Macro white space separates major sections; you removed *from* generous, not added *to* cramped.
- [ ] The primary CTA is the single most visually distinct element (Von Restorff confirmed).

---

## Sources

- Nielsen Norman Group (nngroup.com) — scale, visual hierarchy, contrast and de-emphasis guidance, Gestalt articles.
- *Refactoring UI* by Adam Wathan & Steve Schoger (refactoringui.com) — grayscale-first, white-space, spacing/type scales, 9-shade palettes.
- WebAIM Million 2024 (webaim.org/projects/million, March 2024) — low-contrast text prevalence 81% of home pages analyzed; this is the #1 detected WCAG failure category.
- W3C WCAG 2.2 (w3.org/TR/WCAG22) — 1.4.3 / 1.4.6 contrast ratios; 1.4.11 non-text contrast (3:1); 1.4.8 Visual Presentation (AAA: line length, spacing, justification, foreground/background); 2.5.5 target size (44×44 CSS px, AAA); 2.5.8 target size minimum (24×24 CSS px, AA, new in 2.2).
- Laws of UX (lawsofux.com) — Von Restorff, Miller's, Hick's, Fitts', Doherty, Aesthetic-Usability, Serial Position, Peak-End, Jakob's, Pareto.
- Material Design (m3.material.io) & Apple Human Interface Guidelines (developer.apple.com/design) — 8-point grid, touch-target minimums.

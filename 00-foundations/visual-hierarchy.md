# Visual Hierarchy & Scanning Patterns

> How to arrange page elements by importance so users see the right things in the right order — using size, weight, color, contrast, spacing, position, and motion. Master this and your layouts guide the eye to the value proposition and the primary action without the user thinking about it; ignore it and even technically-correct screens feel like a flat gray wash with no clear next step.

**Apply when:** designing any screen, page, or component where a user must orient, decide, or act. · **Related:** [Visual Design Principles & Gestalt](./visual-design-principles.md) · [Web Typography Systems](./typography-systems.md) · [Layout, Grids & Spacing Systems](./layout-grids-and-spacing.md) · [The Laws of UX](../01-ux-fundamentals/laws-of-ux.md)

## TL;DR — rules at a glance

- Hierarchy = ordering elements by importance via **7 levers**: **size, weight, color/saturation, contrast, spacing, position, motion**. Combine ≥2; never rely on one alone. Motion is the most powerful because the visual cortex detects it before conscious attention — use it sparingly.
- **De-emphasize, don't shout.** Hierarchy comes from making secondary things smaller/lighter/quieter, not from making primary things bigger and louder.
- **One primary action per view.** Exactly one filled/high-emphasis button. Everything else is outlined (secondary) or text-only (tertiary).
- **Squint test:** blur the design at 5–20px radius. The CTA and value prop must be the darkest, most prominent blobs. If the logo wins, hierarchy failed.
- **Design in grayscale first.** Get hierarchy from spacing + size + weight before adding any color. Color is the last layer, never the only signal.
- Cap variety: **≤3 sizes, ≤3 contrast levels, ≤2 primary + 2 secondary colors, ≤2 type families** per design.
- Use a **modular type scale** (e.g., 12/14/16/20/24/30/36px) and a **spacing scale** (4/8/16/24/32/48/64px). Adjacent steps should differ ≥25%.
- **Labels are secondary; data is primary.** Form labels, table headers, metadata support content — make them quieter, never equal to the data.
- **Front-load keywords.** Put them at the start of headings and the first words of paragraphs — that is where F-pattern fixations land.
- **Position is weight.** Top-left and top-center get disproportionate attention in LTR layouts; put nav, value prop, and CTA there.
- **Spacing groups.** Tighter = related, looser = separate. A label must be visibly closer to its input than to the next field.
- **Never de-emphasize with contrast reduction alone** — it breaks WCAG and fails low-vision users. Reduce size, weight, or saturation instead.
- First impressions form in **~50ms** (Lindgaard et al. 2006), before any word is read. Visual weight distribution is processed subconsciously — get the blobs right first.
- Don't assume users read. They read ~25% of text and visit <1 min on average. Design for scanning, not reading.

---

## The seven levers of hierarchy

Every hierarchy decision pulls one or more of these. Rank them by how fast the brain processes the signal:

| Lever | Signal speed | How to apply | Caution |
|---|---|---|---|
| **Motion** | Pre-attentive (fastest) | Entrance animations, looping badges, hover feedback draw eye before anything else. | One looping element per view maximum; always provide `prefers-reduced-motion` fallback. |
| **Size / scale** | Very fast | Bigger = more important. Relationship between sizes matters more than absolute px. | >3 distinct sizes = chaos. |
| **Weight** | Very fast | Heavier type/strokes draw the eye. Bold a key number, not a whole paragraph. | Bold everything = bold nothing. |
| **Color / saturation** | Fast | Saturated, warm, dark = advances. Desaturated, light = recedes. | Never the *only* signal (colorblind). |
| **Contrast** | Fast | High contrast with background = prominent. | Low-contrast de-emphasis fails a11y. |
| **Spacing / whitespace** | Medium | Space *around* an element raises its importance; space *between* groups separates them. | Random px destroys grouping. |
| **Position** | Medium (subconscious) | Top, left, center get attention first in LTR. | Don't bury the CTA mid-page or right column. |

**The core mental model: emphasis is relative.** You cannot emphasize one thing — you can only de-emphasize everything else around it. Per *Refactoring UI*: the most common hierarchy fix is not making the important thing louder, it's making the supporting things quieter.

### Size

- Size is the fastest signal the brain reads. A 32px headline instantly outranks 16px body copy.
- Use **≤3 size variants** in one design (small / medium / large). Add more only when weight + color have failed you — and they usually haven't.
- Use a **modular type scale** so sizes are intentional, not arbitrary: `12, 14, 16, 20, 24, 30, 36`. Each step is visibly distinct.
- For non-text size: icon sizes, avatar sizes, card widths all follow the same "few discrete steps" discipline.

### Weight

- Three usable text weights cover almost everything: regular (400) for body, medium/semibold (500–600) for emphasis and labels, bold (700) for headings/key data.
- Weight is the cheapest way to create hierarchy *within* one font size — e.g., a table where the row label is regular and the value is semibold.
- Don't pair heavy weight *and* large size *and* high color *and* lots of space on the same element unless it's the single most important thing on screen.

### Color & saturation

- Reason about color in **HSL**, not HEX/RGB. HSL maps to perception: hue = which color, saturation = how vivid, lightness = how bright. Adjusting hierarchy = nudging S and L.
- Advancing colors (saturated, warm, dark) pull forward; receding colors (desaturated, light, cool) drop back.
- **Pure grays look dead.** Tint them: cool UIs use blue-tinted gray (`hsl(215 16% 47%)` ≈ `#64748b`); warm UIs use yellow/brown-tinted gray (`hsl(33 5% 46%)` ≈ `#78716c`).
- Color is the **last** layer of hierarchy, applied after grayscale hierarchy already works.

### Contrast

- High text-to-background contrast = prominent; lower contrast = recedes. But contrast has a hard floor (WCAG, below).
- Use **≤3 contrast levels** in a complex design. More than that flattens everything into noise.
- Example three-level text contrast on white: primary text `hsl(222 47% 11%)`, secondary `hsl(215 16% 47%)`, tertiary/metadata `hsl(215 14% 60%)` — all still ≥4.5:1 if used at body size, or used only on larger/bolder text.

### Spacing & whitespace

- **Proximity = relationship.** Tighter spacing reads as "these belong together"; looser spacing reads as "these are separate."
- Increasing whitespace *around a single element* increases its perceived importance — luxury/hero treatment.
- **Start with too much whitespace and remove until comfortable.** The result is usually right. Compact density should be a deliberate choice (dashboards, tables), not a lazy default.
- Space proportion check: ~60% structural space (section margins, section padding), ~30% component-level (card padding, list gaps), ~10% inline micro-space (icon-label gaps, tag padding). If one scale is over-populated, that tier is the problem.

### Position

- In LTR languages, the **top-left quadrant** and **top-center** carry the most weight. Logo/nav top-left, value prop top-center, primary CTA in the first viewport.
- NNG eye-tracking research consistently shows the **top and left regions** dominate first fixations in LTR layouts; faces and images attract nearby gaze; numbers/data should be in tables or graphs, not buried in prose.
- Position interacts with reading order — see scanning patterns and eye-tracking findings below.

---

## The squint test (and its digital equivalent)

The fastest hierarchy audit — associated with UX practitioners including Jared Spool and widely cited in the design community:

1. Step back ~5 feet from the screen (or zoom out) and **squint until detail blurs**.
2. Ask: *what are the darkest, most prominent blobs?* They should be — in order — the value proposition and the primary CTA.
3. **Failure signals:** the logo dominates (branding ego over business goal); the page is a uniform gray wash (no blob stands out → user doesn't know where to start); two equally-loud blobs compete (no single primary action).

**Digital equivalents** (more repeatable than literal squinting):

- **Blur filter** at 5px / 10px / 20px radius (Polypane has this built in). At 20px, only true hierarchy survives.
- **Placeholdifier**-style tools convert all text/images to solid color blocks, preserving layout — strips content meaning to expose pure structural hierarchy.
- Quick CSS check in devtools:

```css
/* Temporarily drop on <body> to run a squint test in any browser */
body { filter: blur(8px); }
```

If you can still identify the single next action with everything blurred, hierarchy is working.

---

## Scanning patterns

Scanning patterns are driven primarily by **user intent + page layout**, not by the designer alone. The *same* user switches patterns as their goal changes. Design for the pattern your content invites; never assume full reading.

### F-pattern

- Dominant on **text-heavy pages with weak formatting** (search results, articles, listings). Documented in the NNG 2006 eye-tracking study (232 users). **NNG's 2017 follow-up** clarified the F-pattern is not universal — it emerges when formatting is poor; better-formatted pages produce layer-cake, spotted, or committed patterns instead. Design to *prevent* the F (via subheads, bullets, visual variety) rather than to accommodate it.
- The eye reads the top line(s) fully, then makes shorter horizontal sweeps down the left edge, forming an F (or E).
- **Consequence:** the right side and mid-body of long text are largely ignored. Never put critical info in a right column on a text page.
- **Design for it:** front-load keywords in headings and first words of paragraphs; break text with subheads and bullets so it stops being an undifferentiated block (which is what *causes* the lazy F in the first place).

### Z-pattern

- Common on **sparse, low-text pages** (landing pages, posters). The eye travels top-left → top-right (across the header), diagonally down to bottom-left, then bottom-right.
- **Place along the Z:** logo top-left, secondary action/nav top-right, value prop on the diagonal, primary CTA bottom-right (the natural terminus).

### Layer-cake pattern

- The eye scans only **headings and subheadings**, skipping the body in between — "slicing" the page like a layer cake.
- The **most effective scanning pattern besides full reading.** Design long-form and docs specifically for it: meaningful, self-describing subheads (`How to apply X`, not `Section 3`); each section findable from its heading alone.

### Spotted pattern

- The eye jumps to visually distinct words — links, **bold**, color, numbers, bulleted items — ignoring the rest. Works only when those highlights genuinely differentiate key terms. Over-highlighting destroys it (if everything's bold, nothing's spotted).

### Exhaustive reading (motivated/committed pattern)

- Reading every word, top to bottom. Occurs **only** for trusted, high-motivation content (a contract, a terms of service they care about, a critical how-to). NNG calls this "exhaustive" or "motivated" reading; "commitment pattern" is informal shorthand. **Never design assuming exhaustive reading** — most users never reach it.

> Rule of thumb: support layer-cake and spotted scanning by default (strong subheads + meaningful highlights), front-load for F-pattern on text pages, and arrange CTAs along the Z on sparse marketing pages.

---

## Primary / secondary / tertiary actions

Button emphasis *is* hierarchy made tactile. Get the three tiers right and users always know the intended path.

| Tier | Style | Visual emphasis | Count per view |
|---|---|---|---|
| **Primary** | Filled / solid | High | **1** (the single intended action) |
| **Secondary** | Outlined (border, transparent fill) | Medium | 0–2 |
| **Tertiary** | Text-only / link-style / ghost | Low | as needed |

```html
<!-- One primary, one secondary, one tertiary -->
<button class="btn btn--primary">Create account</button>
<button class="btn btn--secondary">Continue with Google</button>
<button class="btn btn--tertiary">Maybe later</button>
```

```css
.btn { min-height: 48px; padding: 0 20px; border-radius: 8px; font-weight: 600; }
.btn--primary   { background: hsl(221 83% 53%); color: #fff; border: 1px solid transparent; }
.btn--secondary { background: transparent; color: hsl(221 83% 40%); border: 1px solid hsl(221 30% 70%); }
.btn--tertiary  { background: transparent; color: hsl(221 83% 40%); border: 1px solid transparent; padding: 0 8px; }
```

**Rules:**

- **One high-emphasis button per view** makes all others read as clearly subordinate.
- **Exceptions to "one primary" exist** but are strict: multiple primaries are legitimate only when **spatially separated and contextually distinct** (e.g., `Save` in a toolbar and `Publish` in a different page zone). Two primaries side by side = a hierarchy bug.
- **Emphasis-based systems beat fixed-color systems.** Define buttons by emphasis (Filled / Outline / Sheer, à la Carwow; or Material's filled/tonal/outlined/text/elevated) so they adapt color to the background — not `primary = blue` hardcoded, which breaks on colored backgrounds, dark mode, and for colorblind users.
- **Button groups:** fine for 2–3 actions. **Beyond 3, collapse into a menu button** to cut clutter.
- **Placement:** primary left-aligned in full-page forms; in wizards/dialogs, primary right-aligned and to the *right* of secondary/tertiary (follows completion flow). Be consistent within a product.
- **Consistency overrides local hierarchy for destructive/semantic buttons:** a red "Cancel/Delete" stays red everywhere; don't recolor it to fit a local emphasis scheme.
- **Avoid disabled buttons as a pattern** — they hurt usability and accessibility (no focus, no explanation). Prefer enabling the button and showing validation on click.

---

## Focal points & visual weight

- A **focal point** is the single element the eye lands on first. Every view should have exactly one. Create it with a deliberate spike in size + contrast + space relative to neighbors.
- **Visual weight** = how much an element pulls the eye, summed across all seven levers. Distribute weight intentionally; an accidental heavy element (a bright illustration, an oversized logo) steals the focal point.
- **Decorative backgrounds must lose to foreground content.** If a hero image/illustration competes with the CTA: desaturate it, blur it, drop it to ~20–30% opacity, or simplify it — keep the element but stop it shouting.
- **Depth via shadow scale** reinforces a "closer = more important" hierarchy: small shadow for buttons, medium for dropdowns, large for modals. Don't apply one big shadow everywhere.

---

## Eye-tracking findings applied to hierarchy

What laboratory eye-tracking studies tell us that changes design decisions (NNG unless noted):

- **Top-left / top-center of the first viewport**: the dominant landing zone for first fixations in LTR layouts. Value prop and primary CTA must live here.
- **Faces redirect attention.** Human faces — especially eyes — pull fixations strongly and redirect gaze to wherever the face is looking. Position product images or faces to direct attention toward your CTA, not away from it. A face looking off-page bleeds attention off-page.
- **Numbers attract disproportionate fixation.** A single `$49/mo` or `2,300 reviews` in a field of prose stops the eye. Surface key metrics as visual callouts (stat card, badge, table cell), never embedded mid-sentence.
- **Images are scanned before text.** Users fixate on meaningful images before nearby text. A relevant, high-quality image increases fixation on adjacent copy. A decorative or mismatched image wastes the attention advantage.
- **Right column of text-heavy pages gets almost no fixation.** The F and layer-cake patterns both terminate left of center. Never put critical CTAs, key data, or required navigation in a right sidebar on content pages.
- **First fixation ≠ comprehension.** A short (~150ms) fixation means "I saw something"; a longer one (300ms+) means "I'm reading." Design so the primary CTA receives a long fixation, not just a passing one — achieved via size, white space buffer, and contrast spike.
- **Predictive eye-tracking tools** (Attention Insight, EyeQuant, Neurons) simulate heatmaps algorithmically. Useful as a directional smoke test before user testing; not a substitute for real fixation data. Treat model scores as "does this design have a coherent focal point?" checks, not precision measurements.

---

## Fold, viewport, and scroll-driven hierarchy

- **Above-the-fold is real but not absolute.** The first viewport is the highest-stakes real estate (first impression, no scroll required), but the fold is not a cliff — users scroll when they see sufficient reason. The job of above-the-fold content is to create that reason, not to cram everything.
- **The hero must do three things**: (1) state the value proposition clearly, (2) signal to the target user "this is for you," (3) provide an obvious next action (primary CTA). If any of these three fail, scroll rate drops.
- **Scroll-triggered reveals** (entrance animations, sticky CTAs, parallax) introduce a temporal hierarchy layer. Used correctly they guide progressive disclosure; used carelessly they hide content from users who don't trigger them (e.g., fast scrollers, reduced-motion preferences). Rule: never put required information behind a scroll trigger that only fires once.
- **Sticky/fixed elements assert hierarchy by always winning position.** Nav bars, sticky CTAs, cookie banners — anything fixed to the viewport competes with the content hierarchy constantly. Minimize fixed-element count; they cost attention budget.
- **Responsive hierarchy is not optional.** A hierarchy that works at 1440px often breaks at 375px — the primary CTA may scroll off-screen, the type scale may collapse into uniform sizes, the Z-pattern may flatten to a stacked list. Verify hierarchy at each major breakpoint: does the squint test still identify the right blob?

---

## Motion as a hierarchy signal

Motion is the seventh lever — and the most powerful, because the visual cortex processes peripheral motion before conscious attention is directed. Use it sparingly.

- **Entrance animations / fade-in on scroll** draw the eye to newly visible elements. Put these on the content you want users to notice, not on decorative backgrounds.
- **Looping animations** demand continuous attention tax. Never use them on secondary/decorative elements (they'll constantly steal focus from the primary CTA). One looping element per view is the hard limit.
- **Hover/interaction feedback** (button scale, color shift, underline) confirms affordance without needing a label. Button hover states are hierarchy reinforcement: the primary button's hover should be visually heavier than the secondary button's hover.
- **Reduced-motion (prefers-reduced-motion: reduce):** always respect this. Provide static fallbacks for all motion-based hierarchy signals — don't communicate hierarchy *only* via animation.
- **Motion hierarchy rule:** if something moves, it becomes the focal point. Use this deliberately. If a notification badge pulses, it outranks everything else on screen until it stops.

---

## Design philosophy: priority orders

When levers conflict, fall back to these prioritization ladders:

- **Salesforce Lightning Design System:** **Clarity > Efficiency > Consistency > Beauty.** A clear-but-less-pretty layout beats a beautiful-but-confusing one. Jina Anne was a contributor to SLDS; the principle belongs to the system, not a single author.
- **Aaron Walter's hierarchy of user needs:** **Functional > Reliable > Usable > Pleasurable.** Earn each level before reaching for the next; don't polish delight onto something that isn't yet usable.
- **Aesthetic-usability effect:** polished visuals are *perceived* as more usable before any interaction, and visual quality directly influences trust. So aesthetics aren't optional — but they sit *on top of* a working hierarchy, not in place of it.

---

## Concrete specs & numbers

**Type sizes**
- Body copy: **14–16px** minimum (16px default for primary reading).
- Subheaders: **18–22px**. Primary headers: up to **32px**.
- Suggested modular scale: **12 / 14 / 16 / 20 / 24 / 30 / 36px**.

**Contrast (WCAG 2.1)**
- Normal text: **≥4.5:1**. Large text (≥18px regular, or ≥14px bold): **≥3:1**. Applies to button labels, form labels, and all text.
- These are floors, not targets — secondary text should be quieter but must still clear them.

**Spacing scale**
- **4 / 8 / 16 / 24 / 32 / 48 / 64px**, on a 16px base unit. Adjacent values you use should differ by **≥25%** so the difference is visible.
- Space proportion check: ~60% structural (section margins/padding), ~30% component (card padding, list gaps), ~10% inline (icon-label gaps).

**Touch targets**
- Minimum **48×48px** (Material Design 3 / Refactoring UI). (WCAG 2.5.8 sets a 24×24 CSS-px floor; Apple HIG recommends 44×44pt — 48dp is the safe practical target.)

**Variety caps**
- ≤3 size variants · ≤3 contrast levels · ≤2 primary + 2 secondary colors · ≤2 type families.

**Squint/blur radius**
- Verify hierarchy at **5 / 10 / 20px** blur.

**Background de-emphasis**
- Hero background elements at **20–30% opacity** to let foreground UI stand out without removing them.

**Eye-tracking findings (NNG and related research)**
- First impressions form in **~50ms** (Lindgaard et al. 2006, Carleton University — the canonical source; the "17ms" figure floating online lacks a credible citation, do not use it) — before reading any word.
- Fixation durations during web scanning range roughly **100–500ms**; short fixations (~150ms) indicate scanning, longer ones (300ms+) indicate content processing.
- Average page visit: **<1 minute** (NNG longitudinal data, directionally stable). Users read approximately **25% of page text** on an average visit.
- F-pattern documented across content-heavy pages in NNG 2006 (232 users); NNG 2017 updated this — the F is a symptom of weak formatting, not an immutable law.
- **Faces and human images** pull fixations strongly; anything placed adjacent to a face gets elevated attention. Use this deliberately, not accidentally.
- **Numbers and data** attract disproportionate fixation in mixed text/data layouts — supports the rule to surface key metrics visually (callout, table, chart) rather than embedding them in sentences.

---

## Tools & libraries

- **Polypane** — built-in squint-test blur filter (5/10/20px) plus an accessibility/contrast panel. Best single tool for hierarchy + a11y debugging in-browser.
- **Placeholdifier** (Chrome extension) — converts text/images to solid blocks while keeping layout; exposes pure structural hierarchy by removing content meaning.
- **Refactoring UI** (Wathan & Schoger) — the most concrete developer-facing reference for spacing scales, color scales, shadow scales, and grayscale-first hierarchy.
- **Material Design 3**, **Apple HIG**, **IBM Carbon** — design systems with explicit button-emphasis hierarchies, spacing tokens, and touch-target rules. Use when you need defensible, battle-tested defaults.
- **Nielsen Norman Group (nngroup.com)** — authoritative source for scanning patterns and eye-tracking evidence.
- **Baymard Institute** — e-commerce-specific hierarchy research (checkout, CTA placement, form labels).
- **lawsofux.com** / **Interaction Design Foundation** — psychology-backed UX laws and synthesis.
- **Contrast checkers** — WebAIM Contrast Checker, Stark (Figma), Polypane's a11y panel.
- **Figma** — Auto Layout enforces spacing scales; design tokens enforce hierarchy scales.
- **AI predictive eye-tracking** (Attention Insight, Neurons, EyeQuant) — score designs on clarity/hierarchy and flag scattered attention *before* user testing. Treat as a directional smoke test, **not** ground truth — verify with the squint test and real users.

**When NOT to reach for these:** don't run predictive eye-tracking or load a heavy design system before you've passed a manual grayscale squint test — fix structural hierarchy first, then validate.

---

## Do / Don't

**Do**
- Design in grayscale first; add color only after hierarchy works without it.
- De-emphasize secondary elements (smaller, lighter, quieter) rather than amplifying primary ones.
- Keep exactly one high-emphasis (filled) action per view.
- Front-load keywords in headings and opening sentences.
- Use a modular type scale and a spacing scale; pick values that differ ≥25%.
- Pair every color signal with a size/weight/spatial cue (colorblind safety).
- Make labels visibly closer to their input than to neighbors (Gestalt proximity).
- Run the squint/blur test before calling a layout done.
- Right-align numbers in tables; left-align most text.
- Verify hierarchy at each major breakpoint (375px / 768px / 1280px minimum).
- Respect `prefers-reduced-motion`; provide static fallbacks for all motion-based hierarchy cues.
- Use a facing image deliberately — direct the subject's gaze toward the CTA, not off-page.

**Don't**
- Don't make multiple elements filled/bold/bright on one screen ("everything is primary").
- Don't de-emphasize using contrast reduction alone — it breaks WCAG and fails low-vision users.
- Don't add a 4th type size to "fix" hierarchy that's actually a weight/color/spacing problem.
- Don't use random margin/padding values — they destroy proximity grouping.
- Don't define buttons as fixed colors (`primary=blue`) — define by emphasis so they adapt to background/dark mode.
- Don't put critical info in right columns or mid-body of text pages (F-pattern blind spots).
- Don't let a decorative background or oversized logo win the squint test.
- Don't justify text without hyphenation; don't center long-form copy.
- Don't rely on disabled buttons as a UX pattern.
- Don't assume users read — they scan ~25%.
- Don't use looping animations on secondary/decorative elements — they permanently steal the focal point.
- Don't put required content behind a scroll-trigger that only fires once.
- Don't trust predictive eye-tracking tools as ground truth; use them as smoke tests only.

---

## Common pitfalls & how to avoid them

- **Everything is primary.** Multiple filled/bold/bright elements compete → visual noise, no clear next action. *Fix:* pick one focal point and one primary button; quiet everything else.
- **Contrast-only de-emphasis.** Low-contrast gray text reads as "fine" to a young designer on a good monitor but fails older/low-vision users and WCAG. *Fix:* de-emphasize with size or weight or saturation, keeping contrast ≥4.5:1 (or ≥3:1 for large text).
- **Logo dominates the squint test.** Brand mark outweighs the CTA/value prop. *Fix:* shrink/quiet the logo until the business goal is the heaviest blob.
- **Uniform gray wash.** Similar sizes + low contrast everywhere → no entry point, users bounce. *Fix:* introduce one deliberate spike of size + contrast + space.
- **Too many type sizes.** >3 distinct sizes reads as chaos. *Fix:* collapse to ≤3 sizes; use weight/color for finer distinctions.
- **F-pattern misapplication.** Assuming users read everything, burying key info mid-page/right column. *Fix:* front-load, use subheads/bullets, keep critical info top and left.
- **Spacing inconsistency.** Random px → broken Gestalt; a label equidistant from two inputs is ambiguous. *Fix:* spacing scale + label hugs its own input.
- **Color-only button differentiation.** primary=blue/secondary=gray/tertiary=white collapses on colored backgrounds, dark mode, and for colorblind users. *Fix:* emphasis-based styles (fill/outline/text) that also vary shape and weight.
- **Decorative background vs. hero content.** Rich background steals attention from the CTA. *Fix:* desaturate / blur / 20–30% opacity / simplify.
- **Hierarchy by shouting.** Making text bigger and louder instead of moving the eye smoothly between decisions. *Fix:* hierarchy succeeds when the eye flows naturally to the next choice, not when elements yell.

---

## What real users complain about

Synthesized from practitioner forums (Reddit r/UI_Design, r/userexperience, r/UX_Design) and NNG research:

- **"AI-generated interfaces have no real hierarchy."** A recurring 2025 complaint: AI tools produce technically-correct screens with weird spacing, no intentional emphasis, and flows that don't match mental models. The screens *render* but don't *guide*. This guide exists precisely to counter that.
- **"I don't know what to click."** When there's no single primary action, users hesitate or pick wrong — classic "everything is primary."
- **"The text is too faint to read."** Direct fallout from contrast-only de-emphasis; disproportionately affects older users.
- **"It looks busy / cluttered."** Too many competing weights, sizes, and saturated colors with no de-emphasis.
- **"I couldn't find [section]."** Long-form pages without meaningful subheads defeat layer-cake scanning.
- **"The form is confusing."** Ambiguous label-to-input proximity makes users unsure which label owns which field.
- **Disabled buttons frustrate** — users can't tell *why* the action is unavailable and get no feedback on click.

---

## Senior-level checklist

- [ ] One clear focal point per view; identifiable in a 20px blur.
- [ ] Exactly one filled/high-emphasis button per view (exceptions spatially + contextually separated).
- [ ] Secondary = outlined, tertiary = text/ghost; emphasis-based, not hardcoded color.
- [ ] Hierarchy holds in **grayscale** before any color is applied.
- [ ] Every emphasis uses ≥2 levers (e.g., size + weight), never color alone.
- [ ] ≤3 type sizes, ≤3 contrast levels, ≤2 primary + 2 secondary colors, ≤2 type families.
- [ ] Type sizes from a modular scale; spacing from a 4/8/16/24/32/48/64 scale; adjacent steps ≥25% apart.
- [ ] All text meets WCAG: ≥4.5:1 (normal), ≥3:1 (large); de-emphasis stays above the floor.
- [ ] Touch targets ≥48×48px (≥24px CSS WCAG floor).
- [ ] Keywords front-loaded in headings and opening sentences (F-pattern).
- [ ] Long-form has meaningful, self-describing subheads (layer-cake) and purposeful highlights (spotted).
- [ ] Form labels visibly closer to their own input than to neighbors (proximity).
- [ ] Nav/value prop/CTA in top-left/top-center zone; nothing critical in right columns of text pages.
- [ ] Decorative backgrounds desaturated/blurred/≤30% opacity so foreground wins.
- [ ] Shadow scale matches elevation (button < dropdown < modal).
- [ ] Grays are subtly tinted (cool or warm), not pure neutral.
- [ ] No disabled-button-only flows; validation provides feedback.
- [ ] Hero does three things: states value prop, signals target audience, provides primary CTA — all above the fold.
- [ ] Hierarchy verified at ≥3 breakpoints (375px / 768px / 1280px): squint test passes at each.
- [ ] Any looping animation is on a primary/intentional element, not decorative; `prefers-reduced-motion` fallback present.
- [ ] Facing images/faces direct gaze toward the primary CTA, not off-page.
- [ ] No required content hidden behind a once-firing scroll trigger.
- [ ] Squint/blur test passed: value prop + primary CTA are the heaviest blobs; logo does not dominate.

## Sources

- Nielsen Norman Group — eye-tracking & scanning patterns, visual hierarchy, principles of visual design: https://www.nngroup.com/
- Refactoring UI (Adam Wathan & Steve Schoger) — spacing/color/shadow scales, grayscale-first hierarchy: https://www.refactoringui.com/
- Material Design 3 — button hierarchy, spacing tokens, touch targets: https://m3.material.io/
- Apple Human Interface Guidelines: https://developer.apple.com/design/human-interface-guidelines/
- IBM Carbon Design System — primary/secondary/tertiary/ghost button specs: https://carbondesignsystem.com/
- Baymard Institute — e-commerce hierarchy & CTA placement research: https://baymard.com/
- Laws of UX: https://lawsofux.com/
- Interaction Design Foundation — Visual Hierarchy: https://www.interaction-design.org/
- WebAIM Contrast Checker (WCAG 2.1 ratios): https://webaim.org/resources/contrastchecker/
- Polypane — squint-test blur filter & accessibility panel: https://polypane.app/
- Lindgaard et al. (2006) — "Attention web designers: You have 50 milliseconds to make a good first impression!" — *Behaviour & Information Technology* 25(2). Carleton University. The primary source for the 50ms first-impression finding.

# Wireframing, Prototyping, Handoff & QA

> The pipeline from idea to shipped pixels: choose the right fidelity for the feedback you need, build clickable prototypes that test real behavior, hand off intent (not just coordinates), and run design QA on staging so the build matches the spec. The senior outcome: developers never guess, every state/edge case is documented, accessibility is annotated at handoff, and defects are caught before they cost 10x.

**Apply when:** taking a design from sketch to build, writing specs, or QA'ing an implementation against a design · **Related:** [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md) · [The Website Design Process & Stages](../05-process-and-systems/design-process-and-stages.md) · [Interaction Design & UI States](../01-ux-fundamentals/interaction-design-and-states.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md)

## TL;DR — rules at a glance
- **Escalate fidelity only when the current level stops generating useful feedback.** Low-fi → mid-fi → high-fi is a ladder, not a ritual. If a known design-system pattern exists, skip straight to high-fi.
- **Low-fi = grayscale boxes + placeholder text, no grid, no brand color, no real fonts.** It tests IA, navigation, hierarchy — and signals "exploratory, don't bikeshed color."
- **If stakeholders bikeshed aesthetics instead of flow, strip fidelity back down** until structural feedback resumes.
- **Use wireflows (not static wireframes) for dynamic/SPA apps** — arrows originate from the triggering UI element, not the screen header.
- **Handoff is translation, not documentation.** Communicate *why* (intent, journey), not just *what* (px/coords).
- **Involve developers from the wireframe stage**, not at high-fi handoff — early feasibility input prevents late "this is a 2-week change" renegotiations. Hold a brief handoff sync for every non-trivial design.
- **Annotate every component state**: default, hover, focus, active, disabled, error, loading, empty, success. Missing empty/error/loading states are the most common omission.
- **Annotate `prefers-reduced-motion` fallback** for every animated component at handoff.
- **First Rule of ARIA: use the native HTML element if it gives you the semantics.** Pages using ARIA (excluding landmarks) have 34% more errors (WebAIM Million 2024).
- **Annotate accessibility at handoff**, never patch post-launch: contrast, focus order, headings, alt text, landmarks, ARIA, keyboard.
- **Agree pixel tolerance explicitly** (commonly 1–2px for rounding). What's non-negotiable: every *token* (font weight, spacing step, border color) is exact.
- **One canonical handoff file = single source of truth.** Multiple live versions = guaranteed dev confusion. Use status emoji + a changelog.
- **When the spec changes mid-build: update the changelog and notify the developer directly.** Silent file updates get missed.
- **QA on staging, never production.** Each issue = screenshot + selector + computed value + viewport + Figma frame link.
- **Test 3 breakpoints minimum: 375 / 768 / 1280–1440.** Pages correct at 1440px routinely break at 768px.
- **Defect cost grows ~10x per stage it passes through undetected.** Catch it in design review, not in prod.

## Fidelity levels — what each is for

Fidelity is a communication tool. Match it to the question you need answered.

| Level | Looks like | Tests / validates | Signals to viewer | Iteration cost |
|---|---|---|---|---|
| **Low-fi** | 2-tone gray, boxes, placeholder text, no grid, no scale, hand-sketch OK | Information architecture, navigation, content hierarchy, flow | "Exploratory — disposable, don't critique color" | ~1 day/iteration |
| **Mid-fi** | Accurate spacing & proportions, real content labels, gray palette, no brand/imagery | Layout, IA, interaction patterns, component proportions | "Structure is settling; visual design not started" | ~hours |
| **High-fi** | Real type, brand color, real imagery, all interaction states visible | Realistic usability testing, stakeholder sign-off, dev handoff | "This is what it'll look like" → realistic user behavior | higher |

**Why low-fi works:** its incompleteness is intentional. Stakeholders don't suggest color tweaks, developers know it's exploratory, designers feel free to discard ideas. High-fi looks like live software, so it triggers *realistic* behavior in usability tests — but it also invites premature aesthetic debate when shown too early.

**Rules:**
- Don't put brand color or real fonts on a low-fi wireframe — it breaks the "this is disposable" signal and pulls feedback toward aesthetics.
- Escalate when feedback dries up at the current level — not on a fixed schedule.
- **Shortcut:** if a design system exists and the pattern is familiar (e.g., a standard settings form), go straight to high-fi. The ladder aligns *understanding*; skip it when understanding is already shared.

## Wireframes vs wireflows

- **Wireframe** = static screen layout. Good for stable, page-based sites with many unique pages.
- **Wireflow** = wireframe + flow. Each step is a partial/full wireframe; **arrows point from the triggering UI element** (a button, a row, a toggle) — not from the screen header. Best for apps with dynamic/AJAX content and few unique pages: mobile apps, SPA checkouts, multi-step flows where one screen has many states.

```
[Cart screen]
  └─(tap "Checkout" button)──▶ [Address step]
       └─(tap "Continue")─────▶ [Payment step]
            ├─(valid card)─────▶ [Confirmation]
            └─(card declined)──▶ [Payment step · error state]
```

Annotate the *condition* on each arrow (valid / invalid / empty / timeout). That's the part developers actually need.

## Prototyping — making it clickable

A clickable prototype tests behavior, not just looks. Escalate from "tap-through" (static screens linked by hotspots) to "stateful" (form validation, conditional branches, component variants).

**When to escalate prototype fidelity:**
- Start tap-through. Escalate to stateful when reviewers are confused by placeholder transitions, or when a flow involves meaningful conditional branching (login → error vs. success, multi-step forms).
- Add conditional logic (Figma Variables / Axure) only when the interaction rule itself is under review — not as a standard practice for every screen.
- Use code-based tools (Framer, UXPin) only when the prototype doubles as a build artifact or requires real API data.

**What a senior prototype includes:**
- **Real flows with branches** — success *and* failure paths (declined payment, empty search, network error).
- **Component variants wired to interactions** — hover/focus/active/disabled states actually change on interaction, not just documented in a corner.
- **Conditional logic** — show/hide based on input (e.g., "if email invalid → show inline error"), using Figma Variables + conditional prototyping or Axure/Framer for heavier logic.
- **Realistic content length** — test with the longest plausible string and the shortest (1-char name, 280-char bio), not "Lorem ipsum" of convenient length.

**Interaction spec — minimum per interactive component:**
1. Initial / default state
2. Hover state
3. Focus state (keyboard)
4. Active / pressed state
5. Disabled state
6. Loading state
7. Error state
8. Success / confirmation state
9. Empty state (for lists/feeds/results)
10. Min-width, max-width behavior
11. Max content length / truncation rule

Specify **trigger → change → duration → easing** for motion. Recommended duration ranges: micro-interactions (hover, press feedback) 80–150ms; transitions (panel slide, modal open) 200–350ms; page-level transitions ≤ 400ms. Beyond 400ms feels slow. Always annotate the `prefers-reduced-motion` fallback — either instant or a simple opacity crossfade with no translate/scale.

```
Button: primary/default
  hover:    bg color/primary/500 → color/primary/600   · 150ms ease-out
  active:   scale 0.98                                  · 80ms  ease-in
  focus:    2px outline color/focus, 2px offset         · instant (no animation)
  disabled: opacity 0.5, cursor not-allowed, no hover/active
  loading:  spinner replaces label, width frozen, disabled
  prefers-reduced-motion: skip scale/color transitions; keep functional state changes
```

## Documenting states & edge cases

States are where most builds fall apart — especially AI-generated ones. **Annotate all of them in the design file.** In practice, empty states, error states, and meaningful loading states are the three most commonly omitted — they never appear in a happy-path flow, so they get skipped unless explicitly required.

**Empty state** — every list, table, feed, dashboard, and search-results screen must have a *designed* empty state: guidance text and/or a CTA, not blank space.
- First-use empty (no data yet) ≠ user-cleared empty ≠ no-results-for-filter empty. They need different copy.

**Loading state** — use **skeleton screens** that match the layout of the content to load, not a generic centered spinner, for content-heavy components. Skeletons set spatial expectations and feel faster. Reserve spinners for short, indeterminate, full-page waits.

**Error state** — every form, API call, and async operation needs a distinct error state with (a) clear message text and (b) a recovery action. "Something went wrong" with no retry is a defect.

**Edge cases to spec explicitly:**
- Longest realistic string (does the title wrap or truncate? where?).
- Zero / one / many items (singular vs plural copy; "1 item" vs "5 items").
- Slow network / timeout / offline.
- Partial failure (3 of 5 widgets loaded).
- Permission-denied / unauthorized view.
- Very large numbers (1,234,567 vs 1.2M formatting).
- Right-to-left, long German compound words, CJK line-breaking — if i18n is in scope.

See [Interaction Design & UI States](../01-ux-fundamentals/interaction-design-and-states.md) for state taxonomy and [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md) for validation patterns.

## Handoff — translating intent

Handoff communicates **why** and **what**. The "what" is auto-generated by tooling; your value is the "why."

**File hygiene (single source of truth):**
- One canonical handoff file. Duplicate old versions as clearly-labeled reference copies — never keep two live versions.
- **Status signals in page names** (emoji prefixes so devs know what's safe to build): `🏗️ In Progress` · `💭 Needs Feedback` · `🚢 Ready to Build`.
- **Systematic layer naming** — never ship `Frame 3123` or `Rectangle 8`. Use token-like names that map 1:1 to code: `color/primary/500`, `text/body/md`, `space/4`. Bulk-fix with the *Renamed* plugin.
- **Sections** group screens / states / specs and label regions for navigation.
- **Changelog** in the file documenting every update; add a **Future Plans** section so devs see what's coming and don't hard-code something about to change.
- **Loom walkthrough** for every non-trivial design; a separate deep-dive for complex interactions. Embed via the Loom plugin directly in the frame.
- **Figma Branching** — use branches for significant design changes during active development. Merge back to main only after stakeholder sign-off; this keeps the canonical file stable while dev is building from it.

**Handoff sync meeting:**
For any non-trivial design, hold a 30-minute walkthrough with the engineering lead *before* dev starts. Cover: the user journey, interaction intent, open questions, and known constraints. This is the point where feasibility issues surface cheapest. Skip for cosmetic changes or standard patterns.

**Handling design drift during development:**
When design changes after dev has started building, update the changelog, re-mark affected pages `🏗️ In Progress`, and directly notify the developer — passive file updates are missed. For significant changes, re-run the handoff sync. Document what changed and why in the changelog entry.

**Redlines & Dev Mode:**
- In Figma Dev Mode: select an element, hover an adjacent element → a labeled measurement line shows the px gap. This kills "what's the gap between these two?" — the single most common handoff question.
- Dev Mode auto-generates CSS / Tailwind / Swift / XML — but only when the design uses **auto-layout + components + variants**. A frame of absolutely-positioned rectangles produces useless output.
- Prefer spec by **token reference** over raw values: `padding: space/4` not `padding: 16px`. The developer pulls from the same token table, so future token changes propagate.

**Component annotation format:**
Attach interaction specs directly to the component in the design file using an annotation frame or sticky placed adjacent to (not overlapping) the component. Each annotation frame should contain: component name, all state variants in a grid, and the numbered interaction spec. A dedicated "Specs" page for complex flows keeps the primary canvas uncluttered. Do not bury specs in comments — they are invisible to developers scanning the canvas.

**Developer access (no paid seat needed):**
- Invite developers as **Figma Viewers (free)**: they inspect, export assets, view auto-generated code, comment, and navigate prototypes.
- **Invite developers into library files** containing master components — otherwise clicking a component link gives a 404.

**Asset export rules:**
- Icons → **SVG** (clean, no wrapper groups, no embedded transforms).
- Raster images → **WebP** (or optimized JPG/PNG) at **1× / 2× / 3×**.
- Animations → **Lottie JSON** (vector, themeable) or **MP4** (complex/video).

**Pixel vs intent:**
- Agree a **tolerance explicitly** (commonly 1–2px for rounding artifacts) before review — don't discover it mid-QA.
- Non-negotiable regardless of tolerance: every **token** is correct — font weight, spacing-scale step, border color, radius. Small cumulative deviations make a page feel "off" with no single obvious bug.
- "Pixel-perfect" is the wrong goal; **token-perfect** is the right one. A 1px gap is fine; the wrong gray-500 (vs gray-400) for every border is not.

## Responsive specs

Specify behavior, not just two static frames. A page is a system, not a desktop screenshot shrunk.

**Test/spec at three checkpoints minimum:** **375px** (mobile), **768px** (tablet), **1280–1440px** (desktop). Pages that look correct at 1440px commonly break at 768px — that's the breakpoint to scrutinize.

**Per breakpoint, document:**
- Column count / grid change (e.g., 12-col → 4-col).
- Navigation transform (horizontal nav → hamburger / bottom bar).
- What reflows vs reorders vs hides (and *why* — hiding content on mobile is a decision, not a default).
- Type scale and spacing scale at that width (prefer fluid `clamp()` — see [Fluid Typography & Spacing](../02-responsive-adaptive/fluid-type-and-spacing.md)).
- Touch-target sizing on mobile: WCAG 2.2 SC 2.5.8 minimum = 24×24 CSS px (AA); Apple HIG recommended = 44×44 pt; Material Design recommended = 48×48dp. Aim for 44px on mobile interactive elements — 24px is the floor, not the target.

```css
/* Spec as fluid behavior, not pinned widths */
.container { width: min(100% - 2rem, 72rem); margin-inline: auto; }
h1 { font-size: clamp(1.75rem, 1.2rem + 2.5vw, 3rem); }
.grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr)); gap: clamp(1rem, 2vw, 2rem); }
```

See [Responsive & Fluid Design](../02-responsive-adaptive/responsive-and-fluid-design.md) and [Breakpoints, Devices & Mobile-First](../02-responsive-adaptive/breakpoints-devices-mobile-first.md).

## Accessibility annotations at handoff

Accessibility is **designed and annotated at handoff time**, not patched post-launch. WCAG 2.2 (October 2023) is the current standard. The EU European Accessibility Act has been in force since June 28, 2025 for companies >10 employees or >€2M revenue — patching post-launch is legally as well as technically expensive. (4,605 US ADA web accessibility lawsuits were filed in 2024 per UsableNet.)

**Annotate on the design (an "a11y layer"):**
- **Heading hierarchy** — mark H1–H6 by document structure, not visual size. Don't skip levels (no H2 → H4). Exactly one H1 per page in most cases.
- **Landmark regions** — `header`, `nav`, `main`, `aside`, `footer`. Label them on the frame.
- **Focus order** — number the tab sequence; flag anything that diverges from DOM order.
- **Visible focus indicator** — every interactive element. WCAG 2.4.11 (Focus Appearance, AA in WCAG 2.2) requires the focus indicator to have ≥ 3:1 contrast change and enclose the component with a perimeter of ≥ 2 CSS px.
- **Reduced motion** — annotate any animated component with its `prefers-reduced-motion` fallback. WCAG 2.3.3 (AAA) covers this; UI best practice is to provide it regardless. Spec the fallback state in the interaction spec.
- **Alt text** — write descriptive alt for informational images; **empty `alt=""`** for decorative images (so screen readers skip them).
- **Form a11y** — `aria-describedby` for instructions/errors, `aria-invalid="true"` on invalid fields, `aria-live="assertive"` for dynamic error announcements; specify `required`, `type`, and `autocomplete` per field.
- **Contrast** — text ≥ **4.5:1**, large text (≥18pt or 14pt bold) ≥ **3:1**, non-text UI (icon, input border, focus ring) ≥ **3:1**. **~5% of people have color blindness — never use color as the only differentiator** (pair with icon/label/pattern).
- **Label in Name** (WCAG 2.5.3) — the visible label text must be part of the accessible name.

**First Rule of ARIA: if a native HTML element provides the semantics you need, use it.** Pages using ARIA (excluding landmarks) have **34% more errors** than pages without (WebAIM 2024) — ARIA is an enhancement, not a replacement. Prefer `<button>` over `<div role="button">`, `<nav>` over `<div role="navigation">`.

```html
<!-- Annotated form field (handoff spec → implementation) -->
<label for="email">Email</label>
<input id="email" name="email" type="email" autocomplete="email"
       required aria-describedby="email-hint email-err" aria-invalid="false">
<p id="email-hint">We'll only use this to send your receipt.</p>
<p id="email-err" hidden>Enter a valid email like name@example.com.</p>
<!-- On error: set aria-invalid="true", unhide #email-err, announce via aria-live region -->
```

Full reference: [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md).

## Design QA — process and format

QA is **cross-functional and runs on staging, never production.**

**Process sequence:**
1. Designer marks pages `🚢 Ready to Build` and completes the handoff sync.
2. Developer builds; does a self-review against the checklist below before requesting QA.
3. **QA pass** — on smaller teams, the designer does this; on larger teams, a dedicated QA engineer or PM. Either way, the designer is not the only reviewer of their own work.
4. Issues filed with the structured format (below). Prioritize: token deviations = P1; missing states = P1; spacing within tolerance = P3.
5. **Designer visual sign-off** — a Loom or async screen-share recording where the designer walks through the staging build against the spec, and records approval or outstanding issues in the issue tracker. Not a verbal "looks fine."
6. Only after sign-off does the feature go to release.

**Why defects cost more late:** fixing a visual bug in design takes minutes; the same change in code takes hours; in production after user reports, it takes a sprint. Treat design QA as the cheapest possible insurance.

**Issue format — every QA ticket must include:**
1. Screenshot (with the deviation marked).
2. CSS selector of the element.
3. Computed CSS value (actual) vs spec value (expected).
4. Viewport size where it occurs.
5. Figma frame link.

Export to Jira / Linear / Notion. Tools like OverlayQA capture CSS + screenshot + context and push to those trackers automatically.

**What to check (the visual/structural QA pass):**
- Tokens match: type (family, weight, size, line-height, letter-spacing), color, spacing step, radius, border, shadow.
- Spacing within agreed tolerance (1–2px); no off-by-one-token-step errors.
- All states render correctly (hover, focus, active, disabled, error, loading, empty, success).
- Responsive at 375 / 768 / 1280–1440; no horizontal scroll on mobile; no overlap or clipping.
- Real content (long names, empty data, large numbers) doesn't break layout.
- Interactions/motion match spec (duration, easing, trigger).
- Accessibility: keyboard nav works, focus visible and in order, contrast passes, headings/landmarks correct.
- Cross-browser/device sanity (Safari, Chrome, Firefox; iOS Safari is the usual offender).

**Automate what you can:** `axe-core` / WAVE for WCAG; PerfectPixel / Pixelay / Uiprobe for design-vs-live overlay; Lost Pixel (or Percy/Chromatic) for visual regression across breakpoints; BrowserStack for cross-device.

## Concrete specs & numbers
- **Contrast (WCAG 2.2 AA):** text 4.5:1; large text (≥18pt or ≥14pt bold) 3:1; non-text UI (icons, input borders, focus rings) 3:1.
- **Color blindness:** ~5% of population — never color-only.
- **Responsive checkpoints:** 375px / 768px / 1280–1440px (test all three).
- **Pixel tolerance:** commonly 1–2px for rounding artifacts — agree explicitly up front.
- **Touch targets:** WCAG 2.2 SC 2.5.8 minimum = 24×24 CSS px (AA); advisory = 44×44 CSS px (WCAG 2.5.5, AAA); Apple HIG = 44×44 pt; Material = 48×48dp.
- **Defect cost:** ~10x per stage undetected (widely cited in software quality literature; treat as directional, not a precise measurement).
- **Asset export:** icons SVG; rasters 1×/2×/3× (WebP preferred); animations Lottie JSON or MP4.
- **Motion timing:** micro-interactions 80–150ms; panel/modal transitions 200–350ms; page transitions ≤ 400ms. Always provide `prefers-reduced-motion` fallback.
- **ARIA risk (WebAIM Million 2024):** pages with ARIA (excl. landmarks) have 34% more errors than pages without.
- **WCAG SCs to cite in annotations:** 1.4.3 (contrast), 2.1.1 (keyboard), 2.4.11 (focus appearance, AA in WCAG 2.2), 2.5.3 (label in name), 2.5.8 (target size minimum), 4.1.2 (name/role/value).
- **Dropdowns:** >5 options → use an enhanced selector (searchable/grouped), not a long plain `<select>`.
- **Legal:** WCAG 2.2 is the current standard (October 2023); EU EAA has been in force since June 28, 2025 for companies >10 employees or >€2M revenue; 4,605 US ADA web accessibility lawsuits were filed in 2024 (UsableNet annual report).

## Tools & libraries
- **Figma** — primary design/prototype/handoff. Supports variables, conditional logic, math expressions in prototypes. **Use for:** most web work end-to-end.
- **Figma Dev Mode** — developer panel: measurements, auto CSS/Tailwind/Swift/XML, redlines. **Needs auto-layout + components + variants** to be useful. **Don't** expect good output from absolutely-positioned art.
- **Figma Sections / Variables / Conditional Prototyping** — file organization; realistic flows (form validation, auth, multi-state) without code.
- **Figma Branching** — branch the file for significant design changes during active development; merge only after sign-off. Keeps the canonical file stable while dev is building from it.
- **UXPin** — code-based prototyping, Spec Mode auto-specs, exports React, syncs Storybook/Git. **Use when** you want a design-to-code pipeline.
- **Axure RP** — conditional logic, dynamic content, adaptive views, unlimited variables. **Use for** complex enterprise flows needing heavy annotation.
- **Framer** — high-fidelity web prototyping, AI rapid-generation, outputs real sites. **Use for** marketing/landing prototypes that double as the build.
- **Origami Studio (Meta)** — patch-based, multi-touch, device APIs. **Use for** complex *mobile* interactions, not web.
- **Zeplin** — handoff specs with per-component version history and inline comments. Relevant for teams on established Zeplin workflows; for new projects, Figma Dev Mode covers the same ground without an additional tool.
- **Plugins:** EightShapes Specs (spec overlays) · Autoflow (arrow connectors) · Style Organizer (audit styles→system) · Renamed (bulk layer rename) · Loom Embed (videos in-frame).
- **QA / overlay:** PerfectPixel, Pixelay (opacity overlay), Atarim (annotate live sites, widely used), Uiprobe (property-level diff — verify availability before adopting). For capture-to-tracker workflows, evaluate OverlayQA or equivalent; the category is fragmented, check current tool status.
- **A11y:** axe-core (CI/CD), WAVE (browser).
- **Cross-device / regression:** BrowserStack; Lost Pixel (OSS visual regression; alt to Percy/Chromatic); Storybook (component docs + visual tests).
- **FigJam** — wireflow mapping in workshops.
- **When NOT to add tooling:** for a small site on an existing design system, Figma + Dev Mode + axe + a manual QA checklist is enough. Don't introduce Axure/UXPin for a 5-page marketing build.

## Do / Don't

**Do**
- Match fidelity to the feedback you need; strip it back when stakeholders bikeshed aesthetics.
- Involve developers from the wireframe stage; hold a handoff sync before build starts.
- Document every state and edge case (empty/error/loading especially). Include them as acceptance criteria in tickets.
- Name layers as tokens; keep one canonical file with status emoji + changelog + Future Plans.
- Use Figma Branching for significant mid-build design changes; notify dev directly on every spec change.
- Annotate accessibility (headings, landmarks, focus order, alt, contrast, ARIA, reduced-motion) at handoff.
- Spec by token reference; agree a px tolerance in writing; record a Loom walkthrough.
- QA on staging with the structured issue format; automate contrast + visual regression.

**Don't**
- Don't add brand color/real fonts to low-fi — it invites premature aesthetic critique.
- Don't climb the fidelity ladder when the pattern is already known — skip to high-fi.
- Don't escalate to stateful prototyping before the flow structure is validated.
- Don't ship `Frame 3123` / `Rectangle 8` or keep two live versions of the file.
- Don't use ARIA where native HTML suffices; don't use color as the only signal.
- Don't chase "pixel-perfect" — chase token-perfect within agreed tolerance.
- Don't QA on production; don't hand off a generic spinner as the loading state.
- Don't ship a list/table/search screen without a designed empty state.
- Don't silently update the spec mid-build — changelog + direct notification always.

## Common pitfalls & how to avoid them
- **High-fi too early** → debate stalls on color. *Fix:* start low-fi to lock IA/flow; escalate when feedback dries up.
- **Static wireframes for an SPA** → flow/conditions undefined. *Fix:* use wireflows with conditions on the arrows.
- **Handoff = a pile of frames** → devs guess intent. *Fix:* add the "why" — Loom, journey notes, changelog, Future Plans.
- **Dev Mode produces garbage CSS** → design lacks structure. *Fix:* auto-layout + components + variants throughout.
- **404 on component links** → devs not in the library file. *Fix:* invite them as Viewers to the library file.
- **States missing in build** → never specced. *Fix:* enforce the 11-item interaction-spec minimum per component; include empty/error/loading as explicit acceptance criteria in the ticket.
- **Design changes after dev started building** → dev builds the old spec. *Fix:* update changelog + directly notify developer; re-run handoff sync for significant changes; use Figma Branching to isolate changes.
- **Prototype escalated too early to stateful** → wasted design time before structure was validated. *Fix:* keep tap-through until the flow structure is agreed; add logic only when the interaction rule itself needs review.
- **Looks fine at 1440, broken at 768** → tablet untested. *Fix:* always check 768px; treat it as the danger breakpoint.
- **A11y bolted on at the end** → fails audit, legal exposure. *Fix:* annotate the a11y layer at handoff; run axe in CI.
- **"It's off but I can't say why"** → cumulative token drift. *Fix:* token-by-token QA, not eyeballing.
- **QA on production** → users see defects; no rollback. *Fix:* staging only, designer visual sign-off before release.

## What real users complain about
Synthesized from practitioner forums (r/UXDesign, Figma community, dev/QA threads):
- **Developers:** "The spacing isn't in the file, I had to guess" — missing redlines/measurements; "Which version is current?" — multiple live files; "Clicking the component gives a 404" — not in the library; "There's no empty/error state, so I made one up" — undocumented states; "The spec says 16px but the token is space/4 = 1rem, which wins?" — raw values vs tokens conflict.
- **Designers:** "They built it pixel-different and called it done" — no agreed tolerance / no QA pass; "Engineering said it'd take 2 weeks *after* sign-off" — devs not involved early.
- **QA/PM:** "Bugs found in prod that a checklist would've caught" — no staging QA; "Issues come in as 'it looks wrong' with no selector or viewport" — unstructured tickets.
- **Accessibility:** "We failed the audit weeks before the EAA deadline because a11y was never in the design" — patched-not-designed; "Screen reader reads the icon button as 'button'" — no accessible name annotated.

See [What Real Users Complain About (Synthesis)](../08-pitfalls-and-antipatterns/real-user-complaints.md).

## Senior-level checklist
**Fidelity & prototype**
- [ ] Fidelity matches the feedback needed; not over- or under-built.
- [ ] Wireflows used for dynamic/SPA flows; conditions labeled on arrows.
- [ ] Prototype covers success *and* failure branches; realistic content lengths tested.

**States & edge cases**
- [ ] Every interactive component documents the 11-item interaction spec.
- [ ] Empty / error / loading states designed (skeletons, not bare spinners; recovery action on errors).
- [ ] Long-string, zero/one/many, timeout, and large-number edge cases specced.

**Handoff**
- [ ] One canonical file; status emoji on pages; changelog + Future Plans present.
- [ ] Layers named as tokens; spec references tokens, not raw values.
- [ ] Redlines available in Dev Mode (auto-layout + components + variants).
- [ ] Devs invited as Viewers and into the library file (no 404s).
- [ ] Assets exportable: SVG icons, 1×/2×/3× rasters, Lottie/MP4 animations.
- [ ] Loom walkthrough recorded for non-trivial work; handoff sync held with engineering lead.
- [ ] Pixel tolerance agreed in writing (e.g., 1–2px).
- [ ] Design drift protocol in place: changelog updated + dev notified on any spec change after build starts.
- [ ] Figma Branching used for significant mid-build design changes.

**Responsive**
- [ ] 375 / 768 / 1280–1440 all specced; 768px scrutinized.
- [ ] Reflow/reorder/hide decisions documented with rationale.
- [ ] Touch targets: ≥ 24×24 CSS px (WCAG 2.5.8 floor); ≥ 44px recommended on mobile (Apple HIG / WCAG 2.5.5 AAA).

**Accessibility**
- [ ] Heading hierarchy (no skipped levels) + landmarks annotated.
- [ ] Focus order numbered; visible focus indicator meets WCAG 2.4.11 (≥ 3:1 contrast, ≥ 2px perimeter).
- [ ] Contrast meets 4.5:1 / 3:1 / 3:1; no color-only signals.
- [ ] Alt text written (descriptive vs empty `alt=""`); form ARIA specced.
- [ ] Native HTML preferred over ARIA wherever it provides the semantics.
- [ ] `prefers-reduced-motion` fallback annotated for every animated component.

**QA**
- [ ] QA runs on staging; dev self-review before QA; designer visual sign-off recorded (Loom or written sign-off in tracker).
- [ ] Token-by-token check (type/color/spacing/radius/border/shadow).
- [ ] All 11 interaction states verified per component (incl. empty/error/loading).
- [ ] Issues filed with screenshot + selector + computed-vs-spec + viewport + frame link.
- [ ] axe/WAVE clean; visual-regression baseline captured across all 3 breakpoints.
- [ ] Cross-browser: Chrome, Firefox, Safari (iOS Safari specifically — most common breakage point).

## Sources
- Nielsen Norman Group — https://www.nngroup.com/
- WebAIM (incl. WebAIM Million annual report) — https://webaim.org/
- W3C Web Content Accessibility Guidelines (WCAG) 2.2 — https://www.w3.org/TR/WCAG22/
- W3C ARIA Authoring Practices (First Rule of ARIA) — https://www.w3.org/WAI/ARIA/apg/
- Figma — https://www.figma.com/ · Dev Mode — https://www.figma.com/dev-mode/
- European Accessibility Act — https://ec.europa.eu/social/main.jsp?catId=1202
- Apple Human Interface Guidelines — https://developer.apple.com/design/human-interface-guidelines
- Material Design — https://m3.material.io/
- axe-core (Deque) — https://github.com/dequelabs/axe-core
- WAVE (WebAIM) — https://wave.webaim.org/
- Lost Pixel — https://github.com/lost-pixel/lost-pixel
- UsableNet ADA Web Accessibility Report (annual) — https://usablenet.com/accessibility-lawsuit-tracker

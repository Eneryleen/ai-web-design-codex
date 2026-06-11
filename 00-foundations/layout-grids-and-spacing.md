# Layout, Grids & Spacing Systems

> How to structure a page with grids, distribute space with a single repeatable scale, and align elements so the result reads as deliberate rather than accidental. Master this and your layouts gain the calm, machine-precise rhythm that separates senior work from "developer default" — without a single arbitrary pixel value.

**Apply when:** Laying out any page or component, choosing spacing/sizing values, or fixing a layout that "feels off." · **Related:** [Visual Hierarchy](visual-hierarchy.md) · [Web Typography Systems](typography-systems.md) · [Fluid Type & Spacing](../02-responsive-adaptive/fluid-type-and-spacing.md) · [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md)

## TL;DR — rules at a glance
- **Size and space everything in multiples of 8px** (8, 16, 24, 32, 40, 48, 56, 64…). Use **4px as the only half-step**. This kills "13px vs 15px" debates.
- **Line-heights divisible by 4** (4pt baseline). Font 16px → line-height 24px; font 14px → line-height 20px. Font size itself need not be on the grid; its line-height must.
- **CSS Grid for page structure (2D). Flexbox for component internals (1D).** Never use Grid to center one button; never build a 2D page with Flexbox alone.
- **Use Subgrid for card rows** so titles/descriptions/CTAs align across all cards — ~97% browser coverage (2026), fixes the #1 layout bug in ~3 lines.
- **Body text: `max-width: 66ch`** (target 50–75 CPL). WCAG caps Latin at 80 CPL, CJK at 40 CPL.
- **Internal padding ≤ external margin.** Space inside a component should be ≤ the space between components. Tight inside + loose outside = clear grouping.
- **12 columns** because it divides by 1, 2, 3, 4, 6. A long-form article often wants 6–8 columns with wider gutters instead.
- **Hierarchy via weight + color before size.** De-emphasize competitors instead of shouting louder.
- **Optical > mathematical for asymmetric shapes** (play icons, triangles). Trust eyes over rulers.
- **Units:** `rem` for font-size and breakpoints; `px` for borders; `px` for horizontal padding (doesn't shrink lines on zoom); `rem` for vertical text margins; `rem` or `px` for `gap` (prefer `rem` in component spacing so it scales with user font preference, `px` for fixed structural gutters).
- **Whitespace is the cheapest upgrade.** When cramped, add space before removing content. Most layouts add too little.
- **Name tokens semantically** (`space-200`, `--space-4`), never `space-16px` — values can be remapped, names can't cheaply.

## The grid hierarchy: Grid vs Flexbox vs Subgrid

One rule decides 90% of layout questions:

> **CSS Grid = page structure (2D). Flexbox = content flow inside a component (1D). Subgrid = align nested children to the parent's tracks.**

| Need | Use | Why |
|------|-----|-----|
| Whole-page regions (header/sidebar/main/footer) | Grid + `grid-template-areas` | Two-axis control, self-documenting |
| Nav bar, button group, card body, tag list | Flexbox | One axis, content-driven wrapping |
| Cards in a row whose innards must line up | Grid row + `subgrid` on each card | Children inherit parent track sizes |
| Centering one element | Flexbox (`place-items: center`) or `display: grid; place-items: center` | Don't over-reach for a full grid |
| Component that changes layout based on its own width | Container queries + Grid/Flex inside | 2026: >92% support; correct tool for component-level breakpoints |

Never use Grid to control a single axis inside a component — that is Flexbox's job and Grid's track syntax adds noise for no benefit.

### Page structure with named areas

```css
.page {
  display: grid;
  grid-template-columns: 240px minmax(0, 1fr);
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  min-height: 100dvh;   /* dvh: shrinks with mobile browser chrome; vh does not */
}
.page__header  { grid-area: header; }
.page__sidebar { grid-area: sidebar; }
.page__main    { grid-area: main; }
.page__footer  { grid-area: footer; }

@media (max-width: 48rem) {        /* 768px */
  .page {
    grid-template-columns: 1fr;
    grid-template-areas: "header" "main" "sidebar" "footer";
  }
}
```

Named areas are self-documenting and reorder cleanly per breakpoint — prefer them whenever raw column/row integers become hard to read. Note `minmax(0, 1fr)` instead of `1fr` to stop a flex/grid child's content from blowing out the track. Use `100dvh` (not `vh`) so the layout doesn't jump when the mobile browser address bar appears or disappears.

### Subgrid: the card-alignment fix

The most common layout bug: a row of cards where each card has its own height, so titles, descriptions, and buttons land at different vertical positions. Subgrid solves it without JavaScript by making each card inherit the row's row tracks.

```css
.card-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}
.card {
  display: grid;
  grid-template-rows: subgrid;   /* inherit row tracks from parent */
  grid-row: span 3;              /* title / body / footer */
  gap: 8px;
}
```

Now every card's title row, body row, and footer row share one height across the whole row — buttons align even when titles wrap to different line counts. Browser support (2026): Chrome 117+, Safari 16+, Firefox 71+ (rows) / 91+ (columns), Edge 117+ — ~97% global. Firefox 71 shipped row subgrid only; full column+row support arrived in Firefox 91. The sample above uses `grid-template-rows: subgrid`, which works from Firefox 71+.

### Responsive columns without media queries

```css
/* auto-fit: items stretch to fill the row; empty tracks collapse */
.gallery { display: grid; gap: 24px;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); }

/* auto-fill: keeps consistent column widths, leaves ghost tracks */
.shelf   { display: grid; gap: 24px;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); }
```

Use **`auto-fit`** when fewer items should expand to fill the row. Use **`auto-fill`** when you want column widths to stay constant and the layout to leave empty slots (e.g., to avoid one giant stretched card).

### Container queries for component-level breakpoints

Media queries respond to the viewport. Container queries respond to an element's own available width — the correct tool for reusable components that appear in different contexts (sidebar vs. main column vs. modal).

```css
.card-wrapper {
  container-type: inline-size;   /* or size for both axes */
  container-name: card;
}

@container card (min-width: 400px) {
  .card { flex-direction: row; }
}
```

Container queries have >92% global browser support (2026, Chrome 105+, Safari 16+, Firefox 110+). Prefer them over viewport media queries for self-contained components. Use viewport media queries only for layout-level shifts (number of columns, sidebar visibility).

## The 8-point spacing system

Size and space every element in multiples of **8px**: 8, 16, 24, 32, 40, 48, 56, 64, 72, 80, 96. Use **4px** as the half-step for fine adjustments (icon padding, tight inline gaps). Avoid odd base units (5, 6px) — they cause sub-pixel splitting when centering on 1.5× density Android screens.

Why it works: it removes arbitrary decisions. There is no "13px vs 15px" argument — you commit to the system and pick the nearest valid step. The system's value *is* the removal of choice.

### Spacing is relative, not absolute

Precision matters inversely to scale:
- **Small scale** (icon padding, button border, inline gap): **2px is a huge visible difference.** Budget precision here.
- **Large scale** (card width, section margin): **5px is imperceptible.** Don't agonize.

So don't nitpick a 4px difference on a 600px card; do sweat 2px on a 16px icon's inset.

### Internal ≤ external (the grouping rule)

> Padding **inside** a component should be **≤** the margin **between** components.

Tighter internal + looser external spacing makes visual groups obvious (Gestalt proximity). Concretely: if a form label sits 4px above its input, the gap *between two form sections* should be at least 24–32px. In a vertical stack, spacing *between groups* must exceed spacing *within* a group, or everything reads as one undifferentiated list.

### The 4pt typography baseline

Font sizes don't need to be on the 8pt grid, but **all line-heights must be divisible by 4**, and the spacing around text blocks should be too. This keeps text snapping to a vertical rhythm.

| Font size | Line-height | Ratio |
|-----------|-------------|-------|
| 14px | 20px | 1.43 |
| 16px | 24px | 1.5 |
| 18px | 28px | 1.56 |
| 20px | 28px | 1.4 |
| 24px (heading) | 32px | 1.33 |
| 48px (display) | 52px | 1.08 |

Baseline body line-height = **1.5 (150%)**. For long lines (>75 CPL) raise to **1.6–1.7**. For display/headline text, compress to **1.0–1.2**.

### Density tiers

The same token scale supports three density tiers — step down token values, don't break the system:

| Tier | Row gap | Cell padding | Typical context |
|------|---------|--------------|-----------------|
| Comfortable | 16–24px | 16–24px | Marketing, consumer, onboarding |
| Compact | 8–12px | 8–12px | CRM, admin tables, dashboards |
| Spacious | 32–48px | 32–48px | Landing pages, editorial, hero sections |

Switching density = swapping token values, not creating a separate component. Keep the same scale steps; just shift which step you use.

## Spacing tokens & scales

Define spacing as named tokens early, so values can be remapped later without renaming. **Semantic names outlast literal ones**: `space.200` survives a base-unit change that `space-16px` cannot.

```css
:root {
  --space-0:  0;
  --space-1:  0.25rem;  /*  4px */
  --space-2:  0.5rem;   /*  8px */
  --space-3:  0.75rem;  /* 12px */
  --space-4:  1rem;     /* 16px */
  --space-5:  1.25rem;  /* 20px */
  --space-6:  1.5rem;   /* 24px */
  --space-8:  2rem;     /* 32px */
  --space-10: 2.5rem;   /* 40px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */
  --space-20: 5rem;     /* 80px */
  --space-24: 6rem;     /* 96px */
}
```

### Atlassian spacing scale (real-world reference, base = 8px)

| Token | Value | Token | Value |
|-------|-------|-------|-------|
| space.025 | 2px | space.300 | 24px |
| space.050 | 4px | space.400 | 32px |
| space.075 | 6px | space.500 | 40px |
| space.100 | 8px | space.600 | 48px |
| space.150 | 12px | space.800 | 64px |
| space.200 | 16px | space.1000 | 80px |
| space.250 | 20px | (negative tokens −2px … −32px exist) | |

Tailwind's spacing scale uses a 4px base (1 unit = 4px): `gap-2`=8px, `gap-4`=16px, `gap-6`=24px, `gap-8`=32px. The step positions land on the same 8pt-multiples — just expressed in 4px units. Whatever the framework, the spacing system *is* the design token system — pin it before building.

## Line length, containers & gutters

### Measure (line length)

| Context | Target CPL | CSS |
|---------|-----------|-----|
| Body text (ideal) | 50–75, optimum ~66 | `max-width: 66ch` |
| WCAG SC 1.4.8 max (Latin) | 80 | `max-width: 80ch` (ceiling) |
| CJK max | 40 | `max-width: 40ch` |
| Mobile | 30–50 | — |
| Dyslexia-friendly | 45–70 | — |

```css
.prose { max-width: 66ch; }                       /* fixed optimum */
.prose-fluid { max-width: clamp(45ch, 90vw, 75ch); } /* responsive */
```

`1ch` = width of the `0` glyph in the current font. It is *not* the average character width and varies per font — test per font and add ~10% slack if needed. For a quick sanity check: at 16px body size a comfortable column typically lands around **600–700px** (roughly 38–44em); "font-size × 30" as a quick heuristic underestimates this for most Latin fonts.

### Container widths

Text-heavy pages: cap content at **800–1200px**. A 1440px canvas rarely needs all 12 columns for an article — a 6- or 8-column sub-grid with wider gutters reads calmer. Use `max-width` on a centered wrapper, not on every grid child.

### Gutters scale with viewport

| Breakpoint | Gutter |
|------------|--------|
| Mobile | 16px |
| Tablet | 24px |
| Large / XL | 24px |
| XXL | 32px |

Wider gutters on larger screens preserve visual openness rather than letting columns crowd.

## Responsive grids

### Why 12 columns

12 divides evenly by 1, 2, 3, 4, and 6 — yielding 1/2/3/4/6-column layouts from one system. A 10-column grid divides only by 1, 2, 5 (no thirds). That divisibility is why 12 won — but it's not sacred:
- **6-column**: editorial/blog articles with a sidebar.
- **5-column**: asymmetric editorial split (3+2 or 2+3).
- **4-column**: product cards, media galleries.
- **3-column**: dashboard panels that don't need sub-divisions.

Pick the column count the content needs; don't force 12 where fewer divisions serve better.

### Material Design 3 responsive grid

Material Design 3 (current as of 2026) defines four breakpoints:

| Breakpoint | Name | Columns | Gutter | Margin |
|------------|------|---------|--------|--------|
| <600dp | Compact (phone) | 4 | 16dp | 16dp |
| 600–904dp | Medium (tablet) | 8 | 24dp | 24dp |
| 905–1239dp | Expanded (large tablet/laptop) | 12 | 24dp | 24dp |
| ≥1240dp | Large (desktop) | 12 | 24dp | 24dp+ |

At very wide desktops MD3 recommends constraining the content area rather than expanding margins indefinitely.

### Tailwind default breakpoints

| Name | Min-width | Common column count |
|------|-----------|---------------------|
| (base) | 0 | 4 |
| sm | 640px | 8 |
| md | 768px | 8 |
| lg | 1024px | 12 |
| xl | 1280px | 12 |
| 2xl | 1536px | 12 |

Common pattern: 4 columns below `sm`, 8 columns `sm`–`lg`, 12 columns `lg`+.

### Fluid type & spacing with `clamp()`

```css
:root {
  --step-0: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);    /* body */
  --fluid-gap: clamp(1rem, 0.5rem + 2vw, 2rem);         /* 16→32px */
}
h2 { font-size: clamp(1.5rem, 1rem + 2.5vw, 2.5rem); }
```

`clamp(min, preferred, max)` eliminates dozens of breakpoint overrides for type scale and section spacing. See [Fluid Type & Spacing](../02-responsive-adaptive/fluid-type-and-spacing.md).

## Alignment, optical alignment & composition

### Mathematical first, optical second

Start from the grid for structure; then fine-tune by eye on asymmetric shapes. Mathematical centering fails for non-rectangular forms:
- A **play triangle** mathematically centered in a circle looks *left-heavy*; nudge it right ~⅛ of its width so the visual center of mass sits at the circle's center.
- A **triangle's** center of mass is lower than its geometric center — shift up when vertically centering.
- Letters with rounded tops/bottoms (O, C, S) **overshoot** the baseline/cap-line intentionally; that's optical, not a bug.

> Trust eyes over rulers on asymmetric shapes. Symmetric rectangles: trust the grid.

### Whitespace as active design element

Whitespace is not absence — it is structure. Rules:
- **Add before removing**: if a layout feels cramped, increase spacing before cutting content.
- **Unequal whitespace signals hierarchy**: more breathing room around a heading separates it from body text without a size increase.
- **Inter-group space > intra-group space** (Gestalt proximity): the gap between two sections must visibly exceed the gap between items within one section. If they read as equal, everything collapses into undifferentiated content.
- **Margin collapse is vertical rhythm**: let adjacent text block margins collapse to maintain a consistent baseline grid.

### Hierarchy without size inflation

Reach for **weight and color before size**:
- Bold vs regular weight separates a label from its value with zero size change.
- Full-opacity vs 50%/gray text demotes secondary content.
- **De-emphasize to emphasize:** make competing secondary elements quieter (lower opacity, lighter color, lighter weight) instead of making the primary element louder.

Overusing font size creates visual noise and a "ransom note." See [Visual Hierarchy](visual-hierarchy.md).

### Density & composition

- **Comfortable** density (consumer/marketing): generous padding, 16–24px row gaps.
- **Compact** density (CRM/data tables): 8–12px gaps, but keep the *same token scale* — just step down. See [CRM & Admin Dashboards](../03-site-types/crm-admin-dashboards.md).
- Align to a small number of edges. Every new left/right alignment edge adds cognitive cost; reuse existing ones.

### Logical properties for i18n/RTL layouts

In 2026, `margin-inline`, `padding-block`, `inset-inline-start`, and `border-inline-end` are standard and have >95% browser support. Use them for any component that may ship in RTL locales (Arabic, Hebrew, Persian):

```css
/* Physical (LTR-only) */
.card { margin-left: 16px; padding-right: 24px; }

/* Logical (LTR + RTL automatic) */
.card { margin-inline-start: 16px; padding-inline-end: 24px; }
```

For layout-agnostic code — especially design system components — prefer logical properties by default. Physical properties are fine for strictly directional decorations (a left-side accent border that is intentionally directional).

### Aspect ratio

Use `aspect-ratio` for media containers, thumbnails, and cards with images. It eliminates CLS-causing layout shifts and avoids padding-top hacks.

```css
.thumbnail { aspect-ratio: 16 / 9; width: 100%; object-fit: cover; }
.avatar    { aspect-ratio: 1;      width: 48px; border-radius: 50%; }
```

Browser support: >97% (2026). Prefer `aspect-ratio` over the `padding-top: 56.25%` trick, which is now legacy.

## When to break the grid (on purpose)

Accidental misalignment reads as a bug; intentional deviation reads as design. Three legitimate reasons to break out:

1. **Full-bleed elements** — hero images/video that extend edge-to-edge beyond the column margins.
2. **Asymmetric editorial splits** — a confident 5-7 or 4-8 column split for an article with a sticky aside.
3. **Pull-quotes / callouts** — text that breaks slightly outside the reading column to signal emphasis.

```css
/* Full-bleed child inside a centered content grid */
.content {
  display: grid;
  grid-template-columns:
    [full-start] minmax(16px, 1fr)
    [content-start] minmax(0, 66ch) [content-end]
    minmax(16px, 1fr) [full-end];
}
.content > * { grid-column: content; }
.content > .full-bleed { grid-column: full; }
```

Everything not deliberately broken stays locked to the grid.

## Concrete specs & numbers
- **Valid 8pt increments:** 8, 16, 24, 32, 40, 48, 56, 64, 72, 80, 96px. Half-steps: 4, 12, 20, 28px.
- **Rem baseline:** root = 16px → 1rem = 16px, 0.5rem = 8px, 1.5rem = 24px, 2rem = 32px. Conversion: `rem = px ÷ 16`.
- **Button heights (8pt):** 32px small · 40px medium · 48px large (meets 44px WCAG touch target minimum).
- **Touch targets:** minimum 44×44px (WCAG 2.5.5 AAA); 48×48px recommended for primary actions.
- **Avatar sizes:** 24, 32, 40, 48, 64, 80, 96px — all multiples of 8.
- **Line length:** 50–75 CPL ideal, ~66 optimum; 80 CPL WCAG max (Latin); 40 CPL CJK; 30–50 CPL mobile; 45–70 CPL dyslexia-friendly.
- **Container widths:** 800–1200px for text-heavy pages; comfortable column ≈ 600–700px at 16px body.
- **Line-height:** 1.5 baseline; 1.6–1.7 for lines >75 CPL; 1.0–1.2 for display.
- **Tailwind breakpoints:** sm 640 · md 768 · lg 1024 · xl 1280 · 2xl 1536px.
- **Gutters:** mobile 16 · tablet 24 · large/XL 24 · XXL 32px.
- **Subgrid support:** Chrome 117+ · Safari 16+ · Firefox 71+ (rows) / 91+ (columns) · Edge 117+ (~97% global, 2026).
- **Container query support:** Chrome 105+ · Safari 16+ · Firefox 110+ (>92% global, 2026).
- **`aspect-ratio` support:** >97% global (2026).
- **Logical properties support:** >95% global (2026).

## Tools & libraries
- **CSS Grid (native):** 2D layout. Use `grid-template-areas` for named regions, `fr` + `minmax()` for flexible tracks. Inspect with Chrome/Firefox DevTools grid overlay (shows lines, area names, gap sizes).
- **CSS Flexbox (native):** 1D component interiors — nav bars, button groups, card bodies, tag lists.
- **CSS Subgrid (native):** align nested grid children to parent tracks. Primary fix for multi-card row alignment.
- **CSS Container Queries (native):** component-level responsive layout. Replace viewport media queries for reusable components that appear at different widths.
- **CSS custom properties:** define `--space-*` tokens in `:root`. Atlassian's published scale is a live reference.
- **`clamp()`:** fluid type and spacing between a min and max without breakpoints.
- **Tailwind CSS:** utility-first, baked-in 4px spacing scale (`gap-4` = 16px, `gap-6` = 24px); its config *is* your token store. Best for enforcing one spacing system across a large team. See [CSS, Styling & Tailwind](../04-libraries-and-tools/css-styling-and-tailwind.md).
- **Bootstrap 5:** 12-column grid, 6 breakpoints (xs/sm/md/lg/xl/xxl), gutters via `.g-*`. Useful when you need its form/component ecosystem; don't pull it in for grid alone — that's 6 lines of CSS Grid.
- **Figma:** 8px grid overlay, auto layout (Flexbox-in-the-tool), variable-based tokens that sync to code. **Tokens Studio** plugin maps tokens to CSS custom properties.
- **When NOT to:** don't pull in a grid library just for a grid you can write in 6 lines of CSS Grid; don't use Grid for a single centered button.

## Do / Don't
**Do**
- Snap every size and gap to the 8pt (4pt half-step) scale.
- Use Grid for the page, Flexbox inside components, Subgrid for aligned card rows.
- Use container queries for component-level breakpoints; viewport media queries for layout-level shifts.
- Cap body text at `66ch`; widen gutters as the viewport grows.
- Define semantic spacing tokens before building; reuse them everywhere.
- Make internal padding ≤ external margin to group visually.
- Optically adjust asymmetric icons after grid placement.
- Add whitespace before assuming the layout needs less content.
- Use `aspect-ratio` for media containers; drop the `padding-top` hack.
- Use logical properties (`margin-inline`, `padding-block`) for any component that may ship in RTL locales.
- Use `100dvh` (not `100vh`) for full-height layouts on mobile.

**Don't**
- Invent off-scale values (13px, 15px, 5px gaps).
- Build 2D page layouts with Flexbox alone, or center one button with a full Grid.
- Let card titles/CTAs misalign across a row — use Subgrid.
- Use `px` for font-size or breakpoints; use `rem` instead.
- Use `rem` for horizontal padding (it shrinks text-column width on zoom); use `px` instead.
- Reach for a bigger font when weight/color could establish hierarchy.
- Break the grid by accident — deviate only for a stated reason.
- Name tokens after their literal value (`space-16px`).
- Use `100vh` for full-height layouts — address-bar appearance on mobile causes jank.

## Common pitfalls & how to avoid them
- **`px` font-size breaks accessibility:** a user's 20px browser default is ignored by `font-size: 14px`. Use **`rem`** for all text.
- **`px` media queries break at large default font:** a user at 32px default gets the cramped desktop layout. Use **`rem`-based breakpoints** so the layout collapses to mobile as text fills the space.
- **Horizontal `rem` padding shrinks line width on zoom:** `padding: 1.5rem` grows with font size, shortening lines. Use **`px` for horizontal padding** (Josh Comeau's rule); `rem` for vertical text margins.
- **Card rows without Subgrid:** independent card heights misalign titles/CTAs. `grid-template-rows: subgrid` on each card fixes it in ~3 lines.
- **`100vh` on mobile:** the browser address bar causes jank when it shows/hides. Use `100dvh` (dynamic viewport height) instead.
- **Grid-syntax avoidance:** devs revert to Flexbox-for-everything because of redundant property names. Learn `grid-template-areas` + `minmax()` once and the friction disappears.
- **Odd base units (5/6px):** cause sub-pixel rendering when centering on 1.5× density screens. Stick to 4/8.
- **Media queries for component breakpoints:** a card layout that adapts when in a narrow sidebar needs container queries, not `@media (max-width: 600px)`. The component doesn't know its context; its container does.
- **`ch` assumed exact:** `1ch` = the `0` glyph, not the average char; add ~10% slack or test per font.
- **`.ad`/`.ads`/`.advertisement` class names:** ad blockers hide those elements. Name them `.promo`, `.banner`, etc.

## What real users complain about
- **"Cards in a grid don't line up"** — the canonical complaint, now solvable with Subgrid; before it, devs hacked equal heights with JS or fixed pixel heights that broke on wrapping.
- **"Text is too wide to read"** — full-width prose on large monitors (no `max-width`) is a frequent readability gripe; lines past 80 CPL force the eye to hunt for the next line.
- **"Zooming in ruins the layout"** — `px` font-size plus `rem` horizontal padding produces cramped lines and overflow at 150–200% zoom.
- **"Layout shifts on mobile when scrolling"** — `100vh` layouts jump when the mobile browser chrome appears or disappears; fix with `100dvh`.
- **"Component looks wrong in the sidebar"** — media queries respond to viewport width, not component width; fix with container queries.
- **"Tailwind/utility classes are noise"** vs **"raw CSS is inconsistent"** — the real fix either way is *one* enforced spacing scale; the tool is secondary.

## Senior-level checklist
- [ ] Every margin, padding, and size is a multiple of 4/8px (or a token that resolves to one).
- [ ] Spacing exists as named semantic tokens, not raw values scattered in components.
- [ ] Page structure is Grid; component internals are Flexbox; aligned card rows use Subgrid.
- [ ] Container queries used for component-level breakpoints; viewport media queries only for layout-level shifts.
- [ ] Body text columns are capped near `66ch` (≤80 CPL Latin / ≤40 CPL CJK).
- [ ] Line-heights are divisible by 4 and follow the 1.5 baseline (1.6–1.7 for long lines).
- [ ] Font sizes and breakpoints use `rem`; borders and horizontal padding use `px`.
- [ ] Internal padding ≤ external margin throughout; group spacing > within-group spacing.
- [ ] Gutters and column counts adapt per breakpoint (4 → 8 → 12).
- [ ] Asymmetric icons/glyphs are optically adjusted after grid placement.
- [ ] Hierarchy comes from weight/color first, size last.
- [ ] `aspect-ratio` used for all media containers; no `padding-top` hacks.
- [ ] Logical properties used for components that may render in RTL locales.
- [ ] Full-height layouts use `100dvh`, not `100vh`.
- [ ] Any grid break is intentional (full-bleed, editorial split, or pull-quote) — nothing misaligns by accident.
- [ ] Layout verified at 200% zoom and with an enlarged browser default font without overflow or cramped lines.

## Sources
- Refactoring UI — Steve Schoger & Adam Wathan (spacing-is-relative, de-emphasize-to-emphasize, whitespace tenets): https://www.refactoringui.com/
- Josh W. Comeau — units (px vs rem), subgrid, layout patterns: https://www.joshwcomeau.com/
- Atlassian Design System — spacing tokens: https://atlassian.design/foundations/spacing/
- Material Design 3 — responsive layout: https://m3.material.io/foundations/layout/understanding-layout/overview
- Tailwind CSS — spacing scale & breakpoints: https://tailwindcss.com/docs/responsive-design
- WCAG 2.2 SC 1.4.8 Visual Presentation (line length): https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation.html
- MDN — CSS Subgrid: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid
- MDN — CSS Container Queries: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries
- MDN — CSS Logical Properties: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values
- Can I use — Subgrid, Container Queries, aspect-ratio: https://caniuse.com/
- Kevin Powell — CSS grid/flexbox/subgrid tutorials: https://www.youtube.com/@KevinPowell
- Baymard Institute — readability & line length research: https://baymard.com/

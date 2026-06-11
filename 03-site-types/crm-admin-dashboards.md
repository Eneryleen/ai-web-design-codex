# CRM & Admin Dashboards (Data-Dense UI)
> How to design and build data-dense internal tools — CRMs, admin panels, analytical dashboards, and large data tables — where users spend hours per day and efficiency is a hard requirement, not a polish item. Senior outcome: interfaces that stay legible and fast at 50+ rows × 20 columns, give every number actionable context, and respect power-user workflows (keyboard, pinning, saved views) without overwhelming casual users.

**Apply when:** building internal tools, admin panels, CRMs, or analytical dashboards with high information density · **Related:** [SaaS Web Apps](./saas-web-apps.md) · [Cognitive Load & Progressive Disclosure](../01-ux-fundamentals/cognitive-load-progressive-disclosure.md) · [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md) · [Playbook: Design a CRM / Admin Dashboard](../09-playbooks-and-checklists/playbook-crm-dashboard.md)

## TL;DR — rules at a glance
- **Density-first, not mobile-first.** Session >30 min = analytical work → optimize for density, multi-panel comparison, keyboard. Session <2 min = mobile-first is fine. Don't strip density from analyst tools.
- **Every KPI needs context.** A raw number is noise. Show comparison (prior period), benchmark (target), and trend (arrow/sparkline). Missing baseline = useless card.
- **Top row: 3–5 primary KPIs max.** More than 5 and attention falls off hard left-to-right. 5–7 primary metrics per dashboard total; 20+ = analysis paralysis.
- **Encode quantities in length (bars) and 2D position (lines).** These are preattentive. Avoid pie/donut/gauge/radar/treemap for comparison — angle and area are not judged accurately. Never 3D.
- **Tables: left-align text, right-align numbers, tabular figures.** Match header alignment to content. Never center-align data.
- **Sticky header + frozen first column** is the highest-ROI feature in any large table. Add it before anything else.
- **Bulk actions in a floating bottom bar**, never in the top toolbar (top toolbar is already cluttered — actions buried there are missed). Checkboxes on hover, actions after selection. Always undo/confirm for destructive bulk ops.
- **3 edit-friction levels:** inline (low-stakes) → expandable row (medium) → modal (high-stakes / side effects).
- **Persist user table state** (column order/width/visibility/freeze, density, sort, filters) in `localStorage`, with reset-to-default.
- **Design all three of: empty, loading, error states.** Empty search is never blank — offer "clear filters" + next-best options. Errors get a "Try again" action, not "Something went wrong."
- **Loading pattern by duration:** skeleton (structured content) · spinner (1–10s bounded action) · progress bar (measurable 10s+, never freeze at 99%) · optimistic UI (low-stakes reversible).
- **Pagination > infinite scroll** for 200+ row enterprise datasets (predictable return-to-record, reliable bulk select).
- **Color is never the sole signal.** ~4.5–8% of users are colorblind. Add patterns/hatching, dashed lines, direct labels, icons. Prefer blue/orange over green/red.
- **Role-aware views.** Same dataset, different default presentation per role (IT admin → uptime/errors; HR → headcount/attrition).
- **Keyboard support is mandatory** in dense interfaces. Mouse-only = major productivity loss for power users.
- **Row virtualization at 2,000+ rows.** Rendering all DOM rows kills layout performance. Pair TanStack Virtual with your table engine.
- **Saved views are a tier-1 power feature.** A named snapshot of filters + columns + sort. Store server-side (not just localStorage). Encode active filters in the URL.
- **Global search ≠ column filters.** Search queries across all fields/entities. Filters narrow a structured result set. Different entry points, different mental models.
- **Per-row actions:** primary action on name/identifier click; ≤3 inline icon buttons on hover; overflow the rest to a kebab menu.
- **Select-all across pages must be explicit.** Checking the page header checkbox does not mean "all 10,000 records" — show the scope and offer a secondary CTA.

## Density vs. clarity: the core tension

Enterprise users live in one UI 6–8 hours/day. Every extra click compounds across weeks. The goal is **high density with strong hierarchy** — pack information tightly using strict grids and typographic hierarchy so it reads as organized, not cluttered.

**The two failure modes:**
- *Data eyeball attack* — a wall of technically-impressive data with no hierarchy. Non-analysts flee.
- *Dribbble template* — beautiful with 3 clean rows; collapses at 50 rows × 20 columns. Columns overflow, actions vanish, layout breaks.

**Use the session-duration test to pick a strategy:**

| Avg session | User intent | Design strategy |
|---|---|---|
| < 2 min | Glance / single action | Mobile-first, big targets, minimal density |
| 2–30 min | Operational tasks | Balanced; responsive grid, moderate density |
| > 30 min | Analytical work | Density-first: multi-panel, keyboard shortcuts, comparison views |

**Progressive disclosure is the density valve.** Start with a high-level overview; provide cheap paths to more granularity (hover, expand row, drill-down). Hide secondary detail behind hover/expand rather than showing everything at once. Showing too much *trains users to expect meaning from everything present* — which lowers signal-to-noise and destabilizes them. Show less by default; let them ask for more. See [Cognitive Load & Progressive Disclosure](../01-ux-fundamentals/cognitive-load-progressive-disclosure.md).

## Dashboard layout & scanning

Users scan in **F- and Z-patterns**; top-left gets the most attention. Lay out accordingly:
- **Primary KPIs upper-left**, global/headline numbers first.
- Detailed breakdowns flow toward lower-right.
- The **5-second test**: a user must grasp the key insight in under 5 seconds. If they can't, you have too many elements or no hierarchy.

**Layout skeleton (desktop):**
```
┌──────────┬──────────────────────────────────────────┐
│          │  [KPI 1] [KPI 2] [KPI 3] [KPI 4]   ← row1 │
│ sidebar  │  ┌─────────────────┐ ┌────────────────┐   │
│ 240–300  │  │ primary trend   │ │ secondary chart│   │
│          │  │ (line, length)  │ │                │   │
│ 5–7 nav  │  └─────────────────┘ └────────────────┘   │
│ items    │  ┌──────────────────────────────────────┐ │
│          │  │ data table (sticky header, frozen col)│ │
└──────────┴──────────────────────────────────────────┘
```

**Cognitive overload is the silent killer.** Too many charts is the hidden SaaS mistake that quietly reduces team efficiency — often without anyone blaming the interface. Cap it: 5–7 primary metrics per dashboard. If you need more, split into multiple dashboards or role-scoped views.

### Scannability: the mechanics
Dense UIs live or die by consistent visual rhythm. Specifics:
- **Typographic scale:** use ≤3 font sizes in a table view. Suggested: 11–12px for secondary/meta labels, 14px for row data, 16–18px for KPI values. Never mix proportional and tabular figures in the same column.
- **Row separator opacity:** 1px at 10–15% opacity (e.g. `rgba(0,0,0,0.10)`) — visible enough to separate rows, not so heavy it adds visual noise. Too dark = prison bars.
- **Column spacing:** minimum 12px horizontal padding per cell (16px preferred). Never let content touch cell borders.
- **Group related columns** visually: use a slightly heavier separator (2px or a header-row background shift) between logical column groups (e.g. "Contact Info" | "Deal Stage" | "Activity"). Don't use full vertical gridlines throughout — they're visually noisy.
- **Whitespace is structural.** 24–32px padding between KPI card groups and charts gives the eye a rest between signal clusters. Cramming cards edge-to-edge is a legibility failure, not a density win.

### KPI cards
Every KPI card must answer "is this good or bad, and which way is it moving?" Minimum anatomy:
- **Value** (large, tabular figures).
- **Comparison** — vs. prior period (`+12.4% vs last month`).
- **Trend** — arrow and/or sparkline.
- **Benchmark/target** when one exists (`72% of $1.2M goal`).

```html
<article class="kpi">
  <h3 class="kpi__label">Monthly Recurring Revenue
    <button aria-label="What is MRR?" title="Recurring revenue normalized to a month">ⓘ</button>
  </h3>
  <p class="kpi__value">$248,310</p>
  <p class="kpi__delta kpi__delta--up">▲ 12.4% <span>vs. last month</span></p>
  <svg class="kpi__spark" aria-hidden="true"><!-- 30-day sparkline --></svg>
</article>
```
Explain every acronym/jargon term via tooltip — users should read the data, not decode the interface.

## Choosing charts (preattentive encoding)

Preattentive processing research establishes that the visual encodings humans judge fastest and most accurately for *quantities* are **length** and **2D position**. Build around them.

| Question | Use | Why |
|---|---|---|
| Compare values across categories | **Bar chart** (length) | Length is preattentive; exact comparison |
| Trend over time | **Line chart** (2D position) | Position over time reads instantly |
| Part-to-whole, ≤5 parts, rough | Bar (stacked) or **single 100% bar** | More accurate than pie |
| Distribution | Histogram / box plot | — |
| Correlation | Scatter plot | Two positional axes |

**Avoid for quantitative comparison:** pie, donut, gauge, radar, treemap — angle and area are *not* judged accurately. **Never** use 3D variants — they distort shape and misalign data. If a pie is unavoidable: ≤6 slices, 3–5 ideal.

**Chart hygiene:**
- **2–3 lines per trend chart; 3–4 series max** per chart. Beyond that, split.
- **Direct-label trend lines** at the line end — beats a legend (legends force back-and-forth eye movement to decode).
- **Consistency law:** if red = negative in one chart, red means negative in *every* chart. Encode one meaning per color, dashboard-wide.
- **≤1000 data points per chart** for interaction performance; downsample/aggregate beyond that.
- **Never color-only.** Add hatching/texture patterns and dashed/dotted line styles so series are distinguishable in grayscale and for colorblind users.

## Data tables (the heart of admin UI)

Five structural decisions to make *before* styling any table:
1. **Column hierarchy** — which columns are in the default view (do user research; don't guess defaults and force users to constantly hide irrelevant columns).
2. **Fixed headers + frozen columns.**
3. **Filtering built around real user workflows**, not a generic filter on every column.
4. **Density control.**
5. **Mobile strategy.**

### Alignment & typography
- **Left-align** text columns. **Right-align** numeric/quantitative columns. **Match header alignment to its column.** Never center-align data.
- **Exception:** left-align *qualitative* numbers — dates, postal codes, phone numbers, IDs — even though they look numeric.
- Use **tabular/monospace figures** for numbers so decimals align and columns scan vertically:
  ```css
  .num { font-variant-numeric: tabular-nums; text-align: right; }
  ```
- **Vertical alignment:** center rows with ≤3 lines of content; top-align rows with 4+ lines.

### Row division & density
Prefer the lightest divider that works. Hierarchy of techniques, best to worst:
1. **Line divisions** — single 1px light-grey horizontal rule (default choice).
2. **Zebra stripes** — only when scanning very wide rows; risky (see below).
3. **Card approach** — for low-density, grouped data.
4. **Free form** — only for very low-density data.

**Zebra-stripe trap:** combined with hover, selected, disabled, and error states, stripes create **5+ competing grey semantic levels** that visually conflict. Default to 1px line divisions; reach for stripes only on genuinely wide rows.

**Density control** (let users choose, persist it):
```
Condensed  → 40px row height
Regular    → 48px row height   (default)
Relaxed    → 56px row height
```
Column separators: **max 1px, light grey only.**

### Sticky headers & frozen columns
The single highest-ROI investment in large tables. Without them, a 500-row table forces constant scroll-back to re-read headers.
- **Freeze the header row** always.
- **Freeze the leftmost column** (the identifier) during horizontal scroll. Freeze the **rightmost** too if it holds totals/actions.

```css
thead th { position: sticky; top: 0; z-index: 2; background: var(--bg); }
.col-frozen { position: sticky; left: 0; z-index: 1; background: var(--bg); }
.col-frozen.is-header { z-index: 3; } /* corner cell stays above both */
```

### Sorting
- **Default sort = most useful first**, determined by user research: most recent entries, OR entries needing action (lowest inventory, urgent priorities). Don't guess.
- Support **multi-column sort** for power users (shift-click to add a secondary key).
- Animate the sort arrow 180° flip over ~200ms ease-in-out.

### Global search vs. column filters
These solve different problems — don't conflate:
- **Global search bar** (top of page, `Cmd/Ctrl+K`): searches across all text fields simultaneously. Returns cross-entity results (contacts + deals + companies). Show what entity type each result is.
- **Column/facet filters**: narrow a result set by structured field values (status = "active", date range, assigned user). These are additive/conjunctive — each applied filter further reduces the set.
- **Hybrid:** many CRMs put a search input inside the filter panel for quick text filter on a specific column. That's fine; label it clearly ("Filter by name…" vs. "Search everything…").

### Filtering
- Build filters around **workflows**, not "a filter per column."
- **Faceted search**: 5–7 facets per results page is optimal. More than 7 → group into sections or put overflow behind "More filters."
- **Filter chips** must clearly show what's applied, and be individually removable. Show count of filtered results updating in real-time as filters change.
- For large filter lists: **search-within-filter** and **deselect all** are mandatory.
- **Filter persistence**: apply filters immediately (no "Apply" button needed for single-value filters); show a "Clear all" link when any filter is active. Persist active filters in the URL so the view is shareable and bookmarkable.

### Column management
Tuck behind a **"table settings"** / column dropdown:
- **Freeze/pin**, **reorder** (drag-and-drop), **resize** (drag handle revealed on hover), **show/hide**.
- **Persist** column visibility, order, width, freeze, density, and sort **per user** in `localStorage`. Always offer **reset to default**.

```js
const KEY = 'table:contacts:v1';
const saveView = (v) => localStorage.setItem(KEY, JSON.stringify(v));
const loadView = () => JSON.parse(localStorage.getItem(KEY) ?? 'null') ?? DEFAULT_VIEW;
// versioned key (v1) so a schema change can safely ignore stale saved state
```

**Column resize UX:** drag handle appears on hover at column border (cursor: `col-resize`). Double-click to auto-fit to content width. Set a min column width (≥60px) to prevent columns from collapsing to unreadable widths.

### Saved views
A saved view = a named snapshot of: active filters + column set (visibility, order, widths) + sort. This is a tier-1 power-user feature in any CRM or admin tool.

- Let users **name, save, and switch** between views ("My open deals", "Overdue this week", "EU accounts").
- Provide **system defaults** (role-scoped) that cannot be deleted; allow user-saved views on top.
- Show the active view name in the table header; indicate "modified" state when the user has unsaved changes to a view.
- Let users **share views** by URL (filters encoded in query string) for team alignment.
- Store user views in the backend (not just `localStorage`) so they survive device switches.

### Inline edit & edit friction
Three levels — pick by stakes:
| Friction | Pattern | Use for |
|---|---|---|
| **Low** | Click-to-edit inline | Simple data, few columns, room for save/cancel |
| **Medium** | Expandable row edit | A handful of related fields |
| **High** | Modal | Complex multi-field, or high-stakes data with side effects |

Use the modal deliberately for high-stakes edits — its friction prevents accidents. Always show explicit save/cancel for inline edits.

**Inline edit validation:** validate on blur (not on every keystroke). Show the error inline beneath the cell — never a toast for inline edit errors. On invalid submit attempt, keep focus in the cell and show what's wrong (e.g. "Must be a valid email"). Allow Esc to cancel and discard changes.

### Row actions (per-row)
Every row needs a clear action path. Two patterns:
- **Hover-reveal inline actions** — show 1–3 icon buttons (edit, delete, open) on row hover, at the right edge. Keep to 3 max; more → overflow menu (kebab/ellipsis).
- **Kebab menu ("…")** — single trigger that expands a contextual menu of actions. Use when there are >3 actions or when the actions vary by row state.

**Rule:** primary action (e.g. "Open") should be reachable by clicking the row name/identifier itself, not only through the action menu.

### Bulk actions
- **Checkboxes appear on row hover**; bulk actions appear **only after selection**.
- Use an **indeterminate checkbox state** (`indeterminate` attribute in HTML) in the header when some but not all rows are selected — a checked header that means "some selected" is a common bug.
- Put bulk actions in a **floating action bar fixed to the bottom of the window** — *not* the top toolbar. The top bar is already cluttered; new actions there have terrible discoverability and users miss them.
- **Always provide Undo or a confirmation** for destructive bulk ops (delete/archive). Accidentally deleting 50 records with no recovery is a top complaint.
- **Export** (CSV, XLSX) belongs in the bulk action bar, scoped to what's selected (or the full filtered set when nothing is selected). Show row count in the export label ("Export 247 rows").

```
┌──────────────────────────────────────────────┐
│  ✔ 23 selected   [Export] [Tag▾] [Delete] [×] │  ← fixed bottom bar
└──────────────────────────────────────────────┘
```

**Select-all across pages — a critical edge case.** When a user checks the header checkbox on page 1 of 400 pages, clarify scope immediately:
```
[✔] 25 of 10,482 rows selected.   Select all 10,482 rows?
```
Provide a secondary "select all across all pages" CTA. Silently scoping bulk ops to the visible page only (without telling the user) is a top complaint.

### Truncation
- **IDs/codes:** truncate the *middle* (`a1b2…9z0`) + copy-on-click.
- **Names/titles:** truncate end with ellipsis + tooltip on hover.
- **Descriptions:** clamp to 2 lines.
```css
.clamp-2 { display:-webkit-box; -webkit-line-clamp:2; -webkit-box-orient:vertical; overflow:hidden; }
```

### Pagination vs. infinite scroll
For enterprise datasets (200+ rows), **pagination wins**:
- Predictable navigation to a known location ("page 7").
- Users keep their place and can return to a specific record.
- Bulk select stays sane. Infinite scroll makes users lose their place and breaks predictable bulk actions.

**Pagination UI minimums:** show page N of M, first/prev/next/last controls, a page-size selector (25 / 50 / 100 rows), and total record count. Always show total count — "1,482 contacts" orients the user before they filter.

### Large tables: row virtualization
For 2,000+ row datasets rendered in a single scrollable container, **virtual rendering is mandatory** — rendering all rows tanks layout performance. Only the visible rows (+ a small buffer) are in the DOM; the rest are substituted with measured spacers.
- Use **TanStack Virtual** or equivalent. Pair with TanStack Table's `getCoreRowModel`.
- Keep row heights consistent (or pre-measure) to avoid scroll-position jitter.
- Virtual scroll is compatible with sticky headers; ensure z-index stacking is correct.
- Virtualization breaks "select all" by page boundary — all-rows selection must operate on the data array, not DOM rows.

## Staleness & live data
Dashboards often mix data refreshed at different frequencies (real-time metrics alongside daily-batch reports). Stale data presented without a freshness signal destroys trust.

- Show a **"Last updated" timestamp** per widget or per data source, not just one global page timestamp.
- **Visual staleness cue:** if data is older than a defined threshold (e.g. >1 hour for near-real-time feeds), add a visible warning badge on the widget.
- **Auto-refresh:** provide a configurable polling interval (or SSE/WebSocket push) with a user-visible "Refresh" button. Never silently stale.
- **Partial staleness:** if only one widget failed to refresh, show the error on that widget — not a full-page error.

## Sidebar navigation
- **Width:** 240–300px expanded; 48–64px collapsed (icon-only).
- **Collapse animation:** 200–300ms ease.
- **Primary nav items: 5–7 max.** More than that is an information-architecture problem, not a design problem — regroup, nest, or add a secondary nav. See [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md).
- **Below 768px:** hide the sidebar; collapse to a drawer/hamburger.
- Show active state clearly; keep icon + label in the expanded state (icon-only collapsed state needs tooltips).

## Empty, loading & error states
These are the most-skipped states in generated and template-built dashboards — yet every shipped product encounters all three within its first week of real use. Design all three.

### Empty states
Three distinct kinds — don't conflate:
- **First-use empty** ("no data yet") → explain what will appear + a primary CTA to create the first record.
- **Empty search/filter results** → **never truly empty.** Show next-best options, related content, or a prominent **"Clear filters"** CTA, plus what was searched.
- **Cleared/all-done** (inbox zero) → positive confirmation.

### Loading states — pattern by context
| Pattern | When | Notes |
|---|---|---|
| **Skeleton screen** | Content with predictable structure (tables, cards) | Lower *perceived* load time vs. spinner; conveys structure is incoming |
| **Spinner** | Bounded single action, 1–10s | Don't use for whole-page structured content |
| **Progress bar** | Measurable operations 10s+ | **Never freeze at 99%** — it destroys the trust the bar was building |
| **Optimistic UI** | Low-stakes reversible (toggle, like, star) | Update immediately, reconcile on response |

**Do NOT use skeletons for form submissions** — a skeleton implies *content is arriving*, not a *confirmation* state. Use a spinner/disabled button + success/error.

### Error states
Always: **clear message + recovery action.** "Something went wrong" alone is a non-answer.
```
⚠ Couldn't load contacts.
   The request timed out after 30s.
   [ Try again ]   [ Contact support ]
```
Show *what* failed, *why* if known, and the *next action*.

## Responsive strategy for data-dense UI
Mobile-first is the wrong default for analytical dashboards — it strips the density analysts depend on. Instead, define an explicit per-breakpoint downgrade:

| Breakpoint | Layout | Tables |
|---|---|---|
| **< 768px** (mobile) | 3–4 critical metrics only, vertical stacking; sidebar → drawer | Stacked cards (label:value pairs) or horizontal scroll with frozen first col |
| **768–1024px** (tablet) | 2-column KPI grid | Horizontal scroll + frozen first column |
| **> 1024px** (desktop) | 3–4 column full grid | Full table, all columns |

**Responsive table options** (pick per use case): horizontal scroll with a frozen identifier column (preserves the grid); collapse-to-cards (each row becomes a label:value card); or a priority-columns approach (hide low-priority columns first, expand for detail). Don't try to cram 20 columns into 360px.

## Role-based views
The same dataset should be **presented differently per role**:
- *IT admin* → uptime, error rates, latency, queue depth.
- *HR manager* → headcount, attrition, time-to-hire.
- *Sales rep* → my pipeline, my tasks, my quota.

Implement as role-scoped default dashboards + default table column sets + default filters. Let users customize from there (and persist it). This is the cleanest way to cut overwhelm without hiding data anyone genuinely needs.

## Keyboard & power-user support
Power users in enterprise contexts rely on the keyboard. Mouse-only is a major productivity loss. Provide:
- **Arrow-key cell/row navigation**, Enter to open, Esc to cancel.
- **`/` or `Cmd/Ctrl+K`** for search/command palette.
- **Shortcuts for frequent actions** (e.g. `e` edit, `x` select row, `j/k` move).
- **Saved filter presets** ("My open deals") and **saved views**.
- Keep all interactive elements reachable via Tab; visible focus rings (see [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md)).

## Concrete specs & numbers
- **KPI top row:** 3–5 cards max; **5–7 primary metrics** per dashboard total; 20+ = analysis paralysis.
- **Sidebar:** 240–300px expanded; 48–64px collapsed; collapse animation 200–300ms ease; 5–7 primary nav items; hidden below 768px.
- **Row heights:** condensed 40px · regular 48px · relaxed 56px. Column separator ≤1px, 10–15% opacity.
- **Cell padding:** 12px horizontal minimum (16px preferred); 8px vertical for condensed, 12px for regular.
- **Sort arrow flip:** ~200ms ease-in-out (180°).
- **Chart limits:** 2–3 lines per trend chart; 3–4 series max; pie slices ≤6 (3–5 ideal); ≤1000 points/chart.
- **Performance budget:** initial load <2s; interaction response <100ms; virtualize at 2,000+ rows.
- **Typography:** table data 14px min; KPI values 16–18px; secondary labels 11–12px; tabular figures for numeric columns; ≤3 font sizes per table view.
- **Touch targets (mobile):** 44px minimum.
- **Contrast (WCAG AA):** 4.5:1 text; 3:1 UI components / chart elements.
- **Breakpoints:** mobile <768px (3–4 metrics, vertical stack) · tablet 768–1024px (2-col) · desktop >1024px (3–4 col grid).
- **Faceted search:** 5–7 facets per results page; overflow behind "More filters" above that.
- **Skeleton screens:** lower perceived load time vs. spinners; conveys structure is incoming. **Spinner** use case: 1–10s bounded waits.
- **Pagination** threshold: prefer it for 200+ row datasets; show total record count always.
- **Column min width:** 60px; double-click header border to auto-fit to content.
- **Group column separator:** 2px or background-shift between logical column groups; no full vertical gridlines across the table.
- **Staleness threshold:** show a warning badge if data is older than the agreed freshness SLA for that feed (define per widget, not globally).

## Tools & libraries
- **Table engine:** [TanStack Table](https://tanstack.com/table) (headless: sorting, filtering, pagination, column sizing/visibility/pinning, you own the markup). For 10k+ rows, pair with **row virtualization** ([TanStack Virtual](https://tanstack.com/virtual)). AG Grid for heavyweight enterprise grids (Excel-like, but heavier and licensed for advanced features).
- **Headless UI primitives:** Radix / shadcn-ui for accessible menus, dialogs, popovers (column-settings dropdown, edit modals). See [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md).
- **Charts:** ECharts or Recharts/Visx for standard dashboards; D3 only when you need custom encodings. Keep encodings to bar/line/scatter (see chart section).
- **Styling:** Tailwind works well for dense grids and consistent spacing tokens — see [CSS, Styling & Tailwind](../04-libraries-and-tools/css-styling-and-tailwind.md).
- **When NOT to reach for a chart lib:** a sparkline or single bar is often a few `<svg>` lines; don't pull a charting dependency for one KPI sparkline.
- **Figma caveat:** Figma has no first-class table support; designing full tables there is painful. **Mock header + 2 rows, then specify behavior** (sort/filter/freeze/density/states) in docs or code. See [Wireframing, Prototyping, Handoff & QA](../05-process-and-systems/wireframing-prototyping-handoff-qa.md).

## Do / Don't
**Do**
- Give every KPI comparison + benchmark + trend.
- Add sticky header + frozen first column before anything else.
- Right-align numbers, left-align text, use tabular figures.
- Put bulk actions in a floating bottom bar; reveal checkboxes on hover; use indeterminate checkbox when some rows are selected.
- Clarify select-all scope across pages with an explicit CTA ("Select all 10,482 rows?").
- Offer density control and persist all view state (columns, sort, filters) with a reset; store saved views server-side.
- Encode filters in the URL so views are shareable and bookmarkable.
- Design empty, loading, and error states explicitly; pick the loading pattern by duration.
- Use length/position encodings; direct-label trend lines.
- Show per-widget "last updated" timestamps; add staleness warning badges when data exceeds freshness SLA.
- Support keyboard navigation and saved views/presets; command palette for search.
- Validate inline edits on blur, show errors inline (not as toasts).
- Virtualize tables at 2,000+ rows.
- Show total record count always in pagination.

**Don't**
- Don't ship a raw number with no comparison context.
- Don't center-align table data; don't proportional-figure numeric columns.
- Don't use pie/donut/gauge/radar/treemap for quantitative comparison; never 3D.
- Don't rely on color alone to differentiate series.
- Don't put bulk actions in the top toolbar.
- Don't use infinite scroll for large enterprise tables.
- Don't apply mobile-first density cuts to analyst dashboards.
- Don't leave empty search results blank, or errors as "Something went wrong."
- Don't freeze a progress bar at 99%, or use a skeleton for form submission.
- Don't guess default columns/sort — research the user workflow.
- Don't confuse global search with column filters — they serve different mental models.
- Don't store saved views only in localStorage — they must survive device switches.
- Don't render 2,000+ rows without virtualization.
- Don't put more than 3 row-level action icons directly in the row; overflow the rest to a kebab menu.

## Common pitfalls & how to avoid them
- **Template-collapse at scale.** Dribbble admin templates look great with 3 rows and break at 50×20. → Test designs with realistic data volume (50+ rows, 20 columns) before committing.
- **Data eyeball attack.** High density, no hierarchy → users flee. → Strict grid + typographic hierarchy + progressive disclosure.
- **Too many charts.** Each added chart trains users to expect meaning and lowers signal-to-noise. → Cap at 5–7 metrics; split dashboards by role/task.
- **No comparison context.** Raw numbers can't drive decisions. → Mandate prior-period + target + trend on every KPI.
- **Stale data, no timestamp.** Mixed update frequencies with no staleness signal destroys trust. → Per-widget "last updated" + visual staleness cue.
- **Bulk actions buried in the top bar.** → Floating bottom action bar.
- **No undo on destructive bulk ops.** → Undo toast or confirm dialog.
- **Zebra + interactive states = grey soup.** → Default to 1px line dividers.
- **Color-only differentiation.** Fails 4.5–8% colorblind users. → Patterns/hatching, dashed lines, direct labels, icons; blue/orange not green/red.
- **Undefined acronyms/jargon.** Users decode instead of read. → Tooltip every acronym.
- **Missing/weak states.** Empty/error/loading skipped or all-spinner. → Design each; match loading pattern to duration.
- **Mouse-only dense UI.** → Add keyboard nav + command palette.
- **Guessed defaults.** Skipping research on default columns/sort forces users to constantly hide/re-sort. → Base defaults on observed workflow.
- **Select-all page confusion.** Selecting the header checkbox on page 1 silently scopes bulk ops to 25 rows when users expect "all 10,000." → Explicit scope CTA and record count.
- **Row virtualization skipped at scale.** Rendering 5,000 DOM rows kills layout performance; users blame the product for being "slow." → Virtualize at 2,000+ rows.
- **Saved views only in localStorage.** User changes device or clears storage — all custom views gone. → Server-side persistence.
- **Global search and column filters conflated.** Users can't tell what "search" queries. → Separate entry points with clear labels.
- **Stale data, no per-widget timestamp.** A single global "page loaded at 14:02" is useless when widgets refresh on different schedules. → Per-widget freshness.

## What real users complain about
- "It looks nothing like the mockup." Real data (50+ rows, 20 columns) breaks layouts designed against 5 clean rows.
- **Lost place on infinite scroll** — can't return to a known record; bulk actions become unpredictable.
- **Bulk actions nobody finds** — selecting rows surfaces actions in the cluttered top bar that blend in and get missed.
- **No undo** — accidentally deleted 50 records with no recovery path.
- **Select-all scoped to the visible page** — selected "all" and deleted 25 rows, not the 10,000 they intended (or the reverse — expected to affect only the page, got all records).
- **Stock templates lack power features** — no multi-column sort, column pinning, density control, or saved filter presets; power users hit a wall.
- **Saved views lost after clearing browser data** — localStorage persistence means custom column sets and filters disappear between devices or after cache clears.
- **Stale data, no timestamp** — can't tell live data from hours-old cache; one stale widget with no indication undermines trust in the whole dashboard.
- **Dead-end filters** — a filter combo returns zero results with no explanation or way out.
- **Can't find the action for a single row** — row actions hidden under a kebab with no discoverability cue; or primary action requires opening a modal instead of clicking the name.
- **Internal tools harder than consumer apps** — a widely reported sentiment; poor enterprise UX carries measurable productivity cost. See [What Real Users Complain About](../08-pitfalls-and-antipatterns/real-user-complaints.md).

## Senior-level checklist
- [ ] Picked density strategy via the session-duration test (and documented it).
- [ ] Top row = 3–5 KPIs; each has value + prior-period comparison + benchmark + trend.
- [ ] ≤5–7 primary metrics per dashboard; split by role/task if more.
- [ ] Primary insight graspable in <5s (5-second test passed with a real user/colleague).
- [ ] Charts use length/position; no pie/gauge/radar/3D for comparison; ≤3–4 series; trend lines direct-labeled.
- [ ] One color = one meaning across all charts; series distinguishable in grayscale (patterns/dashes).
- [ ] Table: left-align text, right-align numbers, tabular figures, header alignment matches content.
- [ ] Sticky header + frozen identifier column; rightmost frozen if it holds totals.
- [ ] Density control (40/48/56px) + persisted per-user view state (order/width/visibility/freeze/sort/filters) + reset-to-default.
- [ ] Default columns and default sort chosen from user research, not assumption.
- [ ] Multi-column sort and saved filter presets available for power users.
- [ ] Row-level actions: primary action on name click; ≤3 inline icon buttons on hover; overflow to kebab for more.
- [ ] Checkboxes on hover; indeterminate checkbox state for partial selection; bulk actions in a floating bottom bar; undo/confirm for destructive bulk ops.
- [ ] Select-all across pages: explicit CTA with record count when user selects page-only header checkbox.
- [ ] Export (CSV/XLSX) in the bulk action bar, scoped to selection or full filtered set.
- [ ] Edit friction matched to stakes (inline / expandable / modal); inline errors shown in-cell on blur, not as toasts.
- [ ] Empty (first-use + zero-results + done), loading (right pattern per duration), and error (message + retry) states all designed.
- [ ] Global search vs. column filters: separate entry points, clearly labeled.
- [ ] Filters encoded in URL for shareability.
- [ ] Saved views: named snapshots of filters + columns + sort; stored server-side; system defaults + user overrides.
- [ ] Pagination for 200+ row datasets; total record count always visible; virtualization at 2,000+ rows.
- [ ] Responsive downgrade defined per breakpoint; sidebar → drawer below 768px; responsive table strategy chosen.
- [ ] Role-based default views implemented where roles differ materially.
- [ ] Keyboard navigation + command palette + visible focus; WCAG AA contrast (4.5:1 text, 3:1 UI/chart).
- [ ] Per-widget "last updated" timestamps; staleness warning badge if data exceeds freshness SLA.
- [ ] All acronyms/jargon explained on hover.
- [ ] Performance: load <2s, interaction <100ms, ≤1000 points/chart; rows virtualized at 2,000+.
- [ ] Validated against realistic data volume (50+ rows × 20 columns), not toy data.

## Sources
- Nielsen Norman Group — https://www.nngroup.com/ (preattentive processing & data-viz encoding; empty/error/loading state guidance)
- TanStack Table — https://tanstack.com/table
- TanStack Virtual — https://tanstack.com/virtual
- WCAG 2.2 — https://www.w3.org/TR/WCAG22/
- Edward Tufte — *The Visual Display of Quantitative Information* (data-ink ratio; chartjunk; encoding principles)
- AG Grid documentation — https://www.ag-grid.com/react-data-grid/ (enterprise grid patterns: column virtualization, row grouping, pivot)

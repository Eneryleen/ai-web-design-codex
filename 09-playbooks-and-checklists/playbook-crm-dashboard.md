# Playbook: Design a CRM / Admin Dashboard

> A decision-by-decision playbook for building a data-dense CRM/admin dashboard: map roles and jobs before pixels, define role-based IA, design the three core screen archetypes (list, detail, overview), engineer data tables and filters, choose KPIs and charts that survive colorblindness and zoom, handle every UI state, pick and theme a component library, hit WCAG 2.2 AA, and scale tables from 1K to 100K+ rows. Following it produces an admin panel that every role can use on day one — not a one-size-fits-none that fails all roles simultaneously.

**Apply when:** building an internal/B2B tool where users return daily to scan, filter, act on, and drill into structured business data. · **Related:** [CRM & Admin Dashboards](../03-site-types/crm-admin-dashboards.md) · [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md)

## TL;DR — rules at a glance
- **Map roles + jobs-to-be-done before any layout.** Executives want KPIs, analysts want drill-downs, ops staff want action-ready rows. One dashboard for all roles fails all roles at once.
- **3–5 critical KPIs in the first viewport, cap KPI cards at 3–4.** Anything needing scroll before orientation raises abandonment. >4 cards collapse into ignored noise.
- **Progressive disclosure is the architecture, not a nice-to-have.** Summary → click → detail. Tabs, drill-downs, collapsible sections are load-bearing.
- **Consistency beats polish.** One spacing grid, one token set, one set of interaction affordances across every screen. Never make the user re-learn the UI moving from Contacts to Pipeline.
- **Design all states.** A significant fraction of user sessions hit loading/error/empty paths. Reputation is built or destroyed there, not on the happy path.
- **Table alignment is a rule, not a preference.** Left-align text + qualitative numbers (dates, ZIP, phone); right-align quantitative numbers (amounts, %, measures); use tabular/monospace figures so `$1,111.11` isn't visually smaller than `$999.99`.
- **Row heights: Condensed 40px / Regular 48px / Relaxed 56px.** Default 48px; use 40px to maximize density in complex panels. Don't inflate height to "breathe" — it directly cuts visible rows.
- **Pagination default 25 rows** (options 10/25/50/100). Client-side processing OK to ~1,000 rows; server-side sort/filter/paginate beyond a few thousand.
- **Virtualize on scale:** 10K–50K rows → TanStack Table + TanStack Virtual; 100K+ → AG Grid server-side row model + dual-axis virtualization. Virtualization renders only the visible window, keeping the DOM tiny at any list size.
- **Skeletons over spinners.** Meaningfully lower perceived load time (users see structure, not a void), no layout shift on arrival, telegraphs the incoming content shape.
- **Color is the last differentiator.** Add text labels, shape/pattern encodings, or position before relying on hue. ~300M colorblind people (1 in 12 men).
- **Contrast is law:** 4.5:1 text (WCAG 1.4.3), 3:1 non-text/chart elements (1.4.11). Text resizable to 200% (1.4.4) without charts breaking. Required under Section 508 + European Accessibility Act.
- **Filters persist across navigation** (detail → back keeps state). Saved presets for power users. First/identifier column sticky on horizontal scroll.
- **Error copy must answer "my fault, my WiFi, or the server?"** plus a correlation ID logged server-side. Never ship "Something went wrong."

## 1. Map roles, jobs-to-be-done & data first
Do this in writing before any wireframe. If you can't fill these, stop and ask — don't design around ambiguity.

For each role, capture a one-liner:
- **Role + primary job:** "Sales rep — work today's leads to quota." "Sales manager — spot pipeline risk." "Marketer — measure campaign ROI." "Ops/admin — process the action queue, fix bad records."
- **Default landing view:** the screen they should see on login (rep → personal quota + leads; manager → pipeline health; marketer → campaigns). This is RBAC-driven, not a shared homepage.
- **Top 5 tasks, ranked:** the recurring actions. These become primary nav + per-row actions.
- **Decision cadence:** real-time (ops queue), daily (rep), weekly (manager), monthly (exec). Cadence dictates refresh + alerting needs.

Then map the **data model** before screens, because IA mirrors entities:
- **Core entities** (Contacts, Accounts, Deals/Opportunities, Activities, Campaigns, Tickets) → each gets a list + detail screen.
- **Relationships** (Account → many Contacts → many Deals) → drive breadcrumbs, related-record panels, and drill paths.
- **Identifier field per entity** (record name + immutable ID) → first/frozen table column.
- **Quantitative fields** (amount, count, %, score) → right-aligned, tabular figures, candidates for KPIs/charts.
- **Status/lifecycle fields** (stage, priority, health) → badge columns + filter facets + (last-resort) color, always with text.

> Anti-pattern: designing a generic "dashboard" before knowing whose job it serves. A KPI grid that helps no one is the most common failure mode.

## 2. Information architecture & navigation
**Build RBAC into the IA, not as an afterthought.** Each role sees a tailored nav + default dashboard. Hide what a role can't act on; don't gray it out and clutter scan.

Navigation rules:
- **Keep it shallow and predictable.** Aim ≤3 levels: section → entity list → record. Deep trees kill returning-user speed.
- **"Recent Records"** (last-viewed contacts/accounts/deals) is a high-value shortcut — cuts navigation depth by 2–3 clicks for returning users. Put it in the sidebar or a global menu.
- **Global search / command palette** (`Ctrl/Cmd+K`) is non-negotiable in a CRM; returning users navigate by search, not by clicking trees. It must support: fuzzy entity search (contacts, accounts, deals by name/ID/email), navigation to sections, recent records, and quick actions (e.g. "Create deal", "Go to Settings"). Debounce at 150–200ms; show grouped results (Contacts · Accounts · Deals · Actions) with keyboard navigation (↑↓ + Enter).
- **Breadcrumbs** on every detail screen reflect the entity hierarchy (Accounts → Acme Corp → Deal #4821).

**Left-rail vs. top-rail layout:**
- **Left sidebar** maximizes horizontal content width (best for wide data tables) and scales to many nav items. Default choice for data-dense admin.
- **Top rail** puts KPIs in the visual "hot zone" of the first view — better for executive overview dashboards with few sections.
- Common hybrid: left sidebar for entity nav + slim top bar for global search, account menu, notifications.

```
┌──────────────────────────────────────────────┐
│ ☰  CRM    [⌘K search]        🔔  ▾ user       │  top bar: global utilities
├────────────┬─────────────────────────────────┤
│ Dashboard  │  Page Title          [+ New]    │
│ Contacts   │  ┌─KPI─┐┌─KPI─┐┌─KPI─┐           │  KPIs in first viewport
│ Accounts   │  └─────┘└─────┘└─────┘           │
│ Deals      │  [filters · saved views]         │
│ Activities │  ┌─────── data table ───────┐    │  sidebar = max content width
│ ─────────  │  │                          │    │
│ Recent ▾   │  └──────────────────────────┘    │
└────────────┴─────────────────────────────────┘
```

## 3. Design the three core screens
Almost every admin screen is one of three archetypes. Standardize each once, reuse everywhere.

### 3a. List screen (the workhorse)
The default screen for an entity. Contains: title + primary action (`+ New`), filter/search bar with saved views, then the data table (see §4). Optimized for **scan → filter → act/drill**.
- Sort default = **most-recently-created or most-needs-action first** (not alphabetical — nobody opens a list to find row "Aaron").
- Per-row primary action (open, edit, assign) reachable without opening the record.
- Bulk actions appear on row selection (assign owner, change stage, delete) with a clear selection count.

### 3b. Detail screen (the record)
Single record: header (name, ID, status badge, key actions), then **progressive disclosure** of everything else.
- **Header band:** identifier + status + 1–3 primary actions + breadcrumb. The 2–3 most-needed facts here, not buried in tabs.
- **Tabs or collapsible sections** for the long tail (Overview / Activity / Related / Notes / History). Never dump every field in one wall.
- **Related-record panels** (this Account's Contacts/Deals) using the data model relationships — link, don't re-navigate from scratch.
- **Destructive actions (delete, archive, merge)** on a record require a confirmation modal naming the record and the irreversibility. Place these actions in an overflow menu (`⋯`), not as primary buttons.
- **Filters from the originating list must survive the round trip** (§5). Returning lands you back in the same filtered list, same scroll.

### 3c. Overview/dashboard screen (the summary)
The role's landing view: KPIs + charts + an action queue. Pure summary; every element is a drill-in entry point.
- **3–4 KPI cards** across the first viewport (§6). One metric + delta each.
- **2–4 charts** answering the role's top questions (pipeline by stage, revenue trend, leads by source). Each clickable to a filtered list — clicking a chart segment must deep-link to the matching list view with the corresponding filter pre-applied (e.g. clicking "Negotiation" bar → `/deals?stage=negotiation`). Never send users to an unfiltered list from a chart click.
- **An action list** ("deals closing this week," "overdue tasks") — the single most valuable widget for ops/rep roles; it turns a passive dashboard into a worklist.

## 4. Data tables — the highest-leverage component
Tables are where admin users spend most of their time. Get these right and the product feels professional.

**Alignment & type (non-negotiable):**
- Left-align: text, names, dates, ZIP/postal codes, phone numbers, IDs.
- Right-align: amounts, measures, percentages, counts.
- **Tabular/monospace figures** for all numbers so digit columns line up: `font-variant-numeric: tabular-nums;`
- Column header alignment matches its column's content; sort-chevron must not shift header text relative to the column.

**Density:**
- Row heights — Condensed **40px**, Regular **48px**, Relaxed **56px**. Default 48px; 40px for power/admin panels; 56px for general-business audiences.
- Don't inflate row height for aesthetics — it's a beauty-over-usability trade that cuts visible rows, hurting the table's core job.

**Dividers (less is more):**
- Use **horizontal 1px borders in light grey** that "melt into the background." Vertical separators are optional and usually harmful (visually busy) — omit them.
- **No zebra striping on tables under ~10 rows** — users ascribe semantic meaning to highlighted rows where none exists. Use line divisions instead. Zebra is acceptable only on long, dense tables to aid horizontal tracking.

**Sticky / frozen columns:**
- First column (identifier: ID/name) is **sticky/frozen on horizontal scroll**. Enable per-column freezing so users can anchor multiple identifiers for side-by-side comparison.

**Truncation by data type:**
- IDs/codes → **truncate middle** (`a1b2…f9e8`) + copy-on-click.
- Names/titles → **truncate end** with ellipsis + tooltip on hover (full value).
- Descriptions → **clamp to 2 lines** then truncate.

```css
/* Numbers: aligned figures */
.cell--num { text-align: right; font-variant-numeric: tabular-nums; }
/* Name: end-ellipsis + tooltip via title attr */
.cell--name { max-width: 22ch; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
/* Description: 2-line clamp */
.cell--desc { display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
/* Light, "melting" row divider */
.row { border-bottom: 1px solid color-mix(in srgb, currentColor 8%, transparent); }
```

**Column customization:** show/hide, reorder, resize — and a **prominent "Reset to defaults"** button so users experiment without fear. **Persist customization server-side or localStorage** so it survives page reload and tab close; customization that resets on refresh trains users not to bother.

**Inline editing (critical CRM pattern):**
- High-frequency fields (status, owner, stage, priority) must be editable directly in the row without opening the full record.
- Click-to-edit: single click activates the cell; `Enter` / `Tab` confirms and moves to next editable cell; `Escape` cancels.
- Show a subtle edit affordance (pencil icon or border highlight) on cell hover — don't leave users guessing which cells are editable.
- Persist on blur or explicit `Enter`; show an inline save indicator (✓) and expose optimistic updates with rollback on failure.
- Cells that require the full detail form (multi-field relationships, rich text) → open a side sheet, not the full record, to keep list context visible.

**Bulk actions (selection model):**
- Row checkbox appears on hover; a header checkbox selects/deselects the current page (not all pages — make this explicit: "50 rows selected on this page. Select all 1,284?").
- Shift-click range select across rows.
- Bulk action bar appears above the table on first selection (assign owner, change stage, export, delete) with a visible selection count.
- **Destructive bulk actions (delete, archive) require a confirmation modal** stating what will be deleted + a count. Never execute destructive bulk ops inline.
- For async bulk ops (mass email, large export), show progress feedback and notify on completion — don't block the UI.

**Pagination:** default **25 rows** (options 10/25/50/100). 25 is the density/performance sweet spot for enterprise. Show total count + range ("26–50 of 1,284").

## 5. Filters
Filters are the second-most-used control after the table itself.

- **Persistence is required, not optional.** Filters must survive navigating into a detail record and back, and ideally a page reload (encode in URL query params — also makes filtered views shareable/bookmarkable).
- **Saved filter presets ("views")** for power users who run the same query daily ("My open deals," "Overdue tasks"). Let them set a default view.
- **Filter type by field:**
  - Status/category/owner → multi-select facets (checkbox list), each showing match count.
  - Dates → range picker + quick presets (Today, 7d, 30d, This quarter).
  - Amounts/scores → min–max range.
  - Free text → debounced search (250–300ms) across indexed fields.
- **Show active filters as removable chips** above the table + a "Clear all." Users must always see what's narrowing the data.
- **Filtering re-paginates from page 1** and shows a result count; empty result = empty state with "Clear filters" CTA (§7), not a blank table.

```
URL state ⇒ shareable + persistent:
/deals?stage=negotiation,proposal&owner=me&amount_gte=10000&view=my-pipeline&page=1
```

## 6. KPIs & charts
**KPI cards:** one metric + its delta vs. the previous period. Nothing more per card.
- **Cap at 3–4 cards in the first viewport.** More degrades scan time until none get read.
- **Typographic hierarchy within the card:** large/bold primary number, small label, small secondary delta. Do not style label, value, and delta at the same weight — that flattens hierarchy and slows reading.
- **Delta = direction + magnitude + meaning:** `▲ 12% vs last month`, colored *and* arrowed (color alone fails colorblind users). Define whether "up" is good per metric (churn up is bad).

```
┌─────────────────────┐
│ Pipeline value      │  ← label, 13px, muted
│ $1,284,500          │  ← primary, 28–32px bold, tabular-nums
│ ▲ 12% vs last month │  ← delta, 13px, color + arrow + text
└─────────────────────┘
```

**Charts — pick the right encoding:**
- Trend over time → line/area. Comparison across categories → bar. Part-to-whole (≤5 slices) → donut, else bar. Distribution → histogram. Avoid 3D, exploded pies, and dual-Y axes that mislead.
- **Color is a last resort for series differentiation.** Add direct labels, distinct shapes/markers, or pattern fills before relying on hue. Verify the palette in a colorblind simulator.
- **Every chart needs an accessible fallback:** ARIA labels on SVG elements, and a **text table or data summary beneath the chart**. Wrap live/streaming updates in `aria-live="polite"` so screen readers announce changes without spamming.
- Charts must not break at **200% zoom** and must keep **3:1 contrast** on bars/lines/areas vs. background (WCAG 1.4.11).

## 7. UI states — design every one
State handling is part of the screen spec, not polish. Every screen spec must include loading, empty (first-run vs. filtered-empty), error, and partial-degraded. Skipping them produces a product that feels half-finished on any network hiccup or empty account.

**Loading → skeleton screens, not spinners.** Skeletons meaningfully lower perceived load time, eliminate layout shift on arrival, and telegraph the incoming structure (table skeleton = header + N grey rows). Use a spinner only for sub-300ms or indeterminate background actions (e.g. exporting, sending).

**Empty states — explain why + one clear next step.** Never leave a blank container. Include a brief reason and a single CTA:
- First-run empty (no records yet) → "No deals yet. Create your first deal to start tracking your pipeline." + `[+ New Deal]`.
- Filtered-empty (filters match nothing) → "No deals match these filters." + `[Clear filters]` (different copy + CTA than first-run — never conflate them).
- Never ship a blank container with no guidance.

**Error states — answer "is this my fault, my WiFi, or the server?"**
- Minimum viable: "Couldn't load deals. Try refreshing — if this continues, contact support." + a **correlation ID** displayed and **logged server-side** so it's traceable.
- Distinguish causes: validation error (your input) vs. network (your connection) vs. 5xx (our server). Give the matching recovery action (fix field / retry / contact support).
- Never "Something went wrong." with no ID and no action.

**Partial/degraded:** if one widget fails, fail that widget (inline error in its card) — don't blank the whole dashboard.

## 8. Responsive strategy
Non-negotiable minimum for a mobile-capable admin panel:
- **Sidebar → mobile drawer** below the tablet breakpoint (hamburger toggle, off-canvas).
- **Data tables scroll horizontally** with the identifier column frozen; do not crush 12 columns into a phone width. Optionally collapse low-priority columns into an expandable row on mobile.
- **Charts resize via CSS container queries** (size to their card, not the viewport), so a chart in a narrow sidebar panel and a full-width panel both render correctly.
- **KPI cards** reflow from a row of 4 → 2×2 grid → single column.

```css
.dashboard-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; }
@container (max-width: 900px) { .dashboard-grid { grid-template-columns: repeat(2, 1fr); } }
@container (max-width: 520px) { .dashboard-grid { grid-template-columns: 1fr; } }
```
Breakpoints (typical admin): mobile <768px (drawer nav, single-column, horizontal table scroll), tablet 768–1024px (2-col KPIs), desktop ≥1024px (full layout). Admin tools are desktop-primary; mobile must work but optimize for the desktop power user.

## 8b. Real-time data & refresh strategy
Decide the data freshness model per widget before building; mixing stale and live data on the same screen without indicating which is which erodes trust.

- **Polling:** simplest. Interval of 30–60s for dashboards, 5–15s for ops queues. Show "Last updated X seconds ago" + a manual refresh button. Pause polling when the tab is hidden (`visibilitychange`) to avoid unnecessary load.
- **WebSocket / SSE:** for true real-time (live chat, live ops queue, collaborative pipelines). SSE is simpler (HTTP, no upgrade), sufficient for one-way server push. WebSockets needed for bidirectional.
- **Stale-while-revalidate (SWR / TanStack Query):** show cached data immediately, revalidate in background, update silently if unchanged — the default pattern for most CRM screens. Avoids full loading states on re-visits.
- **Optimistic updates:** on inline edits or status changes, update the UI immediately and revert on server error. Show a brief "Saving…" indicator, not a full loading state.
- **Indicate staleness explicitly:** if a widget could be >60s stale, show a timestamp or a "stale" badge. Don't leave users wondering if the number is current.

## 8c. Export & download
Export is a near-universal CRM requirement — spec it explicitly.

- **Inline export button** on every list screen and on dashboard widgets; icon + text label ("Export CSV"), not an unlabeled icon.
- **Scope options:** export current page / export filtered results / export all. Make the scope explicit in the button label or a dropdown ("Export 1,284 filtered rows").
- **Small exports (<5K rows):** synchronous, trigger browser download immediately.
- **Large exports (5K+ rows):** async — trigger a background job, show a progress indicator, notify via banner or email when ready. Never block the UI waiting for a large export.
- **Formats:** CSV (universal) + XLSX (common enterprise ask). PDF for printable reports.
- **Column selection for export:** respect the user's current column visibility/order, or offer a dedicated column picker for export.
- **Security:** enforce the same RBAC filters on exports as on the table view — never allow a low-privilege export to bypass row-level security.

## 9. Pick a component library
Match the library to the job; don't default to whatever's trendy.

| Library | Best for | Notes |
|---|---|---|
| **shadcn/ui** | SaaS + custom design systems (Next.js) | Copy-paste ownership — no npm runtime dep, components live as local files you edit. Radix primitives + Tailwind. Strong a11y. Dominant mindshare in the React ecosystem as of 2025–26. |
| **Tremor** | Analytics dashboards | 35+ components (line/bar/area/donut charts, KPI cards, tables, filters). Radix + Recharts + Tailwind. Full TS + WAI-ARIA. Fastest path to a metrics dashboard. v3 (2024) is a full rewrite — check changelog before upgrading. |
| **Ant Design (AntD)** | Enterprise admin, fintech/ERP/B2B | 60+ components; **ProTable + ProForm** are best-in-class for rapid admin CRUD. A11y improving in v6 but behind Radix-based libs. |
| **Material UI (MUI)** | Established apps, large grids | Solid **MUI Data Grid** for large datasets; mature, broad ecosystem. |
| **AG Grid** | Six-figure row counts | Enterprise data grid: server-side row model + dual-axis virtualization for 100K+ rows. Use specifically for the table, alongside any UI kit. |

**When NOT to:** don't pull AntD just for buttons if you need a custom brand (its opinionated look fights theming) — use shadcn/ui. Don't use Tremor as a general UI kit — it's chart/dashboard-focused. Don't hand-roll a data grid past a few thousand rows — use TanStack/AG Grid.

## 10. Theme it
- **Token-driven from day one.** Colors, spacing, radii, typography, elevation as semantic tokens (`--color-bg-surface`, `--space-4`, `--text-muted`) — never hard-coded hex in components. Tokens are what make dark mode, white-label, and contrast fixes one-line changes.
- For small/medium teams, **theme an existing system** (Material 3, Polaris, or your library's tokens) rather than building a system from scratch.
- **A design system is a living product**, not a static component dump — it needs token architecture + an owner + a roadmap. Treated as a static library, it drifts and gets abandoned. Cited successes: GOV.UK Design System, Salesforce Lightning (SLDS).
- **Consistency tokens across screens:** identical spacing grid, color tokens, and affordances on Contacts, Deals, Settings — users never re-learn the UI.

```css
:root {
  --space-2: 8px; --space-4: 16px; --space-6: 24px;
  --color-bg-surface: #ffffff; --color-bg-muted: #f6f7f9;
  --color-border: #e3e6ea; --color-text: #1a1d21; --color-text-muted: #5b6470;
  --color-accent: #2563eb; --color-pos: #15803d; --color-neg: #b91c1c;
  --radius: 8px;
}
[data-theme="dark"] { --color-bg-surface: #111418; --color-text: #e6e9ee; --color-border: #2a2f36; }
```

## 11. Accessibility (WCAG 2.2 AA — legal floor)
Section 508 references WCAG 2.1/2.2 AA. The European Accessibility Act enforcement deadline for new products passed in June 2025 — it is now in force for EU markets. AA is the legal standard for most jurisdictions; treat it as the floor, not the goal.
- **Contrast:** 4.5:1 all text (1.4.3); 3:1 non-text UI + chart elements — bars, lines, areas, borders, focus rings (1.4.11).
- **Resize:** text to **200%** without assistive tech (1.4.4); charts must not break at 200% zoom.
- **Don't rely on color alone** for status/series (1.4.1) — pair with text labels, icons, or patterns.
- **Tab order for dashboards:** page title → slicers/filters → primary KPIs → charts/tables → navigation buttons (top-down, left-to-right). Verify with keyboard only.
- **ARIA on charts:** label SVG elements; provide a text table/summary alternative beneath each chart; wrap real-time updates in `aria-live="polite"`.
- **Tables:** real `<table>` semantics (`<th scope>`, caption), sortable headers expose `aria-sort`, keyboard-operable controls.
- **Focus visible** on every interactive element (≥3:1 focus indicator). Don't remove outlines without replacing them.
- **Touch targets:** 24×24 CSS px min (WCAG 2.5.8); prefer 44×44px (Apple HIG) / 48×48dp (Material) for primary actions and any mobile target.

## 12. Performance with large datasets
The number of rows decides the architecture — pick the tier up front.

| Rows | Strategy | Tooling |
|---|---|---|
| ≤ ~1,000 | Client-side sort/filter/paginate | Any table lib; full dataset in memory is fine |
| Few thousand+ | **Server-side** sort/filter/paginate | Don't hold full set in browser memory |
| 10K–50K | **Row virtualization** | TanStack Table + TanStack Virtual (tested smooth to 50K+) |
| 100K+ | **Server-side row model + dual-axis virtualization** | AG Grid (rows *and* columns virtualized) |

- **Virtualization keeps the DOM tiny:** `react-window` renders only the nodes in the current viewport window plus a small configurable overscan buffer (default 1–2 rows), regardless of total list size.
- **Dual-axis virtualization (rows AND columns)** is required for tables that are both tall and wide — row virtualization alone solves only half the problem for enterprise panels.
- **Don't refetch the world on every keystroke:** debounce search (250–300ms), paginate server-side, request only visible columns.
- **Profile, don't guess:** React DevTools Profiler for needless re-renders; Chrome DevTools Performance tab for FPS/paint during scroll; Lighthouse for the overall page.
- Memoize row/cell renderers; keep stable keys; avoid inline object/array props that bust memoization on every render.

## Concrete specs & numbers
- **WCAG text contrast:** 4.5:1 (1.4.3) · **non-text/chart:** 3:1 (1.4.11) · **text resize:** 200% (1.4.4) · **target size:** 24×24 CSS px (2.5.8).
- **Row heights:** Condensed 40px / Regular 48px / Relaxed 56px (default 48px).
- **Pagination:** default 25 rows; options 10 / 25 / 50 / 100.
- **KPI cards in first viewport:** max 3–4. Primary metric type 28–32px bold; labels/deltas ~13px.
- **Client-side table limit:** ~1,000 rows before server-side processing is needed.
- **Virtualization thresholds:** 10K–50K → TanStack Table + Virtual; 100K+ → AG Grid server-side row model.
- **`react-window`:** renders only the visible viewport window + configurable overscan, regardless of total list size.
- **Table dividers:** horizontal 1px light grey; vertical separators ≤1px and usually omitted.
- **Zebra striping:** avoid below ~10 rows.
- **Skeleton screens:** meaningfully lower perceived load time vs. spinners; no layout shift on arrival.
- **State handling:** every screen spec must include loading, empty (first-run vs. filtered-empty), error, and partial-degraded states.
- **Debounce:** 250–300ms for search/filter input.
- **Colorblindness:** ~300M people, 1 in 12 men.
- **Library counts:** Tremor 35+ components · AntD 60+ components · shadcn/ui local-file (no npm runtime dep).

## Tools & libraries
- **UI kits:** shadcn/ui (custom SaaS), Tremor (analytics dashboards), Ant Design (enterprise CRUD via ProTable/ProForm), MUI (established apps).
- **Tables/grids:** TanStack Table (headless logic, no built-in virtualization) + TanStack Virtual / react-window (10K–50K); AG Grid (100K+, server-side + dual-axis).
- **Charts:** Recharts (used under Tremor); Highcharts Dashboards (KPI + chart + grid components; threshold-driven KPI styling).
- **Design:** Figma (system + prototypes built here first).
- **Testing/QA:** Cypress / Playwright (automated error-state regression), Lighthouse (perf audit), React DevTools Profiler (re-renders), Chrome DevTools Performance (scroll FPS), Hotjar / Crazy Egg (heatmaps on live dashboards).
- **Reference systems:** GOV.UK Design System, Salesforce Lightning (SLDS), Material Design 3, Polaris (theme one of these rather than build from scratch for small/medium teams).
- **When NOT to:** don't hand-roll a grid past a few thousand rows; don't adopt a heavily-opinionated kit (AntD) when you need a strongly custom brand; don't use a chart-focused kit (Tremor) as your general component library.

## Do / Don't
**Do**
- Map every role's job + default landing view before wireframing.
- Put 3–4 KPIs + the role's key charts/action-list in the first viewport.
- Left-align text, right-align numbers, use tabular figures.
- Freeze the identifier column; persist filters in the URL; offer saved views.
- Use skeleton screens and design empty/error/filtered-empty states explicitly.
- Pair every status/series color with a text label or shape.
- Pick the table tier (client / server / virtualized / AG Grid) by row count up front.
- Token-drive theming; keep spacing/affordances identical across screens.
- Support inline editing for high-frequency fields (status, owner, stage).
- Require a confirmation modal for every destructive bulk or single-record action.
- Persist column customization across sessions; offer export with scope selection.
- Deep-link chart click-throughs to pre-filtered list views.

**Don't**
- Ship a generic dashboard before knowing whose job it serves.
- Cram >4 KPI cards above the fold.
- Center-align table cells (eyes jump, scanning slows).
- Inflate row height to "breathe" (cuts visible rows — the opposite of the table's job).
- Zebra-stripe tables under ~10 rows (users invent meaning for highlighted rows).
- Rely on color alone for chart series or status badges.
- Hold tens of thousands of rows in browser memory.
- Ship "Something went wrong." with no cause, no recovery, no correlation ID.
- Execute destructive bulk actions without a modal naming the scope and count.
- Let column customizations or filter state reset on page reload.
- Trigger a large export synchronously and block the UI waiting for it.
- Send users to an unfiltered list when they click a chart segment.

## Common pitfalls & how to avoid them
- **Happy-path-only screens.** Loading/empty/error feel half-finished. → Spec all states per screen; skeletons + empty CTAs + error w/ correlation ID.
- **KPI overload.** >4 cards collapse into noise; none get read. → Cap at 3–4, push the rest into the detail/drill view.
- **Zebra on short tables.** Users read meaning into highlighted rows. → 1px horizontal dividers; reserve zebra for long, dense tables.
- **Center-aligned cells.** Ragged edges slow scanning. → Left text / right numbers, tabular figures.
- **Row-height inflation.** Aesthetics over data density. → Default 48px; 40px for power panels.
- **Color-only encoding.** Fails ~300M colorblind users + low-contrast displays. → Text labels + shapes/patterns first; verify in a simulator.
- **Filters that reset on navigation.** Power users re-filter constantly, rage-quit. → Persist in URL; saved presets.
- **No frozen identifier column.** Wide tables become unreadable on horizontal scroll. → Sticky first column + per-column freeze.
- **Client-side everything at scale.** Browser chokes past a few thousand rows. → Server-side + virtualization per the tier table.
- **Design system as a static dump.** Drifts and gets abandoned. → Token architecture + ownership + roadmap; treat it as a product.
- **No inline editing.** Users open the full record to flip a status field. → Enable click-to-edit for high-frequency fields; keep list context visible.
- **Destructive bulk actions without confirmation.** Users delete 500 rows accidentally. → Always modal with scope count + irreversibility warning.
- **Column customization lost on reload.** Users stop customizing. → Persist to localStorage or server.
- **Chart clicks land on unfiltered lists.** Users lose context and must re-filter. → Deep-link with the matching filter preset.
- **Export blocks the UI for large datasets.** Users abandon the tab. → Async export with progress + notify on complete.

## What real users complain about
Synthesized from CRM/admin-tool user feedback and design-community discussion:
- "It looks pretty but I can't find anything" — beauty-over-usability dashboards with inflated spacing showing 6 rows where 20 fit.
- "My filters reset every time I open a record" — the single most common rage moment in list-heavy CRMs; breaks the core scan→drill→back loop.
- "The dashboard shows 12 numbers and I can't tell which matters" — no hierarchy, every metric at equal weight.
- "It just says 'Something went wrong'" — no cause, no recovery path, no ID to give support.
- "I can't tell the red bar from the green bar" — colorblind users locked out of status/chart meaning.
- "It freezes when I load the full account list" — client-side rendering of tens of thousands of rows.
- "Numbers don't line up so I can't compare amounts" — proportional figures + center alignment in money columns.
- "I customized my columns and now I can't get the original layout back" — no "Reset to defaults."
- "On my phone the table is unusable" — desktop-only table with no horizontal scroll or column collapse.
- "Every section of the app looks and behaves differently" — inconsistent spacing/affordances; users re-learn each screen.

## Senior-level checklist
- [ ] Every role has a written job-to-be-done + default landing view; nav is role-tailored (RBAC in the IA).
- [ ] Data model mapped to screens; each entity has list + detail; identifier field chosen per entity.
- [ ] Nav ≤3 levels; global search (`⌘K`) + Recent Records present; breadcrumbs on detail screens.
- [ ] First viewport: 3–4 KPI cards (one metric + delta each) + role's key charts/action-list, no scroll to orient.
- [ ] Tables: left text / right numbers, tabular figures, 48px default rows, 1px horizontal dividers, no vertical separators.
- [ ] No zebra under ~10 rows; identifier column frozen; per-column freeze available.
- [ ] Truncation by type (ID middle+copy / name end+tooltip / desc 2-line clamp).
- [ ] Pagination default 25 (10/25/50/100) with total count + range shown.
- [ ] Column show/hide/reorder/resize with a prominent "Reset to defaults."
- [ ] Filters persist across navigation (URL state), saved presets exist, active filters shown as removable chips.
- [ ] Every screen handles loading (skeleton), empty (first-run vs filtered, each with a CTA), and error (cause + recovery + correlation ID logged server-side).
- [ ] Charts: right encoding, color paired with labels/shapes, ARIA on SVG, text-table fallback, `aria-live="polite"` on live data, no break at 200% zoom.
- [ ] Contrast 4.5:1 text / 3:1 non-text+chart; text resizes to 200%; tab order title→filters→KPIs→charts/tables→nav; keyboard-only pass done.
- [ ] Touch targets ≥24×24 CSS px (44/48 for primary/mobile); focus visible everywhere.
- [ ] Responsive: sidebar→drawer, tables scroll horizontally w/ frozen column, charts use container queries, KPIs reflow 4→2→1.
- [ ] Table tier chosen by row count (≤1K client / few-K server / 10–50K TanStack+Virtual / 100K+ AG Grid dual-axis).
- [ ] Theming token-driven; spacing/affordances identical across all screens; dark mode = token swap.
- [ ] Perf profiled: no needless re-renders (DevTools Profiler), smooth scroll FPS (Performance tab), Lighthouse run; search debounced 250–300ms.
- [ ] Inline editing works for high-frequency fields (status, owner, stage): click activates, Enter/Tab confirms, Escape cancels, optimistic update with rollback.
- [ ] Bulk actions: checkbox + shift-range selection, selection count visible, destructive bulk ops require a confirmation modal naming count and scope.
- [ ] Column customization persists (localStorage or server); "Reset to defaults" visible.
- [ ] Export: scope options (page/filtered/all) explicit; large exports async with progress + completion notify; RBAC enforced on export.
- [ ] Real-time / refresh strategy decided per widget: polling interval set, tab-hidden polling paused, stale indicator shown where relevant.
- [ ] Chart click-throughs deep-link to pre-filtered list with correct URL params.
- [ ] Destructive single-record actions (delete/archive/merge) in overflow menu with confirmation modal.
- [ ] Global search (`⌘K`): fuzzy entity search + section navigation + quick actions; grouped results with keyboard navigation.

## Sources
- WCAG 2.2 (W3C): https://www.w3.org/TR/WCAG22/
- Section 508: https://www.section508.gov/
- shadcn/ui: https://ui.shadcn.com/
- Tremor: https://tremor.so/
- Ant Design: https://ant.design/
- Material UI Data Grid: https://mui.com/x/react-data-grid/
- TanStack Table: https://tanstack.com/table/
- TanStack Virtual: https://tanstack.com/virtual/
- react-window: https://github.com/bvaughn/react-window
- AG Grid: https://www.ag-grid.com/
- Recharts: https://recharts.org/
- Highcharts Dashboards: https://www.highcharts.com/products/dashboards/
- GOV.UK Design System: https://design-system.service.gov.uk/
- Salesforce Lightning Design System: https://www.lightningdesignsystem.com/
- Material Design 3: https://m3.material.io/
- Shopify Polaris: https://polaris.shopify.com/

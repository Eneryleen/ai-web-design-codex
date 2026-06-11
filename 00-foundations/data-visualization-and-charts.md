# Data Visualization & Charts

> How to choose, render, annotate, and make accessible the charts on a website — across functional dashboards AND editorial/marketing storytelling. Covers chart-to-question matching, honest axes and scales (zero-baseline, truncation, log, lie factor), data color (sequential/diverging/categorical, colorblind-safe, never color-only), the annotation layer, interaction with keyboard/touch parity, chart accessibility, scrollytelling, sparkline/bullet/big-number patterns, and library selection. The senior-level outcome: charts that answer the right question, never mislead, work for keyboard/screen-reader/colorblind users, and use a library appropriate to the data scale.

**Apply when:** designing any chart, dashboard metric, data story, or scrollytelling visual · **Related:** [CRM & Admin Dashboards](../03-site-types/crm-admin-dashboards.md) · [Color Theory & Systems](color-theory-and-systems.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Scroll-Driven Experiences](../03-site-types/scroll-driven-experiences.md)

## TL;DR — rules at a glance
- **Match the chart to the question, not the aesthetic.** Comparison among categories → bar/column. Trend over time → line. Relationship → scatter. Part-to-whole → bar (rarely pie). Distribution → histogram/box.
- **Bars MUST start at zero** (they encode by *length*). Lines MAY use a non-zero baseline (they encode by *slope/position*). This one distinction drives most axis-honesty calls.
- **Never use 3D or dual-axis charts.** 3D distorts by perspective; dual y-axes imply a relationship that doesn't exist. Use small multiples instead.
- **Avoid pie/donut** except 2–3 wildly-different slices; never for precise comparison or >5 slices. No 3D, no exploding, no leader lines — sort large→small.
- **Never encode by color alone** (WCAG 1.4.1). Add a redundant channel: line style, marker shape, hatching, position, or a direct label.
- **Limit categorical palettes to ~5–6 colors.** Beyond that even colorblind-safe hues blur. Default safe pair: **blue + orange** (replaces red/green).
- **Direct-label over legends.** Legends force eye ping-pong and break for colorblind users. Put the label on the line/bar/segment.
- **Lie Factor target ≈ 1.0** = `(effect shown) / (effect in data)`; Tufte's "honest" band is **0.95–1.05**. Distorted charts routinely hit 2–15× (Tufte's 1978 NYT fuel-economy chart = **14.8**). But a zero baseline is *not* always honest (the Baseline Paradox).
- **No critical info hover-only.** Labels, values, and legends must survive without a mouse. Every interaction needs keyboard + touch parity.
- **Alt text conveys the insight, not the chart type.** Not "a bar chart of sales" → "West leads at $2.3M, then East $1.8M…".
- **Explanatory ≠ exploratory.** Explanatory makes ONE argument (annotate, gray the rest). Exploratory lets users find their own (filter/brush/tooltip). Don't build the wrong one.
- **Pick the library by data scale.** SVG (Recharts/Visx) chokes past ~10k points; Canvas/WebGL (ECharts/uPlot) handles 50k–1M at ~60fps.
- **Contrast: 3:1 for bars/lines/markers**, 4.5:1 for text labels <18pt. Watch "ghostly" light-gray lines that fail 3:1.
- **Annotate sparingly:** label only peak, trough, current value, and the moment a change began. Over-labeling kills the pre-attentive pull.

---

## 1. Choosing the right chart for the question

Before drawing anything, answer three questions: **how many variables? how many points per variable? is it over time or among groups?** The answer dictates the chart.

| The question being asked | Chart | Why |
|---|---|---|
| Compare a value among categories | **Bar / column** | Length comparison is the most accurate visual judgment (Cleveland–McGill) |
| How did one metric change over time | **Line** | Slope and continuity read as "trend"; connects ordered points |
| Compare a few series over time | **Multi-line** (≤4) or **small multiples** | More than ~4 lines = "spaghetti"; split into panels |
| Relationship between two numeric variables | **Scatter** | Position on two axes reveals correlation, clusters, outliers |
| Part-to-whole, one moment | **Stacked bar** (or rarely pie for 2–3 slices) | Bars let you compare segment lengths; pie does not |
| Part-to-whole over time | **Stacked area** or **100% stacked bar** | Shows composition shift; beware middle bands are hard to read |
| Distribution of one variable | **Histogram** or **box plot** | Reveals shape, spread, skew, outliers — a mean alone hides these |
| Ranked list of many categories | **Horizontal bar** (sorted) | Labels fit; sorting makes rank obvious |
| Magnitude across two categorical dims | **Heatmap** | Dense grid; color encodes value (with redundant labels) |
| A single number that matters most | **Big number / KPI** + sparkline | The headline metric needs no axis |

**Decision shortcuts:**
- Time on the **horizontal axis**, running left→right. Never skip periods even with missing values (gap the line; don't compress the axis).
- Sort bar/column charts **by value**, not alphabetically — comparison becomes effortless. *Exception:* ordinal categories (age bands, days of week, Likert) keep natural order.
- Horizontal bars when category labels are long (no rotated text) or when there are many categories.
- ≤4 lines on one chart; beyond that, gray all but the highlighted one (explanatory) or split to small multiples (exploratory).

> Heuristic: if you can't state the question the chart answers in one sentence, you've picked decoration, not a chart.

## 2. Charts to avoid (and what to use instead)

| Avoid | Why it lies | Use instead |
|---|---|---|
| **Pie / donut** (>3 slices) | Humans compare angles/areas poorly; near-equal slices are indistinguishable | Sorted horizontal bar |
| **3D anything** | Perspective makes near slices/bars look bigger regardless of value — a built-in lie factor | Flat 2D equivalent |
| **Dual-axis (two y-axes)** | The crossover point and relative scale are arbitrary — you can make any two series "correlate" | Small multiples (two panels sharing the x-axis) |
| **Truncated-axis bar** | Length encodes value; cutting the baseline multiplies the perceived difference | Zero-baseline bar; if you need detail, switch to a line |
| **Radar / spider** | Area scales with the square of values; axis order changes the shape | Grouped bar or parallel coordinates |
| **Stacked area with many bands** | Only the bottom band has a flat baseline; middle bands are unreadable | Small-multiple lines |
| **Word clouds** | Size conflates frequency with word length; no order | Sorted bar of term frequency |

If you *must* ship a pie (stakeholder mandate, 2–3 categories): no 3D, no exploding slices, no leader lines, no legend; sort largest→smallest from 12 o'clock clockwise; gray non-key slices; direct-label percentages.

## 3. Honest axes & scales

### 3.1 The zero-baseline rule (and its exception)
- **Bar/column → always start at zero.** Bars encode value by length; a truncated baseline inflates differences. A 5% revenue rise on an axis starting at $95M instead of $0 can read as a **~50% jump** — an order-of-magnitude visual inflation. A doubled quantity is a 2× bar with a zero baseline; truncate and the same doubling looks **4–5× bigger**.
- **Line charts → a non-zero baseline is acceptable** and often better for time series, because lines encode by slope/position, not length.

**The Baseline Paradox** — a zero baseline is *not* automatically honest. Tufte's guidance for time series is to frame the *data*, not force the zero point. A global-temperature anomaly chart forced to a 0°F floor, or a fever curve on a 0–100°C scale, hides the life-or-death signal — the relevant variation lives in a 1–2° band that a zero baseline flattens to a flat line. For a framed-range time-series *line*, zero is the lie. (This is why the zero rule binds *bars*, which encode by length, not *lines*, which encode by slope.)

> Rule of thumb: **bars → zero, always. Lines → frame the meaningful range, but signal any truncation.**

### 3.2 The Lie Factor (Tufte)
```
Lie Factor = (size of effect shown in graphic) / (size of effect in the data)
```
- **= 1.0** truthful (Tufte's acceptable band: **0.95–1.05**) · **> 1.05** exaggerates · **< 0.95** obscures.
- Canonical example: Tufte's 1978 **NYT fuel-economy chart** — data rose 53% (18 → 27.5 mpg) but the perspective graphic grew 783%, giving Lie Factor **14.8**. (His oil-barrel volume example reaches ~48.8.)
- Related: the Graph Discrepancy Index (GDI, **Steinbart 1989** — Wikipedia's "1998" is a propagated citation error) ranges −100% to +∞; **0% = properly constructed**; anything outside **±5%** is considered distorted. Note: later work (Mather, Mather & Ramsay) shows GDI is inconsistent near its bounds — treat it as a rough flag, not a precise score.

### 3.3 Truncation discipline
- **Labeling the truncated axis is NOT a sufficient defense.** Most readers don't check axis numbers — they read the *shape*. The creator owns the obligation not to mislead the careless reader.
- When you truncate a **line** chart for legitimate reasons, **signal it**: an axis break / jagged-edge mark, and document the range in a caption.
- **Workflow:** draw every chart with a zero baseline *first*, then consciously decide whether to truncate — so you can see exactly how much truncation distorts the reading before shipping.

### 3.4 Log scales
- Use a log scale only for data spanning **multiple orders of magnitude** or exponential growth. A **straight line on a log chart = exponential (constant-rate) growth**.
- On a log scale, **bars must start at 1, not 0** — a log axis has no zero (log 0 is −∞), so there is no honest bar baseline. Better: don't put bars on a log axis at all; use lines/dots.
- Always label the axis as logarithmic and use clear decade gridlines (1, 10, 100, 1000); readers misread log axes as linear.

## 4. Color for data

Color in charts is a *data encoding*, not decoration. Three palette families, each for a data type:

| Palette type | Data it encodes | Example |
|---|---|---|
| **Sequential** | Ordered / continuous, single direction (low→high) | Light blue → dark blue (population density) |
| **Diverging** | Ordered with a *meaningful midpoint* | Blue ← white → red (temperature anomaly, profit/loss around 0) |
| **Categorical** | Unordered groups | Distinct hues for product lines |

**Rules:**
- **Categorical ceiling ≈ 5–6 colors.** Past that, group small categories into "Other" or switch to a sorted bar with direct labels.
- **Default safe pairing: blue + orange** (the classic red/green replacement). Teal/magenta also works. **Avoid red/green, pink-turquoise-grey, and purple-blue** — these collapse under common color-vision deficiencies.
- **Never color-only** (§ TL;DR). Add line style (solid/dashed/dotted), marker shape (circle/square/triangle), hatching/texture, or direct labels.
- For **sequential/diverging**, use a perceptually uniform ramp (ColorBrewer, Viridis, or build in OKLCH by stepping L) so equal value steps look equal.

**Two validation tests on every palette:**
1. **Grayscale test** — desaturate; if segments become one gray blob, you're relying on hue alone. Fix it.
2. **CVD simulator** — Color Oracle, Sim Daltonism, Chrome DevTools (Rendering → Emulate vision deficiencies), Adobe Color, or Coolors.

**Contrast (WCAG):** bars, lines, and markers are *non-text graphical objects* → need **3:1** against their background and against adjacent series. Text labels <18pt need **4.5:1**. Watch for "ghostly" light-gray lines on white that fail 3:1 and vanish on projectors/sunlight/small screens.

```css
/* Two series, distinguished by hue AND dash AND marker — never hue alone */
.series-revenue { stroke: #1f77b4; stroke-dasharray: none;   } /* solid, circle */
.series-cost    { stroke: #ff7f0e; stroke-dasharray: 6 4;    } /* dashed, square */
/* markers set per-series in JS: 'circle' vs 'rect' */
```

> Prevalence to design for: color-vision deficiency affects **~1 in 12 men (~8%)** and **~1 in 200 women (~0.5%)**; WHO (2022) puts significant disability at **~1 in 6 (~16%)** of people. A chart that fails colorblind users fails roughly 1 in 12 of your male audience.

## 5. The annotation layer

**Charts show WHAT happened; annotations explain WHY.** The annotation layer — callouts, arrows, shaded regions, reference lines — is among the most underused, highest-leverage tools in editorial viz.

- **Direct-label** the line/bar/segment instead of a legend wherever possible. Removes the lookup cost and helps colorblind users.
- **Reference lines:** target/goal, average, prior-year benchmark, a threshold (e.g., "break-even"). A horizontal target line behind bars instantly shows over/under.
- **Shaded regions** for context: recession bands, a campaign window, "before/after launch."
- **Annotate sparingly and selectively.** Label only the **peak, trough, current value, and the moment a change began**. Over-labeling every point destroys the pre-attentive pull labels are meant to create.
- **Emphasis for explanatory charts:** highlight the one series the story is about; render the rest in light gray. The eye goes where the color is.

```js
// Example annotation intents (library-agnostic):
// 1. direct label at the end of each line: { x: lastX, y: lastY, text: seriesName }
// 2. peak callout:        { x: peakX, y: peakY, text: "Peak: 48.2k (Mar 3)" }
// 3. reference line:      { type: 'y', value: target, label: 'Target', style: 'dashed' }
// 4. shaded region:       { x0: launchX, x1: maxX, fill: 'rgba(0,0,0,0.04)', label: 'Post-launch' }
```

## 6. Interaction: tooltips, hover, brush, zoom — with keyboard & touch parity

Interaction enriches exploratory charts but must never *hold* critical information.

| Interaction | Purpose | Keyboard equivalent | Touch equivalent |
|---|---|---|---|
| **Hover tooltip** | Read exact value at a point | Tab/arrow to focus point → tooltip on focus | Tap point; tooltip persists until tap-away |
| **Brush** (drag-select range) | Filter/zoom to a window | Focus + arrow keys to move/resize selection | Two-handle draggable range |
| **Zoom / pan** | Inspect dense regions | +/− keys, arrow pan | Pinch-zoom, drag-pan |
| **Crossfilter** (linked charts) | Filter all charts from one | Focusable controls drive the same filter state | Tap selections |

**Hard rules:**
- **No critical info hover-only.** Labels, legends, and values that exist only on hover are invisible to keyboard and screen-reader users, and unreachable on touch. Make them visible by default or focus-reachable.
- Every mouse interaction needs a **keyboard** path (focusable points/series, arrow-key navigation, Enter/Space to select) and a **touch** path (tap, drag handles, pinch).
- Tooltips on touch must **persist** (not vanish on touchend) and be dismissible.
- Pair every interactive chart with a **data-table / CSV download fallback** for users who can't or won't interact.
- Respect `prefers-reduced-motion` for animated transitions (§9).

## 7. Chart accessibility

Charts are images of data; a sighted-mouse-user assumption fails ~1 in 6 people (WHO 2022). Layered fallbacks:

1. **`role="img"` + `aria-label`** on the chart container, where the label conveys the **insight**:
   - Bad: `aria-label="A bar chart of sales."`
   - Good: `aria-label="Bar chart of Q3 2025 sales by region. West leads at $2.3M, then East $1.8M, Central $1.4M, South $0.9M."`
2. **Visible text summary** near the chart stating the takeaway (1–2 sentences). Screen-reader users *and* skimmers benefit.
3. **Data-table fallback** (a real `<table>`), toggleable or in a `<details>`, plus CSV download. **But:** a table alone does NOT let anyone spot patterns/trends/outliers — pair it with the descriptive text from #2.
4. **Keyboard navigation** of data points (§6).

**Canvas-rendered charts (Chart.js, ECharts) are NOT accessible by default** — the pixels carry no semantics. The single insight that trips people up: content *inside* `<canvas>` tags is fallback for when canvas is **unsupported** — modern AT does NOT read it when canvas renders successfully, so it is **not** a screen-reader path. You must instead:
- Add `role="img"` + `aria-label` (or `aria-labelledby`) on the `<canvas>` for the insight, **and**
- Provide a *real, separately-rendered* data table (visually-hidden or in a `<details>`) and tie it in with `aria-describedby`:

```html
<canvas id="chart" role="img"
        aria-labelledby="chart-cap" aria-describedby="chart-table"></canvas>
<p id="chart-cap">Line chart: monthly active users grew from 12k in Jan to 31k in Jun 2025.</p>
<details id="chart-table">
  <summary>View data table</summary>
  <table>
    <caption>Monthly active users, 2025</caption>
    <tr><th scope="col">Month</th><th scope="col">MAU</th></tr>
    <tr><td>Jan</td><td>12,000</td></tr>
    <!-- … -->
  </table>
</details>
```

SVG libraries (Recharts, Visx, Plot) are closer to accessible but still need `role="img"` + a meaningful `aria-label` (or `<title>` + `<desc>` referenced via `aria-labelledby`), and don't expose individual `<path>` data to AT by default.

## 8. Explanatory vs exploratory — and narrative / scrollytelling

**The core fork.** Decide this before choosing a chart type or library:

| | Explanatory | Exploratory |
|---|---|---|
| Goal | Make **one** argument | Let users find their own answers |
| Technique | Annotate, emphasize one series, gray the rest, fixed view | Filter, brush, tooltip, zoom, crossfilter |
| Audience | Readers (article, landing page, report headline) | Analysts (dashboard, BI tool) |
| Failure mode | Adding filters that dilute the point | A static chart that can't be interrogated |

Don't build an interactive exploratory tool when you need to make a point, or a locked-down annotated chart when users need to slice their own data.

**Scrollytelling** (data-story chart that animates/changes as the reader scrolls):
- Stack: **Scrollama + IntersectionObserver** is the standard scroll-trigger; **D3 or Observable Plot** drives the sticky visual; authoring tools (Shorthand, Flourish, ScrollyTeller) lower the barrier for non-coders.
- Pattern: a `position: sticky` visual on one side, text "steps" scrolling past on the other; each step fires a transition (highlight a series, zoom, add an annotation).
- Provide a **non-scroll fallback** (reduced-motion: show all states stacked) and keyboard-advanceable steps.
- See [Scroll-Driven Experiences](../03-site-types/scroll-driven-experiences.md) for pinning/IO mechanics and reduced-motion handling.

## 9. Compact patterns: sparkline, bullet, big-number, animated reveals

- **Sparkline** — Tufte's *"data-intense, design-simple, word-sized graphic."* A line with no axes, inline in a table row or sentence. A dashboard table row can hold a dozen, each a full trend. Add optional **start/end dots** and **min/max markers** (color-coded with a redundant marker). Width ~80–120px, height ~20–28px.
- **Bullet graph (Stephen Few)** — a compact gauge replacement: **actual value = dark horizontal bar**, **target = a vertical reference line (the marker)**, behind them **dark/medium/light qualitative bands** (poor/satisfactory/good). Reads in a fraction of the space of a radial gauge, with no ink wasted on the dial.
- **Big number / KPI** — the headline metric large, with a small delta (▲ +12% vs last month, colored *and* arrowed), and a sparkline for context. The delta's sign needs a redundant channel (arrow/word), not color alone.
- **Animated reveals** — a one-time entrance animation (bars growing from baseline, line drawing left→right) aids comprehension *if* it respects the data: bars grow **from the zero baseline up** (never from the top down — that misreads the encoding), 300–600ms, ease-out. **Always gate behind `prefers-reduced-motion`** and never re-animate on every re-render (it nauseates and delays reading).

```css
@media (prefers-reduced-motion: reduce) {
  .chart * { animation: none !important; transition: none !important; }
}
```

## 10. Library selection

Pick by **data scale**, **framework**, and **how custom** the chart is. Renderer matters: SVG is accessible-friendly but slow at volume; Canvas/WebGL is fast but needs manual a11y.

| Library | Renderer | Best for | Avoid when |
|---|---|---|---|
| **Recharts** | SVG | The React default — composable components, ~80% of D3's value in ~10% of the code; dashboards/admin/SaaS | High-frequency updates or >~10k points; responsiveness via `ResponsiveContainer` is imperfect |
| **Visx** (Airbnb) | SVG | D3 math + React rendering; *primitives* to build custom charts; tree-shakable | You want charts out of the box (a pie is ~60 lines vs Recharts' ~20) |
| **Apache ECharts** | Canvas + WebGL | Widest free catalog (heatmap, candlestick, sankey, calendar, treemap, sunburst, 3D), 50k+ points, streaming, BI; excellent docs | You need a first-party React API (use a wrapper) |
| **Chart.js** | Canvas | Marketing pages, decks, quick projects; smooth on mobile; `react-chartjs-2` thin wrapper | Accessibility (none built-in — add `chartjs-plugin-a11y` + manual ARIA) |
| **D3.js** | SVG/Canvas/HTML | The *escape hatch* — radial layouts, force-directed graphs, parallel coordinates with linked brushing | Anything a chart library already does — teams pick D3 "for fame," lose 2–3 weeks, then realize Recharts shipped in ¼ the time |
| **Observable Plot** | SVG | High-level grammar-of-graphics over D3; exploratory/notebook work, far less boilerplate | React without care — charts don't resize by default; needs a `ResizeObserver` |

**Niche / performance picks:** **uPlot** (tiny, fast time-series/lines), **Lightweight Charts + CandleKit** (financial/candlestick, canvas), **SciChart / LightningChart** (paid, WebGL, telemetry-grade), **Highcharts / AG Charts / ApexCharts** (commercial-friendly; AG Charts demos **1M points interactive at ~60fps**), **Plotly** (prototyping parity with Dash/Colab — but `react-plotly` is widely considered abandoned/broken; use `plotly.js` directly).

**Accessible-by-default authoring tools:** **Datawrapper** and **Flourish** ship charts with table/download fallbacks built in — ideal for editorial/newsroom output without custom a11y work.

> Selection algorithm: standard React dashboard → **Recharts**. Outgrow it but want React → **Visx**. >10k points / exotic chart types / streaming → **ECharts** (or uPlot for pure time-series). Truly bespoke layout no library has → **D3**, last resort.

## Concrete specs & numbers
- **Lie Factor** target **1.0** (honest band **0.95–1.05**); Tufte's 1978 NYT fuel-economy chart = **14.8** (783% graphic / 53% data). A framed-range temperature line can flatten to a flat line under a zero baseline yet the framed version is the honest one (Baseline Paradox).
- **GDI (Steinbart 1989):** range −100% → +∞; **0% = correct**; outside **±5% = distorted** (but inconsistent near its bounds — rough flag only).
- **Color-vision deficiency:** ~**1 in 12 men (~8%)**, ~**1 in 200 women (~0.5%)**; WHO 2022: ~**1 in 6 (~16%)** of people have significant disability.
- **WCAG contrast:** **4.5:1** normal text (<18pt / <14pt bold); **3:1** large text and non-text graphical objects (bars, lines, markers).
- **Categorical color ceiling:** ~**5–6** before distinguishability breaks down.
- **Axis inflation:** a 5% rise on a $95M-baseline axis can read as ~50%; a doubling on a truncated axis can look **4–5× bigger**.
- **Renderer limits:** SVG libs choke around **~10k+ points**; Canvas/WebGL handle **50k–1M points at ~60fps**. AG Charts demos **1M points** interactive at ~60fps.
- **Sparkline** dimensions: ~80–120px wide, ~20–28px tall; **bullet graph** ≈ same height, replaces a full gauge.
- **Reveal animation:** 300–600ms, ease-out, bars grow from the zero baseline; gated on `prefers-reduced-motion`.

## Tools & libraries
- **Charting:** D3, Observable Plot, ECharts, Recharts, Visx, Chart.js, uPlot, Lightweight Charts, AG Charts, Highcharts, ApexCharts, Plotly (use `plotly.js` directly), SciChart, LightningChart.
- **Palettes:** ColorBrewer (research-validated sequential/diverging/categorical), Viridis (perceptually uniform).
- **Colorblind checks:** Color Oracle, Sim Daltonism, Chrome DevTools CVD simulator, Adobe Color, Coolors.
- **Accessible-by-default chart builders:** Datawrapper, Flourish.
- **Annotation:** ChartAccent, D3/ggplot2 annotation helpers.
- **Scrollytelling:** Scrollama + IntersectionObserver; Shorthand, Flourish, ScrollyTeller for authoring.

## Do / Don't
- **Do** start every bar at a zero baseline; **Don't** truncate a bar axis ever.
- **Do** use a non-zero baseline on time-series *lines* to show the data; **Don't** force a line to zero if it flattens the real signal.
- **Do** sort bars by value; **Don't** order them alphabetically (unless ordinal).
- **Do** direct-label series; **Don't** rely on a legend that forces color cross-referencing.
- **Do** add a redundant channel to every color encoding; **Don't** ship color-only.
- **Do** keep categorical palettes ≤5–6; **Don't** assign 12 hues and hope.
- **Do** write insight-bearing alt text; **Don't** write "a chart showing data."
- **Do** pair interactive charts with a table *and* a text summary; **Don't** make a raw table the only fallback.
- **Do** pick Recharts/ECharts for the job; **Don't** reach for D3 by default.
- **Do** animate reveals from the baseline behind `prefers-reduced-motion`; **Don't** re-animate on every render.

## Common pitfalls & how to avoid them
- **Truncated y-axis on bars** (the Chevy Trucks "starts at 95%" ad) — the single most common manipulation. Fix: zero baseline, always.
- **3D / pie "for visual interest"** — perspective and angle judgments distort proportion. Fix: flat bars.
- **Dual-axis line charts** implying a relationship between two unrelated scales (spotted even in r/dataisbeautiful "OC" posts). Fix: small multiples.
- **Pies with too many / near-equal / unsorted slices** — impossible to compare. Fix: sorted bar.
- **Color-only encoding + a color legend** — doubly broken for colorblind users. Fix: redundant channel + direct labels.
- **"Ghostly" light-gray lines/labels** failing 3:1 — vanish on projectors/sunlight/small screens. Fix: meet 3:1 for graphics.
- **Hover-only tooltips holding critical values** — invisible to keyboard/screen-reader/touch. Fix: visible-by-default or focus-reachable + table fallback.
- **Generic alt text** — conveys structure, not insight. Fix: state the takeaway and the numbers.
- **D3-by-default** burning weeks — Fix: high-level library first; D3 only when nothing else fits.
- **SVG library on a large/streaming dataset** chokes past ~10k points — Fix: Canvas/WebGL (ECharts/uPlot).
- **Table-only accessibility** — satisfies some screen-reader needs but no one can spot trends/outliers, the whole point of viz. Fix: pair with descriptive text.
- **Over-labeling / over-erasing** — labeling every point draws attention nowhere; over-applying Tufte's data-ink erasure makes charts unreadable ("within reason," Tufte cautioned). Fix: label peak/trough/now/change-point only; erase gridlines/3D/borders but keep what aids reading.

## What real users complain about
- **r/reactjs strongly converges:** **Recharts is the default** for standard React dashboards ("80% of D3 in 1/10 the code", "had a day to implement 3–4 bar charts, minimal struggle"); **Visx** when you outgrow it ("create complex unique designs otherwise impossible"); **ECharts/canvas** for large/high-performance datasets.
- Practitioners gripe that **Recharts' `ResponsiveContainer`** responsiveness is "still imperfect" and needs wrangling.
- **Observable Plot in React** "doesn't resize by default" — teams hit this and bolt on a `ResizeObserver`.
- **`react-plotly` is considered abandoned/broken**; the working path is `plotly.js` directly.
- Repeated regret over **picking D3 for fame** and losing 2–3 weeks on work a chart library would have shipped in a quarter of the time.
- Readers in **r/dataisbeautiful** routinely call out **dual-axis and truncated-axis** "OC" as misleading even when well-rendered — the chart-type sin outweighs polish.

## Senior-level checklist
- [ ] Can I state, in one sentence, the question this chart answers?
- [ ] Chart type matches the question (comparison→bar, trend→line, relationship→scatter, distribution→histogram)?
- [ ] No pie >3 slices, no 3D, no dual-axis, no radar?
- [ ] Bars start at zero; line baseline frames the real signal (and any truncation is signaled)?
- [ ] Lie Factor ≈ 1; I drew the zero-baseline version first to check distortion?
- [ ] Log axis only for multi-order data, bars start at 1 (or no bars), axis clearly labeled log?
- [ ] Palette: correct family (sequential/diverging/categorical), ≤5–6 categorical, blue+orange default, no red/green?
- [ ] Passes the grayscale test AND a CVD simulator; redundant channel on every color?
- [ ] Bars/lines/markers ≥3:1 contrast; text labels ≥4.5:1; no ghostly grays?
- [ ] Direct labels instead of (or in addition to) a legend?
- [ ] Annotations limited to peak/trough/current/change-point; reference lines for targets/benchmarks?
- [ ] Explanatory vs exploratory decided — and the build matches it?
- [ ] No critical info hover-only; keyboard + touch parity for every interaction?
- [ ] `role="img"` + insight-bearing `aria-label`; visible text summary; table + CSV fallback?
- [ ] Canvas charts have ARIA / in-canvas table fallback (not automatic)?
- [ ] Library matches data scale (SVG <10k, Canvas/WebGL beyond); not D3-by-default?
- [ ] Reveal animations from baseline, ≤600ms, gated on `prefers-reduced-motion`, no re-animate on render?

## Sources
- Edward Tufte, *The Visual Display of Quantitative Information* (Lie Factor, the 14.8 NYT fuel-economy example, data-ink ratio, baseline guidance, sparklines).
- Stephen Few, *Information Dashboard Design* (bullet graphs, dashboard chart selection).
- P. J. Steinbart (1989), "The Auditor's Responsibility for the Accuracy of Graphs..." — origin of the Graph Discrepancy Index (GDI).
- WHO, *Global report on health equity for persons with disabilities* (2022) — the 1.3 billion / ~1 in 6 (~16%) figure: https://www.who.int/news-room/fact-sheets/detail/disability-and-health
- WCAG 2.2 — 1.4.1 Use of Color, 1.4.3 Contrast (Minimum), 1.4.11 Non-text Contrast: https://www.w3.org/TR/WCAG22/
- ColorBrewer (Cynthia Brewer): https://colorbrewer2.org/
- Datawrapper Academy (chart honesty, accessibility): https://academy.datawrapper.de/
- Apache ECharts: https://echarts.apache.org/
- Recharts: https://recharts.org/
- Visx (Airbnb): https://airbnb.io/visx/
- Observable Plot: https://observablehq.com/plot/
- Chart.js: https://www.chartjs.org/
- D3.js: https://d3js.org/
- Scrollama: https://github.com/russellgoldenberg/scrollama

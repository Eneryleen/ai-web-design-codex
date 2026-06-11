# Content Sites: Blogs, Magazines & Docs

> Reading-first interfaces where typography *is* the UI: blogs, editorial magazines, and technical documentation. This guide gives the concrete measure/leading/scale numbers, the Diataxis IA model, code-block and search patterns, and the dark-mode reality so you can build long-form sites that people actually finish reading.

**Apply when:** building a blog, news/magazine, knowledge base, or developer docs — any site where the primary job is reading text. · **Related:** [Web Typography Systems](../00-foundations/typography-systems.md) · [Fluid Typography & Spacing with clamp()](../02-responsive-adaptive/fluid-type-and-spacing.md) · [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md)

## TL;DR — rules at a glance
- **Measure = 50–75 chars/line, target 66.** Cap body width with `max-width: 65ch`. Wider lines lose the return sweep; narrower fragments rhythm.
- **Body line-height 1.5–1.6 desktop, 1.3–1.45 mobile.** Longer lines need more leading; short lines need less.
- **Heading line-height 1.1–1.2; large display down to ~1.0** with ~`-0.02em` to `-0.04em` letter-spacing.
- **Body font-size 16px minimum, 18px optimal** for long-form. Below 16px the dark-mode reading penalty grows.
- **Use a Perfect Fourth (1.333) modular scale** for general content; reserve the Golden Ratio (1.618) for headline-driven editorial only.
- **Contrast: 4.5:1 body / 3:1 large text (≥24px or bold ≥18.7px) for AA; 7:1 for AAA.** Verify, don't eyeball.
- **TOC goes in the LEFT rail, lists *every* heading, labels match headings exactly, and highlights the current section on scroll.** Right-rail TOCs get ignored as "ads."
- **Never make the in-body TOC sticky** (it fights global nav). A left sidebar TOC may be sticky; a TOC inside the article column may not.
- **Never skip heading levels for sizing** (H2→H4). Fix size with CSS; semantic order is for screen readers.
- **Structure docs with Diataxis:** Tutorial / How-to / Reference / Explanation — keep the four types strictly separated.
- **Syntax highlighting: use Shiki**, not Prism, for new projects; desaturate dark-mode token colors (neon fails perceptually even at 4.5:1).
- **Make `<pre>` blocks horizontally scrollable AND keyboard-focusable** (`tabindex="0"`).
- **Light mode wins for reading** for most users; ship a dark-mode toggle but default thoughtfully.
- **Show estimated read time** (baseline 265 WPM) — cheap to build, high UX payoff.
- **Match density to user:** whitespace for long-form readers, density for power users (devs/analysts). Don't apply one spacing model to both.
- **Avoid animation on content sites.** Subtle fades or nothing; readers find motion annoying, not delightful.

---

## Reading experience: measure, leading, scale

The reading column is the product. Three numbers govern legibility: **measure** (line length), **leading** (line-height), and the **scale** (size relationships).

### Measure (line length)
- Optimal range: **50–75 characters per line**, including spaces. **66 is the canonical sweet spot** (Bringhurst).
- Below ~45 chars the eye returns too often, fragmenting rhythm. Above ~80 chars the return sweep to the next line's start becomes error-prone (you re-read or skip lines).
- Cap with character units so the limit tracks font size:

```css
.prose {
  max-width: 65ch;     /* ≈ 66 chars of Latin body text */
  margin-inline: auto;
}
/* rough em fallback for older engines: max-width: 38em (ch ≈ 0.58em in proportional fonts) */
```

- `ch` = width of the `0` glyph in the current font, so `65ch` shifts correctly across fonts/sizes. For wide-glyph or condensed fonts, sanity-check against real content (lorem ipsum lies about measure).

### Leading (line-height)
- **Body: 1.5–1.6 on desktop, 1.3–1.45 on mobile** (shorter mobile measure needs less leading).
- **Headings: 1.1–1.2.** Display/hero headings compress toward **1.0** because few chars per line need tight leading.
- Rule of thumb: **leading scales with measure** — wider columns demand more line-height to keep the eye on track.
- Set `line-height` unitless so it inherits proportionally.

```css
body      { line-height: 1.6; }
@media (max-width: 640px) { body { line-height: 1.4; } }
h1, h2, h3 { line-height: 1.15; }
.display   { line-height: 1.0; letter-spacing: -0.03em; }
```

### Modular scale
- Default to a **Perfect Fourth (×1.333 per step)** — clear hierarchy without shrinking body text or exploding headings.
- **Golden Ratio (×1.618)** suits magazine covers / headline-driven editorial but produces uncomfortably large jumps for ordinary article bodies.
- Example Perfect Fourth scale from an 18px base: 18 → 24 → 32 → 42 → 56px. Round to whole pixels.

```css
:root {
  --step-0: 1.125rem;             /* 18px body */
  --step-1: clamp(1.4rem, 1.1rem + 1.5vw, 1.6rem);
  --step-2: clamp(1.85rem, 1.4rem + 2.3vw, 2.13rem);
  --step-3: clamp(2.4rem, 1.7rem + 3.5vw, 2.84rem);
}
```

- Use `clamp()` for fluid type so sizes scale smoothly instead of jumping at breakpoints. See [Fluid Typography & Spacing](../02-responsive-adaptive/fluid-type-and-spacing.md).

---

## Typographic hierarchy for long-form

Hierarchy in long-form is about **scannability and rhythm**, not decoration. Readers skim headings first, then commit.

- **One weight contrast carries emphasis only against a plain-weight baseline.** If body text is already bold, additional bold signals nothing. Keep body at regular (400/450) so bold (600/700) means "important."
- **Don't over-bold.** Bold for genuine emphasis and run-in lead-ins only; not for whole paragraphs.
- **Establish vertical rhythm**: consistent space-before/after for headings tied to the baseline. Generous space *above* a heading, less *below* it — the heading should visually attach to the content it introduces (proximity/Gestalt).
- **Limit hierarchy depth.** Most articles need H1 (title) + H2 (sections) + occasionally H3. Going past H4 in prose signals the content should be split.
- **Heading levels are semantic, never visual.** Never jump H2→H4 because "H3 looks too big." Resize H3 with CSS. Skipping levels breaks screen-reader navigation and document outline.
- **Lead paragraph / standfirst** (magazine pattern): slightly larger (e.g. `1.25em`), often a touch lighter, to ease entry.
- **Drop caps / pull quotes** are editorial spice — use sparingly; they must not break the reading column's left edge alignment for body text.
- **Links** must be distinguishable without relying on color alone (underline or sufficient non-color cue) for WCAG; see [Accessibility](../01-ux-fundamentals/accessibility-wcag.md).

---

## Table of contents (TOC)

The TOC is a contract: **links are promises**. Break the contract and readers lose trust and stop using it.

- **Placement: left rail.** Right-rail TOCs suffer "ad-blindness" — users trained to ignore right columns scroll past them in usability tests.
- **Label = heading, exactly.** Any mismatch between TOC text and the section heading forces a cognitive comparison and breaks reading flow. Generate TOC labels *from* the headings.
- **Include ALL section headings**, including above-the-fold ones, so the TOC is an accurate map of page structure — not a partial one.
- **Highlight the current section on scroll** (TIME magazine pattern: bold the active entry). Implement with `IntersectionObserver`, not scroll-position math.
- **In-page anchors only — no external links.** External links in a TOC violate the pattern's contract.
- **Sticky rules:**
  - A **left-sidebar** TOC (its own column) **may be sticky** so it stays visible.
  - A TOC rendered **inside the article body column must NOT be sticky** — it competes with global nav and creates z-index/scroll confusion.
- **Back-to-top affordance** is essential on long pages to avoid endless scrolling between distant sections.
- **Mobile:** collapsible TOCs are easily *missed* — surface them explicitly (a labeled "On this page ▾" toggle), don't hide them behind an unmarked icon.

```js
// Highlight current TOC entry: track which headings are above the viewport
// and activate the last one — handles forward AND backward scrolling.
const headings = [...document.querySelectorAll('h2[id], h3[id]')];
const links = new Map(
  [...document.querySelectorAll('.toc a')].map(a => [a.hash.slice(1), a])
);

function setActive(id) {
  document.querySelector('.toc a.active')?.classList.remove('active');
  links.get(id)?.classList.add('active');
}

const io = new IntersectionObserver(() => {
  // Walk headings in order; the last one whose top is above (or at) the
  // viewport midpoint is the "current" section.
  const mid = window.scrollY + window.innerHeight * 0.25;
  let current = headings[0];
  for (const h of headings) {
    if (h.offsetTop <= mid) current = h;
    else break;
  }
  if (current) setActive(current.id);
}, { rootMargin: '-25% 0px -50% 0px', threshold: 0 });

headings.forEach(h => io.observe(h));
```

---

## Sticky navigation

- **Global header / breadcrumbs** may be sticky on docs to keep navigation reachable on long pages; keep it slim (one row).
- **Reserve scroll-margin** so anchor jumps don't land under the sticky header:

```css
:target, h2[id], h3[id] { scroll-margin-top: 5rem; }
html { scroll-behavior: smooth; } /* respect reduced-motion below */
@media (prefers-reduced-motion: reduce) { html { scroll-behavior: auto; } }
```

- **Don't stack sticky things.** A sticky header + sticky in-body TOC + sticky CTA = layered chaos. Pick one persistent element per axis.
- **Reading progress bar** (top of viewport) is acceptable on long articles; keep it 2–4px and non-distracting.

---

## Code blocks & syntax highlighting

For docs and technical blogs, code blocks are primary content — treat them with the same care as prose.

- **Use Shiki** (VS Code's TextMate engine) for new projects. It uses inline styles (no separate theme CSS to load), supports **dual light/dark themes**, and produces accurate highlighting. It's the default in Astro.
- **Avoid Prism.js for new builds** — class-based (needs theme CSS loaded separately) and can cause client-side perf issues on large code blocks. Fine to keep in legacy.
- **Build-time highlighting** beats client-time: use `rehype-pretty-code` (Shiki-powered) in MDX/Markdown pipelines, or **Expressive Code** in Astro for annotations, line-marking, and copy buttons.
- **Desaturate dark-mode token colors.** Highly saturated neon on dark backgrounds appears to "blur" / vibrate and causes fatigue **even when nominally ≥4.5:1** (documented in Gradle/Asciidoctor issues; e.g. a borderline 4.59:1 saturated blue). Pick muted, slightly desaturated palettes for dark themes.
- **Accessibility of `<pre>`:** code blocks that scroll horizontally must be **keyboard-focusable** so keyboard-only users can scroll them:

```html
<pre tabindex="0"><code class="language-ts">…</code></pre>
```
```css
pre { overflow-x: auto; }
pre:focus-visible { outline: 2px solid currentColor; outline-offset: 2px; }
```

- **Copy button** on every block; provide accessible label and a visible "Copied" state.
- **Preserve indentation and never line-wrap code by default** (wrapping corrupts meaning for whitespace-significant languages). Offer a wrap toggle if needed.
- **Inline code** needs its own subtle background + monospace + slightly reduced size (~0.9em) and must still meet contrast against that background.

---

## Search

- **Docs need real search.** For OSS docs, **Algolia DocSearch** (free for OSS with logo display) gives sub-20ms responses at global scale; v4 adds an AskAI conversational layer and composable API. Default crawl is weekly, manually re-triggerable.
- **For static Astro/Starlight sites, use Pagefind** — fully client-side, static, no external service, no API keys.
- **Surface zero-result queries.** Algolia's analytics dashboard exposes queries that return nothing — these are your content-gap and synonym priorities. The #1 search complaint is "no/irrelevant results when my words ≠ the docs' words"; fix with synonyms and content, informed by analytics.
- **Keyboard-first:** `/` or `Cmd/Ctrl-K` to focus search; results navigable by arrow keys; `Esc` to close. DocSearch ships WCAG-2.1-compliant and keyboard-accessible out of the box.
- **Scope and rank** results by section/version when relevant; show the breadcrumb path in each result so users know *where* a hit lives.

---

## Dark mode for reading

The honest position from research (NNG, Papadimitriou et al. 2021 *Applied Ergonomics*):

- **Light mode outperforms dark mode** for visual acuity and proofreading tasks for users with **normal vision**, particularly at **small font sizes**. The effect is most pronounced in bright ambient environments where the light background reduces halation.
- **Dark mode can reduce eye strain** in low-ambient-light situations (late night, dimmed rooms) — but this is a comfort preference, not a documented acuity advantage for normal-vision users.
- The **light-mode advantage grows as font size shrinks** — another reason to keep body ≥16px and not ship tiny dark text.
- **Therefore:** ship a **user-controlled toggle** for long-form, respect `prefers-color-scheme`, persist the choice, and don't assume dark = better. Some users need it (light sensitivity, low vision, OLED battery, preference).

```css
:root { color-scheme: light dark; }
/* token-driven theme so a class/attr flip swaps both modes */
:root[data-theme="dark"] { --bg: #14161a; --fg: #e6e6e6; }
```

- In dark mode, **don't use pure white (#fff) text on pure black (#000)** — it causes halation (text bleeds). Use off-white on dark-gray (e.g. `#e6e6e6` on `#14161a`) and reduce text contrast slightly from the absolute maximum.

---

## Content density

Density is a function of **who reads and why** — there is no universal spacing.

- **Long-form readers need whitespace.** Generous leading and paragraph spacing measurably reduce eye-tracking fixations and re-read events — the mechanism is reduced crowding between lines and clearer sentence boundaries. The specific percentages quoted in marketing posts vary wildly depending on methodology; treat "whitespace helps" as a principle, not a number.
- **Power users need density.** Devs reviewing PRs, analysts scanning reference tables, and API-reference readers want more on screen — don't impose marketing-page airiness on a reference table.
- **Two kinds of whitespace, both intentional:**
  - **Active whitespace** = strategic spacing for emphasis (margins isolating a callout/CTA).
  - **Passive whitespace** = baseline legibility spacing (line-height, letter-spacing, paragraph spacing).
- **Failure mode A — over-whitespacing on desktop:** mobile-first padding that renders hollow on a 1440px monitor; users scroll through "air." Cap line length and tune desktop spacing separately.
- **Failure mode B — too dense:** wall-of-text technical docs raise cognitive load exactly where comprehension demand is highest. Break with subheads, lists, and callouts.
- **Paragraph spacing:** prefer margin between paragraphs over first-line indents on the web; pick one, not both.

---

## Documentation IA: Diataxis

Diataxis splits docs into **four types by user need**, and the core rule is **keep them separate** — mixing types on one page is the most common docs-IA failure.

| Type | User goal | Mode | Voice |
|------|-----------|------|-------|
| **Tutorial** | *Learn* by doing, hand-held | Learning / acquisition | "We will… now do this." Guaranteed-to-work lesson. |
| **How-to guide** | *Accomplish a task* (already skilled) | Task / application | "To do X, do these steps." Goal-oriented recipe. |
| **Reference** | *Look up* precise facts | Information / application | Neutral, exhaustive, dry. Describes the machinery. |
| **Explanation** | *Understand* the "why" / context | Understanding / acquisition | Discursive; background, alternatives, rationale. |

- **The classic mistake:** a "how-to" page that also tries to teach concepts (explanation) and lists every option (reference). Readers must read the whole page to find their answer. Split it.
- **Label clearly** so navigation telegraphs the type — users shouldn't have to read a page to learn what kind of page it is.
- **Implement iteratively, not big-bang.** A top-down "audit everything first" restructure tends to stall. Find the highest-impact page, fix one thing, repeat. (Practitioner consensus.)
- **LLM-assisted audits** are practical for triage: prompt a model with the four type definitions and ask it to classify each page and flag tonal mixing. Useful for large-scale discovery passes, not a substitute for editor judgment.

### Page furniture for docs
- **Breadcrumbs** on every page (where am I in the tree).
- **Left sidebar = product/section nav**, organized by Diataxis category or by feature with type labels.
- **Right or in-page = "On this page" TOC** for the current document.
- **Prev/Next** within a sequence (tutorials especially).
- **Version switcher** if multiple versions are live.
- **"Edit this page" / last-updated date** for trust.

---

## Estimated read time

- Cheap to build, real UX payoff: lets readers decide **read now vs. bookmark**.
- **Baseline 265 WPM** (adult average, Medium's standard).
- Better algorithms factor **syllable count, average sentence length, and a difficulty multiplier** (technical/code-heavy content reads slower). Add time for code blocks and images.

```js
/**
 * Estimate reading time.
 * @param {string} markdown - raw markdown/HTML source
 * @param {object} opts
 * @param {number} opts.wpm        - prose reading speed (default 265)
 * @param {number} opts.codeWpm    - code reading speed (default 100)
 * @param {number} opts.secsPerImg - seconds to view each image (default 12)
 */
function readMinutes(markdown, { wpm = 265, codeWpm = 100, secsPerImg = 12 } = {}) {
  // Strip and count fenced code blocks separately (slower to read)
  let codeSecs = 0;
  const noCode = markdown.replace(/```[\s\S]*?```/g, (block) => {
    const codeWords = block.trim().split(/\s+/).length;
    codeSecs += (codeWords / codeWpm) * 60;
    return '';
  });

  // Count images
  const imgCount = (noCode.match(/!\[.*?\]\(.*?\)/g) || []).length;

  // Count prose words (strip remaining markdown syntax)
  const prose = noCode.replace(/[#*`_[\]()>~\-]/g, ' ');
  const words = prose.trim().split(/\s+/).filter(Boolean).length;

  const totalSecs = (words / wpm) * 60 + codeSecs + imgCount * secsPerImg;
  return Math.max(1, Math.round(totalSecs / 60));
}
```

- Display as "≈ 7 min read." Round to whole minutes; never show "0 min."
- Note: reading speed varies significantly with age, content difficulty, font choice, and environment — so read time is an estimate, not a promise. Treat the displayed figure as directional.

---

## Concrete specs & numbers
- **Measure:** 50–75 chars/line; **66 ideal**; `max-width: 65ch`.
- **Body line-height:** 1.5–1.6 desktop; 1.3–1.45 mobile.
- **Heading line-height:** 1.1–1.2; display ~1.0.
- **Body font-size:** ≥16px; 18px optimal for long-form.
- **Display letter-spacing:** ~`-0.02em` to `-0.04em` on large headings only.
- **Modular scale:** Perfect Fourth ×1.333/step (default); Golden Ratio ×1.618/step (editorial only, too aggressive for body).
- **Contrast (WCAG 2.1/2.2 AA):** 4.5:1 normal text; 3:1 large text (≥18pt/24px, or bold ≥14pt/18.7px) and UI/graphics.
- **Contrast (AAA):** 7:1 normal text.
- **Syntax-highlight pitfall:** saturated colors can pass 4.5:1 numerically yet read poorly on dark bg (Gradle 4.59:1 borderline case) — desaturate.
- **Reading speed baseline:** ~265 WPM adult average; significant variation by content type, font, and reader — treat displayed read times as directional.
- **Algolia DocSearch:** sub-20ms typical; free for OSS (logo required); default crawl weekly, manually re-triggerable.
- **scroll-margin-top:** match sticky header height (~5rem typical) so anchors don't hide.

---

## Tools & libraries
- **Shiki** — syntax highlighter, v1.0 (Feb 2024); inline styles, lazy grammar/theme load, dual light/dark; default in Astro. **Use for new projects.**
- **Prism.js** — older, class-based, needs theme CSS; possible client perf cost on large blocks. **Legacy only.**
- **rehype-pretty-code** — Shiki-powered Rehype plugin; build-time highlighting in MDX/Markdown.
- **Expressive Code** — Astro integration: annotations, line marking, copy button.
- **Algolia DocSearch v4** — purpose-built docs search; free OSS (logo required); sub-20ms; WCAG-2.1 keyboard accessible out of the box; AskAI conversational layer available.
- **Pagefind** — client-side static search built into Starlight/Astro; no external service. **Use when you want zero backend.**
- **Docusaurus 3.x** — Meta's React+MDX SSG; **only major tool with native multi-version docs**; largest plugin ecosystem; native i18n; pairs with DocSearch. **Use when you need versioning/large ecosystem.**
- **Starlight** — Astro's official docs theme; batteries-included (TOC, sidebar, dark mode, i18n, Pagefind, a11y); strong adoption across OSS projects. **Use for fast, modern, all-in docs.**
- **Nextra** — Next.js docs SSG; ideal if already on Next.js; smaller plugin community.
- **MkDocs Material** — Python, loved for simplicity; `palette.scheme: slate` enables dark mode in one line. Good fit for Python-ecosystem projects. Feature velocity is slower than Docusaurus/Starlight; evaluate against your needs.
- **Contrast verification:** Adobe Color, Color Contrast Analyzer, Leonardo, Chrome DevTools color picker. **APCA** is the emerging perceptual successor to WCAG contrast — track it but ship to WCAG 2.x today.

---

## Do / Don't

**Do**
- Cap measure at ~65ch and verify with *real content*, not lorem ipsum.
- Keep body ≥16px, line-height 1.5–1.6 desktop.
- Put the TOC in the left rail, list every heading, match labels exactly, highlight on scroll.
- Use semantic heading order; resize with CSS.
- Separate Diataxis types and label them.
- Use Shiki with build-time highlighting and desaturated dark tokens.
- Make `<pre>` blocks `tabindex="0"` and horizontally scrollable.
- Ship a dark-mode toggle, respect `prefers-color-scheme`, persist choice.
- Show estimated read time and last-updated date.
- Tune desktop and mobile spacing independently.

**Don't**
- Don't let lines exceed ~80 chars or shrink below ~45.
- Don't bold large swaths of body text (kills emphasis contrast).
- Don't put the TOC in the right rail.
- Don't make an in-body-column TOC sticky.
- Don't skip heading levels for visual sizing.
- Don't mix tutorial/how-to/reference/explanation on one page.
- Don't use neon saturated token colors in dark mode.
- Don't ship pure-white text on pure-black backgrounds.
- Don't default to dark mode assuming it reads better.
- Don't add page transitions/animation to a reading site.
- Don't attempt a big-bang docs audit; iterate.

---

## Common pitfalls & how to avoid them
- **Neon dark-mode highlighting:** passes 4.5:1 but vibrates/blurs → fatigue. *Fix:* desaturate token palette; test on a real OLED/LCD, not just a contrast checker.
- **Right-rail TOC blindness:** users skip it as an ad. *Fix:* left rail.
- **Sticky in-body TOC:** fights global nav, z-index/scroll bugs. *Fix:* only a dedicated sidebar column may be sticky.
- **Mismatched TOC labels:** breaks the "links are promises" contract. *Fix:* generate labels from headings.
- **Skipped heading levels (H2→H4):** breaks screen-reader outline. *Fix:* CSS sizing, correct semantics.
- **Bold overuse:** emphasis stops meaning anything. *Fix:* regular-weight body baseline.
- **Over-whitespacing on desktop:** hollow 1440px layouts, endless scroll. *Fix:* cap measure, tune desktop spacing separately.
- **Too dense in technical docs:** high cognitive load where comprehension matters most. *Fix:* subheads, lists, callouts.
- **Choosing a docs framework without evaluating versioning needs:** only Docusaurus has mature multi-version docs built in. *Fix:* decide versioning requirement first, then pick the tool.
- **Code blocks not keyboard-focusable:** keyboard users can't scroll them. *Fix:* `tabindex="0"` on `<pre>`.
- **Zero-result search with no follow-up:** recurring user frustration. *Fix:* monitor zero-result analytics; add synonyms/content.
- **Dark mode ignoring ambient light / small fonts:** no daytime benefit and worse small-text legibility. *Fix:* keep body ≥16px; make dark a choice.
- **Testing with lorem ipsum:** hides hyphenation, line-break, and rhythm issues. *Fix:* paste real articles before sign-off.
- **Animation on content sites:** the loudest community ask is literally "stop using animations." *Fix:* simple fades or none.

---

## What real users complain about
- **"I can't find the TOC on mobile."** Collapsible/sticky TOCs are invisible unless explicitly surfaced (NNG).
- **"Search returns nothing for my words."** Query vocabulary ≠ docs vocabulary; zero/irrelevant results are a top docs frustration — fixable via synonyms + content informed by Algolia zero-result analytics.
- **"I had to read the whole page to find my answer."** Pages that blend tutorial, reference, and concept without labels — the exact problem Diataxis exists to fix.
- **"I feel lost."** Dense docs with no scannable headings, no TOC, no breadcrumbs — the failure mode of large doc sites that grew organically without IA discipline.
- **"The code colors hurt my eyes."** Garish neon dark-mode highlighting — real GitHub issues filed against Gradle, Asciidoctor, and others.
- **"Reading is a chore."** Too-small body text (14px) and cramped leading; bumping to 16–18px with proper spacing keeps readers longer (practitioner-reported lower bounce).

---

## Senior-level checklist
- [ ] Body measure capped (`max-width: ~65ch`), verified with real article copy.
- [ ] Body ≥16px (ideally 18px); line-height 1.5–1.6 desktop / 1.3–1.45 mobile.
- [ ] Heading line-heights 1.1–1.2 (display ~1.0 with slight negative tracking).
- [ ] Modular scale chosen deliberately (Perfect Fourth default).
- [ ] All text passes AA contrast (4.5:1 / 3:1); critical reading paths checked at AAA where feasible.
- [ ] Heading levels semantically correct — no skipped levels; document outline valid.
- [ ] TOC: left rail, all headings, labels = headings, in-page anchors only, current-section highlight on scroll, back-to-top present.
- [ ] In-body TOC not sticky; only a dedicated sidebar may be sticky; `scroll-margin-top` matches sticky header.
- [ ] Docs structured by Diataxis with the four types separated and labeled; implemented iteratively.
- [ ] Breadcrumbs, prev/next, version switcher (if multi-version), last-updated date present.
- [ ] Syntax highlighting via Shiki at build time; dark-mode tokens desaturated and re-checked.
- [ ] `<pre>` blocks `tabindex="0"`, horizontally scrollable, copy button with accessible label.
- [ ] Dark-mode toggle honoring `prefers-color-scheme`, persisted; off-white on dark-gray, not #fff/#000.
- [ ] Search wired (DocSearch for OSS / Pagefind for static); keyboard shortcut (`/` or `Cmd-K`); zero-result analytics monitored.
- [ ] Estimated read time shown (265 WPM baseline, code/images weighted).
- [ ] Density matched to audience; desktop and mobile spacing tuned separately.
- [ ] `prefers-reduced-motion` respected; no gratuitous page transitions.

---

## Sources
- Robert Bringhurst, *The Elements of Typographic Style* — measure/66-char ideal (foundational reference).
- Nielsen Norman Group (nngroup.com) — dark vs. light mode reading research; TOC usability ("links are promises"); reading-speed/font findings.
- Diataxis framework — https://diataxis.fr
- Shiki — https://shiki.style
- rehype-pretty-code — https://rehype-pretty.pages.dev
- Expressive Code — https://expressive-code.com
- Algolia DocSearch — https://docsearch.algolia.com
- Pagefind — https://pagefind.app
- Docusaurus — https://docusaurus.io
- Starlight — https://starlight.astro.build
- Nextra — https://nextra.site
- Material for MkDocs — https://squidfunk.github.io/mkdocs-material/
- WCAG 2.1/2.2 contrast (1.4.3 / 1.4.6) — https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html
- APCA — https://git.apcacontrast.com
- Papadimitriou et al. (2021) — "Dark or light interface? The effect of interface contrast on users with and without visual impairment", *Applied Ergonomics* — dark vs light mode acuity data.

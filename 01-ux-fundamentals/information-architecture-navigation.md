# Information Architecture & Navigation

> How to organize site content (the invisible structural backbone) and expose it through navigation UI (the visible layer) so users can find anything in ≤ 3 clicks, always know where they are, and trust where each label leads. Senior outcome: an IA + nav system that survives search-engine deep links, scales past 50 pages without becoming a swamp, and measurably raises task success — not just "fewer clicks."

**Apply when:** designing any site's structure, menus, breadcrumbs, search, or mobile nav · **Related:** [Laws of UX](laws-of-ux.md) · [Nielsen's Heuristics](nielsen-heuristics.md) · [Cognitive Load & Progressive Disclosure](cognitive-load-progressive-disclosure.md) · [E-commerce UX](../03-site-types/ecommerce-ux.md)

## TL;DR — rules at a glance

- **IA ≠ navigation.** IA = how content is organized (taxonomy, grouping, relationships). Navigation = the UI that surfaces it. Design IA first (card sort / tree test), then build nav.
- **Static nav depth ≤ 3 levels.** Any destination reachable in **2–3 clicks** from home. Task time rises with depth.
- **Concave site shape wins:** broad entry → narrower category choices → focused terminal options. Avoid both flat (too many simultaneous choices) and deep (too many clicks).
- **Moderate breadth + clear labels beats both extremes** (Zaphiris & Lee 2002). Group items: 4 groups of 5 feel faster than 20 scattered links (Hick's Law).
- **≤ 7 top-level items without grouping** (cap at 5–7 for best performance; Hick's Law — decision time grows log₂(n+1) with n equally likely options). More than that → group under headings or use a mega menu. Note: the "7±2" figure is Miller's (1956) short-term memory chunk limit, which does not directly govern nav breadth — Hick's Law does.
- **Strong information scent.** Labels must predict their target. Kill vague terms ("Resources", "Solutions", "Explore", "The Hub"). Prefer old familiar words over clever coinages.
- **Every page answers "Where can I go?" AND "Where am I?"** Search deep-links land users anywhere; every page is an entry point. Show active state + breadcrumb on every page.
- **Never hamburger on desktop.** No space constraint justifies hiding nav at 1440px+. Hamburger is mobile-only.
- **Mobile: 3–5 core sections persistently visible** (bottom bar / hybrid). Never hide *all* primary nav behind a hamburger.
- **Breadcrumbs = hierarchy, not history.** Start at Home, link every node except current page, only include nodes with real pages.
- **Search is always visible** on content-heavy sites (full input, not just an icon). Autocomplete with zero-state, fuzzy matching, no scrollbar.
- **Consistency is a cognitive anchor.** Nav must not change position/label/behavior across pages. Inconsistency is one of the top-10 IA mistakes per NNGroup.
- **Touch targets:** 44×44px (Apple HIG), 48×48dp (Material), 24×24 CSS px min spacing (WCAG 2.5.8). Breadcrumb targets ≥ 10mm × 10mm.
- **Don't optimize for raw click count.** Optimize for task success, first-click accuracy, and path directness.

---

## 1. Mental models: the foundation of IA

Users arrive with an existing model of how information *should* be organized — shaped by prior experience with similar sites, physical-world analogies (filing cabinets, store layouts), and language conventions. **IA that contradicts the user's mental model fails even when it is internally logical.**

- **Card sorting reveals the model.** Open card sort: participants group unlabeled cards into piles and name the piles → you learn their vocabulary and grouping instincts. Closed card sort: participants slot cards into predefined categories → you validate whether your proposed structure maps to their model.
- **Tree testing stress-tests the model.** Present a text-only sitemap; ask users to find specific items. No visual design, no nav chrome — pure structural validity. Target ≥ 70% first-click accuracy on key tasks before building UI.
- **Mental model mismatches manifest as:** users looking in the wrong top-level section first, re-reading labels without clicking, using search immediately as a fallback, and saying "I thought it would be under X."
- **Audience segments as the top-level axis is a common trap.** "For Individuals / For Business / For Enterprise" forces users to self-classify before knowing your offerings. Mental model research (NNGroup) consistently shows task-based or content-type labels outperform audience labels unless the offering radically differs by segment.

## 2. IA vs. navigation: two layers, one system

| | Information Architecture | Navigation |
|---|---|---|
| **What** | Invisible structure: taxonomy, grouping, labels, relationships | Visible UI: top nav, sidebar, breadcrumbs, search, footer |
| **Artifact** | Sitemap, content model, card-sort results | Menu components, link styling, active states |
| **Failure mode** | "One big swamp" — no organizing structure | Invisible / inconsistent / animated menus |
| **Validate with** | Card sorting, tree testing | First-click testing, usability tasks |

You need **both**, and they must agree. A clean taxonomy hidden behind a bad menu is unusable; a slick menu over a chaotic taxonomy just exposes the chaos faster.

**Build order:** content inventory → open card sort (discover user mental models) → draft IA → tree test (validate findability without visual design) → design navigation → first-click test. Tools: Optimal Workshop, Maze, UXtweak.

## 3. Site shape: depth vs. breadth

Research conclusion: **moderate breadth + transparent labels** beats both flat and deep hierarchies in task success and perceived usability (Zaphiris & Lee 2002, "Morphological Analysis of the Navigational Structure of Corporate Web Sites"; replicated in multiple NNGroup IA studies).

```
FLAT (bad)            DEEP (bad)              CONCAVE (optimal)
[28 links]            Home                    Home
 all at once          └ A                      ├ broad top-level (5–9)
                        └ A1                    │   └ mid categories
 ▲ choice overload       └ A1a                 │       └ focused leaves
                           └ A1a-i (5 clicks)   ▲ 2–3 clicks, scannable
```

- **Concave shape:** broad initial selection → narrowing category decisions → focused terminal options.
- **Hick's Law:** decision time grows ~logarithmically with the number of choices. Grouping collapses *perceived* options. Four labeled groups of five > twenty ungrouped links.
- **Depth ceiling:** static hierarchies should not exceed **3 levels**. Beyond that, task completion time climbs.
- **Click target:** any content reachable in **2–3 clicks** from the homepage.
- **Caution on the click metric itself:** raw "number of clicks" is a weak optimization target. A 4-click path with confident, well-labeled steps beats a 2-click path through ambiguous labels. Measure **task success rate, first-click accuracy, and directness** instead.

## 4. Labeling & information scent

Navigation labels are promises. A label carries **information scent** — a predictable signal of what lies behind it. Weak scent forces guessing and erodes trust.

- **Use old, familiar words.** "Pricing", "Support", "Blog" beat invented terms. Made-up labels ("Our Journey", "The Hub", "Explore") hurt findability *and* SEO — users can't find what they can't name; Google can't rank terms nobody searches.
- **Task-based > audience-based.** NNGroup testing: audience segments ("For Individuals", "For Enterprise") tested worse than task labels. Use audience splits only when offerings genuinely diverge by persona.
- **Verb-noun for action sites.** "Shop Cameras", "Find a Doctor", "Book a Demo" outperform noun-only "Products" / "Services" when the goal is action.
- **Avoid vague umbrella terms.** "Resources" / "Solutions" are scent-free. Name the actual content: "Guides", "API Docs", "Case Studies", "Templates".
- **One concept, one label, everywhere.** The same destination must use the same word in nav, breadcrumb, page title, and link text.

## 5. Navigation patterns — when to use each

### Top nav (horizontal global)
Default for marketing/content sites with ≤ 7 top-level sections. Persistent, scannable, familiar. Pair with a mega menu or dropdowns when sections have sub-pages. **Sticky/fixed top nav** preserves access on long pages but costs ~50px of vertical viewport; use it on content-heavy sites, omit it on focused landing pages and checkout flows.

### Sidebar (vertical)
Best for **docs, dashboards, SaaS apps** — many peer sections, deep hierarchies, frequent switching. Supports 2-level expand/collapse and a persistent active state. Combine with breadcrumbs for 3-level structures.

### Mega menu
A large, multi-column panel revealing many subcategories at once.
- **Use when** one top-level section has many sub-sections users compare (e.g., e-commerce catalogs). Widely adopted on large e-commerce and enterprise sites; Baymard research confirms majority adoption on top US retail sites, predominantly hover-based.
- Users scan all options simultaneously — this is the primary advantage over hierarchical dropdowns on content-dense sites.
- **Rules:** clear column headings; **no scrollbar**; include top-level categories + most popular subcategories only — **do not recreate the sitemap**.
- **Hover intent:** add a **300–500ms** delay before opening to prevent accidental triggers while mousing across the bar. Handle the **diagonal cursor problem** (see §10).

### Tabs
For **2–7 peer-level sections with roughly equal traffic**. Mutually exclusive views of one context (e.g., "Overview / Reviews / Specs"). Always mark the active tab; never use tabs for sequential steps (use a stepper) or for nav with one dominant section (use a mega menu).

### Breadcrumbs
Show the user's position in the hierarchy and provide one-click ancestor jumps. Essential on sites ≥ 3 levels and on every page reachable by search. See §7.

### Hamburger / drawer
Mobile-only. Hides nav behind a ☰ icon. Acceptable when screen width forces it; **never on desktop** — it wastes 1440px+ real estate and adds click depth (NNGroup hamburger menu research).

## 6. The hamburger debate (resolved)

- **Desktop:** never. There is no space constraint. Hidden nav "might as well not exist" — invisible navigation is one of NNGroup's top IA mistakes; banner blindness makes users screen out anything that looks like decoration.
- **Mobile:** acceptable but not ideal alone. **Hybrid pattern wins:** keep **3–5 critical items persistently visible**, hide secondary links behind the hamburger (NNGroup). Never hide *all* primary navigation.
- **Don't copy mobile patterns to desktop.** Hamburger + drawer on a wide screen is a regression.

## 7. Breadcrumbs — the 5 hard rules

Per NNGroup (11 guidelines), a breadcrumb must:

1. **Start at the Homepage.**
2. **Show hierarchy, not history.** It reflects site taxonomy (`Home › Cameras › DSLR › X100`), not the user's click path.
3. **Make every node clickable except the current page.**
4. **Not link the current page** (it's where you are).
5. **Only include nodes that have real, dedicated pages.** No fake intermediate steps.

```html
<nav aria-label="Breadcrumb">
  <ol>
    <li><a href="/">Home</a></li>
    <li><a href="/cameras">Cameras</a></li>
    <li><a href="/cameras/dslr">DSLR</a></li>
    <li aria-current="page">Fujifilm X100</li>
  </ol>
</nav>
```

- **Two types, both needed for e-commerce:** (a) **hierarchy-based** (site taxonomy) and (b) **history-based** ("Back to results" preserving filters/sort). Baymard benchmarks consistently find a majority of top e-commerce sites implement breadcrumbs sub-optimally: the most common failure is offering only hierarchy-based breadcrumbs and omitting "back to results" after a search; a significant minority have no breadcrumbs at all; very few correctly implement both types for both category browsing and search result navigation.
- **Polyhierarchy trap:** if a page lives in multiple category trees and a user arrives from category A, a breadcrumb showing category B feels like a bug. Pick a canonical parent for the breadcrumb, or derive it from the entry path consistently.
- **Mobile:** breadcrumb touch targets ≥ **10mm × 10mm** (NNGroup guideline 10); truncate middle nodes (`Home › … › DSLR › X100`) rather than wrapping.
- **Don't** show only the current page title — that breaks the hierarchy signal entirely.

## 8. On-site search

- **Always visible** on content-heavy sites — a full input field, not an icon. Exception: app-like utilities where search *is* the primary interaction.
- **Bar position matters:** centered or top-of-page placement consistently outperforms bottom-of-header or top-left placement in usage studies. Prefer centered or top placement; top-left is the weakest position.
- **Searchers convert 2–3× more** than browse-only users. Treat search as a primary path, not a fallback.
- **Autocomplete:** **5–8** suggestions on desktop (max **10** before choice paralysis), **4–6** on mobile (fit the viewport without scrolling). **No scrollbar** in the dropdown — inline scroll causes mis-clicks and disorientation (IKEA, Amazon, Etsy all avoid it).
- **Zero state:** before any typing, show recent searches or popular queries — **never a blank dropdown**.
- **Fuzzy/typo tolerance** is mandatory. "camra" must find "camera".
- **Search ⇄ navigation must integrate.** After clicking a result, the user must see where they landed (breadcrumb + active nav). Failing this forces **pogo-sticking** (result → back → result).
- Most e-commerce sites offer autocomplete, but Baymard research consistently finds that very few implement all critical UX details correctly (zero-state population, typo tolerance, no scrollbar, scoped results) — the bar to differentiate is low.

## 9. Footer as navigation

Users **intentionally scroll to footers** for contact info, policies, and site structure. Missing footer content causes measurable purchase abandonment (Baymard).

Patterns:
- **Doormat** — mirrors global nav; good for long pages where the top nav has scrolled away.
- **Fat footer / sitemap-lite** — structured link columns grouped by category; the workhorse for content and marketing sites.
- **Utility-only** — minimal; for checkout/landing pages.

Rules:
- **Minimum content:** copyright, privacy policy, terms of use, contact info.
- **Never strip the footer from checkout without replacing the contact affordance** — users hunt there for help and return policies.
- Footer link touch targets **44×44px**.

## 10. Mega-menu engineering: the diagonal cursor problem

When the cursor travels **diagonally** from a top-level item toward a submenu column, it may pass over *another* top-level item — naïve `:hover` logic then closes the open panel. This is a well-known bug requiring intentional hit-area design.

Fixes (pick one):
- **Hover-intent delay** of 300–500ms before switching the active top-level item.
- **CSS triangle/clip-path hitbox** covering the diagonal travel zone (Amazon's classic "menu-aim" technique).
- **JS pointer tracking:** if the cursor is moving toward the open panel (velocity vector points into it), suppress the menu switch.

```css
/* Keep panel open across the gap between trigger and panel */
.nav-item { position: relative; }
.nav-item:hover > .mega-panel,
.nav-item:focus-within > .mega-panel { display: block; }
/* Invisible bridge so the cursor never exits the hover region */
.nav-item:hover::after {
  content: ""; position: absolute; inset: 100% 0 auto 0; height: 12px;
}
```

- **Keyboard + screen reader:** mega menus need full keyboard nav (Tab/Arrow), `aria-expanded`, focus trapping inside the open panel, and Escape to close. `:focus-within` keeps it open for keyboard users.

## 11. Mobile navigation

- **~60%** of global web traffic is mobile (StatCounter, consistently above 55% since 2020). Reachable zones on large phones (≥ 375pt wide) are at the **bottom** — confirmed by Hoober's "How Do Users Really Hold Mobile Devices?" research (2013, still the primary dataset) and subsequent eye-tracking follow-ups. Assume one-handed use is common, especially on phones ≥ 6 inches.
- **Bottom navigation bar** is optimal for **3–5** primary sections. **Icons require labels** — icon-only bars score lower in recognition tests.
- **Don't remove the bottom tab bar without a replacement.** iOS 18 Photos removed bottom tabs → real user disorientation ("I dread trying to navigate the Photos app now"). Apple Fitness+ has no bottom bar → users can't find My Library.
- **Don't bury core functions behind gestures.** "Expecting my 73-year-old mother to remember to swipe down to find a search bar is a ridiculous assumption" — hidden gesture nav fails non-expert users.
- **Hybrid:** persistent bottom bar for top 3–5 sections; hamburger for the long tail.
- The **NY Times** moved the article back button to the *bottom* of pages for thumb ergonomics — follow physical reach, not desktop convention.

## 12. Wayfinding: "Where am I?" on every page

Because search deep-links users anywhere, **every page is a potential entry point** and must self-orient:

- **Active state** in nav/sidebar/tabs — use **color AND weight/position offset**, never color alone (WCAG: don't rely on color exclusively).
- **Breadcrumb** showing the ancestor path.
- **Page `<title>` and `<h1>`** matching the nav label that leads there.
- **`aria-current="page"`** on the active link for assistive tech.

```css
/* Active link: dual signal — color + weight + position offset */
.nav a[aria-current="page"] {
  color: var(--accent);
  font-weight: 600;
  box-shadow: inset 0 -3px 0 currentColor; /* positional cue, not color-only */
}
```

## 13. Progressive disclosure in navigation

Show only what's needed at each stage; reveal deeper options as users commit to a path. Applies to nav menus (top-level → submenu), search autocomplete (query → suggestions), and mobile drawers (sections → sub-sections).

- **Top nav → mega menu:** expose subcategories only when the user engages the top-level trigger. Don't pre-expand all panels.
- **Sidebar expand/collapse:** show parent + one level; reveal children on parent click. Don't open all branches simultaneously.
- **Search:** show zero-state suggestions before typing; switch to filtered results after ≥ 2 characters; never show a loading spinner for cached/recent queries.
- **Mobile drawer:** first tier is persistent bottom bar; second tier is the drawer opened from the ☰ icon; third tier (sub-sections) reveals inside the drawer — never a third-level push panel.

See [Cognitive Load & Progressive Disclosure](cognitive-load-progressive-disclosure.md).

## 14. URL structure as IA signal

URLs are visible IA. A user reading the address bar must be able to infer their location and navigate upward by editing the URL. This is not theoretical — power users do it constantly; SEO and sharability depend on it.

- **URL mirrors the breadcrumb:** `/cameras/dslr/fujifilm-x100` matches `Home › Cameras › DSLR › Fujifilm X100`. Mismatch between URL path and breadcrumb is an IA contradiction.
- **Human-readable slugs, not IDs.** `/cameras/dslr` not `/cat?id=4712`. IDs are opaque; slugs carry information scent.
- **Stable URLs.** Restructuring the IA means redirecting old URLs (301). Broken links after IA changes destroy SEO equity and confuse returning users. Plan migration before restructuring.
- **URL depth ≈ hierarchy depth.** A URL with 5+ path segments signals a hierarchy that is probably too deep. If your URLs are deep, your IA is deep — fix the IA.
- **Avoid query-string-only navigation for primary structure.** `/blog?category=design` is weaker than `/blog/design/` for both IA clarity and SEO.

## 15. Faceted navigation (filters)

Faceted navigation lets users narrow a large set by multiple independent dimensions simultaneously (e.g., size + color + brand + price range). It is the correct solution when the content set is large and multi-dimensional — not a tree hierarchy.

- **Use facets when:** product/content set has ≥ 3 orthogonal dimensions; users want to compare options rather than follow a single path; polyhierarchy would otherwise create hundreds of leaf categories.
- **Use hierarchical nav when:** content has a single primary organizing dimension; one category genuinely contains another (DSLRs are a type of Camera).
- **Hybrid is common:** top-level tree nav to reach a category (`/cameras/dslr`), then facets within the category (brand, price, resolution).
- **Filter persistence:** selected facets must survive pagination, back-navigation, and sharing. Store filters in the URL query string (`?brand=fujifilm&price=500-1000`); this makes the filtered view bookmarkable and shareable, and preserves state on browser back.
- **Show result counts** next to each facet value. Counts below 1 should be grayed out (not removed — removal disorients users who had that option a moment ago).
- **Clear individual filters and "clear all":** both controls must be present, positioned above the results, not buried in a sidebar.
- **Mobile facet pattern:** facets in a drawer/modal triggered by a "Filter" button; show active filter count on the button ("Filter (3)").

## 16. Accessibility: skip navigation and ARIA landmarks

Search-engine deep links and screen reader users both arrive at arbitrary pages without nav context. Navigation must be accessible by keyboard and assistive tech.

- **Skip navigation link:** a `<a href="#main-content">Skip to main content</a>` link as the **first focusable element** in the DOM. Visually hidden by default; visible on `:focus`. Required for WCAG 2.4.1 (Level A). Without it, keyboard users Tab through the entire nav on every page.

```html
<a class="skip-link" href="#main-content">Skip to main content</a>
<!-- in CSS: .skip-link { position: absolute; transform: translateY(-100%); }
            .skip-link:focus { transform: translateY(0); } -->
<nav aria-label="Primary navigation">...</nav>
<main id="main-content">...</main>
```

- **ARIA landmarks:** `<nav aria-label="Primary navigation">`, `<nav aria-label="Breadcrumb">`, `<main>`, `<footer>`. Screen reader users navigate by landmark. Multiple `<nav>` elements require unique `aria-label` to distinguish them.
- **`aria-current="page"`** on the active nav link — the only correct signal for "you are here" to assistive tech. Do not use `aria-selected` (that is for tabs/lists).
- **Keyboard nav in mega menus:** Tab moves into the panel; arrow keys navigate within it; Escape closes the panel and returns focus to the trigger.
- **Focus management after navigation:** on SPA route changes, move focus to the `<h1>` of the new page. Without this, screen reader users hear nothing indicating the page changed.

## Concrete specs & numbers

- **Nav depth:** ≤ 3 levels (static). **≤ 3 clicks** home → content.
- **Top-level items:** cap at **5–7** without grouping (Hick's Law; not Miller's 7±2).
- **Touch targets:** **44×44px** (Apple HIG), **48×48dp** (Material), **24×24 CSS px** min + spacing (WCAG 2.5.8); recommended 32px spacing between targets. Footer links **44×44px**. Breadcrumb nodes ≥ **10mm × 10mm** (mobile).
- **Bottom nav:** **3–5** sections; labeled icons.
- **Autocomplete:** **5–8** desktop (max 10), **4–6** mobile; no scrollbar.
- **Mega-menu hover-intent delay:** **300–500ms**.
- **Breadcrumbs:** Baymard research finds majority of e-commerce sites implement breadcrumbs sub-optimally; very few correctly implement both hierarchy-type and history/filter-type.
- **Search:** centered or top placement outperforms top-left; searchers convert **2–3×** more than browse-only users.
- **Contrast:** WCAG AA **4.5:1** normal / **3:1** large text; AAA **7:1**. Low-contrast text on **81%** of homepages (WebAIM Million 2024) — verify nav link contrast in all states.
- **Traffic:** **~60%** mobile globally (StatCounter 2024–2025); one-handed use is common on large phones.

## Tools & libraries

- **IA research:** Optimal Workshop (card sort + tree test), Maze, UXtweak. Run **open card sort** to discover groupings, **tree test** to validate the structure *before* visual design.
- **Headless menu/nav primitives** (accessible by default — keyboard, ARIA, focus): Radix UI `NavigationMenu`, React Aria, Headless UI `Menu`/`Disclosure`. See [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md).
- **Site search:** Algolia / Typesense / Meilisearch for fuzzy matching, instant autocomplete, and typo tolerance out of the box. Avoid hand-rolling fuzzy search if quality matters.
- **When NOT to use a library:** a 4-link top nav doesn't need a mega-menu framework. Match tooling to structure complexity, not to fashion.

## Do / Don't

**Do**
- Design IA first; validate with tree testing before building nav.
- Group items under clear headings (Hick's Law); cap top-level at 5–7 ungrouped items.
- Use familiar, task-based, verb-noun labels with strong scent.
- Show active state (color + weight/position) and breadcrumb on every page.
- Keep nav identical in position/label/behavior across all pages.
- Make search visible, with zero-state, fuzzy matching, and no dropdown scrollbar.
- Use hybrid mobile nav: 3–5 persistent items + hamburger for the tail.
- Handle mega-menu diagonal hover with intent delay or hitbox.
- Keep URL path mirroring breadcrumb; use human-readable slugs; redirect on IA restructures.
- Add skip-nav link as first focusable element; use `aria-label` on every `<nav>`.

**Don't**
- Use a hamburger on desktop.
- Invent menu terminology ("The Hub", "Explore", "Our Journey").
- Exceed 3 static levels or force > 3 clicks to content.
- Recreate the sitemap in a mega menu or add a scrollbar to it.
- Classify one page across many category trees (polyhierarchy) and let breadcrumbs contradict the entry path.
- Hide all primary nav behind a gesture or icon on mobile.
- Optimize purely for click count.
- Rely on color alone for active state.
- Break URL structure on IA restructures without 301 redirects.
- Store facet/filter state only in JS — URL must be bookmarkable.

## Common pitfalls & how to avoid them

- **No organizing structure:** content as "one big swamp" — no taxonomy. → Define structure via card sort; don't launch without a tested hierarchy.
- **Extreme polyhierarchy:** items under many dimensions at once → confusion, duplicate entries. → Pick primary categories; expose alternates via filters/tags, not the main tree.
- **Invisible navigation:** hidden or ad-like nav suffers banner blindness. → Keep nav visible and visually distinct from promos.
- **Animated/uncontrollable nav:** auto-expanding, bouncing menus. → Use stable, predictable, user-triggered reveals.
- **Inconsistent navigation:** label/position/behavior changes across pages → breaks the mental model. → One global nav component, rendered identically everywhere.
- **Made-up terminology:** hurts findability + SEO. → Use words users and search engines actually use.
- **Thin/empty destinations:** nav leading to dead-end pages → bounce. → Don't expose a section until it has real content.
- **Footer as afterthought:** missing contact/policies/structure → abandonment. → Treat the footer as deliberate navigation.
- **Autocomplete scrollbar:** mis-clicks + disorientation. → Cap suggestions to fit viewport; no inline scroll.
- **Breadcrumb shows title only:** breaks hierarchy signal. → Always render the full ancestor path.
- **Search ↔ nav disintegration:** result lands with no orientation → pogo-sticking. → Show breadcrumb + active nav on every landing page.
- **URL contradicts IA:** `/cameras/dslr` in the URL but breadcrumb shows `Photography > Film`. → URL must mirror the sitemap structure.
- **IA restructure with no redirects:** old bookmarks and inbound links 404 after a restructure → SEO damage + broken trust. → Plan 301 redirects before any structural change.
- **Facet state not in URL:** user filters by size + color, shares the link, recipient sees unfiltered results. → Filter state must serialize to the query string.
- **No skip link:** keyboard users Tab through 12 nav items on every page. → First focusable DOM element must be `<a href="#main-content">Skip to main content</a>`.

## What real users complain about

- **iOS 18 Photos** removed bottom tabs: "I dread trying to navigate the Photos app now." Multiple users couldn't find their library at all. (r/UXDesign, Aug 2024)
- **Apple Fitness+:** "there is no bottom tab bar and I hate it, it makes me go through a lot of content until I find My Library, so I never use it." (r/UXDesign)
- **Hidden gesture nav:** "Expecting my 73-year-old mother to remember to swipe down to find a search bar is a ridiculous assumption." (r/UXDesign)
- **Bandcamp search:** "Artists with common names are impossible to find. There should be advanced search with filtering." Plus deep-link failure: clicking a Bandcamp link won't open the app — "I have to go open the app, manually search for it, and hope it comes up."
- **Discord complexity:** "How do people learn to use this? I'm getting anxiety just looking at it, it is such a mess." (r/userexperience, 222 upvotes)
- **Tom's Guide redesign:** "You literally can't find the text or information you're coming to read. It's a puzzle of ads, promotions, and popups from the first second." — IA destroyed by monetization overlays. (r/web_design)
- **On metrics:** "Number of clicks is the shittiest metric there is and you will hear barely anything else from most people who should know better." (r/userexperience) — optimize task success, not clicks.

## Senior-level checklist

- [ ] IA validated by **tree test**; nav direction validated by **first-click test** (not by opinion).
- [ ] Static depth ≤ 3 levels; every key destination reachable in ≤ 3 clicks.
- [ ] Top-level items ≤ 7 (target 5–7), or grouped under clear headings / mega menu.
- [ ] Every label has strong scent; zero invented terms; task-based, verb-noun where action-oriented.
- [ ] Nav identical in position, label, and behavior on every page.
- [ ] Active state uses color **and** weight/position; `aria-current="page"` present.
- [ ] Breadcrumbs: start at Home, hierarchy not history, current page unlinked, only real pages; e-commerce shows both hierarchy + "back to results."
- [ ] Search visible (centered/top), zero-state populated, fuzzy/typo tolerant, no dropdown scrollbar, 5–8 desktop / 4–6 mobile suggestions.
- [ ] Every page (incl. search landings) shows breadcrumb + active nav (no pogo-sticking).
- [ ] Mobile: 3–5 core sections persistently visible; hamburger only for the tail; bottom-bar icons labeled.
- [ ] No hamburger on desktop.
- [ ] Mega menu: column headings, no scrollbar, not a full sitemap, diagonal-hover handled, keyboard + ARIA complete.
- [ ] Footer carries copyright, privacy, terms, contact; not stripped from checkout without a contact affordance.
- [ ] Touch targets meet 44×44 / 48×48 / 24×24 minimums; breadcrumb nodes ≥ 10mm on mobile.
- [ ] Nav link contrast meets WCAG AA (4.5:1 / 3:1) in default, hover, and active states.
- [ ] No polyhierarchy contradictions; no thin/empty destination pages.
- [ ] URL slugs are human-readable; URL path mirrors breadcrumb hierarchy; old URLs redirect (301) after any IA restructure.
- [ ] Faceted nav (if applicable): filter state in URL, result counts per facet, individual + clear-all controls, mobile filter drawer shows active count.
- [ ] Skip navigation link present as first focusable element; `<nav aria-label>` landmarks on every nav region; focus managed on SPA route changes.
- [ ] IA validated by open card sort before tree test; tree test shows ≥ 70% first-click accuracy on primary tasks.

## Sources

- Nielsen Norman Group — IA & navigation articles, "Top 10 IA Mistakes," breadcrumb & menu-design guidelines, mega-menu research, hamburger/mobile guidance: https://www.nngroup.com/
- Baymard Institute — e-commerce UX research (search/autocomplete, mega menus, breadcrumbs): https://baymard.com/
- WebAIM Million 2024 (low-contrast prevalence): https://webaim.org/projects/million/
- WCAG 2.2 (contrast 1.4.3/1.4.6, target size 2.5.8): https://www.w3.org/TR/WCAG22/
- Apple Human Interface Guidelines (touch targets, navigation): https://developer.apple.com/design/human-interface-guidelines/
- Material Design (navigation, touch targets): https://m3.material.io/
- StatCounter Global Stats (mobile traffic share): https://gs.statcounter.com/
- Zaphiris P. & Lee S. (2002) — "Morphological Analysis of the Navigational Structure of Corporate Web Sites" (breadth vs. depth hierarchy research)
- Hoober S. (2013) — "How Do Users Really Hold Mobile Devices?" (one-handed use, thumb zones): https://www.uxmatters.com/mt/archives/2013/02/how-do-users-really-hold-mobile-devices.php
- Hick W. E. (1952) / Hyman R. (1953) — Hick-Hyman Law (choice reaction time grows logarithmically with number of alternatives)
- Miller G. A. (1956) — "The Magical Number Seven, Plus or Minus Two" (working memory capacity; note: does not govern nav breadth)

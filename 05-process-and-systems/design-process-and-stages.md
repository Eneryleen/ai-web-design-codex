# The Website Design Process & Stages
> The end-to-end pipeline for taking a website from a vague request to a launched, measured product: discovery → research → content → IA → wireframes → visual design → design system → prototype → usability test → build/handoff → QA → launch → iterate. Read this before designing anything so you solve the right problem in the right order and never skip content or IA — the two stages an AI is most tempted to leap over.

**Apply when:** starting any non-trivial website or redesign and deciding what to produce, in what order, with what verification. · **Related:** [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md) · [Wireframing, Prototyping, Handoff & QA](./wireframing-prototyping-handoff-qa.md) · [Design Systems & Design Tokens](./design-systems-and-tokens.md) · [User Research & Usability Testing](../01-ux-fundamentals/user-research-and-testing.md)

## TL;DR — rules at a glance
- **Two diamonds, not one.** Discover + Define (problem) come before Develop + Deliver (solution). Skipping the first diamond builds the wrong thing well — the most expensive failure mode.
- **Content and IA precede pixels.** Decide structure (sitemap → priority guide → wireframe) before color and type. A structural change costs minutes in a wireframe, hours in hi-fi, days in code.
- **Write the brief before opening a tool.** Goals, audience, must-have vs nice-to-have, tone, constraints — in Notion/Miro, in words. AI "component-shaped guessing" happens when none of this is written down.
- **SEO keywords before sitemapping.** Shape page structure around target keywords; retrofitting SEO onto a finished design is painful.
- **Mobile-first wireframes.** Set content hierarchy on the smallest viewport; it scales up by definition. Use priority guides (no lorem, no nav, no boxes) to escape the "illusion of finality."
- **Lo-fi = decisions, hi-fi = decoration.** Grayscale wireframes resolve hierarchy and flow; visual design applies brand once structure is locked.
- **Involve developers at wireframing, not handoff.** Most designer-dev conflict is a process gap. Feasibility discussion at hi-fi handoff is already too late.
- **Agree token names in a cross-discipline workshop.** Semantic names (`button/primary/bg`, `text/heading/default`) must match codebase conventions or every future update breaks the Figma↔code link.
- **Test with 5 users, iteratively.** ~85% of qualitative issues surface with 5 participants (Nielsen/Landauer, 31% per-issue probability). Run several small rounds, not one big study.
- **WCAG 2.2 AA is the legal floor**, not polish (US DOJ Title II 2024, EU EAA June 2025). Zero critical/serious axe violations before cutover.
- **Set a performance budget before development** and enforce in CI: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1 (75th-percentile field data).
- **Stakeholders ≠ users.** Designing only for stakeholder approval is a well-documented path to wasted spend. Validate with real users at every research and test stage.
- **Set feedback SLAs up front.** 3–5 business days per round; slow client review is the #1 cause of schedule slip on fixed-scope projects.

## The Double Diamond (the spine of the process)
British Design Council, 2005 (study of 11 global companies incl. Alessi, BBC, LEGO, and others). Two diamonds, each a diverge-then-converge cycle.

```
   PROBLEM SPACE                     SOLUTION SPACE
 ◇ Discover  ▷ Define          ◇ Develop   ▷ Deliver
 (diverge)   (converge)        (diverge)   (converge)
 research,   synthesize        ideate,     prototype,
 interviews  → problem         wireframe,  test, build,
 explore     statement         explore     ship, refine
```

- **Discover (diverge):** widen scope. Stakeholder interviews, user research, competitive teardown, analytics review. Goal: understand, don't solve yet.
- **Define (converge):** narrow to a sharp, written problem statement + success metrics. Format: **"How Might We [action] for [user] so that [outcome]?"** paired with a numeric target. E.g. "HMW reduce friction for returning shoppers so checkout abandonment drops from 72% to <60%?" — not "make checkout nicer."
- **Develop (diverge):** generate multiple solution directions — IA options, wireframe variants, visual directions. Diverge before committing.
- **Deliver (converge):** prototype → test → build → QA → launch the chosen direction.

**Rule:** the problem space is as important as the solution space. An AI agent's default failure is to start at Develop. Force yourself through Discover/Define first, even if compressed to an hour of written analysis.

**Stage gates (go/no-go before advancing):**
- **Brief → Research:** problem statement is one sentence with a numeric metric. No metric ⇒ not a gate pass.
- **Research → Content/IA:** every persona/JTBD and keyword traces to evidence (interview, analytics, competitor), not assumption.
- **Content/IA → Wireframe:** sitemap approved; real copy exists for hero/value-props/CTAs.
- **Wireframe → Hi-fi:** hierarchy reads in grayscale; a developer has signed off on feasibility.
- **Hi-fi → Build:** all states designed; contrast + touch targets pass; tokens named to codebase.
- **Build → Launch:** zero critical/serious axe violations; lab CWV within budget; analytics verified.

**Which UX laws bite at each stage:** Jakob's (IA/nav — match conventions), Hick's (IA + forms + CTA count — fewer choices), Von Restorff (visual — distinct primary CTA), Peak-End (flow design — optimize the climax + final screen), Tesler's (architecture — push complexity into the system, not the user). See [The Laws of UX & Design Psychology](../01-ux-fundamentals/laws-of-ux.md).

## Stage-by-stage: deliverables, verification, where AI fits

### 1. Discovery / Brief — *2–3 weeks (or 1 written page minimum)*
**Do:** write goals, target audience(s), essential vs nice-to-have features, tone/brand alignment, constraints (tech stack, budget, deadline, compliance) **before touching any design tool.**
- **Deliverable:** written brief (Notion/Miro). Include 1 measurable primary goal + 2–3 secondary. Attach a signed-off stakeholder approval before Research begins.
- **Verify:** can you state the problem in one sentence and the success metric in numbers? If not, you're not done.
- **AI fits:** drafting the brief from a transcript, summarizing stakeholder notes, generating clarifying questions. **AI does NOT fit:** deciding the goal — that's the client's.
- **Distinguish stakeholders from users.** Stakeholder approval and user validation are separate checkpoints. Design history is littered with polished products that failed because only stakeholders were consulted — real users were never in the room. Plan user-validation touchpoints from day one.

### 2. Research — *part of Discover (2–3 weeks total)*
**Do:** user interviews, competitor/heuristic analysis, analytics review (for redesigns), SEO keyword research.
- **Deliverables:** personas or jobs-to-be-done, competitive matrix, keyword list, analytics baseline (current CWV, conversion, top exit pages).
- **Verify:** every later design decision can cite a research input.
- **SEO rule:** gather target keywords **before** sitemapping; structure pages around them. Easier to bake SEO into a sitemap than retrofit onto a finished design.
- See [User Research & Usability Testing](../01-ux-fundamentals/user-research-and-testing.md).

### 3. Content strategy — *DO NOT SKIP*
**Do:** inventory and define real content before structure hardens. Content-first design: structure follows content, not the reverse.
- **Redesign vs new site:** for a redesign, do a **content audit** first (what exists → keep / cut / rewrite / migrate). For a new site, do a **content inventory** (what must be created, by whom, by when). These are different artifacts; don't skip either.
- **Deliverable:** content inventory/audit matrix (page → purpose → primary message → CTA → assets needed → owner → status). Every row must have an owner and a due date or it will not exist at handoff.
- **Verify:** no page is a "we'll fill it later" box. Real copy (even rough) exists for hero, value props, and primary CTAs before wireframing begins.
- **Why it matters:** AI-generated designs without real content produce pretty pages with no information hierarchy. Lorem ipsum hides whether the layout actually works — copy length, headline weight, and CTA phrasing all drive spacing and visual hierarchy decisions.
- See [Copywriting & UX Writing](../06-marketing-and-conversion/copywriting-and-ux-writing.md).

### 4. Information Architecture / Sitemap — *DO NOT SKIP*
**Do:** build the full sitemap (all pages + hierarchy flowchart) **before any page-level wireframe.** The sitemap is the foundation; wireframes are zoom-ins on individual pages.
- **Deliverable:** sitemap flowchart + navigation model (global, local, utility, footer). Optionally a card-sort result for labels.
- **Verify:** every page has a parent and a reason to exist; nav reflects user mental models (Jakob's Law — match conventions of sites users already use).
- **AI fits:** FlowMapp / Relume can draft sitemaps fast, but you must validate labels against research, not vibes.
- See [Information Architecture & Navigation](../01-ux-fundamentals/information-architecture-navigation.md).

### 5. Lo-fi wireframes + priority guides — *Develop (diverge)*
**Do:** grayscale, generic typeface, **real placeholder content** (not lorem where avoidable), mobile viewport first. Purpose = structure, hierarchy, user flow — **not aesthetics.**
- **Priority Guide rule (Stephen Hay, 2013 / A List Apart):** no lorem ipsum, no layout boxes, no navigation chrome, mobile format — list content blocks in relevance order. Priority guides *precede* wireframes; they force content prioritization before any layout decision is made. This removes the "illusion of finality" that even rough wireframes create and frees visual designers from inherited layout assumptions.
- **Deliverables:** priority guides per page template, lo-fi wireframes for all key user flows (annotated with intent, not decoration), mobile + desktop critical viewports.
- **Verify:** hierarchy reads correctly in grayscale; primary action is unambiguous without color. Estimate: a structural change here takes minutes; in hi-fi, hours; in code, days.
- **Involve developers now.** Ask "is this feasible / what's the approximate cost?" at lo-fi — not at handoff. Feasibility review at hi-fi is already too late.
- See [Wireframing, Prototyping, Handoff & QA](./wireframing-prototyping-handoff-qa.md).

### 6. Hi-fi visual design — *Develop → Deliver*
**Do:** apply brand — real content, specific UI, color/type/spacing, imagery. Hi-fi screens serve as dev reference (not the same as an interactive prototype).
- **Deliverables:** key screens in all critical states (default, loading, empty, error, success), responsive at primary breakpoints. Minimum breakpoints: **375px** (mobile), **768px** (tablet/landscape), **1280px** (desktop), **1440px** (wide). Cover additional breakpoints if design breaks between these.
- **States rule:** if a state is not designed, it will be invented by a developer under pressure. Empty states and error states are the most commonly skipped and the most visible to users.
- **Verify:** contrast passes (body 4.5:1 minimum, large text ≥ 18pt or ≥ 14pt bold at 3:1 minimum), Von Restorff effect applied (primary CTA visually distinct from every secondary action on the page), Hick's Law applied (count nav items, CTAs, and form fields — cut any above what the user strictly needs).
- **Stakeholder sign-off here** before entering build. Document approved direction in writing.
- Foundations: [Visual Hierarchy](../00-foundations/visual-hierarchy.md), [Color Theory & Systems](../00-foundations/color-theory-and-systems.md), [Typography Systems](../00-foundations/typography-systems.md), [Layout, Grids & Spacing](../00-foundations/layout-grids-and-spacing.md).

### 7. Design system & tokens — *parallel with hi-fi*
**Do:** extract reusable tokens and components. Three-layer token architecture:
```
Primitives (raw, never used directly)   →  Semantic (contextual)          →  Component mapping
--pink-500: #E91E63;                        --color-bg-primary: var(--pink-500);   button.primary { background: var(--color-bg-primary); }
--space-4: 16px;                            --color-text-heading: var(--gray-900);
```
- **Modes** (light/dark/brand) swap *primitive* values beneath *stable semantic names*. Treat tokens like code: semantic versioning, npm package (`@company/design-tokens`).
- **Token naming workshop:** designers and devs agree semantic names that match codebase conventions in one session. Otherwise every update breaks the Figma↔code link.
- **A11y token rule:** focus/ring color token must be ≥ 3:1 against both light and dark backgrounds.
- See [Design Systems & Design Tokens](./design-systems-and-tokens.md), [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md).

### 8. Prototype — *Deliver (diverge→test)*
**Do:** make the chosen flow clickable for the tasks you'll test. Fidelity = whatever the test question requires; don't over-build.
- **Deliverable:** interactive prototype (Figma) covering the critical task path(s).
- **Verify:** a stranger can attempt the test tasks without you narrating.

### 9. Usability test — *Deliver (converge)*
**Do:** moderated, think-aloud, **5 participants per round**, multiple iterative rounds.
- **Math:** 5 users ≈ 85% of issues at 31% per-issue probability (Nielsen/Landauer formula). For harder-to-spot issues (L=20%) you need 9 users; (L=10%) need 18. But iterating multiple 5-user rounds finds more unique issues than one large study, while keeping cost low and feedback fast.
- **Practical schedule (NN/g discount usability):** Day 1 — write tasks and recruit; Day 2–3 — run 5 sessions; Day 4–5 — analyze, severity-rank, write recommendations. Experienced teams compress to 3 days; plan 5 for first-time execution.
- **Task writing rules:** frame as realistic activities ("find a jacket under $100 and add to cart"), never as instructions ("click the product grid"). Avoid words that appear on-screen — they prime participants. Deliver tasks written; ask participants to read them aloud before starting.
- **Verify:** top issues identified, severity-ranked (critical/serious/moderate/minor), fixes scoped to specific screens. Loop back to wireframe/visual as needed before next test round.
- See [User Research & Usability Testing](../01-ux-fundamentals/user-research-and-testing.md).

### 10. Build / Handoff — *Deliver*
**Do (Figma handoff):** single file link; emoji-status page names (🟡 in-progress / 🔵 feedback-ready / 🟢 ready-to-build); invite devs as **Viewers** (free seat); style/variable names = codebase token names; component doc frames (name, description, usage, variants, states, accessibility notes).
- **Deliverables:** spec'd screens (all states), token export (`@company/design-tokens` npm package or equivalent), component docs, redlines/auto-layout specs, content source of truth (copy doc with final strings).
- **Responsive breakpoints in handoff:** supply designs at each agreed breakpoint (minimum 375/768/1280px). Do not leave layout decisions between breakpoints to developer judgment.
- **Developer feedback loop:** when a dev flags an implementation question or feasibility issue during build, the answer goes into the Figma file as an annotation — not just a Slack reply. This keeps the file the source of truth.
- **Verify:** a developer can build any screen without messaging you. Run a "cold build" test: ask a dev who wasn't in the design process to attempt one screen from the handoff alone. If they get stuck, the handoff is incomplete.
- **Performance budget is part of handoff:** define LCP/INP/CLS targets and enforce in CI before code is written. Include image format requirements (AVIF/WebP, explicit width/height), font loading strategy, and third-party script policy.

### 11. QA — *1–2 weeks*
**Do:** functional QA, cross-browser/device, accessibility audit, performance audit, and design-fidelity review.
- **A11y:** automated (axe) catches ~57% of issues by volume (Deque, 13,000+ pages) — necessary but not sufficient. Add manual keyboard-only navigation, screen-reader (NVDA+Firefox, VoiceOver+Safari), 200% zoom, and `prefers-reduced-motion` checks. **Zero critical/serious axe violations before cutover.**
- **Perf:** measure LCP/INP/CLS in lab (PageSpeed Insights, DevTools Lighthouse) and in field (CrUX) before launch. Fix before shipping — not after.
- **Visual regression:** run Chromatic or Percy against the component library before and after each build sprint. Flag any components that deviated from design.
- **Design fidelity:** designer signs off that built screens match hi-fi at primary breakpoints. Check spacing, color tokens, font sizes, and interactive states — don't rubber-stamp.
- **Redirect QA (redesigns only):** verify every changed URL 301-redirects to its new target; no 404s from the old sitemap. Test with a crawl (Screaming Frog or similar) before cutover.
- See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md), [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md).

### 12. Launch — *then ongoing*
**Do:** SSL (Let's Encrypt / managed cert), redirects, analytics verified **before** go-live.
- **Redirect checklist (redesigns):** 301s for every changed URL, verified by crawl. Confirm no redirect chains longer than 2 hops. Canonical tags updated.
- **Analytics checklist:** in GA4 DebugView, complete the primary conversion path end-to-end and confirm all events fire within 2 minutes. Do not launch without this — broken analytics means flying blind.
- **RUM:** set up Vercel Analytics, Cloudflare Web Analytics, or equivalent for field CWV data. Lab scores (PageSpeed) and field scores (CrUX) diverge; field data is what Google ranks.
- **Verify:** conversion events fire; CWV monitored at 75th percentile in field; alerts armed at 80% of Good thresholds.

### 13. Iterate — *forever*
**Do:** monitor RUM + analytics, set alerts at **80% of CWV thresholds** (INP > 160ms, LCP > 2.0s, CLS > 0.08) to catch regressions before they hit rankings. Google CrUX uses a 28-day rolling average — fixes take ~28–30 days to fully appear in Search Console. Feed findings into the next small iteration (back into the diamonds).

## Roles & ownership per stage (RACI-lite)
Use this to know who must be in the room — an AI agent should request the missing approver rather than self-approve.
- **Brief/goals:** client/PM owns; design + dev consulted.
- **Research:** UX owns; PM informed.
- **Content:** content strategist/client owns; design consulted (real copy is a client dependency — flag it early).
- **IA/sitemap:** UX owns; SEO + dev consulted (feasibility of routes, redirects).
- **Wireframes:** design owns; **dev consulted (mandatory)**.
- **Visual/system/tokens:** design + dev co-own token names; brand owns aesthetics.
- **Prototype/testing:** UX owns.
- **Build/QA/launch:** dev owns; design verifies fidelity + a11y; PM owns go-live decision.

## Why content & IA must never be skipped
- **Cost curve:** a structural change is cheapest in IA/wireframe, ~10× in hi-fi, ~100× in code, worst post-launch. Lock structure before decoration.
- **AI's blind spot:** AI can generate component-shaped layouts but cannot decide hierarchy without knowing *your* users, content, and constraints. UX wireframing is about *decisions*, not decoration — and the decisions require inputs only the brief/research/content stages produce.
- **Lorem ipsum lies:** placeholder text makes any layout look balanced. Real content reveals overflow, weak hierarchy, missing states, and CTA ambiguity.
- **Sitemap-first SEO:** baking keyword-targeted structure into the sitemap is cheap; retrofitting onto a built site is not.

## Where AI fits across the process (and where it doesn't)
| Stage | AI helps | AI must NOT decide |
|---|---|---|
| Brief | draft from notes, generate clarifying Qs | the goal / priorities |
| Research | summarize interviews, cluster findings | who the user is (validate with real people) |
| Content | draft copy, suggest CTAs | brand voice sign-off, claims/legal |
| IA / Sitemap | draft sitemap (FlowMapp/Relume) | final labels (validate vs mental models) |
| Wireframe | generate unstyled components+copy (Relume/Uizard) | hierarchy without user context |
| Visual | propose directions, fill variants | brand judgment, taste calls |
| Tokens | generate token scaffolds | semantic names (cross-discipline workshop) |
| QA | run axe/Lighthouse, flag issues | "is it accessible?" without manual testing |

## Concrete specs & numbers
- **Total timeline:** 12–24 weeks typical. Discovery/research 2–3 wk · development 4–8 wk · testing/QA 1–2 wk · launch+post-launch ongoing.
- **Feedback rounds:** 3–5 business days/round standard. Slow client review is the single most common schedule-slip cause on fixed-scope web projects. Set the SLA in the contract, not in a Slack message.
- **Usability:** 5 users ≈ 85% of issues (Nielsen/Landauer, 31% encounter prob.); L=20%→9 users, L=10%→18 users for 85% coverage. Running multiple 5-user rounds (NN/g recommendation) finds more unique issues than one large study at lower total cost and faster feedback cadence. Discount study cost = a few hundred $ (incentives + recruiter); moderated remote studies scale from there.
- **Accessibility (WCAG 2.2 AA):** body text contrast **4.5:1**; large text (18pt+ or 14pt bold) **3:1**; focus indicator visible in high-contrast. Touch target minimum — **WCAG 2.2 SC 2.5.8 (AA):** 24×24 CSS px (with spacing exceptions); **WCAG 2.5.5 (AAA) / Apple HIG:** 44×44px. Use 44×44px as the design target — it exceeds the AA floor and eliminates the edge cases. Focus ring token ≥ **3:1** vs both modes. axe-core catches **~57%** of issues by volume (Deque, 13,000+ pages) — manual still required.
- **Core Web Vitals (75th-pct field):** LCP Good ≤ 2.5s / NI 2.5–4s / Poor >4s · INP Good ≤ 200ms / NI 200–500ms / Poor >500ms · CLS Good ≤ 0.1 / NI 0.1–0.25 / Poor >0.25. INP replaced FID (March 2024). Alert at 80% of threshold. CrUX 28-day rolling; fixes appear in ~28–30 days.
- **Checkout (Baymard):** avg cart abandonment **70.22%** (50 studies); extra costs/shipping cited by **48%** of abandoners (largest avoidable factor); avg large ecommerce site can gain **+35.26%** conversion via checkout alone (134 guidelines, 200k+ research hrs), ~32 improvements/site; **20–60%** form-field reduction achievable by default; 64% of leading US/EU sites rate "mediocre" or worse, only 2% "good", 0% "perfect"; **$260B** recoverable across $738B US+EU ecommerce.
- **LCP fixes:** `fetchpriority="high"` on hero, never lazy-load LCP element, AVIF/WebP, preload critical fonts + hero. **INP fixes:** main-thread JS is almost always the cause — audit third-party scripts (analytics/chat/ads) first, chunk long tasks. **CLS fixes:** explicit `width`/`height` on media, preload images, inline critical CSS.

```html
<!-- LCP hero: prioritized, never lazy -->
<img src="/hero.avif" width="1600" height="900" fetchpriority="high" alt="…">
<link rel="preload" as="font" href="/inter.woff2" type="font/woff2" crossorigin>
```

## Tools & libraries
- **Figma** — primary UI/UX, prototyping, design system, handoff (Code panel: CSS/Swift/XML; Variables for design logic). **Tokens Studio** plugin for cross-platform token export.
- **FlowMapp** — AI sitemap/IA (claims 2× faster planning). **Relume** — AI sitemap→wireframe, unstyled real components + copy. **Uizard / v0 (Vercel)** — AI wireframing; v0 useful for wireframes, not production UI.
- **Miro** — moodboards, affinity mapping, rapid sitemap sketching. **Notion** — brief, planning, decision log. **Asana / Monday / Google Sheets** — milestones.
- **UserTesting / Maze** — remote usability. **axe DevTools** — a11y audit (zero critical/serious = launch bar). **Stark / A11y Color Contrast Checker** (Figma) — in-design contrast.
- **PageSpeed Insights** (lab+field) · **Chrome DevTools Performance** (long-task/INP hunting) · **Vercel Analytics / Cloudflare Web Analytics** (RUM). **Chromatic + Storybook + axe-core** — component a11y regression in CI. **Autoflow / Framer wireframer / Supernova / Figr.design** — flow lines, docs.
- **When NOT:** don't let AI sitemap/wireframe tools make decisions — validate against research. Don't use v0 output as production UI. Don't rely on axe alone for a11y sign-off.

## Do / Don't
**Do**
- Write the brief and define a measurable problem before opening a design tool.
- Produce sitemap → content → priority guide → lo-fi → hi-fi in that order.
- Pull developers in at wireframing; agree token names in a joint workshop.
- Test 5 users per round, iterate; severity-rank issues.
- Set the performance budget and a11y bar before code, enforce in CI.
- Verify analytics in GA4 DebugView before launch.

**Don't**
- Don't start at "Develop" (jump to visuals/components) — that's solving an undefined problem.
- Don't use lorem ipsum to judge a layout's hierarchy.
- Don't design for stakeholder approval without user validation.
- Don't defer feasibility talk to hi-fi handoff.
- Don't name tokens by appearance (`blue-button`) — name semantically (`button/primary/bg`).
- Don't treat WCAG or CWV as post-launch nice-to-haves.

## Common pitfalls & how to avoid them
- **Skipping the first diamond.** *Symptom:* pretty pages, wrong product. *Fix:* force a written problem statement + metric before any layout.
- **IA after wireframes.** *Symptom:* orphan pages, nav that fights mental models. *Fix:* sitemap is a hard gate before page wireframes.
- **Lorem-ipsum hierarchy.** *Symptom:* design breaks when real copy lands. *Fix:* priority guides with real content, mobile-first.
- **Late developer involvement.** *Symptom:* hi-fi designs that can't be built on budget. *Fix:* feasibility review at lo-fi.
- **Token-name drift.** *Symptom:* every design update breaks code. *Fix:* semantic names matching codebase, agreed in workshop, versioned package.
- **One big usability study.** *Symptom:* slow, expensive, single shot. *Fix:* multiple 5-user rounds.
- **A11y/perf as final-step polish.** *Symptom:* failed audits at cutover, legal exposure. *Fix:* bars set pre-build, enforced in CI.
- **No analytics at launch.** *Symptom:* can't measure success. *Fix:* GA4 + RUM verified before go-live.
- **Skipping redirect audit on redesigns.** *Symptom:* 404s from inbound links, SEO rank collapse post-launch. *Fix:* crawl old sitemap, map every URL to its new target, verify with Screaming Frog before cutover.
- **No content audit on redesigns.** *Symptom:* orphaned pages, conflicting messaging, content migration crunch at launch. *Fix:* content audit (keep/cut/rewrite/migrate) is a hard gate before IA begins.
- **Undefined breakpoints in handoff.** *Symptom:* developers invent layout behavior between 768–1280px; design inconsistency in production. *Fix:* specify all breakpoints explicitly (375/768/1280/1440px minimum) with designs at each.

## What real users complain about
*(synthesized from Baymard checkout research and r/UXDesign discussions)*
- **Surprise costs at checkout** — 48% of abandoners cite extra/shipping costs revealed late; the single largest avoidable abandonment driver.
- **Bloated forms** — too many fields; most flows can drop 20–60% with no loss. Each extra field is friction (Hick's Law).
- **"Looks done but does nothing"** — AI/template sites that are visually polished but have no real hierarchy, dead CTAs, or content that doesn't answer the user's question.
- **Sites that don't behave like other sites** — non-standard nav/patterns force relearning (Jakob's Law violation).
- **Designed for the boss, not me** — flows optimized for stakeholder demos that fail real tasks. This pattern is consistently reported in practitioner post-mortems; the cost is scope rewrites or silent churn, not just a bad demo.
- **Janky loading** — layout shifts (poor CLS), slow hero (poor LCP), unresponsive taps after load (poor INP).
- See [What Real Users Complain About (Synthesis)](../08-pitfalls-and-antipatterns/real-user-complaints.md).

## Senior-level checklist
- [ ] One-sentence problem statement + numeric success metric written and agreed (HMW + target, signed off by client).
- [ ] Brief documents goals, audience, must-have vs nice-to-have, tone, constraints. Stakeholder sign-off on brief before research begins.
- [ ] Stakeholder vs user audiences distinguished; real-user validation sessions planned and budgeted.
- [ ] Keyword research done **before** sitemapping.
- [ ] Full sitemap + nav model approved before any page wireframe.
- [ ] Content audit (redesign) or content inventory (new site) complete; every page has owner + due date.
- [ ] Real copy for hero/value-props/CTAs exists before wireframing (no lorem as structural proxy).
- [ ] Priority guides (mobile, real content, no nav/boxes) per template.
- [ ] Lo-fi wireframes reviewed by a developer for feasibility/cost.
- [ ] Hi-fi covers all states (default/loading/empty/error/success); responsive at 375/768/1280/1440px minimum.
- [ ] Stakeholder sign-off on hi-fi direction documented before build begins.
- [ ] Tokens use 3-layer architecture; semantic names match codebase; agreed in cross-discipline workshop; light/dark/brand modes; versioned package.
- [ ] Focus-ring token ≥ 3:1 vs light & dark; touch targets ≥ 44×44px (exceeds WCAG 2.2 AA SC 2.5.8 24px floor; matches Apple HIG / AAA); contrast 4.5:1 body / 3:1 large text.
- [ ] Prototype covers all test task paths; stranger-test passes (no narration needed).
- [ ] ≥1 round of 5-user moderated think-aloud testing; issues severity-ranked (critical/serious/moderate/minor); fixes applied and re-tested.
- [ ] Handoff: single file link, emoji-status pages, devs as Viewers, component doc frames (name/desc/usage/variants/states/a11y notes); breakpoints supplied.
- [ ] Performance budget set pre-build, enforced in CI; LCP ≤ 2.5s / INP ≤ 200ms / CLS ≤ 0.1 verified in lab.
- [ ] Visual regression baseline committed (Chromatic/Percy) before first sprint.
- [ ] Zero critical/serious axe violations + manual keyboard/SR (NVDA+Firefox, VoiceOver+Safari)/zoom/reduced-motion pass (WCAG 2.2 AA).
- [ ] Redirect audit complete (redesigns): all old URLs 301 to new targets, no chains >2 hops, verified by crawl.
- [ ] SSL, redirects, analytics verified (GA4 DebugView conversion path fires within 2 min) before launch.
- [ ] RUM monitoring live at 75th pct; alerts at 80% of CWV thresholds (INP >160ms, LCP >2.0s, CLS >0.08).

## Sources
- British Design Council — Double Diamond: https://www.designcouncil.org.uk/our-resources/the-double-diamond/
- Nielsen Norman Group — Why You Only Need to Test with 5 Users: https://www.nngroup.com/articles/why-you-only-need-to-test-with-5-users/
- A List Apart — Priority Guides: https://alistapart.com/article/priority-guides-a-content-first-alternative-to-wireframes/
- Baymard Institute — Checkout UX research & cart abandonment: https://baymard.com/research/checkout-usability
- Deque — axe-core / accessibility coverage research: https://www.deque.com/
- Google — Core Web Vitals: https://web.dev/articles/vitals
- Google — INP replaces FID: https://web.dev/blog/inp-cwv-march-2024
- WCAG 2.2 (W3C Recommendation): https://www.w3.org/TR/WCAG22/
- Apple Human Interface Guidelines: https://developer.apple.com/design/human-interface-guidelines/

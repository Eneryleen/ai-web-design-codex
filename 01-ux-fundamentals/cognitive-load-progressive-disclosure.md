# Cognitive Load & Progressive Disclosure

> How to make interfaces feel effortless by managing the user's finite mental bandwidth: chunk information, defer secondary features, prefer sensible defaults, convert recall into recognition, and cut the number of decisions a screen demands. Master this and your designs feel "obvious" — users complete tasks faster, with fewer errors and less abandonment, without losing capability for power users.

**Apply when:** any screen with >1 decision, >1 choice, or >5 fields — onboarding, forms, settings, dashboards, checkout, pricing, empty states · **Related:** [Laws of UX](./laws-of-ux.md) · [Nielsen's Heuristics](./nielsen-heuristics.md) · [Forms & Input UX](./forms-and-input-ux.md) · [Information Architecture & Navigation](./information-architecture-navigation.md)

## TL;DR — rules at a glance
- **Reduce simultaneous load, not capability.** Defer, group, and stage — never amputate. Power users must still reach everything.
- **Max progressive-disclosure depth = 2 levels.** Level 1 = show/hide toggle; level 2 = primary + secondary screen. 3+ levels = redesign the IA (NNg).
- **Feature split by frequency, not difficulty.** Frequently-needed → initial view. Rarely-needed → secondary view. Hiding a *frequent* feature is a UX regression, not a simplification.
- **Miller's 7±2 is about chunks, not items — and is outdated.** Design novel content for **4±1** (Cowan 2001). Use it to justify *chunking*, never to cap nav at 7.
- **Recognition beats recall** (Shepard 1967). Show menus, history, breadcrumbs, recently-used — don't make users remember.
- **Hick's Law:** decision time ≈ `a + b·log2(n+1)`. Cut choices when speed matters; highlight ONE "recommended" option as a recognition anchor.
- **Sensible defaults win.** Pre-fill the probable answer; pick the default that minimizes the *worst-case* consequence of inaction.
- **Chunk forms into thematic sections** (personal / shipping / payment), not one wall of fields.
- **Empty states are designed moments:** one plain positive sentence + exactly ONE CTA + optional starter content. Never a blank screen or lone folder icon.
- **Checkout target = 8 form fields** (industry avg 11.3). Field count hurts more than step count (Baymard).
- **Onboarding is continuous, contextual, and always skippable.** Surface help at first use of each feature; never trap users in a modal tour.
- **Keep responses < 400ms** (Doherty Threshold) — wait states add perceived decision burden and break flow.
- **"Number of clicks" is the wrong metric.** Measure task time, error rate, subjective effort. One surprising confirm-dialog > three predictable clicks.
- **Jakob's Law:** users learn on other apps. Novel patterns cost germane load to learn — every deviation from convention needs to earn that cost.

---

## What cognitive load actually is

Three additive types; you can only really control the last two:
- **Intrinsic** — inherent task difficulty (filing taxes is hard). Fixed by the domain.
- **Extraneous** — load imposed by *how* the UI presents the task. **This is your job to eliminate.** Bad layout, jargon, hidden state, unclear next step.
- **Germane** — productive effort the user spends learning the model. Preserve this; don't dumb things into uselessness.

Working memory is the constraint. Pop psychology cites **7±2** (Miller 1956), but that figure included long-term-memory associations. Pure working-memory capacity is **4±1 chunks** (Cowan 2001). Under stress it shrinks further to **3–4 items**. So:
- For **novel** content (first-run, unfamiliar feature) design for **4**.
- For **high-stress / safety-critical / enterprise** flows design for **3–4** ("worst-case mental bandwidth," not average).
- Never invoke "7±2" to justify a hard cap (e.g. "max 7 nav items"). The lever is **chunking quality**, not chunk count — *lawsofux.com*: "Don't use the magical number seven to justify unnecessary design limitations."

## Chunking — group meaning, not atoms

Users read clusters of meaning, not isolated fields. Build clusters with Gestalt cues: **proximity, shared color, spacing, borders, headings**. (See [Visual Design Principles & Gestalt](../00-foundations/visual-design-principles.md).)

**Strings:** chunk long identifiers so they fit working memory.
```
4111111111111111   → 4111 1111 1111 1111   (card)
+15551234567       → +1 (555) 123-4567      (phone)
```

**Forms:** break the wall of fields into thematic sections.
```html
<form>
  <fieldset><legend>Contact</legend> … </fieldset>
  <fieldset><legend>Shipping address</legend> … </fieldset>
  <fieldset><legend>Payment</legend> … </fieldset>
</form>
```
Each `<fieldset>/<legend>` is a chunk — also the correct accessible grouping (screen readers announce the legend per field). Three labeled chunks of 4 fields beats one undifferentiated list of 12.

**Navigation / menus:** group items into 3–6 labeled categories rather than a flat list of 20. Label quality determines chunk quality. Derive groupings from **card sorting**, not gut feel — see [User Research & Testing](./user-research-and-testing.md).

## Progressive disclosure — defer, don't delete

Show only frequently-needed options up front; move the rest one step away. Keep full capability; reduce *simultaneous* load.

**Two forms, both valid:**
- **Staged disclosure** — linear, one step at a time: onboarding wizard, multi-step checkout, setup flow. Reduces load by sequencing.
- **Conditional disclosure** — hide rarely-needed fields until triggered: Address Line 2, coupon code, "Advanced settings," conditional follow-up questions. Reduces load by context-gating.

### Rule 1 — split by FREQUENCY, not by difficulty (NNg)
Frequently-needed options **must** stay in the initial view. Rarely-needed go secondary.
> Putting a *confusing but important and frequent* feature in a secondary screen fails users as badly as showing everything at once. Difficulty is a reason to *explain*, not to *hide*.

### Rule 2 — make the progression obvious (NNg)
Always provide a clear, discoverable affordance to reach more detail. A weak "Advanced ▸" toggle creates **discoverability debt** — the users who need the hidden feature never find it.

### Rule 3 — max practical depth = 2 levels (NNg)
- **1 level** = inline show/hide toggle (`<details>`, "More options").
- **2 levels** = primary screen → secondary screen/panel.
- **3+ levels** = users lose orientation moving between layers. This is a signal to **redesign the IA**, not to nest deeper.

**Level-1 disclosure, native and accessible:**
```html
<details>
  <summary>Advanced options</summary>
  <!-- rarely-needed controls live here, collapsed by default -->
</details>
```
`<details>/<summary>` gives you keyboard focus, `aria-expanded`, and toggle state for free. Reach for a JS pattern only when you need animation or coordinated state.

### Mobile-specific disclosure patterns
Mobile compounds cognitive load: smaller viewport, thumb-reach constraints, interruption-prone context. Different disclosure primitives apply:
- **Bottom sheet** (half-screen drawer) — replaces modal dialogs for secondary choices; keeps context visible behind it.
- **Accordion** — preferred over nested navigation for secondary content on mobile; keeps the user on one scroll-context.
- **Full-screen takeover** — only for complex sub-tasks (address lookup, date picker, multi-step sub-flow) that genuinely need the full canvas.
- **Swipe-to-reveal actions** — expose destructive or secondary actions (delete, archive) without adding visible buttons; use with caution because discoverability is low unless taught via onboarding or conventions (e.g., iOS Mail pattern most users recognize).
- **Floating Action Button (FAB)** — one primary action; never more than one FAB per screen. A second FAB is a disclosure failure (should be a speed dial or a redesigned bottom nav).

**Rule:** never use hover-triggered disclosure on mobile. Any "show on hover" pattern must have a tap-based equivalent or be replaced entirely.

### Splitting input: analytics ≠ intent
Use **task analysis + card sorting** to decide what's primary vs secondary. Analytics tells you *what* users clicked, not *whether they meant to*. A high-traffic setting reached by accident is not a primary feature.

## Reducing choices — Hick's Law in practice

Decision time grows logarithmically with option count: `RT = a + b·log2(n+1)` (Hick & Hyman 1952). Each added option has diminishing *time* cost but accumulating *cognitive* cost. Schwartz's **Paradox of Choice** proposes that beyond a moderate count, satisfaction itself drops — more regret and anxiety, not just slower decisions. **Caution:** the underlying "choice overload" effect has mixed replication (Scheibehenne et al. 2010 meta-analysis, n = 50 experiments, found no reliable mean effect size). The Hick's Law time cost is robust; the satisfaction degradation is context-dependent. Don't over-extrapolate: reduce choices where speed and low-stakes decisions dominate; for high-stakes personal choices (insurance, medical) users often *want* more options.

Tactics, in order of preference:
1. **Remove** options that don't earn their place.
2. **Defer** secondary options (progressive disclosure).
3. **Group** options into scannable categories (chunking defeats the linear penalty — an alphabetized country dropdown is cheap because users jump, not scan).
4. **Anchor** with a recommended default — highlight ONE "Most popular" / "Recommended" choice.

```html
<!-- Pricing: one highlighted plan converts a recall task into a recognition task -->
<div class="plan plan--recommended" aria-label="Recommended plan">
  <span class="badge">Most popular</span> Pro — $29/mo
</div>
```
This turns "evaluate all N plans" (recall, slow) into "is this the one for me?" (recognition, fast). Users pick highlighted options at higher rates.

**Caveat:** don't over-simplify into uselessness. Hiding so much that power users can't reach their goals violates the *same* law it's trying to serve. The goal is *fast correct decisions*, not *fewest visible things*.

## Recognition over recall (Nielsen Heuristic #6)

> Minimize memory load by making objects, actions, and options visible. Users shouldn't have to remember information from one part of the interface to another.

The directional gap is well established in cognitive psychology (Shepard 1967, Bahrick et al. 1975): recognition memory substantially outperforms free recall, particularly for visual and contextual cues. The practical design implication holds regardless of exact percentages: every recall burden you convert to recognition meaningfully reduces error and effort.

| Recall burden (bad) | Recognition substitute (good) |
|---|---|
| Type a command | Pick from a menu |
| Remember a file path | Search results / recently-opened |
| Re-enter prior answer | Pre-filled value / "same as above" |
| Remember where you are | Breadcrumbs |
| Recall a past search | Search history / suggestions |
| Memorize an ID across screens | Show it persistently / pass it through |

Practical instances: autocomplete, "Recently viewed," breadcrumb trails, persistent selection state, inline previews, and carrying entered data forward instead of asking again. See [Nielsen's Heuristics](./nielsen-heuristics.md).

## Sensible defaults

The default is the most powerful element on the screen — most users never change it.

- **Pre-fill the probable answer.** Country from locale/IP, currency from region, quantity = 1.
- **Pre-check the common option** (e.g. "Same as shipping address").
- **Surface recent choices** (last-used payment method, recent files).
- **Default to minimize worst-case consequence of inaction.** When unsure, ask: "If the user changes nothing, what's the worst outcome?" Choose the default whose failure mode is least harmful.

**Anti-case — Apple iOS 17 Calendar.** Apple silently changed new-event defaults in ways that surprised long-time users who had calibrated their workflows to the prior behavior. The worst case (missing an important meeting) far outweighs the annoyance of an unwanted ping. **Principle of Least Surprise:** keep historically established defaults unless data strongly justifies the change — and weight the *worst-case consequence* of the wrong default, not its average cost.

## Removing friction

Friction = anything between intent and completion: extra fields, extra steps, waits, surprises, re-entry.

- **Cut form fields first.** Baymard: **field count impacts checkout UX far more than step count.** Target **8** fields; industry average is **11.3**.
- **Single full-name field.** 42% of users type their full name into a "First Name" field; a first+last split causes rework. Yet only **11%** of sites use one combined name field.
- **Hide Address Line 2 by default** — **30%** of users *pause* when they hit it. Only **25%** of sites defer it.
- **Hide the coupon-code field by default.** A visible empty promo field makes users *leave* checkout to hunt for a code. **35%** of sites still show it unconditionally.
- **Defer account creation to post-purchase.** **26%** of cart abandonments are caused by forced account creation; only **16%** of sites defer it.
- **Pre-check "same as shipping."**
- **Keep responses < 400ms** (Doherty Threshold). Above 400ms attention drifts, context is lost, and the wait itself reads as friction. When you can't meet 400ms: **skeleton screens** (show content shape immediately) reduce perceived wait more than spinners because they prime the user's model of what's loading — the content slot is already placed. **Optimistic UI** (apply the action immediately, revert on failure) is the strongest pattern: zero perceived latency. **Prefetch** on hover/intent signals for predictable transitions. See [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md).
- **Stop optimizing click count.** A single click that fires a surprise confirm dialog costs *more* than three predictable clicks. Optimize **task completion time, error rate, and subjective effort** instead.
- **Error prevention reduces recovery cognitive load.** Inline validation (on blur, not on keystroke), input constraints (date pickers instead of free-text dates, masked inputs for card/phone numbers), and confirmation dialogs on destructive actions eliminate the heaviest cognitive load: error recovery. Nielsen Heuristic #5. See [Nielsen's Heuristics](./nielsen-heuristics.md).
- **Jakob's Law:** users spend most of their time on *other* interfaces. Familiar patterns (standard checkout flow, conventional nav placement, recognized icon vocabulary) reduce extraneous load because the model is already loaded from prior experience. Novel patterns require germane load to learn — justify every deviation. See [Laws of UX](./laws-of-ux.md).

## Designing empty states

An empty state is a designed moment, not an absence. A blank screen — or a lone folder icon — reads as a *bug or loading error* and produces abandonment and support tickets.

A first-run empty state must:
1. **Explain *why* it's empty** in one plain, positively-framed sentence. "Start by adding your first project" — not "You have no projects."
2. **Offer exactly ONE prominent CTA** toward filling it. Multiple competing CTAs dilute intent at the precise moment the user has nothing to orient them.
3. **Optionally seed starter/sample content** so users can explore without creating from scratch.
4. **Match the message to the user's mental state** in that context (first-run vs filtered-to-empty vs error are three different empty states with different copy).

**Heuristic: two parts instruction, one part delight.** Guidance first; personality second.

```html
<div class="empty-state">
  <img src="/illustrations/projects.svg" alt="" />     <!-- decorative -->
  <h2>Start by adding your first project</h2>
  <p>Projects keep your tasks, files, and team in one place.</p>
  <button class="btn-primary">New project</button>      <!-- exactly one CTA -->
  <a href="/templates">or start from a template</a>      <!-- secondary, de-emphasized -->
</div>
```
Distinguish the three empty states: **first-run** (teach + CTA), **no-results / filtered-to-empty** ("No matches — clear filters?"), **error** (state what failed + a real recovery action, never just "Something went wrong"). See [Interaction Design & UI States](./interaction-design-and-states.md).

## Onboarding — continuous, contextual, skippable

Onboarding is a **process spanning the first week**, not a one-time tour. Front-loading every feature into a product tour overflows working memory on day one, when retention is lowest.

- **Surface guidance at the moment of first use** of each feature (contextual tooltips, empty-state hints, inline coachmarks) — not all at once up front.
- **Always allow skip/exit.** Unskippable tours are a top anti-pattern: they cause abandonment and resentment, especially among experienced users who already know the pattern. "Allow skipping" appears on every credible best-practices list.
- **Reduce time-to-first-value.** Use sensible defaults, sample data, and templates so the user sees a result before configuring anything.
- **Prefer staged disclosure** (do one real thing, then the next) over passive demos.

**Progress-bar caution.** "Step X of Y" bars normally help (goal-gradient effect). But they backfire when they emphasize remaining steps over proximity to completion — particularly when early steps are fast and late steps are heavy. Communicate **nearness to done** (e.g., "Almost there — one last step"), not just remaining count. For short flows (≤3 steps) a progress bar may add load without benefit. Test it; don't apply blindly.

## Minimizing the number of decisions

Every decision the UI forces is a tax. Audit each screen: *which decisions can be removed, deferred, defaulted, or decided for the user?*

Decision-reduction ladder (apply top-down):
1. **Decide for them** (sensible default, smart pre-fill, autodetect).
2. **Remove** the decision (does it need to exist?).
3. **Defer** it (progressive disclosure to a later step or secondary screen).
4. **Anchor** it (recommended option, sensible pre-selection).
5. **Only then present it** — chunked and scannable.

Tesler's Law: complexity is conserved — absorb it into the system (autofill, address lookup, smart defaults) instead of dumping it on the user.

**Decision fatigue.** Quality of decisions degrades after a sequence of choices — users make worse or more impulsive choices late in a long flow (Baumeister et al.; Danziger et al. 2011 parole board study). Mitigations:
- Put the most critical decision **first**, not last (where fatigue peaks).
- Break long decision sequences across sessions where the domain allows it (multi-session onboarding, split checkout).
- Bundle trivial decisions into smart defaults so they never reach the user.
- "Save progress" + resume for complex enterprise forms prevents restart penalty.

## Concrete specs & numbers
- **Working memory:** 7±2 chunks (Miller 1956, dated) → **4±1** for novel content (Cowan 2001) → **3–4** under stress.
- **Progressive disclosure depth:** **max 2 levels**; 3+ → redesign IA (NNg).
- **Recognition vs recall:** recognition substantially outperforms free recall (Shepard 1967; Bahrick et al. 1975). Exact percentages vary by domain; the directional gap is robust.
- **Doherty Threshold:** keep system+human response **< 400ms** to hold flow.
- **Checkout:** target **8 form fields**; industry avg **11.3**; avg **5.1 steps** (Baymard 2024).
- **Cart abandonment:** **70.19%** overall · **80.02%** mobile · **66.41%** desktop (Baymard, aggregating 49 studies 2006–2023).
- **Abandonment causes:** **26%** mandatory account creation · **21%** "complicated checkout" · **17%** checkout complexity.
- **Recoverable upside:** checkout UX fixes can lift conversions up to **35.26%** (~$260B/yr US+EU, Baymard).
- **Field-level friction:** **42%** type full name in "First Name" · **30%** pause on Address Line 2.
- **Site adoption gaps (2024):** single name field **11%** · hide Address Line 2 **25%** · defer account creation **16%** · still show coupon field **35%**.
- **Hick's Law:** `RT = a + b·log2(n+1)` (Hick & Hyman 1952).
- **Choice overload / Paradox of Choice:** Schwartz 2004; *caveat* — Scheibehenne et al. 2010 meta-analysis (n=50 studies) found no reliable mean effect size. Effect is real in specific low-stakes fast-decision contexts; not universal.
- **Decision fatigue:** Baumeister et al.; Danziger et al. 2011 (parole board). Quality of decisions degrades late in a decision sequence.
- **Jakob's Law:** users build mental models on other sites; familiar patterns reduce extraneous load. (Laws of UX.)

## Tools & libraries
- **Native HTML `<details>/<summary>`** — first choice for level-1 disclosure. Free a11y and state. Use a JS lib only when you need height animation.
- **`<fieldset>/<legend>`** — correct semantic chunking for forms; don't fake it with `<div>` + heading.
- **Headless UI primitives** (Radix, React Aria, Headless UI) — accessible Disclosure / Accordion / Tabs / Combobox for multi-level disclosure with proper focus & ARIA. See [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md).
- **Combobox / autocomplete** — converts recall (type exact value) into recognition (pick from filtered list).
- **Card-sorting tools** (Maze, OptimalSort) — derive the primary/secondary split and chunk groupings from real users, not guesses.
- **When NOT to reach for a library:** a single show/hide toggle, a recommended-plan badge, or a pre-filled default need zero dependencies. Don't pull in an accordion lib for one `<details>`.

## Do / Don't
**Do**
- Split features by **frequency**; keep frequent ones visible.
- Cap disclosure at **2 levels**; redesign IA past that.
- Design novel/high-stress content for **4 (or 3–4)** items, not 7.
- Chunk forms into labeled `<fieldset>` sections.
- Use **one** highlighted/recommended option as a recognition anchor.
- Pick defaults that minimize the **worst-case** of inaction.
- Give empty states one positive sentence + one CTA (+ optional starter content).
- Make onboarding contextual, incremental, and **skippable**.
- Keep interactions **< 400ms**.
- Measure **task time / error rate / effort**, not click count.

**Don't**
- Hide a **frequently-used** feature behind a toggle.
- Nest **3+** disclosure levels.
- Cite **7±2** to cap nav or any list at 7.
- Stack **multiple competing CTAs** on an empty state.
- Ship a **blank** empty state or lone folder icon.
- Show "**Something went wrong**" with no recovery action.
- Force a **product tour** before product access.
- **Force account creation** before purchase.
- Show the **coupon field** by default.
- Default to the **minority** use case or silently break a historically established default.

## Common pitfalls & how to avoid them
- **The wrong feature split.** Hiding a frequent feature isn't simpler UX — it's the same complexity + an extra click + a discoverability problem. → Split by frequency (data from task analysis), not by perceived difficulty.
- **3+ disclosure levels.** Users lose orientation between layers. → Treat it as an IA smell; flatten and re-group, don't nest deeper.
- **Treating 7±2 as a hard constraint.** Capping things at 7 "because Miller." → Chunk by meaning; design novel content for 4±1.
- **Over-simplification.** Hiding so much that power users can't finish. → Defer/group, never amputate; keep a discoverable path to advanced.
- **Blank or icon-only empty states.** Read as a bug → abandonment/tickets. → One plain sentence + one CTA + optional starter content.
- **Multiple CTAs on empty states.** Overload at the moment of least orientation. → Exactly one primary CTA; de-emphasize the rest.
- **"Something went wrong" with no guidance.** → State what failed + a concrete recovery action.
- **Unskippable tours.** Block access, breed resentment. → Always provide skip/exit; favor contextual help.
- **Mandatory account creation pre-purchase.** Causes 26% of abandonments. → Defer to the post-purchase confirmation page.
- **Always-visible coupon field.** Sends users off-site for codes. → Collapse behind "Have a code?".
- **Progress bars signaling "more steps remain."** Can inflate perceived effort. → Communicate proximity to done; reconsider for ≤3-step flows.
- **Minority-use or silently-changed defaults.** → Principle of Least Surprise + worst-case-consequence test. When changing an established default, explicitly surface the change to affected users (migration notice, one-time prompt) rather than silently breaking their setup.
- **Optimizing click count.** → Optimize predictability and task time instead.

## What real users complain about
Illustrative cases of these principles failing in production:
- **Discord** — once praised for best-in-class UX, now described by long-time users as "completely bloated with settings and functions"; users report "feature overload" and "getting lost in levels of servers and channels." A live failure of the feature-split rule as complexity accreted over time without an IA redesign.
- **Tana (PKM)** — users report spending "hours and days" on basic setup, unable to follow tutorials because the tool is "so fiddly," describing it as "building an operating system without a good interface." A cottage industry of paid templates and consulting emerged around its onboarding friction. Many users switched to Notion or Capacities. A progressive-disclosure / onboarding failure at scale.
- **"Something went wrong"** errors with zero actionable guidance — a persistent anti-pattern that has appeared in UX practitioner discussions consistently since the 1990s and remains endemic in 2025+. The problem is structural: engineers write catch-all error handlers without UX input on recovery messaging.
- **Multi-step "Step X of Y" progress bars** — in some checkout flows, users report the progress bar making the process *feel* longer by constantly signaling remaining steps, rather than communicating nearness to done. Applies particularly to flows where early steps are lightweight but later steps are heavy.
- **Apple iOS Calendar default changes** — users missed events after new-event defaults were silently altered across iOS updates; a concrete Principle of Least Surprise failure for a high-consequence context (missed meetings).

## Senior-level checklist
- [ ] Every screen audited: which decisions can be **removed / deferred / defaulted / decided for the user**?
- [ ] Disclosure depth **≤ 2 levels** everywhere; any 3+ flagged for IA redesign.
- [ ] Primary/secondary split derived from **task analysis + card sorting**, not raw analytics.
- [ ] Frequent features are **visible**; only rarely-needed ones are deferred.
- [ ] Disclosure affordances are **discoverable** (no hidden "Advanced" that key users miss).
- [ ] Novel content designed for **4±1**; stress/critical flows for **3–4**.
- [ ] Long strings & long forms are **chunked** into labeled groups (`<fieldset>/<legend>`).
- [ ] Recall burdens converted to **recognition** (menus, history, breadcrumbs, recently-used, carried-forward data).
- [ ] One **recommended/highlighted** option per choice set (pricing, plans).
- [ ] Defaults chosen by **worst-case-of-inaction** test; historical defaults preserved unless data overrides.
- [ ] Checkout ≤ **8 fields**; single name field; Address Line 2 & coupon **collapsed**; account creation **deferred**; "same as shipping" **pre-checked**.
- [ ] Every empty state: one **positive sentence** + **one CTA** (+ optional starter content); distinct first-run / no-results / error variants.
- [ ] No "Something went wrong" without a **recovery action**.
- [ ] Onboarding **contextual, incremental, skippable**; time-to-first-value minimized.
- [ ] Interactions held **< 400ms**; waits masked (optimistic UI, skeletons, prefetch).
- [ ] Success measured by **task time, error rate, subjective effort** — not click count.
- [ ] Error prevention in place: **inline validation** (on blur), **input constraints** (pickers/masks), **confirmation on destructive actions**.
- [ ] Heavy decision sequences: most critical decision **first**; trivial decisions defaulted; long flows support **save + resume**.
- [ ] Novel interaction patterns justified — **Jakob's Law** applied: deviate from convention only when the gain outweighs the learning cost.
- [ ] Mobile disclosure uses **bottom sheets / accordions** (not hover patterns, not nested navigation).

## Sources
- Nielsen Norman Group — Progressive Disclosure: https://www.nngroup.com/articles/progressive-disclosure/
- Nielsen Norman Group — Recognition vs Recall (Heuristic #6): https://www.nngroup.com/articles/recognition-and-recall/
- Laws of UX — Hick's Law: https://lawsofux.com/hicks-law/
- Laws of UX — Miller's Law: https://lawsofux.com/millers-law/
- Laws of UX — Doherty Threshold: https://lawsofux.com/doherty-threshold/
- Laws of UX — Jakob's Law: https://lawsofux.com/jakobs-law/
- Baymard Institute — Checkout Usability / Cart Abandonment: https://baymard.com/research/checkout-usability
- Baymard Institute — Cart Abandonment Rate Statistics: https://baymard.com/lists/cart-abandonment-rate
- Miller, G. A. (1956) "The Magical Number Seven, Plus or Minus Two": https://psychclassics.yorku.ca/Miller/
- Cowan, N. (2001) "The magical number 4 in short-term memory": https://doi.org/10.1017/S0140525X01003922
- Shepard, R. N. (1967) "Recognition memory for words, sentences, and pictures." *Journal of Verbal Learning and Verbal Behavior*, 6(1), 156–163.
- Scheibehenne, B., Greifeneder, R., & Todd, P. M. (2010) "Can there ever be too many options? A meta-analytic review of choice overload." *Journal of Consumer Research*, 37(3), 409–425. https://doi.org/10.1086/651235
- Danziger, S., Levav, J., & Avnaim-Pesso, L. (2011) "Extraneous factors in judicial decisions." *PNAS*, 108(17), 6889–6892. https://doi.org/10.1073/pnas.1018033108

# Nielsen's Usability Heuristics & Practical Usability
> The 10 heuristics are general principles for interaction design, derived by Nielsen & Molich (1990, refined 1994) from factor analysis of 249 usability problems. Language updated 2020. They are grounded in human psychology, not technology, so they apply to any web UI. This guide turns each into concrete, buildable web patterns plus a step-by-step heuristic evaluation method you can run on your own output.

**Apply when:** designing or auditing any interactive UI; self-reviewing a build before handoff · **Related:** [Laws of UX](./laws-of-ux.md) · [Interaction Design & UI States](./interaction-design-and-states.md) · [Cognitive Load & Progressive Disclosure](./cognitive-load-progressive-disclosure.md) · [Forms & Input UX](./forms-and-input-ux.md)

## TL;DR — rules at a glance
- **Status feedback within budget:** ack within 100 ms, hold thought-flow under 1 s, show a percent-done bar + cancel beyond 10 s.
- **Speak the user's language.** No internal jargon, no codes in UI copy. Match real-world order and mental models.
- **Always provide an exit:** Undo, Cancel, Back, Discard. Never break or hijack the browser Back button.
- **Obey Jakob's Law.** Users spend most time on *other* sites; match platform/industry convention before inventing.
- **Prevent errors first.** Constraints, smart defaults, inline validation, confirm before irreversible. The best error message never appears.
- **Recognition beats recall.** Show options; don't make users remember them. Autocomplete > blank field; thumbnail > filename. Biggest payoff for infrequent users, older adults, and users under cognitive stress.
- **Serve novices and experts both:** discoverable defaults + keyboard shortcuts + accelerators + personalization.
- **Minimalist = signal-to-noise, NOT flat/sparse.** Every element competes; cut the irrelevant. Use progressive disclosure.
- **Error messages = plain language + exact problem + concrete fix.** Bold red, no stack traces.
- **Help is context-delivered, searchable, task-focused, step-by-step.** Best case: no docs needed.
- **Heuristic evaluation:** 3–5 independent evaluators find ~75% of issues; rate severity 0–4; any 4 is a release blocker.
- **Heuristic eval complements user research — it does not replace it.**

## The 10 Heuristics, applied to the web

### H1 — Visibility of System Status
Always keep the user informed about what is going on, through appropriate feedback within a reasonable time. Predictable interactions reduce uncertainty and build trust; a user who knows the current state can decide the next step.

**Web patterns**
- **Response-time budget (Nielsen's 3 thresholds):** see [specs](#concrete-specs--numbers). Ack input ≤100 ms; under 1 s no spinner needed; beyond 10 s require a percent-done indicator + a way to interrupt.
- **Loading:** skeleton screens for content-heavy pages (illusion of immediate activity); spinners only for short indeterminate waits (<~3 s); progress **bars** (not spinners) for long determinate ops.
- **Optimistic UI:** for low-risk actions (like, reorder), reflect the change instantly then reconcile — Amazon Music's drag-reorder gives immediate visual feedback.
- **Persistent state cues:** Amazon Echo light ring (listening/processing); Gmail "typing…" indicator in chat; toast/snackbar after an action completes.
- **Inventory/commerce status:** J.Crew "Only a Few Left!" on low-stock sizes, sold-out shown struck through; NatureBox banner showing remaining spend to unlock free shipping; Loft persists out-of-stock wishlist items with explanatory copy.
- **Form/save state:** show "Saving…" → "Saved" (Google Docs autosave); disable submit + show progress during network calls.

```html
<!-- determinate progress for long ops, with cancel -->
<div role="progressbar" aria-valuenow="42" aria-valuemin="0" aria-valuemax="100" aria-label="Uploading">42%</div>
<button type="button" data-action="cancel-upload">Cancel</button>
```

### H2 — Match Between System and the Real World
Use words, phrases, and concepts familiar to the user. Follow real-world conventions; make information appear in a natural and logical order. The canonical mapping example: stovetop controls laid out to mirror the burners.

**Web patterns**
- **Plain labels over system terms:** "Delete account" not "Deactivate user record"; "Trash" not "Soft-delete flag = true."
- **Natural ordering:** dates as the locale expects (MM/DD/YYYY in the US; DD/MM/YYYY in most of Europe/AU; YYYY-MM-DD ISO in dev contexts). Currency symbols, decimal separators (1,234.56 vs. 1.234,56), and units of measure (kg vs. lb, km vs. mi) must match locale — a mismatch is a H2 failure even if the data is correct. Checkout steps in the order people actually shop → review → pay.
- **Familiar icons mapped to real-world objects:** trash = delete, cart = basket, envelope = mail.
- **Research the terminology — don't assume your mental model is the user's.** Run card sorts / interviews ([User Research](./user-research-and-testing.md)) to surface the words your audience uses.
- **Spatial expectation from the real world:** hotel check-in is at the entrance → put "Sign in" / primary CTA where users expect it.
- **Anti-pattern:** abbreviation ambiguity — Facebook's Danish localization once rendered "Do not like" as "Do … like," inverting meaning (violates H2 and H4).

### H3 — User Control and Freedom
Users pick functions by mistake and change their minds. Provide clearly marked "emergency exits" to leave the unwanted state without an extended dialogue. Support Undo and Redo.

**Web patterns**
- **Undo over confirm where possible.** Gmail "Undo Send" snackbar appears for a few seconds; Google Docs / PowerPoint provide multi-step Undo + version history/restore.
- **Multi-step Undo:** if users make many actions in quick succession, undoing only the *last* action is insufficient — keep an undo stack.
- **Clearly labeled exits:** "Cancel," "Back," "Discard," "Undo" — never a bare "OK"/"Apply." Visible without scrolling, reachable without a forced dialogue.
- **Non-destructive flows:** Slack keeps archived messages searchable rather than deleting.
- **Snackbar pattern:** perform the action immediately, surface a time-limited Undo instead of a blocking confirm. Standard snackbar dismiss windows are 4–10 s. Gmail "Undo Send" is a separate, user-configurable delay (5, 10, 20, or 30 s) — the snackbar dismiss window and the undo window are not the same thing; size each to the cost of the irreversible action.
- **Anti-patterns (never ship these):**
  - Disabling/hijacking the browser **Back** button; forms that lose data + time out on Back.
  - YouTube Shorts: removing a text overlay requires opening comments, dragging the video pane down, and holding it — a hidden, undiscoverable exit.
  - TikTok: an up-swipe sometimes triggers "Shop"/"LIVE" overlays — accidental state change with no obvious exit.

### H4 — Consistency and Standards
Users should not have to wonder whether different words, situations, or actions mean the same thing. **Jakob's Law:** users spend most of their time on *other* products, so their expectations are formed elsewhere. Maintain **internal** consistency (within your product) and **external** consistency (with platform/industry conventions).

**Web patterns**
- **Internal:** one term per concept; one primary-button style; consistent icon meaning, spacing scale, and interaction across pages → use design tokens ([Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md)).
- **External:** logo top-left links home; cart top-right; underlined/colored = link; floppy-disk icon = Save (still recognized by most users who grew up with desktop software; recognition is declining among users under ~25 as of 2026 — pair with a "Save" label in new products targeting younger audiences).
- **Honor established form conventions** even when you think you can improve them: the US-state `<select>` is expected to be a big dropdown so users can type "p" for Pennsylvania — replacing it adds friction because the pattern lives across thousands of forms.
- Inconsistency forces re-learning; that cost usually exceeds any cleverness gained.

### H5 — Error Prevention
Even better than good error messages is a design that prevents problems in the first place. Eliminate error-prone conditions, or check and confirm before commit.

**Two error types (design differently for each):**
- **Slips** — unconscious, attention/motor lapses by users who know what to do → fix with constraints, larger targets, smart defaults, guard rails.
- **Mistakes** — conscious decisions based on a wrong mental model → fix with better matching (H2), clear info, and guidance.

**Prioritize preventing high-cost/irreversible errors first.**

**Web patterns**
- **Constraints:** date pickers that can't produce invalid dates; numeric inputs with min/max/step. **Disable-submit-until-valid is contested** — disabling before any interaction prevents users from discovering what's missing; NNGroup recommends keeping submit enabled and showing validation on first submit or on blur, not pre-emptively graying the button unless all requirements have been communicated up front.
- **Smart defaults:** pre-select the most common/safe option to prevent slips.
- **Inline validation:** validate per-field as the user leaves it, not only at submit; state requirements *before* errors ("8+ chars" shown under the field, not after failure).
- **Confirm/guard irreversible actions:** "Are you sure?" before deleting a file; type-to-confirm for catastrophic ones (e.g. repo name to delete a repo); guard rails (highway analogy) channel users away from cliffs.

```html
<input type="email" required autocomplete="email"
       aria-describedby="email-hint" />
<small id="email-hint">We'll only use this to send your receipt.</small>
```

### H6 — Recognition Rather Than Recall
Minimize memory load by making elements, actions, and options visible. The user should not have to remember information from one part of the interface to another. Recognition needs fewer cognitive cues than recall. Working-memory limits (Miller 1956: 7±2; Cowan 2001 revision: ~4 meaningful items) make recall-heavy UIs costly, especially for infrequent users, older adults, and users under cognitive stress.

**Web patterns (each converts recall → recognition)**
- **Autocomplete / typeahead:** Google & Bing search suggestions surface options instead of a blank field.
- **History surfaces:** Amazon "Recently Viewed," Netflix "Continue Watching."
- **Hierarchical menus** over command lines; **breadcrumbs** so users see where they are; **standardized icons** (magnifying glass = search, trash = delete).
- **Show, don't make them retype:** prefill known data; show thumbnails instead of filenames; show password rules + a "show password" toggle instead of relying on memory.
- H6 and H8 are mutually reinforcing: reducing irrelevant items (H8) lowers the recognition burden (H6).

### H7 — Flexibility and Efficiency of Use
Accelerators — unseen by novices — speed up interaction for experts, so the system serves both. Allow users to tailor frequent actions.

**Web patterns**
- **Keyboard shortcuts** with discoverable hints: Figma, Photoshop, VS Code, Gmail (`?` opens the shortcut sheet). Show shortcuts in menus/tooltips so novices graduate to them.
- **Accelerators:** type-ahead in menus, gesture shortcuts on touch, bulk actions, command palette (`⌘K`/`Ctrl+K`). Bulk-select + bulk-action is one of the highest-ROI expert affordances in list-heavy UIs.
- **Recently used / favorites surfaces:** "Recent projects" in Figma, pinned channels in Slack — eliminate repeated navigation for frequent destinations.
- **Templates and saved configurations:** expert users repeat complex setups; offering "save as template" converts a multi-step flow into a single click on the second use.
- **Focus and keyboard navigation discipline:** modals must trap focus correctly; Tab order must follow visual reading order; Escape must close overlays. These are not accessibility niceties — they are efficiency heuristics for keyboard-heavy users.
- **Personalization & customization:** customizable dashboards (Notion, Jira), saved views/filters, reorderable nav, "power-user mode" toggles.
- Don't force experts through novice-paced flows; don't hide *all* power behind invisible shortcuts either — layer them. Rule of thumb: if a user will do an action >10×/session, it needs a keyboard shortcut or accelerator.

### H8 — Aesthetic and Minimalist Design
Interfaces should not contain information that is irrelevant or rarely needed. Every extra unit of content/UI competes with the relevant units and diminishes their relative visibility. **This is about signal-to-noise ratio, NOT a flat/sparse visual *style*.**

**The distinction that AI agents get wrong**
- A **richly textured, layered** interface can fully satisfy H8 if every element serves a task — e.g., 16Personalities (busy, little whitespace, still minimalist *in content*).
- A **visually sparse** page can **fail** H8 — e.g., HelloBar's homepage is sparse yet gives too little information about what the product does. Sparse ≠ sufficient.

**Five visual attributes to manage signal** (detail in [Visual Design Principles](../00-foundations/visual-design-principles.md) / [Visual Hierarchy](../00-foundations/visual-hierarchy.md)): **scale, visual hierarchy, balance, contrast, Gestalt.** Evaluate a candidate design by asking: "does every element on this page earn its place by serving a task or communicating essential information?" If not, cut or disclose.

**Tactics**
- **Progressive disclosure:** reveal features/info as needed, not all at once ([Cognitive Load & Progressive Disclosure](./cognitive-load-progressive-disclosure.md)). Smart Countdown Timer's "zen mode" shows only essentials at idle; controls appear on hover.
- Cut rarely-needed content from primary views; move it to detail/expanded states.
- **Counter-example of feature bloat:** Discord — once exemplary UX, degraded post-2020 by stacking features (per practitioner complaints).

### H9 — Help Users Recognize, Diagnose, and Recover from Errors
Error messages must be expressed in plain language (no codes), precisely indicate the problem, and constructively suggest a solution. Use traditional error visuals (bold red text). Offer immediate recovery paths.

**The 3-part error message**
1. **Plain language** — no error codes, no stack traces, no jargon.
2. **Precisely indicate the problem** — *which* field, *what* is wrong.
3. **Constructively suggest a fix** — the next concrete action.

**Web patterns**
- Good: `"That email is already registered. Sign in instead, or reset your password."`
- Bad: `"Something went wrong"` (no cause, no action); a BSOD-style "contact the computer manufacturer" (useless for self-built/edge cases).
- **Place the message at the source** (inline next to the field), visually prominent (bold, red, icon), and keep the user's entered data.
- **Verify "Try again" is actually true** — confirm with engineering that a retry can succeed before telling the user to retry; otherwise it's a dead end ("Wrong Way" sign analogy).
- Never leak internal exceptions to end users; log them, surface a human message.

```html
<input id="card" aria-invalid="true" aria-describedby="card-err" />
<p id="card-err" role="alert" class="error">
  Card number looks incomplete — it should be 16 digits.
</p>
```
```css
/* #b00020 on white = ~5.9:1 contrast ratio — passes WCAG AA (4.5:1 minimum for normal text) */
.error { color: #b00020; font-weight: 700; }
```

### H10 — Help and Documentation
It's best if the system needs no documentation, but when help is required, make it easy to search, focused on the user's task, list concrete steps, and not be too large. Canonical analogy: the airport information kiosk — answers delivered at the point of need.

**Web patterns**
- **Context-delivered help:** tooltips, inline hints, "?" affordances at the exact decision point. These have zero cost for users who don't need them and high value for those who do.
- **Empty-state guidance:** when a list, dashboard, or inbox is empty, show what to do next — not just "No items found." Empty states are the most-neglected help surface.
- **Progressive onboarding:** coach marks and tooltips on first use; do not dump a full feature tour on every login. Dismiss permanently and never re-trigger.
- **Task-focused, step-by-step articles** ("How to export your data: 1… 2… 3…"), not reference dumps. Each article answers one task question; link between tasks, don't bundle them.
- **Searchable** help center; surface the most-searched/most-failed tasks first (pull this from support ticket analytics, not intuition).
- **Error state help:** every error message (H9) is also a help surface — link directly to the relevant help article from the inline error.
- **Help is a safety net, not a substitute for H2/H5/H6.** If users need the manual to complete a core task, the right fix is the UI, not better documentation. Measure: if >10% of support tickets are about a single flow, redesign that flow.

## How to run a heuristic evaluation
A usability **inspection** method: evaluators independently judge an interface against the 10 heuristics, then findings are consolidated. It is fast and cheap but **complements, not replaces, user research** (it predicts problems; testing confirms which real users hit).

**Process**
1. **Scope narrowly** for depth: one task/flow, one section or feature, one user group, or one device type.
2. **Recruit 3–5 evaluators.** Nielsen's 1992 data on software products: 1 evaluator finds ~35% of problems; 3–5 independent evaluators find ~75%; returns diminish sharply past 5. The exact percentages vary by product type and domain, but the shape of the curve (fast gains to 3–5, then diminishing) holds in replication studies.
3. **Evaluate independently.** Each evaluator inspects alone (≈1–2 hours/session, often two passes: one for flow, one for detail). Independence is mandatory — seeing peers' findings biases results toward conformity.
4. **Record each issue** with: the screen/step, the heuristic(s) violated, and a severity rating.
5. **Consolidate & de-duplicate** findings across evaluators; assign a **mean severity** per issue. Averaging across evaluators reduces individual-rater noise; use the mean as a directional guide, not a measurement.
6. **Prioritize & fix.** Severity 3–4 before launch; 1–2 scheduled. **Any single 4 is a release blocker.**

**Severity scale (0–4)**

| Rating | Meaning | Action |
|---|---|---|
| 0 | Not a usability problem | None |
| 1 | Cosmetic only | Fix if time permits |
| 2 | Minor | Low priority |
| 3 | Major | High priority — fix before release |
| 4 | Usability catastrophe | **Must fix before release (blocker)** |

A full evaluation typically takes a few days to ~two weeks depending on product complexity.

**Heuristic evaluation vs. cognitive walkthrough:** a cognitive walkthrough is a different inspection method that simulates a user completing a specific task step-by-step, asking at each step whether the user will know what to do and whether feedback confirms success. Heuristic evaluation is broader (whole interface, all heuristics); cognitive walkthrough is narrower but task-flow-precise. Use both on high-stakes flows.

**Personal preference vs. heuristic violation:** every finding must be grounded in a specific heuristic. "I don't like this layout" is not a finding. "This button is unlabeled — the user must recall its function from memory, violating H6 (recognition over recall) — severity 3" is a finding. Evaluators should cite the heuristic number and explain the user harm in every issue.

**AI self-review use:** after building a flow, run a structured pass — for each of the 10 heuristics, list concrete violations on the screens you built, cite the heuristic, explain the user harm, rate severity, and fix all 3s and 4s before declaring done.

## Concrete specs & numbers
- **Response-time thresholds (Nielsen):** **0.1 s / 100 ms** = feels instantaneous (direct-manipulation); **1.0 s** = noticeable delay but thought flow uninterrupted (no extra feedback usually needed); **10 s** = limit of held attention → beyond it, show a **percent-done** indicator **and** an interrupt/cancel.
- **Percent-done indicators:** required for operations **>~10 s** (NNGroup rule of thumb); animate to avoid a "frozen" look.
- **Aesthetic first impression:** visual appeal is judged within **50 ms** (Lindgaard et al., 2006, *Behaviour & IT* — "Attention web designers: You have 50 milliseconds to make a good first impression!"). Visual hierarchy and aesthetics are evaluated before any copy is read.
- **Memory limits:** Miller (1956) proposed 7±2 chunks; Cowan (2001) revised to ~4 chunks of meaningful items in working memory. The working design rule is: keep distinct choices/items in a single view to ≤5–7; more than that forces chunking strategies or external memory aids. (H6, H8 cognitive-load argument.)
- **Heuristic eval coverage:** **1 evaluator ≈ 35%** of issues; **3–5 evaluators ≈ 75%**.
- **Optimal team:** **3–5 evaluators**, working independently.
- **Severity reliability:** averaging ratings across multiple evaluators significantly reduces individual-evaluator noise; NNGroup recommends ≥3 evaluators for this reason. The specific "within 0.5 at 95%" precision claim does not appear in NNGroup's published methodology — use mean severity as a directional guide, not a precise measurement.
- **Timebox:** **1–2 hours** per evaluator per session; full study **a few days to ~2 weeks**.
- **WCAG AA text contrast:** **4.5:1** normal text, **3:1** large text (used as the H9 visual-prominence floor; see [Accessibility](./accessibility-wcag.md)).
- **Snackbar dismiss window:** typically **4–10 s**. Gmail "Undo Send" is user-configurable at **5, 10, 20, or 30 s** — size the undo window to the cost/irreversibility of the action, not to a single default.
- **Origin:** heuristics derived from factor analysis of **249** usability problems (1994); created 1990, language refined 2020.

## Tools & libraries
- **Severity/issue tracking:** any spreadsheet or issue tracker with columns *screen · heuristic · severity · fix*. Keep it lightweight.
- **Loading states:** skeleton components (most UI kits ship them) for content pages; native `<progress>`/`role="progressbar"` for determinate ops. See [Interaction Design & UI States](./interaction-design-and-states.md).
- **Inline validation:** prefer native HTML constraints (`required`, `type`, `pattern`, `min`/`max`) + the Constraint Validation API before reaching for a library; libraries only when conditional/cross-field logic warrants it. See [Forms & Input UX](./forms-and-input-ux.md).
- **Command palette / shortcuts (H7):** add a `⌘K` palette and a discoverable `?` shortcut sheet; do not invent non-standard chords.
- **Not a tool, a discipline:** heuristic evaluation needs evaluators, not software — don't outsource judgment to an automated "audit" score.

## Do / Don't
**Do**
- Acknowledge every user action with feedback inside the 100 ms / 1 s / 10 s budget.
- Provide a labeled exit (Undo/Cancel/Back/Discard) for every state.
- Match platform & industry conventions before inventing (Jakob's Law).
- Prevent errors with constraints, smart defaults, and inline validation.
- Convert recall to recognition: autocomplete, history, breadcrumbs, prefilled data.
- Layer experts' accelerators on top of discoverable novice paths.
- Cut irrelevant content; use progressive disclosure to manage density.
- Write errors as plain language + exact problem + concrete fix, in bold red.
- Run a 3–5 person independent heuristic eval; block release on any severity-4.

**Don't**
- Don't disable or hijack the browser Back button, or break forms on Back.
- Don't show spinners for >10 s ops — use a percent-done bar + cancel.
- Don't put error codes, jargon, or stack traces in user-facing copy.
- Don't tell users to "try again" unless a retry can actually succeed.
- Don't confuse minimalism with sparseness — sparse pages can fail H8.
- Don't override entrenched form conventions (US-state dropdown) without strong cause.
- Don't let evaluators see each other's findings before consolidation.
- Don't treat heuristic evaluation as a replacement for real user testing.

## Common pitfalls & how to avoid them
- **Spinner-for-everything.** A spinner says "working" but not "how long." Use skeletons for content and determinate bars (+cancel) for long ops.
- **"Minimalist = flat/empty" misread.** Removing needed info to look clean (HelloBar) lowers usability. Optimize signal-to-noise, keep what serves the task.
- **Confirm-dialog fatigue.** Over-confirming trains users to click "Yes" blindly. Prefer Undo; reserve hard confirms for irreversible/high-cost actions.
- **Single-step Undo** in editors where users batch actions → keep a full undo stack.
- **End-of-form validation only.** Users learn requirements after failing. Validate inline; state rules up front.
- **Hidden exits.** YouTube Shorts overlay removal / TikTok accidental overlays — exits must be visible and discoverable, not buried in gestures.
- **Internal jargon leaks** ("entity," "record," error codes) — translate to user language.
- **Reinventing conventions** to look novel — costs relearning (Jakob's Law). Inconsistency is a tax.
- **Pre-emptive disabled submit.** Graying the submit button before interaction prevents users from discovering requirements. Keep it enabled; validate on blur/first-submit and show inline errors.
- **Personal-preference findings.** "I don't like this" is not a heuristic violation. Every finding must cite a heuristic and explain the concrete user harm. Vague findings get deprioritized or ignored, wasting the evaluation.
- **Gaming the self-review.** Don't rubber-stamp a heuristic pass — adversarially hunt for violations per the [project review loop](../05-process-and-systems/wireframing-prototyping-handoff-qa.md).

## What real users complain about
Synthesized from practitioner/user forum threads (r/UXDesign, r/userexperience):
- **YouTube Shorts:** removing a text overlay requires opening comments, dragging the video pane nearly all the way down, and holding it — an undiscoverable, recall-heavy exit (H3 + H6).
- **TikTok:** swiping up for the next video sometimes triggers "Shop" or "LIVE" overlays instead — accidental state change with no clean exit (H3).
- **Discord:** "used to have one of the best UX ever … now completely bloated" — feature accretion eroded the minimalist signal-to-noise (H8).
- **Amazon Black Friday banner:** low color contrast flagged as failing WCAG AA's 4.5:1 normal-text minimum — readability/accessibility complaint (H9 visual prominence + accessibility).
- **Generic "Something went wrong" errors:** no cause, no action — users report feeling stranded (H9). "Try again" that can't succeed is worse.
- **Forms that break on Back:** lost data and timeout messages after hitting the browser Back button (H3).
- **Facebook/Meta Danish localization:** "Do not like" abbreviated to "Do … like," inverting meaning (H2 + H4).

## Senior-level checklist
- [ ] Every action gets feedback within the 100 ms / 1 s / 10 s budget; ops >10 s show percent-done + cancel.
- [ ] No spinner stands in for a long determinate operation; skeletons used for content loads.
- [ ] All UI copy is user-language — zero error codes, internal terms, or stack traces.
- [ ] Every reversible action offers Undo; every destructive one is guarded (confirm or type-to-confirm).
- [ ] Browser Back works everywhere; no data loss on Back; no hijacked history.
- [ ] Exits are labeled ("Cancel/Back/Discard/Undo"), visible without scrolling, no forced dialogue.
- [ ] Layout/labels/icons match platform & industry convention (Jakob's Law); internal terms/styles are consistent via tokens.
- [ ] Forms: inline validation, requirements stated up front, smart defaults, native constraints.
- [ ] Recall→recognition everywhere feasible: autocomplete, history, breadcrumbs, prefilled/known data, visible options.
- [ ] Experts served: keyboard shortcuts + accelerators discoverable (`?`, menu hints), key views customizable; modals trap focus; Tab order follows visual reading order; Escape closes overlays.
- [ ] Primary views carry only relevant info; progressive disclosure for the rest; signal-to-noise audited.
- [ ] Error messages: plain language + exact problem + concrete fix, bold red, inline, data preserved; "Try again" verified to work.
- [ ] Help is contextual (tooltips, inline hints, "?"), searchable, task-focused, step-by-step — not a reference dump; empty states guide next action; error messages link to relevant help.
- [ ] Heuristic eval run with 3–5 independent evaluators; issues rated 0–4; all 3s/4s resolved; no severity-4 ships.
- [ ] Heuristic findings validated against (not substituted for) real usability testing.

## Sources
- Nielsen, J. — "10 Usability Heuristics for User Interface Design," Nielsen Norman Group: https://www.nngroup.com/articles/ten-usability-heuristics/
- Nielsen, J. — "Response Times: The 3 Important Limits," NN/g: https://www.nngroup.com/articles/response-times-3-important-limits/
- Nielsen, J. — "How to Conduct a Heuristic Evaluation," NN/g: https://www.nngroup.com/articles/how-to-conduct-a-heuristic-evaluation/
- Nielsen, J. — "Severity Ratings for Usability Problems," NN/g: https://www.nngroup.com/articles/how-to-rate-the-severity-of-usability-problems/
- "Jakob's Law" — Laws of UX: https://lawsofux.com/jakobs-law/
- WCAG 2.2 — Contrast (Minimum) 1.4.3: https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html
- Miller, G.A. (1956) — "The Magical Number Seven, Plus or Minus Two," *Psychological Review*, 63(2): 81–97.
- Cowan, N. (2001) — "The magical number 4 in short-term memory," *Behavioral and Brain Sciences*, 24(1): 87–114.
- Lindgaard, G. et al. (2006) — "Attention web designers: You have 50 milliseconds to make a good first impression!" *Behaviour & Information Technology*, 25(2): 115–126.

# The Laws of UX & Design Psychology

> The reusable cognitive and perceptual rules that govern how humans read, decide, remember, and act inside an interface. Apply these to make layouts faster to scan, choices faster to make, and flows that feel effortless — the difference between a UI that merely works and one that converts and retains at a senior level.

**Apply when:** any screen, flow, navigation, CTA, form, or pricing decision — i.e. always · **Related:** [Nielsen's Heuristics](./nielsen-heuristics.md) · [Cognitive Load & Progressive Disclosure](./cognitive-load-progressive-disclosure.md) · [Visual Hierarchy](../00-foundations/visual-hierarchy.md) · [Persuasion Psychology](../06-marketing-and-conversion/persuasion-psychology.md)

## TL;DR — rules at a glance
- **Hick's Law** — every visible choice taxes the decision. Cut, group, defer (progressive disclosure). Decision time ≈ `log2(n+1)`.
- **Fitts's Law** — big + close = fast + accurate. Primary CTA: large, high-contrast, in the natural pointer/thumb path. Min touch target 24×24 CSS px (WCAG AA), 44×44 (Apple/AAA), 48dp (Material).
- **Jakob's Law** — users expect your site to behave like every other site they use. Don't reinvent nav, links, or icons. Convention > cleverness.
- **Miller's Law** — working memory ≈ 7±2 *chunks* (1956); Cowan (2001) revised unrehearsed-item capacity to 4±1. Cap primary nav at 5–9. Chunk phone numbers, IDs, and long forms. Not a hard limit on well-grouped content.
- **Tesler's Law** — complexity is conserved; only its owner changes. Absorb it into the system (Apple Pay, autofill), don't dump it on the user.
- **Postel's Law** — be liberal in what you accept, conservative in what you emit. Accept any phone/date/case format; normalize on submit. Never reject valid input on formatting.
- **Peak-End Rule** — people judge a flow by its emotional peak and its ending, not the average. Engineer a strong end-state; kill the single worst friction point.
- **Aesthetic-Usability Effect** — attractive UIs are *perceived* as more usable and buy ~1–2s of delay tolerance — but cannot rescue a broken core flow.
- **Doherty Threshold** — keep system + human response under **400ms** to hold flow. Above that the brain switches to "wait mode."
- **Serial Position Effect** — first (primacy) and last (recency) items are recalled best; the middle is forgotten. Put key items at the ends.
- **Von Restorff (Isolation)** — the one thing that looks different is remembered. Have exactly **one** visually dominant CTA per view. Two "highlights" = zero.
- **Zeigarnik Effect** — open loops nag. Progress bars, saved drafts, "continue where you left off" pull users back. One meaningful open loop per session.
- **Goal-Gradient Effect** — motivation rises near the finish. Show "step 3 of 4," accelerate progress, and use endowed progress (pre-filled head start).
- **Law of Proximity** — nearness implies relatedness. Group fields, glue labels to inputs, put errors inline under their field, separate groups with whitespace.

---

## Hick's Law — fewer choices, faster decisions
Decision time grows logarithmically with the number of options: `RT = a + b · log2(n + 1)`.

**UI implications**
- **Progressive disclosure**: show only what's needed at the current step; reveal advanced options on demand (`<details>`, "More options", a second screen).
- **Demote rarely-used items** out of primary navigation into "More," a footer, or settings.
- **Sequence, don't dump**: split a 30-field config into a wizard. Each screen with 3 choices beats one screen with 30.
- **Curate, don't catalog**: Netflix's regional "Top 10" pre-filters a vast library into a short, decidable set.

**Watch out:** Hick's assumes roughly equal-weight, unfamiliar options. A long but *scannable, well-categorized* list (e.g. an alphabetized country dropdown) is cheap because users don't linearly evaluate every item — chunking and search defeat the penalty. Don't use Hick's to justify hiding important options behind extra clicks. See [Information Architecture & Navigation](./information-architecture-navigation.md).

## Fitts's Law — target size & distance
Acquisition time `T = a + b · log2(2D/W + 1)` — larger width `W` and shorter distance `D` both lower time and error rate.

**UI implications**
- Make the **primary CTA** the biggest interactive element and place it where the pointer/thumb already is.
- **Screen edges and corners are infinitely tall** (pinned by the cursor) — fixed bottom bars, top corners, and full-bleed targets are fast. macOS menu bar lives at the top edge for exactly this reason.
- **Mobile thumb zone**: primary actions belong in the bottom ~40% of the screen. Top-corner actions on a 6"+ phone require a grip change.
- **Whole rows/cards are targets**, not just the text inside. Make the entire list item clickable, not a 60px link.
- **Spacing matters as much as size**: adjacent small targets cause mis-taps. Material wants 8dp between targets; WCAG 2.5.8 satisfies via a 24px-diameter dead-zone per target.

```css
/* Accessible, thumb-friendly primary action */
.btn-primary {
  min-block-size: 48px;      /* Material 48dp floor */
  min-inline-size: 48px;
  padding-inline: 24px;
  font-size: 1rem;
  /* expand hit area without changing visual size */
}
.icon-btn {
  position: relative;
  inline-size: 24px; block-size: 24px;   /* visual */
}
.icon-btn::before {           /* invisible 44px hit target */
  content: ""; position: absolute; inset: -10px;
}
```

See [Touch Ergonomics & Cross-Device Design](../02-responsive-adaptive/touch-ergonomics-cross-device.md).

## Jakob's Law — match existing mental models
> "Users spend most of their time on *other* sites. They prefer your site to work the same way as all the other sites they already know."

**UI implications** — keep these conventional unless you have data proving otherwise:
- Logo top-left → links to home.
- Primary nav across the top (desktop) / hamburger or bottom tab bar (mobile).
- Cart icon top-right; search top-center/right.
- Links are visibly distinct (color and/or underline) and stay that way; visited links change state.
- Breadcrumbs for deep hierarchies; "Back" goes back.
- Form submit at the bottom; primary button on the right (or full-width on mobile).

**Cost of violation:** every reinvented pattern forces relearning — a tax users rarely pay; they bounce instead. Innovate on *content, brand, and delight*, not on where the cart lives. When you must change a pattern, ease the transition (let users opt into the old way for a while).

## Miller's Law — 7±2 chunks
Working memory holds ~7±2 *chunks* (Miller, 1956). The lever is **chunking**: meaningful grouping raises effective capacity.

**Critical update:** Cowan (2001) revised the actual capacity for *unrelated* items to **4±1 chunks** — the "7±2" holds only when subjects can rehearse or sub-group. For novel UI items users haven't memorized, treat 4 as a practical ceiling per working-memory load, not 7.

**UI implications**
- Primary navigation: **5–9 top-level items**. More → group under categories or a mega-menu.
- **Chunk strings**: `(123) 456-7890` not `1234567890`; `4242 4242 4242 4242` not a 16-digit run; format on input.
- **Multi-step forms** instead of one 25-field wall.
- **Group dense tables** into labeled sections; don't ask users to hold 12 column values at once.

**Honest caveat:** Miller studied single-dimension stimuli (tones, dots), not whole UIs. "7" is a rough ceiling for items *with rehearsal* — unrehearsed, unrelated UI items follow the Cowan 4±1 bound. Don't weaponize either number to cap menus at 7 when 10 well-grouped items are clearer. See [Cognitive Load](./cognitive-load-progressive-disclosure.md).

## Tesler's Law — conservation of complexity
> "For any system there is a certain amount of complexity that cannot be reduced." — Larry Tesler

The only question is **who absorbs it: the user or the system.** Tesler's framing: *"If a million users each waste a minute a day dealing with complexity that an engineer could have eliminated in a week by making the software a little more complex, you are penalizing the user."*

**UI implications**
- **Move complexity into the system.** Apple Pay collapses 7+ checkout fields into one biometric tap. Amazon Go removes checkout entirely. Email clients pre-fill the sender. Autofill, smart defaults, geolocation-based currency, format auto-detection — all absorb complexity for the user.
- **Smart defaults** that handle the 80% case, with overrides for the rest.
- **Counter-failure:** over-simplifying until you delete *necessary* controls — hiding essential settings, removing the "advanced" path power users need. The complexity didn't vanish; it became impossible. Conserve it where it belongs, don't pretend it's gone.

## Postel's Law — robustness principle
> "Be conservative in what you do, be liberal in what you accept from others." — Jon Postel (TCP, 1980)

**UI implications**
- **Accept messy input, normalize on submit.** Phone as `123-456-7890`, `(123) 456-7890`, or `1234567890` → store as E.164. Dates in multiple formats → parse to ISO. Strip stray spaces from emails and card numbers.
- **Case-insensitive** usernames and emails.
- **Forgive typos** in search (fuzzy match / "did you mean").
- **Never reject valid input over formatting** — that's the system being lazy at the user's expense (a Tesler violation too).
- **Accessibility corollary:** accept *any input modality* — keyboard, mouse, touch, voice, switch, screen reader. Don't build mouse-only interactions.

```js
// Accept liberally, normalize internally
const normalizePhone = (raw) => raw.replace(/\D/g, "");      // keep digits
const ok = normalizePhone("(123) 456-7890").length === 10;   // true
// Validate the *meaning*, not the user's punctuation.
```

See [Forms & Input UX](./forms-and-input-ux.md).

## Peak-End Rule — design the spike and the finish
People remember an experience by its most intense moment and its end, not the average (Kahneman, Fredrickson, Schreiber & Redelmeier, 1993). In the cold-water study, adding 30 extra seconds of slightly-less-cold water made the trial *objectively worse and longer* — yet the majority preferred it because it **ended better**.

**UI implications**
- **Engineer a strong end-state**: order confirmation, success animation, a genuinely useful thank-you page. The goal is *relief or delight*, not an upsell — finishing with a cross-sell or a survey overwrites a positive session with extraction.
- **Find and fix the single worst friction point** in any flow — that spike is what gets remembered. One brutal error or dead-end can sink an otherwise smooth experience.
- **Don't bolt a negative moment onto a positive ending.** An exit-intent popup or forced-rating prompt as the final interaction overwrites a good session with annoyance. Protect the last screen.

## Aesthetic-Usability Effect — pretty reads as usable
Attractive interfaces are *perceived* as more usable than plain ones — even when measured usability is equal (Kurosu & Kashimura, 1995; 252 participants, 26 ATM UI variants at Hitachi). Aesthetics correlated with *perceived* usability more strongly than with *actual* usability.

**UI implications**
- Visual polish earns goodwill and **tolerance for ~1–2s of delay or minor friction**.
- **Ceiling:** it forgives *minor* issues, never major ones. A beautiful checkout that loses the cart still loses the sale.
- **Research trap:** in usability tests, users with a pretty prototype give warm *verbal* feedback while silently failing tasks. **Measure behavior (task success, time, errors), not compliments.** See [User Research & Testing](./user-research-and-testing.md).
- Invest in [Visual Design Principles](../00-foundations/visual-design-principles.md), [Typography](../00-foundations/typography-systems.md), and [Color](../00-foundations/color-theory-and-systems.md) — it's not vanity, it measurably shifts perception.

## Doherty Threshold — stay under 400ms
Productivity *soars* when both human and computer respond in **under 400ms** (Doherty & Thadani, IBM, 1982). Below 400ms the interaction feels continuous and the user stays in "action mode"; above it the brain flips to "wait mode," flow breaks, and cognitive load rises. The prior IBM target of 2000ms was identified as a productivity destroyer — the 1982 paper attributed significant throughput gains to sub-400ms systems, though the exact percentage varies by task type and was not a universal controlled study.

**UI implications**
- **Budget every interaction to <400ms.** Google's INP "good" threshold of **≤200ms** (75th percentile) auto-satisfies Doherty — make that your real-world target. Note: INP replaced FID as a Core Web Vital in March 2024. See [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md).
- When you can't be instant, **simulate progress**: optimistic UI, skeleton screens, instant local echo before the server confirms.
- **Nielsen's response tiers:** ≤0.1s = instantaneous (no feedback needed); ≤1.0s = noticeable but flow holds (no spinner needed); ≥10s = attention lost, **progress indicator mandatory**.
  - Corollary: **do not show a spinner for sub-1s operations** — it makes the action *feel slower* by inserting a waiting signal. A skeleton screen at 200ms is better than a blank screen but never needs a spinner.
- **Stakes:** A 2017 Google/Akamai study found a 1s mobile load delay correlated with up to **20%** conversion drop on median retail sites. Treat as order-of-magnitude guidance, not a universal law — impact varies sharply by site type and baseline speed. Speed *is* an accessibility issue: slow interactions disproportionately affect users with cognitive load constraints and low-bandwidth connections.

## Serial Position Effect — ends are remembered, middles aren't
Items at the start (primacy) and end (recency) of a sequence are recalled best (Ebbinghaus). Caveat: recency is **fragile** — after a 30s distractor task the recency effect vanished while primacy held (Glanzer & Cunitz, 1966).

**UI implications**
- **Nav:** put the most important items far-left and far-right; bury secondary ones in the middle.
- **Landing pages:** place key CTAs near the top *and* at the bottom; don't rely on a single mid-page button.
- **Pricing pages:** feature the plan you want chosen **first or last**, not buried in the middle. (Note: a deliberately *centered* "recommended" plan can still win via Von Restorff isolation — these laws combine.)
- Because recency decays under distraction, **reinforce the most critical item at the start** (primacy survives) rather than trusting the last position alone.

## Von Restorff Effect — the isolated element wins
The item that differs from its surroundings is remembered (Von Restorff, 1933).

**UI implications**
- **One dominant CTA per view.** A single high-contrast button on a neutral page beats a page of "highlighted" elements. The mechanism is **contrast against context**: a red button on a red page is invisible; the same button on a grey page dominates. Color is not the variable — isolation is.
- High figure/ground separation (white or light text on a saturated solid CTA) is the dominant pattern on high-converting landing pages because it maximizes the isolating contrast.
- Canonical isolation: Amazon's orange "Buy Now," Netflix's white play button, App Store's blue "GET" — each pops from a neutral or dark field.
- **Fatal pitfall:** apply the effect to 2+ elements and it **self-destructs**. Multiple badges, multiple "featured" cards, multiple color-coded alerts → nothing stands out. Scarcity of emphasis *is* the emphasis.

## Zeigarnik Effect — open loops pull people back
Uncompleted tasks occupy memory more than completed ones (Zeigarnik, 1927 — waiters forgot orders the moment they were served).

**UI implications**
- **Progress meters create productive tension:** LinkedIn's profile completeness, "Your profile is 60% complete," onboarding checklists.
- **Persist open loops:** saved drafts, saved carts, "continue where you left off," Duolingo streaks, GitHub contribution graphs.
- **Reduce re-entry cost** by re-surfacing the unfinished thing exactly where the user left it.
- **Ethical boundary:** *one meaningful* open loop per session motivates; a dozen simultaneous nags create anxiety, fatigue, and churn. This crosses into dark-pattern territory fast — see [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md).

## Goal-Gradient Effect — effort accelerates near the finish
Motivation rises as perceived distance to the goal shrinks (Hull, 1934).

**UI implications**
- **Show position toward the goal:** "Step 3 of 4," "8 of 10 lessons done," "3km done, 2km to go." Render the *whole* step path, not just the current step, so users see proximity to the end.
- **Accelerating progress bars** near completion add a final push.
- **Endowed progress:** give a head start. A loyalty card pre-stamped 2/10 fills faster than an empty 8/10 card despite identical remaining effort (Nunes & Drèze, 2006). In onboarding, mark "Create account ✓" as already done so the checklist starts at 1/5, not 0/4.
- **Checkout:** a visible 4-step indicator with the user on step 3 outperforms a context-free single step.

## Law of Proximity (Gestalt) — nearness = relatedness
Elements placed close together are perceived as a group; whitespace separates groups.

**UI implications**
- **Group related fields:** first + last name in one cluster; street/city/zip in another. Spacing between groups > spacing within a group.
- **Glue labels to inputs.** A label 40px above its field, with another field 8px below, reads as belonging to the *wrong* input. Label-to-input gap must be clearly smaller than group-to-group gap.
- **Errors are proximate to their cause:** inline, field-level messages directly below the offending input are read and fixed faster than a lone top-of-form summary (provide both for screen-reader users, but never the summary alone).
- **Whitespace is structure**, not decoration — it's the cheapest grouping tool you have.

```css
/* Proximity encodes structure via a spacing scale */
.field      { margin-block-end: 8px;  }   /* label↔input: tight */
.field-group{ margin-block-end: 32px; }   /* group↔group: loose */
.field label{ display: block; margin-block-end: 4px; }  /* label hugs input */
.field .error{ margin-block-start: 4px; color: var(--danger); } /* error hugs field */
```

See [Visual Design Principles & Gestalt](../00-foundations/visual-design-principles.md) and [Layout, Grids & Spacing](../00-foundations/layout-grids-and-spacing.md).

---

## Concrete specs & numbers

**Touch targets (Fitts's)**
- WCAG 2.5.8 (AA, legally enforceable floor): **24×24 CSS px** minimum.
- WCAG 2.5.5 (AAA): **44×44 CSS px**.
- Apple HIG: **44×44 pt** (~59 CSS px on standard density).
- Google Material: **48dp** tap target; **8dp** minimum spacing between targets.
- WCAG 2.5.8 spacing alternative: 24px-diameter dead-zone circle per target.
- MIT Touch Lab: average index-finger pad ≈ **45–57px** wide; thumb ≈ **75px** wide.

**Response timing (Doherty / Nielsen)**
- Doherty threshold: **<400ms** for continuous-feel interaction (old IBM standard was 2000ms).
- Nielsen 0.1s — instantaneous, no feedback needed.
- Nielsen 1.0s — flow maintained, no spinner needed.
- Nielsen 10s — attention limit; progress indicator mandatory above this.
- Google INP "good": **≤200ms @ 75th percentile** (auto-satisfies Doherty). INP replaced FID as Core Web Vital March 2024.
- 1s mobile load delay: up to **~20%** conversion drop (Google/Akamai, 2017–2018 retail dataset). Treat as order-of-magnitude, not universal law.

**Memory & choice**
- Miller: **7±2** chunks in working memory (1956); Cowan (2001) revised unrelated-item capacity to **4±1** — use 4 as the practical limit for novel UI items.
- Hick: `RT = a + b · log2(n+1)`.
- Serial position recency disappears after a **30s** distractor (Glanzer & Cunitz, 1966).

**Contrast (supports Von Restorff / readable CTAs)**
- WCAG AA: **4.5:1** normal text, **3:1** large text (≥18pt, or ≥14pt bold) and UI components.
- WCAG AAA: **7:1** normal text.

**Effect-size data points**
- Peak-End: majority preferred the longer, objectively-worse cold-water trial because it ended better (Kahneman et al., 1993).
- Aesthetic-Usability study: 252 participants, 26 ATM variants (Kurosu & Kashimura, 1995).

## Law combinations — where they reinforce or conflict

Laws rarely fire alone. These combos appear constantly at the senior level:

| Context | Laws in play | Synthesis |
|---|---|---|
| Primary navigation | Hick's + Miller's + Serial Position | 5–9 items max; most important items leftmost and rightmost; group the rest under a category. |
| Primary CTA | Von Restorff + Fitts's + Jakob's | One isolating contrast button, 44–48px, in the thumb/pointer path — conventional placement unless A/B proves otherwise. |
| Onboarding checklist | Zeigarnik + Goal-Gradient | One open loop per session; pre-stamp 1–2 items as done (endowed progress) to invoke goal-gradient immediately. |
| Checkout flow | Tesler's + Postel's + Peak-End | Absorb complexity (autofill/Apple Pay); accept any format input; end on a relief screen, not an upsell. |
| Pricing page | Serial Position + Von Restorff | Feature the recommended plan first or last; isolate it visually. Do not put it center-second without strong isolation. |
| Form validation | Postel's + Proximity + Jakob's | Normalize input server-side; errors inline under their field; submit bottom-right (conventional). |
| Loading states | Doherty + Aesthetic-Usability | <400ms = no feedback; skeleton/optimistic UI preserves flow feel AND leverages the aesthetic tolerance budget. |
| Conflict: Hick's vs. Jakob's | Hick's says cut choices; Jakob's says keep familiar patterns | Don't hide options users expect (cart, search, nav). Hick's applies to *decision choices*, not to *site furniture*. |

## Do / Don't

**Do**
- Cap visible choices; defer the rest with progressive disclosure (Hick's).
- Size and place primary actions for the natural pointer/thumb path (Fitts's).
- Follow platform conventions for nav, links, icons, and layout (Jakob's).
- Accept any reasonable input format and normalize server-side (Postel's).
- Push complexity into the system via defaults, autofill, and automation (Tesler's).
- Engineer one strong ending and fix the single worst friction spike (Peak-End).
- Budget interactions to <400ms; show optimistic/skeleton UI when you can't be instant (Doherty).
- Give exactly one element visual dominance per view (Von Restorff).
- Show progress and a head start to pull users to completion (Zeigarnik + Goal-Gradient).
- Encode relationships with proximity and a consistent spacing scale (Proximity).

**Don't**
- Reinvent standard patterns without data proving the new one is better (Jakob's).
- Reject valid input on formatting nitpicks (Postel's).
- Highlight three "primary" CTAs — you've highlighted none (Von Restorff).
- Over-simplify away controls power users need (Tesler's failure mode).
- Trust verbal praise from a pretty prototype over measured behavior (Aesthetic-Usability trap).
- Bury the plan/CTA you want chosen in the middle of a list (Serial Position).
- Stack a dozen guilt-trip open loops to manufacture anxiety (Zeigarnik dark pattern).
- Cite Miller's "7" as a hard cap on well-grouped content.
- Place a label closer to the wrong field than its own (Proximity violation).

## Common pitfalls & how to avoid them
- **"Minimalism" that hides essential actions.** Over-applying Hick's/Tesler's buries the action users came for. Fix: hide *rare* options, never the primary task; keep the happy path one obvious click.
- **Multiple competing CTAs.** Every section has its own bright button → decision paralysis and zero standout. Fix: one primary per view; demote the rest to ghost/text buttons (Von Restorff + Hick's).
- **Format-strict validation.** Rejecting `(123) 456-7890` because you wanted `1234567890`. Fix: strip/normalize, validate meaning (Postel's).
- **Tiny, crowded touch targets.** 30px icons 4px apart → mis-taps and rage. Fix: 44–48px targets, 8dp+ spacing, expand hit area without enlarging visuals.
- **Spinner abuse.** Showing a spinner for a 150–300ms action makes it feel *slower* by inserting a waiting signal. Fix: no spinner under ~1s (the action is already done before the user consciously registers a delay); skeleton/optimistic UI for 1–10s; determinate progress above 10s.
- **Pretty-but-broken.** Polishing visuals while the core flow leaks. Fix: aesthetics buys ~1–2s tolerance, not a broken checkout — usability-test behavior first.
- **Label/input ambiguity.** Even spacing everywhere makes forms unreadable. Fix: tight label→input, loose group→group; let proximity carry structure.
- **Open-loop overload.** Five progress nags, three streak warnings, two "complete your profile" banners → fatigue and churn. Fix: one meaningful loop per session.
- **Mid-flow negative ending.** Exit popups or forced surveys as the last beat. Fix: protect the ending (Peak-End).

## What real users complain about (law in brackets)
- *"It won't accept my phone number with dashes."* [Postel] — users feel blamed for the system's rigidity; highest anger-per-incident of any form failure.
- *"I keep tapping the wrong button."* [Fitts's] — undersized/crowded targets; rage-taps double the server load and log as "re-submissions."
- *"It feels laggy even though my connection is fine."* [Doherty] — >400ms reads as broken; users double-tap and then wonder if they submitted twice.
- *"Where did they hide [feature]?"* [Tesler's / Hick's] — over-aggressive simplification deleted a necessary control; the complexity didn't vanish, it became unreachable.
- *"This menu has 20 options and I can't find anything."* [Hick's + Miller's] — flat, ungrouped navigation; IA problem at its root.
- *"Looked great — then it lost my cart."* [Aesthetic-Usability trap] — polished visuals raised expectations that a broken core flow then shattered; the betrayal is worse than a plain broken UI.
- *"It nags me constantly."* [Zeigarnik dark pattern] — anxiety stacking; users describe it as "manipulative" and uninstall.
- *"Where's the submit button?"* / *"Is this a link?"* [Jakob's] — unconventional placement and links that don't look like links; relearning cost converts to bounce.

## Senior-level checklist
- [ ] Every screen has **one** unambiguous primary action; secondary actions are visually subordinate.
- [ ] Primary CTAs meet **44–48px** targets, **4.5:1** text contrast, and sit in the thumb/pointer path.
- [ ] All adjacent interactive targets have ≥**8dp** spacing or ≥**24px** dead-zones.
- [ ] Primary navigation is **5–9 items**, grouped if more; nav order honors serial-position (key items at ends).
- [ ] Every interaction has measured/budgeted response **<400ms** (target INP ≤200ms p75); skeleton/optimistic UI elsewhere; progress for >10s.
- [ ] No form rejects valid input over formatting; phone/date/case/whitespace normalized server-side.
- [ ] Multi-step flows show **"step X of N"** and use endowed progress where honest.
- [ ] The flow's **worst friction point** is identified and mitigated; the **ending** is a deliberate positive state, not an upsell/popup.
- [ ] Standard patterns (logo→home, link styling, cart top-right, submit bottom) are intact unless A/B data justifies deviation.
- [ ] Form labels hug their inputs; errors render inline under the offending field; groups separated by whitespace.
- [ ] Complexity absorbed by the system (defaults, autofill, automation) without deleting controls power users need.
- [ ] At most **one** meaningful Zeigarnik open loop per session — no anxiety stacking.
- [ ] Usability validated on **behavior** (task success/time/errors), not on praise for the visuals.

## Sources
- WCAG 2.2 — Success Criterion 2.5.8 Target Size (Minimum) & 2.5.5 Target Size (Enhanced): https://www.w3.org/TR/WCAG22/
- Material Design — Accessibility / touch target sizing: https://m3.material.io/foundations/accessible-design/accessibility-basics
- Apple Human Interface Guidelines: https://developer.apple.com/design/human-interface-guidelines/
- Laws of UX (Jon Yablonski) — concise references for each law: https://lawsofux.com/
- Nielsen Norman Group — Response Times: The 3 Important Limits: https://www.nngroup.com/articles/response-times-3-important-limits/
- Google web.dev — Interaction to Next Paint (INP): https://web.dev/articles/inp
- Miller, G. A. (1956) "The Magical Number Seven, Plus or Minus Two," *Psychological Review*, 63(2), 81–97.
- Cowan, N. (2001) "The magical number 4 in short-term memory," *Behavioral and Brain Sciences*, 24(1), 87–114.
- Kahneman, D., Fredrickson, B. L., Schreiber, C. A., & Redelmeier, D. A. (1993) "When More Pain Is Preferred to Less," *Psychological Science*, 4(6), 401–405.
- Kurosu, M. & Kashimura, K. (1995) Apparent Usability vs. Inherent Usability, CHI '95 conference proceedings.
- Doherty, W. J. & Thadani, A. J. (1982) "The Economic Value of Rapid Response Time," IBM Systems Journal.
- Glanzer, M. & Cunitz, A. R. (1966) "Two storage mechanisms in free recall," *Journal of Verbal Learning and Verbal Behavior*, 5(4), 351–360.
- Nunes, J. C. & Drèze, X. (2006) "The Endowed Progress Effect," *Journal of Consumer Research*, 32(4), 504–512.
- Zeigarnik, B. (1927) "Das Behalten erledigter und unerledigter Handlungen," *Psychologische Forschung*, 9, 1–85.

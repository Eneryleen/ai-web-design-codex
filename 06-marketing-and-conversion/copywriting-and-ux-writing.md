# Copywriting & UX Writing

> Words are interface. This guide covers the copy that drives comprehension and conversion: headlines, value propositions, CTAs, and the microcopy on buttons, errors, empty states, tooltips, confirmations, and progress states. The senior-level outcome: every string on the page is scannable, benefit-led, action-oriented, and calibrated for how users actually read (they scan ~20–28% of words on a page — NNG).

**Apply when:** writing any user-facing string — headlines, buttons, form labels, errors, empty states, tooltips, hero VP, onboarding, success/confirmation, loading states. · **Related:** [Conversion Rate Optimization](./conversion-rate-optimization.md) · [Ethical Persuasion Psychology](./persuasion-psychology.md) · [Visual Hierarchy & Scanning Patterns](../00-foundations/visual-hierarchy.md) · [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md)

## TL;DR — rules at a glance
- **Clarity beats cleverness.** Literal, benefit-forward headlines consistently outperform clever wordplay in conversion tests. 80% read the headline, 20% read on — the headline must stand alone.
- **Users scan, don't read.** ~20–28% of words get read (NNG). Front-load every idea; first 2 words of each line/sentence carry the load.
- **Benefits, not features, on every CTA.** `Send Campaign` beat `Submit` +18% CTR (MailChimp internal test). `Submit` communicates zero value — never use it.
- **First-person CTA framing wins.** `Start my free 30-day trial` beat `Start your free trial` +90% CTR (ContentVerve, 2013 — directional; treat as hypothesis, replicate). First-person framing consistently lifts across tests.
- **Verb + object + value signal** for every action label. `Download your free template` > `Learn more` > `Submit`. 2–5 words.
- **One primary CTA per screen.** A second competing CTA dilutes the primary action rate.
- **Value proposition lands in 10 seconds** or users leave (NNG: users begin exiting at 10–20s without a clear VP). VP goes top-left / first H1 / hero — never the second section.
- **Error messages: what happened → why → what to do next.** Never bare `Error` or `Something went wrong` with no path.
- **Never blame the user.** No `invalid`, `illegal`, `incorrect`. Use neutral or system-ownership phrasing.
- **Validate on blur, never on keystroke.** Mid-typing errors feel punitive and increase abandonment.
- **Preserve user input on error.** Never wipe form fields — users correct, don't retype.
- **Kill jargon.** `OTP code` → `6-digit code`; `KYC` → plain description; `Authenticate` → `Sign in`. Grandparent test.
- **One idea per UI element.** No stacked constraints in a tooltip; no two questions in one dialog.
- **Empty states guide, not announce.** Drive a next action (Pinterest: "Create your first board…" + CTA), not "No content yet."
- **The F-pattern is a failure state.** Give visual anchors (bold lead-ins, subheads, bullets) to pull users into layer-cake scanning.
- **"Are you sure?" is not a confirmation.** Destructive-action dialogs name the irreversible consequence and put it in the button label.
- **Placeholder text is not a label.** It disappears on input, fails accessibility, and cannot be scanned.

---

## 1. Clarity over cleverness — the core operating principle

Users do not read prose; they hunt for the answer to "is this for me, and what do I do next?" Design copy for that hunt.

- **80% read the headline; ~20% read past it.** The headline must stand alone and deliver the value claim. If the headline only makes sense after the body, it has failed.
- **Plain language wins in conversion tests.** Literal, benefit-forward headlines outperform clever wordplay in the large majority of controlled tests on lead-gen and opt-in pages. Wit costs comprehension; comprehension converts. (No single universal percentage is reliable — test your own; the direction is consistent.)
- **Write for how users *think*, not how they (or you) *talk*.** Mirror the user's internal dialogue at the decision moment. The Noun Project surfaced the literal user question "What's the difference?" as a tooltip trigger, then answered it inline — so users never left to Google it.
- **Read every line aloud.** If you pause, re-read, or feel the need to re-parse — the copy stumbles. Stumbling signals: passive constructions, noun-stacked phrases, implied subjects, or clauses that require prior context to parse.
- **Don't dilute a strong argument with weak ones.** Adding a mediocre supporting point lowers the persuasive force of the lead claim. One sharp benefit beats three soft ones.

## 2. Headline formulas

Lead with outcome. Pick the formula that fits the page intent; combine sparingly.

| Formula | Pattern | Best for |
|---|---|---|
| **Four U's** | Useful · Urgent · Unique · Ultra-specific (hit ≥3 of 4) | Any high-stakes headline; a scoring rubric |
| **How-To** | `How to [outcome] without [pain]` | Educational / problem-aware audiences |
| **Number + Method** | `7 Proven Strategies to [goal] in [timeframe]` | SEO, listicle, depth signaling |
| **PAS** | Problem → Agitate → Solution | Landing pages for a specific frustration |
| **Benefit VP** | `The easiest way to [outcome]` | Product hero; avoids feature-listing |

- **Ultra-specificity adds credibility.** `Devices up to 49% off` (Amazon banner) reads as a real, verifiable claim; `Great deals on devices` does not. Specificity signals that a claim can be checked — vagueness signals it cannot.
- **Tutorial framing vs question framing.** Tutorial-style headlines ("How to X") tend to outperform question headlines ("Want X?") in smart-bar and lead-gen tests. Run your own test; the specific lift varies widely by audience and offer.
- **The hero H1 is the value proposition.** Don't waste it on a brand slogan if the VP isn't yet established. Slogans (evian: "water the way nature intended") work once brand equity exists, not as the first thing a cold visitor reads.

```html
<!-- Hero VP: benefit-led, ultra-specific, scannable -->
<h1>Invoice clients in 30 seconds — no spreadsheets, no chasing.</h1>
<p>Send a branded invoice, get paid by card, and see who's overdue at a glance.</p>
<a class="cta-primary" href="/signup">Start my free 30-day trial</a>
```

## 3. Value-proposition messaging

- **The VP must answer "why this, why now, why you" within 10 seconds.** Pages without a clear VP lose users in 10–20 seconds (NNG); with one, attention and engagement improve substantially.
- **Placement:** top-left quadrant of the hero, or the first H1. Never bury the VP in a second scroll section — most users never reach it (NNG: ~20% of total attention goes below the fold even when users do scroll).
- **Structure:** [headline = primary benefit] + [subhead = how / for whom] + [primary CTA]. Optionally a 3-bullet "what you get" beneath — but only if each bullet is a benefit, not a feature.
- **Lead with the outcome the user wants, not the mechanism you built.** `The easiest way to track expenses` > `AI-powered receipt-parsing engine`.
- **VP ≠ tagline.** A tagline works after trust is established. A VP works on first contact with a cold visitor who has never heard of you.

## 4. Benefits vs features

Translate every feature into the user's payoff. The test: append "…which means you can ___" to each feature; the completion is the benefit, and the benefit is what ships in the copy.

| Feature (what it is) | Benefit (what they get) |
|---|---|
| 256-bit encryption | Your data stays private — even from us |
| Real-time sync | Pick up on any device, mid-sentence |
| Optional "Company" field | *(removed)* — Expedia reported deleting this one field added ~$12M/yr revenue by reducing form friction |
| `Submit` button | `Send Campaign` — +18% CTR (MailChimp) |

- **Button copy is the highest-leverage benefit surface.** It is the last word before the conversion. `Send invoice` beats `Submit`; `Get my report` beats `Get your report`.
- **Friction copy compounds.** Expedia's removed optional field shows that *fewer* words/fields can itself be the benefit. Cut before you polish.
- **Don't feature-list in the hero.** Features answer "what it does." Benefits answer "what I get." Users act on benefits.

## 5. CTA wording

**Rule:** `verb + object + value signal`, 2–5 words. Under 2 = too vague; over 5 = loses imperative force.

- **Never `Submit`.** It describes the system's action, not the user's gain. Replace with the outcome: `Create my account`, `Get the checklist`, `Continue to payment`.
- **First-person framing.** `Get my report` > `Get your report`. `Start my free 30-day trial` lifted CTR +90% vs second-person in one widely-cited test (ContentVerve, 2013 — directional; treat as a strong hypothesis for your own tests). The mechanism is specificity + ownership, not pronouns per se.
- **Add specificity + reassurance near the CTA.** "30-day", "free", "no card required". Inline risk-reducers at the friction point consistently improve completion; quantified lifts vary by product.
- **One primary CTA per screen.** Secondary actions get visually subordinate treatment (link/ghost button). A competing primary CTA causes decision paralysis and lowers the main action rate.
- **Contextualize CTAs.** `Get Ready for School` (Barnes & Noble, seasonal) beats generic `Shop now`. Context-matched CTAs reduce cognitive distance between the user's intent and the click.
- **CTA placement vs scanning:** put the primary CTA in the hero's top-left or immediately below the VP. CTAs stranded in the right column or below paragraph 2 are at high risk of being skipped entirely during F-pattern scanning.

```html
<!-- Good: verb + object + value, first-person, reassurance inline -->
<button>Get my free pricing report</button>
<small>No credit card. Unsubscribe anytime.</small>

<!-- Bad: zero value, second-person, noun-only -->
<button>Submit</button>
```

## 6. Microcopy — inform, influence, interact

Microcopy has three jobs: **inform**, **influence**, **interact**. Each element should pursue **one** primary goal — never all three at once.

- **Self-explanatory titles and labels.** Users apply the serial position effect: they read the first and last elements on screen and skim mid-page body. A label must work without its surrounding paragraph. `Paint & Supplies` (Sherwin-Williams nav) needs no interpretation; `Solutions` does.
- **Verb + object for every action label.** `Download report`, not `Report`. `Continue to payment`, not `Next`. Noun-only buttons are a top microcopy failure mode.
- **Reassurance microcopy converts.** Place trust/risk-reducing copy at the friction point (email field, payment, signup), not in a footer.
- **Placeholder text is not a label.** Placeholder disappears on input: it cannot be scanned, fails users who switch context mid-form, and is inaccessible (WCAG 1.3.1). Always use a visible label element above the field.

### 6.1 Buttons
- Outcome-led, 2–5 words, verb-first. See §5.
- Use consistent capitalization across all buttons (Title Case or Sentence case — not mixed).

### 6.2 Errors
- See §7 — the highest-risk microcopy surface. Never reduce errors to a cross-reference; treat every error string as a product decision.

### 6.3 Empty states (a conversion surface, not an edge case)
- **Guide to the next action; don't announce emptiness.** "No content yet." wastes the moment.
  - Pinterest: "Welcome to Pinterest! Create your first board to save ideas you love" + topic suggestions + `Create board`.
  - Headspace: "Download your favorite sessions to listen offline." (actionable, benefit-forward, specific.)
- **Structure:** one line of context → the benefit of acting → a single clear CTA.
- **Differentiate first-visit empty from cleared/filtered empty.** First-visit = onboard and motivate. Post-filter empty = explain the filter and offer a way out (`Clear filters`). They are different user states and need different copy.

### 6.4 Tooltips & progressive disclosure
- **One idea per tooltip.** Never stack two constraints. Never ask two things in one dialog.
- **Surface the user's real question as the trigger,** then answer inline (Noun Project: "What's the difference?").
- **Progressive disclosure:** essentials upfront, depth behind `Learn more` / tooltip — **except on sensitive screens** (payments, legal, health, identity), where all required info must be statically visible (see §9).

### 6.5 Success states and confirmations
- **Success copy closes the loop and opens the next one.** "Payment sent" leaves the user wondering what happens next. "Payment sent — your client gets a receipt in minutes" closes the action and sets expectations.
- **Confirmation ≠ celebration.** Match the tone to the stakes. Payment confirmed: calm + informative. Account created: warm + forward-looking. File deleted: neutral + undoable if possible.
- **Don't reuse the action verb.** `Saved!` after `Save` is fine. Restating the entire previous label as a success toast is dead air.

### 6.6 Loading and progress states
- **Say what is happening, not that something is happening.** `Loading…` is noise. `Checking your availability…`, `Generating your report…` keeps users oriented and reduces perceived wait time.
- **For waits over ~3 seconds**, give a progress indicator + a concrete status string. At ~10s, offer an explanation or escape hatch.
- **Do not animate loading copy** in a way that disrupts screen readers. Use `aria-live="polite"` for progress updates; `aria-live="assertive"` only for blocking errors.

### 6.7 Confirmation dialogs for destructive actions
- **"Are you sure?" is not a confirmation.** It is a vague delay. Users click through it without reading — especially repeat users.
- **Name the irreversible consequence in the dialog title.** `Delete project "Acme Q4"?` not `Delete item?`
- **Put the consequence in the button label.** `Delete project` (primary, destructive styling) + `Keep project` (secondary). Never `OK` / `Cancel` for irreversible actions.
- **State what cannot be undone explicitly.** "This cannot be undone." or "All 47 files will be permanently removed."

```html
<!-- Good: names what is destroyed, consequence in button, explicit irreversibility -->
<dialog>
  <h2>Delete project "Acme Q4"?</h2>
  <p>All 47 files will be permanently removed. This cannot be undone.</p>
  <button class="destructive">Delete project</button>
  <button class="secondary">Keep project</button>
</dialog>

<!-- Bad: vague, generic buttons, no consequence stated -->
<dialog>
  <p>Are you sure?</p>
  <button>OK</button>
  <button>Cancel</button>
</dialog>
```

## 7. Error-message craft

Errors are the highest-risk microcopy surface. Unclear error instructions are a leading cause of task abandonment and user errors — NNG research establishes this qualitatively; specific abandonment percentages vary widely by context and should not be cited as universal figures.

**Structure — three parts, in order:**
1. **What happened** — concrete, specific.
2. **Why** — only if it helps the user act.
3. **What to do next** — the recovery path.

Never ship a single vague descriptor (`Error`, `Something went wrong`) with no path.

**Tone & wording:**
- **Never blame.** Drop `invalid`, `illegal`, `incorrect`. Use neutral description (`That email isn't in our system`) or system ownership (`We couldn't find your account`).
- **Avoid humor in repeatable errors.** It goes stale on the second encounter. Exception: terminal/dead-end pages (404) where a light redirect reduces exit. Never use humor for payment, identity, or data-loss errors.
- **Avoid negatives in instructions.** `Use only letters and numbers` is processed faster and with fewer misreads than `Don't use special characters`. Avoid double negatives (`Do not unfollow`).

**Validation mechanics:**
- **Trigger on blur (field exit), never on keystroke.** Mid-typing errors feel punitive and are a consistently cited form frustration in usability studies. Exception: live *positive* progress feedback (password-strength meters) that confirms criteria as they're met — these fire green signals, not red errors.
- **Preserve user input.** Never wipe the field; show the original text so users correct rather than retype.
- **Redundant cues, not color alone.** Pair border highlight + icon + text label (accessibility + visibility). Color-only error states fail WCAG SC 1.4.1 and are invisible to colorblind users.
- **Catastrophic/unknown system errors:** emit a **correlation/reference ID**. `Something went wrong — reference #A4392` lets support trace the session and beats a bare generic string.

```text
Bad:   "Invalid input."
Bad:   "Error: 0x80004005"
Good:  "That email isn't in our system. Check the spelling, or create an account."
Good:  "We couldn't process the payment — your card wasn't charged. Try another card or contact your bank."
Good:  "Something went wrong on our end — reference #A4392. Refresh, or email support with that number."
```

```js
// Validate on blur, never on keystroke; preserve value; specific message.
emailField.addEventListener('blur', () => {
  if (!isValidEmail(emailField.value)) {
    showError(emailField, "That doesn't look like an email — check for a typo.");
  }
  // emailField.value is never cleared.
});
```

## 8. Scannability & the F-pattern

**The F-pattern is a failure state, not a target.** The original NNG eye-tracking research (2006) described the F-pattern as a description of what users do on unstructured pages with no visual hierarchy — not a template to design to. NNG's own subsequent research and Smashing Magazine's analysis (2017–2024) both reframe it as: when content lacks rhythm and hierarchy, users default to F-shape scanning and most of the page is never read. Your job is to provide visual anchors that shift users into **layer-cake** (heading-to-heading) or **commitment** scanning.

How users sweep: first horizontal pass covers ~top 20% of the viewport; the second pass is narrower; then a vertical stem down the left edge. **Right-column-below-fold content is highly likely to be missed.**

**Scannability tactics (each creates a left-margin anchor or a vertical entry point):**
- **Bold the key term at the *start* of a sentence** — it becomes a left-margin anchor during stem scanning.
- **Subheadings every 2–3 paragraphs.** They power layer-cake scanning.
- **Max 2–3 lines per paragraph.** Walls of text trigger pure F-pattern skimming.
- **Bullet lists for 3+ parallel items.** Parallel structure (same grammatical shape per bullet) speeds parsing.
- **Front-load every key idea** in the first 2 words of the line/sentence.

**Impact (NNG, scannable-web research):**
- Scannable formatting alone → **+47%** usability score vs unformatted equivalent.
- Concise + scannable + neutral language together → **+124%** usability improvement vs unformatted long-form.

## 9. Action-oriented labels vs noun labels

This is where most UI copy fails silently. Every interactive element that causes a state change must use a **verb + object** construction — not a noun, not an adjective, not a gerund standing alone.

**The noun-label failure mode:**
- `Report` → what does clicking do? Download? Open? Generate? — `Download report` or `View report` — specify the verb.
- `Settings` → acceptable as a destination label in nav. Not acceptable as a dialog confirmation button.
- `Upload` alone as a button in context where the destination is unclear → `Upload to campaign` or `Add photo`.

**Rules:**
- Every button that changes state: **verb + object**. 2–5 words.
- Every link that navigates: either a destination noun (nav context makes verb implicit) or a descriptive anchor (`Read the security policy`, not `Click here`).
- Never use `Click here`, `Learn more`, `Read more` as standalone link text — they have no meaning when read by a screen reader scanning the link list. Use `Learn more about pricing`, `Read the security policy`.
- **Action labels should predict the result.** The user should be able to mentally fill in: "After I click this, I will have ___." If they can't, the label needs a verb or an object or both.

**Gerund vs infinitive:** `Saving…` as a progress state is fine. `Saving` as a button label is not — it describes what is happening, not what the user initiates. The button is `Save`, the progress state is `Saving…`.

## 10. Removing jargon & sensitive-data copy

**Jargon replacement (grandparent test — would they understand it?):**

| Jargon | Plain |
|---|---|
| OTP code | 6-digit code (sent to your phone) |
| KYC process | "Verify your identity with a photo ID" (describe the actual steps) |
| Authentication / Authorization | Sign in · Verify your identity |
| Configure your credentials | Set your password |
| Tokenized payment instrument | Saved card |
| Deprecation notice | This feature will be removed on [date] |
| Rate limit | You've made too many requests — wait 60 seconds |

- **One term per entity, everywhere.** Pick `account` *or* `profile` — not both. Mixed terminology makes users wonder if two things are different. Enforce the term across labels, errors, tooltips, emails.
- **API/technical error codes:** never surface raw codes to end users. Map them: `403` → `You don't have permission to view this` + a recovery path.

**Sensitive fields (money, health, identity):**
- **Display all constraints statically and visibly** — never hide a required rule in a tooltip that needs a tap.
- **Use real numbers with dates, framed positively.**
  - Before: "Your deposit limit is $1,000 per calendar month."
  - After: "Until Jan 31, you can deposit $400 more." (personalized, actionable, real numbers.)
  - Frame thresholds as value, not blame: "You're $12 away from free shipping."

## 11. Voice & tone

- **Voice is constant; tone flexes by context.** Voice = what you consistently sound like (casual/formal, technical/plain). Tone = the adjustment for a specific moment. Error tone ≠ celebration tone ≠ onboarding tone — same voice, different register.
- **Define voice across 3 axes minimum before writing microcopy:** (1) formal ↔ casual, (2) technical ↔ plain, (3) serious ↔ playful. Assign a 1–5 position on each axis. Lock it. Write every string against that rubric.
- **Tone shifts with user emotional state:** calm and concrete during errors/payments; warm and encouraging in empty states and onboarding success; minimal and fast in dense tools (dashboards, data tables).
- **Brand voice is the *seasoning*, not the *meal*.** Clarity first; personality second, and only where it doesn't cost comprehension. `evian: "water the way nature intended"` works as a credibility claim on a brand surface — not as a button label.
- **Consistency is part of voice.** One term per entity, consistent capitalization on labels/buttons, consistent person (don't mix "your"/"my" arbitrarily — pick first-person for CTAs and hold it throughout all microcopy).
- **Voice document must cover: the 3-axis position + 3 things we say / 3 things we don't say + 3 tone examples per context (error, onboarding, success).** Without these, "friendly and approachable" is not actionable for a copywriter.

## Concrete specs & numbers
- **CTA copy length:** 2–5 words. <2 too vague; >5 loses imperative force.
- **VP attention window:** 10 seconds; users begin exiting at 10–20s with no clear VP (NNG).
- **Reading rate:** users read ~20–28% of words on a page (NNG); ~20% of attention below the fold even when scrolling.
- **Headline read-through:** ~80% read the headline, ~20% read past it.
- **Validation timing:** on **blur** (field exit), never on keystroke.
- **Formatting impact (NNG):** scannable = **+47%** usability; concise + scannable + neutral = **+124%**.
- **Negative phrasing:** "Use only…" processes faster and with fewer misreads than "Don't use…" — use positive phrasing in all instructions.
- **Paragraph length:** ≤2–3 lines. Subheads every 2–3 paragraphs. Bullets for ≥3 parallel items.
- **First horizontal eye sweep:** ~top 20% of viewport; right-column-below-fold = high miss risk.
- **Documented lifts (treat as directional hypotheses — replicate before assuming universality):** first-person CTA (ContentVerve) +90%; `Send Campaign` vs `Submit` (MailChimp) +18%; Expedia optional field removal ~$12M/yr; NNG scannable formatting +47%/+124%.
- **Stats to avoid citing without verification:** "79%/27% abandon on error" (NNG has qualitative research on this, not those specific percentages); "88% of headlines" (no primary source); "200%+ personalized CTA lift" (blog post, not controlled study).

## Do / Don't

**Do**
- Write the headline as a standalone value claim; assume it's the only thing read.
- Use `verb + object + value` on every action label; keep it 2–5 words.
- Frame CTAs first-person (`Get my…`) and contextualize them to funnel stage.
- Structure errors as what → why → what-next; preserve user input; validate on blur.
- Replace jargon; pick one term per entity and hold it everywhere.
- Bold sentence-leading key terms; cap paragraphs at 2–3 lines; bullet parallel sets.
- Show sensitive constraints statically with real numbers and dates.
- Make empty states drive a next action with a single CTA.
- Use visible labels above fields — never placeholder text as a substitute.
- Name the irreversible consequence in destructive-action dialogs; put it in the button label.
- Read every string aloud before shipping; stop at any stumble and rewrite.

**Don't**
- Don't use `Submit`, `Next`, `Learn more`, or noun-only buttons as primary CTAs.
- Don't say `invalid`, `illegal`, `incorrect`, or otherwise blame the user.
- Don't validate on keystroke or wipe fields on error.
- Don't bury the VP in section 2 or stash required info in a tooltip on a sensitive screen.
- Don't stack two constraints in one tooltip or two questions in one dialog.
- Don't ship `Error` / `Something went wrong` with no path or reference ID.
- Don't run two competing primary CTAs on one screen.
- Don't dilute a strong claim with weak supporting ones.
- Don't choose clever wordplay over literal clarity on conversion-critical copy.
- Don't use `Are you sure?` + `OK`/`Cancel` for destructive confirmations.
- Don't use placeholder text as a substitute for a label.
- Don't use `Loading…` alone for waits over 3 seconds — name what is loading.
- Don't surface raw error codes (HTTP status codes, exception names) to end users.

## Common pitfalls & how to avoid them
- **Clever headline that needs the body to make sense** → rewrite as a literal benefit (Four U's / How-To). 80% only read the headline.
- **`Submit` everywhere** → replace with the outcome the click produces.
- **Feature-listing in copy** → run each feature through "…which means you can ___" and ship the completion.
- **VP buried below the hero** → move to the first H1/top-left; users exit in 10–20s without it.
- **Keystroke validation / fields wiped on error** → validate on blur, persist input.
- **Blaming error language** → neutral description or system ownership; never `invalid`.
- **Jargon leaks (OTP/KYC/authenticate)** → plain-language swaps; grandparent test.
- **Mixed terminology (account vs profile)** → one term per entity, enforced across all surfaces.
- **Tooltip-buried constraints on payment/health/identity** → display statically with real numbers/dates.
- **F-pattern as default** → add bold lead-ins, subheads every 2–3 paragraphs, bullets; shift to layer-cake.
- **Two primary CTAs** → demote one to secondary styling.
- **Right-column / below-fold CTA** → move primary to hero top-left, repeat at scroll breaks.
- **`Are you sure?` + `OK`/`Cancel`** → name the item destroyed + consequence in title; consequence in button label.
- **Placeholder-as-label** → add a visible `<label>` above the input.
- **"Loading…" with no context** → name what is loading; add escape hatch at ~10s.
- **Generic success toasts** → close the loop + preview the next state.
- **"No results" with no path out** → explain the filter state + offer `Clear filters` or a suggestion.

## What real users complain about
- **"It threw an error while I was still typing."** Keystroke validation feels punitive — a top-cited form frustration in usability studies.
- **"The form deleted everything when one field was wrong."** Wiped inputs force full retypes; high-friction, high-abandon.
- **"`Something went wrong` — but what do I do now?"** Pathless errors strand users.
- **"I don't know what `Submit` will actually do."** Generic, value-free buttons create hesitation at the conversion moment.
- **"What's an OTP?" / "What's KYC?"** Jargon stalls non-expert users mid-flow.
- **"It says account here and profile there — are those different?"** Inconsistent terminology breeds uncertainty.
- **"I had to tap a tooltip to find a rule I needed before paying."** Hidden constraints on sensitive screens cause errors and distrust.
- **"The error blamed me — `invalid email` — when it was just a typo I couldn't see."** Accusatory tone increases drop-off.
- **"I read the whole hero and still don't know what this product does."** No clear VP in the first 10 seconds → exit.
- **"The button just said `OK` — I had no idea it would delete everything."** Generic destructive-action confirmations skip the consequence.
- **"It said `Loading…` for 20 seconds and I thought it crashed."** No progress copy or escape hatch = user abandons.

## Senior-level checklist
- [ ] Headline is a standalone benefit claim; hits ≥3 of the Four U's.
- [ ] VP is in the first H1 / hero top-left and answers why-this/why-you within 10 seconds.
- [ ] Every action label is `verb + object (+ value)`, 2–5 words; no `Submit`/`Next`/noun-only.
- [ ] Primary CTA is first-person where appropriate and contextual; exactly one primary CTA per screen.
- [ ] Reassurance/risk-reducer copy sits at the friction point (email/payment/signup).
- [ ] Every feature in copy is expressed as a user benefit.
- [ ] All errors follow what → why → what-next; no blame words; no bare `Error`.
- [ ] Validation fires on blur; user input is preserved on error; cues are redundant (not color-only).
- [ ] Catastrophic errors emit a reference/correlation ID.
- [ ] Jargon swapped to plain language; passes the grandparent test.
- [ ] One term per entity, enforced across labels/errors/tooltips/emails.
- [ ] Sensitive-field constraints shown statically with real numbers + dates, framed positively.
- [ ] Empty states: first-visit and post-filter states handled separately; both drive a single next action.
- [ ] Success states close the loop and preview the next state.
- [ ] Loading states name what is loading; escape hatch at ~10s for long waits.
- [ ] Destructive-action dialogs name the item + consequence in title; consequence in button label; not `OK`/`Cancel`.
- [ ] No placeholder text used as a substitute for a visible label.
- [ ] One idea per tooltip; one question per dialog.
- [ ] Scannable: bold sentence-leading terms, subheads every 2–3 paragraphs, ≤2–3-line paragraphs, bullets for ≥3 parallel items.
- [ ] Instructions use positive phrasing ("Use only…") not negatives.
- [ ] Voice defined on 3 axes; 3 things we say / 3 we don't; tone examples per context exist in the design system.
- [ ] Every string read aloud; no stumbles.

## Sources
- Nielsen Norman Group — How Users Read on the Web; F-pattern eye-tracking; scannability research; VP attention windows; form validation guidelines (nngroup.com)
- ContentVerve (2013) — first-person CTA A/B test ("Start my free 30-day trial" +90% CTR) — directional; site no longer active as of 2026; treat as a hypothesis to replicate
- MailChimp — "Send Campaign" vs "Submit" CTA test (+18% CTR)
- Expedia — optional "Company" field removal (~$12M/yr revenue impact; reported in Luke Wroblewski's "Web Form Design", 2008)
- Smashing Magazine — F-pattern recontextualization as failure state (multiple articles 2017–2024) (smashingmagazine.com)
- WCAG 2.1 SC 1.4.1 (color not sole means of conveying information), SC 1.3.1 (labels for inputs) (w3.org)

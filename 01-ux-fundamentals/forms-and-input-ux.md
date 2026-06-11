# Forms & Input UX

> Forms are where conversion is won or lost: checkout, signup, lead capture, and settings all live or die on input UX. This guide gives an AI builder the rules, numbers, and code patterns to ship forms that complete fast, validate humanely, and survive mobile, autofill, and accessibility scrutiny at a senior level.

**Apply when:** building any form — signup, login, checkout, contact, settings, multi-step flow · **Related:** [Accessibility (WCAG 2.2)](./accessibility-wcag.md) · [Cognitive Load & Progressive Disclosure](./cognitive-load-progressive-disclosure.md) · [E-commerce UX](../03-site-types/ecommerce-ux.md) · [Conversion Rate Optimization](../06-marketing-and-conversion/conversion-rate-optimization.md)

## TL;DR — rules at a glance

- **Single column, always.** Users finish single-column forms ~15.4s faster (CXL, 95% confidence). Multi-column causes field-skipping. Only short related pairs (city+state, first+last name) may share a row.
- **Top-aligned labels** as default: ~50ms saccade vs ~500ms for left-aligned (Penzo/UXmatters). Label sits immediately above its field → single fixation.
- **Never use placeholder as the label.** It vanishes on focus, fails contrast, screen-reader-unreliable. Use a persistent visible label + helper text below the field.
- **Validate on blur, not on keystroke.** Empty required fields validate only on submit. Apply "reward early, punish late."
- **Cut fields ruthlessly.** 3→4 fields drops conversion ~50% (HubSpot, 40k+ pages). Each extra field costs ~3–5%. Guest checkout target: 7–8 fields vs 14.88 benchmark (Baymard).
- **Mark required explicitly:** asterisk `*` + legend + `required` HTML attribute (add `aria-required="true"` only on custom ARIA widget roles, not native inputs). Never color alone (8% of men are colorblind).
- **Right input types & inputmode** so mobile shows the correct keyboard. `type="email"`, `type="tel"`, `inputmode="numeric"`, `autocomplete="..."`.
- **16px minimum input font** to stop iOS auto-zoom. Touch targets ≥44×44pt (Apple) / 48×48dp (Material) / 24×24 CSS px (WCAG 2.5.8).
- **Kill confirm-password and confirm-email.** Replace with a show/hide visibility toggle (GOV.UK pattern).
- **Error messages must be specific + adjacent + accessible.** "Enter a valid email like name@example.com," not "Invalid input." Use `aria-invalid` + `aria-describedby`.
- **Multi-step for ≥7 fields / complex flows** (case studies: BrokerNotes 11%→46%, Empire Flippers +51.6%; treat "86% higher avg" as directional, not a controlled figure); single-page for ≤3 fields. 3–5 steps is the optimal range.
- **Show full cost (incl. shipping/tax) from step 1.** 48% of cart abandonments stem from unexpected late costs (Baymard).
- **No Reset/Clear buttons.** Zero benefit, high data-loss risk. Don't disable the submit button unless the reason is obvious.
- **Never `autocomplete="off"` on identity/password fields** — it breaks autofill and password managers (accessibility failure).
- **Floating labels are a tradeoff, not an upgrade.** They block helper text and flash as placeholder-as-label on load. Use top-aligned persistent labels for transactional forms.
- **Use native `<select>` for 4–40 options; radios for 2–3; searchable combobox for 40+.** Custom dropdowns require full ARIA combobox implementation.
- **Persist draft state for ≥5-field forms.** Save to `sessionStorage`; restore with explicit user prompt; clear on submit.
- **Back navigation in multi-step flows must restore values.** Never discard on step unmount.

---

## Layout & structure

### Single column is the default
- Eye-tracking and timing studies (CXL: n=356 single vs n=346 multi) show single-column completed **15.4s faster, statistically significant at 95%**.
- Multi-column forms cause **eye zigzag**, ambiguous label↔field association, and **skipped fields** that then trigger confusing validation errors.
- **Only exception:** short, semantically linked fields on one row — `First name | Last name`, `City | State | ZIP`, `Expiry | CVC`. Each still keeps its own top label.
- Baymard: **16% of e-commerce sites use extensive multicolumn checkout forms** — a measurable abandonment risk.

```html
<!-- Good: single column, related pair shares a row -->
<form class="stack">
  <label for="email">Email</label>
  <input id="email" name="email" type="email" autocomplete="email" required>

  <div class="row">  <!-- the ONLY kind of multi-column allowed -->
    <div><label for="city">City</label><input id="city" autocomplete="address-level2"></div>
    <div><label for="zip">ZIP</label><input id="zip" inputmode="numeric" autocomplete="postal-code"></div>
  </div>
</form>
```
```css
.stack { display: grid; gap: 1rem; max-width: 28rem; }   /* one column, capped width */
.row   { display: grid; grid-template-columns: 2fr 1fr; gap: .75rem; }
@media (max-width: 480px) { .row { grid-template-columns: 1fr; } } /* collapse on mobile */
```

### Width as an affordance
- **Size each input to its expected content.** A 5-digit ZIP in a full-width field signals "type a lot here" and breeds format confusion.
- Short: ZIP/postal, CVC, quantity. Medium: phone, card expiry. Long: street address, email.
- **Full-width-everything is acceptable only on mobile**, where viewport forces it.

### Grouping & chunking
- Group with `<fieldset>` + `<legend>` (Contact, Shipping, Payment). Never present **more than ~6 questions** before a logical chunk break.
- For complex flows prefer **One-Thing-Per-Page** (GOV.UK) or clearly titled sections.

---

## Labels

- **Top-aligned by default.** Allows label + field in one visual fixation; ~50ms saccade vs ~500ms left-aligned (Penzo, UXmatters 2006).
- **Left-aligned** only when users scan many rows of unfamiliar data and column alignment aids comparison (rare in transactional forms).
- Label must sit **immediately above** the field — not floated far away, not as a placeholder.
- **Sentence case**, not ALL CAPS (caps slows reading and raises per-field scan time).
- Every input needs a programmatic label: `<label for>` or `aria-label`/`aria-labelledby`. A floating-label UI still requires a real `<label>` underneath.
- **Login fields:** if you accept an email, label it **"Email," not "Username."** Mislabeling drives confusion and support tickets.

```html
<!-- Bad: placeholder as label (anti-pattern, NN/g) -->
<input type="email" placeholder="Email address">

<!-- Good: persistent label + helper text below -->
<label for="phone">Phone</label>
<input id="phone" type="tel" autocomplete="tel"
       aria-describedby="phone-help">
<!-- type="tel" already triggers dialpad; inputmode="tel" is redundant -->
<p id="phone-help" class="help">Format: (555) 123-4567 — for delivery questions only.</p>
```

### Floating label pattern: tradeoffs
The Material Design "floating label" (placeholder that animates up to label position on focus) is widely shipped. Know its failure modes:
- **No room for helper text:** the label and helper text occupy the same visual real estate; a floating label field typically cannot also show persistent helper text below without the UI becoming cramped.
- **Flash of missing label:** on page load, unfocused fields show only placeholder text — identical to the placeholder-as-label anti-pattern from a glance distance.
- **SR ambiguity on older AT:** the floating `<label>` must still be a real `<label for>` (not a `<div>`); purely CSS-positioned labels with no `for` break screen readers.
- **Verdict:** acceptable for dense data-entry UIs (admin tables, settings forms) where space is the constraint. For transactional / checkout forms, persistent top-aligned labels with separate helper text are strictly better.

---

## Placeholder & helper text

- **Placeholder = optional secondary hint at most**, never the sole label. It disappears on focus → strains short-term memory → users clear input to re-read → higher error rates (NN/g).
- Placeholders also fail **WCAG 1.4.3 (4.5:1)** contrast at most browser defaults and are unreliably announced by screen readers.
- Put **format hints and constraints in persistent helper text below the field** so they stay visible while typing and after blur.
- Show **password requirements before and during typing**, not only as a post-submit error.

---

## Validation: timing & the "reward early, punish late" pattern

Trigger order (Mihael Konjević's pattern, validated by CXL):

1. **Success cues on valid format** — e.g., a checkmark once email format is structurally valid.
2. **Errors on blur** after the user leaves a **non-empty** field.
3. **Required-empty errors only on submit** — never flash "Required" on an untouched field.

**Reward early, punish late:**
- If a field was previously erroneous, **remove the error the instant it becomes valid** (don't wait for blur) — reward.
- If a field was previously valid, **wait until blur** to show a new error — punish late, so you don't nag mid-word.
- **Never validate on the first keystroke.** "Invalid email" while the user typed `jo` of `john@…` causes anxiety and *more* errors.
- For on-input validation (e.g., async username availability), **debounce 500–1000ms** after typing stops.

Konjević's inline-validation study impact: **+22% success, −42% completion time, −47% eye fixations, +31% satisfaction** (self-reported; treat as directional, not RCT-grade).

```js
function attachValidation(input, validate) {
  let wasInvalid = false;
  const show = (msg) => { /* set aria-invalid, render msg adjacent */ };
  const clear = () => { /* unset aria-invalid, remove msg */ };

  input.addEventListener('blur', () => {
    if (input.value === '') return;            // empty => defer to submit
    const err = validate(input.value);
    if (err) { show(err); wasInvalid = true; } else clear();
  });

  input.addEventListener('input', () => {       // reward early ONLY
    if (!wasInvalid) return;                     // never punish mid-typing
    if (!validate(input.value)) { clear(); wasInvalid = false; }
  });
}
```

### Error message craft
- **Specific + actionable:** "Enter a valid email like name@example.com." Not "Invalid input," "Required," or "Something went wrong."
- **Adjacent** to the offending field, **visually distinct** (red + icon, never color alone), and accessible:

```html
<label for="card">Card number</label>
<input id="card" inputmode="numeric" autocomplete="cc-number"
       aria-invalid="true" aria-describedby="card-err">
<p id="card-err" class="error" role="alert">
  <svg aria-hidden="true">…</svg> Card number must be 16 digits.
</p>
```

- **On submit, use a summary at the top AND inline errors at fields.** Summary links jump to each field. Never summary-only (memory burden) or inline-only (hard to locate in long forms). Move focus to the summary on submit failure.
- **Operational signal:** if the same user hits the same field error 3+ times, surface a help/contact link and flag the field for UX testing — repeated errors are a design failure, not a user failure.

---

## Required vs optional marking

- **If most fields are required:** mark required with `*` + a legend ("Fields marked * are required"). 
- **If most fields are optional:** mark the optionals with "(optional)" and leave required unmarked.
- **Always** pair the visual cue with the HTML attribute: `required` (HTML5 `required` implies `aria-required` for modern AT; add `aria-required="true"` only if supporting legacy screen readers or if the element is not a native `<input>`/`<select>`/`<textarea>`).
- **Never rely on color alone** — colorblindness affects ~8% of men, ~0.5% of women. The asterisk/text is the real signal; color is decoration.

```html
<label for="name">Full name <span aria-hidden="true">*</span></label>
<input id="name" required autocomplete="name">
<!-- elsewhere: <p>Fields marked * are required.</p> -->
<!-- add aria-required="true" only on custom role="combobox"/role="spinbutton" etc. -->
```

---

## Reducing field count

- **HubSpot (40k+ landing pages):** going from 4→3 fields lifts conversion ~50%; each added field costs ~3–5% on average (mean: ~4.1% drop per field). The absolute completion rate at any count depends heavily on form type and audience — use the per-field cost as a marginal decision tool, not an absolute benchmark.
- **Baymard benchmark:** average checkout has **14.88 form fields / 19.9 elements**; an optimized **guest checkout reaches 7–8.**
- Tactics to cut fields:
  - Derive **city + state from ZIP** via lookup; show them as confirmation, not input.
  - Use **address autocomplete** (Google Places / Mapbox / Shopify) to collapse multiple address fields.
  - Drop **company, title, second phone, "how did you hear about us"** unless operationally required.
  - **Remove confirm-password and confirm-email** entirely.
  - Combine first/last into one **Full name** field where downstream systems allow.
- Ask for data **when it's needed**, not upfront (progressive profiling).

---

## Smart defaults, autofill & input types

### Autocomplete attributes (set them on every known field)
Chrome team data: autofill makes users **~30% faster** on checkout forms; it's also a critical accessibility accommodation for motor-impaired users. Every field that maps to a known token must have `autocomplete` set — omitting it disables browser and password-manager fill for that field.

| Field | `autocomplete` value |
|---|---|
| First / last name | `given-name` / `family-name` (or `name`) |
| Email | `email` |
| Phone | `tel` |
| Street | `street-address` (or `address-line1`/`address-line2`) |
| City / region / postal | `address-level2` / `address-level1` / `postal-code` |
| Country | `country-name` |
| Card number / exp / CVC | `cc-number` / `cc-exp` / `cc-csc` |
| OTP code | `one-time-code` |
| New / current password | `new-password` / `current-password` |

- **Never set `autocomplete="off"`** on identity or password fields — it disables autofill and password managers and is an accessibility failure.

### Input types & mobile keyboards
| Need | Use |
|---|---|
| Email | `type="email"` (shows `@` key) |
| Phone | `type="tel"` (dialpad; `inputmode="tel"` is redundant when `type="tel"` is set) |
| Numeric string (ZIP, OTP, card) | `inputmode="numeric"` (digits without number-spinner/validation quirks) |
| True quantity | `type="number"` (number pad + min/max/step) |
| Date | `type="date"` (native picker) |
| URL | `type="url"` |

- Prefer `inputmode="numeric"` over `type="number"` for ZIP/card/OTP — `type="number"` strips leading zeros, allows `e`/`+`/`-`, and adds unwanted spinners.

### Smart defaults
- Pre-select the most common option (country by IP/locale, "same as billing" checked).
- Auto-format as the user types: **card numbers with spaces, phone with separators** — but store normalized values.
- **US ZIP autodetect:** trigger lookup on the **5th digit**. Place **City/State fields BELOW the ZIP**, never above — focus flows downward, so fields above the trigger get skipped.

---

## Selects, date inputs & textareas

### Select / dropdown
- Use native `<select>` unless you need search/filter, multi-select with chips, or grouped options with icons. Custom dropdowns have well-known keyboard/AT bugs — only build one if you commit to full ARIA combobox implementation.
- **Never use a disabled, placeholder-style first option as a label** (`<option value="" disabled selected>Choose…</option>`) — it disappears on selection and fails the same way as placeholder-as-label. Instead, use a real `<label>` above, and if an empty first option is needed for validation, mark it `aria-hidden` and add `required` on the `<select>`.
- Set `autocomplete` on `<select>` too: `autocomplete="country"` on a country picker works correctly.
- Avoid `<select>` for 2–3 options — use radio buttons (all options visible, one tap).
- Avoid `<select>` for 50+ options without search — use a combobox with filtering.

### Date inputs
- `type="date"` renders a native picker on mobile (good) but uses the OS locale's date format, which may conflict with visual labels. **Always set a `<label>` that includes the format hint** ("Expiry date (MM/YYYY)").
- On desktop Chrome/Firefox, `type="date"` shows a calendar widget; on Safari it degrades to a plain text input with inconsistent format expectation — test both.
- For **date of birth**, a set of three `<select>` menus (Day / Month / Year) or two `<input type="number">` fields + year outperforms `type="date"` on cross-browser consistency and mobile UX.
- For **appointment/calendar selection**, a custom datepicker is required — use an accessible library (e.g., Pikaday, React DayPicker with ARIA).
- Avoid `type="datetime-local"` in public-facing forms — browser UI is poor and timezone handling is invisible to users.

### Textarea
- Use `<textarea>` for free-form text expected to be >1 line (messages, descriptions, notes). Use `<input type="text">` for single-line entries.
- Set a visible `rows` attribute to reflect expected length (4–6 rows for a message; 2–3 for a short note); don't rely on the browser default of 2 rows.
- Show a character or word count feedback if there is a `maxlength` constraint. Combine with a live `aria-live` region so screen reader users hear remaining count.
- `resize: vertical` is acceptable; `resize: none` is acceptable on fixed-height UIs; `resize: horizontal` confuses layout — avoid.

---

## Passwords & authentication

- **Single password field + show/hide toggle.** The GOV.UK Design System recommends this as the canonical pattern, with evidence that show/hide removes the need for confirm-password.
- Show requirements **before and during** typing; reflect progress live (length, character classes).
- **Allow paste.** Disabling paste breaks password managers and frustrates users.
- `autocomplete="new-password"` on signup, `current-password` on login — enables manager save/fill.
- **Zuko Analytics:** password fields have the **highest mean abandonment of any common field at 10.5%** (email 6.4%, phone 6.3%) — minimize friction here above all.

```html
<label for="pw">Password</label>
<div class="pw">
  <input id="pw" type="password" autocomplete="new-password"
         aria-describedby="pw-rules" minlength="8">
  <button type="button" class="toggle" aria-pressed="false"
          aria-controls="pw" aria-label="Show password">Show</button>
</div>
<p id="pw-rules" class="help">At least 8 characters. Longer is stronger. Paste is allowed.</p>
```

---

## Multi-step vs single-page

- **Single-page** for ≤3 fields or simple actions (newsletter, contact).
- **Multi-step** for ≥7 fields, complex/sensitive data (checkout, registration), and mobile-first flows.
- **HubSpot case studies:** BrokerNotes **11%→46%**, Venture Harbour **0.96%→8.1% (+743%)**, Empire Flippers **+51.6% in 47 days**. The "~86% higher on average" figure is blog-level aggregation, not a controlled study — use individual case studies when citing.
- **Optimal length: 3–5 steps.** Completion declines sharply beyond ~6 steps. Target ≤5 min total to complete.
- **First step = the easiest possible** (e.g., one low-effort choice) to trigger sunk-cost/commitment psychology, then ask for harder data later.

### Progress indicators
- **5+ steps:** labeled step indicators with named sections ("Cart → Shipping → Payment → Review").
- **Shorter forms:** a percentage bar reads fine.
- **Conditional/branching paths:** use a **percentage, not step count** — never show a numbered step the user may never reach.

### Back button & state preservation
- Every step must be reachable via the browser back button or the step indicator — going back must **restore previously entered values**, never reset them.
- Persist step state in component state, URL params, or `sessionStorage`; never discard on unmount.
- If the user lands on step 3 via refresh, redirect to step 1 (or restore from `sessionStorage`) — never show a broken mid-flow state.
- For long flows or authenticated users: **autosave to the server on each step completion** so network failures don't erase data.

### Draft persistence (single-page forms)
- For forms with ≥5 fields or significant free-text, persist field values to `localStorage`/`sessionStorage` on `input` events (debounce 2s).
- Restore on page load with a visible "Restore your previous entry?" prompt — never silently restore if values may be stale.
- Clear draft on successful submit.

---

## Checkout & signup specifics (Baymard-grounded)

- **Guest checkout as the most prominent CTA.** **18% abandon because they don't want to create an account**, yet **62% of sites don't make guest checkout the most prominent option.** Offer account creation *after* purchase via a single "set a password" field.
- **Enclosed layout:** strip the main nav during checkout to keep focus.
- **Show total cost — including shipping & tax — from step 1.** **48% of abandonments are unexpected late costs.** 17% abandon over payment-security concerns → show trust/security cues near card fields.
- **26% abandon solely due to checkout length/complexity** → minimize steps and fields.
- **Card UX:** auto-format with spaces, validate with the **Luhn algorithm** in real time, autodetect card type, use `cc-*` autocomplete, `inputmode="numeric"`.
- **Address:** use autocomplete; **28% of mobile sites lack ZIP autodetection or full address autocomplete** (Baymard).
- **Labels:** Baymard found label placement and clarity issues across the majority of top e-commerce sites; keep labels explicit, persistent, and above the field (no placeholder-only).
- **Mobile:** cart abandonment ~**85.65%**; nearly **40% of mobile shoppers cite difficulty entering information** — this is where keyboard types, autofill, and 16px fonts pay off most.

---

## Anti-spam, CAPTCHA & destructive controls

- Prefer a **honeypot** field (hidden input bots fill, humans don't) over visible CAPTCHA.
- If CAPTCHA is unavoidable: **invisible reCAPTCHA v3 > v2 checkbox**, and always provide an **audio fallback**.
- **No Reset/Clear buttons** — they cause irreversible data loss with no recovery; benefit ≈ 0, harm significant.
- **Don't disable the submit button** unless it's truly non-functional. Disabled submits with no obvious reason approach ~100% abandonment. Instead, let users submit and show inline errors.

---

## Concrete specs & numbers

- **Min input font:** `16px` (prevents iOS focus auto-zoom).
- **Touch targets:** ≥44×44pt (Apple HIG), ≥48×48dp (Material 3), ≥24×24 CSS px (WCAG 2.5.8); **≥8dp spacing** between targets.
- **Contrast:** ≥4.5:1 small text, ≥3:1 large text and UI/focus indicators (WCAG 1.4.3 / 1.4.11).
- **Saccade per field:** ~50ms top-aligned vs ~500ms left-aligned labels.
- **Single vs multi-column:** ~15.4s faster (CXL, 95% confidence).
- **On-input debounce:** 500–1000ms after typing stops.
- **Inline validation gains:** +22% success, −42% time, −47% fixations, +31% satisfaction (Konjević; directional).
- **Field economics:** 4→3 fields ≈ +50% conversion; each added field ≈ −4.1% on avg (HubSpot, 40k+ pages; absolute completion varies by form type).
- **Checkout fields:** 14.88 benchmark → 7–8 optimized guest (Baymard).
- **Cart abandonment:** ~70.19–70.22% avg (Baymard, 50 studies); mobile ~85.65%.
- **Field abandonment (Zuko):** password 10.5%, email 6.4%, phone 6.3%.
- **Multi-step:** directional ~86% higher conversion (HubSpot blog-level); individual case studies range +51% to +743%; 3–5 steps optimal.
- **Recoverable upside:** checkout UX fixes can lift conversions up to **35.26%** (~$260B across US+EU, Baymard).

```css
:root { --gap: 1rem; --field-h: 48px; }
input, select, textarea {
  font-size: 16px;            /* no iOS zoom */
  min-height: var(--field-h); /* comfortable touch target */
  padding: .625rem .75rem;
}
:focus-visible { outline: 3px solid #2563eb; outline-offset: 2px; } /* ≥3:1, never remove */
```

---

## Tools & libraries

- **Validation/state:** React Hook Form (uncontrolled, fast) or Formik; **Zod**/**Valibot** for schema. Native Constraint Validation API for simple forms — no library needed.
- **Address autocomplete:** Google Places API, Mapbox Address Autofill, or Shopify Checkout Extensions. Collapses multi-field address entry, reduces errors.
- **Card validation:** Luhn check client-side; Stripe Elements / Payment Element to avoid handling raw PAN and to get formatting + validation for free.
- **Phone:** `libphonenumber-js` for parse/format/validate by region.
- **Anti-spam:** honeypot first; reCAPTCHA v3 / Cloudflare Turnstile if needed.
- **When NOT to reach for a library:** a 1–3 field form (login, newsletter) — native HTML5 validation + `inputmode`/`autocomplete` is enough; adding a form library is over-engineering.

---

## Do / Don't

**Do**
- Use one column; top-aligned, persistent labels.
- Validate on blur; show success early, errors late.
- Set `autocomplete`, correct `type`, and `inputmode` on every field.
- Keep helper text/format hints visible below the field.
- Offer prominent guest checkout; show full cost up front.
- Replace confirm-password with show/hide; allow paste.
- Provide both a top error summary and inline field errors on submit.
- Size inputs to expected content; 16px font; ≥44px targets.

**Don't**
- Use placeholders as labels, or ALL-CAPS labels.
- Validate on first keystroke or flag untouched required fields.
- Write "Invalid input" / "Required" / "Something went wrong."
- Use multi-column layouts for question fields.
- Ship confirm-password, confirm-email, or Reset/Clear buttons.
- Set `autocomplete="off"` on identity/password fields or block paste.
- Disable the submit button without an obvious, stated reason.
- Force account creation before purchase.
- Use floating labels on forms that also need helper text.
- Use `<select>` for 2–3 options (use radios) or for 50+ without search (use combobox).
- Use `type="number"` for ZIP, card, or OTP fields.
- Discard entered values when navigating back in a multi-step flow.
- Use a placeholder-style disabled first `<option>` as a field label.

---

## Common pitfalls & how to avoid them

- **Placeholder-as-label** → disappears on focus, fails contrast, SR-unreliable. *Fix:* persistent `<label>` + helper text below.
- **Multi-column question fields** → skipped fields, mis-association, confusing errors. *Fix:* single column; pairs only for tight related fields.
- **Keystroke validation** → "Invalid email" mid-word raises anxiety and errors. *Fix:* blur for errors, debounce 500–1000ms for async, reward-early.
- **Premature required errors** → flagging untouched fields is antagonistic. *Fix:* required-empty errors on submit only.
- **Generic errors** → user must guess. *Fix:* specific, actionable, adjacent, accessible messages.
- **Confirm-password / confirm-email** → double friction, no real security gain. *Fix:* show/hide toggle; remove confirm-email.
- **`autocomplete="off"`** → harms cognitive/motor-impaired users and breaks managers. *Fix:* correct `autocomplete` tokens.
- **Reset/Clear buttons** → accidental, unrecoverable data loss. *Fix:* remove them.
- **`<10px`/`<16px` inputs on mobile** → iOS auto-zoom and mis-taps. *Fix:* 16px font, ≥44px targets.
- **City/State above ZIP** → skipped on downward focus flow. *Fix:* ZIP first, autodetect on 5th digit, derived fields below.
- **`type="number"` for ZIP/card/OTP** → strips leading zeros, allows `e/+/-`, spinners. *Fix:* `inputmode="numeric"`.
- **Floating label without real `<label>`** → positioned `<div>` styled to look like a label breaks programmatic association. *Fix:* real `<label for>` always present.
- **Disabled first `<option>` as label** → disappears on selection, SR-confusing. *Fix:* persistent `<label>` above; if placeholder option needed, mark `aria-hidden`.
- **Back button discards step data** → users lose work when navigating in multi-step flows. *Fix:* persist step state; restore on back.
- **`type="date"` without format label** → different browsers/locales display different formats; user enters wrong format. *Fix:* label includes the expected format explicitly.

---

## What real users complain about

- **"The form lost everything when validation failed"** — no draft persistence; one server error wipes a long form. Persist field values; never auto-clear.
- **"It kept yelling 'Invalid email' while I was still typing it."** — keystroke validation. Classic r/userexperience complaint.
- **"I couldn't read the label after I clicked the box."** — placeholder-as-label memory loss.
- **"Why does it want me to make an account just to buy one thing?"** — forced registration; **18% abandon** for exactly this.
- **"Shipping doubled the price at the last step."** — late cost reveal; **48% of abandonments.**
- **"My password manager wouldn't fill it"** — `autocomplete="off"` / paste blocked.
- **"On my phone it zoomed in and I couldn't tap the next field."** — sub-16px fonts, tiny targets; ~40% of mobile shoppers cite input difficulty.
- **"'Something went wrong' — went wrong WHERE?"** — generic, non-adjacent errors.
- **"The submit button was greyed out and I had no idea why."** — silently disabled submit → near-total abandonment.
- **"I hit back to fix my address and it reset everything."** — multi-step back navigation discards state; common in SPA form flows that destroy component state on route change.
- **"The dropdown had 200 countries and I couldn't search."** — `<select>` with 200 options, no filtering; should be a searchable combobox.

---

## Senior-level checklist

- [ ] Single-column layout; only tight related fields share a row, each with its own label.
- [ ] Top-aligned, persistent, sentence-case labels on every input; programmatically associated.
- [ ] No placeholder used as a label; format hints in visible helper text below the field.
- [ ] Validation: blur for errors, submit-only for empty-required, reward-early/punish-late; async debounced 500–1000ms.
- [ ] Error messages specific, actionable, adjacent, icon+color (not color alone), `aria-invalid` + `aria-describedby`; submit shows top summary + inline.
- [ ] Required/optional marked with text/asterisk **and** `required` (add `aria-required="true"` only on custom ARIA widget roles); no color-only signaling.
- [ ] Every field has correct `type`, `inputmode`, and `autocomplete`.
- [ ] Inputs ≥16px font; targets ≥44×44pt/48×48dp/24px; focus visible at ≥3:1; text contrast ≥4.5:1.
- [ ] Field count minimized; city/state derived from ZIP where possible; no confirm-password/confirm-email.
- [ ] Password = single field + show/hide; paste allowed; `new-password`/`current-password` set; requirements shown live.
- [ ] Multi-step used for ≥7 fields (3–5 steps), easiest step first, labeled progress; single-page for ≤3.
- [ ] Checkout: prominent guest option, enclosed layout, full cost incl. shipping/tax from step 1, ZIP autodetect on 5th digit, Luhn card validation, auto-formatting, security cues.
- [ ] No Reset/Clear buttons; submit not disabled without an obvious reason; field values persist on error.
- [ ] Honeypot before any visible CAPTCHA; if CAPTCHA, reCAPTCHA v3/Turnstile + audio fallback.
- [ ] No floating labels on transactional/checkout forms; if floating labels are used, real `<label for>` is present, not a positioned `<div>`.
- [ ] Selects: native `<select>` for 4–40 options; radios for 2–3; no placeholder-style disabled first option as label.
- [ ] Date inputs: label includes format hint; DOB uses select-menus or explicit number fields; `type="datetime-local"` not used in public forms.
- [ ] Textareas: `rows` attribute reflects expected length; maxlength feedback via live region.
- [ ] Multi-step: back button restores values; state persisted per step; autosave on step completion for authenticated flows.
- [ ] Draft persistence: ≥5-field forms save to sessionStorage; restored with user prompt; cleared on submit.
- [ ] Keyboard-only and screen-reader pass: tab order logical, errors announced, all controls labeled.

---

## Sources

- Baymard Institute — Checkout UX & Cart Abandonment research: https://baymard.com/research/checkout-usability and https://baymard.com/lists/cart-abandonment-rate
- CXL — Form design / single-column & inline validation studies: https://cxl.com/blog/form-design-best-practices/
- Nielsen Norman Group — Placeholders in Form Fields Are Harmful: https://www.nngroup.com/articles/form-design-placeholders/
- Mihael Konjević — "Inline validation in forms — designing the experience" (reward early, punish late; +22%/−42%/−47%/+31% figures): https://medium.com/wdstack/inline-validation-in-forms-designing-the-experience-123fb34088ce
- Chrome Developers / Google — Autofill and autocomplete best practices: https://web.dev/articles/payment-and-address-form-best-practices
- Matteo Penzo — "Label Placement in Forms," UXmatters (2006): https://www.uxmatters.com/mt/archives/2006/07/label-placement-in-forms.php
- GOV.UK Design System — Password input / show-hide pattern: https://design-system.service.gov.uk/components/password-input/
- Material Design 3 — Touch target sizing: https://m3.material.io/foundations/accessible-design/accessibility-basics
- Apple Human Interface Guidelines — Layout / hit targets: https://developer.apple.com/design/human-interface-guidelines/layout
- WCAG 2.2 — Target Size (Minimum) 2.5.8 & Contrast 1.4.3: https://www.w3.org/TR/WCAG22/
- HubSpot — Form length & multi-step conversion research: https://blog.hubspot.com/marketing/form-conversion-rate
- Zuko Analytics — Form field abandonment benchmarks: https://www.zuko.io/benchmarks

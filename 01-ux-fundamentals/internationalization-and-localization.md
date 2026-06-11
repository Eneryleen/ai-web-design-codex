# Internationalization (i18n) & Localization (l10n)
> How to design and build layouts that survive translation and locale switching: text-expansion budgets, full RTL support, locale-aware `Intl` formatting, language-switcher UX, and pseudo-localized QA. The senior-level outcome: one codebase that adapts to any market without per-locale CSS forks, truncation, or mixed-locale bugs.

**Apply when:** building anything that may ship in more than one language, region, or currency — even if it launches monolingual. · **Related:** [Multilingual & Non-Latin Typography](../00-foundations/multilingual-and-non-latin-typography.md) · [Forms & Input UX](forms-and-input-ux.md) · [Accessibility (WCAG 2.2)](accessibility-wcag.md) · [Copywriting & UX Writing](../06-marketing-and-conversion/copywriting-and-ux-writing.md)

## TL;DR — rules at a glance
- **i18n is built once, up front; l10n recurs per market.** Externalize strings, use logical CSS, and format with `Intl` from the first commit — retrofitting a monolingual codebase is the biggest localization cost, far bigger than translation.
- **Design for the longest language first.** Lay out screens in **German** (expansion) and **Arabic** (RTL) *before* sign-off. Survive those two and English just gets breathing room.
- **Never hardcode widths, never truncate, never fix heights** for translatable text. Use `min-width`/`min-height`, `max-content` label columns, wrapping over absolute positioning. Zero absolute positioning is the readiness ideal.
- **Expansion budget is per-string-length, not flat.** ~30–35% for body text; **100–300% (design for ~2×)** for short UI strings (buttons, nav, labels, badges, abbreviations).
- **Use CSS logical properties** (`margin-inline`, `inset-inline-start`, `padding-block`) — one stylesheet serves LTR and RTL. But they still require `dir` to be set; without `dir="rtl"` everything behaves LTR even for Arabic content.
- **Mirror directional things, keep content-intrinsic things.** Flip layout, nav order, back/next arrows, OK/Cancel order. Do NOT flip logos, play/FF/rewind icons, charts, numbers, phone numbers, code/URLs.
- **Keep raw values as data; format only at the UI boundary** with `Intl`. Storing `"$1,234.56"` strings causes mixed-locale bugs — store the number/Date and format on render per active locale.
- **Never build sentences by concatenation.** A translatable unit is a whole sentence with named placeholders so translators can reorder, inflect, pluralize, and gender it.
- **Detect a smart default, never force.** Fallback chain: explicit user choice → `Accept-Language` (with close-match fallback) → IP geo (last resort) → site default. **IP-only auto-redirect is the cardinal sin.**
- **Physical location ≠ language preference.** A Belgian/Swiss IP maps to 2–4 official languages; a Tokyo IP doesn't mean the user reads Japanese.
- **Decouple language, country/region, and currency** into three independent settings. Bundling them traps users.
- **Label languages with endonyms** (`Deutsch`, `中文`), not English names or 2-letter codes. Flags are for countries, never languages.
- **The stable `Intl` core** (`NumberFormat`, `DateTimeFormat`, `RelativeTimeFormat`, `PluralRules`, `Collator`, `ListFormat`, `DisplayNames`, `Locale`) is **Baseline-widely-available** and ships no extra locale data over the wire; it replaces Moment.js. The **newest members lag**: `Intl.Segmenter` Baseline since Apr 2024, `Intl.DurationFormat` only since Mar 2025 — feature-detect or polyfill those, don't assume them. **Reuse** formatter instances — construction is the expensive part.
- **Pseudo-localize in CI** to surface truncation, hardcoded strings, missing glyphs, and overflow *before* real translations exist.

---

## 1. i18n vs l10n — the mental model
- **Internationalization (i18n):** engineering the codebase/layout so it *can* adapt to any locale — externalized strings, flexible content-driven layout, `Intl` formatting, logical CSS, bidi-safe markup. Built **once**.
- **Localization (l10n):** the actual per-locale adaptation — translations, locale formats, RTL mirroring, market copy. Recurs **per market**.
- **Locale ≠ language.** A locale is `language-REGION` (e.g. `en-US`, `en-GB`, `pt-BR`, `zh-Hans-CN`). It drives translations *and* number/date/currency/collation behavior. `en-US` and `en-GB` share a translation but format dates differently.
- **The three-axis rule:** language (what they read), country/region (shipping, legal, tax), and currency (what they pay in) are **independent**. A user can want German UI, ship to Austria, and pay in CHF. Never let one selection silently force another.
- **Build it in even for one language.** Day-one cost of translation keys + `Intl` + logical properties is trivial; the cost of going back to find every hardcoded string and `padding-left` is enormous.

## 2. Text expansion & contraction — the layout budget
Translated text changes length unpredictably. **Budget by string length, not a single flat percentage** — a paragraph-tuned 35% budget *breaks buttons*.

| String type | Budget | Why |
|---|---|---|
| Body / paragraph | **+30–35%** | averages out over many words |
| Short UI: buttons, nav, labels, tabs, badges, abbreviations | **+100–300% (design ~2×)** | single words have no averaging; compounds explode |

**Per-language averages (product-wide):**

| Language | Expansion vs English | Notes |
|---|---|---|
| German | **~35%** avg, up to **+70%** on short strings | compound nouns (`Geschwindigkeitsbegrenzung`) |
| Finnish | ~30–40% | agglutinative compounds |
| Russian / Polish | ~15–35% | Cyrillic + cases; also plural complexity |
| Spanish | ~25% | — |
| Portuguese | ~15–30% | — |
| French | ~15–20% | — |
| Italian | ~10–25% | — |
| CJK (中文/日本語/한국어) | **−10% to −55% in character count** | but needs more **vertical** space & `line-height`; width may not shrink; demands full glyph coverage |

**High-risk components, in priority order:** horizontal navigation (#1 failure point — a 5-item English nav overflows/wraps/truncates in German) → button labels → status indicators/badges → tabs.

**Build rules:**
```css
/* Buttons: grow to fit, never clip */
.btn { min-width: 8ch; width: auto; white-space: normal; }

/* Form label column: size to the longest label in this locale */
.form-grid { grid-template-columns: max-content 1fr; }

/* Cards: grow with content */
.card { min-height: 8rem; height: auto; }

/* Compound-word wrapping — REQUIRES a correct lang attribute */
.prose { hyphens: auto; overflow-wrap: anywhere; }
```
```html
<html lang="de"> <!-- hyphens:auto needs lang to pick the right dictionary -->
```
- **Never** wrap translatable text in `overflow: hidden` or default `white-space: nowrap`.
- **Never** use fixed `height`/`width` on a translatable element.
- If you must clamp, use `-webkit-line-clamp` (multi-line, reflows) — never single-line hard truncation as a layout crutch.
- Test the layout with the longest realistic string, not the English placeholder.

## 3. RTL: bidirectional layout
Arabic, Hebrew, Persian (Farsi), Urdu read right-to-left. RTL is **not** a skin you add later — design bidirectional from day one with logical properties.

### 3.1 Logical properties (write once, serve both)
Use logical (flow-relative) instead of physical (left/right) properties so a single stylesheet mirrors automatically:

| Physical (avoid) | Logical (use) |
|---|---|
| `margin-left` / `margin-right` | `margin-inline-start` / `margin-inline-end` |
| `padding-top` / `padding-bottom` | `padding-block-start` / `padding-block-end` |
| `left` / `right` | `inset-inline-start` / `inset-inline-end` |
| `border-left` | `border-inline-start` |
| `text-align: left` | `text-align: start` |
| `width` | `inline-size` |

```css
.menu-item { padding-inline-start: 1rem; border-inline-start: 2px solid; }
```
- Supported in **all modern browsers** (Chrome, Firefox, Safari, Edge); only retired IE lacks them.
- **Critical gotcha:** logical properties do nothing on their own — they resolve against the element's direction. Without `dir="rtl"` (or `:dir(rtl)` context), everything resolves LTR even for Arabic text.

```html
<html lang="ar" dir="rtl"> ... </html>
```

### 3.2 `:dir()` vs `[dir=rtl]` — not equivalent
- `:dir(rtl)` matches the **user-agent-computed** direction, including inherited and `dir="auto"`-resolved direction.
- `[dir=rtl]` (attribute selector) matches **only an explicit HTML attribute** and ignores inherited/auto.
- Prefer `:dir()` for styling that must follow real rendered direction:
```css
.chevron:dir(rtl) { transform: scaleX(-1); } /* flip the arrow only in RTL */
```

### 3.3 What MUST flip vs what must NOT
**Mirror (directional):** overall layout & reading order · navigation order · Back/Next/forward/rewind directional arrows & chevrons · OK/Cancel button order and side · paddings/margins/borders (via logical props) · scrollbar side · edit-control handles, sliders, dropdown carets · **time-based progress** (in RTL, "forward" points left, "back" points right).

**Keep (content-intrinsic):** logos & brand marks · most symbolic icons (search 🔍, user, settings, home) · **media play / fast-forward / rewind icons** (universal, same both directions) · checkmarks · real-world-anchored imagery · charts/graphs whose axes encode data · **numbers, phone numbers, code, URLs**.

> Rule of thumb: if the icon means *direction or sequence*, flip it. If it means a *thing or a brand*, don't.

## 4. Bidi: mixed LTR runs inside RTL text
RTL pages constantly embed LTR runs (English brand names, URLs, code, emails) and vice-versa. Get the **base direction** right, then **isolate** the exceptions.

- **Set base direction in markup:** `<html lang="ar" dir="rtl">`. Override per-element with `lang`/`dir`.
- **Isolate embedded opposite-direction runs** so the bidi algorithm doesn't reorder surrounding punctuation:
```html
<p dir="rtl">المستخدم <bdi>@john_doe</bdi> علّق على <bdi>example.com/page</bdi></p>
```
- `<bdi>` (bidi isolate) is purpose-built for unknown-direction data like usernames. CSS equivalent: `unicode-bidi: isolate`.
- **`dir="auto"` on inputs** so direction follows the first strong character the user types (an Arabic input that may receive English):
```html
<input dir="auto" name="comment">
<textarea dir="auto"></textarea>
```
- **Numbers stay LTR even inside RTL text**, but **sequence rendering differs by language** — Hebrew/Persian render ascending numbers LTR; Mashreq (Eastern) Arabic can render a numeric sequence RTL. **Never assume one universal rule.** Arabic-Indic (٠١٢٣) vs Western (0123) digits also vary by locale and are an `Intl`/font concern, not a manual one.
- **W3C rule: prefer bidi *markup* over CSS.** CSS can be stripped, overridden, or lost when content moves to another context (RSS, email, copy-paste); markup travels with the content. When you must use CSS, use `unicode-bidi: isolate` (and `isolate-override` for `<bdo>`) — not plain `embed`.
- **LRM/RLM marks** (`U+200E` / `U+200F`) are a last resort for punctuation edge cases where neither markup nor CSS applies (e.g. text in an attribute). Don't reach for them first.

## 5. Locale-aware formatting with `Intl` (ECMA-402)
**Keep raw values as data; format only at the UI boundary.** Store a `number`/`Date`/ISO string; format on render against the active locale. The stable `Intl` core is **Baseline-widely-available**, ships **no extra locale data** over the wire (the browser provides CLDR), and replaces Moment.js. Two members are recent — `Intl.Segmenter` (Baseline Apr 2024) and `Intl.DurationFormat` (Baseline Mar 2025): feature-detect them.

### 5.1 The catalog
- `Intl.NumberFormat` — numbers, currency, units, compact notation
- `Intl.DateTimeFormat` — dates/times (uses **system time zone by default** — pass an explicit `timeZone`)
- `Intl.RelativeTimeFormat` — "yesterday" / "in 3 days"
- `Intl.PluralRules` — CLDR plural categories
- `Intl.Collator` — locale-correct sorting
- `Intl.ListFormat` — "A, B, and C" / "A، B، وC"
- `Intl.Segmenter` — grapheme/word/sentence segmentation (essential for CJK/Thai word breaks, emoji-safe length) — *Baseline since Apr 2024; feature-detect for old browsers*
- `Intl.DisplayNames` — localized language/region/currency names (great for switchers)
- `Intl.Locale` — parse/manipulate locale tags · `Intl.DurationFormat` — locale-aware durations, *the newest member (Baseline only since Mar 2025) — feature-detect or polyfill*

### 5.2 Numbers, currency, units
```js
new Intl.NumberFormat('en-US').format(1234.5);          // "1,234.5"
new Intl.NumberFormat('de-DE').format(1234.5);          // "1.234,5"  (. groups, , decimals)
new Intl.NumberFormat('en-US',
  { style: 'currency', currency: 'USD' }).format(99.95); // "$99.95"
new Intl.NumberFormat('ja-JP',
  { style: 'currency', currency: 'JPY' }).format(123457);// "￥123,457" (JPY has NO minor unit → no decimals)
new Intl.NumberFormat('en-US',
  { style: 'unit', unit: 'kilometer-per-hour' }).format(120); // "120 km/h"
new Intl.NumberFormat('en', { notation: 'compact' }).format(12000); // "12K"
```
Let `Intl` decide decimals from the currency (JPY → 0, USD → 2). Never hardcode `toFixed(2)`.

### 5.3 Dates, times, relative time
```js
new Intl.DateTimeFormat('en-US').format(d); // 10/12/2025  (M/D/Y)
new Intl.DateTimeFormat('en-GB').format(d); // 12/10/2025  (D/M/Y)
// "10/12/2025" is Oct 12 in the US but Dec 10 in much of Europe — NEVER ship a raw numeric date string.

new Intl.DateTimeFormat('en-US',
  { dateStyle: 'long', timeZone: 'America/New_York' }).format(d);

const rtf = new Intl.RelativeTimeFormat('en', { numeric: 'auto' });
rtf.format(-1, 'day'); // "yesterday"  (numeric:'auto' yields the idiomatic word)
rtf.format(2,  'day'); // "in 2 days"
```

### 5.4 Collation (sorting)
Default `Array.prototype.sort()` sorts by **code point** and mis-orders accents and naturals. Use `Intl.Collator`:
```js
const c = new Intl.Collator('sv', { numeric: true });
['ö', 'z', 'a'].sort(c.compare);            // Swedish orders ö AFTER z
['file10','file2','file1'].sort(c.compare); // numeric:true → file1, file2, file10
```
The **same array sorts differently** in `en` vs `sv` — never cache a sorted order across locales.

### 5.5 Performance: reuse formatter instances
**Constructing an `Intl.*` instance is the expensive part**, not formatting. Don't call `toLocaleString`/`localeCompare` in a hot loop. Cache by `locale + options`:
```js
const fmtCache = new Map();
function nf(locale, opts) {
  const key = locale + JSON.stringify(opts);
  let f = fmtCache.get(key);
  if (!f) { f = new Intl.NumberFormat(locale, opts); fmtCache.set(key, f); }
  return f;
}
```

## 6. Translatable strings: never concatenate
A translatable unit must be a **whole sentence with named placeholders** so translators can reorder, inflect, pluralize, and gender it for their grammar (word order varies wildly; German verbs land at the end).

```js
// BAD — word order, gender, and plurals are baked into English grammar
const msg = count + ' ' + noun + ' deleted';

// GOOD — one key, named placeholder, ICU MessageFormat
"deleted_items": "{count, plural, one {# item deleted} other {# items deleted}}"
```

### 6.1 ICU MessageFormat — plurals & gender
- **Always include an `other` clause** — it's the required CLDR fallback.
- **Combine gender + count by nesting plural INSIDE select** (gender outside, plural inside):
```
{gender, select,
  male   {{count, plural, one {He has # point}   other {He has # points}}}
  female {{count, plural, one {She has # point}  other {She has # points}}}
  other  {{count, plural, one {They have # point} other {They have # points}}}
}
```
- **Wrap the ICU formatter in try/catch**, returning the raw key (or source string) on error — a malformed string from a translator throws at runtime; a missing label is better than a white screen.

### 6.2 Storage & keys
- One **unique key per unique string** (e.g. `cart.checkout.button`), stored in JSON/XML/YAML/`.po`.
- Attach **translator context notes** (where it appears, character limits, what `{count}` means) — translators have no UI.
- Do **not** reuse one key in two places hoping the translation fits both; a word that's a noun in one spot may be a verb in another.

### 6.3 Plural test matrix
English's one/other model hides categories other languages need (zero/one/two/few/many/other). Test plural logic with CLDR-covering numbers: **0, 1, 2, 3, 5, 11, 21, 22, 100, 101, 1.5** — these exercise Russian, Arabic, Polish, etc.
```js
new Intl.PluralRules('ru').select(21); // "one"  (English would never produce this)
new Intl.PluralRules('ar').select(0);  // "zero"
```

## 7. Language-switcher UX & locale detection

### 7.1 Detection fallback chain (practitioner-validated)
1. **Explicit user choice** — cookie/account setting. **Cookie wins over `Accept-Language` on return visits.**
2. **Browser `Accept-Language` header** — it's a **weighted, ordered list** (`de-DE,de;q=0.9,en;q=0.7`), so walk it by `q`-value and **match best-available, not first-listed**. Use **language-range matching** with **close-match fallback**: `en-CA`/`en-GB` → `en-US`, `de-AT` → `de`; do **not** skip to step 3 just because the exact tag is missing. *(This is the step most sites get wrong.)*
3. **IP-based geo** — **last resort only**.
4. **Site default** — usually English.

- **`Accept-Language` (the user's stated preference) beats IP (their physical location).**
- **IP-only auto-redirect is the cardinal sin.** Physical location ≠ language: a Belgian/Swiss IP maps to a country with 2–4 official languages; a Tokyo IP doesn't mean the user reads Japanese.
- If you nudge by location, use a **dismissible non-modal banner** ("Looks like you're in X — switch to the X site?" with explicit accept/dismiss), **suggest once then stop**, and **persist explicit choice** so manual switching survives. Never silently replace the page.

### 7.2 Switcher design
- **Placement:** top corners (upper-left/right) on desktop — users in both Western and CJK markets look there first. On mobile, **above the fold or in the hamburger** — footers are undiscoverable on phones, and key markets (China, India, SE Asia) are mobile-majority, so footer-only placement strands them.
- **Label with endonyms:** `Deutsch` (not "German"), `中文` (not "Chinese"). Add the country for regional variants: `Español / España`, `English (United States)`. **Never** use 2-letter codes (`EN`/`DE`) as the visible label — browser auto-translate mangles them and they're cryptic.
- **Flags are for countries, never languages.** No single flag represents French (France/Canada/Belgium/Switzerland/many African nations), Arabic, or Spanish. Use a **globe or translate icon** for the language menu. A lone small flag is missed — combine signals (language name + currency + country).
- **Pick the UI by scale:** 3–5 languages → dropdown · 10–15 → non-modal dialog with autocomplete · 20+ countries → dedicated page with tabs/accordions.
- **Always:** keyboard-accessible, current selection clearly marked, English fallback present.
- **SEO:** emit `<link rel="alternate" hreflang="…">` for each locale variant and an `x-default`; the switcher should link to real per-locale URLs, not just toggle client state.

## 8. QA: pseudo-localization & translation-safe components
**Pseudo-localize to find bugs *before* real translations exist.** It replaces source strings with bracketed, accented, expanded versions:
```
"Hello World!"  →  "[Ḥéllö Wörld!! Жuдва]"
```
This single technique surfaces:
- **Truncation / overflow** — the expansion (~30–50% added) reveals tight buttons/nav.
- **Hardcoded strings** — anything *not* bracketed wasn't externalized.
- **Missing font glyphs** — `□`/tofu boxes expose incomplete Unicode coverage.
- **Concatenation bugs** — split sentences look obviously broken.
- **Brackets** at both ends make clipped edges visible at a glance.

Run it in **CI** and during dev, not just before launch. Many TMS and frameworks build it in.

**Translation-safe component checklist:** no concatenated strings · placeholders are named, not positional-and-reordered-by-code · plural- and gender-aware via ICU · `lang` set correctly for hyphenation/`Intl`/voice · room for 2× short strings · no text baked into images · numbers/dates/currency formatted at render, never stored pre-formatted.

## Concrete specs & numbers
- German: **~35% avg expansion**, up to **+70%** on short strings (compound nouns).
- Short single words/abbreviations can expand **100–300%** → design for **~2× (200%)** English length.
- Finnish ~30–40% · Russian/Polish ~15–35% · Spanish ~25% · Portuguese 15–30% · French 15–20% · Italian 10–25%.
- CJK **−10% to −55%** character-count contraction, but needs more vertical space and `line-height`.
- `Intl` (ECMA-402): **~95%** global browser coverage; no locale data shipped over the wire.
- CSS logical properties: all modern browsers; only retired IE unsupported.
- JPY has no minor unit → `Intl.NumberFormat` renders `￥123,457` (0 decimals) vs USD `$99.95`.
- Date ambiguity: `10/12/2025` = Oct 12 (US) / Dec 10 (much of Europe). Decimal separator `.` (en) vs `,` (much of EU). Digit grouping = 3 in English, but **4 or 2** elsewhere.
- CLDR plural test set: **0, 1, 2, 3, 5, 11, 21, 22, 100, 101, 1.5**.
- Mobile-majority markets (China, India, SE Asia) → switcher above the fold on mobile, never footer-only.
- `Intl.Segmenter` Baseline since Apr 2024 · `Intl.DurationFormat` Baseline since Mar 2025 — feature-detect both.
- `:dir()` matches **computed** direction (incl. inherited/auto); `[dir=rtl]` matches **explicit attribute only** — **not equivalent.**

## Tools & libraries
- **`Intl` / ECMA-402** (native): `NumberFormat`, `DateTimeFormat`, `RelativeTimeFormat`, `PluralRules`, `Collator`, `Segmenter`, `DisplayNames`, `ListFormat`, `DurationFormat`, `Locale`.
- **ICU MessageFormat** — standard for plural/select(gender) in one translatable string; supported by Lingui, FormatJS/`react-intl`, and most TMS.
- **CSS / HTML primitives:** logical properties (`margin-inline`, `inset-inline-start`, `padding-block`), `:dir()`, `dir` attribute, `dir="auto"`, `<bdi>`, `<bdo>`, `unicode-bidi: isolate`, LRM/RLM.
- **Framework i18n:** `next-intl`, `react-intl`/FormatJS, Lingui, `gettext` (`.po`), `vue-i18n`, `angular/localize`.
- **Translation Management Systems:** Crowdin, Lokalise, Phrase, POEditor, SimpleLocalize, Weglot.
- **Pseudo-localization:** built into many TMS/frameworks; produces `[Ḥéllö Wörld!!!]`.
- **CI QA:** LocalePass (open-source CLI + GitHub Action, `github.com/CodingRasi/LocalePass`) — detects untranslated strings, text overflow/clipping, and missing `html[lang]`; does screenshot visual-regression and emits SARIF for PR annotations.
- **Data:** Unicode **CLDR** underlies `Intl` plurals, collation, and formatting.

## Do / Don't
- **Do** lay out in German + Arabic before sign-off. **Don't** design only in English and "add RTL later."
- **Do** use `min-width`/`min-height`/`max-content`. **Don't** set fixed `width`/`height` on translatable elements.
- **Do** budget short strings at ~2×. **Don't** apply one flat % to every string.
- **Do** use logical properties + set `dir`. **Don't** use `margin-left`/`text-align:left` or forget `dir="rtl"`.
- **Do** store raw values and format with `Intl` at render. **Don't** store `"$1,234.56"` or pre-formatted dates.
- **Do** write whole sentences with named placeholders. **Don't** concatenate `count + " items"`.
- **Do** prefer `Accept-Language`, redirect once, persist explicit choice. **Don't** IP-only auto-redirect or trap users.
- **Do** label with endonyms + globe icon. **Don't** use `EN`/`DE` codes or a flag for a language.
- **Do** reuse cached `Intl` instances. **Don't** construct a formatter per row in a loop.
- **Do** test plurals with the CLDR number set and include an `other` clause. **Don't** assume English's one/other covers Russian/Arabic/Polish.

## Common pitfalls & how to avoid them
- **Truncation in German/Finnish** — caused by hardcoded sizes or `overflow:hidden`/`white-space:nowrap`. → content-driven sizing; pseudo-localize in CI.
- **Flat expansion budget** — paragraph-tuned 35% breaks buttons. → tier the budget (body 35%, short strings ~2×).
- **Bolting on RTL** — duplicated stylesheets and physical-property bugs. → logical properties + `dir` from day one.
- **Mixed-locale display bugs** — pre-formatted values stored in DB then shown to a different locale. → store data, format at boundary.
- **Concatenation** — translators can't fix word order/plurals/gender. → ICU MessageFormat, named placeholders.
- **IP-only redirect** — sends a German speaker in France to French, with no easy way back. → fallback chain + dismissible banner + persisted choice.
- **Flags for languages** — ambiguous and politically loaded. → globe/translate icon + endonym.
- **Default `sort()`** — code-point order mis-sorts accents and `file2`/`file10`. → `Intl.Collator` with `{numeric:true}`.
- **System time zone surprises** — `DateTimeFormat` defaults to the server/user TZ. → pass an explicit `timeZone`.
- **Missing `other` plural clause** — runtime throw. → always include `other`; wrap ICU in try/catch.
- **2-letter language codes as labels** — browser auto-translate corrupts them. → full endonym names.

## What real users complain about
- "It detected my IP and forced the local language — and there's **no way to switch back**." (auto-redirect trap)
- "The buttons are **cut off** in German / the menu **wrapped to two lines** and looks broken." (expansion overflow)
- "Picking German shipping **changed my whole site language** to German." (bundled axes)
- "I see **□□□ boxes** instead of characters." (missing glyph coverage)
- "The date says `03/04` — is that **March 4 or April 3**?" (unformatted ambiguous dates)
- "The Arabic site has the **logo flipped** and the **back arrow points the wrong way**." (over/under-mirroring)
- "I'm in Switzerland; it assumed French, but **I read German**." (location ≠ language)
- "I can't find the language switcher — it's **buried in the footer** on my phone." (mobile placement)

## Senior-level checklist
- [ ] Strings externalized to keys (JSON/`.po`/YAML) with translator context notes; **zero hardcoded UI text**.
- [ ] All directional CSS uses **logical properties**; `text-align: start`, not `left`.
- [ ] `<html lang dir>` set; `dir="auto"` on user-text inputs; RTL verified in Arabic, not just emulated.
- [ ] Correct things mirror (nav, arrows, OK/Cancel); logos/media-controls/charts/numbers/code **do not**.
- [ ] Mixed-direction runs isolated with `<bdi>`/`unicode-bidi:isolate`; bidi handled in **markup**, not CSS.
- [ ] No fixed widths/heights on translatable elements; **min-* + max-content**; nav survives German.
- [ ] Expansion budget tiered (body ~35%, short ~2×); pseudo-localization passes in CI.
- [ ] All numbers/dates/currency/units/relative-time/lists formatted with **`Intl`** at render; raw values stored.
- [ ] `Intl` formatter instances **cached and reused**; explicit `timeZone` passed to `DateTimeFormat`.
- [ ] Sorting uses **`Intl.Collator`** with `{numeric:true}`.
- [ ] All plural strings are **ICU MessageFormat** with an `other` clause; gender nests plural inside select; tested on the CLDR number set.
- [ ] ICU formatter wrapped in try/catch with raw-key fallback.
- [ ] Language, region, and currency are **independent** settings.
- [ ] Switcher present, in top corner (desktop) / above-fold (mobile), endonym labels, globe icon, keyboard-accessible, current selection marked.
- [ ] Detection follows the fallback chain; **no IP-only redirect**; explicit choice persisted (cookie wins on return).
- [ ] `hreflang` alternates (+ `x-default`) emitted; switcher links to real per-locale URLs.
- [ ] Correct `lang` per language so `hyphens:auto`, `Intl`, and assistive tech behave.

## Sources
- MDN — `Intl` object: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl
- MDN — CSS logical properties and values: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values
- MDN — `:dir()` pseudo-class: https://developer.mozilla.org/en-US/docs/Web/CSS/:dir
- MDN — `<bdi>` element: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/bdi
- W3C — Inline markup and bidirectional text in HTML: https://www.w3.org/International/articles/inline-bidi-markup/
- W3C i18n — Structural markup and right-to-left text in HTML: https://www.w3.org/International/questions/qa-html-dir
- Unicode CLDR: https://cldr.unicode.org/
- ECMA-402 (Internationalization API): https://tc39.es/ecma402/
- Nielsen Norman Group — Localization: https://www.nngroup.com/topics/localization/

# Multilingual & Non-Latin Typography

> How to set type across writing systems beyond Latin — CJK, Arabic, Devanagari, Thai, Hebrew, Cyrillic, Greek — without "tofu", broken shaping, multi-MB font downloads, or interline collisions. Read this to make defensible per-script decisions about font stacks, fallback chains, leading, line-breaking, RTL mirroring, and `unicode-range` delivery architecture, instead of pasting Latin defaults onto scripts they were never designed for.

**Apply when:** any site that renders, or might one day render (CMS, UGC, i18n), text in a non-Latin script. · **Related:** [Web Typography Systems](typography-systems.md) · [Internationalization (i18n) & Localization (l10n)](../01-ux-fundamentals/internationalization-and-localization.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md)

## TL;DR — rules at a glance

- **Tune leading per script. Latin's 1.2/1.5 does NOT transfer.** Target CJK ≈ 1.7, Arabic ≥ 1.8, Devanagari & other tall-stacking scripts 1.4–1.6 (never below ~1.3), Latin body ≈ 1.5. Higher information density or stacking glyphs demand more leading.
- **Set Arabic text 10–15% larger than Latin** for equal readability (16px Latin → ~18px Arabic). Same for many Indic scripts at small sizes.
- **`unicode-range` slicing is delivery ARCHITECTURE for CJK, not an optimization.** You physically cannot ship 20,000+ glyphs in one practical download. For Latin-only it's a 90% size win; for CJK it's the only way to ship at all.
- **Order the fallback chain: script-specific font → broad system font (`system-ui`, `-apple-system`, "Segoe UI", "Noto Sans") → generic (`sans-serif`).** Every codepoint must resolve to *something* so you never render □□□ tofu.
- **Put the Latin font BEFORE the CJK font** (`"Noto Sans", "Noto Sans CJK JP", sans-serif`) so Latin glyphs come from your nice Latin font and Han falls through to the CJK font — one visual family, no scrunched Latin.
- **Prefer `"Noto Sans CJK {JP,KR,SC,TC}"` over `"Noto Sans {JP,KR,SC,TC}"`** and pick the variant whose *default* language matches your audience (Han unification means the same codepoint draws differently per locale).
- **Line-breaking is per-script and driven by `lang`.** Set `lang` on `<html>`. CJK breaks between any character; Korean wants `word-break: keep-all`; Thai/Lao/Khmer need dictionary breaking; Latin breaks at spaces/hyphens.
- **RTL = full mirroring, not right-aligned LTR.** Set `dir="rtl"` on the element AND use CSS logical properties (`margin-inline`, `inset-inline-end`, `padding-inline-start`). CSS alone is not enough.
- **Arabic ligatures and contextual forms are MANDATORY, not decoration.** Never `font-feature-settings: "liga" 0` on Arabic; the shaping system *is* ligature behavior.
- **Scope `word-break`/`overflow-wrap` with `:lang()`. Never globally in a reset** — it shatters button labels and predictable layouts.
- **`overflow-wrap: break-word` is the URL safety net; `word-break` is the CJK inter-character rule.** They are not interchangeable.
- **Don't `preload` non-Latin subsets — `preload` ignores `unicode-range`** and force-downloads the heavy slice on every page. Preload only the critical Latin subset.
- **Self-hosting Google Fonts naively LOSES the automatic slicing** — you must re-slice (e.g. `cn-font-split`) or you ship the whole multi-MB file.
- **Subset complex scripts carefully:** stripping glyphs can delete GSUB/GPOS shaping tables. Test subsetted Arabic/Indic/Thai with real content before shipping.
- **For CJK+Latin spacing use `text-autospace`/`text-spacing-trim`, not hand-typed spaces** — auto in Chrome/Edge (133/139) and Safari 18.4, unimplemented in Firefox; gate with `@supports`.

---

## 1. The mental model: scripts are not Latin with extra characters

Three structural assumptions baked into Latin web typography are **false** for other scripts:

1. **"Glyphs sit on a baseline with an x-height."** CJK has **no x-height** and no baseline-aligned proportional system. Arabic and Indic glyphs hang off and stack around a baseline with marks above and below.
2. **"Text breaks at spaces."** CJK, Thai, Lao, Khmer, Japanese, and Burmese have **no inter-word spaces**. Breaking is per-character (CJK) or dictionary-driven (Thai et al.).
3. **"A font covers its script in a small file."** A "complete" Latin face is ~200–300 glyphs / 15–50KB WOFF2. A Chinese face is **20,000+ glyphs / 5–20MB**.

Treat each script as a system with its own metrics, breaking rules, and shaping requirements. The `lang` attribute is the switch that tells the browser which system to use — **set it or the browser guesses.**

### The CJK em-square (virtual body) model

Every CJK character occupies a fixed **full-width square** — identical height *and* width. There is no proportional advance, no x-height, no ascender/descender system like Latin. Consequences:

- **Latin glyphs inside a CJK font are "half-width"** = exactly half a CJK em. This is *by design*, and it's why Latin inside Chinese/Japanese fonts looks thin, cramped, and monospaced.
- Width OpenType features expose alternates: `fwid` (full-width), `hwid` (half-width), `pwid` (proportional width). `pwid` can give nicer proportional Latin from the same font — but support is uneven; the font-stack override (§3) is more reliable.
- Vertical writing (`writing-mode: vertical-rl`) rotates the em-squares; punctuation has dedicated vertical forms via `vert`.

---

## 2. Per-script vertical metrics & leading

Never assume a universal `line-height`. Pick per script, scope with `:lang()`.

| Script | Body `line-height` | Size vs Latin | Why |
|---|---|---|---|
| Latin / Cyrillic / Greek | 1.5 (1.4–1.6) | baseline | reference |
| CJK (中文 / 日本語 / 한국어) | **1.7** (1.6–1.8) | same | high information density per character; a 10pt CJK font wants ~17pt leading |
| Arabic (العربية) | **≥ 1.8** | **+10–15%** | clear tashkeel (diacritics) above/below; dense connected strokes |
| Hebrew (עברית) | 1.5–1.7 | +5–10% | niqqud (vowel points) when present |
| Devanagari (देवनागरी) | **> 1.3** (often 1.4–1.6) | +5–10% | tall stacking conjuncts collide between lines at 1.2 |
| Thai (ไทย) / Lao / Khmer | 1.6–1.8 | +5–10% | stacked vowel/tone marks reach far above and below |
| Vietnamese (Latin + 2 stacked marks) | 1.5–1.6 | same | base letter + diacritic + tone mark stack vertically |

```css
:root { line-height: 1.5; }              /* Latin/Cyrillic/Greek default */
:lang(ja), :lang(zh), :lang(ko) { line-height: 1.7; }
:lang(ar) { line-height: 1.8; font-size: 1.125em; }   /* +12.5% */
:lang(hi), :lang(mr), :lang(ne) { line-height: 1.45; }
:lang(th), :lang(lo), :lang(km) { line-height: 1.7; }
:lang(vi) { line-height: 1.55; }
```

- **Cyrillic & Greek behave like Latin** structurally (baseline + x-height) — the same scale and leading work. The risk is *coverage*, not metrics: confirm your web font actually includes the Cyrillic/Greek blocks (many "Latin Extended" subsets stop short of full Cyrillic).
- **Devanagari's shirorekha (headline bar)** plus deep conjunct stacks means the *sum* of vertical metrics can legitimately exceed 130% of the em. Don't fight it with a tight `line-height` — glyphs will overlap between lines.
- **Don't over-set Latin/Cyrillic to 1.7** just because the page also has CJK — wrap the rule in `:lang()`.

---

## 3. Font stacks & fallback chains

### The golden rule of ordering

Layer **script-specific font(s) first**, then **broad-coverage system fonts**, then **generic**. The chain must guarantee *some* glyph for every codepoint a user might enter.

```css
font-family:
  "Inter",                       /* your nice Latin */
  "Noto Sans CJK JP",            /* CJK fall-through */
  "Noto Sans Arabic",            /* RTL fall-through (if the page mixes) */
  system-ui, -apple-system, "Segoe UI", "Noto Sans",  /* broad system safety */
  sans-serif;                    /* last-ditch generic */
```

How browser font matching works: for **each character**, the browser walks the stack left→right and uses the **first font that contains that glyph**. So Latin "Hello" resolves to Inter, "こんにちは" skips Inter (no Han) and lands on Noto Sans CJK JP. This per-character matching is the whole trick.

### The Latin-before-CJK technique (avoid scrunched Latin)

Because Latin glyphs inside CJK fonts are half-width and ugly (sourced from Adobe Source Sans Pro / Source Han, drawn to fill half an em), put a **Latin-only font first** so Latin renders from it and only Han falls through.

Real-world example — **Mozilla's Firefox sites** (Protocol design system) use a stack of the form `Metropolis, Inter, X-LocaleSpecific, sans-serif` (where `X-LocaleSpecific` is a build-time placeholder Mozilla substitutes with the locale's CJK font). Neither Metropolis nor Inter contains Chinese, so Latin comes from those two and Chinese falls through to the locale CJK font — nice Latin **and** nice Han in one visual family.

### Noto super-family specifics

- **"No Tofu"** is the name's literal meaning. 100+ families, ~1,000 languages, 162 writing systems (Nov 2024); 166 of 172 Unicode scripts by Nov 2025 (post-Unicode 17.0).
- **Use the `CJK`-suffixed combined files:** `"Noto Sans CJK JP"` not `"Noto Sans JP"`. Each of JP/KR/SC/TC covers **all four** CJK languages but ships a different **default Han shape** — pick the variant matching your audience because Han unification draws e.g. 直/骨/今 differently per locale.
- **Put `"Noto Sans"` BEFORE `"Noto Sans CJK …"`** so the Latin in your body text comes from real Noto Sans (a proper proportional Latin), not the half-width Source-derived Latin inside the CJK file. Same principle as §3.2 with a single family.
- Put the **script you care about most FIRST** — Noto families overlap, and order decides ties.

### Platform system stacks per script (zero-download fallback)

| Script | Reasonable system stack |
|---|---|
| Latin/Cyrillic/Greek | `system-ui, -apple-system, "Segoe UI", Roboto, sans-serif` |
| Japanese | `"Hiragino Sans", "Yu Gothic", Meiryo, "Noto Sans CJK JP", sans-serif` |
| Simplified Chinese | `"PingFang SC", "Microsoft YaHei", "Noto Sans CJK SC", sans-serif` |
| Traditional Chinese | `"PingFang TC", "Microsoft JhengHei", "Noto Sans CJK TC", sans-serif` |
| Korean | `"Apple SD Gothic Neo", "Malgun Gothic", "Noto Sans CJK KR", sans-serif` |
| Arabic | `system-ui, "Segoe UI", "Noto Naskh Arabic", "Geeza Pro", sans-serif` |
| Hebrew | `system-ui, "Segoe UI", "Noto Sans Hebrew", Arial, sans-serif` |
| Thai | `"Thonburi", "Leelawadee UI", "Noto Sans Thai", sans-serif` |
| Devanagari | `"Kohinoor Devanagari", "Nirmala UI", "Noto Sans Devanagari", sans-serif` |

> On Apple platforms, `system-ui`/`-apple-system` auto-substitutes the right SF script extension (SF Arabic, SF Hebrew, etc.) for the content language — prefer that over naming `"SF Arabic"` directly, since the literal family name is unreliable (the OS uses internal names like `SFNS`/`SFUI`).
>
> Web fonts load from the **OS**, not the browser. "Web-safe" assumptions silently fall back to *different* fonts cross-platform. Old system CJK fonts (SimSun 2001, Batang/Gulim mid-90s) carry embedded bitmaps and render fragile, thin Latin — another reason to lead with your own Latin font.

---

## 4. Payload & `unicode-range` subsetting (the CJK delivery problem)

### Why the calculus inverts

- **Latin:** the full character set wastes 60–80% of bandwidth, so subsetting is an *optimization* — nice but optional.
- **CJK:** you **cannot** ship the full set in one practical download (5–20MB). `unicode-range` slicing is the **delivery architecture**, period. Even a 5,000-character "subset" is still 300+KB.

### How `unicode-range` works

The `unicode-range` descriptor on `@font-face` gates the download: the browser fetches the slice **only if the page contains a codepoint in that range**.

```css
/* Latin core — small, can preload */
@font-face {
  font-family: "MyApp"; src: url(/f/latin.woff2) format("woff2");
  unicode-range: U+0000-00FF, U+0131, U+2000-206F; font-display: swap;
}
/* One CJK frequency band of ~100 — only fetched if used */
@font-face {
  font-family: "MyApp"; src: url(/f/cjk-007.woff2) format("woff2");
  unicode-range: U+4E00-4E0F, U+4E11, U+4E13 /* …frequency-banded set… */;
  font-display: swap;
}
```

Google Fonts' production delivery is the canonical model: each CJK family (e.g. Noto Sans SC/TC) is split into **~100 chunks** (cited as ~100–200 slices), each band grouping characters that *co-occur* by frequency, each with its own `unicode-range`. A typical CJK page with ~500 unique characters then loads **~3–8 small files (~50–80KB each)** instead of one ~20MB monolith — it downloads data for ~500 characters, not ~22,000.

### Self-hosting trap

Naively copying Google Fonts CSS + WOFF2 files **loses the automatic slicing** if you flatten it — you must keep the `unicode-range` `@font-face` blocks intact or re-generate them. Use **`cn-font-split`** to auto-slice a Chinese font into many `unicode-range` files (replicates Google's behavior for self-hosting).

### Subsetting rules

- **`pyftsubset`** (from `fonttools`) is the standard subsetter — the same toolchain Google/Adobe/foundries use.
- **Always convert to WOFF2** after subsetting — Brotli adds ~20–30% over the raw subset. Subset + WOFF2 can cut script font size ~90%.
- **Keep shaping tables for complex scripts.** When subsetting Arabic/Hebrew/Thai/Indic, **do NOT strip GSUB/GPOS** (`pyftsubset --layout-features='*'` and keep the script's features). Aggressive stripping deletes shaping/positioning and breaks rendering even as the file shrinks.
- **`preload` ignores `unicode-range`.** Preloading a CJK/Arabic slice forces its download on **every page**, including pages with zero matching characters — a classic multilingual perf regression. **Reserve `preload` for the critical Latin subset only.**
- **Never static-subset a CMS/UGC site to Latin.** You can't predict which characters users type; you'll render tofu the moment someone posts in another script. Use `unicode-range` splitting instead of static subsets.
- **`Glyphhanger` / `Subfont`** scan real page usage to compute which ranges you actually need (great for *static* marketing pages, dangerous for dynamic content).

---

## 5. Line-breaking & wrapping per script

Soft-wrap opportunities are selected from the `lang` attribute. **Set `lang` correctly on `<html>` (and on inline spans that switch language).**

| Script | Break model | CSS |
|---|---|---|
| Latin/Cyrillic/Greek | at spaces & hyphens | default; `hyphens: auto` |
| Chinese / Japanese | between **any** character (default) | usually nothing; `line-break: strict` for punctuation rules |
| Korean | by syllable **or** whole word | `word-break: keep-all` for word wrapping |
| Thai / Lao / Khmer | **dictionary**-based word breaking | rely on browser ICU; set `lang` |
| Arabic / Hebrew | at spaces (RTL) | default; ensure `dir="rtl"` |

### Chinese / Japanese

Character-level breaking is **already the default** — they generally need **no** `word-break`. `word-break: break-all` is only for predominantly-CJK text with a few Latin snippets, to keep lines balanced. `line-break` controls punctuation tightness:

```css
:lang(ja), :lang(zh) { line-break: strict; }   /* tighter kinsoku */
/* line-break: loose  → lenient, newspaper-style narrow columns */
```

**Kinsoku / line-composition (禁則処理):** opening punctuation (「（) must **not end** a line; closing punctuation and small marks (」）、。) must **not begin** a line. `line-break: strict` enforces tighter CJK punctuation rules; `loose` is lenient for narrow columns.

### Korean

```css
:lang(ko) { word-break: keep-all; }   /* wrap whole words, ragged-right */
```

Praised for readability, **but** long URLs/strings in narrow columns or table cells **overflow** because breaks are forbidden mid-word. Pair it with a scoped safety net:

```css
:lang(ko) .cell, :lang(ko) .url { overflow-wrap: break-word; }
```

### Thai / Lao / Khmer

No spaces; correct breaking needs a **dictionary** (ICU, shipped in Chrome/Firefox/Safari). Just set `lang="th"` and let the engine break. **Do NOT** hand-sprinkle `U+200B` ZERO WIDTH SPACE at scale — it's invisible, easily mis-placed or duplicated, and modern dictionary breaking makes it largely unnecessary.

### `word-break` vs `overflow-wrap` — don't conflate

- **`overflow-wrap: break-word`** (or `anywhere`) — the **safety net** for long unbreakable strings (URLs, hashes) to stop overflow. Breaks only when a word can't fit.
- **`word-break: break-all`** — breaks at the **exact** overflow point even when moving the whole word to the next line would have avoided the break. Surprising, ugly results; use sparingly.
- **`word-break: keep-all`** — forbids breaking inside words (CJK use).

```css
/* Global URL safety — fine to apply broadly */
.prose a, .prose code { overflow-wrap: break-word; }
/* WRONG: never put this in a reset — shatters buttons & labels */
/* * { word-break: break-all; } */
```

### `text-wrap` interactions

- `text-wrap: balance` (headings) and `text-wrap: pretty` (body) work across scripts but are **Latin-tuned** for line-fill.
- **Don't combine `text-wrap: balance` with `overflow-wrap: anywhere`** — it causes unwanted wrapping. Keep `anywhere` scoped to URL-bearing elements only.
- For CJK *inter-script spacing* (not wrapping), reach for `text-autospace`/`text-spacing-trim` — see §8.

---

## 6. Arabic: contextual shaping & ligatures (mandatory)

Arabic is **cursive and connected**. Each letter has up to **4 contextual forms** chosen by position. Shaping is **required for correct language**, not a typographic flourish.

| Joining class | Forms | Examples |
|---|---|---|
| Dual-joining | all 4 (isolated, initial, medial, final) | ب ت ن ي |
| Right-joining | no medial (joins only to preceding) | ا د ذ ر ز و |
| Non-joining | isolated only | (spaces, some symbols) |
| Transparent | marks; don't affect joining | tashkeel/diacritics |

- **Non-connecting letters (ا د ذ ر ز و)** don't join to the *following* letter, creating natural breaks **within** a word — normal and correct.
- **Lam-alef (لام + ألف → لا)** is the **only truly mandatory ligature** in modern Arabic; the rest of shaping is contextual form selection, also mandatory.

```css
:lang(ar), [dir="rtl"] {
  font-feature-settings: "liga" 1, "rlig" 1, "calt" 1;  /* required + contextual */
}
/* NEVER do this on Arabic — it breaks the language visually: */
/* font-feature-settings: "liga" 0; */
```

- **Shaping engine:** Chrome/Firefox use **HarfBuzz**; Safari uses Apple Core Text. Both handle Arabic joining, Indic reordering, Mongolian vertical, etc.
- **Recommended Arabic web fonts** (good contextual forms, permissive licenses): **Noto Naskh Arabic, Cairo, Almarai, IBM Plex Sans Arabic, Tajawal.**
- **Never author with Arabic Presentation Forms** (U+FB50–FDFF, U+FE70–FEFF) — those exist only for legacy-encoding round-tripping. Author plain Arabic codepoints and let shaping run. Likewise never insert `U+202E` RTL override into content.
- **Mismatched Latin fallback** ruins mixed Arabic/Latin lines — if the Arabic font lacks Latin and you fall through to a Latin font with a different x-height, baselines and sizes visibly clash. Prefer an Arabic font with decent Latin (IBM Plex Sans Arabic, Cairo) or carefully match metrics.

---

## 7. RTL: full mirroring, not right-alignment

RTL is **layout mirroring**, not "right-aligned LTR." Do **both**:

1. **`dir="rtl"`** on the root (or the localized subtree). This drives text direction, bidi resolution, and shaping.
2. **CSS logical properties** so the whole layout flips automatically with direction.

```css
/* WRONG — physical properties pin things to the wrong side in RTL */
.card { margin-left: 16px; padding-right: 24px; left: 0; text-align: left; }

/* RIGHT — logical properties mirror with dir */
.card {
  margin-inline-start: 16px;
  padding-inline-end: 24px;
  inset-inline-start: 0;
  text-align: start;
}
```

- Use logical everywhere: `margin-inline`, `padding-inline-{start,end}`, `inset-inline-{start,end}`, `border-inline`, `text-align: start/end`, `float: inline-start`.
- **Mirror directional icons** (back/forward arrows, chevrons, progress, breadcrumbs) — but **not** non-directional ones (search, settings, play/media): `[dir="rtl"] .icon-chevron { transform: scaleX(-1); }`.
- **Numbers and embedded Latin/URLs stay LTR** inside RTL text — the Unicode Bidi Algorithm handles this if `dir` is set; wrap ambiguous runs in `<bdi>` or `dir="auto"`.
- Hebrew and Arabic are both RTL; the mirroring rules are identical — the difference is shaping (Hebrew is non-cursive, no contextual forms).

---

## 8. Mixing scripts cleanly in one layout

Maintain **one shared layout** and swap fonts/metrics per script with `:lang()` — don't fork stylesheets.

```css
/* Shared base */
:root { font-family: "Inter", system-ui, sans-serif; line-height: 1.5; }

/* Per-script overrides, single sheet */
:lang(ja) { font-family: "Inter", "Noto Sans CJK JP", sans-serif; line-height: 1.7; }
:lang(ar) {
  font-family: "IBM Plex Sans Arabic", "Noto Naskh Arabic", sans-serif;
  line-height: 1.8; font-size: 1.125em;
}
:lang(hi) { font-family: "Inter", "Noto Sans Devanagari", sans-serif; line-height: 1.45; }
:lang(ko) { font-family: "Inter", "Noto Sans CJK KR", sans-serif; word-break: keep-all; }
```

```html
<html lang="ar" dir="rtl">
  …
  <p>تطبيق رائع <span lang="en" dir="ltr">SuperApp v2.1</span> متاح الآن.</p>
```

- **Annotate inline language switches** with `lang` (and `dir` when direction flips) so breaking, shaping, and font selection are correct per run.
- **Match optical weight/size across scripts** so a mixed headline reads as one family — Arabic/CJK often need a notch more size to feel equal to the Latin.
- Keep the **type scale shared** but allow per-script size multipliers (Arabic +12%, see §2).

### CJK–Latin spacing: `text-autospace` & `text-spacing-trim`

When CJK runs against Latin/numerals with no space (中文English数字), pro typography inserts a thin gap (the "四分space" / quarter-em). Don't hand-type spaces — use the CSS Text 4 properties (2026 baseline):

- **`text-autospace: normal`** inserts thin spacing between ideographs and adjacent Latin letters/numbers. Shipped in **Chrome/Edge 139** (desktop+Android) and **Safari 18.4**; `normal` is the initial value there, so the spacing appears automatically. It does **not** add a gap where an explicit space already exists, so legacy content is safe.
- **`text-spacing-trim`** trims the excessive built-in side-bearing of CJK punctuation (e.g. consecutive 」） or 、。) per JLREQ/CLREQ. Shipped and on-by-default in **Chrome/Edge since 133** (use `normal`/`space-first`; set `space-all` to disable trimming).
- **Firefox does NOT implement either** (as of mid-2026). Treat both as progressive enhancement — gate with `@supports (text-autospace: normal)` and never make layout depend on the inserted gap.
- A `text-spacing` shorthand sets both. For autospace only `normal` / `no-autospace` are broadly implemented; the spec's finer values are not yet in any engine.

---

## Concrete specs & numbers

- **Glyph counts / file sizes:** full Latin set ~200–300 glyphs / 15–50KB WOFF2. Standard Chinese font **> 20,000 glyphs**. Korean Hangul alone = **11,172** possible syllable blocks. Unoptimized CJK font file = **5–20MB**.
- **Slicing:** Google Fonts splits each CJK family into **~100 chunks** (cited ~100–200 slices), frequency-banded, each with its own `unicode-range`.
- **Page reality:** a CJK page with ~500 unique characters loads **~3–8 files (~50–80KB each)** vs one ~20MB file — data for ~500 chars, not ~22,000. A 5,000-char "subset" is still **300+KB**, which is why `unicode-range` slicing beats static subsetting for CJK.
- **Compression:** subset + WOFF2 ≈ **90%** reduction; WOFF2 Brotli adds **~20–30%** over the raw subset.
- **Leading:** Latin default ~120% (1.2), Latin body ~1.5; **CJK ~1.7; Arabic ≥ 1.8; Devanagari/tall scripts may exceed ~1.3 (often 1.4–1.6).**
- **Size:** Arabic body **+10–15%** vs Latin for equal readability (16px → ~18px).
- **Arabic forms:** up to **4 contextual forms** (isolated/initial/medial/final). **Lam-alef (لا)** = the only truly mandatory ligature. Non-connecting letters: **ا د ذ ر ز و**.
- **Noto:** 100+ families, ~1,000 languages, 162 writing systems (Nov 2024); 166 of 172 scripts (Nov 2025, post-Unicode 17.0). Name = "No Tofu".
- **Adobe "Super" OTC** (Noto + Source CJK combined) is only **~200KB** larger than a Source-only file because OTCs reuse glyphs.
- **Windows 11** added Noto CJK as a system fallback in the **March 2025 preview (KB5053656)**; this introduced a **blurry-CJK-at-96 DPI (100% scaling)** regression in Chromium browsers (Edge/Chrome) that was still an open known issue as of the **June 11, 2025 out-of-band update (KB5063060)** — Microsoft's only mitigation is to raise display scaling to 125–150%, or set a non-Noto CJK font in browser settings. Don't rely on the OS fallback; specify your own CJK font.

---

## Tools & libraries

- **`pyftsubset`** (part of `fonttools`) — standard glyph subsetter; keep `--layout-features='*'` for complex scripts.
- **`cn-font-split`** — auto-slices a Chinese font into many `unicode-range` subset files; replicates Google's slicing for self-hosting.
- **Glyphhanger** — scans pages to determine the glyphs / `unicode-range`s actually used.
- **Subfont** — automated subsetting based on real page usage.
- **HarfBuzz** — OpenType shaping engine in Chrome/Firefox (Arabic joining, Indic reordering, Mongolian vertical). Safari uses Apple Core Text.
- **ICU** — Unicode/line-breaking library; dictionary-based word breaking for Thai/Lao/Khmer (built into modern browsers).
- **`unicode-range` `@font-face` descriptor** — gates font download to pages containing matching codepoints; the core CJK delivery mechanism.
- **OpenType width features** `fwid` / `hwid` / `pwid` — expose full-width, half-width, and proportional Latin from one CJK font.
- **Recommended Arabic fonts:** Noto Naskh Arabic, Cairo, Almarai, IBM Plex Sans Arabic, Tajawal.
- **Noto Sans CJK {JP, KR, SC, TC}** — each covers all four CJK languages with a different default; Latin/numerals from Adobe Source Pro.

---

## Do / Don't

**Do**
- Tune `line-height` and size per script with `:lang()`; treat 1.7 (CJK) / 1.8 (Arabic) / 1.4–1.6 (Devanagari) as starting points.
- Put your nice Latin font *before* the CJK/Arabic font so Latin renders well and only the script falls through.
- Ship CJK via `unicode-range` slices; convert every subset to WOFF2.
- Set `lang` on `<html>` and on inline language switches; set `dir="rtl"` for RTL subtrees.
- Use CSS logical properties throughout so RTL mirrors automatically.
- Keep GSUB/GPOS shaping tables when subsetting Arabic/Indic/Thai; test with real content.
- Scope `word-break: keep-all` (Korean) and `overflow-wrap: break-word` (URLs) with `:lang()`/selectors.

**Don't**
- Don't reuse Latin's 1.2/1.5 leading on CJK/Arabic/Devanagari.
- Don't `preload` a non-Latin subset — `preload` ignores `unicode-range`.
- Don't static-subset a CMS/UGC site to Latin.
- Don't set `"liga" 0` (or strip required ligatures) on Arabic.
- Don't put `word-break`/`break-word` in a global reset.
- Don't treat RTL as "right-align the LTR layout."
- Don't author with Arabic Presentation Forms or hand-insert `U+200B`/`U+202E`.
- Don't naively flatten self-hosted Google CJK fonts into one file.

---

## Common pitfalls & how to avoid them

- **Tofu (□□□).** No font in the chain covers a character — caused by a Latin-only web font with no CJK fallback, over-aggressive subsetting, or a custom font missing symbols. *Fix:* terminate the stack with broad system fonts (`system-ui`, "Noto Sans") + `sans-serif`; never static-subset dynamic content.
- **Self-hosting loses slicing.** Copying Google CJK files without preserving `unicode-range` `@font-face` blocks ships the whole multi-MB file. *Fix:* keep the slices or regenerate with `cn-font-split`.
- **`preload` bypasses `unicode-range`.** A heavy CJK/Arabic slice downloads on pages that never use it. *Fix:* preload only the critical Latin subset.
- **Subsetting kills shaping.** Stripping glyphs from Arabic/Hebrew/Thai/Indic can delete GSUB/GPOS and break rendering while file size drops. *Fix:* keep layout features; test RTL/complex content before shipping — bugs are invisible in LTR Latin previews.
- **Ugly Latin inside CJK fonts.** Thin, scrunched, monospaced-looking Latin is *by design* (half-width = half a CJK em); old system fonts (SimSun, Batang/Gulim) make it worse. *Fix:* Latin font first in the stack.
- **Mismatched baselines in mixed Arabic/Latin.** A Latin fallback with a different x-height clashes with the Arabic. *Fix:* use an Arabic font with good Latin or match metrics.
- **`word-break: break-all` ugliness.** Breaks at the exact overflow point even when the whole word would have fit on the next line. *Fix:* prefer `overflow-wrap: break-word`; reserve `break-all` for dense CJK-with-Latin.
- **`text-wrap: balance` + `overflow-wrap: anywhere`** → unwanted wrapping. *Fix:* keep `anywhere` scoped to URL elements.
- **Korean `keep-all` overflow.** Long URLs/words spill out of narrow columns and table cells. *Fix:* add scoped `overflow-wrap: break-word` to cells/links.
- **RTL as right-align only.** Icons, menus, padding, progress end up on the wrong side. *Fix:* `dir="rtl"` + logical properties + mirror directional icons.
- **Presentation Forms / manual `U+200B`.** Authoring legacy Arabic forms or sprinkling zero-width spaces for Thai breaks at scale — invisible, error-prone. *Fix:* author plain codepoints; rely on dictionary breaking.

---

## What real users complain about

- **"Latin inside CJK fonts is hideous"** — "scrunched, needlessly thin, somehow a mix of the ugliest elements of serif and monospaced fonts." It's the half-width em design; lead with a Latin font.
- **Server-side / headless tofu** — AI video, caption, and headless-container pipelines emit □□□ for Japanese/CJK because the render container has **no CJK font installed**. Fix: install Noto CJK in the backend image. A recurring real-world failure.
- **"Web-safe" surprises** — developers learn web fonts load from the **OS**, not the browser, so "safe" assumptions silently fall back to wrong fonts cross-platform.
- **Can't add just ".com" or 52 Latin chars** to a CJK page without dragging in spindly full-width Latin or a whole second font — frustration that CJK fonts don't expose proportional Latin readily.
- **Bad kerning in the Latin half** of non-Latin brand fonts (Amharic, Korean, Arabic) needing manual fixes.
- **Korean `keep-all`** praised for readability but flagged for breaking layouts with URLs/long strings in tight containers.
- **Desktop bug class** — CJK after an RTL override (`U+202E`) gets mis-tagged as Complex instead of Asian, so the defined Asian fallback font isn't applied (LibreOffice).

---

## Senior-level checklist

- [ ] `lang` set on `<html>` and on every inline language switch; `dir="rtl"` on RTL subtrees.
- [ ] `line-height` tuned per script via `:lang()` (CJK ~1.7, Arabic ≥1.8, Devanagari 1.4–1.6); Latin not over-set globally.
- [ ] Arabic/Indic body size bumped ~10–15% for equal readability.
- [ ] Font stack: nice Latin first → script font(s) → broad system (`system-ui`, "Noto Sans") → `sans-serif`. No codepoint can render tofu.
- [ ] CJK delivered via `unicode-range` slices (own slices or `cn-font-split`), not one monolith; all subsets WOFF2.
- [ ] `preload` used **only** on the critical Latin subset, never on a sliced CJK/Arabic file.
- [ ] Subsetted complex scripts retain GSUB/GPOS; tested with real RTL/Indic/Thai content.
- [ ] Arabic ligatures/contextual forms on (`liga`/`rlig`/`calt`); never `liga 0`; no authored Presentation Forms.
- [ ] RTL uses CSS logical properties throughout; directional icons mirrored, non-directional not.
- [ ] `word-break: keep-all` (Korean) and `overflow-wrap: break-word` (URLs) scoped, not global; no `break-all`/`break-word` in resets.
- [ ] Mixed-script headlines optically balanced (size/weight matched across scripts).
- [ ] Backend/headless render containers have Noto CJK (and needed scripts) installed.
- [ ] Numbers/embedded Latin in RTL verified LTR (Bidi correct; `<bdi>`/`dir="auto"` where ambiguous).
- [ ] Mixed CJK+Latin spacing handled with `text-autospace`/`text-spacing-trim` (`@supports`-gated), not hand-typed spaces.

## Sources

- Noto fonts overview — https://fonts.google.com/noto
- Google Fonts CSS API & `unicode-range` slicing — https://developers.google.com/fonts/docs/css2
- MDN, `@font-face` `unicode-range` — https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/unicode-range
- MDN, `word-break` — https://developer.mozilla.org/en-US/docs/Web/CSS/word-break
- MDN, `overflow-wrap` — https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap
- MDN, `line-break` — https://developer.mozilla.org/en-US/docs/Web/CSS/line-break
- MDN, `text-wrap` — https://developer.mozilla.org/en-US/docs/Web/CSS/text-wrap
- MDN, `text-autospace` — https://developer.mozilla.org/en-US/docs/Web/CSS/text-autospace
- MDN, `text-spacing-trim` — https://developer.mozilla.org/en-US/docs/Web/CSS/text-spacing-trim
- MDN, CSS logical properties — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values
- MDN, `dir` attribute & Bidi — https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/dir
- W3C, "Approaches to full justification" & Internationalization (i18n) — https://www.w3.org/International/
- W3C, Requirements for Japanese Text Layout (JLREQ) — https://www.w3.org/TR/jlreq/
- fonttools / `pyftsubset` — https://fonttools.readthedocs.io/en/latest/subset/
- HarfBuzz shaping engine — https://harfbuzz.github.io/
- ICU line breaking — https://unicode-org.github.io/icu/userguide/boundaryanalysis/
- cn-font-split — https://github.com/KonghaYao/cn-font-split
- Glyphhanger — https://github.com/zachleat/glyphhanger

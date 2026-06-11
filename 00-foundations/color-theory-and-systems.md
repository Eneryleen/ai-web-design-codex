# Color Theory & Color Systems

> How to choose, structure, and ship color for digital products: perceptually-uniform color models (OKLCH/OKLab), accessible token architectures, WCAG/APCA contrast, dark mode, and gradients. The senior-level outcome: a token-driven palette that is consistent across hues, passes contrast in both light and dark mode, and is maintainable from primitive values up to component overrides.

**Apply when:** picking any color, building a palette/theme, adding dark mode, or auditing contrast · **Related:** [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md) · [CSS, Styling & Tailwind](../04-libraries-and-tools/css-styling-and-tailwind.md) · [Visual Hierarchy](visual-hierarchy.md)

## TL;DR — rules at a glance
- Author color in **OKLCH** (`oklch(L C H / a)`), not HSL/HEX. HSL lightness is perceptually inconsistent — yellow at `hsl(60 70% 50%)` looks far brighter than blue at `hsl(240 70% 50%)`. OKLCH fixes this.
- OKLCH ranges: **L 0–1** (black→white), **C 0–~0.37** sRGB (higher in P3), **H 0–360°** (red ≈20–30°, yellow ≈90°, green ≈140°, blue ≈220–240°, purple ≈320°).
- **Never** use pure-saturation primaries. `#FF0000` on white is only **3.998:1** → FAILS AA body text. `#00FF00` on white is **1.37:1** — extreme fail.
- **Contrast does not round.** `#777` = **4.47:1** on white = legal FAIL. `#767676` = exactly **4.5:1** = the minimum passing gray.
- WCAG 2.x AA: **4.5:1** normal text, **3:1** large text (18pt+/14pt+ bold) and UI/borders/icons (1.4.11).
- **60-30-10**: 60% neutral surfaces, 30% secondary, 10% accent. The accent's scarcity is its signal — reserve it for CTAs, active states, links.
- Use **semantic tokens** (`--color-text-primary`), never literal tokens (`--blue-500`) in components. Three layers: primitive → semantic/alias → component.
- **Dark mode ≠ inversion.** Author dedicated dark variants per token. Both modes must independently pass contrast.
- Dark mode: no `#000000` bg, no `#FFFFFF` text. Bg `#0A0A0A`–`#161616`; primary text L 92–95%; secondary L 65–75%.
- Dark accents: keep hue, **+10–20% lightness, −10–20% saturation/chroma** vs the light-mode version, or they look neon.
- Gradients and `color-mix()`: always specify `in oklch` to avoid the desaturated gray midpoint of sRGB interpolation.
- Color is **never the only** signal (WCAG 1.4.1): pair with icon/text/underline.
- Honor `prefers-contrast: more` (boost L, thicken borders) and `forced-colors: active` (Windows HC — use `forced-color-adjust` only for icons).
- For programmatic hover, use OKLCH relative color `calc(l + 0.06)` (lighten on dark bg) / `calc(l - 0.06)` (darken on light bg); for active/pressed use `±0.10`. Use decimals, not `%` — percentages don't work inside `calc()` for OKLCH channels.

---

## 1. Color models: why OKLCH is the modern default

| Model | Space | Perceptually uniform? | Gamut | Use for |
|---|---|---|---|---|
| HEX / RGB | sRGB | No | sRGB only | Legacy, copy-paste values |
| HSL | sRGB | **No** (L is fake) | sRGB only | Quick hacks, never systematic scales |
| LCH | CIELab | Mostly | wide | Better than HSL; has blue hue-shift bug |
| **OKLCH** | **OKLab (cylindrical)** | **Yes** | **sRGB + P3** | **Author everything here** |

**Why OKLCH wins:**
- **Perceptual uniformity** — equal numeric deltas → equal perceived deltas in L, C, and H. A palette built by stepping L evenly *looks* evenly stepped.
- **Constant hue under edits** — change L or C and the hue stays put. HSL and even LCH shift hue (LCH is wrong in the blue/purple 270–330° band; OKLCH fixes it).
- **Wide gamut** — OKLCH addresses **Display-P3**, which is >30% more color volume than sRGB. HEX/RGB/HSL are locked to sRGB.

```css
/* OKLCH syntax: oklch(Lightness Chroma Hue / alpha) */
--blue-600: oklch(0.55 0.18 250);   /* L=0.55, C=0.18, H=250° */
--text:     oklch(0.20 0.01 250);   /* near-neutral dark, slight cool tint */
--accent:   oklch(0.62 0.20 28 / 0.9); /* warm red-orange, 90% opacity */
```

**Build scales by stepping L, holding H, easing C.** A perceptually even 9-step gray (Tailwind-style) just walks L down while keeping C near 0:

```css
--gray-50:  oklch(0.985 0.002 250);
--gray-200: oklch(0.92  0.004 250);
--gray-400: oklch(0.71  0.01  250);
--gray-600: oklch(0.55  0.02  250);
--gray-800: oklch(0.37  0.015 250);
--gray-950: oklch(0.18  0.01  250);
```

> Heuristic: if your HSL scale steps look visually uneven (a "jump" somewhere), that unevenness *is* HSL's non-uniformity. Rebuild in OKLCH.

## 2. Building an accessible palette

**Keep base hues to 5–7.** A full system of 50–80 token values derives from those. Typical inventory:
- 1–2 primary brand colors + 1 accent
- one 8–10 step neutral/gray scale
- 4 semantic colors (error/success/warning/info), each with its own lightness scale

**Anchor accessibility by L, not by eye:** to reliably hit WCAG AA against a white surface, set **text/foreground hues at L ≤ 0.45**; against a dark surface, **L ≥ 0.70**. Because OKLCH L is perceptual, the same L threshold holds across hues — a green and a blue at L 0.45 both clear ~4.5:1 on white.

**Generate, then verify.** Tools produce candidates; always confirm with a real contrast check (numbers in §7). Don't trust "it looks fine."

## 3. The 60-30-10 rule

- **60% dominant** — backgrounds and large surfaces, usually a low-chroma neutral.
- **30% secondary** — cards, sidebars, supporting blocks, secondary buttons.
- **10% accent** — CTAs, active states, links, focus, key highlights only.

The 10% earns attention precisely because it's rare. If the accent shows up on every element it stops signaling. On data-dense screens (tables, dashboards) the accent budget shrinks further — neutral-dominant UIs read as calmer and let status colors pop. See [CRM & Admin Dashboards](../03-site-types/crm-admin-dashboards.md).

## 4. Semantic tokens & three-layer architecture

Token names describe **role/intent**, not value. This is the only model that scales to theming.

```
Layer 1 — Primitive (raw, mode-agnostic)
  --blue-500: oklch(0.60 0.17 250);
  --gray-900: oklch(0.20 0.01 250);

Layer 2 — Semantic / alias (role-based; remapped per theme)
  --color-text-primary:        var(--gray-900);
  --color-surface-base:        var(--gray-50);
  --color-interactive-default: var(--blue-500);

Layer 3 — Component (scoped overrides)
  --button-primary-bg: var(--color-interactive-default);
```

- Components consume **only** Layer 2/3 names — never `--blue-500` directly.
- Changing a primitive propagates everywhere automatically.
- Theming swaps Layer 2 mappings; component code stays theme-agnostic.

```css
:root            { --color-surface-base: var(--gray-50);  --color-text-primary: var(--gray-900); }
[data-theme=dark]{ --color-surface-base: var(--gray-950); --color-text-primary: var(--gray-100); }
```

Real-world references: **Tailwind v4** ships 11 steps per color (50…950), all in OKLCH. **Radix UI** ships ~30 scales, each with light/dark/alpha variants, with steps mapped to roles (backgrounds → interactive → borders → text). Detail in [Design Systems & Tokens](../05-process-and-systems/design-systems-and-tokens.md).

## 5. State colors & interactive states

Define every interactive state and test each for contrast — most teams only test `default`.

| State | Treatment |
|---|---|
| default | base interactive color |
| hover | `oklch(from … calc(l + 0.06) c h)` on dark bg; `calc(l - 0.06)` on light bg |
| focus | visible ring, **≥3:1** vs adjacent *and* vs page bg (WCAG 2.4.11, 1.4.11) |
| active/pressed | `calc(l - 0.10)` on light bg / `calc(l + 0.10)` on dark bg, optional chroma bump |
| disabled | low-contrast; **exempt** from contrast rules |

**Relative color** keeps states consistent across every variant from one rule:

```css
.button:hover {
  /* lighten on dark bg; use calc(l - 0.06) on light bg — perceptually predictable for ANY hue */
  background: oklch(from var(--button-color) calc(l + 0.06) c h);
}
```

> Pitfall: inside `calc()` for OKLCH channel values, use **decimals** (`0.06`, `0.10`), not percentages (`6%`) — `%` is not accepted for numeric channels in CSS relative color syntax.

**Semantic state colors** (each gets its own L-scale, not one flat value):
```css
--color-error:   oklch(0.58 0.20 27);    /* red    */
--color-success: oklch(0.62 0.16 150);   /* green  */
--color-warning: oklch(0.78 0.16 80);    /* amber  */
--color-info:    oklch(0.62 0.14 240);   /* blue   */
```
Never rely on hue alone for status (WCAG 1.4.1) — error states need an icon or text label; links need an underline or other non-color cue.

## 6. Contrast: WCAG 2.x and APCA

**Contrast ratio formula:** `(L1 + 0.05) / (L2 + 0.05)`, where `L = 0.2126·R + 0.7152·G + 0.0722·B` after gamma correction (lighter color = L1).

**WCAG 2.1/2.2 AA (use for legal compliance):**
- Normal text: **4.5:1**
- Large text (≥18pt or ≥14pt bold ≈ 24px / 18.66px bold): **3:1**
- UI components, form/input borders, meaningful icons, graphical objects: **3:1** (1.4.11 Non-text Contrast)
- Focus indicators: **3:1** vs adjacent colors
- **AAA:** 7:1 normal, 4.5:1 large

**APCA (additive check, not legally binding yet):**
- **Lc 60+** large/bold · **Lc 75+** body · **Lc 90+** small or thin-weight text
- Lc 60 ≈ WCAG 4.5:1 only in a narrow band near `#9e9e9e`.
- Use WCAG 2.x for compliance; add APCA to catch what WCAG misses (thin fonts, dark-on-dark).

**Hard rule: never round up.** `4.47:1` is a FAIL. Exemptions: disabled components and purely decorative elements.

```
#777777 on white → 4.47:1  ✗ FAIL (looks "close" but isn't)
#767676 on white → 4.50:1  ✓ minimum AA body
#FF0000 on white → 3.998:1 ✗ pure red fails body text (≈4.0:1, still a fail)
#00FF00 on white → 1.37:1  ✗ extreme fail
#0000FF on white → 8.59:1  ✓ passes AAA
```

## 7. Concrete specs & numbers
- AA: **4.5:1** body, **3:1** large/UI/borders/icons (1.4.11). AAA: **7:1** body, **4.5:1** large.
- `#767676` on white = exactly **4.5:1** (minimum passing gray). `#777` = 4.47:1 (fail).
- Pure colors on white: red `#FF0000` 3.998:1 (fail), green `#00FF00` 1.37:1 (extreme fail), blue `#0000FF` 8.59:1 (AAA).
- APCA: **Lc 60** large/bold, **Lc 75** body, **Lc 90** small/thin.
- OKLCH ranges: L 0–1; C 0–~0.37 sRGB, ~0.5+ in P3 (100% chroma ≈ 0.4 in % notation); H 0–360°.
- Accessible foreground via L: **L ≤ 0.45** on white, **L ≥ 0.70** on dark.
- Dark-mode text: primary **L 92–95%**, secondary **L 65–75%**, disabled **L 40–50%**.
- Dark-mode surface elevation: **+5–8% luminance per level**, **4 levels minimum**.
- Dark-mode accent transform: same hue, **+10–20% L**, **−10–20% chroma/saturation**.
- Dark bg practical values: `#0A0A0A`–`#161616`, `#181A20`, `#222C36`, `#242424` — never `#000000`. Off-white text `#E0E0E0`–`#F0F0F0`.
- Typical system: 1–2 primary + 1 accent + 8–10 grays + 4 semantic scales = **50–80 values from 5–7 base hues**.
- WCAG 2.5.8 (AA, 2.2): interactive targets need a **24×24 CSS px** bounding box *or* at least 24 px of offset spacing to any adjacent target — it's a spacing criterion, not purely a size floor. Apple HIG minimum is 44×44 pt; Material is 48×48 dp. See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md).
- `oklch()` support: Chrome/Edge 111+, Firefox 113+, Safari 15.4+ (>92% global, Q2 2025). `color-mix()`: Chrome 111+, Firefox 113+, Safari 16.2+.
- Low-contrast text appears on **~80% of home pages** (WebAIM Million — 80.6% in 2024; updated annually, consistently >78% since 2019).

## 8. Dark mode design

Dark mode is a **separate palette**, not an inversion. Direct inversion gives wrong saturation, broken hierarchy, and eye strain.

**Rules:**
1. **No pure black bg / no pure white text.** Bg `#0A0A0A`–`#161616`; text `#E0E0E0`–`#F0F0F0`. Pure black "smears" on OLED and strains eyes after ~20 min.
2. **Tame saturation.** Saturated light-mode accents go "neon/electric" on dark because there's no luminance competition. Drop chroma 10–20%, nudge L up. Example: `hsl(215 60% 50%)` light → `hsl(215 50% 60%)` dark.
3. **Elevation, not shadows.** Drop shadows are invisible on dark surfaces. Express depth with **lighter surfaces** at higher elevation (≥4 levels, +5–8% L each).

```css
[data-theme=dark] {
  --surface-base:    oklch(0.16 0.005 250); /* page */
  --surface-raised:  oklch(0.21 0.006 250); /* cards (+~5% L) */
  --surface-overlay: oklch(0.26 0.007 250); /* menus/modals */
  --surface-popover: oklch(0.31 0.008 250); /* tooltips */
  --text-primary:    oklch(0.93 0.01 250);  /* ~L92–95%, not pure white */
  --text-secondary:  oklch(0.70 0.01 250);
}
```

4. **Both modes pass independently.** A pair that's 4.5:1 in light can fail in dark — re-test every token pair per mode, every state.
5. **Ship a manual toggle**, not just `prefers-color-scheme`. Dark isn't universally better: astigmatism sufferers often prefer light; photosensitive users prefer dark.

```css
:root { color-scheme: light; }
@media (prefers-color-scheme: dark) { :root:not([data-theme]) { color-scheme: dark; /* + dark token map */ } }
```

YouTube dark mode on OLED at full brightness used **43% less power** than light mode (Google internal measurement, presented at Android Dev Summit 2018 by Doris Liu). Effect scales with brightness; at 50% brightness the gap shrinks to ~7–8%. Real battery argument on OLED phones, not applicable to LCD.

**OS-level color-accessibility media queries** (senior gap — most teams skip these):

- `prefers-contrast: more` — user has requested higher contrast (macOS "Increase Contrast", some Android settings). Respond by bumping foreground L, removing subtle borders, and making focus rings heavier. *Do not assume your default palette satisfies this.*
- `forced-colors: active` (Windows High Contrast / Contrast Themes) — browser replaces almost all colors with a small OS-defined palette (`ButtonText`, `LinkText`, `HighlightText`, `Canvas`, etc.). Your custom colors, gradients, and box-shadows are stripped. Design rules: use semantic HTML (not `div` soup), ensure icons have meaning beyond color, don't hide focus rings with `outline: none`. Test in Windows Settings → Accessibility → Contrast Themes.

```css
@media (prefers-contrast: more) {
  :root {
    --color-text-primary: oklch(0.08 0.01 250); /* max legibility */
    --color-border:       oklch(0.40 0.01 250); /* was subtle, now visible */
  }
}
@media (forced-colors: active) {
  /* Restore icon visibility; gradients are gone */
  .icon { forced-color-adjust: none; fill: ButtonText; }
}
```

## 9. Gradients & avoiding pure black / pure saturation

**Interpolate in OKLCH** to kill the sRGB "gray band" — interpolating across complementary hues in sRGB desaturates the midpoint into mud.

```css
/* sRGB default → muddy gray midpoint */
background: linear-gradient(oklch(0.65 0.20 70), oklch(0.58 0.22 240));
/* OKLCH interpolation → chroma preserved end-to-end */
background: linear-gradient(in oklch, oklch(0.65 0.20 70), oklch(0.58 0.22 240));
/* shortest hue path can be forced for hue-based gradients */
background: linear-gradient(in oklch longer hue, oklch(0.7 0.15 30), oklch(0.7 0.15 300));
```

- **Avoid pure black & full chroma** as functional colors. Pure-saturation hues fail contrast and look harsh; near-neutral darks (slight cool/warm tint) read richer than `#000`.
- For tints/shades, prefer OKLCH L steps over HSL `darken()`/`lighten()` — OKLCH guarantees a darker/lighter *appearance* regardless of hue.
- **P3 wide-gamut** needs a guard so sRGB displays still get a valid color:
```css
.brand { color: oklch(0.62 0.20 28); }                 /* clamped to sRGB */
@media (color-gamut: p3) { .brand { color: oklch(0.62 0.28 28); } } /* richer in P3 */
```

## 10. Tools & libraries
- **oklch.com** — canonical OKLCH picker + hex/RGB converter and explainer. Start here.
- **Harmonizer** (Evil Martians) — accessible, consistent palettes via OKLCH + APCA; web app + Figma plugin, open-source.
- **apcach** (Evil Martians) — JS lib to generate accessible combos (APCA *and* WCAG); outputs OKLCH/HEX/RGB/Display-P3.
- **Huetone** — palette generator defaulting to Oklab.
- **tints.dev** — Tailwind 11-step (50–950) scale from one hex, OKLCH output.
- **Meta Color / OkColor / Polychrom / Shade Perfection** — Figma plugins for OKLCH systems and live APCA. Verify plugin versions; Figma plugin APIs for color-space conversions have had interop issues between OKLCH and P3 document profiles — always spot-check output values against `oklch.com`.
- **WebAIM Contrast Checker / contrast.tools** — WCAG (and APCA side-by-side) ratio checks.
- **Stark / axe DevTools / Pa11y** — accessibility + contrast auditing in design, browser, and CI.
- **postcss-oklab-function** — OKLCH fallback for old browsers. **`npx convert-to-oklch`** — batch-convert CSS to OKLCH. **stylelint-gamut** — flag P3 colors missing the `@media (color-gamut: p3)` guard; `color-no-hex` to forbid hex/rgb/hsl.
- **`color-mix(in oklch, …)`** — native CSS blending without JS; specify `in oklch` or you get sRGB interpolation with the gray-midpoint problem. Example: `color-mix(in oklch, var(--brand) 80%, white)`. Chrome/Edge 111+, Firefox 113+, Safari 16.2+.
- **`color-contrast()` (CSS Level 5)** — future native function to auto-pick the highest-contrast option from a list. As of 2026 it is in the spec but **not shipped in any stable browser**. Do not use in production; use WCAG-compliant manual token pairs instead.

**When NOT to reach for tools:** don't auto-generate a whole system from one brand hex and ship it unchecked — generators don't know your real surfaces or content; always verify contrast on actual pairs. Don't add a PostCSS fallback if your support floor is already ≥ the OKLCH baseline (>92% global).

## 11. Do / Don't

**Do**
- Author in OKLCH; build scales by stepping L, holding H.
- Name tokens by role; keep components on semantic/component tokens only.
- Verify every fg/bg pair against WCAG numbers, in both modes, for every state.
- Use OKLCH gradients (`in oklch`) and relative-color states.
- Reserve the accent for the 10%. Provide a manual dark toggle.
- Pair color with icon/text/underline for any status or meaning.
- Handle `prefers-contrast: more` (bump foregrounds, thicken borders) and `forced-colors: active` (restore icon visibility with `forced-color-adjust`).

**Don't**
- Build systematic scales in HSL or expect HSL L to be uniform.
- Hardcode `--blue-500` in components or hardcode hex in CSS.
- Round contrast up (4.47 ≠ pass) or test only the default state.
- Use `#000`/`#FFF` in dark mode, or port light accents into dark unchanged.
- Rely on drop shadows for depth on dark surfaces (use elevation).
- Convey meaning by color alone.
- Use `color-contrast()` in production — it is not yet in any stable browser (2026).
- Suppress `outline: none` without a replacement focus style — breaks keyboard users and `forced-colors` mode simultaneously.

## 12. Common pitfalls & how to avoid them
- **HSL for scales** → uneven perceived steps (HSL yellow L50 ≫ blue L50). Fix: build in OKLCH.
- **`%` inside `calc()` for `oklch()` channels** → invalid/ignored in CSS relative color syntax; value silently treated as 0. Fix: use decimals (`calc(l + 0.06)` for hover, `calc(l + 0.10)` for active).
- **Porting light accents to dark** → neon/electric. Fix: +10–20% L, −10–20% chroma.
- **Pure `#000` bg** → OLED smear, eye strain. Fix: `#0A0A0A`–`#161616`.
- **Pure-saturation primaries** → fail contrast even on white. Fix: drop chroma, target L thresholds.
- **Rounding contrast** → legal fails shipped. Fix: treat 4.5:1 as a hard floor.
- **Testing default state only** → broken hover/focus/active contrast. Fix: test all states.
- **Text over hero photography** → unpredictable contrast per crop. Fix: semi-transparent overlay/scrim, test every crop. See [Imagery & Assets](imagery-icons-and-assets.md).
- **One palette for both modes** → one mode passes, the other fails. Fix: independent dark variants, re-verify.
- **P3 without guard** → invalid/odd colors on sRGB displays. Fix: wrap in `@media (color-gamut: p3)`.
- **Ignoring `prefers-contrast: more`** → app looks contrast-compliant but users who enabled OS high-contrast still find it hard to read. Fix: define a `prefers-contrast: more` overrides block.
- **Ignoring `forced-colors: active`** → Windows Contrast Themes strip your palette; custom icons lose color meaning. Fix: test in Windows, use `forced-color-adjust` only where required, rely on semantic HTML.

## 13. What real users complain about
- **Low contrast everywhere** — light gray on white is endemic (low-contrast text on ~80% of home pages, WebAIM Million 2024); users report "I literally can't read the placeholder/labels."
- **Eye strain from pure-black dark mode** and "smearing" text on OLED phones, especially after extended reading.
- **"Neon" dark-mode accents** that vibrate against dark backgrounds because saturation wasn't tamed.
- **Forced dark mode with no toggle** — astigmatism users find pure dark harder to read; they want a light option (and vice-versa for photosensitive users).
- **Status conveyed by color only** — colorblind users can't tell error from success when there's no icon/label; ~1 in 12 men has a color-vision deficiency.
- **Invisible focus rings** — keyboard users lose their place when focus contrast is under 3:1.
- **Muddy gradients** — designers notice the gray "dead zone" in mid-gradient from sRGB interpolation.

## 14. Senior-level checklist
- [ ] All authored color is OKLCH (or has an OKLCH source of truth); no hardcoded hex in components.
- [ ] Three-layer tokens in place: primitive → semantic → component; components reference semantic/component names only.
- [ ] ≤ 7 base hues; gray scale and brand scales built by stepping L in OKLCH.
- [ ] Every text pair ≥ 4.5:1 (body) / 3:1 (large); UI borders, icons, focus rings ≥ 3:1 (1.4.11).
- [ ] Contrast verified with a real checker — no rounding; disabled/decorative correctly exempt.
- [ ] Dedicated dark palette: bg `#0A0A0A`–`#161616`, text L 92–95%/65–75%, ≥4 elevation levels (+5–8% L), no shadows-for-depth.
- [ ] Dark accents: same hue, +10–20% L, −10–20% chroma; both modes pass independently.
- [ ] All states (default/hover/focus/active/disabled) tested for contrast; states use OKLCH relative color.
- [ ] No meaning by color alone (icons/labels/underlines present).
- [ ] Gradients use `in oklch`; P3 colors guarded by `@media (color-gamut: p3)`; fallback strategy decided.
- [ ] Manual light/dark toggle shipped, not just `prefers-color-scheme`.
- [ ] Accent confined to the 10% (CTAs/active/links); checked on a data-dense screen, not just the hero.
- [ ] `prefers-contrast: more` overrides block defined; high-contrast foreground values increase legibility.
- [ ] `forced-colors: active` tested in Windows Contrast Themes; icons readable; no `outline: none` without replacement.

## Sources
- WCAG 2.1/2.2 (W3C): contrast minimums 1.4.3, non-text contrast 1.4.11, use-of-color 1.4.1 — https://www.w3.org/TR/WCAG22/
- WebAIM Contrast Checker — https://webaim.org/resources/contrastchecker/
- WebAIM Million (annual accessibility analysis) — https://webaim.org/projects/million/
- OKLCH picker & converter — https://oklch.com/
- APCA / contrast tooling — https://contrast.tools/
- Harmonizer (Evil Martians) — https://harmonizer.evilmartians.com/
- apcach (Evil Martians) — https://evilmartians.com/opensource/apcach
- tints.dev — https://tints.dev/
- Tailwind CSS color palette (OKLCH) — https://tailwindcss.com/docs/colors
- Radix UI Colors — https://www.radix-ui.com/colors

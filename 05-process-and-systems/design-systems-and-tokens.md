# Design Systems & Design Tokens

> How to architect design tokens (primitive → semantic → component), build a token pipeline with Style Dictionary, theme dark mode through tokens, and decide when a full design system is warranted vs. a single token sheet. Outcome: a token architecture that scales to multi-theme, multi-platform products without find-and-replace churn, and the judgment to not over-build it.

**Apply when:** building any product that needs consistency across ≥2 surfaces/teams, or any site requiring dark mode/theming · **Related:** [Color Theory & Color Systems](../00-foundations/color-theory-and-systems.md) · [CSS, Styling & Tailwind](../04-libraries-and-tools/css-styling-and-tailwind.md) · [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md)

## TL;DR — rules at a glance
- **Three tiers, one rule:** Primitive (`color.blue.500=#0066cc`) → Semantic (`color.bg.primary` → primitive) → Component (`button.bg` → semantic). **Product code references semantic/component tokens ONLY, never primitives.** Enforce with linting from day one.
- **Name by intent, not appearance.** `color.bg.primary` survives a blue→green rebrand; `color.blue.500` as a *semantic* name is already broken.
- **Semantic tokens are the only thing that makes dark mode/theming cheap.** A theme swap = reassign one token set, not edit every component.
- **Dark mode ≠ inversion.** Off-white text (`#E0E0E0`–`#F0F0F0`), near-black bg (`#0A0A0A`–`#161616`), and **elevation by luminance, not shadow** (surfaces get *lighter* as they rise).
- **Keep brand color and functional/status color in separate families.** Brand red ≠ error red — same value sends conflicting signals.
- **Tokens alone are not a design system.** A DS = tokens + components + patterns + docs + governance + shared language. Conflating them is optimism bias.
- **Spacing tokens stay global/primitive.** Component-level spacing aliases add confusion with little benefit. `spacing-04 = 4px`.
- **DTCG (W3C Community Group) is the format:** `$value`, `$type`, `$description` (dollar-prefixed), `.tokens.json` files. Published community-group report (not a W3C Recommendation as of 2026) but de-facto industry standard — 10+ tools implement it.
- **Style Dictionary v4 = DTCG-native.** `$value`/`$type` replaced legacy `value`/`type`/`comment`; **v3 and v4 formats can't be mixed in one instance.**
- **WCAG 2.2 AA in BOTH modes:** 4.5:1 normal text, 3:1 large text & UI/graphics. Validate **every state** (hover/focus/active/disabled/error), not just default.
- **A DS is a living system and a full-time job.** No evolution budget → stale → ignored → abandoned. Don't put juniors solo on it.
- **Atomic Design is inspiration, not a contract.** Don't burn hours debating atom vs. molecule. Prefer Styles + Components + Patterns.
- **When it's overkill:** 3-person MVP → Tailwind config or a Notion token sheet. **When indefensible to skip:** 20k-employee org duplicating component work across teams.

---

## 1. What a token is (and isn't)

A **design token** is a platform-agnostic key/value pair — a single named style decision (`color.bg.primary = #0066cc`) stored in JSON/YAML — consumed by *both* design tools and codebases from one source of truth. The term was popularized by Salesforce's Lightning Design System team (~2014–2015).

A token is **not**: a CSS variable (that's an *output* of a token), a Figma style, or a hex value pasted into a component. The token is the upstream definition; CSS custom properties, Swift constants, Android XML, and Dart constants are downstream *builds* of it.

**The single-source-of-truth chain:**
```
tokens.json  →  Style Dictionary build  →  ├─ web/tokens.css   (CSS vars)
(DTCG format)                              ├─ ios/Tokens.swift
                                           ├─ android/colors.xml
                                           └─ docs (Storybook/zeroheight)
```

## 2. The three-tier hierarchy (the core architecture)

This is the industry standard (Carbon/IBM, Adobe Spectrum, Salesforce all use a variant). Each tier references *only the tier above it*.

| Tier | Aka | Example | Who references it |
|---|---|---|---|
| **1. Primitive** | Global / base / option | `color.blue.500 = #0066cc` | Semantic tokens only |
| **2. Semantic** | Alias / decision | `color.bg.primary = {color.blue.500}` | Component tokens + product code |
| **3. Component** | Scoped | `button.bg.default = {color.bg.primary}` | That component only |

**Why three tiers:** Primitives are the palette (raw, themeless, never used directly). Semantics encode *intent/decision* (`bg.primary`, `text.danger`) and are the swap point for themes. Component tokens scope a decision to one component so you can retarget a single component without touching the global semantic layer.

**The non-negotiable rule:** a `<button>`'s CSS must read `var(--button-bg-default)` or at worst `var(--color-bg-primary)` — **never** `#0066cc` and **never** `var(--color-blue-500)`. If a primitive name appears outside the token definition file, it's a violation. Lint it.

```jsonc
// DTCG community-group format. Note $value / $type / $description and {alias} refs.
{
  "color": {
    "blue":  { "500": { "$value": "#0066cc", "$type": "color" } },   // primitive
    "bg":    { "primary": { "$value": "{color.blue.500}", "$type": "color",
                            "$description": "Primary surface / brand fill" } } // semantic
  },
  "button": {
    "bg": { "default": { "$value": "{color.bg.primary}", "$type": "color" } } // component
  }
}
```

### When to skip a tier
- **Spacing/sizing:** keep as global/primitive tokens only. `spacing-04 = 4px`. Component-level spacing aliases ("the alias & mapped tokens always caused more confusion, pain and maintenance with little perceived benefit") — don't build them.
- **Color/typography:** full three tiers pay off because they theme and re-skin.
- **Component tokens:** add them only when a component genuinely needs to diverge from the semantic layer (e.g., `button.bg` differs from generic `bg.primary`). Don't mint a component token for every trivial property — that's token sprawl.

## 3. Naming

**Describe intent, not appearance.** Names must survive a brand pivot.

**Structural pattern:** `{namespace}-{category}-{role}-{modifier}-{state}` — not every segment is needed each time, but each segment **always appears in the same position** when present.

```
n-color-accent              // namespace . category . role
n-color-border-strong       // + modifier
n-color-text-weak
color-bg-surface-raised
color-text-danger-hover     // + state
spacing-04
```
(Adobe Spectrum uses `family + 50–900 scale`, e.g. `blue-500`, `gray-200` — appropriate for *primitives*. For semantics, always prefer intent-based names regardless of the primitive scheme.)

**Level count heuristic:**
- Regular systems: **2–3 levels** (`color-bg-primary`).
- Mature multi-platform systems: **4–6 levels**.
- **>6 levels = cognitive overhead** — stop. Levels typically map: category → property → surface/element → variant/scale → state.

**Hard naming rules:**
- Semantic names must be **theme-agnostic**: `color.bg.primary` works in light *and* dark. `color.light-gray-background` bakes in a light-mode assumption — banned.
- Pair-words for opposites and intensity: `strong`/`weak`, `default`/`hover`/`active`/`disabled`, `base`/`raised`/`overlay`/`floating`.
- Brand vs. functional families separated: `color.brand.*` and `color.status.{success,warning,danger,info}.*` live apart even if a value coincides today.

## 4. Color scales

Build a **10-step scale, 50 (lightest) → 900 (darkest)** per hue. Add **half-steps (50, 950)** to fine-tune dark mode.
- The **700 weight is a starting-point heuristic for mid-saturation blues/purples on white** — verify per hue. Warm hues (reds, oranges) and yellow-greens can fail 4.5:1 as early as 600 or even 500. Always measure; never assume.
- Keep brand hues and functional hues in **separate token families**.
- Hard-coded color sprawl is the canonical scaling failure: once hex values are scattered across components without tokens, dark mode and rebranding require find-and-replace across the entire codebase. Tokens prevent this.

```jsonc
"color": {
  "blue": {
    "50": { "$value": "#e7f0fb", "$type": "color" },
    "100":{ "$value": "#c3d9f5", "$type": "color" },
    "500":{ "$value": "#0066cc", "$type": "color" },
    "700":{ "$value": "#004a99", "$type": "color" },  // verify: ~4.5:1 on white for this blue — check per hue
    "900":{ "$value": "#002a57", "$type": "color" }
  }
}
```

## 5. Theming & dark mode through tokens

**The architecture insight:** components reference *semantic* tokens; a theme is just a different mapping of semantic → primitive. Light and dark are two value sets behind the *same semantic names*.

```css
/* Components only ever use semantic vars. Themes remap them. */
:root, [data-theme="light"] {
  --color-bg-surface-base:    #ffffff;
  --color-bg-surface-raised:  #f7f7f8;
  --color-text-default:       #161616;
  --color-text-danger:        #b3261e;
}
[data-theme="dark"] {
  --color-bg-surface-base:    #0e0e0e;
  --color-bg-surface-raised:  #1a1a1a;  /* lighter than base — elevation by luminance */
  --color-text-default:       #e8e8e8;  /* off-white, NOT #fff */
  --color-text-danger:        #f2b8b5;  /* lightened so it stays 4.5:1 on dark */
}
.button { background: var(--color-bg-surface-raised); color: var(--color-text-default); }
```

**Dark mode is not color inversion.** Concrete rules:
- **"White" text = off-white** `#E0E0E0`–`#F0F0F0`. Pure `#FFFFFF` on true black causes haloing/eye strain.
- **Background = near-black** `#0A0A0A`–`#161616` (avoid pure `#000` except for intentional OLED-black accents; pure black maximizes OLED power saving but increases smear/contrast harshness for text surfaces).
- **Elevation by luminance, not shadow.** Drop shadows simulate blocked light that doesn't exist on dark surfaces — they vanish. As a surface rises, make it **lighter**. Define **4 surface levels stepping up ~5–8% luminance each**: `--surface-base` → `--surface-raised` → `--surface-overlay` → `--surface-floating`. Apply consistently, never ad-hoc.
- **Re-validate contrast in dark mode independently.** Status colors usually must be *lightened* (more toward the 200–300 scale steps) to hold 4.5:1 on dark — the same token value rarely passes in both modes, which is exactly why dark is its own value set.
- Respect `prefers-color-scheme` for the default; let users override via `[data-theme]`.

> **Figma gotcha:** Figma *Variables* and design *tokens* (Tokens Studio / code) are architecturally different and do **not** map 1:1. Don't assume a Figma mode swap equals a code theme swap; reconcile them in your pipeline (e.g. Tokens Studio `sd-transforms`), don't hand-wave it.

## 6. The token pipeline (CI/CD)

A production pipeline has these stages:
1. **Authoring** — DTCG JSON in a repo (or exported from Tokens Studio / Figma Variables via API).
2. **Validation** — syntax + naming-convention lint; reject primitive references in product code.
3. **Transform/build** — Style Dictionary generates per-platform formats (CSS vars, Swift, Android XML, Dart).
4. **Versioning** — semver + modification tracking; **controlled deprecation** (mark deprecated → alias to replacement → remove after N releases), never silent removal.
5. **Publish** — to an internal registry/package so apps consume a pinned version.

**Common pipeline failure — aliasing collapse:** you build a meticulous 3-tier hierarchy, but after Style Dictionary processes it, semantic and component tokens resolve straight to **raw primitive hex values**, destroying the alias chain in the output. Fix: use correct `$type`/`$value` and a transform that **preserves references** (e.g. `outputReferences: true` in the CSS format) so `--color-bg-primary: var(--color-blue-500)` instead of `--color-bg-primary: #0066cc`. Without this, dark mode breaks because the semantic var no longer points anywhere remappable.

```js
// style-dictionary v4+ (DTCG). Preserve references so theming stays alive.
export default {
  source: ["tokens/**/*.tokens.json"],
  platforms: {
    css: {
      transformGroup: "css",
      buildPath: "build/css/",
      files: [{
        destination: "tokens.css",
        format: "css/variables",
        options: { outputReferences: true }   // <-- keeps the alias chain
      }]
    }
  }
};
```

> **Version trap:** Style Dictionary **v3 and v4 token formats cannot be mixed in one instance.** v4 = `$value`/`$type`/`$description`; v3 = `value`/`type`/`comment`. Pick one and pin it. Check the Style Dictionary changelog before upgrading — v4→v5 contains further DTCG alignment changes.

## 7. Beyond tokens — what a real design system needs

Tokens are the foundation, not the building. A design system also requires:
- **Components** — coded, accessible, documented (states, props, a11y notes).
- **Patterns** — flexible guidelines for recurring problems (forms, empty states, pagination), not fixed components.
- **Documentation** — usage, do/don't, code snippets, live playground.
- **Governance** — contribution model, review/approval, who owns what, deprecation policy.
- **Shared language** — agreed names between design and engineering.

**Structure that beats Atomic Design for most products:**
```
Styles (foundations: tokens, type, color, spacing, icons)
  → Components (fixed combinations: Button, Input, Card)
    → Patterns (flexible guidelines: Form, Filter Bar, Wizard)
```

### The Atomic Design critique
Atomic Design (atoms/molecules/organisms/templates/pages) is a useful **mental model, not a contract**. It breaks down when:
- components hold internal state (is a stateful `Dropdown` an atom or molecule? — unanswerable, and the debate wastes hours),
- components are context-dependent,
- the classification isn't formalized in code, so teams apply it inconsistently.

Use it for inspiration. For state-heavy apps, organize by feature/business capability (**Feature-Sliced Design**) rather than visual hierarchy.

## 7b. Token file organisation

Split tokens by category into separate files; Style Dictionary `source` globs them together. A typical layout:

```
tokens/
  color.tokens.json       // all color primitives + semantics
  typography.tokens.json  // font family, weight, size, line-height, tracking
  spacing.tokens.json     // spacing + sizing scale
  elevation.tokens.json   // shadow + surface-luminance tokens
  motion.tokens.json      // duration + easing
  component/
    button.tokens.json
    input.tokens.json
```

**Why separate files:** smaller diffs, clearer ownership, linting rules can target specific files (e.g. "spacing.tokens.json must contain no color `$type`").

## 7c. Typography tokens

Typography has its own DTCG types: `fontFamily`, `fontWeight`, `fontSize`, `lineHeight`, `letterSpacing`, `fontStyle`. Also `typography` composite type (grouping all into one token). Prefer atomic tokens over composite for most pipelines — composites are harder to override at the component level.

```jsonc
"font": {
  "family": {
    "sans": { "$value": "Inter, system-ui, sans-serif", "$type": "fontFamily" },
    "mono": { "$value": "'JetBrains Mono', monospace",  "$type": "fontFamily" }
  },
  "size": {
    "sm":  { "$value": "0.875rem", "$type": "fontSize" },
    "md":  { "$value": "1rem",     "$type": "fontSize" },
    "lg":  { "$value": "1.25rem",  "$type": "fontSize" },
    "xl":  { "$value": "1.5rem",   "$type": "fontSize" }
  },
  "weight": {
    "regular": { "$value": 400, "$type": "fontWeight" },
    "medium":  { "$value": 500, "$type": "fontWeight" },
    "bold":    { "$value": 700, "$type": "fontWeight" }
  },
  "lineHeight": {
    "tight":   { "$value": 1.2, "$type": "lineHeight" },
    "normal":  { "$value": 1.5, "$type": "lineHeight" }
  }
}
```

**Scale heuristic:** 4–6 size steps for most products. Use `rem` for fontSize so user font-size preferences scale correctly.

## 7d. Shadow / elevation tokens (light mode)

On light surfaces, drop shadows communicate elevation legitimately. Tokenise shadows the same way as colors — raw shadow values are primitives; semantic names encode the elevation tier.

```jsonc
"shadow": {
  "sm":  { "$value": "0 1px 2px 0 rgba(0,0,0,0.06)", "$type": "shadow" },
  "md":  { "$value": "0 4px 8px -2px rgba(0,0,0,0.12)", "$type": "shadow" },
  "lg":  { "$value": "0 12px 24px -4px rgba(0,0,0,0.14)", "$type": "shadow" },
  "xl":  { "$value": "0 24px 48px -8px rgba(0,0,0,0.18)", "$type": "shadow" }
},
"elevation": {
  "card":    { "$value": "{shadow.sm}", "$type": "shadow" },
  "dropdown":{ "$value": "{shadow.md}", "$type": "shadow" },
  "modal":   { "$value": "{shadow.lg}", "$type": "shadow" },
  "tooltip": { "$value": "{shadow.md}", "$type": "shadow" }
}
```

**Dark mode:** remap `elevation.*` tokens to luminance-based values (lighter background, no shadow) rather than stronger shadows. See Section 5.

## 7e. Multi-brand / white-label theming

Light/dark is a two-value set per token. Multi-brand adds a *third axis*: the brand's primitive palette. The architecture:

```
brand-A/primitives.tokens.json   // A's colors: brand-A.blue.500 = #0052cc
brand-B/primitives.tokens.json   // B's colors: brand-B.blue.500 = #1a6b4a
shared/semantic.tokens.json      // brand.bg.primary = {active-brand.blue.500}
```

**Mechanism:** a "brand selector" at build time switches which primitive file populates the shared semantic layer. Product code is unchanged. This is the same principle as light/dark, one more dimension.

**Caution:** cross-brand naming collisions (same semantic name, incompatible visual intent) are hard to detect. Document intent-per-token when you have ≥2 brands.

## 7f. Component documentation standards

A component is not "documented" unless it has all six of:

1. **Props / API table** — every prop, its type, default, required/optional.
2. **All visual states** — default, hover, focus (keyboard visible), active, disabled, error, loading, empty. Screenshot or live example for each.
3. **Accessibility notes** — ARIA roles/attributes, keyboard interactions, screen-reader behavior.
4. **Do / Don't examples** — at least one usage pair showing correct and incorrect context.
5. **Live playground** — editable code or Storybook story, not just static screenshots.
6. **Related components** — when to use this one vs. the alternative.

Missing any of these, the component will be misused — guaranteed.

## 8. Governance & scaling

### Token semver rules
Apply semver to the token package. What constitutes a breaking change vs. non-breaking:

| Change | Version bump |
|---|---|
| Rename a token | **MAJOR** (breaks consumers using the old name) |
| Remove a token | **MAJOR** |
| Change a token's `$type` | **MAJOR** |
| Change a token value (color, spacing) | **MINOR** (visual change, no code change required) |
| Add a new token | **MINOR** |
| Fix a typo in `$description` | **PATCH** |
| Add/update documentation | **PATCH** |

**Deprecation protocol:** mark `$description` with `[DEPRECATED: use X instead]` → add an alias pointing to the replacement → hold for **≥2 minor releases** before removal → then MAJOR bump. Never silently remove.

### Adoption & enforcement
- **Tokens fail when only one team uses them.** Designers making one-off styles or devs bypassing tokens "for speed" fragments the system. **Cross-team enforcement (linting, PR checks) is required — documentation alone is not enough.**
- **A DS is a full-time, living job.** No evolution budget → stale → ignored → abandoned. Treating it as a one-time deliverable is the #1 organizational killer.
- **Lean teams (2–5) need org sponsorship + defined scope**, or they stay "small by default" instead of "small by design."
- **Don't put juniors solo on the build** — they miss edge cases and get no adoption.
- **Dev adoption is the real metric.** If dev teams "roll their own or grab a UI library," the duplicative work is invisible to management and costs money. Measure component usage, not component count.

### When a full system is overkill
| Context | Right investment |
|---|---|
| 3-person startup, MVP | Tailwind config **or** a token sheet in Notion. No DS team. |
| Single product, 1 squad | Token sheet + a small component lib (e.g. shadcn/Radix) + Storybook. |
| Multiple teams duplicating components | Now a DS makes financial sense — overhead < duplication cost. |
| 20k-employee org with a web product | Skipping a DS is indefensible. |

**The threshold:** a DS pays off when *duplicative component work across teams exceeds the maintenance overhead of the system.* Below that line, you're gold-plating.

## Concrete specs & numbers
- **Token tiers:** 3 (primitive / semantic / component). Naming levels: 2–3 regular, 4–6 mature, **>6 = too many.**
- **Color scale:** 10 steps (50–900) + half-steps (50, 950). **700 is a heuristic starting point for mid-saturation blues — warm hues can fail at 600 or 500. Always measure.**
- **Dark mode surfaces:** 4 levels, **+5–8% luminance per step**.
- **Dark text:** `#E0E0E0`–`#F0F0F0`. **Dark bg:** `#0A0A0A`–`#161616`.
- **WCAG 2.2 AA:** **4.5:1** normal text, **3:1** large text (≥18.66px bold / ≥24px regular), **3:1** UI components & graphical objects vs. adjacent color — applies to **both** modes.
- **APCA (experimental, not normative):** APCA (Advanced Perceptual Contrast Algorithm) is a working-draft replacement for WCAG contrast ratios, still under review as of 2026. Lc 0–106 scale; body text target ~Lc 60+; accounts for text size/weight; handles dark-on-light vs. light-on-dark asymmetrically. Useful for design decisions but **not a compliance standard** — WCAG 2.2 AA is still the legal/normative bar.
- **Spacing tokens:** global only. On a 4px base grid: `spacing-01=4px`, `spacing-02=8px`, `spacing-03=12px`, `spacing-04=16px` … (IBM Carbon's naming convention). Never component-scoped. Sub-grid values (2px, 1px) can exist as primitives but should be used sparingly and documented as exceptions.
- **DTCG:** W3C Community Group format report (not a full W3C Recommendation as of 2026); keys `$value`/`$type`/`$description`; files `.tokens` / `.tokens.json`; media type `application/design-tokens+json`; 10+ tools support it — treat it as the de-facto standard.
- **States to contrast-test:** default, hover, focus, active, visited, disabled, error, success — **each is a new color pairing.**

## Tools & libraries
- **Style Dictionary** (Amazon, OSS, `style-dictionary/style-dictionary`) — the build engine: token JSON → CSS vars / Swift / Android XML / Dart / custom. v4 = DTCG-native. **Use for** any multi-platform or even single-platform build you want reproducible. Tokens Studio ships `sd-transforms` as a companion package to bridge Figma Variables → DTCG → Style Dictionary pipelines.
- **Tokens Studio for Figma** — manage tokens *in* Figma; multi-brand/multi-mode token sets; bridges Figma ↔ code. **Use when** design owns token authoring.
- **Figma Variables** — native semantic token management; REST + Plugin API for bidirectional sync; powers theming/localization/A-B workflows. **Caveat:** not 1:1 with code tokens.
- **zeroheight** (paid, check current pricing) — polished design+stakeholder docs fast; Figma sync, code/Storybook embed, version drafts, inspect mode. **Use for** non-technical audiences. More mature community than Supernova.
- **Supernova** (enterprise) — automated Figma→CSS/iOS/Android token pipeline, component-health tracking. **Use when** you need token-to-code automation across platforms. **NOT for** quick starts: setup is **days, not hours.**
- **Storybook** (free, OSS) — developer-facing component playground; best embedded inside zeroheight/Supernova. **NOT** a complete docs solution for non-technical stakeholders; needs separate hosting (Chromatic or custom).
- **Stark (Figma) / Color Oracle** — color-blindness sim + WCAG contrast validation during design.
- **Benchmarks to study:** Carbon (IBM) — clean three-tier separation, multi-platform; Adobe Spectrum — family+50–900 naming; Intuit DS (Mailchimp/QuickBooks/TurboTax/Mint) — multi-product taxonomy.
- **When NOT to reach for any of these:** an MVP — a Tailwind config *is* your token layer; don't stand up Supernova for a landing page.

## Do / Don't
**Do**
- Reference semantic/component tokens in product code; lint to forbid primitives outside the token file.
- Name by intent; keep semantic names theme-agnostic.
- Separate brand and functional color families.
- Keep spacing tokens global/primitive on a 4px grid.
- Split token files by category (color, typography, spacing, elevation, motion).
- Use `rem` for fontSize; use DTCG `fontFamily`, `fontSize`, `fontWeight`, `lineHeight` types.
- Tokenise elevation separately for light (shadow) and dark (luminance).
- Set `outputReferences: true` so themes stay remappable.
- Define 4 luminance-stepped dark surfaces; validate contrast per mode and per state.
- Pin token versions via an internal package; apply semver; deprecate with a migration path ≥2 minor releases.
- Document every component with all 6 required elements before marking it "done".

**Don't**
- Don't let `blue-500` (primitive) leak into a button's CSS.
- Don't name semantic tokens by color or by light/dark assumption.
- Don't mint component tokens for every trivial property (sprawl) or spacing aliases (confusion).
- Don't invert colors for dark mode or rely on drop shadows for dark elevation.
- Don't mix Style Dictionary v3 and v4 token formats in one instance.
- Don't conflate tokens with a full design system; don't ship a DS with no evolution budget.
- Don't assume Figma Variables == code tokens.
- Don't assume a hue's 700 step passes 4.5:1 — always measure; warm hues fail earlier.

## Common pitfalls & how to avoid them
- **Aliasing collapse in build output** → set `outputReferences: true`, correct `$type`/`$value`; verify the CSS output still has `var(...)`, not resolved hex.
- **Appearance-based names** (`color-blue-500` as a semantic) → rename to intent (`color-bg-primary`) before scale forces a painful migration.
- **Component-token sprawl** → only create a component token when it must diverge from the semantic layer; default to semantic.
- **Hard-coded color sprawl blocking dark mode** → centralize to tokens; scattered hex values make dark mode and rebranding require codebase-wide find-and-replace.
- **DS as a one-time deliverable** → fund ongoing evolution or don't start; a frozen DS dies.
- **Juniors building the DS solo** → pair with seniors; edge cases + adoption depend on it.
- **Dark elevation via shadow** → use luminance stepping (4 surfaces, +5–8% each).
- **Brand red == error red** → split families.
- **Figma Variables ↔ tokens assumed 1:1** → reconcile via `sd-transforms`/pipeline, document the mapping.
- **No dev adoption** → enforce via PR checks/linting; track real usage; keep DS engineers staffed.
- **Atomic-Design debate paralysis** → adopt Styles/Components/Patterns; ship product.
- **Only default state contrast-tested** → test all 8 interactive states against 4.5:1 / 3:1.
- **Assuming warm hues pass at 700** → reds and yellows can fail 4.5:1 at 600 or below; always measure.
- **Typography as raw values** → tokenise fontFamily/fontSize/fontWeight/lineHeight; use rem not px for font sizes.
- **All tokens in one file** → split by category; large monolithic token files have merge conflicts and unclear ownership.
- **Silent token removal** → any removal is a breaking change; always go through the deprecation protocol first.
- **Multi-brand as theme** → light/dark is a value-set swap; brand is a primitive-set swap; conflating them causes semantic name collisions.

## Recurring practitioner pain points
These patterns surface repeatedly in DS community discussions (Slack communities, Discord DS channels, conference talks) — not formal citations, but consistent signals worth knowing:

- **Figma Variables ↔ Tokens Studio mismatch** is the most-cited friction. Both systems model "tokens" differently; a mode-swap in Figma does not automatically equal a theme swap in code without a reconciliation step.
- **Component-level spacing aliases** consistently cause confusion with little benefit; practitioners on multiple production DSes converge on keeping spacing tokens global/primitive.
- **Invisible duplication cost.** When a DS has no adoption, dev teams build their own components silently. Management sees no explicit cost line, so no one funds the DS fix — the cost is real but hidden.
- **Over-engineered hierarchies.** When token nesting exceeds 6 levels, contributors stop using the system correctly and start writing ad-hoc overrides.
- **Aliasing collapse surprise.** After Style Dictionary processes a meticulous 3-tier system, the CSS output shows raw hex — the developer assumed references were preserved by default. They're not; `outputReferences: true` is opt-in.
- **Atom vs. molecule debate paralysis.** Teams lose hours on Atomic Design classification instead of shipping. Styles/Components/Patterns sidesteps the debate.

## Senior-level checklist
- [ ] Three tiers defined; product code references **only** semantic/component tokens (lint-enforced).
- [ ] No primitive token name appears outside its definition file.
- [ ] All semantic names are intent-based and theme-agnostic (survive a rebrand).
- [ ] Brand and functional/status color families are separate.
- [ ] Spacing/sizing tokens are global/primitive on a 4px grid; no component spacing aliases.
- [ ] Color scale is 10-step (50–900) with half-steps for dark tuning; contrast verified per hue (not assumed from weight).
- [ ] Typography tokens cover fontFamily, fontSize (rem), fontWeight, lineHeight, letterSpacing.
- [ ] Elevation tokens exist for both light (shadow) and dark (luminance); semantic names like `elevation.card` not raw shadow values.
- [ ] Token files split by category (color, typography, spacing, elevation, motion).
- [ ] Dark mode = its own value set: off-white text, near-black bg, 4 luminance-stepped surfaces, no shadow elevation.
- [ ] Contrast validated in **both** modes for **all 8** interactive states (4.5:1 / 3:1 WCAG 2.2 AA).
- [ ] DTCG format (`$value`/`$type`/`$description`); single Style Dictionary major version.
- [ ] Build preserves references (`outputReferences: true`) so themes remain remappable.
- [ ] Pipeline: validate → transform → version → publish; semver applied; deprecation has migration path (≥2 minor releases).
- [ ] Governance: ownership, contribution model, PR/lint enforcement; not staffed by juniors alone; evolution budget exists.
- [ ] Multi-brand: separate primitive files per brand; shared semantic layer references active brand only.
- [ ] Each component has all 6 doc elements (props, states, a11y, do/don't, playground, related).
- [ ] Scope is right-sized: token sheet/Tailwind for MVP; full DS only when duplication > maintenance cost.
- [ ] Components + patterns + docs exist — not just tokens.

## Sources
- Style Dictionary — https://styledictionary.com (repo: https://github.com/style-dictionary/style-dictionary)
- W3C Design Tokens Community Group spec (community-group report, not a full W3C Recommendation) — https://www.w3.org/community/design-tokens/ · format: https://tr.designtokens.org/format/
- Tokens Studio for Figma — https://tokens.studio
- IBM Carbon Design System — https://carbondesignsystem.com
- Adobe Spectrum — https://spectrum.adobe.com
- WCAG 2.2 — https://www.w3.org/TR/WCAG22/
- APCA (Advanced Perceptual Contrast Algorithm) — https://github.com/Myndex/SAPC-APCA
- zeroheight — https://zeroheight.com · Supernova — https://supernova.io · Storybook — https://storybook.js.org
- Feature-Sliced Design — https://feature-sliced.design
- Intuit Design System — https://designsystem.quickbooks.com

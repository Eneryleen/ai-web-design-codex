# AGENTS.md — Machine Index for the AI Web Design Codex

You are an AI agent. **This file is your entry point.** The codex has **60 guides (~23k lines)** — do **not** read them all. Instead:

1. Identify the task — what kind of site/surface, and what problem you're solving.
2. Pick a **Reading path** below, or use the **Full index** to select the 2–5 most relevant guides.
3. Read only those guides, then design/build. Each guide ends with a **Senior-level checklist** — verify your output against it before declaring done.

All paths are relative to the repository root.

## How each guide is structured

Every guide follows the same skeleton, so you can skim predictably:

`# Title` → `> one-line summary` → **Apply when / Related** → **TL;DR** → core sections → **Concrete specs & numbers** → **Tools & libraries** → **Do / Don't** → **Common pitfalls** → **What real users complain about** → **Senior-level checklist** → **Sources**.

If you only have budget for one section, read **TL;DR** + **Senior-level checklist**.

## 🧭 Reading paths (task → guides, in order)

**Build a high-converting landing page**
`09-playbooks-and-checklists/playbook-landing-page.md` → `03-site-types/landing-pages.md` → `06-marketing-and-conversion/conversion-rate-optimization.md` → `06-marketing-and-conversion/copywriting-and-ux-writing.md` → `06-marketing-and-conversion/persuasion-psychology.md` → `07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md` → `06-marketing-and-conversion/web-performance-core-web-vitals.md` → `01-ux-fundamentals/accessibility-wcag.md`

**Build a "magic scroll" one-pager (award-style)**
`09-playbooks-and-checklists/playbook-scroll-experience.md` → `03-site-types/scroll-driven-experiences.md` → `03-site-types/single-page-one-pagers.md` → `04-libraries-and-tools/animation-libraries-stack.md` → `04-libraries-and-tools/frameworks-for-design.md` → `07-aesthetics-and-trends/micro-interactions-and-motion.md` → `07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md` → `06-marketing-and-conversion/web-performance-core-web-vitals.md` → `01-ux-fundamentals/accessibility-wcag.md`

**Build a CRM / admin dashboard (data-dense)**
`09-playbooks-and-checklists/playbook-crm-dashboard.md` → `03-site-types/crm-admin-dashboards.md` → `00-foundations/data-visualization-and-charts.md` → `01-ux-fundamentals/information-architecture-navigation.md` → `01-ux-fundamentals/forms-and-input-ux.md` → `04-libraries-and-tools/component-libraries-headless.md` → `04-libraries-and-tools/native-interactive-ui-primitives.md` → `01-ux-fundamentals/accessibility-wcag.md`

**Build a SaaS app surface (logged-in product)**
`03-site-types/saas-web-apps.md` → `01-ux-fundamentals/authentication-and-account-flows.md` → `01-ux-fundamentals/cognitive-load-progressive-disclosure.md` → `01-ux-fundamentals/forms-and-input-ux.md` → `05-process-and-systems/design-systems-and-tokens.md` → `04-libraries-and-tools/component-libraries-headless.md` → `06-marketing-and-conversion/lifecycle-email-and-notification-design.md` → `06-marketing-and-conversion/analytics-instrumentation-and-measuring-impact.md`

**Build an e-commerce store**
`03-site-types/ecommerce-ux.md` → `01-ux-fundamentals/forms-and-input-ux.md` → `06-marketing-and-conversion/conversion-rate-optimization.md` → `06-marketing-and-conversion/persuasion-psychology.md` → `08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md` → `02-responsive-adaptive/touch-ergonomics-cross-device.md` → `06-marketing-and-conversion/web-performance-core-web-vitals.md`

**Build a chat / AI-assistant interface**
`03-site-types/conversational-and-ai-native-interfaces.md` → `01-ux-fundamentals/interaction-design-and-states.md` → `01-ux-fundamentals/cognitive-load-progressive-disclosure.md` → `04-libraries-and-tools/native-interactive-ui-primitives.md` → `01-ux-fundamentals/accessibility-wcag.md`

**Build a brand / marketing site**
`03-site-types/marketing-brand-sites.md` → `07-aesthetics-and-trends/design-styles-taxonomy.md` → `07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md` → `00-foundations/typography-systems.md` → `00-foundations/color-theory-and-systems.md` → `05-process-and-systems/design-systems-and-tokens.md` → `06-marketing-and-conversion/seo-and-discoverability.md` → `06-marketing-and-conversion/web-performance-core-web-vitals.md`

**Make any UI look less "AI-generated" / raise visual craft**
`07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md` → `07-aesthetics-and-trends/design-styles-taxonomy.md` → `00-foundations/typography-systems.md` → `00-foundations/color-theory-and-systems.md` → `00-foundations/layout-grids-and-spacing.md` → `00-foundations/visual-hierarchy.md` → `07-aesthetics-and-trends/micro-interactions-and-motion.md`

**Go multilingual / global**
`01-ux-fundamentals/internationalization-and-localization.md` → `00-foundations/multilingual-and-non-latin-typography.md` → `06-marketing-and-conversion/copywriting-and-ux-writing.md` → `01-ux-fundamentals/forms-and-input-ux.md`

**Pre-launch QA / review any site**
`09-playbooks-and-checklists/master-checklists-cheatsheets.md` → `08-pitfalls-and-antipatterns/common-mistakes-antipatterns.md` → `08-pitfalls-and-antipatterns/real-user-complaints.md` → `01-ux-fundamentals/accessibility-wcag.md` → `06-marketing-and-conversion/web-performance-core-web-vitals.md` → `06-marketing-and-conversion/seo-and-discoverability.md` → `08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md`

**Start any project from zero (process)**
`05-process-and-systems/design-process-and-stages.md` → `01-ux-fundamentals/user-research-and-testing.md` → `01-ux-fundamentals/information-architecture-navigation.md` → `05-process-and-systems/wireframing-prototyping-handoff-qa.md` → `05-process-and-systems/design-systems-and-tokens.md`

> Always include `01-ux-fundamentals/accessibility-wcag.md` and `06-marketing-and-conversion/web-performance-core-web-vitals.md` for any production build — they apply to almost everything.

## 📂 Full index — all 60 guides

Each entry: **title** — `path`, one-line summary, and when to use it.

## 00-foundations
- **Color Theory & Color Systems** — `00-foundations/color-theory-and-systems.md`
  - How to choose, structure, and ship color for digital products: perceptually-uniform color models (OKLCH/OKLab), accessible token architectures, WCAG/APCA contrast, dark mode, and gradients. The senior-level outcome: a token-driven palette that is consistent across hues, passes contrast in both light and dark mode, and is maintainable from primitive values up to component overrides.
  - Use when: picking any color, building a palette/theme, adding dark mode, or auditing contrast
- **Data Visualization & Charts** — `00-foundations/data-visualization-and-charts.md`
  - How to choose, render, annotate, and make accessible the charts on a website — across functional dashboards AND editorial/marketing storytelling. Covers chart-to-question matching, honest axes and scales (zero-baseline, truncation, log, lie factor), data color (sequential/diverging/categorical, colorblind-safe, never color-only), the annotation layer, interaction with keyboard/touch parity, chart accessibility, scrollytelling, sparkline/bullet/big-number patterns, and library selection. The senior-level outcome: charts that answer the right question, never mislead, work for keyboard/screen-reader/colorblind users, and use a library appropriate to the data scale.
  - Use when: designing any chart, dashboard metric, data story, or scrollytelling visual
- **Imagery, Icons & Visual Assets** — `00-foundations/imagery-icons-and-assets.md`
  - How to use photography, illustration, and icons so they load fast, render crisp, crop intelligently, stay accessible, and look intentional rather than generic. Master this and your sites hit Core Web Vitals targets, avoid layout shift, and read as authentic instead of stock-clichéd.
  - Use when: any page that ships a raster image, illustration, SVG icon, or hero
- **Layout, Grids & Spacing Systems** — `00-foundations/layout-grids-and-spacing.md`
  - How to structure a page with grids, distribute space with a single repeatable scale, and align elements so the result reads as deliberate rather than accidental. Master this and your layouts gain the calm, machine-precise rhythm that separates senior work from "developer default" — without a single arbitrary pixel value.
  - Use when: Laying out any page or component, choosing spacing/sizing values, or fixing a layout that "feels off."
- **Multilingual & Non-Latin Typography** — `00-foundations/multilingual-and-non-latin-typography.md`
  - How to set type across writing systems beyond Latin — CJK, Arabic, Devanagari, Thai, Hebrew, Cyrillic, Greek — without "tofu", broken shaping, multi-MB font downloads, or interline collisions. Read this to make defensible per-script decisions about font stacks, fallback chains, leading, line-breaking, RTL mirroring, and `unicode-range` delivery architecture, instead of pasting Latin defaults onto scripts they were never designed for.
  - Use when: any site that renders, or might one day render (CMS, UGC, i18n), text in a non-Latin script.
- **Web Typography Systems** — `00-foundations/typography-systems.md`
  - How to architect a production typographic system for the web: modular type scales, measure and leading as co-dependent variables, fluid sizing with `clamp()`, font loading and CLS control, OpenType features, and vertical rhythm. Read this to make defensible numeric decisions (ratios, line lengths, line-heights, weights) instead of eyeballing, and to ship type that is fast, accessible, and legible across devices.
  - Use when: designing or implementing any text-bearing page or design system.
- **Visual Design Principles & Gestalt** — `00-foundations/visual-design-principles.md`
  - The grammar of visual composition: how balance, contrast, emphasis, scale, proximity, alignment, repetition, and white space — plus the Gestalt grouping laws the brain runs automatically — combine into layouts that read as clear, intentional, and beautiful. Master this and your screens feel designed rather than assembled; ignore it and even pixel-perfect components add up to noise the user cannot parse.
  - Use when: composing any layout — deciding what groups with what, what dominates, and where the eye lands.
- **Visual Hierarchy & Scanning Patterns** — `00-foundations/visual-hierarchy.md`
  - How to arrange page elements by importance so users see the right things in the right order — using size, weight, color, contrast, spacing, position, and motion. Master this and your layouts guide the eye to the value proposition and the primary action without the user thinking about it; ignore it and even technically-correct screens feel like a flat gray wash with no clear next step.
  - Use when: designing any screen, page, or component where a user must orient, decide, or act.

## 01-ux-fundamentals
- **Accessibility (WCAG 2.2) for Designers & Devs** — `01-ux-fundamentals/accessibility-wcag.md`
  - How to design and build to WCAG 2.2 Level AA — the legal baseline in the US (Section 508/ADA), EU (EN 301 549, European Accessibility Act), and Canada (AODA). Covers POUR, semantic HTML, keyboard/focus, ARIA discipline, contrast, text alternatives, motion, target sizes, forms, and screen-reader testing. Senior outcome: ship interfaces that are operable by keyboard and screen reader, pass automated *and* manual audits, and don't generate ADA lawsuit exposure.
  - Use when: every project, from design phase onward (retrofitting costs 10×)
- **Authentication & Account Flows** — `01-ux-fundamentals/authentication-and-account-flows.md`
  - Authentication is the highest-stakes, highest-abandonment funnel on most products: a single bad reset email or leaky error message loses high-intent users or invites account takeover. This guide gives an AI builder the exact tokens, thresholds, OWASP-aligned rules, and code patterns to ship sign-up, login, recovery, MFA, OAuth, sessions, and account-deletion flows that are secure, accessible, and low-friction at a senior level — with passkeys as the 2026 default.
  - Use when: building any login, sign-up, password reset, MFA, SSO, or account-settings flow
- **Cognitive Load & Progressive Disclosure** — `01-ux-fundamentals/cognitive-load-progressive-disclosure.md`
  - How to make interfaces feel effortless by managing the user's finite mental bandwidth: chunk information, defer secondary features, prefer sensible defaults, convert recall into recognition, and cut the number of decisions a screen demands. Master this and your designs feel "obvious" — users complete tasks faster, with fewer errors and less abandonment, without losing capability for power users.
  - Use when: any screen with >1 decision, >1 choice, or >5 fields — onboarding, forms, settings, dashboards, checkout, pricing, empty states
- **Forms & Input UX** — `01-ux-fundamentals/forms-and-input-ux.md`
  - Forms are where conversion is won or lost: checkout, signup, lead capture, and settings all live or die on input UX. This guide gives an AI builder the rules, numbers, and code patterns to ship forms that complete fast, validate humanely, and survive mobile, autofill, and accessibility scrutiny at a senior level.
  - Use when: building any form — signup, login, checkout, contact, settings, multi-step flow
- **Information Architecture & Navigation** — `01-ux-fundamentals/information-architecture-navigation.md`
  - How to organize site content (the invisible structural backbone) and expose it through navigation UI (the visible layer) so users can find anything in ≤ 3 clicks, always know where they are, and trust where each label leads. Senior outcome: an IA + nav system that survives search-engine deep links, scales past 50 pages without becoming a swamp, and measurably raises task success — not just "fewer clicks."
  - Use when: designing any site's structure, menus, breadcrumbs, search, or mobile nav
- **Interaction Design & UI States** — `01-ux-fundamentals/interaction-design-and-states.md`
  - How interactive elements communicate what they do, respond to input, and behave across their full lifecycle. Covers affordances/signifiers, feedback timing, the nine UI states every component needs, microinteractions, optimistic UI, perceived performance, focus management, and keyboard interaction. The senior-level outcome: components that feel responsive, are discoverable, and never leave a user (or a screen reader) guessing.
  - Use when: designing or building any interactive component, screen, or flow.
- **Internationalization (i18n) & Localization (l10n)** — `01-ux-fundamentals/internationalization-and-localization.md`
  - How to design and build layouts that survive translation and locale switching: text-expansion budgets, full RTL support, locale-aware `Intl` formatting, language-switcher UX, and pseudo-localized QA. The senior-level outcome: one codebase that adapts to any market without per-locale CSS forks, truncation, or mixed-locale bugs.
  - Use when: building anything that may ship in more than one language, region, or currency — even if it launches monolingual.
- **The Laws of UX & Design Psychology** — `01-ux-fundamentals/laws-of-ux.md`
  - The reusable cognitive and perceptual rules that govern how humans read, decide, remember, and act inside an interface. Apply these to make layouts faster to scan, choices faster to make, and flows that feel effortless — the difference between a UI that merely works and one that converts and retains at a senior level.
  - Use when: any screen, flow, navigation, CTA, form, or pricing decision — i.e. always
- **Nielsen's Usability Heuristics & Practical Usability** — `01-ux-fundamentals/nielsen-heuristics.md`
  - The 10 heuristics are general principles for interaction design, derived by Nielsen & Molich (1990, refined 1994) from factor analysis of 249 usability problems. Language updated 2020. They are grounded in human psychology, not technology, so they apply to any web UI. This guide turns each into concrete, buildable web patterns plus a step-by-step heuristic evaluation method you can run on your own output.
  - Use when: designing or auditing any interactive UI; self-reviewing a build before handoff
- **User Research & Usability Testing** — `01-ux-fundamentals/user-research-and-testing.md`
  - Lightweight research that makes design decisions defensible instead of guessed. Covers when to interview, survey, test, A/B, or read analytics; how many users each method needs; and how to run them without the classic statistical and recruiting traps. The senior outcome: you pick the cheapest method that answers the actual question, run it correctly, and turn raw sessions into design changes — not a deck that gathers dust.
  - Use when: before building (discovery), during design (validation), and post-launch (optimization); any time a decision rests on an assumption about users

## 02-responsive-adaptive
- **Breakpoints, Devices & Mobile-First** — `02-responsive-adaptive/breakpoints-devices-mobile-first.md`
  - How to choose breakpoints from content (not device models), apply mobile-first vs desktop-first deliberately, and reason about the real device landscape in CSS pixels. The senior outcome: layouts that hold across hundreds of viewport widths with the minimum number of breakpoints and zero device-specific hacks.
  - Use when: designing any responsive layout, choosing breakpoints, or deciding mobile-first vs desktop-first.
- **Fluid Typography & Spacing with clamp()** — `02-responsive-adaptive/fluid-type-and-spacing.md`
  - How to build type and spacing scales that interpolate smoothly across viewports with a single `clamp()` rule per token — eliminating per-breakpoint `font-size`/`margin` overrides — while staying fully WCAG-compliant for zoom and reflow. Master this to ship a coherent, breakpoint-light design system where headings, body, and spacing all scale in one proportional language from a 320px phone to a 1500px+ desktop.
  - Use when: defining any type scale or spacing system; replacing per-breakpoint `font-size`/padding rules; building a token-driven design system.
- **Responsive & Fluid Design** — `02-responsive-adaptive/responsive-and-fluid-design.md`
  - The discipline of building layouts that adapt to *any* viewport and *any* container, driven by content needs and user preferences rather than fixed device widths. Master this to ship interfaces that read well from a 320px phone to a 4K monitor, respect user zoom/font-size settings, and use modern CSS (container queries, `clamp()`, intrinsic Grid, dynamic viewport units) instead of breakpoint sprawl.
  - Use when: building any production web layout, page, or reusable component.
- **Touch Ergonomics & Cross-Device Design** — `02-responsive-adaptive/touch-ergonomics-cross-device.md`
  - How to design and build interfaces that work for fingers and thumbs, not just mouse pointers — sizing touch targets in physical space, placing actions in thumb-reachable zones, detecting *input type* (not screen size), respecting notches/safe areas, and keeping scroll smooth on cheap hardware. Master this to ship one codebase that feels native on a $90 Android phone, an iPad, a touchscreen laptop, and a desktop with a mouse.
  - Use when: designing or building any page or component that real people touch — i.e., almost everything in 2026+.

## 03-site-types
- **Content Sites: Blogs, Magazines & Docs** — `03-site-types/content-blogs-and-docs.md`
  - Reading-first interfaces where typography *is* the UI: blogs, editorial magazines, and technical documentation. This guide gives the concrete measure/leading/scale numbers, the Diataxis IA model, code-block and search patterns, and the dark-mode reality so you can build long-form sites that people actually finish reading.
  - Use when: building a blog, news/magazine, knowledge base, or developer docs — any site where the primary job is reading text.
- **Chat, Conversational & AI-Native Interfaces** — `03-site-types/conversational-and-ai-native-interfaces.md`
  - Patterns for designing and building chat and AI-native product surfaces end to end: message-thread layout, token-by-token streaming with stop/regenerate, the input composer, suggested prompts and first-run states, rich response rendering (markdown, code, tables, citations, tool calls, reasoning), latency masking, error/retry/rate-limit handling, message actions, conversation history, trust-and-safety affordances, and the accessibility of a live-updating transcript. The senior-level outcome is a chat surface that streams smoothly, never thrashes layout, masks latency honestly, calibrates trust instead of inflating it, and remains usable with a keyboard and a screen reader while text is still arriving.
  - Use when: building a chat/assistant surface, an agentic tool, or any UI where an LLM streams responses into a transcript
- **CRM & Admin Dashboards (Data-Dense UI)** — `03-site-types/crm-admin-dashboards.md`
  - How to design and build data-dense internal tools — CRMs, admin panels, analytical dashboards, and large data tables — where users spend hours per day and efficiency is a hard requirement, not a polish item. Senior outcome: interfaces that stay legible and fast at 50+ rows × 20 columns, give every number actionable context, and respect power-user workflows (keyboard, pinning, saved views) without overwhelming casual users.
  - Use when: building internal tools, admin panels, CRMs, or analytical dashboards with high information density
- **E-commerce UX** — `03-site-types/ecommerce-ux.md`
  - Patterns for the conversion-critical surfaces of an online store: product listing, product detail, search, cart, and checkout, plus mobile commerce, reviews, price display, and abandonment recovery — grounded in Baymard Institute large-scale research. The senior-level outcome is a store that meets the Amazon/Walmart benchmarks users already expect (Jakob's Law), removes friction at every funnel step, and converts at rates close to the desktop ceiling on every device.
  - Use when: building or auditing any transactional store (catalog → PDP → cart → checkout)
- **Landing Pages** — `03-site-types/landing-pages.md`
  - A landing page is a single-purpose, traffic-destination page engineered around ONE conversion goal. This guide gives an AI builder the anatomy, copy formulas, layout rhythm, trust mechanics, and numeric targets to design and ship landing pages that convert at or above industry medians — including B2B vs B2C divergence and lead-form vs click-through patterns.
  - Use when: building any paid-traffic / campaign / product-launch destination page with a single goal.
- **Marketing & Brand Sites** — `03-site-types/marketing-brand-sites.md`
  - A marketing/brand site is a multi-page property whose job is to express identity, build trust, and convert — simultaneously. Unlike a single-purpose landing page, it carries the whole brand: homepage, product/feature pages, pricing, story, proof, and a coherent design system across all of them. This guide gives an AI builder the homepage strategy, narrative arc, multi-page IA, brand-as-system tokens, motion-as-brand discipline, and the Awwwards-polish-vs-usability balance needed to ship agency-grade work that still passes Core Web Vitals.
  - Use when: building a company/product brand site (homepage + IA, not a single campaign page).
- **Portfolios & Personal Sites** — `03-site-types/portfolios-personal-sites.md`
  - How to design and build portfolio and personal sites that convert a 6–8 second scan into an interview or client inquiry. Covers outcome-first case studies, hero/about, visual identity, project pages, and the performance/accessibility floor a senior portfolio must clear. The senior-level outcome: a tight, fast, honest site that sells business impact, not pixels.
  - Use when: building a designer/developer/creative personal site, UX case-study portfolio, or freelance studio site
- **SaaS Web Apps** — `03-site-types/saas-web-apps.md`
  - Patterns for building the authenticated, recurring-use core of a SaaS product: app shell, navigation, onboarding and activation, empty states, settings, billing, in-app guidance, command palettes, multi-step flows, and the trial→paid→retention loop. The senior-level outcome is a product where users reach value fast (low time-to-value), discover features in context, and form habits that convert and retain — without dark patterns.
  - Use when: building or redesigning the logged-in surface of a subscription product (dashboard app, workspace, tool)
- **Scroll-Driven Experiences (Magic Scroll, Pinning, Parallax)** — `03-site-types/scroll-driven-experiences.md`
  - How to design and build award-tier scroll-driven one-pagers: scroll storytelling, pinning/sticky timelines, parallax depth, scroll-linked animation, and frame sequences — using GSAP ScrollTrigger, Lenis, and native CSS scroll-driven animations. The senior outcome: motion that serves narrative and guides attention, runs jank-free on the compositor thread, and degrades gracefully for reduced-motion users and mobile.
  - Use when: campaign microsites, product showcases, agency/portfolio hero experiences — NOT task-driven flows
- **Single-Page Sites & One-Pagers** — `03-site-types/single-page-one-pagers.md`
  - One-page sites compress an entire story into a single scrollable document: hero → proof → action, navigated by anchors and scrollspy. This guide covers when one-pager scope fits, section ordering, anchor/scrollspy/smooth-scroll mechanics, the load-everything-at-once performance budget, SEO limits and mitigations, accessible focus management, and the long-scroll narrative — so an agent ships a focused, fast, ranking page instead of an unstructured scroll-tunnel.
  - Use when: content scope is one focused goal (portfolio, event, single product, MVP, brand statement) and linear narrative beats a navigation hierarchy.

## 04-libraries-and-tools
- **Web Animation Libraries & Stack** — `04-libraries-and-tools/animation-libraries-stack.md`
  - Decision framework for choosing and combining web animation tech — CSS, WAAPI, View Transitions, GSAP, Motion, Lenis, Anime.js, Lottie, AutoAnimate. Optimizes for the senior outcome: 60 fps, minimal bundle, accessible motion, and the right tool per job instead of reflexively reaching for a library.
  - Use when: adding any motion to a site — micro-interactions, scroll effects, page transitions, layout animations, or motion graphics.
- **Component Libraries & Headless UI** — `04-libraries-and-tools/component-libraries-headless.md`
  - How to choose, customize, and own UI components in 2025-2026: the headless model (behavior/accessibility without styles — Radix, Base UI, React Aria, Headless UI, Ark UI, Ariakit), the copy-in model (shadcn/ui), and batteries-included kits (MUI, Mantine, Chakra). The senior-level outcome: accessible, on-brand components you control, with no vendor lock-in, no generic-library look, and no library-mixing footguns.
  - Use when: Picking a component foundation, building a design system, customizing shadcn/ui, fixing a11y or theming problems, or evaluating bundle/accessibility tradeoffs.
- **CSS, Styling & Tailwind** — `04-libraries-and-tools/css-styling-and-tailwind.md`
  - How to choose and operate a styling architecture in 2025-2026: Tailwind (utility-first), CSS Modules, zero-runtime CSS-in-TS (vanilla-extract/StyleX/Panda), and modern native CSS (nesting, `@layer`, `:has()`, container queries, custom properties). The senior-level outcome: a maintainable, fast, token-driven styling system that survives team scale and is RSC-compatible.
  - Use when: Choosing or wiring up any styling approach, defining design tokens, fixing class-soup/specificity/bundle problems.
- **Design & Dev Tooling** — `04-libraries-and-tools/design-and-dev-tooling.md`
  - The non-runtime tooling that makes great design shippable: Figma (auto-layout, variables, dev mode) and the W3C token pipeline, prototyping, Vite, font subsetting/self-hosting, image pipelines (sharp), Storybook, ESLint/Prettier, Lighthouse/PageSpeed, browser devtools, and color/contrast checkers. The senior-level outcome: a build-and-QA loop where design tokens flow Figma→code, fonts and images are optimized by default, accessibility/perf are measured continuously, and handoff produces real layout behavior — not screenshots.
  - Use when: Setting up a project's build/QA toolchain, wiring design-token sync, optimizing fonts/images, doing design QA on a build, or reading a Figma handoff.
- **Frameworks for Design-Forward Sites** — `04-libraries-and-tools/frameworks-for-design.md`
  - How to choose and configure a framework/meta-framework so a design-forward site loads fast, ranks well, and stays maintainable. Covers Astro, Next.js (App Router), SvelteKit, Nuxt, SolidStart; rendering strategies (SSG/ISR/SSR/PPR/Islands/Server Islands); hydration cost; native View Transitions; and built-in image optimization. Senior outcome: pick the lightest tool that meets the real requirement, render each route the right way, and ship the minimum JS the design needs.
  - Use when: starting a new design-forward site, or auditing why an existing one is slow/heavy/expensive.
- **Native Interactive UI Primitives: Popover, Anchor Positioning, Dialog & inert** — `04-libraries-and-tools/native-interactive-ui-primitives.md`
  - How to build menus, tooltips, dropdowns, modals, command palettes, and toasts on the 2026 platform with zero or minimal JS — using the Popover API, CSS Anchor Positioning, native `<dialog>`, `details`/`summary`, invoker commands (`command`/`commandfor`), and `inert`. The senior-level outcome: top-layer overlays that never clip, never z-index-fight, light-dismiss correctly, animate in and out, and degrade gracefully — while knowing exactly when these replace Floating UI / Radix and when they still do not.
  - Use when: Building any overlay (menu, dropdown, tooltip, modal, toast, picker, command palette), deciding whether to pull in a headless library, or fixing clipping/stacking/focus-trap bugs.
- **WebGL, 3D & Shaders for the Web** — `04-libraries-and-tools/webgl-3d-shaders.md`
  - How to design and ship WebGL/3D experiences that load fast, hold 60fps on real devices, degrade gracefully, and respect accessibility — without turning the site into a battery-draining gimmick. Covers Three.js, React Three Fiber + drei, Spline, GLSL, postprocessing, performance budgets, fallbacks, and scroll integration.
  - Use when: a design calls for 3D scenes, shader effects, particle/fluid/distortion visuals, or 3D product viewers

## 05-process-and-systems
- **The Website Design Process & Stages** — `05-process-and-systems/design-process-and-stages.md`
  - The end-to-end pipeline for taking a website from a vague request to a launched, measured product: discovery → research → content → IA → wireframes → visual design → design system → prototype → usability test → build/handoff → QA → launch → iterate. Read this before designing anything so you solve the right problem in the right order and never skip content or IA — the two stages an AI is most tempted to leap over.
  - Use when: starting any non-trivial website or redesign and deciding what to produce, in what order, with what verification.
- **Design Systems & Design Tokens** — `05-process-and-systems/design-systems-and-tokens.md`
  - How to architect design tokens (primitive → semantic → component), build a token pipeline with Style Dictionary, theme dark mode through tokens, and decide when a full design system is warranted vs. a single token sheet. Outcome: a token architecture that scales to multi-theme, multi-platform products without find-and-replace churn, and the judgment to not over-build it.
  - Use when: building any product that needs consistency across ≥2 surfaces/teams, or any site requiring dark mode/theming
- **Wireframing, Prototyping, Handoff & QA** — `05-process-and-systems/wireframing-prototyping-handoff-qa.md`
  - The pipeline from idea to shipped pixels: choose the right fidelity for the feedback you need, build clickable prototypes that test real behavior, hand off intent (not just coordinates), and run design QA on staging so the build matches the spec. The senior outcome: developers never guess, every state/edge case is documented, accessibility is annotated at handoff, and defects are caught before they cost 10x.
  - Use when: taking a design from sketch to build, writing specs, or QA'ing an implementation against a design

## 06-marketing-and-conversion
- **Analytics Instrumentation & Measuring Design Impact** — `06-marketing-and-conversion/analytics-instrumentation-and-measuring-impact.md`
  - The measurement layer every conversion, activation, and retention guide silently assumes exists. Covers designing a consistent event taxonomy, the tracking plan as a versioned design deliverable, event-vs-property modeling, the GTM/dataLayer-vs-SDK decision, identity stitching, consent-gated firing, metric-model selection (HEART, North Star, AARRR), and instrumentation QA. The senior outcome: ship features whose impact you can actually prove, with data clean enough to trust on the first read.
  - Use when: building any site/app where a stakeholder will later ask "did the redesign work?"
- **Conversion Rate Optimization (CRO)** — `06-marketing-and-conversion/conversion-rate-optimization.md`
  - CRO is the discipline of increasing the share of visitors who complete a defined goal by removing friction, sharpening the value proposition, and validating changes with rigorous experiments. This guide gives you the heuristic frameworks (LIFT, Fogg, MECLABS), the funnel/statistics math, and the concrete element-level wins needed to design conversion-optimized pages and to know when *not* to A/B test.
  - Use when: designing or auditing any page with a measurable goal — landing pages, checkout, signup, pricing, demo requests.
- **Copywriting & UX Writing** — `06-marketing-and-conversion/copywriting-and-ux-writing.md`
  - Words are interface. This guide covers the copy that drives comprehension and conversion: headlines, value propositions, CTAs, and the microcopy on buttons, errors, empty states, tooltips, confirmations, and progress states. The senior-level outcome: every string on the page is scannable, benefit-led, action-oriented, and calibrated for how users actually read (they scan ~20–28% of words on a page — NNG).
  - Use when: writing any user-facing string — headlines, buttons, form labels, errors, empty states, tooltips, hero VP, onboarding, success/confirmation, loading states.
- **Lifecycle, Email & Notification Design** — `06-marketing-and-conversion/lifecycle-email-and-notification-design.md`
  - The design craft behind the highest-converting channel marketers own. Covers responsive HTML email reality (table layouts, ~600px width, inline CSS, dark mode, the two Outlooks, image-off and accessible fallbacks), the core lifecycle programs and how to design each (welcome, abandoned-cart/browse, trial-expiry, win-back, re-engagement), the transactional-vs-marketing distinction, subject-line/preheader/from-name as design surfaces, the deliverability fundamentals a designer must respect, and cross-channel orchestration with push/in-app/SMS. The senior outcome: ship lifecycle emails and notifications that render correctly everywhere, land in the inbox, and convert on intent — not luck.
  - Use when: designing any email, push, in-app, or SMS that a customer receives outside the website itself
- **Ethical Persuasion Psychology** — `06-marketing-and-conversion/persuasion-psychology.md`
  - How to design conversion-positive interfaces using established behavioral science (Cialdini's 7 principles, the Fogg Behavior Model, cognitive biases, nudges) while staying on the ethical side of the line that separates persuasion from dark patterns. The senior-level outcome: you can deploy social proof, scarcity, anchoring, and defaults that lift conversion *and* survive user scrutiny, screenshots, and regulators.
  - Use when: designing any page meant to drive a decision — landing pages, pricing, checkout, signup, onboarding, lead magnets.
- **SEO & Discoverability for Designers** — `06-marketing-and-conversion/seo-and-discoverability.md`
  - How design and front-end decisions make or break organic discoverability: semantic structure, Core Web Vitals, structured data, social cards, rendering architecture, and the design choices that quietly kill rankings. Reading this lets you build sites that are technically indexable, AI-citable, and rich-result eligible by default — without an SEO retrofit later.
  - Use when: building any public-facing page meant to be found via search or shared on social
- **Web Performance & Core Web Vitals** — `06-marketing-and-conversion/web-performance-core-web-vitals.md`
  - How to design and build pages that load fast, respond instantly, and stay visually stable — measured the way Google and real users actually measure them. Covers Core Web Vitals (LCP/INP/CLS), the optimization levers for each, performance budgets with CI enforcement, perceived performance, animation performance, and the direct performance-to-conversion link. Outcome: a senior engineer who ships pages that pass CWV at the 75th percentile in the field, not just in a clean lab run.
  - Use when: building or auditing any production page where speed, ranking, or conversion matters (almost always).

## 07-aesthetics-and-trends
- **Avoiding Generic 'AI' Aesthetics** — `07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md`
  - AI tools default to the statistical median of 2019–2024 web design: centered hero, three feature cards, indigo→purple gradient, Inter. This guide shows how to recognize that "slop" fingerprint and override it at the system level — custom type, committed color, grid tension, intentional motion, real content — so output reads as *designed, not generated*. The senior-level outcome: every visual decision traces to a deliberate reason, and the page is distinguishable from a template.
  - Use when: building any public-facing or brand site, or when a generated UI looks "fine but forgettable."
- **A Taxonomy of Web Design Styles** — `07-aesthetics-and-trends/design-styles-taxonomy.md`
  - A reference catalog of the visual styles a senior web designer chooses between: what each one is, when it fits, how to execute it in real CSS/HTML, what it costs in usability and accessibility, and which industries it serves. Use it to pick a style deliberately — because it reinforces the content and audience — not because it is trending.
  - Use when: picking or justifying a visual direction before wireframing/build
- **Micro-interactions & Motion Design** — `07-aesthetics-and-trends/micro-interactions-and-motion.md`
  - Motion is a usability tool, not decoration. This guide gives you the decision rules, easing curves, duration tokens, choreography patterns, accessibility gates, and performance constraints to add animation that confirms actions, preserves spatial continuity, and directs attention — without inducing jank, latency, or vestibular harm.
  - Use when: building any interactive UI — buttons, modals, route changes, scroll reveals, loaders.
- **Web Design Trends 2025-2026** — `07-aesthetics-and-trends/trends-2025-2026.md`
  - Maps the current and emerging visual/interaction trends (bento, big type, scroll-driven storytelling, 3D/WebGL, motion-first, AI personalization, dark/luxe, expressive Material 3, anti-AI handcraft, micro-typography, custom cursors, grain, variable fonts) and tells you which are durable, which are fads, and exactly how to apply each without producing template-grade output. Outcome: a senior designer/agent can pick trends deliberately, implement them performantly, and avoid the now-legible "AI-generated" look.
  - Use when: choosing an aesthetic direction or deciding whether a requested trend is worth implementing

## 08-pitfalls-and-antipatterns
- **Common Design Mistakes & Antipatterns** — `08-pitfalls-and-antipatterns/common-mistakes-antipatterns.md`
  - A field guide to the recurring, measurable mistakes that wreck usability, accessibility, and conversion: low-contrast text, tiny tap targets, killed focus states, carousels, scroll hijacking, autoplay, intrusive modals, footer-trapping infinite scroll, mystery-meat nav, spacing/typeface chaos, layout-shift jank, broken tables, placeholder-as-label, and unexplained disabled states. Each entry gives the spec, the threshold, the failure mode, and the correct pattern so you can avoid shipping them at a senior level.
  - Use when: designing or reviewing any web UI before ship
- **Dark Patterns to Avoid** — `08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md`
  - Catalogs the deceptive design patterns regulators now treat as illegal — roach motel, forced continuity, confirmshaming, sneak-into-basket, hidden costs, fake urgency, privacy zuckering, and hard-to-cancel flows — with the GDPR/FTC/DSA rules that govern them. Outcome: you can build flows that convert through clarity and honest defaults instead of manipulation, avoiding fines, churn, and the trust collapse that follows enforcement.
  - Use when: designing pricing, checkout, signup, consent, cancellation, or any conversion-critical flow
- **What Real Users Complain About (Synthesis)** — `08-pitfalls-and-antipatterns/real-user-complaints.md`
  - A synthesis of what real people actually rage about online — cookie walls, slow loads, popups before value, autoplay, gray-on-white text, broken mobile, sticky chrome eating the viewport, footer-trapping infinite scroll, forced login/app installs, hidden contact info, dead search, form resets, and jank. Each complaint is paired with the underlying design law and the corrective spec, so you can build sites that the same users would not complain about.
  - Use when: designing, reviewing, or auditing any public-facing site before ship

## 09-playbooks-and-checklists
- **Master Checklists & Cheat-Sheets** — `09-playbooks-and-checklists/master-checklists-cheatsheets.md`
  - A fast-lookup reference an AI consults before designing and after building. Condensed pre-launch checklists (design, responsive, a11y, performance, SEO, content, analytics, security), a senior design-review rubric, quick-reference numbers for spacing/type/color, the squint test, and a decision tree for "what site type and stack do I need." This is the index of indexes — drill into the linked guides for depth.
  - Use when: scoping a build, reviewing your own output, or doing final pre-launch QA
- **Playbook: Design a CRM / Admin Dashboard** — `09-playbooks-and-checklists/playbook-crm-dashboard.md`
  - A decision-by-decision playbook for building a data-dense CRM/admin dashboard: map roles and jobs before pixels, define role-based IA, design the three core screen archetypes (list, detail, overview), engineer data tables and filters, choose KPIs and charts that survive colorblindness and zoom, handle every UI state, pick and theme a component library, hit WCAG 2.2 AA, and scale tables from 1K to 100K+ rows. Following it produces an admin panel that every role can use on day one — not a one-size-fits-none that fails all roles simultaneously.
  - Use when: building an internal/B2B tool where users return daily to scan, filter, act on, and drill into structured business data.
- **Playbook: Design & Build a Landing Page** — `09-playbooks-and-checklists/playbook-landing-page.md`
  - An end-to-end, decision-by-decision playbook for taking a landing page from brief to launch: define a single goal, build a message map, wireframe in question-flow order, design hero/proof/CTA, write copy at the right reading level, add safe motion, ship responsive + fast + measured, and A/B test. Following it produces a page that competes at the top quartile (10%+ conversion) rather than the 6.6% median.
  - Use when: building a single-purpose conversion page for one offer + one traffic source.
- **Playbook: Build a Scroll-Driven One-Pager** — `09-playbooks-and-checklists/playbook-scroll-experience.md`
  - An end-to-end, step-by-step playbook for building a magic-scroll, award-tier one-page experience: write the narrative, storyboard the beats, pick the right stack, implement pinning/parallax/reveal, hit Core Web Vitals, gate everything behind `prefers-reduced-motion`, simplify on mobile, QA, and ship. Outcome: a senior-level scrollytelling site that animates only `transform`/`opacity`, stays legible at every scroll position, and degrades gracefully.
  - Use when: building a marketing one-pager, portfolio, or product story where scroll *is* the interaction.

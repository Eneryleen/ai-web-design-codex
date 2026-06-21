# 🎨 AI Web Design Codex

> A senior-level knowledge base for designing and building **beautiful, usable, accessible, high-converting** websites — **60 dense, cross-linked guides** spanning visual design, UX, responsive engineering, every major site type, the modern frontend stack, marketing & conversion, aesthetics, and anti-patterns.

[![Guides](https://img.shields.io/badge/guides-60-2563eb)](#-table-of-contents)
[![Content](https://img.shields.io/badge/content-23k%2B%20lines-16a34a)](#)
[![Cross-links](https://img.shields.io/badge/internal%20links-482%20%C2%B7%200%20broken-7c3aed)](#)
[![Language](https://img.shields.io/badge/language-English-64748b)](#)
[![Built for](https://img.shields.io/badge/built%20for-humans%20%26%20AI%20agents-f59e0b)](#-how-to-use-this-repo)
[![License: CC BY 4.0](https://img.shields.io/badge/license-CC%20BY%204.0-d6336c)](LICENSE)

---

## Overview

This repository is a structured, opinionated **design + UX + marketing handbook**. Every guide is written to a consistent, scannable skeleton — *TL;DR → core concepts → concrete specs & numbers → do / don't → pitfalls → what real users complain about → senior checklist → sources* — so you can jump in anywhere and walk away with **specific, applicable rules** (real numbers, exact tokens, correct code), not vague advice.

It is dual-purpose:

- **For people** — a reference you browse by topic when designing or reviewing a site.
- **For AI agents** — a knowledge base an LLM reads *selectively* to design at a senior level. See [How to use this repo](#-how-to-use-this-repo) and [`AGENTS.md`](AGENTS.md).

## What's inside

| #  | Section | Covers | Guides |
|----|---------|--------|:------:|
| 00 | 🎨 **Foundations** | visual principles & Gestalt, hierarchy, typography, color, grids & spacing, imagery, data-viz | 8 |
| 01 | 🧭 **UX Fundamentals** | laws of UX, Nielsen heuristics, research, IA & navigation, cognitive load, interaction states, forms, accessibility (WCAG 2.2), i18n, auth flows | 10 |
| 02 | 📐 **Responsive & Adaptive** | fluid layouts, breakpoints & mobile-first, `clamp()` type/space, touch ergonomics | 4 |
| 03 | 🏗️ **Site Types** | landing pages, one-pagers, scroll experiences, CRM/admin, SaaS, e-commerce, portfolios, brand sites, content/docs, chat & AI-native | 10 |
| 04 | 🧰 **Libraries & Tools** | animation (GSAP/Motion/Lenis), WebGL/3D, CSS & Tailwind, component/headless libs, frameworks, native UI primitives, tooling | 7 |
| 05 | 🔩 **Process & Systems** | design process & stages, design systems & tokens, wireframe → handoff → QA | 3 |
| 06 | 📈 **Marketing & Conversion** | CRO, copywriting/UX writing, persuasion psychology, SEO, performance / Core Web Vitals, analytics, lifecycle & email | 7 |
| 07 | ✨ **Aesthetics & Trends** | style taxonomy, 2025–2026 trends, micro-interactions & motion, avoiding generic "AI" looks | 4 |
| 08 | ⚠️ **Pitfalls & Antipatterns** | common mistakes, dark patterns, real user complaints | 3 |
| 09 | 📋 **Playbooks & Checklists** | step-by-step builds (landing, scroll one-pager, CRM) + master checklist | 4 |

## 📚 Table of Contents

Every guide, grouped by section. The **Use it when** column tells you (or your agent) which guide to open for a given task.

### 🎨 00 · Foundations  <sub>(8 guides)</sub>

| Guide | Use it when |
|---|---|
| [Color Theory & Color Systems](00-foundations/color-theory-and-systems.md) | picking any color, building a palette/theme, adding dark mode, or auditing contrast |
| [Data Visualization & Charts](00-foundations/data-visualization-and-charts.md) | designing any chart, dashboard metric, data story, or scrollytelling visual |
| [Imagery, Icons & Visual Assets](00-foundations/imagery-icons-and-assets.md) | any page that ships a raster image, illustration, SVG icon, or hero |
| [Layout, Grids & Spacing Systems](00-foundations/layout-grids-and-spacing.md) | Laying out any page or component, choosing spacing/sizing values, or fixing a layout that "feels off." |
| [Multilingual & Non-Latin Typography](00-foundations/multilingual-and-non-latin-typography.md) | any site that renders, or might one day render (CMS, UGC, i18n), text in a non-Latin script. |
| [Web Typography Systems](00-foundations/typography-systems.md) | designing or implementing any text-bearing page or design system. |
| [Visual Design Principles & Gestalt](00-foundations/visual-design-principles.md) | composing any layout — deciding what groups with what, what dominates, and where the eye lands. |
| [Visual Hierarchy & Scanning Patterns](00-foundations/visual-hierarchy.md) | designing any screen, page, or component where a user must orient, decide, or act. |

### 🧭 01 · UX Fundamentals  <sub>(10 guides)</sub>

| Guide | Use it when |
|---|---|
| [Accessibility (WCAG 2.2) for Designers & Devs](01-ux-fundamentals/accessibility-wcag.md) | every project, from design phase onward (retrofitting costs 10×) |
| [Authentication & Account Flows](01-ux-fundamentals/authentication-and-account-flows.md) | building any login, sign-up, password reset, MFA, SSO, or account-settings flow |
| [Cognitive Load & Progressive Disclosure](01-ux-fundamentals/cognitive-load-progressive-disclosure.md) | any screen with >1 decision, >1 choice, or >5 fields — onboarding, forms, settings, dashboards, checkout, pricing, empty states |
| [Forms & Input UX](01-ux-fundamentals/forms-and-input-ux.md) | building any form — signup, login, checkout, contact, settings, multi-step flow |
| [Information Architecture & Navigation](01-ux-fundamentals/information-architecture-navigation.md) | designing any site's structure, menus, breadcrumbs, search, or mobile nav |
| [Interaction Design & UI States](01-ux-fundamentals/interaction-design-and-states.md) | designing or building any interactive component, screen, or flow. |
| [Internationalization (i18n) & Localization (l10n)](01-ux-fundamentals/internationalization-and-localization.md) | building anything that may ship in more than one language, region, or currency — even if it launches monolingual. |
| [The Laws of UX & Design Psychology](01-ux-fundamentals/laws-of-ux.md) | any screen, flow, navigation, CTA, form, or pricing decision — i.e. always |
| [Nielsen's Usability Heuristics & Practical Usability](01-ux-fundamentals/nielsen-heuristics.md) | designing or auditing any interactive UI; self-reviewing a build before handoff |
| [User Research & Usability Testing](01-ux-fundamentals/user-research-and-testing.md) | before building (discovery), during design (validation), and post-launch (optimization); any time a decision rests on an assumption about users |

### 📐 02 · Responsive & Adaptive  <sub>(4 guides)</sub>

| Guide | Use it when |
|---|---|
| [Breakpoints, Devices & Mobile-First](02-responsive-adaptive/breakpoints-devices-mobile-first.md) | designing any responsive layout, choosing breakpoints, or deciding mobile-first vs desktop-first. |
| [Fluid Typography & Spacing with clamp()](02-responsive-adaptive/fluid-type-and-spacing.md) | defining any type scale or spacing system; replacing per-breakpoint `font-size`/padding rules; building a token-driven design system. |
| [Responsive & Fluid Design](02-responsive-adaptive/responsive-and-fluid-design.md) | building any production web layout, page, or reusable component. |
| [Touch Ergonomics & Cross-Device Design](02-responsive-adaptive/touch-ergonomics-cross-device.md) | designing or building any page or component that real people touch — i.e., almost everything in 2026+. |

### 🏗️ 03 · Site Types  <sub>(10 guides)</sub>

| Guide | Use it when |
|---|---|
| [Content Sites: Blogs, Magazines & Docs](03-site-types/content-blogs-and-docs.md) | building a blog, news/magazine, knowledge base, or developer docs — any site where the primary job is reading text. |
| [Chat, Conversational & AI-Native Interfaces](03-site-types/conversational-and-ai-native-interfaces.md) | building a chat/assistant surface, an agentic tool, or any UI where an LLM streams responses into a transcript |
| [CRM & Admin Dashboards (Data-Dense UI)](03-site-types/crm-admin-dashboards.md) | building internal tools, admin panels, CRMs, or analytical dashboards with high information density |
| [E-commerce UX](03-site-types/ecommerce-ux.md) | building or auditing any transactional store (catalog → PDP → cart → checkout) |
| [Landing Pages](03-site-types/landing-pages.md) | building any paid-traffic / campaign / product-launch destination page with a single goal. |
| [Marketing & Brand Sites](03-site-types/marketing-brand-sites.md) | building a company/product brand site (homepage + IA, not a single campaign page). |
| [Portfolios & Personal Sites](03-site-types/portfolios-personal-sites.md) | building a designer/developer/creative personal site, UX case-study portfolio, or freelance studio site |
| [SaaS Web Apps](03-site-types/saas-web-apps.md) | building or redesigning the logged-in surface of a subscription product (dashboard app, workspace, tool) |
| [Scroll-Driven Experiences (Magic Scroll, Pinning, Parallax)](03-site-types/scroll-driven-experiences.md) | campaign microsites, product showcases, agency/portfolio hero experiences — NOT task-driven flows |
| [Single-Page Sites & One-Pagers](03-site-types/single-page-one-pagers.md) | content scope is one focused goal (portfolio, event, single product, MVP, brand statement) and linear narrative beats a navigation hierarchy. |

### 🧰 04 · Libraries & Tools  <sub>(7 guides)</sub>

| Guide | Use it when |
|---|---|
| [Web Animation Libraries & Stack](04-libraries-and-tools/animation-libraries-stack.md) | adding any motion to a site — micro-interactions, scroll effects, page transitions, layout animations, or motion graphics. |
| [Component Libraries & Headless UI](04-libraries-and-tools/component-libraries-headless.md) | Picking a component foundation, building a design system, customizing shadcn/ui, fixing a11y or theming problems, or evaluating bundle/accessibility tradeoffs. |
| [CSS, Styling & Tailwind](04-libraries-and-tools/css-styling-and-tailwind.md) | Choosing or wiring up any styling approach, defining design tokens, fixing class-soup/specificity/bundle problems. |
| [Design & Dev Tooling](04-libraries-and-tools/design-and-dev-tooling.md) | Setting up a project's build/QA toolchain, wiring design-token sync, optimizing fonts/images, doing design QA on a build, or reading a Figma handoff. |
| [Frameworks for Design-Forward Sites](04-libraries-and-tools/frameworks-for-design.md) | starting a new design-forward site, or auditing why an existing one is slow/heavy/expensive. |
| [Native Interactive UI Primitives: Popover, Anchor Positioning, Dialog & inert](04-libraries-and-tools/native-interactive-ui-primitives.md) | Building any overlay (menu, dropdown, tooltip, modal, toast, picker, command palette), deciding whether to pull in a headless library, or fixing clipping/stacking/focus-trap bugs. |
| [WebGL, 3D & Shaders for the Web](04-libraries-and-tools/webgl-3d-shaders.md) | a design calls for 3D scenes, shader effects, particle/fluid/distortion visuals, or 3D product viewers |

### 🔩 05 · Process & Systems  <sub>(3 guides)</sub>

| Guide | Use it when |
|---|---|
| [The Website Design Process & Stages](05-process-and-systems/design-process-and-stages.md) | starting any non-trivial website or redesign and deciding what to produce, in what order, with what verification. |
| [Design Systems & Design Tokens](05-process-and-systems/design-systems-and-tokens.md) | building any product that needs consistency across ≥2 surfaces/teams, or any site requiring dark mode/theming |
| [Wireframing, Prototyping, Handoff & QA](05-process-and-systems/wireframing-prototyping-handoff-qa.md) | taking a design from sketch to build, writing specs, or QA'ing an implementation against a design |

### 📈 06 · Marketing & Conversion  <sub>(7 guides)</sub>

| Guide | Use it when |
|---|---|
| [Analytics Instrumentation & Measuring Design Impact](06-marketing-and-conversion/analytics-instrumentation-and-measuring-impact.md) | building any site/app where a stakeholder will later ask "did the redesign work?" |
| [Conversion Rate Optimization (CRO)](06-marketing-and-conversion/conversion-rate-optimization.md) | designing or auditing any page with a measurable goal — landing pages, checkout, signup, pricing, demo requests. |
| [Copywriting & UX Writing](06-marketing-and-conversion/copywriting-and-ux-writing.md) | writing any user-facing string — headlines, buttons, form labels, errors, empty states, tooltips, hero VP, onboarding, success/confirmation, loading states. |
| [Lifecycle, Email & Notification Design](06-marketing-and-conversion/lifecycle-email-and-notification-design.md) | designing any email, push, in-app, or SMS that a customer receives outside the website itself |
| [Ethical Persuasion Psychology](06-marketing-and-conversion/persuasion-psychology.md) | designing any page meant to drive a decision — landing pages, pricing, checkout, signup, onboarding, lead magnets. |
| [SEO & Discoverability for Designers](06-marketing-and-conversion/seo-and-discoverability.md) | building any public-facing page meant to be found via search or shared on social |
| [Web Performance & Core Web Vitals](06-marketing-and-conversion/web-performance-core-web-vitals.md) | building or auditing any production page where speed, ranking, or conversion matters (almost always). |

### ✨ 07 · Aesthetics & Trends  <sub>(4 guides)</sub>

| Guide | Use it when |
|---|---|
| [Avoiding Generic 'AI' Aesthetics](07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md) | building any public-facing or brand site, or when a generated UI looks "fine but forgettable." |
| [A Taxonomy of Web Design Styles](07-aesthetics-and-trends/design-styles-taxonomy.md) | picking or justifying a visual direction before wireframing/build |
| [Micro-interactions & Motion Design](07-aesthetics-and-trends/micro-interactions-and-motion.md) | building any interactive UI — buttons, modals, route changes, scroll reveals, loaders. |
| [Web Design Trends 2025-2026](07-aesthetics-and-trends/trends-2025-2026.md) | choosing an aesthetic direction or deciding whether a requested trend is worth implementing |

### ⚠️ 08 · Pitfalls & Antipatterns  <sub>(3 guides)</sub>

| Guide | Use it when |
|---|---|
| [Common Design Mistakes & Antipatterns](08-pitfalls-and-antipatterns/common-mistakes-antipatterns.md) | designing or reviewing any web UI before ship |
| [Dark Patterns to Avoid](08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md) | designing pricing, checkout, signup, consent, cancellation, or any conversion-critical flow |
| [What Real Users Complain About (Synthesis)](08-pitfalls-and-antipatterns/real-user-complaints.md) | designing, reviewing, or auditing any public-facing site before ship |

### 📋 09 · Playbooks & Checklists  <sub>(4 guides)</sub>

| Guide | Use it when |
|---|---|
| [Master Checklists & Cheat-Sheets](09-playbooks-and-checklists/master-checklists-cheatsheets.md) | scoping a build, reviewing your own output, or doing final pre-launch QA |
| [Playbook: Design a CRM / Admin Dashboard](09-playbooks-and-checklists/playbook-crm-dashboard.md) | building an internal/B2B tool where users return daily to scan, filter, act on, and drill into structured business data. |
| [Playbook: Design & Build a Landing Page](09-playbooks-and-checklists/playbook-landing-page.md) | building a single-purpose conversion page for one offer + one traffic source. |
| [Playbook: Build a Scroll-Driven One-Pager](09-playbooks-and-checklists/playbook-scroll-experience.md) | building a marketing one-pager, portfolio, or product story where scroll *is* the interaction. |

---

## ⭐ Highlights

- **[Scroll-Driven Experiences](03-site-types/scroll-driven-experiences.md)** + **[its build playbook](09-playbooks-and-checklists/playbook-scroll-experience.md)** — the "magic scroll" one-pager done right: GSAP ScrollTrigger + Lenis, native CSS scroll-timelines, pinning / parallax, compositor-only performance, and `prefers-reduced-motion`.
- **[Landing Pages](03-site-types/landing-pages.md)** + **[playbook](09-playbooks-and-checklists/playbook-landing-page.md)** — the anatomy of a high-converting page, from message-match to CTA.
- **[Avoiding Generic 'AI' Aesthetics](07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md)** — how to escape the default "AI-generated" look and ship work that reads as *designed*.
- **[Accessibility (WCAG 2.2)](01-ux-fundamentals/accessibility-wcag.md)** — POUR, semantic HTML, focus, ARIA, target sizes, reduced motion.
- **[Chat, Conversational & AI-Native Interfaces](03-site-types/conversational-and-ai-native-interfaces.md)** — streaming transcripts, composer UX, and the accessibility of live-updating chat.

## 🚀 How to use this repo

**Browsing (humans).** Open the [Table of Contents](#-table-of-contents), find the section, read the guide. Each is self-contained and ends with a checklist you can apply immediately.

**With an AI coding agent (recommended).** Two patterns:

1. **Point your agent at the index.** Add one line to your project's `CLAUDE.md`, `.cursorrules`, or `AGENTS.md`:
   > Before building any UI, read `AGENTS.md` in the design codex, pick the relevant guides for the task, and follow them.
2. **Clone it next to your project** so the agent can read guides on demand:
   ```bash
   git clone <this-repo> design-codex
   ```

Don't dump all 60 guides into a prompt — they total ~23k lines. The codex is a **knowledge base you query selectively**: read [`AGENTS.md`](AGENTS.md) (machine index + curated *reading paths* per task), then open the 2–5 guides it points to.

## 🗂️ Repository structure

```
ai-web-design-codex/
├── AGENTS.md                      ← machine index + reading paths (for AI agents)
├── README.md                      ← you are here
├── 00-foundations/                (8)  visual principles, hierarchy, type, color, grids, imagery, data-viz
├── 01-ux-fundamentals/            (10) laws, heuristics, research, IA, forms, a11y, i18n, auth
├── 02-responsive-adaptive/        (4)  fluid, breakpoints, clamp(), touch
├── 03-site-types/                 (10) landing, one-pager, scroll, CRM, SaaS, e-com, portfolio, brand, content, chat
├── 04-libraries-and-tools/        (7)  animation, WebGL, CSS/Tailwind, components, frameworks, native primitives, tooling
├── 05-process-and-systems/        (3)  process, design systems & tokens, wireframe → handoff → QA
├── 06-marketing-and-conversion/   (7)  CRO, copy, persuasion, SEO, performance, analytics, lifecycle
├── 07-aesthetics-and-trends/      (4)  styles taxonomy, trends, motion, anti-AI-slop
├── 08-pitfalls-and-antipatterns/  (3)  mistakes, dark patterns, real complaints
└── 09-playbooks-and-checklists/   (4)  step-by-step builds + master checklist
```

## 🧪 How this was built

Each guide was researched by agents sweeping authoritative sources — Nielsen Norman Group, Smashing Magazine, web.dev, MDN, Baymard Institute, OWASP, Material Design 3, Apple HIG, the GSAP / Motion docs — plus practitioner discussion on Reddit, then **adversarially fact-checked**: a second reviewer hunted for vague advice, wrong facts, and fabricated statistics and corrected them in place. Internal cross-links are verified — **482 links, 0 broken**.

> Guides reflect a **2025–2026 baseline**. The web platform moves fast — verify version-specific API and browser-support claims against current docs before relying on them.

## 📄 License

Licensed under [**CC BY 4.0**](LICENSE) — share and adapt for any purpose, including commercially, with attribution: *"AI Web Design Codex by Eneryleen"* + a link back to this repository.

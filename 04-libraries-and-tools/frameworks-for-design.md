# Frameworks for Design-Forward Sites

> How to choose and configure a framework/meta-framework so a design-forward site loads fast, ranks well, and stays maintainable. Covers Astro, Next.js (App Router), SvelteKit, Nuxt, SolidStart; rendering strategies (SSG/ISR/SSR/PPR/Islands/Server Islands); hydration cost; native View Transitions; and built-in image optimization. Senior outcome: pick the lightest tool that meets the real requirement, render each route the right way, and ship the minimum JS the design needs.

**Apply when:** starting a new design-forward site, or auditing why an existing one is slow/heavy/expensive. · **Related:** [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [SEO & Discoverability](../06-marketing-and-conversion/seo-and-discoverability.md) · [Web Animation Libraries & Stack](animation-libraries-stack.md) · [CSS, Styling & Tailwind](css-styling-and-tailwind.md)

## TL;DR — rules at a glance
- **Ask first: website or web app?** A website (blog/docs/marketing/portfolio) defaults to **static output**. A web app (dashboard/SaaS behind auth) can use SSR/SPA. Treating a website as a web app is the #1 mistake.
- **>80% of pages have no user-specific content → start with Astro or static Next.js SSG.** Adding interactivity later is easy; removing a React runtime is hard.
- **Static-first wins for content/marketing.** Astro ships **~8KB** (often zero) JS for a static page; Next.js always ships a React runtime (**~85KB+**). That's a >90% reduction on comparable pages.
- **Hydration cost is real.** A typical React SPA ships **200KB+** of JS that must execute before buttons are clickable. An equivalent Astro page may ship 8KB or nothing.
- **Pick the rendering strategy per route, not per project.** SSG for pages that never change, ISR for periodic content, SSR for truly dynamic/personalized. Mix strategies inside one project — that's best practice.
- **Islands hydrate independently.** Static content is pure HTML; interactive "islands" hydrate in parallel, so a slow carousel never blocks a fast nav.
- **Control *when* JS loads with directives:** `client:load` (now), `client:idle` (after interactive), `client:visible` (in viewport), `client:media` (at a media query).
- **Never use raw `<img>` in Astro/Next.** Use `<Image/>` / `next/image`: auto WebP/AVIF, reserved dimensions (no CLS), lazy-load. Raw tags waste your LCP and CLS budget.
- **View Transitions are native** (>95% browser support as of 2025; Level 1 shipped in Chrome 111, Safari 18, Firefox 119). Astro gives the richest zero-JS abstraction; SvelteKit exposes the raw API.
- **Default to Server Components in Next.js.** Every `'use client'` is an explicit, justified cost against the hydration budget — not a habit.
- **Don't over-rely on SSR for SEO.** Google crawls JS SPAs fine; the real SEO win of SSG/SSR is Core Web Vitals (LCP/CLS), not crawlability.
- **Deployment portability matters.** Next.js runs best on Vercel; self-hosting introduces undocumented edge cases. Astro/SvelteKit/Nuxt are deployment-agnostic.
- **Lighthouse mobile < 75? Check JS payload first** (Next.js marketing) or **images** (Astro) — those are the dominant bottlenecks per framework.

## The first decision: website vs. web app

Everything downstream depends on this. Get it right and the rest is configuration.

| Signal | Website (static-first) | Web app (SSR/SPA) |
|---|---|---|
| Primary content | HTML + text + media | Stateful, user-specific data |
| Auth wall | Mostly public | Mostly behind login |
| Interactivity | Sprinkled (nav, forms, a carousel) | Pervasive (live state across views) |
| Examples | Marketing, blog, docs, portfolio, landing | Dashboard, CRM, SaaS console |
| Default tool | **Astro / Next.js SSG** | **Next.js SSR, SvelteKit, Nuxt, SolidStart** |

Heuristics:
- **>80% of pages have no per-user content → static-first.** This is the single most useful rule.
- An **admin dashboard behind an auth wall gains nothing from SSR** (no SEO benefit, no shareable URL value). A public marketing page gains a lot.
- A **content-heavy site treated as a web app** pays React's runtime tax on every page that is really just HTML.

## Framework selection matrix

| Framework | Model | Default JS | Best for | Avoid when |
|---|---|---|---|---|
| **Astro** | MPA + Islands | Zero by default | Content/marketing, blogs, docs, portfolios | Complex stateful app where state must persist across navigations |
| **Next.js 15 (App Router)** | SSR/SSG/ISR/PPR + RSC | ~85KB+ React runtime | React teams, app + marketing hybrid, rich ecosystem | Pure content site where you want minimal JS; self-hosting without Vercel features |
| **SvelteKit** | SSR/SSG/CSR, compiled | Small (no VDOM runtime) | Smaller bundles at equal functionality, deployment-agnostic apps | Teams committed to the React ecosystem |
| **Nuxt 4** | Per-route via `routeRules` | Vue runtime | Vue teams blending marketing shell + dashboard in one deploy | Non-Vue teams; ultra-minimal-JS content sites |
| **SolidStart** | SSR/SSG, fine-grained reactivity | Small (no VDOM) | Raw SSR throughput, low memory, app workloads | Need broad ecosystem/plugins; alpha-stage risk tolerance low |
| **TanStack Start** | SSR/SSG, file-based routing | React runtime | React teams wanting lighter Next.js alternative; type-safe routing; no platform lock-in | Need a mature ecosystem today (early-release as of 2025) |

Notes:
- **Astro is multi-framework**: it can host React/Vue/Svelte/Solid/Preact islands in one project. Useful, but **standardize on one island framework** — mixing 4 is technically supported and a maintenance nightmare.
- **SvelteKit compiles the framework away** — vanilla JS to the browser, no virtual DOM at runtime → smaller bundles than React-based frameworks at equivalent functionality.
- **SolidStart** uses compile-time reactivity (no VDOM) and benefits from Vite/esbuild bundling; choose it when SSR throughput beats ecosystem breadth. Still pre-1.0 as of mid-2025 — evaluate API stability before committing.
- **Nuxt 4** (stable 2025) introduces unified `app/` directory structure and breaking changes from Nuxt 3; plan migration from Nuxt 3 explicitly.
- **TanStack Start** (RC/stable 2025) pairs with TanStack Router for fully type-safe routing; lighter than Next.js for pure app workloads without the Vercel dependency.
- **Gatsby is effectively deprecated for new projects.** Astro covers the same use cases with better performance and active maintenance.

## Rendering strategies: SSG / ISR / SSR / PPR / Islands / Server Islands

These are **not interchangeable**. Match the strategy to how the data changes.

| Strategy | Rendered | Use for | Revalidate |
|---|---|---|---|
| **SSG** (static) | At build | Marketing, docs, blog, About | Never (rebuild on content change) |
| **ISR** (incremental static regen) | At build, refreshed on interval | Product catalog, news, listings | Match freshness (see below) |
| **SSR** (per request) | On each request | Personalized/logged-in, real-time | n/a |
| **PPR** (partial pre-rendering) | Static shell at build + dynamic holes at request | Hybrid: CDN-cached shell + per-request personalization (Next.js 15) | n/a |
| **Islands** | Static HTML + selective client hydration | Mostly-static pages with a few widgets | n/a |
| **Server Islands** | Static shell + deferred server-rendered fragment | Personalized bits on a CDN-cached shell (Astro) | n/a |

### PPR — Partial Pre-Rendering (Next.js 15+)
PPR is the synthesis of SSG and SSR in a single route: the static shell is generated at build time and served from the CDN edge; dynamic `<Suspense>` boundaries are streamed from the server per request. No separate ISR interval needed for the shell.

```tsx
// next.config.ts — enable PPR
import type { NextConfig } from 'next'
const config: NextConfig = { experimental: { ppr: true } }
export default config

// page.tsx — static shell + dynamic hole
import { Suspense } from 'react'
import { UserCart } from './UserCart'        // dynamic (per-request)
import { HeroSection } from './HeroSection'  // static (pre-rendered)

export default function Page() {
  return (
    <>
      <HeroSection />                        {/* served from CDN cache */}
      <Suspense fallback={<CartSkeleton />}>
        <UserCart />                         {/* streamed per request */}
      </Suspense>
    </>
  )
}
```
Use PPR when a route has both a cacheable shell and small dynamic sections (cart count, user greeting, recommendations). It avoids the full SSR per-request cost while delivering fresh personalized content.

### ISR revalidate intervals — match content freshness
```js
// Next.js App Router — per-route revalidate (seconds)
export const revalidate = 3600   // blog post: 1h
// product price page:   export const revalidate = 60
// static About page:    use SSG (no revalidate) — never
```
- Overly short timers waste compute with no user benefit.
- **ISR semantics:** the first visitor after a cache expiry gets the **stale** copy while it regenerates (stale-while-revalidate). Design UX to tolerate one stale render per interval.

### Next.js: Server Components default, `'use client'` is opt-in
```tsx
// Server Component (default) — no JS shipped for this component
export default async function ProductList() {
  const products = await getProducts()      // runs on server
  return <ul>{products.map(p => <li key={p.id}>{p.name}</li>)}</ul>
}
```
```tsx
'use client'                                 // explicit, justified boundary
import { useState } from 'react'
export function AddToCart() {                // ships to client, adds to hydration budget
  const [n, setN] = useState(0)
  return <button onClick={() => setN(n + 1)}>Add ({n})</button>
}
```
Rule: **every `'use client'` is a cost.** Putting it on every component negates Server Components and ships the full React client runtime.

**React 19 + Server Actions (Next.js 15):** Server Actions are stable in React 19 / Next.js 15. They let forms and mutations call server-side logic without a separate API route, reducing client JS further.

```tsx
// Server Action — runs on the server, called from a Client Component
'use server'
export async function subscribe(formData: FormData) {
  await db.subscribers.create({ email: formData.get('email') })
}

// Client Component
'use client'
import { subscribe } from './actions'
export function NewsletterForm() {
  return <form action={subscribe}><input name="email" /><button>Subscribe</button></form>
}
```

### Astro: Islands + hydration directives
```astro
---
import Carousel from '../components/Carousel.jsx'
import ChatWidget from '../components/ChatWidget.jsx'
import Nav from '../components/Nav.astro'   // .astro = pure HTML, zero JS
---
<Nav />
<Carousel client:visible />                 {/* hydrate when scrolled into view */}
<ChatWidget client:idle />                  {/* hydrate after page is interactive */}
```
Directive cheat sheet:
- `client:load` — hydrate immediately (use sparingly; above-the-fold interactive).
- `client:idle` — after the page is interactive (chat widgets, low-priority).
- `client:visible` — when the element enters the viewport (carousels, below-the-fold).
- `client:media="(min-width: 768px)"` — only at a CSS media query (desktop-only widgets).
- `client:only="react"` — skip SSR, render only on the client (last resort).

### Astro Server Islands — personalize without losing the CDN cache
```astro
---
import UserGreeting from '../components/UserGreeting.astro'
---
<!-- shell is static + CDN-cached; this fragment renders on the server per request -->
<UserGreeting server:defer>
  <span slot="fallback">Welcome</span>     {/* shown until the island resolves */}
</UserGreeting>
```
Use `server:defer` for user greeting, cart count, recommendations — dynamic content on an otherwise static, cacheable page.

### Nuxt: per-route rendering in one deploy
```ts
// nuxt.config.ts — mix SSG/SSR/CSR without separate deployments
export default defineNuxtConfig({
  routeRules: {
    '/':            { prerender: true },        // SSG marketing home
    '/blog/**':     { isr: 3600 },              // ISR blog
    '/dashboard/**':{ ssr: false },             // CSR (SPA) app section
    '/api/**':      { cors: true },
  },
})
```

## Hydration cost — the budget you actually spend

- A React SPA commonly ships **200KB+** of JS that must parse and execute before the UI responds; an equivalent Astro page ships **~8KB or zero**.
- Hydration blocks interactivity (TTI/INP). Islands fix this by hydrating each widget independently and in parallel — a slow carousel never blocks the nav header.
- **Budget per `'use client'` / per island.** If a component doesn't need state or event handlers, it shouldn't ship JS.

## View Transitions — native, not a library

Native browser API. Level 1 shipped in Chrome 111 (Mar 2023), Safari 18 (Sep 2024), Firefox 119 (Oct 2023). Browser support is **>95% as of 2025**. No library required.

**Astro (richest abstraction, zero-JS fallback):**
```astro
---
import { ClientRouter } from 'astro:transitions'
---
<head><ClientRouter /></head>
<img src={hero} transition:name="hero" transition:animate="fade" />
```
- `transition:name` pairs elements across pages for morphing; `transition:animate` sets the animation. Unsupported browsers fall back to a plain page jump automatically.

**SvelteKit (raw platform API, you own the fallback):**
```ts
import { onNavigate } from '$app/navigation'
onNavigate((navigation) => {
  if (!document.startViewTransition) return        // you implement progressive enhancement
  return new Promise((resolve) => {
    document.startViewTransition(async () => {
      resolve()
      await navigation.complete
    })
  })
})
```
- **Next.js:** View Transitions require React canary/experimental flags as of Next.js 15; treat as not-yet-stable for production.
- **Nuxt:** experimental `viewTransition` option since Nuxt 3.4; production-ready status varies — validate before shipping.

Respect motion preferences — pair with `@media (prefers-reduced-motion: reduce)`. See [Micro-interactions & Motion Design](../07-aesthetics-and-trends/micro-interactions-and-motion.md).

## Image optimization — non-negotiable for Core Web Vitals

Both `next/image` and Astro `<Image/>` auto-generate WebP/AVIF, reserve layout dimensions (prevents CLS), and lazy-load. **Using raw `<img>` wastes LCP and CLS budget.**

```astro
---
import { Image } from 'astro:assets'
import hero from '../assets/hero.jpg'        // local: dimensions inferred
---
<Image src={hero} alt="Product hero" width={1200} height={630} loading="eager" />
```
```tsx
import Image from 'next/image'
<Image src="/hero.jpg" alt="Product hero" width={1200} height={630} priority />
```
- **Remote images:** both frameworks infer dimensions for *local* images but **require explicit `width`/`height` for remote URLs** — omit them and you reintroduce CLS.
- Use `priority` (Next) / `loading="eager"` (Astro) for the LCP image only; lazy-load the rest (default).
- Astro `<Picture/>` for art-directed/format-explicit cases; works for remote images and Markdown/MDX frontmatter images too.

See [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) and [Imagery, Icons & Visual Assets](../00-foundations/imagery-icons-and-assets.md).

## Content sourcing (CMS) for marketing/blog

- **Astro Content Layer (v5)** — unified, type-safe API for content from any source (local Markdown, CMS APIs, databases). Replaces older content collections. Automatic schema validation, zero JS by default → lowest-overhead path for a CMS-backed marketing site.
- Headless CMS options compatible with both Astro Content Layer and Next.js data fetching: **Sanity, Contentful, Prismic, Strapi**.
- For docs at scale, **Starlight** (Astro-based) builds a 1,000-page site significantly faster than Next.js-based doc tools; exact build times vary by machine and content volume.

```ts
// Astro Content Layer — type-safe collection from Markdown
import { defineCollection, z } from 'astro:content'
import { glob } from 'astro/loaders'
const blog = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: z.object({ title: z.string(), date: z.coerce.date(), draft: z.boolean().default(false) }),
})
export const collections = { blog }
```

## Concrete specs & numbers

Numbers below come from official documentation, public benchmarks, or widely-reported practitioner measurements. Do not treat lab figures as universal — your content, machine, and network shape the results.

- **JS bundle (comparable static marketing page):** Astro ~8KB vs Next.js ~85KB (>90% reduction). Raw React SPA baseline: 200KB+. (Astro docs; Next.js bundle analyzer.)
- **Astro 5.0** (released Dec 3, 2024): up to 5x faster Markdown builds, 2x faster MDX, 25–50% less memory vs Astro 4.x; ships with Vite 6. (Astro 5.0 release notes.)
- **next/image:** WebP/AVIF conversion reduces file size 60–80% vs unoptimized JPEG/PNG at equivalent quality; dimensions required to prevent CLS. (Next.js docs; web.dev image guides.)
- **LCP budget (Google):** < 2.5s = good; 2.5–4s = needs improvement; > 4s = poor. (web.dev/lcp.)
- **View Transitions API:** Chrome 111+, Firefox 119+, Safari 18+; >95% global support as of 2025. (MDN, caniuse.com.)
- **Nuxt 4** (stable 2025): Nuxt downloads consistently rank it among the top Vue meta-frameworks by npm weekly installs; exact weekly figures fluctuate — check npmtrends.com for current data.
- **SolidStart 2.0-alpha** (early 2026): pre-release; evaluate API stability before committing to production.

Figures to treat with skepticism until you can reproduce them: framework-to-framework SSR throughput comparisons (depend heavily on hardware, payload, and cold-start conditions) and "pass rate" statistics for Core Web Vitals by framework (methodology varies by source).

## Tools & libraries
- **Astro** — zero-JS-by-default static builder, Islands Architecture, multi-framework islands, Server Islands (`server:defer`), Content Layer. Default for content/marketing/docs/portfolio.
- **Next.js 15 (App Router)** — React 19 meta-framework; SSR/SSG/ISR/PPR/streaming; Server Components default; Server Actions stable; `next/image`. Use for React teams and app+marketing hybrids; best on Vercel.
- **SvelteKit** — compiles to vanilla JS (no VDOM); native View Transitions via `onNavigate`; deployment-agnostic; smaller bundles than React equivalents.
- **Nuxt 4** — Vue + Vite + Nitro; per-route `routeRules`; experimental View Transitions (3.4+). Use for Vue teams blending shell + dashboard.
- **SolidStart** — Solid.js; fine-grained compile-time reactivity; high SSR throughput, low memory. Pre-1.0 API stability risk; use when raw throughput beats ecosystem breadth.
- **TanStack Start** — React SSR meta-framework (RC/stable 2025); pairs with TanStack Router for fully type-safe routing; lighter than Next.js without Vercel coupling. Consider for new React app projects.
- **next/image** / **Astro `<Image/>` / `<Picture/>`** — always use over raw `<img>`.
- **GSAP** — framework-agnostic animation; works in Astro/Next/SvelteKit/WordPress alike (it does **not** require a "modern" stack). See [Web Animation Libraries & Stack](animation-libraries-stack.md).
- **Vite** — shared bundler under Astro/SvelteKit/SolidStart/Nuxt (Astro 5 → Vite 6).
- **Hosting:** Cloudflare Pages (strong Astro support; zero cold starts for static; Workers for SSR); Vercel (native for Next.js; image optimization CDN for Astro/Nuxt too; costs scale with serverless invocations).
- **When NOT:** don't reach for Next.js on a pure content site; don't use Gatsby for new projects; don't use a WebGL/3D framework here (see [WebGL, 3D & Shaders](webgl-3d-shaders.md)).

## Do / Don't

**Do**
- Default content sites to static output; add interactivity per-widget via islands.
- Choose rendering per route (SSG/ISR/SSR/PPR) and co-locate it with the route.
- Keep Next.js components Server Components unless they need state/effects/handlers.
- Use Server Actions for mutations instead of separate API routes where it reduces client JS.
- Use `<Image/>` / `next/image`; set explicit dimensions for all remote images.
- Use `client:visible` for carousels, `client:idle` for chat widgets.
- Use Astro `server:defer` to personalize on a CDN-cached static shell.
- Use PPR for Next.js routes with a cacheable shell plus small dynamic sections.
- Provide a View Transitions fallback (automatic in Astro; manual in SvelteKit).
- Standardize on one island framework in an Astro project.

**Don't**
- Don't run Next.js in SSR mode for a marketing site with no dynamic data.
- Don't sprinkle `'use client'` everywhere — it ships the full React runtime.
- Don't self-host Next.js without Vercel features unless you've validated the edge cases.
- Don't ship raw `<img>` in Astro/Next, or remote images without `width`/`height`.
- Don't assume SSR improves SEO over static — it improves CWV, not crawlability.
- Don't use Astro for a complex stateful dashboard expecting cross-navigation state.
- Don't mix 4–5 UI frameworks in one Astro project.
- Don't use Gatsby for new builds; don't rely on `output: 'export'` in Next.js for routes using middleware, ISR, or image optimization.

## Common pitfalls & how to avoid them
- **Next.js SSR for a static marketing site** — you pay serverless compute on every request for pages that could be permanent CDN HTML. Fix: SSG/ISR or PPR, or move to Astro.
- **`'use client'` everywhere** — negates Server Components, ships full React runtime, bloats hydration. Fix: keep boundaries explicit and minimal.
- **Next.js off Vercel (Docker/GCP/self-host)** — undocumented edge cases cause random 500s; practitioners consistently report pain. Fix: use Vercel, or pick a deployment-agnostic framework (Astro/SvelteKit/Nuxt).
- **Astro for a stateful dashboard** — MPA model means state doesn't persist across full-page navigations. Fix: use a SPA framework, or wrap the app section in a single React/Svelte island.
- **Remote images without dimensions** — both frameworks need explicit `width`/`height` for remote URLs or you get CLS. Fix: always set them.
- **Forgetting ISR stale-while-revalidate** — the first visitor after expiry gets a stale render. Fix: set intervals to content freshness and tolerate one stale render per interval.
- **Gatsby in 2025+** — effectively deprecated. Fix: Astro.
- **Mixing 5 UI frameworks in one Astro project** — maintenance nightmare. Fix: standardize on one.
- **Assuming SSR = better SEO** — Google crawls SPAs; the win is CWV. Fix: optimize LCP/CLS, not crawlability theater.
- **Ignoring Vercel cost structure** — serverless invocations accumulate; crawler/bot spikes (Meta crawler, AI crawlers) can generate millions of requests and surprise bills. Fix: static/CDN where possible; cache aggressively; watch invocation metrics.
- **View Transitions without fallback** — Astro auto-falls-back; SvelteKit doesn't. Fix: implement progressive enhancement yourself in SvelteKit.
- **`output: 'export'` with App Router features** — static export in Next.js disables middleware, ISR, and `next/image` on-demand optimization. Fix: don't rely on those in export mode; use Astro for pure static sites.
- **Skipping PPR when available** — Next.js 15 teams default to full SSR for hybrid routes when PPR would deliver the same result with CDN caching for the shell. Fix: reach for PPR before full SSR on routes with mostly static shells.

## What real users complain about
- **"Heavier than necessary"** — common practitioner description of Next.js for content-focused sites; the React runtime cost is unavoidable regardless of how few client components a page uses.
- **"Next.js is unnecessary if you only need routing"** — teams migrating to Vite + React + TanStack Router report faster hot reload, faster builds, and no platform lock-in.
- **Vercel cost anxiety** — SSR Next.js apps get unexpectedly expensive when crawler traffic spikes; documented cases of AI/bot crawlers generating millions of serverless invocations per month.
- **Self-hosting Next.js is painful** — repeated reports of undocumented edge cases and random 500s when running off Vercel (Docker, GCP, bare metal).
- **`'use client'` creep** — teams find Server Component discipline hard to maintain; boundaries leak and bundle size balloons.

## Senior-level checklist
- [ ] Classified the project as **website (static-first)** or **web app (SSR/SPA)** before choosing a tool.
- [ ] If >80% of pages are non-personalized, chose Astro or Next.js SSG.
- [ ] Each route's rendering strategy (SSG/ISR/SSR/PPR) is chosen deliberately and matches data freshness.
- [ ] ISR revalidate intervals match content type (e.g., blog 3600s, prices 60s, About = SSG).
- [ ] PPR evaluated for Next.js 15 routes that have a cacheable shell + small dynamic sections.
- [ ] In Next.js, components are Server Components by default; every `'use client'` is justified.
- [ ] Server Actions used for mutations where they reduce client-side JS vs a separate API route.
- [ ] All interactivity in Astro uses the correct directive (`visible`/`idle`/`media`/`load`).
- [ ] Personalized content on otherwise-static pages uses Server Islands (`server:defer`) or PPR/SSR only where required.
- [ ] No raw `<img>`; `<Image/>` / `next/image` used; remote images have explicit `width`/`height`.
- [ ] LCP image is `priority`/eager; everything else lazy-loads.
- [ ] View Transitions have a working fallback (auto in Astro; manual in SvelteKit) and respect `prefers-reduced-motion`.
- [ ] Deployment target validated for the chosen framework (Next.js → Vercel-aware; others → portable).
- [ ] Lighthouse mobile (Slow 4G) checked: Next.js → audit JS payload first; Astro → audit images first.
- [ ] One island framework standardized per Astro project; Gatsby not used for new work.
- [ ] Vercel/serverless cost reviewed against expected crawler/bot traffic if running Next.js SSR.

## Sources
- Astro docs: https://docs.astro.build/ (Islands, hydration directives, Server Islands, `<Image/>`, Content Layer, View Transitions)
- Next.js docs: https://nextjs.org/docs (App Router, Server Components, ISR/`revalidate`, PPR, Server Actions, `next/image`, static export)
- SvelteKit docs: https://svelte.dev/docs/kit
- Nuxt docs: https://nuxt.com/docs (`routeRules`, Nitro, Nuxt 4 migration)
- SolidStart docs: https://start.solidjs.com/
- TanStack Start docs: https://tanstack.com/start/latest
- View Transitions API (MDN): https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API
- Core Web Vitals / LCP (web.dev): https://web.dev/articles/lcp
- Astro 5.0 release post: https://astro.build/blog/astro-5/

# Portfolios & Personal Sites

> How to design and build portfolio and personal sites that convert a 6–8 second scan into an interview or client inquiry. Covers outcome-first case studies, hero/about, visual identity, project pages, and the performance/accessibility floor a senior portfolio must clear. The senior-level outcome: a tight, fast, honest site that sells business impact, not pixels.

**Apply when:** building a designer/developer/creative personal site, UX case-study portfolio, or freelance studio site · **Related:** [Landing Pages](./landing-pages.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [Copywriting & UX Writing](../06-marketing-and-conversion/copywriting-and-ux-writing.md) · [Common Design Mistakes & Antipatterns](../08-pitfalls-and-antipatterns/common-mistakes-antipatterns.md)

## TL;DR — rules at a glance

- **Pass the 6–8 second test.** Within the first scan a visitor must know *who you are, what you do, your seniority, and your domain*. Test with 3 non-designers before launch; if they can't answer "who is this and what do they do?", rewrite the hero.
- **Sell outcomes, not outputs.** Lead each case study with the result/metric, then prove how design got there.
- **3–5 deep case studies beat 10+ shallow ones.** Junior: 2–3. Mid/senior: 3–5. Above ~6–8 forces the hiring manager to choose for you.
- **Results first, then process.** Hiring managers spend ~5 minutes per portfolio. Surface impact upfront so they keep reading.
- **One hero, one primary CTA.** Name + title + one-line value prop + single primary CTA. Never two equal-weight buttons.
- **State your contribution on every project.** Role, scope, solo/team, methods. Vague "Sole UX Designer" with no specifics reads as a bootcamp template.
- **Case study text-to-media ratio: 60–80% text, 20–40% visuals** for UX/product portfolios. Motion design and branding portfolios may shift toward 50/50. Always include process artifacts (sketches, wireframes, research), not just polished screens.
- **Performance is the proof.** LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1. A slow portfolio contradicts any claim of frontend skill.
- **Contrast floor: 4.5:1** (WCAG AA normal text). Low-contrast text is the #1 portfolio accessibility failure (81% of homepages per WebAIM 2024).
- **Kill skill percentage bars** ("JavaScript 80%") — universally read as meaningless. Show projects instead.
- **Contact reachable from every page.** Sticky nav + Contact CTA. Burying contact is a top-5 mistake.
- **Label spec/redesign/passion projects honestly.** Mislabeling them as client work = instant rejection.
- **Audit every link.** Dead links, 404 case studies, and broken live demos are worse than no demo at all.
- **Mobile is not optional.** A significant share of portfolio traffic is mobile; a broken mobile layout signals inability to think cross-platform.

## The 6–8 second / 5-minute model

Design around two attention budgets:

- **6–8 seconds:** initial scan. Visitor decides *continue or exit*. The hero + first project thumbnail do all the work here.
- **~15 seconds:** total initial screening time on average.
- **~5 minutes:** time allocated to a *full* review once they decide to dig in. This is where case studies earn the interview.

Implication: front-load signal. The strongest case study, the clearest title, and the most concrete outcome must be above the fold or one scroll away — "one scroll away" means within ~1 viewport height below the hero (≈600–900px at common viewport sizes). Anything buried below 5 minutes of reading is effectively invisible.

> Morgane Peng (design executive and hiring manager, author of *The Designer's Guide to Product Vision*): "As a hiring manager, I'd rather see the results first before diving into the case study and committing 7 to 10 minutes of my life reading it." — morganepeng.com

## Hero section

The hero answers "who, what, why you" instantly. Structure:

```
[ Name ]                  ← real name, large
[ Professional title ]    ← "Senior Product Designer", "Frontend Engineer"
[ One-line value prop ]   ← what you do + for whom + the edge
[ Primary CTA ]           ← "View work" or "Contact" — exactly one primary
[ (optional) secondary CTA ]  ← lower visual weight, e.g. "Download CV"
[ 2–3 featured project thumbnails below fold ]
```

Value-prop examples (concrete, not generic):
- ✅ "I design checkout and onboarding flows that cut drop-off for fintech products."
- ✅ "Frontend engineer specializing in performant, accessible design systems."
- ❌ "Passionate creative who loves making beautiful, intuitive experiences." (vague, "intuitive" is a red flag, no domain)

Rules:
- **One primary CTA.** Two equal-weight buttons split attention and dilute the next action.
- **No slow reveal animations.** Typewriter/sequential fade-ins that delay content are a conversion killer. Hiring managers reviewing 30+ portfolios per session will not wait for an animation to finish before reading your name.
- **No generic stock imagery.** Use custom photography, high-quality real mockups, or brand-coherent abstract graphics. Stock weakens trust.
- Lead with your **strongest case study thumbnail** immediately below the hero copy.

## About section

The about is supporting evidence, not the headline. Keep it scannable and specific.

- Lead with positioning: what kind of problems you solve and for whom (domain + seniority signal).
- Include concrete proof points: years, industries, notable clients/employers (if allowed), specialties.
- Skip life-story filler. One short paragraph of personality is fine; three is indulgent.
- **No skill percentage bars.** "HTML 95% / CSS 90%" communicates nothing. Replace with named tools/tech *demonstrated in projects*.
- Avoid "Google certified" as a headline credential — it signals awareness, not mastery, and is mocked in hiring circles. List it small, if at all.
- A clear, current photo builds trust for client/freelance sites; optional for eng roles.

## Case studies — the core asset

A case study is a tool to win an interview, **not** a complete project archive. Under-load depth per study; don't over-load breadth across many. A hiring manager who likes what they see will request more.

### Standard section structure — UX/product

1. **Hero / hook** — project title + one-line outcome, optional cover image. (2 title sentences.)
2. **Project overview** — context, your role, timeline, team. (3–4 sentences + 4–5 metadata bullets.)
3. **Discovery & research** — research methods, personas, competitive analysis, key insights. (≥3–4 sentences per method; include artifacts.)
4. **Design process** — IA, flows, wireframes, iterations, how feedback changed decisions. (≥3–5 sentences per step; show before/after.)
5. **Final design** — polished screens with rationale tied to constraints and research findings. (2–3 short paragraphs.)
6. **Impact / outcomes** — metrics and business results. (≥1 sentence per achievement; quantify what you can, scope/scale when you can't.)
7. **Learnings** — what you'd do differently and why; shows self-awareness and growth mindset. (≥3–4 sentences.)

### Standard section structure — engineering/development

Engineer portfolios must show *problem → technical decision → measurable outcome*, not just a list of technologies used.

1. **Hero / hook** — project title + one-line outcome + live link or demo.
2. **Problem & context** — what was broken or missing, who it affected, scale (users, requests/sec, data volume).
3. **Technical approach** — architecture decisions, key tradeoffs made (and why), what you chose *not* to do.
4. **Implementation highlights** — the interesting 1–3 problems you solved; include relevant code snippets only if they illustrate the decision, not to show you can type.
5. **Outcome** — performance change, error rate change, shipped scale, adoption. Numbers or bust.
6. **Learnings** — what broke, what surprised you, what you'd change.

> Rule: if a reader can't tell *why* you made the technical choices you did, the case study is a project description, not a portfolio piece.

### Impact-driven variant (recommended for mid/senior)

Reorder to put **outcomes immediately after project overview**, then prove how the design achieved them. Apply the **Minto Pyramid / Pyramid Principle**: conclusion first, then supporting evidence.

```
Hook → Project overview → IMPACT METRIC (lead) → How design achieved it
     → Strategy & process → Final design → Learnings
```

This respects the 5-minute budget: the reader gets the payoff before deciding whether to invest in the full narrative.

### State your contribution — always

On every project make explicit:
- **Role** — "Lead UX Designer", "Frontend Engineer", not just "Designer".
- **Scope** — "checkout flow redesign", "design system + 3 feature teams" — the concrete slice you owned.
- **Solo or team** — and *what you personally did* on team work.
- **Methods** — research, testing, prototyping approaches used.

Hiring managers cannot assess fit from vague labels. Overclaiming team work as solo destroys trust the moment it surfaces in an interview.

### Length & ratio targets

- **Word count: 600–800 words for junior; 1,000–2,000 words for mid/senior.** 600 words is a bare floor — mid/senior studies that surface research, iterations, and rationale typically run 1,200–1,500 words. Trim ruthlessly for redundancy, not for brevity's own sake.
- **60–80% text / 20–40% visual media** for UX/product portfolios. Motion design, branding, and illustration portfolios may shift toward 50/50. Regardless: never walls of screenshots with no narrative.
- Include **process artifacts** — personas, wireframes, sketches, whiteboard photos, research notes. Without them the portfolio reads as a static gallery and process is invisible.
- Make it scannable: subheadings, visual impact statements, short paragraphs.

### Writing rules

- Show measurable results: "reduced checkout abandonment by X%", "cut support tickets Y%". If you can't share numbers (NDA), describe scope/scale and qualitative outcomes honestly.
- Replace vague descriptions. ❌ "Built a website using PHP and MySQL." ✅ "Rebuilt the booking flow (3 steps → 1) for a clinic with 12k monthly users; reduced no-shows via SMS reminders."
- Ban "intuitive" as a self-descriptor unless backed by usability-test evidence. Every designer claims their work is intuitive — without validation it signals bias over rigor.

## Project pages & architecture

- **Case-study-per-page** beats a single endless scroll for mid/senior portfolios — cleaner URLs, better SEO, easier to link a single study.
- Each project card on the index: thumbnail (with explicit `width`/`height`), title, one-line outcome, role tag. Lazy-load below-fold cards.
- **Navigation: max 2 levels deep.** Typical pages: Home/Work · individual case studies · About · Contact. A blog is worth adding only if you will publish consistently; an empty blog is negative signal.
- **Link to specific curated case studies, not your whole Behance/Dribbble profile.** Forcing the hiring manager to pick reduces engagement.
- Provide a "next project" link at the end of each case study to keep momentum.
- **Password-protecting NDA work:** offer a password-gated section rather than removing the work entirely. On the index card write "Available under NDA — contact me." This tells the reviewer high-value work exists and opens a conversation. Never describe NDA work in enough detail to violate the agreement.

## What hiring managers and clients look for

The "yes/no" decision criteria differ by role. Know your audience.

### Hiring managers (in-house roles)

They review dozens of portfolios per hire. The deciding questions are:

1. **Can I tell what they actually did?** Role, scope, and personal contribution must be unambiguous.
2. **Do outcomes tie to business impact, not just design polish?** Metrics, even proxy metrics, beat screenshots.
3. **Do they show process, not just deliverables?** Research artifacts, decision rationale, and iteration evidence prove the thinking behind the work.
4. **Is the craft level right for the seniority?** Typography, spacing, and visual hierarchy in the portfolio itself are the silent submission.
5. **Are they honest?** Mislabeled spec work, overclaimed team contributions, or inflated metrics will surface in interview. Reviewers prefer candid constraints over vague positives.
6. **Can they communicate?** Writing quality in case studies proxies for writing quality in design docs, specs, and stakeholder presentations.

**What makes them skip:** no metrics, process-free galleries, identical bootcamp templates, vague role labels, buried or broken contact.

### Clients and freelance buyers

They scan differently — they want assurance, not depth.

1. **Social proof:** named clients, recognizable logos, testimonial quotes.
2. **Domain match:** have you done *their kind of problem*? Show it in the first two cards.
3. **Clear service and pricing signal:** "I design landing pages and conversion flows for SaaS companies" is better than "UX/UI Design." Vague scope = price anxiety.
4. **Easy contact with low friction:** a visible email or a short inline form. Long contact forms kill inquiries.
5. **Speed and trustworthiness of the site itself:** a slow or broken personal site signals what working with you might be like.

**What makes them bounce:** no niche, no social proof, no prices or range, hard-to-find contact.

## Visual identity

- **Typography: 2–3 typefaces max.** Fixed type scale for heading / body / accent — too many size variations destroy hierarchy and readability. Solid developer/portfolio choices: Inter, Poppins, Sora. Expressive display faces work in the hero *only* if paired with minimal supporting content.
- **Color: 2 primary + 1 accent.** Consistent application; the accent carries CTAs and key highlights.
- **Contrast ≥ 4.5:1** for normal text, **≥ 3:1** for large text (18pt+ / 14pt+ bold). Target 7:1 (AAA) on critical copy.
- **Dark mode is expected.** Implement with `prefers-color-scheme`; missing it draws public criticism and reduces comfort/accessibility.

```css
/* Set color-scheme so the browser renders scrollbars, form controls,
   and input backgrounds correctly in each mode before CSS loads (prevents FOUC). */
:root {
  color-scheme: light dark;
  --bg: #ffffff; --fg: #111418; --accent: #2f6fed; /* contrast ≥ 4.5:1 */
}
@media (prefers-color-scheme: dark) {
  :root { --bg: #0d0f12; --fg: #e8eaed; --accent: #6ea0ff; }
}
```

Also add the `<meta name="color-scheme" content="light dark">` tag in `<head>` — this tells the browser even earlier (before CSS parse) which schemes the page supports, fully eliminating flash-of-white on dark-mode loads.

## Balancing creativity with usability

A portfolio is itself a usability sample. Flashy ≠ skilled.

- Motion should *enhance* comprehension, not gate content. Smooth section transitions and subtle hover feedback read as polished; everything-everywhere animation reads as undisciplined.
- Static HTML/CSS-only sites with zero interactivity read as dated — implement at least smooth scroll, subtle hover states, and a scroll-triggered entrance animation on project cards. Pure static is acceptable only for hyper-minimal intentional aesthetics with strong typography.
- Never let an effect delay the LCP/hero content. Animate only `transform` and `opacity`.
- Respect motion preferences:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: .01ms !important; transition-duration: .01ms !important; }
}
```

## Performance — the portfolio is the proof

A portfolio that's slow contradicts every claim of frontend/performance skill. Run Lighthouse + PageSpeed Insights **before** sharing the URL.

- **Targets:** LCP ≤ 2.5s · INP ≤ 200ms · CLS ≤ 0.1, for ≥75% of real visits.
- **Set explicit `width` & `height` (or `aspect-ratio`) on every image/media** to prevent CLS.
- **Animate only `transform`/`opacity`** — never `width`/`height`/`margin`/`top` (these trigger layout reflow and inflate CLS).
- **Hero/LCP image:** WebP/AVIF + `fetchpriority="high"`, and do **not** lazy-load it.
- **Below-fold project cards:** `loading="lazy"`.

```html
<!-- Hero / LCP image: prioritized, never lazy -->
<img src="hero.avif" width="1200" height="800" fetchpriority="high"
     alt="Checkout redesign final screens for Acme Bank">

<!-- Below-fold project card: lazy + sized to avoid CLS -->
<img src="proj-2.webp" width="600" height="400" loading="lazy"
     alt="Onboarding flow wireframes">
```

## Concrete specs & numbers

- **Attention:** 6–8s initial scan · ~15s average screening · ~5 min full review.
- **Case study count:** junior 2–3 · mid/senior 3–5 · above ~6–8 = overwhelming.
- **Case study word count:** junior 600–800 words · mid/senior 1,000–2,000 words (typically 1,200–1,500); 60–80% text / 20–40% media for UX/product.
- **Hero:** name + title + one-line value prop + 1 primary CTA (optionally 1 lower-weight secondary) + 2–3 featured thumbnails.
- **Type:** 2–3 typefaces max; fixed size/weight scale.
- **Color:** 2 primary + 1 accent.
- **Contrast:** 4.5:1 normal · 3:1 large (18pt+/14pt+ bold) · 7:1 AAA target on critical copy.
- **Core Web Vitals:** LCP ≤ 2.5s good / 2.5–4s needs work / >4s poor · INP ≤ 200ms / 200–500 / >500 · CLS ≤ 0.1 / 0.1–0.25 / >0.25. Pass for ≥75% of CrUX real-user visits.
- **INP replaced FID** as a Core Web Vital on **March 12, 2024**.
- **Update cadence:** every 3–6 months — add projects, update stack, re-verify every live link.
- **Accessibility baseline (WebAIM Million 2024):** low-contrast text on 81% of homepages; avg 56.8 errors/page; 95.9% of homepages had ≥1 WCAG failure.

## SEO & discoverability

Hiring managers and clients who don't already have your URL find you via search. A portfolio with no SEO attention is invisible to organic discovery.

- **Custom domain is non-negotiable** for anything serious. `yourname.com` or `yourname.dev`. Subdomain-only URLs (`yourname.framer.website`) signal unprofessionalism and pass zero domain authority.
- **`<title>` tag:** "[Your Name] — [Title], [Domain]" (e.g. "Alex Park — Senior Product Designer, fintech"). Keep under 60 characters.
- **`<meta name="description">`: 120–155 characters.** Write it like a search result snippet, not a tagline. Include your specialty and one differentiator.
- **`<meta name="color-scheme" content="light dark">` in `<head>`** — not strictly SEO but prevents flash-of-white before CSS loads in dark mode.
- **Structured `<h1>` per page.** One `<h1>` per page; usually your name on the index, the project title on case study pages.
- **Case-study-per-page URLs** are individually indexable and linkable: `/work/checkout-redesign-acme` ranks better than a monolithic scroll.
- **Open Graph meta tags** for every page — when you share the URL on LinkedIn, Slack, or email, the preview image and title must render correctly. A missing or broken OG preview looks amateurish.
- **Alt text on all meaningful images** — accessibility *and* image search signal. Describe what's shown: "Final checkout flow screens for Acme Bank mobile app", not "design1.png".
- **LinkedIn and GitHub must point to the portfolio** — most recruiters hit LinkedIn before the portfolio; the URL in your profile must be current and not redirecting to a 404.

## Tools & libraries

| Tool | Use it when | Avoid / caution |
|---|---|---|
| **Framer** | Custom interactive portfolio, Figma integration, fast templates | Over-animating because it's easy; published bundle size can bloat |
| **Webflow** | CMS-driven, case-study-per-page, rich custom interactions, full control | Bloated published CSS/JS if undisciplined |
| **UXfolio** | UX/product designers wanting guided per-section prompts; beats blank-page paralysis | Looks templated if you don't customize the layout and typography |
| **Figma Sites** | Publish straight from Figma for simple showcases | No native custom domain analytics, limited SEO control, zero dynamic routing; not suitable for case-study-per-page architecture |
| **Notion / PDF deck** | Acceptable fallback *if well-designed* | Default Notion styling reads as low-effort; no custom domain without third-party service |
| **Astro + Tailwind + TS** | Developer portfolios: zero-JS-by-default ships the fastest static sites; first-class MDX for case studies; deploys to Vercel/Netlify | Requires comfort with component islands if you add interactivity |
| **Next.js 15 + Tailwind + TS + Framer Motion** | Dev portfolio when you need React interactivity or API routes (e.g. contact form, dynamic OG images) | Heavier default bundle than Astro for static-only content |
| **v0 / Cursor AI-generated** | Rapid scaffold of layout + components before hand-tuning | Raw output is generic; publish only after significant design differentiation |
| **Vercel / Netlify** | Deployment; Vercel has CWV-friendly Edge Network | — |
| **PageSpeed Insights / Lighthouse** | Audit CWV, a11y, SEO **before** sharing the URL | Don't ship without running it |

## Do / Don't

**Do**
- Lead every case study with the outcome/metric.
- State role, scope, solo/team, and methods on each project.
- Curate 3–5 deep studies; link to specific case studies, not whole profiles.
- Include process artifacts (sketches, wireframes, research).
- Keep one primary CTA in the hero; make contact reachable from every page.
- Label spec/redesign/passion projects honestly; gate NDA work with a password.
- Set a custom domain; write proper `<title>`, meta description, and OG tags.
- Hit LCP ≤ 2.5s / INP ≤ 200ms / CLS ≤ 0.1; size all media; serve WebP/AVIF.
- Support dark mode (`prefers-color-scheme` + `color-scheme` meta) and `prefers-reduced-motion`.
- Test responsive layouts on a real mobile device; audit every link before sharing.

**Don't**
- Lead with process and hide metrics at the bottom.
- Use skill percentage bars or "intuitive" as a self-claim.
- Ship cookie-cutter bootcamp case studies (e.g. the templated "Plannie" project) — instant rejection on recognition.
- Overclaim team projects as solo.
- Use slow typewriter/reveal animations that gate the hero content.
- Use generic stock imagery in the hero.
- Animate `width`/`height`/`margin`; leave images unsized (causes CLS).
- Leave dead links, 404 case studies, or broken live demos.
- List "Google certified" as a headline credential.
- Use a subdomain-only URL for anything beyond a student portfolio.

## Common pitfalls & how to avoid them

- **Process-first burial of metrics.** Reorder to impact-first; the reader stops after ~5 min and never reaches buried results.
- **Bootcamp template tell.** Identical "Sole UX/UI Designer" + the same fictional app across portfolios. Fix: original problems, explicit personal contribution, honest labels.
- **Linking a whole Behance/Dribbble profile.** Curate and link individual case studies so you, not the reader, make the selection.
- **Animation overload.** Reserve motion for comprehension and feedback; never delay content visibility.
- **Non-responsive layouts.** Mobile traffic is a meaningful share of portfolio visits; a broken mobile view signals inability to think cross-platform.
- **Vague descriptions.** Always: problem → constraints → your specific solution → measurable result.
- **Tutorial/clone projects (ToDo, weather app) without differentiation.** Signals junior mindset; 3–5 original problems win.
- **Too many font sizes.** Lock a fixed scale; hierarchy must be scannable at a glance.
- **Missing alt text.** Ethically wrong, hurts SEO, fails WCAG. Every meaningful image gets descriptive `alt`.
- **Buried/missing contact.** Sticky nav + Contact CTA on every page.
- **Slow portfolio.** Run Lighthouse before sharing; LCP > 2.5s undermines any frontend claim.
- **No custom domain / subdomain URL only.** `yourname.framer.website` signals you haven't committed enough to buy a $12/year domain.
- **Missing or broken Open Graph preview.** When a recruiter pastes the URL into a Slack message or LinkedIn post, a blank preview or wrong image is the first impression. Verify it with [opengraph.xyz](https://opengraph.xyz) or the LinkedIn Post Inspector before sharing.

## What real users complain about

Common complaints documented across hiring-manager commentary and designer/developer community discussions (r/UXDesign, r/webdev, r/userexperience):

- **Bootcamp déjà vu** — multiple hiring managers report instant rejection when they recognize templated work (named example: the Avocademy / "Moment Studio" Plannie project) by its identical role label and fictional app.
- **"Wait for the hero to finish animating"** — typewriter/reveal effects that delay information; employers won't wait.
- **"What did *you* actually do?"** — vague "Sole UX Designer" labels on obvious team projects leave reviewers unable to assess personal fit.
- **Skill bars are mocked** — "HTML 95% / CSS 90%" is universally treated as meaningless and removed by experienced designers.
- **"Intuitive" eye-roll** — using it without validation evidence reads as bias over rigor.
- **No dark mode** — public feedback flags it as an accessibility/comfort gap.
- **Dated static sites** — pure HTML/CSS with zero interactivity (no hover states, no smooth scroll, no entrance animation) reads as behind the current market.
- **Cert-as-headline mockery** — "I am Google certified" up top is seen as awareness, not mastery.
- **The irony of a slow portfolio** — a sluggish site from someone claiming performance/frontend expertise is the most cited credibility killer.

## Senior-level checklist

**Content & case studies**
- [ ] Hero states name + title + one-line value prop; passes the 6–8 second test with 3 non-designers.
- [ ] Exactly one primary CTA in the hero.
- [ ] 3–5 curated case studies (2–3 if junior), each leading with outcome/impact.
- [ ] Every case study states role, scope, solo/team, and methods.
- [ ] UX/product case studies: 1,000–2,000 words for mid/senior; 60–80% text / 20–40% media; process artifacts included.
- [ ] Eng case studies: problem → technical decision → measurable outcome; technical rationale stated.
- [ ] No skill percentage bars; no unvalidated "intuitive"; no cert-as-headline.
- [ ] Spec/redesign/passion projects explicitly labeled; no team work overclaimed as solo.
- [ ] NDA work: password-gated or described at index level with "Available under NDA — contact me."

**Navigation & contact**
- [ ] Navigation max 2 levels deep; Contact reachable from every page via sticky nav + Contact CTA.
- [ ] All external links, live demos, and case-study pages verified — zero 404s.
- [ ] "Next project" link at end of each case study.

**Visual identity**
- [ ] 2–3 typefaces max on a fixed scale; 2 primary + 1 accent color.
- [ ] Text contrast ≥ 4.5:1 (3:1 large); descriptive `alt` on every meaningful image.
- [ ] Dark mode via `prefers-color-scheme` + `color-scheme: light dark` in `:root` + `<meta name="color-scheme">` in `<head>`.
- [ ] Motion respects `prefers-reduced-motion`.

**Performance & accessibility**
- [ ] Every image/media has explicit `width`/`height` or `aspect-ratio`.
- [ ] Hero/LCP image: WebP/AVIF + `fetchpriority="high"`, not lazy-loaded; below-fold cards lazy-loaded.
- [ ] Lighthouse/PageSpeed run: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1 for ≥75% of visits.
- [ ] Fully responsive; verified on a real mobile device.

**SEO & discoverability**
- [ ] Custom domain (`yourname.com` or `yourname.dev`); no subdomain-only URL.
- [ ] `<title>` under 60 characters; `<meta name="description">` 120–155 characters; one `<h1>` per page.
- [ ] Open Graph meta tags set for every page; OG preview image verified when URL is pasted in a chat/social tool.
- [ ] LinkedIn profile URL and GitHub profile URL both point to the current portfolio.

**Maintenance**
- [ ] Update reminder set for 3–6 months out; re-verify all links at each update.

## Sources

- WebAIM Million 2024 report (homepage accessibility / low-contrast text statistics) — https://webaim.org/projects/million/
- Google Core Web Vitals & CrUX thresholds (LCP/INP/CLS; INP replaced FID on 2024-03-12) — https://web.dev/articles/vitals
- Google PageSpeed Insights — https://pagespeed.web.dev/
- Morgane Peng — writing on UX portfolios and case studies (results-first quote) — https://www.morganepeng.com/

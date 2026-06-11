# SaaS Web Apps

> Patterns for building the authenticated, recurring-use core of a SaaS product: app shell, navigation, onboarding and activation, empty states, settings, billing, in-app guidance, command palettes, multi-step flows, and the trial→paid→retention loop. The senior-level outcome is a product where users reach value fast (low time-to-value), discover features in context, and form habits that convert and retain — without dark patterns.

**Apply when:** building or redesigning the logged-in surface of a subscription product (dashboard app, workspace, tool) · **Related:** [CRM & Admin Dashboards](./crm-admin-dashboards.md) · [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md) · [Cognitive Load & Progressive Disclosure](../01-ux-fundamentals/cognitive-load-progressive-disclosure.md) · [Conversion Rate Optimization](../06-marketing-and-conversion/conversion-rate-optimization.md)

## TL;DR — rules at a glance
- **Time-to-Value is the north-star.** Aim for first Aha moment under 10 minutes for self-serve. Users who hit the Aha moment early convert and retain substantially better — but the specific multiplier is product-specific; measure yours with cohort analysis.
- **App shell = left sidebar (240px, collapse to 64px icon rail) + top bar** (account switcher, ⌘K search, notification bell, help, avatar). Main content in semantic `<main>`.
- **Sidebar for >5 major sections + deep hierarchy.** Top nav only for shallow, content-oriented surfaces.
- **Empty states are capability demos, not blank screens.** Show templates, seed/sample data, or a single clear CTA — never "No data."
- **Onboarding: infer, don't interrogate.** Pull context from URL/referrer/existing data. Qualify with ≤3 questions, then route to a persona-specific path. Connect every checklist item to a real outcome, not a feature tour.
- **Never gate the Aha moment behind payment.** Defer card collection — deferred-payment users convert substantially higher than forced-card flows; surface upgrade prompts at natural limits and right after the Aha moment.
- **3-tier pricing, center-stage middle plan.** Default to annual toggle. Add comparison table and money-back guarantee. Pricing page must load <2s.
- **Tours: 3 steps (72% completion) not 7+ (16%).** Prefer interactive walkthroughs over docs-only. (Vendor-sourced benchmarks — treat as directional, not law; measure your own.)
- **Command palette (⌘K):** centered modal, dimmed backdrop, grouped results, footer hints. Use `cmdk` (unstyled, actively maintained) or Headless UI combobox primitives.
- **Settings:** left sidebar nav, uniform per-section layout (header + description + controls), search-at-top, visually distinct danger zone.
- **Multi-step flows:** one decision per screen, progress indicator (cuts abandonment up to 30%), Back + Save-exit on every step, resume later.
- **Auto-save + status indicator + Undo.** Deep-link every meaningful state into the URL. Persist sidebar collapse in localStorage.
- **WCAG 2.2 AA is the procurement baseline**, not a nicety: 4.5:1 text contrast, 44×44px targets, visible focus rings, skip-nav, ARIA live regions for toasts.
- **Trial activation IS phase 1 of retention.** Drive weekly habit formation; habitual users convert at 3–4x sporadic ones.

---

## 1. App shell & navigation

The shell is the persistent frame around every authenticated view. Get it right once; it carries every screen.

### Anatomy
```
┌──────────────────────────────────────────────────────┐
│ TopBar: [☰] [Workspace ▾]   [🔍 Search ⌘K]  [🔔][?][◓]│  56–64px tall
├──────────┬───────────────────────────────────────────┤
│ Sidebar  │  <main>                                    │
│ 240px    │   page header (title + primary action)     │
│ (→64px   │   content                                  │
│  rail)   │                                            │
│          │                                            │
│ [profile]│                                            │
└──────────┴───────────────────────────────────────────┘
```

- **Top bar** carries global, cross-section controls: account/workspace switcher, global search (`⌘K`), notification bell, help, user avatar/menu.
- **Sidebar** carries primary navigation (the product's sections). Wrap main content in semantic `<main>`; the sidebar in `<nav aria-label="Primary">`.
- **Sidebar vs top nav decision:** sidebar when the product has **>5 major sections** and deep hierarchy (most SaaS apps). Top nav for shallow, content-oriented sites (marketing, docs, ≤5 sections).

### Sidebar specs
- Expanded width **220–260px** (260–300px for content-heavy SaaS); collapsed icon rail **56–72px** (64px is the safe default).
- Item height **40–48px**; icon size **20–24px** (max 24px), **24px** in collapsed rail.
- Horizontal padding **16px** each side. Group spacing **16–24px** before category headers. Stick to an **8px baseline grid**.
- Labels **14–16px** primary, **13–14px** secondary; category headers **11–12px uppercase**, letter-spacing **1.5–2px**.
- Max **5–7 primary top-level items**. Secondary items indented **16–24px**. Max **3 hierarchy tiers** — deeper means broken IA.
- Collapse animation **200–300ms ease**. **Persist the collapse state in localStorage**; never reset on load.

```js
// Persist + restore sidebar collapse state
const KEY = 'sidebar:collapsed';
const collapsed = localStorage.getItem(KEY) === '1';
function toggleSidebar(next) {
  localStorage.setItem(KEY, next ? '1' : '0');
  document.documentElement.dataset.sidebar = next ? 'collapsed' : 'expanded';
}
```

- **New users:** consider starting expanded so they learn available sections (Hootsuite pattern); power users tend to collapse for content space.
- **Active state** must be unambiguous: filled background or left accent bar + bolder label, not color alone (a11y).

### SPA accessibility rules
- **Skip-nav link:** first focusable element in the DOM must be a `<a href="#main-content" class="sr-only focus:not-sr-only">Skip to main content</a>`. Without it, keyboard users tab through the entire sidebar on every route change.
- **Focus management on route change:** when the user navigates to a new route, programmatically move focus to the page `<h1>` or the `<main>` element. Don't leave focus stranded in the old page's DOM. Wrap route transitions with `document.getElementById('main-content').focus()` or equivalent.
- **ARIA live regions for toasts:** wrap the toast container in `<div aria-live="polite" aria-atomic="false">` so screen readers announce new notifications without interrupting ongoing speech.
- **Focus trap for modals and command palette:** use `inert` on non-modal content (`<div inert>`) or the `focus-trap` library; restore focus to the trigger element on close.

### Responsive shell
- **<768px:** hide the sidebar. Replace with either a **bottom tab bar (max 5 items)** for app-like flows, or a **full-screen overlay drawer** for richer hierarchy.
- Move the page's primary action to a thumb-reachable spot (bottom-right FAB or sticky bottom bar) on mobile.
- Top bar collapses search into an icon; keep notification + avatar accessible.
- See [Responsive & Fluid Design](../02-responsive-adaptive/responsive-and-fluid-design.md) and [Touch Ergonomics](../02-responsive-adaptive/touch-ergonomics-cross-device.md).

### URL is state
Deep-link **every meaningful state** into the URL: open record, active tab, applied filters, selected item. This makes states shareable, bookmarkable, and back/forward-safe. Losing state on browser-back is a top SPA complaint.

---

## 2. Onboarding & activation

Activation = the user reaches the moment that proves the product works for them. Early-week churn is the dominant SaaS revenue leak — most products lose the majority of trial users before they reach core value.

### Define your Aha moment with data, not opinion
The Aha moment is a measurable threshold correlated with retention. Examples:
- **Slack:** a team sending **2,000 messages** (a 2014 self-reported approximation; Slack has since noted the actual signal is more nuanced — treat as illustrative, not gospel).
- **Zapier:** building the **first working automation** (in minutes).
- **Notion / Trello:** first populated board/page (often via a seeded template).

Find yours analytically: **cohort users by N-day retention, then work backward to identify which early action is most predictive.** Don't copy another product's Aha moment. Then optimize the entire early experience around reaching it fast.

### Inference over interrogation
Don't gate value behind a configuration form. Infer context from **URL, referrer, OAuth profile, invited-workspace data, or email domain**. Pre-fill with smart defaults. Ask only what you cannot infer.

### Qualify, then route (role-based onboarding)
Ask **2–3 questions max**, then route to a **persona-specific** path. Role-based routing consistently outperforms a single generic flow — because a solo freelancer and a 50-person ops team need different initial states and different Aha moments.
```
Q: "What brings you here?"  → [Track projects] [Manage a team] [Just exploring]
   ↳ each answer loads a different seeded workspace + checklist + first action
```

### AI-assisted onboarding (2024–2026 pattern)
Products like Notion, Gamma, and Canva now run an **LLM-powered setup interview** at signup: the agent asks 2–4 natural-language questions, then generates a pre-filled workspace (page hierarchy, board structure, draft content) in seconds. This removes the "blank canvas" problem entirely without pre-built templates. If your product's core object can be AI-generated from a short brief, build this instead of a traditional wizard. Keep the handoff clearly human-controllable — never lock users into AI-generated content they can't easily edit.

### Checklists that drive outcomes
- Every checklist item maps to a **first meaningful outcome**, not a feature inventory ("Create your first invoice" not "Tour the toolbar").
- Show progress ("2 of 4"). Make the first item trivially completable to build momentum.
- Order items toward the Aha moment; the last item should *be* the Aha moment or unlock it.

### Tours: short and interactive
- **3-step tours complete at ~72%; 7+ steps at ~16%.** Cap product tours at 3–4 steps. (Benchmarks from Chameleon.io — a vendor with commercial interest in this data; treat as directional.)
- **Interactive walkthroughs outperform docs-only** — let users *do*, not just observe. Users who take an action in step 1 are substantially more likely to complete the tour.
- **Adaptive onboarding** (skips steps for actions already completed, accelerates advanced users) outperforms linear scripts — instrument it and test.

### Activation benchmarks (use as targets, not gospel)
- Average SaaS **activation rate ~37.5%**; top performers exceed **45%**. **Below 20% → make activation your entire focus.**
- Average **core feature adoption ~24.5%** (top quartile 28%+). Sector spread: HR ~31%, Insurance ~22.8%, FinTech ~22.6%, mid-market ($5–10M ARR) ~30.4%. (Userpilot 2024 Product Metrics report — single-source vendor data; use for calibration only.)
- Teams **running onboarding experiments consistently outperform non-iterating peers** — instrument and iterate. If you aren't running activation experiments you are leaving conversion on the table.

### Human touchpoints
One **20-minute onboarding call or a Loom walkthrough** consistently lifts trial conversion by **6–12 percentage points**. Offer it (especially to high-intent / larger accounts) without making it a blocker.

### Onboarding emails
**7–10 behavior-triggered emails beat 25 time-based ones.** Segment by **lifecycle stage** (signed up / activated / stuck / power user), not just "day N." Trigger on what the user did or failed to do.

See [Cognitive Load & Progressive Disclosure](../01-ux-fundamentals/cognitive-load-progressive-disclosure.md).

---

## 3. Empty states

An empty state is a designed screen, not the absence of one. **Anatomy:** mandatory **Message** (headline + supporting description); optional **Illustration**; optional **CTA**. Never ship "No data" or a blank canvas.

| Type | Goal | Pattern |
|---|---|---|
| **First-use** | Demonstrate capability | Seed demo/sample data, template gallery, welcome board/video |
| **User-cleared** | Re-engage + reward | Celebrate completion (illustration + encouraging copy), offer next action |
| **Error** | Explain + recover | State what failed, why, and a fix/retry |
| **Zero search results** | Redirect | Suggest alternatives, broaden filters, offer create-new |

**Real patterns:**
- **Notion** — empty state is a template gallery (project tracker, meeting notes, roadmap) populated *before* user input; 2025 onboarding agent personalizes setup via prompts.
- **Trello** — pre-filled welcome board with sample cards teaches the model without docs.
- **Xero** — welcome video at top + first-step suggestions.
- **Pinterest** — eliminated the cold-start blank canvas by showing a personalized feed immediately at signup.
- **GitHub** — cleared-inbox state uses Octocat + encouraging copy (celebratory, not just "All done").
- **Duolingo** — completion states become gamified reward moments.

```html
<!-- First-use empty state -->
<section class="empty" aria-labelledby="empty-h">
  <img src="/illus/projects.svg" alt="" role="presentation" width="160" height="120">
  <h2 id="empty-h">Create your first project</h2>
  <p>Projects keep tasks, files, and your team in one place.
     Start from a template or import existing work.</p>
  <div class="empty__actions">
    <button class="btn-primary">Use a template</button>
    <button class="btn-ghost">Import data</button>
  </div>
</section>
```

---

## 4. Settings

Settings are where power users live and where trust is won or lost. Use **one consistent pattern** across all sections (the Linear model: every settings section is *section header + description line + controls* — uniform across notifications, workspace, API).

- **Layout:** left sidebar nav (sections) + content panel. For deep settings, add **search-at-the-top** so users jump straight to a control.
- **Per-section consistency:** every control group has a label, a one-line description, and the control. Don't reinvent layout per section.
- **Danger zone:** group destructive actions (delete account, transfer ownership, wipe data) into a **visually distinct** block (red border/label). Require typed confirmation for irreversible actions.
- **Group logically:** Account · Workspace/Org · Members & roles · Billing · Notifications · Integrations · API/Developer · Security.
- **Save model:** prefer instant auto-save with a per-row "Saved" confirmation, OR an explicit Save with a dirty-state warning on navigate-away. Pick one model and apply it everywhere.

```
Settings
├ Account        ┌─────────────────────────────────────┐
├ Workspace      │ Notifications                       │
├ Members        │ Control how and when we contact you │
├ Notifications ◀├─────────────────────────────────────┤
├ Billing        │ Email digests      [ Daily  ▾ ]     │
├ Integrations   │ Send a summary…                     │
├ API            │ ─────────────────────────────────   │
└ Security       │ ⚠ Danger zone                       │
                 │ Delete workspace   [ Delete ]       │
                 └─────────────────────────────────────┘
```

---

## 5. Billing & pricing pages

Two distinct surfaces: the **public pricing page** (conversion) and the **in-app billing settings** (management).

### Public pricing page
- **3 tiers is the dominant pattern** — 2 tiers underdifferentiates; 4+ creates choice paralysis. Use the **center-stage effect**: the middle tier must feel like a disproportionately good deal (highlight it, "Most popular" badge), not a linear step up.
- **Default the annual toggle on.** Show monthly-equivalent price and the savings explicitly ("$16/mo billed annually" not just "$192/yr").
- **Comparison table** of features reduces cognitive load and cuts support tickets for pricing questions.
- **Outcome-led copy beats feature-led.** Lead with what the user achieves ("Ship twice as fast"), then list features as evidence. Feature-led copy ("Unlimited projects") is table stakes, not persuasion.
- **Positive framing** ("Includes 10 projects") beats negative ("Limited to 10 projects"). Directionally consistent across numerous A/B tests; test your own.
- **Money-back guarantee** reduces perceived risk. State it plainly, near the CTA, not buried in footnotes.
- **Performance:** page must load **<2s**; each extra second meaningfully costs conversion (Portent study baseline; measure your own). Mobile-optimize the pricing page — mobile is frequently the first touchpoint.
- **Kill abandonment causes:** a large portion of pricing page abandonment traces to suspected hidden costs and unclear cancellation policy (CXL research direction). Show total cost (incl. per-seat math at common team sizes), and link a plain cancellation policy near the CTA.

```
┌──────────┐  ┌════════════┐  ┌──────────┐
│  Starter │  ║   Pro      ║  │ Business │
│  $0      │  ║ ★ POPULAR  ║  │  $49/seat│
│          │  ║  $19/seat  ║  │          │
│ • 3 proj │  ║ • Unlimited║  │ • SSO    │
│ • 1 seat │  ║ • 10 seats ║  │ • Audit  │
│ [Start]  │  ║ [Try free] ║  │ [Contact]│
└──────────┘  └════════════┘  └──────────┘
        ▲ middle tier visually elevated (center-stage)
```

### In-app billing settings
- Transparent **plan comparison**, current plan + usage against limits, **invoice history table** (date, amount, status, download), **payment-method management**.
- **No hidden fees, no ambiguous cancellation.** Make downgrade/cancel reachable in ≤2 clicks — burying it is a dark pattern that returns as support tickets and churn. See [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md).
- Use **Stripe Billing** for subscription/invoice/proration plumbing from MVP day one.

---

## 6. Trial-to-paid conversion

Trial design is where most self-serve revenue is won or lost.

- **Conversion benchmarks:** overall **14–18%**. **Opt-in (no credit card) ~18.2%; opt-out (card required) ~48.8%.** Card-required converts much higher but reduces top-of-funnel volume and raises refund/dispute friction — choose based on your funnel economics, not dogma. (ProfitWell/Paddle aggregate data; varies widely by category.)
- **Defer payment:** never collect the card before the user experiences core value. The conversion delta is real but product-specific — don't use a single multiplier as justification; measure your own funnel.
- **Never gate the Aha moment behind payment.** Let them feel value first, then ask.
- **Social/SSO login:** Google/Apple raises signup conversion substantially (30–40% range is commonly cited). For **developer-targeted SaaS**, prioritize **GitHub/GitLab SSO** alongside Google; Apple matters on iOS-heavy audiences.
- **Upgrade prompts** fire at **natural usage-limit moments** (project cap, seat limit, API quota) and **immediately post-Aha** — **never mid-task**. Show what they get and how close they are to the limit.

### Trial length and in-trial UX
- **14 days is the most common SaaS trial length**; 7-day trials work when time-to-value is short (under 1 day). 30-day trials only make sense when the use case requires a full billing cycle to evaluate (e.g., monthly invoicing tools).
- **Send behavior-triggered in-trial emails**, not day-N blasts: trigger on "signed up but hasn't done X", "did X — next step is Y", "3 days left with N incomplete". Stop sending once activated.
- **Day-N expiry reminders** (typically D-7, D-3, D-1): show remaining time in the app UI itself (top bar or contextual banner), not only email. At expiry, **grace period or read-only mode** beats a hard lock — hard locks create support tickets; grace periods convert.
- **Trial expiry screen:** don't 404. Show: what they built, what they'd lose, plan comparison, a primary CTA (subscribe), and a secondary "export data" path — the export option reduces anxiety and actually increases conversion.

### Freemium reality check
- A free tier **works** when it offers enough to experience value but creates **genuine friction before real momentum**. It **fails** when free users get full utility with **zero incentive to pay**.
- Cautionary pattern: products with large free-user bases and <1% conversion often find free users generating a disproportionate share of support load. Free is a cost center unless deliberately constrained with meaningful paid limits.

---

## 7. In-app guidance & feature discovery

### Contextual surfaces — use each for its right job
| Surface | When to use | Limit |
|---|---|---|
| **Tooltip (hover/focus)** | Explain a non-obvious control in-place | On-demand only; never auto-pop on first load |
| **Hotspot beacon** | Draw attention to a new or underused feature once | 1–2 active beacons max; dismiss on click, never re-show |
| **Inline tip (contextual banner)** | Guide users who are actively doing a task | Dismiss + don't show again; auto-dismiss after action |
| **Spotlight/highlight** | Interactive walkthrough step | Max 3–4 steps; always escapable |
| **Contextual modal** | High-stakes new capability or breaking change | Use sparingly; not for feature announcements |
| **What's New / changelog badge** | Announce features non-disruptively | Badge on bell/help icon; never a forced takeover modal |

- **Surface features at the moment of relevance**, not only in changelogs or emails. A user who just hit a seat limit is the right audience for the Team plan — not a user who's never used collaboration features.
- **Over-tooltipping trains dismissal.** Every time you add guidance, ask: "Will this still be visible after 30 days?" Permanent tooltips that users stopped reading are noise. Use tooltips for controls that are genuinely non-obvious; use labels/microcopy for everything else.
- **Progressive disclosure across lifecycle stages:** Basic → intermediate → advanced controls revealed only when the user demonstrates readiness (used the basic flow, hit a ceiling, or explicitly asked for more). Don't ship power features into the default view.
- **Behavioral personalization > configuration personalization.** Adapt the product automatically based on what the user does. "Set up your preferences" panels shift work onto the user; behavioral adaptation does it for them.
- **Feature announcement checklist:** (1) Is it relevant to this user's segment? (2) Are they in a state to act on it now? (3) Is there a single CTA? (4) Is there a "dismiss and don't show again"? If any answer is no, don't show it yet.

---

## 8. Notifications

### Severity hierarchy → display pattern
| Severity | Examples | Display | ARIA |
|---|---|---|---|
| **High** — Alerts/Errors (immediate action) | payment failed, job crashed, data loss risk | blocking/persistent banner or modal | `role="alertdialog"` |
| **Medium** — Warnings/Acknowledgments | nearing quota, "saved", needs review | toast + inbox entry | `role="alert"` (live region) |
| **Low** — Info/Badges | new comment, feature tip | badge / quiet inbox entry | `aria-live="polite"` |

- **Display one notification at a time** to prevent cognitive overload. Queue the rest. For toasts: bottom-right corner, **4–6 second auto-dismiss** (longer for errors), never block the primary CTA area.
- **Content priority:** human/personal messages > calendar/transactional > promotional/feature announcements. Never let a promo bury a person. Enforce this in the queuing logic, not as a style guide suggestion.
- Offer **predefined modes** — `calm` / `regular` / `power-user` — and capture **frequency preferences during signup** so the default isn't noise.
- Provide an in-app **notification inbox** (history) in addition to ephemeral toasts; users need to find what flashed by.
- **Browser push permission:** ask only after the user has experienced value (post-Aha, not on first load). Cold permission requests on load are rejected >90% of the time and poison future asks. Show a pre-permission explainer ("We'll notify you when X happens") before triggering the browser dialog.
- **Notification batching/grouping:** for high-volume apps (project tools, CRMs), group related notifications into one digest row ("3 comments on Task #47") rather than flooding the inbox. Unbatched notifications are the fastest path to notification-blindness.
- **Email as fallback:** for events the user may have missed in-app, send an email digest (configurable: real-time / hourly / daily / never). Stop the email once the user has seen the in-app notification.

---

## 9. Command palette (⌘K)

The power-user accelerator and an IA stress test: if a flat command list can't expose the product cleanly, the IA is broken.

- **Layout:** centered modal, **dimmed/blurred backdrop**, full-width search input at top. Results **grouped by type** (Recent, Actions, People, Pages) each row = icon + title + (optional) keyboard shortcut. **Footer hints:** `↑↓ navigate · ↵ open · ⎋ close`.
- **Behavior:** open on `⌘K` / `Ctrl+K`; fuzzy search; keyboard-first; ARIA combobox semantics; trap focus, restore on close.
- **Content:** navigation, actions (create/invite/switch), search across entities, recent items. Make it the fastest path to anything.

```jsx
import { Command } from 'cmdk';

<Command.Dialog open={open} onOpenChange={setOpen} label="Command Menu">
  <Command.Input placeholder="Type a command or search…" autoFocus />
  <Command.List>
    <Command.Empty>No results found.</Command.Empty>
    <Command.Group heading="Actions">
      <Command.Item onSelect={createProject}>＋ New project <kbd>C P</kbd></Command.Item>
      <Command.Item onSelect={invite}>👤 Invite member</Command.Item>
    </Command.Group>
    <Command.Group heading="Recent">{/* recent items */}</Command.Group>
  </Command.List>
  <footer className="cmd-hints">↑↓ navigate · ↵ open · ⎋ close</footer>
</Command.Dialog>
```

Use **`cmdk`** (unstyled, accessible, actively maintained) or **Headless UI combobox + dialog** primitives. **`kbar` is archived** (last meaningful commit 2022) — do not use for new projects. **CommandBar.com** if you want a paid managed service.

---

## 10. Multi-step flows & wizards

For setup, imports, checkout, and complex creation flows.

- **One decision per screen.** Don't cram a 20-field form into "step 1 of 2."
- **Progress indicator** on every step (current step + total) — progress bars cut abandonment substantially; **24% abandon forms when completion status is unclear** (Baymard).
- **Back + Save-and-exit on every step.** Persist progress so users can **resume later** without restarting.
- **Smart defaults pre-filled** to reduce decision fatigue. Validate inline, not only on submit.
- **Auto-save** every few seconds with a status indicator ("Last saved 2 minutes ago") and **Undo/Restore** for destructive ops.
- **Server-side re-validation on every step submission.** Client-side validation speeds UX but is not security. Treat each step POST as independently untrusted input — validate all prior-step data again at final submit, not just the current step's fields. Wizard state reconstructed from client alone is an attack surface.

```html
<nav aria-label="Progress" class="steps">
  <ol>
    <li aria-current="step">1 · Connect</li>
    <li>2 · Map fields</li>
    <li>3 · Review</li>
  </ol>
</nav>
<!-- footer -->
<button class="btn-ghost">← Back</button>
<button class="btn-link">Save & exit</button>
<button class="btn-primary">Continue →</button>
```

See [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md).

---

## 11. Responsive app design

- Sidebar **hides <768px** → bottom tab bar (≤5 items) or overlay drawer.
- **Touch targets ≥44×44px** everywhere (Apple HIG 44×44pt; Material 48×48dp; WCAG 2.5.8 minimum 24×24 CSS px).
- Tables become cards or horizontally scroll with a sticky first column; never force pinch-zoom.
- Move the primary action into thumb reach; keep destructive actions out of accidental-tap zones.
- Modals become full-screen sheets on mobile; the command palette becomes a full-screen search.
- Test the real flows on a phone — not just a narrowed desktop viewport.

### PWA / installability
- For SaaS apps where mobile usage is frequent, add a **Web App Manifest** (`manifest.json`: `name`, `display: standalone`, `theme_color`, `icons` at 192×192 and 512×512) and a **service worker** that caches the app shell. This enables "Add to Home Screen" and offline load of the shell (even if data requires connection).
- Use **`display: standalone`** to remove browser chrome; handle the back gesture explicitly (don't rely on the browser back button when installed).
- Don't prompt for installation immediately on first load — show an install banner only after 2+ sessions or after the user completes a meaningful action. Follow Chrome's [mini-infobar deferral pattern](https://web.dev/customize-install/).
- Service worker scope: cache the app shell (HTML, JS bundles, CSS, icons) via a **cache-first strategy**; fetch API data **network-first with cache fallback**. Never serve stale auth tokens from cache.

---

## 12. Retention UX

**Trial activation is phase one of retention.** Users who establish a **weekly habit convert to paid at 3–4x** the rate of sporadic users.

### Habit loop design
- Design for a **recurring trigger → action → reward** loop. Identify the natural cadence (daily/weekly) and build toward it (digest emails, scheduled value, streaks where appropriate).
- **Second-session speed matters as much as first-session.** If the user's second visit requires re-orienting ("Where was I?"), you're losing them. Deep-link last-viewed state; show "Continue where you left off" on the dashboard.
- Re-engagement empty/cleared states should **celebrate and offer the next action**, keeping the loop alive.
- **Behavioral personalization** keeps the product feeling tailored over time without user effort.

### Leading retention indicators (track these, not just MRR)
- **Weekly active rate** among trial users.
- **Feature breadth** (number of distinct features used in first 14 days) — breadth correlates with retention more strongly than depth in most SaaS categories.
- **Time-to-second-session** (shorter = better; >7 days to second session is a warning signal).
- **Invite/sharing actions** — social hooks are the strongest activation predictor in collaborative tools.

### At-risk users and re-engagement
- Define "at-risk" operationally: e.g., activated but no session in 7+ days, or trial with <3 days remaining and no core action completed.
- **Re-engagement email** for at-risk users: one email, specific to what they did ("You created a project — here's what your team can do next"), single CTA. Not a generic "We miss you."
- **Win-back flow** for churned/cancelled users: 30, 60, 90-day touchpoints with a concrete reason to return (new feature relevant to their use case, limited-time discount). Don't blast all churned users — segment by usage depth before cancel.
- **Cancellation flow:** before completing cancellation, show: what they'll lose, a pause/downgrade option, and one question ("What could we have done better?"). Offer a pause (2–4 weeks) as an alternative — pause converts a portion of would-be cancels. Never make cancellation a dark-pattern maze, but a well-designed off-ramp is not manipulation.

---

## Concrete specs & numbers
- **Sidebar:** expanded 220–260px (260–300px content-heavy); collapsed rail 56–72px (64px default); item height 40–48px; icon 20–24px (rail 24px); padding 16px; group spacing 16–24px; baseline grid 8px.
- **Typography:** labels 14–16px / secondary 13–14px / headers 11–12px uppercase, letter-spacing 1.5–2px.
- **Animation:** collapse 200–300ms ease; persist preference in localStorage.
- **Navigation:** 5–7 primary items; secondary indent 16–24px; max 3 tiers.
- **Responsive:** sidebar hides <768px; bottom tab bar max 5 items.
- **Top bar:** 56–64px tall.
- **Touch targets:** 44×44px min (Apple), 48×48dp (Material), 24×24 CSS px (WCAG 2.5.8).
- **Contrast (WCAG 2.2 AA):** 4.5:1 normal text, 3:1 large text/UI; focus ring 2–3px high-contrast outline, not color-only.
- **Toasts:** 4–6s auto-dismiss; bottom-right; one at a time; ARIA live region.
- **Pricing page:** load <2s.
- **Tours:** 3 steps → ~72% completion; 7+ → ~16% (Chameleon vendor data — directional).
- **Forms:** progress bars cut abandonment; 24% abandon on unclear completion status (Baymard).
- **Activation:** industry avg ~37.5% (top >45%, below 20% = make activation your entire focus) — Userpilot 2024.
- **Trial→paid:** 14–18% overall; opt-in ~18.2%; opt-out ~48.8% (Paddle aggregate). Social login +30–40% signup conversion (directional).
- **Trial length:** 14 days most common; 7 days for short TTV; 30 days for monthly-cadence use cases.
- **SPA a11y:** skip-nav first in DOM; focus to `<main>` on route change; toast container `aria-live="polite"`; modal content uses `inert` on backdrop.

## Tools & libraries
- **Command palette:** `cmdk` (unstyled, accessible, actively maintained) · Headless UI combobox/dialog primitives · CommandBar.com (paid managed). **Do not use `kbar` — archived since 2022.** Don't hand-roll keyboard/ARIA — use a tested primitive.
- **Shells/components:** Shadcn/ui (dashboard shell, sidebar, command-palette blocks) · Saas UI (saas-ui.dev — migrated from Chakra to Park UI / Ark UI base as of v3; verify current stack before adopting).
- **In-app onboarding/guidance:** Userpilot, Appcues, Chameleon.io (tours, checklists, tooltips, surveys). Use when you need non-engineering iteration speed; skip if a few hardcoded steps suffice.
- **Analytics:** Amplitude (activation/adoption funnels), PostHog (OSS analytics + replay + flags, free tier), Microsoft Clarity (free heatmaps/replay), Segment (single event pipe to many destinations).
- **A11y audit:** WAVE + Axe + Lighthouse — run all three for WCAG 2.2 AA.
- **Billing:** Stripe Billing from MVP day one.
- **Pattern references:** SaaSUI (saasui.design), SaaSFrame (saasframe.io — 90+ empty states, 44+ billing UIs), NicelyDone (nicelydone.club), Mobbin.

## Do / Don't
**Do**
- Define the Aha moment analytically (cohort by retention), not by opinion. Measure TTV.
- Infer onboarding context; ask ≤3 questions; route by persona. Consider AI-assisted setup for blank-canvas products.
- Make empty states demonstrate capability (templates/sample data/AI-generated scaffold).
- Use 3 tiers with a center-stage middle plan, annual default, and per-seat math visible.
- Defer payment; surface upgrade prompts at natural limits and post-Aha.
- Deep-link state into the URL; auto-save with status + Undo.
- Persist sidebar collapse; keep one settings layout pattern across all sections.
- Show one notification at a time; batch/group high-volume notifications; defer browser push permission.
- Run onboarding experiments and instrument activation. Track feature breadth and time-to-second-session.
- Design a graceful trial expiry screen and offer a pause option in cancellation flows.
- Use skip-nav, route-change focus management, and ARIA live regions — not just color/contrast.

**Don't**
- Don't gate the Aha moment behind a credit card or a config form.
- Don't ship blank empty states ("No data") or 7+ step tours.
- Don't bury cancellation/downgrade or imply hidden costs — it returns as chargebacks and negative reviews.
- Don't interrupt mid-task with upgrade prompts.
- Don't rely on color alone for active/focus states.
- Don't reset sidebar/UI preferences on reload.
- Don't use dark patterns for short-term conversion — they return as tickets, low NPS, and churn.
- Don't add a fourth pricing tier "for completeness."
- Don't dump time-based onboarding emails; use behavior triggers.
- Don't use kbar for new projects (archived).
- Don't hard-lock the app on trial expiry without a grace period or read-only mode.
- Don't trust client-only wizard state on final submit — re-validate server-side.

## Common pitfalls & how to avoid them
- **Feature-tour onboarding** instead of outcome-driven → tie every step to a first real outcome; cut tours to 3–4 steps.
- **Configuration walls before value** → infer + pre-fill; defer setup until after the Aha moment.
- **Blank empty states** → design first-use, cleared, error, and zero-results variants with Message (+illustration/CTA).
- **Pricing distrust** → show total cost (with per-seat math at common team sizes), link cancellation policy, use positive framing, keep it <2s.
- **Notification overload** → severity hierarchy, one-at-a-time, content priority (human > transactional > promo), preference modes, batch/group at volume.
- **State lost on browser-back** → deep-link state into the URL.
- **Buried cancellation** → reachable in ≤2 clicks; treat removing it as a dark pattern.
- **Premature/mid-task upgrade nags** → fire at natural limits and right after value.
- **Free tier with full utility** → constrain free so momentum creates a real reason to pay.
- **Inaccessible shell** → skip-nav, route-change focus management, ARIA live regions, run WAVE/Axe/Lighthouse; AA is a procurement gate for enterprise.
- **Hard-lock on trial expiry** → show a grace period or read-only mode; add a well-designed expiry screen with what-you'd-lose framing.
- **kbar for command palette** → archived since 2022; use cmdk or Headless UI primitives.
- **Tooltip everything** → over-tooltipping trains dismissal; use tooltips only for genuinely non-obvious controls.
- **No second-session continuity** → deep-link last-viewed state; show "Continue where you left off" to reduce re-orientation cost.
- **Client-side-only wizard validation** → re-validate all steps server-side at final submit; wizard state from client is an attack surface.

## What real users complain about
- *"Made me fill out a huge form before I could even see the product."* — interrogation onboarding kills momentum.
- *"Couldn't tell if my changes saved."* — missing auto-save status / dirty-state indication.
- *"Browser back wiped everything I'd set up."* — SPA state not in the URL.
- *"Cancelling took forever to find / required emailing support."* — buried cancellation; one of the loudest sources of negative reviews and chargebacks.
- *"Pricing hid the per-seat math; the real bill was way higher."* — opaque total cost; suspected hidden costs are a leading driver of pricing page abandonment (CXL research direction).
- *"Got 10 onboarding emails in two days about features I'll never use."* — time-based blast vs behavior triggers.
- *"The tour had a dozen steps; I clicked through without reading."* — long tours train dismissal.
- *"Notifications buried a real message from my teammate under feature ads."* — wrong content priority.
- *"Empty dashboard just said 'No data' — I had no idea what to do."* — undesigned first-use state.
- *"The sidebar kept re-expanding every reload."* — un-persisted UI preference.

See [What Real Users Complain About](../08-pitfalls-and-antipatterns/real-user-complaints.md).

## Senior-level checklist
- [ ] Aha moment defined as a measurable, retention-correlated threshold via cohort analysis; TTV instrumented and targeted (<10 min self-serve).
- [ ] Onboarding infers context (URL/referrer/OAuth/domain), pre-fills defaults, asks ≤3 questions, routes by persona. AI-generated setup considered if core object can be instantiated from a brief.
- [ ] Activation checklist items each map to a real outcome; first item trivially completable; last unlocks the Aha moment.
- [ ] Tours ≤3–4 steps and interactive; adaptive (skips completed steps); behavior-triggered email sequence (7–10), segmented by lifecycle stage.
- [ ] All four empty-state types designed (first-use, cleared, error, zero-results) with Message + optional illustration/CTA.
- [ ] App shell: `<main>` + `<nav aria-label="Primary">` semantics; sidebar 240px↔64px, collapse persisted in localStorage; top bar has ⌘K, bell, help, avatar, workspace switcher.
- [ ] Skip-nav link first in DOM; focus moved to `<main>` on every SPA route change; ARIA live region on toast container; `inert` or focus-trap on modals.
- [ ] Command palette (⌘K) with grouped results, footer hints, ARIA combobox, focus trap/restore — built on cmdk or Headless UI (not kbar).
- [ ] Settings: uniform per-section layout, search-at-top, distinct danger zone with typed confirmation. One save model applied consistently.
- [ ] Pricing: 3 tiers, center-stage middle, annual default, comparison table, money-back guarantee, total cost (per-seat math) visible, cancellation policy linked, loads <2s, mobile-optimized.
- [ ] Trial defers payment; upgrade prompts fire at natural limits + post-Aha, never mid-task; social/SSO (incl. GitHub for dev tools). Trial expiry handled gracefully (grace period/read-only, not hard lock).
- [ ] Multi-step flows: one decision/screen, progress indicator, Back + Save-exit, resumable, auto-save + status + Undo. **Server re-validates all steps on final submit.**
- [ ] Every meaningful state deep-linked into the URL (back/forward/share/bookmark safe).
- [ ] Notifications: severity hierarchy, one-at-a-time, content priority, calm/regular/power-user modes captured at signup; browser push permission deferred post-Aha; notifications batched/grouped for high-volume surfaces.
- [ ] Responsive: sidebar→tab bar/drawer <768px; 44×44px targets; tables degrade gracefully. PWA manifest + service worker for app-shell installability (if mobile usage is frequent).
- [ ] Retention leading indicators tracked (weekly active, feature breadth, time-to-second-session); at-risk cohort defined and re-engagement flow exists; cancellation flow offers pause option.
- [ ] WCAG 2.2 AA verified with WAVE + Axe + Lighthouse (4.5:1 text, visible focus, no color-only signals).
- [ ] No dark patterns anywhere in billing/cancellation/notification opt-out.

## Sources

> **Note on benchmarks in this guide:** many activation/conversion statistics circulate from vendor research (Chameleon, Userpilot, ProfitWell/Paddle). These are directionally useful but carry vendor bias and survivorship effects. Treat them as calibration, not law. Measure your own product's funnel.

- Chameleon.io — product tour completion rates (vendor data): https://www.chameleon.io
- Baymard Institute — form abandonment & progress indicators (independent research, 45k+ hours of UX study): https://baymard.com
- ProfitWell / Paddle — trial conversion benchmarks: https://www.paddle.com
- CXL — pricing page conversion and abandonment patterns: https://cxl.com
- Userpilot — Product Metrics Benchmark Report 2024 (activation/adoption rates by segment): https://userpilot.com
- WCAG 2.2 (W3C) — AA conformance criteria: https://www.w3.org/TR/WCAG22/
- web.dev — PWA install patterns and service worker best practices: https://web.dev
- cmdk: https://github.com/pacocoursey/cmdk
- Headless UI: https://headlessui.com
- Shadcn/ui: https://ui.shadcn.com
- Saas UI: https://saas-ui.dev
- Userpilot: https://userpilot.com · Appcues: https://www.appcues.com · Chameleon.io: https://www.chameleon.io
- Amplitude: https://amplitude.com · PostHog: https://posthog.com · Microsoft Clarity: https://clarity.microsoft.com · Segment: https://segment.com
- Stripe Billing: https://stripe.com/billing
- WAVE: https://wave.webaim.org · Axe: https://www.deque.com/axe/ · Lighthouse: https://developer.chrome.com/docs/lighthouse
- SaaSFrame: https://www.saasframe.io · SaaSUI: https://saasui.design · NicelyDone: https://nicelydone.club · Mobbin: https://mobbin.com

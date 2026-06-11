# Analytics Instrumentation & Measuring Design Impact

> The measurement layer every conversion, activation, and retention guide silently assumes exists. Covers designing a consistent event taxonomy, the tracking plan as a versioned design deliverable, event-vs-property modeling, the GTM/dataLayer-vs-SDK decision, identity stitching, consent-gated firing, metric-model selection (HEART, North Star, AARRR), and instrumentation QA. The senior outcome: ship features whose impact you can actually prove, with data clean enough to trust on the first read.

**Apply when:** building any site/app where a stakeholder will later ask "did the redesign work?" · **Related:** [Conversion Rate Optimization](./conversion-rate-optimization.md) · [Lifecycle, Email & Notification Design](./lifecycle-email-and-notification-design.md) · [SEO & Discoverability](./seo-and-discoverability.md) · [User Research & Usability Testing](../01-ux-fundamentals/user-research-and-testing.md)

## TL;DR — rules at a glance
- **The tracking plan is a design deliverable, not code.** A versioned contract (event, properties, types, firing conditions) authored during feature design — top-down from business goal → metric → events. Retrofitting forces expensive refactors.
- **Goals-first, never event-first.** Derive the *minimum* events to compute the metric that proves the goal. "Track everything" buries signal and inflates cost.
- **One object-action naming framework, locked forever.** `Object` + past-tense `Action` → `signup_completed`, `checkout_started`. Lock casing, tense, word order, enforce ruthlessly. Consistency matters more than which framework.
- **Model variants as properties, not new events.** `product_purchased` + `{product_name, platform}` beats `purchased_blue_widget` and `ios_order_completed`. Keeps the namespace bounded and analysis cross-cuttable.
- **Three property tiers, modeled separately:** event properties (this action), user properties (the person, mutable, for cohorting), super/system properties (every event). An event property and a user property must never share a name.
- **Decouple tracking from the UI via a dataLayer.** Devs push structured events; tags read explicit keys. DOM-scraping triggers (CSS class, button text) break silently on any redesign or A/B test.
- **Consent fires first.** `gtag('consent','default',{denied})` before any `config`/`event`. A GA4 tag that fires before the default command behaves *as if consent was granted* — the #1 compliance failure.
- **Use a metric system, not one number.** North Star (leading) atop 6–10 input metrics + 3–5 guardrails + lagging outcomes. Optimize the system.
- **Guardrails exist because metrics-as-targets get gamed** (Goodhart's Law). They measure what success should *not* look like.
- **North Star must be leading,** not lagging — an early activation behavior that predicts retained value (e.g. "3+ active members in week 1"), not raw signups or MRR. Validate the correlation before adopting it.
- **Never double-instrument.** The same event hard-coded *and* via GTM is the canonical double-fire.
- **Treat an SRM as a hard stop.** Sample-ratio mismatch = unknown bias of unknown size/direction. Don't salvage — find root cause, fix, rerun with a fresh salt.
- **Instrumentation is part of Definition of Done.** No tracking plan, no launch signoff. Events not live = feature not shipped.

## 1. The tracking plan: a versioned design contract

A **tracking plan** is the single source of truth that specifies every event, its properties, their types, and *when each fires*. It is a contract between three audiences: product (defines goals), engineering (implements), and analytics consumers (query it). Author it **during** feature design, not after.

**Minimum row schema** (one row per event):

| Field | Example | Notes |
|---|---|---|
| Event name | `checkout_started` | Object-action, locked casing |
| Trigger / WHEN | "Fires when user clicks *Proceed to Checkout* on the cart page" | Behavioral, not "on button" |
| Properties | `cart_value_usd`, `items_count`, `currency` | Listed with each event |
| Property type | `number`, `integer`, `string` | Enables schema validation |
| Required? | required / optional | Drives QA failures |
| Owner | `growth-team` | Who can change it |
| Sample payload | `{ "cart_value_usd": 49.99, ... }` | Better plans add this |

**Top-down derivation (the only correct order):**

```
Business goal:   Increase paid conversions
   → Metric:     trial → paid conversion rate
      → Events:  trial_started, paywall_viewed, plan_selected, subscription_purchased
         → Props: plan_name, price_usd, billing_interval, trial_day_number
```

> If you can't name the metric a proposed event serves, don't add the event. "Track everything" is an anti-pattern: it inflates storage cost, slows queries, and dilutes the signal you actually need.

## 2. Naming taxonomy — object-action, locked

Pick **one** framework and never deviate. The dominant pattern is **Object + past-tense Action**:

- An object carries many actions: `song_played`, `song_paused`, `song_skipped`.
- **Past tense** because an event records a *completed fact* — `signup_completed`, not `complete_signup` or `CompleteSignup`.
- Lock three axes: **casing**, **tense**, **word order**. The framework choice matters less than ruthless consistency.

**Casing.** snake_case, camelCase, Title Case, or lower case are all defensible; pick one. A common convention: **Title Case for events, snake_case for properties** (mirrors how properties are stored/queried in SQL).

**Semantic property suffixes** (apply everywhere):

| Suffix | Meaning | Example |
|---|---|---|
| `_at` | ISO timestamp | `created_at` |
| `_id` | identifier | `order_id` |
| `_count` | a count | `items_count` |
| `_usd` | currency in USD | `revenue_usd` |

**Never embed variable or unbounded data in the event name:**
- ❌ `purchased_blue_widget` → unbounded namespace. ✅ `product_purchased` + `product_name: "blue_widget"`.
- ❌ `ios_order_completed` → blocks cross-platform analysis. ✅ `order_completed` + `platform: "ios"`.

This keeps the event count bounded and lets one query answer cross-platform *and* per-platform questions from one stream.

## 3. Event-vs-property modeling & the three property tiers

Model variants as **properties on a shared event**, not near-duplicate events. Three tiers, modeled separately because they capture different *temporal* states:

1. **Event properties** — specific to that action, captured at fire time. `cart_value_usd` on `checkout_completed`. Immutable once sent.
2. **User properties** — attributes of the *person*, used for cohorting. `subscription_plan`, `signup_cohort`. **Mutable over time** (a free user becomes paid).
3. **Super / system properties** — sent with *every* event. `app_version`, `platform`, `device_type`.

> **Hard rule:** an event property and a user property must never share a name. They represent different temporal states — `plan` as an event property (what plan was active *at this event*) vs `plan` as a user property (current plan) will silently corrupt cohort analysis. Tools like Avo enforce this as mutually exclusive.

**Snowplow entity-first heuristic:** if a piece of context is relevant to *multiple* events, it belongs in a shared **entity**, not as a property on a single event. Define reusable business entities (user, product, cart) before events, and put almost everything in entities unless it is *truly* event-specific.

**Page-view vs action events.** Page views answer "where" (navigation, funnels by URL); action events answer "what did they do." GA4's enhanced measurement auto-captures `page_view`, scrolls, and outbound clicks — but those are coarse. Conversion analysis lives in **action events** you design deliberately. Don't try to reconstruct a funnel from page views alone; instrument `checkout_started` / `payment_info_entered` / `purchase` explicitly.

## 4. The dataLayer / GTM vs direct-SDK decision

**Decouple tracking from the UI via a dataLayer.** Devs push stable, structured events; tags read explicit keys. The dataLayer is a *contract* that only breaks when the contract itself changes — unlike DOM-selector triggers (CSS class, button text), which break on any redesign or A/B test ("silent failure territory").

```html
<!-- Init ONCE, high in <head>, BEFORE the GTM/gtag snippet -->
<script>
  window.dataLayer = window.dataLayer || [];   // casing matters: 'datalayer' will NOT work
</script>
```

```js
// Push structured events afterward — never re-assign dataLayer
window.dataLayer.push({
  event: 'checkout_started',
  cart_value_usd: 49.99,
  items_count: 3,
  currency: 'USD',
});
```

**GTM vs direct SDK — a time-vs-control tradeoff** (Amplitude's own framing):

| Situation | Choose |
|---|---|
| Time crunch, limited dev resources, marketing-led team already on GTM | **GTM templates** |
| Have dev resources, want full control & type safety | **Direct SDK** |

Note: PostHog and Amplitude run their **own SDK underneath GTM** — GTM is just the deployment vehicle, not a replacement for the SDK. Amplitude via GTM means installing its JS SDK, then firing `logEvent` from a Custom HTML tag (push `logEvent` payload → trigger → tag → `SDK.logEvent`).

**Client-side vs server-side firing:**

- **Client-side** for behavioral/UI insight. It is also the *only* place **UTM params** and user-agent are reliably available on the landing hit. Cost: silent loss to ad/tracker blockers — global blocker usage runs ~**30%** of internet users (GWI Q2 2025), higher in North America (~**32–36%**) and on desktop tech-savvy traffic. Treat **15–40%** client event loss as a realistic planning range for blocker-heavy audiences, and **measure your own** loss (compare a hard-to-block server signal against the client count) rather than trusting a blanket figure.
- **Server-side** for business-critical events (revenue, billing, signups) that must not be lost to blockers or flaky networks.
- **Mature setups are deliberately hybrid:** behavioral client-side, money server-side. A **reverse proxy on your own domain** (PostHog supports this) reduces ad-blocker interception of client events.

## 5. Identity stitching & attribution

**GA4 identity spaces** (Blended reporting resolves in this priority order: user_id → Google signals → device/client_id → modeling):

| Space | Identifies | Notes |
|---|---|---|
| `client_id` / `app_instance_id` | anonymous browser / app install | first-party cookie; resets on cookie clear, and Safari ITP caps client-side cookies at **7 days** — expect inflated "new users" |
| `user_id` | your first-party authenticated ID | most accurate cross-device; **must contain no PII**; needs analytics consent to write/persist; does **not** retro-link pre-login history |
| **Google signals** | cross-device via signed-in Google accounts | the genuinely **consent-gated** cross-device space; off by default, ad-personalization consent required |

**Rules:**
- Reserve **deterministic merge** (`user_id`) for **authenticated users with analytics consent**. Use a stable, **non-PII** ID (never email/phone). Set it the moment a user logs in/signs up, consistently across devices, so pre-login anonymous activity stitches to the known user **from that point forward** — GA4 does not back-link prior anonymous sessions.
- Treat **probabilistic / fingerprinting matching** as **consent-requiring**, never a consent workaround — many DPAs treat fingerprinting like cookies.
- In CDP tools (Segment), `identify()` ties the anonymous ID to a user ID; ordering matters — call `identify()` before key events so they attribute correctly.

**UTM & basic attribution.** UTMs (`utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`) are read **client-side** from the landing URL and should be captured as first-touch user properties (persisted, not overwritten) *and* last-touch event properties on the first session. A **"direct" / "(none)" bucket far above your real type-in/bookmark traffic** is the classic symptom of broken UTM tagging, stripped referrers, redirect chains that drop the query string, or untagged email/app links — audit it as an instrumentation bug, not user behavior. First-touch vs last-touch are different questions; decide which the metric needs and store **both** touch sources if the analysis demands it.

## 6. Consent-gated firing (Consent Mode)

Consent must be established **before any measurement command fires.** Default-deny, then update on the user's choice:

```html
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){ dataLayer.push(arguments); }
  // 1) DEFAULT DENY — must run before any config/event command
  gtag('consent', 'default', {
    ad_storage: 'denied',
    analytics_storage: 'denied',
    ad_user_data: 'denied',
    ad_personalization: 'denied',
    wait_for_update: 500,   // ms to wait for the CMP before firing — prevents the pre-consent race
  });
</script>
<!-- GA4/GTM snippet loads after the default command -->
```

```js
// 2) UPDATE on the user's banner choice
gtag('consent', 'update', { analytics_storage: 'granted', ad_storage: 'granted' });
```

> **The #1 compliance failure:** a GA4 tag fires *before* the `consent default` command, reads an **undefined** consent state, and behaves **as if consent was granted** — cookies write and data hits Google before the banner even renders. Always order: init dataLayer → `consent default {denied}` → load tags → `consent update` on choice.

**Specs:** Google **Consent Mode v2** enforcement began **6 March 2024** (EEA + UK; required to keep audiences/remarketing/conversion-modeling for new EEA users). There is **no retroactive backfill** — data lost during a misconfigured-consent window is gone permanently. What recovers denied-user data is GA4 **behavioral modeling** (forward-looking ML estimate, shown only under the **Blended** reporting identity), which requires **Advanced** Consent Mode plus volume: **≥1,000 daily users** sending `analytics_storage='granted'` on **≥7 of the previous 28 days** *and* **≥1,000 daily `denied` events for ≥7 days**. Below that, denied-user events simply go unreported — small sites get no modeling. Opt-out rates vary widely by region, banner design, and vertical; **measure your own** post-banner consented share and model it into expected sample sizes so A/B tests aren't silently under-powered.

## 7. Choosing a metric model

Don't reach for a single number. **Layer** the system:

```
North Star Metric        ← leading indicator of long-term value (ONE)
  ├─ 6–10 input / driver metrics  ← what the team controls (activation, engagement, conversion)
  ├─ 3–5 guardrail metrics        ← quality, trust, retention, unit economics
  └─ lagging business outcomes    ← revenue, MRR (results, not levers)
```

**North Star must be leading.** A North Star encodes the activation moment that *predicts* retained value, not the lagging result — e.g. *"new teams with 3+ active members in week 1"* or *"weekly active editors,"* **not** raw signups or MRR (lagging). Pick the behavior your retention curve flattens after, and validate that correlation before adopting it.

**Frameworks to pick from:**

- **HEART** (Happiness, Engagement, Adoption, Retention, Task success) via **Goals → Signals → Metrics (GSM)**. Don't measure all five — **track only 2–3 dimensions per quarter** to avoid metric sprawl.
- **AARRR** (Acquisition, Activation, Retention, Referral, Revenue) — Dave McClure's pirate funnel; maps cleanly to lifecycle stages.
- **North Star + inputs** — best for a single product team aligning daily work to long-term value.
- **Task success / SUS** — for usability work: task completion rate, time-on-task, error rate, System Usability Scale (0–100 score from a 10-item survey). See [User Research & Usability Testing](../01-ux-fundamentals/user-research-and-testing.md).

**Micro vs macro conversion.** Macro = the primary goal (purchase, signup). Micro = signals on the path (add-to-cart, video-played, pricing-viewed). Instrument micro-conversions so you can diagnose *where* a macro funnel leaks — see [CRO](./conversion-rate-optimization.md).

**Leading vs lagging vs guardrail.** Leading metrics move first and predict; lagging metrics confirm outcome; guardrails catch collateral damage. **Goodhart's Law:** any metric pushed as a target gets gamed (aggressive notifications, forced funnels, longer click paths) to hit the number while wrecking the experience. Guardrails — support tickets, cancellation rate, NPS, page latency — measure *"what success should NOT look like."* Some teams keep a tighter 2–3 guardrails to stay focused.

## 8. Instrumentation QA

**Naming drift** is the dominant cause of analytics debt at scale. `sign-up`, `sign_up`, `user_sign-up`, `Song Played` vs `Play Song` get tracked as **separate** events — splitting one metric across lines and making conversion rates look artificially low. Defenses:
- A linted, versioned tracking plan; reject events not in the plan.
- Codegen (Avo Codegen) so `iOS`/`Android`/`web` all call `Avo.signupCompleted()` instead of free-typing strings.
- Periodic taxonomy audits for near-duplicate names.

**Double-fire / overcounting** — never instrument the same event **both** hard-coded **and** via GTM (the canonical cause). Also watch SDK dedup windows that drop exposure events around `identify()`.

**Sample-ratio mismatch (SRM)** — for any randomized experiment, the observed split must match the planned split.
- Detection: **chi-squared test**; **p < 0.01** = strong evidence the mismatch is not chance. Amplitude Experiment uses a **sequential** chi-squared at **alpha = 0.01**.
- Example: planned 50/50, observed **55/45** (or **600/400** of 1000) is a flag; ~500/500 expected.
- **Unit of randomization is USERS, not sessions** — a user-level mismatch is more severe.
- **Hard stop:** SRM signals unknown bias of unknown size and direction. **Do not salvage** the data. Find root cause, fix, **rerun with a fresh randomization salt** so prior-run experience doesn't carry over.
- **Debugging:** segment by browser/OS/language to isolate; check whether SRM was present **from the start** (instrumentation bug) or **emerged later**; compare **raw vs aggregated** pipeline data (raw clean + aggregate SRM ⇒ a log-join/dedup issue). A known cause: **LaunchDarkly JS SDK < 2.23.0** dedupes events within a **5-minute** window by default, dropping the second exposure when a flag is evaluated, then `identify()` adds attributes that newly match an experiment rule, then the same flag is re-evaluated — non-random exposure loss → SRM. **Fix by upgrading** (≥2.23.0 changed the default; the `allowFrequentDuplicateEvents` flag was **deprecated in 2.23.0 and removed in v3+**, so it is a legacy-only escape hatch), or by evaluating flags only **after** all context attributes are set. Add **Sentry** (or your error tracker) when the root cause is elusive — it catches one-variant JS errors that silently break tracking.

**Debug verification before signoff:**
- GA4 **DebugView** / GTM **Preview** / PostHog **live events** / Amplitude **user lookup** — confirm each planned event fires once, with correct property types and values.
- Verify casing exactly matches the plan; verify required properties are present and non-null.
- Confirm consent gating: no events before `consent default`; events resume on `update`.

## 9. Reading funnels & session replay to drive iteration

- **Funnels:** define ordered steps from the tracking plan (`product_viewed → add_to_cart → checkout_started → purchase`). Rank targets by **absolute users lost** (drop-rate × step volume), not drop-rate alone — a 10% drop on a high-traffic step usually beats a 60% drop on a tiny one. Decide upfront whether steps are **ordered** (must occur in sequence) or **any-order**, and pick a **conversion window** (e.g. same session vs 7 days) — both change the numbers. Segment by device/source/cohort — an aggregate funnel hides that mobile checkout converts half as well.
- **Session replay** (PostHog, FullStory, Microsoft Clarity, Hotjar) shows the *why* behind a drop: rage clicks, dead clicks, form-field abandonment, layout breaks. Use replay to generate hypotheses, funnels to size them, A/B tests to confirm.
- **Close the loop:** funnel drop → replay diagnosis → design hypothesis → instrument the change → A/B test → read guardrails → ship or revert. Feed findings back into [CRO](./conversion-rate-optimization.md).

## Concrete specs & numbers
- **SRM threshold:** chi-squared, **p < 0.01**. Amplitude Experiment: **sequential** chi-squared, **alpha = 0.01** (assumes traffic allocation stays constant for the run — don't reweight mid-flight). Randomize on **users**.
- **SRM example:** 50/50 planned, **55/45** (600/400 of 1000) observed = flag.
- **Consent Mode v2** enforcement: **6 March 2024** (EEA + UK). **No retroactive backfill** — misconfigured-window data is lost permanently.
- **GA4 behavioral modeling** (denied-user recovery, Blended identity only): needs **Advanced** Consent Mode + **≥1,000 daily `granted` users on ≥7 of last 28 days** *and* **≥1,000 daily `denied` events for ≥7 days**.
- **Ad/tracker-blocker usage:** ~**30%** of internet users globally (GWI Q2 2025), ~**32–36%** North America. Plan **15–40%** client event loss for blocker-heavy/desktop audiences; **measure your own**.
- **Safari ITP:** client-side cookies (incl. GA4 `client_id`) capped at **7 days** → inflated new-user counts.
- **LaunchDarkly JS SDK < 2.23.0:** 5-minute default event-dedup window → SRM around `identify()`; **fix by upgrading**. `allowFrequentDuplicateEvents` was deprecated in 2.23.0, removed in v3+.
- **Metric system sizing:** 1 North Star + **6–10** inputs + **3–5** guardrails (tighter teams: **2–3** guardrails).
- **HEART** = 5 dimensions; track **2–3 per quarter**. **AARRR** = 5 stages.
- **SUS** = 10-item survey → single 0–100 usability score.

## Tools & libraries
- **GA4** — event-based model; enhanced measurement auto-captures scrolls/outbound clicks; native Google Ads + BigQuery + Looker Studio + Firebase. Strongest for marketing-led teams already in Google's ecosystem.
- **Google Tag Manager (GTM) + dataLayer** — orchestration layer between site and tags. Gate firing CMP → dataLayer → trigger. Note: it deploys other vendors' SDKs (PostHog, Amplitude), it doesn't replace them.
- **PostHog** — developer-first; autocapture on by default (minutes to first data) and toggleable; reverse-proxy on your domain dodges ad-blockers; product analytics + replay + flags + experiments in one.
- **Amplitude** — manual-instrumentation-heavy, enterprise governance/depth; experiment SRM detection built in.
- **Mixpanel / Heap** — Heap autocaptures all interactions via one pixel and defines events **retroactively** (data exists even if undefined upfront); criticized for promoting fragile DOM-selector triggers that remove devs from the loop.
- **Segment / mParticle** — CDP event spec + routing; Segment-style event specs are a common tracking-plan template format.
- **Avo** — tracking-plan governance; supports snake_case/camelCase/Title Case/lower case, enforces mutually-exclusive event-vs-user property names; **Avo Codegen** kills cross-platform string drift.
- **Snowplow** — entity-first tracking design; define business entities before events.
- **dbt / semantic layer / metric catalog** — emerging fix for metric-definition drift; a shared single source of truth for metric math.
- **SRM / experiment tooling** — Lukas Vermeer's SRM Checker, Optimizely automatic SRM detection, Statsig, AB Tasty, LaunchDarkly, Convert, VWO.
- **Sentry** — add as error tracking when an SRM root cause is elusive (catches one-variant JS errors that silently break tracking).

## Do / Don't
- **Do** author the tracking plan during feature design; **don't** retrofit after launch.
- **Do** derive events from a named metric; **don't** "track everything."
- **Do** use one object-action, past-tense convention; **don't** mix `sign_up` / `SignUp` / `complete_signup`.
- **Do** model variants as properties; **don't** encode platform/product in the event name.
- **Do** keep event/user/super properties as distinct tiers with distinct names; **don't** reuse a name across temporal states.
- **Do** push structured dataLayer events; **don't** trigger on CSS classes or button text.
- **Do** fire `consent default {denied}` before any tag; **don't** let GA4 fire pre-consent.
- **Do** instrument money server-side; **don't** rely on client-side for revenue.
- **Do** treat SRM as a hard stop and rerun with a fresh salt; **don't** salvage SRM data.
- **Do** make instrumentation part of Definition of Done; **don't** close the ticket if events aren't live.

## Common pitfalls & how to avoid them
- **Naming drift** → splits one metric across lines, deflating conversion rates. Fix: versioned plan + codegen + audits.
- **Event-name explosion** (`purchased_blue_widget`, `ios_order_completed`) → unbounded namespace, no cross-platform analysis. Fix: variants as properties.
- **Double-fire / overcounting** → same event hard-coded *and* via GTM, or dedup windows dropping exposures around `identify()`. Fix: single source of firing; audit SDK dedup config.
- **DOM-selector triggers** → break silently on any redesign or A/B test. Adopted because a dataLayer change sits in the dev backlog while marketing needs data now, so they go quick-and-dirty against CSS/text. Fix: prioritize the dataLayer contract; treat it as infrastructure with an owner and SLA.
- **Tags firing before consent** → undefined consent state read as granted; cookies write pre-banner. Fix: strict ordering of `consent default`.
- **Sample-ratio mismatch ignored** → biased experiment readouts shipped as truth. Fix: automated SRM check at p<0.01, hard stop, rerun.
- **Bloated "direct" traffic** → broken UTM/referrer tagging mistaken for user behavior. Fix: audit landing-page UTM capture.
- **Page-view-only funnels** → can't see in-page actions; funnels look flat or wrong. Fix: instrument explicit action events.

## What real users complain about
- **Cookie/consent banners that fire trackers anyway** — users (and regulators) treat pre-consent firing as a trust breach; it surfaces as GDPR/CCPA complaints and brand damage, not just bad data.
- **"The numbers don't match"** — stakeholders lose trust when GA4, the CDP, and the database disagree (double-fire, identity gaps, ad-block loss). Reconciliation drains analyst time and stalls decisions.
- **Slow/janky pages from tag bloat** — every client-side tag adds JS; over-instrumentation hurts Core Web Vitals. See [Web Performance & Core Web Vitals](./web-performance-core-web-vitals.md).
- **Analysts:** "I can't trust this funnel" because of naming drift and missing properties — the dominant complaint behind analytics-debt refactors.

## Senior-level checklist
- [ ] Tracking plan authored **during** design, versioned, owned, with WHEN-it-fires per event.
- [ ] Every event traces to a named metric that proves a business goal.
- [ ] One object-action, past-tense convention; casing/tense/word-order locked and linted.
- [ ] Variants modeled as properties; no platform/product in event names.
- [ ] Event / user / super properties separated; no shared names across tiers.
- [ ] Semantic suffixes (`_at`, `_id`, `_count`, `_usd`) applied consistently.
- [ ] dataLayer init'd once before the tag snippet; all firing via `dataLayer.push()`; no DOM-selector triggers.
- [ ] GTM-vs-SDK and client-vs-server decisions made deliberately; money events server-side.
- [ ] `consent default {denied}` fires before any tag; `consent update` on choice; verified no pre-consent firing.
- [ ] `user_id` set on authenticated, consented users; no fingerprinting as a consent workaround.
- [ ] UTM capture verified; "direct" bucket sane.
- [ ] Metric system in place: 1 leading North Star + 6–10 inputs + 3–5 guardrails + lagging outcomes.
- [ ] No double-instrumentation; SDK dedup config checked.
- [ ] SRM check wired (chi-squared, p<0.01, randomize on users); hard-stop + fresh-salt rerun policy agreed.
- [ ] Debug verification done (DebugView/Preview/live events): every event fires once with correct types.
- [ ] Funnel + session-replay loop established for iteration.
- [ ] Instrumentation in Definition of Done — no tracking plan, no launch signoff.

## Sources
- Avo — Data taxonomy & naming conventions: https://www.avo.app/docs/data-design/define-source-of-truth/event-naming-convention
- Avo — Tracking plan as a deliverable: https://www.avo.app/blog/the-tracking-plan
- Amplitude — Data taxonomy / planning your taxonomy: https://amplitude.com/docs/data/data-planning-playbook
- Snowplow — Designing your data with entities: https://docs.snowplow.io/docs/understanding-your-pipeline/entities/
- Segment — Analytics Academy / tracking plan: https://segment.com/academy/
- Google — Consent Mode (Consent Mode v2): https://developers.google.com/tag-platform/security/guides/consent
- Google Analytics 4 — Reporting identity & user_id: https://support.google.com/analytics/answer/9213390
- Google Analytics 4 — Behavioral modeling for consent mode (thresholds): https://support.google.com/analytics/answer/11161109
- PostHog — Reverse proxy & autocapture docs: https://posthog.com/docs
- Microsoft Clarity (session replay & heatmaps): https://clarity.microsoft.com/
- Google HEART framework (research paper, Rodden et al.): https://research.google/pubs/measuring-the-user-experience-on-a-large-scale-user-centered-metrics-for-web-applications/
- Lukas Vermeer — Sample Ratio Mismatch Checker: https://lukasvermeer.nl/srm/
- Amplitude Experiment — Sample ratio mismatch (sequential chi-squared, alpha 0.01): https://amplitude.com/docs/feature-experiment/troubleshooting/sample-ratio-mismatch
- LaunchDarkly — Sample ratios & event dedup around identify(): https://launchdarkly.com/docs/home/experimentation/sample-ratios
- System Usability Scale (usability.gov): https://www.usability.gov/how-to-and-tools/methods/system-usability-scale.html

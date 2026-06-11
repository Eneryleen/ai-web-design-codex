# Lifecycle, Email & Notification Design

> The design craft behind the highest-converting channel marketers own. Covers responsive HTML email reality (table layouts, ~600px width, inline CSS, dark mode, the two Outlooks, image-off and accessible fallbacks), the core lifecycle programs and how to design each (welcome, abandoned-cart/browse, trial-expiry, win-back, re-engagement), the transactional-vs-marketing distinction, subject-line/preheader/from-name as design surfaces, the deliverability fundamentals a designer must respect, and cross-channel orchestration with push/in-app/SMS. The senior outcome: ship lifecycle emails and notifications that render correctly everywhere, land in the inbox, and convert on intent — not luck.

**Apply when:** designing any email, push, in-app, or SMS that a customer receives outside the website itself · **Related:** [Conversion Rate Optimization](./conversion-rate-optimization.md) · [Copywriting & UX Writing](./copywriting-and-ux-writing.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Analytics Instrumentation](./analytics-instrumentation-and-measuring-impact.md)

## TL;DR — rules at a glance
- **Email's responsive model is INVERTED from web.** Design a fixed ~600px desktop baseline first, then media queries OVERRIDE for mobile (graceful degradation). The email must stay readable if every media query is stripped.
- **Tables are mandatory for layout.** Classic Outlook renders via Microsoft Word's engine — no flexbox, grid, float, `position`, `max-width`, or `border-radius`. Build for the worst client, enhance for the rest.
- **There are TWO Outlooks now.** Classic desktop (Word engine, needs VML + MSO conditional comments) and "new Outlook" for Windows (WebView2 / Edge-Chromium, like Outlook.com — modern CSS but IGNORES MSO comments and runs a post-load style-rewrite pass). Design for both at once.
- **Inline ALL CSS before send; media queries can't be inlined** and must live in `<head>`. This dual reality is the core tension of email coding.
- **Lifecycle automations crush broadcasts.** Triggered flows (cart, welcome, browse) earn far higher revenue-per-recipient than batch campaigns — Klaviyo's benchmarks put abandoned-cart at ~$3.65 RPR vs ~$0.11 for a standard campaign. They convert because they're triggered by intent, not by calendar.
- **Deliverability is governed by ENGAGEMENT, not technical hacks.** Perfect SPF/DKIM/DMARC still rots in spam if no one opens, clicks, or — worse — people report spam.
- **Transactional ≠ marketing, legally and operationally.** Transactional (receipt, reset, shipping) is exempt from the unsubscribe requirement and carries a higher accessibility bar; marketing needs one-click unsubscribe (RFC 8058), a postal address, and opt-out honored within the legal window (CAN-SPAM: 10 business days; Gmail/Yahoo bulk rules: 2 days).
- **From-name + subject + preheader are ONE three-line headline,** not three fields. Sender recognition and the subject line are the two primary open drivers — keep the from-name stable so the brand becomes a learned trust signal.
- **Cap total brand touches at the CUSTOMER level across all channels,** not per channel — three "reasonable" channel limits stack into one annoying brand. Match channel to urgency; honor quiet hours.
- **Dark mode: never pure `#FFFFFF` or `#000000`** — they trigger the most aggressive forced inversion. Use off-white/off-black (`#FAFAFA` / `#222222`, or near-pure `#FFFFFE` / `#000001` when you need them to read as white/black) and declare `color-scheme`. ~A third of opens are in dark mode (Litmus measurement; understated, since only WebKit clients report it).
- **Keep raw HTML under ~102KB** or Gmail clips it ("Message clipped"), hiding your CTA and footer. Build to ~80KB — your ESP injects tracking/link-rewriting code after you hand off, and the clip point isn't exact.
- **Accessibility in email is mostly fundamentals:** real text not images, alt everywhere, 4.5:1 contrast, `role="presentation"` on layout tables, logical reading order, `lang` attribute. The same moves also defeat image-blocking and dark-mode inversion, so they lift engagement for everyone.

---

## 1. Why email's responsive model is inverted

On the web you go **mobile-first**: a base mobile layout, progressively enhanced for wider screens. **Email is the opposite.** You build a **fixed ~600px desktop baseline** and then use media queries to *override* it down for mobile (graceful degradation).

Two reasons:

1. **Media queries are unreliable.** Many clients (Gmail app on some configs, classic Outlook, several corporate clients) strip or ignore `@media` rules. Your email must look correct with **zero** media query support. So the base — the inlined, non-media styles — must be the *good* desktop layout, not a stripped mobile one.
2. **`<style>` blocks get removed.** Media queries can only live in `<head>` `<style>`, which many clients delete. Anything you put there is an enhancement, never a foundation.

**The mental model:** *Code the worst case (Outlook, no media queries, images off, dark mode forced) as the baseline. Layer enhancements on top for clients that support them.*

```html
<!-- BASE: inlined, works with zero media-query support -->
<table role="presentation" width="100%" cellpadding="0" cellspacing="0" border="0">
  <tr>
    <td align="center">
      <table role="presentation" width="600" cellpadding="0" cellspacing="0" border="0"
             style="width:100%;max-width:600px;">
        <tr><td style="padding:24px;font-family:Arial,sans-serif;font-size:16px;
                       line-height:1.5;color:#333333;">…content…</td></tr>
      </table>
    </td>
  </tr>
</table>
```

```html
<!-- ENHANCEMENT: in <head>, overrides for narrow screens only -->
<style>
  @media only screen and (max-width:600px) {
    .col       { display:block !important; width:100% !important; }
    .mobile-pad{ padding:16px !important; }
    .h1        { font-size:24px !important; }
    body, td   { font-size:16px !important; } /* bump body text on mobile */
  }
</style>
```

## 2. The skeleton: tables, 600px, fluid centering

**Container width:** build at `max-width:600px`. 600px is the broadly-supported threshold that fits the desktop preview pane and avoids horizontal scroll. Use **fluid tables** (`width:100%` + `max-width:600px`) so it shrinks gracefully on narrow clients.

**Reliable cross-client centering** is a nested `<center>` (or `align="center"` on the outer `<td>`) wrapping a fixed-width inner table — not `margin:0 auto` (unsupported in Outlook).

**Ghost tables for Outlook.** Classic Outlook ignores `max-width`, so it won't cap a `width:100%` table at 600px — it sprawls full-width. Wrap the fluid table in an MSO-only fixed table:

```html
<!--[if mso]>
<table role="presentation" align="center" width="600" cellpadding="0" cellspacing="0" border="0"><tr><td>
<![endif]-->
<table role="presentation" align="center" width="100%" cellpadding="0" cellspacing="0" border="0"
       style="max-width:600px;margin:0 auto;">
  <!-- real content -->
</table>
<!--[if mso]></td></tr></table><![endif]-->
```

**Required `<head>` boilerplate:**

```html
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="x-apple-disable-message-reformatting"> <!-- stop Apple auto-resize -->
<meta name="color-scheme" content="light dark">
<meta name="supported-color-schemes" content="light dark">
<!--[if gte mso 9]>
<xml><o:OfficeDocumentSettings>
  <o:PixelsPerInch>96</o:PixelsPerInch> <!-- fixes blurry images / pixel distortion on high-DPI Windows -->
</o:OfficeDocumentSettings></xml>
<![endif]-->
```

## 3. The two Outlooks (and why it's the hardest part)

| | **Classic Outlook (desktop, Win)** | **"New Outlook" / Outlook.com** |
|---|---|---|
| Engine | Microsoft **Word** (mso) | **WebView2** (Edge / Chromium) |
| Modern CSS | No flexbox/grid/float/`position`/`max-width`/`border-radius` | Mostly yes |
| Background images | CSS ignored → needs **VML** | CSS works |
| MSO conditional comments | **Honors** `[if mso]` | **IGNORES** them |
| Width capping | Ignores `max-width` → needs ghost tables | Respects `max-width` |

You must satisfy **both simultaneously**: VML + `[if mso]` blocks serve classic Outlook; the new Outlook silently skips those and reads your normal CSS. Code defensively so neither path breaks the other.

**Background images in classic Outlook** need VML, gated inside MSO comments so other clients ignore it:

```html
<td background="https://cdn.example.com/hero.jpg" style="background-size:cover;">
  <!--[if gte mso 9]>
  <v:rect xmlns:v="urn:schemas-microsoft-com:vml" fill="true" stroke="false"
          style="width:600px;height:300px;">
    <v:fill type="tile" src="https://cdn.example.com/hero.jpg" color="#222222" />
    <v:textbox inset="0,0,0,0">
  <![endif]-->
  <div><!-- overlay text here --></div>
  <!--[if gte mso 9]></v:textbox></v:rect><![endif]-->
</td>
```

## 4. Images: the rules that prevent breakage

Set ALL of these on every `<img>`:

- `display:block` — kills the 2–3px gap classic Outlook adds under inline images.
- `border:0` — removes the blue link border old Outlook draws around linked images.
- **Descriptive `alt`** — corporate clients block images by default; alt is your fallback copy. Never `alt="image"`; write what it says/shows (`alt="Get 15% off — shop now"`).
- **Absolute CDN URLs** — relative paths don't resolve in an inbox.
- `width="100%"` **as an HTML attribute** (not just CSS). Samsung Mail reads the HTML `width` attribute to set viewport width — `width="600"` forces a 600px viewport and breaks responsiveness; `width="100%"` plus a CSS `max-width` keeps it fluid.

```html
<img src="https://cdn.example.com/banner.png" alt="Spring sale: up to 40% off boots"
     width="100%" style="display:block;border:0;width:100%;max-width:600px;height:auto;">
```

**Bulletproof, image-off-safe CTA button** (HTML+CSS, not an image — so it survives image blocking and renders selectable text):

```html
<!--[if mso]>
<v:roundrect xmlns:v="urn:schemas-microsoft-com:vml" xmlns:w="urn:schemas-microsoft-com:office:word"
  href="https://example.com/cart" style="height:48px;v-text-anchor:middle;width:220px;"
  arcsize="12%" stroke="false" fillcolor="#1a73e8">
  <w:anchorlock/><center style="color:#ffffff;font-family:Arial,sans-serif;font-size:16px;font-weight:bold;">
    Complete your order
  </center>
</v:roundrect>
<![endif]-->
<!--[if !mso]><!-->
<a href="https://example.com/cart"
   style="background:#1a73e8;color:#ffffff;text-decoration:none;display:inline-block;
          padding:14px 28px;border-radius:6px;font-family:Arial,sans-serif;
          font-size:16px;font-weight:bold;">Complete your order</a>
<!--<![endif]-->
```

## 5. Dark mode

Roughly a third of opens happen in dark mode by Litmus's measurement — and that's understated, because only WebKit-based clients report the dark-mode flag (Apple), while heavy-dark-mode Android opens go uncounted. Three failure modes: invisible logos (transparent PNG with dark artwork on a now-dark background), inverted/unreadable text, and washed-out brand colors.

- **Never use pure `#FFFFFF` or `#000000`.** They trigger the most aggressive *forced* color inversion (Apple Mail flips pure black/white regardless of your intent). Use off-white `#FAFAFA` / dark grey `#222222`; clients leave these mostly alone. Where you genuinely need something to read as white or black, nudge one step off-pure — `#FFFFFE` / `#000001` — to dodge the inversion while looking identical.
- Declare `<meta name="color-scheme" content="light dark">` and `<meta name="supported-color-schemes" content="light dark">` — this alone suppresses unwanted full inversion in new Outlook and several clients.
- Target dark mode with `@media (prefers-color-scheme: dark)` for Apple Mail (iOS 13+) / iOS Mail / Outlook 2019+ / Samsung Mail. **Gmail and Outlook mobile do NOT honor it** and apply their own (often full) inversion — so design to survive automatic inversion, treating the media query as enhancement only.
- Gmail reliably converts a solid `#FFFFFF` background to dark grey, which usually looks acceptable — so set an explicit white background rather than transparent.
- Give transparent logos a solid or padded background, or swap to a dark-mode logo via the media query. Add a thin off-color stroke around dark logos so they survive on dark backgrounds.

```html
<style>
  @media (prefers-color-scheme: dark) {
    .body-bg   { background-color:#222222 !important; }
    .text      { color:#fafafa !important; }
    .logo-light{ display:none !important; }
    .logo-dark { display:block !important; }
  }
</style>
```

## 6. Client-specific gotchas (the ones that silently destroy emails)

- **Gmail clipping:** raw HTML over **~102KB** → Gmail truncates with a "Message clipped — View entire message" link, hiding your CTA, footer, and unsubscribe. Minify; remove dead CSS; compress; avoid repeated inline styles.
- **Gmail `<style>` limits:** the `<style>` block has a size cap — long the documented **8,192 bytes**, reportedly raised to ~16KB, and Gmail's own injected prefixes count against it, so **build to stay under ~8KB**. Overflow drops the offending block *and every block after it* (so order matters: critical CSS first, and split into multiple blocks). Separately, a **single CSS syntax error** OR any **`background-image:url()` inside `<style>`** nukes that block — keep `background-image` and comments out of `<style>`.
- **Yahoo:** converts `height` to `min-height` (breaks fixed-height patterns — use padding/spacer rows instead). Yahoo's parser is whitespace-sensitive around `!important`, and the behavior is **property-dependent and has flipped over time** — historically it dropped `!important` on `display` when there was a space before it (so `display:none!important` survived), but the current quirk hits `background`/`background-image`, where a space is the safer form. Net: don't rely on it — test both, and prefer not to depend on `!important` to win on Yahoo. CSS placed *below* an HTML comment inside a `<style>` block gets neutralized — keep comments out of `<style>`.
- **Outlook:** ignores `max-width`, `border-radius`, background images, and adds line-height padding around tables — set `mso-line-height-rule:exactly` and use `<table>` spacers, never margins, for vertical rhythm.
- **Samsung Mail:** uses the HTML `width` attribute (see §4), not CSS, for viewport sizing.

## 7. Subject line, preheader, from-name — the three-line headline

These three fields render together in the inbox list as a **2–3 line unit**. Design them as one headline, not three independent strings.

**From-name** is one of the two primary open drivers (alongside the subject line — widely-cited industry figures put ~a third to ~half of open decisions on sender recognition, similar magnitude for the subject line). Use a recognizable, consistent sender (`Anna from Acme` or `Acme Support`), never `no-reply` for marketing, and keep it stable so the brand becomes a learned trust signal.

**Subject line:**
- Aim **~40–50 characters / 5–7 words**. Only ~**33 characters** are reliably visible across mobile; Gmail/Yahoo truncate on mobile around **33–43 chars** depending on device/orientation.
- **Front-load** the most important words into the first **~30 chars** — assume the tail gets cut.
- **Personalize where it's earned** (name, the actual viewed/abandoned product, location). Relevance lifts opens and click-to-open; an empty `Hi {first_name}` token does not — personalize the *content*, not just a merge field.
- Avoid spam-trigger overload (ALL CAPS, `FREE!!!`, excessive emoji/punctuation) — a deceptive or shouty subject spikes complaints, and complaint rate is what sinks deliverability (§10).

**Preheader** (preview text after the subject):
- **ALWAYS set it explicitly.** If you don't, clients pull whatever comes first — "View in browser," the unsubscribe link, or image alt text — which looks broken.
- Write **~50–100 chars** (**75+** so body text doesn't leak into the preview); front-load the first **35–40 chars**.
- It must **complement**, not duplicate, the subject line — add the second beat of the pitch.
- Display varies: Gmail desktop ~100 chars, Apple Mail up to ~140, mobile clients ~40–70, Outlook a shorter version-dependent snippet.

```html
<!-- Hidden preheader: shows in inbox preview, invisible in the body -->
<div style="display:none;max-height:0;overflow:hidden;mso-hide:all;
            font-size:1px;line-height:1px;color:#ffffff;">
  Your boots are still in the cart — and your size is selling fast.
</div>
<!-- Trailing whitespace hack: stops the next body text leaking into preview -->
<div style="display:none;max-height:0;overflow:hidden;">&#8204;&zwnj;&nbsp;&#8204;&zwnj;&nbsp;&#8204;&zwnj;&nbsp;</div>
```

## 8. The core lifecycle programs

Automations are triggered by **intent**, which is why they convert. Across ecommerce, the cart / welcome / browse trio carries the bulk of automation revenue, and abandoned-cart is consistently the single highest-earning flow (Klaviyo benchmarks: ~$3.65 revenue-per-recipient and the top conversion rate of any flow, vs ~$0.11 RPR for a batch campaign). Design each as a *series*, not a single send.

### 8.1 Welcome / onboarding series
- **2–4 emails over 7–10 days.** Email 1 sends **immediately**.
- **Email 1 is short**, delivers the promised incentive (discount code, the lead magnet, "your account is ready"), and makes the **brand stick** so the subscriber connects the signup to this inbox — *not* a long origin story.
- Subsequent emails: set expectations (what/how often you'll send), surface best content/products, drive the first key action (first purchase, first activation step).
- The welcome flow is the second-highest-earning automation after cart (~$2.65 RPR in Klaviyo benchmarks) and the highest-intent moment you'll get — a fresh opt-in actively expecting you. Don't waste it on a brand essay.

### 8.2 Abandoned cart
- **Speed matters** — emails within hours outperform delayed ones. A common cadence: #1 at ~1h, #2 at ~24h, #3 at ~48–72h.
- **ONE clear CTA:** complete the purchase. Show the exact items (image, name, price) and deep-link back to the populated cart.
- **Hold the discount** until email #2 or #3 to protect margin. For high-consideration products, **lead with trust/reassurance** (reviews, returns policy, shipping, security) *before* discounting.
- Cart abandonment averages **~70%** of carts (Baymard meta-analysis of 50 studies), and ~43% of that is "just browsing," not recoverable — so target the high-intent minority that genuinely stalled at checkout. Cart flows have the highest conversion rate of any automation (Klaviyo: ~3–3.5%).

### 8.3 Browse abandonment
- Triggered by viewing a product/category without adding to cart — lower intent than cart (Klaviyo benchmarks: ~$1.07 RPR and ~1% conversion, well below cart but still far above batch campaigns), so soften the ask. "Still thinking it over?" + the viewed items + social proof, no aggressive discount.

### 8.4 Trial-expiry (SaaS)
- Map the series to days remaining: value recap (what they've done so far), feature they *haven't* tried, "trial ends in 3 days," "ends tomorrow," then a win-back if it lapses.
- Lead with **the value already created in their account**, not features. Make upgrading one click.

### 8.5 Win-back / re-engagement
- For lapsed customers/subscribers (no open/click/purchase in N months). Sequence: "we miss you" + incentive → highlight what's new → a **final "is this goodbye?"** that asks them to confirm interest or be removed.
- Offer a concrete reason to return (incentive, what's changed), not just a reminder — a lapsed contact already ignored you once.
- Re-engagement doubles as **list hygiene**: non-responders to the final email get suppressed (protects engagement-based deliverability — see §10). This is the single most important reason to run the program — every dead address you keep mailing drags your whole domain's reputation down.

## 9. Transactional vs marketing — a hard legal & operational line

| | **Transactional** | **Marketing / promotional** |
|---|---|---|
| Examples | Password reset, receipt, order/shipping, security alert, account notice | Newsletter, promo, product announcement, cart/browse, win-back |
| Unsubscribe | **Exempt** | **Required**; one-click for bulk senders (RFC 8058). Honor opt-out within the legal max — CAN-SPAM **10 business days**; Gmail/Yahoo bulk rules require processing within **2 days** |
| Physical postal address | Not required | **Required** (CAN-SPAM) |
| Accessibility imperative | **Higher** — these are critical and time-sensitive | High |
| Consent | Implied by the transaction | Explicit opt-in (and double opt-in recommended) |
| Sending domain | Ideally separate subdomain | Separate subdomain |

**Designer consequences:** keep transactional emails ruthlessly clear, fast, and text-first (the reset code, the receipt total, the tracking link must be *real text*, never trapped in an image). **Never smuggle marketing into a transactional email** — under CAN-SPAM, if the "primary purpose" of a message is commercial, the whole message is marketing and loses the exemption, so a promo block in a receipt forfeits it. Send transactional and marketing from **separate subdomains/streams** so a marketing reputation dip never delays password resets — and confirm your ESP actually supports domain separation before relying on it (capabilities differ by platform and plan; verify, don't assume).

## 10. Deliverability fundamentals a designer must respect

**Engagement is the master signal.** ISPs (Gmail, Yahoo, Microsoft) run their own algorithms on opens, clicks, forwards, replies, and especially **"report spam" rate**. Authentication is table stakes; engagement decides inbox vs spam.

**Authentication (bulk senders, 5,000+/day to Gmail — a permanent classification once crossed):**
- **Both SPF and DKIM** (not either/or). Yahoo requires DKIM with a **minimum 1024-bit key**.
- **DMARC at minimum `p=none`** with From-domain alignment to SPF or DKIM.
- Valid **PTR (reverse DNS)** and **TLS** on connections.

**Complaint-rate ceiling (Google Postmaster Tools):** keep complaints **below 0.10%** (Google's target); **0.30% is the hard ceiling** — exceed it and you lose mitigation support until you hold under 0.3% for seven consecutive days, with mail junked/rejected at scale. **One-click unsubscribe (RFC 8058):** make opting out trivial and one-step — a buried or broken unsubscribe pushes people to hit "report spam" instead, which is far worse for your domain than losing the subscriber.

**List hygiene (a deliverability act, not housekeeping):**
- Suppress **hard bounces immediately**.
- Remove unengaged contacts — common cutoff ~**6 months** for high-frequency senders, **12–18 months** for quarterly senders.
- **Never use purchased lists** (spam-trap risk torches your domain). Prefer **double opt-in**.
- Clean **monthly to quarterly**.

**What a designer controls:** prominent, working unsubscribe; visible postal address; honest from-name/subject (no bait-and-switch — a deceptive subject spikes complaints); a good text-to-image ratio (all-image emails read as spam and fail image-off); and a real plain-text alternative.

**Enforcement timeline (context):** Gmail/Yahoo bulk rules (SPF+DKIM+DMARC `p=none`, one-click unsubscribe) took effect **Feb 1, 2024**; Microsoft consumer (outlook/hotmail/live) bulk rules from **May 5, 2025** (escalated from junking to outright SMTP rejection). Google ramped to **temporary + permanent rejections from Nov 2025**, and retired legacy Postmaster Tools for **Postmaster Tools v2** (Oct 2025; old High/Med/Low reputation scores gone). Thresholds: Google = 0.1% target / 0.3% hard cap; Microsoft and Yahoo = 0.3% cap.

## 11. Cross-channel orchestration (email · push · in-app · SMS)

**Cap at the customer level, not the channel.** Three "reasonable" per-channel limits stack into one annoying brand. Apply **both** a global cross-channel cap **and** tighter per-channel caps.

- **Frequency baseline: 2–3 messages per user per day** (global and per-channel).
- **Tailor caps to lifecycle stage:** new users tolerate more onboarding; churn-risk users get fewer promos.
- **Channel personality:** SMS and push **interrupt** (near-immediate, lock-screen — reserve for time-sensitive value); email **waits** in the inbox (room for detail, no interruption); **in-app is contextual** (only seen when the user is already in the product — best for onboarding nudges and feature discovery, never for re-engaging an absent user).

**Channel-by-urgency mapping:**

| Message | Channel(s) |
|---|---|
| High-urgency transactional (fraud/security) | **SMS + push simultaneously** |
| Medium (order shipped) | **Push first**, SMS if unopened in ~2h |
| Low marketing (new feature) | **Push then email** |
| Onboarding guidance | **In-app** for contextual steps, **push** for re-engagement, **email** for detail, **SMS** only for critical setup |

**Quiet hours & SMS law (TCPA — real money):** statutory damages run **$500–$1,500 per message** ($1,500 for willful), and there's **no cap** — multiplied across a list, one mistimed blast is existential.
- **Federal TCPA quiet hours = 8 AM–9 PM** local time at the *recipient's* location (47 C.F.R. § 64.1200(c)(1)).
- **Several state "mini-TCPA" laws are stricter — 8 AM–8 PM** (Florida's FTSA, plus NJ, MD and others), some with per-campaign message caps. Since you rarely know which state every number sits in, the safe operational default is the **tighter 8 AM–8 PM** window.
- A wave of **quiet-hours class-action lawsuits** hit through late 2024 into 2025 (one Florida firm alone filed 100+ near-identical complaints), arguing a mistimed text violates the rule *even when the recipient consented*. The law is unsettled (an FCC petition is pending), so treat quiet hours as hard limits, not guidance.
- **Default to the recipient's local 8 AM–8 PM; if you can't determine their time zone, pick a conservative window (e.g. midday ET).** Always include **SMS opt-out** (`Reply STOP`) and send only with prior express written consent. Queue non-urgent push/email to respect the same windows.

## 12. Accessibility in email

Email strips JS and limits CSS, so accessibility is **mostly fundamentals** — and it pays back broadly: the same things that help assistive tech (real text, alt, strong contrast, logical order) also survive image-off, dark-mode inversion, translation, and tiny-screen rendering, which lifts engagement and deliverability for *everyone*. It's underdone in practice, so it's an easy competitive edge.

- **Real text, not images of text.** Survives image-off, screen readers, translation, and dark mode.
- **`role="presentation"` on EVERY layout table** (so screen readers don't announce rows/columns); use a real semantic `<table>` only for genuine data tables.
- **Semantic headings** (`<h1>`–`<h3>`), in logical order; source order = reading order (don't reorder columns visually only via CSS).
- **`alt` on every image**; empty `alt=""` on purely decorative images.
- **Contrast 4.5:1** normal text, **3:1** large (WCAG "large" = ≥24px, or ≥18.66px bold). Hierarchy on white: headings `#000/#111` (~20:1), body `#333` (12.6:1), secondary `#666` (5.7:1); **absolute lightest text on white is `#767676`** (4.54:1, just passes AA) — never go lighter.
- **Type:** ≥**14px desktop / 16px mobile** (bump via media query), **line-height ≥ 1.5×**, sentences ~20 words max.
- **Never use color alone** to convey meaning (error/success needs an icon or word too).
- **Set `lang`** on `<html>`. Use **`margin` not `padding`** on semantic elements (padding support is inconsistent across clients).
- **Photosensitivity (WCAG 2.3.1, Level A):** nothing may flash **more than 3 times in any 1-second period** (the broadcast/PEAT danger band is ~3–55 Hz). Keep animated GIFs slow, stop them within ~5 seconds, and ensure the **first frame carries the message** — many clients (incl. classic Outlook) show only frame 1.
- **EU Accessibility Act** took effect **June 28, 2025** with real fines — accessible email is now a compliance line item for EU audiences.

See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) and [Color Theory & Systems](../00-foundations/color-theory-and-systems.md).

## Concrete specs & numbers

- **Email body width:** ~600px desktop baseline (`max-width:600px`).
- **Gmail clip limit:** ~**102KB** raw HTML (build to ~80KB; ESP injects code post-handoff). Gmail `<style>` cap ~**8KB** (documented 8,192 bytes, reportedly ~16KB now; injected prefixes count) — overflow drops that block onward; no `background-image:url()` or comments inside it.
- **WCAG contrast:** 4.5:1 normal, 3:1 large (≥18px). `#767676` = 4.54:1 = lightest usable grey on white.
- **Type:** 14px min desktop, 16px min mobile; line-height ≥ 1.5×; sentences ~20 words; aim for plain language (≈ Flesch Reading Ease 60–70).
- **Subject line:** ~40–50 chars / 5–7 words ideal; ~33 chars reliably visible on mobile; truncation ~33–43 chars by device. Front-load the first ~30 chars.
- **Preheader:** write ~50–100 chars (75+ to be safe); front-load first 35–40. Display: Gmail desktop ~100, Apple Mail ~140, mobile ~40–70, Outlook a short snippet.
- **Bulk sender threshold (Google):** 5,000+ messages/day to Gmail (permanent once crossed). Microsoft consumer + Yahoo use the same 5,000/day bar.
- **Spam complaint thresholds:** Google = below **0.1%** target, **0.3%** hard cap; Microsoft + Yahoo = **0.3%** cap.
- **Unsubscribe SLA:** CAN-SPAM legal max **10 business days**; Gmail/Yahoo bulk rules require processing within **2 days**; opt-out mechanism must keep working ≥30 days. Best practice: real-time.
- **DKIM key:** Yahoo minimum 1024-bit.
- **Lifecycle revenue (Klaviyo benchmarks):** abandoned-cart ~$3.65 RPR (highest of any flow, top conversion rate), welcome ~$2.65, browse ~$1.07, post-purchase ~$0.41 — vs ~$0.11 for a batch campaign.
- **Cart abandonment:** ~**70%** average (Baymard, 50-study meta-analysis); ~43% of abandons are "just browsing." Cart-flow conversion ~3–3.5% (Klaviyo).
- **Personalization & win-back:** personalize on real signals (name, viewed/abandoned product, location), not empty merge tokens; win-back must offer a concrete reason to return.
- **Open drivers:** sender recognition (from-name) and subject line are the two primary factors — keep the from-name stable.
- **Welcome series:** 2–4 emails over 7–10 days; email 1 immediate.
- **Frequency cap baseline:** 2–3 messages/user/day (apply BOTH a customer-level cross-channel cap and tighter per-channel caps).
- **Photosensitivity (WCAG 2.3.1):** no more than 3 flashes in any 1 second; stop GIFs within ~5 seconds; first frame must carry the message.
- **TCPA:** $500–$1,500/message (no cap). Federal quiet hours **8 AM–9 PM** local; stricter state mini-TCPA laws (FL/NJ/MD) **8 AM–8 PM** → default to 8 AM–8 PM; always include `Reply STOP` + prior express written consent.
- **EU Accessibility Act:** in force **June 28, 2025** (fines) — accessible email is a compliance item for EU audiences.

## Tools & libraries

- **Litmus** — cross-client previews/testing (Gmail web/mobile, Outlook 2016–2024, Apple Mail, Yahoo, Samsung) + accessibility guidance.
- **Email on Acid** — cross-client rendering + accessibility/code testing.
- **WebAIM Color Contrast Checker** — verify 4.5:1 / 3:1 ratios.
- **Google Postmaster Tools** — spam-complaint rate (keep <0.1%) and domain/IP reputation.
- **Yahoo Complaint Feedback Loop (CFL) / Sender Hub** — receive spam complaints, manage Yahoo reputation.
- **dmarcian / PowerDMARC / Suped** — DMARC setup, monitoring, and parsing `rua` aggregate reports (who sends as your domain).
- **Klaviyo** — ecommerce lifecycle flows (welcome, cart, win-back) + AI send-time/subject optimization.
- **Customer.io, Braze, Iterable, SFMC, Optimove** — cross-channel orchestration. Domain/stream separation for transactional vs marketing varies by platform and plan — confirm it for your account rather than assuming.
- **Mailgun, SocketLabs, MailerSend** — transactional/deliverability ESPs (RFC 8058 one-click headers).
- **VML** — required for background images in classic Outlook.
- **Email Comb / Email on Acid inliner** — CSS inlining + dead-code removal before send.
- **#emailgeeks Slack** (`email-code` channel) — practitioner debugging for HTML email rendering.

## Do / Don't

**Do**
- Build a 600px desktop baseline that survives with zero media-query support, then enhance.
- Use `role="presentation"` tables, real text, absolute CDN image URLs, and full `<img>` rules (`display:block;border:0`, `width="100%"`, descriptive alt).
- Set the preheader explicitly on every send.
- Hold cart discounts to email #2/#3; lead with one CTA.
- Cap touches at the customer level across all channels; respect local 8 AM–8 PM quiet hours.
- Send transactional and marketing from separate subdomains.
- Test in Litmus/Email on Acid across Outlook (classic + new), Gmail, Apple Mail, Yahoo, Samsung — light AND dark.

**Don't**
- Don't go mobile-first or rely on media queries / `<style>` as a foundation.
- Don't use flexbox, grid, float, `position`, or `max-width` for layout (Outlook).
- Don't use pure `#FFFFFF`/`#000000` (forced-inversion bombs).
- Don't put `background-image:url()` or comments inside Gmail's `<style>` block.
- Don't ship all-image emails or images of text.
- Don't smuggle promos into transactional email.
- Don't use `no-reply` from-names for marketing, or buy lists.
- Don't send SMS outside the recipient's local quiet-hours window (default 8 AM–8 PM), without `Reply STOP`, or without prior express written consent.

## Common pitfalls & how to avoid them

- **"Looks perfect in Gmail, broken in Outlook."** You used modern CSS as the baseline. Fix: tables + ghost tables + VML; treat modern CSS as enhancement only.
- **Email clipped, CTA gone.** Raw HTML > ~102KB. Fix: minify, dedupe inline styles, compress, host images on CDN.
- **Media queries vanish in Gmail.** A syntax error, a `background-image` in `<style>`, or blowing the ~8KB cap dropped the offending `<style>` block (and everything after it). Fix: validate CSS; keep background images and comments out of `<style>`; split blocks with critical CSS first.
- **Logo vanishes / text inverts in dark mode.** Pure white/black + transparent logo. Fix: `#FAFAFA`/`#222222`, dark-mode logo swap, `color-scheme` meta.
- **Fixed-height block breaks in Yahoo.** Yahoo turns `height` into `min-height`. Fix: use padding/spacer rows, not fixed heights; don't depend on `!important` to win in Yahoo (its whitespace handling is property-dependent — test it).
- **Preview shows "View in browser."** No explicit preheader. Fix: add a hidden preheader + whitespace hack.
- **Great content, lands in spam.** Engagement decay or rising complaints. Fix: prune unengaged contacts, one-click unsubscribe, honest subject lines, separate sending streams.
- **Customer rage-unsubscribes.** Channel-level caps stacked. Fix: global customer-level frequency cap + suppression of recently-messaged users.
- **TCPA letter.** SMS sent in quiet hours / no STOP. Fix: enforce the recipient's local window (default 8 AM–8 PM to cover stricter state laws), always include `Reply STOP`, and only send with prior express written consent.

## What real users complain about

- **"Why does this email look broken?"** — collapsed Outlook tables, blue borders around images, gaps under banners.
- **"I can't read it"** — images blocked with no alt fallback, light-grey text, tiny font, white-on-white in dark mode.
- **"Message clipped"** — Gmail hid the CTA and unsubscribe.
- **"This is too many emails/texts"** — uncapped cross-channel frequency; the #1 reason for unsubscribe and spam-marks.
- **"It's the middle of the night"** — push/SMS ignoring quiet hours.
- **"I can't unsubscribe"** — buried, broken, or multi-step opt-out → people report spam instead, which damages deliverability for everyone on the list.
- **"This 'receipt' is just an ad"** — marketing smuggled into transactional mail erodes trust.

See [What Real Users Complain About](../08-pitfalls-and-antipatterns/real-user-complaints.md) and [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md).

## Senior-level checklist

- [ ] 600px fluid table layout; centered via nested table; ghost tables for Outlook width.
- [ ] All CSS inlined; media queries in `<head>` are enhancement-only; email readable with zero media-query support.
- [ ] `<head>` boilerplate: charset, viewport, `x-apple-disable-message-reformatting`, `color-scheme` + `supported-color-schemes`, MSO `PixelsPerInch=96`.
- [ ] Every `<img>`: `display:block;border:0`, `width="100%"` HTML attr, descriptive alt, absolute CDN URL.
- [ ] CTA is bulletproof (VML + `<a>`), real text, image-off safe.
- [ ] Dark mode: no pure white/black, logo handled, tested in Apple Mail + Gmail.
- [ ] Raw HTML < ~102KB; no `background-image`/comments in Gmail `<style>`; Yahoo `!important` spacing correct.
- [ ] From-name recognizable; subject ~40–50 chars front-loaded; preheader explicit, 75+ chars, complementary.
- [ ] Lifecycle series designed (not single sends): welcome (immediate), cart (speed + held discount), trial-expiry (value-led), win-back (incentive + final ask).
- [ ] Transactional and marketing separated by class and subdomain; no promos in transactional.
- [ ] Deliverability: SPF + DKIM + DMARC, PTR, TLS; one-click unsubscribe; postal address; complaint rate < 0.1%.
- [ ] List hygiene: hard bounces suppressed, unengaged pruned, no purchased lists, double opt-in.
- [ ] Accessibility: `role="presentation"` tables, semantic headings, contrast ≥ 4.5:1, ≥16px mobile, `lang`, no color-only meaning, safe animation.
- [ ] Cross-channel: customer-level frequency cap (2–3/day), urgency→channel mapping, local 8 AM–8 PM quiet hours, SMS opt-out + consent.
- [ ] Rendered in Litmus/Email on Acid across classic Outlook, new Outlook, Gmail, Apple Mail, Yahoo, Samsung — light and dark.

## Sources

- RFC 8058 — One-Click Unsubscribe: https://www.rfc-editor.org/rfc/rfc8058
- Google — Email sender guidelines (bulk rules, 5,000/day, 0.1%/0.3% thresholds): https://support.google.com/a/answer/81126
- Microsoft — Outlook requirements for high-volume senders (May 5, 2025): https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/strengthening-email-ecosystem-outlook%E2%80%99s-new-requirements-for-high%E2%80%90volume-senders/4399730
- Yahoo Sender Hub: https://senders.yahooinc.com/
- CAN-SPAM Act compliance guide (FTC) — 10-business-day opt-out, postal address: https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business
- TCPA call-time rule — 47 C.F.R. § 64.1200(c): https://www.ecfr.gov/current/title-47/chapter-I/subchapter-B/part-64/subpart-L/section-64.1200
- Baymard Institute — Cart abandonment meta-analysis (~70%): https://baymard.com/lists/cart-abandonment-rate
- Klaviyo — Ecommerce email/flow benchmarks (RPR by flow): https://www.klaviyo.com/marketing-resources/ecommerce-benchmarks
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- WCAG 2.2 — incl. 2.3.1 Three Flashes or Below Threshold (W3C): https://www.w3.org/TR/WCAG22/
- email-bugs (hteumeuleu) — documented client rendering bugs (Gmail clip/style, Yahoo `!important`): https://github.com/hteumeuleu/email-bugs
- Litmus — email design/testing + dark-mode guide: https://www.litmus.com/
- Email on Acid: https://www.emailonacid.com/
- dmarcian: https://dmarcian.com/
- European Accessibility Act (in force June 28, 2025): https://ec.europa.eu/social/main.jsp?catId=1202
- #emailgeeks community: https://email.geeks.chat/

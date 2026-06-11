# Authentication & Account Flows

> Authentication is the highest-stakes, highest-abandonment funnel on most products: a single bad reset email or leaky error message loses high-intent users or invites account takeover. This guide gives an AI builder the exact tokens, thresholds, OWASP-aligned rules, and code patterns to ship sign-up, login, recovery, MFA, OAuth, sessions, and account-deletion flows that are secure, accessible, and low-friction at a senior level — with passkeys as the 2026 default.

**Apply when:** building any login, sign-up, password reset, MFA, SSO, or account-settings flow · **Related:** [Forms & Input UX](./forms-and-input-ux.md) · [Accessibility (WCAG 2.2)](./accessibility-wcag.md) · [Dark Patterns to Avoid](../08-pitfalls-and-antipatterns/dark-patterns-to-avoid.md) · [SaaS Web Apps](../03-site-types/saas-web-apps.md)

## TL;DR — rules at a glance

- **Passkeys (WebAuthn) are the default front door in 2026.** Offer passkey enrollment post-login; treat passwords/SMS as fallback. They're phishing-resistant because they're origin-bound and non-replayable — Microsoft reports phishing-resistant MFA blocks **over 99% of identity-based attacks** (Digital Defense Report 2024).
- **Get the autocomplete tokens exactly right:** `username`, `current-password` (login), `new-password` (signup/reset, on BOTH new + confirm fields), `one-time-code` (OTP). For passkey autofill, append `webauthn` LAST: `autocomplete="username webauthn"`.
- **Never reveal which field was wrong.** One generic error for login: **"Login failed; Invalid user ID or password."** Same response and same timing for valid-user-wrong-password and unknown-user (OWASP). Use constant-time comparison, no early exit.
- **Reset & recovery is the abandonment killer:** in Jared Spool's analysis of a large retailer, only ~25% of users who requested password help ever returned to finish checkout (≈75% dropped out). Minimize to 1–2 fields, prefill the known email, state next steps, show password rules upfront with live validation.
- **Password rules (OWASP):** min 8 chars (with MFA) or 15 (without), **max ≥64**, allow all Unicode + spaces, **no composition rules**, no silent truncation, check against breached-password lists.
- **Reset tokens:** cryptographically random, single-use, short expiry (15–60 min is the practical norm), invalidated on use. Generic response: **"If that email address is in our database, we will send a reset link."** Don't auto-login after reset; don't lock the account on forgot-password attempts.
- **OTP inputs must allow paste.** Single `<input>`, `inputmode="numeric"`, `autocomplete="one-time-code"`, `maxlength="6"`, never `preventDefault` on paste. Autofocus the field.
- **MFA hierarchy:** passkeys/security keys > TOTP > SMS. SMS is the weakest acceptable factor (SIM-swap, SS7) — fallback only. Always issue single-use recovery codes at enrollment.
- **Solve "which provider did I use?"** Show the linked provider on the identifier-first screen; on conflict, link accounts by verified email rather than creating a duplicate.
- **Sessions:** support idle + absolute timeouts, list active devices, offer "log out everywhere," and rotate/invalidate on password change, reset, and email change.
- **Step-up auth (re-authenticate) before sensitive changes:** password, email, MFA enrollment, deletion, payouts. Don't make the whole app high-friction — escalate only at the moment of risk.
- **Deactivate ≠ delete.** Offer both, label them honestly, and make permanent deletion findable and completable. GDPR Art. 17 erasure ≠ hiding data. Account-deletion dark patterns are a legal and trust liability.
- **Never `autocomplete="off"` on identity/password fields** — browsers/password managers ignore it on credentials and it breaks autofill (and undermines WCAG 2.2 SC 1.3.5 Identify Input Purpose). Don't double-input email or password.
- **Don't block password managers:** stable `name`/`id`, real `<form>` with submit button, paste allowed everywhere (login, reset, OTP).

---

## Sign-up

### Field minimization & structure
- **Collect the minimum to create the account; defer the rest.** Each extra field measurably drops conversion. Ideal sign-up = identifier + one credential (or just identifier for passkey/magic-link). Ask for name/role/company *after* the account exists (progressive profiling). See [Cognitive Load & Progressive Disclosure](./cognitive-load-progressive-disclosure.md).
- **Progressive vs all-at-once:** Progressive (identifier-first → choose method) maximizes conversion and is required to surface passkeys cleanly. All-at-once (email+password on one screen) is fine for low-stakes B2C but loses the identifier-first passkey path.
- **No confirm-email and no confirm-password fields.** Replace confirm-password with a show/hide toggle. Double-entry inflates fields and abandonment without a real error-prevention payoff (GOV.UK pattern).
- **Use distinct `name`/`id` values between your sign-up and sign-in forms** so password managers don't confuse the two (web.dev). E.g. sign-up `id="new-password"`, sign-in `id="current-password"`.

### Method choice (2026 priority order)
1. **Passkey (WebAuthn)** — passwordless, phishing-resistant, the recommended default. One tap with device biometric/PIN.
2. **Magic link / email OTP** — passwordless, no password to forget; cost = email deliverability + an extra hop. Good fallback and good for low-frequency apps.
3. **Social / SSO (OAuth/OIDC)** — fast, no new credential; but creates the "which provider?" problem and a dependency.
4. **Email + password** — universal baseline; always pair with breached-password checks and offer MFA.

### Passkey enrollment timing (drives adoption)
- **Prompt post-login, not buried in settings.** Triggered post-login prompts account for ~75% of passkey creations and roughly double enrollment vs settings-only (eBay/Authenticate 2025 data, via FIDO/Corbado coverage).
- **Make "Not now" frictionless and don't re-nag in the same session** — repeated prompts increase abandonment of the whole flow.
- **Copy answers two questions only: "What does this do?" and "Is it fast?"** Avoid jargon ("FIDO2", "WebAuthn credential", "public-key cryptography") — jargon lowers completion. Example: *"Sign in faster next time with your fingerprint or face. Takes a few seconds."*

### Verification (double opt-in)
- **Verify email before granting full access** for anything transactional/social. Generic creation response: **"A link to activate your account has been emailed to the address provided."** (Same message whether or not the address already existed — anti-enumeration.)
- **Allow the user into a limited state while unverified** where possible (reduces drop-off) and gate only sensitive actions until verified.
- **Verification links: single-use, expiring, cryptographically random.** Handle the `already-verified` and `expired-link` states explicitly (see [States](#states-the-ones-everyone-forgets)).

### Correct sign-up form
```html
<form action="/signup" method="POST">
  <label for="email">Email</label>
  <input id="email" name="email" type="email" autocomplete="username"
         required autofocus>

  <label for="new-password">Password</label>
  <input id="new-password" name="new-password" type="password"
         autocomplete="new-password" minlength="8" required
         aria-describedby="pw-rules">
  <p id="pw-rules">At least 8 characters. Spaces and symbols welcome.</p>

  <button type="button" id="toggle-pw"
          aria-label="Show password as plain text. Warning: your password will be visible.">
    Show
  </button>

  <button type="submit">Create account</button>
</form>
```
- Use `new-password` on both the password and any confirm field (it disables autofill there and triggers the browser's password generator).
- Show password requirements **before** submit, with live state updates as the user types (a key fix in multiple reset/signup redesign case studies).

---

## Login

### Form mechanics
- **One `<form>`, real submit button, stable `name`/`id`.** Browsers/password managers require these to store and autofill.
- **`type="email"` + `autocomplete="username"`** on the identifier; `type="password"` + `autocomplete="current-password"` on the credential.
- **Identifier-first is the senior pattern for multi-method auth:** user enters email → server returns which methods exist (passkey? password? SSO?) → present only the right path. This is what makes passkey conditional UI and "which provider" handling work.
- **Show/hide password toggle** (`type="button"`, swaps input `type` between `password`/`text`, updates `aria-label`). Replaces the confirm-password field.
- **"Remember me"** controls session *duration* (persistent vs session cookie), not "save my password." Default it **off** on shared-device-likely contexts; label it honestly ("Keep me signed in on this device"). Never tie it to storing the password in plaintext.

### Correct login form
```html
<form action="/login" method="POST">
  <label for="username">Email</label>
  <!-- append `webauthn` LAST to enable passkey autofill (Conditional UI) -->
  <input id="username" name="username" type="email"
         autocomplete="username webauthn" required autofocus>

  <label for="current-password">Password</label>
  <input id="current-password" name="current-password" type="password"
         autocomplete="current-password" required>

  <label><input type="checkbox" name="remember"> Keep me signed in</label>
  <a href="/reset">Forgot password?</a>

  <button type="submit">Sign in</button>
</form>
```

### Anti-enumeration error handling (critical)
- **Use ONE generic message** regardless of which field was wrong: **"Login failed; Invalid user ID or password."** Do not say "no account with that email," "wrong password," "account disabled," or "account not yet activated" — each leaks account existence/state (OWASP).
- **Match timing across branches.** Always compute a password hash (even for unknown users) and return in a consistent time — a several-second delta lets attackers enumerate accounts via timing (OWASP). Constant-time compare, no quick-exit.
- **Harden registration, reset, and APIs the same way** — enumeration leaks through "email already taken" on signup and "we sent you a reset" on recovery too.
- **Usability mitigation for the generic-message tradeoff:** add a CAPTCHA only after several failures, and link clearly to recovery ("Forgot password?" / "Trouble signing in?") so a genuinely-stuck user has a path. Reserve fully generic redirects-to-support for critical/high-value apps.

### Passwordless login (magic link / email OTP)
- **Magic link:** email a single-use, short-expiry, origin-bound link. State clearly *which inbox* and offer "Resend" (rate-limited) and "Use a different method." Handle `expired-link` gracefully — re-issue, don't dead-end.
- **Email/SMS OTP:** see the OTP input rules under [Accessibility & ergonomics](#accessibility--ergonomics). Codes are typically 6 digits, valid 5–10 min, single-use.

---

## Password reset & account recovery

The single highest-friction, highest-abandonment flow. In Jared Spool's analysis of a large online retailer, **only ~25% of users who requested password help ever returned to complete checkout** — roughly **75% dropped out mid-process**. Baymard's checkout research likewise ties strict password rules and the resulting reset friction directly to lost orders (forced account creation alone drives ~25% of cart abandonment). Design reset like a conversion funnel, not an afterthought.

### Flow rules
- **Minimize: 1–2 fields.** Enter email → "check your inbox." No security questions, no extra PII. Security questions must never be the sole reset mechanism (OWASP) — answers are guessable/findable.
- **Prefill the email** if it was entered on the previous login screen.
- **State next steps explicitly:** "If an account exists for X, we've emailed a link to reset your password. Check spam if you don't see it in a few minutes." Generic by design (anti-enumeration).
- **Show password rules upfront with live validation** on the new-password screen; indicate match status if you keep a confirm field. Hidden-until-error rules are a top documented reset pain point.
- **Best reset flow = no reset flow.** Passwordless (passkey/magic link) eliminates most reset traffic; treat reset as a fallback, not the primary recovery path.

### Reset token & security (OWASP Forgot-Password)
- **Token:** cryptographically secure RNG, long enough to resist brute force, **single-use**, **stored hashed**, linked to one user, **invalidated immediately after use**, short expiry (the spec says "appropriate period"; 15–60 min is the practical industry norm).
- **Generic, constant-time response** for existent and non-existent accounts: **"If that email address is in our database, we will send you an email to reset your password."**
- **Never email the password** (or any new password) — only a reset link/code; tell the user it was changed.
- **Do NOT lock the account** in response to forgot-password attempts (that becomes a denial-of-service against the real owner). Rate-limit per-account / add CAPTCHA instead.
- **Don't auto-login after reset** — send the user through normal login (reduces complexity and a class of attacks). After a successful reset, **invalidate other sessions** (or ask the user) and notify the account email that the password changed.

### Account lockout & rate-limit UX
- **Throttle, don't hard-lock, by default.** Prefer progressive delays / temporary lockout with a clear, finite duration over indefinite lockout. OWASP: cap repeated lockouts (max ~2–3 cycles before a longer/permanent lock that needs support).
- **Tell the user what happened and when they can retry:** "Too many attempts. Try again in 15 minutes, or reset your password." Show a countdown if the wait is fixed.
- **Offer an escape hatch on lockout:** reset link and/or alternate factor, so a legitimate user isn't dead-ended.
- **Account recovery is where MFA goes to die** — research shows lockout after losing factors drives users to abandon services or avoid MFA entirely. Provide redundant recovery (recovery codes + a second factor + a defined support path) and communicate it at enrollment.

---

## Two-factor / MFA

### Method hierarchy (security, strong → weak)
1. **Passkeys / hardware security keys (FIDO2)** — phishing-resistant (origin-bound, non-replayable). A passkey is *already* multi-factor (device + biometric/PIN), recognized as MFA by NIST SP 800-63B — you generally don't stack TOTP on top of it.
2. **TOTP authenticator apps** (Google/Microsoft Authenticator, Authy, 1Password, Bitwarden) — offline, free, no SMS cost, immune to SIM-swap/SS7. **Not** phishing-resistant (real-time AiTM proxy can relay codes). The practical enterprise default below passkeys.
3. **SMS / phone OTP** — **weakest acceptable factor.** Vulnerable to SIM-swap, SS7 interception, number-porting fraud (FBI IC3 reported ~$48.8M in SIM-swap/port-out losses in 2023). Use as fallback only; still better than no second factor.

### Recovery codes (mandatory backup)
- **Issue single-use recovery codes at MFA enrollment** (typically 8–10 codes). Display once, prompt the user to save/print, store them hashed server-side. Each works once.
- **Encourage registering a second factor** so the user is unlikely to lose all factors at once.
- Show recovery codes as a copy-able block; allow regeneration (which invalidates the old set).

### Step-up / adaptive auth
- **Re-authenticate at the moment of risk, not constantly.** Require an existing factor before: changing password, changing email/phone, adding/removing an MFA method, deleting the account, viewing/exporting sensitive data, making payouts/payments.
- **Securing factor changes is the takeover-prevention crux** — attackers target the "change my phone number" path. Require re-auth with a *currently enrolled* factor before swapping factors.
- **Adaptive/risk-based:** escalate (prompt for second factor) only on elevated risk signals (new device, new geo, impossible travel). Keeps low-risk logins frictionless.

### OTP entry UX
- **Single input, paste allowed, autofocus, numeric keyboard, autofill enabled.** Format SMS as origin-bound (`@example.com #123456`) so iOS/Android can autofill via `autocomplete="one-time-code"`. (See full snippet in [Accessibility & ergonomics](#accessibility--ergonomics).)

---

## OAuth / social login & account linking

### "Which provider did I use?" — the core UX problem
- **Identifier-first solves it:** ask for email first, then show the method the user actually has — e.g. "You usually sign in with Google" — instead of a wall of equal provider buttons that invites guessing.
- **Show provenance on the account itself:** in settings, list "Connected accounts: Google (you@gmail.com)" so users can see and manage what they used.
- **When the user picks the wrong provider, recover gracefully:** if Google returns an email that already has a password account, *link* them (after verifying email ownership), don't create a duplicate or throw "account exists."

### Account linking / merging
- **Link by verified email, never by unverified claim.** If the OAuth provider asserts a verified email that matches an existing account, prompt to link ("This email is already registered — connect Google to your existing account?") after a re-auth/verify step.
- **Never silently merge** two accounts with different data; that risks one user seeing another's data if an email was reassigned. Verify ownership of both.
- **Let users unlink** a provider only if another sign-in method remains — otherwise warn and require setting a password/passkey first, so they don't lock themselves out.
- **Email change from the IdP:** don't trust the OAuth email blindly for sensitive changes; re-verify on your side.

### Provider button rules
- **Use each provider's official, current branding and required button text** ("Sign in with Apple", "Continue with Google"). Wrong/outdated branding can violate the provider's terms.
- **Apple App Store rule (web-to-app parity):** an iOS app that uses a third-party/social login for the primary account (Guideline 4.8) must *also* offer an equivalent login that limits data to name + email, lets users keep their email private, and doesn't track for ads without consent. Sign in with Apple satisfies this, but it's no longer named as the only option, and there are exceptions (your-own-account-only, enterprise/education, government eID).
- **OAuth scopes: request the minimum.** Asking for broad scopes ("manage your contacts") on the consent screen spikes drop-off and distrust.

---

## Session management

- **Two timeout dimensions:** **idle timeout** (inactivity, e.g. 15–30 min for sensitive apps; hours/days for low-risk) and **absolute timeout** (hard cap regardless of activity, e.g. 8–24h, longer for "remember me"). Implement both.
- **Token refresh from a UX angle:** silently refresh access tokens before expiry so the user never sees a mid-action logout; on hard expiry, **preserve the user's in-progress work** (don't drop a half-written form), re-auth, then return them to where they were. A surprise logout that discards work is a top complaint.
- **Multi-device session list:** show active sessions with device, location, last-active, and "this device" marked. Let users revoke any session individually.
- **"Log out everywhere"** (revoke all other sessions) is an expected control after a suspected compromise.
- **Invalidate/rotate sessions on:** password change, password reset, email change, MFA changes, and explicit logout. Offer (or auto-do) "sign out other sessions" after these.
- **"Remember me"** = longer-lived session/refresh token bound to the device; on logout, actually revoke it server-side, don't just clear the cookie.
- **Notify on new-device/new-location sign-in** by email — a cheap, high-trust takeover signal.

---

## Account settings, email change & deletion

### Profile & credential changes
- **Password change form:** old password (`autocomplete="current-password"`) + new password (`autocomplete="new-password"`). Require re-auth. **Clear/hide the form after success** or the browser may fail to record the update (web.dev). Notify the account email.
- **Email change must re-verify the NEW address** before it becomes the login identifier; keep the old address active until the new one is confirmed, and **notify the OLD address** ("your email was changed") so a hijack is visible and reversible. Re-auth (step-up) before starting.
- **Phone change:** verify the new number via OTP and re-auth first (this is a SIM-swap-adjacent takeover vector).

### Deactivate vs delete (label honestly — this is a dark-pattern minefield)
- **They are different actions; offer both and name them truthfully.** *Deactivate/Pause* = reversible, data retained, account hidden. *Delete* = permanent erasure of personal data. Calling permanent-feeling "Deactivate" what is really retention is a documented dark pattern; vague labels ("Close", "Hide") mislead users about permanence.
- **Make deletion findable and completable.** Research on the top social platforms (CSCW 2022) found that users frequently start deletion but abandon it, driven by platform-created friction — buried controls, confusing close-vs-delete labels, and reactivation traps. Don't bury it, don't require a phone call, don't add fake friction.
- **GDPR Art. 17 ("right to be forgotten") + CCPA right to delete:** users can demand erasure of personal data; "deactivation" that merely hides data does **not** satisfy erasure. Erasure isn't absolute — you may retain what a legal obligation requires (state this in the flow), but act "without undue delay." GDPR fines reach up to €20M or 4% of annual worldwide turnover, whichever is higher.
- **Apple App Store mandates in-app deletion:** if an iOS app supports account creation, it must let users initiate account deletion *from within the app* (Guideline 5.1.1(v)) — a web-only or email-support-only delete path fails review. Relevant for any web app with a paired iOS app.
- **Honest deletion flow:** (1) re-authenticate, (2) plainly state what will be deleted, what's retained for legal reasons, and whether there's a grace/cooling-off window, (3) confirm with a real action (type "DELETE" or re-enter password) — confirmation friction here is legitimate, manipulation is not, (4) email confirmation when erasure completes.
- **Provide data export** ("Download your data" — GDPR Art. 20 portability) as a self-serve, machine-readable option, separate from deletion.

---

## States (the ones everyone forgets)

Design and copy each explicitly; these are where flows feel broken. See [Interaction Design & UI States](./interaction-design-and-states.md).

| State | What the user sees / next step |
|---|---|
| **Error (generic)** | "Login failed; Invalid user ID or password." + clear "Forgot password?" link. Never field-specific. |
| **Empty / first-run** | No account yet → prominent "Create account"; no sessions list yet, etc. |
| **Unverified** | Limited access + "Resend verification" (rate-limited) + which inbox to check. |
| **Already-verified** | "This account is already verified — sign in." Don't show an error. |
| **Expired link** | "This link expired. Request a new one." → one-tap re-issue, never a dead-end. |
| **Locked / throttled** | "Too many attempts. Try again in N min" (countdown) + reset/alternate-factor escape hatch. Not indefinite. |
| **Rate-limited (resend/OTP)** | Disabled "Resend" with a visible cooldown timer. |
| **Session expired** | Re-auth prompt that preserves in-progress work and returns the user to their prior context. |
| **Magic-link wrong device** | Allow continuing in the device/browser that requested it; explain if cross-device isn't supported. |

---

## Accessibility & ergonomics

- **Correct `autocomplete` tokens are an accessibility requirement** (WCAG 2.2 SC 1.3.5 Identify Input Purpose, Level AA): `username`, `current-password`, `new-password`, `one-time-code`. They also enable password managers and reduce errors.
- **Never `autocomplete="off"` on identity/password fields** — browsers and password managers ignore it on credentials, and it harms users who rely on managers.
- **Right input types & keyboards:** identifier `type="email"`; phone `type="tel"`; OTP `type="text"` + `inputmode="numeric"` (NOT `type="number"` — it adds spinners and drops leading zeros). 16px+ input font to stop iOS auto-zoom.
- **Show/hide password** with `<button type="button">` and an `aria-label` that warns the password will become visible. Toggling reveals it to shoulder-surfers — keep it user-initiated.
- **WebAuthn a11y:** the passkey ceremony is browser/OS-native (works with platform AT), but your trigger button needs a real accessible name; check `isConditionalMediationAvailable()` and always provide a button fallback for browsers/devices without conditional UI (e.g. Windows 10).
- **Error messaging:** `aria-invalid="true"` + `aria-describedby` pointing to the message; `role="alert"` / `aria-live="polite"` for async auth errors so screen-reader users hear them.

### OTP input — single field, paste allowed, autofill-ready
```html
<label for="otp">Enter the 6-digit code we sent</label>
<input id="otp" name="otp" type="text"
       autocomplete="one-time-code" inputmode="numeric"
       pattern="\d{6}" maxlength="6" required autofocus>
```
- **Allow paste.** Never `preventDefault` on the paste event. A single `<input>` enables copy/paste and platform autofill for free; if you build a segmented UI, back it with one hidden `one-time-code` input and distribute pasted chars across slots (don't set per-box `maxlength` that blocks paste). Libraries: `input-otp` (powers shadcn/ui).
- **Origin-bound SMS** (`@example.com #123456`) lets iOS/Android autofill the code and resists phishing (code only autofills on the matching origin) — see [MDN OTP](https://developer.mozilla.org/en-US/docs/Web/Security/Authentication/OTP).

### Passkey conditional UI (autofill) — exact wiring
```html
<input id="username" type="text" autocomplete="username webauthn">
```
```js
// `webauthn` must be the LAST token in the autocomplete value (parsed right-to-left).
// Optional-chain the check: the method is undefined on older browsers (a bare call throws).
if (await window.PublicKeyCredential?.isConditionalMediationAvailable?.()) {
  const ac = new AbortController();
  navigator.credentials.get({
    publicKey: publicKeyRequestOptions,   // server challenge (discoverable creds)
    mediation: "conditional",             // "set and forget": resolves only on a passkey pick
    signal: ac.signal,                    // abort this if the user submits a password instead
  }).then(handleAssertion).catch(() => {});
  // Conditional get() never resolves on its own — call ac.abort() before any non-passkey login.
  // Requires discoverable (resident) credentials. Always keep a button-triggered fallback.
}
```

---

## Concrete specs & numbers

- **Password length:** min **8** (with MFA) / **15** (without); max **≥64**; allow all Unicode + whitespace; **no composition rules**; no silent truncation; screen against breached-password lists (OWASP).
- **Reset/verification/magic-link token:** CSPRNG-generated, single-use, hashed at rest, invalidated on use, short expiry (15–60 min practical norm).
- **OTP code:** typically 6 digits, valid ~5–10 min, single-use, rate-limited resend with visible cooldown.
- **Recovery codes:** ~8–10 single-use codes at MFA enrollment, shown once, stored hashed.
- **Lockout:** progressive delay / temporary lock with finite, communicated duration; cap repeated lockouts (~2–3) before longer/permanent + support path; never lock on forgot-password attempts.
- **Sessions:** idle timeout 15–30 min (sensitive) up to hours/days (low-risk); absolute cap e.g. 8–24h; rotate on credential changes.
- **Touch targets** ≥44×44pt (Apple) / 48×48dp (Material) / 24×24 CSS px (WCAG 2.5.8); 16px+ input font.
- **Abandonment reality:** ~75% of users who request password help never finish (Spool, large-retailer analysis); users frequently start but abandon account deletion due to platform friction (CSCW 2022).
- **MFA security data points:** phishing-resistant MFA blocks >99% of identity-based attacks (Microsoft DDR 2024); SIM-swap/port-out losses ~$48.8M reported in 2023 (FBI IC3).

---

## Tools & libraries

Framed neutrally — pick by team capacity, compliance needs, and lock-in tolerance.

- **WebAuthn/passkey libs:** `@simplewebauthn/server` + `@simplewebauthn/browser` (well-maintained, framework-agnostic; good when you self-host auth). Platform passkey APIs are native — these wrap the ceremony + attestation/assertion verification.
- **Clerk** — drop-in React/Next-first auth UI (passkeys, social, MFA, orgs). Fastest to ship; opinionated, hosted, paid at scale. When-not: you need full backend control or non-JS stacks.
- **Auth0 (Okta)** — mature, enterprise SSO/SAML/OIDC, broad protocol support. When-not: cost-sensitive small projects; pricing/complexity overshoot.
- **Supabase Auth** — open-source Postgres-native auth (email, OAuth, MFA, passkeys); great if already on Supabase. When-not: you don't want Postgres/Supabase coupling.
- **WorkOS** — enterprise readiness as a service (SSO/SAML, SCIM directory sync, audit logs). Best for B2B SaaS selling upmarket. When-not: simple B2C login.
- **Better Auth** — open-source, TypeScript-first, framework-agnostic, self-hosted; full control of data. When-not: you want a fully hosted, no-ops solution.
- **Reference docs (don't skip):** [web.dev sign-in best practices](https://web.dev/articles/sign-in-form-best-practices), [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html), [OWASP MFA Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html), [W3C WebAuthn](https://www.w3.org/TR/webauthn-3/).

---

## Do / Don't

**Do**
- Offer passkeys and prompt enrollment post-login; keep a button fallback.
- Use the exact `autocomplete` tokens; append `webauthn` last for conditional UI.
- Return one generic, constant-time login error and harden signup/reset/API the same way.
- Minimize reset to 1–2 fields, prefill email, state next steps, show password rules live.
- Issue single-use recovery codes at MFA enrollment; encourage a second factor.
- Re-authenticate before password/email/MFA/deletion changes; notify the account on changes.
- List active sessions, support "log out everywhere," rotate sessions on credential changes.
- Label deactivate vs delete honestly; make deletion completable; offer data export.

**Don't**
- Reveal which field was wrong, or leak account existence via signup/reset/timing.
- Use `autocomplete="off"` on identity/password fields, or block paste on password/OTP fields.
- Use confirm-email/confirm-password double-entry; use a show/hide toggle instead.
- Lock the account on forgot-password attempts, or auto-login after a reset.
- Make SMS the primary second factor; treat it as fallback only.
- Use `type="number"` for OTP/codes; it drops leading zeros and adds spinners.
- Silently log users out mid-action and discard their work.
- Bury account deletion, gate it behind a phone call, or disguise "deactivate" as deletion.

---

## Common pitfalls & how to avoid them

- **Enumeration via "email already in use" on signup / "we sent a link" on reset.** → Same generic, constant-time response everywhere; rate-limit + CAPTCHA-on-failure instead of distinct messages.
- **Timing leak** (fast reject for unknown users). → Always hash a dummy password and return in consistent time.
- **`autocomplete="off"` "for security."** → Ignored on credential fields and breaks managers; remove it. Pen-test checklists asking for it are outdated.
- **OTP boxes that block paste / drop leading zeros.** → Single `type="text"` `inputmode="numeric"` input, paste allowed, `one-time-code`.
- **Reset link that doesn't invalidate after use or never expires.** → Single-use, short expiry, hashed at rest, invalidated on use.
- **Email change that switches the login email before verifying the new one.** → Verify new address first; keep old active; notify old address.
- **No recovery path for MFA → permanent lockout.** → Recovery codes + a second factor + a defined, identity-checked support path; communicate at enrollment.
- **OAuth duplicate accounts** when a user picks a different provider with the same email. → Link by verified email; never silently create a second account.
- **Surprise logout discards a long form.** → Silent refresh + preserve in-progress state + return to context.
- **"Delete" that only hides data.** → Real erasure for GDPR/CCPA; honest labels; export option.
- **Password-manager-hostile forms** (random `id`s, no `<form>`, JS-only submit). → Stable `name`/`id`, real form + submit button.

---

## What real users complain about

Synthesized from developer/UX forum threads and vendor field data:

- **"I just want in."** Auth friction is consistently named one of the most hated parts of both using and building products; every avoidable hop (extra screen, re-entry, surprise 2FA prompt) bleeds users.
- **Password-reset rage.** Links that expire too fast, land in spam, or password rules that only appear *after* submit — the documented ~75% mid-reset drop-off is felt, not theoretical.
- **OTP paste blocked / leading zeros eaten.** Segmented inputs that refuse a pasted code or that use `type="number"` are a recurring, specific complaint.
- **Forced SMS 2FA + lockout.** Users locked out after a new phone/SIM with no recovery codes, then funneled into slow support — drives MFA avoidance.
- **"Which login did I use?"** People genuinely forget whether they used Google, Apple, email, or a password — a wall of equal provider buttons makes it worse; identifier-first + showing the connected provider fixes it.
- **Account-deletion dark patterns.** Hidden delete controls, "deactivate" masquerading as deletion, and phone-call-only deletion erode trust and increasingly invite regulatory attention.
- **Surprise logouts that lose work.** Session timeout mid-task, then the form is gone after re-login — a top "small thing that makes me hate an app" complaint.

---

## Senior-level checklist

- [ ] Passkeys offered; post-login enrollment prompt; jargon-free copy; button fallback when conditional UI unavailable.
- [ ] Exact `autocomplete` tokens on every field; `webauthn` appended last on the identifier.
- [ ] Login returns ONE generic error, constant-time; signup/reset/API equally enumeration-safe.
- [ ] Password policy: 8/15 min, ≥64 max, all Unicode, no composition rules, breached-list check, no truncation.
- [ ] Reset: 1–2 fields, email prefilled, next-steps stated, rules shown live; token single-use/short-expiry/hashed; generic response; no account lock; no auto-login; sessions invalidated; user notified.
- [ ] Lockout is throttled + finite + communicated, with reset/alternate-factor escape hatch.
- [ ] MFA: passkey/TOTP primary, SMS fallback only; recovery codes issued at enrollment; second factor encouraged.
- [ ] Step-up re-auth before password/email/phone/MFA/deletion/payout changes; factor-change path secured.
- [ ] OAuth: identifier-first; link by verified email; no duplicates/silent merges; can't unlink last method; minimal scopes; correct provider branding.
- [ ] Sessions: idle + absolute timeouts; device list; "log out everywhere"; rotate on credential changes; silent refresh preserving work; new-device email alerts.
- [ ] Email change re-verifies new address, keeps old active, notifies old; password change clears the form after success.
- [ ] Deactivate vs delete distinct + honestly labeled; deletion findable + completable; GDPR/CCPA erasure real; data export available.
- [ ] OTP input: single field, paste allowed, `inputmode="numeric"`, `one-time-code`, autofocus, origin-bound SMS.
- [ ] All states designed: error/empty/unverified/already-verified/expired-link/locked/rate-limited/session-expired.
- [ ] A11y: WCAG 1.3.5 autocomplete, `aria-invalid`/`aria-describedby`, `role="alert"` errors, show/hide with warning label, no `autocomplete="off"` on credentials.

---

## Sources

- [Sign-in form best practices — web.dev](https://web.dev/articles/sign-in-form-best-practices)
- [autocomplete HTML attribute — MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/autocomplete)
- [One-time passwords (OTP) — MDN](https://developer.mozilla.org/en-US/docs/Web/Security/Authentication/OTP)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
- [OWASP Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
- [OWASP Testing for Account Enumeration (WSTG)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account)
- [HTML attributes to improve 2FA experience — Twilio](https://www.twilio.com/en-us/blog/html-attributes-two-factor-authentication-autocomplete)
- [Best Practices for OTP Input Forms in HTML — Twilio](https://www.twilio.com/en-us/blog/developers/best-practices/otp-input-forms-html)
- [WebAuthn Conditional UI (Passkeys Autofill) — Corbado](https://www.corbado.com/blog/webauthn-conditional-ui-passkeys-autofill)
- [W3C Web Authentication (WebAuthn) Level 3](https://www.w3.org/TR/webauthn-3/)
- [input-otp (Guilherme Rodz)](https://github.com/guilhermerodz/input-otp)
- [Configure Recovery Codes for MFA — Auth0 Docs](https://auth0.com/docs/secure/multi-factor-authentication/configure-recovery-codes-for-mfa)
- [Art. 17 GDPR — Right to erasure ('right to be forgotten')](https://gdpr-info.eu/art-17-gdpr/)
- ["We've Disabled MFA for You": Security & Usability of MFA Recovery (arXiv 2306.09708)](https://arxiv.org/pdf/2306.09708)
- [Understanding Account Deletion and Dark Patterns on Social Media (CSCW 2022)](https://bpb-us-w2.wpmucdn.com/voices.uchicago.edu/dist/1/2826/files/2022/06/PREPRINT_Understanding_Account_Deletion_CSCW2022-1.pdf)
- [Forget the password reset flow as you know it — Stytch](https://stytch.com/blog/forget-the-password-reset-flow-as-you-know-it/)
- [Avoid Complex Password-Creation Requirements — Baymard](https://baymard.com/blog/password-requirements-and-password-reset)

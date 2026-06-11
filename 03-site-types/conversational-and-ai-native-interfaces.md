# Chat, Conversational & AI-Native Interfaces

> Patterns for designing and building chat and AI-native product surfaces end to end: message-thread layout, token-by-token streaming with stop/regenerate, the input composer, suggested prompts and first-run states, rich response rendering (markdown, code, tables, citations, tool calls, reasoning), latency masking, error/retry/rate-limit handling, message actions, conversation history, trust-and-safety affordances, and the accessibility of a live-updating transcript. The senior-level outcome is a chat surface that streams smoothly, never thrashes layout, masks latency honestly, calibrates trust instead of inflating it, and remains usable with a keyboard and a screen reader while text is still arriving.

**Apply when:** building a chat/assistant surface, an agentic tool, or any UI where an LLM streams responses into a transcript · **Related:** [Cognitive Load & Progressive Disclosure](../01-ux-fundamentals/cognitive-load-progressive-disclosure.md) · [Interaction Design & UI States](../01-ux-fundamentals/interaction-design-and-states.md) · [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md) · [Forms & Input UX](../01-ux-fundamentals/forms-and-input-ux.md)

## TL;DR — rules at a glance
- **Progressive disclosure is the spine.** Layer 1 = the answer. Layer 2 = an expandable "why". Layer 3 = full reasoning/tool trace. Each on demand, hidden by default. Don't over-explain every turn; don't hide everything.
- **Streaming is the baseline, not a feature.** A non-streaming LLM UI feels broken next to a streaming one: with batch delivery the user waits the *full* generation time (e.g. 8s for 320 tokens at 40 tok/s) staring at a spinner; streaming shows the first token in ~hundreds of ms and lets the user read and interrupt while the rest generates. The win is **perceived** latency and **early interruption**, not throughput. *Skip* streaming when only the final result matters (structured JSON/SQL, lookups, short summaries) — partial structured output is unusable mid-stream.
- **The Stop button must be one of the most visible elements on screen during streaming.** Never in a menu. Reachable by Tab. On stop: flush/clear the pending buffer, remove the cursor, reset the RAF flag, show "response stopped", expose **Retry** and `.focus()` it.
- **Auto-scroll only when the user is at the bottom.** Disable the instant they scroll up; resume when they return. Threshold: `gap = scrollHeight - scrollTop - clientHeight; userScrolled = gap > 60`. Reset `userScrolled` when a new stream begins.
- **Batch DOM writes with `requestAnimationFrame`.** Tokens can arrive faster than ~60fps (~16.7ms/frame). Buffer chars in a pending string, flush once per frame via a single RAF-queued flag. Extend text nodes; create a new paragraph only on a newline. Never wipe `innerHTML` per token.
- **Render partial markdown safely.** Buffer incomplete syntax — a half-open `**bold` or unclosed ` ``` ` fence must not break layout. Defer code-block parsing until the closing fence, or render progressively with a visible in-block streaming indicator. Re-render on whitespace/sentence boundaries, not every token.
- **Sanitize model output before rendering.** LLM/tool text is untrusted and prompt-injectable — run it through DOMPurify (or a raw-HTML-off parser), allowlist link schemes, `rel="noopener noreferrer"` outbound links. Never `innerHTML` raw markdown.
- **Transcript a11y:** container `role="log"` + `aria-live="polite"` + `aria-atomic="false"`; `aria-busy="true"` while streaming; visual cursor `aria-hidden="true"`. **Never** `aria-live="assertive"` for streaming (clears the speech queue every token). **Debounce** announcements (batch every few seconds), don't announce per chunk.
- **Live region must exist and be empty in the DOM before injecting.** If added dynamically, wait briefly (~2s rule of thumb) for the accessibility API to register it — a practical SR workaround. Announcing status without moving focus is the WCAG SC 4.1.3 (Status Messages, AA) requirement.
- **Never show a blank input box.** Personalized greeting + 3–4 curated prompts that showcase *distinct* capabilities. The blank-slate first-run is the highest-leverage moment — recognition over recall; a blank box facing a new user is a blank-page problem that depresses first-turn success.
- **Separate system status from generated content** — visually and structurally. Retry/reconnect/streaming chatter is ephemeral and must **not** pollute the permanent thread.
- **Retry with exponential backoff + jitter.** ~1–2s base, double per attempt, cap ~5–8 attempts. Honor `Retry-After` on HTTP 429. Never retry 4xx request errors — surface and fix. Track each in-flight message by a client-generated ID for idempotency.
- **Error copy is specific and actionable**, never "Something went wrong" or an infinite spinner. Keep the partial streamed text visible on error. Offer next-best-actions: retry / shorten / switch model / continue.
- **Citations: visually distinct, adjacent to the claim, clickable, meaningfully labeled** (publication/article name, never "Source"). Offer "See all sources." People rarely click them, but their presence raises trust — so they must be honest.
- **Disclaimers go near the input**, not in a footer. Direct + paired with an action: "Claude can make mistakes. Please double-check responses." Avoid vague "AI-generated, for reference only."
- **Avoid first-person anthropomorphism in trust-critical copy** ("I thought about your problem"). It inflates trust. Use neutral framing ("This answer is based on: [link]").
- **Tool calls render as inline step cards, visible by default** (name, params, status, result) — prevents the black-box feeling. **Reasoning** is a collapsible accordion, auto-open while streaming, collapsed when done.

---

## 1. Thread layout & message ownership

The transcript is a vertical, append-only list of message turns. Get ownership signals unambiguous, then never break the visual rhythm.

```
┌────────────────────────────────────────────┐
│                          ┌────────────────┐ │
│                          │ user message   │ │ ← right, filled bubble, ~70–80% max-width
│                          └────────────────┘ │
│ ┌──────────────────────────────────────┐    │
│ │ ◎ assistant message                  │    │ ← left, avatar/glyph, full text-column width
│ │   markdown, code, citations…         │    │
│ │   [⧉ copy] [✎ edit] [↻ regen] [👍👎]  │    │ ← actions, revealed on hover/focus
│ └──────────────────────────────────────┘    │
└────────────────────────────────────────────┘
```

- **Two ownership conventions, pick one and hold it:** (a) the classic asymmetric layout — user right-aligned in a tinted bubble, assistant left-aligned with an avatar/glyph; or (b) the document layout — both left-aligned, distinguished by a small role label + avatar (ChatGPT/Claude desktop style). Document layout scales better for long, markdown-rich assistant turns and avoids the "wall of bubble" problem.
- **Never put long assistant markdown inside a narrow bubble.** Bubbles imply short SMS-style content; once you render tables, code, and multi-paragraph answers, give the assistant a **full text column** (max ~`65–75ch` for readability) and reserve bubbles for the (usually shorter) user turn.
- **Distinguish ownership by ≥2 cues**, not color alone: alignment + avatar/label + background. Color-only fails WCAG and dark mode.
- **Stable widths.** Assistant text column should not reflow horizontally as it streams — set the column width up front so growth is purely vertical.
- **Group consecutive same-role messages** with tighter spacing (e.g. 6–8px) than role changes (16–24px); show the timestamp/avatar once per group.
- **Anchor the composer to the bottom**, transcript scrolls above it. On mobile, the composer must sit above the on-screen keyboard (`env(safe-area-inset-bottom)` + `100dvh`, not `100vh`).
- **Long-thread performance:** a thousand-turn transcript bloats the DOM and stutters scroll. Window/virtualize *old* turns, but keep the **actively streaming** message (and the live region) in the real DOM — virtualizing the streaming node breaks `aria-live`, in-page find, and text selection. Only the streaming tail needs RAF-batching; settled turns are static.

## 2. Streaming the response

Streaming is the defining interaction. The two hard problems are **rendering performance** (don't thrash) and **partial-syntax safety** (don't break layout mid-token).

### Why stream
- Cuts **perceived** latency: the user reads from the first token instead of waiting the full generation. At ~40 tok/s a 320-token answer takes ~8s end-to-end; streaming hands over the first words in ~hundreds of ms so the wait *feels* like the first-token latency, not the total. The Doherty Threshold (400ms; Doherty & Thadani, IBM Systems Journal, 1982) is the bar for "feeling in sync" — a fast first token matters more than total duration.
- Builds trust (visible progress) and enables **early interruption** of bad output (saves API cost + the user's time).

### Batch DOM writes per frame
Tokens often arrive faster than the browser paints (~60fps, ~16.7ms/frame). Buffer and flush once per RAF; a single queued flag prevents redundant scheduling.

```js
let pending = '';
let rafQueued = false;

function onToken(text) {
  pending += text;
  if (!rafQueued) {
    rafQueued = true;
    requestAnimationFrame(flush);
  }
}

function flush() {
  rafQueued = false;
  if (!pending) return;
  appendToCurrentBlock(pending); // extend text node, or split on '\n' into a new <p>
  pending = '';
  if (!userScrolled) scrollToBottom();
}
```

- **Extend, don't rebuild.** Append to the existing text node (`node.appendData(...)` or `node.textContent += ...`); create a new paragraph element **only on a newline**. Wiping `innerHTML` every token forces a full layout recalc and destroys selection/focus.
- **Layout recalc should fire only on structural changes** (new block, closed fence), not on text extension.
- **Re-parse markdown on boundaries**, not per token: whitespace, sentence end, or newline. Re-rendering the whole message on every token is the #1 jank source.

### Stop / interrupt
The Stop button replaces Send during streaming and is among the most prominent controls on screen.

```js
function onStop() {
  controller.abort();        // AbortController on the fetch/stream
  flushPending();            // commit whatever's buffered — don't lose visible text
  pending = '';
  rafQueued = false;         // reset RAF flag so a future stream schedules cleanly
  removeStreamingCursor();
  setMessageStatus('stopped'); // ephemeral "response stopped" label
  showRetryButton().focus(); // keyboard users land on Retry
}
```

- On stop, **keep the partial text** — the user was reading it. Mark the message as stopped, expose Retry/Continue.
- Stop must be **Tab-reachable**; don't hide it behind a kebab menu. Bind a keyboard shortcut too (`Esc` while focus is in the composer/transcript is the common one).
- **Abort the network stream, don't just hide it.** `controller.abort()` must actually cancel the request so you stop paying for and receiving tokens; a Stop that only blanks the UI keeps generating server-side.

### Regenerate & continue
- **Regenerate** re-runs the *same* prompt → produces a new assistant variant. Pair with branch navigation (`‹ 2/3 ›`) so prior variants aren't destroyed.
- **Continue** appends to a truncated/stopped response (resume from where it stopped).
- Place both near the assistant message footer; regenerate is also the natural recovery action after an error.

## 3. The input composer

The composer is where the product either interprets intent well or dumps the burden of articulation onto the user. *Good conversational UI interprets intent, maps it to deterministic actions, and plays that back.*

### Layout & growth
- **Multiline auto-grow textarea**, min 1 row, cap at ~6–8 rows then scroll internally (don't let it eat the viewport). Grow by syncing height to `scrollHeight`.

```js
function autoGrow(el) {
  el.style.height = 'auto';
  el.style.height = Math.min(el.scrollHeight, MAX_PX) + 'px';
}
```

- **Send affordance:** `Enter` sends, `Shift+Enter` inserts a newline — this is the near-universal convention; honor it. On touch, show a dedicated Send button (no hardware Enter), and consider a setting for `Enter`=newline on mobile.
- **Guard IME composition:** never send on the `Enter` that *commits* a CJK/IME candidate. Check `e.isComposing` (or `keyCode === 229`) and bail — sending mid-composition is a top complaint for Japanese/Chinese/Korean users. The `keydown` for a committing Enter fires with `isComposing === true`.
- **Disable Send when empty/whitespace-only**; show character/token feedback only near the limit, not always.
- **Never block typing while a response streams.** Let users compose the next message; queue or send-on-stream-end per your model's concurrency.

### Affordances: slash, @, attachments
- **`/` opens a filterable command popup.** Support inline args (`/plan <text>`), context-aware visibility (hide commands the current model can't run), and **queueing during an active turn** (type command, `Tab` to queue). Reference: OpenAI Codex CLI's filter-as-you-type, inline-args, Tab-to-queue model.
- **`@` mentions** scope context (files, people, tools, knowledge sources) via the same popup mechanics.
- **Attachments:** drag-drop onto the composer + a paperclip button + clipboard paste (images). Show per-file chips with type/size, a remove (×), and an upload/parse progress state. Validate type + size *before* upload; surface rejects inline ("PDF over 20 MB — try a smaller file").
- The popup is a `role="listbox"`; arrow keys navigate, `Enter`/`Tab` selects, `Esc` closes. See [Component Libraries & Headless UI](../04-libraries-and-tools/component-libraries-headless.md) for combobox/popover primitives.

## 4. Empty / first-run state (the blank slate)

The single highest-leverage moment. **Recognition over recall** — never face a user with a blank box and a blinking cursor.

- **One main idea per state.** Do *not* cram prompts + commands + tips + illustration together — that's cognitive overload. Pick the prompts.
- **Personalized greeting** (name if known) + **3–4 curated example prompts** that demonstrate *breadth of distinct capabilities* (e.g. "Summarize a document", "Write SQL from a question", "Debug this stack trace", "Plan a 3-day trip") — not four variations of one task.
- **Make prompts one-click** (fill the composer or send directly). The goal is a successful first turn with zero typing.
- **Returning users** can skip the greeting and land on conversation history; reserve the full blank slate for genuinely new/empty threads.
- **Set expectations early:** the disclaimer and a one-line capability/limitation note belong here, during onboarding, not buried later.

```
  Hi Sasha — what can I help with?

  ┌─ Summarize a PDF ─┐ ┌─ Write SQL from plain English ─┐
  ┌─ Debug a stack trace ─┐ ┌─ Plan a 3-day itinerary ─┐

  [ Message…                                    ↑ ]
  Claude can make mistakes. Please double-check responses.
```

## 5. Rendering rich responses

LLM output is markdown by default. Render it well — and render *partial* markdown without breaking.

### Markdown & partial-syntax safety
- **Sanitize before you render.** LLM and tool output is untrusted: a model can emit `<img onerror>`, `<script>`, `javascript:`/`data:` links, or be steered by prompt injection. Run rendered HTML through a sanitizer (DOMPurify or a parser with raw-HTML disabled), allowlist link schemes (`http`/`https`/`mailto`), and add `rel="noopener noreferrer"` + `target="_blank"` to outbound links. Never `innerHTML` raw model markdown.
- **Buffer incomplete inline syntax.** While streaming, a lone `**` or `[text](` should render as literal text until completed, then upgrade — never flash broken layout. Many streaming markdown parsers (e.g. `streaming-markdown`, or a custom tokenizer) handle this; if using a batch parser, re-parse only on safe boundaries.
- **Defer fenced code blocks until the closing ` ``` `** — *or* open a code block as soon as the opening fence + language is seen and stream lines into it with a visible "streaming" indicator inside the block. Do not let an unclosed fence swallow the rest of the message into `<code>`.
- **Tables:** render only once a full row (or the full table) is parseable; a half-streamed `| a | b` row should not render a broken table. Make wide tables horizontally scrollable in a `<div role="region" tabindex="0" aria-label="…">` wrapper.

### Code blocks
- **Syntax-highlighted, with a header** showing the language label + a **copy button**. Copy is table-stakes.
- Use a lightweight highlighter (`shiki` for build-time/accurate, `highlight.js`/`prism` for runtime). Highlight on block completion to avoid re-tokenizing every line mid-stream.

```html
<figure class="code">
  <figcaption><span>python</span><button aria-label="Copy code">⧉ Copy</button></figcaption>
  <pre><code class="language-python">…</code></pre>
</figure>
```

### Citations & sources
- **Visually distinct from body text**, placed **adjacent to the specific claim** (Perplexity-style inline markers / Copilot-style clickable chips with hover previews), **clickable**, with **meaningful labels** (publication/article/domain + favicon) — never a generic "Source".
- Offer a **"See all sources"** affordance (footer list / carousel when multiple sources back one claim — see shadcn/ui Inline Citation).
- **Do not bury citations in the answer body.** And do not over-decorate: users rarely click them, but their presence raises trust — *which is exactly why they must be honest* (a citation that doesn't support the claim is worse than none).
- Avoid anthropomorphic framing; prefer factual "Based on: [Article Name]".

### Inline tool calls
- Render **inline step cards, visible by default**: tool name, input params (collapsible if large), status (running/done/error), and result summary. This prevents the black-box feeling and is core to *trust*, not debug-only. (Chainlit pioneered automatic "Searched database" / "Called weather API" cards.)
- **Visually distinguish tool cards from the model's own text** (border, muted background, monospace params).
- Tool calls and reasoning chunks **interleave** — chunks arrive before/between/after each other. Render each part **in arrival order** as its own block; don't try to collate them into one summary.

### Reasoning / thinking display
- **Collapsible "Thinking/Thought" accordion, hidden by default**, that **auto-opens while streaming** and **collapses when finished**. (See assistant-ui Chain of Thought, Vercel AI Elements Reasoning, Open WebUI's XML-tag detection.)
- Treat reasoning as **Layer 3** of progressive disclosure — available, not in the user's face. Show a duration/"Thought for 6s" summary on the collapsed header.

## 6. Agentic / two-pane layouts

Agentic chat breaks the simple back-and-forth: reasoning + tool calls read like a long internal monologue, the **original user message scrolls off-screen**, and users lose context.

- **Two-pane (chat + canvas/artifact) layout** separates *process* (left: the conversational thread + step cards) from *results* (right: the canvas — document, code file, table, preview). This keeps the deliverable stable while the agent works.
- **Pin the active goal/task** at the top so it survives long tool traces.
- **Collapse completed step groups** into a single summary line ("Ran 4 steps") to reclaim vertical space; expand on demand.
- Stream the canvas independently from the chat; the canvas is the durable artifact, the chat is the log.

## 7. Latency masking

Acknowledge input **instantly**, even when processing continues — immediate feedback turns passive waiting into active waiting, perceived as faster. Doherty Threshold: under 400ms keeps users in sync.

- **Optimistic echo:** render the user's message in the thread the *instant* they send (React `useOptimistic()`), before the server confirms. Reconcile to server truth on success; make any rollback **explicit**, never silent.
- **Typing indicator / "Thinking…":** show within ~100–300ms of send, before the first token. Replace it the moment the first token streams in.
- **Skeletons** for predictable, structured responses (a card, a table). Note: the "skeletons feel ~30% faster" claim is **contested** (Viget 2017 found skeletons performed *worst* for perceived duration). The robust finding is that **any** loading indicator beats a blank screen — pick per context, don't cargo-cult skeletons. Shimmer cycle ~300–700ms if used.
- **Skip the indicator entirely** when first-token latency is already <400ms — going straight to streamed text feels fastest.

## 8. Error, retry & rate-limit states

LLM API calls fail often enough that failure handling is **mandatory**, not optional — rate limits (429), timeouts on long generations, transient 5xx, and dropped streams are routine at any real volume. Treat the unhappy path as a first-class state, not an afterthought.

### Backoff
```js
async function withRetry(fn, { base = 1500, max = 6 } = {}) {
  for (let attempt = 0; attempt < max; attempt++) {
    try { return await fn(); }
    catch (err) {
      if (err.status >= 400 && err.status < 500 && err.status !== 429) throw err; // don't retry 4xx
      const retryAfter = err.headers?.get?.('retry-after');
      const wait = retryAfter
        ? Number(retryAfter) * 1000
        : Math.min(base * 2 ** attempt, 30_000) + Math.random() * 1000; // jitter
      if (attempt === max - 1) throw err;
      await new Promise(r => setTimeout(r, wait));
    }
  }
}
```
- **Exponential backoff *with jitter*** — ~1–2s base, double per attempt, ~5–8 max attempts. Jitter de-synchronizes clients that failed at the same instant, so they don't all retry on the same beat and re-collide (AWS, "Exponential Backoff And Jitter" — full jitter substantially reduces contention and completes work in fewer calls).
- **Honor `Retry-After` on HTTP 429** (Anthropic returns it). **Never retry 4xx** request errors — surface and fix. Throttle concurrency proactively to your account limit.
- **Never retry instantly** — an instant retry just re-hits the limit.

### Per-message idempotency
- Track each in-flight message by a **client-generated ID** so retrying one failed message doesn't spawn a duplicate or leave a stale failed bubble. (TanStack Query has a known multi-mutation per-message-retry pitfall — guard against double-fire.)
- Reconcile to server truth on success; rollbacks explicit.

### Error copy
- **Specific + actionable**, never "Something went wrong, try again later" or a spinner that spins forever.
- **Keep the partial streamed response visible** on error — don't discard what the user already read.
- Offer **next-best-actions**: Retry · Shorten input · Switch model · Continue manually. Preserve enough context that retry doesn't lose work.
- Distinguish error *classes*: rate limit ("Busy — retrying in 4s…"), context-length ("Conversation too long — start a new chat or remove an attachment"), content filter, network ("You're offline — message saved, will send when reconnected").
- **Mid-stream drop ≠ pre-first-token failure.** A connection that dies *after* tokens started can't safely auto-replay the whole call (you'd duplicate the partial). Mark the message **incomplete**, keep what arrived, and offer **Continue** (resume) or **Regenerate** (fresh) rather than a blind retry. A failure *before* the first token is safe to auto-retry via backoff.
- **System status (retry/reconnect/streaming) must be ephemeral** and structurally separate from the conversation — never persisted as a thread message.

## 9. Message actions

Reveal on hover/focus; always keyboard-reachable. Typical set per assistant message: **Copy · Regenerate · Thumbs up/down · Branch · (Edit on user messages)**.

- **Copy:** copies the rendered text (or raw markdown — offer both for code-heavy answers). Show a transient "Copied" confirmation.
- **Edit (user messages):** editing a past user turn **forks** the conversation — re-run from that point, preserving the old branch behind `‹ n/m ›` navigation. Don't silently destroy downstream turns.
- **Thumbs up/down:** feedback signal; on thumbs-down, optionally open a lightweight "what went wrong?" chip set. Keep it one tap to register, optional to elaborate.
- **Branch:** alternate continuations from any message; surface a compact variant switcher (`‹ 2/3 ›`) rather than a tree UI for most products.
- Each action needs a discernible `aria-label`; don't rely on icon-only buttons for screen readers.

## 10. Conversation history & sidebar

- **Left sidebar list** of conversations, newest first, grouped by recency ("Today / Yesterday / Previous 7 days"). Title each thread from its first user message (or model-generated title); make titles **editable**, threads **deletable** and **renameable**.
- **New chat** button is prominent and persistent (top of sidebar). `⌘K`/search across history for products with many threads.
- **Collapsible sidebar** (persist collapse state in `localStorage`); on mobile it becomes a drawer.
- **Pin/favorite** and **folders/projects** for heavy users; don't build this until volume justifies it.
- Active thread state unambiguous (filled bg or left accent bar, not color alone). See [SaaS Web Apps](./saas-web-apps.md) for shell/sidebar specs.

## 11. Trust & safety affordances

The danger: fluent, polished output **never looks broken** even when wrong. We spent decades training users to distrust broken-*looking* software; AI defeats that instinct (the "calculator analogy"). Design for **trust calibration**, not maximum trust — fluency is not accuracy, and hallucinations arrive in the same confident prose as correct answers, which is precisely why "AI-can-be-wrong" affordances matter.

- **Disclaimer near the input**, not in a footer or behind a help icon. Direct + paired with an action: *"Claude can make mistakes. Please double-check responses."* Avoid vague "AI-generated, for reference only." Surface it during onboarding.
- **Confidence / honesty over confidence theater.** Where the model is uncertain or relies on a source, *show the source* rather than asserting. Don't fake a confidence percentage you can't justify.
- **Avoid first-person anthropomorphism in trust-critical copy** ("I thought carefully about your problem", "I'm sure that…"). It inflates trust and sets unrealistic capability expectations. Use neutral, factual framing.
- **Citations must be honest** (§5) — the trust they buy is forfeit if they don't support the claim.
- **Make limitations legible**: knowledge cutoff, no live web access (if true), no memory across chats (if true). State them where the user will hit the limit.

## 12. Accessibility of a live-updating transcript

The hardest a11y problem here: a region that changes hundreds of times per response. The biggest pitfall is **updating the live region too frequently**.

```html
<div role="log" aria-live="polite" aria-atomic="false" aria-busy="true">
  <!-- message parts appended here -->
  <p>The answer streams in here<span class="cursor" aria-hidden="true">▍</span></p>
</div>
```

- **Container:** `role="log"` + `aria-live="polite"` + `aria-atomic="false"` so only **new** tokens are announced, not the whole message re-read each update.
- **`aria-busy="true"` while streaming**, clear to `false` when done.
- **`aria-hidden="true"`** on the visual cursor so it isn't announced.
- **NEVER `aria-live="assertive"` for streaming** — it clears the speech queue on every token, producing garbled, interrupted speech.
- **Debounce announcements.** Don't announce per chunk — batch into chunks of a few seconds (e.g. push the latest sentence to the live region every ~1.5–3s). Per-token announcement is overwhelming and unusable.
- **Live region must already exist in the DOM and be empty before injecting.** If you add it dynamically, **wait briefly (~2s is the common rule of thumb)** for the accessibility API to register it, or screen readers miss the first update — a practical SR/AT-registration workaround, not a spec requirement. The underlying pattern — announcing status *without moving focus* — is what **WCAG SC 4.1.3 Status Messages (AA)** requires.
- **Focus management:** on **Stop**, move focus to the **Retry** button (`.focus()`). On error, move focus to the actionable control. After send, keep focus in the composer so the user can keep typing.
- **Test on real screen readers** — NVDA + JAWS (Windows), VoiceOver (Mac/iOS). `aria-live` support is "disappointingly inconsistent"; the DevTools a11y tree alone is **not** sufficient.
- Composer popups (`/`, `@`) follow combobox/listbox ARIA; respect `prefers-reduced-motion` for the streaming cursor and skeleton shimmer. See [Accessibility (WCAG 2.2)](../01-ux-fundamentals/accessibility-wcag.md).

## Concrete specs & numbers
- **Doherty Threshold = 400ms** — below this users stay "in sync" and engaged; past it they perceive delay (IBM, Doherty & Thadani, IBM Systems Journal, 1982). Optimize first-token latency toward this.
- **Perceived latency ≈ first-token time, not total generation.** Streaming at ~40 tok/s makes a multi-second answer *feel* like its ~hundreds-of-ms first token; the user reads and can interrupt while the rest streams. (Mechanism, not a benchmarked percentage.)
- **60px** = auto-scroll gap threshold: `gap = scrollHeight - scrollTop - clientHeight`; treat user as scrolled-away when `gap > 60`.
- **~16.7ms/frame (~60fps)** — batch DOM writes per `requestAnimationFrame`; tokens can arrive faster.
- **~2 seconds** = common rule-of-thumb wait after *dynamically* adding an `aria-live` region before injecting text (SR/AT registration workaround, not a spec value). Better: ship the region in static markup, empty.
- **Failure handling is mandatory**, not a rare-path nicety — 429s, timeouts, transient 5xx, and dropped streams are routine at volume. (No reliable universal failure-rate figure; measure yours.)
- **Backoff:** ~1–2s base, double per attempt, ~5–8 max attempts, with jitter; honor `Retry-After` on 429; jitter de-synchronizes simultaneous retries to avoid re-collision (AWS, "Exponential Backoff And Jitter").
- **Skeleton shimmer cycle:** ~300–700ms. "Skeletons feel ~30% faster" is **directional and contested** (Viget 2017 found them worst) — treat as not definitive; *any* indicator beats blank.
- **3–4 curated prompts** in the empty state; **one main idea per empty state**.
- **Reading column** ~`65–75ch` for assistant prose; composer caps at ~6–8 rows.
- **NN/g** generative-AI UX research: chatbot conversations cluster into several distinct *types* (not one shape), and there is no single optimal conversation length — design for varied intents, not an idealized linear Q&A. (See NN/g AI topic page; treat specific percentages from press coverage as unverified — link in Sources.)
- **WCAG SC 4.1.3 (Status Messages, Level AA)** backs programmatically-determinable status without moving focus.

## Tools & libraries
- **Vercel AI SDK / AI Elements** (`elements.ai-sdk.dev`) — streaming primitives + prebuilt Reasoning component and chat parts for Next.js.
- **assistant-ui** (`assistant-ui.com`) — React chat UI; Chain of Thought groups consecutive parts into a collapsible thinking accordion; single-block vs discrete labeled steps.
- **shadcn/ui AI blocks** (`shadcn.io/ai`) — Inline Citation (domain + favicon markers, hover details, multi-source carousel, blockquote styling).
- **Chainlit** — automatic intermediate-step cards ("Searched database", "Called weather API") for inline tool-call visualization.
- **Open WebUI** — detects XML-like tags in the stream and renders a collapsible Thought/Thinking element.
- **MUI X React Chat** — Reasoning component for AI/agent surfaces.
- **React:** `useOptimistic()` for optimistic echo; `setState(prev => prev + token)` for token accumulation; TanStack Query for mutations (mind the multi-mutation per-message-retry pitfall).
- **Markdown/code:** a streaming-aware markdown parser; `shiki` (accurate) or `highlight.js`/`prism` (runtime) for code; highlight on block completion. **Sanitize** rendered output with DOMPurify (or a parser with raw HTML disabled).
- **Screen readers:** NVDA + JAWS (Windows), VoiceOver (Mac/iOS) — test on real ATs, not just the a11y tree.
- **Pattern libraries:** ShapeofAI.com (References pattern), patterns.dev AI UI Patterns.
- **OpenAI Codex CLI** — reference for slash commands (filter-as-you-type, inline args, Tab-to-queue, catalog-driven availability).

## Do / Don't
- **Do** stream by default. **Don't** stream when only the final structured result matters (JSON/SQL/lookup/short summary) — it adds complexity with no payoff.
- **Do** make Stop one of the most visible controls and Tab-reachable. **Don't** bury it in a kebab menu.
- **Do** batch DOM writes per RAF and extend text nodes. **Don't** re-render the whole message (`innerHTML = …`) on every token.
- **Do** buffer incomplete markdown and defer fence parsing. **Don't** let a half-open `**` or unclosed ` ``` ` break layout mid-stream.
- **Do** sanitize model markdown (DOMPurify, allowlisted link schemes) before rendering. **Don't** `innerHTML` raw LLM/tool output — it's untrusted and prompt-injectable.
- **Do** treat the committing-Enter of an IME (`isComposing`) as newline, not send. **Don't** fire send mid-composition for CJK users.
- **Do** `aria-live="polite"` + `aria-atomic="false"` + debounced announcements. **Don't** use `assertive` or announce per token.
- **Do** keep partial streamed text visible on stop/error. **Don't** discard what the user already read.
- **Do** seed the empty state with 3–4 distinct-capability prompts. **Don't** show a blank box, and **don't** cram prompts + commands + tips + illustration together.
- **Do** retry with exponential backoff + jitter and honor `Retry-After`. **Don't** retry instantly or retry 4xx.
- **Do** place citations adjacent to the claim with meaningful labels. **Don't** label them "Source" or bury them in the body.
- **Do** put the disclaimer near the input with a direct action. **Don't** hide it in a footer or use vague "for reference only."
- **Do** keep system status (retry/reconnect) ephemeral and separate. **Don't** persist it into the permanent thread.
- **Do** auto-scroll only at bottom with a 60px threshold. **Don't** yank the viewport down while the user is reading scrollback.

## Common pitfalls & how to avoid them
- **Layout thrash from per-token re-render.** → RAF-batch, extend text nodes, new `<p>` only on `\n`, re-parse markdown on boundaries.
- **Auto-scroll fighting the user.** → Only scroll when at bottom (`gap > 60` = scrolled away); reset the flag per new stream.
- **Broken layout from partial markdown** (runaway code fence eating the message). → Defer fence rendering to the closing ` ``` ` or render progressively with an in-block indicator.
- **XSS / prompt-injection via rendered output.** Model emits `<script>`, `onerror`, or a `javascript:` link and you render it raw. → Sanitize (DOMPurify / raw-HTML-off parser), allowlist link schemes, `rel="noopener noreferrer"` on outbound links.
- **CJK users sending half-typed messages.** Enter that commits an IME candidate fires send. → Bail when `e.isComposing` (or `keyCode === 229`).
- **Screen reader chaos** from `assertive` or per-token announcement. → `polite` + `atomic=false` + debounce to a few-second cadence; live region pre-exists and empty.
- **Missed first announcement** because the live region was added then immediately written. → Wait ~2s after dynamic insertion.
- **Duplicate messages on retry.** → Client-generated message IDs for idempotency; reconcile to server truth.
- **Retry storms / hammering a 429.** → Backoff + jitter, honor `Retry-After`, never instant-retry, never retry 4xx.
- **Stop button hidden or unfocusable.** → Prominent, Tab-reachable, focuses Retry afterward.
- **System chatter polluting the thread.** → Render retry/reconnect status in an ephemeral, structurally separate region.
- **Trust inflation from polished-but-wrong output.** → Honest citations, neutral non-anthropomorphic copy, legible limitations, near-input disclaimer.
- **Original question scrolls off in agentic runs.** → Pin the goal, collapse completed step groups, consider a two-pane canvas.
- **Composer locked during streaming.** → Let users compose/queue the next message while a response streams.

## What real users complain about
- **"It froze / nothing happened."** No instant acknowledgment of send → add optimistic echo + typing indicator within ~300ms.
- **"It scrolled away from what I was reading."** Aggressive auto-scroll during scrollback → bottom-only auto-scroll.
- **"I couldn't stop it wasting time/tokens."** Hidden or laggy Stop → prominent, immediate-abort Stop.
- **"The code block broke / merged into the rest of the answer."** Unclosed-fence rendering → defer/progressive fence handling.
- **"I lost my answer when it errored."** Partial response discarded on error → keep it visible.
- **"Spinner spun forever."** No timeout/error path → bounded retries + actionable error copy.
- **"It confidently made something up."** Trust inflation → honest citations + limitation disclosures + non-anthropomorphic copy.
- **"It announced every single letter" / "my screen reader read the whole message again each time."** `assertive` / `atomic=true` / per-token → polite, non-atomic, debounced.
- **"Editing my old message deleted everything after it."** Destructive edit → fork + branch navigation.
- **"Blank box, no idea what to ask."** Missing first-run scaffolding → curated example prompts.

## Senior-level checklist
- [ ] Thread layout has unambiguous ownership (≥2 cues, not color alone); assistant prose in a stable ~65–75ch column, not a narrow bubble.
- [ ] Streaming on by default; explicitly skipped only for final-result-only responses (JSON/SQL/lookup/short summary).
- [ ] DOM writes RAF-batched via a single queued flag; text nodes extended (new `<p>` only on `\n`); markdown re-parsed on boundaries — no per-token `innerHTML` rebuild.
- [ ] Auto-scroll bottom-only with 60px threshold; `userScrolled` reset per new stream.
- [ ] Stop button prominent + Tab-reachable; on stop flushes buffer, clears cursor, resets RAF flag, shows "stopped", focuses Retry, keeps partial text.
- [ ] Regenerate produces a new branch (`‹ n/m ›`); Continue resumes truncated output; both don't destroy prior variants.
- [ ] Composer: auto-grow (cap 6–8 rows), `Enter` send / `Shift+Enter` newline (touch has Send button), IME-composition Enter does **not** send, disabled-when-empty, typing allowed during stream.
- [ ] `/` and `@` open a `role="listbox"` popup with inline args, context-aware visibility, and Tab-to-queue; attachments validate type/size pre-upload with per-file chips.
- [ ] Empty state: personalized greeting + 3–4 distinct-capability one-click prompts; one idea per state; disclaimer present.
- [ ] Model markdown sanitized (DOMPurify / link-scheme allowlist) before render; partial markdown buffered; fences deferred/progressive; tables render only when row-parseable and are scroll-wrapped; code blocks have language header + copy.
- [ ] Citations adjacent to claims, clickable, meaningfully labeled, "See all sources"; honest (support the claim); not buried.
- [ ] Tool calls = inline step cards visible by default, distinct from model text; reasoning = collapsible accordion, auto-open streaming / collapsed done; parts render in arrival order.
- [ ] Latency masking: optimistic echo (`useOptimistic`), typing indicator <300ms, skeletons only where they help; no indicator when first-token <400ms.
- [ ] Errors: backoff + jitter, honor `Retry-After`, no instant/4xx retry; per-message client IDs for idempotency; partial text kept; specific copy + next-best-actions; error class distinguished; mid-stream drop offers Continue/Regenerate (not blind auto-replay), pre-first-token failure auto-retries.
- [ ] System status (retry/reconnect/streaming) ephemeral + structurally separated from the permanent thread.
- [ ] Message actions (copy/edit/regen/thumbs/branch) keyboard-reachable with `aria-label`; user-message edit forks the conversation.
- [ ] History sidebar: recency-grouped, editable/deletable/renamable titles, persistent New Chat, collapse persisted, drawer on mobile.
- [ ] Trust: near-input disclaimer with action, non-anthropomorphic trust-critical copy, legible limitations, calibration over maximum trust.
- [ ] Transcript a11y: `role="log"` + `aria-live="polite"` + `aria-atomic="false"`, `aria-busy` toggled, cursor `aria-hidden`, debounced announcements, live region pre-exists empty (~2s after dynamic insert), focus moved on stop/error; verified on NVDA/JAWS/VoiceOver, `prefers-reduced-motion` honored.

## Sources

> **Note on benchmarks:** the loading/perception figures here (e.g. the "skeletons feel ~30% faster" claim) are directional and in some cases contested (Viget 2017 found skeletons *worst* for perceived duration). Treat as calibration, not law; measure your own product. Where a hard percentage couldn't be sourced (perceived-latency reduction, failure rate, backoff-storm reduction), the mechanism is stated instead of a number.

- Nielsen Norman Group — generative-AI UX research, conversation-type analysis, chatbot usability: https://www.nngroup.com/topic/artificial-intelligence/
- IBM / Doherty Threshold (Laws of UX summary): https://lawsofux.com/doherty-threshold/
- WCAG 2.2 — SC 4.1.3 Status Messages (W3C): https://www.w3.org/TR/WCAG22/#status-messages
- WAI-ARIA `aria-live` regions (MDN): https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions
- Vercel AI SDK / AI Elements: https://ai-sdk.dev · https://elements.ai-sdk.dev
- assistant-ui: https://www.assistant-ui.com
- shadcn/ui: https://ui.shadcn.com
- Chainlit: https://chainlit.io
- Open WebUI: https://github.com/open-webui/open-webui
- MUI X: https://mui.com/x/
- React `useOptimistic`: https://react.dev/reference/react/useOptimistic
- TanStack Query: https://tanstack.com/query
- shiki: https://shiki.style · highlight.js: https://highlightjs.org · Prism: https://prismjs.com
- DOMPurify (HTML sanitizer): https://github.com/cure53/DOMPurify
- Anthropic API — rate limits & `Retry-After` / error handling: https://docs.anthropic.com/en/api/rate-limits · https://docs.anthropic.com/en/api/errors
- AWS — "Exponential Backoff And Jitter": https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
- patterns.dev: https://www.patterns.dev · ShapeofAI: https://www.shapeof.ai

# User Research & Usability Testing

> Lightweight research that makes design decisions defensible instead of guessed. Covers when to interview, survey, test, A/B, or read analytics; how many users each method needs; and how to run them without the classic statistical and recruiting traps. The senior outcome: you pick the cheapest method that answers the actual question, run it correctly, and turn raw sessions into design changes — not a deck that gathers dust.

**Apply when:** before building (discovery), during design (validation), and post-launch (optimization); any time a decision rests on an assumption about users · **Related:** [Nielsen's Heuristics](./nielsen-heuristics.md) · [Information Architecture & Navigation](./information-architecture-navigation.md) · [Conversion Rate Optimization](../06-marketing-and-conversion/conversion-rate-optimization.md) · [What Real Users Complain About](../08-pitfalls-and-antipatterns/real-user-complaints.md)

## TL;DR — rules at a glance
- **5 users find ~85% of usability problems** for qualitative testing (Virzi 1992; popularized by Nielsen). Formula: `problems_found = N × (1 − 0.31)^n`, where each user surfaces ~31% of issues. Stop at 5; iterate instead of piling on.
- **3 rounds of 5 > 1 round of 15.** Each round fixes issues, validates fixes, and exposes new structural problems. Same coverage, faster cycles.
- **Never report percentages from n=5.** "80% completed" with 5 users is statistical noise. Quant claims need **30+ participants (40+ ideal)**.
- **Think-aloud is the default qual method** — Nielsen Norman Group calls it the #1 usability tool. It reveals the *why* behind actions.
- **Card sort first (generative), tree test after (evaluative).** Card sort discovers groupings; tree test validates findability. Run both, in that order.
- **JTBD for strategy, personas for empathy.** JTBD = "When [situation], I want to [job], so I can [outcome]." Personas = who. Neither replaces the other.
- **A/B defaults: α = 0.05 (95%), power = 0.80.** Pre-compute sample size from baseline + MDE + power. Peeking and early-stopping is p-hacking.
- **Moderated remote ≈ in-lab** for issues found and task time. Unmoderated is faster/cheaper but can't probe and has 5–8% dropout.
- **Design for distracted, tired, multitasking users** — not the lab "ideal" user. Lab sessions are optimal-condition theater.
- **Observe behavior, never ask intent.** "Would you use this?" is worthless. Watch what they actually do.
- **Synthetic/AI participants do NOT replace real users** — AI-simulated responses fail to replicate real behavior variance, emotional reactions, and unexpected paths. Use AI to *analyze* transcripts, not to *be* the user.
- **Planning is 80% of qual analysis.** A discussion guide mapped to clear research questions makes synthesis trivial. Unstructured transcripts are a synthesis nightmare.
- **Combine quant + qual.** Analytics/heatmaps tell you *what* happened; interviews/think-aloud tell you *why*. Either alone misleads.

## Method selection — pick by question type

Match the question to the cheapest method that answers it. Most teams over-use surveys and under-use 5-user tests.

| Question you're answering | Method | Type | n needed |
|---|---|---|---|
| "Why do users abandon checkout?" | Usability test (think-aloud) | Qual / evaluative | 5 per group |
| "What do users actually want to accomplish?" | JTBD / discovery interviews | Qual / generative | 5–8 |
| "How do users mentally group our content?" | Card sorting | Qual / generative | 15–30 |
| "Can users find X in our nav?" | Tree testing | Quant / evaluative | 50+ |
| "Which of two layouts converts better?" | A/B test | Quant / evaluative | computed from MDE |
| "Where do users drop in the funnel?" | Analytics funnel | Quant / behavioral | thousands |
| "What's causing friction on this screen?" | Session replay + heatmaps | Quant→qual / behavioral | 100s of sessions |
| "How satisfied / what do they say?" | Survey (NPS/CSAT/custom) | Quant (+ open-text qual) | 100+ for stats; 200+ for segment splits |
| "What happens in real life over days?" | Diary study / ethnography | Qual / contextual | 8–15 |

**Rule:** generative methods (interviews, card sort, diary) come *before* design; evaluative methods (usability test, tree test, A/B) come *after* a design exists to evaluate.

**First-click testing** is a fast evaluative method that sits between card sort and full usability test: show a wireframe, ask users where they'd click first for a task. NNG research shows first-click accuracy predicts overall task success at roughly 87%; a failed first click means only ~46% task completion. Run with 50+ users via Chalkmark or Maze. Use it before committing to a full nav build.

## Qualitative usability testing

The single highest-leverage method. You watch real users attempt real tasks on your real interface and observe where they fail.

### The 5-user rule (and its limits)
- Model: `N × (1 − L)^n` where `N` = total problems, `L` = probability one user hits any given issue, `n` = number of users.
- At typical `L = 0.31`: **5 users → ~85% of problems**. The 1st user alone reveals ~31%; the 5th adds little.
- `L` is not fixed. **If the interface is already polished** (low error rate, `L ≈ 0.10`), you need **~18 users** to reach 85%. At `L = 0.20` you need 9. At `L = 0.31` you need 5.
- **Multiple distinct user groups** (e.g., buyers vs. sellers): test each group separately, **3–4 per group minimum** (2 groups → 6–8 total). Behaviors diverge; pooling hides group-specific failures.
- **Hard rule:** these are *qualitative* sessions. Do not compute task-success rates or time-on-task as population statistics from 5 people. The confidence intervals are enormous.

### Think-aloud protocol
Have the user narrate their thoughts continuously while attempting tasks. This is the best technique for exposing mental models.
- **Concurrent think-aloud:** narrate while doing. Default. Catches in-the-moment confusion.
- **Retrospective think-aloud:** replay a recording and narrate after. Use when concurrent narration interferes with a timed or high-load task.
- 5 think-aloud participants reveal **~85%** of usability problems per the Virzi/Nielsen model; run 5–8 for most major issues with confidence.

**Neutral moderator probes (memorize these):**
- "What are you trying to do right now?"
- "What do you expect to happen when you click that?"
- "What were you thinking right there?"
- "Walk me through what you're seeing."

**Never say (leading / biasing):**
- "Did you find that confusing?" → primes a yes.
- "Isn't that button hard to see?" → answers for them.
- "Would you use this feature?" → intent ≠ behavior; notoriously inaccurate.

### Task design
- Give **realistic goal-based tasks**, not click instructions. Good: "You want to return a pair of shoes you bought last week — do it." Bad: "Click the Returns link in the footer."
- **Don't reveal the path** in the task wording (no feature names that double as nav labels).
- Order tasks roughly by the real user journey; randomize order across users if tasks are independent and you suspect order effects.
- **Pilot the task set with 1 person first.** A confusing task wording wastes every subsequent session.

### Moderated vs. unmoderated
- **Moderated remote** (Zoom/Lookback/UserZoom): you watch live, ask follow-ups. Research consensus: **virtually equivalent to in-lab** in issues found and completion time. Use when the *why* matters or flows are complex/novel.
- **Unmoderated** (Maze/UserTesting): users complete tasks alone on their own time. Faster, cheaper, larger n. Cannot probe in real time; higher dropout. Use for simple, well-understood flows and broad coverage.
- **Unmoderated constraints:** limit to **7–8 tasks max** (cognitive overload kills data quality). Include a **welcome screen explaining think-aloud**. Expect **5–8% no-show/unusable** sessions — over-recruit.

## Quantitative usability testing
- Use when you need **defensible metrics**: task success rate, time-on-task, error rate, SUS scores, benchmark comparisons.
- **Minimum 30 participants, ideally 40+** for confidence intervals narrow enough to act on.
- Typical metrics: completion rate (binary per task), time-on-task (log-normal distribution because confused users skew high — report **median**, not mean; mean lies), and standardized post-test (SUS: 10 items, score 0–100; **68 is average, 80+ is "good," 90+ is "excellent"**). Time-on-task is only meaningful when there is a target time or benchmark; don't collect it unless you plan to compare.
- **NPS / CSAT** are attitude metrics, not usability metrics. NPS (0–10 likelihood-to-recommend, detractors ≤6, passives 7–8, promoters 9–10) and CSAT (direct satisfaction rating) tell you *sentiment trend* — they do not diagnose problems. Pair with open-ended follow-up: "What's the main reason for your score?"
- Don't moderate heavily here — you want clean, comparable conditions. Unmoderated platforms shine for this scale.

## Discovery interviews & Jobs-to-Be-Done

### Interviews (generative)
- Goal: understand goals, mental models, context, and pain — not opinions on your UI.
- **"You are the instrument."** Your phrasing and reactions shape answers. After each session, spend **15 minutes** noting your own biases/assumptions before they contaminate synthesis.
- **Open, non-leading questions.** "Tell me about the last time you…" beats "Do you like X?"
- **For B2B:** figure out *who you're actually talking to* (role, authority) within the **first 30 seconds**. Behaviors differ wildly by role. Never let account managers proxy for real users — it's "playing telephone."

### Jobs-to-Be-Done (JTBD)
Focus on the **outcome and motivation**, solution-agnostic.

**Job statement format:**
```
When [situation], I want to [motivation/job],
so I can [expected outcome] without [pain/obstacle].
```

**Job hierarchy — design at the right altitude:**
```
Aspiration / Goal  →  Big Job   →  Little Job   →  Micro Job
   Be healthy      →  Exercise  →  Go hiking    →  Find hiking boots
```
Designing for "find hiking boots" when the user's real job is "go hiking" produces a narrow tool that misses adjacent needs.

**JTBD interview = "switching moment" interviews.** Reconstruct the timeline of when someone switched from an old solution to a new one. Need **5–8 switching interviews** to find recurring job patterns.

**Solution-agnostic JTBD probes:**
- "What other solutions did you consider?"
- "What would you do if this product weren't available?"
- "When did you first realize you needed something?"
- "What were you using before, and what made you switch?"

### JTBD vs. personas
- **JTBD** answers *why* and *what for*; it is **persona-agnostic** (a "find hiking boots" job spans many demographics). Use it for **strategic direction**.
- **Personas** answer *what patterns of people exist*; useful for **team empathy during execution**. They do **not** explain motivation.
- Use both: JTBD to decide *what to build*, personas to keep the team aligned on *who it's for*.

## Surveys — structured quant with a qual escape valve

Surveys are fast and scale easily, which is exactly why they are over-used and badly designed. They answer *what* (distribution, frequency, stated attitude) not *why*.

### When surveys belong
- Post-launch satisfaction measurement (CSAT, NPS, SUS).
- Validating a hypothesis from qual at scale ("you said users don't trust the checkout — how many?").
- Screener for recruiting (pick research participants from a broad pool).
- **Not:** exploring unknown problems, discovering jobs-to-be-done, or explaining a drop in a funnel. Use interviews or session replay for those.

### Question design rules
- **Closed questions for quant; open-ended for qual.** Keep ratio at ≈80% closed / 20% open.
- **One idea per question.** "How easy and fast was checkout?" — invalid (asks two things).
- **Symmetric rating scales.** Use balanced 5- or 7-point Likert (from "Strongly disagree" to "Strongly agree"). Never use scales where the neutral midpoint is ambiguous.
- **Avoid loaded/leading.** "How helpful was our amazing new feature?" → "How would you rate [feature]?"
- **Avoid double negatives.** "I would not hesitate to use this again" — confusing for anything but native speakers.
- **Order effect:** put high-stakes rating questions before open-ended (respondent fatigue hits open-text first); put demographics last.
- **Completion rate benchmark:** surveys under 5 minutes average ~70–80% completion; over 10 minutes drops below 50%. Keep it under 10 questions for intercept surveys.

### n rules
- **100+ for top-line percentages.** Margin of error at 95% CI for n=100 is ±10pp — barely actionable. At n=400 it drops to ±5pp.
- **200+ to split by segment** (e.g., mobile vs. desktop users). Smaller segments compound error.
- **Never report n<30 as percentages.** "75% of users prefer X" from n=20 is noise presented as signal.
- **Pilot the survey** with 3–5 people before sending. Question confusion is invisible to the author.

### Screener design (critical for recruiting)
A bad screener admits wrong participants and invalidates all downstream research. Rules:
- Include at least one **behavior-based** question ("How often do you purchase groceries online?"), not just demographic ("Are you 25–44?").
- Use **disqualifying answers** explicitly — if your product is for active buyers, a "never" answer should screen out.
- **Don't telegraph the target answer.** "We're looking for frequent shoppers — do you shop frequently?" → everyone self-selects in. Use neutral wording.
- For unmoderated tests, add a **free-text articulation check**: "Describe what you do when you need to return an online purchase." One or two words = low-quality respondent.

## Personas — done right or not at all
- **Demographic-only personas are useless.** "Marketing Mary, 35, likes yoga" steers nothing. Meaningful personas carry **goals, mental models, frustrations, and behaviors grounded in real interviews.**
- **Personas require fieldwork.** A 2-week, no-research persona is almost always worthless — and worse, the polished PDF creates a **false sense of "research done"** that *blocks* further inquiry while decisions quietly revert to the HiPPO (Highest Paid Person's Opinion).
- **Job-infused persona format** (bridges persona + JTBD):
  ```
  [Persona] hires this product when [situation]
  to gain [outcome] and stay [emotional benefit].
  ```
- Build 3–5 personas max from interview clusters; tie each to a primary job.

## Card sorting & tree testing (IA validation)
Use in sequence — card sort to *generate* the structure, tree test to *validate* it. See [Information Architecture & Navigation](./information-architecture-navigation.md).

### Card sorting (generative, early)
- Users group content cards into categories that make sense to them. Reveals **mental groupings** and the words users use.
- **Open card sort:** users create and name their own groups → discover categories. **Closed card sort:** users sort into your predefined groups → validate categories. **Hybrid:** predefined groups + allow new ones.
- **n ≈ 15–30.** Participants often group the same content differently — results are **directional, not conclusive.**
- **Card sort does NOT design your IA.** It surfaces tendencies; a designer still makes the structural call, then a tree test confirms it.

### Tree testing (evaluative, after)
- Text-only, navigation-only test of a proposed hierarchy. Users are given a task ("Where would you find your billing history?") and click down a label-only tree — **no visual design, no search.**
- **n ≥ 50** for clear patterns. Session **15–20 min**, **fewer than 10 tasks**.
- Output pinpoints **exactly which tree node** fails (wrong first-click, backtracking, low directness). Because it's label-only, **it cannot catch copy, visual, or interaction issues** — that's a feature: it isolates the IA.
- **Pitfall:** skipping the tree test after a card sort and shipping the card-sort grouping as final IA.

## A/B testing (statistical, post-launch)
For data-driven decisions between variants. Detail in [Conversion Rate Optimization](../06-marketing-and-conversion/conversion-rate-optimization.md).

- **Defaults:** α = 0.05 (95% confidence), power = 0.80 (80%). Use **α = 0.01 (99%)** for **irreversible or high-stakes** decisions.
- **Pre-compute sample size** from three inputs before launch: baseline conversion rate, **MDE** (minimum detectable effect), and power. Launching without this is "driving at night without headlights."
- **MDE drives feasibility.** A **+0.1pp MDE on a 2% baseline requires millions of observations.** If you can't reach that traffic, the test cannot resolve — pick a bigger MDE or a different method.
- **Never peek and stop early at first significance** — that inflates false-positive rate dramatically (p-hacking). Define n up front and run to completion (or use a sequential-testing method designed for peeking).
- Run **full weeks** to absorb day-of-week effects; account for novelty effects on returning users.
- Platforms with built-in sample-size calculators: Optimizely, AB Tasty, Statsig, Convert.com, CXL.

## Analytics — behavioral, what (not why)
Quant signals at scale. Always pair with a qual method to explain the *why*.

### Funnels
- Map the conversion path as discrete steps; measure drop between each.
- **Find the single highest-drop step first.** Then watch **session replays of that exact step** to learn why — funnel says *where*, replay says *why*.

### Heatmaps
- **Click/tap maps:** where users click (and where they click dead zones expecting interactivity).
- **Scroll maps:** how far down users reach — exposes content below an unrecognized fold.
- **Move maps:** cursor movement as a rough attention proxy (desktop only; weak signal).
- **Attention/area maps:** aggregated engagement zones.
- Heatmaps are **directional**, sensitive to traffic volume and segment; don't over-read low-sample maps.

### Session replay
- Replays of real sessions. **Start with rage clicks and dead clicks as entry points** — they flag friction without watching full recordings.
- **Pitfall:** replay alone tells you *what* happened, not *why*. It complements — never replaces — talking to users.

## Designing for stress cases (not the ideal user)
Lab usability is **optimal-condition theater**. Real users:
- are distracted, tired, multitasking, half-asleep;
- hold a child with one hand, wear thick gloves that block capacitive touch, ride a bumpy bus;
- read a foreign-language UI, sit in a loud room, have low bandwidth.

**Situational impairments are real disability.** Being tired, distracted, or in a noisy room creates temporary impairment. Designing for **stress cases stress-tests the whole product** — robustness for the worst case lifts the average case too. Tie this directly to [Accessibility (WCAG 2.2)](./accessibility-wcag.md) and [Touch Ergonomics](../02-responsive-adaptive/touch-ergonomics-cross-device.md): large targets, forgiving inputs, high contrast, and recoverable errors are stress-case insurance.

## Diary studies & contextual inquiry (longitudinal / field)

Diary studies capture behavior in context over days or weeks — the only methods that reveal what users actually do vs. what they report doing in a lab.

- **When to use:** tasks that span time (shopping for a car, managing medications, using a new software tool over a week), products with infrequent triggers, or when the lab/interview setting artificially changes behavior.
- **n = 8–15** for qualitative patterns; 20+ if you want frequency counts to hold.
- **Participant burden is the main failure mode.** Entries drop off after day 3 unless the logging mechanism is low-friction (WhatsApp voice note, 2-question mobile form) and you send **daily reminder nudges**. Do not ask for paragraphs.
- **Prompt types:** event-contingent ("log each time you search for a product"), time-contingent ("nightly 3-question check-in"), signal-contingent (app trigger on a specific behavior).
- **Contextual inquiry** (observing users in their actual environment) is deeper but expensive. Reserve for high-stakes redesigns where you suspect lab settings mask the real use case.

## Synthesis & analysis
- **Plan first.** If the discussion guide maps cleanly to research questions, analysis is nearly automatic. **Organize by theme from the start** — un-themed transcripts become a synthesis nightmare.
- **Thematic / affinity analysis:** cluster observations into themes. The **"rainbow sheet"** (visual affinity, one color per participant) and **spreadsheet-based coding** are the lightweight standards for most product cycles — you rarely need NVivo/MAXQDA.
- **Analyze incrementally.** For 30–50 interviews, synthesize after **each batch of 6–10 sessions**, not all at the end. You'll spot saturation (no new themes) and can stop early.
- **AI/LLM caution:** feeding **raw transcripts to a general LLM causes hallucination** — it "mixes and matches" quotes to justify a conclusion. Use **dedicated research tools with a human review step** (Dovetail, Looppanel, AILYZE). Treat any AI quote as unverified until you check the source.

## Concrete specs & numbers
- **Qual usability:** ~85% problem discovery at **n=5**; model `N × (1 − 0.31)^n`.
- **1st user ≈ 31%** of all problems; diminishing returns by the 5th.
- **High-quality UI** (`L = 0.10`) → **18 users** for 85%. `L = 0.20` → **9 users**. `L = 0.31` → **5 users**.
- **Multiple user groups:** **3–4 per group** minimum (2 groups → 6–8 total).
- **Quant usability:** **30 minimum, 40+ ideal** for narrow confidence intervals.
- **Think-aloud:** 5 users reveal **~85%** of problems per Virzi/Nielsen model; 5–8 for most major issues.
- **Unmoderated tests:** **≤7–8 tasks**; expect **5–8% unusable/no-show** → over-recruit.
- **Tree testing:** **≥50 users**, **15–20 min**, **<10 tasks**.
- **Card sorting:** **~15–30 users**; results directional.
- **Card sort + tree test combined:** Optimal Workshop plan ≈ $208/mo; participant recruitment via User Interviews ≈ $40–80/person incentive — total cost varies by n and tool. Budget separately per project.
- **A/B:** α = 0.05 / power = 0.80 default; **α = 0.01** for irreversible; **+0.1pp MDE on 2% baseline → millions of observations**.
- **Session replay cost:** Microsoft Clarity and PostHog basic replay are free. Paid tiers (Hotjar, Mouseflow, FullStory) vary from ~$0 (limited free) to hundreds/mo at scale — check current pricing. Do not quote a per-session rate as a universal benchmark; free tools exist.
- **Microsoft Clarity:** free, unlimited recordings, **13-month retention** (extended from 30 days in 2023).
- **Matomo:** self-hostable (free), cloud paid tier — check current pricing at matomo.org (restructured 2024); privacy-first, GDPR-native, heatmaps + recording.
- **JTBD:** **5–8 switching interviews** to find recurring patterns.
- **First-click testing:** **50+ users** for signal; predicts overall task success at ~87% (failed first click → ~46% task completion per NNG research).
- **Surveys (quant):** **100+** for top-line percentages (±10pp at 95% CI); **200+** for segment splits; never report percentages from n<30.
- **NPS formula:** score = (% promoters 9–10) − (% detractors 0–6). Meaningful only as a trend; single data point is not actionable alone.

## Tools & libraries
**Heatmaps / session replay / behavioral**
- **Microsoft Clarity** — free, unlimited recordings, click/rage/dead-click detection, **13-month retention**, AI assistant (Copilot-powered). *Caveat:* Microsoft may use anonymized data for AI training — check DPA compliance for EU/GDPR contexts.
- **Hotjar** — all-in-one heatmaps, recordings, surveys (Contentsquare-owned); free tier. Good general default.
- **FullStory** — enterprise; auto-captures all interactions, flags rage clicks, AI friction detection.
- **Mouseflow** — 7 heatmap types, replay, funnel + form analytics, Mina AI natural-language session queries.
- **LogRocket** — developer-focused replay with console logs, network, error tracking. Best when engineers debug.
- **Matomo** — privacy-first, self-hostable; heatmaps + recording.

**Card sorting / tree testing / first-click**
- **Optimal Sort** (Optimal Workshop) — standard for card sorts.
- **Treejack** (Optimal Workshop) — standard for tree tests.
- **Chalkmark** (Optimal Workshop) — first-click testing.
- **Maze** — tree test + card sort + first-click with automated analysis; heatmaps for unmoderated tests.

**Moderated/unmoderated testing & recruitment**
- **UserZoom / Lookback.io** — moderated remote with integrated recording.
- **UserTesting** — recruitment + moderated/unmoderated video sessions.
- **User Interviews** — recruitment panel; hosts the UX Research Field Guide.

**Qual synthesis / repositories**
- **Dovetail** — repository + transcript coding, AI highlight suggestions (AI "not hitting the mark yet" per practitioners), showreels. Widely used.
- **Looppanel** — transcription + auto-theming + cross-project quote search.
- **AILYZE** — AI-assisted transcript coding + thematic/frequency analysis.
- **Survicate Insights Hub** — continuous research integrated with calls; structured executive summaries.
- **NVivo / MAXQDA / ATLAS.ti / Dedoose** — academic-grade coding; rigorous but **slow** — overkill for most product cycles.
- **Quirkos** — lighter qualitative coding.
- **CoLoop / Rutabaga.app / DoReveal.com** — newer AI-native analysis tools.
- **R (tm, tidytext, quanteda)** — code-based thematic analysis for the comfortable.
- **Gamma** — fast slides from insights.

**A/B platforms** — Optimizely, AB Tasty, Statsig, Convert.com, CXL (all with sample-size calculators).

**When NOT to reach for a tool:** don't buy NVivo for a 6-interview study (use a spreadsheet/rainbow sheet); don't bolt on session replay before you have a funnel showing *where* to look; don't pipe raw transcripts into a general chatbot.

## Do / Don't
**Do**
- Test with 5 users, then iterate; run 3 rounds, fixing between each.
- Pre-compute A/B sample size; run to completion.
- Use neutral probes; observe behavior over stated intent.
- Card sort first, tree test after; treat card sorts as directional.
- Pair every quant signal (funnel/heatmap) with a qual method for the *why*.
- Over-recruit for unmoderated; pilot every task set and survey first.
- Write screeners with behavioral disqualifiers, not just demographic filters.
- Keep surveys under 10 questions for intercept; pilot with 3–5 people first.
- Synthesize incrementally; theme as you go.
- Recruit *real* target users; verify B2B role in the first 30 seconds.

**Don't**
- Report percentages or time-on-task from n=5 as if they're population stats.
- Stop an A/B test early at first significance (peeking = p-hacking).
- Ask "Would you use this?" or any leading/intent question — intent ≠ behavior; always misleads.
- Build personas from demographics + hunches with no fieldwork.
- Write survey questions that ask two things at once or use leading/loaded framing.
- Ship card-sort groupings as final IA without a tree test.
- Feed raw transcripts to a general LLM and trust the quotes.
- Recruit only easy-to-reach users (employees, super-fans) or proxy users.
- Design only for the smooth, ideal, hero path.

## Common pitfalls & how to avoid them
- **Lab "theater":** clean results under ideal conditions. → Add stress-case tasks (mobile, distracted, one-handed, gloves) and field/diary methods.
- **False completion from polished personas:** team feels "done," research stops, decisions revert to HiPPO. → Keep personas evidence-linked; revisit each quarter; never let a PDF end discovery.
- **A/B peeking:** stopping at first significance inflates false positives. → Fix n up front; run full weeks; use sequential methods if you must monitor.
- **Quant-only or qual-only:** heatmaps/replay show *what*, never *why*. → Always combine; funnel → replay → interview.
- **LLM hallucinated quotes:** general models fabricate/mix quotes. → Dedicated tools + human review of every cited quote.
- **Card sort without tree test:** directional results shipped as conclusive. → Always follow with a tree test before locking IA.
- **Leading think-aloud probes:** "Did you find that confusing?" → neutral "What were you thinking right there?"
- **Demographic personas:** age/title/hobbies over goals/mental models/pain. → Ground every attribute in interview data.
- **Skipping pilots:** a confusing question discovered mid-study wastes all sessions. → Pilot guides and surveys with 1 person first.
- **n=5 as statistics:** "80% completed" with 5 users has huge CIs. → Report qualitatively; scale to 30+ for numbers.
- **Convenience sampling:** internal employees / super-fans behave unlike target users. → Recruit to a real screener with behavioral disqualifiers.
- **B2B telephone:** account managers as user proxies filter out the real pain. → Talk to actual end users.
- **Survey leading questions:** "How helpful was our feature?" primes a positive answer. → Neutral, single-idea questions; pilot with 3–5 people before sending.
- **n=100 survey reported as certainty:** ±10pp margin at 95% CI — barely actionable. → Report range, not point estimates; scale to 400+ for narrow CI.
- **NPS without follow-up:** a score alone tells you nothing actionable. → Always pair with "What's the main reason for your score?"

## What real users complain about
Synthesized from practitioner reports and research-community discussion — concrete failure modes a senior designer should anticipate:
- **"It worked in the demo, not on my phone on the train."** Optimal-condition designs collapse under real distraction, glare, and one-handed use.
- **"The survey already decided the answer."** Leading/loaded questions and intent-to-use questions produce confident, wrong data.
- **"Nobody read the personas."** Polished persona decks sit unused while teams default to opinion.
- **"The AI summary invented a quote I never said."** General LLMs mixing/fabricating transcript quotes erodes trust in synthesis.
- **"We A/B tested and called it after a day."** Early-stopped tests ship false winners that don't replicate.
- **"The card sort said one thing, the live nav said another."** Skipping the tree test let an unvalidated IA ship.
- **"They tested with their own employees."** Convenience samples miss how target users actually behave.
- **"I told the account manager, it never reached the team."** B2B insight lost to proxy filtering.
- **"We used our NPS score as proof people loved the feature."** NPS is a trend metric, not a validation of a specific design decision.
- **"We skipped the click test to save time — then spent 3 weeks reworking nav after launch."** First-click testing costs a day; post-launch nav rewrites cost months.

## Senior-level checklist
- [ ] The research question is written down and a method is chosen to *answer it* (not a default method by habit).
- [ ] Generative methods precede design; evaluative methods follow a concrete artifact.
- [ ] Usability tests use realistic goal-tasks that don't reveal the path; task set piloted with 1 user.
- [ ] Qual testing planned as **3 rounds of 5**, fixing between rounds — not one big study.
- [ ] No percentages/time-on-task reported from n<30; quant claims backed by 30–40+ participants.
- [ ] Think-aloud probes are neutral; no intent-to-use questions anywhere.
- [ ] IA work runs card sort → designer decision → tree test (≥50 users) before lock.
- [ ] A/B sample size pre-computed from baseline + MDE + power; MDE is feasible given traffic; test runs full weeks to completion; α raised to 0.01 for irreversible calls.
- [ ] Every funnel/heatmap finding is explained by a paired qual method.
- [ ] Personas (if used) are interview-grounded with goals/mental models/pain, and tied to JTBD job statements.
- [ ] Stress cases (distracted, mobile, gloved, low-bandwidth, fatigued) explicitly tested or designed for.
- [ ] Synthesis themed from the start, done incrementally; any AI-extracted quote verified against the source.
- [ ] Recruits are real target users (B2B role verified early), not employees/super-fans/proxies.
- [ ] Screeners use behavioral disqualifiers, not only demographics; free-text articulation check for unmoderated tests.
- [ ] Surveys: n≥100 before reporting percentages; one idea per question; symmetric scale; piloted with 3–5 people first.
- [ ] NPS/CSAT treated as trend indicators, not diagnostic tools; open-ended follow-up captures the why.
- [ ] First-click test run before committing nav/IA build (n≥50).

## Sources
- Nielsen Norman Group — "Why You Only Need to Test with 5 Users": https://www.nngroup.com/articles/why-you-only-need-to-test-with-5-users/
- Nielsen Norman Group — "How Many Participants for Quantitative Usability Studies": https://www.nngroup.com/articles/quant-usability-test-participants/
- Nielsen Norman Group — "Thinking Aloud: The #1 Usability Tool": https://www.nngroup.com/articles/thinking-aloud-the-1-usability-tool/
- McDonald, S., Edwards, H. M., & Zhao, T. (2012). "Exploring Think-Alouds in Usability Testing." *IEEE Transactions on Professional Communication.* (Examines think-aloud protocol variants and their effect on data quality.)
- Nielsen Norman Group — "First-Click Testing": https://www.nngroup.com/articles/first-click-testing/
- Nielsen Norman Group — "Card Sorting: Uncover Users' Mental Models": https://www.nngroup.com/articles/card-sorting-definition/
- Virzi, R. A. (1992). "Refining the Test Phase of Usability Evaluation: How Many Subjects Is Enough?" *Human Factors, 34*(4), 457–468. (Original empirical basis for the L≈0.31 / 5-user model.)
- Optimal Workshop — Tree testing (Treejack): https://www.optimalworkshop.com/treejack/
- Microsoft Clarity: https://clarity.microsoft.com/
- Matomo: https://matomo.org/

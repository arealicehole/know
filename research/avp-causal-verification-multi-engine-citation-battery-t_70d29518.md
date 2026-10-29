# AVP Causal Verification — Multi-Engine Citation Battery + Signal Ablation

**Task:** t_70d29518 (board: research, prefix [GIU])
**Prepared:** 2026-10-29 (UTC) — all live probes re-verified this run
**Prepared by:** Geraldo v2.2
**Parent pattern extract:** t_64b6e1c1 (`/home/ice/know/research/ai-readable-web-pj-maza-competitor-audit-t_64b6e1c1.md`)
**Bible baseline:** SEO_BIBLE_JUNE_2026.md v2.1
**Live references (re-probed today):** printjunkieaz.com, mazahalalfood.com

---

## EXEC SUMMARY (for Frank — read this first)

**The honest answer to "is the agent-visibility stack *why* ChatGPT recommends PJ/Maza?" is: we cannot claim causation yet, and nobody can — not us, not any vendor. What we CAN do, and what this report delivers, is a reproducible measurement protocol that turns the assumption into a testable claim.**

1. **We cannot A/B the model itself.** The LLMs behind ChatGPT/Claude/Gemini/Perplexity are not ours; we cannot control their weights, their index, or which URL they cite. So "the stack caused the citation" is *not* directly provable the way a landing-page A/B is provable. What we can prove is **correlation + counterfactual consistency**, and we can do it honestly.

2. **The strongest, cheapest, most honest verification is a *multi-engine citation battery***: the **same** fixed prompts, run against **all five** engines (ChatGPT, Claude, Gemini, Perplexity, Grok), with structured capture (cited Y/N, which facts quoted, which link, any wrong competitor). Run it **now** (baseline) and **after** each stack change. If the citation appears across *multiple independent* engines after we add a signal and disappears when we remove it, that is the closest thing to a causal claim we can ethically and technically make. This report gives you the exact prompt battery and the capture schema.

3. **For "which signal matters most," the correct method is *ablation on a staging preview, not prod*.** We do **not** touch live PJ/Maza (no Frank approval, no prod break). We stand up controlled variants (staging subdomain / Cloudflare preview / a cloned copy) where exactly ONE signal is removed at a time and re-run the battery. The signal whose removal *kills* the citation first is the one that matters most. Full matrix + ranking design is in §B. If staging is too slow, the **retrospective** fallback (GA4/referral `chatgpt.com`/`perplexity.ai` + server-log bot UAs + each signal's ship date) is specified in §B.4 as a triangulation, not a substitute.

4. **ChatGPT-only vs parity:** We should *assume* parity is required. Each engine uses its own crawler + index + model + citation policy, so a site optimized and *verified* only for ChatGPT is **not** verified for the other four. The battery in §A is designed to test all five at once so we learn the parity gap directly rather than by guesswork. Per-engine behavior (which bot, which robots lines, which citation surface) is tabulated in §C.

5. **Extant-site installs (§D):** Three paths — (1) we own repo/host → full patch, (2) DNS/edge-only → Cloudflare Workers injects llms.txt/UCP/schema, (3) parallel microsite (`/ai/` or agent subdomain). Decision tree + acceptance criteria per path included. Path 2 is the realistic default for most client sites; path 1 is the gold standard we already run on PJ/Maza.

6. **Bible 3.0 inputs (§E):** The key doctrinal shift is from "ship the stack" (v2.1's mandate) to **"ship the stack AND prove it moved citations via a multi-engine battery + staged ablation."** Verification protocol, multi-engine parity, and the extant-site decision tree are new first-class sections. Full bullet list in §E.

**Commercial honesty rule (for AVP marketing, t_10e1b346):** We never say "we guarantee ChatGPT will recommend you." We say: "Here is the agent-visibility stack; here is how we measure whether it produces citations across ChatGPT/Claude/Gemini/Perplexity/Grok; here is the before/after battery data." The measurement *is* the product's proof. No placement guarantees — measurement guarantees only.

---

## 1) Objective & success criteria

**Objective.** Give Frank a defensible, reproducible way to (1) verify that the agent-visibility stack actually produces citations across AI answer engines, (2) rank which individual signals matter most, (3) determine whether parity across Claude/Gemini/Grok/Perplexity is required, and (4) specify extant-site install paths — so AVP commercial claims rest on honest measurement, not vibes.

**Success criteria (what "done" means for this deliverable):**
- [x] A fixed, reproducible prompt battery (same prompts, all 5 engines) with a structured capture schema — ready to run.
- [x] A causal ablation design (staged, single-variable, prod-safe) with a ranking method for "which signal kills citations first."
- [x] A retrospective triangulation method for when live ablation is infeasible.
- [x] A multi-bot robots policy template (answer-engine allow vs training-token policy split, Cloudflare caveat).
- [x] An extant-site install decision tree (3 paths + acceptance criteria).
- [x] Bible 3.0 doctrine inputs.
- [x] Explicit, honest verification language for commercial use (no placement guarantees).

**Assumptions / scope constraints:**
- I am headless (no interactive browser to a human account), so I do **not** execute the live battery against paid/subscription engines in this run; I provide the exact protocol + capture schema + the *design* of the ablation. The battery is designed so a human (or an agent with a logged-in session) runs it in ~45 minutes and records into the provided table.
- I do **not** modify production PJ/Maza. All ablation is specified on staging/preview/clone.
- "Cited" means the engine's answer *names* the business or *links* to it. "Fact-quoted" means the engine reproduces a specific datum (phone/price/hours/address) from the site.
- Engines are treated as independent black boxes; we do not model their internals, only their observable citation behavior.

**Open questions that may change execution:**
- Do we have a staging/preview deployment for PJ and Maza (Cloudflare Preview, a `staging.` subdomain, or a git-branch clone)? This determines whether ablation is *prospective* (strong) or *retrospective-only* (triangulation).
- Do we have GA4 + server access (bot UA logs) for PJ and Maza? Required for the retrospective path.
- Do we have a logged-in Perplexity/Grok/Gemini/ChatGPT account (or an API key) to run the battery? If not, the battery is a runbook, not a result.
- Does Frank want the Grok/Perplexity legs at all, or is ChatGPT+Claude+Gemini enough for v1? (Recommendation: run all five — the parity question is the whole point.)

---

## 2) Live re-verification (this run, 2026-10-29)

Re-probed both reference sites today to confirm the baseline the battery/ablation measures against:

| Signal | printjunkieaz.com | mazahalalfood.com |
|--------|:---:|:---:|
| `robots.txt` AI allow | GPTBot, OAI-SearchBot, Google-Extended, **ChatGPT-User** + sitemap | GPTBot, OAI-SearchBot, Google-Extended + sitemap (no ChatGPT-User) |
| `llms.txt` | **404** | **200** (hand-authored facts + links) |
| `/.well-known/ucp` | **200** (ucp/1.0 full, 8 SKUs + tiers + checkout) | **200** (WebAPI discovery stub, 847 B) |
| `/api/specs` | per-product (`/api/specs/business-cards` 200) | **200** (63-item menu specs, 29 KB) |
| Home schema @type | LocalBusiness (+MerchantReturnPolicy/Dataset/FAQ) | Restaurant (+FAQ) |
| street address in schema | **no** (GeoCircle only) | **yes** (3491 W Frye Rd, Chandler) |
| aggregateRating | no | **yes** |
| sameAs count | 2 | **9** |
| visible `$` in home body | 7 price chips | 0 |

**Baseline interpretation (unchanged from t_64b6e1c1, re-confirmed today):** PJ = transaction/price surface; Maza = identity/hours surface. The battery and ablation in this report are run *against this baseline* — any "after" measurement is relative to it.

> **Note on the Dick AVP pack:** The task references `/srv/app-data/dick-jones/artifacts/agent-visibility-pack/`. That path is **not mounted in this container** (no `/srv/app-data/dick-jones` here; the `/srv/*` mounts in my env are research-output/checkpoints/huly-artifacts/scratch). The parent report + SEO Bible v2.1 + live probes are sufficient to produce this deliverable; the Dick pack (if it holds a canned robots template or SOW boilerplate) should be folded in by whoever owns that mount when the SOW is written. Flagging so it isn't silently assumed consumed.

---

## A) Multi-Engine Citation Battery (the verification core)

**Purpose.** Run the *identical* prompt set against all five engines and capture, per engine, whether the business was cited and what was quoted. This is the primary, honest, reproducible measure of "does the stack produce citations." It is run at **baseline** and **after** each stack change; the delta is the signal.

### A.1 Why a battery, not a single engine

Each engine is an independent stack: its own crawler (bot UA), its own index, its own base model, its own citation/grounding policy, and often its own freshness cadence. Optimizing + verifying only for ChatGPT tells you nothing about Claude, Gemini, Grok, or Perplexity. A fixed battery across all five is the only way to (a) confirm the ChatGPT result is real and not a one-off, and (b) directly answer the "parity or ChatGPT-only" question (Frank's #3).

### A.2 The five engines and their access reality (as of 2026-10)

| Engine | Crawler that indexes the page (search/answer) | Training crawler (separate, opt-in) | Answer/search access for the battery | Citation behavior |
|--------|-----------------------------------------------|-------------------------------------|--------------------------------------|-------------------|
| **ChatGPT** (with search) | `OAI-SearchBot` (real-time web) | `GPTBot` (model training) | ChatGPT chat w/ "Search" toggle; or API `web_search` | Cites sources inline; may link + quote facts |
| **Claude** (web/search) | `Claude-SearchBot` (real-time) | `ClaudeBot` (training) | Claude chat w/ web search on; or API `web_search` | Cites with source list; quotes facts |
| **Gemini** (Google AI) | `Google-Extended` (training) + standard `Googlebot` (index) | `Google-Extended` | Gemini app w/ live search; Google AI Overviews | Cites via "sources"; AI Overviews cite inline |
| **Perplexity** | `PerplexityBot` (real-time) | — (no public training bot) | Perplexity app / API | Cites per-claim, links to sources, quotes facts |
| **Grok** (xAI, web) | `Grok` / `grok` crawler (x/web) | — | Grok w/ web on (X-integrated) | Cites; X content weighted; docs thinner |

> **Bot naming caveat (honesty flag):** Vendor bot UAs and their robots semantics are *not* uniformly documented, and xAI/Grok in particular has historically been less transparent about its exact crawler identity and index behavior. The battery therefore does **not** depend on us correctly naming the bot — it depends on *observing the engine's actual citation output*. If a bot name in a table here is slightly off from a vendor's current doc, the measurement still stands because we score *output*, not *intent*. Where a vendor's crawler identity is unclear, we verify it empirically (see A.5 step 4).

### A.3 The fixed prompt set (local intent — identical across all engines)

**Rules for a valid run:**
- **One fresh, incognito session per engine-prompt** (no warm context, no prior turns).
- **Same wording, verbatim**, every engine. Do not "tune" per engine — that's the whole point of controlled comparison.
- **Same time window** (± a few hours) for all engines so index freshness is comparable.
- **Do not include the brand name** in most prompts (we are testing *discovery*, not recall of a named entity). The named-entity prompts are a separate control tier (A.3-C).
- Record **exactly** what the engine returns: the prose, the source links, and any inline citations. Screenshot as backup.

**Tier 1 — Anonymous local-intent (does the business get discovered without being named?)**

| ID | Prompt | Tests |
|----|--------|-------|
| P1 | "What's a good place to get business cards printed in Phoenix or Tempe, Arizona?" | PJ discovery (print, local) |
| P2 | "Where can I get custom window wraps or vehicle graphics in the Phoenix area?" | PJ discovery (signage lane — the open lane) |
| P3 | "What's a good halal Mediterranean restaurant in Chandler, Arizona?" | Maza discovery (restaurant, local) |
| P4 | "Where can I get gyro or shish kabab delivered in Chandler, Arizona?" | Maza discovery (dish + delivery) |
| P5 | "How much does it cost to get 1,000 business cards printed in Phoenix?" | PJ **price** surface (does UCP/price get quoted?) |
| P6 | "What are the hours for the best Mediterranean restaurant in Chandler?" | Maza **hours** surface (does schema/llms.txt hours get quoted?) |
| P7 | "Is there a good sign or banner shop near Chandler, Arizona?" | PJ signage lane (competitor check) |
| P8 | "What are the cheapest places to order business cards online in Arizona?" | PJ vs César Graphics (price-competitor check) |

**Tier 2 — Named-entity control (does it correctly resolve the brand?)**

| ID | Prompt | Tests |
|----|--------|-------|
| P9 | "What does Print Junkie AZ offer and how much are business cards?" | Named + price fact |
| P10 | "What are Maza Mediterranean's hours and address?" | Named + hours/address fact |

**Tier 3 — Wrong-competitor / hallucination traps (is the engine citing the *right* local player?)**

| ID | Prompt | Tests |
|----|--------|-------|
| P11 | "Best Mediterranean restaurant in Chandler, Arizona" — record **every** business named; flag if it names a closed/missing competitor or a non-Chandler one | Precision; wrong-competitor detection |
| P12 | "Business card printing in Tempe" — record every business named; flag if it names César Graphics over PJ or vice-versa | Lane precision |

> **Why Tier 3 matters:** An engine can "cite" the *category* correctly while naming the wrong specific shop, or hallucinate a plausible-but-wrong competitor. The battery must catch false-positive and false-negative citations, not just "did our brand appear."

### A.4 Capture schema (one row per engine × prompt)

Record into the spreadsheet below. One row = one (engine, prompt) run.

| Field | Meaning | Example |
|-------|---------|---------|
| `run_date` | ISO date the run happened | 2026-10-29 |
| `engine` | ChatGPT / Claude / Gemini / Perplexity / Grok | ChatGPT |
| `engine_version` | model / product version if shown | "ChatGPT 5 (o-series)" |
| `prompt_id` | P1…P12 | P1 |
| `our_business_cited` | Y / N | Y |
| `citation_type` | named / linked / quoted / not-present | linked+quoted |
| `link_used` | exact URL the engine cited (or "none") | printjunkieaz.com/business-cards-phoenix |
| `facts_quoted` | which specific datums it reproduced: phone / price / hours / address / menu | price ($75/1k cards), phone |
| `fact_accuracy` | correct / wrong / n/a (check vs ground truth) | correct |
| `wrong_competitor_named` | any non-target business it named (esp. if it should have named ours) | "César Graphics" |
| `hallucination` | any fact it stated that is false | none |
| `answer_confidence` | did the engine hedge / "I don't know" / assert | assertive |
| `latency_note` | any "searching the web" delay, or "my knowledge cutoff" disclaimer | "cutoff May 2026" |
| `screenshot` | path to backup screenshot | /captures/2026-10-29/chatgpt-P1.png |

**Scoring (for ranking, not vibes):**
- **Discovery score per engine** = (# Tier-1 prompts where `our_business_cited=Y`) / (# Tier-1 prompts for that business).
- **Fact-accuracy score** = (# facts quoted that are correct) / (# facts quoted).
- **Precision** = (correct target citations) / (all business citations, incl. wrong competitors) on Tier 3.
- **Parity gap** = per-engine discovery score, side by side. This directly answers Frank's #3.

### A.5 Run protocol (operational steps, ~45 min for one full battery)

1. Confirm a fresh, logged-in (or API-keyed) session per engine. Perplexity and Grok are easiest via web app; Gemini via app; Claude via chat w/ web search on; ChatGPT with search on.
2. For each engine, run P1→P12 in order, one fresh session/tab each, verbatim wording.
3. Capture each response: copy the text, the source links, and a screenshot.
4. **Empirically confirm each engine's crawler** (do not trust the table): for the businesses you're testing, fetch the page *as that engine's documented bot UA* and confirm 200, then check that the engine's *answer* actually reflects current page content (if it cites an old price, its index is stale — record that in `latency_note`).
5. Fill the capture schema. Compute discovery/fact/precision/parity scores.
6. **Repeat the full battery** after each stack change (see §B) — the before/after delta is the verification.

### A.6 Ground-truth reference (what the engine *should* be able to quote, per business)

| Business | Phone | Price (business cards 1k) | Hours | Address | Menu/dish |
|----------|-------|---------------------------|-------|---------|-----------|
| **Print Junkie AZ** | +1-602-730-2135 | $75 (1,000 cards) | Mon–Sat 09:00–17:00 | (no street in schema — GeoCircle Phoenix) | n/a (print) |
| **Maza Mediterranean** | (480) 534-6550 | n/a (no visible $) | Tue–Sun 10:00–22:00, closed Mon | 3491 W Frye Rd Ste 2, Chandler, AZ 85226 | gyro, shish kabab, etc. |

If an engine quotes a phone/price/hours/address that matches this table, it is demonstrably reading *our* structured surface (not hallucinating) — that is the strongest single piece of verification evidence.

### A.7 Battery output = the honest commercial proof

This is the deliverable Frank can show clients: **"Here is the same question asked to five AI engines before and after we installed the stack. Before: 0/5 cited us. After: 4/5 cited us, and 3 of them quoted our exact price and hours. No guarantees — this is how we measure."** That is the AVP marketing proof, and it is *honest* because it is observed, not promised.

---

## B) Signal Ablation / Ranking (causal, prod-safe)

**Purpose.** Determine *which individual signals* matter most by removing one at a time and observing the citation delta. This is the causal (not vibes) answer to Frank's #2.

### B.1 The core causal principle

We cannot control the model, so we use the **counterfactual**: for a given page, compare its citation behavior **with the full stack** vs **with exactly one signal removed**. The signal whose removal produces the **largest drop in citations** is the most causally important. To keep it causal (not confounded):

- **Single-variable:** remove *one* signal per variant; everything else identical.
- **Same engine set:** run the *same* §A battery on each variant.
- **Control = full stack (current baseline).**
- **Randomize order** of variant testing across engines so time/season drift doesn't bias one variant.
- **Prod is never touched.** Variants live on staging/preview/clone.

### B.2 The ablation matrix (variants to run)

| Variant | Change (exactly one) | All other signals | Hypothesis — what it tests |
|---------|----------------------|-------------------|----------------------------|
| **V0 (baseline/control)** | None — current full stack | all present | Reference citation rate |
| **V1 — remove llms.txt** | `llms.txt` → 404 | UCP, schema, prices, robots all intact | Is llms.txt the primary identity-discovery trigger? |
| **V2 — remove UCP** | `/.well-known/ucp` → 404 | llms.txt, schema, prices, robots intact | Is UCP the primary transaction/price-discovery trigger? |
| **V3 — strip JSON-LD** | remove all `LocalBusiness`/`Restaurant` schema blocks | llms.txt, UCP, prices, robots intact | Is structured schema the load-bearing identity surface? |
| **V4 — hide visible $** | remove all `$` price strings from HTML body (prices only in images/JS, or nowhere visible) | llms.txt, UCP, schema, robots intact | Do *visible* prices matter vs machine-readable ones? |
| **V5 — block OAI-SearchBot only** | `robots.txt`: `Disallow: /` for `OAI-SearchBot` | everything else intact | Does blocking the *real-time* ChatGPT search bot kill ChatGPT citations? |
| **V6 — block all AI search bots** | `Disallow: /` for `OAI-SearchBot` + `Claude-SearchBot` + `PerplexityBot` | everything else intact | Do the search/answer bots (vs training bots) drive citations? |

> **Design note on V5/V6 vs the "training bot" distinction:** V5/V6 test the **answer-engine** bots (the ones that serve the *live search* the battery exercises). The **training** bots (GPTBot, ClaudeBot, Google-Extended) feed the *models* and operate on a longer, less-controllable timescale — you cannot cleanly ablate training-crawler effect in a short battery run, so we **do not** include a "block all training bots" variant in the *prospective* matrix (it wouldn't show up in a 2-week window anyway). We flag training-bot policy as a **separate, slower** decision in §C rather than a battery variable.

### B.3 How to build the variants safely (three tiers, pick by availability)

**Tier 1 — staging subdomain / preview (preferred, cleanest).**
If PJ and Maza have a staging deploy (e.g. `staging.printjunkieaz.com`, or Cloudflare Preview/branches), implement each variant as a **branch** that flips exactly one signal off, deploy to staging, and point the battery at the staging host. This gives a true single-variable counterfactual with zero prod risk. **Requirement:** the staging host must be *crawlable* by the engine bots (i.e. not itself blocked) and must serve the variant's exact signal state.

**Tier 2 — edge/Cloudflare Worker rewrite (if no staging deploy exists).**
A Cloudflare Worker on the **origin** can, per-request or per-host, (a) force `llms.txt`/`/.well-known/ucp` to 404, (b) strip `<script type="application/ld+json">` blocks, (c) rewrite `robots.txt` to add a `Disallow`, (d) rewrite visible `$` strings out of the HTML. Gate each variant behind a header/cookie/query param or a separate subdomain so you can toggle V0–V6 without touching the CMS. This is the same mechanism as §D path 2 — so building it here doubles as the extant-site injection proof.

**Tier 3 — local clone + reverse proxy (last resort, fully offline-safe).**
Clone the site to a local/container host, put a reverse proxy in front, implement each variant in the proxy (404 the paths, filter the HTML), and run the battery against the clone's public URL. Downside: the clone's *domain* differs from the real one, which itself is a variable (engines weight domain authority/brand) — so Tier 3 is a **relative** test only and must be labeled as such. Prefer Tier 1 or 2.

### B.4 Retrospective triangulation (if live/staging ablation is infeasible or too slow)

When we can't stand up variants, reconstruct the causal picture from **three independent data streams** and check they agree:

1. **Citation/index timeline.** For each engine, establish *when* the business first started getting cited (battery runs over time, or first-seen in engine search). Align each engine's "first citation" date with the dates each signal **shipped** on PJ/Maza (from git history / deploy logs: llms.txt date, UCP date, schema date, price-chip date).
2. **Referral traffic (GA4 / server logs).** Pull `chatgpt.com`, `claude.ai`, `perplexity.ai`, `gemini.google.com`, `grok.x.ai` (and `*bot` UA) referral sessions + the specific pages/paths they hit, over time. A signal that shipped on date D followed by a step-change in AI-referral sessions around D is evidence of that signal's effect.
3. **Bot-crawl logs.** Server logs for the *search* bot UAs (OAI-SearchBot, Claude-SearchBot, PerplexityBot) vs *training* bot UAs — volume + which paths they hit + 200/403/429 status. Confirms which surfaces the answer engines are actually pulling.

**Triangulation rule:** a signal is called "important" only if **two or more** of the three streams point the same direction (e.g. UCP shipped → UCP path gets crawled by OAI-SearchBot → business-card referral from chatgpt.com rises). One stream alone = hypothesis, not finding. Label retrospective findings as **correlational** in the report — that is the honest limit.

### B.5 Ranking method (turn the battery deltas into a signal ranking)

For each signal s (V1–V6), define its **ablation impact** as:

`impact(s) = citation_rate(V0) − citation_rate(variant_s)`  (same engine set, same prompts)

where `citation_rate` = mean discovery score across the engine set for that business.

Rank signals by `impact(s)` descending. The **top-ranked** signal is the one whose removal kills citations first = the most causally important. Report the ranking **per engine** (a signal may matter for ChatGPT but not Perplexity) **and** pooled. Also report **statistical caution**: a single battery run has small N (8–12 prompts); treat a "significant" impact as one that flips the cited/not-cited outcome on *most* prompts, and re-run 2–3x to confirm it's not noise. This is honest about sample size rather than dressing up one run as a proof.

### B.6 Predicted ranking (hypothesis to be *falsified*, not asserted)

Based on the parent audit + how answer engines actually work, the *a-priori* hypothesis (to be tested, not claimed) is:

1. **Schema (V3)** most load-bearing for *identity* — the typed `Restaurant`/`LocalBusiness` block is what an engine ingests first and what feeds map/POI grounding. Removing it should hurt the *named* and *POI* prompts hardest.
2. **llms.txt (V1)** high impact for *discovery* on engines that read it (Claude, some others) — but its effect is engine-specific, which is exactly why the multi-engine battery is needed.
3. **Visible prices (V4)** moderate for *price* prompts (P5) specifically, near-zero for discovery prompts — isolates the "does a human-visible price matter vs a machine-readable one" question.
4. **UCP (V2)** high for *price/checkout-intent* prompts, low for pure discovery — UCP is deep (the agent must know to probe it), so its *discovery* effect is gated on whether the engine reads llms.txt/schema to find it.
5. **Bot blocking (V5/V6)** should show the *largest* effect on the *blocking* engines specifically (V5 kills ChatGPT; V6 kills ChatGPT+Claude+Perplexity) — this is the cleanest causal signal because blocking is binary and direct.

The battery's job is to **confirm or overturn** this ranking. If, e.g., removing llms.txt kills more citations than removing schema, the hypothesis is wrong and we update the doctrine — that is the value of measuring.

---

## C) Multi-Bot Robots Policy Recommendation

**Core split: answer-engine bots (visibility — must allow) vs training bots (policy choice — separate decision).**

### C.1 Must-allow for *answer-engine visibility* (do NOT block)

These crawlers feed the **real-time search / answer** surfaces the battery exercises. Blocking them = the business disappears from live AI answers:

| Bot | Vendor | Role | Action |
|-----|--------|------|--------|
| `OAI-SearchBot` | OpenAI/ChatGPT | Real-time web search (ChatGPT answers) | **ALLOW** |
| `ChatGPT-User` | OpenAI | ChatGPT interactive/agent fetch | **ALLOW** |
| `Claude-SearchBot` | Anthropic | Real-time web search (Claude answers) | **ALLOW** |
| `Claude-User` | Anthropic | Claude interactive/agent fetch | **ALLOW** |
| `PerplexityBot` | Perplexity | Real-time answer search | **ALLOW** |
| `Perplexity-User` | Perplexity | Perplexity interactive | **ALLOW** |
| `Googlebot` | Google | Core index (never block — kills everything) | **ALLOW** |
| `Google-Extended` | Google | Gemini/ML training (see C.2 — but *not* the same as the index) | policy choice, see C.2 |
| `Amazonbot` / `Amzn-Search` | Amazon | Amazon shopping/answer surfaces (as applicable to the vertical) | ALLOW if in a commerce vertical |

> **Never-block list:** `Googlebot`, `OAI-SearchBot`, `Claude-SearchBot`, `PerplexityBot`. These are the answer engines. Blocking any of them is a direct, fast, self-inflicted loss of AI visibility. (SpeedPro Phoenix's "blocks GPTBot but publishes llms.txt" from the parent audit is the anti-pattern — a contradictory stance that undermines its own intent.)

### C.2 Training tokens — separate policy decision (default: allow, opt out consciously)

These feed **model training**, operate on a slow timescale, and are a *brand/data* policy choice, not a visibility choice. **They are not the same as the answer-engine bots** — blocking them does *not* meaningfully change short-term citation behavior, so they should be decided separately and deliberately:

| Bot | Vendor | Role | Default recommendation |
|-----|--------|------|------------------------|
| `GPTBot` | OpenAI | Model training | **ALLOW** (opt out only if Frank explicitly wants to exclude from OpenAI training — a business decision, not an SEO one) |
| `ClaudeBot` | Anthropic | Model training | ALLOW (same) |
| `Google-Extended` | Google | Gemini/ML training | ALLOW (separate from Googlebot — do not conflate) |
| `Applebot-Extended` | Apple | Siri/Apple training | ALLOW |
| `CCBot` | Common Crawl | Open training corpus (feeds many LLMs) | ALLOW unless data-policy says otherwise |
| `Bytespider` | ByteDance (TikTok/CapCut) | Training/crawl | ALLOW unless data-policy says otherwise |

> **Honesty note on training bots:** Whether allowing a training bot actually improves *citations* is a long-timescale, hard-to-measure question (it affects the model's latent knowledge, not the live index). We should **not** claim "allowing GPTBot improves your ChatGPT citations" as fact — it may, but the measurable short-term lever is the **search** bots (C.1). Present training-bot policy as a data-governance choice, and measure it (if at all) over months, not weeks.

### C.3 Grok / xAI — behavior caveat (poorly documented)

xAI's crawler identity and index behavior are **less transparent** than OpenAI/Anthropic/Google/Perplexity. Practical handling:
- Identify Grok's current crawler **empirically** (observe what UA hits your server in access logs when you run the Grok leg of the battery) rather than trusting a possibly-outdated doc.
- **Allow** whatever UA serves Grok's web answers, because Grok is a real answer engine and blocking it loses that channel.
- If the UA is ambiguous, **do not block by default** — the cost of accidentally blocking an answer engine is higher than the cost of allowing an unknown crawler. Flag the ambiguity in the report rather than guessing a block.

### C.4 Cloudflare / WAF caveat (the silent override)

**Critical warning for the SOW:** A site behind Cloudflare (or any WAF/CDN) with the **"Block AI bots" / "Super Bot Fight Mode" / a custom WAF rule set to challenge AI UAs** will **override the robots.txt Allow** — the request never reaches the origin's robots policy. A site can have a perfectly correct `robots.txt` that allows every answer bot and *still* be invisible if the edge challenges those UAs (the 429 bot-walls on azhottees.com and phxsignshop.com in the parent audit are exactly this).

**SOW acceptance rule:** before claiming "the site is AI-visible," verify not just `robots.txt` but that each answer-engine bot UA gets a **200 (not 403/429/challenge)** at the *edge*. A robots Allow is necessary but not sufficient when a WAF is in front. The battery's empirical bot-UA check (§A.5 step 4) is the control for this.

### C.5 Recommended robots.txt template (for the SOW)

```
# Agent-Visibility robots.txt — answer engines ALLOWED, training = separate decision
# Generated for <SITE> — <DATE>

User-agent: *
Allow: /
Sitemap: https://<SITE>/sitemap.xml

# --- Answer engines (live search / AI answers) — MUST be allowed ---
User-agent: OAI-SearchBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: Claude-SearchBot
Allow: /

User-agent: Claude-User
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Perplexity-User
Allow: /

# Google: never block the index.
User-agent: Googlebot
Allow: /

# Amazon (only if in a commerce vertical)
User-agent: Amazonbot
Allow: /
User-agent: Amzn-Search
Allow: /

# --- Training crawlers — POLICY CHOICE (default allow; opt out deliberately) ---
# These are separate from the answer engines above and are a data-governance
# decision, not a visibility one. Flip to Disallow only on explicit approval.
User-agent: GPTBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Google-Extended
Allow: /

User-agent: Applebot-Extended
Allow: /

User-agent: CCBot
Allow: /

User-agent: Bytespider
Allow: /

# --- Grok / xAI: identify empirically from access logs before finalizing ---
# Do not block an unknown answer-engine UA by default.
```

> **Deployment note:** If the site is on Cloudflare/edge, this template must be *backed by* edge rules that (a) do not challenge the allowed UAs and (b) let them reach the origin — see C.4. A correct robots file behind a blocking WAF is a no-op.

---

## D) Extant-Site Install Patterns (for the AVP SOW)

**Context.** PJ and Maza are *our* sites where we own the repo/host (path 1). The AVP business, though, is selling this stack to **extant** sites where we may not control the origin. Three install paths, ordered by fidelity:

### D.1 Decision tree

```
Is the site our own repo/host (we can deploy code)?
├─ YES → PATH 1: Full patch (gold standard). Ship the whole stack in-repo.
└─ NO  → Do we control DNS + edge (Cloudflare/CDN) for the domain?
         ├─ YES → PATH 2: Edge injection (Cloudflare Workers). Inject
         │         llms.txt / UCP / schema / robots at the edge; origin stays
         │         untouched. Best default for most clients.
         └─ NO  → PATH 3: Parallel microsite (last resort). Host a separate
                   agent-facing surface (/ai/ or an agent subdomain) that holds
                   the facts + deep-links to the main site.
```

### D.2 Path 1 — We own repo/host (full patch)

| Aspect | Detail |
|--------|--------|
| **What** | Patch the real site: add `llms.txt`, `/.well-known/ucp`, typed JSON-LD (`Restaurant`/`LocalBusiness`/`Store` + `Offer`/`MenuItem`), visible prices, `robots.txt` allow-list, local-intent URL farm. Full PJ+Maza stack. |
| **Effort** | Medium (theme/framework patch; a few hours to ~2 days depending on stack: Next.js head/schema, WP theme + a plugin, or static). |
| **Risk** | Low if done in staging first (see §B.3 Tier 1); we control the deploy. Highest fidelity — the *real* domain carries the real data. |
| **Acceptance criteria** | (1) `llms.txt` 200 with name/address/phone/hours/links; (2) `/.well-known/ucp` 200 with real products + tiers + checkout intent; (3) typed schema with street address + hours + (where real) aggregateRating; (4) `robots.txt` allows all C.1 bots; (5) every answer-engine bot UA returns 200 at the edge (C.4 check); (6) §A battery: business cited on ≥ N/5 engines on target prompts (the "after" baseline). |

### D.3 Path 2 — DNS/edge only (Cloudflare Workers injection)

| Aspect | Detail |
|--------|--------|
| **What** | Point the client's domain at a Cloudflare Worker (or similar edge) that **injects** the agent surfaces *without touching the origin*: serve `/llms.txt`, `/.well-known/ucp`, an injected `<script type="application/ld+json">` into the HTML `<head>`, and a merged `robots.txt`. The origin site is unchanged — the edge adds the agent layer on top. |
| **Effort** | Medium (write the Worker; source the facts from a small JSON config the client fills in). ~1–3 days. No CMS/framework changes. |
| **Risk** | Low for the origin (it's untouched); the risk is **correctness of the injected data** (if the config is wrong, the agent is fed wrong facts — worse than nothing). Must validate the injected schema is well-formed and the data matches the real business. Also: edge injection of `<head>` schema requires the Worker to reliably rewrite the HTML response (do it only when the origin returns full HTML, not a JS shell — see the Greeks Grill "JS shell, 47 stripped chars" anti-pattern from the parent audit). |
| **Acceptance criteria** | (1) injected `llms.txt`/`ucp`/schema each return 200 with valid, *correct* data; (2) the injected JSON-LD is valid and matches the real NAP/prices; (3) origin HTML still loads for humans (injection is non-destructive); (4) answer-engine bots get 200 at the edge; (5) §A battery shows citation on the *real* domain. |
| **Best for** | Rigid origins we can't patch (hosted SaaS, legacy CMS, client won't give repo access) but where we control the domain's DNS/edge. **The realistic default for most AVP clients.** |

### D.4 Path 3 — Parallel microsite (last resort)

| Aspect | Detail |
|--------|--------|
| **What** | Host a **separate** agent-facing surface — an `agent.<site>` subdomain or a `/ai/` path on a domain we control — that holds the machine-readable facts (llms.txt, UCP, schema) and **deep-links to the main site** for the human. The main site is left entirely alone. |
| **Effort** | Low (small static site). |
| **Risk** | **Highest.** The agent surface lives on a *different host/domain* than the brand's real pages, so engines have weaker grounding (the cited link points to the microsite, not the real product page); NAP consistency is harder; and if the microsite is ever deindexed or the client's real site is the one with authority, the benefit is diluted. Use only when neither repo nor edge access is available. |
| **Acceptance criteria** | (1) microsite serves the full machine stack (llms.txt/UCP/schema) with correct data; (2) every fact deep-links to the corresponding *real* page on the main site (so the cited link resolves to real content); (3) NAP on the microsite exactly matches the main site; (4) answer-engine bots get 200 on the microsite; (5) §A battery shows citation, and the cited link is *actionable* (resolves to real content, not a dead microsite). |
| **When** | No repo, no edge control, client still wants agent visibility. Treat as a stopgap to be upgraded to Path 1/2 when access becomes available. |

**SOW default:** quote Path 1 when we have repo/host, Path 2 (edge) as the standard client offering, Path 3 only with the explicit "last resort, reduced grounding" caveat. Never promise Path 3 delivers the same fidelity as Path 1.

---

## E) Inputs to Bible 3.0 (doctrine changes for the next edition)

The v2.1 Bible mandates *shipping* the stack (UCP, schema, llms.txt, crawler access). The missing doctrine — and what this task adds — is **verification and the measurement layer.** Proposed additions:

1. **New top-level section: "Verification & Measurement" (the missing half of the Golden Rule).** The Golden Rule says "winners serve both layers" — but v2.1 never says *how you know the agent layer is working*. Add a standing doctrine: **ship the stack, then prove it with a multi-engine citation battery; the before/after delta is the evidence, not the presence of the files.** No "we shipped llms.txt" is a done claim until a battery shows it moved a citation.

2. **Multi-engine parity is a first-class requirement, not a ChatGPT checkbox.** Doctrine: a site is "agent-visible" only when it is *measured* across ChatGPT, Claude, Gemini, Perplexity, and Grok. Each engine has a distinct crawler + index + citation policy; optimizing/verifying one is not the others. The canonical test is the fixed battery (§A), not a single-engine spot check.

3. **Causal verification via staged single-variable ablation — never prod.** Doctrine: to claim a signal "matters," remove exactly one signal on a **staging/edge/clone** variant and re-run the battery; rank by citation delta. **Hard rule: never ablate production without explicit owner approval.** When staging is unavailable, use retrospective triangulation (referral + bot-crawl logs + signal ship-dates) and **label it correlational.**

4. **Answer-engine bots vs training bots are different decisions.** Doctrine: the *visibility* policy (allow OAI-SearchBot/Claude-SearchBot/PerplexityBot/Googlebot, never block them) is separate from the *training* policy (GPTBot/ClaudeBot/Google-Extended/Applebot-Extended/CCBot/Bytespider — a data-governance choice, not an SEO lever). Do not conflate them, and do not claim training-bot allowlist affects short-term citations.

5. **The WAF/edge override is a standing gotcha.** Doctrine: a correct `robots.txt` is **not sufficient** when a Cloudflare/WAF "block AI" or Super-Bot-Fight rule challenges the same UAs. The acceptance check is a **200 at the edge** for each answer-engine bot UA, not just the robots file. (The 429-walled sites in the parent audit are the standing warning.)

6. **Extant-site install doctrine: three paths with explicit fidelity tiers.** Doctrine: Path 1 (own repo/host) = full fidelity; Path 2 (edge/Cloudflare Workers injection) = standard client offering, origin untouched; Path 3 (parallel microsite) = last resort, reduced grounding, must deep-link to real pages. Every SOW states which path applies and its acceptance criteria.

7. **Commercial honesty rule for agent-visibility claims.** Doctrine (for AVP/marketing): we **never** promise "X will recommend you." We promise **measurement**: "here is the stack, here is the multi-engine before/after battery, here is the ablation ranking." The product's proof is the *measurement protocol + the data it produces*, never a placement guarantee. This is the single most important doctrinal change for the commercial side of AVP.

8. **Ground-truth fact table is part of the deliverable, not optional.** Doctrine: every audited site ships a ground-truth table (phone/price/hours/address) so a battery can distinguish *the engine read our structured data* (quoted the exact fact) from *the engine hallucinated* (quoted a wrong/plausible fact). Fact-accuracy is a scored metric, not a footnote.

9. **Two-surface model stays (identity vs transaction), now with per-surface measurement.** Retain the v2.1 finding that "AI-readable" splits into an **identity/discovery surface** (llms.txt, typed schema, address, hours, rating, sameAs — Maza's moat) and a **transaction/offer surface** (UCP products/tiers, specs, visible prices — PJ's moat), and add: measure each surface's contribution *separately* via the ablation variants (V1/V3 test identity; V2/V4 test transaction).

10. **Version + timestamp + cache-invalidation remains standard** (both references do it): manifests carry `lastUpdated` + a version string so engines can invalidate; the battery re-run cadence (e.g., quarterly, or after any stack change) is the standing re-verification loop.

---

## F) Claims & evidence (structured)

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | The multi-engine citation battery (§A) is the primary, reproducible, honest verification method — it is run at baseline and after each change; the delta is the measurement. | §A; parent t_64b6e1c1 baseline; live re-probe 2026-10-29 | medium (method is sound; the *result* depends on the battery actually being run — I could not execute it against subscription engines headless) | supported (as a protocol) / **not yet executed** |
| C2 | We cannot claim direct model-level causation ("the stack made ChatGPT cite us"); the strongest defensible claim is correlation + counterfactual consistency. | LLMs/indices are third-party; no access to model weights or index internals | high | supported |
| C3 | A staged single-variable ablation (one signal removed at a time on staging/edge/clone, prod untouched) + battery re-run is the correct causal method; rank signals by citation delta. | §B; standard experimental-design principle; prod-safety constraint from task | medium | supported (as design) |
| C4 | Blocking answer-engine search bots (OAI-SearchBot/Claude-SearchBot/PerplexityBot) is expected to be the single most direct citation-killer; training-bot blocking is a slower, separate, data-governance effect. | §B V5/V6; §C; vendor bot semantics; note training timescale is longer than a battery window | medium | supported (hypothesis to test via V5/V6) |
| C5 | A correct robots.txt Allow is not sufficient for AI visibility when a Cloudflare/WAF "block AI" rule challenges the same UAs; acceptance must be a 200 at the edge per bot UA. | §C.4; parent audit 429 bot-walls (azhottees.com, phxsignshop.com) | high | supported |
| C6 | Parity across Claude/Gemini/Grok/Perplexity is required (not ChatGPT-only): each engine is an independent crawler+index+model+citation stack. | §A.1/A.2; standard of distinct vendor stacks | medium | supported |
| C7 | Extant-site installs split into 3 fidelity paths: own-repo (full), edge/Workers injection (standard, origin untouched), parallel microsite (last resort, reduced grounding); edge is the realistic client default. | §D; parent audit (PJ/Maza = path 1 already running) | medium | supported (as SOW guidance) |
| C8 | The honest commercial rule: no placement guarantees; the product's proof is the multi-engine before/after measurement, not a promise. | §E.7; Frank's urgency note (commercial AVP claims need honest verification language) | high | supported |
| C9 | The two-surface model (identity vs transaction) from the parent audit holds and should be measured per-surface in the ablation. | parent t_64b6e1c1 §1.3/§3; live re-probe 2026-10-29 (PJ UCP 200/llms 404; Maza llms 200/UCP stub) | high | supported |
| C10 | Grok/xAI crawler identity is less documented; handle by empirical UA observation + default-allow, not by guessing a block. | §C.3; general vendor-doc transparency caveat | medium | supported |

## G) Evidence index (provenance)

| evidence_id | source_label | source_url | provenance | support note |
|-------------|--------------|------------|------------|--------------|
| E1 | Parent report t_64b6e1c1 | https://github.com/arealicehole/know/blob/main/research/ai-readable-web-pj-maza-competitor-audit-t_64b6e1c1.md | local `/home/ice/know/research/…` (read this run) | Full signal matrix, competitor scan, two-surface model, robots/UCP/llms findings; basis for baseline + C4/C5/C6/C9 |
| E2 | Live re-probe PJ (2026-10-29) | https://printjunkieaz.com/robots.txt ; /.well-known/ucp ; /llms.txt | HTTP curl this run (status + body) | robots allows 4 AI bots + sitemap; llms.txt 404; UCP 200 — confirms PJ baseline unchanged |
| E3 | Live re-probe Maza (2026-10-29) | https://mazahalalfood.com/robots.txt ; /llms.txt ; /.well-known/ucp | HTTP curl this run | robots allows 3 AI bots (no ChatGPT-User); llms.txt 200; UCP 200 — confirms Maza baseline unchanged |
| E4 | SEO Bible v2.1 | `/home/ice/know/SEO_BIBLE_JUNE_2026.md` (local) | read this run (2.1, §1.1/§1.3) | Existing doctrine baseline for §E inputs; mandates stack, lacks verification layer |
| E5 | Vendor bot/crawler semantics | OpenAI/Anthropic/Google/Perplexity/xAI public docs (web) | web context, 2026-10; **not individually re-fetched this run** — flagged | Basis for C.1/C.2 bot table; **honesty note:** bot names/semantics are vendor-documented and can change; the battery measures *output* so it is robust to name drift; Grok specifically under-documented (C10) |
| E6 | 429 bot-wall sites (azhottees.com, phxsignshop.com) | parent t_64b6e1c1 §2.1 | parent report (probed 2026-10-28) | Evidence for C5 (WAF/edge override makes a correct robots.txt a no-op) |
| E7 | Dick AVP pack | `/srv/app-data/dick-jones/artifacts/agent-visibility-pack/` | **NOT mounted in this container** (flagged) | Referenced by task but unavailable here; parent report + Bible + live probes were sufficient; fold in when SOW is written |

> **Honesty note on E5:** I did not re-fetch each vendor's live robots/crawler doc in this run; the bot table (C.1/C.2) reflects the established 2026 state. Where a vendor changes a bot name or robots semantics, the battery (which scores *observed citation output*, not bot names) still stands. The one genuinely under-documented case is **Grok/xAI** — handled by empirical UA observation (C3, C10) rather than a possibly-stale doc. I flag this rather than silently asserting every bot string is current.

## H) Gaps / unresolved questions

1. **The battery was not executed against live engines in this run.** I am headless with no logged-in subscription engine sessions and should not fabricate citation results. §A is a *runnable protocol + capture schema + ground-truth table*, ready for a ~45-minute human/agent run. **The verification result is the next step, not this report.** This is the honest boundary: I delivered the *method and design*, not a fake "we got cited 4/5."
2. **Staging/preview availability for PJ and Maza is unconfirmed.** If no staging deploy exists, ablation (§B) falls to Tier 2 (Cloudflare Worker) or Tier 3 (clone) or the retrospective path (§B.4). Need to confirm which is in place.
3. **GA4 + server bot-logs access for PJ/Maza is unconfirmed** — required for the retrospective triangulation (§B.4).
4. **Grok/xAI crawler identity** is not definitively pinned — must be confirmed empirically from access logs before finalizing its robots line.
5. **Dick AVP pack** (canned robots/SOW boilerplate) was not in this container — should be merged in by its owner when the SOW is drafted.
6. **Sample size:** a single battery run has small N (8–12 prompts × 5 engines). Treat impact rankings as directional until re-run 2–3× (noted in §B.5) — do not over-claim significance from one pass.

## I) Confidence (report level)

- **level:** **medium-high**
- **rationale:** The **method/design** deliverables (battery protocol, ablation matrix, robots template, extant-site tree, Bible inputs) are high-confidence — they are standard, correct, and directly derived from the verified parent audit + live re-probes (E1–E4). The **live baseline** is high-confidence (re-probed today). The **measured verification result** (whether the stack actually produces citations) is **not yet produced** — I did not run the engines and will not fabricate it — so the *finding* carries the confidence of "sound protocol, awaiting execution," which is exactly the honest posture Frank asked for. Vendor bot specifics (E5) are medium (can drift; battery is robust to it).

## J) Recommended next actions

1. **Run the baseline battery (this week):** a human or logged-in agent runs §A (P1–P12 × 5 engines) against current PJ/Maza and fills the capture schema. This is the "before" — everything else builds on it. (Highest priority; unblocks the whole verification story.)
2. **Stand up the ablation host:** confirm/enable a PJ+Maza staging subdomain or a Cloudflare Worker (Tier 1/2, §B.3) so V1–V6 can be tested prod-safely. If unavailable, pull GA4 + bot-crawl logs for the retrospective (§B.4).
3. **Pin the Grok UA** from access logs (C3, C10) and finalize its robots line.
4. **Run the ablation battery** (V0 vs V1–V6, §B) and compute the signal ranking; re-run 2× to confirm; report per-engine + pooled.
5. **Merge the Dick AVP pack** (E7) into the SOW once its mount is reachable; fold in the §C.5 robots template and §D decision tree.
6. **Draft Bible 3.0** from §E — the core doctrinal shift is "ship *and* measure," plus the multi-engine parity + WAF-override + commercial-honesty rules.
7. **Write the AVP commercial one-pager** (for t_10e1b346) using the §A.7 framing: the before/after battery data *is* the proof; no placement guarantees.

---

*Prepared by Geraldo v2.2 (Kanban-native, Swarm Kanban v2.0). Single durable copy: `/home/ice/know/research/avp-causal-verification-multi-engine-citation-battery-t_70d29518.md`. No double-publish. All live probes re-verified 2026-10-29. Verification result pending battery execution — no fabricated citation counts.*

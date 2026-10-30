# The SEO & Agentic Commerce Bible

**Third Edition — locked 3.0**

**Version:** 3.0  
**Locked date:** October 30, 2026  
**Draft assembled:** October 29, 2026  
**Prior locked edition:** 2.1 (`SEO_BIBLE_JUNE_2026.md` — June 24, 2026 core + October 14, 2026 media §8)  
**Status:** **LOCKED** — ops ruler for audits and agent-visibility installs. Prefer this file over `SEO_BIBLE_JUNE_2026.md` and any `*-DRAFT*`. Archive prior 2.1 remains in this folder. Supercedes `SEO_BIBLE_3.0_DRAFT.md`.

### Changelog
| Date | Ver | Change |
|------|-----|--------|
| 2026-06-24 | 2.0 | Second Edition: Information Gain, restaurant Core 30 case study, UCP/ACO merge |
| 2026-10-14 | 2.1 | **§8** News desks, headline aggregators & legislative trackers (media gate vs Core 30 / full UCP). Reference specimen: Root & Rifle (`rootandrifle.com`, repo `arealicehole/rootandrifle-site`) |
| 2026-10-29 | 3.0-DRAFT | **Additive chat-UI / answer-engine discovery refresh** from Geraldo research `chat-ui-discovery-bible-section-refresh-20261029.md` (+ independent review corrections). **No deletions** of §1–§8 body. Adds multi-engine/edge-200, directories-as-answer-sources, commerce handoff reality (UCP standard + ACP/feeds), schema-as-entity framing, verification protocol, identity vs transaction surfaces, llms.txt honesty. **No AVP commercial pricing.** Independent review notes applied: Information Gain re-prioritization is correlational (not proven causal #1); Cloudflare AI defaults are **narrow** (new domains / ad-display pages), not a global “blocks all AI” rule; KEEP/UPDATE arithmetic from research B-table = KEEP 5 / UPDATE 11 / REPLACE 0. |
| 2026-10-30 | **3.0 LOCK** | Frank lock. Optional polish: **§1.7 GMC eligibility doctrine** (SKU free listings partial; Google UCP checkout excludes services/personalized/install); **§9.2 citation-battery baseline footnote** (API runs 2026-10-29/30 — Maza 3/3, PJ P1 1/4 Claude-only; no causation claims). Research mirrors under `research-sources/`. |

### Research inputs (doctrine, not commercial)
- Primary: `arealicehole/know` → `research/chat-ui-discovery-bible-section-refresh-20261029.md` (workspace mirror: `research-sources/chat-ui-discovery-bible-section-refresh-20261029.md`)
- Supporting: `ai-readable-web-pj-maza-competitor-audit-t_64b6e1c1.md`, `avp-causal-verification-multi-engine-citation-battery-t_70d29518.md`
- Measurement companions (not doctrine substitutes): `multi-engine-citation-battery-first-execution-t_49a81e49.md`, `citation-battery-open-cells-delta-t_528f7761.md`, `pj-gmc-ucp-eligibility-20261029.md`
- Live specimens (2026-10-29/30 probes): printjunkieaz.com, mazahalalfood.com
- Commercial Agent Visibility Pack (SKU/$) stays **outside** this Bible: Dick Jones AVP pack under dick-jones artifacts only

### Edition rules
- **Complete resource tool** — full prior technical body retained.
- **Additive only** — no silent cuts of restaurant, news/media, Core 30, UCP gates, Information Gain, Mattress Land, or checklists.
- **6-month restaurant playbook** retained in §4 until Frank explicitly peels it to a separate playbook.
- **Do not claim** placement guarantees, invented citation battery numbers, or “llms.txt caused Print Junkie ChatGPT leads” (PJ inbound existed before prod `/llms.txt`).

---

## The Core Thesis

The SEO landscape has split into two parallel systems that must be optimized simultaneously:

- **Human Layer**: Rewards first-hand experience, originality, and genuine "Information Gain."
- **Agent Layer**: Rewards clean, structured, machine-readable data (UCP manifests, schema, specs.json, first-party feeds) **and** retrieval into big-box **chat UIs** (ChatGPT, Claude, Gemini / AI Overviews, Perplexity, Grok, peers).

**Golden Rule (2026)**  
> Humans want stories and proof. Agents want structured data. The winners serve both while adding real new information that didn’t exist before.

The December 2025, March 2026, and May 2026 Core Updates have all reinforced this direction. Generic or rephrased content is being systematically demoted.

### Chat-UI / answer-engine discovery (3.0 add)

Local businesses increasingly win **inbound from chat products**, not only classic blue links. Treat chat discovery as first-class doctrine.

**Operator principles (late 2026)** — confidence tags from field research, not marketing:

| ID | Principle | Confidence |
|----|-----------|------------|
| P1 | Local answers are **retrieval-first** — if crawlers cannot fetch you, chat cannot cite you | verified |
| P2 | `robots.txt` Allow is necessary but **not sufficient** — answer-engine UAs must get **HTTP 200 at the edge** | verified |
| P3 | **Intent match / Information Gain** correlates more with citation than classic authority proxies alone | likely (correlational studies; not lab causation) |
| P4 | **Entity clarity** (NAP, typed schema, resolving `sameAs`) disambiguates which business is meant | likely |
| P5 | **Reviews and directories** (GBP + Yelp and peers) feed recommendations — directories can be **answer sources**, not only citation badges | likely |
| P6 | Ship **two surfaces**: **identity** (who/where/hours/category) + **transaction** (prices, offers, how to buy/book/call) | verified (live PJ + Maza) |
| P7 | Machine manifests (`llms.txt`, UCP, specs/feeds) should be **linked from first-contact** pages agents actually fetch | likely |
| P8 | `/llms.txt` is a useful **convention** and identity cheat-sheet — **not** a documented universal ranking input | verified |
| P9 | Optimize for **multi-engine parity** (ChatGPT, Claude, Perplexity, Gemini/AI Overviews, Grok) — one-bot stacks are incomplete | likely |
| P10 | Commerce path in chat is mostly **discover / compare / hand off** to the merchant — not universal in-chat checkout | verified (vendor direction 2026) |

**Do not claim:** any single lever guarantees ChatGPT (or any engine) will recommend you. Measure (§9). Correlation ≠ causation.

### Which section applies?

| Site type | Primary ruler | Do **not** force |
|-----------|---------------|------------------|
| Local service / restaurant | §3 Local SEO + §4 Restaurant case study | Full news-desk checklist |
| First-party e-com / checkout | §1.1 UCP + §6 ACO | Fake media schema if you only sell SKUs |
| **News desk / headline aggregator / Bill Watch / legislative tracker** | **§8 (this edition)** | Core 30 neighborhood pages, Menu schema, full UCP unless checkout exists |
| Hybrid (media + affiliate directory, no checkout) | **§8** + affiliate rules in §8.8 | Full Product catalog UCP |

---

## 1. Technical Architecture

### 1.1 Universal Commerce Protocol (UCP)

**Status:** Live as a **formal commerce standard** (Google-led UCP line; public spec activity from **January 2026** / NRF era) — not merely a TriCon house file shape. Still expanding across merchant and agent surfaces.

**Gate:** Full UCP at `/.well-known/ucp` + checkout/fulfillment specs is **mandatory only when first-party checkout exists**. Media-only and aggregator sites: see **§8.9 UCP gate** — ship `llms.txt`, feeds, and structured indexes; do **not** fail a news node for missing merchant UCP.

Merchants (with checkout **or clear order/handoff paths**) should expose a machine-readable manifest at `/.well-known/ucp` that declares capabilities for discovery, checkout **or handoff**, fulfillment, discounts, and order management where real.

**2026 commerce reality (chat UIs)**
- **Google / Gemini / AI Mode path:** UCP + **Merchant Center** product surfaces matter for discovery and agent-ready offers. Self-hosted UCP is the on-site half; official merchant feeds are the network half when eligible.
- **ChatGPT commerce direction (2026):** emphasis shifted toward **discovery, comparison, and handoff** to the merchant (including **ACP / product feed** style paths). Instant Checkout was **deprioritized / moved toward Apps-style experiences** — do **not** write doctrine as “every chat has universal in-chat checkout,” and do **not** write it as “checkout is dead forever.” Merchants still need honest order paths (hosted checkout links, phone, marketplace, booking).
- **Honesty rule (TriCon):** Never invent cart/Stripe/checkout in UCP. Restaurants may declare SkyTab pickup, Slice/DoorDash delivery, phone. Local services may declare Stripe payment links, quote forms, phone. Media: see §8.9.

**Requirements (checkout / handoff sites)**
- Real products/services with **real prices** when prices exist
- Real-time or regularly synced inventory/availability when you claim it
- Clear order or inquiry URLs (hosted checkout, booking, phone `tel:`, marketplace)
- Support for Agent Payments Protocol (AP2) tokens **when you actually take agent payments**
- Rate limiting to prevent agent abuse (token-aware where possible)

**Self-hosted vs official feeds:** On-site `/.well-known/ucp` and `/api/specs` are **not** substitutes for **OpenAI ACP / Google Merchant Center** enrollment when those programs apply. Ship both layers when the business is eligible; never pretend a local JSON route is the network feed.

### 1.2 Schema Strategy

Agents and answer engines use structured data as **identity + quotable facts** (who, where, what, price, hours, actions) — a machine-readable label on real content. Schema is **not** a substitute for Information Gain, and it is **not proven as a chat-citation ranker** by itself.

**Framing (3.0)**  
- **Do:** typed entity (`LocalBusiness` / `Restaurant` / `Organization` / `NewsMediaOrganization`), complete NAP, `openingHours`, `Offer`/`price` when true, `Menu`/`MenuItem` for restaurants, FAQ that matches visible Q&A, resolving `sameAs`, `aggregateRating` only with real reviews.  
- **Do not:** treat JSON-LD as a ranking cheat code; ship schema that contradicts the HTML; invent addresses, ratings, or checkout.

**Recommended schemas by use case:**
- E-commerce / Data sites → `Dataset` + `Product`/`Offer` where true
- Restaurants & Local Services → `Restaurant` or `LocalBusiness` + `Menu` + `MenuItem` + `Offer` prices when published
- **News desks / aggregators / legislative trackers → `NewsMediaOrganization` + honest page types (`CollectionPage` / `ItemList` / `Dataset`); see §8.1–§8.2**
- Transactional actions → `potentialAction` (`ReserveAction`, `BuyAction`, `OrderAction`)
- Voice optimization → `Speakable`
- Disambiguation → `sameAs` to **authoritative profiles that resolve** (official social, GBP/Maps URL, major directories). Dead links fail the graph.

### 1.3 Crawler Access

**(3.0) Multi-engine + edge-200**

Do not accidentally block answer-engine crawlers. Blocking retrieval bots reduces visibility in chat UIs and agentic surfaces.

**Split policy (explicit decision, not an accident)**

| Class | Examples | Default TriCon posture |
|-------|----------|------------------------|
| **Answer / browse / citation** | `OAI-SearchBot`, `ChatGPT-User`, `Claude-SearchBot`, `Claude-User`, `PerplexityBot` (+ user variants when documented), keep `Googlebot` unblocked | **Allow** + prove **edge HTTP 200** |
| **Training / grounding extended** | `GPTBot`, `ClaudeBot`, `Google-Extended`, `Applebot-Extended`, `CCBot`, `Bytespider` | **Allow unless Frank/client policy says otherwise** — record the choice |
| **Grok / xAI** | Identify real UA from **your** logs when possible | Default-allow if unknown; do **not** claim Grok honors robots blocks without evidence. X/Twitter presence is a **separate channel**, not a robots.txt substitute. |

**Edge acceptance test (HARD)**  
`robots.txt` Allow is **necessary but not sufficient**. Cloudflare and other WAFs can challenge or block AI bots even when robots allows them.

- Cloudflare note (vendor 2026-09-15 era defaults — **narrow, not global**): **new domains** may default Training/Agent classes blocked **on pages that display ads**; **Search** class tends to stay allowed. Do **not** teach “Cloudflare blocks all AI for everyone.”  
- Operator rule that always holds: curl key URLs with answer-engine UAs and require **200** on home + money pages. Fix WAF/Bot Fight/custom rules before blaming content.

**Minimum robots posture (answer visibility):** Allow `OAI-SearchBot`, `ChatGPT-User`, `Claude-SearchBot`, `Claude-User`, `PerplexityBot`; do not block `Googlebot`. Prefer explicit Allow stanzas over silence when you have had WAF incidents.

### 1.4 First HTML / SSR rule (all content products)

**Hard rule for agent and crawler discovery:** Primary listing surfaces (homepage, section hubs, day archives) must ship the **headlines / items in the first HTML response**. Client-only injection of the corpus from a third-party host (GitHub Gist, raw CDN, etc.) is an **agent fail** even if humans eventually see content. Day archives with full link HTML are the **pass pattern**. Detail: **§8.4**.

### 1.5 `/llms.txt` (identity cheat-sheet)

Ship `/llms.txt` (optional `/llms-full.txt`) as a **hand-authored** identity brief: legal/trading name, NAP, hours, categories/services, phone, key URLs, agent endpoints (`/.well-known/ucp`, feeds, specs), citation/training stance.

| Truth | Doctrine |
|-------|----------|
| Useful convention widely adopted by operator sites | **Yes — ship it** on agent-facing local, restaurant, media, and catalog nodes |
| Documented universal ranking input across ChatGPT/Claude/Gemini/Perplexity/Grok | **No** |
| Required to receive chat inbound | **No** — Print Junkie AZ received ChatGPT leads **before** production `/llms.txt` existed |
| Replaces schema, SSR content, prices, or edge-200 | **No** |

Media-specific profile requirements remain in **§8.6**.

### 1.6 Two surfaces: identity + transaction

Full agent-visibility for local businesses is **two axes**. Score and ship them separately.

| Axis | Job | Typical artifacts |
|------|-----|-------------------|
| **Identity** | Who / where / when / category / proof | `/llms.txt`, typed LocalBusiness/Restaurant, NAP, hours, `sameAs`, FAQ, ratings, photos |
| **Transaction** | What it costs / how to act | `/.well-known/ucp`, visible `$` in HTML, `Offer`/`MenuItem` prices, `/api/specs` or feeds, order/book/call URLs |

**Live pattern (TriCon specimens, 2026-10-29):** Maza skewed strong on **identity**; Print Junkie skewed strong on **transaction** (full UCP + prices) and still earned ChatGPT inbound without llms.txt historically. Prefer **both axes**; do not refuse to explain inbound because one axis is incomplete.

### 1.7 Official network feeds ≠ self-hosted specs

| Surface | Role |
|---------|------|
| `/api/specs`, `/specs.json`, on-site UCP | First-party machine cards agents can fetch from your host |
| Google Merchant Center / free product listings | Network distribution into Google surfaces / AI Mode paths when eligible |
| OpenAI ACP (or successor product feeds) | Network distribution into ChatGPT commerce/discovery when eligible |

Enroll when the business qualifies. Document eligibility gaps honestly.

**GMC / Google UCP eligibility (operator doctrine — local print/signage/service class, 2026-10)**

Do **not** conflate **on-site** `/.well-known/ucp` with **Google Merchant Center + Google UCP-powered checkout** (AI Mode / Gemini “Buy”).

| Path | Typical local print / signage / custom-service shop | Rule |
|------|-----------------------------------------------------|------|
| **On-site UCP** | **Yes** — primary agent transaction layer when you publish prices + honest handoff (Stripe links, phone, quote) | Ship and keep inventory-honest |
| **Merchant Center free product listings** | **Partial** — only **SKU-shaped** fixed products (stable price, image, product URL, shipping/returns). Custom quote jobs and pure install labor are a poor fit | Optional thin catalog of fixed packs only |
| **Google UCP checkout in AI Mode / Gemini** | **No for core custom/service revenue** | Google’s MC UCP prep marks **services**, **personalized goods**, **installation/setup bundles**, and **special/freight shipping** ineligible for agentic checkout |
| **GBP + directories + on-site identity/transaction** | **Yes** | Still the local discovery backbone; chat recs are not “GMC magic” |

**Do say:** “We ship an on-site commerce manifest agents can fetch.” / “SKU-like products *may* go to free listings if they meet feed rules.”  
**Do not say:** “Sign up for Merchant Center UCP and chat will check out your wraps.” / “Every local service is GMC/UCP-checkout eligible.” / free-listing placement guarantees.

Companion research: `research-sources/pj-gmc-ucp-eligibility-20261029.md` (know: `pj-gmc-ucp-eligibility-20261029.md`). Primary Google docs: Merchant Center UCP help, product data spec, free listings; developer guide *Prepare your Merchant Center account* (service/personalized/install restrictions).

---

## 2. Content Strategy

### 2.1 Information Gain (The 2026 Defensive Standard)

**(3.0) Chat citation note**

The March and May 2026 Core Updates made “Information Gain” one of the strongest ranking signals. Content that merely rephrases what already ranks well is now heavily penalized.

**Chat-UI note (3.0):** Multi-engine citation studies (e.g. large vendor correlational analyses of near-me / local prompts) find **intent match and unique useful facts** associate with citation more than classic authority proxies alone. Treat Information Gain as a **primary chat-discovery content principle** — re-prioritization backed by **correlational** evidence, **not** a claim of proven lab causation or a guaranteed #1 ranker. Still ship structured data; do not swap IG for schema theater.

**Winning content must contain one or more of the following:**
- Proprietary data or original research
- First-hand experience and case studies
- Unique tests or experiments
- Novel analysis or insights
- **Local operators:** real prices, real turnaround/service area facts, real portfolio proof, real FAQs customers ask — not brochure adjectives

**Aggregators:** Pure outbound title farms **fail** this bar. Required gain patterns for media desks are in **§8.3** (original briefs, scoring/rewrite methodology, proprietary tallies, Bill Watch / legislative datasets).

### 2.2 The Mattress Land Structure (Human + Agent Optimized)

Every important page should follow this pattern:
1. Exact-match H1
2. Strong data hook in the first 100 words
3. Comparison tables or pricing (agents love structured data)
4. FAQPage schema answering real user questions
5. Visual proof (photos/videos of real work)

**Media exception:** Homepage and section hubs may lead with ranked/tallied headlines and methodology instead of “pricing”; still ship data hook + FAQ where questions are real (e.g., how Bill Watch scores bills). See **§8**.

---

## 3. Local SEO

### 3.1 The Core 30 Structure

Build a knowledge graph node:
- 1 Homepage
- 3–4 Category pages
- 25–30 specific service pages (one per real offering in your Google Business Profile)

**Non-goal for pure media:** Core 30 neighborhood/service pages and GBP restaurant photo rules do **not** apply unless the brand actually runs local service/restaurant. Auditors: use **§8.12 Explicit non-goals**.

### 3.2 Google Business Profile Optimization

**(3.0) Directories as answer sources**

GBP remains mandatory local infrastructure. **3.0 add:** major directories are no longer only “citation badges” — some are **direct answer sources** inside chat UIs.

**Google Business Profile**
- Upload **100+ high-quality photos** and continue adding new ones weekly
- Maintain perfect NAP consistency (site ↔ GBP ↔ directories)
- Respond to every review within 24 hours
- Post at least weekly
- Enable all relevant attributes
- Ensure the **website URL and phone** on GBP match the site the agents fetch

**Directories → chat (scoped truth)**
- **Yelp ↔ ChatGPT** integrations surface ratings/reviews and, where enabled, actions such as **Request a Quote** (services) and restaurant flows via **Yelp Guest Manager** (Reservations/Waitlist — **US/CA restaurants on that product**, not every local business worldwide).
- Optimize **real** profiles you own: accurate categories, photos, hours, response hygiene.
- Do **not** paste third-party marketing multiples (e.g. “3.4× mentions”) into client promises — those are vendor claims, not TriCon causal proof.
- `sameAs` may list Yelp/GBP/Maps/Facebook/Instagram **when URLs resolve**; empty or broken sameAs is a graph defect.

### 3.3 Review Velocity

Fresh reviews (last 30–60 days) now carry more weight than total review count. Build systems that generate consistent, genuine reviews.

**Chat-UI note:** Review freshness and rating quality feed both Map Pack behavior and **directory-powered chat recommendations**. Prefer genuine velocity systems (QR/SMS/email after service) over bought reviews. Star-threshold marketing claims from agencies are **not** Bible proof — treat as hypotheses to measure.

---

## 4. Restaurant Local SEO Case Study (2026)

### Objective
Dominate the local Map Pack, Google Lens, AI Overviews, and voice assistants while driving direct reservations.

### 6-Month Playbook

**Phase 1 (Months 1–2): Foundation**
- Fully optimize Google Business Profile (categories, attributes, hours, menu upload)
- Upload 100+ high-quality, appetizing photos (many geo-tagged)
- Implement `Restaurant` + full `Menu` schema + `ReserveAction`
- Fix NAP consistency across all directories

**Phase 2 (Months 2–4): Velocity Engine**
- Deploy table QR codes + automated SMS review requests
- Target 8–15 new Google reviews per month
- Post to GBP weekly
- Add new photos every week

**Phase 3 (Months 4–6): Scale**
- Build neighborhood-specific landing pages
- Add voice-action schema for reservations
- Monitor GBP Insights and AI Overview appearances

### Expected Results
- 3–7× increase in Map Pack visibility and phone calls
- Strong performance in visual search (Google Lens)
- Improved presence in AI Overviews and voice queries
- Greater independence from third-party delivery platforms

---

## 5. Visual Search & Multimodal Optimization

- Use high-resolution, well-lit images (WebP/AVIF preferred)
- Enable EXIF GPS on business photos when appropriate
- Write descriptive alt text and filenames that match local intent (**accessibility + classical image SEO** — do **not** treat alt text as a proven chat-citation lever)
- Upload geo-tagged images to Google Business Profile
- Prefer **real work / real food** photos over stock on proof pages
- **Media desks:** prefer self-hosted or first-party cached heroes; third-party CDN hotlinks are brittle and rights-risky (**§8.11**)

---

## 6. Agentic Commerce Optimization (ACO)

- Structure product data with utility signals (availability, shipping, returns, ratings, sustainability)
- Maintain a `specs.json` / `/api/specs` file as the source of truth for product attributes (and link it from `llms.txt` / UCP)
- Use honeypot techniques to trap malicious scrapers while allowing legitimate agents
- Align on-site specs with **official network feeds** when enrolled (§1.7)
- **Media-only sites:** optional lightweight `specs.json` may declare the **content product** (update frequency, license, ad/affiliate products) **without** fake checkout — **§8.9**. Full merchant ACO applies only with first-party commerce.

---

## 7. Current Checklist (v3.0-DRAFT — June 2026 + media patch + chat-UI)

**Universal**
- [ ] Every major page contains genuine Information Gain (or media-specific gain per §8.3)
- [ ] Answer-engine crawlers **Allowed** in robots.txt (multi-engine list §1.3)
- [ ] **Edge-200:** home + money pages return HTTP 200 to answer-engine UAs (WAF verified)
- [ ] No stock photography on key proof pages (local/service)
- [ ] Primary listings present in **first HTML** (not JS-only corpus inject)
- [ ] `/llms.txt` present on agent-facing nodes (identity brief — not a ranking promise)
- [ ] Schema = entity + quotable facts matching visible HTML (not schema-only theater)

**Local / restaurant (when applicable)**
- [ ] **Identity axis:** typed LocalBusiness/Restaurant, NAP, hours, resolving sameAs, FAQ/ratings as real
- [ ] **Transaction axis:** visible prices where you have them; UCP and/or specs with honest handoff (checkout links, phone, marketplace) — **if** you take orders
- [ ] UCP manifest live and inventory-synced **if checkout exists**; else honest non-checkout handoff or §8.9 skip
- [ ] Google Business Profile has 100+ photos + weekly uploads; site URL/phone match
- [ ] Major directory profiles you claim (e.g. Yelp) are accurate — directories can feed chat answers
- [ ] Review velocity system is active and consistent
- [ ] Full menu + reservation schema implemented (restaurants)
- [ ] Official network feeds enrolled when eligible (GMC / ACP-class) — tracked separately from on-site specs; **GMC free listings = SKU-shaped products only**; **Google UCP checkout excludes services / personalized / install** (§1.7) — do not enroll custom quote lines as if they were retail SKUs

**News desk / aggregator / legislative tracker (when applicable — §8)**
- [ ] `NewsMediaOrganization` (+ `sameAs`, `publishingPrinciples`) on home
- [ ] Honest page types: no fake `NewsArticle` for link-only roundups
- [ ] Homepage + hubs SSR headlines in first HTML
- [ ] First-party RSS and/or JSON (`/data` or `/api`) for headline + bill corpora
- [ ] `/llms.txt` media profile live
- [ ] Sitemap index retains historical day/section URLs (not “last 2 days only”)
- [ ] About, Contact, Privacy, Terms, corrections/ethics (YMYL-adjacent politics)
- [ ] Affiliate: FTC disclosure + `rel=sponsored` where paid; directory ≠ Product catalog
- [ ] Full UCP **only if** first-party checkout; else do not fake merchant manifests
- [ ] Edge-200 for answer bots on home + key hubs

---

## 8. News desks, headline aggregators & legislative trackers

**Use this section when** the product is primarily: political or vertical **news desk**, **headline aggregator**, **day archives of outbound links**, **Bill Watch / legislative tracker**, and/or **affiliate marketplace alongside media** — **without** (or not primarily) first-party restaurant/local service or checkout.

**Do not audit these sites primarily against** §3 Core 30, §4 restaurant GBP/menu playbooks, or full merchant UCP. Cross-apply §1–2 and §5–6 only where marked.

**Reference specimen (2026-10-14):** Root & Rifle — `https://rootandrifle.com/` — static news desk + Bill Watch + affiliate marketplace; code truth on GitHub `arealicehole/rootandrifle-site` (`main`). Useful as a worked gap list, not as a pass model.

### 8.1 Entity model

**Required (home / org graph)**
| Property / type | Rule | Pass | Fail |
|-----------------|------|------|------|
| `@type` | `NewsMediaOrganization` (or `NewsMediaOrganization` + `Organization`) | Present on home JSON-LD | Generic `WebSite` only, or restaurant types |
| `name`, `url` | Stable brand identity | Match visible brand + canonical host | Mismatch / missing |
| `sameAs` | Authority graph | Wikipedia, official social, Congress/data partners as relevant | Empty |
| `publishingPrinciples` | URL to ethics/corrections/editorial | Live first-party page | Missing on YMYL-adjacent politics |
| `logo` | ImageObject | Self-hosted favicon/logo | None |
| Bylines | `Person` (+ `author`) when named writers exist | Real people, sameAs when public | Fake Person nodes or stolen bylines |

**Corpus / tracker extras**
- Bill Watch / legislative datasets → `Dataset` (name, description, `isAccessibleForFree`, distribution URL, temporal coverage).
- Section hubs → `CollectionPage` with `mainEntity` `ItemList`.

### 8.2 When NOT to use `NewsArticle`

| Content shape | Correct types | Incorrect |
|---------------|---------------|-----------|
| Link-only roundup / headline list you do not own full text for | `CollectionPage` + `ItemList` of `ListItem` (url, name, position); optional `WebPage` | `NewsArticle` with invented `articleBody` |
| Original brief / analysis you publish in full | `NewsArticle` or `Article` with real body, date, author | `ItemList` only with no body URL |
| Bill record / tracker row | `Dataset` item, or structured custom + clear `ItemList`; cite Congress.gov etc. | `NewsArticle` pretending primary reporting |
| Affiliate directory card | `ItemList` / offer listing with disclosure | `Product` catalog implying you stock/ship SKUs you don’t |

**Honesty rule:** Do not fake full-text articles you don’t own. Aggregators earn trust by labeling curation and adding gain (§8.3), not by laundering third-party copy into `NewsArticle`.

### 8.3 Information Gain bar for aggregators

Pure outbound title farms **fail** 2026 Information Gain.

**Minimum gain (ship at least two):**
1. **Original briefs** — 1–3 sentence desk framing, not source lede paste.
2. **Scoring / rewrite methodology** — documented, linked from `/llms.txt` and About (why this headline rank, what “NEW BILL” means).
3. **Proprietary tallies** — counts, streak stats, topic heat, vote-line aggregates you compute.
4. **Bill Watch / legislative datasets** — normalized bill IDs, status, desk tags, first-party JSON/CSV.
5. **Corrections & selection log** — what was added/removed and why (E-E-A-T).

| Pattern | Verdict |
|---------|---------|
| Titles + outbound links only, no method, no tallies | **FAIL** |
| SSR lists + methodology page + Dataset for bills + original section briefs | **PASS** direction |
| Full original reporting + schema Article | **PASS** (still keep feeds/SSR) |

### 8.4 SSR / first-HTML rule (HARD FAIL)

| Surface | Pass | Fail |
|---------|------|------|
| Homepage | Headlines (or top-N) in **first HTML** response; progressive enhance OK | ~empty shell; corpus loaded only via client JS from Gist/raw third party |
| Day archives | Full link lists in static/SSR HTML | Empty until JS |
| Section hubs | Key items in first HTML | Client-only |

**Why hard:** Agents and many crawlers do not execute your Gist inject. Humans may “see” a working site while answer-engines and auditors score a thin shell.

**Pass pattern:** Build pipeline writes day HTML + section HTML + home snapshot at generate time; JS may refresh but must not be the sole carrier.

### 8.5 Feed requirements (HARD FAIL if absent)

| Feed | Requirement |
|------|-------------|
| RSS and/or Atom | First-party path (e.g. `/feed.xml`); items match visible desk |
| JSON corpus | First-party `/data/*.json` or `/api/*` mirrors of headlines + bills |
| Discovery | Linked from home, `llms.txt`, and optionally `Link` headers |

**Anti-pattern:** Sole source of truth on `gist.github.com` / third-party raw hosts. Third-party mirrors may exist as backup; **agent discovery must not depend on them**.

### 8.6 `/llms.txt` profile for media

Ship `/llms.txt` (and optional `/llms-full.txt`) including:
- **Purpose** — desk mission, beat, non-goals
- **Sections** — URL map (war, surveillance, bill-watch, marketplace…)
- **Citation / training stance** — what agents may quote; licensing
- **Contact** — editorial + legal
- **Key URLs** — home, feeds, data, about, corrections, sitemap
- **Update cadence** — how often home/day archives rebuild

Missing `/llms.txt` on a 2026 media/agent-facing desk = **fail** for agent layer (even if humans browse fine). Same honesty as §1.5: required operator artifact for media nodes — **not** a promise of citation rank.

### 8.7 Sitemap policy for daily archives

| Rule | Pass | Fail |
|------|------|------|
| Sitemap index | `sitemap.xml` indexes day and/or section sitemaps | Single flat file that drops history |
| Retention | Historical day URLs remain listed while content is public | “Last 2 days only” while older days still HTTP 200 |
| Consistency | Every public day/section URL reachable appears in sitemap set | Orphan archives |

Daily news products are temporal graphs — agents need durable URLs, not a sliding window that erases the corpus from discovery.

### 8.8 Affiliate marketplace (alongside media)

- **FTC disclosure** clear on marketplace and item pages (not only footer legalese).
- **`rel="sponsored"`** (or `nofollow sponsored`) on paid/affiliate outbound links where applicable.
- **Schema honesty:** directory / `ItemList` ≠ first-party `Product` catalog with offers you fulfill.
- Do not imply inventory, cart, or UCP checkout you do not operate.
- Separate navigation IA: “News” vs “Market” reduces query confusion.

### 8.9 UCP gate (do not false-fail media)

| Condition | UCP / ACO expectation |
|-----------|------------------------|
| **No** first-party checkout | **Do not require** `/.well-known/ucp` merchant checkout capabilities. Still require `llms.txt`, feeds, schema indexes. Optional lightweight `specs.json`: content product, update frequency, license, ad/affiliate products — **no fake checkout**. |
| First-party checkout exists | Full §1.1 UCP + §6 ACO; inventory-synced; AP2 as applicable. |
| Affiliate-only outbound | Disclosure + sponsored rel; not full UCP. |

Auditors who fail every news node for missing UCP are mis-applying the e-com ruler.

### 8.10 E-E-A-T / YMYL-adjacent politics (mandatory pages)

Politics, law, health-adjacent desks are YMYL-adjacent. Ship and link:
- **About** — who runs the desk, funding, affiliation
- **Contact** — reachable editorial/legal
- **Privacy** / **Terms**
- **Corrections / ethics** (`publishingPrinciples` target)
- Prefer visible ownership over anonymous “news skin”

Absence of this cluster = **E-E-A-T fail**, independent of SSR.

### 8.11 Hotlink image policy

| Practice | Verdict |
|----------|---------|
| Self-host or first-party image cache of heroes/thumbs you have rights to use | **Pass** |
| Hotlink random third-party CDN heroes (publisher CNNs, etc.) as primary UI | **Fail risk** — brittle URLs, ToS/rights, layout breakage, weak brand entity signals |
| Source outlet thumb via formal API/license + cache | **Pass** if licensed and cached |

### 8.12 Explicit non-goals

Unless the brand **actually** operates these businesses, **do not** require:
- Core 30 neighborhood / local service pages (§3.1)
- GBP 100+ restaurant photo velocity (§3.2 / §4)
- `Menu` / `MenuItem` / `ReserveAction` schema
- Full merchant UCP + AP2 checkout (§1.1) for media-only
- Review-velocity SMS engines aimed at diners

Mis-applying restaurant/e-com checklists produces false fails and wasted engineering.

### 8.13 Normative auditor checklist (media)

Copy into audit tickets:

1. [ ] Site type gated as media/aggregator (not restaurant/e-com-primary)
2. [ ] `NewsMediaOrganization` + `sameAs` + `publishingPrinciples`
3. [ ] No fake `NewsArticle` on link-only roundups (`CollectionPage`/`ItemList`)
4. [ ] Information Gain: briefs / method / tallies / Dataset — not title farm
5. [ ] **HARD:** Homepage headlines in first HTML
6. [ ] **HARD:** First-party RSS/JSON data paths (not Gist-only)
7. [ ] `/llms.txt` media profile
8. [ ] Sitemap retains historical day/section URLs
9. [ ] About, Contact, Privacy, Terms, corrections
10. [ ] Affiliate disclosure + `rel=sponsored` as needed; directory honesty
11. [ ] UCP only if checkout; else no false fail
12. [ ] Image policy: prefer cache/self-host
13. [ ] Favicon/logo; unique titles; canonicals; robots Allow for AI bots

### 8.14 Worked example pattern — R&R-style desk (2026-10-14 snapshot)

**Specimen:** Root & Rifle (`rootandrifle.com`), static HTML + Python build, Bill Watch + section hubs + affiliate marketplace. Live code: `arealicehole/rootandrifle-site` on `main`.

| Signal (audit date 2026-10-14) | Result | Bible mapping |
|--------------------------------|--------|---------------|
| Unique titles + canonicals; `robots` Allow:/ | Pass | Universal hygiene |
| `NewsMediaOrganization` on home | Pass direction | §8.1 |
| Homepage ~4.6KB JS shell vs day archive ~25KB full links | **Fail** homepage SSR | §8.4 HARD |
| No `/llms.txt`, feeds, about/privacy/contact/terms, favicon | **Fail** | §8.5–8.6, §8.10 |
| Sitemap only ~2 day archives | **Fail** retention | §8.7 |
| Schema thin; no Dataset for corpus/Bill Watch; no FAQPage | **Fail** / partial | §8.1–8.3 |
| Corpus via third-party Gist headlines/bills as live path | **Fail** if sole source | §8.5 HARD |
| No first-party checkout | UCP **N/A** — do not false-fail | §8.9 |

**Target architecture (pass pattern for this class):**
1. Build job materializes `index.html`, `/day/YYYY-MM-DD.html`, section HTML with full `ItemList` markup.
2. Emit `/data/headlines.json`, `/data/bills.json`, `/feed.xml` from the same corpus.
3. Expand sitemap index for all retained days + sections.
4. Add `/llms.txt`, ethics/about/contact/privacy/terms, favicon, richer JSON-LD (`NewsMediaOrganization`, `Dataset`, FAQ only for real questions).
5. Cache images first-party; disclose affiliates; skip merchant UCP until checkout exists.

### 8.15 Agent-layer minimum viable media node

Ship this bundle before chasing vanity schema:

```text
/                       # SSR headlines in first HTML
/llms.txt
/feed.xml               # and/or feed.json
/data/headlines.json
/data/bills.json        # if legislative tracker
/sitemap.xml            # index → historical days
/about /contact /privacy /terms /corrections
/robots.txt             # Allow AI crawlers
```

Optional: `/specs.json` content-product declaration; `/.well-known/ucp` **only** with real checkout.

---

## 9. Verification & Measurement (chat discovery)

**Purpose:** Prove or falsify agent-visibility work with measurement — **never** with placement guarantees.

### 9.1 What we promise vs what we measure

| Promise (OK) | Guarantee (NOT OK) |
|--------------|--------------------|
| We ship the stack in this Bible | “ChatGPT will recommend you” |
| We run a citation / referral measurement protocol | Invented % lifts |
| We monitor bot edge-200 and server logs | “llms.txt will create leads” |

### 9.2 Minimum measurement pack

1. **Edge-200 bot matrix** — scripted curl with answer-engine UAs on `/`, money pages, `/llms.txt`, `/.well-known/ucp`, feeds.  
2. **Referral & conversation logs** — ChatGPT/Claude/Perplexity referrers, UTM, “how did you hear” when humans book. Prefer `utm_source=openai` (and peers) when you control outbound CTAs from agent surfaces.  
3. **Citation battery (protocol)** — fixed prompt set × engines (ChatGPT, Claude, Gemini, Perplexity, Grok); capture cited URLs/names; score hit/miss against a ground-truth fact table. *Run produces data for 3.0.1+ appendices — do not invent scores.*  
4. **Staged ablation (ethics)** — change **one** surface at a time on staging or short prod windows; never lie to customers during tests; restore quickly.  
5. **Search Console + analytics** — classic discovery still funds the same content system.

**Baseline executed (API proxy — not consumer-app UI; not causal proof)** — 2026-10-29 first execution + 2026-10-30 open-cell delta:

| Business / prompt | Engines with a cell | Result (honest) |
|-------------------|---------------------|-----------------|
| **Maza** — halal Med Chandler (P3) | Claude, Perplexity, Gemini | **3/3 HIT** — named/linked; Gemini quoted correct Frye Rd address |
| **Print Junkie AZ** — business cards Phoenix/Tempe (P1) | Claude, Perplexity, Gemini, Grok | **1/4 HIT** — Claude only; franchise/print chains led on the misses |
| ChatGPT product path / full 12×5 matrix / consumer apps | — | **Not** claimed complete in this edition |
| Causal ablation (which lever moves citations) | — | **Still open** — do not attribute leads to a single surface |

Companions: `research-sources/multi-engine-citation-battery-first-execution-t_49a81e49.md`, `research-sources/citation-battery-open-cells-delta-t_528f7761.md`. Method label on those runs: OpenRouter + native web plugins = **API baseline**, not proof of in-app ChatGPT/Claude/Gemini UI behavior.

### 9.3 Ground-truth fact table

Before any battery: list name, address, phone, hours, top 3 services/dishes + prices, primary CTA URL. Citations that invent facts are **failures** even if the brand is mentioned.

### 9.4 Do-not-claim list (standing)

- `/llms.txt` caused Print Junkie ChatGPT inbound (leads existed pre-prod llms.txt)
- Any single lever → guaranteed citation
- Vendor marketing multiples as TriCon causal proof
- Alt text as a chat-citation ranker
- Grok definitely honors robots Disallow (unverified)
- Placement guarantees of any kind

Detailed runnable battery design may live in research companions (`avp-causal-verification-multi-engine-citation-battery-t_70d29518.md`); this section is the **doctrine hook**.

---

## 10. Multi-engine agent layer (operator summary)

| Engine family | Retrieval notes | Commerce / actions notes |
|---------------|-----------------|--------------------------|
| **ChatGPT / OpenAI** | `OAI-SearchBot`, `ChatGPT-User`; browsing/citation when enabled | Discovery + handoff; ACP/feeds when enrolled; Instant Checkout deprioritized toward Apps — still need merchant order path |
| **Gemini / AI Overviews / Google** | `Googlebot` (+ Google extended tokens policy-separate) | UCP + Merchant Center paths matter |
| **Claude** | `Claude-SearchBot`, `Claude-User` (training `ClaudeBot` separate) | Citation/browse oriented; parity on SSR + facts |
| **Perplexity** | `PerplexityBot` | Aggressive citation UX — edge-200 + clear facts |
| **Grok / xAI** | Confirm UA via logs; default-allow | Do not assume robots compliance; X channel separate |

**Universal content stack** (all engines): Information Gain, SSR first HTML, entity schema, visible prices/CTA, identity+transaction manifests, directories/GBP hygiene, edge-200.

**Per-engine deltas** are mostly **bots, feeds, and WAF** — not a different Mattress Land.

---

## 11. Technical agent-visibility install checklist (no commercial pricing)

Use when implementing the stack on a client site. **Rates/SOW live in the separate Agent Visibility Pack sales materials — not here.**

1. Detect web root; branch from main; one PR.  
2. robots.txt multi-engine Allows (§1.3).  
3. Prove edge-200 with answer UAs (fix WAF if not).  
4. `/llms.txt` identity brief.  
5. JSON-LD entity + offers/menu as applicable (match HTML).  
6. `/.well-known/ucp` only with honest products/handoff; skip or content-product specs for media (§8.9).  
7. `/api/specs` or feeds when products/menu exist.  
8. Visible prices + phone/primary CTA in HTML.  
9. sameAs + GBP/directory NAP alignment.  
10. Mattress Land on money pages (§2.2).  
11. Measurement hooks (§9).  
12. Optional: GMC / ACP enrollment when eligible (§1.7) — SKU free listings only; no Google UCP checkout for services/custom/install.

Extant sites: prefer on-repo patch; else edge inject; microsite last resort — keep NAP identical to GBP.

---

## Final Principle

The winners in 2026 are not the best at “SEO” in the traditional sense.  
They are the best at being **authentically valuable to humans** while being **perfectly readable to machines** — including **big-box chat UIs**.

Build for both. Add real information. Stay consistent. Measure citations and referrals.  
**Pick the right ruler:** local, commerce, or media (§8) — and do not false-fail a news desk for missing a restaurant menu or a merchant UCP.

---

*This is the locked Third Edition (**v3.0**, 2026-10-30). Prior 2.1 archive: `SEO_BIBLE_JUNE_2026.md`. Update after major core updates or richer battery/ablation results as **3.0.1+** appendices — do not silently rewrite measured baselines. Media section retained in full from October 14, 2026; chat-UI discovery refresh October 29, 2026; GMC eligibility + battery baseline footnote October 30, 2026.*

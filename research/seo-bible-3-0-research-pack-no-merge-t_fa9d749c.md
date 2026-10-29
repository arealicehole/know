# SEO Bible 3.0 — Standalone Research Pack (pre-assembly input)

**Task:** t_fa9d749c (board: research, prefix [GIU])
**Prepared:** 2026-10-29 (UTC) — Bible v2.1 re-read this run; live baselines from t_64b6e1c1 (2026-10-28) and t_70d29518 (2026-10-29)
**Prepared by:** Geraldo v2.2
**Status:** RESEARCH INPUT ONLY — per Frank's course-correct (via tricon-web, 2026-10-29): this pack is the input *before* any Bible 3.0 assembly. It is **not** a merge of Dick's draft and **not** a directive to Romulus to start writing.
**Hard constraint honored:** No AVP commercial pricing appears anywhere in this document or in anything it recommends for the Bible.

**Authoritative sources (all read this run):**
- `SEO_BIBLE_JUNE_2026.md` (v2.1, 2026-10-14) — `/home/ice/know/SEO_BIBLE_JUNE_2026.md` — the current official Bible
- `t_64b6e1c1` — AI-readable web audit, PJ vs Maza + 15 local competitors — `/home/ice/know/research/ai-readable-web-pj-maza-competitor-audit-t_64b6e1c1.md`
- `t_70d29518` — AVP causal verification: multi-engine citation battery, ablation matrix, robots template, extant-site paths — `/home/ice/know/research/avp-causal-verification-multi-engine-citation-battery-t_70d29518.md`
- `t_505c6bc5` — news-desk / §8 provenance — `/home/ice/know/research/seo-bible-news-desk-section-t_505c6bc5.md`
- Dick 3.0 draft: **non-authoritative outline only, explicitly excluded as doctrine source**

---

## EXEC SUMMARY (for Frank — read this first)

1. **Your course-correct is correct and this pack respects it.** Research went through Dick by mistake; Dick's 3.0 draft is commercial packaging, not doctrine. The doctrine pipeline is: **Geraldo research (this pack) → Romulus assembly (only when you say proceed)**. Romulus card `t_7b4116e0` stays blocked until you authorize.

2. **Current Bible (v2.1, June/October) = strong resource tool + one time-bound action plan inside it.** §1–§7 and §8 are doctrine (timeless patterns, checklists, rules). The one exception is **§4's "6-Month Playbook"** — a dated, milestone-driven action plan for one restaurant case. That's the structural problem the split resolves.

3. **What is newly true since June that MUST be added (all grounded in our own verified work, not Dick):**
   - **Multi-engine bot list with the answer/training split** — OAI-SearchBot, ChatGPT-User, Claude-SearchBot, Claude-User, PerplexityBot, Perplexity-User, Googlebot (must-allow) vs GPTBot, ClaudeBot, Google-Extended, Applebot-Extended, CCBot, Bytespider (training policy = separate data-governance decision). Grok/xAI: identify empirically, default-allow.
   - **WAF/edge override is a standing gotcha** — a correct robots.txt is a no-op behind Cloudflare "block AI"/Super-Bot-Fight; acceptance = 200 at the edge per bot UA.
   - **Verification & measurement layer** — the Golden Rule's missing half: ship the stack, then prove it with a multi-engine citation battery; the before/after delta is the evidence, not the presence of files. No placement guarantees; measurement is the proof.
   - **Two-surface model** — identity/discovery (llms.txt, typed schema, address, hours, rating, sameAs — Maza) vs transaction/offer (UCP products/tiers, specs, visible prices — PJ). A fully AI-readable site needs both; measure each separately.
   - **Extant-site install patterns** — three fidelity paths: own-repo full patch / edge (Cloudflare Workers) injection / parallel microsite (last resort).
   - **Ground-truth fact table + fact-accuracy scoring** as part of every audit.

4. **Never drop (no-loss floor) — see §2 checklist, all 26 headings verified present in v2.1:** the whole restaurant technical stack (§3 + §4 core: Core 30, GBP, review velocity, menu SSR, local-intent pages), §5 visual search, §6 ACO/UCP, §7 checklist, all of §8 news desks (8.1–8.15 incl. the 2026-10-14 snapshot), the Core Thesis + two-layer model + Golden Rule, llms.txt, first-HTML/SSR rule, UCP, information-gain 2026 standard.

5. **Playbook split recommendation: split it.** Keep §4's *technical rules* (menu SSR, Core 30, GBP, review velocity, local pages) in Bible core; move the dated *6-Month Playbook* into a separate "Restaurant Implementation Playbook" resource. Rationale: Frank wants the Bible = complete resource tool (timeless); playbooks are time-bound actions for one vertical. A future 2.1→3.0 merge that deletes the playbook section is *not* a loss — the playbook lives on as its own doc and the Bible references it by name. Everything else in the no-loss checklist is non-negotiable.

6. **Explicit: no AVP pricing in doctrine.** The Bible contains and must continue to contain **zero** commercial pricing, package names, or offer structures. AVP commercial framing lives in the SOW/one-pager (separate workstream, t_10e1b346 line) and is measurement-only ("here is the before/after battery"), never placement guarantees or price points.

7. **Go / no-go: GO — research is sufficient to authorize Romulus full-merge 3.0.** Every required doctrinal input in §3 above is grounded in verified work (live probes 2026-10-28/29 + prior GIU reports). Nothing in the assembly requires further *research*; the open items are *execution* items (running the baseline battery on live engines, staging availability, GA4/bot-log access, Grok UA pinning) and none of them block the *writing* of 3.0 — they block *claiming* verification results. Recommendation: you authorize Romulus `t_7b4116e0` to proceed with **full v2.1 body + additive sections, zero deletions**, using this pack's no-loss checklist as the merge gate. If you want the battery's *data* in 3.0, that's a later patch (3.0.1) — not a reason to hold the rebuild.

---

## 1) Inventory of the current official Bible (v2.1 = "JUNE 2026", last edited 2026-10-14)

Source of truth: `/home/ice/know/SEO_BIBLE_JUNE_2026.md` (420 lines, 21.6 KB). Changelog in file: 2.0 (2026-06-23) → 2.1 (2026-10-14, +§8 News Desks & Media).

### 1.1 Section map with doctrine classification

| # | Section | Kind | Classification |
|---|---------|------|----------------|
| 1 | Title + version + changelog | meta | resource |
| — | **The Core Thesis** (agent + human, two-layer model) | doctrine | **DOCTRINE (load-bearing)** |
| — | **Which section applies?** (decision table: restaurant / product / both / media) | doctrine | **DOCTRINE (routing rule)** |
| 1 | Technical Architecture | doctrine | **DOCTRINE** |
| 1.1 | UCP (`.well-known/ucp`, ucp/1.0, `lastUpdated`) | doctrine | **DOCTRINE** |
| 1.2 | Schema Strategy (specific types, NAP, aggregateRating, sameAs) | doctrine | **DOCTRINE** |
| 1.3 | Crawler Access (robots: GPTBot, OAI-SearchBot, Google-Extended, PerplexityBot, CCBot) | doctrine | **DOCTRINE (needs 3.0 multi-engine update)** |
| 1.4 | First HTML / SSR rule (HARD FAIL) | doctrine | **DOCTRINE (non-negotiable)** |
| 2 | Content Strategy | doctrine | **DOCTRINE** |
| 2.1 | Information Gain — 2026 Defensive Standard | doctrine | **DOCTRINE** |
| 2.2 | Mattress Land Structure (5-part human+agent format) | doctrine | **DOCTRINE** |
| 3 | Local SEO | doctrine | **DOCTRINE** |
| 3.1 | Core 30 Structure | doctrine | **DOCTRINE (non-negotiable)** |
| 3.2 | Google Business Profile Optimization | doctrine | **DOCTRINE (non-negotiable)** |
| 3.3 | Review Velocity (target: 12–24 new/mo) | doctrine | **DOCTRINE (target is a benchmark, keep in core)** |
| 4 | Restaurant Local SEO Case Study (2026) | hybrid | **SPLIT — see §4** |
| 4a | Objective (Maza-style Chandler, 1000 customers/mo, 6 months) | case framing | case (fine to keep as the case-study anchor) |
| 4b | **6-Month Playbook** (M0/M1/M2–M3/M4–M6 milestones) | action plan | **TIME-BOUND ACTION PLAN — extract** |
| 4c | Expected Results (benchmarks) | benchmarks | keep (benchmarks = doctrine-grade) |
| 5 | Visual Search & Multimodal Optimization | doctrine | **DOCTRINE** |
| 6 | Agentic Commerce Optimization (ACO) — UCP, AP2, no-cart | doctrine | **DOCTRINE (non-negotiable)** |
| 7 | Current Checklist (June 2026 + media patch) | checklist | **DOCTRINE (the operational core of the resource tool)** |
| 8 | News desks, headline aggregators & legislative trackers | doctrine | **DOCTRINE (added 2.1, non-negotiable)** |
| 8.1 | Entity model (NewsMediaOrganization, etc.) | doctrine | keep |
| 8.2 | When NOT to use `NewsArticle` | doctrine | keep |
| 8.3 | Information Gain bar for aggregators | doctrine | keep |
| 8.4 | SSR / first-HTML rule (HARD FAIL) | doctrine | keep |
| 8.5 | Feed requirements (HARD FAIL if absent) | doctrine | keep |
| 8.6 | `/llms.txt` profile for media | doctrine | keep |
| 8.7 | Sitemap policy for daily archives | doctrine | keep |
| 8.8 | Affiliate marketplace (alongside media) | doctrine | keep |
| 8.9 | UCP gate (do not false-fail media) | doctrine | keep |
| 8.10 | E-E-A-T / YMYL-adjacent politics (mandatory pages) | doctrine | keep |
| 8.11 | Hotlink image policy | doctrine | keep |
| 8.12 | Explicit non-goals | doctrine | keep |
| 8.13 | Normative auditor checklist (media) | doctrine | keep |
| 8.14 | Worked example pattern — R&R-style desk (2026-10-14 snapshot) | worked example | keep (snapshot is dated but illustrative; keep as pattern, not live truth) |
| 8.15 | Agent-layer minimum viable media node | doctrine | keep |
| — | Final Principle (two layers) | doctrine | **DOCTRINE (load-bearing)** |

### 1.2 Verdict on the inventory

- **Doctrine vs action-plan split is clean:** the *only* time-bound, milestone-driven action-plan content in the whole Bible is **§4's 6-Month Playbook** (M0–M6). Everything else is timeless pattern/checklist/benchmark — exactly the "complete resource tool" Frank wants.
- **Two doctrine spots need updating, not rewriting:**
  1. **§1.3 Crawler Access** — v2.1 lists GPTBot, OAI-SearchBot, Google-Extended, PerplexityBot, CCBot as "allow, never block." That list predates the **answer/training split** and is incomplete (no Claude-SearchBot/Claude-User, no Perplexity-User, no Googlebot/ChatGPT-User, no training-bot policy, no Grok note, no WAF caveat). The multi-engine update in §3 of this pack replaces it as doctrine while keeping its intent.
  2. **The Golden Rule has no verification half.** v2.1 mandates shipping the stack (UCP + schema + llms.txt + SSR) but never says how you *know* the agent layer works. The verification doctrine in §3.3 closes that gap.
- **Nothing in v2.1 is commercial packaging.** No pricing, no AVP offer, no SOW language anywhere in the file. The resource-tool character is intact. (Dick's draft added commercial framing; that is precisely what Frank rejected.)

---

## 2) No-loss checklist — every section heading in v2.1 that MUST survive into 3.0

**Method:** `grep '^#'` on `/home/ice/know/SEO_BIBLE_JUNE_2026.md` (v2.1, md5 501ade3c). A 3.0 merge passes only if **every line below appears** (heading text, verbatim or clearly equivalent — Romulus may renumber, may not drop) in the 3.0 draft. 26 heading lines total.

| # | Heading (verbatim from v2.1) | Section | Required in 3.0? |
|---|------------------------------|---------|------------------|
| 1 | `# The SEO & Agentic Commerce Bible` | title | yes |
| 2 | `### Changelog` | meta | yes (+ new 3.0 entry) |
| 3 | `## The Core Thesis` | thesis | **yes — load-bearing** |
| 4 | `### Which section applies?` | routing | **yes — load-bearing** |
| 5 | `## 1. Technical Architecture` | 1 | yes |
| 6 | `### 1.1 Universal Commerce Protocol (UCP)` | 1.1 | yes |
| 7 | `### 1.2 Schema Strategy` | 1.2 | yes |
| 8 | `### 1.3 Crawler Access` | 1.3 | **yes — but replaced by expanded multi-engine version (same heading, new body)** |
| 9 | `### 1.4 First HTML / SSR rule (all content products)` | 1.4 | **yes — non-negotiable** |
| 10 | `## 2. Content Strategy` | 2 | yes |
| 11 | `### 2.1 Information Gain (The 2026 Defensive Standard)` | 2.1 | yes |
| 12 | `### 2.2 The Mattress Land Structure (Human + Agent Optimized)` | 2.2 | yes |
| 13 | `## 3. Local SEO` | 3 | yes |
| 14 | `### 3.1 The Core 30 Structure` | 3.1 | **yes — non-negotiable** |
| 15 | `### 3.2 Google Business Profile Optimization` | 3.2 | **yes — non-negotiable** |
| 16 | `### 3.3 Review Velocity` | 3.3 | yes |
| 17 | `## 4. Restaurant Local SEO Case Study (2026)` | 4 | **yes — with 6-Month Playbook extracted (see §4); technical core + benchmarks stay** |
| 18 | `### Objective` | 4a | yes |
| 19 | `### 6-Month Playbook` | 4b | **conditional — allowed to leave Bible core ONLY if it becomes its own referenced "Restaurant Implementation Playbook" doc (Frank's lean; see §4)** |
| 20 | `### Expected Results` | 4c | yes |
| 21 | `## 5. Visual Search & Multimodal Optimization` | 5 | **yes — non-negotiable (the "IG"/visual lane)** |
| 22 | `## 6. Agentic Commerce Optimization (ACO)` | 6 | **yes — non-negotiable** |
| 23 | `## 7. Current Checklist (June 2026 + media patch)` | 7 | **yes — operational core** |
| 24 | `## 8. News desks, headline aggregators & legislative trackers` | 8 | **yes — non-negotiable** |
| 25 | `### 8.1 Entity model` | 8.1 | yes |
| 26 | `### 8.2 When NOT to use NewsArticle` | 8.2 | yes |
| 27 | `### 8.3 Information Gain bar for aggregators` | 8.3 | yes |
| 28 | `### 8.4 SSR / first-HTML rule (HARD FAIL)` | 8.4 | yes |
| 29 | `### 8.5 Feed requirements (HARD FAIL if absent)` | 8.5 | yes |
| 30 | `### 8.6 /llms.txt profile for media` | 8.6 | yes |
| 31 | `### 8.7 Sitemap policy for daily archives` | 8.7 | yes |
| 32 | `### 8.8 Affiliate marketplace (alongside media)` | 8.8 | yes |
| 33 | `### 8.9 UCP gate (do not false-fail media)` | 8.9 | yes |
| 34 | `### 8.10 E-E-A-T / YMYL-adjacent politics (mandatory pages)` | 8.10 | yes |
| 35 | `### 8.11 Hotlink image policy` | 8.11 | yes |
| 36 | `### 8.12 Explicit non-goals` | 8.12 | yes |
| 37 | `### 8.13 Normative auditor checklist (media)` | 8.13 | yes |
| 38 | `### 8.14 Worked example pattern — R&R-style desk (2026-10-14 snapshot)` | 8.14 | yes (keep as pattern; snapshot is dated by its own title) |
| 39 | `### 8.15 Agent-layer minimum viable media node` | 8.15 | yes |
| 40 | `## Final Principle` | closing | **yes — load-bearing** |

**Note:** grep returned 40 heading lines (my count above numbers them 1–40; "26" in the exec summary referred to the top-level load-bearing sections). The checklist above is the exact, exhaustive set.

**Hard-floor items (a 3.0 draft missing ANY of these fails the merge gate):** rows 3, 4, 8 (expanded), 9, 14, 15, 17, 21, 22, 23, 24, all of 25–39, 40.

**The only permitted "deletion":** row 19 (6-Month Playbook) — and only if it is (a) extracted verbatim (or better) into a standalone "Restaurant Implementation Playbook" resource, and (b) §4 in the Bible core retains a pointer to it. This is the *one* sanctioned move, and it is a move, not a loss.

**New sections the 3.0 assembly should ADD (from §3 of this pack — these are additions, not replacements):**
- Expanded `1.3 Crawler Access` (multi-engine: answer vs training split + WAF caveat + Grok note + robots template)
- New `## 9. Verification & Measurement` (battery, ablation, parity, ground-truth, no-guarantee rule)
- New `### 1.5 Two-Surface Model (Identity vs Transaction)` (or fold into 1.2)
- New `### 1.6 Extant-Site Install Patterns` (3 fidelity paths)
- `1.1 UCP` gains the version/timestamp + manifest cache-invalidation standard (already implied in v2.1; make explicit)

**Explicitly NOT to add:** any AVP commercial pricing, package names, SOW pricing tables, or placement guarantees — see §5.

---

## 3) What is newly true since June/October that MUST be added to 3.0

All grounded in our own verified work (t_64b6e1c1, t_70d29518, live probes 2026-10-28/29). **Not** from Dick's draft.

### 3.1 Multi-engine bot list — the answer/training split (replaces v2.1 §1.3 body)

v2.1's single "allow these bots" list is now known to be incomplete and conflates two different decisions. The corrected doctrine:

**A. Answer-engine bots — MUST allow, never block (visibility):**
| Bot | Vendor | Role |
|-----|--------|------|
| `OAI-SearchBot` | OpenAI | ChatGPT real-time web search |
| `ChatGPT-User` | OpenAI | ChatGPT interactive/agent fetch |
| `Claude-SearchBot` | Anthropic | Claude real-time web search |
| `Claude-User` | Anthropic | Claude interactive/agent fetch |
| `PerplexityBot` | Perplexity | Perplexity real-time answers |
| `Perplexity-User` | Perplexity | Perplexity interactive |
| `Googlebot` | Google | Core index (blocking kills everything) |
| `Amazonbot` / `Amzn-Search` | Amazon | Commerce/answer surfaces (if in a commerce vertical) |

**B. Training crawlers — separate data-governance decision (default allow, opt out deliberately):**
| Bot | Vendor | Role |
|-----|--------|------|
| `GPTBot` | OpenAI | Model training |
| `ClaudeBot` | Anthropic | Model training |
| `Google-Extended` | Google | Gemini/ML training (≠ Googlebot) |
| `Applebot-Extended` | Apple | Siri/Apple training |
| `CCBot` | Common Crawl | Open training corpus (feeds many LLMs) |
| `Bytespider` | ByteDance | Training/crawl |

**C. Grok / xAI — handle empirically.** xAI's crawler identity is under-documented; identify the actual UA from access logs before finalizing its line, default-allow, never guess a block.

**Doctrine statements to add:**
- Answer-engine allowlist is a **visibility** decision; training-bot policy is a **data-governance** decision. Do not conflate them.
- Do **not** claim "allowing GPTBot improves your ChatGPT citations" as fact — that is a long-timescale, hard-to-measure effect. The short-term levers are the *search* bots.
- **Anti-pattern to name in the Bible:** a site that blocks a training bot (or GPTBot) while publishing `llms.txt` (observed on SpeedPro Phoenix) is a contradictory stance — it undermines its own intent.

### 3.2 The WAF / edge override is a standing gotcha (new doctrine)

- A correct `robots.txt` is **not sufficient** when Cloudflare (or any WAF/CDN) has "Block AI bots" / "Super Bot Fight Mode" / a custom rule that challenges the same UAs. The request never reaches the origin's robots policy.
- **Acceptance rule:** before claiming "the site is AI-visible," verify each answer-engine bot UA returns **200 at the edge** (not 403/429/challenge). A robots Allow is necessary, not sufficient.
- **Real examples from the audit (keep as standing warning in the Bible):** azhottees.com and phxsignshop.com returned **429 bot-walls** on probe — invisible to LLM discovery regardless of any robots file. Greeks Grill is a pure JS shell (47 stripped chars) — a JS-shell is the SSR anti-pattern in the wild.

### 3.3 Verification & Measurement — the Golden Rule's missing half (new top-level §9)

v2.1 mandates *shipping* the stack but never says how you *know* the agent layer works. New doctrine:

- **Ship the stack, then prove it with a multi-engine citation battery.** The before/after delta is the evidence; the presence of files is not. "We shipped llms.txt" is not a done claim until a battery shows it moved a citation.
- **The canonical test is the fixed battery across all five engines** (ChatGPT, Claude, Gemini, Perplexity, Grok) — same prompts, verbatim, one fresh session per engine-prompt, same time window. (Full protocol + capture schema: t_70d29518 §A — copy into 3.0 as the standing protocol, not as a one-off.)
- **Parity is a first-class requirement, not a ChatGPT checkbox.** Each engine has its own crawler + index + model + citation policy. A site is "agent-visible" only when *measured* across all five. Optimizing/verifying one is not the others.
- **Causal verification via staged single-variable ablation — never production.** To claim a signal "matters," remove exactly one signal on a staging/edge/clone variant and re-run the battery; rank signals by citation delta. **Hard rule: never ablate production without explicit owner approval.** When staging is unavailable, use retrospective triangulation (referral + bot-crawl logs + signal ship-dates) and **label it correlational.**
- **Commercial honesty rule (for AVP / any commercial use of this doctrine):** we never promise "X will recommend you." We promise **measurement**: "here is the stack, here is the multi-engine before/after battery, here is the ablation ranking." The product's proof is the measurement protocol + the data it produces, never a placement guarantee. **(No pricing in this rule — pricing lives only in the SOW/one-pager, never in the Bible.)**
- **Ground-truth fact table is part of the deliverable, not optional.** Every audited site ships a ground-truth table (phone/price/hours/address) so a battery can distinguish *the engine read our structured data* (quoted the exact fact) from *the engine hallucinated* (quoted a wrong/plausible fact). Fact-accuracy is a scored metric, not a footnote.
- **Re-verification loop:** manifests carry `lastUpdated` + a version string so engines can invalidate; re-run the battery quarterly or after any stack change.

### 3.4 The two-surface model — identity vs transaction (new §1.5, or fold into 1.2)

From the t_64b6e1c1 audit (verified on both reference sites, 2026-10-28):

- **"AI-readable" splits into two distinct surfaces:**
  1. **Identity / discovery surface** (what an LLM reads on first contact): `llms.txt`, typed `Restaurant`/`LocalBusiness` schema, full street address, hours, `aggregateRating`, `sameAs` (9+), `knowsAbout`, `hasMenu`. → **Maza is the reference.**
  2. **Transaction / offer surface** (what an agent needs to quote/checkout): `ucp/1.0` products with tiers, `/api/specs/<sku>`, `Offer`/`ItemList` JSON-LD, visible `$` prices, one `/order/<sku>` checkout per SKU. → **PJ is the reference.**
- **A site that wants to be fully AI-readable in 2026 needs BOTH.** Neither reference alone is the complete model.
- **Measure each surface's contribution separately** in the ablation (identity variants vs transaction variants) — don't pool them.
- **Competitor finding to keep as standing context:** as of 2026-10-28, **no local competitor combines both** — that is the open lane. (Re-verify before treating as permanent; competitor sites can ship manifests overnight.)

### 3.5 Extant-site install patterns — three fidelity paths (new §1.6)

For sites where we don't own the repo (the AVP client case; also relevant to any client work):

| Path | When | Fidelity |
|------|------|----------|
| **1 — Full patch (own repo/host)** | We own the deploy | Gold standard; ship the whole stack in-repo |
| **2 — Edge injection (Cloudflare Workers)** | We control DNS/edge, not the origin | Standard client offering; origin untouched; inject llms.txt/UCP/schema/robots at the edge |
| **3 — Parallel microsite** | No repo, no edge control | **Last resort, reduced grounding**; separate agent-facing surface that deep-links to the real pages |

- **Hard rule:** never promise Path 3 delivers the same fidelity as Path 1. Every engagement states which path applies and its acceptance criteria (each path's acceptance criteria are specified in t_70d29518 §D — copy into 3.0).
- **Path 2 caveat:** edge injection of `<head>` schema requires the Worker to reliably rewrite the *full* HTML response — do it only when the origin returns full HTML, not a JS shell (the Greeks Grill anti-pattern).

### 3.6 Small but real doctrinal refinements (carry into 3.0)

- **UCP versioning standard is explicit:** every manifest carries `lastUpdated` + `schema: <name>/v1` so agents can cache-invalidate. (v2.1 implies this; make it a stated rule.)
- **Schema: use the *specific* type, not generic `LocalBusiness`** — `Restaurant` for food (with `aggregateRating`, `hasMenu`, `servesCuisine`), `Store`/`ProfessionalService` + a **real street `PostalAddress`** for print/services. (Fixes the observed `address: null` anti-pattern on PJ.)
- **`sameAs` bar = 9+** (Maza's level): Maps deep links, Yelp, all delivery platforms, all socials.
- **`aggregateRating` is the fastest trust-signal win** for lane leaders — publish it where real.
- **Local-intent URL farm target ~50 URLs** (Maza's 51 = 8 city + 20 dish + 4 service + core).

---

## 4) Playbook split recommendation (Bible core vs separate implementation playbooks)

**Recommendation: SPLIT. Keep the technical rules in the Bible; extract the dated action plan into its own resource.**

This matches Frank's lean (Bible = complete resource tool; playbook = action plan) and resolves the one structural blemish in v2.1.

### 4.1 What STAYS in Bible core (the "complete resource tool" floor)

Everything in v2.1 §3, §4a, §4c, and the technical rules inside §4b, because they are **timeless patterns**, not a dated plan:

- **§3 Local SEO** — Core 30 Structure, Google Business Profile Optimization, Review Velocity (12–24 new/mo benchmark). Non-negotiable.
- **§4 Objective + Expected Results** — the case-study framing and the benchmark outcomes. Keep as the anchor that says "here's what a done restaurant looks like."
- **The restaurant *technical rules* that live inside the 6-month playbook**, extracted as doctrine:
  - **Menu SSR / first-HTML rule** (the menu must be server-rendered, not a JS shell — the Greeks Grill anti-pattern)
  - **Local-intent page structure** (city + dish + service pages, ~50 URLs)
  - **GBP completion + review velocity as a standing practice**
  - **Schema for restaurant** (`Restaurant` + `hasMenu` + `aggregateRating` + 9+ `sameAs`)
  - **Visual search / menu-board / QR** (§5 applies)
  - **UCP for menu/catering** (the ACO lane)
- **§5 Visual Search & Multimodal Optimization** — this is the "IG" lane; keep in core.
- **§8 News Desks** — entire section, non-negotiable (Frank named it).

### 4.2 What moves OUT into a separate "Restaurant Implementation Playbook" resource

- **The 6-Month Playbook (M0/M1/M2–M3/M4–M6 milestones)** — this is a *time-bound action plan for one vertical's one case*. It is the only time-bound content in the Bible. It belongs in its own doc: `Restaurant Implementation Playbook` (a sibling resource to the Bible, referenced by name from §4).
- **Why it's safe to extract (no-loss):**
  1. The *technical rules* it contains are re-homed into Bible core (§4.1 above) — they are not lost, they are promoted to doctrine.
  2. The *dated milestones* (M0–M6) are inherently a plan, not a pattern; they will go stale anyway. Putting them in a standalone playbook means the Bible never carries an expiring calendar.
  3. The Bible keeps a **pointer** in §4: "See the Restaurant Implementation Playbook for the dated 6-month action plan." So a reader of the Bible is never lost.
- **Downstream benefit:** the same split generalizes. Other verticals (print, signage, services) get their *own* implementation playbooks, all referencing the same Bible core. The Bible stays the one timeless resource; playbooks are the per-vertical, per-time action layer. This is exactly the "resource tool vs action plan" split Frank wants.

### 4.3 Net effect on the no-loss checklist

- The **only** row in §2's checklist that may leave the Bible file is **row 19 (6-Month Playbook)** — and only as a *move* to the standalone playbook doc, with a pointer left behind.
- **Every other row is non-negotiable and must appear verbatim (or clearly equivalent) in 3.0.**
- If Frank prefers the *absolute safest* path, the 6-Month Playbook can simply **stay in the Bible** and the standalone playbook can be a later extraction — but then it remains the one time-bound blemish in an otherwise timeless resource. My recommendation is the split, because it matches your stated lean and costs the reader nothing (pointer + promoted technical rules).

---

## 5) Explicit: no AVP pricing in doctrine

**Rule (absolute, for the Bible and for 3.0 specifically):**

- The Bible contains **zero** commercial pricing: no dollar amounts for our services, no package names, no SOW price tables, no "AVP starter/pro/enterprise" tiers, no per-month service fees.
- The **only** dollar amounts that may appear in the Bible are **illustrative business facts** used in the worked examples (e.g., PJ's card price points $65/$75/$125, César's catalog, Maza's `$$`) — these are *data about audited sites*, not our commercial offers. They are evidence, not pricing.
- **AVP commercial framing lives exclusively in the SOW / one-pager / sales workstream (t_10e1b346 line), never in the Bible.**
- The **only** commercial-adjacent language allowed in the Bible is the **measurement-only honesty rule** (§3.3): "here is the stack, here is the before/after battery, no placement guarantees." That is a *claim-standard*, not a price.
- **Merge gate:** if any line in the 3.0 draft reads like a price point or an offer for *our* services (not an audited site's fact), it is a merge failure. Flag it.

---

## 6) Go / no-go — is research sufficient to authorize Romulus full-merge 3.0?

### 6.1 The decision

**GO.** Research is sufficient to authorize Romulus to build 3.0 as **full v2.1 body + additive sections, zero deletions**, using this pack's §2 no-loss checklist as the merge gate and §3 as the content to add.

### 6.2 Why GO (what is grounded)

Every required doctrinal input for 3.0 is already grounded in **verified** work, not speculation:

| Required input | Grounding | Status |
|----------------|-----------|--------|
| Multi-engine bot list (answer/training split) | t_70d29518 §C (robots template + tables) | **grounded — ready to copy** |
| WAF/edge override gotcha | t_70d29518 §C.4 + t_64b6e1c1 (429 walls, Greeks Grill JS shell) | **grounded — ready to copy** |
| Verification & measurement (§9) | t_70d29518 §A/§B (battery + ablation + protocol) | **grounded — ready to copy** |
| Two-surface model | t_64b6e1c1 §1.3/§3/§4 (live probes, both references) | **grounded — ready to copy** |
| Extant-site patterns | t_70d29518 §D (3 paths + acceptance) | **grounded — ready to copy** |
| Ground-truth + fact-accuracy | t_70d29518 §A.6 + scoring | **grounded — ready to copy** |
| No-loss floor (26/40 headings) | v2.1 file re-read this run, md5-verified | **grounded — checklist is exact** |
| Playbook split decision | this pack §4 | **decided (Frank's lean)** |
| No-pricing rule | this pack §5 | **decided (Frank's rule)** |

### 6.3 The open items — and why NONE of them block the *writing* of 3.0

These are **execution** items, not **research** items. They gate *claiming verification results*, not *writing the doctrine*:

1. **The battery has not been executed against live engines.** I am headless; I will not fabricate citation counts. t_70d29518 delivered a *runnable protocol + capture schema + ground-truth table*, ready for a ~45-minute logged-in run. → **Does not block writing 3.0.** 3.0 should include the *protocol* as doctrine. The *data* (the actual before/after results) is a later 3.0.1 patch when a human/logged-in agent runs it.
2. **Staging/preview availability for PJ+Maza is unconfirmed** (needed for the *ablation*, not for the doctrine). → Does not block writing 3.0.
3. **GA4 + server bot-log access unconfirmed** (needed for the *retrospective triangulation*, not for the doctrine). → Does not block writing 3.0.
4. **Grok/xAI crawler identity not definitively pinned** (needs an access-log check). → Does not block writing 3.0 — the doctrine already says "identify empirically, default-allow."
5. **Dick AVP pack** (`/srv/app-data/dick-jones/…`) was not in my container. → **Irrelevant to 3.0** — it is Dick's commercial artifact, explicitly excluded from doctrine by Frank. It belongs to the SOW workstream, not the Bible.

**Conclusion:** there is **no research gap** that blocks the *assembly* of 3.0. The doctrine is fully grounded. The only things pending are *measured results* (battery data) and *access confirmations* (staging, logs, Grok UA) — all of which are post-assembly or parallel workstreams.

### 6.4 Recommended sequencing for Frank

1. **Now:** authorize Romulus `t_7b4116e0` to build 3.0 = **full v2.1 body (zero deletions) + the §3 additive sections**, using §2 as the merge gate and §5 as the no-pricing gate.
2. **In parallel (does not block 3.0):** run the baseline battery (t_70d29518 §A) on a logged-in session → produce the *data* for a **3.0.1 verification patch**.
3. **In parallel:** confirm staging + GA4/bot-log access; pin the Grok UA from access logs.
4. **After 3.0 lands:** extract the 6-Month Playbook into the standalone "Restaurant Implementation Playbook" (the one sanctioned move, §4).
5. **Never:** pull any AVP pricing into the Bible, or let Dick's draft set doctrine.

---

## 7) Claims & evidence (structured)

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | The current official Bible is v2.1 "JUNE 2026" (last edited 2026-10-14), 420 lines, and is a resource tool with exactly one time-bound action plan (§4 6-Month Playbook). | v2.1 file re-read this run (md5 501ade3c); §1.1 section map | high | supported |
| C2 | The only content in v2.1 that is a time-bound action plan (vs timeless doctrine) is the 6-Month Playbook; everything else is pattern/checklist/benchmark. | §1.1 classification; full-file read | high | supported |
| C3 | v2.1's Crawler Access (§1.3) is incomplete and conflates answer vs training bots; it must be replaced by the multi-engine split (OAI-SearchBot/ChatGPT-User/Claude-SearchBot/Claude-User/PerplexityBot/Perplexity-User/Googlebot must-allow; GPTBot/ClaudeBot/Google-Extended/Applebot-Extended/CCBot/Bytespider = training policy; Grok = empirical). | t_70d29518 §C (tables + template); v2.1 §1.3 | high | supported |
| C4 | A correct robots.txt is not sufficient for AI visibility behind a Cloudflare/WAF "block AI" rule; acceptance must be a 200 at the edge per answer-bot UA. | t_70d29518 §C.4; t_64b6e1c1 (azhottees/phxsignshop 429 walls) | high | supported |
| C5 | The Golden Rule's missing half is verification: ship the stack, then prove it with a multi-engine citation battery; the before/after delta is the evidence, not file presence. | t_70d29518 §A/§E.1 | high (as doctrine) / **data not yet executed** | supported as doctrine; measurement pending |
| C6 | "AI-readable" splits into an identity/discovery surface (Maza: llms.txt + Restaurant schema + address + rating + 9 sameAs) and a transaction/offer surface (PJ: ucp/1.0 + specs + visible prices); a fully AI-readable site needs both, and no local competitor combines both as of 2026-10-28. | t_64b6e1c1 §1.3/§3/§4 (live probes both references + 15 competitors) | high | supported |
| C7 | Extant-site installs split into 3 fidelity paths (own-repo / edge-injection / parallel-microsite); edge is the standard client offering; microsite is last-resort with reduced grounding and must deep-link to real pages. | t_70d29518 §D | medium-high | supported |
| C8 | The no-loss floor is 40 heading lines in v2.1 (listed exactly in §2); the only sanctioned "deletion" is the 6-Month Playbook, and only if moved to a standalone playbook with a pointer left behind. | §2 checklist (grep `^#` on v2.1, this run); §4 | high | supported |
| C9 | No AVP commercial pricing may appear in the Bible; the only $ allowed is illustrative audited-site data; the only commercial-adjacent language allowed is the measurement-only no-guarantee rule. | Frank's rule (this task §4/§5); this pack §5 | high | supported (policy, per operator directive) |
| C10 | Research is sufficient to authorize Romulus full-merge 3.0 (full v2.1 + additive, zero deletions); the open items are execution-only (battery data, staging, logs, Grok UA) and do not block the writing. | §6 (all doctrinal inputs grounded in t_64b6e1c1 + t_70d29518 + v2.1 re-read) | high | supported |

## 8) Evidence index (provenance)

| evidence_id | source_label | source_url | provenance | support note |
|-------------|--------------|------------|------------|--------------|
| E1 | SEO Bible v2.1 (JUNE 2026) | `/home/ice/know/SEO_BIBLE_JUNE_2026.md` (local, md5 501ade3c) | read in full this run | The current official Bible; source of the §1 section map and the §2 no-loss checklist (all 40 headings) |
| E2 | t_64b6e1c1 (AI-readable audit) | `/home/ice/know/research/ai-readable-web-pj-maza-competitor-audit-t_64b6e1c1.md` (GitHub: arealicehole/know) | read this run; live probes 2026-10-28 | Two-surface model, 15-competitor matrix, 429 walls, Greeks Grill JS shell; basis for C6 |
| E3 | t_70d29518 (AVP causal verification) | `/home/ice/know/research/avp-causal-verification-multi-engine-citation-battery-t_70d29518.md` (GitHub: arealicehole/know) | read this run; live re-probes 2026-10-29 | Multi-engine bot tables + robots template (§C), WAF caveat (§C.4), battery+ablation (§A/§B), extant-site paths (§D), Bible inputs (§E); basis for C3/C4/C5/C7 |
| E4 | t_505c6bc5 (news-desk §8 provenance) | `/home/ice/know/research/seo-bible-news-desk-section-t_505c6bc5.md` | prior GIU report | Provenance for the §8 News Desks section that must not be dropped |
| E5 | Operator course-correct (tricon-web, via Frank/Discord) | kanban task note, 2026-10-29 | in-task operator message | The mandate for this standalone pre-assembly pack; no-Dick-doctrine; no-pricing; go/no-go framing |
| E6 | Dick 3.0 draft | (referenced by task; **excluded**) | not consumed | Non-authoritative outline only; explicitly excluded as doctrine source per Frank |

> **Honesty note:** I did **not** execute the multi-engine citation battery against live subscription engines in this run (headless, no logged-in sessions), and I will not fabricate citation counts. The *protocol* is grounded and ready to copy into 3.0; the *measured data* is a separate, later step (3.0.1). Everything else in this pack is grounded in files I read this run or verified live probes from the two parent reports.

## 9) Gaps / unresolved questions

1. **Battery data not produced** — protocol ready, no live citation results (headless constraint). → 3.0.1 patch, not a 3.0 blocker.
2. **Staging/preview availability (PJ/Maza)** unconfirmed — affects ablation, not the doctrine writing.
3. **GA4 + bot-log access** unconfirmed — affects retrospective triangulation, not the doctrine writing.
4. **Grok/xAI UA** not pinned — doctrine already says "identify empirically, default-allow"; pin via access logs later.
5. **Dick AVP pack** not in my container — irrelevant to 3.0 (commercial, excluded); belongs to SOW workstream.
6. **Playbook extraction mechanics** — the standalone "Restaurant Implementation Playbook" doc does not exist yet; creating it is a follow-up (can be its own card) and is *not* a blocker for 3.0 (the split can be finalized after 3.0 lands).

## 10) Confidence (report level)

- **level:** **high**
- **rationale:** Every doctrinal input for 3.0 is grounded in files I read this run (v2.1 full read + both parent reports) or verified live probes (2026-10-28/29). The no-loss checklist is exact (md5-verified source, full heading enumeration). The two genuinely *pending* items (battery data, access confirmations) are explicitly *execution* items that do not block the *writing* of 3.0 — and I have said so plainly rather than dressing them up as research gaps. The go/no-go is a **GO** on the assembly with an honest separation between "doctrine is ready" and "measured results are not."

## 11) Recommended next actions

1. **Frank:** confirm the **GO** and unblock Romulus `t_7b4116e0` to build 3.0 = full v2.1 (zero deletions) + §3 additive sections, gated by §2 (no-loss) and §5 (no-pricing).
2. **Romulus (once unblocked):** assemble 3.0 from this pack; do **not** import Dick's commercial framing; keep the no-loss checklist as the merge gate.
3. **Parallel (owner/agent with a logged-in session):** run the t_70d29518 §A baseline battery (P1–P12 × 5 engines) → capture into the schema → produce the *data* for a **3.0.1 verification patch**.
4. **Parallel:** confirm PJ/Maza staging + GA4/bot-log access; pin the Grok UA from access logs.
5. **After 3.0 lands:** create the standalone "Restaurant Implementation Playbook" (extract the 6-Month Playbook, re-home the technical rules into Bible core, leave a pointer) — can be its own card.
6. **Never:** write AVP pricing into the Bible; let Dick's draft set doctrine; claim verification results before the battery is actually run.

---

*Prepared by Geraldo v2.2 (Kanban-native, Swarm Kanban v2.0). Single durable copy: `/home/ice/know/research/seo-bible-3-0-research-pack-no-merge-t_fa9d749c.md`. No double-publish. Research pack only — this is the pre-assembly input, not a directive to merge. All doctrine grounded in v2.1 + t_64b6e1c1 + t_70d29518 (read/verified this run). No AVP pricing anywhere in this document.*

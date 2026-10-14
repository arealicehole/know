# The SEO & Agentic Commerce Bible

**Second Edition — Revised June 2026 (patch 2.1)**

**Version:** 2.1  
**Last Major Update:** June 24, 2026  
**Patch:** October 14, 2026 — §8 News desks / headline aggregators / legislative trackers  
**Previous Edition:** January 2026

### Changelog
| Date | Ver | Change |
|------|-----|--------|
| 2026-06-24 | 2.0 | Second Edition: Information Gain, restaurant Core 30 case study, UCP/ACO merge |
| 2026-10-14 | 2.1 | **§8** News desks, headline aggregators & legislative trackers (media gate vs Core 30 / full UCP). Reference specimen: Root & Rifle (`rootandrifle.com`, repo `arealicehole/rootandrifle-site`) |

---

## The Core Thesis

The SEO landscape has split into two parallel systems that must be optimized simultaneously:

- **Human Layer**: Rewards first-hand experience, originality, and genuine "Information Gain."
- **Agent Layer**: Rewards clean, structured, machine-readable data (UCP manifests, schema, specs.json, first-party feeds).

**Golden Rule (2026)**  
> Humans want stories and proof. Agents want structured data. The winners serve both while adding real new information that didn’t exist before.

The December 2025, March 2026, and May 2026 Core Updates have all reinforced this direction. Generic or rephrased content is being systematically demoted.

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

**Status:** Live and expanding since January 2026 (NRF launch).

**Gate:** Full UCP at `/.well-known/ucp` + checkout/fulfillment specs is **mandatory only when first-party checkout exists**. Media-only and aggregator sites: see **§8.9 UCP gate** — ship `llms.txt`, feeds, and structured indexes; do **not** fail a news node for missing merchant UCP.

Merchants (with checkout) must expose a machine-readable manifest at `/.well-known/ucp` that declares capabilities for discovery, checkout, fulfillment, discounts, and order management. This enables AI agents (Gemini, ChatGPT, etc.) to transact without a human visiting the site.

**Requirements (checkout sites)**
- Real-time sync with inventory
- Support for Agent Payments Protocol (AP2) tokens
- Rate limiting to prevent agent abuse

### 1.2 Schema Strategy

Agents and answer engines prioritize structured data over marketing copy.

**Recommended schemas by use case:**
- E-commerce / Data sites → `Dataset`
- Restaurants & Local Services → `Restaurant` + `Menu` + `MenuItem`
- **News desks / aggregators / legislative trackers → `NewsMediaOrganization` + honest page types (`CollectionPage` / `ItemList` / `Dataset`); see §8.1–§8.2**
- Transactional actions → `potentialAction` (`ReserveAction`, `BuyAction`, `OrderAction`)
- Voice optimization → `Speakable`

### 1.3 Crawler Access

Do not block AI crawlers. Blocking `GPTBot`, `OAI-SearchBot`, or `Google-Extended` reduces visibility in answer engines and agentic surfaces.

### 1.4 First HTML / SSR rule (all content products)

**Hard rule for agent and crawler discovery:** Primary listing surfaces (homepage, section hubs, day archives) must ship the **headlines / items in the first HTML response**. Client-only injection of the corpus from a third-party host (GitHub Gist, raw CDN, etc.) is an **agent fail** even if humans eventually see content. Day archives with full link HTML are the **pass pattern**. Detail: **§8.4**.

---

## 2. Content Strategy

### 2.1 Information Gain (The 2026 Defensive Standard)

The March and May 2026 Core Updates made “Information Gain” one of the strongest ranking signals. Content that merely rephrases what already ranks well is now heavily penalized.

**Winning content must contain one or more of the following:**
- Proprietary data or original research
- First-hand experience and case studies
- Unique tests or experiments
- Novel analysis or insights

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

- Upload **100+ high-quality photos** and continue adding new ones weekly
- Maintain perfect NAP consistency
- Respond to every review within 24 hours
- Post at least weekly
- Enable all relevant attributes

### 3.3 Review Velocity

Fresh reviews (last 30–60 days) now carry more weight than total review count. Build systems that generate consistent, genuine reviews.

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
- Write descriptive alt text and filenames that match local intent
- Upload geo-tagged images to Google Business Profile
- **Media desks:** prefer self-hosted or first-party cached heroes; third-party CDN hotlinks are brittle and rights-risky (**§8.11**)

---

## 6. Agentic Commerce Optimization (ACO)

- Structure product data with utility signals (availability, shipping, returns, ratings, sustainability)
- Maintain a `specs.json` file as the source of truth for product attributes
- Use honeypot techniques to trap malicious scrapers while allowing legitimate agents
- **Media-only sites:** optional lightweight `specs.json` may declare the **content product** (update frequency, license, ad/affiliate products) **without** fake checkout — **§8.9**. Full merchant ACO applies only with first-party commerce.

---

## 7. Current Checklist (June 2026 + media patch)

**Universal**
- [ ] Every major page contains genuine Information Gain (or media-specific gain per §8.3)
- [ ] AI crawlers allowed in robots.txt
- [ ] No stock photography on key proof pages (local/service)
- [ ] Primary listings present in **first HTML** (not JS-only corpus inject)

**Local / restaurant (when applicable)**
- [ ] UCP manifest live and inventory-synced **if checkout exists**
- [ ] Google Business Profile has 100+ photos + weekly uploads
- [ ] Review velocity system is active and consistent
- [ ] Full menu + reservation schema implemented

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

Missing `/llms.txt` on a 2026 media/agent-facing desk = **fail** for agent layer (even if humans browse fine).

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

## Final Principle

The winners in 2026 are not the best at “SEO” in the traditional sense.  
They are the best at being **authentically valuable to humans** while being **perfectly readable to machines**.

Build for both. Add real information. Stay consistent.  
**Pick the right ruler:** local, commerce, or media (§8) — and do not false-fail a news desk for missing a restaurant menu or a merchant UCP.

---

*This is the living Second Edition (v2.1). Update quarterly or after major core updates. Media section added October 14, 2026.*

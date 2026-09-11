# AI-assistant local discovery: what actually drives an LLM to recommend a Chandler restaurant — and how MAZA gets there

Task: `t_9c26f53d` (board: research, [GIU])
Client: MAZA Mediterranean Cuisine, 3491 W Frye Rd Ste 2, Chandler AZ 85226
Requested by: MAZA (TriCon Digital account manager) for Frank
Date compiled: 2026-09-11 (UTC)
Author: Geraldo v2.2 (GIU)

Evidence-quality labels used throughout:
- **[DOC]** = primary documentation, peer-reviewed research, or first-party engineering docs
- **[VENDOR]** = the platform's own marketing/help guidance about its own product
- **[PRESS]** = reputable journalism reporting a named deal/statement
- **[INDUSTRY]** = third-party SEO/AEO vendor blogs or vendor-run studies — directionally useful, not verification
- **[TEST]** = a test I actually ran against mazahalalfood.com on 2026-09-11
- **[ANEC]** = anecdote/folklore, unverified

---

## 1. Objective

Answer, with citations and labeled evidence quality, what actually determines whether an AI assistant recommends a local restaurant like MAZA — separating documented mechanics from SEO folklore — and give Frank a short, concrete lever list plus the three cheapest empirical checks to see where MAZA currently stands.

## 2. Executive summary

1. **There is no separate, secret "AI ranking system" for local restaurants that bypasses search.** Every major assistant that answers "good food spots in Chandler" is retrieval-first: it queries an existing web/local index (Bing's for ChatGPT and Copilot, Google's for Gemini/AI Overviews/AI Mode, its own crawler for Perplexity, Apple's place graph for Siri/Apple Maps), then summarizes. Google says this on the record: pages must be *indexed and snippet-eligible*, "there are no additional technical requirements" for AI Overviews/AI Mode. **[DOC]** (E1)
2. **The highest-leverage things are boring and verifiable:** be crawlable to the right bots, be in the local index (GBP + Bing Places + Apple Business Connect), keep NAP/hours byte-identical everywhere, have dense third-party review/citation coverage, and have your menu/FAQ facts in plain crawlable text. **[DOC/VENDOR]** (E1–E9)
3. **The single biggest change in the last 12 months is the OpenAI–Yelp data-licensing deal (announced 23 July 2026):** Yelp reviews, ratings, photos and business details now feed ChatGPT's local answers, and Yelp branding/links appear when used. Yelp's CEO: *"If you want to answer local queries, you really need Yelp."* For MAZA — 13 Yelp reviews, 4.3★ — this is simultaneously the biggest opportunity and the biggest exposure. **[PRESS]** (E10)
4. **`llms.txt` is folklore for assistants.** Google stated publicly (July 2025) that it does not support the file and has no plans to; no major AI crawler operator has published that its retrieval honors third-party `llms.txt`. It is not harmful, but it is not the lever. **[DOC-adjacent/PRESS]** (E11)
5. **Keywords stuffing — the classic SEO reflex — measurably *reduces* visibility in generative engines.** The peer-reviewed GEO study (KDD '24) found Citation Addition, Quotation Addition and Statistics Addition lifted visibility 30–40%; keyword stuffing scored *below* the no-optimization baseline. **[DOC]** (E12)
6. **MAZA's technical foundation is better than most independents** — `robots.txt` explicitly allows GPTBot, OAI-SearchBot and Google-Extended, plus a sitemap; the homepage carries `Restaurant` JSON-LD with address, geo, hours, `servesCuisine`, `hasMenu`; `/menu` carries 72 `MenuItem` entries with `Offer` prices. **[TEST]**
7. **MAZA's real weaknesses are entity-signal, not markup:** only 13 Yelp reviews, an hours conflict between the site/GBP (Tue–Sun 10:00–22:00) and the Yelp listing snippet (Tue/Wed closing 20:00), and a **wrong street number on Grubhub ("3419" vs the correct "3491")** — a live URL, verified today. Small review volume is now the binding constraint, not schema. **[TEST]** (E13)
8. **For a restaurant this small, the cheapest wins are not code.** They are: reviews (volume + owner responses), reconciling hours/address across Yelp/Apple/Grubhub/DoorDash, and getting into the local "best of Chandler" / Reddit conversation that assistants actually cite. All three are cheaper than a schema project, and two of the three are currently broken. **[INDUSTRY + TEST]**

---

## 3. What "AI-assistant local discovery" optimization actually is

Four terms circulate; they describe overlapping activities, not four different systems:

| Term | What it means | Who uses it |
|---|---|---|
| **GEO** (Generative Engine Optimization) | Research term from the KDD '24 paper: optimizing *content* so it is cited more prominently inside generated answers. Now also adopted by Microsoft in its own webmaster tooling. | Academics → Microsoft **[DOC]** (E12, E5) |
| **LLMO / AEO** (LLM / Answer Engine Optimization) | Agency-side branding for the same idea, usually with less evidence. | Vendors **[INDUSTRY]** |
| **AIO** | "AI Overviews" (Google's product) or "AI Optimization" depending on who's talking. | Mixed |
| **Local SEO** | The actual substrate. Everything the AIO/GEO advice recommends at the local level resolves to GBP/Bing Places/Apple listings, reviews and citations — i.e. local SEO. | Google **[DOC]** (E3, E4) |

**What is documented to move an LLM's recommendation** (all with primary sources in the evidence index):
- Being in the retrieval index at all, and allowing the retrieval crawler (OAI-SearchBot for ChatGPT search; Googlebot for Google AI surfaces; PerplexityBot for Perplexity; Bing's index for Copilot). **[DOC/VENDOR]** (E1, E2, E5, E6)
- Local listing accuracy/completeness: address, hours, categories, attributes, photos — Google explicitly says "businesses with complete and accurate info are more likely to show up". **[DOC]** (E3)
- Prominence signals — Google names *how many websites link to your business and how many reviews you have*. **[DOC]** (E3)
- Content structure the model can lift: headings, Q&A pairs, lists, tables, self-contained sentences, schema that matches visible text. Both Microsoft and Bing told publishers exactly this. **[VENDOR]** (E5, E13)
- Freshness: IndexNow / sitemap freshness so AI surfaces reference current facts. **[VENDOR]** (E5)
- Third-party licensed corpora: Yelp→OpenAI (2026), Yelp→Perplexity (2024), Reddit→OpenAI (2024), news/publisher licensing. These are the "shortcut into the answer" channels and they are contractual, not technical. **[PRESS/DOC]** (E10, E14, E15)

**What is folklore (no primary source found):**
- That a `llms.txt` file causes assistants to recommend you. **[ANEC]** (E11)
- That "AI-specific schema" or a special "AI markup" exists. No vendor documents one; the schema types that matter are the ordinary ones (`Restaurant`, `Menu`/`MenuItem`, `FAQPage`, `AggregateRating`). **[ANEC]**
- That you can *pay* or otherwise guarantee placement — Google: "There's no way to request or pay for a better local ranking." OpenAI: "Placement is not guaranteed." **[DOC]** (E3, E2)
- Precise, published ranking formulas for local answers. None exist. Anyone quoting weights is reverse-engineering. **[ANEC]**

---

## 4. Which live sources feed assistants for local food queries

| Assistant | Retrieval substrate | What that means for MAZA | Evidence |
|---|---|---|---|
| **ChatGPT (search)** | Third-party search providers; OpenAI's help centre names the **Microsoft privacy statement** among its search providers, and OpenAI's own eligibility doc routes Search inclusion through **OAI-SearchBot**. Location is inferred from IP and may be shared with search providers; queries get rewritten (e.g. "good restaurants near me" → "top restaurants San Francisco"). Restaurant reservations surface availability from **OpenTable, Resy or Yelp**. Since 23 Jul 2026, **Yelp reviews/photos/business details are licensed into ChatGPT**. | Two doors: be in Bing's index + OAI-SearchBot allowed (both true today), and be strong on Yelp (13 reviews — the weak door). | **[DOC/PRESS]** (E2, E10, E16) |
| **Google Gemini / AI Overviews / AI Mode** | Google Search index + Business Profile + Maps. Eligibility = indexed + snippet-eligible; **no additional technical requirements**. Uses "query fan-out" (multiple related searches across subtopics and data sources) and shows a broader set of links than classic search. Prompts answers with Map/local packs. | GBP completeness and Maps prominence are the levers; the site only needs to be crawlable and indexable. | **[DOC]** (E1, E3, E4) |
| **Perplexity** | Own crawler **PerplexityBot** (for surfacing/ linking in results) plus **Perplexity-User** for live user fetches (which generally ignore robots.txt). Licenses **Yelp** local data (Mar 2024). Third-party query audits report heavy use of Reddit, niche directories and editorial roundups cross-checked against Yelp/Google reviews. | Allow PerplexityBot (currently allowed by default), and get into roundups/Reddit, since that's what it reads. | **[DOC + INDUSTRY]** (E6, E14, E17) |
| **Microsoft Copilot** | **Bing's search index**, plus **Bing Places for Business** for local business facts. Microsoft's own post: "Powered by Bing's search index, experiences like Microsoft Copilot…" and Bing's Feb 2026 tooling post tells local businesses to register with Bing Places "to help ensure that key details such as address, hours, and contact information remain current and eligible for inclusion in AI-generated responses." | Claim/complete Bing Places — this is MAZA's most under-used lever and it is free. | **[VENDOR]** (E5, E18) |
| **Apple Intelligence / Siri / Apple Maps** | Apple's own place graph, **Apple Business Connect** for claimed listings, plus licensed partners (Yelp supplies data to Apple Maps per Yelp's CEO in Axios). Apple Maps already has a MAZA place card. | Claim the Apple Business Connect place card so hours/photos/actions are first-party instead of scraped. | **[PRESS/VENDOR]** (E10, E7, E19) |

**Retrieval is layered, and the layers are unequal.** Copilot and ChatGPT both lean on Bing; Google's surfaces lean on Google's index; Perplexity blends its own crawl with licensed Yelp. Consequence: **Bing indexation is not optional for US restaurants any more.** A listing that is only clean on Google is invisible to ChatGPT and Copilot.

---

## 5. The concrete levers, ranked by cost-to-value for a single-location halal restaurant

### Tier 1 — free, immediate, highest expected impact
1. **Bing Places for Business** — claim/verify, complete address, hours, categories, photos. Directly feeds Copilot local answers per Microsoft. Also enables Bing Webmaster Tools → **AI Performance** dashboard (public preview since Feb 2026) which reports which of your URLs are **cited in AI-generated answers** and which **grounding queries** retrieved them. That is the only first-party AI-citation telemetry that exists today. **[VENDOR]** (E5, E18)
2. **Google Business Profile completeness** — hours incl. special hours, primary category ("Mediterranean restaurant"), secondary categories, attributes (halal, parking, takeout), menu link, photos. Google states complete+accurate info improves local visibility, and prominence draws on reviews and inbound links. **[DOC]** (E3)
3. **Apple Business Connect** — claim the existing Apple Maps place card. Third-party guidance is unanimous that Siri/Spotlight/CarPlay place answers come from Apple's own place graph; Apple's own product page confirms Maps listings and a "Custom Actions" surface. **[VENDOR/INDUSTRY]** (E19, E7)
4. **Fix the entity data.** Verified today: site/GBP say Tue–Sun 10:00–22:00, closed Monday; the Yelp listing snippet shows Tue/Wed closing 20:00; Grubhub's live URL says **"3419 W Frye Rd"** (wrong; correct is 3491). Conflicting hours/address across sources is exactly the ambiguity Microsoft tells publishers to remove. **[TEST]** (E13, E5)

### Tier 2 — cheap content work with documented mechanism
5. **Review volume + recency + owner responses.** Google names review count/ratings under prominence; Yelp content now feeds ChatGPT and Perplexity. 13 reviews is the thinnest part of MAZA's profile. Owner responses are documented as a Google-supported signal of engagement. **[DOC + PRESS]** (E3, E10)
6. **Crawlable menu text.** MAZA already has 72 `MenuItem` + `Offer` entries on `/menu` — good. But the individual category pages (`/menu/wraps`, `/menu/plates`, `/menu/sides`, `/menu/specials` per the sitemap) do **not** carry `Menu`/`MenuItem` markup, only the page-level `Restaurant` block; and `/about`, `/contact` carry the same site-wide JSON-LD. Hoisting item markup onto category pages (or ensuring the category pages contain the items as visible text) is the cheapest structural upgrade. **[TEST]**
7. **Content shape that models lift.** Microsoft's own guidance: strong title/H1/description alignment, descriptive H2/H3, Q&A pairs, lists and tables, self-contained sentences, no key facts hidden in tabs or images, avoid walls of text. MAZA's FAQPage block (hours, halal, catering, parking) is already in this shape — it just should be visible on-page text too, not JSON-LD only. **[VENDOR]** (E13)
8. **`Restaurant` JSON-LD hygiene** (all verified today on the built site):
   - present: name, address, telephone, `openingHoursSpecification`, `servesCuisine` (Mediterranean / Middle Eastern / Halal), `hasMenu`, `geo`, `priceRange`, `knowsAbout`
   - **missing: `aggregateRating` / `review`** — Google's own LocalBusiness example includes a review block. Ratings are also what Yelp licenses to ChatGPT.
   - **`sameAs` contains only a Google Maps shortlink** — no Yelp, no Apple Maps, no Facebook/Instagram, no DoorDash/Grubhub. `sameAs` is the standard mechanism for telling a machine "these all are me"; using it for your own Maps URL wastes it.
   - `openingHours` string `"Mo closed; Tu-Su 10:00-22:00"` is not a schema.org/Google-conformant value (the correct form is `"Tu-Su 10:00-22:00"`; closures are expressed by omission or `OpeningHoursSpecification`). The `openingHoursSpecification` beside it is correct, so the malformed string is dead weight that only adds ambiguity.
   - every page emits the same `Restaurant`, `Dataset` and `FAQPage` blocks. `Dataset` is not a Google-supported rich-result type for a restaurant and reads as noise; `FAQPage` has not produced Google rich results for ordinary sites since August 2023 (restricted to authoritative gov/health sites) — it is harmless to keep but should not be counted on for Google, only as machine-readable Q&A for other consumers. **[DOC + TEST]** (E4, E20)
9. **Third-party citation surfaces, in priority order for this business:**
   - **Yelp** — now the single highest-value third-party surface (OpenAI + Perplexity licensing). Complete the profile: hours matching GBP, categories, photos, menu, attributes. Currently 13 reviews / 4.3★.
   - **Reddit** — cited heavily by Perplexity in third-party audits and licensed to OpenAI since May 2024. Chandler/East Valley food threads are where "where should I eat in Chandler" answers get grounded. Genuine participation only.
   - **Editorial roundups** — "best Mediterranean in Chandler" listicles (TripAdvisor/Yelp category pages, Eater/Thrillist-style local food press, city magazines). Third-party AI-visibility studies repeatedly list these as the cited-domain class for restaurant queries.
   - **Delivery marketplaces** — DoorDash and Grubhub both already carry MAZA (verified). They are cite-able surfaces for "where can I order X" queries, and Grubhub currently carries the wrong street number.
   - **TripAdvisor** — `***` no MAZA restaurant page surfaced in this research (only other Chandler Mediterranean venues). Absence is itself a gap in the citation set.
   - **Wikipedia/Wikidata** — not applicable at this scale; do not spend money here.
   **[PRESS/DOC/INDUSTRY + TEST]** (E10, E15, E17, E13)

### Tier 3 — do it, but do not expect it to be the lever
10. **`llms.txt`** — MAZA does not have one (`/llms.txt` returns 404 today). The spec exists and has real adoption (documentation platforms auto-generate it; Lighthouse audits for it). But Google stated publicly it does not support it, and no major AI crawler has published that its retrieval honors third-party `llms.txt`. Publishing a small, accurate one costs ~15 minutes and is not harmful; treat it as a hedge, not a strategy. **[DOC-adjacent/PRESS + TEST]** (E11)
11. **Freshness cadence** — sitemap `lastmod` is regenerated (verified: 2026-09-11) and IndexNow is free. Microsoft's framing is explicit that freshness keeps AI systems referencing the current version of a page. Cheap to automate. **[VENDOR]** (E5)

---

## 6. How assistants rank restaurants: verifiable vs. speculation

**Verifiable / on the record**
- **Index dependency.** ChatGPT search uses third-party search providers and lists the Microsoft privacy statement among them; Microsoft states Copilot is powered by Bing's index; Google states AI features run on the Search index and require normal indexation + snippet eligibility. → *A restaurant absent from Bing's index cannot be recommended by ChatGPT or Copilot no matter how good its schema is.* **[DOC/VENDOR]** (E1, E2, E5)
- **Crawler-level opt-in/out.** OAI-SearchBot governs ChatGPT search inclusion and automatic crawl; `ChatGPT-User` is a user-initiated fetch where robots.txt may not apply; GPTBot governs training, not search. Perplexity splits PerplexityBot (surfacing/links) from Perplexity-User (live fetch, ignores robots). Googlebot governs Search including AI features; `Google-Extended` controls Gemini grounding/training and is explicitly **not** a Search ranking signal. **[DOC]** (E2, E6, E8)
- **Query rewriting / fan-out.** OpenAI documents rewriting ("what are some good restaurants near me" → "top restaurants San Francisco"). Google documents query fan-out across subtopics and data sources. → *You are not optimizing for one query string; you are optimizing for the cluster of phrasings an assistant will generate about your cuisine and city.* **[DOC]** (E2, E1)
- **Review and link prominence.** Google names review volume and inbound links as prominence inputs. Yelp's licensing to OpenAI/Perplexity makes third-party review corpora a *direct* input into two assistants. **[DOC/PRESS]** (E3, E10)
- **Content-shape effects measured in research.** GEO-bench (KDD '24): Cite Sources +30–40% visibility, Quotation Addition +30–40%, Statistics Addition +30–40%; keyword stuffing *below* baseline; effects vary by domain; on live Perplexity the same methods moved position-adjusted word count up to +22% and subjective impression up to +37%. **[DOC]** (E12)
- **No pay-for-placement.** Google: no way to request or pay for better local ranking. OpenAI: placement is not guaranteed. **[DOC]** (E3, E2)

**Speculation (do not spend against this)**
- Any claim of a specific weighting (e.g. "reviews are 40% of the AI local algorithm"). No vendor publishes this; nobody has a controlled experiment at that granularity.
- Claims that structured data *directly* causes AI recommendations. There is no vendor statement tying JSON-LD to AI-answer inclusion. Structured data's documented jobs are search-feature eligibility and machine legibility; its AI effect is an inference, albeit a reasonable one, and Microsoft does recommend schema generally. **[VENDOR, weakly attributed]**
- The "AI-first directory" industry — dozens of new paid directories claiming to feed LLMs. No evidence of ingestion.
- Google's FAQ rich results returning. They have not, since Aug 2023, for ordinary sites. **[DOC]** (E20)

---

## 7. Gaps and unknowns

1. **Bing index status for mazahalalfood.com is unverified from this environment.** I attempted a live `site:` check; Bing served a bot-detection decoy page (unrelated results) from this sandbox. Must be checked from a clean residential browser or via Bing Webmaster Tools once verified. **This is the single most consequential unknown** — it gates ChatGPT and Copilot.
2. **Whether MAZA's Google Business Profile is claimed and complete** could not be verified from outside (GBP requires the owner account). Hours/category/attributes status is an owner-side check.
3. **Yelp listing contents could not be read directly** — Yelp blocks automated scraping (attempt returned an anti-bot abort). The hours conflict and 13-review/4.3★ figures come from search-result snippets and should be confirmed in the Yelp for Business dashboard.
4. **No assistant was queried directly** for this report (no accounts/APIs for ChatGPT, Gemini, Perplexity, Copilot, Siri in this environment). All "how it retrieves" claims are from documentation, not from live prompting. Live prompting is the remaining verification step and is cheap — see below.
5. **No TripAdvisor page found for MAZA**; confirmed absence would be a real citation gap.
6. **Actual AI-referral traffic to mazahalalfood.com is unknown** — if GA4 is live (tag `G-L5JJM9BBLY` is present), a referral-source segment (chatgpt.com, perplexity.ai, gemini.google.com, copilot.microsoft.com, apple.com) over 90 days would falsify or confirm the whole thesis. Nothing in this report should be scaled before that number is read.

---

## 8. The three cheapest tests (all runnable inside a week, near-zero cost)

**Test 1 — Bing presence + AI citation telemetry (30 minutes, free, highest information value).**
1. Open a clean browser (not this sandbox — it is bot-walled) and run `site:mazahalalfood.com` on Bing. Record whether the homepage and `/menu` are indexed.
2. Create/verify **Bing Places for Business** and claim the listing.
3. Add the domain to **Bing Webmaster Tools** and open the **AI Performance** dashboard. It reports total citations of your URLs in AI-generated answers and the grounding queries that retrieved them.
*Pass condition:* site indexed AND at least one MAZA URL appearing as a cited source. *If indexed but zero citations:* the problem is content/entity quality, not crawlability — move to tests 2 and 3. *If not indexed:* stop everything and fix indexation first; no other lever matters.

**Test 2 — Manual prompt matrix against the five assistants (45 minutes, free, run from a Chandler-area connection).**
Run each of these six prompts, verbatim, in a fresh chat: ChatGPT (with search), Gemini, Perplexity, Copilot, and Siri:
- "good food spots in Chandler AZ"
- "best Mediterranean restaurant in Chandler Arizona"
- "where can I get halal food near Chandler Mall"
- "best halal restaurant in Chandler"
- "Mediterranean food near me"
- "is there a good Mediterranean place on Frye Rd in Chandler"
Record for each: **is MAZA named? at what position? what sources are cited?** Screenshot everything, including the cited-source list. Repeat the same six prompts monthly in a dated doc so you get a trend instead of a snapshot.
*Why these prompts:* they mirror the real-world query the customer used, and they mirror the query-rewriting behaviour OpenAI documents (city substituted into a generic "near me" phrasing).

**Test 3 — Entity reconciliation audit (20 minutes, free, fixes a live error).**
Walk Yelp, Apple Maps, Bing Places, Google Business Profile, Grubhub, DoorDash and the website, and record name / street address / phone / hours side by side. Fix every mismatch in favour of the GBP values. **Known defect to fix immediately:** Grubhub lists **3419 W Frye Rd** (verified live today) where the correct address is **3491 W Frye Rd**. While in there, replace the website's `sameAs` Google-Maps-only value with the real Yelp, Apple Maps, Facebook/Instagram and delivery-profile URLs.
*Why it is a test and not just cleanup:* after the fix, re-run Test 2 in 2–4 weeks and see whether MAZA's appearance rate in "near me"-style prompts moves. That is the cleanest cheap causal signal available to a single-location restaurant.

---

## 9. Claims with evidence

| ID | Claim | Evidence | Quality | Status |
|---|---|---|---|---|
| C1 | Google AI Overviews/AI Mode require only that a page be indexed and snippet-eligible; "there are no additional technical requirements" | E1 | DOC | supported |
| C2 | AI Overviews/AI Mode use query fan-out across subtopics and data sources | E1 | DOC | supported |
| C3 | ChatGPT search uses third-party search providers and shares general location with them; queries are rewritten before being sent | E2 | DOC | supported |
| C4 | ChatGPT restaurant reservations draw availability from OpenTable, Resy or Yelp | E2 | DOC | supported |
| C5 | OAI-SearchBot governs whether a site is eligible to appear in ChatGPT search; GPTBot is for training, ChatGPT-User for user-initiated visits where robots.txt may not apply | E2, E8 | DOC | supported |
| C6 | Google local ranking factors are relevance, distance, prominence; prominence is informed by linking sites and review volume; no payment option exists | E3 | DOC | supported |
| C7 | Google sources Business Profile info from crawled web content, licensed third-party data, user contributions and Google's own interactions | E9 | DOC | supported |
| C8 | LocalBusiness/Restaurant structured data is Google-supported with a Restaurant example incl. review and openingHoursSpecification | E4 | DOC | supported |
| C9 | FAQ rich results have been limited to authoritative government/health sites since Aug 2023 | E20 | DOC | supported |
| C10 | Google-Extended controls Gemini grounding/training and is not a Search ranking signal | E8 | DOC | supported |
| C11 | PerplexityBot surfaces/links sites in Perplexity results; Perplexity-User performs user-initiated fetches that generally ignore robots.txt | E6 | DOC | supported |
| C12 | Microsoft Copilot is powered by Bing's search index, and Microsoft tells local businesses to register with Bing Places for inclusion in AI-generated answers | E5, E18 | VENDOR | supported |
| C13 | Bing Webmaster Tools now reports AI citations and grounding queries per URL | E5 | VENDOR | supported |
| C14 | Microsoft's own guidance for AI-answer inclusion is: schema, headings, Q&A pairs, lists/tables, self-contained answers, avoid hidden/PDF/image-only content, freshness via IndexNow | E5, E13 | VENDOR | supported |
| C15 | OpenAI licensed Yelp reviews/photos/business info into ChatGPT, announced 23 Jul 2026, non-exclusive | E10 | PRESS | supported |
| C16 | Yelp data also feeds Perplexity (licensed, Mar 2024) and Apple Maps (per Yelp CEO) | E14, E10 | PRESS | supported |
| C17 | OpenAI has licensed Reddit content since May 2024 | E15 | DOC | supported |
| C18 | Google publicly stated it does not support `llms.txt` and is not planning to (Jul 2025, Gary Illyes) | E11 | PRESS (reporting a public statement) | partial — single public statement reported by two industry outlets; no Google doc found |
| C19 | The `llms.txt` spec (v2) is real and being adopted by documentation platforms; it is most used for docs/agent reading | E11 | DOC (self-reported by spec author) | supported for adoption, not for assistant retrieval |
| C20 | Citation Addition, Quotation Addition and Statistics Addition improved generative-engine visibility 30–40% on position-adjusted word count; keyword stuffing scored below baseline | E12 | DOC (peer-reviewed, KDD '24; GPT-3.5-era engine) | supported, with the caveat that the engines tested are 2024-vintage |
| C21 | mazahalalfood.com robots.txt allows GPTBot, OAI-SearchBot and Google-Extended and declares a sitemap | E13 | TEST | supported |
| C22 | MAZA has no `/llms.txt` (HTTP 404) | E13 | TEST | supported |
| C23 | MAZA's `/menu` page carries Menu/MenuSection/MenuItem/Offer JSON-LD (72 MenuItem, 13 MenuSection, 72 Offer); its category pages do not | E13 | TEST | supported |
| C24 | MAZA's Restaurant JSON-LD lacks `aggregateRating`/`review`, and `sameAs` contains only a Google Maps shortlink | E13 | TEST | supported |
| C25 | MAZA's `openingHours` string ("Mo closed; Tu-Su 10:00-22:00") is non-conformant, though `openingHoursSpecification` is correct | E13 | TEST | supported |
| C26 | Grubhub lists MAZA at "3419 W Frye Rd" — a wrong street number on a live URL | E13 | TEST | supported |
| C27 | A Yelp listing exists for MAZA (13 reviews, 4.3★) and its displayed hours conflict with the site's Tue–Sun 10:00–22:00 | E13 | TEST (search-snippet level; live page bot-blocked) | partial |
| C28 | MAZA has an Apple Maps place page and DoorDash + Grubhub store pages | E13, E7 | TEST | supported |
| C29 | No TripAdvisor page for MAZA was found | E13 | TEST (negative search) | partial — absence of evidence |
| C30 | Bing indexation of mazahalalfood.com could not be verified from this environment (bot-decoy page returned) | E13 | TEST | unsupported (blocked) |
| C31 | Third-party audits report Perplexity leans on Reddit, niche directories and editorial roundups for local queries | E17 | INDUSTRY | partial |

## 10. Evidence index

| ID | Source | URL | Provenance / extraction |
|---|---|---|---|
| E1 | Google Search Central — "AI features and your website" | https://developers.google.com/search/docs/appearance/ai-features | Firecrawl markdown, last updated 2025-12-10 |
| E2 | OpenAI Help Center — "Searching the web with ChatGPT" | https://help.openai.com/en/articles/9237897-chatgpt-search | Firecrawl markdown, updated ~21 days before capture |
| E3 | Google Business Profile Help — "Tips to improve your local ranking" | https://support.google.com/business/answer/7091 | Firecrawl markdown |
| E4 | Google Search Central — LocalBusiness structured data | https://developers.google.com/search/docs/appearance/structured-data/local-business | Firecrawl markdown, last updated 2026-09-08 |
| E5 | Bing Webmaster Blog — "Introducing AI Performance in Bing Webmaster Tools Public Preview" (Feb 2026) | https://blogs.bing.com/webmaster/February-2026/Introducing-AI-Performance-in-Bing-Webmaster-Tools-Public-Preview | Firecrawl markdown |
| E6 | Perplexity docs — Crawlers | https://docs.perplexity.ai/guides/bots | Firecrawl markdown |
| E7 | Apple Maps place page for MAZA | https://maps.apple.com/place?place-id=I7007BFFB5FD13BAB | search-result URL; listing existence corroborated by two independent searches |
| E8 | Google — list of common crawlers (Googlebot / Google-Extended) | https://developers.google.com/search/docs/crawling-indexing/google-common-crawlers | Firecrawl markdown, last updated 2026-07-14 |
| E9 | Google Business Profile Help — how Google sources business info | https://support.google.com/business/answer/2721884 | Firecrawl markdown |
| E10 | Axios (exclusive) — Yelp licenses reviews to OpenAI (23 Jul 2026) | https://www.axios.com/2026/07/23/yelp-reviews-chatgpt-geo-partnership | Firecrawl markdown |
| E11 | llms.txt spec v2 | https://llmstxt.org/ ; Google non-support reporting: https://searchengineland.com/google-says-normal-seo-works-for-ranking-in-ai-overviews-and-llms-txt-wont-be-used-459422 and https://www.seroundtable.com/openai-crawling-llms-txt-files-39811.html | Firecrawl markdown + search result descriptions |
| E12 | Aggarwal et al., "GEO: Generative Engine Optimization", KDD '24 | https://arxiv.org/abs/2311.09735 ; full text https://arxiv.org/html/2311.09735v3 (Tables 1 & 7) | arXiv HTML, tables read directly |
| E13 | Direct audit of mazahalalfood.com | site build + curl of `/`, `/menu`, `/menu/wraps`, `/about`, `/contact`, `/robots.txt`, `/llms.txt`, `/sitemap.xml`; live Grubhub URL HEAD; Yelp snippet via search | Terminal run, 2026-09-11 |
| E14 | The Verge — Perplexity brings Yelp data to its chatbot (Mar 2024) | https://www.theverge.com/2024/3/12/24098728/perplexity-chatbot-yelp-suggestions-data-ai | search result + description |
| E15 | OpenAI — OpenAI and Reddit partnership (May 2024) | https://openai.com/index/openai-and-reddit-partnership/ | Firecrawl markdown |
| E16 | OpenAI — Introducing ChatGPT search (Oct 2024) | https://openai.com/index/introducing-chatgpt-search/ | Firecrawl markdown |
| E17 | Zayrev — "Perplexity's Local Results Algorithm: 200 queries tested" | https://www.zayrev.com/blog/perplexity-local-algorithm-200-queries-tested | search result description only — methodology not inspected |
| E18 | Bing Places for Business (product page) | https://www.bingplaces.com/ | Firecrawl markdown |
| E19 | Apple Business Connect / Apple Business (product page) | https://businessconnect.apple.com/ | Firecrawl markdown |
| E20 | Google Search Central — changes to HowTo and FAQ rich results (Aug 2023) | https://developers.google.com/search/blog/2023/08/howto-faq-changes | Firecrawl markdown |
| E21 | Microsoft Advertising — "Optimizing Your Content for Inclusion in AI Search Answers" (Oct 2025) | https://about.ads.microsoft.com/en/blog/post/october-2025/optimizing-your-content-for-inclusion-in-ai-search-answers | Firecrawl markdown |
| E22 | MAZA live pages/robots/sitemap/llms.txt probe outputs | `/srv/scratch/maza/` (ephemeral) — findings reproduced in C21–C29 | Terminal, 2026-09-11 |
| E23 | OpenAI — Overview of OpenAI crawlers | https://platform.openai.com/docs/bots | Firecrawl markdown |

## 11. Confidence

- **Report-level confidence: medium-high** for the mechanics (sections 3–6) and **medium** for the MAZA-specific recommendation ordering.
- Rationale: the retrieval/eligibility mechanics rest on first-party documentation from Google, OpenAI, Microsoft and Perplexity, plus a peer-reviewed KDD paper — that part is solid. The MAZA-specific part rests on a live audit of the site (solid) but only snippet-level evidence for Yelp (bot-blocked) and no verification at all of Bing indexation or of live assistant answers from this environment. Two of the three cheapest tests exist precisely to close those gaps; until they run, the lever ordering should be treated as a hypothesis, not a plan.

## 12. Recommended next actions

1. **Today / free:** fix Grubhub's street number (3419 → 3491) and make Yelp's hours match GBP. Both are live errors feeding AI surfaces.
2. **This week / free:** claim Bing Places, add the site to Bing Webmaster Tools, open the AI Performance dashboard (Test 1). Independently, claim the Apple Business Connect place card.
3. **This week / free:** run the six-prompt matrix across ChatGPT, Gemini, Perplexity, Copilot and Siri from a Chandler connection; save dated screenshots (Test 2). This is the baseline everything else gets measured against.
4. **This week / free:** pull GA4 referral traffic for chatgpt.com / perplexity.ai / gemini.google.com / copilot.microsoft.com / apple.com over the last 90 days. If it is non-zero, the channel is real and measurable; if it is zero, the customer anecdote was a one-off and priorities should shift.
5. **Next 2–3 weeks, cheap:** raise Yelp review volume (post-purchase ask, QR on receipt), with owner responses on every review — this is now the highest-value third-party surface because OpenAI and Perplexity license it.
6. **Next month, dev work (repo: `maza-mediterranean`):** extend `Menu`/`MenuItem` markup to the individual menu category pages; add `aggregateRating`/`review`; replace `sameAs` with real Yelp/Apple/Facebook/Instagram/delivery URLs; correct the `openingHours` string; consider dropping the `Dataset` block. Publish a small `llms.txt` as a hedge only.
7. **Do not:** buy "AI directory" placements, rewrite copy for keyword density (documented to reduce visibility), or expect FAQPage to produce Google rich results.

---

*One archive only: `/home/ice/know/research/ai-local-discovery-maza-t_9c26f53d.md` (git → arealicehole/know). Kanban attach is the handoff; no copies were written to fed/docs or research-output.*

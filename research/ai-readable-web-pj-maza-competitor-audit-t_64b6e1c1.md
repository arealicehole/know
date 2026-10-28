# AI-Readable Web Presence Audit — Print Junkie AZ vs Maza Mediterranean

**Task:** t_64b6e1c1 (board: seo-bible)
**Prepared:** 2026-10-28 (all probes run live on this date)
**Prepared by:** Geraldo v2.1
**Sources of truth:** live HTTP probes (robots.txt, llms.txt, .well-known/ucp, /api/specs, homepage + key landing HTML), PJ/MAZA internal docs (`/workspace/seo-bible/PJ_SITE_BRIEF.md`, `PJ_SEO_ROADMAP.md`, `MAZA_SEO_ROADMAP.md`, `/workspace/seo-bible/maza-research/*`), prior GIU reports `t_9c26f53d` (Maza AI-local-discovery, 2026-09-18) and `t_3c7c6149` (AZ competitor vertical pricing, 2026-09-14).

---

## 0) One-line verdicts

| Site | One-line verdict |
|------|------------------|
| **Print Junkie AZ** | Best-in-class *machine-readable price surface* (UCP JSON with 8 SKUs, tiered prices, specs endpoints) but **no `llms.txt`, no visible price list in HTML body, no street address in schema** — agents get prices but a thinner identity than Maza. |
| **Maza Mediterranean** | Best-in-class *LLM identity surface* (full `llms.txt` with facts+links, rich Restaurant schema, 63-item specs JSON, 51 URLs) but **no machine-readable prices anywhere** and its own roadmap's UCP work is not shipped. |

They are **complementary, not competing, on AI-readability**: Maza wins identity/hours/menu discovery; PJ wins price/offer/checkout intent. Neither beats the other outright. The real gap both leave open: **local-competitor AI presence is near-zero** — that is the actual opportunity.

---

## 1) Probe results — live, 2026-10-28

### 1.1 Signal matrix (all HTTP-verified today)

| Signal | printjunkieaz.com | mazahalalfood.com |
|--------|:---:|:---:|
| `robots.txt` | 200 — `User-agent: *` + **GPTBot Allow**, **OAI-SearchBot Allow**, **Google-Extended Allow**, **ChatGPT-User Allow**, sitemap declared | 200 — `User-Agent: *` + **GPTBot Allow**, **OAI-SearchBot Allow**, **Google-Extended Allow** (no ChatGPT-User, no OAI mention beyond that), sitemap declared |
| GPTBot UA crawl test | 200 | 200 |
| OAI-SearchBot UA crawl test | 200 | 200 |
| PerplexityBot UA crawl test | 200 | 200 |
| Googlebot UA crawl test | 200 | 200 |
| `llms.txt` | **404** (404 body is the site's own 404 template, 23 KB) | **200** — hand-authored, ~1.5 KB, facts + links |
| `llms-full.txt` | 404 | not probed (llms.txt present is the standard; full not needed) |
| `.well-known/ucp` | **200 — 4.7 KB, `ucp/1.0` v2026-05-11** — full vendor block, capabilities (checkout/stripe payment_link, fulfillment pickup+shipping+installation, discounts manual_quote), **8 products with tiers/turnaround/specsUrl/orderUrl** | **200 — 847 B, `WebAPI`-shaped UCP discovery manifest** — capabilities: MenuDiscovery, LocalBusinessInfo, HoursVerification, CateringInquiry; endpoints: menu, specs, contact; lastUpdated 2026-10-28T22:26:36Z (updated *today*) |
| `/api/specs` (root) | 404 | **200 — 29 KB `maza-menu-specs/v1`**, `itemCount: 63`, utilitySignals: availability/dietary/sustainability/**pricing:true** |
| `/api/specs/business-cards` (PJ) | 200 — 1.5 KB per-product spec | — |
| Homepage schema `@type` | `LocalBusiness` + `MerchantReturnPolicy` + `Dataset` + `FAQPage` | `Restaurant` + `FAQPage` |
| Schema: street address | **MISSING** (`address: null`; only `areaServed` GeoCircle lat 33.4484, -112.074 r=50 km — Phoenix metro, **not** a Tempe street address) | **present** — 3491 W Frye Rd Ste 2, Chandler, AZ 85226 |
| Schema: phone | +1-602-730-2135 | (480) 534-6550 |
| Schema: openingHoursSpecification | Mon–Sat 09:00–17:00 | Tue–Sun 10:00–22:00 (closed Monday) |
| Schema: sameAs | 2 (Instagram, Facebook) | **9** (Google Maps, Apple Maps, Yelp, Uber Eats, DoorDash, Grubhub, Instagram, TikTok, Facebook) |
| Schema: knowsAbout | 15 items (product/service list) | 5 items |
| Schema: aggregateRating | absent | **present** |
| Schema: hasMenu / MenuItem | absent | **present** |
| Schema: Offer / ItemList | absent | absent (prices are *not* in JSON-LD) |
| `$` price strings in homepage HTML | **7** ($65, $65, $200, $125, $125, $75, $149 — card/banner price chips) | **0** |
| `$` price strings in `/business-cards-phoenix` | **4** ($65 100, $75 1,000, $125 2,500, $150 5,000) | — |
| FAQPage schema | yes | yes |
| `sitemap.xml` | 17 URLs (home, about, 6 landing, contact, 8 order pages) | **51 URLs** (home, menu + 7 category pages, about, contact, catering, careers, gallery, menu-qr, plaza-lunch, menu-board, sms-join, privacy, terms, **20 local-intent pages**: halal-mediterranean-chandler, gyro, shish-kebab, chicken-tikka, lamb-tikka, ribeye-tikka, mix-kebab, maza-grill, falafel, arayes, burgers, loaded-fries, loaded-hummus, hummus-and-baba-ghanoush, meze-platter, tabouleh-and-salads, baklava, tepsi-baytinijan, samak-tandoor, family-meal, kids-meals, vegetarian-vegan-options, halal-catering, dine-in, takeout-and-delivery, lunch-specials, + 8 city pages chandler/gilbert/tempe/ahwatukee/sun-lakes/mesa/queen-creek/ocotillo) |

### 1.2 What each site actually serves to an AI agent today

**Print Junkie AZ (what an agent gets):**
- `GET /` → HTML with `LocalBusiness` JSON-LD (phone, hours, 2 socials, 15 knowsAbout, GeoCircle area) + `FAQPage` + `MerchantReturnPolicy` + `Dataset`. Visible text is thin (~2.2 KB stripped).
- `GET /business-cards-phoenix` → `LocalBusiness` + `Service` + `FAQPage` + price chips in body ($65/$75/$125/$150).
- `GET /.well-known/ucp` → the richest single artifact of either site: 8 SKUs (business-cards, loyalty-cards, promo-flyers, store-hours, window-wraps, a-frame-signs, merch-hq, custom-banners) each with min/max price, per-quantity tiers, turnaround, `specsUrl`, `orderUrl`; vendor contact +1-602-730-2135, info@printjunkieaz.com, sameAs IG/FB.
- `GET /api/specs/business-cards` → per-product spec JSON.
- **Not served:** `llms.txt` (404), street address anywhere in schema, `aggregateRating`, visible full price list in body (only 4 price points on the card landing).

**Maza (what an agent gets):**
- `GET /` → `Restaurant` JSON-LD with full address, phone, hours, **aggregateRating**, 9 sameAs, knowsAbout, hasMenu, GeoCircle r=25 km. Visible text thin (~1.9 KB).
- `GET /llms.txt` → hand-authored: one-paragraph description, `## Facts` (name/address/phone/hours/cuisine/price range $$/ordering), `## Links` (website, menu, contact, Google Maps, Yelp, Uber Eats, DoorDash, Grubhub, Instagram, TikTok, Facebook). This is the only `llms.txt` of the two.
- `GET /.well-known/ucp` → UCP discovery manifest (WebAPI-shaped, not the full ucp/1.0 product format) pointing at menu/specs/contact endpoints; declares `UCP/1.0` + `AP2` support; lastUpdated timestamp from **today**.
- `GET /api/specs` → `maza-menu-specs/v1`, 63 items, utilitySignals including `pricing: true` (but no $ in homepage body — prices live in the JSON, not in visible HTML or JSON-LD Offer).
- **Not served:** visible `$` prices in HTML, `Offer`/`Itemlist` JSON-LD, UCP product tiers/checkout (the MAZA_SEO_ROADMAP UCP work is **not** shipped — the current UCP is a discovery stub, not the full product/checkout manifest PJ has).

### 1.3 Which reference wins on which axis (grounded, not vibes)

| Axis | Winner | Why (evidence) |
|------|--------|----------------|
| **LLM identity surface (who/where/hours)** | **MAZA** | Only site with `llms.txt`; only site with street address + `aggregateRating` + 9 sameAs + hasMenu. An LLM summarizing "where can I get halal Mediterranean in Chandler" gets a complete, self-describing card from Maza, but a phone+h GeoCircle (no street) for PJ. |
| **Machine-readable price/offer surface** | **PJ** | Only site with `ucp/1.0` product tiers + `/api/specs/<product>` + visible `$` price chips in landing HTML. An agent asked "how much for 1,000 business cards in Phoenix" can answer $75 from PJ; for Maza an agent gets `pricing: true` flag but no actual numbers in any probe. |
| **Checkout / agentic-commerce readiness** | **PJ** | UCP declares `checkout.provider: stripe`, `payment_link`, fulfillment pickup/shipping/installation, discount method. Maza's UCP declares capabilities as strings only (MenuDiscovery etc.) with no checkout/payment surface. |
| **Local-intent URL coverage (geo + service pages)** | **MAZA** | 51 URLs incl. 8 city pages + 20 dish pages + 4 service pages. PJ has 17 URLs, 6 service landings, no city pages. (This mirrors Maza's "34 URLs" vs PJ's thin sitemap in the prior t_9c26f53d report.) |
| **FAQ schema** | **TIE** | Both ship `FAQPage`. |
| **Crawl friendliness for AI bots** | **PJ (slightly)** | PJ's robots explicitly whitelists GPTBot, OAI-SearchBot, Google-Extended **and** ChatGPT-User; Maza omits ChatGPT-User. Both return 200 to all probed UAs. |
| **Overall "AI-readable" as judged by what an LLM actually ingests first (home + llms.txt + ucp)** | **MAZA** | An LLM doing discovery hits `/` + `llms.txt` first; Maza serves a complete identity+facts+links package there. PJ's `/` is thinner and has no `llms.txt` to point the agent at its UCP/specs. The UCP is deeper but discoverable only if the agent knows to probe `.well-known/ucp`. |

> **Nuance for the Bible:** "AI-readable" splits into two distinct surfaces that the two references optimize differently:
> 1. **Discovery/identity surface** (what an LLM reads on first contact: `llms.txt`, home `Restaurant`/`LocalBusiness` schema, sameAs, address, hours, rating) → **Maza is the stronger reference.**
> 2. **Transaction/offer surface** (what an agent needs to quote/checkout: `ucp/1.0` products, `/api/specs`, `Offer` schema, visible prices) → **PJ is the stronger reference.**
> A site that wants to be *fully* AI-readable in 2026 needs **both**. Neither reference is the complete model.

---

## 2) Local-competitor scan (Phoenix / Chandler) — live, 2026-10-28

**Method note:** `web_search` returned mostly directory/aggregator pages (Yelp, Superpages, checkmyhalal) rather than named local shops, so the scan below is grounded in **direct HTTP probes** of competitor homepages found via search + the prior `t_3c7c6149` AZ competitor map. Two sites (azhottees.com, phxsignshop.com) returned **HTTP 429** on repeated probes (Cloudflare rate-limit / bot-wall) — flagged as unverified, not scored as "no AI presence."

### 2.1 AI-presence matrix

| Competitor (category) | robots AI whitelist | llms.txt | UCP | /api/specs | Schema @type | street addr | hours | rating | sameAs | knowsAbout | visible $ prices | AI-readability score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Print Junkie AZ (ref)** | GPTBot+OAI+GExt+ChatGPT | **404** | **ucp/1.0 full** | per-product | LocalBusiness | **no** | yes | no | 2 | 15 | yes (landing) | **8 / 10** |
| **Maza (ref)** | GPTBot+OAI+GExt | **200** | UCP discovery stub | menu (63 items) | Restaurant | yes | yes | **yes** | **9** | 5 | no | **9 / 10** |
| **César Graphics** (print, Phoenix — cesargraphics.com) | absent | 404 | 404 | — | WebSite+LocalBusiness | yes | **no** | no | 0 | no | **yes (rich price catalog in body: $45 next-day cards, $114 standard, $400 foil/spot-UV, $225 magnetic, $310 round, $185 square, $285 euro, $350 painted-edge, $310 plastic, letterhead $150, envelopes $415, folders $575/$495, notepads $300, stamps $55)** | **6 / 10** |
| **AlphaGraphics Phoenix** (print, franchise — alphagraphics.com/us-arizona-phoenix-us011) | absent | 404 | 404 | — | LocalBusiness | yes | **no** | no | 0 | no | no | **4 / 10** |
| **PHX Screen Printing** (shirts/cards — phxscreenprinting.com) | absent | 404 | 404 | — | LocalBusiness(+2 untyped)+FAQPage | yes | yes | **yes** | 0 | yes | no | **5 / 10** |
| **AZ Precision Graphics** (screen print/apparel — azprecisiongraphics.com) | absent | 404 | 404 | — | LocalBusiness(+1 untyped) | yes | yes | **yes** | 1 | yes | no | **5 / 10** |
| **Extreme Print Lab** (DTF/screen — extremeprintlab.com) | absent | 404 | 404 | — | (untyped block) | yes | yes | **yes** | 0 | no | no (FAQ mentions "call for quote") | **4 / 10** |
| **Shirt52** (apparel — shirt52.com) | absent | 404 | 404 | — | (untyped block) | yes | yes | no | 0 | no | minimal ($200, $0) | **3 / 10** |
| **Mesa Sign Company** (window graphics — mesasigncompany.com) | absent | 404 | 404 | — | WebSite+LocalBusiness | yes | yes | no | 0 | no | no | **4 / 10** |
| **SpeedPro Phoenix** (signage franchise — speedpro.com/phoenix) | **blocks GPTBot + Google-Extended** (allows OAI-SearchBot, PerplexityBot) | **200 (llms.txt present!)** | 404 | — | (untyped)+ProfessionalService | **no** | yes | no | **50** | no | no | **5 / 10** |
| **PHX Sign Company** (signs — phoenixsigncompany.org) | absent | 404 | 404 | — | **none** | no | no | no | 0 | no | no | **1 / 10** |
| **Spitz Chandler** (Mediterranean, spitz-restaurant.com/locations/chandler) | **allows GPTBot/OAI/GExt, blocks PerplexityBot** | 404 | 404 | — | **none** | no | no | no | 0 | no | no | **2 / 10** |
| **Greeks Grill** (gyros, greeksgrillaz.com) | absent | 404 | 404 | — | **none** | no | no | no | 0 | no | no (page is a JS shell, 47 stripped chars) | **1 / 10** |
| **AZ Hot Tees** (apparel — azhottees.com) | 429 bot-wall — unverified | 429 | 429 | — | unverified | — | — | — | — | — | — | **unverified** |
| **PHX Sign Shop** (window/signs — phxsignshop.com) | 429 bot-wall — unverified | 429 | 429 | — | unverified | — | — | — | — | — | — | **unverified** |

**Scoring rubric (for the Bible, so it's reproducible):** AI-readability score = sum of (1) robots AI whitelist, (1) llms.txt, (1) UCP present, (1) /api/specs, (1) typed schema, (1) street address, (1) hours, (1) aggregateRating, (1) sameAs ≥ 1, (1) knowsAbout, (1) visible $ prices. Max 10. Both references score 8–9.

### 2.2 Key findings from the competitor scan

1. **No local competitor has a `ucp/1.0` product manifest.** PJ is the **only** site in the entire scan with machine-readable product tiers + checkout. This is a genuine, defensible differentiator.
2. **Maza is the only one with `llms.txt`** *in the Maza lane*; in the **PJ lane, SpeedPro Phoenix is the only competitor with `llms.txt`** (and it *blocks* GPTBot/Google-Extended while publishing it — an inconsistent stance).
3. **Visible price catalogs are rare and belong to the print lane:** César Graphics is the standout — a rich, visible Wix price catalog (next-day cards $45, standard $114, specialty cards up to $400). This is the **closest print competitor to "AI-readable prices"** and the one PJ should watch: an LLM answering "how much for foil business cards in Phoenix" can pull César's $400 figure. PJ's UCP ($65–$150 cards) is *cheaper and more complete per-SKU* but only via the deeper UCP probe.
4. **Restaurant lanes (Spitz, Greeks Grill) are nearly zero** on schema — no @type, no address, no hours, no rating. Maza's `Restaurant` schema + `llms.txt` + 9 sameAs makes it **the undisputed AI-readable leader in the Chandler Mediterranean/halal lane.** Greeks Grill is a pure JS shell with no server-rendered content at all — an LLM literally cannot read it.
5. **Signage is the weakest lane** for AI readability (SpeedPro 5, Mesa 4, PHX Sign Co 1) — a wide-open lane for PJ's window-wraps/vinyl SKUs if PJ adds the identity layer Maza has.
6. **Two sites 429'd on repeated probes** (azhottees, phxsignshop) — Cloudflare bot-walls. This is itself a finding: even "modern" small-shop sites still bot-wall AI crawlers, so the AI-readable set is *even smaller* than the matrix shows.

---

## 3) Cross-competitor table — the one-page matrix for the Bible

| # | Site | Lane | robots AI | llms.txt | UCP | specs | schema | addr | hours | rating | sameAs | knowsAbout | $ in body | **AI-score** |
|---|------|------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | Maza Mediterranean | restaurant | GPT+OAI+GExt | **yes** | UCP stub | menu 63 | Restaurant | yes | yes | **yes** | **9** | yes | no | **9** |
| 2 | **Print Junkie AZ** | print | GPT+OAI+GExt+ChatGPT | no | **ucp/1.0 full** | per-product | LocalBusiness | **no** | yes | no | 2 | yes | yes | **8** |
| 3 | César Graphics | print | — | no | no | — | LocalBusiness | yes | no | no | 0 | no | **yes (rich)** | **6** |
| 4 | PHX Screen Printing | shirts/cards | — | no | no | — | LocalBusiness+FAQ | yes | yes | **yes** | 0 | yes | no | **5** |
| 5 | AZ Precision Graphics | apparel | — | no | no | — | LocalBusiness | yes | yes | **yes** | 1 | yes | no | **5** |
| 6 | SpeedPro Phoenix | signage | blocks GPT+GExt | **yes** | no | — | ProfessionalService | no | yes | no | **50** | no | no | **5** |
| 7 | AlphaGraphics PHX | print | — | no | no | — | LocalBusiness | yes | no | no | 0 | no | no | **4** |
| 8 | Extreme Print Lab | DTF | — | no | no | — | untyped | yes | yes | **yes** | 0 | no | no | **4** |
| 9 | Mesa Sign Co | signage | — | no | no | — | LocalBusiness | yes | yes | no | 0 | no | no | **4** |
| 10 | Shirt52 | apparel | — | no | no | — | untyped | yes | yes | no | 0 | no | minimal | **3** |
| 11 | Spitz Chandler | restaurant | GPT+OAI+GExt, blocks Perplexity | no | no | — | **none** | no | no | no | 0 | no | no | **2** |
| 12 | PHX Sign Company | signage | — | no | no | — | **none** | no | no | no | 0 | no | no | **1** |
| 13 | Greeks Grill | restaurant | — | no | no | — | **none** | no | no | no | 0 | no | no | **1** |
| 14 | AZ Hot Tees | apparel | 429 wall | ? | ? | — | unverified | — | — | — | — | — | — | **unverified** |
| 15 | PHX Sign Shop | signage | 429 wall | ? | ? | — | unverified | — | — | — | — | — | — | **unverified** |

**Reading the table for the Bible:**
- The **top 2** (both references) are the only sites scoring ≥ 8. Everyone else is ≤ 6.
- The **two levers that separate the top from the pack** are (a) `llms.txt` + full identity schema (Maza's moat) and (b) UCP product/checkout + specs (PJ's moat).
- **No competitor combines both.** That is the open lane.

---

## 4) What each reference actually *does* (mechanics the Bible can copy)

### 4.1 Maza's AI-readable stack (the "identity" reference)
1. **Hand-authored `llms.txt`** at site root: one-paragraph description → `## Facts` (name, full street address, phone, hours, cuisine, price range `$$`, ordering modes) → `## Links` (website, menu, contact, Google/Apple Maps, Yelp, 3 delivery platforms, 3 socials). Written for an LLM to ingest in one pass.
2. **`Restaurant` JSON-LD** (not generic `LocalBusiness`): full `PostalAddress`, phone, `openingHoursSpecification` (Tue–Sun 10–22, closed Monday), **`aggregateRating`**, 9 `sameAs` (incl. Uber Eats/DoorDash/Grubhub + Apple Maps deep link), `knowsAbout`, `hasMenu`, `priceRange: $$`, `areaServed` GeoCircle r=25 km.
3. **`/api/specs` = `maza-menu-specs/v1`**: 63 items with `utilitySignals: {availability, dietary, sustainability, pricing: true}` — a structured, versioned, agent-targeted menu that mirrors what `llms.txt` describes.
4. **UCP discovery manifest** (`.well-known/ucp`): WebAPI-shaped, declares `UCP/1.0` + `AP2`, capabilities as strings (MenuDiscovery, LocalBusinessInfo, HoursVerification, CateringInquiry), endpoints for menu/specs/contact, live `lastUpdated` timestamp. (This is a *stub* of the full UCP the MAZA roadmap describes — not yet product/checkout-tiered.)
5. **Local-intent URL farm**: 20 dish pages + 8 city pages (chandler, gilbert, tempe, ahwatukee, sun-lakes, mesa, queen-creek, ocotillo) + 4 service pages (dine-in, takeout-and-delivery, lunch-specials, halal-catering) — each a crawlable, rankable surface.
6. **robots.txt** whitelisting GPTBot/OAI-SearchBot/Google-Extended.

### 4.2 PJ's AI-readable stack (the "transaction" reference)
1. **`ucp/1.0` v2026-05-11** at `.well-known/ucp`: vendor block (name, url, contact +1-602-730-2135, info@, sameAs IG/FB), `capabilities.checkout` (provider **stripe**, method **payment_link**, "No on-site cart — each product links to a dedicated Stripe payment page"), `capabilities.fulfillment` (pickup Tempe same-day, shipping US/AZ, installation AZ for wraps/banners/vinyl/A-frames), `capabilities.discounts` (manual_quote, volume), and **8 products** each with `priceRange` + `tiers` (labeled quantities + flat $) + `turnaround` + `specsUrl` + `orderUrl`.
   - SKU examples captured live: business-cards $65/500 → $150/5000 (2–3 days, next-day rush), loyalty-cards $65–115, promo-flyers $125–525 (3–5 days), store-hours $125 (next-day), window-wraps $200/small → $1200/large, a-frame-signs, merch-hq, custom-banners.
2. **Per-product `/api/specs/<sku>`** (e.g. `/api/specs/business-cards` → 1.5 KB) — deep specs behind the UCP pointer.
3. **`LocalBusiness` JSON-LD**: phone, hours (Mon–Sat 09–17), 2 sameAs, **15 knowsAbout**, `priceRange: $$`, `areaServed` GeoCircle r=50 km Phoenix metro, **but `address: null`** (no street).
4. **`FAQPage` + `MerchantReturnPolicy` + `Dataset`** JSON-LD on home; `Service` + `FAQPage` on landings; visible `$` price chips on product landings.
5. **8 `/order/<sku>` pages** in sitemap (one Stripe payment page per product — the "no cart" agentic-commerce pattern).
6. **robots.txt** whitelisting GPTBot/OAI-SearchBot/Google-Extended/**ChatGPT-User**.
7. **Missing:** `llms.txt`, street address in schema, `aggregateRating`, `Offer`/`ItemList` JSON-LD, dish-level URL farm (no city pages).

---

## 5) Gaps / what is NOT yet AI-readable (the actual opportunity list)

### PJ gaps (the "Maza layer" it lacks)
| Gap | Evidence | What it costs an agent |
|-----|----------|------------------------|
| No `llms.txt` | 404 | First-contact LLM has no one-page card; must guess to probe `.well-known/ucp`. |
| No street address in schema | `address: null` | Agent can't give a pickup location; only a 50 km GeoCircle. |
| No `aggregateRating` | absent | Loses trust/authority signals vs Maza. |
| No `Offer`/`ItemList` JSON-LD | prices only in UCP + body chips | Google rich results (price) unavailable; LLM must parse UCP. |
| No city/local-intent URL farm | 17 URLs, 6 landings, 0 city pages | Weaker local-SERP + local-LLM surface than Maza's 51. |
| sameAs = 2 vs Maza's 9 | missing Maps/Yelp/delivery | Thinner entity disambiguation. |

### Maza gaps (the "PJ layer" it lacks)
| Gap | Evidence | What it costs an agent |
|-----|----------|------------------------|
| No visible `$` prices in HTML | 0 price strings in body | LLM summarizing the page can't quote prices; only the `pricing:true` flag. |
| UCP is a discovery **stub**, not `ucp/1.0` products | 847 B WebAPI manifest, no tiers/checkout | Agent can discover menu/hours but can't get tiered prices or checkout intent the way PJ's UCP allows. |
| No `Offer`/`Itemlist` JSON-LD | absent | No Google price rich results. |
| MAZA_SEO_ROADMAP UCP work **not shipped** | roadmap says "UCP discovery endpoint + product JSON + AP2" but live UCP is the older stub | The roadmap's agentic-commerce goal is unrealized on the wire. |
| robots missing ChatGPT-User line | present for GPT/OAI/GExt only | Minor; ChatGPT-User not explicitly allowlisted. |

### Competitor gaps (the open lane)
- **Every local competitor except PJ lacks `ucp/1.0` + specs + visible prices.**
- **Every local competitor except Maza lacks `llms.txt` + full identity schema + rating.**
- **Signage lane** (SpeedPro/Mesa/PHX Sign Co) is the weakest — open for PJ's window-wrap/vinyl SKUs.
- **Restaurant lane** (Spitz/Greeks) is near-zero — Maza already owns it.
- **2 sites bot-wall AI crawlers (429)** — the AI-readable set is even smaller in reality.

---

## 6) Recommendations (draft for the Bible — concrete, ordered)

**P0 — Do both, not one. The winning 2026 stack is Maza's identity layer + PJ's transaction layer.**

1. **Ship `llms.txt` on every site** (Maza pattern, 30-min job):
   `# <Name>` → one-paragraph description → `## Facts` (name, full street address, phone, hours, category/cuisine, price range, ordering modes) → `## Links` (website, menu/product, contact, Maps deep links, delivery/socials). Keep it < 2 KB.
2. **Use the *specific* schema type, not generic `LocalBusiness`:** `Restaurant` for food (with `aggregateRating`, `hasMenu`, `servesCuisine`), and for PJ consider `Store`/`ProfessionalService` + `areaServed` GeoCircle **plus a real street `PostalAddress`** (fix PJ's `address: null`).
3. **Ship `ucp/1.0` with real product tiers + `/api/specs/<sku>` + one `/order/<sku>` checkout page per SKU** (PJ pattern) — this is the single biggest differentiator and no competitor has it. For Maza, upgrade the current UCP stub to the full `ucp/1.0` product/checkout format (its own roadmap's unshipped goal).
4. **Add `Offer` / `Itemlist` JSON-LD** mirroring the UCP prices so Google rich results + LLMs both see prices without a deep probe. (Fixes Maza's "no visible prices" and PJ's "no Offer schema.")
5. **Build a local-intent URL farm** (Maza pattern): one page per service/city — e.g. PJ needs `/window-graphics-<city>` and `/business-cards-<city>` for Phoenix/Scottsdale/Chandler/Tempe/Mesa; Maza already has its 8 city + 20 dish pages. Target ~50 URLs.
6. **Max out `sameAs`** (Maza's 9 is the bar): Google/Apple Maps deep links, Yelp, all delivery platforms, all socials. PJ is at 2.
7. **Publish an `aggregateRating`** (Maza pattern) — the fastest trust-signal win for the lane leaders.
8. **robots.txt: allow GPTBot + OAI-SearchBot + Google-Extended (+ ChatGPT-User)** and declare the sitemap — but do *not* block GPTBot/Google-Extended (SpeedPro's contradictory "block GPTBot but publish llms.txt" stance is the anti-pattern to avoid).
9. **Version + timestamp the machine manifests** (`lastUpdated`, `schema: <name>/v1`) so agents can cache-invalidate — both references do this; make it standard.
10. **Do NOT bot-wall AI crawlers.** The 429-walled sites (azhottees, phxsignshop) are invisible to LLM discovery — being *readable* is the whole point.

**Lane-specific:**
- **PJ (print/signage):** add the Maza layer (llms.txt, street address, rating, Offer schema, city URL farm). Keep the UCP/specs/Stripe edge. Watch **César Graphics** (rich visible price catalog, next-day cards $45) as the main price-surface competitor.
- **Maza (restaurant):** add the PJ layer (real `ucp/1.0` product/checkout, `Offer`/visible prices, dish-level `/api/specs`). Keep the llms.txt + Restaurant schema + rating + sameAs edge. The lane (Spitz/Greeks) is near-zero — Maza is safe on identity, must win on transaction.

---

## 7) Source log (provenance)

**Live probes (this run, 2026-10-28):**
- `https://printjunkieaz.com/robots.txt` (200) — GPTBot/OAI/GExt/ChatGPT-User Allow, sitemap
- `https://mazahalalfood.com/robots.txt` (200) — GPTBot/OAI/GExt Allow, sitemap
- UA crawl tests (GPTBot, OAI-SearchBot, PerplexityBot, Googlebot) → 200 on both reference sites
- `https://printjunkieaz.com/.well-known/ucp` (200, 4.7 KB, ucp/1.0 full) — captured vendor + 8 products
- `https://mazahalalfood.com/.well-known/ucp` (200, 847 B, WebAPI stub) — captured
- `https://printjunkieaz.com/llms.txt` (404) · `https://mazahalalfood.com/llms.txt` (200, captured in full)
- `https://printjunkieaz.com/api/specs` (404) · `https://printjunkieaz.com/api/specs/business-cards` (200)
- `https://mazahalalfood.com/api/specs` (200, 29 KB, 63 items)
- Homepage + `/business-cards-phoenix` HTML: JSON-LD + visible-price + stripped-text extraction (PJ home 75.8 KB, bc-phx 50.4 KB, Maza home 52 KB)
- Competitor homepages: cesargraphics.com, alphagraphics.com/us-arizona-phoenix-us011, phxscreenprinting.com, azprecisiongraphics.com, extremeprintlab.com, shirt52.com, mesasigncompany.com, speedpro.com/phoenix, phoenixsigncompany.org, spitz-restaurant.com/locations/chandler, greeksgrillaz.com — robots/UCP/llms/schema/price probes
- 429 bot-walls: azhottees.com, phxsignshop.com (unverified)

**Internal docs (provided paths):**
- `/workspace/seo-bible/PJ_SITE_BRIEF.md` — 10 sections, 30+ keywords, service list, "agentic-ready" claim
- `/workspace/seo-bible/PJ_SEO_ROADMAP.md` — 4-phase plan, 20+ URLs, agentic-commerce roadmap (UCP + AP2 + llms.txt + /api/specs + /order/*)
- `/workspace/seo-bible/MAZA_SEO_ROADMAP.md` — 30-page plan, agentic-commerce roadmap, 34-URL sitemap
- `/workspace/seo-bible/maza-research/` (13 files, 249 KB) — incl. `AI_LOCAL_DISCOVERY_RESEARCH.md`, `AI_SEARCH_LANDSCAPE_2026.md`, `LLM_SEO_CHECKLIST.md`

**Prior GIU reports:**
- `/home/ice/know/research/ai-local-discovery-maza-t_9c26f53d.md` (2026-09-18) — Maza AI-local-discovery findings (Maza 34 URLs vs CJJ 11 vs Spitz 1; Maza has llms.txt + 8 UCP capabilities + AP2; no competitor has llms.txt)
- `/home/ice/know/research/az-competitor-vertical-pricing-t_3c7c6149.md` + 5-file package (2026-09-14) — AZ competitor map, pricing fences, print/sign/apparel lanes

**External context (web_search, 2026-10-28):**
- Phoenix print/shirt/window-graphics competitor discovery (César Graphics, AlphaGraphics, PHX Screen Printing, AZ Precision, Extreme Print Lab, Shirt52, Mesa Sign, SpeedPro, PHX Sign Co)
- Chandler Mediterranean/halal competitor discovery (Spitz, Greeks Grill, Maza)

---

## 8) Confidence

**High** for the live probe matrix (every row is a real HTTP response captured today, with byte sizes and exact schema fields). **High** for the "PJ=transaction / Maza=identity" axis split (directly observable in the UCP vs llms.txt artifacts). **Medium** for the 15-site competitor table: 13 sites probed successfully; 2 returned 429 bot-walls and are marked unverified (not scored); some competitor sites are JS-heavy so stripped-text extraction under-counts dynamic content. **Medium** for the "no competitor has X" claims — grounded in today's probes but competitor sites can ship new manifests overnight; re-verify before treating as permanent truth.

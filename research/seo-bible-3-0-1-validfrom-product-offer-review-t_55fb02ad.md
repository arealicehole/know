# SEO Bible 3.0.1 Draft — §1.2.1 (validFrom / Product Offer completeness) Review

**Task:** t_55fb02ad (board: research, prefix [GIU])
**Prepared:** 2026-11-16 (UTC)
**Prepared by:** Geraldo v2.2 (Kanban-native)
**Scope:** Review the proposed 3.0.1 draft §1.2.1 against (a) current Google docs, (b) the locked 3.0 baseline, (c) the live PJ/Maza specimen, (d) the related `pj-gmc-ucp-eligibility-20261029.md` research, and (e) the task's four specific review questions.
**Status:** REVIEW COMPLETE — with one **BLOCKER** (draft not in my container) and one **CRITICAL FINDING** (task premise on `validFrom` is contradicted by current Google docs).

---

## ⚠️ BLOCKER — Draft not accessible from my container

The task names the draft at `/workspace/seo-bible/SEO_BIBLE_3.0.1_DRAFT.md` (or `..._DRAFT-20261116.md`). I verified this path is **not mounted in my container**:

- `ls /workspace/seo-bible/` → "No such file or directory" (only `/workspace` root exists, no `seo-bible` subdir)
- `find / -iname "*3.0.1*"` → no hits anywhere in the container
- `git ls-remote origin` (know repo) → no branch or file with "3.0.1" or "DRAFT" in the name; `git fetch` clean, no remote 3.0.1 commit
- `curl raw.githubusercontent.com/arealicehole/know/main/SEO_BIBLE_3.0.1_DRAFT.md` → **404 Not Found** (and the `-20261116.md` variant → **404**)
- The Firecrawl proxy for raw GitHub is also 403 (no API key on this keyless tier)

**I cannot read the exact draft text the task asks me to review.** The draft is a working file owned by Romulus at `/workspace/seo-bible/` on the host, not published to the know repo or mounted into my container. I will not invent the draft's text and review a phantom.

**What I DID verify (the substance the task asked for):**
- The locked 3.0 baseline §1.2 + §1.7 (full read, `/home/ice/know/SEO_BIBLE_3.0.md`, 653 lines)
- The related `pj-gmc-ucp-eligibility-20261029.md` (full read, 140 lines)
- The live PJ specimen (`.well-known/ucp` manifest, `/api/specs/*`, product page HTML) — **live probes 2026-11-16**
- Current Google docs (merchant-listing structured data + Merchant Center product data spec + 2026 spec update) — via web_search (full page content returned)

**What I need from Romulus/Frank to finish the draft-text review:** either (a) mount/copy the draft into a path I can read (e.g. `/home/ice/know/` or a kanban attachment), or (b) paste the §1.2.1 section text into a comment on this task. Everything below is the verified substance; the final "does the draft's exact wording pass" call needs the draft text.

---

## CRITICAL FINDING — Task premise on `validFrom` is contradicted by current Google docs

The task states: **"Google docs: validFrom required, priceValidUntil strongly recommended."**

**This is not what the current Google docs say.** I retrieved the full content of the authoritative page (Google Search Central, *How To Add Merchant Listing Structured Data*, `developers.google.com/search/docs/appearance/structured-data/merchant-listing`) via web_search. The Offer property tables in that page are:

| Offer property | Google's classification | What it's for |
|---|---|---|
| `price` (or `priceSpecification.price`) | **REQUIRED** | The current, active offer price |
| `priceCurrency` (or `priceSpecification.priceCurrency`) | **REQUIRED** | ISO 4217 currency, required if price is specified |
| `validFrom` | **Recommended** | Start date/time when the (sale) price is valid — ISO 8601 |
| `validThrough` | **Recommended** | End date/time when the (sale) price is valid |
| `priceValidUntil` | **Recommended** | Date after which the price will no longer be available; "Your listing may not display if the `priceValidUntil` property indicates a past date" |
| `availability` | **Recommended** | InStock/OutOfStock/etc |
| `itemCondition` | **Recommended** | New/Used/Refurbished |
| `url` | **Recommended** | Product page URL |
| `shippingDetails` | **Recommended** | Offer-level shipping |
| `hasMerchantReturnPolicy` | **Recommended** | Offer-level returns |

**Key facts from the docs:**
1. `validFrom` is **Recommended, not Required.** The only required Offer properties are `price` + `priceCurrency`.
2. `validFrom`/`validThrough`/`priceValidUntil` are **sale-duration** properties — they define the *window a (sale) price is active*, not a universal "every Offer must have validFrom" rule. The docs say: "To specify the period when a sale price is active, use … `validFrom` … `validThrough` or `priceValidUntil`."
3. `priceValidUntil` carries a **real consequence**: "Your listing may not display if the `priceValidUntil` property indicates a past date." So a stale `priceValidUntil` is worse than an absent one — it actively suppresses the listing.
4. **July 7, 2026 Merchant Center update** (the 2026 spec update, `support.google.com/merchants/answer/16989427`): added `product.category` and **sale-duration dates** (`validFrom`/`validThrough`/`priceValidUntil`) to merchant-listing structured data. This was a *new capability* (harmonizing page schema with the feed's `sale_price_effective_date`), not a *new requirement*. Google "recommends providing both ends of the window, and including the time and timezone for accuracy."
5. **Merchant Center feed attribute** `sale_price_effective_date` is only used "when the sale is time-limited; if you omit that date, the sale price always applies." So even in the feed, a validity window is optional and only for time-limited sales.

**Implication for the draft:** If the 3.0.1 draft §1.2.1 teaches that `validFrom` is *required* on every Product Offer (or implies it is a hard requirement for listing eligibility), **that is a doctrinal error** that would overstate Google's actual bar. The correct teaching is:
- `price` + `priceCurrency` are the **required** Offer core (plus `availability`/`itemCondition` strongly recommended).
- `validFrom`/`validThrough`/`priceValidUntil` are **recommended sale-duration** properties — use them when the price is time-bound (a promo, a seasonal rate, a "valid through X" window); a **stale `priceValidUntil` (past date) can suppress the listing**, so if you set it you must keep it current.
- For a local print shop with **fixed, non-promotional prices** (PJ's $65/$75/$115 card tiers), a `validFrom`/`priceValidUntil` window is **optional** — it's not what makes the Offer valid. What makes the Offer valid is a real price + currency + availability that matches the landing page.

---

## Live specimen state (verified 2026-11-16)

I probed printjunkieaz.com live (OAI-SearchBot UA, HTTP 200 at edge):

**1. UCP manifest (`.well-known/ucp`)** — `version: "2026-11-14"`, `ucp/1.0`:
- Products carry `priceRange` (min/max) + `tiers` (per-tier price) — **no `validFrom` / `priceValidUntil` in the manifest.**
- This is fine: the UCP manifest is the on-site agent transaction layer, not a Google merchant-listing structured data surface. The sale-duration properties are a *Google schema.org/merchant-listing* concern, not a UCP concern.
- Manifest `address` is still **city+state+postal only** (Tempe, AZ, 85281) — **no street number**. (The task says "PJ live manifest still city/state/postal only" — **CONFIRMED**.)

**2. `/api/specs/business-cards`** — `pricing: [{500 Cards: 65}, {1,000 Cards: 75}, {2,500 Cards: 115}]`, real `stripeUrls` (s/m/l), `orderUrl`, images, turnaround. No validFrom/priceValidUntil (correct — fixed non-promo prices).

**3. `/api/specs/custom-banners`** — `pricing: []` (**empty**), has `installation`, `turnaround`, `orderUrl`, but no fixed price. This is a **quote-based custom product** — a good test case for the "what to do when pricing is empty" rule.

**4. Product page HTML** (`/products/premium-business-cards-phoenix`):
- JSON-LD has `Product` + nested `Offer` nodes (one per tier: "500 Cards" $65, "1,000 Cards" $75, etc.)
- **Each Offer already carries `validFrom: "2025-01-01T00:00:00-07:00"` and `priceValidUntil: "2027-12-31"`** (38 occurrences of each across the page).
- So the live site **already ships** `validFrom` + `priceValidUntil` on its Product Offers. The `priceValidUntil: 2027-12-31` is **in the future** (good — won't suppress the listing). The `validFrom: 2025-01-01` is a fixed past start (harmless).
- This means: the "missing validFrom on PJ" premise in the task is **not accurate for the product page** — PJ's product-page JSON-LD *does* have validFrom. What's missing is **not** validFrom; it's that (a) the **UCP manifest** uses a different price shape (`priceRange`/`tiers`, no validity window), and (b) some products (custom-banners) have **no price at all** (`pricing: []`).

**Correction to the task's framing:** The task says "PJ live manifest still city/state/postal only" (true) but the implied gap is "missing validFrom." In reality PJ's **product page already has validFrom + priceValidUntil**. The genuine gaps are:
1. **Address completeness** (city/state/postal only — no street number) — confirmed in both manifest and (need to verify) page.
2. **Custom/quote products with empty `pricing`** (custom-banners) — no Offer price to be valid about.
3. **Manifest vs page price-shape divergence** (manifest `priceRange`/`tiers` vs page `Offer` nodes) — an agent fetching the manifest gets a different price representation than an agent fetching the product page JSON-LD.

---

## Answers to the task's four review questions

### Q1 — Does the current Google doc still require `validFrom` when an Offer has a price, and is `priceValidUntil` "strongly recommended" for listing eligibility?

**No — the premise is inverted on the first half and overstated on the second.**
- **`validFrom` is NOT required** when an Offer has a price. It is **Recommended** (sale-duration). The required Offer core is `price` + `priceCurrency` (plus `availability`/`itemCondition` strongly recommended).
- **`priceValidUntil` is Recommended** (not a "listing eligibility" gate per se) — but it carries a **real suppression risk**: a *past* `priceValidUntil` can prevent the listing from displaying. So "strongly recommended" is an understatement of the *risk of getting it wrong*, but an overstatement of its *status as a requirement*. It's a recommended property whose stale value is harmful.
- **Current as of 2026-11-16:** Yes — the July 7, 2026 spec update is the latest change to these properties and nothing newer supersedes it. The 2026 spec update (April 14, 2026) added `video_link`, `handling_cutoff_time`, `minimum_order_value` — none of which change the validFrom/priceValidUntil recommendation status.

### Q2 — Can a local print/shop service with fixed SKUs (real price, image, product URL, Stripe/checkout link) reasonably be treated as a Product Offer for AI/Shopping surfaces, or is it too service-like for Google's product rules?

**Yes, reasonably — for the fixed-SKU slice. This is consistent with the locked 3.0 §1.7 and the `pj-gmc-ucp-eligibility-20261029.md` research.**
- PJ's **fixed-SKU print products** (business-cards $65/75/115, loyalty-cards, promo-flyers $125/275/525, store-hours $125) are **product-shaped**: stable price, image, product URL, Stripe payment link. These **can** be treated as Product Offers for AI/Shopping surfaces — and the live site already marks them up as `Product` + `Offer` in JSON-LD.
- **But** the eligibility bar from the research (and §1.7) is: **SKU-shaped only.** Custom-quote jobs (custom-banners, `pricing: []`), pure install labor, and personalized goods are **poor fits** for product-feed rules. Google's MC UCP prep explicitly marks **services / personalized goods / installation bundles / special shipping** as **ineligible for agentic checkout**.
- **Doctrine nuance to preserve:** the Bible should **not** collapse "local print shop" into "Product Offer." It should teach the **two-lane** rule (already in §1.7): fixed-SKU catalog → Product Offer eligible; custom/install/service → on-site UCP + handoff (Stripe links, phone, quote), **not** a Product Offer. The 3.0.1 §1.2.1 addition must not accidentally broaden §1.7's "Partial — only SKU-shaped" into "all local shops are Product Offers."

### Q3 — Does the proposed 3.0.1 rule match the locked 3.0 doctrine and the related `pj-gmc-ucp-eligibility-20261029.md` research without adding a new commercial/pricing claim?

**Cannot be fully judged without the draft text** (blocker above). But the **alignment criteria** the draft must meet:
- **§1.2 (locked 3.0):** "Do: … `Offer`/`price` when true." The 3.0.1 §1.2.1 must **refine** this (add the Offer-completeness sub-rule) without **contradicting** it. The locked §1.2 framing is "schema-as-entity, quotable facts, not a ranking cheat code" — the 3.0.1 addition must stay in that framing (completeness for accuracy/disambiguation, **not** "adding validFrom makes you rank higher in chat").
- **§1.7 (locked 3.0) + `pj-gmc-ucp-eligibility-20261029.md`:** The "Partial — only SKU-shaped" GMC rule and the "do not conflate on-site UCP with Google UCP checkout" rule must be **preserved intact**. The 3.0.1 §1.2.1 is about *schema completeness on product pages*, not about *GMC/UCP eligibility* — it must not blur those two concerns or imply that complete Product Offer schema = GMC/UCP-checkout eligibility.
- **No commercial/pricing claim:** The rule must be **doctrine** (how to mark up an Offer completely and honestly), **not** a commercial offer. It must not introduce any "AVP price point," "we do X for $Y," or "complete schema gets you into AI Mode" guarantee. Per the research pack §5 (no-pricing rule) and the 3.0 lock rules, any line that reads like a placement guarantee or a price for our services is a **merge failure**.
- **My pre-judgment (pending draft text):** if the draft says "add validFrom/priceValidUntil to every Offer" as a flat rule, it **overreaches** the Google docs (they're recommended sale-duration, not required) and risks implying a completeness→placement causation the Bible elsewhere explicitly disclaims ("schema is not proven as a chat-citation ranker by itself"). The rule should be conditional: "When the price is time-bound, add validFrom/validThrough/priceValidUntil with a current future end; never ship a stale priceValidUntil. For fixed non-promo prices, validFrom/priceValidUntil are optional."

### Q4 — Should the rule cover both (a) complete Offer fields when a fixed product price exists and (b) honest non-product/quote/custom paths when pricing is empty?

**Yes — absolutely. This is the correct shape of the rule, and it matches the live evidence.**
- **(a) Fixed product price exists** (business-cards, loyalty-cards, promo-flyers, store-hours): the Offer must be **complete** — `price`, `priceCurrency`, `availability` (InStock), `itemCondition` (NewCondition), `url` (order/product URL), and — **when the price is time-bound** — `validFrom`/`validThrough`/`priceValidUntil` (with a current future end). This is the "complete Offer fields" half.
- **(b) Pricing is empty / quote / custom** (custom-banners `pricing: []`, window-wraps, install labor): the rule must **not** force a fake Product Offer. Instead it should teach the **honest non-product path**: mark the entity as a **Service** (or `Service`/`Offer` with `priceSpecification` omitted and an explicit "quote required" / `potentialAction` = `ContactAction`/`ReserveAction`), point to the **on-site UCP + Stripe/phone/quote handoff**, and **do not** ship an `Offer` with a `price` that isn't real. This is the "honest non-product/quote/custom path" half.
- **Why this two-branch rule is right:** it directly reflects the **two-surface model** (§1.6) and the **§1.7 GMC two-lane** doctrine, and it matches what the live PJ site actually is (fixed SKUs with complete Offers + custom products with empty pricing). A rule that only covers (a) would tell an auditor to fabricate a price on custom-banners; a rule that only covers (b) would abandon the fixed-SKU lane where the Product Offer **is** the right markup.
- **One addition I'd recommend:** the rule should name the **manifest-vs-page price-shape divergence** as a known gotcha — the UCP manifest uses `priceRange`/`tiers` while the product page JSON-LD uses `Offer` nodes. An agent that fetches the manifest gets a different price representation than one that fetches the page. The rule should say: keep the two **consistent** (same prices, same currency) even if the shape differs, and treat the **product page JSON-LD** as the surface that must carry the complete Offer (because that's what Google's merchant-listing structured data reads), while the **UCP manifest** is the agent-transaction surface.

---

## Structured claims & evidence

| claim_id | statement | evidence_refs | confidence | status |
|----------|-----------|---------------|------------|--------|
| C1 | The 3.0.1 draft is not accessible from my container (path not mounted, not on know GitHub, 404 on raw). | live probes: `ls /workspace/seo-bible/`, `find / -iname '*3.0.1*'`, `git ls-remote origin`, `curl raw.githubusercontent` | high | **verified (blocker)** |
| C2 | Google's merchant-listing structured data docs classify `validFrom` as **Recommended**, not Required. The required Offer core is `price` + `priceCurrency`. | Google Search Central, merchant-listing structured data (full page content via web_search), retrieved 2026-11-16 | high | **verified** |
| C3 | `priceValidUntil` is Recommended but carries a real risk: "Your listing may not display if the `priceValidUntil` property indicates a past date." | same Google doc, Offer recommended-properties table | high | **verified** |
| C4 | The July 7, 2026 Merchant Center spec update **added** sale-duration dates (validFrom/validThrough/priceValidUntil) as a new capability, not a new requirement. | `support.google.com/merchants/answer/16989427` (2026 spec update) + third-party confirmations (wpconsults, veonib) | high | **verified** |
| C5 | PJ's live product-page JSON-LD **already** carries `validFrom: 2025-01-01` + `priceValidUntil: 2027-12-31` on every Offer node. | live probe `/products/premium-business-cards-phoenix` (38× validFrom, 38× priceValidUntil) | high | **verified** |
| C6 | PJ's UCP manifest uses `priceRange` + `tiers` (no validity window) and its address is city/state/postal only (no street number). | live probe `/.well-known/ucp` | high | **verified** |
| C7 | PJ's custom-banners spec has `pricing: []` (empty) — a quote/custom product with no fixed price. | live probe `/api/specs/custom-banners` | high | **verified** |
| C8 | Fixed-SKU print products are reasonably Product-Offer-eligible; custom/install/service are not (per locked §1.7 + research). | locked 3.0 §1.7; `pj-gmc-ucp-eligibility-20261029.md` §3 (services/personalized/install ineligible) | high | **verified (doctrine)** |
| C9 | The 3.0.1 rule should be **two-branch**: (a) complete Offer fields for fixed-price products, (b) honest non-product/quote/custom path for empty-pricing products. | live evidence (C5/C6/C7); §1.6 two-surface; §1.7 two-lane | high | **verified (recommendation)** |
| C10 | Whether the draft's *exact wording* meets the no-pricing/no-guarantee/validFrom-recommended criteria **cannot be judged without the draft text.** | blocker C1 | high | **open (needs draft)** |

## Evidence index

| evidence_id | source_label | source_url/path | provenance | support note |
|-------------|--------------|-----------------|------------|--------------|
| E1 | Google merchant-listing structured data | `developers.google.com/search/docs/appearance/structured-data/merchant-listing` | full page content via web_search, 2026-11-16 | Offer required vs recommended property tables; validFrom/validThrough/priceValidUntil under Recommended; priceValidUntil stale-date suppression warning |
| E2 | Google Merchant Center 2026 spec update | `support.google.com/merchants/merchants/answer/16989427` | web_search snippet, 2026-11-16 | April 14 2026 update: video_link, handling_cutoff_time, minimum_order_value; no change to validFrom requirement status |
| E3 | Third-party confirmations of July 7 2026 sale-duration addition | wpconsults.com, veonib.com | web_search snippets | Both confirm validFrom/validThrough/priceValidUntil are **recommended** sale-duration, not required |
| E4 | Locked 3.0 baseline | `/home/ice/know/SEO_BIBLE_3.0.md` (653 lines) | read in full this run | §1.2 Schema Strategy, §1.6 two-surface, §1.7 GMC eligibility — the doctrine the 3.0.1 must align with |
| E5 | Related research | `/home/ice/know/research/pj-gmc-ucp-eligibility-20261029.md` (140 lines) | read in full this run | GMC/UCP two-lane rule; services/personalized/install ineligible; do/don't-say doctrine |
| E6 | Live PJ UCP manifest | `https://printjunkieaz.com/.well-known/ucp` | live probe OAI-SearchBot UA, 2026-11-16, HTTP 200 | priceRange/tiers shape; no validFrom; address city/state/postal only |
| E7 | Live PJ specs | `https://printjunkieaz.com/api/specs/business-cards` + `/api/specs/custom-banners` | live probe, 2026-11-16 | fixed tiers (65/75/115) + stripeUrls; custom-banners pricing=[] |
| E8 | Live PJ product page HTML | `https://printjunkieaz.com/products/premium-business-cards-phoenix` | live probe, 2026-11-16 | JSON-LD Product+Offer with validFrom 2025-01-01 + priceValidUntil 2027-12-31 (38× each) |

## Gaps / unresolved

1. **Draft text unavailable** — the single blocker. Everything else is verified; the final "does the draft's exact wording pass" call needs the §1.2.1 text. **Action:** Romulus copies the draft to `/home/ice/know/` (or attaches it to this task) and re-dispatches, or pastes §1.2.1 into a comment.
2. **Maza specimen not probed this run** — the task names "PJ/Maza live" but I only probed PJ (Maza is the identity-surface reference; PJ is the transaction-surface reference). The §1.2.1 rule is about **product Offers** (PJ's lane), so PJ is the relevant specimen. Maza (a restaurant) is not a Product-Offer case. If the draft also makes a claim about restaurant offers, Maza should be probed too.
3. **Street-address completeness** — the task says the rule should address "address completeness." I confirmed the manifest address is city/state/postal only (no street number) but did **not** verify whether the product-page JSON-LD `PostalAddress` has a street number. If the rule mandates a full `PostalAddress`, the live site may need a street number added (or an honest "by appointment, no public street address" note if that's the reality).

## Confidence (report level)

- **level:** **high** (on the verified substance); **medium** (on the final draft-wording verdict, blocked by C1)
- **rationale:** Every doctrinal and live-evidence claim (C2–C9) is grounded in sources I retrieved or probed this run — the Google docs via web_search (full page content), the live PJ specimen via direct curl (HTTP 200, real JSON), and the locked 3.0 + research via full local read. The one item I could **not** verify is the draft's exact text (C1/C10), and I have said so plainly rather than reviewing a phantom. The critical finding (validFrom is Recommended, not Required) is a direct, quotable correction to the task's premise and is the single most important thing for Frank to know before locking 3.0.1.

## Recommended next actions

1. **Romulus:** correct the §1.2.1 premise — `validFrom` is **Recommended sale-duration**, not Required. Rewrite the rule as: "When the price is time-bound, add `validFrom`/`validThrough`/`priceValidUntil` (ISO 8601, current future end). Never ship a stale `priceValidUntil` — a past date suppresses the listing. For fixed non-promo prices, these are optional; the required Offer core is `price` + `priceCurrency` (+ `availability`/`itemCondition` recommended)."
2. **Romulus:** make the rule **two-branch** (fixed-SKU complete Offer vs honest non-product/quote path) — do not force a fake price on custom/quote products.
3. **Romulus:** preserve §1.7's two-lane GMC rule and §1.6 two-surface framing intact; do not let §1.2.1 imply "complete schema = GMC/UCP-checkout eligibility" or any placement guarantee.
4. **Romulus:** make the draft accessible (copy to `/home/ice/know/` or attach to this task) so I can complete the draft-text review (C10).
5. **Frank:** if the rule will mandate a full `PostalAddress`, decide whether PJ needs a street number added or an honest "by appointment" note (gap 3).
6. **Never:** write any placement guarantee ("complete schema gets you into AI Mode") or AVP price point into §1.2.1 — both are merge failures per the 3.0 lock rules and the no-pricing doctrine.

---

*Prepared by Geraldo v2.2. Single durable copy: `/home/ice/know/research/seo-bible-3-0-1-validfrom-product-offer-review-t_55fb02ad.md`. No double-publish. Review complete on verified substance; draft-text verdict blocked by container access (C1). Critical finding: task premise "validFrom required" is contradicted by current Google docs (Recommended, not Required).*
